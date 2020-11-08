---
tags: notes, install, driver, nvidia
---

# NVIDIA Tesla Driver installation note

- env :  CentOS 7 
- arch : x86_64
- link : [kmo/notes_nvidia_tesla_driver_install](https://hackmd.io/@kmo/notes_nvidia_tesla_driver_install)
- 說明 :  此文是針對 Tesla 的 driver 安裝筆記

## 前言

Tesla 系列的 GPU 通常用於 
Data Center 和 HPC 等大型機群的情境。  
NVIDIA 針對 Tesla 的 Driver 特別拉出一份[文件](https://docs.nvidia.com/datacenter/tesla/index.html)說明，並有不同 [support 週期](https://docs.nvidia.com/datacenter/tesla/tesla-software-lifecycle/index.html)。  
除此之外，建議 production 環境是從 NVIDIA Driver 頁面下載安裝，而非在 CUDA 安裝包裡面順道安裝 NVIDIA Driver。

> 可以在 [CUDA release note](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html) 搜尋 `Tesla`，有上述最後一段的說明
![](https://i.imgur.com/ANzZM6d.png)



## Check GPU type
- 確認 GPU 是 Tesla 型號
```bash=
lshw -numeric -C display | grep -i Tesla
```

## Check release note
- [Tesla Driver Release Notes](https://docs.nvidia.com/datacenter/tesla/index.html)
- 建議查看 `Fixed Issues` 及 `Known Issues`

## Choose Tesla Driver version
- [NVIDIA Tesla Driver Software Lifecycle Policy](https://docs.nvidia.com/datacenter/tesla/tesla-software-lifecycle/index.html)
- 建議選擇 Long Term Service Branch (LTS)  
目前建議選擇 `R450` 系列。 [參考 Table ](https://docs.nvidia.com/datacenter/tesla/cuda-drivers-support/index.html)
![](https://i.imgur.com/2pWOGeg.png)

## Download Tesla Driver
- https://www.nvidia.com/Download/Find.aspx
- 語言選 Chinse(Traditional)，下載的 link 會是 tw 網域
![](https://i.imgur.com/DF8Eztx.png)


## Create local repository
當維運上百上千台主機，建議有 local repository，並把機群的 yum 指向 local repo，以免因操作失誤而不預期升級 driver。  

- local repo server 
```bash=
# download
curl -O https://tw.download.nvidia.com/tesla/450.80.02/nvidia-driver-local-repo-rhel7-450.80.02-1.0-1.x86_64.rpm

# extract rpm
rpm2cpio nvidia-driver-local-repo-rhel7-450.80.02-1.0-1.x86_64.rpm |cpio -idv

# move directory to specify path
mv ./var/nvidia-driver-local-repo-rhel7-450.80.02 nv_rpms_450.80.02
```
- 機群 (cluster node) 的 yum 設定 (/etc/yum.repos.d/nvidia-local.repo)
```bash=
[nv-450.80.02]
name=yum repository for nv_rpms_450.80.02
baseurl=http://your_repo_server_ip_and_path/nv_rpms_450.80.02
enabled=1
gpgcheck=0
```

## Install Tesla Driver
- install driver
```bash=
yum clean expire-cache
yum install nvidia-driver-latest-dkms
```

## Install Data Center GPU Manager (DCGM)
- https://developer.nvidia.com/dcgm (建議註冊)
- DCGM 可以檢查 Tesla GPU 健康狀況以及 GPU 壓力測試，建議一併安裝
- 下載後可以一併加入 local repo，如同安裝 driver 方法
```bash=
# download
curl -O https://developer.download.nvidia.com/compute/redist/dcgm/2.0.13/RPMS/x86_64/datacenter-gpu-manager-2.0.13-1-x86_64.rpm

# install 
yum install datacenter-gpu-manager
```

## Enable service and reboot
```bash=
systemctl enable nvidia-persistenced dcgm
reboot
```

## Check rpm scripts
- 檢查安裝的 rpm 的 pre/post scripts，掌握裝 nvidia driver 幫系統加了什麼
- add nvidia-persistenced user (nvidia-persistenced-latest*.rpm)
- add kernel cmd (nvidia-driver-latest-*.rpm)
```bash=
# check rpm pre/post scripts
rpm -qp --scripts nvidia-persistenced-latest-dkms-450.80.02-1.el7.x86_64.rpm
rpm -qp --scripts nvidia-driver-latest-dkms-450.80.02-1.el7.x86_64.rpm
```

# Ansible
NVIDIA 有維護安裝 driver 的 ansible role。
- https://github.com/NVIDIA/ansible-role-nvidia-driver

# 其他相關
- https://github.com/NVIDIA/yum-packaging-precompiled-kmod
- https://github.com/NVIDIA/yum-packaging-nvidia-plugin

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png