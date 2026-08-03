---
title: "01 Daily Journal"
date: 2026-07-25
tags: []
categories: ["Qcom"]
draft: false
---


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
