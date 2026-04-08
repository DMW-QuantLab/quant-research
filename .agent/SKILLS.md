# SKILLS: Alpha Research Scientist
## Agent: quant-research-agent

---

## SKILL 1 — Hypothesis Formation

**Trigger:** Any new edge idea, market observation, or anomaly.

**Protocol:**
1. State the mispricing in one sentence: "Market X is systematically overpriced because Y."
2. Identify the generative mechanism — why does the mispricing exist? (information lag, retail bias, thin liquidity, resolution ambiguity, etc.)
3. Identify why it hasn't been arbitraged away (capacity constraint, complexity, obscurity, etc.)
4. Define the falsifiable prediction: "If this edge exists, then [measurable outcome] will hold in data."
5. Assign prior confidence: Low / Medium / High with justification.

**Output:** A one-page `hypothesis.md` inside `experiments/EXP-XXX/`.

**Hard rule:** No experiment begins without a written hypothesis. No exceptions.

---

## SKILL 2 — Data Request & Validation

**Trigger:** Hypothesis is written; data needed to test it.

**Protocol:**
1. Open GitHub Issue in `market-data-pipeline` repo with:
   - Exact Polymarket market condition IDs (not descriptions — IDs)
   - Time range (ISO 8601)
   - Fields: `[mid_price, best_bid, best_ask, volume, timestamp]`
   - Required granularity (tick-by-tick vs 1min bars)
2. When data arrives in `data/processed/`, validate:
   - No look-ahead: timestamp of signal must precede timestamp of entry
   - No gaps > 10% of series without documented reason
   - Distribution sanity: prices must stay in (0, 1)
   - Schema matches expected columns

**Output:** `data/processed/<dataset_name>_validated.parquet` + validation log.

---

## SKILL 3 — Signal Construction

**Trigger:** Validated data available; hypothesis ready to test.

**Protocol:**
1. Write pseudocode first — no code before the logic is clear
2. Vectorise with NumPy. Never loop over rows where broadcasting works.
3. Construct signal as: `signal_t ∈ [-1, 0, +1]` or continuous `edge_t ∈ R`
4. Apply causal mask: `signal_t` can only use information available at `t`
5. Normalise: compute rolling Z-score with window = 168 periods (1 week of hourly data) or justified alternative
6. Document all hyperparameters in `config/signal_params.yaml` — no magic numbers in code

**Signal taxonomy (pick one):**
- `COHERENCE` — logical probability violation between markets
- `SENTIMENT` — news/social signal mapped to probability update
- `MICROSTRUCTURE` — OBI, trade flow, spread signal
- `RESOLUTION_BIAS` — systematic over/underpricing by resolution type
- `STALE_PRICE` — staleness score relative to correlated markets

**Output:** `src/signals/<signal_name>.py` with full type hints and docstrings.

---

## SKILL 4 — Statistical Modelling & Probability Calibration

**Trigger:** Signal constructed; need to convert to P_true estimate.

**Protocol:**
1. Choose generative model (document the choice):
   - Binary outcome → Beta-Binomial: `p ~ Beta(α,β); k|p ~ Binomial(n,p)`
   - Time-series → Bayesian state-space with Kalman filter
   - Classification → Isotonic regression calibration on logistic output
2. Estimate parameters on IN-SAMPLE data only (first 70% by time)
3. Validate calibration on OUT-OF-SAMPLE (last 30% by time):
   - Brier Score: target < 0.20 for binary markets
   - Reliability diagram: bins of 0.1 width, visual inspection
   - Expected Calibration Error (ECE): target < 0.05
4. Bayesian update formula: `P(H|E) = P(E|H)·P(H) / P(E)`
   — always start with base rate prior before updating on signal

**Libraries:** `scipy.stats`, `statsmodels` — no sklearn unless justified.

**Output:** `src/models/<model_name>.py` with calibration diagnostics in `results/calibration/`.

---

## SKILL 5 — Backtesting

**Trigger:** Model calibrated; ready to simulate strategy PnL.

**Protocol — non-negotiable steps:**
1. **Cost model first.** Fills at `best_ask` (buys) and `best_bid` (sells). Apply taker fee of 2bps. If spread > 6bps, flag the market as illiquid.
2. **No look-ahead.** All signals computed with `t-1` data. Verify with timestamp audit.
3. **Split:** 70% in-sample (IS), 30% out-of-sample (OOS). OOS never touched during development.
4. **Entry/exit logic.** Document exactly: entry at `signal_t > threshold`, exit at `signal_t < exit_threshold` or time stop.
5. **Kelly sizing.** Use quarter-Kelly on calibrated `p_true`. Hard cap: 5% bankroll per trade.
6. **Compute metrics:**
   - Sharpe (annualised): `μ_excess / σ · √T`
   - Max drawdown
   - Win rate
   - Turnover (positions/day)
   - Capacity estimate ($ before market impact degrades edge by 50%)
7. **Bias audit checklist:**
   - [ ] Survivorship bias: are resolved markets included?
   - [ ] Look-ahead: does signal use future data?
   - [ ] Overfitting: is Sharpe dramatically higher IS vs OOS?
   - [ ] Selection bias: was this market chosen because it looked good?

**Graduate threshold:** OOS Sharpe ≥ 1.0 AND IS/OOS ratio < 2.0 AND bias audit all-clear.

**Output:** `results/backtest_<experiment_id>.html` + `results/metrics.json`.

---

## SKILL 6 — Statistical Arbitrage (Multi-Market)

**Trigger:** Hypothesis involves correlated or logically linked markets.

**Protocol:**
1. Run Engle-Granger cointegration test (via `statsmodels.tsa.stattools.coint`)
2. If cointegrated: fit OLS hedge ratio β, compute spread `Z_t = P_A_t - β·P_B_t`
3. Normalise: `z_score_t = (Z_t - μ_Z) / σ_Z` (rolling 168-period window)
4. Signal: enter when `|z_score_t| > 2.0`, exit at `|z_score_t| < 0.5`
5. For logical arbitrage (mutually exclusive exhaustive markets):
   - Check: `sum(P_i) - 1.0 > fee_threshold` → exploitable
   - Check: `P(A∩B) > min(P(A), P(B))` → coherence violation
6. Always compute: correlation of arb trades to existing positions before sizing

**Output:** Spread diagnostics + cointegration report in `results/arb_analysis/`.

---

## SKILL 7 — Experiment Documentation

**Trigger:** Experiment complete (pass or fail).

**Protocol:**
The `experiments/EXP-XXX/README.md` must contain:
- **Hypothesis** (one paragraph)
- **Mathematical model** (LaTeX-style notation)
- **Data** (exact sources, time range, snapshot reference)
- **Methodology** (step-by-step, reproducible)
- **Results** (IS and OOS metrics)
- **Failure modes** (minimum 3, even for passing experiments)
- **Decision** (GRADUATE / REJECT / CONTINUE) with justification
- **Related experiments** (cross-reference)

The `metrics.json` must conform to the canonical schema (see §4 of ecosystem doc).

**Non-negotiable:** Every experiment must be reproducible from `config.yaml` alone.

---

## SKILL 8 — Edge Decay & Monitoring Design

**Trigger:** Strategy graduated; need to specify monitoring requirements.

**Protocol:**
1. Estimate edge half-life: fit exponential decay to rolling Sharpe over time
2. Specify: "This edge is expected to decay to Sharpe < 1.0 within X months"
3. Define monitoring trigger: "If 30-day rolling Sharpe drops below 0.8, escalate to human review"
4. Document correlation to market events (elections, regulatory changes, platform updates)
5. Write the monitoring spec in `docs/monitoring/<strategy_name>_decay.md`

**Output:** Monitoring spec consumed by quant-infrastructure-agent.

---

## Quick Reference: Core Formulas

| Concept | Formula | When to use |
|---|---|---|
| Edge | `P_true - P_market` | Always compute before entering |
| EV | `p·profit - (1-p)·stake` | Size validation |
| Full Kelly | `f* = (p·b - q) / b` | Then divide by 4 |
| Brier Score | `(1/N)·Σ(p_i - o_i)²` | Calibration check |
| OBI | `(bid_vol - ask_vol)/(bid_vol + ask_vol)` | Microstructure signal |
| Z-score | `(x - μ) / σ` | Normalise spreads |
| Sharpe | `μ_excess / σ · √T` | Strategy quality gate |
| ECE | `Σ(n_b/N)·\|acc_b - conf_b\|` | Calibration quality |
