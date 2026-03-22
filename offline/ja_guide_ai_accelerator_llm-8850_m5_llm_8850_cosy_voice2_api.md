# CosyVoice2-API

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_cosy_voice2_api

CosyVoice2-API は、CosyVoice2 音声合成モデルを OpenAI API 互換のインターフェースで利用する方法を提供します。StackFlow パッケージをインストールするだけで簡単に使用できます。

## 準備作業

[RaspberryPi & LLM8850 ソフトウェアパッケージ取得チュートリアル](https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_quick_start)を参考に、以下のモデルパッケージとソフトウェアパッケージのインストールを完了してください。

```bash
sudo apt install lib-llm llm-sys llm-cosy-voice llm-openai-api
```

```bash
sudo apt install llm-model-cosyvoice2-0.5b-axcl
```

> **注意:** 新しいモデルをインストールした後は、モデルリストを更新するために以下のコマンドを実行してください。
> ```bash
> sudo systemctl restart llm-openai-api
> ```

> **注意:** CosyVoice2 は LLM ベースの音声合成モデルで、自然で滑らかな音声を生成できます。ただし、リソースや設計上の制限により、一度に生成できる音声の最大長は 27 秒です。また、初回のモデルロードには時間がかかりますので、しばらくお待ちください。

## Curl による呼び出し

```bash
curl http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "model": "CosyVoice2-0.5B-axcl",
    "response_format": "wav",
    "input": "君不見黄河之水天上来，奔流到海不復回。君不見高堂明鏡悲白髪，朝如青糸暮成雪。人生得意須尽歓，莫使金樽空対月。天生我材必有用，千金散尽還復来。"
  }' \
  -o output.wav
```

## Python による呼び出し

```python
from pathlib import Path
from openai import OpenAI

client = OpenAI(
    api_key="sk-",
    base_url="http://127.0.0.1:8000/v1"
)

speech_file_path = Path(__file__).parent / "output.wav"
with client.audio.speech.with_streaming_response.create(
  model="CosyVoice2-0.5B-axcl",
  voice="prompt_data",
  response_format="wav",
  input='君不見黄河之水天上来，奔流到海不復回。君不見高堂明鏡悲白髪，朝如青糸暮成雪。人生得意須尽歓，莫使金樽空対月。天生我材必有用，千金散尽還復来。',
) as response:
  response.stream_to_file(speech_file_path)
```

## 音色クローン

音色クローン機能を使用すると、任意の音声サンプルから話者の声質を再現できます。[モデルを手動でダウンロード](https://huggingface.co/M5Stack/CosyVoice2-scripts)して Raspberry Pi 5 にアップロードするか、以下のコマンドでリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone --recurse-submodules https://huggingface.co/M5Stack/CosyVoice2-scripts
```

### ファイル説明

```
m5stack@raspberrypi:~/rsp/CosyVoice2-scripts $ ls -lh
total 28K
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  6 15:18 asset
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  6 15:18 CosyVoice-BlankEN
drwxrwxr-x 2 m5stack m5stack 4.0K Nov  6 15:19 frontend-onnx
drwxrwxr-x 3 m5stack m5stack 4.0K Nov  6 15:18 pengzhendong
-rw-rw-r-- 1 m5stack m5stack   24 Nov  6 15:18 README.md
-rw-rw-r-- 1 m5stack m5stack  103 Nov  6 15:18 requirements.txt
drwxrwxr-x 3 m5stack m5stack 4.0K Nov  6 15:18 scripts
```

### 仮想環境の作成

```bash
python -m venv cosyvoice
```

### 仮想環境の有効化

```bash
source cosyvoice/bin/activate
```

### 依存パッケージのインストール

```bash
pip install -r requirements.txt
```

### process_prompt スクリプトの実行

以下のコマンドで、音声サンプルから話者の特徴を抽出します。

```bash
python3 scripts/process_prompt.py --prompt_text  asset/zh_woman1.txt --prompt_speech asset/zh_woman1.wav --output zh_woman1
```

### 音声特徴ファイルの生成結果

```
(cosyvoice) m5stack@raspberrypi:~/rsp/CosyVoice2-scripts $ python3 scripts/process_prompt.py --prompt_text  asset/zh_woman1.txt --prompt_speech asset/zh_woman1.wav --output zh_woman1
2025-11-06 15:54:43.619688866 [W:onnxruntime:Default, device_discovery.cc:164 DiscoverDevicesForPlatform] GPU device discovery failed: device_discovery.cc:89 ReadFileContents Failed to open file: "/sys/class/drm/card1/device/vendor"
prompt_text 希望你以后能够做的比我还好呦。
fmax 8000
prompt speech token size: torch.Size([1, 87])
```

### 音声特徴ファイルをモデルディレクトリにコピーし、モデルを再初期化

生成した音声特徴ファイルをモデルディレクトリにコピーし、サービスを再起動して反映させます。

```bash
cp -r zh_woman1 /opt/m5stack/data/CosyVoice2-0.5B-axcl/
```

```bash
sudo systemctl restart llm-sys # モデル設定をリセット
```

> **ヒント:** デフォルトのクローン音色を置き換えたい場合は、`/opt/m5stack/data/models/mode_CosyVoice2-0.5B-axcl.json` ファイルの `prompt_dir` フィールドを変更してください。音色を置き換えるたびにモデルの再初期化が必要です。

### Curl による呼び出し

```bash
curl http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "model": "CosyVoice2-0.5B-axcl",
    "voice": "zh_woman1",
    "response_format": "wav",
    "input": "君不見黄河之水天上来，奔流到海不復回。君不見高堂明鏡悲白髪，朝如青糸暮成雪。人生得意須尽歓，莫使金樽空対月。天生我材必有用，千金散尽還復来。"
  }' \
  -o output.wav
```

### Python による呼び出し

```python
from pathlib import Path
from openai import OpenAI

client = OpenAI(
    api_key="sk-",
    base_url="http://127.0.0.1:8000/v1"
)

speech_file_path = Path(__file__).parent / "output.wav"
with client.audio.speech.with_streaming_response.create(
  model="CosyVoice2-0.5B-axcl",
  voice="zh_woman1",
  response_format="wav",
  input='君不見黄河之水天上来，奔流到海不復回。君不見高堂明鏡悲白髪，朝如青糸暮成雪。人生得意須尽歓，莫使金樽空対月。天生我材必有用，千金散尽還復来。',
) as response:
  response.stream_to_file(speech_file_path)
```
