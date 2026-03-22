# RIFE

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_rife

RIFE（Real-Time Intermediate Flow Estimation）は、ディープラーニングベースのリアルタイム動画フレーム補間モデルです。中間フレームを効率的に予測してフレームレートを向上させることができます。動画のスムーズ化やスローモーション生成、アニメーション制作などの分野に適用されます。

## モデルのダウンロード

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/RIFE.axera)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/RIFE.axera
```

ファイル一覧:

```text
m5stack@raspberrypi:~/rsp/RIFE.axera $ ls -lh
total 52K
-rw-rw-r-- 1 m5stack m5stack 1.1K Sep 29 14:28 build_config.json
-rw-rw-r-- 1 m5stack m5stack    0 Sep 29 14:28 config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 29 14:28 model
-rw-rw-r-- 1 m5stack m5stack 6.9K Sep 29 14:28 ms_ssim.py
-rw-rw-r-- 1 m5stack m5stack 1.8K Sep 29 14:28 README.md
-rw-rw-r-- 1 m5stack m5stack   88 Sep 29 14:28 requirements.txt
-rw-rw-r-- 1 m5stack m5stack 9.2K Sep 29 14:28 run_axmodel.py
-rw-rw-r-- 1 m5stack m5stack 9.2K Sep 29 14:28 run_onnx.py
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 29 14:28 video
```

## 環境構築

まず、Python 仮想環境を作成します。

```bash
python -m venv rife
```

仮想環境のアクティベート:

```bash
source rife/bin/activate
```

次に、必要な依存パッケージをインストールします。

```bash
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install -r requirements.txt
```

## 実行

以下のコマンドでフレーム補間を実行します。`--model` でモデルファイル、`--video` で入力動画ファイルを指定します。

```bash
python3 run_axmodel.py --model ./model/rife_x2_720p.axmodel --video ./video/demo.mp4
```

実行結果:

```text
(rife) m5stack@raspberrypi:~/rsp/RIFE.axera $ python3 run_axmodel.py --model ./model/rife_x2_720p.axmodel --video ./video/demo.mp4
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 4.2 77cdc0c2
./video/demo.mp4, 128.0 frames in total, 25.0FPS to 50.0FPS
The audio will be merged after interpolation process
 99%|██████████████████████████████████████▋| 127/128.0 [02:45<00:01,  1.30s/it]
```
