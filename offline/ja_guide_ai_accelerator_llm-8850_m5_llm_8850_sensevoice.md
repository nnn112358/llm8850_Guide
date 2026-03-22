# SenseVoice

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_sensevoice

SenseVoice は、音声認識・音声理解に対応したモデルです。音声コンテンツを効率的かつ正確にテキストへ変換でき、多言語・多シーンの音声処理をサポートしています。

## モデルのダウンロード

手動でモデルをダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/SenseVoice
```

## ファイル説明

```
m5stack@raspberrypi:~/rsp/SenseVoice $ ls -lh
total 464K
-rw-rw-r-- 1 m5stack m5stack  11K Aug 12 16:38 am.mvn
-rw-rw-r-- 1 m5stack m5stack 369K Aug 12 16:38 chn_jpn_yue_eng_ko_spectok.bpe.model
-rw-rw-r-- 1 m5stack m5stack    0 Aug 12 16:38 config.json
-rw-rw-r-- 1 m5stack m5stack  108 Aug 12 16:38 download_dataset.sh
-rw-rw-r-- 1 m5stack m5stack  893 Aug 12 16:38 download_utils.py
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:38 embeddings
-rw-rw-r-- 1 m5stack m5stack  17K Aug 12 16:38 frontend.py
-rw-rw-r-- 1 m5stack m5stack 1.1K Aug 12 16:38 LICENSE
-rw-rw-r-- 1 m5stack m5stack 1.6K Aug 12 16:38 main.py
-rw-rw-r-- 1 m5stack m5stack 3.2K Aug 12 16:38 print_utils.py
-rw-rw-r-- 1 m5stack m5stack 1.5K Aug 12 16:38 README.md
-rw-rw-r-- 1 m5stack m5stack   71 Aug 12 16:38 requirements.txt
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:38 sensevoice_ax650
-rw-rw-r-- 1 m5stack m5stack 9.1K Aug 12 16:38 SenseVoiceAx.py
-rw-rw-r-- 1 m5stack m5stack 2.5K Aug 12 16:38 test_wer.py
-rw-rw-r-- 1 m5stack m5stack 4.7K Aug 12 16:38 tokenizer.py
```

## 仮想環境の作成

```bash
uv venv sensevoice
```

## 仮想環境の有効化

```bash
source sensevoice/bin/activate
```

## 依存パッケージのインストール

```bash
uv pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
uv pip install -r requirements.txt
```

## 実行

以下のコマンドで音声ファイルの認識を実行します。

```bash
python main.py -i test.mp3
```

利用可能なパラメータは以下の通りです。

| パラメータ名 | 説明 | デフォルト値 |
|---|---|---|
| `--input` / `-i` | 入力音声ファイル | - |
| `--language` / `-l` | 認識言語。auto, zh, en, yue, ja, ko をサポート | auto |

> **ヒント:** 初回実行時にはモデルが自動的にダウンロードされます。

実行結果は以下の通りです。

```
(sensevoice) m5stack@raspberrypi:~/rsp/SenseVoice $ python main.py -i test_en.mp3
[INFO] Available providers:  ['AXCLRTExecutionProvider']
input_audio: test_en.mp3
language: auto
use_itn: True
model_path: /home/m5stack/rsp/SenseVoice/models/SenseVoice/sensevoice_ax650/sensevoice.axmodel
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 4.0 156de6f7
RTF: 0.015400904924311537    Latency: 0.463259220123291s  Total length: 30.08s
['You want to be a nurse or an archi', "A lawyer or a member of our military. you''re going to need a", 'Eduducation for every single one of those caree', 'Not drop out of school and just drop into a good j', "You''ve got to train for it", "And learn for it. And this isn't just impo", 'lifeIn your own future. what you make', 'Will decide nothing less than the future of this country..']
```
