# Whisper

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_whisper

Whisper は OpenAI が開発した汎用音声認識モデルで、多言語の音声をテキストに変換できます。本セクションでは、Raspberry Pi 5 上でプリコンパイル済みの Whisper Small ベースの音声認識サンプルを実行する方法を説明します。モデル変換やサンプルソースコードのコンパイルについては [whisper.axcl](https://github.com/ml-inory/whisper.axcl) を参照してください。

## ダウンロード

```bash
git clone https://github.com/ml-inory/whisper.axcl.git
```

## コンパイル

```bash
cd whisper.axcl
mkdir -p build && cd build
cmake -DCMAKE_INSTALL_PREFIX=../install -DCMAKE_BUILD_TYPE=Release ..
make install -j4
```

## プリコンパイル済みモデル

プリコンパイル済みモデルは以下のリンクからダウンロードできます。

- 百度網盤（中国国内向けファイル共有サービス）
- [Hugging Face whisper-tiny](https://huggingface.co/M5Stack/whisper-tiny-axmodel)
- [Hugging Face whisper-base](https://huggingface.co/M5Stack/whisper-base-axmodel)
- [Hugging Face whisper-small](https://huggingface.co/M5Stack/whisper-small-axmodel)

> **ヒント:** ダウンロードしたモデルファイル（`.axmodel`）は `install/models/` ディレクトリに配置してください。

## Whisper の実行

```bash
cd install
./whisper -w ../demo.wav
```

実行結果は以下の通りです。

```
(base) axera@raspberrypi:~/qtang/whisper.axcl/install $ ./whisper -w ../demo.wav
encoder: ../models/small-encoder.axmodel
decoder_main: ../models/small-decoder-main.axmodel
decoder_loop: ../models/small-decoder-loop.axmodel
wav_file: ../demo.wav
language: zh
Load encoder take 3336.25 ms
Load decoder_main take 6091.89 ms
Load decoder_loop take 5690.05 ms
Read positional_embedding
Encoder run take 190.26 ms
First token: 17556       take 51.49ms
Next Token: 20844        take 30.15 ms
Next Token: 7781         take 30.21 ms
Next Token: 20204        take 30.20 ms
Next Token: 28455        take 30.17 ms
Next Token: 31962        take 30.02 ms
Next Token: 6336         take 30.09 ms
Next Token: 254          take 30.22 ms
Next Token: 2930         take 30.14 ms
Next Token: 236          take 30.14 ms
Next Token: 36135        take 30.12 ms
Next Token: 15868        take 30.18 ms
Next Token: 252          take 30.01 ms
Next Token: 1546         take 30.17 ms
Next Token: 46514        take 30.17 ms
Next Token: 50257        take 30.15 ms
All Token: take 503.68ms, 31.77 token/s
All take 735.09ms
Result: 甚至出现交易几乎停滞的情况
(base) axera@raspberrypi:~/qtang/whisper.axcl/install $
```
