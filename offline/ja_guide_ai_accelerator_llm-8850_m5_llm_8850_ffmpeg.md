# FFmpeg

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_ffmpeg

AXCL FFmpeg は、LLM-8850 カードのハードウェアアクセラレーションによるビデオデコード機能を FFmpeg で利用するためのモジュールです。

## 環境準備

AXCL FFmpeg のダイナミックライブラリは `/usr/lib/axcl/ffmpeg` に、実行ファイルは `/usr/bin/axcl/ffmpeg` にそれぞれ格納されています。

FFmpeg を実行する前に、まずダイナミックライブラリの検索パスを設定する必要があります。以下のコマンドで環境変数を設定してください：

```bash
export LD_LIBRARY_PATH="/usr/lib/axcl/ffmpeg:$LD_LIBRARY_PATH";
```

## 使用方法

以下は、ハードウェアアクセラレーション対応の H.264 デコーダを使用してビデオファイルをデコードする例です：

```bash
m5stack@raspberrypi5:~ $ /usr/bin/axcl/ffmpeg/ffmpeg -c:v h264_axdec -i input.mp4 -f rawvideo -pix_fmt yuv420p  output.yuv
```

## FFmpeg の再コンパイル方法

SDK FFmpeg はバージョン 7.1 をベースに開発されています。コンパイル済みの .so ファイルと FFmpeg バイナリファイルが提供されており、そのまま直接リンクおよび実行が可能です。

FFmpeg を再コンパイルする必要がある場合は、以下の手順に従ってください：

1. [GitHub](https://github.com) から FFmpeg-n7.1.tar.gz をダウンロードし、`axcl/3rdparty/ffmpeg` ディレクトリにコピーします。

2. 解凍します：

```bash
tar -zxvf FFmpeg-n7.1.tar.gz
```

3. パッチを適用します：

```bash
patch -p3 < FFmpeg-n7.1.patch
```

4. コンパイルを実行します：

### arm64

```bash
cd axcl/3rdparty/ffmpeg
make host=arm64 clean all install
```

出力ファイルパス：
- lib: `axcl/out/axcl_linux_arm64/lib/ffmpeg`
- bin: `axcl/out/axcl_linux_arm64/bin/ffmpeg`

### x86

```bash
cd axcl/3rdparty/ffmpeg
make host=x86 clean all install
```

出力ファイルパス：
- lib: `axcl/out/axcl_linux_x86/lib/ffmpeg`
- bin: `axcl/out/axcl_linux_x86/bin/ffmpeg`
