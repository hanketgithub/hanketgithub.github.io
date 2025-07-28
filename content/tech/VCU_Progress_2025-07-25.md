---
title: "VCU 開發進度總結 — 2025-07-25"
date: 2025-07-25
tags: ["VCU"]
draft: false
---

# VCU 開發進度總結（截至 2025-07-25）

---

## 🧩 Callback 機制總覽

在使用 `AL_Encoder_Process()` 將 frame 推入 encoder 後，bitstream 並非主動由 API 回傳，而是透過 **callback 機制**主動推送完成的編碼結果。

---

## ✅ Callback 實作範例

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
    hexprint("XX", &data[section.uOffset], 64);
  }
}
```

---

## 🏗 綁定 callback 到 encoder

```cpp
AL_CB_EndEncoding endEncodingCb = {
  .func = myEndEncoding,
};

AL_Encoder_Create(..., endEncodingCb);
```

- `AL_CB_EndEncoding` 是 Allegro 定義的 callback 包裝結構。
- `.func` 即為綁定的函式指標。

---

## 📦 補充說明

- Bitstream 實際內容透過 `AL_TStreamMetaData` 描述，可能包含多個 section。
- 每個 section 都由 `uOffset`（於 AL_Buffer 資料區內的起始位置）與 `uLength`（長度）組成。
- 若要串接封裝（封為 .h265 / .mp4）或分析 bitstream，可直接由此處複製資料。

---

## 🧷 Attach MetaData：Bitstream Buffer

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

---

## 🧷 Attach MetaData：Source Buffer (YUV Input)

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

✅ **此機制已成功驗證，建議納入 encoding pipeline 的後處理模組。**

---
## ✅ 1. `AL_Encoder_PutStreamBuffer()` 失敗，bitstream buffer 缺少 pMetaData（2025-07-17）

- **錯誤訊息**：
  ```
  [/lib_encode/Com_Encoder.c:164] pMetaData
  lib_rtos.c:39: Rtos_AssertWithMessage: Assertion `false' failed.
  ```

- **原因說明**：
  - `pBsBuf` 為 encoder 輸出用的 bitstream buffer，但 `AL_TBuffer->pMetaData == NULL`。
  - encoder 內部需要 metadata 來記錄 output size、NAL info 等，若為 null 會觸發硬性 assert。

- **目前狀況**：
  - 尚未找到正確建立 `AL_TBuffer` 給 `AL_Encoder_PutStreamBuffer()` 使用的完整做法。
  - 計畫進一步深入 trace `exe_encoder`，確認實際建立與使用的流程。
  - 此項後續補充更新。

---

## ✅ 2. `AL_Encoder_Create()` 失敗原因非參數錯誤，而是 allocator 錯用（2025-07-17）

- **原先假設**：`AL_TEncSettings` 設定有誤，與 `exe_encoder` 對齊後仍失敗。
- **真正原因**：使用了錯誤的 allocator。
  ```cpp
  AL_TAllocator* pAllocator = AL_GetDefaultAllocator(); // ❌ 僅為 malloc()，非 DMA
  ```
- **分析**：
  - 該 allocator 分配的是 host heap memory，硬體無法 DMA 存取。
  - `AL_Encoder_Create()` 在檢查 input/output buffer 是否 DMA buffer 時觸發錯誤: (al5e a0200000.al5e: VCU: unavailable resources or wrong configuration)。

- **正確做法**：
  ```cpp
  AL_TAllocator* pAllocator = AL_DmaAlloc_Create("/dev/allegroIP"); // ✅
  ```
  - 該函式透過 driver 分配 DMA buffer（實體記憶體），並用 `mmap()` 映射到 host，可安全交由 encoder 使用。

---

## ✅ 3. `allegro_min_enc` crash 原因排查完成（2025-07-10）

- **錯誤訊息**：
  ```
  pSubHrdParam->bit_rate_value_minus1[0] <= (UINT32_MAX - 1)
  allegro_min_enc: Rtos_AssertWithMessage: Assertion `false' failed.
  ```

- **推論**：
  - `AL_TEncSettings` 結構極度複雜，缺乏參考值與預設組合。
  - 可能欄位之間衝突，導致 encoder 建立失敗。

- **處理方式**：
  - 暫時跳過 `allegro_min_enc` 測試。
  - 改採已確認穩定的 `exe_encoder` 執行檔進行開發與驗證。
  - 利用 `EncoderSink` 及 print debug 技術分析 `AL_TEncSettings` 正確值。

---

## ✅ 4. 使用 `exe_encoder` 並列印 `AL_TEncSettings`（2025-07-10）

- 成功執行 `exe_encoder`，完成測試 YUV 編碼。
- 修改 encoder 主程式，**完整印出 `AL_TEncSettings` 結構所有欄位**，便於後續程式比對與調參。

---

## ✅ 5. Build 正確版本的 VCU Control SW：`VCU 2025.1`（2025-07-10）

- **關鍵發現**：
  - 雖然 `VCU 2023.1` 可編譯，但 encoder 實際運行會 crash。
  - 必須使用 `VCU 2025.1`，才能穩定進行 encoding。

- **編譯步驟**：

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

- **成果**：
  - 成功產出 `AL_Encoder.exe`，可於 ZCU106 正常運行。
  - 輸出 YUV encode 結果正確，bitrate 符合預期，證實 encoder pipeline 可用。

---