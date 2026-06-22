---
title: "新機 GitHub SSH Clone + Hugo 環境 Setup 小抄"
date: 2025-04-09
tags: [""]
draft: false
---

## 目標

快速在新機設定 GitHub SSH，  
將 Hugo 專案 clone 到 `~/Projects/` 目錄下。

---

## 1. 確認 SSH 金鑰是否已存在

```bash
ls -al ~/.ssh
```

有看到 id_ed25519 / id_ed25519.pub 就可以跳到 Step 3

如果沒有，執行：

```bash
ssh-keygen -t ed25519 -C hanketgithub@email.com
```

一路按 Enter，不用改檔名或密碼。

## 2. 將公鑰加到 GitHub 帳戶

```bash
cat ~/.ssh/id_ed25519.pub
```

複製那一整行內容 ➔ 到 GitHub：

Settings → SSH and GPG keys → New SSH key ➔ 貼上。

Title 建議：「MacBook 2025」


### 3. 測試 SSH 連線是否成功

```bash
ssh -T git@github.com
```

看到：

```bash
Hi hanketgithub! You've successfully authenticated, but GitHub does not provide shell access.
```

✅ 表示 SSH OK！

### 4. Clone Hugo 專案到 ~/Projects/

```bash
git clone --recurse-submodules git@github.com:hanketgithub/hank-tech.github.io.git
```

✅ 注意：這次一定是用 git@github.com:xxx/xxx.git （SSH格式）


### 5. 進入專案開始 Git 操作

```bash
cd ~/Projects/hank-tech.github.io
```

常用指令：

```bash
# 拉最新
git pull

# 加檔案
git add .

# 提交
git commit -m "你的 commit 訊息"

# 推上 GitHub
git push
```

