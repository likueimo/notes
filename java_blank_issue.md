# 遠端桌面下，開啟 .jnlp 應用程式呈現空白

> 最近接手舊的專案，用到`Java Web Start 應用程式(.jnlp)`。而連到遠端電腦，執行遠端 win10 底下的 .jnlp 應用程式，卻呈現全白。而實際坐在該電腦前操作，可以正常執行和顯示畫面。此問題苦惱許久，最近終於解決了  
本文連結: https://hackmd.io/@kmo/java_blank_issue  

::: info 
- [Java Web Start 應用程式](https://en.wikipedia.org/wiki/Java_Web_Start) 為已經過時的技術，通常只有舊專案才有可能碰到  
- 本文為紀錄遇到此問題，解決思路和解法
::: 

## 問題簡述


| 情境             | 狀態                   |
| ---------------- | ---------------------- |
| win10 電腦前     | 開啟 `.jnlp` 正常顯示 |
| 遠端桌面到 win10 | 開啟 `.jnlp` 呈現空白 |


## 問題關鍵字搜尋
- 由於是使用 [Chrome Remote Desktop](https://remotedesktop.google.com) 發生的，習慣帶應用程式名稱，並添加異常情況去搜尋

```bash=
Chrome Remote Desktop Java blank
```
- 此時`Remote Desktop` 剛好是一個通用詞彙，幸運在 teamviewer 的社群看到類似問題討論，並且持續討論 3 年以上。之所以判斷 teamviewer 討論也適用，是因為此現象看起來都和 Java 應用程式有關，並且觀察其他搜尋結果，不同遠端程式都有類似 issue
- [Some applications does not show content (white window) Windows 10 host](https://community.teamviewer.com/English/discussion/10718/some-applications-does-not-show-content-white-window-windows-10-host)  
![](https://i.postimg.cc/tJb6fwNw/image.png =70%x)

## 測試解法
- 該文底下網友 `@mattmill30`，測試各種解法，比如測試 Java 環境變數都失敗，最後是針對 win10 的 DXDiag 硬體加速設定關閉，即可修正  
![](https://i.postimg.cc/nhr309z7/image.png =70%x)

::: spoiler 網友 `@mattmill30` 回覆完整內容
This answer expands upon @Julia's, second point 'Window contents that are drawn hardware-accelerated (e.g. browser content or 3D applications / games) are no longer drawn (you cannot see the application)'.

See 'Solution' below.

### Abstract
I have noticed this issue commonly relates to Java applications when the primary monitor is no-longer attached to the device; such as certain laptops which "disconnect" the built-in screen when the lid is shut - corroborated by @ashimet - or workstations when no screens are attached.

Based upon the behaviour of applications with and without a monitor attached, a surface level understanding of graphics accelerators (https://en.wikipedia.org/wiki/Framebuffer#Graphics_accelerators), and experiences in the forum posts I've referenced below, I believe the cause of the contentless windows is the result of application dependencies upon either the DirectDraw or Direct3D sub-systems - which interface with 3D acceleration hardware - when the accelerators are disabled as a result of the disconnection of a monitor.

I think I've also experienced similar problems when attempting to run applications which depend upon 3D acceleration within a Linux environment using the Wine compatibility layer (https://bugs.winehq.org/buglist.cgi?component=directx-d3d&product=Wine&resolution=---)

Perhaps this is a power-saving feature of 3D graphics processors which activates when no screens are detected, or perhaps this is related to the framebuffer being unaware of the supported output resolutions and so disables access to the GPU for the Windows shell, which causes Windows to revert to software-only rendering; or some other reason.

Whatever the cause, my assumption that the dedicated graphics hardware was unavailable for 3D processing when the monitor is disabled, and the common denominator for all software experiencing the problem was Java, I questioned whether the JVM was depending upon D3D for rendering, even though Windows only had software-only rendering available.

### Solution
Ref: https://superuser.com/a/496775
Using DXDiag, I checked the status of Display DirectX Features:
```
DirectDraw Acceleration: Enabled

Direct3D Acceleration: Enabled

AGP Texture Acceleration: Enabled
```
I manually created these registry values, in addition to creating the Direct3D\Drivers key:
```shell=
Reg Add HKLM\SOFTWARE\Microsoft\DirectDraw /V EmulationOnly /T REG_DWORD /D %_Mode% /F
Reg Add HKLM\SOFTWARE\Microsoft\Direct3D\Drivers /V SoftwareOnly /T REG_DWORD /D %_Mode% /F
```
Then, using DXDiag, I checked the new status of Display DirectX Features:
```
DirectDraw Acceleration: Disabled

Direct3D Acceleration: Disabled

AGP Texture Acceleration: Not Available
```
The Java program window now rendered content correctly via VNC Viewer, when monitor is detached.

I've not tried the "ForceDirectDrawEmulation" option within the Compatibility Wizard suggestion from the referenced SuperUser thread.

### Research and references
- [Disable hardware acceleration in Windows Display Settings](https://www.auslogics.com/en/articles/disable-hardware-acceleration-in-windows)
   Not tried because Display Settings window crashes
- [Attempt to prevent GPU (ATI Mobility Radeon HD 4500) from disabling hardware acceleration when monitor disconnects](https://social.technet.microsoft.com/forums/windows/en-us/8a9b5aa7-fe33-4e6d-b39b-8ac80a21fdc2/disable-monitor-off-detection-how?forum=w7itprogeneral)
```
# Disabled ?Display Detection?, to prevent disconnection being detected to avoid disabling of HW acceleration
HKLM\SYSTEM\ControlSet001\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000 Display_Detection_DEF DWORD 1

# Create/NULL DMMEnableDDCPolling for ATI cards - Ref: Post by NetMage
HKLM\SYSTEM\ControlSet001\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000 DMMEnableDDCPolling DWORD 0
```
- [System-wide (Java Control Panel) disabling of D3D acceleration for Java applications](https://forums.guru3d.com/threads/disable-hardware-acceleration-for-java.296918/#post-4096709)
- [System-wide (Envionment variables) disabling of D3D acceleration for Java applications](https://stackoverflow.com/a/36235217)
  - [System Properties for Java 2D](http://docs.oracle.com/javase/8/docs/technotes/guides/2d/flags.html)
    Tried setting user Environment Variables, but it didn't resolve the contentless Window issue:
```shell=
_JAVA_OPTION=-Dsun.java2d.d3d=false -Dsun.java2d.noddraw=true
J2D_D3D=false
```
- [Disable Direct3D acceleration on Windows 7](https://stackoverflow.com/questions/14497545/how-to-disable-direct3d-acceleration-on-windows-7)
  - [Guide for directx.cpl - changes 32-bit DirectDraw and Direct3D, but not 64-bit (dxdiag 32/64-bit)](https://stackoverflow.com/a/25508331)
- [Disable DirectDraw and Direct3D acceleration on Windows 8](https://superuser.com/questions/495303/how-do-i-disable-directdraw-and-direct3d-acceleration-on-windows-8)
  - [Guide for directx.cpl - changes 32-bit DirectDraw and Direct3D, but not 64-bit (dxdiag 32/64-bit)](https://superuser.com/a/504510)
  - [DirectDraw and Direct3D registry entries](https://superuser.com/a/496775)
:::

## 詳述解法
- [腳本來自 2009 年文章，網友 TheOutcaste 的回覆](https://www.techsupportforum.com/threads/shortcut-batch-file-code-to-disable-direct-draw-acceleration.437584/post-2493259)，主要是切換遠端 win10 系統的 `DirectDraw 加速` 和 `Direct3D 加速`，從 `已啟用` 變成 `已停用`
- 執行前開啟 `dxdiag.exe` 檢查，點選`顯示`，看到 `DirectX 功能`地方，預設應是`已啟用` 狀態
- 修改自上述腳本，用記事本存檔成 `script.cmd`，使用系統管理員權限執行 `script.cmd` 即可
```powershell=
@Echo Off
::Set Mode=1 to Disable, Mode=0 to Enable as the default if nothing specified on the command line.
Set _Mode=1
Reg Add HKLM\SOFTWARE\Microsoft\DirectDraw /V EmulationOnly /T REG_DWORD /D %_Mode% /F
Reg Add HKLM\SOFTWARE\Microsoft\Direct3D\Drivers /V SoftwareOnly /T REG_DWORD /D %_Mode% /F
```
- 執行後開啟 `dxdiag.exe` 檢查，點選`顯示`，看到 `DirectX 功能`地方，應改為`已停用` 狀態
![](https://i.postimg.cc/TwfmNB9S/dxd.png =90%x)
::: warning
- 測試筆電為工作用途，並無顯卡及遊戲需求，停用後並無明顯效能差異，但不確定其他情境停用是否有效能差異
- 若修改完有任何異常情況，只需要回到腳本，設定 `Set _Mode=0` 再執行一次即可恢復
:::
---
{%hackmd @kmo/widget_license %}