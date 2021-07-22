---
tags: notes, timer, systemd
---

# 使用 systemd 取代 crontab
- 當代主流的 Linux 作業系統 (CentOS、Ubuntu 等) ，預設皆使用到 systemd 做系統控制
- systemd timer 可取代 crontab，較易 debug 和操作，推薦重要的工作可以花時間改寫
- 為了避免文章過於複雜，簡化或省略部分功能描述
- 本文假設用戶有 root 權限
- 本文連結 : https://hackmd.io/@kmo/notes_systemd_timer

## systemd timer 好處
::: spoiler 
- 除了較容易 debug 之外，個人欣賞 systemd timer 好處，當上一時間步 cron job 還沒執行完，下一時間步到了也不會重複執行，避免系統同時執行卡死
- 提及一個較少人注意功能，systemd timer 可精確到秒，以秒執行，而 crontab 較難以實現。比如 30 秒執行一次腳本， systemd timer 可輕易實現
- 可透過 `.service` 承接 Linux 的功能(進階用戶)
:::

## systemd timer 壞處
::: spoiler 
- 相對於 crontab，使用 systemd timer 需要撰寫 2 個檔案
- 學習曲線相對高，且需要對 Linux 系統相對高的熟悉度，才會比較好操作或理解
:::

## systemd 基本介紹
::: spoiler 
- systemd 常見於系統以下 2 位置

```bash=
# 用戶自定義檔案
/etc/systemd/system/*

# 通常為系統安裝套件預設放置位置 (ex: deb, rpm...)
/usr/lib/systemd/system/*
```
- systemctl 基本指令
```bash=
# 立即執行指定的 service
systemctl start foo.service

# 開機時會自動執行指定的 service
systemctl enable foo.service

# 重啟指定的 service
systemctl restart foo.service

# 停止指定的 service
systemclt stop foo.service
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
:::

### systemd 檔案參數說明
::: spoiler
- `/etc/systemd/system/foo.timer`
```bash=
OnBootSec=5min #開機後 5 分鐘執行
OnUnitActiveSec=5min #每 5 分鐘執行一次
AccuracySec=1min #精確到分鐘
```


:::

### 測試及啟用 systemd 
::: spoiler 
- 執行一次 `foo.service`，確認能成功執行
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
- 由於沒有詳盡到每個參數解釋，網路上也有很多不同詳細中文/英文資料，當有不懂推薦可以 google 關鍵字查看
:::


## ref
::: spoiler 
- https://wiki.archlinux.org/title/systemd
- https://www.freedesktop.org/software/systemd/man/systemd.unit.html
:::


---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png