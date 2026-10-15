# Retrospective Validation

The validation page (`/validation`) answers a practical question: if the digital twin had issued a forecast one or two years ago, how close would it have been to what is observed today? It also shows how the MambaLSTM hybrid compares with simpler methods on the same case.

## How a case works

1. **Ground truth.** The latest extracted observation of the AOI is the target.
2. **Cut-offs.** The service finds earlier years with an observation in the same calendar month as the target (within a look-back window, five years by default) and suggests cases, for example one year and two years before the target.
3. **Re-forecast.** For each case, the forecaster is run from the cut-off month using only data up to that month, exactly as it would have seen the record then.
4. **Score.** Each case's forecast is compared with the target on every transect, and shown as map overlays and error metrics.

Same-month cut-offs keep the seasonal cycle out of the comparison: a one-year case forecasts from the same season it is scored against.

## Methods scored in every case

For MambaLSTM AOIs, each case scores several methods from the same origin:

| Method | What it shows |
| --- | --- |
| **Hybrid (MambaLSTM)** | The deployed forecaster |
| Base | The statistical base without the MambaLSTM correction: the value added by the network |
| Trend | A straight per-transect trend, extrapolated |
| Naive LSTM | A plain LSTM on the transect's own history |

Persistence ("no change") is not offered as a method on this page. At same-month horizons on slowly changing coasts it mainly reflects how little the edge moved; its rolling-origin results are reported in [Training & Evaluation](ai-model/training-evaluation.md).

## One case is one draw

A single retrospective case is one sample of forecast error and can favour any method by chance. The page therefore also shows `model_skill`: the deployed model's rolling-origin test RMSE at this AOI, computed over every forecast origin in the 24-month test period. That is the statistically meaningful number; the cases make it visible on the map.

## Diagnostics

When an expected cut-off year is missing, the backfill plan explains why: no imagery in that month, scenes rejected by cloud screening or extraction quality, history not yet extracted, or a window too short. It also suggests how to recover the missing months.

## Interpretation

Validation does not guarantee future accuracy. It shows whether the forecaster has behaved consistently on recent history, how error grows with horizon, and where on the coast forecasts tend to miss.
