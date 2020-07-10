---
tags: notes, install, parallel
---

# GNU Parallel installation note
- env :  linux/x86_64/bash
- link : [kmo/notes_gnu_parallel_install](https://hackmd.io/@kmo/notes_gnu_parallel_install)
- 說明 :  
gnu parallel 安裝筆記，適用大部分 Linux 系統  
這邊列舉 rpm 是作者維護的版本，隨時會保持最新  
除此之外，還可透過 [conda-forge](https://anaconda.org/conda-forge/parallel)，由社群持續更新的 gnu parallel  
其他系統，可參考[官網](https://www.gnu.org/software/parallel/)提及 Official packages exist for

# Package Manager

- install gnu parallel through package manager


## yum (RHEL/CentOS)

- create repo file
- RHEL
```bash=
cat << EOF > /etc/yum.repos.d/gnu_parallel.repo
[gnu_parallel]
name=tanges Project (RHEL_7)
type=rpm-md
baseurl=https://download.opensuse.org/repositories/home:/tange/RHEL_7/
gpgcheck=1
gpgkey=https://download.opensuse.org/repositories/home:/tange/RHEL_7/repodata/repomd.xml.key
enabled=1
EOF
```
- CentOS
```bash=
cat << EOF > /etc/yum.repos.d/gnu_parallel.repo
[gnu_parallel]
name=tanges Project (CentOS_7)
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

## conda (Linux/Windows/macOS)
- install gnu parallel through [conda-forge channel](https://anaconda.org/conda-forge/parallel)
- install
```bash=
conda create --name parallel_env --channel conda-forge parallel
```
- use
```bash=
conda activate parallel_env
```

- uninstall
```bash=
conda remove --name parallel_env --all
```

# Manual Installation

- required command : perl, bzip2
- install to ~/parallel
```bash=
# change directory to /tmp
cd /tmp

# download
wget https://ftpmirror.gnu.org/parallel/parallel-latest.tar.bz2

# untar *.tar.bz2
tar jxvf parallel-latest.tar.bz2

# change directory to parallel directory 
cd parallel-*

# configure, make and install
./configure --prefix=/home/$USER/parallel && make && make install

# add gnu parallel path to environment variable
cat << EOF >> ~/.bash_profile
PATH=~/parallel/bin:\$PATH #_gnu_parallel_path_#
EOF

# clean
rm -r /tmp/parallel-*
```
- logout (Ctrl+D) and login for initializing ~/.bash_profile
- check installation successful
```bash=
parallel --help
man parallel
```

- uninstall
```bash=
# remove gnu parallel
rm -r ~/parallel

# remove rclone_path in profile
sed -i '/#_gnu_parallel_path_#/d' ~/.bash_profile
```


# reference
- https://www.gnu.org/software/parallel
- https://build.opensuse.org/repositories/home:tange/parallel
- README in parallel-latest.tar.bz2

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png