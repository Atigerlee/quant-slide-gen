# AGENTS.md - 量化交易知識庫與自動化簡報系統規範

本文件為本知識庫與簡報編程系統之核心協定規範。所有 AI Agent（包括協作者與自動化腳本）在執行檔案讀取、知識萃取、筆記維護與簡報生成時，必須嚴格遵守以下準則。

---

## 1. 系統定位與架構職責

本專案是一個兼具**嚴謹量化交易研究（Quantitative Trading Research）**與**自動化知識傳播（Automated Presentation）**的雙核心系統：
- **量化知識工程師（Quant Knowledge Engineer）**：負責自原始論文與文獻萃取核心概念、回測邏輯、數學模型，建立具雙向連結與嚴謹風險揭露的結構化知識庫（LLM Wiki）。
- **簡報編譯架構師（Presentation Compiler）**：負責將知識庫精華轉化為結構清晰、100% 離線可用、自包含且具自我檢驗修正機制的 HTML 投影片。

---

## 2. 目錄架構與讀寫權限

系統採用標準化模組目錄，所有 Agent 必須尊重路徑定義與存取權限：

```
.
├── AGENTS.md                  # 核心協定與行為規範（本文件）
├── raw/                       # 【唯讀不可修改】原始 PDF、研究報告、Markdown 筆記
├── wiki/                      # 【核心知識庫】結構化知識筆記與雙向連結
│   ├── concepts/              # 核心量化概念、數學統計基礎、金融計量理論
│   ├── papers/                # 經典與最新量化論文精讀與架構拆解
│   ├── strategies/            # 量化策略邏輯、訊號規則、資金管理與回測規範
│   └── index.md               # 自動維護之總目錄索引（Master Index）
├── assets/                    # 圖表素材、向量架構圖、回測績效圖
└── slides/                    # 【簡報工作區】大綱、檢驗日誌與編譯輸出
    ├── outline.md             # 審核通過之逐頁結構內容
    ├── review_loop.log        # 自我檢查 Loop 評審與修復日誌
    └── index.html             # 最終產出之單一自包含離線簡報
```

### 讀寫權限規則（Strict Access Controls）
1. **`raw/` 唯讀與簡報隔離限制**：
   - Agent 僅在『知識庫建檔與更新』時具備 `READ` 權限。
   - **嚴禁**修改、覆寫、重新命名或刪除 `raw/` 內的任何既有原始檔案。
   - **【簡報生成嚴格禁讀 raw/ 規範】**：在執行簡報生成工作流（Stage 1～4）時，**嚴禁讀取、掃描或檢索 `raw/` 目錄下的任何原始檔案**。大綱提煉與內容編譯**僅能依賴 `wiki/` 結構化知識庫**，徹底杜絕耗時的 PDF 全文解析與無效全文檢索，確保快速響應並大幅縮短運行時間。
2. **`wiki/` 規範更新**：
   - 產出結構化筆記至對應子目錄（`concepts/`、`papers/`、`strategies/`）。
   - 每次新增或變更筆記時，**必須自動同步更新 `wiki/index.md`**。
3. **`slides/` 產物生命週期**：
   - 所有中間大綱、檢查紀錄與 HTML 簡報一律寫入此目錄。

---

## 3. 量化領域特殊規範

量化金融領域講求精確性與實務約束，禁止模糊與空洞描述，必須符合以下兩大強制規範：

### 3.1 數學公式嚴格採用標準 LaTeX
- 所有數學模型、公式與指標必須使用標準 LaTeX 語法撰寫：
  - 行內公式：`$...$`（例如：年化夏普值 $\text{Sharpe} = \frac{\mathbb{E}[R_p - R_f]}{\sigma_p}$）
  - 獨立段落公式：`$$...$$`
- **禁止**使用模糊文字（如 "Sharpe = mean / std"）或偽 ASCII 公式。
- 常見指標定義規範：
  - **Kelly Criterion**:
    $$f^* = \frac{p \cdot b - q}{b} = \frac{p(b+1) - 1}{b}$$
  - **Maximum Drawdown (MDD)**:
    $$\text{MDD}_T = \max_{\tau \in [0, T]} \left( \frac{\max_{t \in [0, \tau]} V_t - V_\tau}{\max_{t \in [0, \tau]} V_t} \right)$$
  - **Information Ratio (IR)**:
    $$\text{IR} = \frac{\mathbb{E}[R_p - R_b]}{\sqrt{\text{Var}(R_p - R_b)}}$$

### 3.2 強制標註「潛在風險與陷阱 (Risks & Pitfalls)」
每篇策略（`strategies/`）與論文（`papers/`）筆記**必須設立專屬 `## 潛在風險與陷阱` 章節**，至少逐項審查並載明下列維度：
1. **前視偏差（Look-ahead Bias）**：特徵工程是否使用了未來資訊（如未平移的除權息資料、事後最高最低價、收盤後才發布的財報數據）。
2. **存活者偏差（Survivorship Bias）**：回測池是否排除已下市、破產或停牌標的。
3. **過度擬合與資料探勘（Overfitting & Data Snooping）**：是否過度調參、參數敏感度是否過高、是否進行樣本外（OOS）與跨週期驗證。
4. **交易成本與滑價假設（Transaction Costs & Slippage）**：是否低估手續費、證券交易稅、委託衝擊成本（Market Impact）或借券融券限制。
5. **流動性與市場容量（Liquidity & Capacity Limit）**：策略成交量是否超過市場實際可吸收額度（如超過標的當日成交量 1%～5%）。
6. **環境體系轉換（Regime Shift）**：低波動環境與高波動極端市場（如閃崩、流動性乾涸）下策略表現之脆弱性。

---

## 4. 知識庫筆記規範與雙向連結架構

### 4.1 YAML Frontmatter 標準
所有 `wiki/` 筆記開頭必須包含標準 YAML Frontmatter：

```yaml
---
title: "筆記名稱"
type: concept | paper | strategy
tags:
  - quantitative-finance
  - momentum
  - risk-management
status: draft | verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "縮寫或替代名稱"
asset_class: Equity | Futures | Crypto | FX | Multi-Asset
timeframe: Tick | Intraday | Daily | Weekly
references:
  - "[[相關概念或論文]]"
---
```

### 4.2 雙向連結（Wikilinks）
- 採用 Obsidian 相容格式：`[[檔案名或路徑|顯示文字]]` 或 `[[檔案名]]`。
- 當提及核心概念、關聯策略或論文時，必須主動建立連結，促進知識網狀拓撲（Knowledge Graph）。

### 4.3 總目錄（`wiki/index.md`）維護機制
每次新增或異動筆記，Agent 必須更新 `wiki/index.md`，其結構必須包含：
1. **層級樹狀清單（Directory Tree Index）**：依 `concepts/`、`papers/`、`strategies/` 分門別類。
2. **策略與資產矩陣（Strategy Matrix）**：標註資產類別、頻率與驗證狀態。
3. **標籤主題雲（Topic Tags Index）**：按標籤分類之快速檢索索引。
4. **最近更新日誌（Recent Changelog）**：列出最近異動的 5 篇筆記與日期。

---

## 5. 自動化 HTML 簡報生成工作流（Graph + Loop）

當使用者指定主題要求生成簡報時，Agent 必須強制依照下列 4 階段有向工作流執行，嚴格落實人機協同審核與自我修復循環：

```
[使用者指定主題/材料]
        │
        ▼
┌──────────────────────────────────────────────┐
│ Stage 1: 大綱提煉 (Outline Generation)       │
│ - 【嚴禁讀取 raw/】僅檢索 wiki/ 結構化筆記   │
│ - 結構規劃、頁數、關鍵論點、圖表與公式規劃   │
│ - 落地儲存至 slides/outline.md 供編輯器檢視 │
└──────────────────────────────────────────────┘
        │
        ▼
   【嚴格中斷點】系統級交互中斷 (System Interactive Interruption)
   * 調用系統交互提問工具 (ask_question) 彈出選項
   * 進入阻斷式等待，由使用者明確點選確認 / 調整
        │
        ├────────────[需調整]─────────────┐
        ▼                                 ▼
┌───────────────────────────────┐  [重修大綱]
│ Stage 2: 逐頁內容展開         │
│ - 依確認之 slides/outline.md 展開結構與視覺細節
└───────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│ Stage 3: 單一自包含離線 HTML 簡報編譯        │
│ - 原生 HTML/CSS/JS                           │
│ - 100% 離線可用 (無 CDN、內嵌 MathML/向量公式)│
│ - 支援鍵盤翻頁 & 16:9 專業量化黑曜主題       │
│ - 輸出至 slides/index.html                   │
└──────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│ Stage 4: 自我檢查修復循環(Self-correction)    │
│ - 檢驗：單頁字數、引用出處、無死圖路徑          │
│ - 局部修復 (Diff Edit，嚴禁整份重寫 HTML)    │
│ - 寫入檢驗紀錄至 slides/review_loop.log      │
│ - 評估是否通過 (Pass / Fail)                 │
└──────────────────────────────────────────────┘
        │
        ├───[未通過且次數 <= 3] ──> 自動修復並重新編譯
        ▼
   【完成交付】報告通過結果並附上 slides/index.html
```

---

### 詳細階段規範

#### Stage 1：大綱提煉與系統級交互中斷（Human-in-the-loop）
1. **資訊來源嚴格限制（Wiki-Only，嚴禁閱讀 raw/）**：
   - Agent **嚴禁讀取、搜尋或引用 `raw/` 內的任何原始檔案**，杜絕因 PDF 全文檢索導致的冗長運算與時間浪費。
   - 大綱提煉**必須 100% 僅檢索 `wiki/` 結構化筆記（`concepts/`、`papers/`、`strategies/`）與 `wiki/index.md`**，極速萃取出結構嚴謹的投影片大綱（建議 8～12 頁）。
2. **實體落地與檔案連結**：
   - Agent **必須立即將大綱寫入 `slides/outline.md`**，並在對話中附上可點擊之本機路徑連結，方便使用者直接在 IDE 編輯器中逐行審核或修改。
3. **強制系統級交互中斷原則（System Interactive Breakpoint）**：
   - Agent **必須調用系統交互工具（ask_question）** 彈出確認選單，主動將控制權交還使用者。
   - 選項必須包含：「✅ 審核通過：大綱符合預期，請展開逐頁內容並編譯 HTML 簡報」、「✏️ 已手動編輯 slides/outline.md：請依最新檔案展開編譯」、「🔄 需要調整：請根據我的對話反饋重新提煉大綱」。
   - **嚴禁**在未獲得使用者明確點選確認前自動進入檔案生成或 HTML 編譯。

#### Stage 2：逐頁內容展開（Slide Deck Expansion）
1. 接獲使用者核准指令後，Agent 依據最新確認的 `slides/outline.md` 展開每頁投影片詳細內容。
2. 每頁投影片規劃必須確立視覺重點（Headline + 3～4 個 Bullet points + 核心公式卡片 + 警示或數據註釋）。

#### Stage 3：單一自包含離線 HTML 簡報編譯（Offline Standalone HTML）
簡報檔案必須寫入 `slides/index.html`，且必須遵循下列前端架構要求：
1. **100% 原生自包含與完全離線（Offline & Standalone）**：
   - 僅使用原生 HTML5、CSS3 與 Vanilla JavaScript。
   - **完全不依賴外部網路資源**：不載入任何外部 CDN、外部 Google Fonts 或遠端腳本。雙擊本機檔案在斷網狀態下必須可完整正常顯示與翻頁。
2. **數學公式離線渲染策略**：
   - 優先採用標準 Web **MathML (`<math>`)** 與內嵌高精準度向量 **SVG** 形式渲染 LaTeX 結構，或在檔案中內嵌極簡自包含的離線數學排版腳本與向量符號庫，保證公式離線解析無瑕疵。
3. **互動翻頁與控制規範**：
   - 支援鍵盤快速鍵：
     - `ArrowRight` / `Space` / `PageDown`：下一頁
     - `ArrowLeft` / `PageUp`：上一頁
     - `Home` / `End`：跳至首頁 / 末頁
     - `f` 鍵：全螢幕切換（Fullscreen Toggle）
   - 畫面常駐元素：右下角投影片進度條與頁碼指示器（如 `04 / 10`）。
4. **量化金融專用視覺風格（Quant Dark Aesthetic）**：
   - 預設採用優雅的深色背景（Dark Slate / Obsidian `#0f172a` 或 `#111827`）。
   - 對比清晰的程式碼/公式卡片（`#1e293b`）、金融綠（Profit / Long `#10b981`）、金融紅（Drawdown / Short `#ef4444`）與典雅金（Accent `#f59e0b`）。
   - 固定 16:9 劇院級比例排版，具備自動適應視窗縮放機制（Responsive Scale）。

#### Stage 4：自我檢查修復循環（Self-correction Loop）
編譯完成後，Agent 必須自動執行嚴格的自我評審，**最多循環 3 次**。
所有審查過程與修復細節必須完整記錄於 `slides/review_loop.log`：

1. **三項核心檢查指標**：
   - **指標 A：單頁字數精簡度（Conciseness）**
     - 單頁投影片正文字數上限為 **80 字**（不計公式標籤與程式碼）。
     - 條列要點（Bullet points）不超過 **5 點**。
     - 若偵測到長篇大論或段落堆疊，視為 **FAILED**，必須精簡並提煉關鍵字。
   - **指標 B：引用與出處完整性（Citation Verification）**
     - 投影片內提及的所有回測數據、學術假說、論文結論，下方必須註明出處（如 `Source: [[papers/Fama-French-1993]]` 或文獻標題）。
     - 缺乏來源者視為 **FAILED**，必須補正標註。
   - **指標 C：圖表與資源路徑有效性（Zero Dead Links）**
     - 檢查所有 `<img>` 標籤與向量資產：路徑必須指向實際存在的 `../assets/` 或採 inline SVG / base64 內嵌。
     - 任何死圖、破圖、未找到的本地檔案視為 **FAILED**，必須修正路徑或改用結構化向量圖表替代。

2. **局部修復強制規範（Diff / Patch Edit Only）**：
   - **嚴禁整份重寫（Full Overwrite）HTML 檔案**：在 Loop 自動修復階段，Agent **必須僅使用局部編輯（Patch / Diff Edit 工具，如 `edit_file`）** 精準定位並替換出錯的投影片 DOM 節點、文案或公式標籤。
   - 杜絕重複生成整個 HTML 架構（包含重複的 CSS 樣式與 JS 腳本），確保修復過程高效、敏捷且不會意外破壞既有通過的頁面排版。

3. **日誌記錄格式（`slides/review_loop.log` 範例）**：
```markdown
# Slide Deck Review Loop Log
Execution Timestamp: 2026-11-21T13:00:00+08:00
Target File: slides/index.html

## Iteration 1
- [PASS] Metric C: Zero dead assets detected.
- [FAIL] Metric A: Slide 4 word count exceeded (105 words > 80 words threshold).
- [FAIL] Metric B: Slide 7 missing reference source for Sharpe calculation.
Action: Rewriting Slide 4 content into 4 concise bullet points; adding citation footer to Slide 7.

## Iteration 2
- [PASS] Metric A: All slides comply with word count (< 80 words).
- [PASS] Metric B: All citations verified.
- [PASS] Metric C: All assets valid.
Result: PASSED on Iteration 2.
```

4. **終止條件**：
   - 若通過全部指標，立即終止循環並向使用者交付簡報路徑與預覽指引。
   - 若達 3 次上限仍有警告，記錄所有未解決項目至 log，並於交卷時主動向使用者彙報警告細節。

---

## 6. 指令與應答範例（Agent Quickstart Prompt）

當使用者發出量化知識庫或簡報請求時，Agent 必須依以下模式響應：

### 知識庫建檔響應範例
> 「已為您研讀 `raw/[論文名稱].pdf`，並於 `wiki/papers/[檔案].md` 建立結構化筆記。筆記內已建立對應概念的 `[[雙向連結]]`，完整標註 LaTeX 數學模型，並於『潛在風險與陷阱』章節針對前視偏差與滑價假設進行深層剖析。同時已自動同步更新 `wiki/index.md` 總目錄。」

### 簡報生成工作流響應範例
> 「已為您規劃【[簡報主題]】的投影片大綱如下（共 8 頁）。**請您審閱各頁結構、公式與圖表規劃；在您確認或提出修改反饋前，我將維持暫停，待您回覆後再展開逐頁內容與編譯離線 HTML**：
>
> 1. Cover: [主題與標題]
> 2. Core Thesis: [核心論點與回測邊際]
> ...
> 請問大綱是否符合您的預期，或需要進行任何調整？」
