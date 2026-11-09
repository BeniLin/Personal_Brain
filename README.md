# Personal Brain — Project Memory Pack

這份文件包用來讓 Codex 與 Claude Code 共用 Personal Brain 專案背景、架構、決策與進度。

## 放置方式
將本資料夾中的檔案複製到 Personal Brain Git repository 根目錄，保留以下結構：

```text
AGENTS.md
CLAUDE.md
docs/
  PROJECT_CONTEXT.md
  ARCHITECTURE.md
  DECISIONS.md
  ROADMAP.md
  CURRENT_STATUS.md
```

## 開始新工作
在 Codex 中開啟 repository，請它先讀取 `AGENTS.md` 與 `docs/` 全部文件，再依 `CURRENT_STATUS.md` 繼續。

## 重要
這些文件是依目前已知討論整理的初始版本，不代表 repository 已經實作任何功能。請先比對實際程式碼與 Git 狀態。未知資訊已標示為待確認，不要自行補成事實。

每次完成 milestone 或確認新決策，更新對應文件並提交到 Git。
