# AI-Agent-Subagent-Skill 索引

本文件是這個 repo 內所有 Subagent 與 Skill 的結構化總表，跟隨 `.claude/` 一起版控維護。
新增、刪除、重新命名任何 Agent 或 Skill 時，請同步更新本文件，讓任何 clone 這個 repo 的人不用逐一開檔案就能理解整體架構。

---

## 架構速覽

```
① 協調層 Orchestrator     → 判斷請求該找誰、要串幾個工作流
② 領域工作流層 Workflow    → 各自完整生命週期（規劃→執行→驗證→交付）
③ 能力模組層 Capability    → 單一職責、無領域判斷、被上層呼叫
④ 記憶/狀態層 Memory       → 排程執行之間的狀態延續（外部化，不寫在 Agent 定義內）
```

## 命名規則

| 層級 | 前綴 | 範例 |
|---|---|---|
| ①協調層 | `orch-` | `orch-main` |
| ②領域工作流 | `wf-` | `wf-dev`、`wf-data-academic` |
| ③能力模組 | `mod-` | `mod-web-scraper` |
| Skill | 不加前綴，語意導向命名 | `mermaid-syntax` |

**已知限制**：Claude Code 的 Skill 探索目前僅掃描 `.claude/skills/` 頂層，不支援巢狀資料夾分類，因此 Skill 一律放在同一層級，分類靠本索引表與命名語意，不靠實體資料夾。

**開源 Skill 處理原則**：不直接複製第三方 Skill 原始檔案（避免內部路徑引用被破壞），一律改寫成自己的版本後收錄，並在下方表格「參考來源」欄位註明靈感來源。

---

## ① 協調層

| 名稱 | 可重用範圍 | 觸發型態 | 需人在場 | 狀態 | 依賴關係 | 參考來源 |
|---|---|---|---|---|---|---|
| `personal-assistant` | 跨領域 | 互動式 | 是 | ✅ 已建立 | 無 | 自建 |
| `orch-main` | 跨領域 | 互動式 | 視情況 | ⏸️ 延後（待 Agent 數量門檻到達後排入時程） | 需彙整所有②③的路由邏輯 | 自建（規劃） |

## ② 領域工作流層

| 名稱 | 可重用範圍 | 觸發型態 | 需人在場 | 狀態 | 依賴關係 | 參考來源 |
|---|---|---|---|---|---|---|
| `wf-design` | 僅本領域 | 互動式 | 是（需審閱回饋） | ✅ 已建立 | 可能呼叫 `mod-diagram-generator`、`mod-slide-generator` | 自建 |
| `wf-dev` | 僅本領域 | 互動式為主，部署階段可事件觸發 | 視階段而定 | ❌ 未建立 | 呼叫 `mod-test-runner` | 自建 |
| `wf-data-media` | 僅本領域 | 互動式或排程 | 視情況 | ✅ 已建立 | 呼叫 `mod-web-scraper`；依賴 Skill `source-reliability-assessment` | 自建 |
| `wf-data-academic` | 僅本領域 | 互動式 | 是（需查核引用） | ✅ 已建立 | 依賴 Skill `academic-citation-review` | 自建 |
| `wf-data-market` | 僅本領域 | 互動式或排程 | 視情況 | ✅ 已建立 | 呼叫 `mod-web-scraper`；依賴 Skill `financial-analysis-framework` | 自建 |
| `wf-news-digest` | 僅本領域 | 排程式（建議 Cloud Routine） | 否 | ✅ 已建立 | 呼叫 `mod-web-scraper`、`mod-summarizer` | 自建 |

## ③ 能力模組層

| 名稱 | 可重用範圍 | 觸發型態 | 需人在場 | 狀態 | 依賴關係 | 參考來源 |
|---|---|---|---|---|---|---|
| `mod-web-scraper` | 跨領域 | 兩者皆可 | 否 | ✅ 已建立 | 依賴 Skill `web-scraping-patterns` | 自建 |
| `mod-summarizer` | 跨領域 | 兩者皆可 | 否 | ✅ 已建立 | 依賴 Skill `content-summarization` | 自建 |
| `mod-diagram-generator` | 跨領域 | 兩者皆可 | 否 | ✅ 已建立 | 依賴 Skill `mermaid-syntax` | 自建 |
| `mod-test-runner` | 跨領域 | 兩者皆可（CI 事件或手動） | 否 | ❌ 未建立 | 依賴 Skill `test-script-conventions` | 自建 |
| `mod-slide-generator` | 跨領域 | 互動式為主 | 視排版需求 | ✅ 已建立 | 沿用平台內建 `pptx` Skill | 沿用內建 Skill |
| `mod-script-writer` | 跨領域 | 互動式為主 | 視情況 | ❌ 未建立 | 依賴 Skill `speech-script-structure`；輸出可沿用內建 `docx` Skill | 自建 |

## Skill

| 名稱 | 可重用範圍 | 狀態 | 支援對象 | 參考來源 |
|---|---|---|---|---|
| `mermaid-syntax` | 跨領域 | ✅ 已建立 | `mod-diagram-generator` | 自建 |
| `content-summarization` | 跨領域 | ✅ 已建立 | `mod-summarizer` | 自建 |
| `web-scraping-patterns` | 跨領域 | ✅ 已建立 | `mod-web-scraper` | 自建 |
| `source-reliability-assessment` | 跨領域 | ✅ 已建立 | `wf-data-media` | 自建 |
| `academic-citation-review` | 跨領域 | ✅ 已建立 | `wf-data-academic` | 自建 |
| `financial-analysis-framework` | 跨領域 | ✅ 已建立 | `wf-data-market` | 自建 |
| `test-script-conventions` | 跨領域 | ❌ 未建立 | `mod-test-runner` | 自建 |
| `speech-script-structure` | 跨領域 | ❌ 未建立 | `mod-script-writer` | 自建 |

## ④ 記憶/狀態層

| 名稱 | 可重用範圍 | 狀態 | 說明 | 參考來源 |
|---|---|---|---|---|
| Notion 登記簿 | 跨領域 | ⏸️ 待建 | 追蹤各工作流的執行狀態、排程設定、最後執行結果；透過 Notion MCP 讀寫 | 外部平台（Notion）+ 自建 schema |

---

## 統計

- ①協調層：2（1 已建立、1 延後）
- ②領域工作流：6（0 已建立、1 規劃中、5 未建立）
- ③能力模組：6（1 已建立、5 未建立）
- Skill：8（1 已建立、7 未建立）
- ④記憶/狀態層：1（待建）

**建置優先序**：`mod-web-scraper` → `mod-summarizer` → `wf-news-digest` → 依序展開其餘②③ → 視 Agent 數量成長評估 `orch-main` 與 Notion 登記簿的建置時機。
