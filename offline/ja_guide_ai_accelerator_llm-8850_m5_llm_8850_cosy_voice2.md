# CosyVoice2

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_cosy_voice2

CosyVoice2 は、LLM ベースの高品質な音声合成（TTS）モデルです。自然で滑らかな音声を生成でき、中国語・英語を含む多言語に対応しています。本セクションでは、LLM-8850 上で CosyVoice2 を実行する手順を説明します。

## モデルのダウンロード

手動でモデルをダウンロードして Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md) を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/CosyVoice2
```

## ファイル説明

```
m5stack@raspberrypi:~/rsp/CosyVoice2 $ ls -lh
total 26M
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 27 15:54 asset
-rwxrwxr-x 1 m5stack m5stack    0 Sep 18 14:32 config.json
drwxrwxr-x 2 m5stack m5stack 4.0K Sep  5 19:12 CosyVoice-BlankEN-Ax650-prefill_512
drwxrwxr-x 2 m5stack m5stack 4.0K Sep  5 19:11 frontend-onnx
-rwxrwxr-x 1 m5stack m5stack 9.3M Sep 27 15:33 main_api_ax650
-rwxrwxr-x 1 m5stack m5stack 1.9M Sep 27 15:33 main_api_axcl_aarch64
-rwxrwxr-x 1 m5stack m5stack 2.0M Sep 27 15:33 main_api_axcl_x86
-rwxrwxr-x 1 m5stack m5stack 9.2M Sep 18 09:35 main_ax650
-rwxrwxr-x 1 m5stack m5stack 1.8M Sep 27 15:33 main_axcl_aarch64
-rwxrwxr-x 1 m5stack m5stack 1.9M Sep 27 15:33 main_axcl_x86
drwxrwxr-x 3 m5stack m5stack 4.0K Sep 27 15:33 onnxruntime-linux-aarch64-1.23.0
drwxrwxr-x 3 m5stack m5stack 4.0K Sep 27 15:33 onnxruntime-linux-x64-1.23.0
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 15 10:07 prompt_files
-rwxrwxr-x 1 m5stack m5stack 7.9K Sep 27 15:33 README.md
-rwxrwxr-x 1 m5stack m5stack  895 Sep 27 15:33 run_api_ax650.sh
-rwxrwxr-x 1 m5stack m5stack  997 Sep 27 15:33 run_api_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  989 Sep 27 15:33 run_api_axcl_x86.sh
-rwxrwxr-x 1 m5stack m5stack  865 Sep 27 15:33 run_ax650.sh
-rwxrwxr-x 1 m5stack m5stack  967 Sep 27 15:33 run_axcl_aarch64.sh
-rwxrwxr-x 1 m5stack m5stack  959 Sep 27 15:33 run_axcl_x86.sh
drwxrwxr-x 5 m5stack m5stack 4.0K Sep 27 15:33 scripts
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 27 15:33 token2wav-axmodels
```

## 仮想環境の作成

```bash
python -m venv cosyvoice
```

## 仮想環境の有効化

```bash
source cosyvoice/bin/activate
```

## 依存パッケージのインストール

```bash
pip install -r scripts/requirements.txt
```

## tokenizer サービスの起動

以下のコマンドで tokenizer サービスを起動します。

```bash
cd scripts
python cosyvoice2_tokenizer.py
```

tokenizer サービスが起動すると、デフォルトでホスト IP は localhost、ポート番号は 12345 に設定されます。起動後の出力情報は以下の通りです。

```
(cosyvoice) m5stack@raspberrypi:~/rsp/CosyVoice2/scripts $ python cosyvoice2_tokenizer.py
[14990, 1879]
http://localhost:12345
```

> **ヒント:** 以降の操作を行うには、Raspberry Pi で新しいターミナルを開く必要があります。

## CosyVoice2 モデルの単発推論

以下のコマンドを実行すると、単発推論が行われ、出力音声ファイル `output.wav` が生成されます。

```bash
./run_axcl_aarch64.sh
```

起動が成功すると、以下のような情報が表示されます。

```
m5stack@raspberrypi:~/rsp/CosyVoice2 $ ./run_axcl_aarch64.sh
rm: cannot remove 'output*.wav': No such file or directory
[I][                            main][ 291]: device: 0
[I][                             run][  30]: AXCLWorker start with devid 0
[I][                            Init][ 135]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
bos_id: 0, eos_id: 1773
  3% | ██                                |   1 /  27 [0.00s<0.03s, 1000.00 count  7% | ███                               |   2 /  27 [0.13s<1.81s, 14.93 count/s] embed_selector init ok[I][                            Init][ 174]: attr.axmodel_num:24
 11% | ████                              |   3 /  27 [1.35s<12.13s, 2.23 count/s 14% | ███████████████████████████████   |  26 /  27 [7.92s<8.22s, 3.28 count/s]100% | ████████████████████████████████ |  27 /  27 [10.59s<10.59s, 2.55 count/s] init post axmodel ok,remain_cmm(6323 MB)
[I][                            Init][ 225]: max_token_len : 1023
[I][                            Init][ 230]: kv_cache_size : 128, kv_cache_num: 1023
[I][                            Init][ 238]: prefill_token_num : 128
[I][                            Init][ 242]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 242]: grp: 2, prefill_max_token_num : 128
[I][                            Init][ 242]: grp: 3, prefill_max_token_num : 256
[I][                            Init][ 242]: grp: 4, prefill_max_token_num : 384
[I][                            Init][ 242]: grp: 5, prefill_max_token_num : 512
[I][                            Init][ 246]: prefill_max_token_num : 512
________________________
|    ID| remain cmm(MB)|
========================
|     0|           6323|
¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯
[I][                            Init][ 308]: LLM init ok
2025-09-29 11:09:20.814330123 [W:onnxruntime:, device_discovery.cc:164 DiscoverDevicesForPlatform] GPU device discovery failed: device_discovery.cc:89 ReadFileContents Failed to open file: "/sys/class/drm/card1/device/vendor"

inputs:
                 mel: 1 x 80 x 50
output:
                   s: 1 x 1 x 24000

inputs:
                 mel: 1 x 80 x 58
output:
                   s: 1 x 1 x 27840
[I][                            Init][ 213]: Token2Wav init ok, remain_cmm(5863 MB)
[I][                             Run][ 449]: input token num : 142, prefill_split_num : 2
[I][                             Run][ 483]: input_num_token:128
[I][                             Run][ 483]: input_num_token:14
[I][                             Run][ 642]: ttft: 339.72 ms
[Main/Token2Wav Thread] Processing batch of 28 tokens...
Successfully saved audio to output_0.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 53 tokens...
Successfully saved audio to output_1.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_2.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_3.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_4.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_5.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_6.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_7.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_8.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_9.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_10.wav (32-bit Float PCM).
[I][                             Run][ 765]: hit eos, llm finished
[I][                             Run][ 795]: llm finished
[Main/Token2Wav Thread] Buffer is empty and LLM finished. Exiting.

[I][                             Run][ 800]: total decode tokens:300
[N][                             Run][ 801]: hit eos, decode avg 14.54 token/s

Successfully saved audio to output_11.wav (32-bit Float PCM).
Successfully saved audio to output.wav (32-bit Float PCM).
[I][                             tts][ 225]: tts total use time: 21.555 s

Voice generation pipeline completed.
[I][                             run][  80]: AXCLWorker exit with devid 0
```

## CosyVoice2 対話型推論モード

対話型推論モードを使用するには、`run_axcl_aarch64.sh` 内の `continue` パラメータを `1` に変更してから再度実行します。このモードでは、テキストを入力するたびに音声が生成され、出力音声ファイルは `output.wav` に保存されます。

```bash
m5stack@raspberrypi:~/rsp/CosyVoice2 $ cat ./run_axcl_aarch64.sh
export LD_LIBRARY_PATH=onnxruntime-linux-aarch64-1.23.0/lib:$LD_LIBRARY_PATH

LLM_DIR=CosyVoice-BlankEN-Ax650-prefill_512/
TOKEN2WAV_DIR=token2wav-axmodels/

rm output*.wav
./main_axcl_aarch64 \
--template_filename_axmodel "${LLM_DIR}/qwen2_p128_l%d_together.axmodel" \
--token2wav_axmodel_dir $TOKEN2WAV_DIR \
--n_timesteps 10 \
--axmodel_num 24 \
--bos 0 --eos 0 \
--filename_tokenizer_model "http://127.0.0.1:12345" \
--filename_post_axmodel "${LLM_DIR}/qwen2_post.axmodel" \
--filename_decoder_axmodel "${LLM_DIR}/llm_decoder.axmodel" \
--filename_tokens_embed "${LLM_DIR}/model.embed_tokens.weight.bfloat16.bin" \
--filename_llm_embed "${LLM_DIR}/llm.llm_embedding.float16.bin" \
--filename_speech_embed "${LLM_DIR}/llm.speech_embedding.float16.bin" \
--continue 1 \
--devices "0," \
--prompt_files prompt_files \
--text "君不见黄河之水天上来，奔流到海不复回。君不见高堂明镜悲白发，朝如青丝暮成雪。"

chmod 777 output*.wav
```

起動が成功すると、以下のような情報が表示されます。

```
m5stack@raspberrypi:~/rsp/CosyVoice2 $ ./run_axcl_aarch64.sh
[I][                            main][ 291]: device: 0
[I][                             run][  30]: AXCLWorker start with devid 0
[I][                            Init][ 135]: LLM init start
[I][                            Init][  34]: connect http://127.0.0.1:12345 ok
bos_id: 0, eos_id: 1773
  3% | ██                                |   1 /  27 [0.00s<0.03s, 1000.00 count  7% | ████████████████████████████████ |  27 /  27 [10.57s<10.57s, 2.55 count/s] init post axmodel ok,remain_cmm(6323 MB)
[I][                            Init][ 225]: max_token_len : 1023
[I][                            Init][ 230]: kv_cache_size : 128, kv_cache_num: 1023
[I][                            Init][ 238]: prefill_token_num : 128
[I][                            Init][ 242]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 242]: grp: 2, prefill_max_token_num : 128
[I][                            Init][ 242]: grp: 3, prefill_max_token_num : 256
[I][                            Init][ 242]: grp: 4, prefill_max_token_num : 384
[I][                            Init][ 242]: grp: 5, prefill_max_token_num : 512
[I][                            Init][ 246]: prefill_max_token_num : 512
________________________
|    ID| remain cmm(MB)|
========================
|     0|           6323|
¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯
[I][                            Init][ 308]: LLM init ok
2025-09-29 11:22:34.715409537 [W:onnxruntime:, device_discovery.cc:164 DiscoverDevicesForPlatform] GPU device discovery failed: device_discovery.cc:89 ReadFileContents Failed to open file: "/sys/class/drm/card1/device/vendor"

inputs:
                 mel: 1 x 80 x 50
output:
                   s: 1 x 1 x 24000

inputs:
                 mel: 1 x 80 x 58
output:
                   s: 1 x 1 x 27840
[I][                            Init][ 213]: Token2Wav init ok, remain_cmm(5863 MB)
[I][                             Run][ 449]: input token num : 142, prefill_split_num : 2
[I][                             Run][ 483]: input_num_token:128
[I][                             Run][ 483]: input_num_token:14
[I][                             Run][ 642]: ttft: 342.93 ms
[Main/Token2Wav Thread] Processing batch of 28 tokens...
Successfully saved audio to output_0.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 53 tokens...
Successfully saved audio to output_1.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_2.wav (32-bit Float PCM).
...
[I][                             Run][ 765]: hit eos, llm finished
[I][                             Run][ 795]: llm finished

[I][                             Run][ 800]: total decode tokens:278
[N][                             Run][ 801]: hit eos, decode avg 14.37 token/s

Successfully saved audio to output_10.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Buffer is empty and LLM finished. Exiting.
Successfully saved audio to output_11.wav (32-bit Float PCM).
Successfully saved audio to output.wav (32-bit Float PCM).
[I][                             tts][ 225]: tts total use time: 20.305 s

Voice generation pipeline completed.
Type "q" to exit, Ctrl+c to stop current running
text >> 很高兴认识你，Nice to meet you
[I][                             Run][ 449]: input token num : 115, prefill_split_num : 1
[I][                             Run][ 483]: input_num_token:115
[I][                             Run][ 642]: ttft: 153.41 ms
[Main/Token2Wav Thread] Processing batch of 28 tokens...
Successfully saved audio to output_0.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 53 tokens...
Successfully saved audio to output_1.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_2.wav (32-bit Float PCM).
[I][                             Run][ 765]: hit eos, llm finished
[I][                             Run][ 795]: llm finished

[I][                             Run][ 800]: total decode tokens:93
[N][                             Run][ 801]: hit eos, decode avg 15.73 token/s

[Main/Token2Wav Thread] Buffer is empty and LLM finished. Exiting.
Successfully saved audio to output_3.wav (32-bit Float PCM).
Successfully saved audio to output.wav (32-bit Float PCM).
[I][                             tts][ 225]: tts total use time: 6.617 s

Voice generation pipeline completed.
text >>
```
