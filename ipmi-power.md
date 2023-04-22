# 使用 IPMI 取得和變更 server 電源狀態
- 使用 [`freeipmi`](https://www.gnu.org/software/freeipmi/) 套件提供的 `ipmi-power` 指令為範例
- 文章中若是在 terminal 下，進行 Linux 指令操作，會在指令前加 `$` 
- 本文環境: [WSL2](https://aka.ms/wsl2-install)(Ubuntu)，`freeipmi` 版本為 `1.6.4`
- share link: https://hackmd.io/@kmo/ipmi-power
## 前置作業
- 安裝 `freeipmi`:  
  - 主流 linux distribution 都有提供 `freeipmi` 套件，用 package manager 安裝即可  
  - 推薦被管理的 server 也都在 OS 上安裝
  ```bash=
  # Debian/Ubuntu 系列
  $ apt install freeipmi-tools
  
  # RHEL/CentOS 系列
  $ yum install freeipmi
  ```
- 設定 `/etc/freeipmi/freeipmi.conf`:  
  - 同 cluster 機器，通常 IPMI 帳號密碼會有相同一組  
    此時建議把帳密設定在 `freeipmi.conf`，省去每次下指令都要指定帳密  
    以及避免下指令時，被系統 log 帳密  
  ```bash=
  # modify /etc/freeipmi/freeipmi.conf
  username $your_ipmi_username
  password $your_ipmi_password
  ```
  
## 操作
::: info 
:scroll: 情境說明  
現在有 3 台 server，並在 `/etc/hosts` 設定好對應 IP 和 hostname
```bash=
10.50.1.1 computer01-ipmi
10.50.1.2 computer02-ipmi
10.50.1.3 computer03-ipmi
```

- `ipmi-power` 常用的 option 

| option(short) | option(long) | 說明                            |
| ------------- | ------------ | ------------------------------- |
| `-s`          | `--stat`     | Get power status                |
| `-n`          | `--on `      | Power on                        |
| `-f`          | `--off`      | Power off                       |
| `-c`          | `--cycle`    | Power 先 off 再 on              |
| `-r`          | `--reset`    | Power 持續供應下重啟 OS         |
| `-h`          | `--hostname` | 可用 IP 或 hostname 指定 server |

- `--off`: 通常是正常關機卡住，才會用到 ipmi 關機。(正常 Linux 系統關機，是在 OS 下指令 `systemctl poweroff`)
- `--cycle` : 如果硬體有異常，或是 BMC(Baseboard Management Controller) 有當機情況，可以嘗試`--cycle`看能不能復原
- `--reset` : 通常用在 OS 當機後需要重啟 OS。類似在舊款 Win 系統下 `Ctrl` + `Alt` + `Del` 重開機
- `--hostname`: freeipmi 特色可指定連續 IP 或 hostname 範圍，可以參考下面範例
- 補充說明: 主流 Linux 系統皆已採用 systemd，因此在 OS 正常關機和重開機指令如下

| 說明   | 指令               |
| ------ | ------------------ |
| 關機   | systemctl poweroff |
| 重開機 | systemctl reboot   |

:::

### Power status
::: warning
:dart: 目標
- 使用 `ipmi-power`，用 1 行指令取得 3 台 server 的電源狀態
:::

```bash=
# 指定遠端 server 通常用 IP，而 freeipmi 特色，就是一次指定連續 IP 範圍
$ ipmi-power -s -h 10.50.1.[1-3]

# freeipmi 採用並行處理，所以輸出結果不會按順序排序，結果如下
# 看出 10.50.1.1 為 Power off 狀態，其他為 Power on 狀態
10.50.1.2: on
10.50.1.1: off
10.50.1.3: on

# 已經在 /etc/hosts 設定好對應 hostname，也可以指定連續 hostname 範圍
$ ipmi-power -s -h computer[01-03]-ipmi

# 輸出結果如下，可以看到輸出結果，顯示名稱為 hostname
computer03-ipmi: on
computer01-ipmi: off
computer02-ipmi: on
```

### Power on
::: warning
:dart: 目標
- 使用 `ipmi-power`，用 1 行指令確保 3 台 server 都是 Power on 狀態
:::

```bash=
# 下述指令都可以做到一樣事情
# 為了確保 Power on，通常會一次指定範圍內節點，已經 Power on 狀態不受影響
$ ipmi-power --on -h computer[01-03]-ipmi

# 輸出結果如下，有正常回應指令就會顯示 ok
computer03-ipmi: ok
computer01-ipmi: ok
computer02-ipmi: ok
```

### Power off/reset
::: info 
:scroll: 情境說明   
computer03 在 OS 正常關機(或是重開機)，但過了很久還是保持 Power on 狀態  
此時用 BMC 的 KVM console 查看，才發現卡在關機過程

::: 
::: warning
:dart: 目標
- 使用 `ipmi-power`，強制關機(或是重開機)
:::

```bash=
# 先確認 Power status，此時會發現是 Power on 狀態
$ ipmi-power -s -h computer03-ipmi

# 再利用 BMC 的 KVM console 的畫面確認
# 確認卡在關機過程，此時強制關機或強制重開機
$ ipmi-power --off -h computer03-ipmi
## or
$ ipmi-power --reset -h computer03-ipmi
```

### Power cycle
::: info 
:scroll: 情境說明   
computer03 的 BMC 當機，使用遠端 IPMI 指令都沒反應  
幸運的是，還能進去 computer03 的 OS，在 OS 下 IPMI 指令還有反應  

正常 SOP 會是在 OS 用 IPMI 指令，對 BMC 做 cold reset，大部份就會正常  
而少數異常需要做到 Power cycle 才正常  
用 Power cycle 去試著修復 BMC 當機，比較像是民俗療法，碰碰運氣看看  
:::

::: warning
:dart: 目標
- 在 computer03 的 OS 下，使用 IPMI 指令針對 BMC 做 cold reset
- 上述動作若無效，試著在 OS 下用 `ipmi-power` 進行 Power cycle
:::

```bash=
# 進到 computer03 的 OS
$ ssh computer03

# 在 computer03 的 OS 下已經裝好 freeipmi
# 使用尚未提到 bmc-device 指令，進行 BMC cold reset
$ bmc-device --cold-reset

# 上述指令等待 3 分鐘後，遠端 IPMI 還是沒反應
# 此時試著在 OS 下，用 Power cycle 試試看
$ ipmi-power --cycle
```


---
{%hackmd @kmo/widget_license %}