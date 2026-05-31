# Retrospective Validation

The validation workflow tests forecast credibility by comparing historical cutoff forecasts against the latest accepted observation.

## Validation Logic

The service:

1. extracts the latest usable observation for the selected AOI;
2. treats that current observation as strict ground truth;
3. builds a same-month historical lookback window;
4. defines yearly cutoff cases, such as one-year and two-year historical forecasts;
5. reruns the forecast from those historical cutoffs toward the present target month;
6. compares each case against the current target using compact error metrics and map overlays.

## Why Same-Month Cutoffs Matter

Vegetation edge position can show seasonal behavior. Same-month retrospective testing reduces seasonal mismatch and makes the comparison easier to interpret.

## Backfill Diagnostics

The validation service can diagnose missing recent cutoff years, explain why cases are unavailable, and generate backfill planning information for operational recovery.

## Expected Use

Retrospective validation is intended to support public and internal confidence in the forecast route. A shorter-horizon case should generally remain closer to the current target than a longer-horizon case, subject to data availability and scene quality.
