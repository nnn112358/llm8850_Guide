# YOLO11

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_yolo11

YOLO11 は、Ultralytics が提供する最新世代のリアルタイム物体検出モデルシリーズです。物体検出、セグメンテーション、姿勢推定など、幅広いコンピュータビジョンタスクに対応しています。

Ultralytics YOLO11 シリーズモデルの詳細なモデルエクスポート、量子化、コンパイルの手順については、[「AX650N ベースの YOLO11 デプロイ」](https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_yolo11)を参照してください。

## 統合パッケージの取得

まず、統合パッケージを取得して Raspberry Pi にアップロードするか、Raspberry Pi のターミナルで以下のコマンドを実行してください。

ダウンロード:

```bash
wget https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/axcl_demo.zip
```

次に、ダウンロードしたファイルを解凍します。

```bash
unzip axcl_demo.zip
```

## YOLO11x

YOLO11x は、YOLO11 シリーズの中で最も高精度な物体検出モデルです。以下のコマンドで推論を実行します。

```bash
./axcl_demo/axcl_yolo11 -i axcl_demo/ssd_horse.jpg -m axcl_demo/yolo11x.axmodel
```

実行結果:

```text
m5stack@raspberrypi:~/rsp $ ./axcl_demo/axcl_yolo11 -i axcl_demo/ssd_horse.jpg -m axcl_demo/yolo11x.axmodel
--------------------------------------
model file : axcl_demo/yolo11x.axmodel
image file : axcl_demo/ssd_horse.jpg
img_h, img_w : 640 640
--------------------------------------
axclrtEngineCreateContextt is done.
axclrtEngineGetIOInfo is done.

grpid: 0

input size: 1
    name:   images
        1 x 640 x 640 x 3

output size: 3
    name: /model.23/Concat_output_0
        1 x 80 x 80 x 144

    name: /model.23/Concat_1_output_0
        1 x 40 x 40 x 144

    name: /model.23/Concat_2_output_0
        1 x 20 x 20 x 144

==================================================

Engine push input is done.
--------------------------------------
post process cost time:0.88 ms
--------------------------------------
Repeat 1 times, avg time 24.69 ms, max_time 24.69 ms, min_time 24.69 ms
--------------------------------------
detection num: 6
17:  96%, [ 216,   71,  423,  370], horse
16:  93%, [ 144,  203,  196,  345], dog
 0:  89%, [ 273,   14,  349,  231], person
 2:  88%, [   1,  105,  132,  197], car
 0:  82%, [ 431,  124,  451,  178], person
19:  46%, [ 171,  137,  202,  169], cow
--------------------------------------
```

## YOLO11x-Seg

YOLO11x-Seg は、物体検出に加えてインスタンスセグメンテーションを行うモデルです。以下のコマンドで推論を実行します。

```bash
./axcl_demo/axcl_yolo11_seg -i axcl_demo/ssd_horse.jpg -m axcl_demo/yolo11x-seg.axmodel
```

実行結果:

```text
m5stack@raspberrypi:~/rsp $ ./axcl_demo/axcl_yolo11_seg -i axcl_demo/ssd_horse.jpg -m axcl_demo/yolo11x-seg.axmodel
--------------------------------------
model file : axcl_demo/yolo11x-seg.axmodel
image file : axcl_demo/ssd_horse.jpg
img_h, img_w : 640 640
--------------------------------------
axclrtEngineCreateContextt is done.
axclrtEngineGetIOInfo is done.

grpid: 0

input size: 1
    name:   images
        1 x 640 x 640 x 3

output size: 7
    name: /model.23/Concat_1_output_0
        1 x 80 x 80 x 144

    name: /model.23/Concat_2_output_0
        1 x 40 x 40 x 144

    name: /model.23/Concat_3_output_0
        1 x 20 x 20 x 144

    name: /model.23/cv4.0/cv4.0.2/Conv_output_0
        1 x 80 x 80 x 32

    name: /model.23/cv4.1/cv4.1.2/Conv_output_0
        1 x 40 x 40 x 32

    name: /model.23/cv4.2/cv4.2.2/Conv_output_0
        1 x 20 x 20 x 32

    name:  output1
        1 x 32 x 160 x 160

==================================================

Engine push input is done.
--------------------------------------
post process cost time:5.62 ms
--------------------------------------
Repeat 1 times, avg time 34.72 ms, max_time 34.72 ms, min_time 34.72 ms
--------------------------------------
detection num: 6
17:  96%, [ 216,   71,  423,  370], horse
16:  93%, [ 144,  203,  196,  345], dog
 0:  89%, [ 273,   14,  349,  231], person
 2:  88%, [   1,  105,  132,  197], car
 0:  82%, [ 431,  124,  451,  178], person
19:  46%, [ 171,  137,  202,  169], cow
--------------------------------------
```

## YOLO11x-Pose

YOLO11x-Pose は、人体のキーポイントを検出して姿勢推定を行うモデルです。以下のコマンドで推論を実行します。

```bash
./axcl_demo/axcl_yolo11_pose -i axcl_demo/football.jpg -m axcl_demo/yolo11x-pose.axmodel
```

実行結果:

```text
m5stack@raspberrypi:~/rsp $ ./axcl_demo/axcl_yolo11_pose -i axcl_demo/football.jpg -m axcl_demo/yolo11x-pose.axmodel
--------------------------------------
model file : axcl_demo/yolo11x-pose.axmodel
image file : axcl_demo/football.jpg
img_h, img_w : 640 640
--------------------------------------
axclrtEngineCreateContextt is done.
axclrtEngineGetIOInfo is done.

grpid: 0

input size: 1
    name:   images
        1 x 640 x 640 x 3

output size: 6
    name: /model.23/Concat_1_output_0
        1 x 80 x 80 x 65

    name: /model.23/Concat_2_output_0
        1 x 40 x 40 x 65

    name: /model.23/Concat_3_output_0
        1 x 20 x 20 x 65

    name: /model.23/cv4.0/cv4.0.2/Conv_output_0
        1 x 80 x 80 x 51

    name: /model.23/cv4.1/cv4.1.2/Conv_output_0
        1 x 40 x 40 x 51

    name: /model.23/cv4.2/cv4.2.2/Conv_output_0
        1 x 20 x 20 x 51

==================================================

Engine push input is done.
--------------------------------------
post process cost time:0.36 ms
--------------------------------------
Repeat 1 times, avg time 25.09 ms, max_time 25.09 ms, min_time 25.09 ms
--------------------------------------
detection num: 6
 0:  94%, [1350,  337, 1632, 1036], person
 0:  93%, [ 492,  477,  658, 1000], person
 0:  92%, [ 756,  219, 1126, 1154], person
 0:  91%, [   0,  354,  314, 1108], person
 0:  73%, [   0,  530,   81, 1017], person
 0:  54%, [ 142,  589,  239, 1013], person
--------------------------------------
```
