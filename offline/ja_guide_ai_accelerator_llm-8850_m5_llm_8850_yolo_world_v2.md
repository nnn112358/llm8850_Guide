# YOLO World v2

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_yolo_world_v2

YOLO-World v2 は、テキストプロンプトによるオープンボキャブラリ物体検出モデルです。事前に定義されたクラスに限定されず、任意のテキストで指定したカテゴリの物体をリアルタイムで検出できます。

YOLO-World v2 モデルの詳細なモデルエクスポート、量子化、コンパイルの手順については、[「YOLO World デプロイ再考」](https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_yolo_world_v2)を参照してください。

- モデル: `yoloworldv2_4cls_50_npu3.axmodel`
- 入力画像: `ssd_horse.jpg`
- 入力テキスト: `dog.bin`（対応する 4 クラス分類: 'dog' 'horse' 'sheep' 'cow'）

> **ヒント:** 統合パッケージの取得と解凍については [YOLO11 セクション](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolo11.md) を参照してください。

## 実行

以下のコマンドでオープンボキャブラリ検出を実行します。

```bash
./axcl_demo/axcl_yolo_world_open_vocabulary -m axcl_demo/yoloworldv2_4cls_50_npu3.axmodel -t axcl_demo/dog.bin -i axcl_demo/ssd_horse.jpg
```

実行結果:

```text
m5stack@raspberrypi:~/rsp $ ./axcl_demo/axcl_yolo_world_open_vocabulary -m axcl_demo/yoloworldv2_4cls_50_npu3.axmodel -t axcl_demo/dog.bin -i axcl_demo/ssd_horse.jpg
--------------------------------------
model file : axcl_demo/yoloworldv2_4cls_50_npu3.axmodel
image file : axcl_demo/ssd_horse.jpg
text_feature file : axcl_demo/dog.bin
img_h, img_w : 640 640
--------------------------------------
axclrtEngineCreateContextt is done.
axclrtEngineGetIOInfo is done.

grpid: 0

input size: 2
    name:   images
        1 x 640 x 640 x 3

    name: txt_feats
        1 x 4 x 512

output size: 3
    name:  stride8
        1 x 80 x 80 x 68

    name: stride16
        1 x 40 x 40 x 68

    name: stride32
        1 x 20 x 20 x 68

==================================================

Engine push input is done.
--------------------------------------
post process cost time:0.25 ms
--------------------------------------
Repeat 1 times, avg time 4.52 ms, max_time 4.52 ms, min_time 4.52 ms
--------------------------------------
detection num: 2
 1:  91%, [ 215,   71,  421,  374], class2
 0:  67%, [ 144,  204,  197,  346], class1
--------------------------------------
```

## YOLO World v2 CLI

コマンドラインインターフェースを使用して、任意のクラスを指定した検出を行うことができます。まず、ソースコードを取得して Raspberry Pi 5 にアップロードするか、git コマンドで取得してください。

コードの取得:

```bash
git clone https://github.com/AXERA-TECH/yoloworld.axera.git
```

次に、プロジェクトディレクトリに移動します。

```bash
cd yoloworld.axera
```

> **ヒント:** 以降の操作はすべてこのディレクトリ内で実行してください。

ファイル一覧:

```text
m5stack@raspberrypi:~/rsp/yoloworld.axera $ ls -lh
total 36K
-rwxrwxr-x 1 m5stack m5stack 2.1K Aug 12 11:26 build_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  775 Aug 12 11:26 build.sh
-rw-rw-r-- 1 m5stack m5stack 1.5K Aug 12 11:26 CMakeLists.txt
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:26 include
drwxrwxr-x 3 m5stack m5stack 4.0K Aug 12 14:46 pyyoloworld
-rw-rw-r-- 1 m5stack m5stack 1.9K Aug 12 11:26 README.md
drwxrwxr-x 4 m5stack m5stack 4.0K Aug 12 11:26 src
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:26 tests
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:26 toolchains
```

依存パッケージのインストール:

```bash
sudo apt update
sudo apt install cmake libopencv-dev -y
```

コードのビルド:

```bash
mkdir build && cd build
cmake .. && make -j4 install
cd ..
```

モデルと語彙リストの取得:

```bash
wget https://github.com/AXERA-TECH/yoloworld.axera/releases/download/v0.1/clip_b1_u16_ax650.axmodel
wget https://github.com/AXERA-TECH/yoloworld.axera/releases/download/v0.1/yolo_u16_ax650.axmodel
wget https://github.com/AXERA-TECH/yoloworld.axera/releases/download/v0.1/vocab.txt
```

以下のコマンドで推論を実行します。`--classes` オプションで検出したいカテゴリをカンマ区切りで指定します。

```bash
./build/test_detect_by_text --yoloworld yolo_u16_ax650.axmodel --tenc clip_b1_u16_ax650.axmodel -v vocab.txt -i ssd_horse.jpg --classes person,car,dog,horse
```

実行結果:

```text
m5stack@raspberrypi:~/rsp/yoloworld.axera $ ./build/test_detect_by_text --yoloworld yolo_u16_ax650.axmodel --tenc clip_b1_u16_ax650.axmodel -v vocab.txt -i ssd_horse.jpg --classes person,car,dog,horse
open libax_sys.so failed
open libax_engine.so failed
[I][                             run][  31]: AXCLWorker start with devid 0
label-0 class_names: person
label-1 class_names: car
label-2 class_names: dog
label-3 class_names: horse
yoloworld_path: yolo_u16_ax650.axmodel
text_encoder_path: clip_b1_u16_ax650.axmodel
tokenizer_path: vocab.txt

input size: 2
    name:   images [unknown] [unknown]
        1 x 640 x 640 x 3   size: 1228800

    name: txt_feats [unknown] [unknown]
        1 x 4 x 512   size: 8192

output size: 3
    name:  stride8
        1 x 80 x 80 x 68   size: 1740800

    name: stride16
        1 x 40 x 40 x 68   size: 435200

    name: stride32
        1 x 20 x 20 x 68   size: 108800

[I][                       yw_create][ 408]: num_classes: 4, num_features: 512, input w: 640, h: 640
is_output_nhwc: 1

input size: 1
    name: text_token [unknown] [unknown]
        1 x 77   size: 308

output size: 1
    name:     2202
        1 x 1 x 512   size: 2048

[I][               load_text_encoder][  44]: text feature len 512
[I][                  load_tokenizer][  60]: text token len 77
[I][                  yw_set_classes][ 463]: label-0 class_names: person
[I][                  yw_set_classes][ 463]: label-1 class_names: car
[I][                  yw_set_classes][ 463]: label-2 class_names: dog
[I][                  yw_set_classes][ 463]: label-3 class_names: horse
yw_set_classes time: 12.76 ms
preprocess 1.04
inference 19.06
postprocess 0.27
yw_detect time: 20.45 ms
object-0 class_id: 3 score: 0.88 box: 215 69 206 304
object-1 class_id: 0 score: 0.88 box: 271 13 76 220
object-2 class_id: 0 score: 0.77 box: 430 123 20 54
object-3 class_id: 2 score: 0.75 box: 144 203 52 143
object-4 class_id: 1 score: 0.65 box: 0 105 131 91
object-5 class_id: 0 score: 0.37 box: 402 130 7 16
object-6 class_id: 0 score: 0.16 box: 413 134 13 16
[I][                             run][  81]: AXCLWorker exit with devid 0
[2025-08-12 16:00:53.491][55020][C][device][print][18]: allocated device memories: 7
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x1497ae000] : 0x134 bytes
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x1495de000] : 0x1a900 bytes
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x1497af000] : 0x800 bytes
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x149573000] : 0x6a400 bytes
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x1493ca000] : 0x1a9000 bytes
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x1493c8000] : 0x2000 bytes
[2025-08-12 16:00:53.491][55020][C][device][print][20]: [0x14929c000] : 0x12c000 bytes
```

## YOLO World v2 Web

Gradio ベースの Web インターフェースを使用して、ブラウザから対話的に検出を行うこともできます。

仮想環境の作成:

```bash
python -m venv yolo
```

仮想環境のアクティベート:

```bash
source yolo/bin/activate
```

依存パッケージのインストール:

```bash
pip install -r pyyoloworld/requirements.txt
```

ダイナミックライブラリのコピー:

```bash
cp build/libyoloworld.so pyyoloworld/
```

サービスの起動:

```bash
python pyyoloworld/gradio_example.py --yoloworld yolo_u16_ax650.axmodel --tenc clip_b1_u16_ax650.axmodel --vocab vocab.txt
```

起動後の出力:

```text
(yolo) m5stack@raspberrypi:~/rsp/yoloworld.axera $ python pyyoloworld/gradio_example.py --yoloworld yolo_u16_ax650.axmodel --tenc clip_b1_u16_ax650.axmodel --vocab vocab.txt
Trying to load: /home/m5stack/rsp/yoloworld.axera/pyyoloworld/aarch64/libyoloworld.so

Failed to load: /home/m5stack/rsp/yoloworld.axera/pyyoloworld/aarch64/libyoloworld.so
   /home/m5stack/rsp/yoloworld.axera/pyyoloworld/aarch64/libyoloworld.so: cannot open shared object file: No such file or directory
File not found. Please verify that libclip.so exists and the path is correct.

Trying to load: /home/m5stack/rsp/yoloworld.axera/pyyoloworld/libyoloworld.so
open libax_sys.so failed
open libax_engine.so failed
Successfully loaded: /home/m5stack/rsp/yoloworld.axera/pyyoloworld/libyoloworld.so
[I][                             run][  31]: AXCLWorker start with devid 0

input size: 2
    name:   images [unknown] [unknown]
        1 x 640 x 640 x 3   size: 1228800

    name: txt_feats [unknown] [unknown]
        1 x 4 x 512   size: 8192

output size: 3
    name:  stride8
        1 x 80 x 80 x 68   size: 1740800

    name: stride16
        1 x 40 x 40 x 68   size: 435200

    name: stride32
        1 x 20 x 20 x 68   size: 108800

[I][                       yw_create][ 408]: num_classes: 4, num_features: 512, input w: 640, h: 640
is_output_nhwc: 1

input size: 1
    name: text_token [unknown] [unknown]
        1 x 77   size: 308

output size: 1
    name:     2202
        1 x 1 x 512   size: 2048

[I][               load_text_encoder][  44]: text feature len 512
[I][                  load_tokenizer][  60]: text token len 77
* Running on local URL:  http://0.0.0.0:7860
* To create a public link, set `share=True` in `launch()`.
```

サービスが起動したら、ブラウザで `127.0.0.1:7860` を開いてください。画像をアップロードし、検出したいクラス名を入力して **検出** ボタンをクリックすると、検出結果が表示されます。
