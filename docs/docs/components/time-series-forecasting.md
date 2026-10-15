# Coastal Change Time-Series Analysis and Forecasting

This component turns the vegetation-edge record of each AOI into monthly per-transect time series, forecasts the edge 1 to 24 months ahead with the **MambaLSTM hybrid forecaster**, and supports retrospective validation of those forecasts.

```text
VE observations -> monthly series per transect -> statistical base + MambaLSTM correction -> p10 / p50 / p90 -> maps, erosion risk, validation
```

## Monthly time series

Scenes arrive irregularly because of cloud. For forecasting, each AOI's record is turned into a regular monthly series:

- **monthly composite:** the median of the month's accepted scenes for every transect;
- **alongshore outlier rejection:** a value that disagrees strongly with its alongshore neighbours in the same month is removed, because real change is coherent over about 100 m of coast while extraction errors are not;
- **patch layout:** each transect is stored with its neighbourhood of 27 transects (390 m), the input unit of the MambaLSTM model;
- **versioned datasets:** each dataset version records its extraction parameters and time span, and the model that serves an AOI is pinned to a dataset version.

## Forecasting

Forecasts are produced by the hybrid forecaster described in [MambaLSTM Forecasting Model](../ai-model/mambalstm.md):

\[
\text{p50} \;=\; \underbrace{\text{level} + \text{season} + \text{short-term memory}}_{\text{refitted at every request}} \;+\; g_h \cdot \text{MambaLSTM correction}
\]

- The **statistical base** is refitted on the AOI's whole record up to the newest observation at every request, so the forecast always starts from the current state.
- The **MambaLSTM** correction, trained on all 15 AOIs, adds what the base misses: persistent level offsets, trend reversion and alongshore context.
- **p10 and p90** come from conformal error quantiles calibrated per AOI and horizon.

| Horizon | Test RMSE (hybrid) | Improvement over the base | Improvement over persistence |
| --- | --- | --- | --- |
| 1 month | 9.15 m | 3.4% | 9.3% |
| 3 months | 10.54 m | 4.2% | 25.1% |
| 6 months | 11.12 m | 2.9% | 33.2% |
| 12 months | 10.86 m | 6.5% | 15.7% |
| 24 months | 12.11 m | 5.9% | 13.9% |

Full results and the comparison with a plain LSTM are in [Training & Evaluation](../ai-model/training-evaluation.md).

## Forecast products

| Product | Description |
| --- | --- |
| Forecast lines | p50 (median), p10 and p90 vegetation-edge lines reconstructed on the map from transect distances |
| Uncertainty band | The area between p10 and p90 |
| Erosion-risk areas | Polygons where the conservative p10 forecast lies landward of the current vegetation edge |
| Forecast table | Per-transect p10 / p50 / p90 distances, stored as queryable measurements |
| Historical coastlines | The vegetation edge in the same calendar month of each of the last N years, with the median net change between the first and last year |

Horizons beyond 24 months are returned with a `horizon_extrapolated` flag, because the intervals are only calibrated up to 24 months.

## Historical coastlines

Comparing the same month every year removes the seasonal cycle (5 to 33% of the temporal variance at these sites), so the differences between lines show year-to-year change and measurement noise. If a year lacks that month because of cloud, the adjacent month is used and labelled. Lines are smoothed over about 100 m, the measured alongshore correlation length, so per-transect noise is removed while real coastal shape is kept.

## Public interfaces

- `GET /aoi/<aoi_id>/timeseries/status`
- `POST /aoi/<aoi_id>/history/context`
- `POST /aoi/<aoi_id>/history/coastlines`
- `POST /aoi/<aoi_id>/forecast/latest`
- `POST /aoi/<aoi_id>/validation/history`
- `POST /aoi/<aoi_id>/validation/run`
- `POST /aoi/<aoi_id>/validation/backfill-plan`

## Current status

Implemented and in operation: monthly datasets for 15 AOIs, the MambaLSTM hybrid forecaster with calibrated intervals, erosion-risk products, historical coastline comparison, and retrospective validation that scores the hybrid against the base, a trend model and a plain LSTM.

## Planned work

- add exogenous drivers (wave height, storm surge, water level, precipitation) as model inputs, the main lever for longer horizons;
- widen the uncertainty calibration with a dedicated calibration period, to bring p10 to p90 coverage up to its nominal 80%.
