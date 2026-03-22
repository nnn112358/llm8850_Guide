# SDK サンプル

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_samples

このページでは、AXCL SDK に含まれる各種サンプルプログラムの使用方法について説明します。

## runtime

### axcl_sample_runtime

Runtime API の基本的な使い方を示すサンプルです。以下の手順でランタイムの初期化からデバイスの操作までを行います。

1. `axclInit` でランタイムを初期化
2. `axclrtSetDevice` でデバイスを有効化
3. main スレッド用に `axclrtCreateDevice` でコンテキストを作成（オプション）
4. スレッドコンテキストの作成と破棄（必須）
5. main スレッドのコンテキストを破棄
6. `axclrtResetDevice` でデバイスを無効化
7. `axclFinalize` でランタイムを非初期化

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_runtime --help
[INFO ][                            main][  36]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:26 =============
usage: axcl_sample_runtime [options] ...
options:
  -d, --device    device index [-1, connected device num - 1], -1: traverse all devices (int [=-1])
      --json      axcl.json path (string [=./axcl.json])
      --reboot    reboot device
  -?, --help      print this message
```

| パラメータ | デフォルト値 | 説明 |
|---|---|---|
| -d, --device | -1、範囲: [-1, 0 - 接続デバイス数 -1] | デバイスインデックスを指定。-1: 全デバイスを走査 |
| --json | axcl.json | 設定ファイルパス |
| --reboot | NA | デバイスを Panic させて再起動 |

#### 例 1

デバイス #0 の属性をクエリします：

```
m5stack@raspberrypi5:~ $ axcl_sample_runtime -d 0
[INFO ][                            main][  36]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:26 =============
json file
[INFO ][                      operator()][ 130]: [0001:01.0] software version: Ax_Version V3.6.3_20250722020142

[INFO ][                      operator()][ 131]: [0001:01.0] uid             : 0x600408144b65d204
[INFO ][                      operator()][ 132]: [0001:01.0] temperature     : 49263
[INFO ][                      operator()][ 133]: [0001:01.0] total mem size  : 968356   KB
[INFO ][                      operator()][ 134]: [0001:01.0] free  mem size  : 815084   KB
[INFO ][                      operator()][ 135]: [0001:01.0] total cmm size  : 7208960  KB
[INFO ][                      operator()][ 136]: [0001:01.0] free  cmm size  : 7190084  KB
[INFO ][                      operator()][ 137]: [0001:01.0] avg cpu loading : 0.0%
[INFO ][                      operator()][ 138]: [0001:01.0] avg npu loading : 0.0%
[INFO ][                      operator()][ 149]: malloc 1048576 bytes memory from device  1 success, addr = 0x14926f000
[INFO ][                            main][ 215]: ============== V3.6.3_20250722020142 sample exited Jul 22 2025 02:31:26 ==============
```

#### 例 2

`echo c > /proc/sysrq-trigger` を送信してデバイス #0 を手動で Panic させ、再起動してファームウェアを再ダウンロードします：

```
m5stack@raspberrypi5:~ $ axcl_sample_runtime -d 0 --reboot
[INFO ][                            main][  36]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:26 =============
json file
[INFO ][                      operator()][ 130]: [0001:01.0] software version: Ax_Version V3.6.3_20250722020142
...
[INFO ][                            main][ 178]: panic device 0x1
[INFO ][                            main][ 185]: wait for device 0x1 to reboot in 60 seconds
[FAIL ][                            main][ 203]: device 0x1 reboot timeout
[INFO ][                            main][ 215]: ============== V3.6.3_20250722020142 sample exited Jul 22 2025 02:31:26 ==============
```

---

### axcl_sample_memory

ホスト（HOST）とデバイス（DEVICE）間のメモリコピーを行うサンプルです。以下のフローでデータの転送と検証を行います。

```
HOST          |               DEVICE
      host_mem[0] -----------> dev_mem[0]
                                    |---------> dev_mem[1]
      host_mem[1] <----------------------------------|
```

1. `axclrtMallocHost` で host_mem[2] を、`axclrtMalloc` で dev_mem[2] を確保
2. `axclrtMemcpy(dev_mem[0], host_mem[0], size, AXCL_MEMCPY_HOST_TO_DEVICE)`
3. `axclrtMemcpy(dev_mem[1], dev_mem[0], size, AXCL_MEMCPY_DEVICE_TO_DEVICE)`
4. `axclrtMemcpy(host_mem[1], dev_mem[1], size, AXCL_MEMCPY_DEVICE_TO_HOST)`
5. `memcmp(host_mem[0], host_mem[1], size)`

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_memory --help
[INFO ][                            main][  32]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:27 ==============

usage: axcl_sample_memory [options] ...
options:
  -d, --device    device index from 0 to connected device num - 1 (int [=0])
      --json      axcl.json path (string [=./axcl.json])
  -?, --help      print this message
```

| パラメータ | デフォルト値 | 説明 |
|---|---|---|
| -d, --device | 0、範囲: [0 - 接続デバイス数 -1] | デバイスインデックスを指定 |
| --json | axcl.json | 設定ファイルパス |

#### 例

```
m5stack@raspberrypi5:~ $ axcl_sample_memory  -d 0
[INFO ][                            main][  32]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:27 ==============

[INFO ][                           setup][ 112]: json: ./axcl.json
json file
[INFO ][                           setup][ 131]: device index: 0, bus number: 1
[INFO ][                            main][  51]: alloc host and device memory, size: 0x800000
[INFO ][                            main][  63]: memory [0]: host 0x7fff277fc010, device 0x14926f000
[INFO ][                            main][  63]: memory [1]: host 0x7fff26ff8010, device 0x149a6f000
[INFO ][                            main][  69]: memcpy from host memory[0] 0x7fff277fc010 to device memory[0] 0x14926f000
[INFO ][                            main][  75]: memcpy device memory[0] 0x14926f000 to device memory[1] 0x149a6f000
[INFO ][                            main][  81]: memcpy device memory[1] 0x149a6f000 to host memory[0] 0x7fff26ff8010
[INFO ][                            main][  88]: compare host memory[0] 0x7fff277fc010 and host memory[1] 0x7fff26ff8010 success
[INFO ][                         cleanup][ 146]: deactive device 1 and cleanup axcl
[INFO ][                            main][ 106]: ============== V3.6.3_20250722020142 sample exited Jul 22 2025 02:31:27 ==============
```

---

## native

### axcl_sample_sys

ネイティブ SYS モジュールの使用例です。以下の機能を含みます：

- 非キャッシュ CMM メモリの確保と解放
- キャッシュ CMM メモリの確保と解放
- 共通メモリプールの確保と解放
- プライベートメモリプールの確保と解放
- モジュールリンクとクエリ

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_sys --help
usage: axcl_sample_sys [options] ...
options:
  -d, --device    device index from 0 to connected device num - 1 (unsigned int [=0])
      --json      axcl.json path (string [=./axcl.json])
  -?, --help      print this message
```

| パラメータ | デフォルト値 | 説明 |
|---|---|---|
| -d, --device | 0、範囲: [0 - 接続デバイス数 -1] | デバイスインデックスを指定 |
| --json | axcl.json | 設定ファイルパス |

#### 例

```
m5stack@raspberrypi5:~ $ axcl_sample_sys -d 0
[INFO ][                            main][  35]: json: ./axcl.json
json file
[INFO ][                            main][  55]: device index: 0, bus number: 1
[INFO ][           sample_sys_alloc_free][  82]: sys_alloc_free begin
[INFO ][           sample_sys_alloc_free][  91]: alloc PhyAddr= 0x14926f000,pVirAddr=0xffffafafe000
...
[INFO ][           sample_sys_alloc_free][ 103]: sys_alloc_free end success
[INFO ][     sample_sys_alloc_cache_free][ 115]: sys_alloc_cache_free begin
...
[INFO ][     sample_sys_alloc_cache_free][ 136]: sys_alloc_cache_free end success
[INFO ][          sample_sys_commom_pool][ 148]: sys_commom_pool begin
...
[INFO ][          sample_sys_commom_pool][ 288]: sys_commom_pool end success!
[INFO ][         sample_sys_private_pool][ 310]: sys_private_pool begin
...
[INFO ][         sample_sys_private_pool][ 451]: sys_private_pool end success!
[INFO ][                 sample_sys_link][ 472]: sample_sys_link begin
...
[INFO ][                 sample_sys_link][ 557]: sample_sys_link end success!
```

---

### axcl_sample_vdec

H264、H265 ビデオデコーダ（VDEC）のサンプルです。以下の処理を行います：

- `.mp4` ファイルまたは `.h264`/`.h265` 生ストリームファイルをロード
- FFmpeg で nalu フレームをデマルチプレクス
- nalu をフレーム単位でビデオデコーダに送信
- デコードされた NV12 画像情報を受信

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_vdec --help
[INFO ][                            main][  43]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:31 ==============

usage: axcl_sample_vdec --url=string [options] ...
options:
  -i, --url       mp4|.264|.265 file path (string)
  -d, --device    device index from 0 to connected device num - 1 (unsigned int [=0])
      --count     grp count (int [=1])
      --json      axcl.json path (string [=./axcl.json])
  -w, --width     frame width (int [=1920])
  -h, --height    frame height (int [=1080])
      --VdChn     channel id (int [=0])
      --yuv       transfer nv12 from device (int [=0])
  -?, --help      print this message
```

| パラメータ | デフォルト値 | 説明 |
|---|---|---|
| -d, --device | 0、範囲: [0 - 接続デバイス数 -1] | デバイスインデックスを指定 |
| --json | axcl.json | 設定ファイルパス |
| -i, --url | NA | ストリームファイルのパスを指定 |
| --count | 1 | グループ数を指定 |
| -w, --width | 1920 | デコードされた YUV 画像の幅を指定 |
| -h, --height | 1080 | デコードされた YUV 画像の高さを指定 |
| --VdChn | 0 | VDEC のチャンネルを指定。0: 入力ストリームの元の解像度で出力。1: 出力範囲 [48x48, 4096x4096]、スケーリング対応。2: 出力範囲 [48x48, 1920x1080]、スケーリング対応 |
| --yuv | 0 | 1: デバイスからデコード済み YUV 画像を転送。0: 転送しない |

> **注意:** 以下のエラーが発生した場合:
> ```
> $ axcl_sample_vdec --help
> axcl_sample_vdec: error while loading shared libraries: libavcodec.so.61: cannot open shared object file: No such file or directory
> ```
> まず以下のコマンドで環境変数を設定してください：
> ```bash
> $ export LD_LIBRARY_PATH=/usr/lib/axcl/ffmpeg:$LD_LIBRARY_PATH
> ```

#### 例

```
m5stack@raspberrypi5:~ $ axcl_sample_vdec -i input.mp4 -d 0
[INFO ][                            main][  43]: ============== V3.6.3_20250722020142 sample started Jul 22 2025 02:31:31 ==============

[INFO ][                            main][  68]: json: ./axcl.json
json file
[INFO ][                            main][  88]: device index: 0, bus number: 1
[INFO ][             ffmpeg_init_demuxer][ 439]: [0] url: input.mp4
[INFO ][             ffmpeg_init_demuxer][ 502]: [0] url input.mp4: codec 96, 1280x688, fps 30
...
[INFO ][             ffmpeg_demux_thread][ 435]: [0] demuxed    total 281 frames ---
[INFO ][          ffmpeg_dispatch_thread][ 272]: [0] dispatched total 281 frames ---
[WARN ][ sample_get_decoded_image_thread][ 357]: [decoder  0] flow end
[INFO ][ sample_get_decoded_image_thread][ 392]: [decoder  0] total decode 281 frames
...
[INFO ][                            main][ 247]: ============== V3.6.3_20250722020142 sample exited Jul 22 2025 02:31:31 ==============
```

---

### axcl_sample_venc

H264、H265、JPEG、MJPEG エンコーダ（VENC）のサンプルです。

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_venc --help
[INFO][SAMPLE-VENC][main][39]: Build at Jul 22 2025 02:31:34

Usage:  axcl_sample_venc [options] -i input file

  -H --help                        help information

## Options for sample

  -i[s] --input                         Read input video sequence from file. [input.yuv]
  -o[s] --output                        Write output HEVC/H.264/jpeg/mjpeg stream to file.[stream.hevc]
  -W[n] --write                         whether write output stream to file.[1]
  -f[n] --dstFrameRate                  1..1048575 Output picture rate numerator. [30]
  -j[n] --srcFrameRate                  1..1048575 Input picture rate numerator. [30]
  -n[n] --encFrameNum                   the frame number want to encode. [0]
  -N[n] --chnNum                        total encode channel number. [0]
  -t[n] --encThdNum                     total encode thread number. [1]
  -p[n] --bLoopEncode                   enable loop mode to encode. 0: disable. 1: enable. [0]
  --codecType                           encoding payload type. [0]
                                        0 - SAMPLE_CODEC_H264
                                        1 - SAMPLE_CODEC_H265
                                        2 - SAMPLE_CODEC_MJPEG
                                        3 - SAMPLE_CODEC_JPEG
  ...
```

#### 例 1

2 チャンネルで 1080p NV12 フォーマットをエンコードします（チャンネル 0: H.264、チャンネル 1: H.265）：

```bash
$ axcl_sample_venc -w 1920 -h 1080 -i 1080p_nv12.yuv -N 2 -l 3
```

#### 例 2

2 チャンネルでループエンコード 3840x2160 NV21 フォーマット（チャンネル 0: H.264、チャンネル 1: H.265）、10 フレームエンコード：

```bash
$ axcl_sample_venc -w 3840 -h 2160 -i 3840x2160_nv21.yuv -N 2 -l 4 -n 10
```

#### 例 3

MJPEG ストリームをエンコードします（解像度 1920x1080、YUV420P フォーマット、5 フレーム）：

```bash
$ axcl_sample_venc -w 1920 -h 1080 -i 1920x1080_yuv420p.yuv -N 1 --bChnCustom 1 --codecType 2 -l 1 -n 5
```

---

### axcl_sample_dmadim

DMA（Direct Memory Access）のサンプルです。以下の操作を含みます：

- `AXCL_DMA_MemCopy` で 2 つのデバイスメモリ間のメモリコピー（memcpy）
- `AXCL_DMA_MemCopy` でデバイスメモリを 0xAB に設定（memset）
- `AXCL_DMA_CheckSum` でチェックサムを計算
- `AXCL_DMA_MemCopyXD`（`AX_DMADIM_2D`）で (0, 0) から 1/4 画像を切り取り

#### 使用方法

```
usage: ./axcl_sample_dmadim --image=string --width=unsigned int --height=unsigned int [options] ...
options:
  -d, --device    device index from 0 to connected device num - 1 (unsigned int [=0])
  -i, --image     nv12 image file path (string)
  -w, --width     width of nv12 image (unsigned int)
  -h, --height    height of nv12 image (unsigned int)
      --json      axcl.json path (string [=./axcl.json])
  -?, --help      print this message
```

| パラメータ | デフォルト値 | 説明 |
|---|---|---|
| -d, --device | 0、範囲: [0 - 接続デバイス数 -1] | デバイスインデックスを指定 |
| --json | axcl.json | 設定ファイルパス |
| -i, --image | NA | 入力画像ファイルパス |
| -w, --width | NA | 入力画像の幅 |
| -h, --height | NA | 入力画像の高さ |

#### 例

```
$ axcl_sample_dmadim -i 1920x1080.nv12.yuv -w 1920 -h 1080 -d 0
[INFO ][                            main][  30]: ============== V3.5.0_20250515190238 sample started May 15 2025 19:29:17 ==============

[INFO ][                            main][  46]: json: ./axcl.json
json file
[INFO ][                            main][  66]: device index: 0, bus number: 3
[INFO ][                        dma_copy][ 119]: dma copy device memory succeed, from 0x14926f000 to 0x14966f000
[INFO ][                      dma_memset][ 139]: dma memset device memory succeed, 0x14926f000 to 0xab
[INFO ][                    dma_checksum][ 166]: dma checksum succeed, checksum = 0xaaa00000
[INFO ][                      dma_copy2d][ 281]: [0] dma memcpy 2D succeed
[INFO ][                      dma_copy2d][ 281]: [1] dma memcpy 2D succeed
[INFO ][                      dma_copy2d][ 308]: ./dma2d_output_image_960x540.nv12 is saved
[INFO ][                      dma_copy2d][ 328]: dma copy2d nv12 image pass
[INFO ][                            main][  82]: ============== V3.5.0_20250515190238 sample exited May 15 2025 19:29:17 ==============
```

---

### axcl_sample_ive

インテリジェントビデオエンジン（IVE）のサンプルです。画像処理の各種アルゴリズムの使い方を示します。

> **注意:**
> - サンプルコードは API デモ用です。実際には、ユーザーのコンテキストに応じた具体的な設定パラメータを使用する必要があります。
> - パラメータ制限については `42 - AX IVE API` のドキュメントを参照してください。
> - 入出力データのメモリはユーザーが確保する必要があります。
> - 入出力の画像データはユーザーが指定する必要があります。
> - 2 次元画像のデータ型は明示的に定義するか、デフォルト値を使用する必要があります。
> - これらのキーパラメータは JSON 文字列または JSON ファイルとしてフォーマットされます。`/opt/data/ive/` の各ディレクトリにある `.json` ファイルとコードを参照してください。

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_ive --help
Usage : axcl_sample_ive -c case_index [options]
        -d | --device_id: Device index from 0 to connected device num - 1, optional
        -c | --case_index:Calc case index, default:0
                0-DMA.
                1-DualPicCalc.
                2-HysEdge and CannyEdge.
                3-CCL.
                4-Erode and Dilate.
                5-Filter.
                6-Hist and EqualizeHist.
                7-Integ.
                8-MagAng.
                9-Sobel.
                10-GMM and GMM2.
                11-Thresh.
                12-16bit to 8bit.
                13-Multi Calc.
                14-Crop and Resize.
                15-CSC.
                16-CropResize2.
                17-MatMul.
        -e | --engine_choice:Choose engine id, default:0
                0-IVE; 1-TDP; 2-VGP; 3-VPP; 4-GDC; 5-DSP; 6-NPU; 7-CPU; 8-MAU.
        ...
        -? | --help:Show usage help.
```

#### 例 1

DMA の使用方法（ソース解像度: 1280 x 720、入出力タイプ: U8C1、JSON ファイルで制御パラメータを設定）：

```bash
$ axcl_sample_ive -c 0 -w 1280 -h 720 -i /opt/data/ive/common/1280x720_u8c1_gray.yuv -o /opt/data/ive/dma/ -t 0 0 -p /opt/data/ive/dma/dma.json
```

#### 例 2

MagAndAng の使用方法（ソース解像度: 1280 x 720、入力パラメータ (grad_h, grad_v) データ型: U16C1、出力パラメータ (ang_output) データ型: U8C1）：

```bash
$ axcl_sample_ive -c 8 -w 1280 -h 720 -i /opt/data/ive/common/1280x720_u16c1_gray.yuv /opt/data/ive/common/1280x720_u16c1_gray_2.yuv -o /opt/data/ive/common/mag_output.bin /opt/data/ive/common/ang_output.bin -t 9 9 9 0
```

#### JSON 設定ファイル

各種 JSON 設定ファイルのパラメータの詳細を以下に示します：

- **dma.json**: `mode`, `x0`, `y0`, `h_seg`, `v_seg`, `elem_size`, `set_val` は `AX_IVE_DMA_CTRL_T` 構造体の各メンバの値です。`w_out` と `h_out` は出力画像の幅と高さ（`AX_IVE_DMA_MODE_DIRECT_COPY` モードの DMA 用）です。
- **dualpics.json**: `x` と `y` は ADD CV の `AX_IVE_ADD_CTRL_T` 構造体の `u1q7X` と `u1q7Y` の値です。`mode` は Sub CV の `AX_IVE_SUB_CTRL_T` 構造体の `enMode` の値です。`mse_coef` は MSE CV の `AX_IVE_MSE_CTRL_T` 構造体の `u1q15MseCoef` の値です。
- **ccl.json**: `mode` は CCL CV の `AX_IVE_CCL_CTRL_T` 構造体の `enMode` の値です。
- **ed.json**: `mask` は Erode CV の `AX_IVE_ERODE_CTRL_T` または Dilate CV の `AX_IVE_DILATE_CTRL_T` 構造体の `au8Mask[25]` の全値です。
- **filter.json**: `mask` は Filter CV の `AX_IVE_FILTER_CTRL_T` 構造体の `as6q10Mask[25]` の全値です。
- **hist.json**: `histeq_coef` は EqualizeHist CV の `AX_IVE_EQUALIZE_HIST_CTRL_T` 構造体の `u0q20HistEqualCoef` の値です。
- **integ.json**: `out_ctl` は Integ CV の `AX_IVE_INTEG_CTRL_T` 構造体の `enOutCtrl` の値です。
- **sobel.json**: `mask` は Sobel CV の `AX_IVE_SOBEL_CTRL_T` 構造体の `as6q10Mask[25]` の値です。
- **gmm.json**: `init_var`, `min_var`, `init_w`, `lr`, `bg_r`, `var_thr`, `thr` はそれぞれ GMM CV の `AX_IVE_GMM_CTRL_T` 構造体の各メンバの値です。
- **gmm2.json**: `init_var`, `min_var`, `max_var`, `lr`, `bg_r`, `var_thr`, `var_thr_chk`, `ct`, `thr` はそれぞれ GMM2 CV の `AX_IVE_GMM2_CTRL_T` 構造体の各メンバの値です。
- **thresh.json**: `mode`, `thr_l`, `thr_h`, `min_val`, `mid_val`, `max_val` はそれぞれ Thresh CV の `AX_IVE_THRESH_CTRL_T` 構造体の各メンバの値です。
- **16bit_8bit.json**: `mode`, `gain`, `bias` はそれぞれ 16BitTo8Bit CV の `AX_IVE_16BIT_TO_8BIT_CTRL_T` 構造体の各メンバの値です。
- **crop_resize.json**: CropImage が有効な場合、`num` は `AX_IVE_CROP_IMAGE_CTRL_T` 構造体の `u16Num` の値で、`boxs` はクロップ画像の配列型です。CropResize または CropResizeForSplitYUV モードが有効な場合、`num` は `AX_IVE_CROP_RESIZE_CTRL_T` 構造体の `u16Num` の値です。
- **crop_resize2.json**: `num` は `AX_IVE_CROP_IMAGE_CTRL_T` 構造体の `u16Num` の値で、`res_out` は出力画像の幅と高さの配列、`src_boxs` はソース画像からのクロップ範囲の配列、`dst_boxs` はリサイズ画像への範囲の配列です。
- **matmul.json**: `mau_i`, `ddr_rdw`, `en_mul_res`, `en_topn_res`, `order`, `topn` はそれぞれ `AX_IVE_MAU_MATMUL_CTRL_T` 構造体の各メンバの値です。

---

### axcl_sample_ivps

画像ビデオ処理システム（IVPS）のサンプルです。クロップ、スケーリング、回転、ストリーミング、CSC、OSD、モザイク、四角形描画などの機能を提供します。

#### 使用方法

```
m5stack@raspberrypi5:~ $ axcl_sample_ivps -h
AXCL IVPS Sample. Build at Jul 22 2025 02:31:34
Usage: /axcl_sample_ivps
        -d             (required) : device index from 0 to connected device num - 1
        -v             (required) : video frame input
        -g             (optional) : overlay input
        -s             (optional) : sp alpha input
        -n             (optional) : repeat number
        -r             (optional) : region config and update
        -l             (optional) : 0: no link 1. link ivps. 2: link venc. 3: link jenc
        --pipeline     (optional) : import ini file to config all the filters in one pipeline
        --pipeline_ext (optional) : import ini file to config all the filters in another pipeline
        --change       (optional) : import ini file to change parameters for one filter dynamicly
        --region       (optional) : import ini file to config region parameters
        --dewarp       (optional) : import ini file to config dewarp parameters
        --cmmcopy      (optional) : cmm copy API test
        --csc          (optional) : color space covert API test
        --fliprotation (optional) : flip and rotation API test
        --alphablend   (optional) : alpha blending API test
        --cropresize   (optional) : crop resize API test
        --osd          (optional) : draw osd API test
        --cover        (optional) : draw line/polygon API test
        -a             (optional) : all the sync API test
        --json         (optional) : axcl.json path
```

> **注意:**
> - `-v` は必須で、入力ソース画像のパスとフレーム情報を指定します。
> - クロップウィンドウはソース画像の範囲内である必要があります（CropX0 + CropW <= Stride, CropY0 + CropH <= Height）。
> - クロップしない場合: CropW = Width, CropH = Height, CropX0 = 0, CropY0 = 0。
> - `-n` はソース画像に対する処理回数を指定します。-1 を指定すると無限ループで実行されます。
> - IVPS の proc 情報を確認する場合は、処理回数を大きな値に設定するか無限ループにする必要があります。
> - `-r` は重畳する REGION 数を指定します（現在最大 4）。REGION の重畳は非同期操作のため、数フレーム後に実際に重畳されます。

#### 例 1

ソース画像（3840x2160 NV12 フォーマット）を 1 回処理します：

```bash
$ axcl_sample_ivps -v /opt/data/ivps/3840x2160.nv12@3@3840x2160@0x0+0+0 -d 0 -n 1
```

#### 例 2

ソース画像（800x480 RGB888 フォーマット）をクロップ（X0=128 Y0=50 W=400 H=200）して 3 回処理します：

```bash
$ axcl_sample_ivps -v /opt/data/ivps/800x480logo.rgb24@161@800x480@400x200+128+50 -d 0 -n 3
```

#### 例 3

ソース画像（3840x2160 NV12 フォーマット）を 5 回処理し、3 つの REGION を重畳します：

```bash
$ axcl_sample_ivps -v /opt/data/ivps/3840x2160.nv12@3@3840x2160@0x0+0+0 -d 0 -n 5 -r 3
```

---

### axcl_sample_transcode

トランスコードサンプルです。以下のパイプライン（PPL）で動画のトランスコードを行います：

1. `.mp4` ファイルまたは `.h264`/`.h265` 生ストリームファイルをロード
2. FFmpeg で nalu をデマルチプレクス
3. nalu フレームを VDEC に送信
4. VDEC がデコードした NV12 を IVPS に送信（リサイズする場合）
5. IVPS が NV12 を VENC に送信
6. VENC がエンコードした nalu フレームをホストに送信

```
| ----------------------------- |
| sample                        |
| ----------------------------- |
| libaxcl_ppl.so                |
| ----------------------------- |
| libaxcl_lite.so               |
| ----------------------------- |
| AXCL SDK                      |
| ----------------------------- |
```

#### 使用方法

```
$ axcl_sample_transcode --help
usage: axcl_sample_transcode --url=string [options] ...
options:
  -i, --url       mp4|.264|.265 file path (string)
  -d, --device    device index from 0 to connected device num - 1 (unsigned int [=0])
  -w, --width     output width, 0 means same as input (unsigned int [=0])
  -h, --height    output height, 0 means same as input (unsigned int [=0])
      --codec     encoded codec: [h264 | h265] (default: h265) (string [=h265])
      --json      axcl.json path (string [=./axcl.json])
      --loop      1: loop demux for local file  0: no loop(default) (int [=0])
      --dump      dump file path (string [=])
      --hwclk     decoder hw clk, 0: 624M, 1: 500M, 2: 400M(default) (unsigned int [=2])
      --ut        unittest
  -?, --help      print this message
```

| パラメータ | デフォルト値 | 説明 |
|---|---|---|
| -d, --device | 0、範囲: [0 - 接続デバイス数 -1] | デバイスインデックスを指定 |
| --json | axcl.json | 設定ファイルパス |
| -i, --url | NA | ストリームファイルのパスを指定 |
| -w, --width | 0 | 出力 YUV 画像の幅（0: 入力と同一） |
| -h, --height | 0 | 出力 YUV 画像の高さ（0: 入力と同一） |
| --codec | h265 | エンコードコーデックを指定 |
| --loop | 0 | 1: CTRL+C まで繰り返しトランスコード、0: ループなし |
| --dump | （空） | ダンプファイルパスを指定 |
| --hwclk | 2 (400M) | VDEC のハードウェアクロックを MHz 単位で設定 |
| --ut | | 内部使用 |

> **注意:** 以下のエラーが発生した場合:
> ```
> axcl_sample_transcode: error while loading shared libraries: libavcodec.so.61: cannot open shared object file: No such file or directory
> ```
> まず `export LD_LIBRARY_PATH=/usr/lib/axcl/ffmpeg:$LD_LIBRARY_PATH` で環境変数を設定してください。

#### 例

入力の 1080P@30fps H.264 を 1080P@30fps H.265 にトランスコードし、ファイルに保存します：

```
$ axcl_sample_transcode -i bangkok_30952_1920x1080_30fps_gop60_4Mbps.mp4 -d 0 --dump /tmp/axcl/transcode.265
[INFO ][                            main][  67]: ============== V3.5.0_20250515190238 sample started May 15 2025 19:29:29 pid 91189 ==============

[WARN ][                            main][  92]: if enable dump, disable loop automatically
json file
[INFO ][                            main][ 131]: pid: 91189, device index: 0, bus number: 3
...
[INFO ][             ffmpeg_demux_thread][ 435]: [91189] demuxed    total 470 frames ---
...
[INFO ][                            main][ 283]: total transcoded frames: 470
[INFO ][                            main][ 284]: ============== V3.5.0_20250515190238 sample exited May 15 2025 19:29:29 pid 91189 ==============
```

なお、`launch_transcode.sh` スクリプトを使用すると、最大 16 個の `axcl_sample_transcode` プロセスを起動でき、`LD_LIBRARY_PATH` が自動的に設定されます：

```bash
$ ./launch_transcode.sh 16 -i bangkok_30952_1920x1080_30fps_gop60_4Mbps.mp4  -d 3 --dump /tmp/axcl/transcode.265
```

最初のパラメータは `axcl_sample_transcode` プロセスの数です。指定範囲は [1, 16] です。
