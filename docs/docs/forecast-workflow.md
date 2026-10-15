# Forecast Workflow

The main web page (`/`) takes a user from a coastal site to a forecast in five steps. Every step works in the AOI's fixed measurement frame, so the current edge, its history and the forecast can be compared directly on the map.

## Step 1: Select AOI

Choose one of the live AOIs. The list shows only AOIs with fixed geometry and an active forecast model; retired AOIs are hidden. The map zooms to the AOI and draws its reference line and transects.

## Step 2: Latest observation and current VE

**Extract Latest Observation** searches recent Sentinel-2 scenes, screens them for cloud over the AOI, and extracts the vegetation edge from the newest usable scene. The page shows:

- the accepted scene preview on the map;
- the current vegetation edge;
- the acquisition date, imagery source and run identifier.

If the latest usable scene has already been processed, the stored observation is returned without reprocessing. If no recent scene passes screening, the page explains why.

## Step 3: Historical context

**Load Historical Context** loads the AOI's monthly vegetation-edge record used by the forecaster, and reports the number of historical observations and the earliest and latest dates.

## Step 4: Transect forecast

Choose a horizon in years and select **Run Transect Forecast**. The MambaLSTM hybrid forecaster (see [MambaLSTM Forecasting Model](ai-model/mambalstm.md)):

1. adds the observation from Step 2 to the AOI's record;
2. refits the statistical base on the whole record;
3. applies the gated MambaLSTM correction for every transect;
4. adds the calibrated p10 and p90 bounds.

The map shows:

| Layer | Meaning |
| --- | --- |
| Current VE | The edge extracted in Step 2 |
| Forecast p50 | The median forecast edge |
| p10 / p90 | The bounds of the 80% forecast interval |
| Uncertainty band | The area between p10 and p90 |
| Erosion risk (toggle) | Areas where even the conservative p10 forecast lies landward of the current edge |

The status line reports the deployed model's held-out RMSE for this AOI at the chosen horizon, so each forecast comes with an honest error scale. Horizons beyond two years are produced but flagged as extrapolated, because the intervals are calibrated up to 24 months.

## Step 5: Historical coastline change

Choose how many past years to show (1 to 15). The page draws the vegetation edge in the same calendar month of each year, from lightest (earliest) to deepest red (latest), and reports the median net change between the first and last year, positive seaward. Using the same month every year keeps the seasonal cycle out of the comparison.

## Why fixed geometry matters

The reference line and transects are fixed before anything is measured or forecast. A change in the reported distance therefore means the vegetation edge moved, not that the measurement frame changed.
