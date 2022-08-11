---
title: Docker permission denied 錯誤解決方法
categories: DevOps 筆記
tags: docker
date: 2022-08-11 16:39:53
---


Docker 預設需要透過 sudo 取得 superuser 權限才能夠執行，否則會得到如下錯誤：

> Got permission denied while trying to connect to the Docker daemon socket ...

<!-- more -->

解決方法如下：

確認已有名為 docker 的 user group

```bash
sudo groupadd docker
```

將目標使用者加入 docker user group

```bash
sudo usermod -aG docker ${USER}
```

重新登入或執行以下指令更新 session 以套用新群組設定

```bash
su -s ${USER}
```

接下來便可以嘗試不透過 sudo 直接執行 docker 指令了
