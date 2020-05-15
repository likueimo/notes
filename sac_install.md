# SAC (Seismic Analysis Code) 安裝說明

>> 目標: 在自己目錄解壓縮 sac，並把 sac 初始化腳本加到環境變數。

- 1. 家目錄下創立資料夾，資料夾名稱 sac-101.61
`mkdir -p ~/sac-101.6a`
- 2. 解壓縮至 ~/sac-101.6a
`tar -xzvf sac-101.6a-linux_x86_64.tar.gz -C ~/sac-101.6a`
- 3. sac 有提供環境變數初始化腳本，但他腳本內的 `SACHOME` 路徑要改為 sac 安裝路徑。
底下指令會把初始化腳本(`~/sac-101.6a/sac/bin/sacinit.sh`)內的 `SACHOME`路徑， 從`/usr/loca/sac`更改為`~/sac-101.6a/sac`。第二行指令幫他 `PATH`變數輪轉位移一下。
```
sed -i 's#SACHOME=/usr/local/sac#SACHOME=~/sac-101.6a/sac#' ~/sac-101.6a/sac/bin/sacinit.sh
sed -i 's#PATH=${PATH}:${SACHOME}/bin#PATH=${SACHOME}/bin:${PATH}#' ~/sac-101.6a/sac/bin/sacinit.sh
```
- 4. 把初始化腳本加到環境變數，登入時會自動執行
`echo "source ~/sac-101.6a/sac/bin/sacinit.sh" >> ~/.bashrc`

- 5. 登出再登入系統，這時候打 sac 就會有反應。如有錯誤再看缺什麼套件
比如出現此錯誤，代表缺 library `libSM.so.6`，再找來安裝。
![](https://i.imgur.com/0ZftOfA.png)


- 成功畫面

![](https://i.imgur.com/ckjjpDw.png)


