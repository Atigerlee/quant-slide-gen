---
title: "The Probability of Backtest Overfitting (Bailey, Borwein, López de Prado, Zhu, 2015)"
type: paper
tags:
  - quantitative-finance
  - backtest-overfitting
  - pbo
  - cscv
  - cross-validation
  - risk-management
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "PBO Paper"
  - "Bailey et al. 2015"
  - "CSCV Cross-Validation"
asset_class: Multi-Asset
timeframe: Daily
references:
  - "[[wiki/concepts/probability-of-backtest-overfitting|回測過度擬合機率 (PBO)]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
  - "[[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]"
  - "[[wiki/papers/tang-2025-alpha-agent|AlphaAgent]]"
---

# The Probability of Backtest Overfitting (PBO)

## 論文背景與核心概述

本篇經典論文由 **David H. Bailey**、**Jonathan M. Borwein**、**Marcos López de Prado** 與 **Qiji Jim Zhu** 於 2015 年發表於學術界。論文直指量化金融與投資管理中最嚴重的痛點：**回測過度擬合（Backtest Overfitting）與假陽性（False Positives）**。

傳統金融業界常用的「保留樣本法（Hold-out Method）」在量化多策略篩選中存在重大缺陷——只要研究人員嘗試了多種參數組合，過度擬合就必然存在。為此，作者提出了一套**無模型（Model-free）、非參數（Non-parametric）的測度理論架構**，並透過**組合對稱交叉驗證（Combinatorially Symmetric Cross-Validation, CSCV）**精確計算策略篩選過程中的「回測過度擬合機率（PBO）」。

---

## 核心數學理論與演算法

### 1. 測度論定義：回測過度擬合 (Definition of Overfitting)

設有 $N$ 個候選策略，在機率空間 $(\mathcal{T}, \mathcal{F}, \text{Prob})$ 上，$\mathbf{R} = (R_1, \dots, R_N)$ 代表樣本內（In-Sample, IS）績效向量，$\bar{\mathbf{R}} = (\bar{R}_1, \dots, \bar{R}_N)$ 代表樣本外（Out-of-Sample, OOS）績效向量。
令 $\mathbf{r}, \bar{\mathbf{r}}$ 為其績效在排列空間 $\Omega$ 中的排序（Rankings）。定義子集 $\Omega_n^* = \{f \in \Omega \mid f_n = N\}$，即第 $n$ 個策略在 IS 中表現最優。

* **回測過度擬合之定義（Backtest Overfitting）**：
  若在 IS 中最佳的策略，其 OOS 的期望排名低於所有策略的**中位數排名（Median Rank, $N/2$）**，則該策略篩選過程發生過度擬合：
  $$\sum_{n=1}^N \mathbb{E}[\bar{r}_n \mid \mathbf{r} \in \Omega_n^*] \text{Prob}[\mathbf{r} \in \Omega_n^*] \le \frac{N}{2}$$

* **回測過度擬合機率（PBO, Probability of Backtest Overfitting）**：
  $$\text{PBO} = \sum_{n=1}^N \text{Prob}\left[\bar{r}_n < \frac{N}{2} \;\middle|\; \mathbf{r} \in \Omega_n^*\right] \text{Prob}[\mathbf{r} \in \Omega_n^*]$$

### 2. CSCV 演算法流程 (Combinatorially Symmetric Cross-Validation)

1. **構建績效矩陣 $\mathbf{M}$**：維度為 $T \times N$，代表 $N$ 個策略在 $T$ 個時間切片上的收益序列。
2. **切分矩陣**：沿時間軸切分為偶數 $S$ 個子矩陣 $\mathbf{M}_s$（每個大小為 $\frac{T}{S} \times N$）。
3. **組合生成**：從 $S$ 個區塊中取 $\frac{S}{2}$ 個作為 IS 訓練集 $J$，剩餘 $\frac{S}{2}$ 個自然成為 OOS 測試集 $\bar{J}$。總組合數為：
   $$C_S = \binom{S}{S/2} = \prod_{i=0}^{S/2 - 1} \frac{S - i}{S/2 - i}$$
   *(例如 $S=16$ 時，可產生 $12,780$ 組對稱回測配對)*。
4. **計算 Logit 分布**：
   * 找出 IS 最佳策略索引 $n^*$。
   * 計算該策略在 OOS 的相對排名 $\bar{\omega}_c = \frac{\bar{r}_{n^*}^c}{N+1} \in (0, 1)$。
   * 映射至 Logit 空間：
     $$\lambda_c = \ln\left(\frac{\bar{\omega}_c}{1 - \bar{\omega}_c}\right)$$
5. **數值積分評估 PBO**：
   $$\phi = \int_{-\infty}^0 f(\lambda) \, d\lambda$$
   若 $\phi \approx 0$，代表 IS 最佳策略在 OOS 仍高度領先；若 $\phi > 0.5$，代表在過半數的樣本外測試中，IS 最優策略甚至落後於隨機中位數表現。

### 3. 四大互補分析維度

1. **PBO**：量化過度擬合機率（建議拒絕臨界點 $\phi > 0.05$）。
2. **績效退化（Performance Degradation）**：
   對回歸式 $\bar{R}_{n^*}^c = \alpha + \beta R_{n^*}^c + \varepsilon^c$ 進行配適。在過度擬合嚴重的策略中，$\beta$ 通常顯著為負值（即 IS 表現越突出，OOS 潰敗越慘烈）。
3. **虧損機率（Probability of Loss）**：$\text{Prob}[\bar{R}_{n^*}^c < 0]$。
4. **隨機優勢（Stochastic Dominance）**：檢驗最優策略 OOS 分布是否一階或二階隨機優勢於全體策略平均分布。

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* **CSCV 假定時間塊內部無洩漏**：CSCV 重新組合了 $S$ 個時間子塊，若某些技術指標的滾動窗口跨越了子塊邊界，或在提取特徵時使用了全樣本的均值與標準差（Z-Score Normalization），將導致未來信息滲漏至 IS 中，使 PBO 估算失準。

### 2. 存活者偏差（Survivorship Bias）
* 若回測矩陣 $\mathbf{M}$ 中的資產池未計入已退市標的，則即便 PBO 檢驗顯示低過度擬合，整體回測績效仍可能來自存活者偏差帶來的假性 Alpha。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* **把度量指標當作最佳化目標（Goodhart's Law / Strathern's Quote）**：
  論文作者強烈警告：**嚴禁將 PBO 作為目標函數來搜尋最優策略**！「When a measure becomes a target, it ceases to be a good measure.」任何試圖直接最小化 PBO 的元搜尋演算法（Meta-search）都會引發二階過度擬合。
* **文件抽屜問題（File Drawer Problem）**：研究員若隱瞞失敗的嘗試次數（只匯報最後幾次調參），將嚴重低估真實的 PBO。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* CSCV 架構是無模型（Model-free）的，它直接建立在回測產出的 P&L 矩陣 $\mathbf{M}$ 上。若底層 P&L 未扣除合理的換手滑價、借券費或買賣手續費，PBO 評估出的即便是不過度擬合的噪音，在實盤中亦會因交易摩擦而死亡。

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 候選策略集 $N$ 若涵蓋多種不同交易頻率或不同部位規模，矩陣 $\mathbf{M}$ 必須標準化為相同資本規模下的收益率。若高容量策略與低容量高滑價策略混雜，排名將產生結構性失真。

### 6. 環境體系轉換（Regime Shift）
* CSCV 要求整個回測區間 $T$ 必須涵蓋未來的潛在市場結構（如包含多頭、空頭與震盪）。若歷史樣本 $T$ 僅存在單一單邊行情（如 10 年大牛市），CSCV 即使交叉切分也無法識別出面臨總經體系突變時的脆弱性。

---

## 關聯概念與筆記
* 概念連結：[[wiki/concepts/probability-of-backtest-overfitting]]、[[wiki/concepts/backtest-metrics]]、[[wiki/concepts/alpha-decay]]
* 論文延伸：[[wiki/papers/kakushadze-2016-101-formulaic-alphas]]、[[wiki/papers/tang-2025-alpha-agent]]
