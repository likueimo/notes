# SAC 安裝說明  
本文連結 : https://hackmd.io/@kmo/sac_install_tutorial

>  Seismic Analysis Code (SAC)  
>  在 Linux 環境下，安裝`sac-101.6a-linux_x86_64.tar.gz`。  


[TOC]

## 0. 全部指令
```
# create directory
mkdir -p ~/sac-101.6a

# untar sac
tar -xzvf sac-101.6a-linux_x86_64.tar.gz -C ~/sac-101.6a

# modify sacinit.sh
sed -i 's#SACHOME=/usr/local/sac#SACHOME=~/sac-101.6a/sac#' ~/sac-101.6a/sac/bin/sacinit.sh
sed -i 's#PATH=${PATH}:${SACHOME}/bin#PATH=${SACHOME}/bin:${PATH}#' ~/sac-101.6a/sac/bin/sacinit.sh

# append command to ~/.bashrc
echo "source ~/sac-101.6a/sac/bin/sacinit.sh" >> ~/.bashrc

# install dependent library 
## for RHEL/CentOS
sudo yum install libSM libICE libXpm libX11

## for Debian/Ubuntu
sudo apt install libsm6 libice6 libxpm4 libx11-6 
```


## 1. 創立資料夾
- `mkdir -p ~/sac-101.6a`  
Linux 指令說明:  
使用指令 `mkdir` 創立資料夾 `sac-101.61`。  
`-p` 意思是，該路徑若已有目錄會跳過，若無將創立資料夾。  

## 2. 解壓縮至 ~/sac-101.6a

- `tar -xzvf sac-101.6a-linux_x86_64.tar.gz -C ~/sac-101.6a`  
Linux 指令說明:  
使用指令 `tar` ，搭配以下 option 解壓縮檔案。  
`-xzvf` 解壓縮 `*.tar.gz` 檔案，  
`-C` 指定解壓縮路徑。  

## 3. 修改初始化腳本
根據 sac `README` 說明(`~/sac-101.6a/sac/README`)，  
有提供環境變數初始化腳本。  
需依據現有環境修改初始化腳本。
- `sed -i 's#SACHOME=/usr/local/sac#SACHOME=~/sac-101.6a/sac#' ~/sac-101.6a/sac/bin/sacinit.sh`  
Linux 指令說明:  
使用指令 `sed` 修改初始化腳本 `sacinit.sh` 內容，  
變數`SACHOME`從 `/usr/local/sac` 改為 `~/sac-101.6a/sac`  
`-i` 意思是直接修改檔案內容。
- (非必要) `sed -i 's#PATH=${PATH}:${SACHOME}/bin#PATH=${SACHOME}/bin:${PATH}#' ~/sac-101.6a/sac/bin/sacinit.sh`  
Linux 指令說明:  
使用指令 `sed` 修改初始化腳本 `sacinit.sh` 內容，  
變數`PATH`，從`${PATH}:${SACHOME}/bin` 改為`${SACHOME}/bin:${PATH}`。  
目的是為了讓他先讀到我們裝的`sac`指令，  
避免系統存在相同名稱程式，而執行到其他程式。
## 4. 設定環境變數
- `echo "source ~/sac-101.6a/sac/bin/sacinit.sh" >> ~/.bashrc`  
把初始化腳本加到環境變數設定檔，登入時會自動執行。  
Linux 指令說明:  
使用指令`echo`把雙引號內容，寫入`~/.bashrc`。  
`~/.bashrc` 是環境變數設定檔，登入時會讀取並執行裡面內容。  
`echo` 搭配 `>>`，能把原本輸出螢幕的內容，append 內容至檔案。  
`source *.sh` 讀取並執行腳本內容，並應至 bash 環境。

## 5. 讓環境變數生效
- 登出再登入系統 or `source ~/.bashrc`。  
這時候打 `sac` 就會有反應。  

## 6. 查閱及安裝相依 Library 
- 以 CentOS 舉例說明  
在 CentOS 7 最小安裝環境下，可以看到缺少 library `libSM.so.6`、`libICE.so.6`、`libXpm.so.4`、`libX11.so.6`。
![](https://i.imgur.com/FTLGY8t.png)
查閱哪些套件提供`libSM.so.6`。
![](https://i.imgur.com/v1Yc8oH.png)
- `ldd $(which sac)`  
壓縮檔`sac-101.6a-linux_x86_64.tar.gz` 內提供，為`sac`預先編譯好的 dynamic binary 版本。有可能使用的 Linux 系統，尚未安裝相依的 library。  
Linux 指令說明:  
`ldd` 列出相依的 library。  
`which` 查閱該 Linux 指令路徑。  
`$()` 為 Bash Command substitution 功能，輸出`$()`內執行結果。
- `yum whatprovides libSM.so.6`  
查詢哪些套件提供 `libSM.so.6`。  
Linux 指令說明:  
`yum` 是`CentOS`套件管理指令。
- `sudo yum install libSM libICE libXpm libX11`  
安裝相依的 libaray。

# 安裝成功執行畫面

![](https://i.imgur.com/ckjjpDw.png)  


---
>   This tutorial is licensed under a [CC BY-SA 4.0][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]  

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png  
