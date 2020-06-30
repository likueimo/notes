---
tags: notes, installation, parallel
---

# gnu parallel installation guide
Thanks Ole Tange for developing gnu parallel and keeping updating rpm by [website](https://build.opensuse.org/package/show/home:tange/parallel).
- environment :  linux/x86_64/bash
- **can use sudo**  -> [go to Superuser](#Superuser)
- **can't use sudo** -> [go to Normal user](#Normal-user)
- share link :  
https://hackmd.io/@kmo/notes_gnu_parallel_install
- question/suggestion :  
If any question or suggestion,  
welcome to comment on hackmd or open [github issue](https://github.com/likueimo/notes/issues).
- 中文說明 :  
gnu parallel 安裝教學，可適用大部分 Linux 系統  
有 sudo 權限用戶，可參考 [Superuser](#Superuser) 部分  
無 sudo 權限用戶，可參考 [Normal user](#Normal-user) 部分  
大部分 Linux 都有現成 gnu parallel 可安裝，可用套件管理指令安裝(如apt/yum)。  
缺點是提供的可能都 out-of-date 沒更新了。  
這邊列舉 rpm 是作者維護的版本，隨時會保持最新。  
除此之外，還可透過 [conda-forge](https://anaconda.org/conda-forge/parallel) 安裝 gnu parallel。

## Superuser
- superuser privilege (you can use sudo command)
- install gnu parellel by rpm
### RHEL/CentOS

- create repo file

```bash=
cat << EOF | sudo tee -a /etc/yum.repos.d/gnu_parallel.repo
[gnu_parallel]
name=tanges Home Project (CentOS_7)
type=rpm-md
baseurl=https://download.opensuse.org/repositories/home:/tange/CentOS_7/
gpgcheck=1
gpgkey=https://download.opensuse.org/repositories/home:/tange/CentOS_7/repodata/repomd.xml.key
enabled=1
EOF
```


- install
```bash=
sudo yum install parallel
```
- uninstall
```bash=
sudo yum remove parallel
```

### Other Linux distribution

- take a look whether already builded deb/rpm etc.
https://build.opensuse.org/repositories/home:tange/parallel
https://www.gnu.org/software/parallel
- or through [Normal user steps](#Normal-user)

## Normal user 
- no superuser privilege (you can not use sudo command)
- install gnu parallel through conda 

### conda
install gnu parallel through conda-forge channel.
- prepare and install conda first  
https://docs.conda.io/projects/conda/en/latest/user-guide/install
- install
```bash=
conda create --name gnu_parallel --channel conda-forge parallel
```
- use
```bash=
conda activate gnu_parallel
```

- uninstall
```bash=
conda remove --name gnu_parallel --all
```


# reference
- https://www.gnu.org/software/parallel
- https://build.opensuse.org/repositories/home:tange/parallel



---
[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa] This work is licensed under a [CC BY-SA 4.0][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
