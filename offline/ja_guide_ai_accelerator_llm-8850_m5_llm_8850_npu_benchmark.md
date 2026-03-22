# NPU ベンチマーク

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_npu_benchmark

ベンチマークは、ハードウェアプラットフォーム上でのモデルの実行速度を把握するための最良の方法です。以下のデータは Raspberry Pi 5 をHostとしてテストした結果であり、コミュニティでの参考用です。商用出荷時の最終性能を保証するものではありません。

## テスト条件

| 項目 | 内容 |
|---|---|
| 更新日 | 2024-11-22 |
| ツールチェーン | Pulsar2 3.2-patch2 |
| テストツール | `axcl_run_model` |
| バッチサイズ | 1 または 8 |
| 速度単位 | IPS（Images/Second） |

> **注意:** `axcl_run_model` は Device（NPU）上でのモデル推論時間のみを計測しています。Hostが異なると memcopy や PCIe の性能差により、エンドツーエンドの処理時間は変動します。

## ビジョンモデル

| モデル | 入力サイズ | Batch 1 (IPS) | Batch 8 (IPS) |
|---|---:|---:|---:|
| Inceptionv1 | 224 | 1073 | 2494 |
| Inceptionv3 | 224 | 478 | 702 |
| MobileNetv1 | 224 | 1508 | 4854 |
| MobileNetv2 | 224 | 1366 | 5073 |
| ResNet18 | 224 | 1066 | 2254 |
| ResNet50 | 224 | 576 | 1045 |
| SqueezeNet11 | 224 | 1560 | 5961 |
| Swin-T | 224 | 342 | 507 |
| ViT-B/16 | 224 | 162 | 207 |
| YOLOv5s | 640 | 326 | 394 |
| YOLOv6s | 640 | 282 | 322 |
| YOLOv8s | 640 | 248 | 279 |
| YOLOv9s | 640 | 237 | — |
| YOLOv10s | 640 | 298 | — |
| YOLOv11n | 640 | 860 | — |
| YOLOv11s | 640 | 305 | — |
| YOLOv11m | 640 | 114 | — |
| YOLOv11l | 640 | 87 | — |
| YOLOv11x | 640 | 41 | — |

## オーディオモデル

| モデル | RTF（リアルタイムファクター） |
|---|---:|
| Whisper-Tiny | 0.03 |
| Whisper-Small | 0.18 |
| MeloTTS | 0.04 |

> **補足:** RTF（Real-Time Factor）は処理時間を音声の長さで割った値です。RTF < 1.0 であればリアルタイムより高速に処理できていることを意味します。

## 大規模言語モデル（LLM）

| モデル | プロンプト長 (tokens) | TTFT (ms) | 生成速度 (tokens/s) |
|---|---:|---:|---:|
| Qwen2.5-0.5B | 128 | 188 | 28 |
| Qwen2.5-1.5B | 128 | 407.75 | 9.05 |
| Qwen2.5-1.5B-Int4 | 128 | 407.75 | 9.05 |

> **用語説明:** TTFT（Time To First Token）は、プロンプト入力から最初のトークンが生成されるまでの時間です。

## マルチモーダルモデル（VLM）

| モデル | 画像入力 | Image Encoder (ms) | プロンプト長 (tokens) | TTFT (ms) | 生成速度 (tokens/s) |
|---|---|---:|---:|---:|---:|
| InternVL2-1B | 448×448 | 4200 | 320 | 425 | 29 |
