---
tags: notes, build, rpm, gpudirect
---

# Build GPUDirect rpms
- env :  centos7/x86_64/bash
- link : [kmo/notes_gpudirect_build](https://hackmd.io/@kmo/notes_gpudirect_build)
- 說明 :  
此文是紀錄維運的系統編譯筆記

## Mellanox GPUDirect (nv_peer_mem)
- https://github.com/Mellanox/nv_peer_memory
- [Mellanox GPUDirect 官網](https://www.mellanox.com/products/GPUDirect-RDMA)
- [Mellanox GPUDirect 文件](https://www.mellanox.com/related-docs/prod_software/Mellanox_GPUDirect_User_Manual.pdf)
- 加速 GPU 計算非常重要的 kernel module 和 service
- 和 OS kernel、NVIDIA Driver、Mellanox OFED 有相依性，要是上述 3 樣有任一版本升版，
建議重編譯 nvidia-peer-memory 的 rpm
- Build and copy rpm
```bash=
# build
cd /tmp
wget https://github.com/Mellanox/nv_peer_memory/archive/1.0-9.tar.gz
tar -zxvf 1.0-9.tar.gz
cd nv_peer_memory-1.0-9
./build_module.sh
rpmbuild --rebuild /tmp/nvidia_peer_memory-1.0-9.src.rpm

# copy rpm and src.rpm to repo server
rsync -avP /tmp/nvidia_peer_memory-1.0-9.src.rpm $repo_server
rsync -avP /root/rpmbuild/RPMS/x86_64/nvidia_peer_memory-1.0-9.x86_64.rpm $repo_server
```
- Check deps
```bash=
# check with depmod
$ depmod -n |grep -i nv_peer_mem
extra/nv_peer_mem.ko: extra/nvidia.ko.xz extra/mlnx-ofa_kernel/drivers/infiniband/core/ib_core.ko extra/mlnx-ofa_kernel/compat/mlx_compat.ko

# check with modinfo
$ modinfo nv_peer_mem
filename:       /lib/modules/3.10.0-1127.el7.x86_64/extra/nv_peer_mem.ko
version:        1.0-9
license:        Dual BSD/GPL
description:    NVIDIA GPU memory plug-in
author:         Yishai Hadas
retpoline:      Y
rhelversion:    7.8
srcversion:     AA782B9B020AB3A3C722EBD
depends:        ib_core,nvidia
vermagic:       3.10.0-1127.el7.x86_64 SMP mod_unload modversions
```

## GDRcopy
- https://github.com/NVIDIA/gdrcopy
- opensource project
- 主要用於 MPI compiler ，和 [UCX](https://github.com/openucx/ucx) 及 [MVAPICH2-GDR](http://mvapich.cse.ohio-state.edu/downloads/#mv2gdr-23) 相關
- 和 NVIDIA Driver 有相依性，建議 Driver 升版也要一起重編
- Build and copy rpm
```bash=
# build
cd /tmp
git clone https://github.com/NVIDIA/gdrcopy.git
cd gdrcopy
yum install rpm-build make check check-devel subunit subunit-devel
cd packages
CUDA=/usr/local/cuda ./build-rpm-packages.sh

# copy rpm and src.rpm to repo server
rsync -avP /tmp/gdrcopy/packages/*.rpm $repo_server
```

- Check deps
```bash=
# check with depmod
$ depmod -n |grep -i gdrdrv
kernel/drivers/misc/gdrdrv.ko: extra/nvidia.ko.xz

# check with modinfo
$ modinfo gdrdrv
filename:       /lib/modules/3.10.0-1127.el7.x86_64/kernel/drivers/misc/gdrdrv.ko
version:        2.0
description:    GDRCopy kernel-mode driver
license:        MIT
author:         drossetti@nvidia.com
retpoline:      Y
rhelversion:    7.8
srcversion:     FD7B2331948367151CC6C69
depends:        nv-p2p-dummy
vermagic:       3.10.0-1127.el7.x86_64 SMP mod_unload modversions
parm:           dbg_enabled:enable debug tracing (int)
parm:           info_enabled:enable info tracing (int)
```

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
