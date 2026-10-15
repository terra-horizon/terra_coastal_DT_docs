# Frequently Asked Questions

## General

### What is TERRA Coastal DT?

The coastal digital twin of the TERRA platform. It measures the vegetation edge of dune coasts from Sentinel-2 imagery, keeps a monthly history for each site, forecasts the edge 1 to 24 months ahead with uncertainty, and validates those forecasts against later observations.

### Which sites are covered?

15 live AOIs on the east and west coasts of Scotland, from St Andrews West Sands to Morrich More, in 8 independent coastal groups.

### What is a transect?

A fixed cross-shore line along which the vegetation edge is measured. Transects are spaced 15 m apart, and distances are in metres from the AOI's reference line, positive seaward.

### Why the vegetation edge and not the waterline?

The waterline moves by tens of metres with tide, waves and surge within hours. The vegetation edge reflects erosion and accretion over months and years, which is the change that matters for coastal management.

## The AI model

### What is MambaLSTM?

The core AI model of the digital twin: a deep sequence model with three Mamba-style blocks (causal convolution with gating) followed by an LSTM, 192,641 parameters in total. It reads 12 months of a statistical forecast's errors across 27 neighbouring transects (390 m of coast), plus 16 context features, and predicts the error of the next forecast. See [MambaLSTM Forecasting Model](ai-model/mambalstm.md).

### Why not let the neural network forecast the coast directly?

Because a closed-form model already captures level, seasonality and short-term memory well and adapts instantly to new data. Asking the network to learn only what that model misses makes it smaller, safer (predicting zero reproduces the base) and more accurate. Against a plain LSTM that forecasts directly from a transect's history, the hybrid's RMSE is 3 to 11% lower.

### How much does MambaLSTM improve the forecast?

On held-out test data it reduces RMSE against the statistical base by 2.9 to 6.5%, most at 12 and 24 months, and the hybrid is the most accurate of all methods tested at every horizon. See [Training & Evaluation](ai-model/training-evaluation.md).

### What if the network learns something that does not generalise?

A per-horizon gate, fitted on held-out data, scales the correction between 0 and 1. Where the correction does not help, the gate shrinks it and the forecast falls back to the statistical base.

### Is there one model per site?

No. One MambaLSTM serves all 15 AOIs; each site's behaviour enters through its own data. Training on all sites together gives the network far more examples than any single site's record.

### Does the model need retraining when new images arrive?

No. The statistical base is refitted on the whole record at every forecast, so new observations count immediately. The shared network is retrained occasionally, when substantial new history or new sites are added.

## Forecasts

### What does the forecast show?

The median forecast edge (p50), an 80% interval (p10 to p90) and, optionally, erosion-risk areas where even the conservative p10 forecast lies landward of today's edge.

### How reliable is the uncertainty band?

On test data the nominal 80% band contains 72 to 76% of outcomes, so it is slightly narrow. Treat it as a guide to relative certainty along the coast rather than an exact probability.

### What about horizons beyond two years?

They are produced but flagged `horizon_extrapolated`, because the intervals are calibrated only up to 24 months.

### Are forecasts decisions?

No. They are decision support, to be read alongside local knowledge, expert assessment and other evidence. Storms, surges and coastal works are not yet model inputs.

## Observations and validation

### Why might the latest observation fail?

Recent scenes may be too cloudy over the AOI, the imagery provider may be unavailable, or extraction may not find a reliable edge. The page reports which.

### What does retrospective validation mean?

The forecast is rerun from the same month one or more years earlier, using only data available then, and compared with today's observation. Each case also scores the statistical base, a trend model and a plain LSTM.

### Why might a validation case be missing?

No usable scene in that month, scenes rejected by screening or quality control, history not yet extracted, or a look-back window that is too short. The backfill plan explains the cause.

## Integration

### How does data reach the digital twin in the platform?

The TERRA ingestion service runs the UC3 acquisition profile, commits screened scenes to shared storage, and Airflow asks the digital twin to assimilate them. See [Data Ingestion](data-ingestion.md).

### Where is the OpenAPI description?

The service publishes it at `/openapi.yaml`; a copy is available here as [openapi.yaml](openapi.yaml).
