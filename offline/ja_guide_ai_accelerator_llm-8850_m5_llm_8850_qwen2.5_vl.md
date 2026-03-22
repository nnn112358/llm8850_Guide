# Qwen2.5-VL-3B-Instruct

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_qwen2.5_vl

Qwen2.5-VL-3B-Instruct は Alibaba Cloud が開発した約30億パラメータのマルチモーダル（視覚言語）モデルです。LLM-8850 上で画像および動画の理解・推論が可能です。

## モデルのダウンロード

モデルを手動でダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/Qwen2.5-VL-3B-Instruct
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/Qwen2.5-VL-3B-Instruct $ ls -lh
total 10M
-rw-rw-r-- 1 m5stack m5stack    0 Aug 12 16:37 config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:52 image
-rw-rw-r-- 1 m5stack m5stack 6.3M Aug 12 16:52 main
-rwxrwxr-x 1 m5stack m5stack  132 Aug 12 16:37 main_ax650
-rwxrwxr-x 1 m5stack m5stack 1.8M Aug 12 16:52 main_axcl_aarch64
-rwxrwxr-x 1 m5stack m5stack 1.8M Aug 12 16:52 main_axcl_x86
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:37 python
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:52 qwen2_5-vl-3b-image-ax650
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:47 Qwen2.5-VL-3B-Instruct-AX650-chunk_prefill_512
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:55 qwen2_5-vl-3b-video-ax650
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:37 qwen2_5-vl-tokenizer
-rw-rw-r-- 1 m5stack m5stack 9.0K Aug 12 16:37 qwen2_tokenizer_images.py
-rw-rw-r-- 1 m5stack m5stack  16K Aug 12 16:37 qwen2_tokenizer_video_308.py
-rw-rw-r-- 1 m5stack m5stack  14K Aug 12 16:37 README.md
-rwxrwxr-x 1 m5stack m5stack  708 Aug 12 16:37 run_qwen2_5_vl_image_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  704 Aug 12 16:37 run_qwen2_5_vl_image_axcl_x86.sh
-rwxrwxr-x 1 m5stack m5stack  780 Aug 12 16:37 run_qwen2_5_vl_image.sh
-rwxrwxr-x 1 m5stack m5stack  705 Aug 12 16:37 run_qwen2_5_vl_video_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  701 Aug 12 16:37 run_qwen2_5_vl_video_axcl_x86.sh
-rwxrwxr-x 1 m5stack m5stack  777 Aug 12 16:37 run_qwen2_5_vl_video.sh
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 16:37 video
```

> **ヒント:** 以前に `qwen` の仮想環境を作成済みの場合は、再作成は不要です。アクティベートするだけで構いません。

## 仮想環境の作成

推論に必要な Python 仮想環境を作成します。

```bash
uv venv qwen
```

### 仮想環境のアクティベート

```bash
source qwen/bin/activate
```

### 依存パッケージのインストール

```bash
uv pip install transformers jinja2
```

## 画像推論

このセクションでは、画像を入力としてモデルに推論させる手順を説明します。

### 画像 tokenizer パーサーの起動

以下のコマンドで画像用の tokenizer サービスを起動します。

```bash
python qwen2_tokenizer_images.py --port 12345
```

tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 12345 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(qwen) m5stack@raspberrypi:~/rsp/Qwen2.5-VL-3B-Instruct $ python qwen2_tokenizer_images.py --port 12345
None None 151645 <|im_end|>
[151644, 8948, 198, 2610, 525, 264, 10950, 17847, 13, ...]
281
[151644, 8948, 198, 2610, 525, 264, 10950, 17847, 13, 151645, 198, 151644, 872, 198, 14990, 1879, 151645, 198, 151644, 77091, 198]
21
http://localhost:12345
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

### 実行権限の設定

推論用の実行ファイルとスクリプトに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64 run_qwen2_5_vl_image_axcl_aarch64.sh
```

### Qwen2.5-VL モデル画像推論サービスの起動

以下のコマンドで画像推論サービスを起動します。

```bash
./run_qwen2_5_vl_image_axcl_aarch64.sh
```

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen2.5-VL-3B-Instruct $ ./run_qwen2_5_vl_image_axcl_aarch64.sh
[I][                            Init][ 162]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
bos_id: -1, eos_id: 151645
img_start_token: 151652
img_context_token: 151655
  2% | █                                 |   1 /  39 [0.01s<0.20s, 200.00 count/s] tokenizer init ok
  [I][Init][  45]: LLaMaEmbedSelector use mmap
100% | ████████████████████████████████ |  39 /  39 [72.58s<78.62s, 0.50 count/s] init post axmodel ok,remain_cmm(-1 MB)
input size: 1
    name: hidden_states [unknown] [unknown]
        1 x 1024 x 392 x 3 size:1204224

output size: 1
    name: hidden_states_out
        256 x 2048 size:2097152

[I][                            Init][ 267]: IMAGE_CONTEXT_TOKEN: 151655, IMAGE_START_TOKEN: 151652
[I][                            Init][ 328]: image encoder output float32

[I][                            Init][ 340]: max_token_len : 1023
[I][                            Init][ 343]: kv_cache_size : 256, kv_cache_num: 1023
[I][                            Init][ 351]: prefill_token_num : 128
[I][                            Init][ 355]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 355]: grp: 2, prefill_max_token_num : 128
[I][                            Init][ 355]: grp: 3, prefill_max_token_num : 256
[I][                            Init][ 355]: grp: 4, prefill_max_token_num : 384
[I][                            Init][ 355]: grp: 5, prefill_max_token_num : 512
[I][                            Init][ 359]: prefill_max_token_num : 512
________________________
|    ID| remain cmm(MB)|
========================
|     0|             -1|
¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯
[E][                     load_config][ 278]: config file(post_config.json) open failed
[W][                            Init][ 452]: load postprocess config(post_config.json) failed
[I][                            Init][ 456]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
prompt >> who are you?
image >>
[I][                             Run][ 625]: input token num : 23, prefill_split_num : 1
[I][                             Run][ 659]: input_num_token:23
[I][                             Run][ 796]: ttft: 558.68 ms
I am a large language model created by Alibaba Cloud. I am called Qwen.

[N][                             Run][ 949]: hit eos,avg 4.81 token/s

prompt >>
```

### 画像分析

画像ファイルのパスを指定することで、モデルが画像の内容を分析・説明します。

```
prompt >> describe this picture
image >> image/ssd_car.jpg
[I][                          Encode][ 539]: image encode time : 773.948975 ms, size : 524288
[I][                             Run][ 625]: input token num : 280, prefill_split_num : 3
[I][                             Run][ 659]: input_num_token:128
[I][                             Run][ 659]: input_num_token:128
[I][                             Run][ 659]: input_num_token:24
[I][                             Run][ 796]: ttft: 2076.95 ms
The picture shows a busy urban street scene in what appears to be a city center. A red double-decker bus is prominently featured in the foreground, with a large advertisement on its side that reads, "THINGS GET MORE EXITING WHEN YOU SAY 'YES' VirginMoney.co.uk." The bus is driving on the right side of the road, which is typical in the UK.

In the background, there are several pedestrians walking on the sidewalk, and a few cars are visible on the road. The buildings lining the street are modern and have large windows, suggesting a commercial or business district. The street is well-maintained, with clear pedestrian crossings and a clear separation between the road and the sidewalk.

The overall atmosphere of the image is bustling and typical of a busy city environment.

[N][                             Run][ 949]: hit eos,avg 4.83 token/s
```

## 動画推論

このセクションでは、動画フレームを入力としてモデルに推論させる手順を説明します。

> **ヒント:** 動画推論を開始する前に、先に前の画像用 tokenizer サービスを `CTRL + C` で停止する必要があります。

### 動画 tokenizer パーサーの起動

以下のコマンドで動画用の tokenizer サービスを起動します。

```bash
python qwen2_tokenizer_video_308.py --port 12345
```

tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 12345 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(qwen) m5stack@raspberrypi:~/rsp/Qwen2.5-VL-3B-Instruct $ python qwen2_tokenizer_video_308.py --port 12345
None None 151645 <|im_end|>
[151644, 8948, 198, 2610, 525, 264, 10950, 17847, 13, ...]
510
[151644, 8948, 198, 2610, 525, 264, 10950, 17847, 13, 151645, 198, 151644, 872, 198, 14990, 1879, 151645, 198, 151644, 77091, 198]
21
http://localhost:12345
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

### 実行権限の設定

推論用の実行ファイルとスクリプトに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64 run_qwen2_5_vl_video_axcl_aarch64.sh
```

### Qwen2.5-VL モデル動画推論サービスの起動

以下のコマンドで動画推論サービスを起動します。

```bash
./run_qwen2_5_vl_video_axcl_aarch64.sh
```

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen2.5-VL-3B-Instruct $ ./run_qwen2_5_vl_video_axcl_aarch64.sh
[I][                            Init][ 162]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
bos_id: -1, eos_id: 151645
img_start_token: 151652
img_context_token: 151656
  2% | █                                 |   1 /  39 [0.00s<0.12s, 333.33 count/s] tokenizer init ok
100% | ████████████████████████████████ |  39 /  39 [75.71s<77.70s, 0.50 count/s] init post axmodel ok,remain_cmm(3220 MB)
input size: 1
    name: hidden_states [unknown] [unknown]
        1 x 484 x 392 x 3 size:569184

output size: 1
    name: hidden_states_out
        121 x 2048 size:991232

[I][                            Init][ 267]: IMAGE_CONTEXT_TOKEN: 151656, IMAGE_START_TOKEN: 151652
[I][                            Init][ 328]: image encoder output float32

[I][                            Init][ 340]: max_token_len : 1023
[I][                            Init][ 343]: kv_cache_size : 256, kv_cache_num: 1023
[I][                            Init][ 351]: prefill_token_num : 128
[I][                            Init][ 355]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 355]: grp: 2, prefill_max_token_num : 128
[I][                            Init][ 355]: grp: 3, prefill_max_token_num : 256
[I][                            Init][ 355]: grp: 4, prefill_max_token_num : 384
[I][                            Init][ 355]: grp: 5, prefill_max_token_num : 512
[I][                            Init][ 359]: prefill_max_token_num : 512
________________________
|    ID| remain cmm(MB)|
========================
|     0|             -1|
¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯
[E][                     load_config][ 278]: config file(post_config.json) open failed
[W][                            Init][ 452]: load postprocess config(post_config.json) failed
[I][                            Init][ 456]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
prompt >> Describe the content of this video
image >> video
video/frame_0000.jpg
video/frame_0008.jpg
video/frame_0016.jpg
video/frame_0024.jpg
video/frame_0032.jpg
video/frame_0040.jpg
video/frame_0048.jpg
video/frame_0056.jpg
[I][                          Encode][ 539]: image encode time : 1480.802002 ms, size : 991232
[I][                             Run][ 625]: input token num : 511, prefill_split_num : 4
[I][                             Run][ 659]: input_num_token:128
[I][                             Run][ 659]: input_num_token:128
[I][                             Run][ 659]: input_num_token:128
[I][                             Run][ 659]: input_num_token:127
[I][                             Run][ 796]: ttft: 3032.74 ms
The video depicts two ground squirrels engaging in a playful interaction on a rocky path. The squirrels are facing each other, and their front paws are raised, as if they are playfully bumping or scratching each other. The background features a scenic mountain landscape with green hills and a clear blue sky, suggesting a sunny day. The squirrels' fur is a mix of brown, black, and white, and their tails are bushy and fluffy. The overall atmosphere of the video is light-hearted and playful, capturing a moment of natural behavior in a beautiful outdoor setting.

[N][                             Run][ 949]: hit eos,avg 4.86 token/s

prompt >>
```

## ベンチマーク

以下は LLM-8850 上での Qwen2.5-VL の推論パフォーマンスです。

| モデル | 量子化方式 | image encode (ms) | ttft (ms) | token/s |
|---|---|---|---|---|
| Qwen2.5-3B-VL | w8a16 | 773.95 | 558.68 | 4.81 |
