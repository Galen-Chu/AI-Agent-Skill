# 評測迴圈試跑紀錄

記錄 `orch-main` → `mod-eval-report` 這條可追溯性評測路徑的實際試跑結果，用來驗證評測迴圈是否如預期運作。日後若累積多次試跑、觀察到重複出現的落差模式，可作為評估「4. 優化」階段（見 `AI-Agent-Skill-ArchitectureDesign.md` 第十一節）是否值得拆成獨立元件的依據。

---

## 試跑一：驗收 `mod-eval-report.md` 本身是否符合 Agent 撰寫規範

**日期**：2026-08-11
**觸發方式**：使用者直接請 `orch-main` 試跑一次驗收流程
**待評測目標**：`.claude/agents/mod-eval-report.md`
**Schema 提供者**：使用者（人工整理，非由任何 Agent 自動產生——對應第十一節「指標規劃仍需呼叫方的領域判斷，不由通用模組代為決定」）

### 評測 Schema

| 指標 | 測量方式 | 通過門檻 | 關鍵度 |
|---|---|---|---|
| Frontmatter 完整性 | 檢查 YAML frontmatter 是否包含 `name`、`description`、`tools`、`model` 欄位（若內容依賴 Skill，還須有 `skills` 欄位） | 所有必要欄位皆存在 | 關鍵 |
| Description 三段式公式 | description 是否可清楚切分為「做什麼功能。觸發情境/關鍵字。所屬層級與可重用範疇。」三段，且第三段有明確標示所屬層級（①②③） | 三段皆可辨識且對應到位 | 關鍵 |
| 命名前綴正確 | 檔名與 frontmatter 的 `name` 是否使用對應層級的前綴（此檔案屬③能力模組層，應為 `mod-` 前綴） | 前綴完全符合 | 關鍵 |
| 五段落結構完整 | Body 是否包含「角色定位」「核心原則」「工作流程」「輸出格式」「限制邊界」五個標題區塊 | 五個標題皆存在 | 關鍵 |
| 無領域判斷邊界聲明 | 核心原則或限制邊界中，是否有明確一句話聲明「不自創判斷標準／不代呼叫方做領域判斷」（③能力模組層應具備此邊界聲明） | Body 中可找到至少一句對應聲明 | 非關鍵 |

### orch-main 分工說明

> 分工說明：本次由 `mod-eval-report` 負責依評測 Schema 比對目標檔案並產出結構化報告，`orch-main` 僅負責路由與原樣呈現結果，未自行改寫或簡化。

### 評測報告（`mod-eval-report` 回傳原文）

**整體判定**：✅ 通過

| 指標 | 通過門檻 | 實際觀察 | 結果 | 依據 |
|---|---|---|---|---|
| Frontmatter 完整性 | 所有必要欄位皆存在（含 `skills`，因內容依賴 Skill） | frontmatter 含 `name: mod-eval-report`、`description`、`tools: Read, Write`、`skills: evaluation-schema-design`、`model: sonnet`，五個欄位皆齊全 | ✅ | 第 2–6 行 |
| Description 三段式公式 | 三段皆可辨識且對應到位 | 第一段「依呼叫方提供的評測 Schema…具體落差清單。」（做什麼功能）；第二段「當任何工作流或人員完成產出後需要驗收其是否符合既定品質指標、或需要對照 Schema 檢視報告數據時使用。」（觸發情境）；第三段「屬於③能力模組層，無領域判斷，可被任何①②層呼叫，是可追溯性評測迴圈中「執行填報」階段的通用模組。」（層級，明確標示③並提及①②） | ✅ | 第 3 行 |
| 命名前綴正確 | 前綴完全符合（③能力模組層應為 `mod-`） | 檔名為 `mod-eval-report.md`，frontmatter `name: mod-eval-report`，兩者皆使用 `mod-` 前綴 | ✅ | 檔名；第 2 行 |
| 五段落結構完整 | 五個標題皆存在 | 「角色定位」（第 9 行）、「核心原則」（第 15 行）、「工作流程」（第 22 行）、「輸出格式」（第 30 行）、「限制邊界」（第 57 行）皆存在 | ✅ | 第 9、15、22、30、57 行 |
| 無領域判斷邊界聲明 | Body 中可找到至少一句對應聲明 | 角色定位段：「你不判斷『這個輸出該用什麼指標評測』、不自創判定標準——那是呼叫方的領域判斷」；限制邊界段：「不自創評測指標或判定標準，只使用呼叫方提供的 Schema」，兩處皆明確聲明 | ✅ | 第 13 行；第 59 行 |

**落差清單**：無不通過項目。

**與上次評測比較**：本次任務未提供上次評測報告，無對照資料。

### 觀察

- `orch-main` 正確判斷此請求屬於「依 Schema 比對輸出」的職責，路由給 `mod-eval-report`，自己未動手評測，符合①協調層「不執行領域工作，只做路由」的邊界。
- `mod-eval-report` 全程只依給定 Schema 執行比對，未自創判定標準，每筆判定皆附具體行號依據，符合可追溯性要求。
- 本次結果為全數通過、無落差，尚未累積到可觀察「重複出現的修正建議」的案例；待後續有實際落差案例時，再評估第 4 階段（優化）是否值得拆成獨立元件。

---

## 試跑二：驗收 `.claude/agents/` 全體 15 個 Agent 定義檔

**日期**：2026-10-07
**觸發方式**：使用者請求（延續專案優化流程——同日已完成一輪修復：mod-registry-sync 工具白名單補 `mcp__notion`、目錄改小寫、personal-assistant 移至使用者層級、文件同步——隨後對修復後的全體定義檔執行批量驗收）
**待評測目標**：`.claude/agents/` 下全部 15 個 Agent 定義檔（orch-main、6 個 wf-、8 個 mod-）
**Schema 提供者**：使用者（人工整理。六指標版＝試跑一的五指標，加上本次新增的「依賴涵蓋一致性」——該指標的設計動機正是試跑一 Schema 抓不到工具白名單缺口的教訓）

### 評測 Schema

| 指標 | 測量方式 | 通過門檻 | 關鍵度 |
|---|---|---|---|
| Frontmatter 完整性 | YAML frontmatter 是否包含 `name`、`description`、`tools`、`model` 欄位（內容依賴 Skill 者須有 `skills` 欄位；依賴 MCP 者須有對應 `mcp__` 白名單項目） | 所有必要欄位皆存在 | 關鍵 |
| Description 三段式公式 | description 是否可清楚切分為「做什麼功能。觸發情境/關鍵字。所屬層級與可重用範疇。」三段，且第三段有明確標示所屬層級（①②③） | 三段皆可辨識且對應到位 | 關鍵 |
| 命名前綴正確 | 檔名與 frontmatter 的 `name` 是否使用對應層級前綴（①`orch-`、②`wf-`、③`mod-`） | 前綴完全符合 | 關鍵 |
| 五段落結構完整 | Body 是否包含「角色定位」「核心原則」「工作流程」「輸出格式」「限制邊界」五個標題區塊 | 五個標題皆存在 | 關鍵 |
| 無領域判斷邊界聲明 | 僅適用③能力模組層：核心原則或限制邊界中是否有明確聲明「不自創判斷標準／不代呼叫方做領域判斷」 | 至少一句對應聲明 | 非關鍵 |
| 依賴涵蓋一致性 | Agent body 內文明確宣告的依賴（使用的 Skill、MCP 服務、需透過 Agent 工具委派子任務）是否都有 frontmatter 對應（`skills` 欄位／`tools` 白名單內的 `mcp__` 項目／`tools` 含 `Agent`） | 宣告的依賴全部有對應欄位 | 關鍵 |

**範圍界定（呼叫方補充）**：指標 5 僅適用 mod- 開頭檔案，①②層標 N/A；指標 6 的依賴以 body 內文明確宣告者為準，外部依賴的實際可用性（如 pptx Skill、Notion MCP 是否已配置）不在評測範圍；frontmatter 額外欄位（permissionMode、maxTurns、mcpServers）不列為扣分項。

### orch-main 分工說明

> 分工說明：本次由 `mod-eval-report` 負責依評測 Schema 逐檔比對全部 15 個目標檔案並產出報告，`orch-main` 僅負責路由與原樣呈現結果，未自行改寫或簡化。此路徑同時驗證了 subagent 內再透過 Agent 工具委派③層模組的巢式協作可正常運作。

### 評測報告（`mod-eval-report` 回傳原文）

**Schema 來源**：呼叫方提供（6 指標版，含範圍界定補充）　**整體判定**：✅ 通過

#### 一、每檔一行彙總表

| 檔案 | 指標1 Frontmatter完整性 | 指標2 三段式 | 指標3 前綴 | 指標4 五段落 | 指標5 無領域判斷聲明 | 指標6 依賴一致性 | 整體判定 |
|---|---|---|---|---|---|---|---|
| orch-main.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| wf-news-digest.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| wf-data-media.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| wf-data-academic.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| wf-data-market.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| wf-design.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| wf-dev.md | ✅ | ✅ | ✅ | ✅ | N/A | ✅ | ✅ 通過 |
| mod-web-scraper.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-summarizer.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-diagram-generator.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-slide-generator.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-test-runner.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-script-writer.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-registry-sync.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |
| mod-eval-report.md | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ 通過 |

#### 二、❌ 項目具體依據

本次評測無任何 ❌ 項目。為滿足可追溯性要求，列出數筆「非顯而易見」判定的觀察依據（皆為通過判定）：

- **指標2（wf-news-digest.md L3）**：description 共 4 句，前 2 句（「每日蒐集重要新聞事件…避免跨日重複蒐集。」「協調 mod-web-scraper 與 mod-summarizer…完整管線。」）同屬功能描述段，可與觸發情境段（「當需要產出每日新聞摘要…時使用」）、層級段（「屬於②領域工作流層，僅本領域，通常由排程觸發，無人值守執行」）清楚切分為三段，第三段標示②。mod-diagram-generator.md L3 同為 4 句結構，同樣可切分，判定通過。
- **指標4（wf-news-digest.md L35、L49）**：body 含第 6 個標題「狀態記錄檔格式（`news-digest-state.json`）」（L35），且「輸出格式」標題帶註記後綴（L49）。Schema 僅要求五個標題區塊存在，未禁止額外區塊或後綴，五者皆存在，判定通過。
- **指標5（mod- 層 8 檔）對應聲明位置**：mod-web-scraper L46「不判斷內容的重要性、真偽或可信度評分（交給呼叫方…）」／L49「不代替呼叫方做可信度判斷」；mod-summarizer L48「不對摘要內容做領域專業判斷」；mod-diagram-generator L20「不代替呼叫方做內容決策」／L40「不判斷業務邏輯或架構設計的合理性」；mod-slide-generator L37「不自行決定簡報的敘事重點或內容取捨」；mod-test-runner L46「不判斷『這次修改該測什麼』」；mod-script-writer L41「不自行決定內容重點取捨」；mod-registry-sync L48「不判斷執行結果的成功/失敗，只忠實記錄呼叫方提供的判斷」；mod-eval-report L59「不自創評測指標或判定標準」。均至少一句對應，判定通過。
- **指標1／指標6（mod-registry-sync.md L4）**：tools 含 `mcp__notion`，與 body L25「透過 Notion MCP 寫入登記簿」、L26「從 Notion 登記簿讀取」的 MCP 依賴宣告對應。body 未宣告任何 Skill 依賴，故無需 skills 欄位。判定通過。
- **指標1／指標6（orch-main.md、wf-news-digest.md、wf-design.md、wf-dev.md）**：frontmatter 無 skills 欄位，但 body 亦未宣告任何 Skill 依賴，僅宣告 Agent 工具委派（orch-main L30；wf-news-digest L16/L29/L31；wf-design L12/L24/L26；wf-dev L24/L26），tools 均含 Agent，對應一致。判定通過。
- **指標1／指標6（mod-slide-generator.md L5、L15）**：body L15 明確宣告「本模組依賴 `pptx` 這份 Skill」，frontmatter L5 有 `skills: pptx` 對應。依範圍界定，pptx Skill 的實際存在性不在本次評測範圍。判定通過。
- **範圍界定遵循**：permissionMode／maxTurns（wf-news-digest L5-L6）、mcpServers（mod-registry-sync L5）等額外欄位依呼叫方補充未列為扣分項。

#### 三、全體統計與整體判定

- **通過**：15／15 檔（關鍵指標 1~4、6 全數通過；mod- 層 8 檔非關鍵指標 5 亦全數通過）
- **不通過**：0 檔
- **整體判定**：✅ 通過

#### 備註

- 評測模組另指出：呼叫方未提供第一次試跑的報告作對照，故本次報告不含跨次比對；如需追蹤兩次試跑差異，後續可提供前次報告一併比對。
- 評測過程未修改任何目標檔案；目標檔案位於 `D:\AI-Agent-Skill\.claude\agents\`（orch-main、wf-news-digest、wf-data-media、wf-data-academic、wf-data-market、wf-design、wf-dev、mod-web-scraper、mod-summarizer、mod-diagram-generator、mod-slide-generator、mod-test-runner、mod-script-writer、mod-registry-sync、mod-eval-report）。

### 觀察

- `orch-main` 正確將批量驗收請求路由給 `mod-eval-report`，自己未動手評測；本次同時驗證「subagent 內透過 Agent 工具再委派③層模組」的巢式協作路徑可正常運作（試跑一是使用者直接觸發單一②③路徑，本次多了①→②/③→③的完整深度）。
- 新增的指標 6（依賴涵蓋一致性）正是本次修復項目的驗收：`mod-registry-sync` 的 `mcp__notion` 白名單項目與 body 宣告對應一致，判定通過。此類缺口在試跑一的五指標 Schema 下不會被抓到（試跑一僅檢查欄位存在，不檢查依賴對應），已由本次 Schema 版本補上。
- `mod-eval-report` 在無任何 ❌ 項目的情況下，仍主動對「非顯而易見的通過判定」附上行號依據（如 4 句式 description 的切分理由、額外標題區塊不扣分的理由），可追溯性要求落實到位，未因全數通過而省略依據。
- 兩次試跑皆全數通過，仍無「重複出現的落差模式」可觀察；差距在於本次目標從單檔擴及全體、Schema 從五指標增為六指標。待日後出現實際落差案例（尤其同類缺口反覆出現）時，再評估第 4 階段（優化）是否值得拆成獨立元件——本次新增指標的經驗（Schema 本身也會隨教訓演進）可作為日後 Schema 維護的參照。
- 跨次比對的適用性備註：試跑一（單檔、五指標）與試跑二（全體、六指標）範圍不同，逐項跨次比對本質上不對齊；未來若對同一批目標反覆評測，提供前次報告做對照才有意義。
