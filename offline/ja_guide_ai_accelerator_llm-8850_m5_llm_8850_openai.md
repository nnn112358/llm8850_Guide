# OpenAI API

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_openai

LLM-8850 では、OpenAI API 互換のインターフェースを利用できます。StackFlow パッケージをインストールするだけで、curl、Python、ChatBox などさまざまなクライアントからアクセス可能です。

## 準備作業

[RaspberryPi & LLM8850 環境設定](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_software_install.md)を参照し、ドライバのインストールを完了した上で、以下のモデルパッケージとソフトウェアパッケージをインストールしてください。

```bash
sudo apt install lib-llm llm-sys llm-llm llm-openai-api
```

```bash
sudo apt install llm-model-qwen3-1.7b-int8-ctx-axcl
```

> **注意:** 新しいモデルをインストールするたびに、以下のコマンドを実行してモデルリストを更新する必要があります。
>
> ```bash
> sudo systemctl restart llm-openai-api
> ```

## Curl による呼び出し

以下のコマンドで、利用可能なモデルの一覧を取得できます。

```bash
curl http://127.0.0.1:8000/v1/models \
  -H "Content-Type: application/json"
```

チャット補完 API を呼び出すには、以下のコマンドを実行します。

```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxxxxx" \
  -d '{
    "model": "qwen3-1.7B-Int8-ctx-axcl",
    "messages": [
      {"role": "developer", "content": "You are a helpful home assistant."},
      {"role": "user", "content": "Write a one-sentence bedtime story about a unicorn."}
    ]
  }'
```

## Python による呼び出し

Python の OpenAI クライアントライブラリを使用して、モデル一覧を取得する例です。

```python
from openai import OpenAI
client = OpenAI(
    api_key="sk-",
    base_url="http://127.0.0.1:8000/v1"
)

client.models.list()
print(client.models.list())
```

チャット補完 API を呼び出す例です。

```python
from openai import OpenAI
client = OpenAI(
    api_key="sk-",
    base_url="http://127.0.0.1:8000/v1"
)

completion = client.chat.completions.create(
  model="qwen3-1.7B-Int8-ctx-axcl",
  messages=[
    {"role": "developer", "content": "You are a helpful home assistant."},
    {"role": "user", "content": "Turn on the light!"}
  ]
)

print(completion.choices[0].message)
```

## ChatBox による呼び出し

[ChatBox](https://chatboxai.app/) は、さまざまな LLM API に対応したデスクトップチャットクライアントです。以下の手順で LLM-8850 と接続できます。

1. [ChatBox](https://chatboxai.app/) をダウンロードしてインストールします。
2. 設定画面を開き、モデルプロバイダーを追加します。
3. API Host に Raspberry Pi の IP アドレスと API パスを入力し、インストール済みモデルを取得して追加します。
4. 新しいチャットを作成し、LLM-8850 が提供する `qwen3-1.7B-Int8-ctx-axcl` モデルを選択します。
5. 最大コンテキストメッセージ長を 0 に変更します。
6. System Prompt の設定もサポートされています。
