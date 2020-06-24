---
tags: notes, rclone
---
# rclone installation guides

## env
- os : linux x86_64

## installtion steps

```bash=
# tmp for rclone download
mkdir -p ~/tmp_rclone
cd ~/tmp_rclone

# download
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip

# unzip
unzip rclone-current-linux-amd64.zip

# install rclone binary at /home/$USER/bin
mkdir -p ~/bin
mv $(find . -name 'rclone') ~/bin

# install rclone manunal at /home/$USER/man (option)
mkdir -p ~/man/man1
mv $(find . -name 'rclone.1') ~/man/man1

# add rclone to path environment
cat << EOF >> ~/.profile
#rclone path
PATH=~/bin:\$PATH
EOF

# init profile
source ~/.profile

# clean tmp_rclone
cd ~/
rm -rf ~/tmp_rclone
```

## uninstall steps

```bash=
rm -rf ~/bin/rclone
rm -rf ~/man/man1/rclone.1

# check and clean profile
cat ~/.profile
```