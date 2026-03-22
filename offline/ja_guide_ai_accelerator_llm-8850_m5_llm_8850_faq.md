# よくある質問（FAQ）

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_faq

## デバイスが見つからない（エラーコード 0x8030010b）

### 現象

`axcl-smi` を実行すると、以下のようなエラーが表示される：

```text
m5stack@raspberrypi:~ $ axcl-smi
[08-11 16:01:16:933][E][smi][device_manager][  20]: axcl init fail, ret = 0x8030010b
[2025-08-11 16:01:16.933][2110][E][device manager][init][39]: open /dev/axcl_host fail, errno: 2 No such file or directory
```

### トラブルシューティング手順

以下の順序で確認してください：

**1. PCIeでカードが認識されているか確認**

```bash
lspci | grep -Ei "Axera|0650"
```

`Multimedia video controller: Axera Semiconductor Co., Ltd Device 0650 (rev 01)` が表示されれば、カードは物理的に認識されています。何も表示されない場合は、カードが正しく挿入されていない可能性があります。

**2. カードの物理的な接続を確認**

- LLM-8850 Card がスロットにしっかり挿さっているか確認してください
- Raspberry Pi の電源を切り、カードを抜き差しした後、再度電源を入れてください
- 電源供給が十分か確認してください（PiHat使用時: > 9V@3A、M.2 HAT+使用時: DC 5V@3A）

**3. ドライバ（DKMS）が現在のカーネルに対応しているか確認**

カーネルをアップグレードした場合、DKMSが新しいカーネル向けにモジュールをビルドしていない可能性があります：

```bash
uname -r
dkms status | grep -i axcl
```

現在のカーネルバージョンに対応するモジュールが表示されない場合は、再ビルドしてください：

```bash
sudo /usr/sbin/dkms autoinstall -k "$(uname -r)"
sudo modprobe axcl_host
ls -l /dev/axcl_host
axcl-smi
```

**4. ドライバの再インストール**

上記の手順で解決しない場合は、ドライバをアンインストールして再インストールしてください：

```bash
sudo apt remove axclhost
sudo apt update
sudo apt install -y dkms axclhost
source /etc/profile
axcl-smi
```
