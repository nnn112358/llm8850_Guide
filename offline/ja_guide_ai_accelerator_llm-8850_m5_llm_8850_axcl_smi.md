# AXCL-SMI

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_axcl_smi

## 概要

AXCL-SMI（System Management Interface）は、LLM-8850 カードのデバイス情報の収集やデバイスの設定を行うための管理ツールです。以下のデバイス情報の収集をサポートしています：

- ハードウェアデバイスモデル
- ファームウェアバージョン
- ドライババージョン
- デバイス使用率
- メモリ使用状況
- デバイスチップ接合温度
- その他の情報

## 使用説明

### クイック使用

AXCL ドライバパッケージを正しくインストールすると、AXCL-SMI も自動的にインストールされます。`axcl-smi` を実行すると、以下のような内容が表示されます：

```
+------------------------------------------------------------------------------------------------+
| AXCL-SMI  V3.6.4_20250805020145                                  Driver  V3.6.4_20250805020145 |
+-----------------------------------------+--------------+---------------------------------------+
| Card  Name                     Firmware | Bus-Id       |                          Memory-Usage |
| Fan   Temp                Pwr:Usage/Cap | CPU      NPU |                             CMM-Usage |
|=========================================+==============+=======================================|
|    0  AX650N                     V3.6.4 | 0001:01:00.0 |                149 MiB /      945 MiB |
|   --   41C                      -- / -- | 1%        0% |                 18 MiB /     7040 MiB |
+-----------------------------------------+--------------+---------------------------------------+

+------------------------------------------------------------------------------------------------+
| Processes:                                                                                     |
| Card      PID  Process Name                                                   NPU Memory Usage |
|================================================================================================|
```

### フィールド説明

| フィールド | 説明 |
|---|---|
| Card | デバイスインデックス番号（PCIe のデバイス番号ではない点にご注意ください） |
| Name | デバイス名称 |
| Fan | ファン回転速度比（未サポート） |
| Temp | デバイスチップ接合温度 Tj |
| Firmware | デバイスファームウェアバージョン |
| Pwr: Usage/Cap | 消費電力（未サポート） |
| Bus-Id | デバイス Bus ID |
| CPU | CPU 平均使用率 |
| NPU | NPU 平均使用率 |
| Memory-Usage | システムメモリ：使用量 / 合計 |
| CMM-Usage | メディアメモリ：使用量 / 合計 |
| PID | ホスト制御プロセスの PID |
| Process Name | ホスト制御プロセス名 |
| NPU Memory Usage | デバイス NPU で使用中の CMM メモリ量 |

> **注意:** DDR 帯域幅と NPU 使用率の詳細については、FAQ の「DDR 帯域幅」と「NPU 使用率」のセクションを参照してください。

## ヘルプ (-h) とバージョン (-v)

### axcl-smi -h

ヘルプ情報を表示します：

```
m5stack@raspberrypi5:~ $ axcl-smi -h
usage: axcl-smi [<command> [<args>]] [--device] [--version] [--help]

axcl-smi System Management Interface V3.6.3_20250722020142

Commands
    info                                    Show device information
        --temp                                  Show SoC temperature
        --mem                                   Show memory usage
        --cmm                                   Show CMM usage
        --cpu                                   Show CPU usage
        --npu                                   Show NPU usage
    proc                                    cat device proc
        --vdec                                  cat /proc/ax_proc/vdec
        --venc                                  cat /proc/ax_proc/venc
        --jenc                                  cat /proc/ax_proc/jenc
        --ivps                                  cat /proc/ax_proc/ivps
        --rgn                                   cat /proc/ax_proc/rgn
        --ive                                   cat /proc/ax_proc/ive
        --pool                                  cat /proc/ax_proc/pool
        --link                                  cat /proc/ax_proc/link_table
        --cmm                                   cat /proc/ax_proc/mem_cmm_info
    set                                     Set
        -f[MHz], --freq=[MHz]                   Set CPU frequency in MHz. One of: 1200000, 1400000, 1700000
    log                                     Dump logs from device
        -t[mask], --type=[mask]                 Specifies which logs to dump by a combination (bitwise OR) value of blow:
                                                  -1: all (default) 0x01: daemon 0x02: worker 0x10: syslog 0x20: kernel
        -o[path], --output=[path]               Specifies the path to save dump logs (default: ./)
    sh                                      Execute a shell command
        cmd                                     Shell command
        args...                                 Shell command arguments
    reboot                                  reboot device
-d, --device                            Card index [0, connected cards number - 1]
-v, --version                           Show axcl-smi version
-h, --help                              Show this help menu
```

### axcl-smi -v

AXCL-SMI ツールのバージョンを表示します：

```
m5stack@raspberrypi5:~ $ axcl-smi -v
AXCL-SMI V3.6.3_20250722020142 BUILD: Jul 22 2025 02:30:24
```

## オプション

### デバイス番号 (-d, --device)

```
-d, --device                             Card index [0, connected cards number - 1]
```

`[-d, --device]` でデバイスインデックスを指定します。指定範囲は [0, 接続デバイス数 - 1] で、**デフォルトはデバイス 0** です。

### 情報クエリ (info)

`axcl-smi info` は、デバイスの詳細情報を表示するためのコマンドです。以下のサブコマンドをサポートしています：

| サブコマンド | 説明 |
|---|---|
| --temp | デバイスチップ接合温度を表示（単位：摂氏 x1000） |
| --mem | デバイスシステムの詳細メモリ使用状況を表示 |
| --cmm | デバイスメディアメモリ使用状況を表示。より詳細なメディアメモリ情報は `axcl-smi sh cat /proc/ax_proc/mem_cmm_info -d xx`（xx は PCIe デバイス番号）で取得可能 |
| --cpu | デバイス CPU 使用率を表示 |
| --npu | デバイス NPU 使用率を表示 |

**例：** インデックス 0 のデバイスのメディアメモリ使用状況を表示します：

```
m5stack@raspberrypi5:~ $ axcl-smi info --cmm -d 0
Device ID           : 1 (0x1)
CMM Total           :  7208960 KiB
CMM Used            :    18876 KiB
CMM Remain          :  7190084 kiB
```

### PROC クエリ (proc)

`axcl-smi proc` は、デバイスモジュールの proc 情報をクエリするためのコマンドです。以下のサブコマンドをサポートしています：

| サブコマンド | 説明 |
|---|---|
| --vdec | VDEC モジュール proc をクエリ (`cat /proc/ax_proc/vdec`) |
| --venc | VENC モジュール proc をクエリ (`cat /proc/ax_proc/venc`) |
| --jenc | JENC モジュール proc をクエリ (`cat /proc/ax_proc/jenc`) |
| --ivps | IVPS モジュール proc をクエリ (`cat /proc/ax_proc/ivps`) |
| --rgn | RGN モジュール proc をクエリ (`cat /proc/ax_proc/rgn`) |
| --ive | IVE モジュール proc をクエリ (`cat /proc/ax_proc/ive`) |
| --pool | POOL モジュール proc をクエリ (`cat /proc/ax_proc/pool`) |
| --link | LINK モジュール proc をクエリ (`cat /proc/ax_proc/link_table`) |
| --cmm | CMM モジュール proc をクエリ (`cat /proc/ax_proc/mem_cmm_info`) |

**例：** デバイス 0 の VENC proc 情報をクエリします：

```
m5stack@raspberrypi5:~ $ axcl-smi proc --venc -d 0
-------- VENC VERSION ------------------------
[Axera version]: ax_venc V3.6.3_20250722020142 Jul 22 2025 02:22:04 JK

-------- MODULE PARAM ------------------------
MaxChnNum   MaxRoiNum   MaxProcNum
64          8           32
```

### パラメータ設定 (set)

`axcl-smi set` は、デバイスの設定を変更するためのコマンドです。以下のサブコマンドをサポートしています：

| サブコマンド | 説明 |
|---|---|
| -f[MHz], --freq=[MHz] | デバイスの CPU 周波数を設定。1200000、1400000、1700000 の 3 種類のみサポート |

**例：** インデックス 0 のデバイスの CPU 周波数を 1200MHz に設定します：

```
m5stack@raspberrypi5:~ $ axcl-smi set -f 1200000 -d 0
set cpu frequency 1200000 to device 1 succeed.
```

### ログダウンロード (log)

`axcl-smi log` は、デバイスのログファイルをホスト側にダウンロードするためのコマンドです。以下のパラメータをサポートしています：

| パラメータ | 説明 |
|---|---|
| -t[mask], --type=[mask] | ダウンロードするログカテゴリを指定。デバイス側のログカテゴリ：-1: 全ログ、0x01: デーモンプロセス、0x02: ビジネスプロセス、0x10: syslog、0x20: カーネルログ。`-1` で全ログのダウンロードを推奨 |
| -o[path], --output=[path] | ログの保存パスを指定。絶対パスと相対パスの両方をサポート。デフォルトはカレントディレクトリ。ディレクトリに書き込み権限が必要 |

**例：** インデックス 0 のデバイスの全ログをダウンロードし、カレントディレクトリに保存します：

```
m5stack@raspberrypi5:~ $ axcl-smi log -d 0
[2025-07-25 10:04:30.332][1802][C][log][dump][73]: log dump finished: ./dev1_log_20250724210251.tar.gz
```

### shell コマンド (sh)

`axcl-smi sh` は、shell コマンドでデバイスの情報をクエリするためのコマンドです。主にデバイス側モジュールの実行 proc 情報をクエリするために使用されます。

**例：** インデックス 0 のデバイスの CMM 情報をクエリします：

```
m5stack@raspberrypi5:~ $ axcl-smi sh cat /proc/ax_proc/mem_cmm_info -d 0
--------------------SDK VERSION-------------------
[Axera version]: ax_cmm V3.6.3_20250722020142 Jul 22 2025 02:21:25 JK
+---PARTITION: Phys(0x148000000, 0x2FFFFFFFF), Size=7208960KB(7040MB),    NAME="anonymous"
 nBlock(Max=0, Cur=23, New=0, Free=0)  nbytes(Max=0B(0KB,0MB), Cur=19329024B(18876KB,18MB), New=0B(0KB,0MB), Free=0B(0KB,0MB))  Block(Max=0B(0KB,0MB), Min=0B(0KB,0MB), Avg=0B(0KB,0MB))
   |-Block: phys(0x148000000, 0x148013FFF), cache =non-cacheable, length=80KB(0MB),    name="TDP_DEV"
   |-Block: phys(0x148014000, 0x148014FFF), cache =non-cacheable, length=4KB(0MB),    name="TDP_CMODE3"
   ...

---CMM_USE_INFO:
 total size=7208960KB(7040MB),used=18876KB(18MB + 444KB),remain=7190084KB(7021MB + 580KB),partition_number=1,block_number=23
```

> **注意:** shell コマンドのパラメータに `-`、`--`、`>` などの特殊文字が含まれる場合は、ダブルクォーテーション `"-l"` でコマンドとパラメータを 1 つの文字列にまとめることができます。例：`axcl-smi sh "ls -l" -d 0`。なお、shell コマンドでデバイスの設定を変更する場合は慎重に行ってください。

### 再起動 (reboot)

`axcl-smi reboot` コマンドは、指定デバイスをリセットし、その後自動的にファームウェアを再ロードします：

```
m5stack@raspberrypi5:~ $ axcl-smi reboot
Do you want to reboot device 0 ? (y/n): y
```
