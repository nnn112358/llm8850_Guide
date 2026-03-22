# Jina CLIP v2

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_jina_clip_v2

Jina CLIP v2 は、多言語対応の汎用マルチモーダル埋め込みモデルです。テキストと画像の両方を同一のベクトル空間に埋め込むことで、クロスモーダル検索やマッチングといったタスクに活用できます。

## モデルのダウンロード

手動でモデルをダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/jina-clip-v2
```

## ファイル説明

```
(ppocr) m5stack@raspberrypi:~/rsp/jina-clip-v2 $ ls -lh
total 1.7G
-rw-rw-r-- 1 m5stack m5stack  51K Oct 20 12:24 beach1.jpg
-rw-rw-r-- 1 m5stack m5stack    0 Oct 20 12:24 config.json
-rw-rw-r-- 1 m5stack m5stack 351M Oct 20 12:25 image_encoder.axmodel
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 20 12:24 jina-clip-v2
-rw-rw-r-- 1 m5stack m5stack 1.3K Oct 20 12:24 README.md
-rw-rw-r-- 1 m5stack m5stack 3.2K Oct 20 12:24 run_axmodel.py
-rw-rw-r-- 1 m5stack m5stack 1.3G Oct 20 12:26 text_encoder.axmodel
```

## 仮想環境の作成

```bash
python -m venv clip
```

## 仮想環境の有効化

```bash
source clip/bin/activate
```

## 依存パッケージのインストール

```bash
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install torch pillow transformers timm torchvision
```

## 実行

以下のコマンドでモデルを実行し、テキストと画像の類似度を計算します。

```bash
python run_axmodel.py
```

実行結果は以下の通りです。

```
(clip) m5stack@raspberrypi:~/rsp/jina-clip-v2 $ python3 run_axmodel.py -i beach1.jpg -t "beautiful sunset over the beach" -iax ./image_encoder.axmodel -tax ./text_encoder.axmodel --hf_path ./jina-clip-v2
[INFO] Available providers:  ['AXCLRTExecutionProvider']
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 4.2 df480136
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 4.2 27910799
Using a slow image processor as `use_fast` is unset and a slow processor was saved with this model. `use_fast=True` will be the default behavior in v4.52, even if the model was saved with a slow processor. This will result in minor differences in outputs. You'll still be able to use a slow processor with `use_fast=False`.
A new version of the following files was downloaded from https://huggingface.co/jinaai/jina-clip-implementation:
- transform.py
. Make sure to double-check they do not contain any added malicious code. To avoid downloading new versions of the code file, you can pin a revision.
text -> image: 0.3140323
```
