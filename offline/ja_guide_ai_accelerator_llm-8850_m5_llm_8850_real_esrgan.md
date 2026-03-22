# Real-ESRGAN

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_real_esrgan

Real-ESRGAN は、ディープラーニングベースの画像超解像モデルです。低解像度画像のディテールを復元しながらノイズを効果的に除去することができ、画像や動画の高画質化処理に適しています。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/Real-ESRGAN)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリを取得してください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/Real-ESRGAN
```

ファイル一覧:

```text
m5stack@raspberrypi:~/rsp/Real-ESRGAN $ ls -lh
total 428K
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 13 09:12 ax630c
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 13 09:12 ax650
-rw-rw-r-- 1 m5stack m5stack    0 Aug 13 09:11 config.json
-rw-rw-r-- 1 m5stack m5stack 2.9K Aug 13 09:11 main.py
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 13 09:12 onnx
-rw-rw-r-- 1 m5stack m5stack 387K Aug 13 09:12 output_test_256.jpg
-rw-rw-r-- 1 m5stack m5stack 3.9K Aug 13 09:11 README.md
-rw-rw-r-- 1 m5stack m5stack  19K Aug 13 09:11 test_256.jpeg
```

## 環境構築

まず、Python 仮想環境を作成します。

```bash
uv venv esrgan
```

仮想環境のアクティベート:

```bash
source esrgan/bin/activate
```

次に、必要な依存パッケージをインストールします。

```bash
uv pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
uv pip install argparse numpy opencv-python
```

## 実行

以下のコマンドで超解像処理を実行します。`--input` で入力画像、`--output` で出力ファイル名を指定します。

```bash
python3 main.py --input test_256.jpeg --output test_256_20e.jpeg --model ax650/realesrgan-x4-256.axmodel
```

実行結果:

```text
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

元画像:

（元の低解像度画像）

出力:

（超解像処理後の画像）
