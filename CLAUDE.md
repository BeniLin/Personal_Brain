# CLAUDE.md — Claude Code 開發規則

## 專案
Personal Brain。請先讀取：
- `AGENTS.md`
- `docs/PROJECT_CONTEXT.md`
- `docs/ARCHITECTURE.md`
- `docs/DECISIONS.md`
- `docs/ROADMAP.md`
- `docs/CURRENT_STATUS.md`

## 開發原則
- 以使用者已確認的決策為準，不重新要求使用者選擇已定案事項。
- 不擴大當前 milestone 的範圍。
- 修改前先檢查 repository、Git 狀態及現有實作。
- 不可自行安裝 Android Studio、Android SDK、adb、Gradle 或其他系統工具；使用者已表示 Android 開發環境由他自行安裝。
- 不可自行啟用付費 LLM API 或其他計費服務。
- 不可把 NAS 服務公開到 Internet，不可設定 router port forwarding。
- 不可把 secrets、token、API key 寫入 repository 或 log。
- 不可把語音辨識結果自動修正後覆蓋原始文字。
- Capture retry 必須沿用相同 `capture_id`，不可因重試產生新 ID。
- 不可將「已收到手機」或「語音辨識完成」顯示為「NAS 已保存」。
- 不可宣稱未實際執行的 build/test 已通過。

## W1 範圍限制
W1 只完成可靠的語音捕捉與保存：
Watch voice capture → Watch local persistence → Phone Data Layer receive → Phone Room persistence → WorkManager upload → NAS FastAPI → SQLite commit。

W1 不包含：
- AI Organizer
- Claude API / 雲端 LLM 整合
- Project / Task / Priority / Daily Planner
- 語意搜尋
- 原始音訊上傳或保存
- 自動改寫原始辨識文字

## 實作回報格式
完成工作後請回報：
1. 修改摘要
2. 修改檔案
3. 重要設計決策
4. 執行過的測試命令及真實結果
5. 尚未測試或尚未完成的部分
6. 風險與下一步建議

若使用者要求「只審查」，不得修改任何檔案。
