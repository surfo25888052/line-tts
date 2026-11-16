# 里民意見表｜GitHub Pages + Google Sheets + GAS + LINE

**功能僅三件事：** 填寫意見、存入試算表、由 LINE 官方帳號回覆原填寫者。無管理後台、帳密系統、額外伺服器、群組轉發或案件狀態流程。

## 專案檔案

- `index.html`：公開的行動版里民意見表（姓名、電話、地址、意見）。
- `config.js`：填入你的 LIFF ID 與 GAS Web App URL；不放任何密鑰。
- `Code.gs`：獨立 GAS 後端程式由本次交付的 ZIP 提供，請在新建 Google 試算表的 Apps Script 內貼上；本公開 GitHub Pages 目錄只放前端。
- `tests/test.cjs`：由本次交付 ZIP 提供的離線模擬測試。

## 收件與回覆

**建議入口（LINE 官方帳號聊天視窗）：** 讓里民從 `https://liff.line.me/你的LIFF_ID` 開啟此 GitHub Pages 頁面。這個 LIFF 連結會顯示同一個 GitHub 頁面，啟用 `chat_message.write` 後，`liff.sendMessages()` 會以里民本人身份，把完整意見直接發到官方帳號聊天室。GAS 收到該文字 Webhook 後，記錄試算表，並以 Reply API 回覆受理編號。管理員在原本的 LINE 聊天室直接看到是誰反映、反映了什麼。

**一般 GitHub Pages 網址：** 也能填寫。LIFF 登入後以表單 POST 送至 GAS；GAS 向 LINE 取得正確的 User ID、寫入試算表，再以 Push API 推播給本人。這種情況不是「里民傳訊到官方帳號聊天室」，因此如果管理員要在 LINE 立即收到通知，請設定 `ADMIN_TO`（管理員 User ID 或官方帳號所在群組 ID）。

> **重要：** 若你希望里民的完整意見直接出現在官方帳號既有聊天室，選單應設定 **LIFF 網址**，而不是原始 GitHub Pages 網址。LIFF 的 Endpoint URL 仍然是這份 GitHub Pages 網址。LINE 官方文件規定，只有從聊天開啟的 LIFF 瀏覽器才可以使用 `liff.sendMessages()`。

## 按照你的部署順序設定

1. **Google Drive → 新增 → Google 試算表**。建立一份空白表，選「擴充功能（Extensions）→ Apps Script」，刪除預設內容，把 `Code.gs` 全部貼入。若你建立的是 `.xlsx` 檔，請先轉為**原生 Google 試算表**，以便綁定 GAS。
2. 在 GAS 的「專案設定（Project Settings）→ 指令碼屬性（Script Properties）」新增 `LINE_CHANNEL_ACCESS_TOKEN`，內容是官方帳號 Messaging API 的 Channel Access Token。不要把 Token 放在 GitHub。若需要把一般 GitHub 網址表單也通知給管理員，再新增 `ADMIN_TO`，值為管理員的 LINE User ID 或 LINE 群組 ID。**這兩個都是私密設定**。
3. 在 GAS 編輯器手動執行 `setupSheet()` 一次並允許權限。系統會建立 `里民意見` 工作表及標題欄（也可在第一筆提交時自動建立）。
4. 選「部署（Deploy）→ 新增部署（New deployment）→ 網頁應用程式（Web app）」，`Execute as`（執行身分）選 **Me（我）**，`Who has access`（存取權限）選 **Anyone（所有人）**；複製 `/exec` 網址。
5. 在 LINE Developers 的**與官方帳號同一個 Provider（提供者）**下建立 LINE Login Channel，在 LIFF 分頁建立 LIFF App。`Endpoint URL` 設成 GitHub Pages 表單網址；Scopes 勾選 `profile` 與 `chat_message.write`，網頁開啟方式建議 Full（全螢幕）。取得 LIFF ID。
6. 編輯 GitHub Pages 目錄下的 `config.js`：`liffId` 改成真實 LIFF ID，`gasUrl` 改為步驟 4 的 `/exec` 網址。GitHub Pages 發布後，確認 `/resident-feedback/` 可開啟。
7. 到 LINE Developers → Messaging API → Webhook settings（Webhook 設定），貼入同一支 GAS `/exec` 網址並開啟 Use webhook（使用 Webhook）；在官方帳號選單或圖文選單加入 **LIFF 網址**：`https://liff.line.me/你的LIFF_ID`。若只接受一般 GitHub 頁面連結也可，但記得 `ADMIN_TO`。
8. 用**測試 LINE 帳號**從官方帳號聊天室開啟表單，填寫測試意見送出。驗證（a）原聊天室出現填寫者自己傳出的意見，（b）原聊天室收到官方帳號受理回覆，（c）Google 試算表新增一列、LINE User ID 與表單編號一致。

### 欄位

`填表時間｜送出方式｜表單編號｜LINE使用者ID｜LINE顯示名稱｜姓名｜聯絡電話｜地址｜反映意見｜LINE回覆狀態`

### 不要誤切舊 Webhook

LINE 一個 Messaging API Channel 同時間只有一個 Webhook URL。將網址改成這份 GAS 後，原本 `LINE_to_Sheet` 及群組轉發 GAS **不再收到新的 Webhook**。本版本只處理里民意見；若仍需舊群組轉發，不能直接替換而不整合。

### 限制

- LINE 推播需對方與官方帳號有可接收訊息的關係；對方封鎖官方帳號時可能收不到 Push。
- GAS `doPost(e)` 不提供 HTTP Request Headers，所以此極簡版不能核驗 `x-line-signature`。有需要可在 Script Properties 設 `WEBHOOK_KEY`，並在 LINE Webhook URL 使用 `/exec?key=該值`，至少減少公開端點遭濫用的風險；但此方法不等於 LINE 簽章驗證。
- 使用者填寫個資僅存於你建立的 Google 試算表；GitHub Pages 不保存資料、Token 或 LINE Access Token。
- LIFF 與官方帳號應位於同一個 Provider，否則一般 GitHub 網址模式取得的 LINE User ID 可能與 Messaging API 不一致。
- 上述為部署步驟，仍須以實際 LINE / GAS 操作驗收；原始碼測試不代表正式訊息已送達。