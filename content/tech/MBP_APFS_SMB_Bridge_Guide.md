---
title: "使用 Mac 作為 Virtual File System"
date: 2025-08-10
tags: ["Virtual File System"]
draft: false
---

# 🎯 核心目標
- 用現有 **M3 MacBook Pro** 直接當「APFS → SMB/NFS 跨平台橋接器」  
- 讓 **Windows / Linux** 都能讀寫 APFS 卷宗，而不需在意底層檔案系統相容性  

---

## 💡 為什麼保留 MBP 而不是換 Mac mini
- 有螢幕與鍵盤，隨時可管理、排錯
- 目前設定已可直接跨平台存取  

---

## ⚙ 建議設定方案（雙場景）

### 1. 在 M3 MBP 上設定檔案共享
1. System Settings → General → **Sharing** → 開啟 **File Sharing**  
2. Options → 勾選 **Sharing ... SMB**，並勾選你的使用者帳號（輸入密碼啟用）  
3. 將 **LaCie SSD（APFS）** 加入共享清單，設定需要的帳號為 **Read & Write** 權限  

---

### 2. Windows 連線方法
- **檔案總管**：
  ```
  \\<MBP_IP>
  ```
  範例：
  ```
  \\172.17.6.88
  ```
- 登入時：
  - 使用者：hank（不加 @domain）
  - 密碼：macOS 登入密碼

---

### 3. Linux 連線方法
2. 檔案總管 GUI：
   ```
   smb://172.17.6.88
   ```

---

### 4. 固定 IP 與雙場景切換
- 在 macOS **網路位置 (Locations)** 建立：
  - 公司 NAS → 固定 IP（例：192.168.50.10）
  - 家 NAS → 固定 IP（例：192.168.1.10）
- 到不同地點切換位置即可使用固定 IP 連線

---

### 5. Thunderbolt Networking（可選加速）
1. 用 TB4 線直連 M3 MBP ↔ Windows / Linux  
2. Mac 端 Thunderbolt Bridge 設固定 IP（例：10.0.2.1）  
3. Windows / Linux 端同網段 IP（例：10.0.2.2）  
4. 按上面 SMB 方法連線，速度可達 900–1500MB/s  

---

## 📌 實際效果
- 到公司插網路 → 切換到「公司 NAS」位置 → Windows/Linux 馬上能用  
- 回家切換到「家 NAS」位置 → 家中裝置立即可用  
- 完全不需要考慮 filesystem 相容性  
- 保留 APFS 的快照與安全性功能  
