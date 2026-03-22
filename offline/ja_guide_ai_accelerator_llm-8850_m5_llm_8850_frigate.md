# Frigate

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_frigate

Frigate は、リアルタイム AI 物体検出機能を備えたオープンソースの NVR（ネットワークビデオレコーダー）です。すべての処理はローカルハードウェア上で実行されるため、カメラの映像ストリームが外部に送信されることはありません。

## リソースの取得

[プログラムを手動でダウンロード](https://huggingface.co/AXERA-TECH/frigate-resource)して Raspberry Pi 5 にアップロードするか、以下のコマンドでリポジトリをクローンしてください。

> **ヒント:** `git lfs` がインストールされていない場合は、先に [git lfs インストール手順](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_git_lfs.md)を参照してインストールしてください。

```bash
git clone -b rpi-axcl https://huggingface.co/AXERA-TECH/frigate-resource
```

### ファイル説明

以下は、クローンしたリポジトリに含まれるファイルの一覧です。

```
m5stack@raspberrypi:~/rsp/frigate-resource $ ls -lh
total 2.8G
-rw-rw-r-- 1 m5stack m5stack  48M Oct  9 16:46 axcl_host_aarch64_V3.6.5_20250908154509_NO4973.deb
-rw-rw-r-- 1 m5stack m5stack  648 Oct  9 16:41 docker-compose.yml
-rw-rw-r-- 1 m5stack m5stack 2.8G Oct  9 16:46 frigate-rpi-axcl-f8f387a.tar
-rw-rw-r-- 1 m5stack m5stack 3.7K Oct  9 16:41 README.md
```

## Docker イメージのインポート

> **ヒント:** `docker` がインストールされていない場合は、先に [RaspberryPi docker インストール手順](https://docs.docker.com/engine/install/debian/)を参照してインストールしてください。

```bash
docker load -i frigate-resource/frigate-rpi-axcl-f8f387a.tar # イメージファイルはアップグレードされる可能性があります。実際のファイル名を使用してください。
```

## 作業ディレクトリの準備

設定ファイルとストレージ用のディレクトリを作成し、docker-compose ファイルをコピーします。

```bash
mkdir -p ~/frigate-runtime/{config,storage}
cp frigate-resource/docker-compose.yml ~/frigate-runtime/
```

## コンテナの起動

以下のコマンドで Frigate コンテナを起動します。

```bash
cd ~/frigate-runtime/
docker compose up -d
```

起動後、ブラウザで `https://server_ip:8971` にアクセスして Frigate 管理画面を開きます。

> **ヒント:** デフォルトのログイン情報は以下の通りです。
> - ユーザー名: `admin`
> - パスワード: `axera123456`

## 設定

管理画面の設定メニューを開き、以下の設定例を参考にパラメータを入力します。`go2rtc:` セクションにはお使いの IP カメラのアドレスを指定してください。設定を保存した後、Frigate を再起動します。

### 設定例

```yaml
#ffmpegグローバル変数、必須
ffmpeg:
  global_args: ["-hide_banner", "-loglevel", "warning", "-threads", "1"]
  output_args:
    detect: ["-threads", "1", "-f", "rawvideo",  "-pix_fmt", "yuv420p"]

mqtt:
  enabled: false

go2rtc:
  streams:
    #メインストリーム
    road1:
      - rtsp://192.168.20.57:8554/road1.264
    #サブストリーム
    road1_sub:
      - rtsp://192.168.20.57:8554/road1_sub.264
cameras:
  road1:
    enabled: true
    ffmpeg:
      inputs:
        #録画ストリームのパス、ここではgo2rtcで設定したメインストリームを使用
        #デバッグ段階ではローカルストリームファイルを使用可能
        - path: rtsp://127.0.0.1:8554/road1
          roles:
            - record
        #検出ストリームのパス、ここではgo2rtcで設定したサブストリームを使用
        #デバッグ段階ではローカルストリームファイルを使用可能
        - path: rtsp://127.0.0.1:8554/road1_sub
          roles:
            - detect
      #preset-rpi-64-h264はh264ストリームのデコード用
      #preset-rpi-64-h265はh265ストリームのデコード用
      hwaccel_args: preset-rpi-64-h264

record:
  enabled: true

#検出機能を有効化
#検出の幅と高さを設定しない場合、検出ストリームのネイティブ解像度がデフォルトで使用されます
detect:
  enabled: true
  width: 1280
  height: 720
  fps: 5

#検出エンジンにaxengineを使用するよう設定
detectors:
  axengine:
    type: axengine

#axengineの物体検出モデルを設定
model:
  path: yolov5s_320
  width: 320
  height: 320
  input_pixel_format: bgr
  labelmap_path: /labelmap/coco-80.txt

#追跡する物体の種類
objects:
  track:
    - person
    - car
    - bicycle
    - motorcycle

version: 0.16-0
```

設定完了後、プレビュー画面でカメラ映像と検出結果を確認できます。
