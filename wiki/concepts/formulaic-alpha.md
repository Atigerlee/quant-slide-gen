---
title: "公式型 Alpha 因子架構 (Formulaic Alpha Architecture)"
type: concept
tags:
  - quantitative-finance
  - formulaic-alpha
  - feature-engineering
  - operator-algebra
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "Formulaic Alpha"
  - "Alpha Operators"
  - "價量公式因子"
asset_class: Equity
timeframe: Daily
references:
  - "[[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]"
  - "[[wiki/papers/tang-2025-alpha-agent|AlphaAgent]]"
  - "[[wiki/concepts/alpha-decay|Alpha 衰減機制]]"
---

# 公式型 Alpha 因子架構 (Formulaic Alpha Architecture)

## 定義與基本概念

在量化投資領域，**公式型 Alpha（Formulaic Alpha）**是指將市場原始數據（如股價、成交量、委託單簿、財務數據等）透過精確定義的數學函數與算子（Operators），直接映射為對未來資產報酬率預測值的代數表達式：
$$f: \mathbf{X}_t \to \hat{r}_{t+1}$$
其中 $\mathbf{X}_t \in \mathbb{R}^{N \times D}$ 為 $t$ 時刻全市場 $N$ 檔標的的特徵向量，$\hat{r}_{t+1}$ 為標的在 $t+1$ 期的預期相對收益信號。

公式型 Alpha 的優勢在於：
1. **運算高效且符號化自包含**：可由純代數直譯器解析並快速向量化回測。
2. **易於組合**：各因子輸出可經截面標準化後線性或非線性疊加為 Mega-Alpha。
3. **便於機器自動挖掘**：可透過遺傳編程（GP）、強化學習（RL）或 LLM 多 Agent 進行語法樹搜索。

---

## 核心運算子代數 (Operator Algebra)

公式型 Alpha 依賴截面（Cross-sectional）與時序（Time-series）兩大維度的基礎運算子組合：

### 1. 截面運算子 (Cross-Sectional Operators)
* **截面排名（Cross-Sectional Rank）**:
  $$\text{rank}(x_i) = \frac{\text{Rank}(x_i) - 1}{N - 1} \in [0, 1]$$
  將當日全市場股票的特定指標進行升序排序並歸一化。
* **行業/板塊中性化（Industry Neutralization）**:
  $$\text{indneutralize}(x_i, g) = x_i - \frac{1}{|G_g|} \sum_{j \in G_g} x_j$$
  扣除同產業群組均值，消除行業風格暴露。
* **槓桿尺度縮放（Scale / L1 Normalization）**:
  $$\text{scale}(x_i, a) = a \cdot \frac{x_i}{\sum_{j=1}^N |x_j|}$$
  將因子多空權重總和約束在槓桿倍數 $a$（通常 $a=1$）。

### 2. 時間序列運算子 (Time-Series Operators)
* **時序延遲（Delay）**:
  $$\text{delay}(x_t, d) = x_{t-d}$$
* **時序差分（Delta）**:
  $$\text{delta}(x_t, d) = x_t - x_{t-d}$$
* **時序分位數排名（Ts_Rank）**:
  滾動回看窗口 $d$ 天內，當前值在歷史區間的分位數排名：
  $$\text{ts\_rank}(x_t, d) = \frac{\sum_{\tau=0}^{d-1} \mathbb{I}(x_t \ge x_{t-\tau})}{d}$$
* **線性衰減均線（Decay Linear）**:
  賦予較近期數據更高權重的加權移動平均：
  $$\text{decay\_linear}(x_t, d) = \frac{\sum_{k=0}^{d-1} (d - k) x_{t-k}}{\sum_{k=0}^{d-1} (d - k)} = \frac{2}{d(d+1)} \sum_{k=0}^{d-1} (d - k) x_{t-k}$$

---

## 因子信號生成機制

* **動量型（Momentum）**:
  $$\alpha_{\text{mom}} = \ln\left(\frac{P_{t-1}^{\text{close}}}{P_{t-1}^{\text{open}}}\right)$$
* **均值回歸型（Mean-Reversion / Contrarian）**:
  $$\alpha_{\text{mr}} = -\ln\left(\frac{P_t^{\text{open}}}{P_{t-1}^{\text{close}}}\right)$$

---

## 關聯文獻與概念
* 論文精讀：[[wiki/papers/kakushadze-2016-101-formulaic-alphas]]
* 因子挖掘機制：[[wiki/papers/tang-2025-alpha-agent]]
* 衰減與過度擬合：[[wiki/concepts/alpha-decay]]、[[wiki/concepts/probability-of-backtest-overfitting]]
