# Git LFS インストール手順

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_git_lfs

Git LFS（Large File Storage）は、大容量ファイルを効率的に管理するための Git 拡張機能です。LLM モデルなどの大きなファイルをリポジトリからクローンする際に必要となります。以下の手順でインストールしてください。

## apt ソースの更新

まず、パッケージリストを最新の状態に更新します：

```bash
sudo apt update
```

## Git LFS のインストール

次に、Git LFS パッケージをインストールします：

```bash
sudo apt install git-lfs
```

## Git LFS の初期化

最後に、Git LFS を初期化します。これにより、Git で大容量ファイルの追跡が有効になります：

```bash
git lfs install
```

> **ヒント:** 初期化は一度実行すれば、以降のリポジトリ操作で自動的に Git LFS が使用されます。
