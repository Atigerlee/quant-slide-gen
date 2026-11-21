---
title: "101 Formulaic Alphas (Kakushadze, 2016)"
type: paper
tags:
  - quantitative-finance
  - formulaic-alpha
  - cross-sectional-momentum
  - mean-reversion
  - factor-investing
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "WorldQuant 101 Alphas"
  - "Kakushadze 2016"
asset_class: Equity
timeframe: Daily
references:
  - "[[wiki/concepts/formulaic-alpha|公式型 Alpha 因子架構]]"
  - "[[wiki/concepts/alpha-decay|Alpha 衰減機制]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
  - "[[wiki/papers/tang-2025-alpha-agent|AlphaAgent LLM-Driven Alpha Mining]]"
---

# 101 Formulaic Alphas

## 論文背景與核心概述

本論文由前 WorldQuant 顧問、第比利斯自由大學教授 **Zura Kakushadze** 於 2015 年底撰寫（2016 年發表）。論文首度向學術界與公開市場公開了 101 個在對沖基金實際生產環境中運行的電腦程式碼公式型價量 Alpha 因子（其中 80 個於論文發表時仍在生產線上實盤交易）。

論文主要研究了 101 個真實世界 Alpha 的統計與經驗特性，指出在現代量化交易中，單一 Alpha 因子的訊號極為微弱（faint）且短暫（ephemeral），唯有透過建立包含數萬至數十萬個 Alpha 的「超級 Alpha（Mega-Alpha）」並利用內部撮合（internal crossing）節省交易成本，方能維持高夏普比率（Sharpe Ratio）。

---

## 核心數學模型與實證定律

### 1. 績效與因子特徵定義

對於第 $i$ 個 Alpha 因子（$i = 1, \dots, N$，此處 $N = 101$）：

* **年化夏普值（Annualized Daily Sharpe Ratio）**:
  $$S_i = \sqrt{252} \cdot \frac{P_i}{V_i}$$
  其中 $P_i$ 為平均每日損益（Dollar P&L），$V_i = \sigma_i I_i$ 為每日投資組合波動度，$\sigma_i$ 為日報酬率波動度，$I_i$ 為總投資額（Long + Short 總頭寸）。

* **每日換手率（Daily Turnover）**:
  $$T_i = \frac{D_i}{I_i}$$
  其中 $D_i$ 為平均每日交易美元額度（Dollar volume traded）。平均持倉天數為：
  $$\text{Holding Period}_i = \frac{1}{T_i}$$

* **每股盈餘美分（Cents-per-share）**:
  $$C_i = 100 \cdot \frac{P_i}{Q_i}$$
  其中 $Q_i$ 為每日交易股數（買進股數加賣出股數）。

### 2. 實證冪律關係（Power Law Scaling）

論文在美國股市 2,000 檔流動性最佳標的（回測區間 2010 年 1 月 4 日至 2013 年 12 月 31 日，共 1,006 個交易日）上實證發現：

* **報酬率與波動度呈現強烈冪律關聯**:
  $$R_i \sim \sigma_i^X \quad (X \approx 0.761)$$
  對數迴歸模型：
  $$\ln(R_i) = -3.509 + 0.761 \ln(\sigma_i) \quad (R^2 = 0.737, t = 16.65)$$
* **報酬率與換手率（Turnover）無統計顯著關聯**:
  加入 $\ln(T_i)$ 的多變量迴歸中，$\ln(T_i)$ 之迴歸係數為 $-0.023$（$t = -0.57$），證實**因子的換手率本身並不能解釋 Alpha 報酬率的高低**。
* **低兩兩相關性（Low Pair-wise Correlation）**:
  101 個因子的平均兩兩相關性僅為 $15.9\%$（中位數 $14.3\%$），具備優良的組合分散潛力。

---

## 因子運算子體系與信號分類

Kakushadze 將因子分為兩大操作機制：
1. **動量因子（Momentum / Trend-following）**：順應價格運動方向（例如 Alpha#101: `(close - open) / ((high - low) + 0.001)`）。
2. **均值回歸因子（Mean-reversion / Contrarian）**：逆向操作（例如 Alpha#42: `rank(vwap - close) / rank(vwap + close)`）。

### 運算子體系（Operator Algebra）
* **截面運算子**：
  * $\text{rank}(x)$：全市場截面排序並標準化至 $[0, 1]$。
  * $\text{indneutralize}(x, g)$：按產業/板塊分組扣除組內均值（行業中性化）。
  * $\text{scale}(x, a)$：縮放使得 $\sum |x_i| = a$（預設為 1，即槓桿標準化）。
* **時間序列運算子**：
  * $\text{ts\_rank}(x, d)$：滾動 $d$ 日的時間序列分位數排序。
  * $\text{decay\_linear}(x, d)$：權重線性衰減之滾動均值，權重為 $d, d-1, \dots, 1$。
  * $\text{correlation}(x, y, d)$、$\text{covariance}(x, y, d)$：滾動皮爾森相關係數與共變異數。

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* **Delay-0 vs Delay-1 陷阱**：
  論文中 101 個因子中有 4 個因子為 **Delay-0**（Alpha#42, 48, 53, 54），要求在當日收盤前（或開盤當下）以極接近該時點的價格成交。若回測系統假設能以當日收盤價（Close）撮合基於當日 Close 計算出的信號，即構成嚴重前視偏差。實務上絕大多數因子必須設定為 **Delay-1**（以 $t$ 日資料計算，於 $t+1$ 日開盤執行）。
* **除權息與拆股未適當平移**：使用後復權或前復權資料若未考慮除權息時間點，將導致未來的配息金額洩漏至歷史訊號中。

### 2. 存活者偏差（Survivorship Bias）
* 論文測試集為美國市場流動性前 2,000 檔股票。若股票池（Universe）採「靜態回溯」方式選取（如選取今日仍為 S&P 500 的成分股進行 2010 年回測），則完全排除了在 2010～2013 年間下市、被併購或破產的標的，嚴重虛增策略夏普值。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* 論文中的 101 個公式充滿了精細調節的經驗參數（例如 Alpha#59: `vwap * 0.728317 + vwap * (1 - 0.728317)`、滾動窗口 `16.2289` 天）。這些小數點參數極有可能是基因演算法或參數網格搜索過度擬合歷史噪聲的產物，參見 [[wiki/papers/bailey-prado-2015-pbo|Bailey et al. 回測過度擬合檢定]]。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* 論文表 1 明确載明：「**The performance figures are exclusive of any trading or transaction costs, price impact, etc.**」。
* 該 101 個因子的平均每日換手率高達 $54.56\%$（平均持倉期僅 2.39 天）。若計入千分之一的買賣手續費與借券成本，絕大部分毛利潤將被摩擦成本侵蝕殆盡。

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 高換手（Turnover $> 50\%$）策略的容量（Capacity）非常有限。對於中小型市值標的，委託單量若超過日均成交量（ADV）的 1%～2%，將引發顯著的市場衝擊成本（Market Impact），使實盤績效迅速崩潰。

### 6. 環境體系轉換（Regime Shift）
* 2010～2013 年為美股量化寬鬆（QE）初期的單邊多頭與低波動環境。該批因子多以短線價量均值回歸與微結構動量為主，在 2020 年新冠崩盤或 2022 年急遽升息引發的高波動體系下，均面臨極大的因子崩潰風險。

---

## 關聯概念與筆記
* 概念連結：[[wiki/concepts/formulaic-alpha]]、[[wiki/concepts/alpha-decay]]、[[wiki/concepts/backtest-metrics]]
* 論文延伸：[[wiki/papers/bailey-prado-2015-pbo]]、[[wiki/papers/tang-2025-alpha-agent]]
