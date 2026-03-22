# Depth Anything V2

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_depth_anything_v2

Depth Anything V2 は、単眼画像から高品質な深度マップを推定するモデルです。1 枚の画像から各ピクセルの奥行き情報を予測でき、3D シーン理解やロボティクスなどに活用できます。

DepthAnything モデルの詳細なモデルエクスポート、量子化、コンパイルの手順については、[「AX650N ベースの Depth Anything」](https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_depth_anything_v2)を参照してください。

> **ヒント:** 統合パッケージの取得と解凍については [YOLO11 セクション](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_yolo11.md) を参照してください。

## 実行

以下のコマンドで深度推定の推論を実行します。

```bash
./axcl_demo/axcl_depth_anything -m axcl_demo/depth_anything_v2_vits_ax650.axmodel -i axcl_demo/ssd_horse.jpg
```

実行結果:

```text
m5stack@raspberrypi:~/rsp $ ./axcl_demo/axcl_depth_anything -m axcl_demo/depth_anything_v2_vits_ax650.axmodel -i axcl_demo/ssd_horse.jpg
--------------------------------------
model file : axcl_demo/depth_anything_v2_vits_ax650.axmodel
image file : axcl_demo/ssd_horse.jpg
img_h, img_w : 384 640
--------------------------------------
axclrtEngineCreateContextt is done.
axclrtEngineGetIOInfo is done.

grpid: 0

input size: 1
    name:    input
        1 x 518 x 518 x 3

output size: 1
    name:   output
        1 x 1 x 518 x 518

==================================================

Engine push input is done.
--------------------------------------
post process cost time:4.18 ms
--------------------------------------
Repeat 1 times, avg time 33.24 ms, max_time 33.24 ms, min_time 33.24 ms
--------------------------------------
--------------------------------------
```
