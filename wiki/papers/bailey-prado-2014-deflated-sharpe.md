---
title: "The Deflated Sharpe Ratio (Bailey & López de Prado, 2014)"
type: paper
tags:
  - quantitative-finance
  - deflated-sharpe-ratio
  - selection-bias
  - multiple-testing
  - non-normality
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "DSR"
  - "Deflated Sharpe"
  - "Bailey & López de Prado 2014"
asset_class: Multi-Asset
timeframe: Multi-Scale
references:
  - "[[wiki/papers/bailey-prado-2015-pbo|回測過度擬合機率 (PBO)]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
---

# The Deflated Sharpe Ratio (DSR)

## 論文背景與核心概述

由 **David H. Bailey** 與 **Marcos López de Prado** 於 2014 年發表於 *Journal of Portfolio Management*。論文指出現代量化藉由強大算力可測試數百萬個策略，導致嚴重的「選擇偏差（Selection Bias）」與「贏家的詛咒（Winner's Curse）」。傳統夏普比率（Sharpe Ratio, SR）在高階矩偏離常態（偏態與峰態）以及多重測試下會被大幅虛增。

為此，作者推導出在多重獨立測試下「預期最大夏普值」的極值分佈封閉解，並提出 **通縮夏普比率（Deflated Sharpe Ratio, DSR）**，直接對選擇偏差與非假定常態分配進行統計去膨脹校正。

---

## 核心數學模型

### 1. 多重測試下預期最大夏普比率 (Expected Maximum SR)

設有 $N$ 個獨立策略回測，其夏普值服從均值為 $\mathbb{E}[\{\widehat{\text{SR}}_n\}]$、變異數為 $\mathbb{V}[\{\widehat{\text{SR}}_n\}]$ 的常態分佈。當 $N \gg 1$ 時，最大夏普值的期望值為：
$$\mathbb{E}[\max_n \{\widehat{\text{SR}}_n\}] \approx \mathbb{E}[\{\widehat{\text{SR}}_n\}] + \sqrt{\mathbb{V}[\{\widehat{\text{SR}}_n\}]} \left( (1 - \gamma) Z^{-1}\left[1 - \frac{1}{N}\right] + \gamma Z^{-1}\left[1 - \frac{1}{N e}\right] \right)$$
其中 $\gamma \approx 0.5772$ 為歐拉-馬斯刻若尼常數（Euler-Mascheroni Constant），$Z$ 為標準常態累積分布函數。

### 2. 通縮夏普比率 (Deflated Sharpe Ratio, DSR)

在零假說 $H_0: \text{SR} = 0$ 下，將預期最大夏普值設為拒絕門檻 $\widehat{\text{SR}}_0$。DSR 定義為該策略真實夏普值大於 0 的統計機率：
$$\text{DSR} = \text{PSR}(\widehat{\text{SR}}_0) = Z\left[ \frac{(\widehat{\text{SR}} - \widehat{\text{SR}}_0)\sqrt{T - 1}}{\sqrt{1 - \hat{\gamma}_3 \widehat{\text{SR}} + \frac{\hat{\gamma}_4 - 1}{4}\widehat{\text{SR}}^2}} \right]$$
其中：
* $T$：樣本長度。
* $\hat{\gamma}_3, \hat{\gamma}_4$：策略日收益率的偏態（Skewness）與峰態（Kurtosis）。
* 若 $\text{DSR} < 0.95$，即便名義夏普比率高達 2.5，亦不能拒絕為統計假陽性（Statistical Fluke）。

### 3. 最優停止法則 (Optimal Stopping / 1/e Rule)

論文引用秘書問題（Secretary Problem）指出，面對候選參數網格，應隨機抽樣前 $1/e \approx 36.8\%$ 的策略衡量表現，隨後選取第一個超越該基準的策略即停止搜尋，能將過度擬合機率降至最低。

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* 若回測日曆對齊錯誤或在非對齊樣本上計算高階矩（偏態 $\hat{\gamma}_3$ 與峰態 $\hat{\gamma}_4$），會使分母標準誤失真，導致 DSR 估計值偏誤。

### 2. 存活者偏差（Survivorship Bias）
* 若僅對「未爆雷基金或現存成功標的」計算 DSR，測試池樣本將缺乏左尾極端虧損，從而低估 $\mathbb{V}[\{\widehat{\text{SR}}_n\}]$，使門檻 $\widehat{\text{SR}}_0$ 過低。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* **隱瞞嘗試次數（File Drawer Problem）**：DSR 高度敏感於獨立測試數 $N$。若研究員實際嘗試了 10,000 次卻僅回報 $N=100$，DSR 將被大幅高估。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* DSR 矯正的是統計估計誤差，而非交易摩擦。未計滑價的高頻夏普值即便通過 DSR 檢驗，實盤依然會虧損。

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 高夏普策略常呈現高負偏態（平時賺小錢、偶爾巨虧），若容量超限遭遇黑天鵝，高階矩修正項將急劇擴大。

### 6. 環境體系轉換（Regime Shift）
* $T$ 期的歷史報酬常態性假設在宏觀體系突變下破裂，極端肥尾事件可能超出樣本矩的描述範圍。

---

## 關聯概念與筆記
* 概念連結：[[wiki/concepts/probability-of-backtest-overfitting]]、[[wiki/concepts/backtest-metrics]]
* 論文延伸：[[wiki/papers/bailey-prado-2015-pbo]]
