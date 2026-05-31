# Retrospective Validation

Retrospective validation helps users understand forecast credibility by comparing historical forecasts with later observed vegetation edge positions.

## Validation Logic

The service:

1. extracts or loads the current target observation for the selected AOI;
2. treats that observation as the reference state for validation;
3. builds a historical lookback window for the same target month;
4. defines one or more historical cutoff cases;
5. reruns forecasts from those cutoff dates toward the target date;
6. compares each forecast case with the observed target using map overlays and error metrics.

## Why Same-Month Cutoffs Matter

Vegetation edge position can show seasonal behavior. Same-month retrospective testing reduces seasonal mismatch and makes comparisons easier to interpret.

## Diagnostics

When expected historical cases are unavailable, the validation workflow can explain whether the issue is caused by missing imagery, scene quality, missing extracted history, or an insufficient time window.

## Expected Use

Validation does not prove that every future forecast will be correct. It gives users a practical way to inspect whether the forecast route has been consistent for recent historical cases and whether shorter-horizon cases are generally closer to the observed target than longer-horizon cases.
