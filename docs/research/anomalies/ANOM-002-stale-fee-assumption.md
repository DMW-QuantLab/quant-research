# ANOM-002: Fee assumption survived two revisions without re-checking the live platform

## What happened
Fee assumptions moved from a flat 2% (original), to "over-estimated, replaced
by dynamic modeling" per prior notes, to a flat 2bps hardcoded in
EXP-001/config.yaml — none matching Polymarket's actual live schedule (Fee
Structure V2, category-based, 5% for Culture, price-quadratic, effective
since 2026-03-30, confirmed still current 2026-09-19).

## Why it's surprising given the model
The "over-estimated" conclusion was reasonable when it was reached. Nothing
in the process re-checked it after Polymarket's fee structure actually
changed — an external, third-party-controlled parameter was treated as a
settled internal one.

## What reality might know that the model doesn't
Platform fees are policy, not physics — they change independent of anything
the strategy does. The assumption needed a "last verified against live
platform: <date>" stamp, not just a derivation.

## Status: explained and fixed — fees.py implements the verified Fee V2
model (merge pending). Re-verify category rates before every live capital
milestone, not just once.
