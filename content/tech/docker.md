---
title: "使用 docker 作為 petalinux build"
date: 2025-12-10
tags: ["Docker"]
draft: false
---
1. 準備好 `Dockerfile`，請注意檔名必須是 **Dockerfile（D 大寫）**，這是 Docker 的預設慣例，執行 `docker build .` 時會自動尋找此檔名。

2. 在 `Dockerfile` 所在目錄建立新的 image，使用以下指令進行 build，完成後可用 `docker images` 確認 image 是否成功建立。

```bash
docker build -t xyz .
docker images
```

3. 使用以下指令啟動 container，指定固定名稱 `kuma`，並將 host 的 `$HOME` 目錄掛載到 container 內相同路徑，方便重複進入同一個開發環境且保留所有使用者資料。

```bash
docker run -it -v "$HOME:$HOME" --name kuma xyz
```

`--name kuma` 讓你之後可以直接用  

```bash
docker start -ai kuma
```

重新啟動 container