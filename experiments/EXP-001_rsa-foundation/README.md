# EXP-001: RSA Foundation — Data + Model Validation

## Hypothesis
Range Sniping Arbitrage exploits probability mass leakage in Polymarket mutually exclusive
range markets. When Σ(ask_i) for a feasibility-filtered subset S < 0.85 (time-adjusted),
buying the subset yields positive EV conditional on resolution within S.

## Expected Edge
Edge = C × (1/P_S - 1) where P_S = Σ ask_i for i ∈ S
(proportional-to-price allocation: stake_i = p_i × C/P_S, equal shares C/P_S across S)

Break-even, net of Polymarket Fee Structure V2 taker fees (5% Culture-category rate,
fee = shares × rate × price × (1-price)):
  P_S < 1 - rate × Σ_i p_i(1 - p_i)
Per-subset, not a flat constant — see bots/rsa_bot/fees.py:fee_adjusted_breakeven().
(Prior "≈0.9608 at φ=2%" was both stale — real fee is 5% category-based, not a flat
2bps — and internally inconsistent: (1-2φ)/(1+φ) at φ=2% evaluates to 0.941, not 0.9608.)

## Data Required
- Polymarket CLOB API: historical tweet-count markets (90 days)
- Twitter/X API v2: @elonmusk tweet history (90 days)

## Phase 1 Exit Criteria
- [ ] Poisson model Brier Score < 0.10
- [ ] 15+ qualifying signals under min(ps_threshold, fee_adjusted_breakeven) identified in historical data
- [ ] Calibration Surface C(K,τ) validated on 10 historical markets

## Status
PENDING — Phase 1 data collection not yet started

## Decision
TBD
