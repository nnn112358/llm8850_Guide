# クイック体験

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_quick_start

## モデルベンチマーク

環境設定とドライバインストールの 2 つのステップが完了すると、モデルベンチマークツール `axcl_run_model` が使用可能になります。このツールは NPU 上でモデルの実行速度を測定するためのユーティリティで、多くのパラメータが用意されています。`axcl_run_model --help` で使用可能なパラメータの一覧を確認できます。

なお、実装の仕組みに興味がある場合は、対応する sample ディレクトリのソースコードを確認することもできます。このツールやその他の CV / LLM サンプルはすべてソースコード形式で提供されており、ユーザーが API の使い方を理解できるようになっています。

モデルの実行速度をテストするには、`axcl_run_model -m your_model.axmodel -r 10` のような形式を使用します。`-m` で実行するモデルを指定し、`-r` で繰り返し回数を指定することで、簡単にモデルの速度をベンチマークできます。

まず、テスト用のモデルを取得します：

```bash
wget https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/yolo11s.axmodel
```

次に、ベンチマークを実行します：

```bash
axcl_run_model -m yolo11s.axmodel -r 10
```

実行結果は以下のようになります：

```
m5stack@raspberrypi5:~ $ axcl_run_model -m yolo11s.axmodel -r 10
   Run AxModel:
         model: yolo11s.axmodel
          type: 3 Core
          vnpu: Disable
        warmup: 1
        repeat: 10
         batch: { auto: 1 }
    axclrt ver: 1.0.0
   pulsar2 ver: 3.2 99cf147d
      tool ver: 0.0.1
      cmm size: 10488066 Bytes
  ------------------------------------------------------
  min =   3.391 ms   max =   3.414 ms   avg =   3.402 ms
  ------------------------------------------------------
```

上記の実行結果からわかるように、モデルの実行時間だけでなく、ツールチェーンのバージョンやモデルタイプなどの情報も表示されます。

## CV サンプル

コンピュータビジョン（CV）サンプルを使って、画像分類や物体検出などの基本的な推論を体験できます。

> **注意:** 解凍ツールがインストールされていない場合は、まず以下のコマンドでインストールしてください：

```bash
sudo apt install zip
```

Demo の取得：

```bash
wget https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/cv_demo.zip
```

Demo の解凍：

```bash
unzip cv_demo.zip
```

ディレクトリに移動：

```bash
cd cv_demo
```

### 分類モデル

MobileNetV2 は軽量な画像分類モデルです。ここでは、imagenet データセットの `imagenet_cat.jpg` を分類対象として使用します。サンプルの実行が完了すると、以下のような出力が得られます（モデルと入力画像は実際の状況に応じて調整してください）。

実行コマンド：

```bash
./axcl_sample_classification -m mobilenetv2.axmodel -i cat.jpg
```

結果：

```
m5stack@raspberrypi5:~/cv_demo $ ./axcl_sample_classification -m mobilenetv2.axmodel -i cat.jpg
axcl initializing...
axcl inited.
Select axcl device{index: 0} as {1}.
axclrt Engine inited.
--------------------------------------
model file : mobilenetv2.axmodel
image file : cat.jpg
img height : 224
img width  : 224
--------------------------------------
282:  9.8%,  tiger cat
285:  9.8%,  Egyptian cat
283:  9.5%,  Persian cat
281:  9.4%,  tabby, tabby cat
463:  7.5%,  bucket, pail
--------------------------------------
```

### 検出モデル

YOLOv5s はリアルタイム物体検出モデルです。ここでは、PASCAL VOC データセットの `voc_dog.jpg` を検出対象として使用します。サンプルの実行が完了すると、以下のような出力が得られます（モデルと入力画像は実際の状況に応じて調整してください）。

実行コマンド：

```bash
./axcl_sample_yolov5s -m yolov5s.axmodel -i dog.jpg
```

結果：

```
m5stack@raspberrypi5:~/cv_demo $ ./axcl_sample_yolov5s -m yolov5s.axmodel -i dog.jpg
axcl initializing...
axcl inited.
Select axcl device{index: 0} as {1}.
axclrt Engine inited.
--------------------------------------
model file : yolov5s.axmodel
image file : dog.jpg
img height : 640
img width  : 640
--------------------------------------
post process cost time:0.61 ms
--------------------------------------
Repeat 1 times, avg time 8.10 ms, max_time 8.10 ms, min_time 8.10 ms
--------------------------------------
16:  91%, [ 138,  218,  310,  541], dog
 2:  69%, [ 470,   76,  690,  173], car
 1:  56%, [ 158,  120,  569,  420], bicycle
```

3 つのターゲットが検出され、カテゴリ ID、信頼度、座標がそれぞれ表示されています。また、サンプルを実行したディレクトリに `yolov5s_out.jpg` という名前の検出結果画像が保存されます。画像ビューアで開いて出力結果をプレビューできます。
