# Replication Roadmap — Knotek & Zaman (2014) Inflation Nowcasting

**Goal:** deeply understand the paper by rebuilding it, milestone by milestone, in `nowcast_skeleton.ipynb`. You write the code for every `TODO` stub; the tutor model reviews it; the self-tests confirm correctness; `solutions/` is the last resort.

**Scope decision (agreed 2026-07-11):** final-vintage FRED data first (pseudo out-of-sample). ALFRED real-time vintages are a stretch milestone (M7), not the starting point.

**Rules for yourself:**
1. Attempt every stub before asking for help. When stuck, ask the tutor for a *concept hint*, not code.
2. Open a solution section only after 30+ minutes stuck AND after the tutor's hints failed. Log it below when you do.
3. After each milestone, answer the checkpoint questions **in writing in the notebook** before moving on. If you can't, you're not done.
4. Run the milestone's self-test cell. Green = move on.

Solution-peek log: (record milestone + date + why)

---

## M0 — Data (provided, ~30 min)
The download cell is written for you (this part is tedious, not instructive). Run it; inspect every series.
- **Do:** plot all 8 series; check frequencies, start dates, missing values.
- **Checkpoint:** Why does the paper insist on *seasonally adjusted* monthly data everywhere except retail gasoline? What breaks if you mix SA and NSA? (paper_reference §2–3, paper footnote 11)
- **Self-test:** provided cell asserts series count, frequencies, and date coverage.

## M1 — Inflation arithmetic (eqs 1–2, ~1 hr)
Implement `monthly_inflation(level_series)` and `quarterly_annualized(level_series)` (quarterly avg of monthly levels, then annualized 4th-power growth).
- **Checkpoint:** Why does averaging three monthly *levels* differ from averaging three monthly inflation *rates*? Which months of quarter T−1 still matter for quarter T inflation, and why does that give the model a head start before T even begins?
- **Self-test:** your quarterly PCE inflation from monthly PCEPI must match FRED's quarterly PCECTPI-based rate within 0.2 pp for recent quarters (independent cross-check — BEA aggregates the same way).
- **Pitfall:** off-by-one on `pct_change`; forgetting the ^4 annualization; using sum instead of mean for quarterly levels.

## M2 — Core & food: the moving-average default (eq 4, ~1 hr)
Implement `ma12_forecast(infl_series, n_ahead)`: recursive 12-month moving average (each forecasted month feeds back into the next month's average).
- **Checkpoint:** This is the Atkeson–Ohanian idea. Why is such a naive rule so hard to beat for core inflation at short horizons? What property of core inflation does it exploit?
- **Self-test:** on a constant series the forecast must be exactly constant; on real core CPI, MA12 must beat a monthly random walk (pi_t = pi_{t-1}) on RMSE over the evaluation sample.
- **Pitfall:** "recursive" means forecasts enter the window for multi-step forecasts — a fixed trailing window is a different (wrong) model.

## M3 — The gasoline block (eqs 6–7 + footnote 7, ~2–3 hrs, hardest)
Three pieces:
1. `weekly_to_monthly_gas()`: average weekly EIA prices within month → NSA monthly gasoline inflation.
2. `seasonal_adjust_gas()`: sf for calendar month m = 3-year average of (NSA gas inflation − SA CPI-gasoline inflation); SA nowcast = NSA − sf.
3. `oil_gas_ecm()`: oil random-walk extension + two-stage regression (long-run level eq, then ECM in changes with lagged gap), 60-month window.
- **Checkpoint:** (a) Why seasonally adjust with CPI-gasoline seasonal factors instead of X-13 on the EIA series itself? (Hint: real-time feasibility & consistency with the target.) (b) In the ECM, what economic behavior does the `c * (gap)` term capture at the pump?
- **Self-test:** SA gasoline inflation nowcast must correlate > 0.9 with actual SA CPI-gasoline inflation over the eval sample; ECM beta (long-run oil coefficient) must be positive.
- **Pitfall:** using future data to build seasonal factors (only use the *past* 3 years); mixing $/gallon and index units — the regressions are in price *levels* and their changes, per the paper.

## M4 — Bridge equations (eqs 5 & 9, ~1 hr)
Implement `bridge(y_series, x_series, window=24)`: OLS of PCE-measure inflation on same-month CPI-measure inflation, used only in the state "CPI released, PCE not yet."
- **Checkpoint:** Why does a CPI→PCE bridge work at all — what do the two indexes share, and name two reasons they diverge (weights, scope, formula)? Why estimate on 24 months instead of the full sample?
- **Self-test:** bridge nowcast of core PCE must beat the MA12 forecast of core PCE on RMSE (paper Table 7 line 14 says bridging helps).
- **Pitfall:** regressing on *lagged* CPI instead of same-month CPI — the whole point is the two-week release gap for the *same* month.

## M5 — Headline regressions + model switching (eqs 8–11, ~2 hrs)
Implement `headline_regression()` per eq (10) with its zero restrictions (CPI eq: own core + food + gas; PCE eq: own core + food + gas; 24-month window), and `nowcast_month(t, info_date)` that picks eq (9), (10), or (11) based on what's observable at `info_date` — the deterministic model switching.
- **Checkpoint:** Write out, in a table, which equation governs each of these information states for target month t: (i) nothing released, no gas data; (ii) weekly gas data only; (iii) CPI_t released, PCE_t not. This table IS the paper's core innovation — if you can produce it from memory, you understand the model.
- **Self-test:** with all-zero gasoline and food inflation, the eq-(10) nowcast must approximately equal intercept + core term (sanity of restrictions); switching function must return the correct regime label on 5 hand-constructed dates.
- **Pitfall:** letting the regression see disaggregate values that would not have been observable at `info_date` — information discipline is the exercise.

## M6 — Calendar simulation & evaluation (~2–3 hrs)
Simulate 3 monthly info-dates (paper uses 6; we simplify): (A) last day before target month; (B) mid-month, ~2 gasoline weeks in; (C) day after CPI release. Loop over 2005–2019 (skip 2020–21 COVID months in the headline RMSE, analogous to the paper's outlier exclusions — note this deviation from the paper, which excludes 2001 and 2008Q4). Compute RMSE per case; benchmark against monthly random walk and MA12.
- **Checkpoint:** Reproduce the paper's qualitative signature: headline RMSE falls substantially from case A to case C while core RMSE barely moves. Explain *why* core barely moves in one sentence.
- **Self-test:** RMSE(C) < RMSE(B) < RMSE(A) for headline CPI; model beats random walk at case C.
- **Note:** your RMSE *levels* will differ from the paper's Tables (different sample, final-vintage data). Matching the *pattern* is the success criterion, matching levels is not — do not let a tutor model convince you to chase the paper's exact numbers.

## M7 — Stretch (pick any, unbounded)
- **Real-time honesty:** pull ALFRED vintages for CPI/PCE (`fredapi` supports `get_series_as_of_date`; Yi has an API key) and rerun M6. Compare pseudo vs true real-time RMSE. Known data gap: gasoline-CPI vintages don't exist before Apr 2011 (paper used Haver) — treat pre-2011 as final-vintage and note the caveat. EIA weekly gasoline and Brent are effectively unrevised, so vintages are irrelevant for them. Blue Chip is proprietary; Greenbook/Tealbook and SPF nowcasts come from the Philadelphia Fed website, not FRED.
- **DM tests:** implement Diebold–Mariano with the Harvey small-sample correction; test model vs random walk.
- **ML extension:** promoted to a full milestone — see **M8** below (decided 2026-07-13; interpretability-first). Further ideas that stayed in the stretch pile: U-MIDAS with ridge (weekly/daily data directly instead of the two-stage ECM); composites (equal-weight average of model + MA12 + UC-SV, then stacking with rolling-window weights).
- **Quarterly layer:** aggregate monthly nowcasts to quarterly (M1 machinery) and reproduce the 14-case quarterly RMSE decline.

## M8 — Interpretable ML horse-race (~3–4 hrs)

Scope agreed 2026-07-13: ML models may enter only if their fitted behavior can be *read* — coefficients, selection sets, shape functions. Pure black boxes appear only as accuracy ceilings to interpret against. All sklearn, no new dependencies. Avoid LSTMs/deep nets: ~250 monthly obs, and they fail the interpretability bar.

**Non-negotiable discipline (inherited from the paper):** same M6 calendar harness and information sets; any tuning uses data *before* the forecast date only (expanding/rolling — never the evaluation sample); the benchmark to beat is **your replicated eq-(10) model**, not the random walk. The self-test harness isolates the model-class question by feeding every model the *actual* month-t disaggregates as hats ("perfect disaggregates") — hat generation is identical across models anyway, so this compares regressions, not pipelines.

### M8a — Ridge: shrinkage vs short windows (~1 hr)
Implement `ridge_headline_forecast(...)`: eq (10) with an L2 penalty (standardize regressors inside the window; never penalize the intercept). Study the provided rolling coefficient-path plot: OLS coefficients on 24-month windows jump around (the parameter instability that motivated the paper's short windows) while ridge smooths them — shrinkage and short windows are rival cures for the same disease.
- **Checkpoint:** Why do short rolling windows and shrinkage address the same problem? Under what data conditions would ridge on a 60-month window beat OLS on a 24-month window?
- **Self-test:** `alpha=0` reproduces the OLS forecast; `alpha→∞` collapses to the window mean; expanding-window-tuned ridge RMSE ≤ 1.05 × eq-(10) OLS RMSE on the case-B harness.
- **Pitfall:** standardizing on the full sample (leakage) — fit the scaler inside each window; tuning alpha on the evaluation sample.

### M8b — LASSO: does selection rediscover the paper? (~1 hr)
Widen the regressor set with 4 more FRED series — `WPS0571` (PPI gasoline, SA), `IR` (import price index, NSA — deliberate: watch what LASSO does with a noisy NSA series), `AHETPI` (avg hourly earnings, SA), `DGASUSGULF` (Gulf Coast gasoline spot, daily) — and implement `lasso_selection(...)`: rolling `LassoCV` (TimeSeriesSplit) recording which regressors get nonzero coefficients in each window.
- **Checkpoint:** Which regressors survive, and how stable is the selection across windows? The gasoline "twins" (CPI gas, PPI gas, spot gas) are nearly collinear — what does LASSO do with collinear twins, and why does that mean you should report the selection *family*, not the individual pick?
- **Self-test:** at least one gasoline-family regressor is selected in ≥80% of windows; a selection-frequency table prints.
- **Pitfall:** full-sample standardization (leakage); random K-fold CV on time series — use `TimeSeriesSplit`.

### M8c — Nonlinearity you can read: DIY GAM (~1–1.5 hrs)
Implement `spline_additive_forecast(...)`: per-feature `SplineTransformer` + `Ridge` in one sklearn `Pipeline` — an *additive* model whose per-feature shape functions you plot and read directly. Target question: is headline's response to gasoline swings asymmetric (the "rockets and feathers" pass-through story)? Ceiling: `HistGradientBoostingRegressor`, read through `partial_dependence` — an accuracy ceiling to interpret, not a candidate model.
- **Checkpoint:** From the fitted gasoline shape function: is the pass-through asymmetric? Why does additivity keep the model readable while a random forest is not? Did nonlinearity actually pay in RMSE — and if not, why might 60-month windows be too short for it to?
- **Self-test:** the spline model beats the monthly random walk on the case-B harness; RMSE vs eq (10) and vs the boosting ceiling is reported (no assert on who wins — honest horse-race); the gasoline shape plot renders.
- **Pitfall:** splines on 24-month windows will overfit badly — M8c uses 60 months; disclose that deviation from eq (10)'s window whenever you compare RMSEs.

### M8 closing checkpoint
Which of the paper's design choices did you keep in the ML version, and why? (Expected: the calendar discipline, the information sets, past-only tuning — the model class is the least important part.)

---

## Suggested pace
M0–M1 in one sitting (concepts are light, sets up everything). M2 + M4 together (both are "simple model, deep why"). M3 alone (hardest block). M5 alone (the paper's actual contribution). M6 to close the loop. Total ≈ 10–14 focused hours. M8 only after M6 is green (it reuses the harness and the eq-(10) benchmark); ideally do M7's Diebold–Mariano tests first so the M8 horse-race comes with significance tests, +3–4 hrs.
