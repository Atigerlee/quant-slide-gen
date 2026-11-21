# 量化交易知識庫總目錄索引 (Master Wiki Index)

歡迎使用量化交易核心知識庫（Quantitative Trading LLM Wiki）。本知識庫嚴格遵循標準 LaTeX 數學規範、雙向連結網絡（Wikilinks）與量化風險揭露準則。

---

## 1. 層級樹狀清單 (Directory Tree Index)

```
wiki/
├── concepts/                  # 核心量化概念與計量理論
│   ├── [[wiki/concepts/formulaic-alpha|formulaic-alpha.md]]           # 公式型 Alpha 因子架構與算子代數
│   ├── [[wiki/concepts/probability-of-backtest-overfitting|probability-of-backtest-overfitting.md]] # 回測過度擬合機率 (PBO) 與 CSCV
│   ├── [[wiki/concepts/alpha-decay|alpha-decay.md]]                 # Alpha 衰減機制、擁擠效應與正則化探索
│   └── [[wiki/concepts/backtest-metrics|backtest-metrics.md]]            # 回測與因子評估指標體系 (IC, RankIC, Sharpe, IR, MDD)
├── papers/                    # 經典與最新量化論文精讀與架構拆解
│   ├── [[wiki/papers/kakushadze-2016-101-formulaic-alphas|kakushadze-2016-101-formulaic-alphas.md]] # 101 Formulaic Alphas (Kakushadze 2016)
│   ├── [[wiki/papers/bailey-prado-2015-pbo|bailey-prado-2015-pbo.md]]             # The Probability of Backtest Overfitting (Bailey et al. 2015)
│   ├── [[wiki/papers/bailey-prado-2014-deflated-sharpe|bailey-prado-2014-deflated-sharpe.md]] # The Deflated Sharpe Ratio (Bailey & López de Prado 2014)
│   ├── [[wiki/papers/yu-2023-alphagen-synergistic-rl|yu-2023-alphagen-synergistic-rl.md]]   # AlphaGen: Synergistic Formulaic Alpha via RL (KDD 2023)
│   ├── [[wiki/papers/tang-2025-alpha-agent|tang-2025-alpha-agent.md]]             # AlphaAgent: LLM-Driven Mining with Regularized Exploration (KDD 2025)
│   ├── [[wiki/papers/li-2025-rd-agent-quant|li-2025-rd-agent-quant.md]]            # R&D-Agent(Q): Data-Centric Multi-Agent Co-Optimization (NeurIPS 2025)
│   └── [[wiki/papers/tang-2026-alphaagentevo-arl|tang-2026-alphaagentevo-arl.md]]       # AlphaAgentEvo: Evolution via Agentic RL & GRPO (ICLR 2026)
└── strategies/                # 策略邏輯、信號規則與回測規範
    └── [[wiki/strategies/alpha-portfolio-selection|alpha-portfolio-selection.md]]     # 截面多因子 Alpha 投資組合建構與動態篩選策略
```

---

## 2. 策略與論文資產矩陣 (Strategy & Paper Matrix)

| 筆記名稱 | 類別 | 資產類別 | 時間週期 | 核心模型 / 方法論 | 驗證狀態 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [[wiki/papers/kakushadze-2016-101-formulaic-alphas\|101 Formulaic Alphas]] | Paper | US Equity | Daily | 冪律縮放 $R \sim \sigma^{0.76}$、算子代數、Mega-Alpha | Verified |
| [[wiki/papers/bailey-prado-2015-pbo\|Probability of Backtest Overfitting]] | Paper | Multi-Asset | Multi-Scale | CSCV 組合對稱交叉驗證、Logit 分布、PBO 積分 | Verified |
| [[wiki/papers/bailey-prado-2014-deflated-sharpe\|The Deflated Sharpe Ratio]] | Paper | Multi-Asset | Multi-Scale | DSR 公式、多重測試預期最大夏普、1/e 秘書停止法則 | Verified |
| [[wiki/papers/yu-2023-alphagen-synergistic-rl\|AlphaGen Synergistic RL]] | Paper | China Equity | Daily | 非平穩 MDP、PPO 策略梯度、組合 MSE 損失解析 | Verified |
| [[wiki/papers/tang-2025-alpha-agent\|AlphaAgent (KDD '25)]] | Paper | CSI 500, S&P 500 | Daily | AST 同構正則化、假說對齊、Multi-Agent 閉環 | Verified |
| [[wiki/papers/li-2025-rd-agent-quant\|R&D-Agent(Q) (NeurIPS '25)]] | Paper | CSI 300, NASDAQ 100 | Daily | Co-STEER 演化調度、上下文 Thompson 老虎機協同 | Verified |
| [[wiki/papers/tang-2026-alphaagentevo-arl\|AlphaAgentEvo (ICLR '26)]] | Paper | HS 300, CSI 500 | Daily | 智能體強化學習 (ARL)、GRPO 多輪工具反思、階層獎勵 | Verified |
| [[wiki/strategies/alpha-portfolio-selection\|截面多因子組合動態篩選]] | Strategy | Equity | Daily | Top-$k$ Dropout、LightGBM 集成、非線性衝擊成本 | Verified |

---

## 3. 標籤主題雲 (Topic Tags Index)

* **#backtest-overfitting & #statistics**
  * [[wiki/concepts/probability-of-backtest-overfitting|回測過度擬合機率 (PBO)]]
  * [[wiki/papers/bailey-prado-2015-pbo|Bailey et al. (2015) 論文]]
  * [[wiki/papers/bailey-prado-2014-deflated-sharpe|通縮夏普比率 (DSR)]]
* **#formulaic-alpha & #factor-mining**
  * [[wiki/concepts/formulaic-alpha|公式型 Alpha 因子架構]]
  * [[wiki/papers/kakushadze-2016-101-formulaic-alphas|101 Formulaic Alphas]]
  * [[wiki/papers/yu-2023-alphagen-synergistic-rl|AlphaGen 強化學習協同因子]]
* **#multi-agent & #agentic-rl**
  * [[wiki/papers/tang-2025-alpha-agent|AlphaAgent 多 Agent 因子挖掘]]
  * [[wiki/papers/li-2025-rd-agent-quant|R&D-Agent(Q) 全棧研發多 Agent]]
  * [[wiki/papers/tang-2026-alphaagentevo-arl|AlphaAgentEvo 智能體自演化強化學習]]
* **#alpha-decay & #risk-management**
  * [[wiki/concepts/alpha-decay|Alpha 衰減機制與防禦]]
  * [[wiki/concepts/backtest-metrics|量化回測指標體系]]
  * [[wiki/strategies/alpha-portfolio-selection|多因子投資組合建構策略]]

---

## 4. 最近更新日誌 (Recent Changelog)

1. **2026-11-21**：新增論文筆記 [[wiki/papers/tang-2026-alphaagentevo-arl|AlphaAgentEvo (ICLR 2026)]]。
2. **2026-11-21**：新增論文筆記 [[wiki/papers/li-2025-rd-agent-quant|R&D-Agent(Q) (NeurIPS 2025)]]。
3. **2026-11-21**：新增論文筆記 [[wiki/papers/yu-2023-alphagen-synergistic-rl|AlphaGen (KDD 2023)]]。
4. **2026-11-21**：新增論文筆記 [[wiki/papers/bailey-prado-2014-deflated-sharpe|The Deflated Sharpe Ratio (2014)]]。
5. **2026-11-21**：更新總目錄索引矩陣與標籤索引。
