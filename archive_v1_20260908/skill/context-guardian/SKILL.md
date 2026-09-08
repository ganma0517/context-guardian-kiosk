---
name: context-guardian
description: Proactively manage context window usage during long-running agentic work — compaction, sub-agent decomposition, file-system offloading, scope filtering, and session resets. Use this skill automatically whenever a task involves large tool outputs (literature database dumps, statistical program logs, cloud document downloads, interview transcripts, batch search results), multi-step or multi-file work, or any session likely to run many turns — even if the user never mentions "context", "token limit", or "overflow". Do not wait to be asked; check these rules before letting large outputs accumulate raw in the conversation.
---

# Context Guardian

被動式上下文管理規則。目標：讓 Claude 自己判斷何時該壓縮、卸載、拆分子任務，不需要使用者手動喊 `/compact` 或盯著 token 用量。

## 觸發時機（不用使用者提起）

一旦符合以下任一條件，立刻套用本 skill 的規則，不必等使用者要求：

- 即將呼叫或剛呼叫完會回傳大量內容的工具（文獻資料庫全文檢索、雲端文件下載、統計程式執行紀錄、逐字稿讀取、多次疊加的搜尋結果）
- 任務預期會跨多個 turn（例如多階段的分析流程、跨專案的共用設定檔建置）
- 正在處理的檔案／文獻數量超過 3 份，或單一工具回傳超過 500 行
- 準備要派工給子代理，或正在彙整多個子代理的回傳結果
- 一個邏輯階段（例如一份文獻的分析、一次模型迭代）已經跑完，準備切換到下一個獨立階段

## 決策流程

```
工具即將回傳大量內容？
├─ 是 → 內容是否需要逐字保留（引註原文、審稿意見、變數操作型定義）？
│        ├─ 是 → 寫入 scratch 檔案，對話中只放路徑 + 3-5 行摘要（見「卸載規則」）
│        └─ 否 → 摘要成重點清單，原始輸出不進入對話歷史
│
任務可拆成獨立子單元（每篇文獻／每份逐字稿／每個模型）？
├─ 是 → 逐一派子代理處理，要求結構化回傳而不是自由文字（見「子代理規範」）
└─ 否 → 主線直接處理，但仍套用上面的卸載規則

一個階段做完，下一階段不需要這階段的中間過程？
└─ 是 → 提醒使用者可以開新對話，並先把該階段的結論寫入設定檔，
         不要留在對話歷史裡等它被動壓縮
```

## 壓縮規則（Compaction）

**不可壓縮清單**：下列內容絕對不能被自動摘要掉，壓縮前一律先抽出寫進檔案。
- 審稿人的原始文字（逐字保留，之後回應審稿意見要精確對應）
- 變數的操作型定義、資料集的變數代碼
- 統計模型的確切設定（估計方法、群集選項、插補次數）
- 任何會被引用查核機制檢查的內容

其餘（工具呼叫的中間過程、已經讀過但已下結論的長文件全文、探索性搜尋的原始清單）可以摘要或直接捨棄，只留結論。

摘要本身要當作有損轉換對待，這正是摘要容易把模糊處講成確定結論的地方。摘要後，對不確定的地方要標記「待查證」，不要自己補完。

## 卸載規則（Offloading）

- 大型工具輸出一律先寫入 `.claude/scratch/<task-slug>/` 底下的檔案，對話中只保留檔案路徑 + 一段 3-5 行的摘要
- 需要回頭查閱時用 `grep -n` / `head` / `tail` 精準抓取，不要整份重新讀入對話
- 專案的長期結論（不是暫存）要寫回專案的設定檔，不要留在 scratch。scratch 是這次 session 用的，設定檔是跨 session 持久的

## 子代理規範（Sub-agents）

子代理只回傳結論不夠，沒有可追溯的引用鏈，主線沒辦法核實。子代理回傳格式至少要包含：

```
- 結論（1-2 句）
- 支持依據（來源檔案 + 行號或頁碼，不能是敘述性的說明）
- 待驗證項目（有沒有無法確認來源的地方）
```

主線收到回報後，只把結論摘要進主對話，支持依據留在檔案裡備查。

## 範圍過濾（Scope Control）

在專案的 `.claude/settings.json` 或等效設定裡排除：
- 原始 PDF、大型二進位檔案（改用已抽取的純文字版本）
- `.git`、`node_modules`、統計軟體產生的暫存檔
- 已經完成分析、進入知識庫的舊逐字稿全文（改用知識庫的結構化輸出）

思考預算：例行、格式化、確定性高的步驟用低思考預算；真正需要判斷的步驟（理論架構比對、審稿意見的實質回應）才用高思考預算。

## 階段切換與重置

完成一個獨立階段後，先確認結論已經寫進對應的設定檔，再建議使用者開新對話，而不是讓舊階段的探索過程留在上下文裡等自動壓縮。跨專案的工作不要共用同一個 session。

## 讓這個 skill 真正「被動」的關鍵

Skill 本身是按需載入的機制，Claude 要先判斷任務符合 description 才會讀取這份規則本體。若要它在每個 turn 都自動生效、不依賴 Claude 主動判斷觸不觸發，需要搭配專案根目錄的 `CLAUDE.md`（每次都會載入），放一段極短的提醒，把判斷責任從「要不要觸發 skill」改成「一律檢查」。建議寫入 `CLAUDE.md` 的內容見 `references/claude-md-snippet.md`。
