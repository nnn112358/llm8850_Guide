# CLIP

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_clip

CLIP（Contrastive Language-Image Pre-Training）は、テキストと画像の意味的な関連性を学習するマルチモーダルモデルです。テキストによる画像検索などのタスクに活用でき、LLM-8850 上で推論を実行できます。

## モデルのダウンロード

手動でモデルをダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/LibCLIP
```

## ファイル説明

```
m5stack@raspberrypi:~/rsp/LibCLIP $ ls -lh
total 157M
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 11 09:09 cnclip
-rw-rw-r-- 1 m5stack m5stack 156M Aug 11 09:08 coco_1000.tar
-rw-rw-r-- 1 m5stack m5stack 3.8K Aug 11 09:08 config.json
-rw-rw-r-- 1 m5stack m5stack 779K Aug 11 09:08 gradio_01.png
drwxrwxr-x 5 m5stack m5stack 4.0K Aug 11 09:08 install
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 11 09:08 pyclip
-rw-rw-r-- 1 m5stack m5stack 6.0K Aug 11 09:08 README.md
```

## 仮想環境の作成

```bash
uv venv clip
```

## 仮想環境の有効化

```bash
source clip/bin/activate
```

## 依存パッケージのインストール

```bash
uv pip install -r pyclip/requirements.txt
```

## 環境変数の設定

```bash
export LD_PRELOAD=/usr/lib/
```

## 動的ライブラリのコピー

```bash
cp install/lib/aarch64/libclip.so pyclip/
```

## テスト画像の解凍

```bash
tar xf coco_1000.tar
```

## Web サービスの起動

以下のコマンドで、Gradio ベースの画像検索 Web サービスを起動します。

```bash
python pyclip/gradio_example.py --ienc cnclip/cnclip_vit_l14_336px_vision_u16u8.axmodel --tenc cnclip/cnclip_vit_l14_336px_text_u16.axmodel --vocab cnclip/cn_vocab.txt --isCN 1 --db_path clip_feat_db_coco --image_folder coco_1000/
```

起動が成功すると、以下のような情報が表示されます。

```
(clip) m5stack@raspberrypi:~/rsp/LibCLIP $ python pyclip/gradio_example.py --ienc cnclip/cnclip_vit_l14_336px_vision_u16u8.axmodel --tenc cnclip/cnclip_vit_l14_336px_text_u16.axmodel --vocab cnclip/cn_vocab.txt --isCN 1 --db_path clip_feat_db_coco --image_folder coco_1000/
Trying to load: /home/m5stack/rsp/LibCLIP/pyclip/aarch64/libclip.so

Failed to load: /home/m5stack/rsp/LibCLIP/pyclip/aarch64/libclip.so
   /home/m5stack/rsp/LibCLIP/pyclip/aarch64/libclip.so: cannot open shared object file: No such file or directory
File not found. Please verify that libclip.so exists and the path is correct.

Trying to load: /home/m5stack/rsp/LibCLIP/pyclip/libclip.so
open libax_sys.so failed
open libax_engine.so failed
Successfully loaded: /home/m5stack/rsp/LibCLIP/pyclip/libclip.so
可用設備: {'host': {'available': True, 'version': '', 'mem_info': {'remain': 0, 'total': 0}}, 'devices': {'host_version': 'V3.6.3_20250722020142', 'dev_version': 'V3.6.3_20250722020142', 'count': 1, 'devices_info': [{'temp': 62, 'cpu_usage': 1, 'npu_usage': 0, 'mem_info': {'remain': 7022, 'total': 7040}}]}}
[I][                             run][  31]: AXCLWorker start with devid 0

input size: 1
    name:    image [unknown] [unknown]
        1 x 3 x 336 x 336

output size: 1
    name: unnorm_image_features
        1 x 768

[I][              load_image_encoder][  50]: nchw 336 336
[I][              load_image_encoder][  60]: image feature len 768

input size: 1
    name:     text [unknown] [unknown]
        1 x 52

output size: 1
    name: unnorm_text_features
        1 x 768

[I][               load_text_encoder][  44]: text feature len 768
[I][                  load_tokenizer][  60]: text token len 52
100%|███████████████████████████████████| 1000/1000 [00:00<00:00, 163246.95it/s]
* Running on local URL:  http://0.0.0.0:7860
* To create a public link, set `share=True` in `launch()`.
```

ブラウザで http://127.0.0.1:7860 を開き、テキストボックスに検索したい内容を入力して、画像検索ボタンをクリックしてください。
