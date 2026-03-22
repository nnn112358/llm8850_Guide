# Qwen3-VL-4B-Instruct-GPTQ-Int4

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_qwen3_vl_4b

Qwen3-VL-4B-Instruct-GPTQ-Int4 は Alibaba Cloud が開発した約40億パラメータのマルチモーダル（視覚言語）モデルの GPTQ Int4 量子化版です。LLM-8850 上で高精度な画像および動画の理解・推論が可能です。

## モデルのダウンロード

[モデルを手動でダウンロード](https://huggingface.co/M5Stack/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/M5Stack/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel $ ls -lh
total 7.4M
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  7 11:48 images
-rwxrwxr-x 1 m5stack m5stack 7.3M Nov  7 15:55 main_axcl_aarch64
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  7 11:49 Qwen3-VL-4B-Instruct-ax8850
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  7 11:42 qwen3-vl-tokenizer
-rw-rw-r-- 1 m5stack m5stack   24 Nov  7 16:02 README.md
-rwxrwxr-x 1 m5stack m5stack  715 Nov  7 16:19 run_image_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  715 Nov  7 16:20 run_video_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack 9.4K Nov  7 14:42 tokenizer_images.py
-rwxrwxr-x 1 m5stack m5stack 9.3K Nov  7 14:41 tokenizer_video.py
drwxr-xr-x 2 m5stack m5stack 4.0K Nov  7 12:04 video
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
(qwen) m5stack@raspberrypi:~/rsp/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel $ python tokenizer_images.py
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
m5stack@raspberrypi:~/rsp/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel $ ./run_image_axcl_aarch64.sh
[I][                            Init][ 163]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:8080 ok
bos_id: -1, eos_id: 151645
img_start_token: 151652
img_context_token: 151655
...
100% | ████████████████████████████████ |  39 /  39 [81.89s<84.05s, 0.46 count/s] init post axmodel ok,remain_cmm(1894 MB)
...
[I][                            Init][ 457]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
prompt >> Describe the content of the image
image >> images/recoAll_attractions_1.jpg
[I][                     EncodeImage][ 531]: pixel_values size 1
[I][                     EncodeImage][ 532]: grid_h 24 grid_w 24
[I][                     EncodeImage][ 580]: image encode time : 196.322998 ms, size : 1
...
[I][                             Run][ 909]: ttft: 984.20 ms
This image captures a classic view of the Giza Pyramids in Egypt, set against a vast, clear blue sky.

In the foreground, the desert sands stretch out, with a few small figures of people visible, providing a sense of scale. The most prominent structure is the Great Pyramid of Giza, its massive stone blocks forming a steep, triangular silhouette. To its left, the smaller Pyramid of Khafre is visible, and to its right, the Pyramid of Menkaure can be seen, though less distinct.

The pyramids are bathed in bright sunlight, casting long, dark shadows that emphasize their monumental scale and the precision of their construction. The clear, cloudless sky creates a dramatic contrast with the ancient, enduring stone, evoking a sense of timeless grandeur and mystery.

This scene is one of the most iconic and recognizable in the world, symbolizing the enduring legacy of ancient Egypt and the awe-inspiring achievements of its civilization.

[N][                             Run][1062]: hit eos,avg 5.92 token/s

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
(qwen) m5stack@raspberrypi:~/rsp/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel $ python tokenizer_video.py
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
m5stack@raspberrypi:~/rsp/Qwen3-VL-4B-Instruct-GPTQ-Int4-axmodel $ ./run_video_axcl_aarch64.sh
[I][                            Init][ 163]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:8080 ok
bos_id: -1, eos_id: 151645
img_start_token: 151652
img_context_token: 151656
...
100% | ████████████████████████████████ |  39 /  39 [67.42s<67.42s, 0.58 count/s] init post axmodel ok,remain_cmm(1894 MB)
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
[I][                     EncodeImage][ 580]: image encode time : 938.083008 ms, size : 5
...
[I][                             Run][ 909]: ttft: 3385.97 ms
This video appears to be a static, low-angle shot of an indoor hallway or corridor, likely in a modern building or office. The scene is composed of several key elements:

- **The Setting**: The hallway is lined with dark, polished, reflective tiles that create a sleek, contemporary look. The reflections on the floor and walls add depth to the scene. On the right wall, there are two small, green-lit exit signs, indicating a fire safety feature. Below them, a red fire extinguisher is mounted on the wall, and a small, white control panel or thermostat is also visible.

- **The People**: Two individuals are present in the frame. One, wearing a white t-shirt with a blue graphic and dark pants, is walking away from the camera, moving toward the elevator or stairwell. The second person, dressed in a dark shirt and shorts, is also walking away, slightly to the right of the first person. Their movement is captured in motion, suggesting the video was taken in a moment of activity.

- **The Technology**: In the foreground, there are three laptops, each displaying a blue screen with a glowing, abstract image that resembles a jellyfish or a similar organic form. The laptops are placed on a gray, curved stand or cart, which is positioned near the wall. The laptops are connected to a network of cables, suggesting they are part of a monitoring or surveillance system. The laptops are not in use, as they are displaying a static image, which may indicate they are in standby mode or are being used for a specific purpose, such as monitoring or recording.

- **The Overall Atmosphere**: The scene is calm and uncluttered, with a focus on the technology and the movement of the people. The lighting is soft and ambient, and the overall mood is one of quiet observation, as if the viewer is being invited to watch the scene unfold.

The video seems to be a snapshot of a moment in time, capturing the interplay between human activity and technology in a modern, well-maintained environment. The presence of the laptops and the fire safety equipment suggests that this is a space that is monitored and maintained for safety and efficiency.

[N][                             Run][1062]: hit eos,avg 5.94 token/s

prompt >>
```
