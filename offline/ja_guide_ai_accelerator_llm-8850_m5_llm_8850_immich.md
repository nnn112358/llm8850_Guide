# Immich

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_immich

Immich は、オープンソースのセルフホスト型写真・動画管理プラットフォームです。自動バックアップ、スマート検索、クロスデバイスアクセスなどの機能をサポートしており、LLM-8850 の AI アクセラレーションを活用して高速な画像検索を実現できます。

## リソースの取得

[プログラムを手動でダウンロード](https://huggingface.co/AXERA-TECH/immich)して Raspberry Pi 5 にアップロードするか、以下のコマンドでリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)を参照してインストールしてください。

```bash
git clone https://huggingface.co/AXERA-TECH/immich
```

### ファイル説明

以下は、クローンしたリポジトリに含まれるファイルの一覧です。

```
m5stack@raspberrypi:~/rsp/immich $ ls -lh
total 421M
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 10 09:12 asset
-rw-rw-r-- 1 m5stack m5stack 421M Oct 10 09:20 ax-immich-server-aarch64.tar.gz
-rw-rw-r-- 1 m5stack m5stack    0 Oct 10 09:12 config.json
-rw-rw-r-- 1 m5stack m5stack 7.6K Oct 10 09:12 docker-deploy.zip
-rw-rw-r-- 1 m5stack m5stack 104K Oct 10 09:12 immich_ml-1.129.0-py3-none-any.whl
-rw-rw-r-- 1 m5stack m5stack 9.4K Oct 10 09:12 README.md
-rw-rw-r-- 1 m5stack m5stack  177 Oct 10 09:12 requirements.txt
```

## Docker イメージのインポート

> **ヒント:** `docker` がインストールされていない場合は、先に [RaspberryPi docker インストール手順](https://docs.docker.com/engine/install/debian/)を参照してインストールしてください。

```bash
cd immich
docker load -i ax-immich-server-aarch64.tar.gz
```

## 作業ディレクトリの準備

Docker Compose の設定ファイルを展開し、環境変数ファイルを準備します。

```bash
unzip docker-deploy.zip
cp example.env .env
```

## コンテナの起動

以下のコマンドで Immich の Docker コンテナを起動します。

```bash
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

正常に起動すると、以下のような出力が表示されます。

```
m5stack@raspberrypi:~/rsp/immich $ docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
WARN[0000] /home/m5stack/rsp/immich/docker-compose.override.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 3/3
 ✔ Container immich_postgres  Started                                      1.0s
 ✔ Container immich_redis     Started                                      0.9s
 ✔ Container immich_server    Started                                      0.9s
```

## 仮想環境の作成

機械学習サービス用の Python 仮想環境を作成します。

```bash
python -m venv mich
```

## 仮想環境の有効化

作成した仮想環境を有効化します。

```bash
source mich/bin/activate
```

## 依存パッケージのインストール

必要なパッケージをインストールします。

```bash
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl
pip install -r requirements.txt
pip install immich_ml-1.129.0-py3-none-any.whl # プリコンパイルパッケージはアップグレードされる可能性があります。実際のファイル名を使用してください。
```

## immich_ml サービスの起動

以下のコマンドで機械学習サービスを起動します。

```bash
python -m immich_ml
```

正常に起動すると、以下のような出力が表示されます。

```
(mich) m5stack@raspberrypi:~/rsp/immich $ python -m immich_ml
[10/10/25 09:50:12] INFO     Starting gunicorn 23.0.0
[10/10/25 09:50:12] INFO     Listening at: http://[::]:3003 (8698)
[10/10/25 09:50:12] INFO     Using worker: immich_ml.config.CustomUvicornWorker
[10/10/25 09:50:12] INFO     Booting worker with pid: 8699
2025-10-10 09:50:13.589360675 [W:onnxruntime:Default, device_discovery.cc:164 DiscoverDevicesForPlatform] GPU device discovery failed: device_discovery.cc:89 ReadFileContents Failed to open file: "/sys/class/drm/card1/device/vendor"
[INFO] Available providers:  ['AXCLRTExecutionProvider']
/home/m5stack/rsp/immich/mich/lib/python3.11/site-packages/immich_ml/models/clip/cn_vocab.txt
[10/10/25 09:50:16] INFO     Started server process [8699]
[10/10/25 09:50:16] INFO     Waiting for application startup.
[10/10/25 09:50:16] INFO     Created in-memory cache with unloading after 300s
                             of inactivity.
[10/10/25 09:50:16] INFO     Initialized request thread pool with 4 threads.
[10/10/25 09:50:16] INFO     Application startup complete.
```

## Web アクセスと設定

以下の手順で Immich の Web インターフェースを設定します。

1. ブラウザで Raspberry Pi の IP アドレスとポート 3003 を入力してアクセスします（例: `http://192.168.20.27:3003`）。
2. 初回アクセス時は管理者アカウントの登録が必要です。アカウントとパスワードはローカルに保存されます。
3. アカウント登録が完了したら、画像をアップロードできます。
4. 初回は機械学習サーバーの設定が必要です。管理画面から設定ページに移動してください。
5. URL フィールドに Raspberry Pi の IP アドレスとポート 3003 を入力します（例: `http://192.168.20.27:3003`）。
6. CLIP モデルの設定では、中国語検索を使用する場合は `ViT-L-14-336-CN__axera` を、英語検索を使用する場合は `ViT-L-14-336__axera` を入力してください。
7. 設定が完了したら、保存ボタンをクリックします。
8. 初回のみ、Jobs メニューから SMART SEARCH を手動でトリガーする必要があります。
9. 検索バーに画像の説明を入力すると、関連する画像を検索できます。
