---
title: "R&D-Agent-Quant: A Multi-Agent Framework for Data-Centric Factors and Model Joint Optimization (Li et al., NeurIPS 2025)"
type: paper
tags:
  - quantitative-finance
  - multi-agent
  - factor-model-co-optimization
  - contextual-bandits
  - co-steer
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "R&D-Agent(Q)"
  - "Li et al. NeurIPS 2025"
  - "Microsoft RD-Agent"
asset_class: Equity
timeframe: Daily
references:
  - "[[wiki/concepts/formulaic-alpha|公式型 Alpha 因子架構]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
  - "[[wiki/papers/tang-2025-alpha-agent|AlphaAgent]]"
---

# R&D-Agent(Q): Data-Centric Factors & Model Joint Optimization (NeurIPS 2025)

## 論文背景與核心概述

本論文發表於神經資訊處理系統大會 **NeurIPS 2025**，由卡內基美隆大學（CMU）、微軟亞洲研究院（MSRA）與牛津大學團隊聯合研發。

論文指出現代量化研究面臨「自動化程度低」、「代碼可解釋性弱易幻覺」以及「因子挖掘與模型研發割裂（Siloed Optimization）」三大限制。R&D-Agent(Q) 提出了首個以數據為中心的全棧量化研發多智能體系統，將研發流程解耦為五大專業單元，並透過**上下文多臂老虎機（Contextual Thompson Sampling Bandit）**自適應調度因子端與模型端的協同演化，實現以低於 10 美元的 API 成本大幅超越傳統 SOTA 因子庫與深度模型。

---

## 系統架構與數學建模

### 1. 五大功能單元閉環

1. **規格單元 (Specification Unit)**：形式化定義情境元組 $\mathcal{S} = (\mathcal{B}, \mathcal{D}, \mathcal{F}, \mathcal{M})$，隔離底層代碼細節，杜絕數據洩漏。
2. **合成單元 (Synthesis Unit)**：基於歷史假說森林生成結構化假說，並分解為具體任務 DAG。
3. **實現單元 (Implementation Unit - Co-STEER)**：基於知識樹檢索與圖依賴拓撲排序，實現可自修復的 Python/PyTorch 代碼生成。
4. **驗證單元 (Validation Unit)**：去重過濾（剔除與 SOTA 庫相關性 $\ge 0.99$ 者）並調用 Qlib 進行嚴格回測。
5. **分析單元 (Analysis Unit)**：評估實驗成敗並向合成單元反思反饋。

### 2. 因子與模型協同調度：上下文 Thompson 抽樣 (Contextual Bandit)

在每一輪 $t$，系統觀察 8 維量化績效狀態向量：
$$\mathbf{x}_t = \left[ \text{IC}, \text{ICIR}, \text{RankIC}, \text{RankICIR}, \text{ARR}, \text{IR}, -\text{MDD}, \text{SR} \right]^\top \in \mathbb{R}^8$$
動作空間為 $\mathcal{A} = \{\text{factor}, \text{model}\}$。對每個動作 $a$，維護高斯後驗分布 $\tilde{\boldsymbol{\theta}}^{(a)} \sim \mathcal{N}(\boldsymbol{\mu}^{(a)}, (\mathbf{P}^{(a)})^{-1})$，抽取預期收益：
$$\hat{r}^{(a)} = (\tilde{\boldsymbol{\theta}}^{(a)})^\top \mathbf{x}_t$$
選擇最大者執行，並在獲得真實績效改進後進行貝氏共軛更新：
$$\mathbf{P}^{(a_t)} \leftarrow \mathbf{P}^{(a_t)} + \frac{1}{\sigma^2} \mathbf{x}_t \mathbf{x}_t^\top, \quad \boldsymbol{\mu}^{(a_t)} \leftarrow (\mathbf{P}^{(a_t)})^{-1} \left( \mathbf{P}^{(a_t)} \boldsymbol{\mu}^{(a_t)} + \frac{1}{\sigma^2} r_t \mathbf{x}_t \right)$$

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* 規格單元嚴格只向 LLM 揭露數據 Schema（欄位名稱與型態），嚴禁將時間軸區間或未來收益截面洩漏給提示詞，防止 LLM 無意識「背誦」未來行情。

### 2. 存活者偏差（Survivorship Bias）
* CSI 300 / CSI 500 / NASDAQ 100 實驗須落實逐日退市動態更新，避免因只選當前成分股產生虛擬 Alpha。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* 因子端與模型端頻繁交替調參易增加整體自由度。必須配合樣本外 Walk-forward 測試與 PBO 檢定，避免 Bandit 策略過度追逐訓練集特定局部噪聲。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* 論文設定 A 股買入 0.05%、賣出 0.15% 加 5 元最低佣金；美股無漲跌停但有 0.1% 滑價。實盤高波時若未計委託衝擊成本，超額收益將大打折扣。

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 因子庫動態去重門檻設為 0.99，但若多個因子相關性在 0.8～0.9 之間，依然可能引發組合層面在特定高流動性板塊的過度集中。

### 6. 環境體系轉換（Regime Shift）
* 雖然 Bandit 機制能在因子與模型間動態切換，但在劇烈的市場流動性危機（如熔斷）時，模型權重調整可能落後於環境劇變。

---

## 關聯概念與筆記
* 概念連結：[[wiki/concepts/formulaic-alpha]]、[[wiki/concepts/backtest-metrics]]
* 論文延伸：[[wiki/papers/tang-2025-alpha-agent]]、[[wiki/papers/tang-2026-alphaagentevo-arl]]
