# 維運 Linux 伺服器的陷阱 -  EDAC 競爭

>  現代伺服器，大多會搭載獨立的 BMC 系統。BMC 除了可遠端管理伺服器，也提供記憶體錯誤修正功能 (EDAC, Error Detection Ａnd Correction)。該功能取材自 Linux kernel module，此時若維運的作業系統，也是 Linux 系統，那相似的功能，可能會產生競爭  
>  本文連結:　https://hackmd.io/@kmo/edac_race 
::: warning
:warning: EDAC 競爭
- EDAC 會輪巡(poll)暫存器(register)，當讀到記憶體錯誤，會嘗試修正並"清除"紀錄
- 伺服器記憶體出現 CE(correctable errors) 紀錄，是非常常見的現象，且大部分不需要擔心。但如果相同的記憶體，不斷出現 CE 紀錄，通常也代表快要壞掉的徵兆
- BMC 系統可能和 Linux 作業系統，產生 EDAC 競爭。先讀到錯誤記憶體的一方，就會嘗試修復並清除紀錄，造成 2 邊都沒有完整的錯誤紀錄
- 大部分伺服器硬體供應商，通常僅依據 BMC 日誌的錯誤訊息，判斷是否要進行硬體更換
:::

::: info
:mag_right: 為了避免 EDAC 競爭產生的問題
- 通常伺服器供應商，會建議在 Linux 系統裡，禁用 EDAC 相關設定。僅讓 BMC 進行記憶體錯誤修正，讓 BMC 日誌收集完整錯誤紀錄
::: 

## 操作範例
::: warning
:memo: 更新參數前
- 建議查閱伺服器手冊，或詢問伺服器供應商，是否有相關議題說明
- 務必清楚維運的硬體、作業系統和使用情境，並從小規模測試開始
:::
- 以 CentOS 7 操作為例

```bash=
# 由於 CPU 型號不同，檢查 Linux kernel 載入的 EDAC module
# 這邊以 `xxx_edac` 為例
lsmod | grep -i edac

# 禁止開機載入 kernel module 方式很多，範例統一用 grubby 指令處理，
# 並僅對 default kernel 操作
grubby --update-kernel=DEFAULT --args='mce=ignore_ce modprobe.blacklist=xxx_edac'
# mce=ignore_ce -> machine check event 忽視 corrected errors
# modprobe.blacklist=xxx_edac -> 禁止載入 xxx_edac module

# 檢查是否有成功寫入 kernel cmd
grubby --info=DEFAULT

# 重開機就會生效
systemctl reboot

# 若需要移除新增的參數，可以進行以下操作，最後再重開機
grubby --update-kernel=DEFAULT --remove-args='mce=ignore_ce modprobe.blacklist=xxx_edac'
```

## 還可以做更多 - 作業系統啟用 `rasdaemon` 和 `ipmiseld`
- Rasdaemon is a RAS (Reliability, Availability and Serviceability) logging tool。  
  其中 `rasdaemon` 已經取代以前 `mcelog`   
  [RHEL7: Which of mcelog and rasdaemon should I use for monitoring of hardware, for which usecases?](https://access.redhat.com/solutions/1412953)
- Twitter 維運數十萬台伺服器經驗，在文章推薦大家啟用 `rasdaemon`，可以有一致方式收集硬體相關錯誤訊息  
  [How Twitter uses rasdaemon for hardware reliability](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2023/how-twitter-uses-rasdaemon-for-hardware-reliability)
- `rasdaemon` 可以記錄 MC (Memory Controller) events、MCE (Machine Check Exception) events 和 Disk Errors 等。其中需要依靠 EDAC 才能收集記憶體錯誤，而本篇文章因 EDAC 競爭關係，建議系統禁用 EDAC，但不會影響 `rasdaemon` 啟用收集其他硬體錯誤訊息
- 可以透過例如 [freeipmi](https://www.gnu.org/software/freeipmi) 提供的 `ipmiseld` 服務，在系統啟用此服務，就會把 BMC 日誌自動寫入系統 syslog 裡，來補足系統日誌缺少記憶體 CE 紀錄 
- 儘管作業系統禁用 EDAC，但系統透過啟用`rasdaemon` 和 `ipmiseld`，不管 BMC 日誌或作業系統日誌，都會完整收集硬體錯誤訊息 :+1:

## 各家廠商和相關說明文章
- ION Computer
  - [Linux EDAC modules on Server Systems](https://blog.ioncomputer.com/2019/05/06/linux-edac-modules-on-server-systems)
  - 美國 ION 伺服器廠商，在部落格深入淺出描述此議題，可能是目前網路上最清楚一篇。非常推薦大家閱讀。
- Lenovo
  - [Special consideration when using Linux error detection and correction (EDAC) for tracking memory errors - Lenovo Server](https://support.lenovo.com/us/en/solutions/ht107942)  
- Fujitsu
  - [[PY-CIB068-01] Linux OS - Memory error recorded in syslog](https://www.fujitsu.com/us/imagesgig5/PY-CIB068-01.pdf)
  - 補充說明。當作業系統安裝 ServerView Agents (srvmagt*.rpm)，啟用 `eecd.service` 就會自動卸載 EDAC module
- HP
  - [Notice: (Revision) Linux - To Ensure Efficient Firmware First Handling of Memory Failures HPE Recommends Booting With the mce=ignore_ce Boot Parameter in Addition to Disabling EDAC](https://support.hpe.com/hpesc/public/docDisplay?docId=emr_na-a00016026en_us)
- Dell
  - [EDAC Errors in 'messages' Log in RedHat Enterprise Linux (RHEL) and PowerEdge](https://www.dell.com/support/kbdoc/en-us/000177028)
- IBM
  - [Linux EDAC reports memory errors - IBM BladeCenter and System x](https://www.ibm.com/support/pages/node/853252)
- Redhat  
  - [EDAC vs vendor monitoring
](https://access.redhat.com/discussions/3545531)
  - [memory scrubbing error shows in Red Hat Enterprise Linux
](https://access.redhat.com/solutions/739063)
  - [Bug 501906 - EDAC of Redhat impact system BIOS and BMC function](https://bugzilla.redhat.com/show_bug.cgi?id=501906)  
    10 年前就有類似的議題
- SUSE
  - [Considerations for dealing with correctable memory error messages](https://www.suse.com/support/kb/doc/?id=000019052)
- SAP on Azure
  - https://learn.microsoft.com/en-us/azure/sap/large-instances/os-upgrade-hana-large-instance
- stackexchange 網友類似議題詢問
  - [How EDAC get error notification? from BIOS or memory controller?](https://unix.stackexchange.com/a/607315)
- Linux kernel 關於 `mce=ignore_ce` 描述
  - https://docs.kernel.org/arch/x86/x86_64/boot-options.html 
```
mce=ignore_ce
    Disable features for corrected errors, e.g. polling timer and CMCI.   
    All events reported as corrected are not cleared by OS and remained in its error banks.     
    Usually this disablement is not recommended,   
    however if there is an agent checking/clearing corrected errors (e.g. BIOS or hardware monitoring applications), 
    conflicting with OS's error handling, and you cannot deactivate the agent, then this option will be a help.
```


## 後記

Linux 系統維運工作 5 年，經手過 1000 台左右實體伺服器。其中一部分是國產伺服器，一部分是國外品牌伺服器。相關 EDAC 議題國外品牌伺服器，都有說明和對應處理方式。可惜國產伺服器廠，似乎還沒有類似公開說明文件。本文角度主要以追求穩定的系統維運，若以效能角度，系統關掉 EDAC 也有效能方面改善，比如 Redhat 文件提及可以改善反應時間 ([5.2. Improving response times by disabling error detection and correction units](https://access.redhat.com/documentation/zh-tw/red_hat_enterprise_linux_for_real_time/9/html/optimizing_rhel_9_for_real_time_for_low_latency_operation/setting-bios-parameters-for-system-tuning_optimizing-rhel9-for-real-time-for-low-latency-operation#proc_configuring-edac-units_setting-bios-parameters-for-system-tuning))

相關議題還有 BIOS 的 `Memory scrubbing` 設定，為了系統穩定通常預設開啟，但開啟就會影響些微效能，通常國外品牌廠都會有 BIOS 文件指引，看你使用情境作不同建議和調整。甚至如果是 HPC 等非常大 scale 的 cluster，也有文章討論頻繁 CE 會影響效能的討論 ([Understanding the Effects of DRAM Correctable
Error Logging at Scale](https://www.osti.gov/servlets/purl/1881688))

我們身處高速變化時代，上述文章可能很快就過時，也歡迎大家不吝給予建議!

---
{%hackmd @kmo/widget_license %}