---
tags: notes, timer, systemd
---

# 使用 systemd timer 取代 crontab
- 現代主流的 Linux 系統，預設使用 systemd 做系統管理
- systemd timer 可取代 crontab，較易 debug 和操作，重要的工作推薦花時間改寫
- 避免文章過於複雜，簡化或省略詳細的功能描述
- 本文假設用戶有 root 權限
- 本文連結 : https://hackmd.io/@kmo/notes_systemd_timer

## systemd timer 優點
::: spoiler 
- 較容易 debug
- 當上一時間步 `foo.service` 還處於 `activating` 狀態，下一時間步到了也不會重複執行 `foo.service`，可以避免重複執行造成 process 壘加，導致系統異常
  - 舉例來說:
  每 5 分鐘執行一次 `foo.service`，當 09 點 10 分到了，結果 09 點 05 分的 `foo.service` 還在執行中，他判斷 `foo.service` 處於`activating` 狀態，就不會啟動 `foo.service`  
- systemd timer 可精確到秒，以秒執行，而 crontab 較難以實現。比如 30 秒執行一次腳本，使用 systemd timer 可輕易實現
- 可透過 `foo.service` 承接 Linux 的功能，比如限制使用的 CPU 和記憶體等
:::

## systemd timer 缺點
::: spoiler 
- 相對於 crontab，使用 systemd timer 需要撰寫 2 個檔案
- 學習曲線相對高。需要對 Linux 系統較高的熟悉度，才會較好理解，也比較不容易出錯
:::

## systemd 基本介紹
::: spoiler 
- systemd 常見於系統以下 2 位置

```bash=
# 用戶自定義檔案，也是接下來會用到路徑
/etc/systemd/system/*

# 通常為系統安裝套件預設放置位置 (ex: deb, rpm...)
/usr/lib/systemd/system/*
```
- systemctl 基本指令
```bash=
# 立即執行 or 停止指定的 service
systemctl {start,stop} foo.service

# 啟用 or 取消開機時會自動執行指定的 service
systemctl {enable,disable} foo.service

# 重啟指定的 service
systemctl restart foo.service

# 當有編輯 systemd，記得 reload
systemctl daemon-reload
```
- systemctl 操作以上指令，會記錄到作業系統的 syslog，如 CentOS 7 作業系統的 `/var/log/messages`
- 當使用 `systemctl` 指令操作時，若沒有加結尾，預設你是指 `.service`
  舉例如下，以下是相同指令
```bash=
systemctl status foo
systemctl status foo.service
```

- 其他結尾有特定功能。例如本文介紹 `*.timer`，`*.timer` 會去執行和自己同樣名稱的 `*.service`。如 `foo.timer` 時間到會去執行`foo.service`
:::


## systemd timer 範例

- 為最簡單範例，只留下必要設定
### 情境 
- 每 5 分鐘執行一次腳本 `/opt/foo.sh`
### 撰寫 systemd 的檔案
::: spoiler 
- 分別撰寫 2 個檔案，`foo.service` 和 `foo.timer` 放底下指定路徑 
- service 功能 : 定義執行什麼工作 (what)
- timer 功能 : 定義何時執行 (when)
- `/etc/systemd/system/foo.service`
```bash=
[Service]
Type=oneshot
ExecStart=/opt/foo.sh

[Install]
WantedBy=multi-user.target
```
- `/etc/systemd/system/foo.timer`
```bash=
[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
AccuracySec=1min

[Install]
WantedBy=timers.target
```
- 當有編輯 systemd 記得 reload
```bash=
systemctl daemon-reload
```

:::

### systemd 檔案參數說明
::: spoiler
- `/etc/systemd/system/foo.service`
```bash=
Type=oneshot #oneshot 是指執行一次
ExecStart=/opt/foo.sh #這邊指定你要執行的腳本路徑，權限要可執行
```

- `/etc/systemd/system/foo.timer`
```bash=
OnBootSec=5min #開機後 5 分鐘執行
OnUnitActiveSec=5min #每 5 分鐘執行一次
AccuracySec=1min #精確到分鐘
```


:::

### 測試及啟用 systemd 
::: spoiler 
- 主要腳本是透過 service 執行，先執行一次 `foo.service`，確認能成功執行
```bash=
# start
systemctl start foo.service
# check status
systemctl status foo.service
```
- 執行 `foo.timer`，當設定每 5 分鐘時間到，`foo.timer` 會自動執行 `foo.service`
```bash=
# start
systemctl start foo.timer
```
- 列出 timer 並查看 `foo.timer` 執行的狀態 (很重要的功能)
```bash=
systemctl list-timers
```
- 確認符合預期後，啟用讓他開機能自動執行
```bash=
systemctl enable foo.timer
```

:::

## 後記
::: spoiler 
- 由於沒有詳盡到每個參數解釋，網路上也已經有很多不同詳細中文/英文資料說明 systemd，當有不懂推薦可以先 google 關鍵字查看看 :)
:::


## 參考連結
::: spoiler 
- [ ubuntu 論壇網友回覆](https://askubuntu.com/a/1083647): 本文參考此網友說明的概念
- [arch linux 的 wiki](https://wiki.archlinux.org/title/systemd): 針對 systemd 說明的非常簡潔清楚
- [systemd unit 的官方文件](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)
- [systemd timer 的官方文件](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
:::

---
{%hackmd @kmo/widget_license %}