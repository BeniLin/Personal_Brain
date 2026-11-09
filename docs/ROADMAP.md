# Personal Brain — Roadmap

## Product direction (long-term)
預期方向：

`Capture → Inbox → AI Organizer → Project Memory → Daily Planner`

Project Memory 預期可整理 Idea、Task、Question、Note、Decision 等內容。這是產品方向，不代表已完成或已納入 W1。

## W1 — Voice Capture Pipeline
目標：讓使用者透過 Galaxy Watch 7 口述內容，可靠保存到 DS223。

### W1-1 — Watch voice capture + local persistence
- 建立 Wear OS Watch app 基礎。
- 完成語音輸入與最終辨識結果取得。
- 辨識完成後自動送出。
- 建立 capture UUID。
- 在 Watch 本機持久化保存。
- 確保 app/process 中斷後仍可恢復待同步 capture。

### W1-2 — Watch → Phone Data Layer + Room + ACK
- 使用 DataClient / DataItem。
- Phone 接收後先寫入 Room。
- Room commit 成功後才 ACK Watch。
- 處理重複送達及離線恢復。
- README 說明 Google relay 與 E2E encryption / metadata 限制。

### W1-3 — Phone → NAS FastAPI + SQLite + WorkManager + HTTPS/Tailscale
- 建立 FastAPI API。
- 建立 SQLite schema 與唯一性約束。
- SQLite 使用 NAS persistent volume。
- Phone 使用 WorkManager 執行上傳與重試。
- 透過 Tailscale Serve HTTPS endpoint 連線。
- NAS 只有在 SQLite commit 後回報成功。
- 驗證重複 HTTP request 不會產生重複資料。

### W1-4 — End-to-end reliability tests
- Watch 離線捕捉與恢復。
- Watch → Phone ACK 邊界測試。
- Phone 離線及 WorkManager retry。
- NAS 重複 request / idempotency。
- App、container restart 後資料保留。
- NAS commit 與 UI 狀態一致。
- 端到端流程測試。

## W1 明確不包含
- AI Organizer
- Claude API / 雲端 LLM
- Project / Task / Priority
- Daily Planner
- 語意搜尋
- 原始音訊上傳

## 後續階段
W1 完成後，再根據實際使用情況規劃 Inbox 管理、AI Organizer、Project Memory、Daily Planner。不得在沒有使用者確認前自行擴張 W1 範圍或預先實作後續功能。
