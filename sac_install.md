---
tags: opentutorial, sac, seismic, geophysics 
---


# SAC 安裝說明  
>  Seismic Analysis Code (SAC)  
>  安裝環境 :  Linux 作業系統  
>  安裝檔案 : `sac-101.6a-linux_x86_64.tar.gz`  
>  本文連結 : https://hackmd.io/@kmo/opt_sac_install

[TOC]

# 安裝指令  
本文安裝 `sac` 所使用的指令。依序執行指令，即可安裝完成。  
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
`mkdir` 創立資料夾 `sac-101.61`。  
`-p` 若指定路徑已有目錄會跳過，若無將創立資料夾。  

## 2. 解壓縮至 ~/sac-101.6a
```bash
tar -xzvf sac-101.6a-linux_x86_64.tar.gz -C ~/sac-101.6a  
```
- Linux 指令說明:  
`tar -xzvf` 解壓縮 `*.tar.gz` 格式的檔案，  
`-C` 指定解壓縮路徑。
- ps:  
壓縮檔內有 `sac` 資料夾，故解壓縮後，現在 `sac` 實際安裝位置是 `~/sac-101.6a/sac`。

## 3. 修改初始化腳本

```bash
sed -i 's#SACHOME=/usr/local/sac#SACHOME=~/sac-101.6a/sac#' ~/sac-101.6a/sac/bin/sacinit.sh  
sed -i 's#PATH=${PATH}:${SACHOME}/bin#PATH=${SACHOME}/bin:${PATH}#' ~/sac-101.6a/sac/bin/sacinit.sh
```
`sac` 有提供初始化腳本。需依據安裝路徑，修改初始化腳本。  

- Linux 指令說明:  
`sed -i` 修改初始化腳本 `sacinit.sh` 內容。  
修改變數 `SACHOME` 從 `SACHOME=/usr/local/sac` -> `SACHOME=~/sac-101.6a/sac`。  
修改變數 `PATH` 從`PATH=${PATH}:${SACHOME}/bin` ->`PATH=${SACHOME}/bin:${PATH}`。  
修改 `PATH` 變數的順序，目的是讓系統先讀到我們裝的 `sac`，避免系統因存在相同名稱的程式，而執行到其他指令。  

## 4. 設定環境變數檔
```bash
echo "source ~/sac-101.6a/sac/bin/sacinit.sh" >> ~/.bashrc
```  
新增初始化腳本至環境變數設定檔，登入時會自動讀取。  

- Linux 指令說明:  
`echo` 把雙引號內容，寫入`~/.bashrc`。  
搭配 `>>`，能把原本應輸出至螢幕的內容，append 內容至檔案。  
`~/.bashrc` 是環境變數設定檔，登入時會讀取及執行內容。  

## 5. 讓環境變數檔生效
```bash
source ~/.bashrc
```
or 登出 (Ctrl + D) 再登入系統。  
這時候打 `sac` 就會有反應。  
- Linux 指令說明:  
`source` 讀取腳本內容套用至 shell 環境。

## 6. 安裝相依 Library

```bash
# install dependent library 
## for RHEL/CentOS
sudo yum install libSM libICE libXpm libX11

## for Debian/Ubuntu
sudo apt install libsm6 libice6 libxpm4 libx11-6 
```
壓縮檔 `sac-101.6a-linux_x86_64.tar.gz` 內提供的 `sac`，為預先編譯的執行檔。Linux 系統需具備相依的 library，才能正常使用。  
- Linux 指令說明:  
目前知名 Linux distribution，主要為 `RHEL/CentOS` or `Debian/Ubuntu` 兩種派系。兩者主要差別之一，是套件管理工具不同，一個是 `yum` 一個是 `apt` 。若用戶沒有管理員權限，沒法用 `sudo` 安裝套件，請向主機管理員請求協助。
- ps:  
若管理員沒法幫忙安裝，可以自行下載 library 套件(`*.rpm` or `*.deb`)，解壓縮至指定目錄，並新增環境變數 `LD_LIBRARY_PATH` 指向解壓縮的路徑，就能讓 sac 正常運行。


## 7. 查詢相依的 Library (參考內容，可跳過)
以 CentOS 7 最小安裝的環境為例，如下截圖，可以看到系統缺少 `sac` 所需的 library `libSM.so.6`、`libICE.so.6`、`libXpm.so.4`、`libX11.so.6`。  
```bash
ldd $(which sac)
```  
![](https://i.imgur.com/FTLGY8t.png)  


- Linux 指令說明:  
`ldd` 列出相依的 library。  
`which` 查閱該指令路徑。  
`$()` 為 Bash Command substitution 功能，輸出`$()`內執行結果。  
```bash
## for RHEL/CentOS
yum whatprovides libSM.so.6

## for Debian/Ubuntu
apt-file search libSM.so.6
```  
查詢哪些套件提供 `libSM.so.6`。可把 `libSM.so.6` 替換其他也是 `not found` 的 library。(在 `Debian/Ubuntu` 環境，`apt-file` 並非預先安裝指令，可能需自行安裝。)

### 7.1 Library 對應的套件名稱 
幫大家查好了相依的 library 的套件名稱   

| Library     | RHEL/CentOS | Debian/Ubuntu |
| ----------- | ----------- | ------------- |
| libSM.so.6  | libSM       | libsm6        |
| libICE.so.6 | libICE      | libice6       |
| libXpm.so.4 | libXpm      | libxpm4       |
| libX11.so.6 | libX11      | libx11-6      |

### 7.2 確認 Library 
安裝完套件後，確認 `sac` 都有找到所需的 library。  
![](https://i.imgur.com/RHVLolU.png)  

# 安裝成功畫面  
執行 `sac`，進入 `sac` 環境。  
![](https://i.imgur.com/oYPMgXC.png)

## 參考資料
- sac README 文件 (`$SACHOME/README`)
- sac 網頁提供的[安裝說明](https://seiscode.iris.washington.edu/projects/sac/wiki/Binary_Installation)

# 後記  
畢業已久，脫離地科領域好一陣子。  
臉書看到學生，輾轉詢問如何安裝 `sac`。  
(7年前我也卡在安裝 sac 過 XD)。  
因此撰寫此文，補充一些 Linux 系統的說明，  
希望能幫助初學者，減少熟悉 Linux 系統所花的成本。  
安裝完 `sac`，趕緊進到地震學的重頭戲，利用 `sac` 進行震波分析。  

---
>   This tutorial is licensed under a [CC BY-SA 4.0][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]  

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/ 
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png  
