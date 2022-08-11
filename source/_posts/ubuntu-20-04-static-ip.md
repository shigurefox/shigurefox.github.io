---
title: Ubuntu 20.04 設定固定 IP
categories: Linux 筆記
tags:
  - linux
  - ubuntu
date: 2022-08-11 12:00:32
---


本文記錄建立 Ubuntu VM 後手動設定固定 IP 的方式。

<!-- more -->

## 關閉 cloud-init

如果 Ubuntu 環境是使用 cloud-init 建立的雲端 VM，需先關閉 cloud-init。  
在 `/etc/cloud` 或 `/var/lib/cloud` 下建立 `cloud-init.disabled` 檔案。

```bash
sudo touch /etc/cloud/cloud-init.disabled
```

## 修改 ip config

執行 `ip a` 查看目前使用的網路介面名稱，通常是 `ensXXX`。

搜尋 `/etc/netplan` 下的設定檔，用 `ls -l` 查看第一個檔案名稱，若為01－開頭，則可自行建立00-network-config.yaml。

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ensXXX:
      dhcp4: no # 不需要 DHCP 配給 IPv4 位址可加入這行
      addresses: [<你的固定 IP 位址>/<遮罩位數>]
      gateway4: <Gateway IP 位址>
      nameservers: [<DNS IP 位址>]

```

接著輸入以下指令套用變更。

```bash
sudo netplan try
```

這時便可以再執行 `ip a` 確認結果。
