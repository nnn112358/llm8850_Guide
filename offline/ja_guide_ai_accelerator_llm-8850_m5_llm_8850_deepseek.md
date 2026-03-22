# DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_deepseek

DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4 は、DeepSeek-R1 の推論能力を Qwen-1.5B アーキテクチャに蒸留した軽量モデルの GPTQ Int4 量子化版です。LLM-8850 上で推論チェーン（思考過程）付きのテキスト生成が可能です。

## モデルのダウンロード

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4)して Raspberry Pi 5 にアップロードしてください。

### ファイル説明

ダウンロードしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4$ ls -lh
total 2.9M
-rw-rw-r-- 1 m5stack m5stack    0 Aug 12 10:56 config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:00 deepseek-r1-1.5b-gptq-int4-ax650
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 10:56 deepseek-r1_tokenizer
-rw-rw-r-- 1 m5stack m5stack 4.2K Aug 12 10:56 deepseek-r1_tokenizer.py
-rw-rw-r-- 1 m5stack m5stack 976K Aug 12 10:58 main_axcl_aarch64
-rw-rw-r-- 1 m5stack m5stack 999K Aug 12 10:58 main_axcl_x86
-rw-rw-r-- 1 m5stack m5stack 932K Aug 12 10:58 main_prefill
-rw-rw-r-- 1 m5stack m5stack  277 Aug 12 10:56 post_config.json
-rw-rw-r-- 1 m5stack m5stack 7.4K Aug 12 10:56 README.md
-rw-rw-r-- 1 m5stack m5stack  533 Aug 12 10:56 run_deepseek-r1_1.5b_gptq_int4_ax650.sh
-rw-rw-r-- 1 m5stack m5stack  538 Aug 12 10:56 run_deepseek-r1_1.5b_gptq_int4_axcl_aarch64.sh
-rw-rw-r-- 1 m5stack m5stack  534 Aug 12 10:56 run_deepseek-r1_1.5b_gptq_int4_axcl_x86.sh
```

## 仮想環境の作成

推論に必要な Python 仮想環境を作成します。

```bash
python -m venv deepseek
```

### 仮想環境のアクティベート

```bash
source deepseek/bin/activate
```

### 依存パッケージのインストール

```bash
pip install transformers jinja2
```

## tokenizer パーサーの起動

以下のコマンドで tokenizer サービスを起動します。

```bash
python deepseek-r1_tokenizer.py --port 12345
```

tokenizer サービスを実行すると、Host IP はデフォルトで `localhost`、ポート番号は `12345` に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(deepseek) m5stack@raspberrypi:~/rsp/DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4 $ python deepseek-r1_tokenizer.py --port 12345
None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available and only tokenizers, configuration and file/data utilities can be used.
151646 <|begin_of_sentence|> 151643 <|end_of_sentence|>
<|begin_of_sentence|>You are DeepSeek-R1, You are a helpful assistant.<|User|>hello world<|Assistant|>
[151646, 151646, 2610, 525, 18183, 39350, 10911, 16, 11, 1446, 525, 264, 10950, 17847, 13, 151644, 14990, 1879, 151645]
http://localhost:12345
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

## 実行権限の設定

推論用の実行ファイルとスクリプトに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64 run_deepseek-r1_1.5b_gptq_int4_axcl_aarch64.sh
```

## DeepSeek-R1-Distill-Qwen-1.5B モデル推論サービスの起動

以下のコマンドでモデルの推論サービスを起動します。

```bash
./run_deepseek-r1_1.5b_gptq_int4_axcl_aarch64.sh
```

### 起動成功後の情報

正常に起動すると、以下のようなログが表示されます。`<think>` タグ内にモデルの思考過程が表示されるのが DeepSeek-R1 の特徴です。

```
m5stack@raspberrypi:~/rsp/DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4$ ./run_deepseek-r1_1.5b_gptq_int4_axcl_aarch64.sh
build time: Feb 13 2025 15:44:57
[I][                            Init][ 111]: LLM init start
bos_id: 151646, eos_id: 151643
  3% | ██                                |   1 /  31 [0.00s<0.09s, 333.33 count/s] tokenizer init
  100% | ████████████████████████████████ |  31 /  31 [28.78s<28.78s, 1.08 count/s] init post axmodel ok
[I][                            Init][ 226]: max_token_len : 1023
[I][                            Init][ 231]: kv_cache_size : 256, kv_cache_num: 1023
[I][                     load_config][ 282]: load config:
{
    "enable_repetition_penalty": false,
    "enable_temperature": true,
    "enable_top_k_sampling": true,
    "enable_top_p_sampling": false,
    "penalty_window": 20,
    "repetition_penalty": 1.2,
    "temperature": 0.9,
    "top_k": 10,
    "top_p": 0.8
}

[I][                            Init][ 288]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
>> hello
<think>
Alright, let me take a look at the user's message. They've greeted me with "You DeepSeek-R1, You are a helpful assistant." Seems like they're thanking me and expressing gratitude for the assistance I've provided before. They ended with "hello"—a friendly greeting in Chinese.

Hmm, they might be testing the AI's response generation capabilities. Maybe they're checking if I can understand their greeting properly or just trying to get feedback. I should respond in a friendly and professional manner, acknowledging their gratitude and offering further assistance. Perhaps they were expecting some interaction but didn't see the response yet. I'll keep it open to see if they have a specific question in mind.
</think>

Hello! I'm DeepSeek-R1, an AI assistant ready to help you with any questions or tasks you may have. How can I assist you today?

[N][                             Run][ 610]: hit eos,avg 13.29 token/s

>>
```

## ベンチマーク

以下は LLM-8850 上での DeepSeek-R1 の推論パフォーマンスです。

| モデル | 量子化方式 | ttft (ms) | token/s |
|---|---|---|---|
| DeepSeek-R1-Distill-Qwen-1.5B-GPTQ-Int4 | w4a16 | - | 13.29 |
