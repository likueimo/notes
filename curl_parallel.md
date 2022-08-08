---
tags: notes, bash, tool, cli, curl
---

# 使用 `curl` 的 `--parallel` 

> :memo: `curl` 指令就像是電腦系統上的瑞士刀，可靠、好用且無所不在。除了 Linux 和 Mac 系統內有提供，[Win10 也在 2017 年後內建 `curl.exe`](https://curl.se/windows/microsoft.html)。時至今日，你可以說在任何平台的系統都能見到他。隨著 RESTful API 標準興起，很有可能你呼叫第一個 API，就是透過 `curl` 指令完成
>
> `curl` 在 [2019 的 7.66 版](https://daniel.haxx.se/blog/2019/07/22/curl-goez-parallel) 迎來新功能 `--parallel`，遊戲規則產生變化，我們不需再藉由外部工具如 `xargs` 或 `GNU Parallel`，即可達成並行化 (concurrency) 工作。curl 開發者持續新增和修正相關功能，建議若要使用此功能直接[選用最新版`curl`](https://curl.se/download.html)

::: warning
撰文者為系統管理員，測試環境為 Bash 搭配 curl `7.84.0`  
透過 server 的 BMC 提供的 redfish，演示 `--parallel` 功能  
本文連結: https://hackmd.io/@kmo/curl_parallel  
任何回饋歡迎留言在此篇 hackmd :)  
:::

## 實例演示

::: info
:scroll: 情境說明
- 管理的 server 分 3 個群組，每個群組有 10 台電腦，共 30 台電腦
- 現代 server 的 BMC 服務，幾乎都有提供 redfish api
- 使用 redfish api，備份 server 的 BIOS 設定檔

```bash=
# 群組 1 的電腦 IP 範圍
10.50.1.[1-10]
# 群組 2 的電腦 IP 範圍
10.50.2.[1-10] 
# 群組 3 的電腦 IP 範圍
10.50.3.[1-10] 
```
- 透過 api 取得單一台電腦 BIOS 設定的範例  
```bash=
curl -sku usr:pw 'https://10.50.1.1/redfish/v1/Systems/1/bios'
```
- ps: 不同硬體廠牌的 api 都略有差異，實際應用務必查詢自己機器的 redfish 說明文件，或是詢問硬體 vendor
:::

### 匹配多個 URL

- 透過 curl 的 [匹配 URL 語法](https://everything.curl.dev/cmdline/globbing)，可以一次匹配多個 URL 並執行 ，簡化指令成為
```bash=
curl -sku usr:pw 'https://10.50.[1-3].[1-10]/redfish/v1/Systems/1/bios'
```
- 實務上會希望把 bios 設定檔備份下來，但 URL 結尾名稱都是 `bios`，若使用 `-O` 直接存檔，檔名相同會彼此覆蓋，最後只存到 1 個 `bios` 檔案
- 因此 curl 開發者也考慮到此情況，所以 URL 匹配語法支援輸出變數，可改寫為
```bash=
# 檔名的 `#1` 對應第 1 個 globbing `[1-3]`
# 檔名的 `#2` 對應第 2 個 globbing `[1-10]`
curl -sku usr:pw 'https://10.50.[1-3].[1-10]/redfish/v1/Systems/1/bios' -o '10.50.#1.#2.bios.conf'
```

### 使用 `--parallel` 

::: success
- 當 request 的 URL 是不同來源，curl 開發者[建議可以使用 `--parallel` 並行化處理](https://everything.curl.dev/cmdline/urls/parallel)
- 除此之外還有 2 個 option 可搭配使用
  - `--parallel-immediate`: curl 會儘可能重複使用已建立的連線(multiplex)，可加速處理需求並減少負載，不過當 URL 是不同來源，那 multiplex 效果就不大。使用此選項會儘可能開啟新連線，把考慮 multiplex 的因子權重降低。本文的實例就很適合使用
  - `--parallel-max`: 預設是 50 個並行工作，可以調整自己適用的情境
:::

- 採用 `--parallel` 一次進行 10 個並行連線，此時指令為

```bash=
curl --parallel --parallel-immediate --parallel-max 10 -sku usr:pw 'https://10.50.[1-3].[1-10]/redfish/v1/Systems/1/bios' -o '10.50.#1.#2.bios.conf'
```

### 取得回應資訊
- 當處理的 URL request 數量多，難免會遇到對方 server 忙碌或當機，此時需要取得回應資訊判斷。可以透過 `--write-out` 取得 http 的 `response_code`，並且搭配印出 `url` 對照。此時指令改為
```bash=
curl --write-out 'code %{response_code} url %{url}\n' \
     --parallel --parallel-immediate --parallel-max 10 \
     -sku usr:pw \
     'https://10.50.[1-3].[1-10]/redfish/v1/Systems/1/bios' -o '10.50.#1.#2.bios.conf'
```
- 執行時可以看到如下資訊印出在螢幕。由於是並行處理，所以並不會照順序執行或完成。仔細看會注意到 `10.50.3.3` 的回應是 `000`，代表`10.50.3.3`有不預期狀況發生
```bash=
code 200 url https://10.50.1.1/redfish/v1/Systems/1/bios
code 200 url https://10.50.1.4/redfish/v1/Systems/1/bios
code 200 url https://10.50.1.9/redfish/v1/Systems/1/bios
... 略 ...
... 略 ...
... 略 ...
code 200 url https://10.50.3.10/redfish/v1/Systems/1/bios
code 200 url https://10.50.3.9/redfish/v1/Systems/1/bios
code 000 url https://10.50.3.3/redfish/v1/Systems/1/bios
```

### 讓 curl 更可靠

- 安全性: 上述例子透過 `-u` 後面加 `usr:pw` 指定帳號密碼，這種方式在 linux 系統上是非常不安全的行為，若你使用的電腦為多人登入使用，此時其他用戶下 `ps -ef|grep curl` 之類指令，即可看到你使用 curl 指定的參數(當然包含密碼)。此時務必參閱 curl [`--config`](https://everything.curl.dev/cmdline/configfile) 方式，至少把密碼包在檔案特定檔案
- 可靠性: 若需要透過 curl 指令提供穩定的服務，那勢必要考慮錯誤處理，當 timeout 或 fail 等不符合預期的回傳值，要怎麼處理。此時可以參考 [manpage](https://curl.se/docs/manpage.html) 關於 `retry` or `fail` 等關鍵字的 option，依據你的情況做適當處理。當然最快方式參考 github 上其他大神們腳本

## 後記

- curl 是系統管理老朋友，但以驚人速度持續更新，開發者在 7.66 release 的 [ blog 畫這張圖](https://daniel.haxx.se/blog/2019/09/11/curl-7-66-0-the-parallel-http-3-future-is-here/)，來表明 curl 功能以指數成長。在不久將來，我們勢必還可以期待，來自這位老朋友的新消息 :)
![](https://i.imgur.com/KTFofnW.png =x300)


## 參考連結
[Everything curl](https://everything.curl.dev): curl 開發者寫的 book，讚  
[curl 的 manpage](https://curl.se/docs/manpage.html): 網頁上查詢 option 好用  
[curl 的 globbing 範例](https://unix.stackexchange.com/a/91574): stackoverflow 網友的範例  
[curl 的 parallel 範例](https://stackoverflow.com/a/71967814): stackoverflow 網友的範例  



