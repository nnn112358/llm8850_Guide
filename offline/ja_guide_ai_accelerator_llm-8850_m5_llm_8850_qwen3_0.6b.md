# Qwen3-0.6B

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_qwen3_0.6b

Qwen3-0.6B は Alibaba Cloud が開発した約6億パラメータの軽量言語モデルです。LLM-8850 AIアクセラレータカード上で高速な推論が可能です。

## モデルのダウンロード

モデルを手動でダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/Qwen3-0.6B
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/Qwen3-0.6B $ ls -lh
total 4.4M
-rw-r--r-- 1 m5stack m5stack    0 Jul 25 15:48 config.json
-rw-r--r-- 1 m5stack m5stack 959K Jul 25 15:54 main_ax650
-rw-r--r-- 1 m5stack m5stack 1.7M Jul 25 15:54 main_axcl_aarch64
-rw-r--r-- 1 m5stack m5stack 1.8M Jul 25 15:54 main_axcl_x86
-rw-r--r-- 1 m5stack m5stack  277 Jul 25 15:47 post_config.json
drwxr-xr-x 2 m5stack m5stack 4.0K Jul 25 15:47 qwen2.5_tokenizer
drwxr-xr-x 2 m5stack m5stack 4.0K Jul 25 15:47 qwen3-0.6b-ax630c
drwxr-xr-x 2 m5stack m5stack 4.0K Jul 25 15:47 qwen3-0.6b-ax650
drwxr-xr-x 2 m5stack m5stack 4.0K Jul 25 15:47 qwen3_tokenizer
-rw-r--r-- 1 m5stack m5stack 7.6K Jul 25 15:47 qwen3_tokenizer_uid.py
-rw-r--r-- 1 m5stack m5stack  11K Jul 25 15:47 README.md
-rw-r--r-- 1 m5stack m5stack  577 Jul 25 15:47 run_qwen3_0.6b_int8_ctx_ax630c.sh
-rw-r--r-- 1 m5stack m5stack  574 Jul 25 15:47 run_qwen3_0.6b_int8_ctx_ax650.sh
-rw-r--r-- 1 m5stack m5stack  594 Jul 25 15:47 run_qwen3_0.6b_int8_ctx_axcl_aarch64.sh
-rw-r--r-- 1 m5stack m5stack  590 Jul 25 15:47 run_qwen3_0.6b_int8_ctx_axcl_x86.sh
```

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
chmod +x main_axcl_aarch64 run_qwen3_0.6b_int8_ctx_axcl_aarch64.sh
```

## Qwen3 モデル推論サービスの起動

以下のコマンドでモデルの推論サービスを起動します。

```bash
./run_qwen3_0.6b_int8_ctx_axcl_aarch64.sh
```

### 起動成功後の情報

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen3-0.6B$ ./run_qwen3_0.6b_int8_ctx_axcl_aarch64.sh
[I][                            Init][ 136]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
[I][                            Init][  57]: uid: abf93a3d-2d6a-4ddb-8c9b-42a208a012f7
bos_id: -1, eos_id: 151645
  3% | ██                                |   1 /  31 [1.11s<34.47s, 0.90 count/s] tokenizer init ok[I][Init][  45]: LLaMaEmbedSelector use mmap
  6% | ███                               |   2 /  31 [1.11s<17.25s, 1.80 count/s] embed_selector init ok
  96% | ███████████████████████████████   |  30 /  31 [31.91s<32.98s, 0.94 count/s] init 27 axmodel
  100% | ████████████████████████████████ |  31 /  31 [36.09s<36.09s, 0.86 count/s] init post axmodel ok,remain_cmm(5068 MB)
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
|     0|           5068|
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
[I][                             Run][1030]: ttft: 670.51 ms
<think>

</think>

Hello! How can I assist you today?

[N][                             Run][1182]: hit eos,avg 12.88 token/s

[I][                      GetKVCache][ 597]: precompute_len:46, remaining:2002
prompt >>
```

## ベンチマーク

以下は LLM-8850 上での各モデルの推論パフォーマンスの比較です。

| モデル | 量子化方式 | ttft (ms) | token/s |
|---|---|---|---|
| Qwen3-0.6B | w8a16 | 670.51 | 12.88 |
| Qwen3-1.7B | w8a16 | 796.38 | 7.38 |
| Qwen2.5-0.5B | w4a16 | - | 27.05 |
| Qwen2.5-1.5B | w4a16 | - | 15.06 |
