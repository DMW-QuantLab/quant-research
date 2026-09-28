# ANOM-001: Project narrative described work that wasn't in the repos

## What happened
PROJECT_CONTEXT.md described three finished modules (arbitrage_core.py,
backtest_engine.py, kelly_optimizer.py) and a completed 100-market synthetic
backtest returning 0 trades. A full clone and grep of all five DMW-QuantLab
repos (2026-09-19) found none of those three files anywhere. The only formal
experiment record, EXP-001, showed status PENDING, all metrics null, Phase 1
data collection not started.

## Why it's surprising given the model
The zero-trade result was already being reasoned about with real Stage-3
(Break) discipline — "filters working, not a code failure" — applied to a
result that appears to have never been produced by any committed code.

## What reality might know that the model doesn't
A narrative continuity artifact (a .md summary written at the end of a
session) is not the same thing as a version-controlled experiment record, and
nothing before now reconciled the two. The gap itself, not its content, is
the finding.

## Status: explained — EXP-001 is now the single source of truth for backtest
status going forward, not any narrative .md file.
