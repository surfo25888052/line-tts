# 里民意見表｜既有 LINE 官方帳號 + GitHub Pages + GAS + Google 試算表

## 最簡單、不另建 LINE 官方帳號或 LIFF 的方法

1. 在現有 LINE 官方帳號的圖文選單設定此 GitHub Pages 網址：`https://surfo25888052.github.io/line-tts/resident-feedback/`。
2. 里民填寫姓名、電話、地址及反映意見，點擊「送出里民意見」。
3. 網頁會依 LINE 官方文件的 `https://line.me/R/oaMessage/.../?` 格式，**開啟既有 LINE 官方帳號聊天室，預先填入完整意見**。
4. **里民必須在 LINE 再按一次「傳送」**。這一步不能由一般 GitHub 網站自動代按。只有實際送出聊天訊息後，才算完成通報。
5. GAS 的 LINE Webhook 收到里民本人訊息，寫入綁定的 Google 試算表 `里民意見` 工作表，並用 Messaging API Reply API 回覆表單編號。管理員在原官方帳號聊天室可辨認送件人。

此方法不需要 LIFF、LINE Login Channel、LINE 官方帳號重建或其他服務；但需要 **LINE Messaging API 的 Webhook 指向本 GAS，且必須有可用的 Channel Access Token**。GAS 現在能以既有 Channel ID 與 Channel secret 自動申請 15 分鐘有效的 stateless token。

## 從既有官方帳號取得所需資訊

既有 OA：`@971zpojs`。在 LINE Official Account Manager（官方帳號管理後台）→ 設定 → Messaging API 中可看到 **Channel ID** 和 **Channel secret**。這兩者請只填入 Apps Script 的 `Script Properties`（指令碼屬性），不要貼在公開 GitHub 或交給任何人。Channel secret 是敏感憑證。

| Script Property | 作用 | 必填 |
|---|---|---|
| `LINE_CHANNEL_ID` | 既有 Messaging API Channel ID（數字） | 是，若未提供 Access Token |
| `LINE_CHANNEL_SECRET` | 同一頻道的 Channel secret（機密） | 是，若未提供 Access Token |
| `LINE_CHANNEL_ACCESS_TOKEN` | 可選：若已有可用 token，直接使用而不申請 stateless token | 否 |
| `ADMIN_TO` | 可選：另通知某個管理員 LINE User ID 或群組 ID | 否 |
| `WEBHOOK_KEY` | 可選：Webhook 網址密鑰；設定後需加在 LINE Webhook URL | 否 |

GAS Token 作業：`getLineToken_()` 先採用 `LINE_CHANNEL_ACCESS_TOKEN`；否則使用 Channel ID + secret 呼叫 `POST https://api.line.me/oauth2/v3/token`，並在 `CacheService` 快取到期前的短效 Token。Stateless token 的發行不影響舊的長期 Token。

## 綁定 Google 試算表的部署

1. 既有試算表：**三星里長里民意見表**。在其選單 **擴充功能 → Apps Script** 貼上 `Code.gs` 全部程式。若你已部署過前版，請以本版取代原版，再做「部署 → 管理部署 → 編輯 → 新版本」更新原 `/exec`。
2. 在「專案設定 → 指令碼屬性」新增 `LINE_CHANNEL_ID` 與 `LINE_CHANNEL_SECRET`，不要把 secret 貼到前端。
3. 手動執行 `setupSheet()`（建立 `里民意見` 工作表）；再執行 `testLineConnection()`（只查詢現有 LINE bot 資訊、不發訊息）。
4. GAS 網頁應用程式由 **我（Me）** 執行，存取權限 **所有人（Anyone）**。
5. 在**既有** OA 管理後台的「設定 → Messaging API」準備修改 Webhook URL，但在確認不再需要 OakMega 的舊 webhook 處理前，**不要按儲存**。每個頻道只能設定一個 Webhook，切換後舊 OakMega 服務不再收到事件。
6. 在正式切換後用另一支 LINE 測試帳號開啟 GitHub 表單、填寫內容、在原 OA 聊天室**按傳送**，然後核對該 LINE 聊天回覆與試算表新增列。

## 進階選配：LIFF 全自動傳送

若日後取得既有 OA 所屬 Provider 權限，可建立一個 **LINE Login channel（不是另一個 LINE 官方帳號）** 與 LIFF，配置 `config.js` 的 `liffId`，在原官方帳號聊天窗透過 LIFF URL 開啟後可用 `liff.sendMessages` 直接傳送；這仍須在正確聊天室與 `chat_message.write` 權限下才有用。一般 GitHub Pages 入口不需要 LIFF。

## 程式保留的其它功能

- 原表單四個欄位，新增時間／表單編號／LINE 使用者 ID／LINE 顯示名稱／回覆結果共 10 欄。
- 由 LINE 訊息發出的表單會去重並用 Reply API 回覆本人。
- 可選的 `ADMIN_TO` 推播，不會傳送到未指定的人。
- GAS `doPost(e)` 不能取得 LINE 要求的 `x-line-signature` header，本架構無法提供等價簽章驗證；可使用保密 `WEBHOOK_KEY` 降低濫用，但不是相同安全性。
- 本地 Node 測試屬模擬，不代表正式 Webhook、GAS 或手機端已驗收。

參考 LINE 官方文件：
- https://developers.line.biz/en/docs/messaging-api/using-line-url-scheme/
- https://developers.line.biz/en/reference/messaging-api/#issue-stateless-channel-access-token
- https://developers.line.biz/en/tips/2026/07/23/using-multiple-api-tools/
