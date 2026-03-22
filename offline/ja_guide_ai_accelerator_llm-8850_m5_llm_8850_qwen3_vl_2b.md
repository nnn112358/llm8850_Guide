# Qwen3-VL-2B-Instruct

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_qwen3_vl_2b

Qwen3-VL-2B-Instruct は Alibaba Cloud が開発した約20億パラメータのマルチモーダル（視覚言語）モデルです。LLM-8850 上で画像および動画の理解・推論が可能です。

## モデルのダウンロード

[モデルを手動でダウンロード](https://huggingface.co/M5Stack/Qwen3-VL-2B-Instruct-axmodel)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/M5Stack/Qwen3-VL-2B-Instruct-axmodel
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/Qwen3-VL-2B-Instruct-axmodel $ ls -lh
total 7.4M
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 28 09:37 images
-rwxrwxr-x 1 m5stack m5stack 7.3M Nov  7 15:55 main_axcl_aarch64
-rwxrwxr-x 1 m5stack m5stack  276 Nov  7 11:58 post_config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 28 09:38 Qwen3-VL-2B-Instruct-ax8850
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 28 09:31 qwen3-vl-tokenizer
-rw-rw-r-- 1 m5stack m5stack   24 Nov  7 14:12 README.md
-rwxrwxr-x 1 m5stack m5stack  715 Nov  7 14:17 run_image_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  715 Nov  7 14:17 run_video_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack 9.4K Nov  7 14:42 tokenizer_images.py
-rwxrwxr-x 1 m5stack m5stack 9.3K Nov  7 14:41 tokenizer_video.py
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  7 11:26 video
```

> **ヒント:** 以前に `qwen` の仮想環境を作成済みの場合は、再作成は不要です。アクティベートするだけで使用できます。

## 仮想環境の作成

推論に必要な Python 仮想環境を作成します。

```bash
python -m venv qwen
```

## 仮想環境のアクティベート

```bash
source qwen/bin/activate
```

## 依存パッケージのインストール

```bash
pip install transformers jinja2
```

## 画像推論

このセクションでは、画像を入力としてモデルに推論させる手順を説明します。

### 画像 tokenizer パーサーの起動

以下のコマンドで画像用の tokenizer サービスを起動します。

```bash
python tokenizer_images.py
```

画像 tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 8080 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(qwen) m5stack@raspberrypi:~/rsp/Qwen3-VL-2B-Instruct-axmodel $ python tokenizer_images.py
None None 151645 <|im_end|>
[151644, 8948, 198, ...]
281
[151644, 8948, 198, ...]
21
http://localhost:8080
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

### 実行権限の設定

推論用の実行ファイルとスクリプトに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64 run_image_axcl_aarch64.sh
```

### Qwen3-VL モデル画像推論サービスの起動

以下のコマンドで画像推論サービスを起動します。

```bash
./run_image_axcl_aarch64.sh
```

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen3-VL-2B-Instruct-axmodel $ ./run_image_axcl_aarch64.sh
[I][                            Init][ 163]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:8080 ok
bos_id: -1, eos_id: 151645
img_start_token: 151652
img_context_token: 151655
...
100% | ████████████████████████████████ |  31 /  31 [61.71s<61.71s, 0.50 count/s] init post axmodel ok,remain_cmm(3341 MB)
...
[I][                            Init][ 457]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
prompt >> Describe the content of the image
image >> images/recoAll_attractions_1.jpg
[I][                     EncodeImage][ 531]: pixel_values size 1
[I][                     EncodeImage][ 532]: grid_h 24 grid_w 24
[I][                     EncodeImage][ 580]: image encode time : 191.645004 ms, size : 1
...
[I][                             Run][ 909]: ttft: 596.82 ms
This is a photograph of the Great Pyramids of Giza, the three largest and most famous pyramids in the world, located in the Giza Plateau in Egypt. The image captures the pyramids under a clear blue sky, with the vast, sandy desert landscape surrounding them. The pyramids are constructed from large stone blocks, and their massive, stepped structures are clearly visible. In the foreground, a few small figures can be seen, providing a sense of scale to the immense structures. The image is a clear and detailed depiction of the iconic ancient monuments.

[N][                             Run][1062]: hit eos,avg 7.80 token/s

prompt >>
```

## 動画推論

このセクションでは、動画フレームを入力としてモデルに推論させる手順を説明します。

### 動画 tokenizer パーサーへの切り替え

画像 tokenizer パーサーを `CTRL + C` で停止してから、動画用の tokenizer パーサーを起動します。

```bash
python tokenizer_video.py
```

動画 tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 8080 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(qwen) m5stack@raspberrypi:~/rsp/Qwen3-VL-2B-Instruct-axmodel $ python tokenizer_video.py
None None 151645 <|im_end|>
[151644, 8948, 198, ...]
281
[151644, 8948, 198, ...]
21
http://localhost:8080
```

### Qwen3-VL モデル動画推論サービスの起動

以下のコマンドで動画推論サービスを起動します。

```bash
./run_video_axcl_aarch64.sh
```

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen3-VL-2B-Instruct-axmodel $ ./run_video_axcl_aarch64.sh
[I][                            Init][ 163]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:8080 ok
bos_id: -1, eos_id: 151645
img_start_token: 151652
img_context_token: 151656
...
100% | ████████████████████████████████ |  31 /  31 [45.05s<46.55s, 0.67 count/s] init post axmodel ok,remain_cmm(3341 MB)
...
[I][                            Init][ 457]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
prompt >> Describe the content of the video
video >> video
video/out_0001.jpg
video/out_0002.jpg
video/out_0003.jpg
video/out_0004.jpg
video/out_0005.jpg
video/out_0006.jpg
video/out_0007.jpg
video/out_0008.jpg
video/out_0009.jpg
[I][                     EncodeImage][ 531]: pixel_values size 5
[I][                     EncodeImage][ 532]: grid_h 24 grid_w 24
[I][                     EncodeImage][ 580]: image encode time : 915.744019 ms, size : 5
...
[I][                             Run][ 909]: ttft: 2033.21 ms
The video captures a person walking through a modern, well-lit hallway. The individual, seen from behind, is wearing a white t-shirt with a blue graphic on the back and dark pants. They are moving towards a glass door, which is part of a larger set of doors, possibly in a building's entrance or a corridor.

The hallway features dark, polished tiles with a subtle, elegant pattern. A green emergency exit sign is visible on the wall, indicating the direction to the exit. To the right of the person, a laptop is placed on a stand, its screen displaying a blue image, possibly a video or a live feed. The laptop is connected to a cable, suggesting it might be used for monitoring or communication purposes.

The overall atmosphere of the scene is clean, professional, and modern, with a focus on safety and technology. The person's movement and the surrounding environment suggest a routine activity, such as entering a building or checking on a system.

[N][                             Run][1062]: hit eos,avg 7.77 token/s

prompt >>
```
