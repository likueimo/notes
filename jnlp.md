---
tags: notes, jnlp, java
---

# 開啟 *.jnlp 檔案的方式
> 本文連結: https://hackmd.io/@kmo/notes_jnlp  
> 才疏學淺，任何謬誤都歡迎登入 hackmd 後修改本文內容  :) 

:::info
#### 情境描述
我是個系統管理員，平常用筆電遠端管理 server，不需要使用到 java  
最近接手管理年紀較大的 server，系統的操作頁面要下載 *.jnlp 檔案才能開啟，沒有 html5 替代方案  
但又擔心用公司電腦裝 Oracle Java，公司被 Oracle 敲門收費  
:::

::: warning
#### 魔鬼藏在細節
為了開啟 *.jnlp 而安裝"新版"的 Oracle Java，在公司的電腦環境可能被 Oracle 討錢  
(如果一般家用情境是不用擔心被收錢)  
:::


## 解決方式

- 如果原廠有 support，先詢問原廠建議開啟 *.jnlp 方式
- 底下提及方法建議先從 OpenWebStart 試試
- 如果 OpenWebStart 失敗且不想花太多時間測試 IcedTea + OpenJDK，可直接參考 Oracle JRE 8u202 部分

::: spoiler OpenWebStart
- opensource 專案，主要由德國公司 Karakun 維護
- [官網連結](https://openwebstart.com) | [github 連結](https://github.com/karakun/OpenWebStart)
- 當開啟 *.jnlp 時，預設會下載適合的 java (OpenJDK)
- 透過 JVM 管理，可自由選擇背後 OpenJDK 來源(OpenJDK 版本請選擇 8 or 11)
  ![](https://openwebstart.com/docs/images/OWS_jvm_mgmt.png)
:::

::: spoiler IcedTea + OpenJDK
- IcedTea 是 opensource 的專案，開啟 java web start(*.jnlp) 用
- 背後需要裝有 java (OpenJDK) 來啟動
- 底下列出 2 個知名的 prebuilt 的 OpenJDK 專案，需同時安裝 IcedTea + OpenJDK
- IcedTea 會讀 `JAVA_HOME` 變數，`JAVA_HOME` 變數內容為 OpenJDK 的路徑。安裝過程若沒設定需自行設定
### azul(zulu)
- [IcedTea](https://www.azul.com/products/components/icedtea-web) | [OpenJDK](https://www.azul.com/downloads)   
  (依據 azul 的 IcedTea 網頁說明，OpenJDK 版本請選擇 8 or 11)  
- azul 的 OpenJDK 有包含 JavaFX
### adoptium
- [IcedTea](https://adoptopenjdk.net/icedtea-web.html) | [OpenJDK](https://adoptium.net)  
  (依據 adoptium 的 IcedTea 網頁說明，OpenJDK 版本請選擇 8)  
- adoptium 的 OpenJDK 並沒有包含 JavaFX
:::

::: spoiler Oracle JRE 8u202 
- 使用 2019 年以前的 Oracle JRE (最後一版是 8u202)，Oracle 並不會針對單純開啟 *.jnlp 用途收費，但可能有安全性風險
- 網友備份安裝檔 [github 連結](https://github.com/frekele/oracle-java/releases/tag/8u202-b08)
- 開啟 *.jnlp 僅需要安裝 JRE (Java Runtime Environment)，建議下載 `jre-*.tar.gz`(避免自動更新的步驟)。並加上 `PATH` 和 `JAVA_HOME`，即可用 `javaws` 指令開啟 *.jnlp
  [環境變數參考說明](https://www.java.com/en/download/help/path.html)
- 舉例來說，在 Windows 平台下載 `jre-8u202-windows-x64.tar.gz`，解壓縮後放到 `C:\Program Files\JAVA\jre1.8.0_202` 資料夾。並在環境變數新增 `JAVA_HOME`，以及在環境變數 `Path` 加上 java 的 bin 路徑
   - `JAVA_HOME`: `C:\Program Files\JAVA\jre1.8.0_202`
   - `PATH`: `C:\Program Files\JAVA\jre1.8.0_202\bin`
- 打開 Powershell 之類環境，下指令如下  
  `javaws.exe -verbose https://yourservice_ip:yourservice_ports/jnlp`
:::

## 技術細節補充
::: spoiler
- *.jnlp 是類似 xml 的描述檔案，是已被拋棄的技術。新版 java 已不提供 javaws 指令來開啟
- 社群 prebuilt 的 OpenJDK 不是全功能都能取代 Oracle Java。可參考 adoptium 做的對照表 https://adoptopenjdk.net/migration.html
![](https://i.imgur.com/8pD89fa.png =x350)
- 僅有 Oracle, Azul, BellSoft 和 Amazon 提供的 java 有包 JavaFX 
:::

## 參考資料
::: spoiler
- https://blog.darkthread.net/blog/open-jnlp-with-openjdk
- http://duckfly-tw.blogspot.com/2019/07/java-web-startjwsjdk11.html
- https://stackoverflow.com/a/58257869
- https://www.facebook.com/groups/twjug/posts/10160180533835013
  ![](https://i.imgur.com/lTqN2BH.png =x200)
- https://gist.github.com/hgomez/9650687?permalink_comment_id=4036679
- https://openwebstart.com/docs/FAQ.html#_how_to_run_openjfx_based_javafx_applications_with_openwebstart
:::

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png