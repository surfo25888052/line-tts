# 三星里民意見表｜2026-11-16 v5 自動回覆修復

## 修正的問題與目前狀態

- 先前 GitHub 前端沒有呼叫 GAS 入庫，現在會在 `liff.sendMessages()` 之後，使用瀏覽器正常 POST 向 GAS 提交同一個表單編號。
- 先前貼入 GAS 的免大橡版 `doPost(e)` 只接受網頁表單，不能接 LINE 的 JSON Webhook。這份 `Code.gs` **同時支援兩種事件**。
- LINE Webhook 以 `HtmlService.createHtmlOutput('OK')` 回應，避免 `ContentService` 一定會產生的重新導向；仍須用真實 LINE 服務驗證 HTTP 200。
- **表單編號僅保留在 GAS/Google 試算表內部**；會員送出的 LINE 訊息、官方回覆、成功/失敗受理頁均不顯示編號。Webhook 與表單先後抵達時，使用內容與近期唯一紀錄做保守合併；無法唯一確認則不猜測身分。
- **GitHub 前端已更新，GAS 仍需要以 v5 更新既有部署。** 修改後必須做實際 Webhook 與 LINE 自動回覆驗收，沙盤測試不能取代。

## 2026-11-16 v5 修復與驗收條件

- **姓名、聯絡電話、地址、意見內容四欄都必填**：由網頁填表時驗證；GAS 收到網頁 POST 時亦作資料完整性檢查。**LINE Webhook 不以電話、地址、編號或欄位完整性作為自動回覆的前提**，僅辨識表單訊息標記。
- GitHub 表單已移除標題下、輸入欄位附近、送出按鈕旁及頁尾的說明小字；僅在操作失敗或傳送進行中顯示必要狀態。
- **LINE 官方帳號自動回覆與 GAS 成功受理頁面完全一致**：`已收到您陳情，待陳情處理完畢後會由承辦人員回覆。`。不附表單編號、不附其他文字。GAS 會先呼叫 LINE Reply API，再做試算表紀錄，避免收件紀錄操作阻斷回覆。
- 要讓新回覆程序生效，請把 Apps Script `Code.gs` 完整換成 **v5**，透過「部署 → 管理部署 → 編輯 → 新版本 → 部署」更新原 `/exec`。**不用更換 URL 或指令碼屬性。** GitHub 更新不會自動修改已部署的 GAS。

## 2026-11-16 最新操作原則：不再要求里民手動複製

- **正式入口是原官方 LINE 帳號圖文選單中的 LIFF URL** `https://liff.line.me/2012638796-D0JchPir`。它會載入同一份 GitHub Pages 網頁，但保留 LINE 聊天室的發訊能力。
- 從原官方 LINE 聊天室開啟後，只需填表並按一次「送出」。前端先使用 `liff.sendMessages()` 將**無表單編號的意見**送到會員自己的官方帳號聊天室，再以 POST 將內部編號和資料送入 GAS；LINE Webhook 收到文字時立即自動回覆。
- **桌面 Chrome/Edge、一般 GitHub Pages 入口及不具 LINE 聊天室 context 的視窗，現在會停止送出**，不會再顯示「已入庫但請複製去 LINE」的假完成流程，也不會把這種情形寫入新案件。
- 限制不是權限或驗證步驟可消除：LINE 官方 `liff.sendMessages()` 限於 LINE App 聊天室。若需要電腦首次填表也直接出現在原官方帳號對話，且無任何人工操作，必須另外具備可對應原 Messaging API User ID 的正式身分映射機制；目前不同 Provider 的 LINE Login User ID 不可直接拿來推播。
- 請將 LIFF 的 `profile`、`chat_message.write` 開啟。若 Channel 仍為 `Developing`，只有 Admin / Tester 能使用；開放里民前須改為 `Published`。

## 已核對的設定

| 用途 | 值 |
|---|---|
| 原 LINE 官方帳號 | `@971zpojs` |
| 原官方帳號 Messaging API Channel ID | `2006432390` |
| 新 LINE Login Channel ID | `2012638796` |
| LIFF ID | `2012638796-D0JchPir` |
| GitHub Pages 表單 | `https://surfo25888052.github.io/line-tts/resident-feedback/` |
| 原官方 LINE 圖文選單建議網址 | `https://liff.line.me/2012638796-D0JchPir` |
| GAS Web App 部署網址 | `https://script.google.com/macros/s/AKfycbwkx_k2l6GzY09tJjyqAJYrwWWLEA9jWOlxE7wT75bmHIqoLX9qjULa7jhpZsSv4BI/exec` |
| Google 試算表 | `https://docs.google.com/spreadsheets/d/1Pztfq9lz_5ivAPLlAiv1qUKWeVVtdFqx1BF4854bNp4/edit` |
| GAS 記錄分頁 | `里民意見`（13 欄，已由 ChatGPT 在試算表建立） |

## 這一次必須由 Google Apps Script 編輯器完成的步驟

1. 開啟已有 GAS 專案「三星里長里民意見表」，把目前 `Code.gs` 的**全部內容**換成本包 `Code.gs`。此檔只有一組 `doGet()` 與 `doPost()`，**勿與舊版 GAS 並存**。
2. GAS → 左側「專案設定（Project Settings）」→「指令碼屬性（Script Properties）」：
   - `LINE_CHANNEL_ID`：填入 **`2006432390`**（原官方帳號的 Messaging API 頻道 ID；絕不是新 LINE Login Channel ID）。
   - `LINE_CHANNEL_SECRET`：進入 `https://manager.line.biz/`，選原官方帳號「三星里長簡新發」→ 設定 → Messaging API → 在 **Channel secret** 旁按「複製」。直接貼入此 GAS 私密屬性，不要傳到聊天、不要上傳 GitHub。
   - `LINE_CHANNEL_ACCESS_TOKEN`：**可留空**。若已存在，程式會優先使用該值；若不是原頻道 Token，請移除錯誤的值。
   - `WEBHOOK_KEY`：若先前未設定，就保持空白。只有設定此值時，LINE Webhook URL 才必須同時附加 `?key=該值`。
3. GAS 上方選單依序執行 `setupSheet()`、`testLineConnection()`，授權 Google 權限。前者應回傳 `columns:13`，後者必須回傳 `basicId:"@971zpojs"`。
4. GAS →「部署（Deploy）」→「管理部署（Manage deployments）」→ 對現有 Web App 按鉛筆→ 版本選「新版本（New version）」→「部署」。**維持原有 `/exec` 網址**，不必建立新部署。Web App 執行者為 `Me`，存取權限為 `Anyone`。
5. LINE Official Account Manager（原官方帳號）→ 設定 → Messaging API：確認你的 Webhook 網址仍是本表上方的 `/exec`；到「回應設定」確認 Webhook 已啟用。
6. LINE Developers 新建的 LINE Login Channel 目前顯示 `Developing`，只能由 Admin/Tester 測試。確認 LIFF 權限 `profile`、`chat_message.write`，以及 Endpoint URL 是 GitHub Pages 表單網址。先用管理員/測試者手機測試；驗收成功後才切成 `Published` 對里民開放。
7. 從原 LINE 官方帳號聊天室開啟 LIFF 網址，填寫一筆**測試**意見，核對：
   - 聊天室出現該會員本人送出的完整意見，**不包含表單編號**。
   - Google「里民意見」分頁新增一列，B 欄保留內部編號；Webhook 若成功並能唯一比對該筆表單，M 欄會記錄事件 ID。
   - 原官方帳號另外回覆**且僅回覆**：`已收到您陳情，待陳情處理完畢後會由承辦人員回覆。`。

## 運作與權限限制

- 即使 Bot Token 缺失、回覆失敗，GAS 的 Webhook 與 LIFF 表單仍會盡力保存意見；不會把入庫當成成功送出 LINE，也不會在真正回覆失敗時聲稱已回覆。
- 新 LINE Login Channel 和原 Messaging API Channel 可能屬於不同 Provider，使用者 ID **不相同**。H 欄存原官方帳號 User ID，I 欄存新 LINE Login User ID，不能混用。
- GAS 的 `doPost(e)` 無法讀到 LINE 原始 `x-line-signature` HTTP header，因此**不能提供 LINE 官方建議的簽章驗證**。目前為了依照你既有 GAS 直連 Webhook 的架構，沒有新增任何代理服務。可自行設定 `WEBHOOK_KEY` 降低偽造風險，但不等價於簽章。
- LINE 官方 Messaging API Channel 只支援一個 Webhook URL。切換到 GAS 後，原 OakMega Webhook 無法繼續同時接收同一頻道的事件；若其他原先依賴它的功能仍需保留，必須另外評估。
- `liff.sendMessages()` 只有在 LINE App 中從聊天室開啟、已啟用 `chat_message.write` 且符合官方使用條件時可直接送訊。**其他入口現在禁用送出，不再提供手動複製的替代路徑。**
- 只有一個真正的 GAS Web App，不需要新官方 LINE、其他伺服器或 GitHub Token。
- 目前測試是**沙盤模擬與 GitHub 回讀**，不等於已在手機、正式 LINE Webhook 與 GAS 真實回覆完成驗收。

## 修正出現問題時該看哪裡

- 沒有新資料列：到 GAS「執行項目（Executions）」看 `doPost` 的原始錯誤；核對 Web App 版本、GAS 的任何權限授予、`里民意見` 13 欄表頭與 LIFF 的登入狀態。
- 資料列有，但 K 欄停在「LIFF前端回報已傳送」：GAS 的表單端點已收到，但 Webhook 尚未確認收到 LINE 事件。檢查 LINE 官方帳號 Webhook 是否「啟用」，並使用 LINE 的 Verify 測試檢查 HTTP 回應。
- 資料列有，K 欄「Webhook已確認收到」，L 欄「未自動回覆」：資料已存，但 Bot Token／Reply API 有問題。先執行 `testLineConnection()`；回傳必須是 `@971zpojs`。
- `testLineConnection()` 回傳錯誤：檢查 `LINE_CHANNEL_ID` / `LINE_CHANNEL_SECRET` 是否是原官方帳號之 Messaging API 欄位，不要使用新 LINE Login 欄位，也不要上傳密鑰。
- 若 LINE 官方 Verify 顯示非 HTTP 200，保留實際 HTTP 狀態碼與錯誤內容；沙盤 `HtmlService` 模擬通過不代表 Google 正式環境已驗收。
- GAS v5 新增唯讀函式 `diagnoseWebhookEndpoint()`，可向 LINE 官方讀取實際 Webhook `active`（啟用狀態）與 endpoint（端點），輸出是否指向本 GAS、`WEBHOOK_KEY` 保護是否與 URL 配套。函式**不改 Webhook、不洩露憑證或完整 URL**。若資料只入試算表卻無官方回覆，先使用此函式與 GAS「執行項目」的 `RF_WEBHOOK_REPLY` 訊息區分事件未抵達、缺少 replyToken 或 Reply API 失敗。

## 官方依據

- LIFF sendMessages：`https://developers.line.biz/en/reference/liff/#send-messages`
- LINE Webhook：`https://developers.line.biz/en/docs/messaging-api/receiving-messages/`
- LINE Webhook 驗證：`https://developers.line.biz/en/docs/messaging-api/verify-webhook-url/`
- 短效無狀態 Channel Token：`https://developers.line.biz/en/reference/messaging-api/#issue-stateless-channel-access-token`
- GAS ContentService redirects：`https://developers.google.com/apps-script/guides/content`