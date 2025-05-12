---
title: "SwinIR Experiment Summary — March 31, 2025"
date: 2025-03-31
tags: ["swinir", "super-resolution", "pytorch"]
draft: false
---

🧪 SwinIR Experiment Summary — March 31, 2025

## 1. Environment Setup
- OS: Ubuntu 24.04.2 LTS (noble)
```bash
lsb_release -a
```

- Python: 3.12.3
```bash
python3 --version
```

- GPU: NVIDIA GeForce RTX 4090
  - Driver: 570.124.06
  - CUDA: 12.8
```bash
nvidia-smi
```

- PyTorch: 2.5.1+cu121
  - CUDA Available: True
```python
import torch
print(torch.__version__)
print(torch.cuda.is_available())
```

## 2. Project Setup
- Install Python venv and create environment:
```bash
sudo apt install python3.12-venv
python3 -m venv ~/venvs/swinir
source ~/venvs/swinir/bin/activate
```

- Install PyTorch (CUDA 12.1):
```bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
```

- Clone SwinIR repo and install dependencies:
```bash
git clone https://github.com/JingyunLiang/SwinIR.git
cd SwinIR
pip install numpy opencv-python matplotlib tqdm yacs timm einops
```

## 3. Inference — Set5 Benchmark

Lightweight SR (Scale ×2):
```bash
python main_test_swinir.py --task lightweight_sr --scale 2 --training_patch_size 64 \
  --model_path model_zoo/swinir/002_lightweightSR_DIV2K_s64w8_SwinIR-S_x2.pth \
  --folder_lq testsets/Set5/LR_bicubic/X2 \
  --folder_gt testsets/Set5/HR
```

Results (×2):
```text
- Average PSNR/SSIM (RGB): 35.95 dB / 0.9445
- Average PSNR/SSIM (Y):   38.14 dB / 0.9611
```

Lightweight SR (Scale ×4):

```text
- Average PSNR/SSIM (RGB): 30.52 dB / 0.8679
- Average PSNR/SSIM (Y):   32.44 dB / 0.8976
```

Per-Image PSNR_Y/SSIM_Y (×2):
```text
- bird      → 43.28 dB / 0.9906
- baby      → 38.94 dB / 0.9681
- butterfly → 35.75 dB / 0.9795
- head      → 36.07 dB / 0.8922
- woman     → 36.65 dB / 0.9752
```

## 4. Time Spent

✅ Estimated Duration: 1–2 days  
✅ Actual Time Spent: ~2 hours  

💡 Tools like ChatGPT significantly accelerated setup and debugging.  
This time savings allows greater focus on core research (e.g., VQ and domain-specific knowledge).