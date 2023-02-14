---
title: GitLab pg_dump server version mixmatch 問題解決方法
categories: DevOps 筆記
tags: linux,postgresql,docker,gitlab
date: 2023-02-14 17:11:09
---


在 gitlab.rb 中，可以透過調整 db 相關參數，讓 GitLab 使用外部的 PostgreSQL 資料庫伺服器儲存資料。不過，日後需要對 GitLab 製作備份時，由於 `pg_dump` 會要求使用的版本與伺服器相同，可能會因為 postgres 版本不同而遇到 server version mixmatch 的錯誤。

首先可以在以下網址查看 GitLab 各版本附帶的 postgres 版本：

[https://docs.gitlab.com/ee/administration/package_information/postgresql_versions.html](https://docs.gitlab.com/ee/administration/package_information/postgresql_versions.html)

其中目前最新 GitLab 版本 (15.8) 只有附 12.12 和 13.8 版本的 postgres 程式。也就是說，如果外部 postgres 資料庫的版本在 14 以上，進行備份時便會遇到版本不合的問題。解決方法之一是透過在 GitLab 機器上安裝與 postgres 伺服器版本相同的客戶端。

以下以 GitLab 安裝在 Ubuntu 20.04 機器上，欲連接 postgres 版本 14 為例。如果 Ubuntu 版本不是 20.04 (Focal)，須將 `focal` 改為您使用版本的對應代號。

```bash
sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt focal-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -

apt install postgresql-client-14
```

之後在 gitlab-backup 所在目錄 `/opt/gitlab/bin` 建立目標版本 pg_dump 和 psql 的連結，gitlab-backup 便會優先使用我們放在該目錄的連結。

```bash
ln -s /usr/lib/postgresql/14/bin/pg_dump /usr/lib/postgresql/14/bin/psql /opt/gitlab/bin/
```

之後重新跑指令建立 GitLab 備份，應該就能正常執行。

```bash
gitlab-backup create
```

而如果 GitLab 是運行在 Docker 容器上面，幸好由於 GitLab 的 docker 容器是基於 Ubuntu 製作，我們一樣可以在容器內執行上述指令來手動安裝不同版本的 postgres 客戶端。不過由於在容器上的操作不會影響到原本的 image，當容器重啟或是來源 docker image 更新，我們便需要重新執行以上的指令做變更，或是另行撰寫 Dockerfile 自動重新 build image。
