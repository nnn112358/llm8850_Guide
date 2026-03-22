# LivePortrait

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_liveportrait

LivePortrait は、ディープラーニングベースのポートレート駆動生成モデルです。静止画のポートレートを入力として、リアルな表情や動作を持つ動的なビデオに変換することができます。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/LivePortrait)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/LivePortrait
```

### ファイル説明

以下は、クローンしたリポジトリに含まれるファイルの一覧です。

```
m5stack@raspberrypi:~/rsp/LivePortrait $ ls -lh
total 16K
drwxrwxr-x 3 m5stack m5stack 4.0K Aug 13 09:41 assets
-rw-rw-r-- 1 m5stack m5stack    0 Aug 13 09:41 config.json
drwxrwxr-x 5 m5stack m5stack 4.0K Aug 13 09:41 python
-rw-rw-r-- 1 m5stack m5stack 5.7K Aug 13 09:41 README.md
```

## 仮想環境の作成

まず、Python の仮想環境を作成します。

```bash
uv venv lvpr
```

## 仮想環境の有効化

作成した仮想環境を有効化します。

```bash
source lvpr/bin/activate
```

## 依存パッケージのインストール

必要なパッケージをインストールします。

```bash
uv pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
uv pip install -r python/requirements.txt
uv pip install requests tqdm
```

## 静止画による駆動（Image）

ソース画像とドライビング画像を指定して、表情を転写した静止画を生成します。

### 実行

```bash
python ./python/infer.py --source ./assets/examples/source/s0.jpg --driving ./assets/examples/driving/d8.jpg --models ./python/axmodels/ --output-dir ./axmodel_infer
```

### 実行結果

```
(lvpr) m5stack@raspberrypi:~/rsp/LivePortrait $ python ./python/infer.py --source ./assets/examples/source/s0.jpg --driving ./assets/examples/driving/d8.jpg --models ./python/axmodels/ --output-dir ./axmodel_infer
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 144960ad
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 144960ad
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 0f7260e8
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 144960ad
FaceAnalysisDIY warmup time: 0.237s
LandmarkRunner warmup time: 0.183s
2025-08-13 10:04:42.914 | INFO     | __main__:main:727 - Start making driving motion template...
2025-08-13 10:04:43.782 | INFO     | __main__:main:747 - Prepared pasteback mask done.
2025-08-13 10:04:44.565 | INFO     | __main__:main:787 - The output of image-driven portrait animation is an image.
2025-08-13 10:04:50.566 | DEBUG    | __main__:warp_decode:647 - warp time: 5.997s
2025-08-13 10:04:50.902 | INFO     | __main__:main:881 - Animated image: ./axmodel_infer/s0--d8.jpg
2025-08-13 10:04:50.902 | INFO     | __main__:main:882 - Animated image with concat: ./axmodel_infer/s0--d8_concat.jpg
2025-08-13 10:04:50.920 | DEBUG    | __main__:<module>:894 - LivePortrait axmodel infer time: 18.068s
```

## 動画による駆動（Video）

ソース画像とドライビング動画を指定して、動画形式のアニメーションを生成します。

### 実行

```bash
python ./python/infer.py --source ./assets/examples/source/s0.jpg --driving ./assets/examples/driving/d0.mp4 --models ./python/axmodels/ --output-dir ./axmodel_infer
```

### 実行結果

```
(lvpr) m5stack@raspberrypi:~/rsp/LivePortrait $ python ./python/infer.py --source ./assets/examples/source/s0.jpg --driving ./assets/examples/driving/d0.mp4 --models ./python/axmodels/ --output-dir ./axmodel_infer

[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 144960ad
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 144960ad
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 0f7260e8
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 144960ad
FaceAnalysisDIY warmup time: 0.213s
LandmarkRunner warmup time: 0.180s
2025-08-13 10:14:47.779 | INFO     | __main__:main:727 - Start making driving motion template...
2025-08-13 10:15:04.110 | INFO     | __main__:main:747 - Prepared pasteback mask done.
2025-08-13 10:15:04.903 | INFO     | __main__:main:785 - The animated video consists of 78 frames.
2025-08-13 10:15:11.468 | DEBUG    | __main__:warp_decode:647 - warp time: 6.561s
2025-08-13 10:25:50.630 | DEBUG    | __main__:warp_decode:647 - warp time: 8.343s
2025-08-13 10:25:51.493 | INFO     | __main__:has_audio_stream:114 - Error occurred while probing video: ./assets/examples/driving/d0.mp4, you may need to install ffprobe! (https://ffmpeg.org/download.html) Now set audio to false!
2025-08-13 10:25:54.608 | INFO     | __main__:main:870 - Animated video: ./axmodel_infer/s0--d0.mp4
2025-08-13 10:25:54.609 | INFO     | __main__:main:871 - Animated video with concat: ./axmodel_infer/s0--d0_concat.mp4
2025-08-13 10:25:54.644 | DEBUG    | __main__:<module>:894 - LivePortrait axmodel infer time: 670.930s
```
