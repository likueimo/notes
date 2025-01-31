---
title: iPhone 透過 USB 傳輸線分享網路
tags: [notes, iPhone, USB, cable, network, Windows, ' win10', MS, Microsoft]

---

# iPhone 透過 USB 傳輸線分享網路
- 本文URL: https://hackmd.io/@kmo/iphone_hotspot_usb
## 前言
> 2014 年開始用 Android 手機，當需要手機分享網路給電腦，這 10 年來不曾有問題，USB 傳輸線插上就可使用。2024 年底，手機摔壞後換 iPhone 試試，沒想到用 USB 傳輸線分享網路給電腦，動用到我的工程師魂，才能完成... :innocent:

## 測試環境
- 電腦系統 : Windows 10  和 Windows 11，皆為 64 位元
- 手機系統 : iPhone 16 Plus with iOS 18.2.1
- 解壓縮軟體 : 7-Zip 24.09
## 操作步驟
- 大部分網路文章推薦安裝 iTunes，在安裝過程理論上會自動裝好 driver
  - 若完整安裝 iTunes，仍然無法透過 USB 傳輸線分享網路，  
    可試試步驟 [安裝 usbaapl64.inf 和 netaapl64.inf](#安裝-usbaapl64.inf-和-netaapl64.inf)
- 本篇文章步驟不需完整安裝 iTunes，只整理必裝的檔案資訊
### 安裝 AppleMobileDeviceSupport64.msi
- 下載 iTunes : https://www.apple.com/itunes/download/win64
- 透過 7-zip 解壓縮 iTunes 檔案: 
  - 對 `iTunes64Setup.exe` 點右鍵->7-zip-> 解壓縮至 `"iTunes64Setup\"`
- 僅需安裝 `AppleMobileDeviceSupport64.msi`: 
  - 進去資料夾 `iTunes64Setup` -> 對 `AppleMobileDeviceSupport64.msi` 點左鍵兩下進行安裝 
### 安裝 usbaapl64.inf 和 netaapl64.inf
- 透過 7-zip 解壓縮 `AppleMobileDeviceSupport64.msi`
  - 對 `AppleMobileDeviceSupport64.msi` 點右鍵->7-zip-> 解壓縮至 `"AppleMobileDeviceSupport64\"`
- 安裝 `usbaapl64.inf`
  - 進去資料夾 `AppleMobileDeviceSupport64` -> 尋找檔案 `usbaapl64.inf` -> 點右鍵按安裝
- 安裝 `netaapl64.inf`
  - 進去資料夾 `AppleMobileDeviceSupport64` -> 尋找檔案 `netaapl64.inf` -> 點右鍵按安裝
### iPhone「個人熱點」
  - 開啟「允許其他人加入」
  - 手機透過 USB 傳輸線連結電腦，理論上會自動分享網路給電腦

## Powershell 步驟
- 用 Windows 內建 Powershell 指令，也可以完成任務
  - 應使用管理員權限開啟 Powershell，才能完成安裝 inf 檔動作
### 安裝 AppleMobileDeviceSupport64.msi
```powershell=
# 命名暫存資料夾
$temp_itune_dir = "$env:TEMP\temp_itunes"

# 建立暫存資料夾，並進到暫存資料夾
cd (mkdir $temp_itune_dir)

# 下載 iTunes64Setup.exe
curl.exe -L -o iTunes64Setup.exe https://www.apple.com/itunes/download/win64

# 解壓縮 iTunes64Setup.exe 
.\iTunes64Setup.exe  /extract

# 安裝 AppleMobileDeviceSupport64.msi
msiexec /i AppleMobileDeviceSupport64.msi /qb
```
### 安裝 usbaapl64.inf 和 netaapl64.inf
```powershell=
# 解壓縮 AppleMobileDeviceSupport64.msi
msiexec /a AppleMobileDeviceSupport64.msi /qb TARGETDIR="$temp_itune_dir\AppleMobileDeviceSupport64"
# 取得 usbaapl64.inf 絕對路徑
$usbaapl64_path = (Get-ChildItem -Path "$temp_itune_dir" -Filter usbaapl64.inf -Recurse).FullName
# 取得 netaapl64.inf 絕對路徑
$netaapl64_path = (Get-ChildItem -Path "$temp_itune_dir" -Filter netaapl64.inf -Recurse).FullName

# 安裝 usbaapl64.inf 
pnputil /add-driver "$usbaapl64_path" /install
# 安裝 netaapl64.inf
pnputil /add-driver "$netaapl64_path" /install
```
- 確認已安裝成功 usbaapl64.inf 和 netaapl64.inf
```powershell=
pnputil /enum-drivers | Select-String apple -context 2,5
```
- 上述指令輸出範例
```powersehll=
PS C:\Users\username\AppData\Local\Temp\temp_itunes> pnputil /enum-drivers | Select-String apple -context 2,5

  發佈名稱:     oem54.inf
  原始名稱:      usbaapl64.inf
> 提供者名稱:      Apple, Inc.
  類別名稱:         通用序列匯流排控制器
  類別 GUID:         {36fc9e60-c465-11cf-8056-444553540000}
  驅動程式版本:     05/19/2017 6.0.9999.69
  簽署人名稱:        Microsoft Windows Hardware Compatibility Publisher

  發佈名稱:     oem67.inf
  原始名稱:      netaapl64.inf
> 提供者名稱:      Apple
  類別名稱:         網路介面卡
  類別 GUID:         {4d36e972-e325-11ce-bfc1-08002be10318}
  驅動程式版本:     07/15/2013 1.8.5.1
  簽署人名稱:        Microsoft Windows Hardware Compatibility Publisher
```
- 補充說明: 依照指令輸出的 oem 編號，若需要移除 usbaapl64.inf 和 netaapl64.inf，可透過下述指令
```powershell=
pnputil /delete-driver oem54.inf /uninstall
pnputil /delete-driver oem67.inf /uninstall
```

## 後記
- 特別感謝工作認識的夥伴李宇正，在我急需要用 iPhone 分享網路時候，指導我安裝 iTunes，完成用 USB 傳輸線分享網路給電腦
- 完整安裝 iTunes 大部分情況能成功，卻有幾次失敗。當下趕時間急需要網路，只剩下 iPhone 能分享網路，卻苦惱找不到解決方法，因此事後花時間深入了解，並催生這篇文章出來，避免之後也有人需要使用。在撰寫本文章過程，也意外練習了 Powershell 指令，感覺有玩出興趣 (感謝 iPhone XD)
- Microsoft Update Catalog (底下URL)，其實也有提供 driver。似乎電腦有網路情況下，插上 iPhone 設備， Windows 會試著從 update 系統安裝 USB driver (appleusb.inf, 可透過 pnputil 查詢)，那此時只需安裝 Net driver (netaapl64.inf) 即可。netaapl64.inf 也可從 Microsoft Update Catalog 安裝，名稱叫 `Apple - Net - 7/15/2013 12:00:00 AM - 1.8.5.1 `
  - https://www.catalog.update.microsoft.com/Search.aspx?q=Apple



## 表格整理
- iPhone 透過 USB 傳輸線分享網路給電腦，應裝 3 個檔案 (其中 USB driver 擇一即可)

| 檔案說明   | 檔案名稱 | 檔案來源 |
| ---------- | ---- | ---- |
| `Apple Mobile Device Support Installer` | `AppleMobileDeviceSupport64.msi` | `iTunes64Setup.exe` |
| `Apple Mobile Device Ethernet driver`  | `netaapl64.inf` | `AppleMobileDeviceSupport64.msi` or `Microsoft Update Catalog`|

- 底下 USB driver 安裝其中之一即可，兩者同時存在也不影響

| 檔案說明  | 檔案名稱 | 檔案來源 |
| ---------- | ---- | ---- |
| `Apple Mobile Device driver` | `AppleUsb.inf` | `Microsoft Update Catalog`    |
| `Apple USB driver`  | `usbaapl64.inf` | `AppleMobileDeviceSupport64.msi` |


## 參考資料
- Apple 官網說明及討論
  - https://support.apple.com/zh-tw/guide/iphone/iph45447ca6/ios
  - https://support.apple.com/guide/iphone/share-internet-connection-personal-hotspot-iph45447ca6/18.0/ios/18.0
  - https://discussions.apple.com/thread/255073651
- github 網友分享
  - https://github.com/NelloKudo/Apple-Mobile-Drivers-Installer
  - https://github.com/Jiangxiaogang/iphone_usb_driver
- Powershell 用到一些技巧
  - https://silentinstallhq.com/apple-itunes-install-and-uninstall-powershell
  - https://github.com/PowerShell/PowerShell/issues/17949
  - https://www.advancedinstaller.com/extract-msi-msix-exe-installer-content.html

---
{%hackmd @kmo/widget_license %}# iPhone 透過 USB 傳輸線分享網路
- 本文URL: https://hackmd.io/@kmo/iphone_hotspot_usb
## 前言
> 2014 年開始用 Android 手機，當需要手機分享網路給電腦，這 10 年來不曾有問題，USB 傳輸線插上就可使用。2024 年底，手機摔壞後換 iPhone 試試，沒想到用 USB 傳輸線分享網路給電腦，動用到我的工程師魂，才能完成... :innocent:

## 測試環境
- 電腦系統 : Windows 10  和 Windows 11，皆為 64 位元
- 手機系統 : iPhone 16 Plus with iOS 18.2.1
- 解壓縮軟體 : 7-Zip 24.09
## 操作步驟
- 大部分網路文章推薦安裝 iTunes，在安裝過程理論上會自動裝好 driver
  - 若完整安裝 iTunes，仍然無法透過 USB 傳輸線分享網路，  
    可試試步驟 [安裝 usbaapl64.inf 和 netaapl64.inf](#安裝-usbaapl64.inf-和-netaapl64.inf)
- 本篇文章步驟不需完整安裝 iTunes，只整理必裝的檔案資訊
### 安裝 AppleMobileDeviceSupport64.msi
- 下載 iTunes : https://www.apple.com/itunes/download/win64
- 透過 7-zip 解壓縮 iTunes 檔案: 
  - 對 `iTunes64Setup.exe` 點右鍵->7-zip-> 解壓縮至 `"iTunes64Setup\"`
- 僅需安裝 `AppleMobileDeviceSupport64.msi`: 
  - 進去資料夾 `iTunes64Setup` -> 對 `AppleMobileDeviceSupport64.msi` 點左鍵兩下進行安裝 
### 安裝 usbaapl64.inf 和 netaapl64.inf
- 透過 7-zip 解壓縮 `AppleMobileDeviceSupport64.msi`
  - 對 `AppleMobileDeviceSupport64.msi` 點右鍵->7-zip-> 解壓縮至 `"AppleMobileDeviceSupport64\"`
- 安裝 `usbaapl64.inf`
  - 進去資料夾 `AppleMobileDeviceSupport64` -> 尋找檔案 `usbaapl64.inf` -> 點右鍵按安裝
- 安裝 `netaapl64.inf`
  - 進去資料夾 `AppleMobileDeviceSupport64` -> 尋找檔案 `netaapl64.inf` -> 點右鍵按安裝
### iPhone「個人熱點」
  - 開啟「允許其他人加入」
  - 手機透過 USB 傳輸線連結電腦，理論上會自動分享網路給電腦

## Powershell 步驟
- 用 Windows 內建 Powershell 指令，也可以完成任務
  - 應使用管理員權限開啟 Powershell，才能完成安裝 inf 檔動作
### 安裝 AppleMobileDeviceSupport64.msi
```powershell=
# 命名暫存資料夾
$temp_itune_dir = "$env:TEMP\temp_itunes"

# 建立暫存資料夾，並進到暫存資料夾
cd (mkdir $temp_itune_dir)

# 下載 iTunes64Setup.exe
curl.exe -L -o iTunes64Setup.exe https://www.apple.com/itunes/download/win64

# 解壓縮 iTunes64Setup.exe 
.\iTunes64Setup.exe  /extract

# 安裝 AppleMobileDeviceSupport64.msi
msiexec /i AppleMobileDeviceSupport64.msi /qb
```
### 安裝 usbaapl64.inf 和 netaapl64.inf
```powershell=
# 解壓縮 AppleMobileDeviceSupport64.msi
msiexec /a AppleMobileDeviceSupport64.msi /qb TARGETDIR="$temp_itune_dir\AppleMobileDeviceSupport64"
# 取得 usbaapl64.inf 絕對路徑
$usbaapl64_path = (Get-ChildItem -Path "$temp_itune_dir" -Filter usbaapl64.inf -Recurse).FullName
# 取得 netaapl64.inf 絕對路徑
$netaapl64_path = (Get-ChildItem -Path "$temp_itune_dir" -Filter netaapl64.inf -Recurse).FullName

# 安裝 usbaapl64.inf 
pnputil /add-driver "$usbaapl64_path" /install
# 安裝 netaapl64.inf
pnputil /add-driver "$netaapl64_path" /install
```
- 確認已安裝成功 usbaapl64.inf 和 netaapl64.inf
```powershell=
pnputil /enum-drivers | Select-String apple -context 2,5
```
- 上述指令輸出範例
```powersehll=
PS C:\Users\username\AppData\Local\Temp\temp_itunes> pnputil /enum-drivers | Select-String apple -context 2,5

  發佈名稱:     oem54.inf
  原始名稱:      usbaapl64.inf
> 提供者名稱:      Apple, Inc.
  類別名稱:         通用序列匯流排控制器
  類別 GUID:         {36fc9e60-c465-11cf-8056-444553540000}
  驅動程式版本:     05/19/2017 6.0.9999.69
  簽署人名稱:        Microsoft Windows Hardware Compatibility Publisher

  發佈名稱:     oem67.inf
  原始名稱:      netaapl64.inf
> 提供者名稱:      Apple
  類別名稱:         網路介面卡
  類別 GUID:         {4d36e972-e325-11ce-bfc1-08002be10318}
  驅動程式版本:     07/15/2013 1.8.5.1
  簽署人名稱:        Microsoft Windows Hardware Compatibility Publisher
```
- 補充說明: 依照指令輸出的 oem 編號，若需要移除 usbaapl64.inf 和 netaapl64.inf，可透過下述指令
```powershell=
pnputil /delete-driver oem54.inf /uninstall
pnputil /delete-driver oem67.inf /uninstall
```

## 後記
- 特別感謝工作認識的夥伴李宇正，在我急需要用 iPhone 分享網路時候，指導我安裝 iTunes，完成用 USB 傳輸線分享網路給電腦
- 完整安裝 iTunes 大部分情況能成功，卻有幾次失敗。當下趕時間急需要網路，只剩下 iPhone 能分享網路，卻苦惱找不到解決方法，因此事後花時間深入了解，並催生這篇文章出來，避免之後也有人需要使用。在撰寫本文章過程，也意外練習了 Powershell 指令，感覺有玩出興趣 (感謝 iPhone XD)
- Microsoft Update Catalog (底下URL)，其實也有提供 driver。似乎電腦有網路情況下，插上 iPhone 設備， Windows 會試著從 update 系統安裝 USB driver (appleusb.inf, 可透過 pnputil 查詢)，那此時只需安裝 Net driver (netaapl64.inf) 即可。netaapl64.inf 也可從 Microsoft Update Catalog 安裝，名稱叫 `Apple - Net - 7/15/2013 12:00:00 AM - 1.8.5.1 `
  - https://www.catalog.update.microsoft.com/Search.aspx?q=Apple



## 表格整理
- iPhone 透過 USB 傳輸線分享網路給電腦，應裝 3 個檔案 (其中 USB driver 擇一即可)

| 檔案說明   | 檔案名稱 | 檔案來源 |
| ---------- | ---- | ---- |
| `Apple Mobile Device Support Installer` | `AppleMobileDeviceSupport64.msi` | `iTunes64Setup.exe` |
| `Apple Mobile Device Ethernet driver`  | `netaapl64.inf` | `AppleMobileDeviceSupport64.msi` or `Microsoft Update Catalog`|

- 底下 USB driver 安裝其中之一即可，兩者同時存在也不影響

| 檔案說明  | 檔案名稱 | 檔案來源 |
| ---------- | ---- | ---- |
| `Apple Mobile Device driver` | `AppleUsb.inf` | `Microsoft Update Catalog`    |
| `Apple USB driver`  | `usbaapl64.inf` | `AppleMobileDeviceSupport64.msi` |


## 參考資料
- Apple 官網說明及討論
  - https://support.apple.com/zh-tw/guide/iphone/iph45447ca6/ios
  - https://support.apple.com/guide/iphone/share-internet-connection-personal-hotspot-iph45447ca6/18.0/ios/18.0
  - https://discussions.apple.com/thread/255073651
- github 網友分享
  - https://github.com/NelloKudo/Apple-Mobile-Drivers-Installer
  - https://github.com/Jiangxiaogang/iphone_usb_driver
- Powershell 用到一些技巧
  - https://silentinstallhq.com/apple-itunes-install-and-uninstall-powershell
  - https://github.com/PowerShell/PowerShell/issues/17949
  - https://www.advancedinstaller.com/extract-msi-msix-exe-installer-content.html

---
{%hackmd @kmo/widget_license %}