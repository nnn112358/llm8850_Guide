# NPU サンプル

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_npu_samples

## NPU ツールチェーン

AI チップ上でアルゴリズムモデルのデプロイを行うには、チップメーカーが提供する NPU ツールチェーンを使用してモデルを変換する必要があります。LLM-8850 では **Pulsar2** というツールチェーンを使用します。Pulsar2 は、ONNX などの汎用モデルフォーマットを LLM-8850 の NPU で実行可能な .axmodel フォーマットに変換するためのツールです。

- [Pulsar2 オンラインドキュメント](https://pulsar2-docs.readthedocs.io/)
  - [インストールガイド](https://pulsar2-docs.readthedocs.io/en/latest/user_guides_quick/install.html)
  - [クイックスタート](https://pulsar2-docs.readthedocs.io/en/latest/user_guides_quick/quick_start.html)
  - [NPU オペレータサポートリスト](https://pulsar2-docs.readthedocs.io/en/latest/op_support_list/index.html)
  - [大規模モデル変換](https://pulsar2-docs.readthedocs.io/en/latest/llm/index.html)

## AXCL-Samples

AXCL-Samples は、Axera（愛芯元智）が主導で開発しているサンプルコード集です。このプロジェクトでは、Axera の SoC をベースとした PCIe コンピューティングカード製品上で、一般的なディープラーニングオープンソースアルゴリズムを実行するサンプルコードを提供しています。コミュニティの開発者が迅速に評価・適応できるようになっています。

### axcl-samples

このリポジトリは、Ultralytics の YOLO シリーズ、DepthAnything、YOLO-Worldv2 など、よく使われるオープンソースモデルをシンプルな方法で動作させるサンプルを提供しています。

## サンプルの取得

AXCL-Samples のプリコンパイル済み ModelZoo は以下からダウンロードできます：

- [百度網盤](https://pan.baidu.com/)
- [Hugging Face](https://huggingface.co/)
- モデルとプログラムの OSS ダウンロード
- モデルとプログラムの Hugging Face ダウンロード

> **ヒント:** 大容量モデルファイルのダウンロードには Git LFS が必要な場合があります。Git LFS のインストール手順については、[Git LFS インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)を参照してください。

## LLM サンプル

大規模言語モデル（LLM）の変換方法については、[大規模モデルコンパイルドキュメント](https://pulsar2-docs.readthedocs.io/en/latest/llm/index.html)を参照してください。

プリコンパイル済み ModelZoo-LLM は以下からダウンロードできます：

- [百度網盤](https://pan.baidu.com/)
- [Hugging Face](https://huggingface.co/)
