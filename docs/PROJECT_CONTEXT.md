# Personal Brain — Project Context

## 1. 願景
建立一套個人使用的 Personal Brain，降低記錄想法與工作事項的摩擦。使用者能在想到事情時快速輸入或口述，系統可靠保存原始內容，之後再由 AI 協助整理、分類、連結與規劃。

預期長期工作流：

`Capture → Inbox → AI Organizer → Project Memory → Daily Planner`

Project Memory 未來可能包含：
- Idea
- Task
- Question
- Note
- Decision

以上是長期方向，不代表這些功能已在 W1 實作。

## 2. 核心產品原則
1. 捕捉要快，不能要求使用者先決定分類。
2. 原始內容必須可靠保存。
3. 原始語音辨識文字不可被 AI 或其他流程直接覆蓋。
4. 網路中斷時，資料應留在裝置端，恢復後可重試。
5. 使用者必須能分辨資料目前保存在哪一層。
6. 先做可靠的最小流程，再逐步加入 AI 整理能力。

## 3. 目前第一階段：W1 Voice Capture Pipeline
使用者希望透過 Samsung Galaxy Watch 7 快速口述內容，由 Android 手機轉送到 Synology DS223 NAS。

使用體驗目標：
- 抬手並啟動語音輸入。
- 使用語音辨識取得文字。
- 辨識完成後自動送出，不要求額外確認。
- 手錶與手機可在離線時暫存。
- 只有 NAS 確認資料已寫入 SQLite transaction 後，才能顯示「NAS 已保存」。

語音辨識可能有專有名詞誤辨識，例如 Python 被辨識成 Pitch。W1 應保留辨識原文，不擅自修正。

## 4. 使用者設備與已知環境
- 手錶：Samsung Galaxy Watch 7。
- 手機：Android 手機，確切型號尚未確認。
- NAS：Synology DS223。
- NAS 作業環境：DSM / Container Manager；版本需在實作時確認。
- NAS 連線：只透過 Tailscale 私網。
- DS223 上另有一個練習用 Verdaccio container，應與 Personal Brain 獨立，不可任意修改或移除。
- DS223 記憶體為 2GB，應採低資源設計。
- 使用者有 Tailscale。
- 使用者目前尚未確認已安裝 Android Studio、Android SDK、adb 或 Gradle；使用者表示 Android 開發環境會自行安裝，Agent 不可代為安裝。

## 5. 非本案設備注意事項
DS720+ 是使用者另一台 NAS，曾經遭遇勒索軟體事件。Personal Brain 本案目標設備是 DS223。不得混淆兩台 NAS，也不得將 DS720+ 的事件當成 DS223 的狀態。
