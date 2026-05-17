---
title: "VCU 開發進度總結 — 2026-04-01"
date: 2026-04-01
tags: ["VCU"]
draft: false
---

## **編譯步驟**：

  ```bash
  # Setup cross compile environment
  $ source ~/vega3006_zynq/xilinx_sdk_r5_vega3006/sdk/environment-setup-cortexa72-cortexa53-xilinx-linux

  # 進入正確版本的 source tree
  $ cd ~/veg3006_linux_dev/vcu-ctrl-sw-xlnx_rel_v2025.1

  # 編譯
  $ make -j8

  # Check binary format
  $ file bin/AL_Encoder.exe
  bin/AL_Encoder.exe: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (GNU/Linux), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, BuildID[sha1]=60c7aab8c2506fecbb00e0200aa072ca73393cab, for GNU/Linux 3.14.0, with debug_info, not stripped
  ```

---

## 更新概要（2026-05-15）

增加 USB boot

---

### VEGA3006 Golden Image Recovery Guide 

Use the `usb_boot_3006_recovery` folder to recover and update the golden image.



#### 1. Set VEGA3006 to USB Boot Mode

Configure `SW2` as follows.

#### eMMC Boot Mode

```text
L   R
x
    x
    x
x
````

#### USB Boot Mode

```text
L   R
X
    X
    X
    X
```

After setting USB boot mode:

* Connect the USB cable.
* Connect the UART cable.
* Connect both cables to the same Ubuntu machine.

Power on VEGA3006 after all cables are connected.



#### 2. Check USB DFU Device Path

Run the following command on the Ubuntu machine:

```bash
sudo ./dfu-util -l
```

Example output:

```text
Found DFU: [03fd:0050] ver=0100, devnum=3, cfg=1, intf=0, path="5-2", alt=0, name="Xilinx DFU Downloader"
```

Find the USB DFU path from the output.

Example:

```text
path="5-2"
```

Update the `-p` argument in the following scripts:

```text
load_fsbl_uboot.sh
load_kernl_ramfs.sh
```

Example:

Original:

```bash
-p "1-11"
```

Updated:

```bash
-p "5-2"
```



#### 3. Load FSBL and U-Boot

Run:

```bash
sudo ./load_fsbl_uboot.sh
```



#### 4. Start USB DFU Boot from UART Console

Open the PS UART console on the Ubuntu machine.

Run:

```bash
setenv script_size_f 0x20000000
run bootcmd_usb_dfu0
```

If the command does not work:

1. Press `Ctrl+C` to stop autoboot.
2. Run the commands again:

```bash
setenv script_size_f 0x20000000
run bootcmd_usb_dfu0
```



#### 5. Load Kernel and Ramfs

Run:

```bash
sudo ./load_kernl_ramfs.sh
```

After the download completes, return to the UART console and run:

```bash
bootm 0x20000000
```

VEGA3006 will boot into the Linux recovery system.



#### 6. Login to Linux Recovery System

Use the PS UART console to login.

Credentials:

```text
ID: petalinux
Password: adv123456
```



#### 7. Power Off VEGA3006

After the update finishes:

1. Power off VEGA3006.
2. Restore `SW2` to normal eMMC boot mode if required.
3. Power on the system again.

---


### stream_app 裡 VCU 如何接收 input source frame?（2026-04-20）

短答: fpga 並未直接送 input source frame 給 VCU, VCU 該 encode 哪張 frame, 完全靠自己猜出來.

主要的 function 是 `fill_from_mem()`. 當它 return 時, VCU 會自動 encode 下一個 buffer.

換言之, 假如有 4 個 `buf[0] - [3]`, 若 VCU 最後 encode 了 `buf[1]`, 那當 `fill_from_mem()` return, 它便會 encode `buf[2]`.

因此, 核心問題就是, 何時 VCU 可以 encode 下一張? 答案就是當 fpga 完成至少一次寫入, 也就是 frame_write_cnt 增加.

  ```cpp
  while ( write_frame_num <= read_frame_num ) {
    // wait until fpga write new frame
    GetRxVideoFrameTransmitCnt(write_frame_num);
  }
  ```

此 snippet 就是最主要等待 fpga 寫入的 logic. 當它跳出, `fill_from_mem()` 便可 return.


### 格式轉換（2025-08-04）

`CYuvSrcConv::ConvertSrcBuf(AL_TBuffer const *pSrcIn, AL_TBuffer *pSrcOut)`

- 根據 `pSrcOut` 的 FOURCC，選擇對應的轉換 case  
  例如，若輸出為 NV12，將執行：
  ```cpp
  case FOURCC(NV12):
    return convertToNV12(pSrcIn, tFourCCIn, pSrcOut);
  ```

- `convertToNV12(AL_TBuffer const* pSrcIn, TFourCC inFourCC, AL_TBuffer* pSrcOut)`  
  根據輸入格式 `inFourCC`，選擇對應的轉換邏輯，例如：
  ```cpp
  case FOURCC(I420):
    return I420_To_NV12(pSrcIn, pSrcOut);
  ```

- 最終將透過對應函式將輸入格式轉為 NV12。

---


## Send Input Frame Trace（2025-08-01）

`exe_encoder` 主邏輯在 `SafeChannelMain()` 裡：

```cpp
while (...) {
  LayerResources::SendInput(...);
}
```

---

### 🔁 Function Call Trace

```cpp
SafeChannelMain()
└─ LayerResources::SendInput(cfg, enc.get(), traceHooker)
   └─ sendInputFileTo(
        frameReader,
        SrcBufPool,
        SrcYuv.get(),
        cfg,
        FileInfo,
        pSrcConv.get(),
        firstSink,
        iPictCount,
        iReadCount
      )
      └─ GetSrcFrame(
           iReadCount,
           iPictCount,
           frameReader,
           FileInfo,
           SrcBufPool,
           Yuv,
           tChParam,
           cfg,
           pSrcConv
         )
         └─ ReadSourceFrame(SrcBufPool, Yuv, frameReader, tUpdatedDim, pSrcConv)
            └─ ReadSourceFrameBuffer(pBuffer, conversionBuffer, frameReader, tUpdatedDim, pSrcConv)
               ├─ if conversion needed:
               │    ├─ frameReader->ReadFrame(conversionBuffer)
               │    └─ pSrcConv->ConvertSrcBuf(conversionBuffer, pBuffer)
               └─ else:
                    └─ frameReader->ReadFrame(pBuffer)
```

---

### 🧩 Input Components 說明

| 變數名        | 類型 / 來源                            | 備註                                     |
|---------------|----------------------------------------|------------------------------------------|
| `frameReader` | `unique_ptr<FrameReader>`              | Instance variable（UnCompFrameReader）    |
| `SrcBufPool`  | `PixMapBufPool`                        | DMA allocator buffer pool                 |
| `Yuv`         | `AL_TBuffer *`                         | 使用 `SrcYuv.get()`（default allocator） |
| `pSrcConv`    | `IConvSrc *`                           | 若需轉換格式，實體為 `CYuvSrcConv`       |
| `firstSink`   | `IFrameSink *`                         | encoder sink；`enc.get()`                 |
| `iPictCount`  | `int`                                  | frame 計數器                              |
| `iReadCount`  | `int`                                  | read 計數器                               |

---

### 🏗️ 來源建構流程

- `frameReader`  
  → `LayerResources::InitializeFrameReader()`  
  → 類型：`UnCompFrameReader`

- `pSrcConv`  
  → `AllocateSrcConverter()`  
  → 若格式不同，return `make_unique<CYuvSrcConv>`

- `SrcYuv`  
  → `AllocateConversionBuffer(...)`  
  → 僅在需要 format conversion 時配置

---

### 📂 Read 行為細節

- **若無需格式轉換**：

  ```cpp
  frameReader->ReadFrame(pBuffer);
  ```

- **若需格式轉換（例如 I420 → NV12）**：

  ```cpp
  frameReader->ReadFrame(conversionBuffer);
  pSrcConv->ConvertSrcBuf(conversionBuffer, pBuffer);
  ```

---

## Fetch ES Output 機制修正（2025-07-30）

原本的 callback 實作錯誤，導致 encoder 卡住無法繼續輸出 ES。經查證，必須在 callback return 前 **明確呼叫 `AL_Encoder_PutStreamBuffer()`** 歸還 bitstream buffer，encoder 才能繼續使用該 buffer 進行編碼。

### 修正後範例：

```cpp
void myEndEncoding(void *userParam, AL_TBuffer *pStream, AL_TBuffer const *pSrc, int iLayerID)
{
  uint8_t *data = AL_Buffer_GetData(pStream);

  AL_TPictureMetaData *pPicMeta = (AL_TPictureMetaData *) AL_Buffer_GetMetaData(pStream, AL_META_TYPE_PICTURE);
  printf("Picture Type %s %s\n", PictTypeToString(pPicMeta->eType).c_str(), pPicMeta->bSkipped ? "is skipped" : "");

  AL_TStreamMetaData *pStreamMeta = (AL_TStreamMetaData *) AL_Buffer_GetMetaData(pStream, AL_META_TYPE_STREAM);
  for (uint16_t i = 0; i < pStreamMeta->uNumSection; i++) {
    AL_TStreamSection section = pStreamMeta->pSections[i];
    printf("Section %d: size=%d layer=%d, type=%s\n",
           i, section.uLength, iLayerID, SectionFlagToString(section.eFlags).c_str());
  }

  // 必須在 callback 中還原 buffer，否則 encoder 會卡住
  bool bRet = AL_Encoder_PutStreamBuffer(__hEnc__, pStream);
  if (!bRet) {
    printf("AL_Encoder_PutStreamBuffer must always succeed\n");
  }
}
```

### 註解

這種做法令人費解。一般而言若使用 malloc 分配 buffer，應在呼叫 callback 後在呼叫端 free：

```c
uint8_t *buf = malloc(100);
callback(buf);
free(buf);  // 呼叫端負責釋放
```

但 Allegro SDK 的模式卻是：
```cpp
AL_TBuffer *pStream = ...; // 由 encoder 填寫完的 bitstream buffer
callback(pStream);         // callback 內部要歸還 pStream ???
```

這種設計不符合 callback 的一般用法邏輯，屬於設計隱晦的 API 行為，需額外註明。


---

## Fetch ES Output 機制總覽（2025-07-23）

在使用 `AL_Encoder_Process()` 將 frame 推入 encoder 後，ES bitstream 並非主動呼叫 API 回傳，而是透過 **callback 機制**觸發推送完成的編碼結果。

### Callback 實作範例

```cpp
void myEndEncoding(void *userParam, AL_TBuffer *pStream, AL_TBuffer const *pSrc, int iLayerID)
{
  uint8_t *data = AL_Buffer_GetData(pStream);

  // 解析 Picture metadata
  AL_TPictureMetaData *pPicMeta = (AL_TPictureMetaData *) AL_Buffer_GetMetaData(pStream, AL_META_TYPE_PICTURE);
  printf("Picture Type %s %s\n",
         PictTypeToString(pPicMeta->eType).c_str(),
         pPicMeta->bSkipped ? "is skipped" : "");

  // 解析 Stream metadata
  AL_TStreamMetaData *pStreampMeta = (AL_TStreamMetaData *) AL_Buffer_GetMetaData(pStream, AL_META_TYPE_STREAM);
  printf("sections = %d / %d\n", pStreampMeta->uNumSection, pStreampMeta->uMaxNumSection);

  for (uint16_t i = 0; i < pStreampMeta->uNumSection; i++) {
    AL_TStreamSection section = pStreampMeta->pSections[i];
    printf("section %d: size = %d\n", i, section.uLength);
    hexprint("XX", &data[section.uOffset], min(section.uLength, 64));
  }
}
```

### 綁定 callback 到 encoder

```cpp
AL_CB_EndEncoding endEncodingCb = {
  .func = myEndEncoding,
};

AL_Encoder_Create(..., endEncodingCb);
```

- `AL_CB_EndEncoding` 是 Allegro 定義的 callback 包裝結構。
- `.func` 即為綁定的函式指標。

### 補充說明

- Bitstream 實際內容透過 `AL_TStreamMetaData` 描述，可能包含多個 section。
- 每個 section 都由 `uOffset`（於 AL_Buffer 資料區內的起始位置）與 `uLength`（長度）組成。
- 若要串接封裝（封為 .h265 / .mp4）或分析 bitstream，可直接由此處複製資料。

---

## Attach MetaData：Bitstream Buffer（2025-07-23）

```cpp
printf("AL_MAX_SECTION=%d\n", AL_MAX_SECTION);
AL_TStreamMetaData *pStreamMeta = AL_StreamMetaData_Create(AL_MAX_SECTION);

bool isOk = AL_Buffer_AddMetaData(pBsBuf, (AL_TMetaData *) pStreamMeta);
if (!isOk) {
  printf("Failed to attach metadata to bitstream buffer\n");
  return -1;
} else {
  printf("%d ok!\n", __LINE__);
}
```

[解決 issue](#al_encoder_putstreambuffer-failed-bitstream-buffer-needs-pmetadata2025-07-17)

---

## Attach MetaData：Source Buffer (YUV Input)（2025-07-23）

```cpp
AL_TDimension tDim = { settings.tChParam[0].uSrcWidth, settings.tChParam[0].uSrcHeight };
AL_TPlane tYPlane = {
  .iChunkIdx = 0,
  .iOffset = 0,
  .iPitch = tDim.iWidth,
};
AL_TPlane tUVPlane = {
  .iChunkIdx = 0,
  .iOffset = tYPlane.iPitch * tDim.iHeight,
  .iPitch = tYPlane.iPitch
};
AL_TPicFormat tPictFormat = {
  .eChromaMode = AL_CHROMA_4_2_0,
  .eAlphaMode = AL_ALPHA_MODE_DISABLED,
  .uBitDepth = 8,
  .eStorageMode = AL_FB_RASTER,
  .ePlaneMode = AL_PLANE_MODE_SEMIPLANAR,
  .eComponentOrder = AL_COMPONENT_ORDER_YUV,
  .eSamplePackMode = AL_SAMPLE_PACK_MODE_BYTE,
  .bCompressed = false,
  .bMSB = false,
};
TFourCC tFourCC = AL_GetFourCC(tPictFormat);

AL_TMetaData *pPixMapMeta = (AL_TMetaData *) AL_PixMapMetaData_Create(tDim, tYPlane, tUVPlane, tFourCC);

bool isOk = AL_Buffer_AddMetaData(pYuvBuf, pPixMapMeta);
if (!isOk) {
  printf("Failed to attach metadata to source buffer\n");
  return -1;
} else {
  printf("%d ok!\n", __LINE__);
}
```

---

## AL_Encoder_PutStreamBuffer() failed, bitstream buffer needs pMetaData（2025-07-17）

**錯誤訊息**：
```
[/lib_encode/Com_Encoder.c:164] pMetaData
lib_rtos.c:39: Rtos_AssertWithMessage: Assertion `false' failed.
```

**原因說明**：
- `pBsBuf` 為 encoder 輸出用的 bitstream buffer，但 `AL_TBuffer->pMetaData == NULL`。
- encoder 內部需要 metadata 來記錄 output size、NAL info 等，若為 null 會觸發硬性 assert。

**目前狀況**：
- 尚未找到正確建立 `AL_TBuffer` 給 `AL_Encoder_PutStreamBuffer()` 使用的完整做法。
- 計畫進一步深入 trace `exe_encoder`，確認實際建立與使用的流程。
- 此項後續補充更新。

---

## `AL_Encoder_Create()` 失敗原因非參數錯誤，而是 allocator 錯用（2025-07-17）

**原先假設**：`AL_TEncSettings` 設定有誤，與 `exe_encoder` 對齊後仍失敗。
**真正原因**：使用了錯誤的 allocator。
  ```cpp
  AL_TAllocator *pAllocator = AL_GetDefaultAllocator(); // ❌ 僅為 malloc()，非 DMA
  ```
**分析**：
- 該 allocator 分配的是 host heap memory，硬體無法 DMA 存取。
- `AL_Encoder_Create()` 在檢查 input/output buffer 是否 DMA buffer 時觸發錯誤: (al5e a0200000.al5e: VCU: unavailable resources or wrong configuration)。

**正確做法**：
  ```cpp
  AL_TAllocator *pAllocator = AL_DmaAlloc_Create("/dev/allegroIP"); // ✅
  ```
  - 該函式透過 driver 分配 DMA buffer（實體記憶體），並用 `mmap()` 映射到 host，可安全交由 encoder 使用。

---

## `allegro_min_enc` crash 原因排查完成（2025-07-10）

**錯誤訊息**：
```
pSubHrdParam->bit_rate_value_minus1[0] <= (UINT32_MAX - 1)
allegro_min_enc: Rtos_AssertWithMessage: Assertion `false' failed.
```

**推論**：
- `AL_TEncSettings` 結構極度複雜，缺乏參考值與預設組合。
- 可能欄位之間衝突，導致 encoder 建立失敗。

**處理方式**：
- 暫時跳過 `allegro_min_enc` 測試。
- 改採已確認穩定的 `exe_encoder` 執行檔進行開發與驗證。
- 利用 `EncoderSink` 及 print debug 技術分析 `AL_TEncSettings` 正確值。

---

## 使用 `exe_encoder` 並列印 `AL_TEncSettings`（2025-07-10）

- 成功執行 `exe_encoder`，完成測試 YUV 編碼。
- 修改 encoder 主程式，**完整印出 `AL_TEncSettings` 結構所有欄位**，便於後續程式比對與調參。

---

## Build 正確版本的 VCU Control SW：`VCU 2025.1`（2025-07-10）

**關鍵發現**：
雖然 `VCU 2023.1` 可編譯，但 encoder 實際運行會 crash。
必須使用 `VCU 2025.1`，才能穩定進行 encoding。

**編譯步驟**：

  ```bash
  # 設定 cross compile 環境
  source ~/VEGA2200/trunk/xilinx_sdk_r5_vega6000/sdk/environment-setup-cortexa72-cortexa53-xilinx-linux

  # 進入正確版本的 source tree
  cd ~/vcu-ctrl-sw-xlnx_rel_v2025.1

  # 編譯
  make -j8

  # 確認輸出檔案型態
  file bin/AL_Encoder.exe
  ```

**成果**：
- 成功產出 `AL_Encoder.exe`，可於 ZCU106 正常運行。
- 輸出 YUV encode 結果正確，bitrate 符合預期，證實 encoder pipeline 可用。

---