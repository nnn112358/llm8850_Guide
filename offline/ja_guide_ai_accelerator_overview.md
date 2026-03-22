# LLM-8850 Card 使用ガイド

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/overview

## 概要

M5Stack LLM-8850 Card は、エッジデバイス向けの **M.2 M-Key 2242** 規格のAIアクセラレータカードです。AXERAチップを搭載し、ビジョン・言語・音声など多様なAIモデルをローカルで高速に実行できます。AXERA 公式オープンソース SDK については公式サイトをご参照ください。

## クイックスタート

| ステップ | ドキュメント |
|---|---|
| 1. ハードウェアの取り付け | [LLM-8850 Card ハードウェアインストール](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_hardware_install.md) |
| 2. 環境設定（Raspberry Pi） | [LLM-8850 Card 環境設定](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_software_install.md) |
| 3. クイック体験 | [LLM-8850 Card クイック体験](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_quick_start.md) |
| 4. Windows 環境設定 | [LLM-8850 Card Windows 環境設定](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_windows_install.md) |

## 対応モデル一覧

### ビジョンモデル（画像認識・検出・超解像等）

| モデル | 用途 |
|---|---|
| YOLO11 | 物体検出・セグメンテーション・姿勢推定 |
| Yolo-World-V2 | オープンボキャブラリ物体検出 |
| Yolov7-face | 顔検出 |
| Depth-Anything-V2 | 単眼深度推定 |
| MixFormer-V2 | 物体追跡 |
| Real-ESRGAN | 画像超解像 |
| SuperResolution | 動画超解像 |
| RIFE | 動画フレーム補間 |
| PP-OCRv5 | OCR（文字認識） |
| DEIMv2 | リアルタイム物体検出 |

### 大規模言語モデル（LLM）

| モデル | パラメータ数 |
|---|---|
| Qwen3-0.6B | 0.6B |
| Qwen3-1.7B | 1.7B |
| Qwen2.5-0.5B-Instruct | 0.5B |
| Qwen2.5-1.5B-Instruct | 1.5B |
| DeepSeek-R1-Distill-Qwen-1.5B | 1.5B |
| MiniCPM4-0.5B | 0.5B |

### マルチモーダルモデル（VLM / CLIP）

| モデル | 用途 |
|---|---|
| InternVL3-1B | 画像理解・VQA |
| Qwen2.5-VL-3B-Instruct | 画像/動画理解 |
| Qwen3-VL-2B-Instruct | 画像/動画理解 |
| Qwen3-VL-4B-Instruct-GPTQ-Int4 | 画像/動画理解（量子化版） |
| SmolVLM2-500M-Video-Instruct | 軽量動画理解 |
| LibCLIP | テキスト-画像検索 |
| Jina CLIP v2 | テキスト-画像エンベディング |

### オーディオモデル（音声認識・音声合成）

| モデル | 用途 |
|---|---|
| Whisper | 音声認識（テキスト変換） |
| MeloTTS | テキスト音声合成 |
| SenseVoice | 多言語音声認識 |
| CosyVoice2 | 高品質音声合成・音色クローン |
| 3D-Speaker-MT | 話者認識 |

### 生成モデル（画像生成・アニメーション）

| モデル | 用途 |
|---|---|
| lcm-lora-sdv1-5 | 高速画像生成 |
| SD1.5-LLM8850 | Stable Diffusion（Web UI付き） |
| LivePortrait | ポートレートアニメーション |

## アプリケーション一覧

| アプリケーション | 説明 |
|---|---|
| Frigate NVR | AIベースのネットワークビデオレコーダー |
| Immich | セルフホスト型写真管理 |
| OpenAI API | OpenAI互換APIサービス |
| CosyVoice2 API | 音声合成APIサービス |
| sherpa-onnx | リアルタイム音声認識ツールキット |

## 上級者向け

- LLM-8850 Card AXCL-SMI（デバイス管理・診断ツール）
- LLM-8850 Card NPU 使用例
- LLM-8850 Card NPU ベンチマーク
- LLM-8850 Card AXCL-Sample 使用説明
- LLM-8850 Card AXCL FFmpeg 使用例
- LLM-8850 Card AXCL SDK API

## よくある質問

- [LLM-8850 Card AXCL FAQ](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_faq.md)
