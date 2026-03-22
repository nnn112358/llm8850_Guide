# MiniCPM4-0.5B

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_minicpm4

MiniCPM4-0.5B は ModelBest が開発した約5億パラメータの軽量で効率的な多言語対話モデルです。優れた理解力と生成能力を備えており、リソースが限られた環境に適しています。

## モデルのダウンロード

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/MiniCPM4-0.5B)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/MiniCPM4-0.5B
```

### ファイル説明

クローンしたリポジトリには、以下のファイルおよびディレクトリが含まれています。

```
m5stack@raspberrypi:~/rsp/MiniCPM4-0.5B $ ls -lh
total 4.4M
-rw-rw-r-- 1 m5stack m5stack    0 Aug 12 10:58 config.json
-rw-rw-r-- 1 m5stack m5stack 967K Aug 12 11:00 main_ax650
-rw-rw-r-- 1 m5stack m5stack 1.7M Aug 12 11:00 main_axcl_aarch64
-rw-rw-r-- 1 m5stack m5stack 1.8M Aug 12 11:00 main_axcl_x86
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:00 minicpm4-0.5b-int8-ctx-ax630c
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:00 minicpm4-0.5b-int8-ctx-ax650
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 12 11:00 minicpm4_tokenizer
-rw-rw-r-- 1 m5stack m5stack 7.1K Aug 12 10:58 minicpm4_tokenizer_uid.py
-rw-rw-r-- 1 m5stack m5stack  277 Aug 12 10:58 post_config.json
-rw-rw-r-- 1 m5stack m5stack 5.2K Aug 12 10:58 README.md
-rw-rw-r-- 1 m5stack m5stack  642 Aug 12 10:58 run_minicpm4_0.5b_int8_ctx_ax630c.sh
-rw-rw-r-- 1 m5stack m5stack  639 Aug 12 10:58 run_minicpm4_0.5b_int8_ctx_ax650.sh
```

## 仮想環境の作成

推論に必要な Python 仮想環境を作成します。

```bash
python -m venv minicpm
```

## 仮想環境のアクティベート

```bash
source minicpm/bin/activate
```

## 依存パッケージのインストール

```bash
pip install transformers jinja2
```

## tokenizer パーサーの起動

以下のコマンドで tokenizer サービスを起動します。

```bash
python minicpm4_tokenizer_uid.py --port 12345
```

tokenizer サービスを実行すると、Host IP はデフォルトで localhost、ポート番号は 12345 に設定されます。正常に起動した場合、以下のような情報が表示されます。

```
(minicpm) m5stack@raspberrypi:~/rsp/MiniCPM4-0.5B $ python minicpm4_tokenizer_uid.py --port 12345
None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available and only tokenizers, configuration and file/data utilities can be used.
Server running at http://0.0.0.0:12345
```

> **ヒント:** 以下の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

## 実行権限の設定

推論用の実行ファイルに実行権限を付与します。

```bash
chmod +x main_axcl_aarch64
```

## MiniCPM4-0.5B モデル推論サービスの起動

以下のコマンドでモデルの推論サービスを起動します。

```bash
./main_axcl_aarch64 --system_prompt "You are MiniCPM4, created by ModelBest. You are a helpful assistant." --kvcache_path "./kvcache" --template_filename_axmodel "minicpm4-0.5b-int8-ctx-ax650/MiniCPMForCausalLM_p128_l%d_together.axmodel" --axmodel_num 24 --tokenizer_type 2 --url_tokenizer_model "http://127.0.0.1:12345" --filename_post_axmodel "minicpm4-0.5b-int8-ctx-ax650/MiniCPMForCausalLM_post.axmodel" --filename_tokens_embed "minicpm4-0.5b-int8-ctx-ax650/model.embed_tokens.weight.bfloat16.bin" --tokens_embed_num 73448 --tokens_embed_size 1024 --use_mmap_load_embed 0 --live_print 1 --devices 0
```

正常に起動すると、以下のようなログが表示されます。

```
m5stack@raspberrypi:~/rsp/MiniCPM4-0.5B$ ./main_axcl_aarch64 --system_prompt "You are MiniCPM4, created by ModelBest. You are a helpful assistant." --kvcache_path "./kvcache" --template_filename_axmodel "minicpm4-0.5b-int8-ctx-ax650/MiniCPMForCausalLM_p128_l%d_together.axmodel" --axmodel_num 24 --tokenizer_type 2 --url_tokenizer_model "http://127.0.0.1:12345" --filename_post_axmodel "minicpm4-0.5b-int8-ctx-ax650/MiniCPMForCausalLM_post.axmodel" --filename_tokens_embed "minicpm4-0.5b-int8-ctx-ax650/model.embed_tokens.weight.bfloat16.bin" --tokens_embed_num 73448 --tokens_embed_size 1024 --use_mmap_load_embed 0 --live_print 1 --devices 0
[I][                            Init][ 136]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
[I][                            Init][  57]: uid: 9f086096-ff95-40cd-91f0-7551875c5e61
bos_id: 1, eos_id: 73440
  3% | ██                                |   1 /  27 [0.51s<13.72s, 1.97 count/s] tokenizer init ok
  100% | ████████████████████████████████ |  27 /  27 [22.21s<22.21s, 1.22 count/s] init post axmodel ok,remain_cmm(6450 MB)
[I][                            Init][ 237]: max_token_len : 1023
[I][                            Init][ 240]: kv_cache_size : 128, kv_cache_num: 1023
[I][                            Init][ 248]: prefill_token_num : 128
...
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
...
[I][                            main][ 237]: system_prompt: You are MiniCPM4, created by ModelBest. You are a helpful assistant.
prompt >> hello
[I][                      SetKVCache][ 629]: prefill_grpid:2 kv_cache_num:128 precompute_len:25 input_num_token:10
[I][                      SetKVCache][ 632]: current prefill_max_token_num:384
[I][                             Run][ 870]: input token num : 10, prefill_split_num : 1
[I][                             Run][ 902]: input_num_token:10
[I][                             Run][1031]: ttft: 212.91 ms
Hi, Iam a MiniCPM seriesmodel, developedby Modelbestand OpenBMB. Formore information,please visit https://github.com/OpenBMB/

[N][                             Run][1183]: hit eos,avg 21.05 token/s

[I][                      GetKVCache][ 598]: precompute_len:71, remaining:441
prompt >>
```

## ベンチマーク

以下は LLM-8850 上での MiniCPM4-0.5B の推論パフォーマンスです。

| モデル | 量子化方式 | ttft (ms) | token/s |
|---|---|---|---|
| MiniCPM4-0.5B | w8a16 | 212.91 | 21.05 |
