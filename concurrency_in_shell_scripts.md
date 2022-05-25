# Shell script 的並行化 (Concurrency)
 
> 本文目標是讓初學腳本的新手，較易入門學會腳本並行化  
> 設計 2 個小試身手，加上 1 個最近我用到的實際使用情境  
> 共 3 個例子來熟悉並行化操作，每個例子都有 2 種作法實現  
> 小試身手內容非常簡易，可以熟練，讓未來使用腳本的工作更有效率  
>
> 本文連結: https://hackmd.io/@kmo/concurrency_in_shell_scripts  
> 任何回饋歡迎留言在此篇 hackmd，或直接登入 hackmd 後修改本文內容 :)  
>
> ps: 可能是因為 GNU Parallel 的關係  
> 在腳本領域，有些人習慣會稱做平行化 (parallel)  
> 但精準用法應該要用並行化 (concurrency) 來敘述  

[TOC]

::: success
:memo: 本文範例都有 2 種實踐方法，方法分別如下
- GNU Parallel: 
  GNU Parallel 由 Perl 語言撰寫的指令，可以用一行指令實踐並行化
  - 優點: 已發展 20 年，功能強大，不限於 bash 內使用，可呼叫 cmd 腳本或程式都能使用
  - 優點: 大部分的 for loop 工作，通常只需要一行 parallel 指令即可完成
  - 缺點: 由 Perl 語言撰寫，需安裝套件才能使用，有些特殊環境可能不易安裝套件 
  - [安裝 GNU Parallel 筆記](https://hackmd.io/@kmo/notes_gnu_parallel_install)
- Bash built-in 方法: 
  透過 `()`, `&` 和 `wait` 實踐並行化
  - 優點: 只需要純 bash 語法達成，不需依賴外部套件完成
  - 優點: 撰寫邏輯和原本沒有並行化的腳本，幾乎一樣，較容易改寫
  - 缺點: 當迴圈越多，或加越多功能，腳本會變較難以直覺閱讀和理解

:::

::: info 
:bulb: Hint:  
在 Windows 系統，可以啟用 WSL2 安裝 Ubuntu 容器，來使用 Shell 腳本 or GNU Parallel   
也就是說，在 Windows 系統底下，也能輕易並行化處理工作
:::

> 情境假設電腦有 4 cores，通常並行的 process 要等於小於電腦的 core 數  
> 所以底下都用 4 個並行 process 為例子，可以根據你自己環境，修改數字測試  

## 小試身手 - 單一迴圈
:::info 
:scroll: 情境說明:
- 印出 1..100 的數字，每次處理 4 個數字
:::

### GNU Parallel 方法
::: spoiler 
- 透過 `{}` 可讀取 `:::` 右邊的 input，並透過 `--jobs` 指定同時執行 4 jobs
- `{1..100}` 是 bash 語法，可產出連續數字
- 這個例子主要是介紹指令 parallel 語法，實際執行會非常快，快到無法分辨每次執行 4 個 jobs XD 
```bash=
parallel --jobs 4 "printf 'NUMB: %s\n' {}" ::: {1..100}
```
:::
   
### Bash built-in 方法 
::: spoiler

```bash=
#!/bin/bash

# 指定要並行處理的 process 數量，
# 假設電腦有 4 Cores，通常並行的 process 要等於小於電腦的 core 數
n_process=4

# 用 bash 語法 {}，產出連續數字，依序進到迴圈
for ii in {1..100}; do
  #透過 () 包住要執行指令，並且用 & 背景執行，達到並行處理
  (
    printf 'NUMB: %s\n' "$ii"
  ) &
  # 數字除以指定 process 數，當餘數為 0，使用 wait 等待所有 child process 完成
  remainder=$(( ii % n_process ))
  if [ "$remainder" -eq 0 ]; then
    # 這行 printf 方便確認 wait 是在哪個數字時啟用，當需要確認時可以打開註解
    #printf 'WE ARE WAITING AT NUMB: %s\n' "$ii"
    wait
    # 每次印出數字很快，可以透過 sleep 達到暫停效果，來確認同時印出 4 個數字
    # 實際使用應該註解掉
    sleep 1
  fi
done
```
:::


## 小試身手 - 巢狀迴圈 (Nested loop)
:::info 
:scroll: 情境說明:
- 印出 1-1, 1-2, 1-3 ... 1-39, 1-40，共 40 個數字
- 印出 2-1, 2-2, 2-3 ... 2-39, 2-40，共 40 個數字
- 印出 3-1, 3-2, 3-3 ... 3-39, 3-40，共 40 個數字
- 每次處理 4 個數字
::: 


### GNU Parallel 方法
::: spoiler
- 透過 `{1}` 讀取第一個 `:::` 右邊的 input，透過 `{2}` 讀取第二個 `:::` 右邊的 input，並透過 `--jobs` 指定同時執行 4 jobs
```bash=
parallel --jobs 4 "printf 'NUMB: %s-%s\n' {1} {2}" ::: {1..3} ::: {1..40}
```
:::
  
### Bash built-in 方法 
::: spoiler
```bash=
#!/bin/bash

# 指定要並行處理的 process 數量，
# 假設電腦有 4 Cores，通常並行的 process 要等於小於電腦的 core 數
n_process=4

# 用 bash 語法 {}，產出連續數字，依序進到迴圈
for ii in {1..3}; do
  for jj in {1..40}; do
  (
    printf 'NUMB: %s-%s\n' "$ii" "$jj"
  ) &
  # 數字除以指定 process 數，當餘數為 0，使用 wait 等待所有 child process 完成
  remainder=$(( jj % n_process ))
  if [ "$remainder" -eq 0 ]; then
    # 這行 printf 方便確認 wait 是在哪個數字時啟用，當需要確認時可以打開註解
    #printf 'WE ARE WAITING AT NUMB: %s-%s\n' "$ii" "$jj"
    wait
    # 每次印出數字很快，可以透過 sleep 達到暫停效果，來確認同時印出 4 個數字
    # 實際使用應該註解掉
    sleep 1
  fi
  done
done
```
:::



## 實際使用情境
:::info
:scroll: 情境說明
- 管理的節點 BMC IP，分 3 個 group，共 120 台節點
- 使用 redfish api，備份及更新 BIOS 設定檔，關掉 Hyper-Threading
```bash=
#節點 node01[01-40] (node0101,node0102,...,node0140) 的 IP 範圍
10.50.1.[1-40]
#節點 node02[01-40] (node0201,node0202,...,node0240) 的 IP 範圍
10.50.2.[1-40] 
#節點 node03[01-40] (node0301,node0302,...,node0340) 的 IP 範圍
10.50.3.[1-40] 
```
- 透過 redfish api 取得單一節點的 BIOS 設定指令範例
```bash=
curl -sku usr:pw 'https://10.50.1.1/redfish/v1/Systems/1/bios'
```
- 透過 redfish api 更新單一節點的 BIOS 設定指令範例
```bash=
curl -sku usr:pw -H 'Content-Type: application/json' -X PATCH -d @update_bios.json 'https://10.50.1.1/redfish/v1/Systems/1/bios/settings'
```
- update_bios.json 是一個 json 格式的檔案，內容如下
```json=
{
  "Attributes":{
    "Hyper-Threading":"Disabled",
  }
}
```
:::

:::warning
:warning: 注意:
- 本文 redfish api 指令範例是參考 [redfish 官方範例](https://www.dmtf.org/sites/default/files/Redfish_School-BIOS_Configuration.pdf)
- 不同硬體廠牌的 api 都略有差異，實際應用務必查詢自己機器的 redfish 說明文件，或是詢問硬體 vendor
- 更新 BIOS 設定的部分，部分廠牌機器需要先用 GET 取得 etag 值，在 PATCH 時候 header 指定 etag 值，來確保 PACTH 正確
:::


### GNU Parallel (2 個作法)

::: spoiler
- 方法 1，較為單純用法
- 使用 curl 指令備份 BIOS，指定 `-o` 產出檔案 (檔名如 10.50.1.1.bios.conf, 10.50.1.2.bios.conf 以此類推)
```bash=
# 備份 BIOS 設定檔
parallel --jobs 4 --progress curl -sku usr:pw https://10.50.{1}.{2}/redfish/v1/Systems/1/bios -o 10.50.{1}.{2}.bios.conf ::: {1..3} ::: {1..40}

# 更新 BIOS 設定檔
parallel --jobs 4 --progress curl -sku usr:pw -H 'Content-Type: application/json' -X PATCH -d @update_bios.json https://10.50.{1}.{2}/redfish/v1/Systems/1/bios/settings ::: {1..3} ::: {1..40}
```
- 方法 2，搭配 bash 的 function 用法
- 好處是可以簡化複雜的操作，但搭配 function 語法要改，比如從 `{1}` 改成 `$1`
```bash=
#!/bin/bash -eu

backup_bios_setting(){
printf 'downloading 10.50.%s.%s\n' "$1" "$2"
  curl -sku usr:pw \
  "https://10.50.$1.$2/redfish/v1/Systems/1/bios" -o "10.50.$1.$2.bios.conf"
}

update_bios_setting(){
printf 'updating 10.50.%s.%s\n' "$1" "$2"
curl -sku usr:pw \
  -H 'Content-Type: application/json' \
  -X PATCH \
  -d @update_bios.json \
 "https://10.50.$1.$2/redfish/v1/Systems/1/bios/settings"
}

export -f backup_bios_setting
export -f update_bios_setting
parallel --jobs 4 --progress backup_bios_setting ::: {1..3} ::: {1..40}
parallel --jobs 4 --progress update_bios_setting ::: {1..3} ::: {1..40}
```
:::


### Bash built-in 方法 

::: spoiler

```bash=
#!/bin/bash

backup_bios_setting(){
printf 'downloading 10.50.%s.%s\n' "$ii" "$jj"
  curl -sku usr:pw \
  "https://10.50.$ii.$jj/redfish/v1/Systems/1/bios" -o "10.50.$ii.$jj.bios.conf"
}

update_bios_setting(){
printf 'updating 10.50.%s.%s\n' "$ii" "$jj"
curl -sku usr:pw \
  -H 'Content-Type: application/json' \
  -X PATCH \
  -d @update_bios.json \
 "https://10.50.$ii.$jj/redfish/v1/Systems/1/bios/settings"
}

main_function(){
# 指定要並行處理的 process 數量，
# 假設電腦有 4 Cores，通常並行的 process 要等於小於電腦的 core 數
n_process=4

for ii in {1..3} ; do
  for jj in {1..40}; do
  (
  backup_bios_setting
  update_bios_setting
  ) &
  # 數字除以指定 process 數，當餘數為 0，使用 wait 等待所有 child process 完成
  remainder=$(( jj % n_process ))
  if [ "$remainder" -eq 0 ]; then
    # 這行 printf 方便確認 wait 是在哪個數字時啟用，當需要確認時可以打開註解
    #printf 'WE ARE WAITING AT 10.50.%s.%s\n' "$ii" "$jj"
    wait
    # 每次印出數字很快，可以透過 sleep 達到暫停效果，來確認同時印出 4 個數字
    # 實際使用應該註解掉
    sleep 1
  fi
  done
done
}

main_function
```
:::


## 補充 GNU Parallel 常用的 options
::: spoiler
- `--progress`: 顯示執行進度，非常實用
- `--joblog 指定檔名.log`: 產出每次 job 執行完的 exit code 等，非常實用 
- `--keep-order`: 每次 jobs 不一定依照順序執行完成，可以透過此選項依照順序
:::

## 補充 Curl 常用的 options
::: spoiler
- `-s`: silent 
- `-k`: `--insecure`，若遇到憑證問題，可以先指定 `-k` 試試
- `-o`: 讓 curl 指令，產出結果存成指定檔案
- `-w`: 讓 curl 指令執行結束時，印出資訊出來，比如 `-w return code %{http_code}`，印出 http 的回傳 code
:::

## 參考資料
::: spoiler
- https://unix.stackexchange.com/a/423398
- https://unix.stackexchange.com/questions/103920/parallelize-a-bash-for-loop
- https://stackoverflow.com/a/1050257
- https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs#comment730533_306115
:::

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png