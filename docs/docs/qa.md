# Quality Assurance

Quality assurance covers four layers: the code, the observations, the model and the deployment.

## Automated tests

| Suite | Scope |
| --- | --- |
| Service tests | API contracts, web pages, extraction quality control, hybrid forecaster, validation service, storage settings and isolation, provenance and downloads |
| Ingestion package tests | Contract schemas and examples, catalogue handling, AOI screening, raster preflight, orchestration decisions; fully offline with synthetic rasters |
| Storage contract tests | Run against throwaway authenticated MongoDB and MinIO instances, never against live stores |

Tests that need live services are skipped when those are not configured, so the suites run anywhere.

## Observation quality

- **AOI-level cloud screening** before download, and structural and coastal preflight checks after.
- **Extraction quality control:** per-scene transect coverage, reach limits from the reference line, and the temporal prior against single-scene failures.
- **Signal gate** for every AOI before it is forecast.
- **Review:** operators can record a review decision for any observation.
- **Rejection records:** every rejected scene keeps its reasons.

## Model quality

- **Rolling-origin evaluation** with no look-ahead: the most recent 24 months of every AOI are held out, and metrics are averaged per origin, then per AOI.
- **Comparison with baselines** in every evaluation: persistence, trend only, a plain LSTM and the statistical base.
- **Per-horizon gates** limit the MambaLSTM correction to where it has proven useful.
- **Interval coverage** is reported for every horizon (72 to 76% for a nominal 80%).
- **Independent window:** the protocol can be repeated on an earlier period to check that rankings hold.
- **Shared code for training and serving:** model inputs are built by the same functions, so the deployed model sees exactly what it was evaluated on.

See [Training & Evaluation](ai-model/training-evaluation.md) for the results.

## Provenance and reproducibility

- Every observation carries a pipeline signature; every forecast records its observation, dataset and model versions.
- Identical requests reuse stored results rather than recomputing them.
- Objects are immutable and checksummed; cached copies are verified before use.

## Workflow checks

Before a release, the main workflows are checked end to end on a live AOI: select an AOI, load geometry, extract or reuse the latest observation, load history, forecast with bounds and erosion risk, compare historical coastlines, run validation cases, and download an artifact.

## Interpretation

Automated checks support interpretation but do not replace expert judgement. Storms, engineering works, sediment supply and vegetation management can change a coast in ways an automated forecast does not capture.
