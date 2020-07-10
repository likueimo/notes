---
tags: notes, install, driver, nvidia
---

# NVIDIA Tesla driver installation note

- env :  centos7/x86_64/bash
- link : [kmo/notes_nvidia_tesla_driver_install](https://hackmd.io/@kmo/notes_nvidia_tesla_driver_install)
- 說明 :  
  此文是紀錄維運的系統安裝筆記

## Check GPU type
- 確認機器是 Tesla 型號
```bash=
lshw -numeric -C display | grep -i Tesla
```

## Check release note
- [Tesla Driver Release Notes](https://docs.nvidia.com/datacenter/tesla/index.html)
- 建議查看 `Fixed Issues` 及 `Known Issues`

## Choose Tesla driver version
- [NVIDIA Tesla Driver Software Lifecycle Policy](https://docs.nvidia.com/datacenter/tesla/tesla-software-lifecycle/index.html)
- 建議選擇 Long Term Service Branch (LTS)
![Imgur](https://i.imgur.com/h40wA2O.png)

## Download Tesla driver
- https://www.nvidia.com/Download/Find.aspx
- 語言選 Chinse(Traditional)，下載的 link 會是 tw 網域
![Imgur](https://i.imgur.com/vnmzZGx.png)

## Create repo
- 建議把 cluster 的 yum 導向內網的 repo server，  
  故把下載好的 rpm 解壓縮，放置 repo server 的指定位置
- local repo server 
```bash=
# download
curl -O http://tw.download.nvidia.com/tesla/450.51.05/nvidia-driver-local-repo-rhel7-450.51.05-1.0-1.x86_64.rpm

# untar rpm
rpm2cpio nvidia-driver-local-repo-rhel7-450.51.05-1.0-1.x86_64.rpm |cpio -idv

# move directory to specify name
mv ./var/nvidia-driver-local-repo-rhel7-450.51.05 nv_rpms_450.51.05
```
- cluster node
- /etc/yum.repos.d/nvidia-local.repo
```bash=
[nv-450.51.05]
name=yum repository for nv_rpms_450.51.05
baseurl=http://your_repo_server_ip_and_path/nv_rpms_450.51.05
enabled=1
gpgcheck=0
```

## Install Tesla driver
- install precompiled driver
```bash=
yum install cuda-drivers
```
- start service
```bash=
systemctl start nvidia-persistenced
```

## Install DCGM
- https://developer.nvidia.com/dcgm
- 可以在安裝完 driver 後，檢查 Tesla GPU 健康狀況以及 GPU 壓力測是
- start service
```bash=
systemctl start dcgm
```
- 壓力測試
```bash=
dcgmi diag -r 3
```

## Check rpm scripts
- 檢查安裝的 rpm 的 pre/post scripts，掌握裝 nvidia driver 幫系統加了什麼
- add nvidia-persistenced user (nvidia-persistenced-latest*.rpm)
- add kernel cmd (nvidia-driver-latest-*.rpm)
```bash=
# check rpm pre/post scripts
rpm -qp --scripts nvidia-persistenced-latest-450.51.05-1.el7.x86_64.rpm
rpm -qp --scripts nvidia-driver-latest-450.51.05-1.el7.x86_64.rpm
```

## reference
- http://developer.download.nvidia.com/compute/cuda/preview/repos/rhel7/x86_64/README.html  
![Imgur](https://i.imgur.com/ke7ij1x.png)

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png