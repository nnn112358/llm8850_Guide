# LLM-8850 Card（M5Stack AIアクセラレータカード）

## 目次

- [お使いの環境について](#お使いの環境について)
- [ディレクトリ構成](#ディレクトリ構成)
- [Raspberry Pi 5：ハードウェア取り付け](#raspberry-pi-5ハードウェア取り付け)
- [Raspberry Pi 5：環境とドライバのインストール（AXCL Host）](#raspberry-pi-5環境とドライバのインストールaxcl-host)
- [Windows：環境とドライバのインストール](#windows環境とドライバのインストール)
- [共通ツールとコマンド](#共通ツールとコマンド)
  - [axcl-smi（デバイス管理/診断）](#axcl-smiデバイス管理診断)
  - [AXCL FFmpeg（ハードウェアデコード/トランスコード）](#axcl-ffmpegハードウェアデコードトランスコード)
  - [Docker（任意）](#docker任意)
  - [Git LFS（任意）](#git-lfs任意)
  - [systemctl（StackFlowサービス管理）](#systemctlstackflowサービス管理)
  - [プロキシ/ミラー（ネットワーク）](#プロキシミラーネットワーク)
  - [ディスクとログのクリーンアップ](#ディスクとログのクリーンアップ)
- [一般的な実行パターン（モデル/アプリケーション）](#一般的な実行パターンモデルアプリケーション)
  - [ビジョンモデルクイック体験（axcl_demo.zip）](#ビジョンモデルクイック体験axcl_demozip)
  - [Whisper（whisper.axcl）オフライン音声テキスト変換](#whisperwhisperaxclオフライン音声テキスト変換)
  - [SenseVoice（Python）音声認識](#sensevoicepython音声認識)
  - [OpenAI互換API（テキスト対話）](#openai互換apiテキスト対話)
  - [OpenAI互換API（CosyVoice2テキスト音声変換）](#openai互換apicosyvoice2テキスト音声変換)
- [NPUサンプルとツールチェーン](#npuサンプルとツールチェーン)
- [NPUベンチマーク（性能参考）](#npuベンチマーク性能参考)
- [FAQ（よくある質問）](#faqよくある質問)
- [オフラインドキュメント索引（トピック別）](#オフラインドキュメント索引トピック別)

## お使いの環境について

- **LLM-8850 Card**：PCIe経由でHost（通常はRaspberry Pi 5）に接続するAIアクセラレータカード。
- **識別特徴**：Raspberry Pi 5上で`lspci`を実行すると`Axera Semiconductor Co., Ltd Device 0650`が表示される（例は後述）。
- **ドライバ/ランタイム**：ドキュメントでは`axclhost`（AXCL Hostサイドドライバパッケージ）を核として、インストール後に`axcl-smi`等のツールでデバイス情報と状態を確認可能。

## ディレクトリ構成

- `README.md`：本整理ドキュメント（今お読みいただいているもの）。
- `offline/`：Webページから直接取得した元のMarkdown（ナビゲーション/重複あり、必要に応じて原文との照合に使用）。

## Raspberry Pi 5：ハードウェア取り付け

> 取り付け前に必ず電源を切断してください：**通電状態での挿抜は不可**。

Raspberry Pi 5にLLM-8850アクセラレータカードを取り付ける際、公式ドキュメントには2つの一般的な方法が記載されています：

1) **M5Stack LLM-8850 PiHat**拡張ボードの使用
   - 電源推奨：変換ボードの**PD電源ポート**からシステム全体に給電
   - 給電能力：**> 9V@3A（27W）**

2) Raspberry Pi公式**M.2 HAT+**拡張ボードの使用
   - 電源推奨：**DC 5V@3A**のスイッチング電源を推奨（**非PDプロトコル**）
   - 説明：PD電源を使用した場合、プロトコルのネゴシエーションにより最大出力が得られず、異常が発生する可能性あり

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_hardware_install.md`

## Raspberry Pi 5：環境とドライバのインストール（AXCL Host）

### 1) システムとEEPROMの更新（Raspberry Pi 5推奨）

```bash
sudo apt update && sudo apt full-upgrade

sudo rpi-eeprom-update
```

EEPROMの日付が**2023-12-06より前**の場合：

```bash
sudo raspi-config
# Advanced Options > Bootloader Version > Latest

sudo rpi-eeprom-update -a
sudo reboot
```

### 2) PCIeデバイスが認識されているか確認

```bash
lspci
```

出力例（キーとなる行）：

```text
Multimedia video controller: Axera Semiconductor Co., Ltd Device 0650 (rev 01)
```

### 3) ドライバとランタイムライブラリのインストール（axclhost）

> ヒント：ドライバのインストールにはビルド環境（`gcc`/`make`/`patch`/`linux-header-$(uname -r)`等）への依存がある場合があります。事前に準備するか、インストール時にネットワークが利用可能であることを確認してください。

ソフトウェアソースを追加してインストール：

```bash
sudo wget -qO /etc/apt/keyrings/StackFlow.gpg https://repo.llm.m5stack.com/m5stack-apt-repo/key/StackFlow.gpg
echo 'deb [signed-by=/etc/apt/keyrings/StackFlow.gpg] https://repo.llm.m5stack.com/m5stack-apt-repo axclhost main' \
  | sudo tee /etc/apt/sources.list.d/axclhost.list

sudo apt update
sudo apt install -y dkms axclhost
```

### 3.5) StackFlow（LLM8850）ソフトウェアソース（OpenAI API / CosyVoice2等）

`axclhost`ソースは主にドライバと基本ツールを提供します。`llm-openai-api`、CosyVoice2、Qwen等の**StackFlowモデル/サービスパッケージ**もインストールする場合は、`bookworm/llm8850`コンポーネントの追加も必要です：

```bash
# 前のステップでaxclhostをインストール済みの場合、このkeyは通常既に存在します。再実行しても問題ありません
sudo wget -qO /etc/apt/keyrings/StackFlow.gpg https://repo.llm.m5stack.com/m5stack-apt-repo/key/StackFlow.gpg

echo 'deb [signed-by=/etc/apt/keyrings/StackFlow.gpg] https://repo.llm.m5stack.com/m5stack-apt-repo bookworm llm8850' \
  | sudo tee /etc/apt/sources.list.d/llm8850.list

sudo apt update
```

インストール完了後、新しくインストールされた実行ファイルをすぐに使用するため、ターミナル環境を更新します：

```bash
source /etc/profile
```

### 4) axcl-smiでドライバとデバイスの状態を確認

```bash
axcl-smi
```

デバイスリスト/ファームウェア/ドライババージョン/CMM使用状況が表示されれば正常です（例は`AXCL-SMI`セクション参照）。

### 5) ドライバのアンインストール

```bash
sudo apt remove axclhost
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_software_install.md`

## Windows：環境とドライバのインストール

### 1) システム要件

- **Windows 10 64ビット（22H2）**、**Windows 11 64ビット（>= 23H2）**のみサポート
- システムスリープの無効化を推奨（スリープを「なし」に設定）
- セキュリティソフトの無効化を推奨（誤検出回避のため）
- **Visual Studio 2022ランタイムライブラリ** `VC_redist.x64.exe`のインストール（管理者権限が必要）

### 2) デバイスの認識

シャットダウン・電源切断後にカードを挿入して起動：

- `Win+R` → `devmgmt.msc`（デバイスマネージャー）
- 「ほかのデバイス」に不明な「マルチメディアビデオコントローラー」が表示される
- プロパティで`VENDOR`が`1F4B`であることを確認

### 3) ドライバのインストール/アンインストール

- 管理者権限でリリースパッケージを実行：`axcl_win64_setup_Vx.x.x_yyyymmdd_NOxxxx.exe`
  - 自動インストール内容：ドライバ、DLL（インポートライブラリ含む）、実行ファイル（`axcl-smi.exe`等）、サンプルソースコード
  - 全選択でのインストールを推奨、インストールパスに注意（ヘッダファイル、動的ライブラリ、アンインストーラー等を含む）
  - インストール完了後にPCを再起動
- 再起動後、デバイスマネージャーの「システムデバイス」に`Axera NPU Accelerator Device`が表示される
- アンインストール：インストールパス`AXCL`に移動し、`uninst.exe`を実行

### 4) SDK/サンプルのビルド（任意）

ドキュメントに記載の依存関係：

- Visual Studio Community 2026（コンポーネント：C++によるデスクトップ開発）
- CMake（> 3.20）
- ninja（サンプルではv1.31.1を推奨、システムPathに追加）
- Python3（condaも利用可能）

ビルドスクリプトと実行サンプル：

```bat
:: AXCL\axcl\scripts内で実行
build_win64.cmd

:: ビルド成果物ディレクトリで実行（モデルパスは適宜置換してください）
axcl_run_model.exe -m yolo11s.axmodel -r 10
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_windows_install.md`

## 共通ツールとコマンド

### axcl-smi（デバイス管理/診断）

用途（ドキュメント記載）：

- デバイスモデル/ファームウェア/ドライババージョン
- CPU/NPU使用率
- メモリ使用量、CMM（メディアメモリ）使用量
- SoCジャンクション温度
- デバイス側ログのダウンロード、デバイス側シェルの実行、デバイスの再起動等

よく使うコマンド例：

```bash
# デバイス概要（最もよく使う）
axcl-smi

# CMM使用量の確認
axcl-smi info --cmm -d 0

# VENCモジュールproc確認（他のモジュールも同様：vdec/ivps/ive/...）
axcl-smi proc --venc -d 0

# デバイスCPU周波数の設定（対応値：1200000/1400000/1700000のみ）
axcl-smi set -f 1200000 -d 0

# ログのダウンロード（デフォルト推奨 -1：全ログ）
axcl-smi log -t -1 -d 0

# デバイス側コマンドの実行（特殊文字を含む場合はクォートで囲むことを推奨）
axcl-smi sh "ls -l" -d 0

# デバイスの再起動（確認プロンプトあり）
axcl-smi reboot -d 0
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_axcl_smi.md`

### AXCL FFmpeg（ハードウェアデコード/トランスコード）

ドキュメントに記載のパス規約：

- 動的ライブラリ：`/usr/lib/axcl/ffmpeg`
- 実行ファイル：`/usr/bin/axcl/ffmpeg`

実行前に動的ライブラリの検索パスを設定する必要があります：

```bash
export LD_LIBRARY_PATH="/usr/lib/axcl/ffmpeg:$LD_LIBRARY_PATH"
```

例（h264ハードウェアデコードからyuv出力）：

```bash
/usr/bin/axcl/ffmpeg/ffmpeg -c:v h264_axdec -i input.mp4 -f rawvideo -pix_fmt yuv420p output.yuv
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_ffmpeg.md`

### Docker（任意）

Raspberry PiへのDockerインストール（Debian系）：

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER
newgrp docker
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_docker.md`

### Git LFS（任意）

多くのモデルリポジトリがGit LFSを使用しています：

```bash
sudo apt update
sudo apt install -y git-lfs
git lfs install
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md`

### systemctl（StackFlowサービス管理）

StackFlowの**OpenAI互換API / LLM / TTS / ASR**は、多くの場合systemdサービスとして実行されます（対応する`llm-*`パッケージのインストール後に自動でインストール/有効化）。一般的なサービス（ファームウェア/バージョンにより若干異なる場合あり）：

- `llm-sys`：モデル/リソース管理とバックエンドスケジューリング（先に起動することを推奨）
- `llm-openai-api`：OpenAI互換APIサービス（デフォルトで`0.0.0.0:8000`をリッスン）
- `llm-llm`：テキスト大規模モデルバックエンド（対応する`llm-model-*`インストール後に使用）
- `llm-cosy-voice`：CosyVoice2 TTSバックエンド

よく使うコマンド：

```bash
# ステータス確認
sudo systemctl status llm-sys llm-openai-api llm-cosy-voice

# 新モデルインストール/設定変更後によく使う：サービス再起動
sudo systemctl restart llm-sys llm-openai-api llm-cosy-voice

# ログ確認（トラブルシューティングで最もよく使う）
sudo journalctl -u llm-openai-api -n 200 --no-pager
sudo journalctl -u llm-sys -n 200 --no-pager
sudo journalctl -u llm-cosy-voice -n 200 --no-pager
```

ヒント：

- ドキュメントでは通常「新しいモデルのインストール後に`llm-openai-api`を再起動してモデルリストを更新する」ことが強調されています。モデルが表示されない場合は、`sudo systemctl restart llm-sys`も追加で実行してみてください。
- 現在利用可能なモデルの確認：`curl http://127.0.0.1:8000/v1/models -H "Content-Type: application/json"`

### プロキシ/ミラー（ネットワーク）

中国国内のネットワーク環境では、サイトごとに異なる戦略が必要になることが多いです（Raspberry Pi上のローカルプロキシ`127.0.0.1:7890`を例として）：

- **GitHub**：安定したアクセスにはプロキシが必要なことが多い
- **Hugging Face**：モデルファイルが大きいため、`hf-mirror.com`の使用を推奨し、プロキシ帯域を節約するためこのコマンドでは**プロキシを無効化**

グローバルに`export http_proxy=...`を設定しないことを推奨します（オフにし忘れやすいため）。必要なときに`env`で一時的に設定してください：

```bash
# 1) GitHub：プロキシ経由（git clone / pip / curl 等すべてこの書式で可能）
env http_proxy=http://127.0.0.1:7890 https_proxy=http://127.0.0.1:7890 \
  git clone https://github.com/xxx/yyy.git

# 2) Hugging Face：プロキシ無効 + hf-mirror使用（curl -L はリダイレクトに対応）
env no_proxy='*' http_proxy= https_proxy= \
  curl -fL -o model.axmodel 'https://hf-mirror.com/<org>/<repo>/resolve/main/<path>'
```

ツールが大文字の変数のみを認識する場合は、同時に設定してください：`HTTP_PROXY`、`HTTPS_PROXY`、`NO_PROXY`。

補足：Hugging FaceのPythonダウンローダー（`huggingface_hub` / `transformers` / `datasets`）を使用する場合、ミラーエンドポイントの設定で高速化できることもあります：

```bash
export HF_ENDPOINT="https://hf-mirror.com"
```

### ディスクとログのクリーンアップ

LLM/音声モデルパッケージは通常非常に大きく（例：CosyVoice2モデルパッケージのダウンロードは約1.2GB）、Raspberry Piのシステムディスクが逼迫しやすくなります。以下の操作は比較的安全で、最もよく使われます：

1) 使用量の素早い確認

```bash
df -h /
sudo journalctl --disk-usage
sudo du -sh /opt/m5stack/data 2>/dev/null || true
```

2) journald（システムログ）のクリーンアップ

```bash
sudo journalctl --disk-usage
sudo journalctl --vacuum-size=100M
sudo journalctl --disk-usage
```

3) APTキャッシュと不要パッケージのクリーンアップ

```bash
sudo apt autoremove --purge -y
sudo apt clean
```

4) Docker（使用している場合）

```bash
docker image ls

# 例：容量の大きいROSイメージの削除（実際のイメージ名に合わせて調整）
docker rmi yahboomtechnology/ros2-foxy:2.0.1 yahboomtechnology/ros-melodic:1.2

# 注意して使用：未使用のすべてのイメージ/コンテナ/ネットワークを削除（既存サービスに影響する可能性あり）
docker system prune -a
```

5) 重複モデルファイルの削除

- `whisper.axcl`を例にすると：`install/models/`に1部のみ保持することを推奨し、プロジェクトディレクトリに同じモデルファイルの2部目を保存しないでください。

6) 不要なモデルパッケージのアンインストール（StackFlow APT）

StackFlowのモデルは`llm-model-*`形式でインストールされることが多く、アンインストールで大量のスペースを解放できます（CosyVoice2を例として）：

```bash
# インストール済みモデルパッケージの確認
dpkg -l | grep -E '^ii\\s+llm-model-' || true

# モデルのアンインストール（例）
sudo apt remove -y llm-model-cosyvoice2-0.5b-axcl

# モデルデータディレクトリが残っている場合は手動削除可能（該当モデルの機能に影響します）
sudo rm -rf /opt/m5stack/data/CosyVoice2-0.5B-axcl
```

## 一般的な実行パターン（モデル/アプリケーション）

各モデル/アプリケーションの提供形式は完全に同じではありませんが、ドキュメントでよく見られるパターンは大きく3種類に分類されます：

1) **単一統合パッケージ（zip）**：ダウンロード・解凍後にバイナリを直接実行（ビジョンdemoで一般的）。
2) **Hugging Faceリポジトリ（LFS含む場合あり）**：`git clone`で取得後、スクリプト実行/Pythonサービス起動/バイナリ起動。
3) **StackFlow APTパッケージ + systemdサービス**：`apt install`でモデルとサービスをインストールし、OpenAI互換APIで呼び出し。

### ビジョンモデルクイック体験（axcl_demo.zip）

YOLO11ドキュメントを例として、まず統合パッケージを取得：

```bash
wget https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/axcl_demo.zip
unzip axcl_demo.zip
```

その後、同じディレクトリで異なるdemoを実行（異なるモデルには異なる実行ファイルと`.axmodel`が対応）：

```bash
# 物体検出：YOLO11x
./axcl_demo/axcl_yolo11 -i axcl_demo/ssd_horse.jpg -m axcl_demo/yolo11x.axmodel

# 深度推定：Depth Anything V2（ドキュメントでは同じ統合パッケージを再利用と記載）
./axcl_demo/axcl_depth_anything -m axcl_demo/depth_anything_v2_vits_ax650.axmodel -i axcl_demo/ssd_horse.jpg
```

ビジョンモデル参考ドキュメント：

- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolo11.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolo_world_v2.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolov7_face.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_depth_anything_v2.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_mixformer_v2.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_real_esrgan.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_superresolution.md`
- `offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_rife.md`

### Whisper（whisper.axcl）オフライン音声テキスト変換

公式Whisperセクションでは「ビルド＋実行」のコマンドが示されていますが、`.axmodel`の入手元が明確にされていません。`whisper.axcl`はサンプルコードと実行ファイルのみで、**推論を実行するにはプリコンパイル済みモデルファイル（`.axmodel`）のダウンロードも必要**です。

ここでの「ビルドと実行」は2つの作業に分解できます：

- **ビルド**：リポジトリ内のC/C++ソースコードを実行ファイルにコンパイル（典型的な成果物は`install/whisper`）。このステップでは`.axmodel`は生成されません。
- **実行**：`./whisper ...`を実行し、`.axmodel`、トークン、エンベディング等の実行時ファイルをロードして推論。`.axmodel`は**NPU側実行可能モデル**で、通常Pulsar2等のツールチェーンで事前に変換/コンパイルされ、ダウンロードして使用するものとして提供されます。

`whisper-small`を例として、プリコンパイルモデルはHugging Face：`M5Stack/whisper-small-axmodel`（AX650バージョンは`ax650/`ディレクトリ内）にあります。以下はRaspberry Pi 5上でドキュメント通りにサンプルを実行するための再利用可能な手順です（現在のネットワーク制約を含む：GitHubは`7890`プロキシ経由、Hugging Faceは`hf-mirror.com`でプロキシ帯域を消費しない）。

1) ソースコードの取得（GitHub経由はプロキシ使用）

```bash
mkdir -p ~/rsp && cd ~/rsp
env http_proxy=http://127.0.0.1:7890 https_proxy=http://127.0.0.1:7890 \
  git clone --depth 1 https://github.com/ml-inory/whisper.axcl.git
```

2) ビルドとインストール（`install/whisper`を取得）

```bash
cd ~/rsp/whisper.axcl
./build.sh
```

3) プリコンパイルモデルのダウンロード（Hugging Faceは`hf-mirror.com`使用、プロキシ無効化）

> 説明：一部のネットワーク環境では`git lfs pull`がLFSドメインの名前解決/認証で停止する場合があります。その場合は`resolve/main/...`での直接ダウンロードが最も安定します（`curl -L`はリダイレクトに自動対応）。

```bash
cd ~/rsp/whisper.axcl/install
mkdir -p models

env no_proxy='*' http_proxy= https_proxy= \
  curl -fL -o models/small-encoder.axmodel \
  'https://hf-mirror.com/M5Stack/whisper-small-axmodel/resolve/main/ax650/small-encoder.axmodel'

env no_proxy='*' http_proxy= https_proxy= \
  curl -fL -o models/small-decoder-main.axmodel \
  'https://hf-mirror.com/M5Stack/whisper-small-axmodel/resolve/main/ax650/small-decoder-main.axmodel'

env no_proxy='*' http_proxy= https_proxy= \
  curl -fL -o models/small-decoder-loop.axmodel \
  'https://hf-mirror.com/M5Stack/whisper-small-axmodel/resolve/main/ax650/small-decoder-loop.axmodel'

env no_proxy='*' http_proxy= https_proxy= \
  curl -fL -o models/small-positional_embedding.bin \
  'https://hf-mirror.com/M5Stack/whisper-small-axmodel/resolve/main/small-positional_embedding.bin'

env no_proxy='*' http_proxy= https_proxy= \
  curl -fL -o models/small-tokens.txt \
  'https://hf-mirror.com/M5Stack/whisper-small-axmodel/resolve/main/small-tokens.txt'
```

4) 実行時設定ファイルの準備（ないと`t2s.json`が見つからないエラーが発生）

`whisper`は実行時にOpenCCの`t2s.json`（繁体字→簡体字変換設定）を読み込み、**カレントワーキングディレクトリ（CWD）**からファイルを検索します。ドキュメントの例では`install/`ディレクトリで実行するため、これらのファイルをコピーします：

```bash
cd ~/rsp/whisper.axcl/install
cp ../t2s.json ../TSCharacters.ocd2 ../TSPhrases.ocd2 .
```

5) 実行（公式サンプルと同じ）

```bash
cd ~/rsp/whisper.axcl/install
./whisper -w ../demo.wav
```

正常であれば末尾に`Result: ...`の認識結果が出力されます。

> ディスク容量逼迫時のヒント：モデルファイルは大きいため、`install/models/`に1部のみ保持してください（`~/rsp/whisper.axcl/models/`にもう1部の重複ファイルを置く必要はありません）。

### SenseVoice（Python）音声認識

SenseVoiceのドキュメントでは**Python方式**（`pyaxengine` + 推論スクリプト）が示されています。典型的な特徴：

- メリット：導入が容易、`auto/zh/en/yue/ja/ko`等の言語パラメータをサポート；初回実行時にモデルが自動ダウンロードされる
- デメリット：Python仮想環境が必要；初回実行時にネットワークへの依存あり；リポジトリがGit LFSを使用する場合あり

ドキュメントに従いRaspberry Pi 5で実行する大まかな手順：

1) リポジトリの取得（Hugging Face、Git LFSが必要な場合あり）

```bash
sudo apt update
sudo apt install -y git-lfs
git lfs install

mkdir -p ~/rsp && cd ~/rsp
# Hugging Faceミラー（プロキシ不使用）の例：
env no_proxy='*' http_proxy= https_proxy= \
  git clone https://hf-mirror.com/AXERA-TECH/SenseVoice

# 直接huggingface.coに接続する方が速い場合はこちらも可：
# git clone https://huggingface.co/AXERA-TECH/SenseVoice
cd SenseVoice
```

2) 仮想環境の作成と依存パッケージのインストール

```bash
python -m venv sensevoice
source sensevoice/bin/activate

# pyaxengine wheelはGitHub Releaseにあります：GitHubアクセスにプロキシが必要な場合は一時的にhttp_proxy/https_proxyを追加
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install -r requirements.txt
```

3) 実行（初回はモデルが自動ダウンロードされる）

```bash
python main.py -i test.mp3

# オプション：認識言語の指定（デフォルトはauto）
python main.py -i test.mp3 -l auto
python main.py -i test.mp3 -l zh
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_sensevoice.md`

#### WhisperとSenseVoiceの選び方

- 「ビルド後にバイナリ1つで直接実行＋手動でモデルファイル管理（完全オフライン）」を望む場合：**Whisper（whisper.axcl）**向き
- 「Pythonで手軽に始める＋モデル自動ダウンロード＋より豊富な言語/パラメータオプション」を望む場合：**SenseVoice**向き
- 最も確実な方法：自分の音声サンプルで両方を実行してから判断（精度、遅延、リソース使用量の違いは顕著に現れます）

### OpenAI互換API（テキスト対話）

ドキュメントには**OpenAI API互換**のローカルサービス（デフォルトでローカルホストの`8000`ポート）が提供されており、StackFlowパッケージのインストール後すぐに使用できます：

```bash
sudo apt install lib-llm llm-sys llm-llm llm-openai-api
sudo apt install llm-model-qwen3-1.7b-int8-ctx-axcl

# 新しいモデルのインストール後は毎回、モデルリストを更新するためにサービスの再起動が必要
sudo systemctl restart llm-openai-api
```

呼び出し例：

```bash
curl http://127.0.0.1:8000/v1/models -H "Content-Type: application/json"

curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxxxxx" \
  -d '{
    "model": "qwen3-1.7B-Int8-ctx-axcl",
    "messages": [
      {"role": "developer", "content": "You are a helpful home assistant."},
      {"role": "user", "content": "Turn on the light!"}
    ]
  }'
```

Python（OpenAI SDK）例：

```python
from openai import OpenAI

client = OpenAI(api_key="sk-", base_url="http://127.0.0.1:8000/v1")
print(client.models.list())
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_openai.md`

> 注意：特定の「単一モデルdemo/独立推論サービス」を実行する場合、ドキュメントではリソース解放のために先に`sudo systemctl stop llm-openai-api`を実行する必要があると記載されています（例は`Qwen3-1.7B`ドキュメント参照）。

### OpenAI互換API（CosyVoice2テキスト音声変換）

前提：前述の**「3.5 StackFlow（LLM8850）ソフトウェアソース」**に従い`bookworm llm8850`ソースを追加済み（そうでないと`apt`で`llm-*`パッケージが見つかりません）。

#### 1) インストール（APT）

```bash
sudo apt update
sudo apt install -y lib-llm llm-sys llm-cosy-voice llm-openai-api llm-model-cosyvoice2-0.5b-axcl

# モデルのインストール/更新後は再起動を推奨（少なくともllm-openai-apiを再起動してモデルリストを更新）
sudo systemctl restart llm-sys llm-cosy-voice llm-openai-api
```

ヒント：

- CosyVoice2はLLMベースの音声生成モデルで、現在のバージョンでは1回の音声生成の最大長は約**27秒**です。
- `llm-model-cosyvoice2-0.5b-axcl`パッケージは大きく（ダウンロード約1.2GB）、インストール後のモデルデータはデフォルトで`/opt/m5stack/data/CosyVoice2-0.5B-axcl/`に配置されます。

#### 2) モデルが登録されていることを確認

```bash
curl http://127.0.0.1:8000/v1/models -H "Content-Type: application/json"
```

正常であれば`CosyVoice2-0.5B-axcl`が表示されます。

#### 3) 音声の生成（wav）

現在のバージョンでは明示的に`voice`を指定することを推奨します。デフォルト付属のプロンプト音色ディレクトリは`prompt_data`です：

```bash
curl http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "model": "CosyVoice2-0.5B-axcl",
    "voice": "prompt_data",
    "response_format": "wav",
    "input": "こんにちは、CosyVoice2のテストです。"
  }' \
  -o output.wav

# 再生（Raspberry Piでよく使う）
aplay output.wav
```

#### 4) 音色クローン（任意）

ドキュメントには「プロンプト音声＋対応テキスト」から特徴ファイルディレクトリ（例：`zh_woman1/`）を生成し、`voice`として使用するための`CosyVoice2-scripts`が提供されています：

```bash
mkdir -p ~/rsp && cd ~/rsp

# メインリポジトリの取得（Hugging Faceミラー；プロキシ帯域を節約したい場合はプロキシ無効化可能）
env no_proxy='*' http_proxy= https_proxy= \
  git clone https://hf-mirror.com/M5Stack/CosyVoice2-scripts
cd CosyVoice2-scripts

# リポジトリにはsubmoduleが含まれます：submoduleがGitHub上にあり、GitHubアクセスにプロキシが必要な場合はこのステップでプロキシを追加
git submodule update --init --recursive
# env http_proxy=http://127.0.0.1:7890 https_proxy=http://127.0.0.1:7890 \
#   git submodule update --init --recursive

python -m venv cosyvoice
source cosyvoice/bin/activate
pip install -r requirements.txt

python3 scripts/process_prompt.py --prompt_text asset/zh_woman1.txt --prompt_speech asset/zh_woman1.wav --output zh_woman1

# 生成したディレクトリをモデルディレクトリに配置（権限不足の場合はsudoを追加）
cp -r zh_woman1 /opt/m5stack/data/CosyVoice2-0.5B-axcl/

# llm-sysにモデル設定を再初期化させる
sudo systemctl restart llm-sys llm-openai-api
```

その後、呼び出し時に`voice`をディレクトリ名に変更するだけです：

```bash
curl http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "model": "CosyVoice2-0.5B-axcl",
    "voice": "zh_woman1",
    "response_format": "wav",
    "input": "君不見黄河之水天上來、奔流到海不復回。"
  }' \
  -o output.wav
```

デフォルト音色をクローンした音色に置き換えたい場合は、`/opt/m5stack/data/models/mode_CosyVoice2-0.5B-axcl.json`内の`prompt_dir`フィールドを対応するディレクトリ名に変更し、`llm-sys`を再起動してください。

#### 5) よくあるエラーのトラブルシューティング

- `POST /v1/audio/speech`が`500`を返し、内容が`{"detail":"Expecting value: line 1 column 1 (char 0)"}`の場合：通常はバックエンドが準備できていないか、パラメータ/音色ディレクトリが正しくありません。以下の順序で確認することを推奨：
  1) `curl /v1/models`で`CosyVoice2-0.5B-axcl`が表示されるか確認
  2) リクエストに明示的に`"voice": "prompt_data"`を追加
  3) `sudo systemctl restart llm-sys llm-cosy-voice llm-openai-api`
  4) `sudo journalctl -u llm-openai-api -n 200 --no-pager`

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_cosy_voice2_api.md`

## NPUサンプルとツールチェーン

ドキュメントに記載：

- **Pulsar2**：モデルをチップNPUにデプロイするためのツールチェーン（オペレータサポートリスト、大規模モデル変換等を含む）。
- **AXCL-Samples**：愛芯元智が主導で開発したサンプルプロジェクト。同社SoC搭載PCIeアクセラレータカード上での一般的なオープンソースモデルの実行サンプルを含み、プリコンパイルModelZoo（百度网盘 / Hugging Face / OSS等のダウンロードチャネル入口がドキュメントに記載）も提供。

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_npu_samples.md`

## NPUベンチマーク（性能参考）

ドキュメント記載（Raspberry Pi 5 Hostでのテスト、参考値のみ）：

- 更新日：**2024-11-22**
- ツールチェーンバージョン：**Pulsar2 3.2-patch2**
- テストツール：`axcl_run_model`
- バッチサイズ：1または8
- 単位：IPS（Image/Second）；音声はRTF；LLM/VLMはTTFTとtokens/s
- `axcl_run_model`はDevice側推論時間のみを計測（HostごとのmemcopyやPCIe性能が全体に影響）

**ビジョンモデル（IPS）**

| モデル | 入力 | Batch1 | Batch8 |
|---|---:|---:|---:|
| Inceptionv1 | 224 | 1073 | 2494 |
| Inceptionv3 | 224 | 478 | 702 |
| MobileNetv1 | 224 | 1508 | 4854 |
| MobileNetv2 | 224 | 1366 | 5073 |
| ResNet18 | 224 | 1066 | 2254 |
| ResNet50 | 224 | 576 | 1045 |
| SqueezeNet11 | 224 | 1560 | 5961 |
| Swin-T | 224 | 342 | 507 |
| ViT-B/16 | 224 | 162 | 207 |
| YOLOv5s | 640 | 326 | 394 |
| YOLOv6s | 640 | 282 | 322 |
| YOLOv8s | 640 | 248 | 279 |
| YOLOv9s | 640 | 237 | — |
| YOLOv10s | 640 | 298 | — |
| YOLOv11n | 640 | 860 | — |
| YOLOv11s | 640 | 305 | — |
| YOLOv11m | 640 | 114 | — |
| YOLOv11l | 640 | 87 | — |
| YOLOv11x | 640 | 41 | — |

**オーディオモデル（RTF）**

| モデル | RTF |
|---|---:|
| Whisper-Tiny | 0.03 |
| Whisper-Small | 0.18 |
| MeloTTS | 0.04 |

**LLM**

| モデル | プロンプト(tokens) | TTFT(ms) | 生成(tokens/s) |
|---|---:|---:|---:|
| Qwen2.5-0.5B | 128 | 188 | 28 |
| Qwen2.5-1.5B | 128 | 407.75 | 9.05 |
| Qwen2.5-1.5B-Int4 | 128 | 407.75 | 9.05 |

**VLM**

| モデル | 画像入力 | Image Encoder(ms) | プロンプト(tokens) | TTFT(ms) | 生成(tokens/s) |
|---|---|---:|---:|---:|---:|
| InternVL2-1B | 448×448 | 4200 | 320 | 425 | 29 |

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_npu_benchmark.md`

## FAQ（よくある質問）

### axcl-smiでデバイスが見つからないエラー（0x8030010b）

現象（例）：

```text
axcl init fail, ret = 0x8030010b
open /dev/axcl_host fail, errno: 2 No such file or directory
```

トラブルシューティング方針（ドキュメント記載）：

1) `lspci`で`Axera Semiconductor Co., Ltd Device 0650`が認識されているか確認
2) アクセラレータカードがしっかり挿さっているか確認；電源を切って再起動
3) ドライバをアンインストールして再インストール（`sudo apt remove axclhost`後に再インストール）

補足：最近カーネルをアップグレードした場合（`uname -r`が変わった場合）、DKMSが**現在のカーネル**用にモジュールをコンパイルしていないため、デバイスノードが作成されていない可能性があります。以下を試してください：

```bash
sudo /usr/sbin/dkms autoinstall -k "$(uname -r)"
sudo modprobe axcl_host
ls -l /dev/axcl_host
axcl-smi
```

参考原文：`offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_faq.md`

## オフラインドキュメント索引（トピック別）

> 以下のファイルはすべて`offline/`にあります。原文と照合する場合は、対応するファイルを直接開いてキーワード（モデル名、`wget`、`git clone`、`python -m venv`、`systemctl`等）で検索してください。

### クイックスタート / インストール

- [ハードウェア取り付け](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_hardware_install.md)
- [Raspberry Pi環境とドライバ](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_software_install.md)
- [Windowsインストール](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_windows_install.md)
- [Docker](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_docker.md)
- [Git LFS](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)

### ツール / 上級者向け

- [AXCL-SMI](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_axcl_smi.md)
- [AXCL SDK API](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_axcl_api.md)（原文は長い）
- [AXCLサンプル使用説明](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_samples.md)
- [FFmpeg](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_ffmpeg.md)
- [NPUサンプル](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_npu_samples.md)
- [NPUベンチマーク](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_npu_benchmark.md)

### ビジョンモデル

- [YOLO11](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolo11.md)
- [Yolo-World-V2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolo_world_v2.md)
- [Yolov7-face](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolov7_face.md)
- [Depth-Anything-V2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_depth_anything_v2.md)
- [MixFormer-V2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_mixformer_v2.md)
- [Real-ESRGAN](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_real_esrgan.md)
- [SuperResolution](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_superresolution.md)
- [RIFE](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_rife.md)
- [PPOCR v5](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_ppocr_v5.md)

### 大規模言語モデル（LLM）

- [Qwen3-0.6B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen3_0.6b.md)
- [Qwen3-1.7B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen3_1.7b.md)
- [Qwen2.5-0.5B-Instruct](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen2.5_0.5b.md)
- [Qwen2.5-1.5B-Instruct](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen2.5_1.5b.md)
- [DeepSeek-R1-Distill-Qwen-1.5B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_deepseek.md)
- [MiniCPM4-0.5B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_minicpm4.md)

### マルチモーダル（VLM/CLIP）

- [InternVL3-1B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_internvl3.md)
- [Qwen2.5-VL](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen2.5_vl.md)
- [Qwen3-VL-2B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen3_vl_2b.md)
- [Qwen3-VL-4B](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_qwen3_vl_4b.md)
- [SmolVLM2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_smolvlm2.md)
- [LibCLIP](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_clip.md)
- [Jina CLIP v2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_jina_clip_v2.md)

### オーディオモデル

- [Whisper](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_whisper.md)
- [MeloTTS](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_melotts.md)
- [SenseVoice](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_sensevoice.md)
- [CosyVoice2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_cosy_voice2.md)
- [3D-Speaker-MT](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_3d_speaker_mt.md)
- [sherpa-onnx](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_sherpa-onnx.md)

### 生成/クリエイティブ

- [lcm-lora-sdv1-5](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_lcm_lora_sd.md)
- [SD1.5-LLM8850](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_sd_demo.md)
- [LivePortrait](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_liveportrait.md)
- [Deimv2](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_deimv2.md)

### アプリケーション

- [Frigate NVR](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_frigate.md)
- [Immich](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_immich.md)
- [OpenAI API](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_openai.md)
- [CosyVoice2 API](offline/ja_guide_ai_accelerator_llm-8850_m5_llm_8850_cosy_voice2_api.md)
