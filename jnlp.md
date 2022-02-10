# 開啟 *.jnlp 檔案的方式

:::info
#### 情境描述
我是個系統管理員，平常用筆電搭配 web + ssh client 遠端管理 server，不需要使用到 java
但較舊的 server 相依的服務，如 BMC 的 console 之類，沒有 html5 的替代方案，僅能下載 *.jnlp 使用
:::

::: warning
#### 魔鬼藏在細節
為了開啟 *.jnlp 而安裝 Oracle Java，在公司的電腦環境可能被 Oracle 討錢
:::


## 解決方式

- 如果原廠有 support，先詢問原廠建議開啟 *.jnlp 方式
- 建議先從 OpenWebStart 試試，如果開啟 *.jnlp 失敗，且不想花太多時間試，可直接參考 Oracle JRE 8u202 部分

::: spoiler OpenWebStart
- opensource 專案，主要由德國公司 Karakun 維護
- [官網連結](https://openwebstart.com) | [github 連結](https://github.com/karakun/OpenWebStart)
- 當開啟 *.jnlp 時，預設會下載 prebuilt 的 OpenJDK
- 透過 JVM 管理，可自由選擇背後 prebuilt OpenJDK 來源
  ![](https://openwebstart.com/docs/images/OWS_jvm_mgmt.png)
:::

::: spoiler IcedTea + OpenJDK
- IcedTea 是 opensource 的專案，開啟 java web start(*.jnlp) 用
- 背後需要裝有 java (OpenJDK) 來啟動
- 底下列出 2 個知名的 prebuilt 的 OpenJDK 專案
### azul(zulu)
- prebuilt openjdk: https://www.azul.com/downloads
- prebuilt IcedTea: https://www.azul.com/products/components/icedtea-web/
### adoptium
- prebuilt openjdk: https://adoptium.net
- prebuilt IcedTea: https://adoptopenjdk.net/icedtea-web.html
:::

::: spoiler Oracle JRE 8u202 
- 使用 2019 年以前的版本(最後一版是 8u202)，Oracle 並不會收費，但可能有安全性風險
- 網友備份安裝檔 [github 連結](https://github.com/frekele/oracle-java/releases/tag/8u202-b08)，阿榮福利味也可以下載舊版 [連結](https://www.azofreeware.com/2007/04/windows-platform-javatm-se-runtime.html)
- 開啟 *.jnlp 僅需要安裝 JRE (Java Runtime Environment)，例如檔名 `jre-8u202-windows-x64.exe`
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
- https://stackoverflow.com/a/58257869
- https://www.facebook.com/groups/twjug/posts/10160180533835013
  ![](https://i.imgur.com/lTqN2BH.png =x200)
- https://gist.github.com/hgomez/9650687?permalink_comment_id=4036679
:::

