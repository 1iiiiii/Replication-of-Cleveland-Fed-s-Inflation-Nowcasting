# Paper Reference (Ground Truth Document)

**Knotek, Edward S. II, and Saeed Zaman (2014), "Nowcasting U.S. Headline and Core Inflation," Federal Reserve Bank of Cleveland Working Paper 14-03, May 2014.**

Full text: `paper_full_text.txt` (extracted from the PDF). **Any claim about "what the paper does" must be checkable against this file. If it is not in this file or in this summary, say "the paper does not say this."**

---

## 1. What the paper does

Builds a parsimonious model to nowcast monthly and quarterly US inflation — headline CPI, core CPI, headline PCE, core PCE — using only **8 data series**, simple OLS regressions, and moving averages. No factor models, no Kalman filter. Evaluated with **real-time (vintage) data** 1999–2013 against statistical benchmarks, Blue Chip, SPF, and the Fed Greenbook (Sections IV–V).

## 2. The 8 data series (Section III)

| # | Series | Frequency | Paper's source | FRED code (kit addition, NOT from paper) |
|---|--------|-----------|----------------|------------------------------------------|
| 1 | CPI, all items, SA | monthly | BLS via ALFRED | CPIAUCSL |
| 2 | Core CPI (ex food & energy), SA | monthly | BLS via ALFRED | CPILFESL |
| 3 | Food CPI, SA | monthly | BLS via ALFRED | CPIUFDSL |
| 4 | Gasoline CPI, SA | monthly | BLS/Haver | CUSR0000SETB01 |
| 5 | PCE price index | monthly | BEA via ALFRED | PCEPI |
| 6 | Core PCE price index | monthly | BEA via ALFRED | PCEPILFE |
| 7 | Retail gasoline price, all grades, NSA | weekly (Mondays) | EIA | GASALLW |
| 8 | Brent crude spot | daily | Financial Times | DCOILBRENTEU |

Release timing exploited by the model: CPI for month *t* released ~mid-month *t+1*; PCE for month *t* released ~end of month *t+1* (about 2 weeks after CPI).

## 3. Key equations

Notation: monthly inflation is month-over-month percent change; quarterly inflation is annualized.

**Eq (1)–(2): quarterly aggregation.** Quarterly price level = average of the 3 monthly levels: `P_T = (1/3)(P_{T,1} + P_{T,2} + P_{T,3})`. Quarterly annualized inflation: `pi_T = 100 * [ (P_T / P_{T-1})^4 − 1 ]`. The model nowcasts *missing monthly price levels*, then aggregates. This matches BEA / Blue Chip convention.

**Eq (3): general form.** `A_s(t) Z_t = B_s(t) + C_s(t) X_t + Σ_j D_{j,s(t)} Z_{t−j} + e`, where Z = aggregates, X = disaggregates, and the coefficient matrices **switch deterministically with the information set s(t)** — "time-varying weights with deterministic model switching."

**Eq (4): core inflation default.** With no usable disaggregates, forecast monthly core CPI and core PCE as a **recursive 12-month moving average** of past monthly inflation (A = I, B = 0, D_j = (1/12)I, J = 12). Spirit of Atkeson–Ohanian (2001).

**Eq (5): core bridge.** When month-t core CPI is released but core PCE is not: OLS `pi_t^corePCE = b + a * pi_t^coreCPI`, estimated on a rolling window of **τ = 24 monthly observations**.

**Food:** 12-month moving average, same as eq (4). No disaggregates (raw food prices judged too weakly linked; parsimony).

**Gasoline block:**
- If weekly gasoline data exist in month t: average them → NSA monthly gasoline inflation → **seasonally adjust** by subtracting the average seasonal factor from the last 3 years, where the seasonal factor sf_t = mean over 3 yrs of (NSA gasoline inflation − SA CPI-gasoline inflation) for that calendar month (footnote 7). The CPI-gasoline series is used ONLY for seasonal adjustment.
- If no gasoline data yet for month t: assume **oil follows a random walk** at daily frequency (footnote 9, citing Alquist–Kilian 2010), then a two-stage regression estimated on **τ_L = 60 months**:
  - Eq (6) long-run: `P_gas = alpha + beta * P_oil + error`
  - Eq (7) ECM: `ΔP_gas_t = b * ΔP_oil_t + c * (P_gas_{t−1} − fitted_gas_{t−1}) + error`

**Eq (8)–(11): headline.** Z = [CPI, PCE]; X = [core CPI, core PCE, food, gasoline].
- Eq (9): CPI released, PCE not → headline bridge `pi^PCE = b + a * pi^CPI` (24-month window).
- Eq (10): gasoline nowcast available → OLS with restrictions: `pi^CPI = b1 + c11*coreCPI + c13*food + c14*gasoline` and `pi^PCE = b2 + c22*corePCE + c23*food + c24*gasoline` (24-month window). Note the zero restrictions: CPI eq uses core CPI, PCE eq uses core PCE; both use food and gasoline.
- Eq (11): no gasoline data → fall back to 12-month moving average.

## 4. Estimation & evaluation design

- Rolling windows: **τ = 24 months** for eqs (5), (9), (10); **τ_L = 60 months** for eqs (6)–(7). Short windows are essential (Section VI): τ = 120 raises headline RMSE 16–47%.
- Real-time evaluation starts Feb 1999 (CPI) / Jul 2000 (PCE). "Truth" = BEA's **third estimate** (same convention for CPI).
- Outliers excluded: Sep/Oct 2001 PCE (9/11 insurance effect); 2008Q4 for quarterly RMSE comparisons.
- Metric: RMSE; **Diebold–Mariano** tests with Harvey et al. (1997) small-sample adjustment, two-sided, on MSE.
- Monthly evaluation at 6 representative dates ("cases") within the month; quarterly at 14 cases within the quarter.

## 5. Headline results (for sanity-checking replication)

- Nowcast RMSEs **fall as the month/quarter progresses**. Headline RMSEs roughly **halve** between just-before-month-start and just-before-release (monthly exercise). By end of month 1 of a quarter, model RMSE ≈ half of best statistical benchmark.
- Just before the quarterly release: headline errors ≈ ¼ pp annualized; core ≈ 0.1–0.3 pp.
- Beats Blue Chip on headline CPI (increasingly as quarter progresses); beats SPF on headline CPI & PCE; core ≈ tie with SPF/Greenbook; overall ≈ comparable to Greenbook.
- Sensitivity (Table 7): dropping gasoline disaggregate → headline RMSE +40–104% (the single most important ingredient). Dropping oil → +9–11% early in quarter. Dropping food → minor. Long windows → clearly worse. Replacing MAs with AR(1) ≈ similar; higher-order AR worse.

## 6. What the paper does NOT do (guard against drift)

- No factor models, no large datasets, no Kalman filtering / state-space estimation, no Bayesian methods, no machine learning.
- No food futures/spot prices as disaggregates; no Billion Prices Project data.
- Does not nowcast GDP; does not claim gasoline data help *core* inflation.
- Core goods/services split is mentioned as infeasible in real time, not implemented.
