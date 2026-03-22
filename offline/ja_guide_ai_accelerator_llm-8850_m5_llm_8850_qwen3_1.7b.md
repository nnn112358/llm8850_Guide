# Qwen3-1.7B

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_qwen3_1.7b

Qwen3-1.7B は Alibaba Cloud が開発した約17億パラメータの言語モデルです。LLM-8850 AIアクセラレータカード上でテキスト推論および API サービスの提供が可能です。

## モデルのダウンロード

モデルを手動でダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/Qwen3-1.7B
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/Qwen3-1.7B$ ls -lh
total 21M
-rw-rw-r-- 1 m5stack m5stack    0 Aug 12 09:07 config.json
-rw-rw-r-- 1 m5stack m5stack 1.1M Oct 13 09:46 main_api_ax650
-rw-r--r-- 1 m5stack m5stack  132 Oct 13 11:45 main_api_axcl_aarch64
-rw-rw-r-- 1 m5stack m5stack 8.5M Oct 13 09:46 main_api_axcl_x86
-rw-rw-r-- 1 m5stack m5stack 963K Oct 13 09:46 main_ax650
-rw-rw-r-- 1 m5stack m5stack 1.7M Oct 13 09:46 main_axcl_aarch64
-rw-rw-r-- 1 m5stack m5stack 8.1M Oct 13 09:46 main_axcl_x86
-rw-rw-r-- 1 m5stack m5stack  277 Aug 12 09:07 post_config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 09:07 qwen2.5_tokenizer
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 13 11:46 qwen3-1.7b-ax650
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 09:10 qwen3_tokenizer
-rw-rw-r-- 1 m5stack m5stack 7.6K Aug 12 09:07 qwen3_tokenizer_uid.py
-rw-rw-r-- 1 m5stack m5stack  12K Oct 13 09:43 README.md
-rw-rw-r-- 1 m5stack m5stack 2.5K Oct 13 09:43 run_qwen3_1.7b_int8_ctx_ax650.sh
-rw-rw-r-- 1 m5stack m5stack 2.5K Oct 13 09:43 run_qwen3_1.7b_int8_ctx_axcl_aarch64.sh
-rw-rw-r-- 1 m5stack m5stack 2.5K Oct 13 09:43 run_qwen3_1.7b_int8_ctx_axcl_x86_api.sh
-rw-rw-r-- 1 m5stack m5stack 2.5K Oct 13 09:43 run_qwen3_1.7b_int8_ctx_axcl_x86.sh
```

> **ヒント:** 以前に `qwen` の仮想環境を作成済みの場合は、再作成は不要です。アクティベートするだけで構いません。

## 仮想環境の作成

推論に必要な Python 仮想環境を作成します。

```bash
uv venv qwen
```

### 仮想環境のアクティベート

```bash
source qwen/bin/activate
```

### 依存パッケージのインストール

```bash
uv pip install transformers jinja2
```

## tokenizer パーサーの起動

以下のコマンドで tokenizer サービスを起動します。

```bash
python qwen3_tokenizer_uid.py --port 12345
```

tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 12345 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(qwen) m5stack@raspberrypi:~/Qwen3-0.6B $ python qwen3_tokenizer_uid.py --port 12345
None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available and only tokenizers, configuration and file/data utilities can be used.
Server running at http://0.0.0.0:12345
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

## 実行権限の設定

推論用の実行ファイルとスクリプトに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64 run_qwen3_1.7b_int8_ctx_axcl_aarch64.sh
```

## Qwen3 モデル推論サービスの起動

以下のコマンドでモデルの推論サービスを起動します。

```bash
./run_qwen3_1.7b_int8_ctx_axcl_aarch64.sh
```

### 起動成功後の情報

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen3-1.7B$ ./run_qwen3_1.7b_int8_ctx_axcl_aarch64.sh
[I][                            Init][ 136]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
[I][                            Init][  57]: uid: 95e7d5f3-fc8d-48ea-b489-1de9f37924d1
bos_id: -1, eos_id: 151645
  3% | ██                                |   1 /  31 [1.08s<33.54s, 0.92 count/s] tokenizer init ok[I][                            Init][  45]: LLaMaEmbedSelector use mmap
  6% | ███                               |   2 /  31 [1.08s<16.77s, 1.85 count/s] embed_selector init ok
[I][                             run][  30]: AXCLWorker start with devid 0
  100% | ████████████████████████████████ |  31 /  31 [64.75s<64.75s, 0.48 count/s] init post axmodel ok,remain_cmm(3788 MB)
[I][                            Init][ 237]: max_token_len : 2559
[I][                            Init][ 240]: kv_cache_size : 1024, kv_cache_num: 2559
[I][                            Init][ 248]: prefill_token_num : 128
[I][                            Init][ 252]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 252]: grp: 2, prefill_max_token_num : 512
[I][                            Init][ 252]: grp: 3, prefill_max_token_num : 1024
[I][                            Init][ 252]: grp: 4, prefill_max_token_num : 1536
[I][                            Init][ 252]: grp: 5, prefill_max_token_num : 2048
[I][                            Init][ 256]: prefill_max_token_num : 2048
________________________
|    ID| remain cmm(MB)|
========================
|     0|           3788|
¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯
[I][                     load_config][ 282]: load config:
{
    "enable_repetition_penalty": false,
    "enable_temperature": false,
    "enable_top_k_sampling": true,
    "enable_top_p_sampling": false,
    "penalty_window": 20,
    "repetition_penalty": 1.2,
    "temperature": 0.9,
    "top_k": 1,
    "top_p": 0.8
}

[I][                            Init][ 279]: LLM init ok
Type "q" to exit, Ctrl+c to stop current running
[I][          GenerateKVCachePrefill][ 335]: input token num : 21, prefill_split_num : 1 prefill_grpid : 2
[I][          GenerateKVCachePrefill][ 372]: input_num_token:21
[I][                            main][ 236]: precompute_len: 21
[I][                            main][ 237]: system_prompt: You are Qwen, created by Alibaba Cloud. You are a helpful assistant.
prompt >> hello
[I][                      SetKVCache][ 628]: prefill_grpid:2 kv_cache_num:512 precompute_len:21 input_num_token:12
[I][                      SetKVCache][ 631]: current prefill_max_token_num:1920
[I][                             Run][ 869]: input token num : 12, prefill_split_num : 1
[I][                             Run][ 901]: input_num_token:12
[I][                             Run][1030]: ttft: 796.38 ms
<think>

</think>

Hello! How can I assist you today?

[N][                             Run][1182]: hit eos,avg 7.38 token/s

[I][                      GetKVCache][ 597]: precompute_len:46, remaining:2002
prompt >>
```

## API の使い方

Qwen3-1.7B は、対話型推論だけでなく HTTP API 経由でのサービス提供にも対応しています。以下の手順で API サービスを起動できます。

### tokenizer サービスが実行中であることを確認

まず、tokenizer サービスが起動済みであることを確認してください。

```
(qwen) m5stack@raspberrypi:~/Qwen3-0.6B $ python qwen3_tokenizer_uid.py --port 12345
None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available and only tokenizers, configuration and file/data utilities can be used.
Server running at http://0.0.0.0:12345
```

### スクリプトのコピーと実行権限の設定

`run_qwen3_1.7b_int8_ctx_axcl_x86_api.sh` を `run_qwen3_1.7b_int8_ctx_axcl_aarch_api.sh` にコピーして実行権限を設定します。

```bash
cp run_qwen3_1.7b_int8_ctx_axcl_x86_api.sh run_qwen3_1.7b_int8_ctx_axcl_aarch_api.sh
chmod +x main_api_axcl_aarch64 run_qwen3_1.7b_int8_ctx_axcl_aarch_api.sh
```

### `run_qwen3_1.7b_int8_ctx_axcl_aarch_api.sh` のファイル内容を修正

コピーしたスクリプトの内容を以下のように編集してください。

```bash
./main_api_axcl_aarch64 \
--system_prompt "You are Qwen, created by Alibaba Cloud. You are a helpful assistant." \
--template_filename_axmodel "qwen3-1.7b-ax650/qwen3_p128_l%d_together.axmodel" \
--axmodel_num 28 \
--url_tokenizer_model "http://127.0.0.1:12345" \
--filename_post_axmodel qwen3-1.7b-ax650/qwen3_post.axmodel \
--filename_tokens_embed qwen3-1.7b-ax650/model.embed_tokens.weight.bfloat16.bin \
--tokens_embed_num 151936 \
--tokens_embed_size 2048 \
--use_mmap_load_embed 1 \
--devices 0
```

> **注意:** StackFlow が提供する openai-api サービスが既にインストールされている場合は、手動で `sudo systemctl stop llm-openai-api` を実行してサービスを停止する必要があります。

### Qwen3 モデル推論 API サービスの起動

以下のコマンドで API サービスを起動します。

```bash
./run_qwen3_1.7b_int8_ctx_axcl_aarch_api.sh
```

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen3-1.7B $ ./run_qwen3_1.7b_int8_ctx_axcl_aarch_api.sh
[I][                            Init][ 130]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
[I][                            Init][  57]: uid: 3f3c54ef-ddfa-4fbc-bd2f-74523109857e
bos_id: -1, eos_id: 151645
  3% | ██                                |   1 /  31 [0.95s<29.33s, 1.06 count/s] tokenizer init ok[I]
[I][                            Init][ 221]: max_token_len : 2047
[I][                            Init][ 224]: kv_cache_size : 1024, kv_cache_num: 2047
[I][                            Init][ 232]: prefill_token_num : 128
[I][                            Init][ 236]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 236]: grp: 2, prefill_max_token_num : 128
[I][                            Init][ 236]: grp: 3, prefill_max_token_num : 256
[I][                            Init][ 236]: grp: 4, prefill_max_token_num : 384
[I][                            Init][ 236]: grp: 5, prefill_max_token_num : 512
[I][                            Init][ 236]: grp: 6, prefill_max_token_num : 640
[I][                            Init][ 236]: grp: 7, prefill_max_token_num : 768
[I][                            Init][ 236]: grp: 8, prefill_max_token_num : 896
[I][                            Init][ 236]: grp: 9, prefill_max_token_num : 1024
[I][                            Init][ 240]: prefill_max_token_num : 1024
________________________
|    ID| remain cmm(MB)|
========================
|     0|           3665|
¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯
[I][                     load_config][ 282]: load config:
{
    "enable_repetition_penalty": false,
    "enable_temperature": false,
    "enable_top_k_sampling": true,
    "enable_top_p_sampling": false,
    "penalty_window": 20,
    "repetition_penalty": 1.2,
    "temperature": 0.9,
    "top_k": 1,
    "top_p": 0.8
}

[I][                            Init][ 263]: LLM init ok
Server running on port 8000...
```

## API 一覧

利用可能な API エンドポイントは以下の通りです。

| メソッド | パス | 機能 |
|---|---|---|
| GET | /api/stop | 現在の推論タスクを停止します |
| POST | /api/reset | コンテキストをリセットします（新しい system prompt の設定が可能） |
| POST | /api/generate | 非同期でテキストを生成します（ストリーミング出力は /api/generate_provider で取得） |
| GET | /api/generate_provider | 現在の生成の増分出力を取得します（ポーリング用） |
| POST | /api/chat | 同期的に質疑応答を行います（単一ターン） |

### 1. POST /api/generate

テキスト生成を非同期で開始します。

```bash
curl -X POST "http://localhost:8000/api/generate" \
    -H "Content-Type: application/json" \
    -d '{
           "prompt": "Hello, please introduce yourself.",
           "temperature": 0.7,
           "top-k": 40
         }'
```

レスポンス:

```json
{"status": "ok"}
```

説明:
- `prompt` は必須パラメータです
- `temperature`、`top-k`、`top-p`、`repetition_penalty` などはオプションのサンプリングパラメータです
- 呼び出し後すぐに `"status": "ok"` が返され、バックグラウンドで生成が開始されます

### 2. GET /api/generate_provider

生成内容と進捗を取得します（ストリーミングポーリング方式）。

```bash
curl "http://localhost:8000/api/generate_provider"
```

レスポンス:

```json
{"done":false,"response":"<think>\n\n</think>\n\nHello! I'm a large language model developed by Alibaba"}
```

`"done": true` の場合は、生成が完了したことを示します。
200～500ms ごとにリクエストすることで、クライアント側でモデル出力のストリーミング取得を実現できます。

### 3. POST /api/reset

LLM コンテキストをリセット（対話履歴をクリア）し、必要に応じて新しい system prompt を設定できます。

```bash
curl -X POST "http://localhost:8000/api/reset" \
    -H "Content-Type: application/json" \
    -d '{"system_prompt": "You are a helpful assistant."}'
```

レスポンス:

```json
{"status": "ok"}
```

KV cache のクリアや対話シーンの切り替えに使用します。

### 4. GET /api/stop

現在の生成タスクを即座に中断します。

```bash
curl "http://localhost:8000/api/stop"
```

レスポンス:

```json
{"status": "ok"}
```

### 5. POST /api/chat

メッセージを一括入力し、結果を同期的に直接返します（非ストリーミング方式）。

```bash
curl -X POST "http://localhost:8000/api/chat" \
    -H "Content-Type: application/json" \
    -d '{
          "messages": [
            {"role": "user", "content": "Hello, please introduce yourself in one sentence."}
          ],
          "temperature": 0.7
        }'
```

レスポンス:

```json
{"done":true,"message":"<think>\n\n</think>\n\nHi there! I'm a large language model developed by Alibaba Cloud, designed to assist with a wide range of tasks and answer questions."}
```

> **注意:**
> - `/api/generate` + `/api/generate_provider` は非同期/ストリーミングモードです（UI 連携のシーンに適しています）
> - `/api/chat` は同期ブロッキングモードです（完全な回答を一括取得する場合に適しています）
> - モデルが実行中の場合、リクエストは `{"error": "llm is running"}` を返します
> - モデルが初期化されていない場合は `{"error": "Model not init"}` を返します

## 典型的な呼び出しフロー（非同期）

非同期モードでの典型的な呼び出し手順は以下の通りです。

1. `POST /api/generate` で prompt を送信します
2. クライアントが数百ミリ秒ごとに `GET /api/generate_provider` をリクエストします
3. `done:true` が返されたらポーリングを停止します
