---
title: "截面多因子 Alpha 投資組合建構與動態篩選策略"
type: strategy
tags:
  - quantitative-finance
  - cross-sectional-momentum
  - top-k-dropout
  - mega-alpha
  - portfolio-optimization
status: verified
created: 2026-11-21
updated: 2026-11-21
aliases:
  - "Multi-Alpha Portfolio Strategy"
  - "Top-k Dropout Selection"
asset_class: Equity
timeframe: Daily
references:
  - "[[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]"
  - "[[wiki/papers/tang-2025-alpha-agent|AlphaAgent]]"
  - "[[wiki/papers/bailey-prado-2015-pbo|PBO Paper]]"
  - "[[wiki/concepts/backtest-metrics|量化回測評估指標]]"
---

# 截面多因子 Alpha 投資組合建構與動態篩選策略

## 策略哲學與經濟邏輯

單一 Alpha 因子具有邊際報酬微弱、容量受限且隨時間衰減的特質。本策略融合 [[wiki/papers/kakushadze-2016-101-formulaic-alphas|Kakushadze (2016)]] 的 **Mega-Alpha 內部撮合理論** 與 [[wiki/papers/tang-2025-alpha-agent|Tang et al. (2025)]] 的 **Top-$k$ Dropout 動態篩選模型**，建立具備抗衰減能力的多因子股票多空投資組合。

---

## 策略建構與訊號生成邏輯

### 1. 因子標準化與正交化
全市場標的集合 $\mathcal{S} = \{s_1, \dots, s_N\}$。對於每個 Alpha 因子 $f_m$（$m = 1, \dots, M$）：
1. **極值去極值化（Winsorization）**：截斷至 3 倍標準差 $[ \mu - 3\sigma, \mu + 3\sigma ]$。
2. **截面 Z-Score 標準化**：
   $$z_{i, m} = \frac{f_{i, m} - \mu_m}{\sigma_m}$$
3. **行業中性化與市值中性化**：
   透過截面 OLS 回歸去除行業 Dummy 變數與對數市值變數的暴露。

### 2. 機器學習信號集成與 Top-$k$ Dropout
將處理後的因子輸入梯度提升樹（LightGBM）預測次日收益率 $\hat{y}_i$。
* **選股清單（Top-$k$ Dropout）**：
  * 按 $\hat{y}_i$ 降序排序，選取排名前 $K=50$ 檔股票。
  * 剔除前 $K$ 檔中動量反轉風險最高的後 $5$ 檔（Dropout机制，防範極端反轉）。
* **頭寸權重（L1 槓桿標準化）**：
  $$w_i = \frac{\hat{y}_i}{\sum_{j \in \text{Selected}} |\hat{y}_j|}$$

---

## 潛在風險與陷阱 (Risks & Pitfalls)

### 1. 前視偏差（Look-ahead Bias）
* 特徵工程中必須嚴格落實 **Delay-1 規則**：$t$ 日收盤後提取的因子特徵，只能用於預測 $t+1$ 日收盤到 $t+2$ 日收盤的報酬，或在 $t+1$ 日以 VWAP/開盤價進場。禁止使用當日結算價作為可即時撮合的信號。

### 2. 存活者偏差（Survivorship Bias）
* 股票池（如 CSI 500 或 S&P 500）必須採用**歷史逐日動態成分股名單（Point-in-time Constituents）**。若使用當前成分股回測過去 10 年，將排除下市及長期停牌標的，導致超額收益虛高 2%～5% 年化。

### 3. 過度擬合與資料探勘（Overfitting & Data Snooping）
* 因子庫擴展時必須透過 [[wiki/papers/bailey-prado-2015-pbo|CSCV / PBO 檢定]]，確保候選策略集的過度擬合機率 $\text{PBO} < 0.05$。避免透過反覆調整 LightGBM 樹深（例如調整 depth、learning rate）來迎合特定歷史區間。

### 4. 交易成本與滑價假設（Transaction Costs & Slippage）
* 每日調倉 $K=50$ 檔股票將帶來可觀的換手成本。回測中必須嚴格設定單邊手續費與印花稅（買入 0.05%、賣出 0.15%），並加入基於委託量佔比的非線性衝擊成本模型（Square-root Law Impact Model）：
  $$\text{Impact} \sim \eta \sigma \sqrt{\frac{V_{\text{order}}}{\text{ADV}}}$$

### 5. 流動性與市場容量（Liquidity & Capacity Limit）
* 策略頭寸不得超過標的當日成交量（ADV）的 2%。若組合規模達 5 億以上，Top-50 選股中流動性較低標的將難以在不劇烈影響市價的情況下平倉。

### 6. 環境體系轉換（Regime Shift）
* 面臨流動性枯竭、宏觀黑天鵝或板塊風格極致分化（如大盤價值股狂飆而中小盤微盤股暴跌）時，因子相關性會瞬間由原本的 15% 激增至 80% 以上，引發組合風險分散效應失靈。

---

## 關聯概念與筆記
* 參考文獻：[[wiki/papers/tang-2025-alpha-agent]]、[[wiki/papers/kakushadze-2016-101-formulaic-alphas]]
* 風險檢驗：[[wiki/concepts/probability-of-backtest-overfitting]]
* 因子體系：[[wiki/concepts/formulaic-alpha]]
