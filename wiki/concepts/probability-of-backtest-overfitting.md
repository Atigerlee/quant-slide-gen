---
title: "回測過度擬合機率 (Probability of Backtest Overfitting, PBO)"
type: concept
tags:
  - quantitative-finance
  - pbo
  - cscv
  - cross-validation
  - risk-management
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "PBO"
  - "CSCV"
  - "Backtest Overfitting"
asset_class: Multi-Asset
timeframe: Daily
references:
  - "[[wiki/papers/bailey-prado-2015-pbo|PBO Paper (Bailey et al. 2015)]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
  - "[[wiki/concepts/alpha-decay|Alpha 衰減機制]]"
---

# 回測過度擬合機率 (Probability of Backtest Overfitting, PBO)

## 核心定義與理論背景

**回測過度擬合（Backtest Overfitting）**是指當研究人員在同一份歷史數據集上嘗試了大量的策略變形、參數網格搜尋或特徵組合時，最佳表現的策略往往僅是配適了歷史噪聲（Noise），而非捕捉到真正的經濟規律（Signal）。

傳統的單一 Hold-out 測試無法計入「嘗試次數（Multiplicity of Trials）」。Bailey、López de Prado 等人（2015）提出了**回測過度擬合機率（PBO）**，定義為：**樣本內（IS）表現最優的策略，在樣本外（OOS）表現低於全體策略中位數排名的條件機率**。

$$\text{PBO} = \sum_{n=1}^N \text{Prob}\left[\bar{r}_n < \frac{N}{2} \;\middle|\; \mathbf{r} \in \Omega_n^*\right] \text{Prob}[\mathbf{r} \in \Omega_n^*]$$

---

## 組合對稱交叉驗證 (CSCV) 演算法

CSCV 透過無模型（Model-free）且非參數的方式估計 PBO：

### 1. 數據分塊與組合
將歷史回測長度為 $T$、策略數為 $N$ 的收益矩陣 $\mathbf{M} \in \mathbb{R}^{T \times N}$ 等分為 $S$ 個時間子區塊（$S$ 為偶數，實務常取 $S=16$）。
從中選取 $S/2$ 個區塊作為訓練集 $J$，其餘 $S/2$ 個為測試集 $\bar{J}$。組合總數為：
$$C_S = \binom{S}{S/2}$$

### 2. 相對排名與 Logit 轉換
對於每組劃分 $c \in C_S$：
* 找出訓練集中夏普值最高者 $n^*$。
* 計算該策略在測試集中的相對排名 $\bar{\omega}_c = \frac{\bar{r}_{n^*}^c}{N+1} \in (0, 1)$。
* 轉換為對數勝率（Logit）：
  $$\lambda_c = \ln\left( \frac{\bar{\omega}_c}{1 - \bar{\omega}_c} \right)$$

### 3. PBO 數值積分
PBO $\phi$ 等同於負 Logit 的累積機率密度：
$$\phi = \int_{-\infty}^0 f(\lambda) \, d\lambda$$
* $\phi < 0.05$：過度擬合風險極低，策略具備真正的樣本外預測能力。
* $\phi > 0.50$：策略篩選過程嚴重過度擬合，樣本內最優者在樣本外甚至劣於隨機猜測。

---

## 互補診斷工具

1. **績效退化圖（Performance Degradation Plot）**:
   以 IS 夏普值為橫軸、OOS 夏普值為縱軸繪製散佈圖。若迴歸斜率 $\beta < 0$，表示過度擬合正在摧毀未來收益。
2. **隨機優勢檢驗（Stochastic Dominance）**:
   若最優策略的累積分布函數（CDF）未能在統計上一階或二階優於平均隨機策略，則優化無效。

---

## 關聯文獻與概念
* 論文精讀：[[wiki/papers/bailey-prado-2015-pbo]]
* 因子衰減關係：[[wiki/concepts/alpha-decay]]
* 評估指標：[[wiki/concepts/backtest-metrics]]
