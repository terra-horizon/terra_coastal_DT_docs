# Technical Approach

This page explains the main technical decisions behind TERRA Coastal DT and the evidence that shaped them.

## 1. Coastal state as a digital twin variable

The digital twin treats the coastline as an evolving state, not as a line extracted from one image. For each AOI it maintains:

- a fixed measurement frame (reference line and transects);
- the observed vegetation-edge history on that frame;
- the latest observed state;
- forecasts with uncertainty, and evidence of how well past forecasts did.

## 2. The vegetation edge as the state variable

The waterline moves by tens of metres with tide, waves and surge within hours. The vegetation edge moves with erosion and accretion over months and years, which is the change coastal management needs to track. It is the primary state variable; other lines are supporting information only.

## 3. A fixed measurement frame

Observations from different dates must be measured in the same frame, or real change gets mixed with changes in the frame.

- The **reference line** is traced on a temporal median composite of many clear scenes. This replaced an earlier approach that seeded the line from one scene with a segmentation network: the network failed on coasts it had not seen, its traced boundary followed the image border where vegetation reached it, and a single date biased the datum at sites with a trend. The composite needs no trained model and sits at the long-term mean edge.
- **Transects** every 15 m give a dense, regular alongshore grid, oriented positive seaward.
- Site-specific problems (a line ending beyond the beach, a hook the line cut across, a spit tip) are fixed as new geometry versions, not by changing the history.

## 4. Robust extraction

Scenes acquired within the same month, when the coast has not really moved, show the structure of extraction error. Extraction is precise most of the time (median repeat difference 3.2 m, a third of a Sentinel-2 pixel), but when it fails, it fails coherently over runs of neighbouring transects. A coherent run looks smooth alongshore, so spatial smoothing alone cannot catch it; time can, because a run that is wrong in one scene is right in its neighbours in time. The extraction design reflects that:

- candidates from several cues (threshold crossing, contour, gradient) on each transect;
- joint alongshore selection with a smoothness penalty, preserving well-supported headlands and spits;
- a leave-one-out temporal prior over the nearest scenes in time, which roughly halved the noise floor at every AOI while preserving known trends;
- a **signal gate** before any AOI is forecast: if month-to-month differences show that the record is mostly measurement noise, no model can forecast it, and the AOI is not used.

## 5. Transect distances, not image masks

Forecasting distances along fixed transects, rather than future image masks, gives values with physical meaning (metres), uses the same frame for history, present and future, makes missing or weak transects visible, and turns uncertainty into lower, median and upper lines on the map.

## 6. Monthly composites at native spacing

Each month's accepted scenes are combined into a per-transect median, and values that disagree strongly with their alongshore neighbours are removed. The series keep the native 15 m transect spacing, and gaps stay missing values that the model masks.

An earlier design resampled every coastline to a fixed grid and compressed it with principal components. It was dropped: the fixed grid made the same position mean a different place at each AOI and forced gaps to be filled in. The measured alongshore correlation length, 90 to 105 m at every AOI, sets instead the patch width, the outlier-rejection window and the smoothing scale. Because the patch layout does not depend on coastline length, AOIs of any size share one model.

## 7. A hybrid AI forecaster

The forecaster combines a statistical base with the [MambaLSTM](ai-model/mambalstm.md) deep sequence model:

- **The base** captures level, seasonal cycle and short-term memory, refitted on the whole record at every request, so new observations count immediately.
- **The trend sets the level but is not extrapolated.** Trends fitted on the two halves of a record correlate at only -0.49 to +0.33. Dropping trend extrapolation cut 24-month error by 9 to 16% in two independent test windows.
- **MambaLSTM learns the base's errors** from a year of residuals across 27 neighbouring transects and 16 context features, shared across all AOIs. It finds what the base cannot: persistent level offsets (40 to 60% of a year-long offset persists into 12 to 24-month targets) and trend reversion (about half the fitted trend).
- **A per-horizon gate** scales the correction by its proven usefulness on held-out data, so the hybrid falls back to the base wherever the network adds nothing.
- **Calibrated intervals** come from conformal error quantiles per AOI and horizon.

## 8. Evaluation without look-ahead

Every result is produced under a rolling-origin protocol: each forecast uses only data up to its own origin, the most recent 24 months of every AOI are held out for testing, and metrics are averaged per origin and then per AOI. This protocol exposed that an earlier static baseline's apparent skill came from look-ahead, and it is the basis for every reported number. Under it, the hybrid is the most accurate of all methods tested at every horizon (see [Training & Evaluation](ai-model/training-evaluation.md)).

## 9. Database-first digital twin

State lives in MongoDB and MinIO, not in the service container. Observations, datasets, models and forecasts are versioned and linked by signatures, so every forecast can be traced to the observation, dataset, model and code that produced it, and an identical request reuses the stored result.

## Limitations

- Accuracy depends on clear-sky imagery; long cloudy periods leave gaps.
- The model uses only the coastline's own history; storms, surges and wave climate are not yet inputs.
- Uncertainty intervals are slightly narrow (72 to 76% coverage for a nominal 80%).
- The approach is validated on Scottish dune coasts; other settings need their own validation.
- Forecasts are decision support and should be read alongside local knowledge and expert assessment.
