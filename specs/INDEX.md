# Personal Brain 規格索引

最後更新：2026-11-12

> 需求討論時 Claude 會先讀這份索引，再挑相關的文件閱讀。
> 「涵蓋內容」請寫出具體的規則與關鍵字，讓人看到需求中的詞就能對應到文件。

> **專案現況（2026-11-12 盤點）**：repository 目前只有文件，尚無任何程式碼。
> 下列文件描述的是「已確認的設計與決策」，不是已實作的行為。
> 目前階段為 W1 — Voice Capture Pipeline 的技術設計／準備進入實作。

## 專案層文件（跨領域）

| 文件 | 涵蓋內容 |
|---|---|
| `docs/PROJECT_CONTEXT.md` | 產品願景與長期工作流（Capture → Inbox → AI Organizer → Project Memory → Daily Planner）；六項核心產品原則（捕捉要快、原文可靠保存、原文不可被覆蓋、離線暫存後重試、可分辨資料保存在哪一層）；W1 使用體驗目標；使用者設備（Galaxy Watch 7、Android 手機、Synology DS223、Tailscale、Verdaccio）；DS223 與 DS720+ 不可混淆 |
| `docs/DECISIONS.md` | 已確認決策 D-001 ～ D-017：W1 範圍與不包含項目、DS223 僅走 Tailscale 私網、接受 Google Data Layer relay、辨識完成自動送出、「NAS 已保存」以 SQLite commit 為準、Backend first、Tailscale Serve（不用 Funnel）、FastAPI + SQLite、保留 Verdaccio、`capture_id` UUID 跨層冪等、原始辨識文字不可覆蓋、不上傳原始音訊、Codex／Claude 以 repository 文件為共用記憶、不先建立獨立 Orchestrator |
| `docs/ROADMAP.md` | 里程碑切分 W1-1（Watch 語音捕捉＋本機持久化）、W1-2（Watch → Phone Data Layer＋Room＋ACK）、W1-3（Phone → NAS FastAPI＋SQLite＋WorkManager＋HTTPS）、W1-4（端到端可靠性測試項目）；W1 明確不包含的功能；後續階段方向 |
| `docs/CURRENT_STATUS.md` | 目前階段、已完成的規劃、尚待進行／驗證的七個項目（設備與版本確認、Android 開發環境、API 細節未定義等）、狀態標示規則（已完成／已測試／待驗證／待確認） |

## 語音捕捉管線（W1）

| 文件 | 涵蓋內容 |
|---|---|
| `docs/ARCHITECTURE.md` | W1 整體架構（Watch → Phone → NAS）。**Watch**：語音辨識、自動送出、本機持久化、同步狀態 `LOCAL_PENDING`／`PHONE_ACKNOWLEDGED`／`NAS_CONFIRMED`。**Watch → Phone**：DataClient／DataItem、relay 與 metadata 限制、Phone 先寫入 Room 才 ACK、不做自訂 Bluetooth RFCOMM。**Phone**：驗證 payload 與 `capture_id`、Room、WorkManager 上傳與重試、Doze。**NAS**：單一 container、FastAPI、SQLite、persistent volume、2GB RAM 限制、不加 PostgreSQL／Redis／worker。**網路**：HTTPS over Tailscale Serve、不公開、不設 port forwarding。**冪等**：`capture_id` 各層唯一性約束、重複送達不產生重複資料。**原文保存**：不正規化、不改寫、不翻譯、不保存音訊。**API 合約**：必須支援的能力（endpoint、JSON schema、認證、時間格式、回應格式皆尚未定義）。**狀態語意**：辨識完成／Watch 已存／Phone 已存／NAS 已 commit 四個階段，只有第四個算「NAS 已保存」 |

## 尚未文件化的功能

以下功能已規劃但**尚未實作**，也沒有獨立的規格文件，只在上面的架構與 Roadmap 文件中有設計層級的描述。討論相關需求時，細節需要向使用者確認。

| 功能領域 | 說明 | 主要程式碼位置 |
|---|---|---|
| Watch 語音捕捉 | 在 Galaxy Watch 7 上啟動語音輸入、取得最終辨識文字、自動送出、產生 `capture_id`、本機持久化、中斷後恢復待同步項目、顯示同步狀態（W1-1） | 尚未實作 |
| Watch → Phone 同步 | 透過 Data Layer 把 capture 送到手機、手機寫入 Room 後回 ACK、重複送達與離線恢復的處理（W1-2） | 尚未實作 |
| Phone → NAS 上傳 | 手機用 WorkManager 把 capture 經 Tailscale Serve HTTPS 上傳到 NAS、失敗重試、記錄 NAS 確認並回傳狀態給 Watch（W1-3） | 尚未實作 |
| NAS Capture API | FastAPI 接收 capture、以 `capture_id` 冪等寫入 SQLite、commit 後才回成功、查詢 capture 是否已保存。endpoint、payload、認證、時間格式、回應格式皆未定義（W1-3） | 尚未實作 |
| NAS 部署 | DS223 Container Manager 上的 container 設定、SQLite volume 路徑、Tailscale Serve 轉發與 ACL 設定。實際版本、路徑、hostname、port 皆未確認 | 尚未實作 |

W1 之後的方向（Inbox 管理、AI Organizer、Project Memory、Daily Planner、語意搜尋）只有產品方向，尚未規劃，見 `docs/ROADMAP.md`。

## Test Case

| 文件 | 涵蓋內容 |
|---|---|
