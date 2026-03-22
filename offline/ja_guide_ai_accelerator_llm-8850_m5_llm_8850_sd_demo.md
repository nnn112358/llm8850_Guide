# SD1.5-LLM8850

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_sd_demo

SD1.5-LLM8850 は、Stable Diffusion v1.5 をベースとした画像生成デモアプリケーションです。Web インターフェースを通じて、テキストプロンプトからの画像生成を手軽に体験できます。

## モデルの取得

[モデルを手動でダウンロード](https://huggingface.co/M5Stack/SD1.5-LLM8850)して Raspberry Pi 5 にアップロードするか、以下のコマンドでモデルリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)を参照してインストールしてください。

```bash
git clone https://huggingface.co/M5Stack/SD1.5-LLM8850
```

### ファイル説明

以下は、クローンしたリポジトリに含まれるファイルの一覧です。

```
m5stack@raspberrypi5:~/rsp/SD1.5-LLM8850 $ ls -lh
total 92K
-rw-rw-r-- 1 m5stack m5stack  12K Sep 26 11:07 api_10steps.py
-rw-rw-r-- 1 m5stack m5stack  12K Sep 26 11:07 api_server.py
-rw-rw-r-- 1 m5stack m5stack  19K Sep 26 11:07 axengine-0.1.3-py3-none-any.whl
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 26 11:11 client
drwxrwxr-x 2 m5stack m5stack 4.0K Sep 26 11:11 client_jp
-rw-rw-r-- 1 m5stack m5stack  13K Sep 26 11:07 dpm20_infer.py
-rw-rw-r-- 1 m5stack m5stack 1.3K Sep 26 11:07 gen_img.py
drwxrwxr-x 4 m5stack m5stack 4.0K Sep 26 11:08 models
-rw-rw-r-- 1 m5stack m5stack 2.5K Sep 26 11:10 README.md
-rw-rw-r-- 1 m5stack m5stack  112 Sep 26 11:07 requirements.txt
-rw-rw-r-- 1 m5stack m5stack  165 Sep 26 11:07 sd-launch.desktop
-rw-rw-r-- 1 m5stack m5stack 2.4K Sep 26 11:07 sd-launch.sh
```

## 仮想環境の作成

まず、Python の仮想環境を作成します。

```bash
uv venv sd
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
uv pip install -r requirements.txt
uv pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
```

## バックエンドの起動

API サーバーを起動します。

```bash
uvicorn api_10steps:app --host 0.0.0.0 --port 7888
```

## Web インターフェースの起動

新しいターミナルを開き、以下のコマンドを実行して Web フロントエンドを起動します。

```bash
source sd/bin/activate
cd client && python app.py
```

ブラウザで `http://127.0.0.1:5000` にアクセスしてください。

画像生成パラメータを選択するか、「ランダム記述」を選択して「即座に生成」ボタンをクリックすると、画像が生成されます。
