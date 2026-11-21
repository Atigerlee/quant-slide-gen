# 簡報大綱（新版）：現代量化研究演化前沿：從回測偏差防禦到自演化智能體 Alpha 挖掘
> 依據 `raw/` 最新文獻與 `wiki/` 知識庫全面更新提煉（共 10 頁）

---

## Slide 1: 封面 (Title Cover)
- **類別**: Cover
- **主標題**: 現代量化研究演化前沿：從回測偏差防禦到自演化智能體 Alpha 挖掘
- **副標題**: 從 PBO/DSR 統計檢驗、協同算子代數到 Agentic RL 自主閉環
- **核心要點**:
  - 統計誠信與防禦性架構：PBO 測度論、通縮夏普比率（DSR）與 1/e 最優停止
  - 智能體因子工程前沿：從 AlphaGen、AlphaAgent 到 R&D-Agent(Q) 與 AlphaAgentEvo
- **引用出處**: `Source: [[wiki/index.md]]`

---

## Slide 2: 現代量化之痛：Alpha 衰減、假陽性與贏家的詛咒 (The Core Dilemma)
- **類別**: Problem Formulation
- **主標題**: 策略為何在實盤中迅速崩潰？
- **核心要點**:
  - 多重測試（Multiplicity of Trials）爆炸：算力無節制搜尋歷史噪聲，產生大量偽陽性高夏普。
  - 因子擁擠（Factor Crowding）：同質化策略推升進場成本，平倉引發流動性踩踏。
  - 贏家的詛咒（Winner's Curse）：在海量候選者中勝出者，實質上承擔了最嚴重的均值回歸。
- **核心公式**:
  $$\frac{\partial \mathbb{E}[\text{RankIC}_t]}{\partial t} < 0$$
- **引用出處**: `Source: [[wiki/concepts/alpha-decay]], [[wiki/papers/bailey-prado-2014-deflated-sharpe]]`

---

## Slide 3: 統計防禦線一：回測過度擬合機率 (PBO) 與 CSCV (CSCV & PBO)
- **類別**: Mathematical Theory
- **主標題**: 測度論視角下的回測過度擬合檢定
- **核心要點**:
  - 破解 Hold-out 盲點：多次嘗試下樣本外驗證必然失效。
  - 過度擬合定義：樣本內（IS）最佳策略，樣本外（OOS）期望排名落入中位數以下。
  - CSCV 對稱切分：序列等分 $S$ 塊對稱生成 $\binom{S}{S/2}$ 配對，投影至 Logit 空間積分。
- **核心公式**:
  $$\text{PBO} = \int_{-\infty}^0 f(\lambda) \, d\lambda, \quad \lambda_c = \ln\left(\frac{\bar{\omega}_c}{1 - \bar{\omega}_c}\right)$$
- **引用出處**: `Source: [[wiki/papers/bailey-prado-2015-pbo]], [[wiki/concepts/probability-of-backtest-overfitting]]`

---

## Slide 4: 統計防禦線二：通縮夏普比率 (DSR) 與 1/e 最優停止 (DSR & Optimal Stopping)
- **類別**: Statistical Rigor
- **主標題**: 矯正選擇偏差、非高斯偏態與試驗次數
- **核心要點**:
  - 預期最大夏普極值解：試驗次數 $N$ 增長時，最大夏普隨 $\sqrt{\mathbb{V}[\text{SR}]} Z^{-1}[1 - 1/N]$ 膨脹。
  - DSR 去膨脹公式：納入樣本長度 $T$、偏態 $\hat{\gamma}_3$ 與峰態 $\hat{\gamma}_4$ 修正統計顯著性。
  - 1/e 最優停止法則（秘書問題）：隨機評估前 $36.8\%$ 策略確立基準，選取首個超越者即停止，防範過擬合。
- **核心公式**:
  $$\text{DSR} = Z\left[ \frac{(\widehat{\text{SR}} - \widehat{\text{SR}}_0)\sqrt{T - 1}}{\sqrt{1 - \hat{\gamma}_3 \widehat{\text{SR}} + \frac{\hat{\gamma}_4 - 1}{4}\widehat{\text{SR}}^2}} \right]$$
- **引用出處**: `Source: [[wiki/papers/bailey-prado-2014-deflated-sharpe]]`

---

## Slide 5: 量化回測的六大致命陷阱與審計矩陣 (Six Deadly Pitfalls Audit)
- **類別**: Risk Audit
- **主標題**: 跨越學術模擬到實盤交易的六大深淵
- **核心要點**:
  - 前視偏差：Delay-0 盤中無延遲成交妄想；未平移除權息。
  - 存活者偏差：排除退市與停牌標的，虛增年化收益 2%～5%。
  - 資料探勘與抽屜效應：過度調參；隱瞞失敗嘗試次數。
  - 摩擦成本低估：忽略非線性市場衝擊 $\text{Impact} \sim \eta \sigma \sqrt{V/\text{ADV}}$。
  - 容量瓶頸與體系轉換：持倉超過 ADV 2% 引發自踩；極端行情下分散效應瓦解。
- **視覺規劃**: 六宮格警示卡片矩陣（Warning Cards）
- **引用出處**: `Source: [[wiki/strategies/alpha-portfolio-selection]], [[wiki/papers/kakushadze-2016-101-formulaic-alphas]]`

---

## Slide 6: 因子協同演化：從 101 公式算子到 AlphaGen 強化學習 (Synergistic Alpha Mining)
- **類別**: Factor Engineering
- **主標題**: 突破單因子思維：以組合協同效應為導向
- **核心要點**:
  - Kakushadze 101 算子：截面與時序算子庫，實證確立 $R_i \sim \sigma_i^{0.761}$ 且換手率無解釋力。
  - 傳統兩兩 IC 過濾盲點：高相關因子線性組合可能產生正交於原方向的全新超額收益。
  - AlphaGen（KDD '23）：以組合 MSE 損失解析為回饋，PPO 直接搜尋對現有池最具協同增益的因子。
- **核心公式**:
  $$\mathcal{L}(\mathbf{w}) = \frac{1}{n} \left[ 1 - 2 \sum_{i=1}^k w_i \bar{\sigma}_y(f_i) + \sum_{i=1}^k \sum_{j=1}^k w_i w_j \bar{\sigma}(f_i, f_j) \right]$$
- **引用出處**: `Source: [[wiki/papers/kakushadze-2016-101-formulaic-alphas]], [[wiki/papers/yu-2023-alphagen-synergistic-rl]]`

---

## Slide 7: 符號正則化與認知探索：AlphaAgent 假說對齊與 AST 同構 (AlphaAgent AST)
- **類別**: Multi-Agent System
- **主標題**: 兼顧結構原創性與經濟理性
- **核心要點**:
  - AST 最大子樹同構：計算新因子與現有庫（Alpha101）的子樹相似度 $\mathcal{S}(f)$，強迫結構創新防擁擠。
  - 假說與因子語義對齊：LLM 雙重一致性評分 $\mathcal{C}(h,d,f)$，杜絕無因果邏輯之偽因子。
  - 三 Agent 閉環架構：Idea Agent $\to$ Factor Agent $\to$ Eval Agent（Qlib 滾動回測）。
- **核心公式**:
  $$\mathcal{R}_g(f, h) = \alpha_1 \text{SL}(f) + \alpha_2 \text{PC}(f) + \alpha_3 \left[ \beta_1 \mathcal{S}(f) + \beta_2 \mathcal{C}(h,d,f) + \beta_3 \log(1+|\mathcal{F}_f|) \right]$$
- **引用出處**: `Source: [[wiki/papers/tang-2025-alpha-agent]]`

---

## Slide 8: 全棧協同研發：R&D-Agent(Q) 與上下文 Thompson 抽樣 (R&D-Agent(Q))
- **類別**: Autonomous Research
- **主標題**: 因子與模型交替演化的貝氏多臂老虎機
- **核心要點**:
  - 五大功能模組：規格（Schema 隔離）、合成（假說森林）、實現（Co-STEER 代碼自修復）、驗證、分析。
  - 上下文 Thompson 抽樣：依據 8 維量化績效狀態向量 $\mathbf{x}_t$，自適應調度「因子優化」或「模型調優」。
  - 極致成本效益：不到 10 美元 API 成本，年化超額收益達基準因子庫 2 倍，且因子數量減少 70%。
- **核心公式**:
  $$\hat{r}^{(a)} = (\tilde{\boldsymbol{\theta}}^{(a)})^\top \mathbf{x}_t, \quad \tilde{\boldsymbol{\theta}}^{(a)} \sim \mathcal{N}\left(\boldsymbol{\mu}^{(a)}, (\mathbf{P}^{(a)})^{-1}\right)$$
- **引用出處**: `Source: [[wiki/papers/li-2025-rd-agent-quant]]`

---

## Slide 9: 終極自演化範式：AlphaAgentEvo 智能體強化學習 (AlphaAgentEvo ARL)
- **類別**: Frontier Paradigm
- **主標題**: 從單步試錯邁向連續演化軌跡 (ICLR 2026)
- **核心要點**:
  - 突破「搜尋-回測-重啟」循環：以 GRPO 演化策略，在多輪工具反思軌跡中自我演化。
  - 多層次階層獎勵：整合工具調用成本 $R_{\text{tool}}$、種子錨定 $R_{\text{cons}}$、探索 $R_{\text{expl}}$ 與連續進步加成 $R_{\text{streak}}$。
  - 輕量強韌：僅 4B 參數開源模型在 HS300/CSI500 牛熊跨期 Pass@5 達 $97\%$～$99\%$，超越閉源大模型。
- **核心公式**:
  $$R(\tau) = \frac{\min(R_{\text{cons}}, C_{\text{cons}}) + \min(R_{\text{expl}}, C_{\text{expl}})}{\min(R_{\text{tool}}, C_{\text{tool}})} + \min(R_{\text{perf}}, C_{\text{perf}}) \cdot \min(R_{\text{streak}}, C_{\text{streak}})$$
- **引用出處**: `Source: [[wiki/papers/tang-2026-alphaagentevo-arl]]`

---

## Slide 10: 結語與防禦性量化研究最佳實踐 (Conclusion & Actionable Rules)
- **類別**: Best Practices
- **主標題**: 打造兼具統計誠信、因果邏輯與自演化能力的量化系統
- **核心要點**:
  - 嚴守數據紀律：落實 Delay-1 執行與 Point-in-time 動態股票池，根除前視與存活者偏差。
  - 統計誠信準則：強制計算 PBO 與 DSR，絕不將去膨脹指標當作目標函數（Strathern 定律）。
  - 擁抱自演化智能體：利用 Agentic RL 與 AST 正則化，在動態市場環境中實現可持續的 Alpha 探索。
- **引用出處**: `Source: [[wiki/strategies/alpha-portfolio-selection]], [[wiki/papers/bailey-prado-2015-pbo]], [[wiki/papers/tang-2026-alphaagentevo-arl]]`
