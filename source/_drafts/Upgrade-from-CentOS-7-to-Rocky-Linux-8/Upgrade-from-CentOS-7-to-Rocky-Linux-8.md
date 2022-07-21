---
title: 從 CentOS 7 升級遷移到 Rocky Linux 8
categories:
tags: 
    - linux
    - centos
    - rocky-linux
---

由於 CentOS 7 即將在 2024 年底進入 End Of Life，而 Red Hat ，
，其中 Rocky Linux 是，因此希望藉由這篇文章整理並分享我嘗試升級的操作過程。

以下操作過程**不保證**系統完整性，請養成操作前備份資料的習慣，並**避免直接在正式營運環境做嘗試**

整個升級過程大致可以切分為以下部分：

1. 從 CentOS Linux 7 升級到 CentOS Linux 8
2. 從 CentOS Linux 8 遷移到 Rocky Linux 8

---

## 從 CentOS Linux 7 升級到 CentOS Linux 8

由於 Rocky Linux 是基於 RHEL 8 而來，這一步的主要目的為將系統更新為 CentOS 8.5，才能順利遷移到 Rocky Linux。

### 更新 CentOS 7

先更新現有軟體包

```bash
sudo yum update -y
sudo reboot
```

安裝 `epel-release` 以安裝第三方擴充資源庫

```bash
sudo yum install -y epel-release
```

安裝升級所需工具

```bash
sudo yum install -y rpmconf yum-utils
```

檢查軟體包或配置是否有衝突，如果有衝突可選擇保留原有版本或以開發者版本覆蓋

```bash
sudo rpmconf -a
```

列出未被依賴的軟體包

```bash
sudo package-cleanup --leaves
sudo package-cleanup --orphans
```

如果有，可以使用以下指令清除

```bash
sudo yum remove -y <package>
```

安裝 DNF，為 YUM 下一代的軟體包管理工具

```bash
sudo yum install -y dnf
```

移除 YUM

```bash
sudo yum remove yum yum-metadata-parser
```

清除 YUM 相關設定檔

```bash
sudo rm -Rf /etc/yum
```

用 DNF 做一次 CentOS 7 系統更新

```bash
sudo dnf upgrade -y
```

### 更新至 CentOS 8

安裝 CentOS 8 資源庫

```bash
sudo dnf install http://vault.centos.org/8.5.2111/BaseOS/x86_64/os/Packages/{centos-linux-repos-8-3.el8.noarch.rpm,centos-linux-release-8.5-1.2111.el8.noarch.rpm,centos-gpg-keys-8-3.el8.noarch.rpm}
```

更新 epel-release

```bash
dnf -y upgrade https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

更新來源資源庫檔

```bash
cd /etc/yum.repos.d
sudo mkdir backups
sudo mv CentOS-* backups

# Create new for CentOS BaseOS repo
sudo tee CentOS-Linux-BaseOS.repo<<EOM
[baseos]
name=CentOS Linux \$releasever - BaseOS
baseurl=http://vault.centos.org/8.5.2111/BaseOS/\$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
EOM

# Create new for CentOS AppStream repo
sudo tee CentOS-Linux-AppStream.repo<<EOM
[appstream]
name=CentOS Linux \$releasever - AppStream
baseurl=http://vault.centos.org/8.5.2111/AppStream/\$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
EOM
```

移除暫存檔

```bash
dnf clean all
```

移除舊版本 kernel 及衝突軟體包

```bash
sudo rpm -e `rpm -q kernel` --nodeps
sudo rpm -e `rpm -q kernel-devel` --nodeps
sudo rpm -e --nodeps sysvinit-tools
```

安裝 CentOS 8 系統更新

```bash
dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
```

* 如果遇到 conflicting requests 錯誤，代表該軟體包可能在 CentOS 8 更改名稱導致更新時未找到相容版本依賴的軟體包，可以先透過 `dnf remove` 指令將衝突的舊軟體包移除後，再重新透過以上指令更新

安裝新版本 kernel

```bash
dnf install -y kernel-core
```

最後以最精簡內容安裝 CentOS 8

```bash
dnf -y groupupdate "Core" "Minimal Install"
```

安裝後重新啟動系統
檢查作業系統及 kernel 版本是否已更新

```bash
$ cat /etc/centos-release
CentOS Linux release 8.5.2111
$ uname -r
4.18.0-348.7.1.el8_5.x86_64
```

---

## 從 CentOS Linux 8 遷移到 Rocky Linux 8

安裝 `wget` 以下載遷移腳本

```bash
sudo dnf -y install wget
wget https://raw.githubusercontent.com/rocky-linux/rocky-tools/main/migrate2rocky/migrate2rocky.sh
```

執行遷移腳本

```bash
sudo chmod a+x migrate2rocky.sh
sudo migrate2rocky.sh -r
```

重新啟動系統，確認更新結果

```shell
# cat /etc/os-release
NAME="Rocky Linux"
VERSION="8.6 (Green Obsidian)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="8.6"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Rocky Linux 8.6 (Green Obsidian)"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:rocky:rocky:8:GA"
HOME_URL="https://rockylinux.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
ROCKY_SUPPORT_PRODUCT="Rocky Linux"
ROCKY_SUPPORT_PRODUCT_VERSION="8"
REDHAT_SUPPORT_PRODUCT="Rocky Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8"
```
