---
tags: notes, rclone
---

# rclone installation guide
- environment :  linux/x86_64/bash  
- **can use sudo**  -> [go to Superuser](#Superuser)
- **can't use sudo** -> [go to Normal user](#Normal-user)
- share link :  
https://hackmd.io/@kmo/notes_rclone_install  
- question/suggestion :  
If any question or suggestion,  
welcome to comment on hackmd or open [github issue](https://github.com/likueimo/notes/issues).  
- 中文說明 :  
rclone 安裝教學，適用大部分 Linux 系統  
有 sudo 權限用戶，可參考 [Superuser](#Superuser) 部分  
無 sudo 權限用戶，可參考 [Normal user](#Normal-user) 部分  
這篇英文夠簡單就不寫中文版惹 XD

## Superuser
- superuser privilege (you can use sudo command)
- install rclone by deb/rpm

### Ubuntu/Debian  
- install 
```bash=
# download rclone deb
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.deb

# install deb
sudo apt install ./rclone-current-linux-amd64.deb

# clean deb
rm rclone-current-linux-amd64.deb
```
- uninstall 
```bash=
# uninstall rclone
sudo apt remove rclone
```

### RHEL/CentOS
- intall

```bash=
# install rpm
sudo yum install https://downloads.rclone.org/rclone-current-linux-amd64.rpm
```
- uninstall
```bash=
# uninstall rclone
sudo yum remove rclone
```

## Normal user 
- no superuser privilege (you can not use sudo command)
- install rclone to your home directory


### required command
- unzip 
> If there is no unzip command,  
> you can ask system admin to install unzip.


### install

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

# add rclone path environment
cat << EOF >> ~/.bash_profile
PATH=~/rclone/bin:\$PATH #_rclone_path_#
EOF

# clean rclone zipfile and unzip directory
rm rclone-current-linux-amd64.zip
rm -r rclone-*-linux-amd64
```
- logout (Ctrl+D) and login for initializing ~/.bash_profile

### check install success

```bash=
rclone help
man rclone
```

### uninstall

```bash=
# remove rclone
rm ~/rclone/bin/rclone
rm ~/rclone/man/man1/rclone.1

# remove directory if empty
find ~/rclone -type d -empty -delete

# remove rclone_path in profile
sed -i '/#_rclone_path_#/d' ~/.bash_profile
```

# reference
- https://rclone.org/downloads  
- https://rclone.org/install.sh  
- https://rclone.org/install  

---
[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa] This work is licensed under a [CC BY-SA 4.0][cc-by-sa]  

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png  
