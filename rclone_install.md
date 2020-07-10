---
tags: notes, install, rclone
---

# Rclone installation note
- env :  linux/x86_64/bash
- link : [kmo/notes_rclone_install](https://hackmd.io/@kmo/notes_rclone_install)
- 說明 :  
rclone 安裝筆記，適用大部分 Linux 系統  
rclone 是 go 語言撰寫，無 library 相依性問題  
編譯好後可直接執行  
可直接下載 rclone 作者預編譯好的檔案使用

# Package Manager

- install rclone through package manager

## apt (Ubuntu/Debian)

- install 
```bash=
# download rclone deb
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.deb

# install deb
sudo apt install ./rclone-current-linux-amd64.deb

# clean
rm rclone-current-linux-amd64.deb
```

- uninstall 
```bash=
# uninstall rclone
sudo apt remove rclone
```

## yum (RHEL/CentOS)
- install
```bash=
# install rpm
sudo yum install https://downloads.rclone.org/rclone-current-linux-amd64.rpm
```
- uninstall
```bash=
# uninstall rclone
sudo yum remove rclone
```

## pacman (Arch Linux/Manjaro Linux)
- install rclone through arch linux [community repo](https://www.archlinux.org/packages/community/x86_64/rclone) 
- install
```bash=
sudo pacman -S rclone
```
- uninstall
```bash=
sudo pacman -Rs rclone
```

## conda (Linux/Windows/macOS)
- install rclone through [conda-forge channel](https://anaconda.org/conda-forge/rclone)
- install
```bash=
conda create --name rclone_env --channel conda-forge rclone
```
- use 
```bash=
conda activate rclone_env
```
- uninstall
```bash=
conda remove --name rclone_env --all
```

# Manual Installation
-  required command : 
unzip
- install to ~/rclone

```bash=
# change directory to /tmp
cd /tmp

# download
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip

# unzip
unzip rclone-current-linux-amd64.zip

# install rclone in /home/$USER/rclone/bin
mkdir -p ~/rclone/bin
mv rclone-*-linux-amd64/rclone ~/rclone/bin

# install rclone manpage in /home/$USER/rclone/man
mkdir -p ~/rclone/man/man1
mv rclone-*-linux-amd64/rclone.1 ~/rclone/man/man1

# add rclone path to environment variable
cat << EOF >> ~/.bash_profile
PATH=~/rclone/bin:\$PATH #_rclone_path_#
EOF

# clean
rm rclone-current-linux-amd64.zip
rm -r rclone-*-linux-amd64
```
- logout (Ctrl+D) and login for initializing ~/.bash_profile
- check installation successful
```bash=
rclone help
man rclone
```
- uninstall

```bash=
# remove rclone
rm -r ~/rclone

# remove rclone_path in profile
sed -i '/#_rclone_path_#/d' ~/.bash_profile
```

# Support Matrix
- latest pre-compiled rclone [support matrix](https://rclone.org/downloads)

![](https://i.imgur.com/PkbyjKz.png)


# reference
- rclone
  - https://rclone.org/downloads
  - https://rclone.org/install.sh
  - https://rclone.org/install
- conda
  - https://anaconda.org/conda-forge/rclone
  - https://github.com/conda-forge/rclone-feedstock
- arch linux
  - https://wiki.archlinux.org/index.php/Pacman/Rosetta

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
