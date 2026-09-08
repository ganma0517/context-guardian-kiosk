# Context Guardian

Claude Code 用的被動式上下文管理規則：任務跑得長、工具一次回傳一大包內容時，讓 Agent 自己判斷該壓縮、卸載，還是拆給子代理處理，不需要使用者盯著用量、手動喊停。

## 線上介紹頁

https://ganma0517.github.io/context-guardian-kiosk/

互動式簡報，說明規則背後的問題、決策邏輯、卸載與子代理規範、以及怎麼讓它在 Claude Code 裡真正被動生效。

## Skill 套件

`skill/context-guardian/` 是可以直接安裝進 Claude Code 的 skill：

```
skill/
  context-guardian.skill          可直接安裝的封裝檔
  context-guardian/
    SKILL.md                      規則本體
    references/
      claude-md-snippet.md        建議加進專案 CLAUDE.md 的提醒段落
```

安裝方式：把 `context-guardian/` 資料夾放進專案的 `.claude/skills/` 底下，或直接安裝 `context-guardian.skill`。要讓規則真正被動生效（不用等 Claude 自行判斷要不要觸發），另外把 `references/claude-md-snippet.md` 裡的段落貼進專案的 `CLAUDE.md`。
