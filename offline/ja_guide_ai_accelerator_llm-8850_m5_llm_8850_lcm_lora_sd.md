# lcm-lora-sd

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_lcm_lora_sd

LCM-LoRA-SD は、Latent Consistency Model（LCM）と LoRA を組み合わせた Stable Diffusion v1.5 ベースの高速画像生成モデルです。少ないステップ数で高品質な画像を生成できます。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/AXERA-TECH/lcm-lora-sdv1-5)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/lcm-lora-sdv1-5
```

### ファイル説明

以下は、クローンしたリポジトリに含まれるファイルの一覧です。

```
m5stack@raspberrypi:~/rsp/lcm-lora-sdv1-5 $ ls -lh
total 100K
drwxrwxr-x 2 m5stack m5stack 4.0K Aug 11 17:10 asserts
-rw-rw-r-- 1 m5stack m5stack    0 Aug 11 17:09 config.json
-rw-rw-r-- 1 m5stack m5stack 1.2K Aug 11 17:09 Disclaimer.md
-rw-rw-r-- 1 m5stack m5stack 1.5K Aug 11 17:09 LICENSE
drwxrwxr-x 4 m5stack m5stack 4.0K Aug 11 17:14 models
-rw-rw-r-- 1 m5stack m5stack 5.7K Aug 11 17:09 README.md
-rw-rw-r-- 1 m5stack m5stack  117 Aug 11 17:09 requirements.txt
-rw-rw-r-- 1 m5stack m5stack  23K Aug 11 17:09 run_img2img_axe_infer.py
-rw-rw-r-- 1 m5stack m5stack  23K Aug 11 17:09 run_img2img_onnx_infer.py
-rw-rw-r-- 1 m5stack m5stack 7.8K Aug 11 17:09 run_txt2img_axe_infer_new.py
-rw-rw-r-- 1 m5stack m5stack 7.3K Aug 11 17:09 run_txt2img_axe_infer.py
-rw-rw-r-- 1 m5stack m5stack 7.4K Aug 11 17:09 run_txt2img_onnx_infer.py
```

## 仮想環境の作成

まず、Python の仮想環境を作成します。

```bash
python -m venv sd
```

## 仮想環境の有効化

作成した仮想環境を有効化します。

```bash
source sd/bin/activate
```

## 依存パッケージのインストール

必要なパッケージをインストールします。

```bash
sudo apt install cmake -y
pip install -r requirements.txt
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
```

## テキストから画像生成（txt2img）

テキストプロンプトから画像を生成するには、以下のコマンドを実行します。

```bash
python run_txt2img_axe_infer.py
```

### 実行結果

```
(sd) m5stack@raspberrypi:~/rsp/lcm-lora-sdv1-5$ python run_txt2img_axe_infer.py
[INFO] Available providers:  ['AXCLRTExecutionProvider']
prompt: Self-portrait oil painting, a beautiful cyborg with golden hair, 8k
text_tokenizer: ./models/tokenizer
text_encoder: ./models/text_encoder
unet_model: ./models/unet.axmodel
vae_decoder_model: ./models/vae_decoder.axmodel
time_input: ./models/time_input_txt2img.npy
save_dir: ./txt2img_output_axe.png
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.4 9215b7e5
text encoder take 4466.2ms
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 972f38ca
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 972f38ca
load models take 22582.9ms
unet once take 433.6ms
unet once take 433.5ms
unet once take 433.3ms
unet once take 433.5ms
unet loop take 1736.7ms
vae inference take 914.2ms
save image take 254.3ms
```

## 画像から画像生成（img2img）

既存の画像をベースに新しい画像を生成するには、以下のコマンドを実行します。

```bash
python run_img2img_axe_infer.py
```

### 実行結果

```
(sd) m5stack@raspberrypi:~/rsp/lcm-lora-sdv1-5$ python run_txt2img_axe_infer.py
[INFO] Available providers:  ['AXCLRTExecutionProvider']
prompt: Self-portrait oil painting, a beautiful cyborg with golden hair, 8k
text_tokenizer: ./models/tokenizer
text_encoder: ./models/text_encoder
unet_model: ./models/unet.axmodel
vae_decoder_model: ./models/vae_decoder.axmodel
time_input: ./models/time_input_txt2img.npy
save_dir: ./txt2img_output_axe.png
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.4 9215b7e5
text encoder take 2962.1ms
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 972f38ca
[INFO] Using provider: AXCLRTExecutionProvider
[INFO] SOC Name: AX650N
[INFO] VNPU type: VNPUType.DISABLED
[INFO] Compiler version: 3.3 972f38ca
load models take 14483.6ms
unet once take 433.6ms
unet once take 433.3ms
unet once take 433.2ms
unet once take 433.4ms
unet loop take 1736.4ms
vae inference take 914.0ms
save image take 233.2ms
```
