---
tags: notes, installation, rclone
---

# rclone installation guide
- environment :  linux/x86_64/bash
- share link :  
https://hackmd.io/@kmo/notes_rclone_install
- question/suggestion :  
If any question or suggestion,  
welcome to comment on hackmd or open [github issue](https://github.com/likueimo/notes/issues).
- 中文說明 :  
rclone 安裝教學，適用大部分 Linux 系統  
由於 rclone 是 go 語言撰寫，編譯好後基本上沒有相依性問題  
可以參考[手動安裝](##Manual-Installation)


# Package manager

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
- environment :  linux/x86_64/bash
-  required command : 
unzip  
- install

```bash=
# change directory to /tmp
cd /tmp

# download
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip

# unzip
unzip rclone-current-linux-amd64.zip

# install rclone binary in /home/$USER/rclone/bin
mkdir -p ~/rclone/bin
mv rclone-*-linux-amd64/rclone ~/rclone/bin

# install rclone manual in /home/$USER/rclone/man
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
rm ~/rclone/bin/rclone
rm ~/rclone/man/man1/rclone.1

# remove directory if empty
find ~/rclone -type d -empty -delete

# remove rclone_path in profile
sed -i '/#_rclone_path_#/d' ~/.bash_profile
```

# Support matrix
- latest pre-compiled rclone [support matrix](https://rclone.org/downloads)

![](https://i.imgur.com/aUXqtJ0.png)


# reference
- https://rclone.org/downloads
- https://rclone.org/install.sh
- https://rclone.org/install
- https://anaconda.org/conda-forge/rclone
- https://github.com/conda-forge/rclone-feedstock

---
[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa] This work is licensed under a [CC BY-SA 4.0][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
