# Personal Brain — Decision Log

本文件只記錄使用者已明確確認的決策。尚未確認的技術細節應標示為待確認，不得當成既定規格。

## D-001 — W1 優先完成可靠語音捕捉
W1 先建立 Capture → Watch/Phone 持久化 → NAS 保存流程，不做 AI Organizer 或 Daily Planner。

## D-002 — 目標 NAS 為 Synology DS223
Personal Brain 部署在 DS223，不是 DS720+。DS720+ 是另一台 NAS，與本案分開。

## D-003 — DS223 透過 Tailscale 私網存取
NAS API 僅透過 Tailscale 私網使用，不公開到 Internet，不設定 router port forwarding。

## D-004 — Watch-to-Phone 接受 Google Data Layer relay
使用 Wear OS Data Layer API（DataClient / DataItem）。Bluetooth 可用時可走 Bluetooth；不可用時允許 Google 網路中繼。不得宣稱永遠只走 Bluetooth。Data Layer 通訊端對端加密，但仍可能經 Google 伺服器並存在連線 metadata。

## D-005 — Watch 辨識完成後自動送出
語音辨識完成後自動送出，不要求使用者再按確認。

## D-006 — NAS 成功狀態必須以資料庫 commit 為準
只有 NAS 確認 SQLite transaction 已 commit，才可顯示「NAS 已保存」。辨識完成、Watch 本機保存、Phone 收到或 Phone Room 保存都不能冒充 NAS 成功。

## D-007 — Android 開發環境由使用者自行安裝
使用者尚未確認 Android Studio / SDK / adb / Gradle 安裝完成。使用者選擇自行安裝。Agent 不可自行安裝或修改使用者環境。

## D-008 — W1 先 Backend first
先進行 Backend 工作，再處理 Android app build / device integration。Android source 可準備，但沒有實際環境驗證前不得宣稱 build/test 通過。

## D-009 — HTTPS 使用 Tailscale Serve
Phone → DS223 使用 HTTPS over Tailscale，選用 Tailscale Serve 提供私有 HTTPS endpoint/certificate。不要使用 Funnel。

## D-010 — NAS Backend 使用 FastAPI + SQLite
W1 使用單一輕量 FastAPI app container + SQLite。SQLite 必須放在 NAS persistent volume。DS223 只有 2GB RAM，避免一開始加入 PostgreSQL、Redis 或獨立 worker。

## D-011 — 保留 DS223 現有 Verdaccio
DS223 上的練習用 Verdaccio container 與 Personal Brain 分開，不可任意修改或移除。

## D-012 — capture_id UUID 與跨層冪等
每筆 capture 使用 UUID。Watch、Data Layer、Phone、WorkManager、HTTP retry 都沿用同一 ID。Watch、Room、SQLite 皆需唯一性約束，避免重複插入。

## D-013 — 原始辨識文字不可被覆蓋
W1 保存原始語音辨識文字，不自動修正專有名詞或錯字。未來 AI 若提出修正，必須另存，不可覆蓋原文。

## D-014 — W1 不上傳原始音訊
W1 只處理語音辨識後的文字，不上傳或保存原始錄音檔。

## D-015 — 不在 W1 加入 AI 與規劃功能
W1 不做 AI Organizer、Claude API、Project、Task、Priority、Daily Planner 或語意搜尋。

## D-016 — Codex/Claude 共用 repository 文件作為專案記憶
長期協作以 Git repository 文件為共同事實來源，不依賴單一 AI 的聊天記憶。Codex 作為協調者與 Reviewer；Claude Code 作為可委派的實作 Agent。實際呼叫方式與自動化設定仍需驗證，不可假設已完成。

## D-017 — 不先建立獨立 Orchestrator
目前優先驗證 Codex 現有能力是否可透過本機命令或 MCP 呼叫 Claude Code。未證明原生設定不可行前，不建立大型 Python bridge、獨立 Orchestrator、Docker service 或資料庫。
