# MeloTTS

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_melotts

MeloTTS は、高品質なテキスト読み上げ（TTS）モデルで、中国語・英語・日本語・スペイン語など複数の言語に対応しています。本セクションでは、Raspberry Pi 5 上でプリコンパイル済みの MeloTTS サンプルを実行する方法を説明します。モデル変換やサンプルソースコードのコンパイルについては [melotts.axcl](https://github.com/ml-inory/melotts.axcl) を参照してください。

## ダウンロード

```bash
git clone https://github.com/ml-inory/melotts.axcl.git
chmod +x build_aarch64.sh
./build_aarch64.sh
```

## プリコンパイル済みモデル

以下のコマンドで、プリコンパイル済みモデルをダウンロードします。

```bash
cd melotts.axcl
./download_models.sh
```

ダウンロード可能なモデルは以下の通りです。

- Hugging Face MeloTTS-Chinese
- Hugging Face MeloTTS-English
- Hugging Face MeloTTS-Japanese
- Hugging Face MeloTTS-Spanish

## コンパイル

aarch64 プラットフォーム向けにビルドする場合は、以下のコマンドを実行します。

```bash
./build_aarch64.sh
```

## MeloTTS の実行

melotts.axcl プロジェクトのルートディレクトリで以下のコマンドを実行します。

```bash
./install/melotts -s 句子
```

英語モデルを使用する場合の実行例は以下の通りです。

```bash
./install/melotts -e ../MeloTTS-English-ax650/encoder-en.onnx -d ../MeloTTS-English-ax650/decoder-en-au.axmodel -l ../MeloTTS-English-ax650/lexicon-en.txt -t ../MeloTTS-English-ax650/tokens-en.txt --g ../MeloTTS-English-ax650/g-en-au.bin -s "M5Stack is a leading provider of IoT solutions, committed to providing developers worldwide with convenient and flexible development components and tools. "
```

実行結果は以下の通りです。

```
m5stack@raspberrypi5:~/melotts.axcl $ ./install/melotts -e ../MeloTTS-English-ax650/encoder-en.onnx -d ../MeloTTS-English-ax650/decoder-en-au.axmodel -l ../MeloTTS-English-ax650/lexicon-en.txt -t ../MeloTTS-English-ax650/tokens-en.txt --g ../MeloTTS-English-ax650/g-en-au.bin -s "M5Stack is a leading provider of IoT solutions, committed to providing developers worldwide with convenient and flexible development components and tools. "
encoder: ../MeloTTS-English-ax650/encoder-en.onnx
decoder: ../MeloTTS-English-ax650/decoder-en-au.axmodel
lexicon: ../MeloTTS-English-ax650/lexicon-en.txt
token: ../MeloTTS-English-ax650/tokens-en.txt
sentence: M5Stack is a leading provider of IoT solutions, committed to providing developers worldwide with convenient and flexible development components and tools.
wav: output.wav
speed: 0.800000
sample_rate: 44100
Load encoder
Load decoder model
Encoder run take 535.47ms
decoder slice num: 9
Decode slice(1/9) take 40.15ms
Decode slice(2/9) take 39.87ms
Decode slice(3/9) take 39.86ms
Decode slice(4/9) take 39.75ms
Decode slice(5/9) take 40.19ms
Decode slice(6/9) take 39.79ms
Decode slice(7/9) take 39.77ms
Decode slice(8/9) take 39.82ms
Decode slice(9/9) take 40.34ms
Saved audio to output.wav
```
