---
title: "01 Daily Journal"
date: 2026-08-13
tags: []
categories: ["Qcom"]
draft: false
---


# 2026-08-12

## Secure Code I
I don't believe you shall always check input parameter like pointer or buffer size. It quickly makes the code unreadable to have if (ptr == NULL || XYZ) everywhere.

My proposal is to separate task into checking and operating:

```

input parameter -> check -> ok -> parsing -> processing
                     |
                    ng       
                     |
                     v
                  return


/**
 * @brief 內部核心業務 logic (Data flow / Core algorithm)
 * @note  Contract: 呼叫者必須保證 'in' 已通過驗證且合法。
 *        內部不重複做 Defensive Check，專注於執行任務。
 */
static ERR_CODE_E doRealWork(const Input_T *in) {
    // 開發階段 (Debug Build) 用的防線：違約直接抓出呼叫者 Bug，Release 版零開銷
    assert(in != NULL);

    // --------------------------------------------------
    // 乾乾淨淨的主業務邏輯 (極致連續性，無任何雜訊)
    // --------------------------------------------------
    process_payload(in->buffer, in->size);
    update_internal_state(in->flags);

    return STATUS_OK;
}

/**
 * @brief  對外暴露的 API 邊界 (Trust Boundary)
 * @details 負責門禁控制，將防禦、特例、邊界條件與主邏輯完全解耦。
 */
ERR_CODE_E doSomething(Input_T *in) {
    ERR_CODE_E status = STATUS_OK;

    // 1. Gatekeeper: 邊界檢查與特例處理
    //    所有的 input 驗證、版本相容、硬體 Workaround 全部塞在這裡
    status = validate_and_sanitize_input(in);
    if (status != STATUS_OK) {
        // 在門口精準攔截並記錄 Log，不影響核心 logic
        LOG_ERROR("Input validation failed: 0x%X", status);
        return status;
    }

    // 2. Core Business Logic: 暢行無阻的主邏輯
    if (doRealWork(in) != STATUS_OK) { ... }

    return STATUS_OK;
}
```


# 2026-08-11

## Video Preprocessing Subsystem
- Downscaling
- Color Space Conversion
- Rotation: landscape <-> protrait
- Flip
- ...


## vpss_m2m: memory to memory. Basically, store pre-processed image to buffer.

```
Addr[x]              Addr[y]
landscape -> VPSS -> portrait
```


# 2026-08-04 (Day 9)

## Self Intro
- Likability - 很好溝通（Easy to work with）
- Mentor & hq - contact window
- Tasks - docs, code, etc...
- VVC spec
- ES Analyzer

## Chroma QP Offset
Chroma QP = Luma QP + Chroma QP Offset
- Negative value only(in qualcomm platform)
- Small value improves color. Like HDR or Anime.

## Inline Decode Downscalar
- Decoder -> 4k -> downscalar -> 1080p -> frame buffer -> Display
- Reduce memory bandwidth, save power.

## Slice base encoder
Byte Mode: set number of byte per slice.

```
[NALU] [NALU] [NALU] [..] -> [MTU]
```

Alignment: stuffing byte and the end of MTU(?) Ask

## Noise Reduction Impact for motion estimation

```
- - - -     - - - -
- - - -     - - - -
- - - -  →  - - - -
x - x -     o - - -
- o - -     x x - -
```


o is object, x is noise

As you can tell, this can affect motion prediction.

Applying noise reduction:

```
- - - -     - - - -
- - - -     - - - -
- - - -  →  - - - -
- - - -     o - - -
- o - -     - - - -
```

↖ is clearly the candidate

Because NR may cause detail to lose, can we do aggresively NR in a separate path just for motion prediction?


# 2026-08-03 (Day 8)




# 2026-07-31 (Day 5)

- DJI4
  - Propose a method to compare DJI4 and Snapdragon, assume its a blk box.
  - The method is to raise the DJI4 bitrate. So if X is the raw input to DJI4, then we can get X'=X-2 if compression lose is small
  - Then feed X' to Snapdragon:
    X  -> DJI-12Mbps -> dji.hevc
    X' -> Q-12Mbps -> q.hevc



# 2026-07-30 (Day 4)

- DJI4 meeting
  - 討論如何比較 DJI4 encoder vs Snapdragon
  - DJI4 is blk box, can't acces its encoder. Can't feed the encoder raw YUV.



# 2026-07-29 (Day 3)

## What happened

- 部門聚餐
  - John Liu (Director)
  - Owen Feng (TSS Boss)

## Learned

- SD = San Diego
- QI = Qualcomm Israel
- J Building: 瑞光
- D Building: 統一


---

# 2026-07-28 (Day 2)

- AI Code - no copyright protect. So if you use prompt and AI write all coding, then these are not protected.


---


# 2026-07-27 (Day 1)

## What happened

- Orientation
- IT setup
- Camera IQ introduction

## Learned

- APV
- MV-HEVC
- EVA

## Questions

- Why APV?
- Which team owns EVA?

## Meetings

09:00 New Hire
