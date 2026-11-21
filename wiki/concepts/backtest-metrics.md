---
title: "量化回測與因子評估指標體系 (Quantitative Evaluation Metrics)"
type: concept
tags:
  - quantitative-finance
  - performance-metrics
  - sharpe-ratio
  - information-ratio
  - ic
  - rank-ic
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "Backtest Metrics"
  - "Sharpe Ratio"
  - "RankIC"
  - "Information Ratio"
asset_class: Multi-Asset
timeframe: Daily
references:
  - "[[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]"
  - "[[wiki/papers/bailey-prado-2015-pbo|回測過度擬合機率 (PBO)]]"
  - "[[wiki/papers/tang-2025-alpha-agent|AlphaAgent]]"
---

# 量化回測與因子評估指標體系 (Quantitative Evaluation Metrics)

在量化交易研究中，評估策略與因子的表現必須使用嚴謹、無歧義的數學公式。

---

## 1. 因子預測能力指標 (Factor Predictive Quality)

### 資訊係數 (Information Coefficient, IC)
衡量因子預測值 $f_t$ 與標的未來實際收益率 $r_{t+1}$ 之間的截面皮爾森相關係數：
$$\text{IC}_t = \frac{\sum_{i=1}^N (f_{i, t} - \bar{f}_t)(r_{i, t+1} - \bar{r}_{t+1})}{\sqrt{\sum_{i=1}^N (f_{i, t} - \bar{f}_t)^2} \sqrt{\sum_{i=1}^N (r_{i, t+1} - \bar{r}_{t+1})^2}}$$

### 秩資訊係數 (Rank Information Coefficient, RankIC)
消除異常值與非線性扭曲的斯皮爾曼等級相關係數：
$$\text{RankIC}_t = \rho\left(\text{rank}(f_t), \text{rank}(r_{t+1})\right)$$

### 資訊係數比率 (IC Information Ratio, ICIR)
評估因子預測穩定性（值越大代表月度/日度穩定獲利能力越強）：
$$\text{ICIR} = \frac{\mathbb{E}[\text{IC}_t]}{\sigma(\text{IC}_t)}$$

---

## 2. 投資組合績效與風險指標 (Portfolio Performance & Risk)

### 年化夏普比率 (Annualized Sharpe Ratio)
$$\text{Sharpe} = \sqrt{252} \cdot \frac{\mathbb{E}[R_p - R_f]}{\sigma_p}$$
其中 $R_p$ 為日投資組合報酬率，$R_f$ 為無風險利率，$\sigma_p$ 為日報酬率波動度。

### 資訊比率 (Information Ratio, IR)
衡量主動投資組合相對於基準（Benchmark, $R_b$）的風險調整後超額報酬：
$$\text{IR} = \sqrt{252} \cdot \frac{\mathbb{E}[R_p - R_b]}{\sqrt{\text{Var}(R_p - R_b)}}$$

### 最大回撤 (Maximum Drawdown, MDD)
衡量資本在特定時間區間內從峰值下跌的最大幅度：
$$\text{MDD}_T = \max_{\tau \in [0, T]} \left( \frac{\max_{t \in [0, \tau]} V_t - V_\tau}{\max_{t \in [0, \tau]} V_t} \right)$$
其中 $V_t$ 為投資組合在時刻 $t$ 的資產淨值（NAV）。

### 凱利準則 (Kelly Criterion)
最佳槓桿比率與資金分配模型：
$$f^* = \frac{p \cdot b - q}{b} = \frac{p(b+1) - 1}{b}$$
其中 $p$ 為勝率，$q = 1-p$ 為敗率，$b$ 為盈虧比（獲利除以損失）。

---

## 關聯文獻與概念
* 論文精讀：[[wiki/papers/kakushadze-2016-101-formulaic-alphas]]、[[wiki/papers/tang-2025-alpha-agent]]、[[wiki/papers/bailey-prado-2015-pbo]]
* 因子衰減：[[wiki/concepts/alpha-decay]]
