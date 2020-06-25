---
tags: notes, rclone
---

# rclone installation guide

- share link :  
https://hackmd.io/@kmo/notes_rclone_install  
- question/suggestion :  
If any question or suggestion,  
welcome to comment on hackmd or open github issue.  
https://github.com/likueimo/notes/issues
- ref :  
https://rclone.org/downloads  
https://rclone.org/install.sh  
https://rclone.org/install  

# envirmont
- os : linux x86_64
- shell : bash

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
# tmp for rclone download
mkdir -p ~/tmp_rclone
cd ~/tmp_rclone

# download
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip

# unzip
unzip rclone-current-linux-amd64.zip

# install rclone binary in /home/$USER/bin
mkdir -p ~/bin
mv $(find . -name 'rclone') ~/bin

# install rclone manual in /home/$USER/man (option)
mkdir -p ~/man/man1
mv $(find . -name 'rclone.1') ~/man/man1

# add rclone path environment
cat << EOF >> ~/.bash_profile
PATH=~/bin:\$PATH #_rclone_path_#
EOF

# init profile 
source ~/.bash_profile
# or logout and login

# clean tmp_rclone
cd ~/
rm -rf ~/tmp_rclone
```

### uninstall

```bash=
# remove rclone
rm ~/bin/rclone
rm ~/man/man1/rclone.1

# remove directory if empty
find ~/bin -type d -empty -delete
find ~/man -type d -empty -delete

# remove rclone_path in profile
sed -i '/#_rclone_path_#/d' ~/.bash_profile
```

---
[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa] This work is licensed under a [CC BY-SA 4.0][cc-by-sa]  

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png  
