# DEIMv2

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_deimv2

DEIMv2 は、DETR（Detection Transformer）をベースとした効率的なリアルタイム物体検出モデルです。Transformer アーキテクチャを活用して高精度な物体検出を実現しつつ、推論速度の最適化も図られています。

## モデルのダウンロード

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/DEIMv2)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/DEIMv2
```

ファイル一覧:

```text
m5stack@raspberrypi:~/rsp/DEIMv2 $ ls -lh
total 19M
-rw-rw-r-- 1 m5stack m5stack 8.9K Nov 10 10:11 axmodel_inf.py
-rw-rw-r-- 1 m5stack m5stack    0 Nov 10 10:11 config.json
-rw-rw-r-- 1 m5stack m5stack  18M Nov 10 10:11 deimv2_dinov3_s_coco.axmodel
-rw-rw-r-- 1 m5stack m5stack  87K Nov 10 10:11 onboard_result.jpg
-rw-rw-r-- 1 m5stack m5stack 261K Nov 10 10:11 people.jpg
-rw-rw-r-- 1 m5stack m5stack  888 Nov 10 10:11 README.md
```

## 環境構築

まず、Python 仮想環境を作成します。

```bash
python -m venv deim
```

仮想環境のアクティベート:

```bash
source deim/bin/activate
```

次に、必要な依存パッケージをインストールします。

```bash
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install opencv-python torch torchvision
```

## 実行

以下のコマンドで物体検出の推論を実行します。`-ms n` オプションはマルチスケール推論を無効にする設定です。

```bash
python3 axmodel_inf.py --axmodel deimv2_dinov3_s_coco.axmodel --input people.jpg -ms n
```

実行結果:

```text
(deim) m5stack@raspberrypi:~/rsp/DEIMv2 $ python3 axmodel_inf.py --axmodel deimv2_dinov3_s_coco.axmodel --input people.jpg -ms n
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 4.2 8a28aa57
Image processing complete. Result saved as 'result.jpg'.
```

入力画像:

（入力画像）

出力画像:

（検出結果の出力画像）
