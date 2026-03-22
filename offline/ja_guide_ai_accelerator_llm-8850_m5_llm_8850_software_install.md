# LLM-8850 Card 環境設定

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_software_install

## Raspberry Pi 環境設定

以下のドキュメントまたは[公式リンク](https://www.raspberrypi.com/documentation/)を参考にして、Raspberry Pi の環境設定を行ってください。

> **注意:** 以下のコマンドはすべて Raspberry Pi のターミナルで実行します。

PC の BIOS と同様に、EEPROM の設定は OS を書き込んだ TF カードとは独立しています。そのため、最新の Raspberry Pi イメージを書き込んだり、イメージバージョンを切り替えたりしても、EEPROM の設定は自動的に更新されません。

まず、パッケージの update を実行します：

```bash
sudo apt update && sudo apt full-upgrade
```

次に、EEPROM のバージョンを確認します：

```bash
sudo rpi-eeprom-update
```

表示された日付が **2023年12月6日** より前の場合は、以下のコマンドを実行して Raspberry Pi 設定 CLI を開いてください：

```bash
sudo raspi-config
```

Advanced Options > Bootloader Version（ブートローダーバージョン）で Latest（最新）を選択します。その後、Finish または ESC キーで raspi-config を終了してください。

続いて、以下のコマンドを実行してファームウェアを最新バージョンに更新します：

```bash
sudo rpi-eeprom-update -a
```

最後に、以下のコマンドで再起動します。再起動後、EEPROM のファームウェア更新が完了します。

```bash
sudo reboot
```

設定変更と再起動が完了したら、`lspci` コマンドでアクセラレータカードが正しく認識されているか確認できます：

```bash
lspci
```

コマンドの実行結果は以下のようになります：

```
m5stack@raspberrypi5:~ $ lspci
0001:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 30)
0001:01:00.0 Multimedia video controller: Axera Semiconductor Co., Ltd Device 0650 (rev 01)
0002:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 30)
0002:01:00.0 Ethernet controller: Raspberry Pi Ltd RP1 PCIe 2.0 South Bridge
```

このうち `Multimedia video controller: Axera Semiconductor Co., Ltd Device 0650 (rev 01)` が LLM-8850 アクセラレータカードを表しています。

## LLM-8850 Card ドライバインストール

> **ヒント:** 開発ボードではコンパイルサポートが必要です。gcc、make、patch、linux-header-$(uname -r) の各パッケージに依存しているため、事前にインストールしておくか、インストール時にネットワークが使用可能な状態にしてください。

### ソフトウェアソースの更新

まず、以下のコマンドで aarch64/x86 向けの deb パッケージリポジトリを追加します：

```bash
sudo wget -qO /etc/apt/keyrings/StackFlow.gpg https://repo.llm.m5stack.com/m5stack-apt-repo/key/StackFlow.gpg
echo 'deb [signed-by=/etc/apt/keyrings/StackFlow.gpg] https://repo.llm.m5stack.com/m5stack-apt-repo axclhost main' | sudo tee /etc/apt/sources.list.d/axclhost.list
```

次に、以下のコマンドでドライバをインストールします：

```bash
sudo apt update
sudo apt install dkms
sudo apt install axclhost
```

インストールはすぐに完了します。インストール時に環境変数が自動的に追加され、.so ファイルおよび実行可能プログラムが使用可能になります。ただし、実行可能プログラムをすぐに使用するには、現在の bash ターミナルの環境を更新する必要があります：

```bash
source /etc/profile
```

なお、SSH でリモート接続している場合は、SSH を再接続することで環境が自動更新されます（ローカルターミナルの場合は、新しいターミナルを開くことでも自動更新されます）。

以下のコマンドを実行すると、デバイスの情報を確認できます：

```bash
axcl-smi
```

出力例：

```
+------------------------------------------------------------------------------------------------+
| AXCL-SMI  V3.6.4_20250805020145                                  Driver  V3.6.4_20250805020145 |
+-----------------------------------------+--------------+---------------------------------------+
| Card  Name                     Firmware | Bus-Id       | Memory-Usage           |
| Fan   Temp                Pwr:Usage/Cap | CPU      NPU | CMM-Usage              |
|=========================================+==============+=======================================|
| 0  AX650N                     V3.6.4   | 0001:01:00.0 | 148 MiB /      945 MiB |
| --   41C                      -- / --  | 2%        0% | 18 MiB /     7040 MiB  |
+-----------------------------------------+--------------+---------------------------------------+

+------------------------------------------------------------------------------------------------+
| Processes:                                                                                     |
| Card      PID  Process Name                                                   NPU Memory Usage |
|================================================================================================|
+------------------------------------------------------------------------------------------------+
```

詳細については、[AXCL-SMI](ja_guide_ai_accelerator_llm-8850_m5_llm_8850_axcl_smi.md) のセクションをご参照ください。

## LLM-8850 Card ドライバのアンインストール

> **ヒント:** インストールに問題が発生した場合や、ドライバの更新が必要な場合は、以下のコマンドでドライバをアンインストールしてから再インストールしてください。

```bash
sudo apt remove axclhost
```
