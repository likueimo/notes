---
tags: notes, shell, scripts, bash
---

# Shell script compiler (shc) 是不可靠的

> 一直到今天還有人深信，透過 shc 就能"加密" shell script  
> 特別寫一篇來澄清這個觀念  

> 本文連結: https://hackmd.io/@kmo/decrypt_shc  
> 任何回饋歡迎留言在此篇 hackmd，或直接登入 hackmd 後修改本文內容 :)  

## shc 是什麼?
- 能把 shell 腳本轉換成 binary 的"形式"
- 舉例`foo.sh` 轉換成 `foo.sh.x`，此時用編輯器打開 `foo.sh.x` 看到就是亂碼
- shc 專案頁面: https://github.com/neurobin/shc

## 如何破解?
- 畢竟還是得在 shell 層執行，換個"形式"，但依然還是腳本 
- 只需 bash 內建環境變數`SHELLOPTS`，指定 `verbose`，接著執行 `foo.sh.x` 即可看到腳本內容 print 出來
```bash=
env SHELLOPTS=verbose ./foo.sh.x
```
- ref: https://unix.stackexchange.com/a/64765

## 那我該怎麼做?
- 若系統管理員，不希望一般 user 看到 script 的內容，可考慮朝 `/etc/sudoers.d/` 這邊發展(之後再補一篇來舉例)
- 若是廠商交付給客戶，不希望客戶看到 script 的內容，可能要改用其他真的能 compile 的語言，或是透過合約之類，嚴格規範 script 的使用範圍限制

---
[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] This work is licensed under a [CC BY-NC-SA 4.0][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png