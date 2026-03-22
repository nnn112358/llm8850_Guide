# dev_skill.md — LLM-8850 / Raspberry Pi 5 開発デバッグ経験の蓄積

このドキュメントは、`rpi5`上で「ドライバインストール → Whisper / CosyVoice2（OpenAI API）の動作確認 → 音色クローン（clone） → ディスククリーンアップ」を一通り行った際のトラブルシューティング過程を、再利用可能なチェックリストとして整理したものです。今後の再現やトラブルシューティングに役立ちます。

## 0. 環境の前提条件（今回の実機テスト）

- Host：Raspberry Pi 5（Debian bookworm、aarch64）
- アクセラレータカード：LLM-8850（PCIe）
- 接続方法：`ssh rpi5` パスワードなしログイン；`sudo` パスワード不要
- プロキシ方針：
  - GitHubはプロキシ経由：`127.0.0.1:7890`
  - HuggingFaceのダウンロードは`hf-mirror.com`を使用し、できるだけプロキシを通さない（プロキシ帯域の節約）

> 以下の例では`PROXY=http://127.0.0.1:7890`を使用しています。プロキシが異なる場合は適宜置き換えてください。

## 1. ネットワーク/プロキシ：「コマンドごとに有効化」する2種類の環境変数

`~/.gitconfig`にグローバルで`http.proxy`を設定しないでください。そうしないとプロキシが不要なダウンロードまでプロキシ経由になりやすくなります（特に大規模モデルファイル）。

### 1.1 GitHub（プロキシ経由）

```bash
PROXY=http://127.0.0.1:7890
env http_proxy="$PROXY" https_proxy="$PROXY" all_proxy="$PROXY" \
  git clone https://github.com/xxx/yyy.git
```

### 1.2 HuggingFace（プロキシなし、hf-mirror.com経由）

```bash
env no_proxy='*' http_proxy= https_proxy= all_proxy= HTTP_PROXY= HTTPS_PROXY= ALL_PROXY= \
  curl -L -o /tmp/file.bin 'https://hf-mirror.com/<org>/<repo>/resolve/main/<path>'
```

経験から得た結論：

- `hf-mirror.com`では`curl .../resolve/main/...`の直リンクダウンロードが`git lfs`より安定（後述「LFSの落とし穴」参照）。
- `http_proxy/https_proxy/all_proxy`を明示的に空にした上で`no_proxy='*'`を追加すれば、ほぼ確実に「プロキシを通さない」ことができます。

## 2. ドライバ/デバイスのトラブルシューティング：`/dev/axcl_host`が存在しない

典型的なエラー（`axcl-smi`またはサービスログから）：

```text
axcl init fail, ret = 0x8030010b
open /dev/axcl_host fail, errno: 2 No such file or directory
```

優先的に以下の順序で調査：

1) PCIeがカードを認識しているか：

```bash
lspci | grep -Ei "Axera|0650" || true
```

2) ドライバ（DKMS）が現在のカーネル用にビルドされているか（特にカーネルアップグレード後）：

```bash
uname -r
dkms status | grep -i axcl || true
```

3) 現在のカーネル用にモジュールを再ビルドしてロードし、ノードの作成を確認：

```bash
sudo /usr/sbin/dkms autoinstall -k "$(uname -r)"
sudo modprobe axcl_host
ls -l /dev/axcl_host
axcl-smi
```

それでも解決しない場合は、電源を切って再挿入、電源供給の確認、または`axclhost`パッケージの再インストールを検討してください。

## 3. APTソースとコアパッケージ（axclhost + llm8850）

経験から得た結論：`axclhost`ソースは「ドライバと基本ツール」を提供し、`bookworm llm8850`ソースは「StackFlowのモデル/サービス（llm-*）」を提供します。

```bash
sudo wget -qO /etc/apt/keyrings/StackFlow.gpg https://repo.llm.m5stack.com/m5stack-apt-repo/key/StackFlow.gpg

echo 'deb [signed-by=/etc/apt/keyrings/StackFlow.gpg] https://repo.llm.m5stack.com/m5stack-apt-repo axclhost main' \
  | sudo tee /etc/apt/sources.list.d/axclhost.list

echo 'deb [signed-by=/etc/apt/keyrings/StackFlow.gpg] https://repo.llm.m5stack.com/m5stack-apt-repo bookworm llm8850' \
  | sudo tee /etc/apt/sources.list.d/llm8850.list

sudo apt update
```

## 4. StackFlowサービス：クイックセルフチェックと500エラーのトラブルシューティング

### 4.1 サービスセルフチェック（最短経路）

```bash
systemctl is-active llm-sys llm-cosy-voice llm-openai-api
curl -s http://127.0.0.1:8000/v1/models
```

以下のような表示が期待されます：

```json
{"data":[{"id":"CosyVoice2-0.5B-axcl", ...}],"object":"list"}
```

### 4.2 CosyVoice2 TTSリクエストには明示的に`voice`を記述

実機テストでは、`voice`を記述しないと500エラーが発生しました（サーバー側が空のbodyを返し、クライアント側でJSON解析時に`Expecting value`等のエラー）。

推奨リクエストテンプレート：

```bash
curl -X POST http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{"model":"CosyVoice2-0.5B-axcl","voice":"prompt_data","response_format":"wav","input":"こんにちは、TTSテストです。"}' \
  -o /tmp/cosyvoice2_test.wav
```

### 4.3 ワンクリック「緊急再起動」

```bash
sudo systemctl restart llm-sys llm-cosy-voice llm-openai-api
```

併せてログを確認：

```bash
journalctl -u llm-openai-api -n 200 --no-pager
journalctl -u llm-cosy-voice -n 200 --no-pager
```

## 5. Whisper（`whisper.axcl`）：「ビルドと実行」とは何か

ここでの「ビルドと実行」とは：

1) ソースコードの取得（サブモジュール/依存関係を含む）
2) 本機（rpi5）でのコンパイルによる実行ファイルと依存ライブラリの生成
3) プロジェクトの規約に従い`.axmodel`等のモデルファイルを実行ディレクトリに配置
4) 正しいワーキングディレクトリ（CWD）から実行ファイルを起動

典型的な流れ（概要）：

```bash
cd ~/rsp/whisper.axcl
./build.sh
cd install
./whisper -w ../demo.wav
```

### 5.1 `.axmodel`の入手元：HFの`M5Stack/whisper-small-axmodel`

オフラインドキュメントでは「モデルファイルの入手元」が明記されていないことが多く、実際にはHF（ミラー推奨）から直接`install/models/`にダウンロードできます。

例（`hf-mirror.com`使用、プロキシなし）：

```bash
cd ~/rsp/whisper.axcl/install/models
env no_proxy='*' http_proxy= https_proxy= all_proxy= \
  curl -L -O 'https://hf-mirror.com/M5Stack/whisper-small-axmodel/resolve/main/whisper-small.axmodel'
```

### 5.2 実行ディレクトリ（CWD）が重要

実機テストでは`whisper`実行ファイルが「カレントディレクトリ」から`./models/...`を検索し、同ディレクトリの設定/リソース（`t2s.json`等）に依存する可能性があります。

経験的なアプローチ：常に`install/`ディレクトリから実行し、不足するものがあれば`install/`にリソースを補充（またはプロジェクトの説明に従い相対パスに配置）。

### 5.3 モデルが2部重複する問題（今回実際に遭遇）

Whisperプロジェクトではモデルディレクトリが2部になりやすい：

- `~/rsp/whisper.axcl/models`（ソースコードディレクトリ下）
- `~/rsp/whisper.axcl/install/models`（実行ディレクトリ下）

実際の実行には`install/models`の1部のみ必要で、もう1部は「誤ダウンロード/誤コピー」によるスペースの無駄です。

調査コマンド：

```bash
du -sh ~/rsp/whisper.axcl/models ~/rsp/whisper.axcl/install/models 2>/dev/null
```

## 6. CosyVoice2音色クローン（`CosyVoice2-scripts`）のいくつかの大きな落とし穴

### 6.1 `git lfs`はhf-mirrorで失敗する可能性あり：大きなファイルの取得に使わない

現象：`git lfs pull`が`cas-bridge.xethub.hf-mirror.org`のようなドメインにアクセスし、DNS/ネットワーク環境の不一致で失敗する。

より安定したアプローチ：

- リポジトリはLFSをスキップしてクローン：

```bash
GIT_LFS_SKIP_SMUDGE=1 git clone https://hf-mirror.com/FunAudioLLM/CosyVoice2.git CosyVoice2-scripts
```

- 必要なLFS大容量ファイルは`curl .../resolve/main/...`で正確に取得

### 6.2 `asset/zh_woman1.wav`だけではない：フロントエンドONNX（大容量）が2つ不足

`scripts/process_prompt.py`がプロンプトデータを生成する際、サンプル音声に加えて以下も必要：

- `frontend-onnx/campplus.onnx`（約27MB）
- `frontend-onnx/speech_tokenizer_v2.onnx`（約500MB）

ミラーの直リンクでダウンロードすることを推奨（プロキシなし）：

```bash
cd ~/rsp/CosyVoice2-scripts
env no_proxy='*' http_proxy= https_proxy= all_proxy= \
  curl -L -o asset/zh_woman1.wav \
  'https://hf-mirror.com/FunAudioLLM/CosyVoice2/resolve/main/asset/zh_woman1.wav'

env no_proxy='*' http_proxy= https_proxy= all_proxy= \
  curl -L -o frontend-onnx/campplus.onnx \
  'https://hf-mirror.com/FunAudioLLM/CosyVoice2/resolve/main/frontend-onnx/campplus.onnx'

env no_proxy='*' http_proxy= https_proxy= all_proxy= \
  curl -L -o frontend-onnx/speech_tokenizer_v2.onnx \
  'https://hf-mirror.com/FunAudioLLM/CosyVoice2/resolve/main/frontend-onnx/speech_tokenizer_v2.onnx'
```

### 6.3 submoduleの初期化が必須（`wetext`）

初期化しないと`tagger.fst`等のリソースが存在しないエラーが発生します。

```bash
cd ~/rsp/CosyVoice2-scripts
git submodule update --init --recursive
```

### 6.4 パッケージインストール時のトラブル（ミラー指定・キャッシュ無効化）

`uv` は `pip` の設定ファイル（`~/.config/pip/pip.conf` 等）を読まないため、`piwheels` 混在による hash mismatch 問題は発生しません。

ネットワークが不安定な場合やキャッシュが原因でエラーが出る場合は、ミラーとキャッシュ無効化を明示的に指定してください：

```bash
cd ~/rsp/CosyVoice2-scripts
uv pip install --no-cache --index-url https://mirrors.cloud.tencent.com/pypi/simple -r requirements.txt
```

> **補足:** `uv pip` では `--no-cache-dir` ではなく `--no-cache` を使用します。

### 6.5 voiceの生成とCosyVoice2-0.5B-axclへの組み込み

プロンプトデータの生成（例）：

```bash
cd ~/rsp/CosyVoice2-scripts
python3 scripts/process_prompt.py \
  --input_wav asset/zh_woman1.wav \
  --input_text 'CosyVoice2へようこそ。' \
  --output zh_woman1
```

StackFlowのCosyVoice2モデルディレクトリ（`/opt/m5stack/data`が規約）への組み込み：

```bash
sudo cp -a zh_woman1 /opt/m5stack/data/CosyVoice2-0.5B-axcl/zh_woman1
sudo systemctl restart llm-sys llm-openai-api
```

その後、OpenAI APIでテスト：

```bash
curl -X POST http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{"model":"CosyVoice2-0.5B-axcl","voice":"zh_woman1","response_format":"wav","input":"これはクローン音色のテストです。"}' \
  -o /tmp/zh_woman1_test.wav
```

## 7. ディスクとログ：どれだけ節約でき、何を優先的に削除するか（今回の実機データ）

今回rpi5上の主要ディレクトリの容量（2026-03-01実測）：

- `/opt/m5stack/data/CosyVoice2-0.5B-axcl`：約**1.4G**（推論に必要、削除非推奨）
- `~/rsp/whisper.axcl`：約**747M**（ソースコード/成果物/モデルを含む。ソースコードキャッシュは必要に応じてクリーンアップ可能だが、`install/`内の実行に必要なファイルは変更しないことを推奨）
- `~/rsp/CosyVoice2-scripts`：約**2.1G**（主に「クローンツールチェーン + ONNX + Python依存関係」で、**クローン完了後は削除可能**）
- `~/.cache/uv`：約**191M**（削除可能、`uv cache clean` でも可）
- `journalctl`：約**99M**（制御可能）

よく使うクリーンアップコマンド：

```bash
# 1) uvキャッシュ
uv cache clean

# 2) systemd journal
sudo journalctl --vacuum-size=100M

# 3) APTキャッシュ + 不要な依存関係
sudo apt clean
sudo apt autoremove --purge -y

# 4) 任意：クローンツールチェーンの削除（voiceを/opt/m5stack/dataにコピー済みの場合）
rm -rf ~/rsp/CosyVoice2-scripts
```

## 8. 1ページトラブルシューティングフロー（ブックマーク推奨）

1) `lspci`で`Axera ... 0650`が確認できる
2) `ls -l /dev/axcl_host`が存在する；`axcl-smi`が正常
3) `systemctl is-active llm-sys llm-openai-api`が`active`
4) `curl http://127.0.0.1:8000/v1/models`に`CosyVoice2-0.5B-axcl`がある
5) TTSリクエストには`voice`が必須；500エラーの場合はまず3サービスを再起動してからログを確認
6) 大容量ファイルのダウンロードは`hf-mirror.com/resolve/main/...`の直リンク優先、`git lfs`の不安定性を回避
7) 容量不足の場合は`~/rsp/CosyVoice2-scripts`の削除、`uv cache clean`、`journalctl`の圧縮を優先
