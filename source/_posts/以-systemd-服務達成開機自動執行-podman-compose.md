---
title: 以 systemd 服務達成開機自動執行 podman-compose
categories: Linux 筆記
tags: linux,podman,podman-compose,systemd
date: 2022-08-09 18:18:10
---


一般執行 `podman run`, `podman-compose up` 的時候，都是只有單次的執行，如果系統因故重新開機，便需要手動重新執行容器。如果我們希望在開機時能夠自行執行定義在 compose file 裡的服務，這個時候可以運用撰寫 `systemd` 的服務檔。而 `podman-compose` 目前最新的開發中版本提供了 `systemd` 子命令，更是可以直接幫我們自動產生 podman-compose 服務對應的 systemd 服務檔，省去自行撰寫的功夫。

<!-- more -->

需要先將 podman-compose 更新至目前最新的開發中版本(1.0.4)，才能夠使用 `systemd` 子命令

```bash
pip3 install https://github.com/containers/podman-compose/archive/devel.tar.gz
```

先移動到 compose file 所在目錄，執行以下命令：

```bash
podman-compose systemd --action create-unit
```

會自動依 compose file 所在目錄建立 `podman-compose@<PROJECT>.service`

之後執行以下命令將服務檔註冊至 systemd 並啟用：

```bash
podman-compose -a register
systemctl --user enable podman-compose@<PROJECT>.service
```

之後便可以如下列命令操作 podman-compose 服務

```bash
systemctl --user start podman-compose@<PROJECT>.service
systemctl --user stop podman-compose@<PROJECT>.service
systemctl --user status podman-compose@<PROJECT>.service
```
