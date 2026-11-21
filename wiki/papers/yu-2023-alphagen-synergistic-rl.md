---
title: "Generating Synergistic Formulaic Alpha Collections via Reinforcement Learning (Yu et al., KDD '23)"
type: paper
tags:
  - quantitative-finance
  - alpha-mining
  - reinforcement-learning
  - ppo
  - synergistic-alphas
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "AlphaGen"
  - "Yu et al. 2023"
  - "KDD 2023 Synergistic Alphas"
asset_class: Equity
timeframe: Daily
references:
  - "[[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]"
  - "[[wiki/concepts/formulaic-alpha|公式型 Alpha 因子架構]]"
  - "[[wiki/concepts/alpha-decay|Alpha 衰減機制]]"
---

# Generating Synergistic Formulaic Alpha Collections via Reinforcement Learning (AlphaGen)

## 論文背景與核心概述

本論文發表於 **ACM SIGKDD 2023**，由中國科學院計算技術研究所（ICT, CAS）與華為雲團隊共同發表。

論文直指傳統遺傳演算法（GP）或單一因子挖掘框架的瓶頸：**傳統方法逐一挖掘個別高 IC 因子，或僅以兩兩相關性（Mutual-IC < 0.7）作為多樣性過濾條件，嚴重忽視了多因子組合時的「協同效應（Synergistic Effect）」**。論文提出 **AlphaGen**，將因子挖掘形式化為非平穩馬可夫決策過程（Non-stationary MDP），並以「下游組合模型預測效能的實質增益」作為強化學習（PPO）的獎勵信號，直接挖掘協同因子集合。

---

## 核心數學模型

### 1. 組合模型均方誤差（MSE Loss）之協同解析

設有 $k$ 個因子 $\mathcal{F} = \{f_1, \dots, f_k\}$ 與權重向量 $\mathbf{w} \in \mathbb{R}^k$。在因子與報酬已進行截面零均值與單位長度標準化下，組合收益預測模型之 MSE 損失可解析為：
$$\mathcal{L}(\mathbf{w}) = \frac{1}{n} \left[ 1 - 2 \sum_{i=1}^k w_i \bar{\sigma}_y(f_i) + \sum_{i=1}^k \sum_{j=1}^k w_i w_j \bar{\sigma}(f_i(\mathbf{X}), f_j(\mathbf{X})) \right]$$
其中 $\bar{\sigma}_y(f_i)$ 為單因子對目標報酬之 IC，$\bar{\sigma}(f_i, f_j)$ 為兩因子間的截面皮爾森相關係數。此公式允許在梯度下降時直接基於因子協方差矩陣更新權重，免除重複計算大矩陣之開銷。

### 2. 強化學習探索架構

* **狀態空間（State Space）**：由逆波蘭表示法（RPN）構成之符號 token 序列。
* **動作空間（Action Space）**：結合無效動作遮罩（Invalid Action Masking）嚴格保證語法結構合法性。
* **獎勵函數（Reward Function）**：
  $$r_t = \bar{\sigma}_y\left( \sum_{i=1}^k w_i f_i \right) = \text{IC}_{\text{new}}$$
  以加入新因子後動態組合之整體 IC 作為獎勵回饋，強迫 PPO Policy 尋找能為現有池提供正交互補資訊的因子。

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* 滾動時序算子（`TS-Mean`, `TS-Max`）必須嚴格向歷史回看，且目標標籤為次日或未來 20 日收益（`Ref(close, -20)/close - 1`）。回測時因子數值必須平移至 $t+1$ 日開盤方能作為調倉依據。

### 2. 存活者偏差（Survivorship Bias）
* 論文實驗採用 CSI 300 與 CSI 500。若採用固定今日成分股回測 2009～2021 年長週期，將忽略期間被剔除或下市的股票，導致 IC 虛高。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* 雖然以組合 IC 作為獎勵，但若訓練集長達 10 年且缺乏樣本外滾動 Walk-forward 檢定，PPO 策略網路仍可能在長期訓練後收斂至特定歷史風格的局部最優。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* 論文評估主要聚焦於 IC 與 RankIC。在實盤模擬時，若 $k=50$ 檔標的調倉未充分考慮衝擊成本，高相關因子組合的邊際增益可能被換手手續費抵銷。

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 針對 CSI 500 中小盤股，組合多頭權重若過度集中，將受限於每日成交金額（ADV）上限。

### 6. 環境體系轉換（Regime Shift）
* 2020 年前單邊行情訓練出之因子權重矩陣，在 2021 年後風格劇烈切換（如價值與成長輪動）時，歷史協方差矩陣將發生結構性失真。

---

## 關聯概念與筆記
* 概念連結：[[wiki/concepts/formulaic-alpha]]、[[wiki/concepts/alpha-decay]]
* 論文延伸：[[wiki/papers/kakushadze-2016-101-formulaic-alphas]]、[[wiki/papers/tang-2025-alpha-agent]]
