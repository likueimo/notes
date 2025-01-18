# 解決 Microsoft Teams 2603 錯誤

## 問題描述
- Microsoft Teams 是許多公司用於開會和內部聯繫的工具。然而更改密碼後，重新登入可能會遇到 2603 錯誤。  
 - 錯誤訊息出現:`沒有網路連線`。`請檢察您的網路設定, 然後再試一次. [2603]`，如下圖所示：  
 - ![2603 錯誤訊息](https://i.postimg.cc/3xcGdznF/image.png)
## 解決方法
當遇到此錯誤時，且確定網路沒有問題，可以試著以下步驟
- 使用瀏覽器，開啟 [Microsoft Teams 網頁版](https://teams.microsoft.com) 並登入，確認可以正常使用 Teams 網頁版
- 回到 Microsoft Teams 桌面應用程式，重新登入，通常 2603 錯誤就不會發生

## 錯誤說明
- 推測錯誤和憑證(oauth)有關，猜測網頁正常登入 Teams 後，能把相關資訊更新，解決異常
- 除了 Teams 之外，有些 Microsoft 產品也會發生 2603 錯誤，也許也能透過類似方式解決異常

## 補充說明
- 可參考 [Use Microsoft Teams on the web
 說明頁面](https://support.microsoft.com/en-us/office/use-microsoft-teams-on-the-web-33f84aa9-2e8b-47ac-8676-02033454e385)，取得最新 Teams 網頁版網址
- 上述步驟是個人遇到的 Teams 異常紀錄，根據經驗應適用大部分情況。若問題持續發生，建議尋求公司的 IT 部門協助

---
{%hackmd @kmo/widget_license %}