# SuperResolution

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_superresolution

SuperResolution（超解像）は、ディープラーニングやその他のアルゴリズムを用いて低解像度画像を高解像度画像に変換するモデルです。画像のディテールと視覚的品質を向上させることを目的としており、動画のアップスケーリングにも対応しています。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/SuperResolution)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリを取得してください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/SuperResolution
```

ファイル一覧:

```text
m5stack@raspberrypi:~/rsp/SuperResolution $ ls -lh
total 20K
-rw-rw-r-- 1 m5stack m5stack 3.8K Sep  4 18:49 config.json
drwxrwxr-x 4 m5stack m5stack 4.0K Sep  4 18:49 model_convert
drwxrwxr-x 5 m5stack m5stack 4.0K Sep  4 19:12 python
-rw-rw-r-- 1 m5stack m5stack 2.3K Sep  4 18:49 README.md
drwxrwxr-x 2 m5stack m5stack 4.0K Sep  4 19:03 video
```

## 環境構築

まず、Python 仮想環境を作成します。

```bash
python -m venv sr
```

仮想環境のアクティベート:

```bash
source sr/bin/activate
```

次に、必要な依存パッケージをインストールします。

```bash
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install opencv-python torch torchvision tqdm scikit-image
```

## 実行

以下のコマンドで動画の超解像処理を実行します。`--model` でモデルファイル、`--dir_demo` で入力動画ファイルを指定します。

```bash
python python/run_axmodel.py --model model_convert/axmodel/edsr_baseline_x2_1.axmodel --dir_demo video/test_1920x1080.mp4
```

実行結果:

```text
(sr) m5stack@raspberrypi:~/rsp/SuperResolution $ python python/run_axmodel.py --model model_convert/axmodel/edsr_baseline_x2_1.axmodel --dir_demo video/test_1920x1080.mp4
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 4.2 6bff2f67
100%|█████████████████████████████████████████| 267/267 [06:21<00:00,  1.43s/it]
Total time: 275.618 seconds for 267 frames
Average time: 1.032 seconds for each frame
```

元動画のスクリーンショット:

（元の低解像度動画フレーム）

出力動画のスクリーンショット:

（超解像処理後の動画フレーム）
