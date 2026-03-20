# EXP-001: RSA Foundation — Data + Model Validation

## Hypothesis
Range Sniping Arbitrage exploits probability mass leakage in Polymarket mutually exclusive
range markets. When Σ(ask_i) for a feasibility-filtered subset S < 0.85 (time-adjusted),
buying the subset yields positive EV conditional on resolution within S.

## Expected Edge
Edge = C × (1/P_S - 1) where P_S = Σ ask_i for i ∈ S
Break-even (with 2bps fee): P_S < (1 - 2φ) / (1 + φ) ≈ 0.9608

## Data Required
- Polymarket CLOB API: historical tweet-count markets (90 days)
- Twitter/X API v2: @elonmusk tweet history (90 days)

## Phase 1 Exit Criteria
- [ ] Poisson model Brier Score < 0.10
- [ ] 15+ qualifying P_S < 0.85 signals identified in historical data
- [ ] Calibration Surface C(K,τ) validated on 10 historical markets

## Status
PENDING — Phase 1 data collection not yet started

## Decision
TBD

