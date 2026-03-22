# MixFormer V2

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_mixformer_v2

MixFormerV2 は、改良型の統合 Transformer ベースのオブジェクトトラッキングモデルです。特徴抽出とマッチングプロセスを融合することで、トラッキングの精度と速度を向上させています。動画内の指定オブジェクトをリアルタイムで追跡する用途に適しています。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/MixFormerV2)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリを取得してください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/MixFormerV2
```

ファイル一覧:

```text
m5stack@raspberrypi:~/rsp/MixFormerV2 $ ls -lh
total 63M
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 11 18:28 ax630c
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 11 18:28 ax650
-rw-rw-r-- 1 m5stack m5stack  63M Aug 11 18:28 car.avi
-rw-rw-r-- 1 m5stack m5stack    0 Aug 11 18:28 config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 11 18:28 onnx
-rw-rw-r-- 1 m5stack m5stack 4.0K Aug 11 18:28 README.md
-rw-rw-r-- 1 m5stack m5stack  15K Aug 11 18:28 run_mixformer2_axmodel.py
-rw-rw-r-- 1 m5stack m5stack  14K Aug 11 18:28 run_mixformer2_onnx.py
```

## 環境構築

まず、Python 仮想環境を作成します。

```bash
python -m venv mixformer
```

仮想環境のアクティベート:

```bash
source mixformer/bin/activate
```

次に、必要な依存パッケージをインストールします。

```bash
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install argparse numpy opencv-python glob2
```

## 実行

以下のコマンドでオブジェクトトラッキングを実行します。`-r` オプションで処理するフレーム数を指定できます。

```bash
python3 run_mixformer2_axmodel.py --model-path ax650/mixformer_v2.axmodel --frame-path car.avi -r 10
```

実行結果:

```text
(mixformer) m5stack@raspberrypi:~/rsp/MixFormerV2 $ python3 run_mixformer2_axmodel.py --model-path ax650/mixformer_v2.axmodel --frame-path car.avi -r 10
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.4-dirty 4ff37520-dirty
====================type================= [1079, 482] <class 'list'> <class 'list'>
第一帧初始化完毕！
Video: tracking     1011.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Video: tracking     8.0fps
Reached the maximum number of frames (10). Exiting loop.
video: average finale average tracking fps 121.2 fps
```
