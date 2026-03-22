# 3D-Speaker-MT

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_3d_speaker_mt

3D-Speaker-MT は、マルチタスク学習を活用した話者認識モデルです。話者検証や話者識別などのタスクを同時に処理でき、さまざまなシナリオで高精度な音声 ID 認識を実現します。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/3D-Speaker-MT.axera)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/3D-Speaker-MT.axera
```

### ファイル説明

```
m5stack@raspberrypi:~/rsp/3D-Speaker-MT.axera $ ls -lh
total 64K
-rwxrwxr-x 1 m5stack m5stack 7.7K Sep 29 14:39 ax_meeting_transc_demo.py
drwxrwxr-x 4 m5stack m5stack 4.0K Sep 29 14:39 ax_model
-rwxrwxr-x 1 m5stack m5stack    0 Sep 29 14:39 config.json
-rwxrwxr-x 1 m5stack m5stack  33K Sep 29 14:39 model.py
-rwxrwxr-x 1 m5stack m5stack 3.4K Sep 29 14:39 README.md
-rwxrwxr-x 1 m5stack m5stack   74 Sep 29 14:39 requirements.txt
drwxrwxr-x 5 m5stack m5stack 4.0K Sep 29 14:39 utils
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 29 14:39 wav
```

## 仮想環境の作成

```bash
uv venv speaker
```

## 仮想環境の有効化

```bash
source speaker/bin/activate
```

## 依存パッケージのインストール

```bash
uv pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
uv pip install -r requirements.txt
```

## 実行

以下のコマンドで、音声ファイルに対して話者認識（会議音声の話者分離）を実行します。

```bash
python3 ax_meeting_transc_demo.py --output_dir output_dir --wav_file wav/vad_example.wav
```

### 実行結果

```
(esrgan) m5stack@raspberrypi:~/rsp/Real-ESRGAN $ python3 main.py --input test_256.jpeg --output test_256_20e.jpeg --model ax650/realesrgan-x4-256.axmodel
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.4 3dfd5692
input.1 [1, 256, 256, 3] uint8
1895 [1, 1024, 1024, 3] float32
Original Image Shape: (243, 243, 3)
Preprocessed Image Shape: (1, 256, 256, 3)
Inference Time: 454.03 ms
Output Shape: (1, 1024, 1024, 3)
Final Output Image Shape: (1024, 1024, 3)
```
