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

4. 重啟 container。

```bash
docker start -ai kuma
```


---


PS: Dockerfile 範例
```
# -----------------------------------------------------------------------------
# Dockerfile for a reproducible Ubuntu 22.04 development environment
#
# Purpose:
#   - Provide a clean, predictable build environment
#   - Avoid host OS dependency issues
#   - Intended for interactive development and build, not production runtime
# -----------------------------------------------------------------------------
#
# Build image:
#   $ docker build -t xyz .
#
# List images:
#   $ docker images
#
# Run container (interactive shell):
#   $ docker run -it -v "$HOME:$HOME" xyz
#
# Notes:
#   - $HOME is bind-mounted to preserve user files and SSH config
#   - Container is expected to be used as a dev shell
#

FROM ubuntu:22.04

# Base packages required by PetaLinux / Yocto build environment
RUN apt update
RUN apt install -y vim
RUN apt install -y \
  bc build-essential net-tools xterm autoconf libtool \
  python3 less rsync texinfo zlib1g-dev gcc-multilib \
  libncurses5-dev libtinfo5 libtinfo6 zstd

# Locale setup to avoid encoding-related toolchain issues
RUN apt install -y locales
RUN locale-gen en_US.UTF-8

# Force /bin/sh to point to bash
RUN ["/bin/bash", "-lc", "rm -f /bin/sh && ln -s /bin/bash /bin/sh"]

ENV SHELL=/bin/bash
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# Create non-root user for build consistency
# Avoid building as root to match real-world dev environment
RUN useradd -m hank
RUN echo "hank:advantech" | chpasswd

USER hank
WORKDIR /home/hank

# Default to interactive bash shell
CMD ["bash"]
```