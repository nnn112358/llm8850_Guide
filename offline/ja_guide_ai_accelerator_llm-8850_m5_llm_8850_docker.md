# Raspberry Pi Docker インストール手順

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_docker

この手順では、Raspberry Pi 上に Docker をインストールする方法を説明します。以下のコマンドを順番に実行してください。

## apt ソースの更新

まず、パッケージリストを最新の状態に更新します：

```bash
sudo apt update
```

## 依存パッケージのインストール

Docker のインストールに必要な依存パッケージをインストールします：

```bash
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
```

## Docker 公式 GPG 鍵の取得

次に、Docker の公式 GPG 鍵をダウンロードして設定します：

```bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

## Docker 公式 apt リポジトリの追加

Docker の公式 apt リポジトリアドレスを追加します：

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## apt ソースの再更新

リポジトリを追加した後、パッケージリストを再度更新します：

```bash
sudo apt update
```

## Docker および関連コンポーネントのインストール

Docker 本体および関連するコンポーネントをインストールします：

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 現在のユーザーを docker グループに追加

最後に、現在のユーザーを docker グループに追加します。これにより、`sudo` なしで Docker コマンドを実行できるようになります：

```bash
sudo usermod -aG docker $USER
newgrp docker
```

> **ヒント:** グループの変更を完全に反映するには、ログアウトして再度ログインすることを推奨します。
