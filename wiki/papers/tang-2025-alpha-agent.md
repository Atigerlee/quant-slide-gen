---
title: "AlphaAgent: LLM-Driven Alpha Mining with Regularized Exploration to Counteract Alpha Decay (Tang et al., 2025, KDD '25)"
type: paper
tags:
  - quantitative-finance
  - alpha-mining
  - large-language-models
  - multi-agent
  - alpha-decay
  - ast
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "AlphaAgent"
  - "Tang et al. 2025"
  - "KDD 2025 AlphaAgent"
asset_class: Equity
timeframe: Daily
references:
  - "[[wiki/concepts/alpha-decay|Alpha 衰減機制與防禦]]"
  - "[[wiki/concepts/formulaic-alpha|公式型 Alpha 因子架構]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
  - "[[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]"
  - "[[wiki/papers/bailey-prado-2015-pbo|回測過度擬合機率 (PBO)]]"
---

# AlphaAgent: LLM-Driven Alpha Mining with Regularized Exploration to Counteract Alpha Decay

## 論文背景與核心概述

本論文發表於頂級資料探勘會議 **ACM SIGKDD 2025**，由中山大學、新南威爾斯大學、南洋理工大學等研究團隊共同研發。

論文針對量化因子投資中的核心痛點——**Alpha 衰減（Alpha Decay）**展開研究。傳統遺傳程式規劃（GP）與強化學習（RL）容易陷入過度擬合（p-hacking）而忽略經濟直覺；而新興的純大型語言模型（LLM）方法則容易退化為已知常見因子（如 RSI、傳統動量反轉），造成嚴重的**因子擁擠（Factor Crowding）**並加速衰減。

AlphaAgent 提出了融合「領域假說導向」與「符號正則化探索」的自主多 Agent 閉環架構，透過三大關鍵正則化機制——**獨創性強制（Originality Enforcement via AST Isomorphism）**、**假說對齊（Hypothesis-Factor Alignment）**與**複雜度控制（Complexity Control）**，成功在中國 CSI 500 與美國 S&P 500 兩大市場實現顯著抗衰減的高品質因子挖掘。

---

## 核心數學模型與最佳化目標

### 1. 正則化最佳化目標

將 Alpha 因子挖掘定義為帶有正則化約束的交替最佳化問題：
$$f^* = \arg\max_{f \in \mathcal{F}} \left[ \mathcal{L}(f(\mathbf{X}), \mathbf{y}) - \lambda \mathcal{R}_g(f, h) \right]$$
其中：
* $\mathcal{F}$：候選因子表達式空間。
* $\mathbf{X} \in \mathbb{R}^{N \times T \times D}$：特徵矩陣（$OHLCV$ 等基礎價量數據）。
* $\mathbf{y}$：次日報酬率真實標籤。
* $\mathcal{L}$：預測效能度量指標（如資訊比率 $\text{IR}$、$\text{IC}$ 或 $\text{RankIC}$）。
* $\mathcal{R}_g(f, h)$：多維度正則化懲罰項。

### 2. 正則化項 $\mathcal{R}_g(f, h)$ 的數學拆解

$$\mathcal{R}_g(f, h) = \alpha_1 \cdot \text{SL}(f) + \alpha_2 \cdot \text{PC}(f) + \alpha_3 \cdot \text{ER}(f, h)$$
1. **表達式長度（Symbolic Length, $\text{SL}(f)$）**：語法樹節點數。
2. **自由參數數量（Parameter Count, $\text{PC}(f)$）**：回看窗口長度等自由調參數量。
3. **探索正則化項（Exploration Regularization, $\text{ER}(f, h)$）**：
   $$\text{ER}(f, h) = \beta_1 \cdot \mathcal{S}(f) + \beta_2 \cdot \mathcal{C}(h, d, f) + \beta_3 \cdot \log(1 + |\mathcal{F}_f|)$$

### 3. 三大核心防衰減機制

#### A. 抽象語法樹最大子樹同構（AST Isomorphism Originality）
將因子表達式解析為 AST 樹 $T(f)$。計算新因子 $f$ 與現有因子庫 $\mathcal{Z} = \{\phi_1, \dots, \phi_N\}$（如 [[wiki/papers/kakushadze-2016-101-formulaic-alphas|Alpha101]]）的最大子樹相似度：
$$s(f_i, f_j) = \max_{t_i \subseteq T(f_i), t_j \subseteq T(f_j)} \{ |t_i| : t_i \cong t_j \}$$
$$\mathcal{S}(f) = \max_{\phi \in \mathcal{Z}} s(f, \phi)$$
有效懲罰與現有因子過於相似的結構，強迫 Agent 探索未被市場擁擠的創新市場無效率。

#### B. 假說與因子語義一致性打分（Hypothesis Alignment）
$$\mathcal{C}(h, d, f) = \alpha c_1(h, d) + (1 - \alpha) c_2(d, f)$$
* $c_1(h, d) \in [0, 1]$：評估因子文字描述 $d$ 是否忠實呈現金融假說 $h$。
* $c_2(d, f) \in [0, 1]$：評估數學表達式 $f$ 是否與文字描述 $d$ 保持邏輯一致。
此項防範「聲稱捕捉流動性但公式中完全無成交量資訊」的虛假因子。

#### C. 特徵簡約性懲罰
$\log(1 + |\mathcal{F}_f|)$ 懲罰過度濫用原始特徵欄位，鼓勵奧坎剃刀原則。

---

## 三 Agent 協同架構 (Multi-Agent Architecture)

1. **Idea Agent（金融假說生成）**：
   * 整合領域文獻與前期回測反思（Reflective feedback），透過思維鏈（Chain-of-Thought）產出結構化假說 $h = (\text{Observation}, \text{Knowledge}, \text{Justification}, \text{Specification})$。
2. **Factor Agent（符號化因子建構）**：
   * 依據假說 $h$ 調用運算子庫 $\mathcal{O}$ 組合因子 AST，維護成功與失敗經驗庫（Memory Bank）。
3. **Eval Agent（執行驗證與回測回饋）**：
   * 執行語法解析、數值穩定性檢驗，並於 Qlib 框架中進行滾動回測（計算 IC, RankIC, ICIR, IR, AR, MDD）。

---

## 實證成果 (Empirical Benchmarks, 2021-2024)

在長達 4 年（2021/01～2024/12）橫跨牛熊週期的樣本外回測中：
* **中國 CSI 500**：
  * 年化超額報酬（Annualized Excess Return, $\text{AR}$）達 **$11.00\%$**，資訊比率 $\text{IR} = 1.488$，最大回撤 $\text{MDD} = -9.36\%$，$\text{RankIC} \approx 0.025$。
* **美國 S&P 500**：
  * 年化超額報酬達 **$8.74\%$**，$\text{IR} = 1.0545$，$\text{MDD} = -9.10\%$。
* **抗衰減表現**：
  * 傳統 Alpha158、遺傳演算法（GP）與 RSI 因子的 IC 隨時間衰減至趨近於 0，而 AlphaAgent 保持平穩，Hit Ratio 提升 $81\%$。

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* **截面標準化洩漏**：AlphaAgent 在輸入 LightGBM 前採用截面 Z-Score 標準化：
  $$z_{i, t} = \frac{x_{i, t} - \mu_t}{\sigma_t}$$
  若計算 $\mu_t, \sigma_t$ 時納入了全市場當日收盤後才獲知的資訊或包含尚未上市標的，將引入前視偏差。必須嚴格保證僅使用當期已可獲得的截面資料。

### 2. 存活者偏差（Survivorship Bias）
* 回測採用 Baostock (CSI 500) 與 Yahoo Finance (S&P 500)。若使用當前靜態成分股進行 2015～2024 年長週期回測，將剔除大量退市及爆雷公司，顯著美化超額報酬率。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* **LLM 預訓練資料污染（Data Contamination）**：LLM（GPT-3.5、GPT-4、DeepSeek-R1）的訓練語料中已包含大量公開財經文獻與 Alpha101 程式碼，可能在無意識中「記憶」了歷史有效模式，而非真正現場推理發現。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* 論文回測設定：CSI 500 買入 0.05%、賣出 0.15%；S&P 500 僅賣出 0.05%。在實盤中，小盤股（CSI 500）流動性吃緊時的**衝擊成本與滑價**常超過 0.2%～0.3%，且頻繁 Top-50 調倉的實際摩擦成本可能侵蝕過半超額利潤。

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 策略固定買入預測報酬前 50 檔並剔除後 5 檔（Top-50 Dropout）。若組合資產管理規模（AUM）擴大至數億人民幣，前 50 檔中流動性較差的股票將無法迅速建倉。

### 6. 環境體系轉換（Regime Shift）
* 論文指出 2024 年初中國 A 股發生極端的微盤股流動性踩踏（Size factor crash）。若 AlphaAgent 生成之因子隱含小市值暴露，一旦市場風格急劇轉向高股息紅利股，策略將面臨巨大回撤。

---

## 關聯概念與筆記
* 概念連結：[[wiki/concepts/alpha-decay]]、[[wiki/concepts/formulaic-alpha]]、[[wiki/concepts/backtest-metrics]]
* 論文延伸：[[wiki/papers/kakushadze-2016-101-formulaic-alphas]]、[[wiki/papers/bailey-prado-2015-pbo]]
