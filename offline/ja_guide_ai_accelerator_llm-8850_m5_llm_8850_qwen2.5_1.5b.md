# Qwen2.5-1.5B

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_qwen2.5_1.5b

Qwen2.5-1.5B-Instruct-GPTQ-Int4 は Alibaba Cloud が開発した約15億パラメータの言語モデルの量子化版です。GPTQ Int4 量子化により、LLM-8850 上でバランスの取れた推論性能を発揮します。

## モデルのダウンロード

モデルを手動でダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/Qwen2.5-1.5B-Instruct-GPTQ-Int4
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/Qwen2.5-1.5B-Instruct-GPTQ-Int4$ ls -lh
total 2.9M
-rw-rw-r-- 1 m5stack m5stack    0 Aug 12 10:48 config.json
-rw-rw-r-- 1 m5stack m5stack 976K Aug 12 10:48 main_axcl_aarch64
-rw-rw-r-- 1 m5stack m5stack 999K Aug 12 10:48 main_axcl_x86
-rw-rw-r-- 1 m5stack m5stack 932K Aug 12 10:48 main_prefill
-rw-rw-r-- 1 m5stack m5stack  277 Aug 12 10:48 post_config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 10:49 qwen2.5-1.5b-gptq-int4-ax650
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 10:48 qwen2.5_tokenizer
-rw-rw-r-- 1 m5stack m5stack 4.2K Aug 12 10:48 qwen2.5_tokenizer.py
-rw-rw-r-- 1 m5stack m5stack 6.8K Aug 12 10:48 README.md
-rw-rw-r-- 1 m5stack m5stack  521 Aug 12 10:48 run_qwen2.5_1.5b_gptq_int4_ax650.sh
-rw-rw-r-- 1 m5stack m5stack  526 Aug 12 10:48 run_qwen2.5_1.5b_gptq_int4_axcl_aarch64.sh
-rw-rw-r-- 1 m5stack m5stack  522 Aug 12 10:48 run_qwen2.5_1.5b_gptq_int4_axcl_x86.sh
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
python qwen2.5_tokenizer.py --port 12345
```

tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 12345 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(qwen) m5stack@raspberrypi:~/rsp/Qwen2.5-1.5B-Instruct-GPTQ-Int4 $ python qwen2.5_tokenizer.py --port 12345
None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available and only tokenizers, configuration and file/data utilities can be used.
None None 151645 <|im_end|>
<|im_start|>system
You are Qwen, created by Alibaba Cloud. You are a helpful assistant.<|im_end|>
<|im_start|>user
hello world<|im_end|>
<|im_start|>assistant

[151644, 8948, 198, 2610, 525, 1207, 16948, 11, 3465, 553, 54364, 14817, 13, 1446, 525, 264, 10950, 17847, 13, 151645, 198, 151644, 872, 198, 14990, 1879, 151645, 198, 151644, 77091, 198]
http://localhost:12345
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

## 実行権限の設定

推論用の実行ファイルとスクリプトに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64 run_qwen2.5_1.5b_gptq_int4_axcl_aarch64.sh
```

## Qwen2.5 モデル推論サービスの起動

以下のコマンドでモデルの推論サービスを起動します。

```bash
./run_qwen2.5_1.5b_gptq_int4_axcl_aarch64.sh
```

### 起動成功後の情報

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/Qwen2.5-1.5B-Instruct-GPTQ-Int4$ ./run_qwen2.5_1.5b_gptq_int4_axcl_aarch64.sh
build time: Feb 13 2025 15:44:57
[I][                            Init][ 111]: LLM init start
bos_id: -1, eos_id: 151645
  3% | ██                                |   1 /  31 [0.00s<0.09s, 333.33 count/s] tokenizer init
  100% | ████████████████████████████████ |  31 /  31 [28.75s<28.75s, 1.08 count/s] init post axmodel ok
[I][                            Init][ 226]: max_token_len : 1024
[I][                            Init][ 231]: kv_cache_size : 256, kv_cache_num: 1024
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
Hello! How can I assist you today?

[N][                             Run][ 610]: hit eos,avg 15.03 token/s

>>
```
