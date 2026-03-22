# SDK API

ソース: https://docs.m5stack.com/ja/guide/ai_accelerator/llm-8850/m5_llm_8850_axcl_api

## 概要

AXCL API は、LLM-8850 カードを制御するためのプログラミングインターフェースで、2 つの部分で構成されています。第 1 部分は **Runtime API**、第 2 部分は **Native API** です。

Runtime API は独立した API の組み合わせであり、現在はメモリ管理用の Memory API と、NPU を駆動するための Engine API が含まれています。AXCL API をコンピューティングカード形態で使用し、エンコード/デコード機能を使用しない場合は、Runtime API のみで全ての計算タスクを完了できます。エンコード/デコード機能が必要な場合は、Native API および FFmpeg モジュールの関連内容を理解する必要があります。

---

## runtime

Runtime API を使用すると、ホストシステムから NPU を呼び出して計算処理を実行できます。Memory API ではホストとコンピューティングカードのそれぞれでメモリ空間の確保・解放が可能です。また、Engine API ではモデルの初期化、IO 設定から推論まで、全ての NPU 機能を利用できます。

### axclInit

```c
axclError axclInit(const char *config);
```

**説明:** システムを初期化する同期インターフェースです。

**パラメータ:**
- `config [IN]`: JSON 設定ファイルのパスを指定します。ユーザーは JSON 設定ファイルでシステムパラメータを設定できます（現在はログレベルをサポート、フォーマットは FAQ を参照）。NULL または存在しない JSON ファイルを渡した場合、システムはデフォルト設定を使用します。

**制限:**
- `axclFinalize` とペアで呼び出してシステムをクリーンアップしてください。
- AXCL インターフェースを呼び出してアプリケーションを開発する際は、最初にこのインターフェースを呼び出す必要があります。
- 1 プロセス内で 1 回のみ呼び出してください。

### axclFinalize

```c
axclError axclFinalize();
```

**説明:** システムの非初期化を行い、プロセス内の AXCL リソースを解放する同期インターフェースです。

**制限:**
- `axclInit` とペアで呼び出してください。
- アプリケーションプロセスの終了前に、明示的にこのインターフェースを呼び出して非初期化を行ってください。
- C++ アプリケーションの場合、デストラクタ内での呼び出しは推奨しません（シングルトンの破棄順序が不定のため、プロセス異常終了の原因になることがあります）。

### axclrtGetVersion

```c
axclError axclrtGetVersion(int32_t *major, int32_t *minor, int32_t *patch);
```

**説明:** システムバージョン番号をクエリする同期インターフェースです。

**パラメータ:**
- `major [OUT]`: メジャーバージョン番号
- `minor [OUT]`: マイナーバージョン番号
- `patch [OUT]`: パッチバージョン番号

**制限:** 特に制限はありません。

### axclrtGetSocName

```c
const char *axclrtGetSocName();
```

**説明:** 現在のチップ SoC の文字列名をクエリする同期インターフェースです。

**制限:** 特に制限はありません。

### axclrtSetDevice

```c
axclError axclrtSetDevice(int32_t deviceId);
```

**説明:** 現在のプロセスまたはスレッドで使用するデバイスを指定し、同時に暗黙的にデフォルト Context を作成する同期インターフェースです。

**パラメータ:**
- `deviceId [IN]`: デバイス ID

**制限:**
- 内部で暗黙的に作成されるデフォルト Context は `axclrtResetDevice` でシステムが自動回収します。`axclrtDestroyContext` で明示的に破棄することはできません。
- 同一プロセスの複数スレッドで同じ deviceId を指定した場合、暗黙的に作成される Context も同一になります。
- `axclrtResetDevice` とペアで呼び出してデバイスリソースを解放してください。内部参照カウントにより複数回の呼び出しが可能で、参照カウントが 0 の時のみリソースが解放されます。
- マルチ Device シナリオでは、このインターフェースまたは `axclrtSetCurrentContext` で Device を切り替えることができます。

### axclrtResetDevice

```c
axclError axclrtResetDevice(int32_t deviceId);
```

**説明:** デバイスをリセットし、暗黙的または明示的に作成された Context を含むデバイス上のリソースを解放する同期インターフェースです。

**パラメータ:**
- `deviceId [IN]`: デバイス ID

**制限:**
- `axclrtCreateContext` で明示的に作成した Context は、`axclrtDestroyContext` で先に破棄してからこのインターフェースでデバイスリソースを解放することを推奨します。
- `axclrtSetDevice` とペアで使用してください。
- 内部参照カウントにより複数回の呼び出しが可能です。
- アプリケーションプロセスの終了時に `axclrtResetDevice` が呼び出されることを確認してください（特に異常シグナルのキャッチ処理後）。

### axclrtGetDevice

```c
axclError axclrtGetDevice(int32_t *deviceId);
```

**説明:** 現在使用中のデバイス ID を取得する同期インターフェースです。

**パラメータ:**
- `deviceId [OUT]`: デバイス ID

**制限:** `axclrtSetDevice` または `axclrtCreateContext` でデバイスを指定していない場合、エラーを返します。

### axclrtGetDeviceCount

```c
axclError axclrtGetDeviceCount(uint32_t *count);
```

**説明:** 接続されたデバイスの総数を取得する同期インターフェースです。

**パラメータ:**
- `count [OUT]`: デバイス数

**制限:** 特に制限はありません。

### axclrtGetDeviceList

```c
axclError axclrtGetDeviceList(axclrtDeviceList *deviceList);
```

**説明:** 接続された全デバイスの ID を取得する同期インターフェースです。

**パラメータ:**
- `deviceList [OUT]`: 全接続デバイスの ID 情報

**制限:** 特に制限はありません。

### axclrtSynchronizeDevice

```c
axclError axclrtSynchronizeDevice();
```

**説明:** 現在のデバイスの全タスクを同期実行する同期インターフェースです。

**制限:** 少なくとも 1 つのデバイスが有効化されている必要があります。

### axclrtGetDeviceProperties

```c
axclError axclrtGetDeviceProperties(int32_t deviceId, axclrtDeviceProperties *properties);
```

**説明:** デバイスの UID、CPU 使用率、NPU 使用率、メモリなどの情報を取得する同期インターフェースです。

### axclrtCreateContext

```c
axclError axclrtCreateContext(axclrtContext *context, int32_t deviceId);
```

**説明:** 現在のスレッドで明示的に Context を作成する同期インターフェースです。

**パラメータ:**
- `context [OUT]`: 作成された Context ハンドル
- `deviceId [IN]`: デバイス ID

**制限:**
- ユーザーが作成したサブスレッドで AXCL API を呼び出す場合は、このインターフェースで明示的に Context を作成するか、`axclrtSetCurrentContext` で Context をバインドする必要があります。
- 指定デバイスが有効化されていない場合、内部でまずデバイスを有効化します。
- `axclrtDestroyContext` で明示的に Context リソースを解放してください。
- 複数スレッドで 1 つの Context を共有することも可能ですが（`axclrtSetCurrentContext` でバインド）、タスクの実行はスレッドスケジューリングの順序に依存します。マルチスレッド環境では各スレッドに専用の Context を作成することを推奨します。

### axclrtDestroyContext

```c
axclError axclrtDestroyContext(axclrtContext context);
```

**説明:** 明示的に Context を破棄する同期インターフェースです。

**パラメータ:**
- `context [IN]`: Context ハンドル

**制限:** `axclrtCreateContext` で作成した Context リソースのみ破棄できます。

### axclrtSetCurrentContext

```c
axclError axclrtSetCurrentContext(axclrtContext context);
```

**説明:** スレッドが実行する Context をバインドする同期インターフェースです。

**パラメータ:**
- `context [IN]`: Context ハンドル

**制限:**
- 複数回呼び出した場合、最後に設定した Context が適用されます。
- バインドする Context に対応する Device が `axclrtResetDevice` でリセット済みの場合、その Context をスレッドに設定することはできません。

### axclrtGetCurrentContext

```c
axclError axclrtGetCurrentContext(axclrtContext *context);
```

**説明:** スレッドにバインドされた Context ハンドルを取得する同期インターフェースです。

**パラメータ:**
- `context [OUT]`: 現在のコンテキストハンドル

**制限:** `axclrtSetCurrentContext` でバインドするか、`axclrtCreateContext` で Context を作成した後に取得できます。

---

## memory

### axclrtMalloc

```c
axclError axclrtMalloc(void **devPtr, size_t size, axclrtMemMallocPolicy policy);
```

**説明:** デバイス側で非 CACHED 物理メモリを確保し、`devPtr` で確保されたメモリへのポインタを返す同期インターフェースです。

**パラメータ:**
- `devPtr [OUT]`: 確保されたデバイス側物理メモリポインタ
- `size [IN]`: 確保するメモリサイズ（バイト単位）
- `policy [IN]`: メモリ確保ルール（現在未使用）

**制限:**
- デバイス側の CMM メモリプールから連続物理メモリを確保します。
- 非 CACHED メモリのため、一貫性処理は不要です。
- `axclrtFree` でメモリを解放してください。
- 頻繁なメモリの確保・解放はパフォーマンスに影響するため、事前確保または二次管理を推奨します。

### axclrtMallocCached

```c
axclError axclrtMallocCached(void **devPtr, size_t size, axclrtMemMallocPolicy policy);
```

**説明:** デバイス側で CACHED 物理メモリを確保する同期インターフェースです。

**制限:**
- デバイス側の CMM メモリプールから連続物理メモリを確保します。
- CACHED メモリのため、ユーザーが一貫性処理を行う必要があります。
- `axclrtFree` でメモリを解放してください。

### axclrtFree

```c
axclError axclrtFree(void *devPtr);
```

**説明:** デバイス側で確保されたメモリを解放する同期インターフェースです。

**制限:** `axclrtMalloc` または `axclrtMallocCached` で確保したデバイス側メモリのみ解放できます。

### axclrtMemFlush

```c
axclError axclrtMemFlush(void *devPtr, size_t size);
```

**説明:** キャッシュ内のデータを DDR にフラッシュし、キャッシュの内容を無効にする同期インターフェースです。

**パラメータ:**
- `devPtr [IN]`: フラッシュ対象の DDR メモリ開始アドレスポインタ
- `size [IN]`: フラッシュ対象の DDR メモリサイズ（バイト単位）

### axclrtMemInvalidate

```c
axclError axclrtMemInvalidate(void *devPtr, size_t size);
```

**説明:** キャッシュ内のデータを無効にする同期インターフェースです。

**パラメータ:**
- `devPtr [IN]`: キャッシュを無効にする DDR メモリ開始アドレスポインタ
- `size [IN]`: DDR メモリサイズ（バイト単位）

### axclrtMallocHost

```c
axclError axclrtMallocHost(void **hostPtr, size_t size);
```

**説明:** HOST 上で仮想メモリを確保する同期インターフェースです。

**パラメータ:**
- `hostPtr [OUT]`: 確保されたメモリの先頭アドレス
- `size [IN]`: 確保するメモリサイズ（バイト単位）

**制限:**
- `axclrtMallocHost` で確保したメモリは `axclrtFreeHost` で解放する必要があります。
- HOST でのメモリ確保は malloc でも可能ですが、`axclrtMallocHost` の使用を推奨します。

### axclrtFreeHost

```c
axclError axclrtFreeHost(void *hostPtr);
```

**説明:** `axclrtMallocHost` で確保したメモリを解放する同期インターフェースです。

**制限:** `axclrtMallocHost` で確保した HOST メモリのみ解放できます。

### axclrtMemset

```c
axclError axclrtMemset(void *devPtr, uint8_t value, size_t count);
```

**説明:** `axclrtMalloc` または `axclrtMallocCached` で確保したデバイス側メモリを初期化する同期インターフェースです。

**パラメータ:**
- `devPtr [IN]`: 初期化対象のデバイス側メモリ先頭アドレス
- `value [IN]`: 設定する値
- `count [IN]`: 初期化するメモリの長さ（バイト単位）

**制限:**
- `axclrtMallocCached` で確保したメモリの初期化後は、`axclrtMemInvalidate` で一貫性を保つ必要があります。
- `axclrtMallocHost` で確保した HOST メモリは、memset 関数で初期化してください。

### axclrtMemcpy

```c
axclError axclrtMemcpy(void *dstPtr, const void *srcPtr, size_t count, axclrtMemcpyKind kind);
```

**説明:** HOST 内、HOST と DEVICE 間、DEVICE 内の同期メモリコピーを実行する同期インターフェースです。

**パラメータ:**
- `dstPtr [IN]`: 宛先メモリアドレスポインタ
- `srcPtr [IN]`: ソースメモリアドレスポインタ
- `count [IN]`: コピーするメモリ長（バイト単位）
- `kind [IN]`: メモリコピーの種類:
  - `AXCL_MEMCPY_HOST_TO_HOST`: HOST 内のメモリコピー
  - `AXCL_MEMCPY_HOST_TO_DEVICE`: HOST 仮想メモリから DEVICE へのコピー
  - `AXCL_MEMCPY_DEVICE_TO_HOST`: DEVICE から HOST 仮想メモリへのコピー
  - `AXCL_MEMCPY_DEVICE_TO_DEVICE`: DEVICE 内のメモリコピー
  - `AXCL_MEMCPY_HOST_PHY_TO_DEVICE`: HOST 連続物理メモリから DEVICE へのコピー
  - `AXCL_MEMCPY_DEVICE_TO_HOST_PHY`: DEVICE から HOST 連続物理メモリへのコピー

### axclrtMemcmp

```c
axclError axclrtMemcmp(const void *devPtr1, const void *devPtr2, size_t count);
```

**説明:** DEVICE 内のメモリ比較を実行する同期インターフェースです。

**パラメータ:**
- `devPtr1 [IN]`: デバイス側アドレス 1 のポインタ
- `devPtr2 [IN]`: デバイス側アドレス 2 のポインタ
- `count [IN]`: 比較長（バイト単位）

**制限:** デバイス側メモリの比較のみサポートしています。メモリが同一の場合のみ AXCL_SUCC (0) を返します。

---

## engine

### axclrtEngineInit

```c
axclError axclrtEngineInit(axclrtEngineVNpuKind npuKind);
```

**説明:** Runtime Engine を初期化する関数です。Runtime Engine を使用する前に呼び出す必要があります。

**パラメータ:**
- `npuKind [IN]`: 初期化する VNPU タイプを指定

**制限:** Runtime Engine の使用完了後、`axclrtEngineFinalize` でクリーンアップする必要があります。

### axclrtEngineGetVNpuKind

```c
axclError axclrtEngineGetVNpuKind(axclrtEngineVNpuKind *npuKind);
```

**説明:** Runtime Engine で初期化された VNPU タイプを取得する関数です。

**パラメータ:**
- `npuKind [OUT]`: VNPU タイプを返す

**制限:** 事前に `axclrtEngineInit` を呼び出す必要があります。

### axclrtEngineFinalize

```c
axclError axclrtEngineFinalize();
```

**説明:** Runtime Engine のクリーンアップを完了する関数です。全ての操作完了後に呼び出してください。

**制限:** 事前に `axclrtEngineInit` を呼び出す必要があります。

### axclrtEngineLoadFromFile

```c
axclError axclrtEngineLoadFromFile(const char *modelPath, uint64_t *modelId);
```

**説明:** ファイルからモデルデータをロードし、モデル ID を作成する関数です。

**パラメータ:**
- `modelPath [IN]`: オフラインモデルファイルの格納パス
- `modelId [OUT]`: ロード後に生成されるモデル ID（後続の操作で識別子として使用）

### axclrtEngineLoadFromMem

```c
axclError axclrtEngineLoadFromMem(const void *model, uint64_t modelSize, uint64_t *modelId);
```

**説明:** メモリからオフラインモデルデータをロードし、システム内部でモデル実行用メモリを管理する関数です。

**パラメータ:**
- `model [IN]`: メモリ内のモデルデータ
- `modelSize [IN]`: モデルデータのサイズ
- `modelId [OUT]`: ロード後に生成されるモデル ID

**制限:** モデルメモリはデバイスメモリでなければならず、ユーザーが管理・解放する必要があります。

### axclrtEngineUnload

```c
axclError axclrtEngineUnload(uint64_t modelId);
```

**説明:** 指定モデル ID のモデルをアンロードする関数です。

### axclrtEngineGetModelCompilerVersion

```c
const char* axclrtEngineGetModelCompilerVersion(uint64_t modelId);
```

**説明:** モデルビルドツールチェーンのバージョンを取得する関数です。

### axclrtEngineSetAffinity

```c
axclError axclrtEngineSetAffinity(uint64_t modelId, axclrtEngineSet set);
```

**説明:** モデルの NPU アフィニティを設定する関数です。

**制限:** ゼロは不可です。設定するマスクビットはアフィニティ範囲を超えてはなりません。

### axclrtEngineGetAffinity

```c
axclError axclrtEngineGetAffinity(uint64_t modelId, axclrtEngineSet *set);
```

**説明:** モデルの NPU アフィニティを取得する関数です。

### axclrtEngineGetUsage

```c
axclError axclrtEngineGetUsage(const char *modelPath, int64_t *sysSize, int64_t *cmmSize);
```

**説明:** モデルファイルに基づいて、モデル実行に必要なシステムメモリサイズと CMM メモリサイズを取得する関数です。

**パラメータ:**
- `modelPath [IN]`: メモリ情報取得用のモデルパス
- `sysSize [OUT]`: モデル実行に必要なワーキングシステムメモリサイズ
- `cmmSize [OUT]`: モデル実行に必要な CMM メモリサイズ

### axclrtEngineGetUsageFromMem

```c
axclError axclrtEngineGetUsageFromMem(const void *model, uint64_t modelSize, int64_t *sysSize, int64_t *cmmSize);
```

**説明:** メモリ内のモデルデータに基づいて、モデル実行に必要なメモリ使用量を取得する関数です。

### axclrtEngineGetUsageFromModelId

```c
axclError axclrtEngineGetUsageFromModelId(uint64_t modelId, int64_t *sysSize, int64_t *cmmSize);
```

**説明:** モデル ID に基づいて、モデル実行に必要なメモリ使用量を取得する関数です。

### axclrtEngineGetModelType

```c
axclError axclrtEngineGetModelType(const char *modelPath, axclrtEngineModelKind *modelType);
```

**説明:** モデルファイルに基づいてモデルタイプを取得する関数です。

### axclrtEngineGetModelTypeFromMem

```c
axclError axclrtEngineGetModelTypeFromMem(const void *model, uint64_t modelSize, axclrtEngineModelKind *modelType);
```

**説明:** メモリ内のモデルデータに基づいてモデルタイプを取得する関数です。

### axclrtEngineGetModelTypeFromModelId

```c
axclError axclrtEngineGetModelTypeFromModelId(uint64_t modelId, axclrtEngineModelKind *modelType);
```

**説明:** モデル ID に基づいてモデルタイプを取得する関数です。

### axclrtEngineGetIOInfo

```c
axclError axclrtEngineGetIOInfo(uint64_t modelId, axclrtEngineIOInfo *ioInfo);
```

**説明:** モデル ID に基づいてモデルの IO 情報を取得する関数です。

**制限:** モデル ID を破棄する前に、`axclrtEngineDestroyIOInfo` で `axclrtEngineIOInfo` を解放してください。

### axclrtEngineDestroyIOInfo

```c
axclError axclrtEngineDestroyIOInfo(axclrtEngineIOInfo ioInfo);
```

**説明:** `axclrtEngineIOInfo` 型のデータを破棄する関数です。

### axclrtEngineGetShapeGroupsCount

```c
axclError axclrtEngineGetShapeGroupsCount(axclrtEngineIOInfo ioInfo, int32_t *count);
```

**説明:** IO シェイプグループの数を取得する関数です。

**制限:** Pulsar2 ツールチェーンではモデル変換時に複数シェイプを指定できます。通常のモデルはシェイプが 1 つのみであるため、通常この関数を呼び出す必要はありません。

### axclrtEngineGetNumInputs / axclrtEngineGetNumOutputs

```c
uint32_t axclrtEngineGetNumInputs(axclrtEngineIOInfo ioInfo);
uint32_t axclrtEngineGetNumOutputs(axclrtEngineIOInfo ioInfo);
```

**説明:** モデルの入力数または出力数を取得する関数です。

### axclrtEngineGetInputSizeByIndex / axclrtEngineGetOutputSizeByIndex

```c
uint64_t axclrtEngineGetInputSizeByIndex(axclrtEngineIOInfo ioInfo, uint32_t group, uint32_t index);
uint64_t axclrtEngineGetOutputSizeByIndex(axclrtEngineIOInfo ioInfo, uint32_t group, uint32_t index);
```

**説明:** 指定した入力または出力のサイズを取得する関数です。

**パラメータ:**
- `ioInfo [IN]`: axclrtEngineIOInfo ポインタ
- `group [IN]`: シェイプグループインデックス
- `index [IN]`: サイズを取得する入力/出力のインデックス（0 始まり）

### axclrtEngineGetInputNameByIndex / axclrtEngineGetOutputNameByIndex

```c
const char *axclrtEngineGetInputNameByIndex(axclrtEngineIOInfo ioInfo, uint32_t index);
const char *axclrtEngineGetOutputNameByIndex(axclrtEngineIOInfo ioInfo, uint32_t index);
```

**説明:** 指定した入力または出力の名前を取得する関数です。

**制限:** 返されるテンソル名のライフサイクルは `ioInfo` と同一です。

### axclrtEngineGetInputIndexByName / axclrtEngineGetOutputIndexByName

```c
int32_t axclrtEngineGetInputIndexByName(axclrtEngineIOInfo ioInfo, const char *name);
int32_t axclrtEngineGetOutputIndexByName(axclrtEngineIOInfo ioInfo, const char *name);
```

**説明:** テンソル名に基づいて入力または出力のインデックスを取得する関数です。

### axclrtEngineGetInputDims / axclrtEngineGetOutputDims

```c
axclError axclrtEngineGetInputDims(axclrtEngineIOInfo ioInfo, uint32_t group, uint32_t index, axclrtEngineIODims *dims);
axclError axclrtEngineGetOutputDims(axclrtEngineIOInfo ioInfo, uint32_t group, uint32_t index, axclrtEngineIODims *dims);
```

**説明:** 指定した入力または出力のディメンション情報を取得する関数です。

**制限:** `axclrtEngineIODims` のストレージ空間はユーザーが確保し、`axclrtEngineIOInfo` を破棄する前に解放してください。

### axclrtEngineCreateIO / axclrtEngineDestroyIO

```c
axclError axclrtEngineCreateIO(axclrtEngineIOInfo ioInfo, axclrtEngineIO *io);
axclError axclrtEngineDestroyIO(axclrtEngineIO io);
```

**説明:** `axclrtEngineIO` 型データの作成および破棄を行う関数です。

**制限（作成時）:** モデル ID を破棄する前に `axclrtEngineDestroyIO` で解放してください。

### axclrtEngineSetInputBufferByIndex / axclrtEngineSetOutputBufferByIndex

```c
axclError axclrtEngineSetInputBufferByIndex(axclrtEngineIO io, uint32_t index, const void *dataBuffer, uint64_t size);
axclError axclrtEngineSetOutputBufferByIndex(axclrtEngineIO io, uint32_t index, const void *dataBuffer, uint64_t size);
```

**説明:** IO インデックスで入力/出力データバッファを設定する関数です。

**制限:** データバッファはデバイスメモリでなければなりません。ユーザーが管理・解放する必要があります。

### axclrtEngineSetInputBufferByName / axclrtEngineSetOutputBufferByName

```c
axclError axclrtEngineSetInputBufferByName(axclrtEngineIO io, const char *name, const void *dataBuffer, uint64_t size);
axclError axclrtEngineSetOutputBufferByName(axclrtEngineIO io, const char *name, const void *dataBuffer, uint64_t size);
```

**説明:** IO 名で入力/出力データバッファを設定する関数です。

### axclrtEngineGetInputBufferByIndex / axclrtEngineGetOutputBufferByIndex

```c
axclError axclrtEngineGetInputBufferByIndex(axclrtEngineIO io, uint32_t index, void **dataBuffer, uint64_t *size);
axclError axclrtEngineGetOutputBufferByIndex(axclrtEngineIO io, uint32_t index, void **dataBuffer, uint64_t *size);
```

**説明:** IO インデックスで入力/出力データバッファを取得する関数です。

### axclrtEngineGetInputBufferByName / axclrtEngineGetOutputBufferByName

```c
axclError axclrtEngineGetInputBufferByName(axclrtEngineIO io, const char *name, void **dataBuffer, uint64_t *size);
axclError axclrtEngineGetOutputBufferByName(axclrtEngineIO io, const char *name, void **dataBuffer, uint64_t *size);
```

**説明:** IO 名で入力/出力データバッファを取得する関数です。

### axclrtEngineSetDynamicBatchSize

```c
axclError axclrtEngineSetDynamicBatchSize(axclrtEngineIO io, uint32_t batchSize);
```

**説明:** ダイナミックバッチシナリオで 1 回に処理する画像数を設定する関数です。

### axclrtEngineCreateContext

```c
axclError axclrtEngineCreateContext(uint64_t modelId, uint64_t *contextId);
```

**説明:** モデル ID に対してモデル実行環境コンテキストを作成する関数です。

**制限:** 1 つのモデル ID で複数の実行コンテキストを作成できます。各コンテキストは独自の設定とメモリ空間で実行されます。

### axclrtEngineExecute

```c
axclError axclrtEngineExecute(uint64_t modelId, uint64_t contextId, uint32_t group, axclrtEngineIO io);
```

**説明:** モデルの同期推論を実行し、推論結果が返るまで待つ関数です。

**パラメータ:**
- `modelId [IN]`: モデル ID
- `contextId [IN]`: モデル推論コンテキスト
- `group [IN]`: モデルシェイプグループインデックス
- `io [IN]`: モデル推論の IO

### axclrtEngineExecuteAsync

```c
axclError axclrtEngineExecuteAsync(uint64_t modelId, uint64_t contextId, uint32_t group, axclrtEngineIO io, axclrtStream stream);
```

**説明:** モデルの非同期推論を実行する関数です。

---

## native

AXCL NATIVE モジュールは SYS、VDEC、VENC、IVPS、DMADIM、ENGINE、IVE の各モジュールをサポートしています。

AXCL NATIVE API と AX SDK API のパラメータは完全に一致しており、違いは関数名のプレフィックスが AX から AXCL に変更された点のみです:

```c
AX_S32 AXCL_SYS_Init(AX_VOID);
AX_S32 AXCL_SYS_Deinit(AX_VOID);

/* CMM API */
AX_S32 AXCL_SYS_MemAlloc(AX_U64 *phyaddr, AX_VOID **pviraddr, AX_U32 size, AX_U32 align, const AX_S8 *token);
AX_S32 AXCL_SYS_MemAllocCached(AX_U64 *phyaddr, AX_VOID **pviraddr, AX_U32 size, AX_U32 align, const AX_S8 *token);
AX_S32 AXCL_SYS_MemFree(AX_U64 phyaddr, AX_VOID *pviraddr);

...

AX_S32 AXCL_VDEC_Init(const AX_VDEC_MOD_ATTR_T *pstModAttr);
AX_S32 AXCL_VDEC_Deinit(AX_VOID);

AX_S32 AXCL_VDEC_ExtractStreamHeaderInfo(const AX_VDEC_STREAM_T *pstStreamBuf, AX_PAYLOAD_TYPE_E enVideoType,
                                         AX_VDEC_BITSTREAM_INFO_T *pstBitStreamInfo);

AX_S32 AXCL_VDEC_CreateGrp(AX_VDEC_GRP VdGrp, const AX_VDEC_GRP_ATTR_T *pstGrpAttr);
AX_S32 AXCL_VDEC_CreateGrpEx(AX_VDEC_GRP *VdGrp, const AX_VDEC_GRP_ATTR_T *pstGrpAttr);
AX_S32 AXCL_VDEC_DestroyGrp(AX_VDEC_GRP VdGrp);

...
```

詳細については、AX SDK API のドキュメント（例：『AX SYS API ドキュメント.docx』、『AX VDEC API ドキュメント.docx』等）を参照してください。

### ダイナミックライブラリ対照表

ダイナミックライブラリの命名は、従来の libax_xxx.so から libaxcl_xxx.so に変更されています:

| モジュール | AX SDK | AXCL NATIVE SDK |
|---|---|---|
| SYS | libax_sys.so | libaxcl_sys.so |
| VDEC | libax_vdec.so | libaxcl_vdec.so |
| VENC | libax_venc.so | libaxcl_venc.so |
| IVPS | libax_ivps.so | libaxcl_ivps.so |
| DMADIM | libax_dmadim.so | libaxcl_dmadim.so |
| ENGINE | libax_engine.so | libaxcl_engine.so |
| IVE | libax_ive.so | libaxcl_ive.so |

### 未サポート API 一覧

一部の AX SDK API は AXCL NATIVE ではサポートされていません:

| モジュール | AXCL NATIVE API |
|---|---|
| SYS | AXCL_SYS_EnableTimestamp, AXCL_SYS_Sleep, AXCL_SYS_WakeLock, AXCL_SYS_WakeUnlock, AXCL_SYS_RegisterEventCb, AXCL_SYS_UnregisterEventCb |
| VENC | AXCL_VENC_GetFd, AXCL_VENC_JpegGetThumbnail |
| IVPS | AXCL_IVPS_GetChnFd, AXCL_IVPS_CloseAllFd |
| DMADIM | AXCL_DMADIM_Cfg（コールバック関数の設定は非対応。AX_DMADIM_MSG_T.pfnCallBack） |
| IVE | AXCL_IVE_NPU_CreateMatMulHandle, AX_IVE_NPU_DestroyMatMulHandle, AX_IVE_NPU_MatMul |

---

## PPL

### アーキテクチャ

`libaxcl_ppl.so` は高度に統合されたモジュールで、典型的なパイプライン（PPL）を実装しています。アーキテクチャは以下の通りです:

```
| ----------------------------- |
| アプリケーション              |
| ----------------------------- |
| libaxcl_ppl.so                |
| ----------------------------- |
| libaxcl_lite.so               |
| ----------------------------- |
| axcl SDK                      |
| ----------------------------- |
| PCIe ドライバ                 |
| ----------------------------- |
```

> **注意:** `libaxcl_lite.so` と `libaxcl_ppl.so` はオープンソースライブラリで、それぞれ `axcl/sample/axclit` と `axcl/sample/ppl` ディレクトリに格納されています。

### transcode

トランスコード PPL のサンプルコードは `axcl/sample/ppl/transcode` パスの `axcl_transcode_sample` を参照してください。

### API

#### axcl_ppl_init

axcl ランタイムシステムと PPL を初期化します。

```c
axclError axcl_ppl_init(const axcl_ppl_init_param* param);
```

```c
typedef enum {
    AXCL_LITE_NONE = 0,
    AXCL_LITE_VDEC = (1 << 0),
    AXCL_LITE_VENC = (1 << 1),
    AXCL_LITE_IVPS = (1 << 2),
    AXCL_LITE_JDEC = (1 << 3),
    AXCL_LITE_JENC = (1 << 4),
    AXCL_LITE_DEFAULT = (AXCL_LITE_VDEC | AXCL_LITE_VENC | AXCL_LITE_IVPS),
    AXCL_LITE_MODULE_BUTT
} AXCLITE_MODULE_E;

typedef struct {
    const char *json; /* axcl.json path */
    AX_S32 device;
    AX_U32 modules;
    AX_U32 max_vdec_grp;
    AX_U32 max_venc_thd;
} axcl_ppl_init_param;
```

| パラメータ | 説明 | 入力/出力 |
|---|---|---|
| json | `axclInit` API に渡す JSON ファイルパス | 入力 |
| device | デバイス ID | 入力 |
| modules | PPL に応じた AXCLITE_MODULE_E ビットマスク | 入力 |
| max_vdec_grp | 全プロセスの VDEC グループ総数 | 入力 |
| max_venc_thrd | 各プロセスの最大 VENC スレッド数 | 入力 |

> **重要:**
> - AXCL_PPL_TRANSCODE の場合: modules = AXCL_LITE_DEFAULT
> - `max_vdec_grp` は全プロセスの VDEC グループ総数以上に設定してください（例: 16 プロセスで各 1 デコーダの場合、最低 16）。
> - `max_venc_thrd` は通常、各プロセスの VENC チャンネル総数と同じ値に設定します。

**戻り値:** 成功時 0 (AXCL_SUCC)、それ以外は失敗。

#### axcl_ppl_deinit

axcl ランタイムシステムと PPL の非初期化を行います。

```c
axclError axcl_ppl_deinit();
```

#### axcl_ppl_create

PPL を作成します。

```c
axclError axcl_ppl_create(axcl_ppl* ppl, const axcl_ppl_param* param);
```

```c
typedef enum {
    AXCL_PPL_TRANSCODE = 0, /* VDEC -> IVPS ->VENC */
    AXCL_PPL_BUTT
} axcl_ppl_type;

typedef struct {
    axcl_ppl_type ppl;
    void *param;
} axcl_ppl_param;
```

AXCL_PPL_TRANSCODE のパラメータは以下の通りです:

```c
typedef AX_VENC_STREAM_T axcl_ppl_encoded_stream;
typedef void (*axcl_ppl_encoded_stream_callback_func)(axcl_ppl ppl, const axcl_ppl_encoded_stream *stream, AX_U64 userdata);

typedef struct {
    axcl_ppl_transcode_vdec_attr vdec;
    axcl_ppl_transcode_venc_attr venc;
    axcl_ppl_encoded_stream_callback_func cb;
    AX_U64 userdata;
} axcl_ppl_transcode_param;
```

| パラメータ | 説明 | 入力/出力 |
|---|---|---|
| vdec | デコーダ属性 | 入力 |
| venc | エンコーダ属性 | 入力 |
| cb | エンコード済み NALU フレームデータを受信するコールバック関数 | 入力 |
| userdata | axcl_ppl_encoded_stream_callback_func に透過されるユーザーデータ | 入力 |

> **注意:** `axcl_ppl_encoded_stream_callback_func` 内で高遅延の処理を行わないようにしてください。

**VDEC 属性:**

```c
typedef struct {
    AX_PAYLOAD_TYPE_E payload;
    AX_U32 width;
    AX_U32 height;
    AX_VDEC_OUTPUT_ORDER_E output_order;
    AX_VDEC_DISPLAY_MODE_E display_mode;
} axcl_ppl_transcode_vdec_attr;
```

| パラメータ | 説明 |
|---|---|
| payload | PT_H264 / PT_H265 |
| width | 入力ストリームの最大幅 |
| height | 入力ストリームの最大高さ |
| output_order | AX_VDEC_OUTPUT_ORDER_DISP / AX_VDEC_OUTPUT_ORDER_DEC |
| display_mode | AX_VDEC_DISPLAY_MODE_PREVIEW / AX_VDEC_DISPLAY_MODE_PLAYBACK |

> **注意:**
> - `output_order`: デコード順序と表示順序が同一の場合（例: IP ストリーム）、メモリ節約のため AX_VDEC_OUTPUT_ORDER_DEC を推奨します。異なる場合（例: IPB ストリーム）は AX_VDEC_OUTPUT_ORDER_DISP に設定してください。
> - `display_mode`: PREVIEW はフレームドロップ可（RTSP ストリーム等）、PLAYBACK はフレームドロップ不可（ローカルファイル等）。

**VENC 属性:**

```c
typedef struct {
    AX_PAYLOAD_TYPE_E payload;
    AX_U32 width;
    AX_U32 height;
    AX_VENC_PROFILE_E profile;
    AX_VENC_LEVEL_E level;
    AX_VENC_TIER_E tile;
    AX_VENC_RC_ATTR_T rc;
    AX_VENC_GOP_ATTR_T gop;
} axcl_ppl_transcode_venc_attr;
```

| パラメータ | 説明 |
|---|---|
| payload | PT_H264 / PT_H265 |
| width | エンコード NALU フレームの出力幅 |
| height | エンコード NALU フレームの出力高さ |
| profile | H264 / H265 プロファイル |
| level | H264 / H265 レベル |
| tile | タイリング |
| rc | レート制御設定 |
| gop | GOP 設定 |

#### axcl_ppl_destroy

PPL を破棄します。

```c
axclError axcl_ppl_destroy(axcl_ppl ppl);
```

#### axcl_ppl_start

PPL を開始します。

```c
axclError axcl_ppl_start(axcl_ppl ppl);
```

#### axcl_ppl_stop

PPL を停止します。

```c
axclError axcl_ppl_stop(axcl_ppl ppl);
```

#### axcl_ppl_send_stream

nalu フレームを PPL に送信します。

```c
axclError axcl_ppl_send_stream(axcl_ppl ppl, const axcl_ppl_input_stream* stream, AX_S32 timeout);
```

```c
typedef struct {
    AX_U8 *nalu;
    AX_U32 nalu_len;
    AX_U64 pts;
    AX_U64 userdata;
} axcl_ppl_input_stream;
```

| パラメータ | 説明 | 入力/出力 |
|---|---|---|
| nalu | nalu フレームデータへのポインタ | 入力 |
| nalu_len | nalu フレームデータのバイト数 | 入力 |
| pts | nalu フレームのタイムスタンプ | 入力 |
| userdata | `axcl_ppl_encoded_stream.stPack.u64UserData` に透過されるユーザーデータ | 入力 |

#### axcl_ppl_get_attr

PPL の属性を取得します。

```c
axclError axcl_ppl_get_attr(axcl_ppl ppl, const char* name, void* attr);
```

利用可能な属性一覧:

```
/**
 *            name                                     attr type        default
 *  axcl.ppl.id                             [R  ]       int32_t                            increment +1 for each axcl_ppl_create
 *
 *  axcl.ppl.transcode.vdec.grp             [R  ]       int32_t                            allocated by ax_vdec.ko
 *  axcl.ppl.transcode.ivps.grp             [R  ]       int32_t                            allocated by ax_ivps.ko
 *  axcl.ppl.transcode.venc.chn             [R  ]       int32_t                            allocated by ax_venc.ko
 *
 *  the following attributes take effect BEFORE the axcl_ppl_create function is called:
 *  axcl.ppl.transcode.vdec.blk.cnt         [R/W]       uint32_t          8                depend on stream DPB size and decode mode
 *  axcl.ppl.transcode.vdec.out.depth       [R/W]       uint32_t          4                out fifo depth
 *  axcl.ppl.transcode.ivps.in.depth        [R/W]       uint32_t          4                in fifo depth
 *  axcl.ppl.transcode.ivps.out.depth       [R  ]       uint32_t          0                out fifo depth
 *  axcl.ppl.transcode.ivps.blk.cnt         [R/W]       uint32_t          4
 *  axcl.ppl.transcode.ivps.engine          [R/W]       uint32_t   AX_IVPS_ENGINE_VPP      AX_IVPS_ENGINE_VPP|AX_IVPS_ENGINE_VGP|AX_IVPS_ENGINE_TDP
 *  axcl.ppl.transcode.venc.in.depth        [R/W]       uint32_t          4                in fifo depth
 *  axcl.ppl.transcode.venc.out.depth       [R/W]       uint32_t          4                out fifo depth
 */
```

> **重要:** `axcl.ppl.transcode.vdec.blk.cnt` はブロック数で、入力ストリームの DPB サイズに依存します。dbp + 1 に設定することを推奨します。

#### axcl_ppl_set_attr

PPL の属性を設定します。

```c
axclError axcl_ppl_set_attr(axcl_ppl ppl, const char* name, const void* attr);
```

---

## エラーコード

### 定義

```c
typedef int32_t axclError;

typedef enum {
    AXCL_SUCC                   = 0x00,
    AXCL_FAIL                   = 0x01,
    AXCL_ERR_UNKNOWN            = AXCL_FAIL,
    AXCL_ERR_NULL_POINTER       = 0x02,
    AXCL_ERR_ILLEGAL_PARAM      = 0x03,
    AXCL_ERR_UNSUPPORT          = 0x04,
    AXCL_ERR_TIMEOUT            = 0x05,
    AXCL_ERR_BUSY               = 0x06,
    AXCL_ERR_NO_MEMORY          = 0x07,
    AXCL_ERR_ENCODE             = 0x08,
    AXCL_ERR_DECODE             = 0x09,
    AXCL_ERR_UNEXPECT_RESPONSE  = 0x0A,
    AXCL_ERR_OPEN               = 0x0B,
    AXCL_ERR_EXECUTE_FAIL       = 0x0C,

    AXCL_ERR_BUTT               = 0x7F
} AXCL_ERROR_E;

#define AX_ID_AXCL           (0x30)

/* module */
#define AXCL_RUNTIME         (0x00)
#define AXCL_NATIVE          (0x01)
#define AXCL_LITE            (0x02)

/* runtime sub module */
#define AXCL_RUNTIME_DEVICE  (0x01)
#define AXCL_RUNTIME_CONTEXT (0x02)
#define AXCL_RUNTIME_STREAM  (0x03)
#define AXCL_RUNTIME_TASK    (0x04)
#define AXCL_RUNTIME_MEMORY  (0x05)
#define AXCL_RUNTIME_CONFIG  (0x06)
#define AXCL_RUNTIME_ENGINE  (0x07)
#define AXCL_RUNTIME_SYSTEM  (0x08)

/**
 * |---------------------------------------------------------|
 * | |   MODULE    |  AX_ID_AXCL | SUB_MODULE  |    ERR_ID   |
 * |1|--- 7bits ---|--- 8bits ---|--- 8bits ---|--- 8bits ---|
 **/
#define AXCL_DEF_ERR(module, sub, errid) \
    ((axclError)((0x80000000L) | (((module) & 0x7F) << 24) | ((AX_ID_AXCL) << 16 ) | ((sub) << 8) | (errid)))

#define AXCL_DEF_RUNTIME_ERR(sub, errid)    AXCL_DEF_ERR(AXCL_RUNTIME, (sub), (errid))
#define AXCL_DEF_NATIVE_ERR(sub,  errid)    AXCL_DEF_ERR(AXCL_NATIVE,  (sub), (errid))
#define AXCL_DEF_LITE_ERR(sub,    errid)    AXCL_DEF_ERR(AXCL_LITE,    (sub), (errid))
```

> **注意:** エラーコードは AXCL ランタイムライブラリと AX NATIVE SDK の 2 種類に分かれます。`axclError` の第 3 バイトで区別できます。第 3 バイトが AX_ID_AXCL (0x30) の場合は AXCL ランタイムライブラリのエラーコード、それ以外はデバイスの AX NATIVE SDK モジュールから透過されたエラーコードです。

### エラーコードの解析

[AXCL エラーコード解析ツール](https://axcl-docs.readthedocs.io/)でオンラインでエラーコードを解析できます。

- AXCL ランタイムライブラリのエラーコード: `axcl_rt_xxx.h` ヘッダファイルを参照してください。
- デバイスの NATIVE SDK エラーコードは HOST 側に透過されます。詳細は『AX ソフトウェアエラーコードドキュメント』を参照してください。
