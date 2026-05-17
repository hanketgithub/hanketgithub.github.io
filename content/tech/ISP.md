---
title: "ISP 與 Encoder 整體感知品質最佳化之研究心得"
date: 2026-01-01
---

# 結合 ISP 增進 Encoder Quality 的方式

ISP (Image Signal Processor) 與 Encoder 並非完全獨立系統。

許多 ISP 處理結果會直接影響：

- Motion Estimation 精度
- Residual Energy
- Skip Mode 使用率
- Bitrate 分布
- Temporal Prediction 穩定性
- 主觀畫質 (Subjective Quality)

因此現代手機 SoC 中，ISP 與 Video Encoder 往往會共享：

- Motion information
- Gyro data
- Scene analysis
- ROI metadata
- Noise estimation
- HDR statistics

目的並非單純「畫面變漂亮」，而是：

> 在有限 bitrate / power 下，提高 perceived quality。

---

# ISP Pipeline

```text
Sensor
  ↓
Black Level Correction
  ↓
Demosaic
  ↓
Denoise
  ↓
Deblur
  ↓
Sharpen
  ↓
HDR Merge
  ↓
Tone Mapping
  ↓
Color Correction
  ↓
Stabilization
  ↓
Encoder
```

---

# Denoise

感光元件亂數會出現雜點。

比如在一片背景中：

```text
100 100 100 100
100 250 100 100
100 100 100 100
```

其中 `250` 很可能是 sensor noise。

高 ISO 時尤其明顯。

---

## 如何判斷是雜點或是細節

### Spatial Denoise

觀察鄰近 pixel。

若：

```text
100 100 100
100 250 100
100 100 100
```

中心值與周圍差異極大，可能是 noise。

---

### Temporal Denoise

前後 frame 比較：

```text
frame 0:
100 250 100

frame 1:
100 100 100

frame 2:
100 100 100
```

若只存在單 frame，高機率是 noise。

若多 frame 穩定存在：

```text
frame 0:
100 250 100

frame 1:
100 248 101

frame 2:
101 251 100
```

則可能是真實細節。

---

## 如何影響 Encoder

Noise 會增加：

- Residual
- Motion Estimation complexity
- False motion vector
- Bitrate consumption

---

### Example

本來靜止背景：

```text
frame 0:
100 100 100

frame 1:
100 100 100
```

最佳模式：

```text
SKIP
MV = 0
Residual = 0
```

但若出現 noise：

```text
frame 1:
100 250 100
```

Encoder 可能：

- 認為此 block 發生變化
- 嘗試搜尋其他 block
- 產生錯誤 MV
- 增加 residual bits

結果：

```text
MV != 0
Residual > 0
```

浪費 bitrate。

---

## Denoise 與 Subjective Quality 的 Trade-off

Denoise 太強：

- 細節流失
- 皮膚像蠟
- 草地變成油畫

Denoise 太弱：

- Encoder bitrate 爆炸
- Block artifacts 增加
- Low light 品質崩潰

因此 ISP 常做：

- Adaptive denoise
- Motion-aware denoise
- ROI-based denoise

例如：

- 人臉保留細節
- 天空強力 denoise

---

# Deblur

Shutter speed 過慢或手震造成：

- Motion Blur
- Rolling Shutter Distortion

---

## Blur 對 Encoder 的問題

Blur 並不是單純「變模糊」。

而是：

> 特徵被破壞。

例如：

```text
原本:
||||||||

Blur 後:
////////
```

Block matching 變困難。

因此：

- MV 不穩定
- Residual 增加
- Prediction accuracy 降低

---

## 如何修復

傳統方法：

- Wiener Filter
- Deconvolution
- Optical model inverse

但效果有限。

現代大多：

- AI Deblur
- CNN-based Restoration
- Temporal Reconstruction

本質上：

> AI 根據 training data 腦補「合理細節」。

---

## Deblur 如何幫助 Encoder

Deblur 後：

- Edge clearer
- Texture more stable
- Motion estimation easier

因此：

- MV 更準
- Residual 更低
- Prediction 更穩

尤其：

- 文字
- 人臉
- 動物毛髮
- 樹葉

差異非常大。

---

# Sharpen

Sharpen 並非增加真實細節。

而是：

> 增強 edge contrast。

例如：

```text
100 101 102 103
```

Sharpen 後：

```text
95 100 110 115
```

邊界更明顯。

---

## 對 Encoder 的影響

優點：

- Subjective quality 提升
- 細節看起來更多

缺點：

- High frequency 增加
- Residual 增加
- 壓縮效率下降

因此：

> Sharpen 太強會讓 bitrate 爆炸。

---

## 現代做法

Adaptive Sharpen：

- Flat area 少 sharpen
- Edge area 強 sharpen

甚至：

- Encode-aware sharpen
- Bitrate-aware sharpen

根據 target bitrate 動態調整 sharpen strength。

---

# HDR

HDR (High Dynamic Range) 目的：

> 保留亮部與暗部細節。

傳統 SDR：

```text
暗部黑掉
亮部爆掉
```

HDR 可同時保留：

- 陰影細節
- 高亮細節

---

## HDR Capture

常見方法：

### Multi Exposure Merge

```text
Short exposure:
保留亮部

Long exposure:
保留暗部
```

再 merge。

---

### Interleaved Sensor Readout

Sensor 不同 pixel / line 使用不同 exposure。

手機常見。

---

# HDR 對 Encoder 的影響

HDR 會增加：

- Dynamic range
- Local contrast
- High frequency detail

因此：

- Encoder complexity 上升
- Residual 增加

尤其：

- 火焰
- 太陽反射
- 夜景燈光

非常難壓。

---

## HDR Metadata

Encoder 常需傳遞：

- MaxCLL
- MaxFALL
- Color Primaries
- Transfer Function
- Tone Mapping Info

讓 decoder 正確顯示 HDR。

---

# Tone Mapping

HDR Display 與 SDR Display 能力不同。

Tone Mapping 目的：

> 將 HDR luminance mapping 到 target display。

---

## Example

HDR：

```text
0 ~ 4000 nit
```

SDR monitor：

```text
0 ~ 300 nit
```

因此必須壓縮亮度範圍。

---

## Local Tone Mapping

不是整張圖同一條 curve。

而是：

- 天空一條 curve
- 人臉一條 curve
- 陰影另一條 curve

類似：

> 每區域不同 gamma。

---

## 對 Encoder 的影響

Tone mapping 可：

- 降低 extreme contrast
- 減少 residual
- 降低 bitrate

但過強會：

- Halo
- Artificial look
- Flicker

---

# Stabilization

假設使用 600mm 錄影：

```text
frame 0 = 0
frame 1 = -5 pixel
frame 2 = +3 pixel
frame 3 = -6 pixel
```

手抖會造成：

- 整張畫面亂動
- Encoder 認為發生 global motion

---

## 如何修復

ISP / Stabilization engine：

- 分析 frame motion
- Gyro integration
- Optical flow estimation

再將畫面反向平移：

```text
-5 -> +5 correction
```

讓主體穩定。

---

## 對 Encoder 的幫助

若不做 stabilization：

Encoder MV：

```text
background:
MV = huge
```

即使背景根本沒動。

結果：

- Motion vector 增大
- Residual 增加
- Bitrate 浪費

---

## Stabilization 本質

它其實是一種：

> Global motion removal。

將：

```text
Camera motion
```

與：

```text
Object motion
```

分離。

---

# ISP 與 Encoder Shared Motion Information

現代 SoC 常共享：

- ISP optical flow
- Gyro motion
- Global motion
- ROI map

給 encoder。

---

## 為何有效

Encoder Motion Estimation 最耗：

```text
Search Range
```

例如：

```text
(-64, +64)
```

若 ISP 已知：

```text
global motion = (+5, -3)
```

Encoder 可直接：

```text
search around (+5, -3)
```

而不是 full search。

---

## 好處

- 降低 power
- 降低 latency
- 提升 encode fps
- 提高 MV accuracy

---

# ROI (Region of Interest)

ISP 可辨識：

- 人臉
- 動物
- 車牌
- 主體

並通知 encoder：

```text
此區重要
```

Encoder 可：

- Lower QP
- Preserve detail

背景則：

- Higher QP
- Aggressive compression

---

# AI 與未來方向

未來 ISP 與 Encoder 越來越模糊。

AI 可直接：

- Denoise
- Deblur
- Super Resolution
- Frame Generation
- Artifact Removal

甚至：

> Encode-aware ISP

與：

> ISP-aware Encoder

共同最佳化。

方向已經逐漸從：

```text
獨立模組
```

變成：

```text
整體感知品質最佳化
```