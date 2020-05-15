---
tags: opentutorial, sac
---


# SAC 安裝說明  
>  Seismic Analysis Code (SAC)  
>  安裝環境 :  Linux 系統  
>  安裝檔案 : `sac-101.6a-linux_x86_64.tar.gz`  
>  本文連結 : https://hackmd.io/@kmo/opentutorial_sac_install

[TOC]

## 0. 安裝 sac 指令  
依序執行指令，即可安裝完成。  
```bash=
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
```bash
mkdir -p ~/sac-101.6a
```  
- Linux 指令說明:  
使用指令 `mkdir` 創立資料夾 `sac-101.61`。  
`-p` 意思是，該路徑若已有目錄會跳過，若無將創立資料夾。  

## 2. 解壓縮至 ~/sac-101.6a
```bash
tar -xzvf sac-101.6a-linux_x86_64.tar.gz -C ~/sac-101.6a  
```
- Linux 指令說明:  
使用指令 `tar` ，搭配 option 解壓縮檔案。  
`-xzvf` 解壓縮 `*.tar.gz` 格式的檔案，  
`-C` 指定解壓縮路徑。

## 3. 修改初始化腳本

```bash
sed -i 's#SACHOME=/usr/local/sac#SACHOME=~/sac-101.6a/sac#' ~/sac-101.6a/sac/bin/sacinit.sh  
sed -i 's#PATH=${PATH}:${SACHOME}/bin#PATH=${SACHOME}/bin:${PATH}#' ~/sac-101.6a/sac/bin/sacinit.sh
```
`README`文件 (`~/sac-101.6a/sac/README`) 提及的`Environment Setup`步驟，  
依據現有環境修改初始化腳本。  

- Linux 指令說明:  
使用指令 `sed` 修改初始化腳本 `sacinit.sh` 內容。  
變數`SACHOME`從 `/usr/local/sac` 改為 `~/sac-101.6a/sac`。  
變數`PATH`，從`${PATH}:${SACHOME}/bin` 改為`${SACHOME}/bin:${PATH}`。目的是為了讓他先讀到我們裝的`sac`指令，避免系統存在相同名稱程式，而執行到其他程式。  
`-i` 意思是直接修改檔案內容。  

## 4. 設定環境變數
```bash
echo "source ~/sac-101.6a/sac/bin/sacinit.sh" >> ~/.bashrc
```  
把初始化腳本加到環境變數設定檔，登入時會自動讀取。  

- Linux 指令說明:  
使用指令`echo`把雙引號內容，寫入`~/.bashrc`。  
`echo` 搭配 `>>`，能把原本輸出螢幕的內容，append 內容至檔案。  
`~/.bashrc` 環境變數設定檔，登入時會讀取並執行裡面內容。  
`source *.sh` 讀取並執行腳本內容，並應用至 bash 環境。

## 5. 讓環境變數生效
```bash
source ~/.bashrc
```
or 登出 (Ctrl + D) 再登入系統。  
這時候打 `sac` 就會有反應。  

## 6. 查閱及安裝相依 Library 
- 以 CentOS 舉例說明  
在 CentOS 7 最小安裝環境下，
壓縮檔`sac-101.6a-linux_x86_64.tar.gz` 內提供的 sac，為預先編譯好的 dynamic binary 版本。如下截圖，可以看到缺少 library `libSM.so.6`、`libICE.so.6`、`libXpm.so.4`、`libX11.so.6`。   

![](https://i.imgur.com/FTLGY8t.png)
查閱哪些套件提供`libSM.so.6`。
![](https://i.imgur.com/v1Yc8oH.png)
```bash=1
ldd $(which sac)
```  

- Linux 指令說明:  
`ldd` 列出相依的 library。  
`which` 查閱該 Linux 指令路徑。  
`$()` 為 Bash Command substitution 功能，輸出`$()`內執行結果。

```bash=2
yum whatprovides libSM.so.6
```  

查詢哪些套件提供 `libSM.so.6`。(可自行把 `libSM.so.6` 替換其他找不到 library)  
```bash=3
sudo yum install libSM libICE libXpm libX11
```
安裝相依的 libaray。
- Linux 指令說明:  
`yum` 是`CentOS`套件管理指令。


- 幫大家查好了相依的 library 對應的套件名稱

| library     | RHEL/CentOS | Debian/Ubuntu |
| ----------- | ----------- | ------------- |
| libSM.so.6  | libSM       | libsm6        |
| libICE.so.6 | libICE      | libice6       |
| libXpm.so.4 | libXpm      | libxpm4       |
| libX11.so.6 | libX11      | libx11-6      |



# 安裝成功畫面
- 確認都有找到相依的 library  
![](https://i.imgur.com/RHVLolU.png)  
- 執行 sac
![](https://i.imgur.com/oYPMgXC.png)

# 後記
畢業已久，脫離地科領域好一陣子。  
臉書看到學生，輾轉詢問如何安裝 `sac`。  
因此撰寫此文，額外補充一些 Linux 系統的說明，  
希望能幫助初學者，減少熟悉 Linux 系統所花的成本。。  
安裝完`sac`，趕緊進到地震學的重頭戲，利用 `sac` 進行震波分析。  

---
>   This tutorial is licensed under a [CC BY-SA 4.0][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]  

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png  
