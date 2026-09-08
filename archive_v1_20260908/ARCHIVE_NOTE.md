# 封存說明 — 2026-09-08

這個資料夾是 Context Guardian kiosk 第一階段完成時的快照，對應 commit `a406999`。

## 這個版本包含什麼

- `index.html`：8 張投影片（首頁／問題／決策邏輯／規則一／規則二／被動生效／設計啟示／參考資料）。
  - 問題頁、設計啟示頁：樹狀圖／三角圖節點可點選，說明文字在下方共用區塊顯示。
  - 決策邏輯頁：判斷流程圖，含操作範例。
  - 被動生效頁：改為真正的流程圖（CLAUDE.md 觸發保證 → 判斷 → SKILL.md／規則被跳過）。
  - 參考資料頁：獨立成一頁，四篇原始出處，字體已放大 1.2 倍。
  - 手機版圖表裁切問題已修（拿掉 SVG min-width，取消巢狀橫向捲動）。
  - 首頁加了 QR code，掃描可直接開啟本頁。
- `skill/context-guardian/`：可安裝的 Claude Code skill（`SKILL.md` + `references/claude-md-snippet.md`），已拿掉私人專案代號的通用版。
- `skill/context-guardian.skill`：上面兩個檔案打包好的安裝檔。
- `README.md`：當時的 repo 說明文件快照。

## 之後要調整時

正式版位置維持在 repo 根目錄（`index.html`、`skill/`、`README.md`），這個資料夾只是留存紀錄，不要直接改。要更新時改根目錄的檔案，需要對照舊版再另外複製一份新快照。

線上頁面：https://ganma0517.github.io/context-guardian-kiosk/
