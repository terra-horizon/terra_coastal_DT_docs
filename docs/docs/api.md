# API Surface

TERRA Coastal DT exposes a JSON HTTP API used by its web interface, by the TERRA platform workflows and by integrators. The service publishes its own OpenAPI description at `/openapi.yaml`; a copy is available here as [openapi.yaml](openapi.yaml).

## Conventions

- Requests and responses are JSON. Most responses carry `success` and, on failure, an `error` message.
- Long-running work returns `202 Accepted` with a job identifier; poll `GET /jobs/<job_id>` for phase events and the result.
- Job submissions accept an `idempotency_key`, so a retried request returns the existing job.
- Map outputs are GeoJSON in WGS 84; distances are metres along transects, positive seaward.
- Downloads redirect to short-lived presigned object-storage links.

## Authentication

- `/models/*` (promotion and rollback) requires an administrator bearer token: `Authorization: Bearer <token>`.
- User and service authentication is handled by the TERRA platform gateway in front of the service.

## Typical forecast sequence

1. `GET /aoi/prepared`: list AOIs available for forecasting.
2. `GET /aoi/<aoi_id>/geometry/fixed`: load the reference line and transects.
3. `POST /aoi/<aoi_id>/extract/latest`: extract the newest usable observation.
4. `POST /aoi/<aoi_id>/history/context`: load the historical context.
5. `POST /aoi/<aoi_id>/forecast/latest`: forecast from that observation.
6. Display the returned layers; fetch artifacts through `GET /artifacts/<artifact_id>`.

## Health

| Route | Purpose |
| --- | --- |
| `GET /health/live` | Process liveness; no dependency checks |
| `GET /health/ready` | Readiness: MongoDB, UC3 collections and MinIO reachable (`503` otherwise) |
| `GET /healthz` | Combined health and imagery-access flag |
| `GET /check_sentinel_hub_status` | Imagery provider availability |

## AOIs and geometry

| Route | Purpose |
| --- | --- |
| `GET /aoi/prepared` | AOIs with fixed geometry and a forecaster; retired AOIs are hidden |
| `GET /aoi` | All AOIs |
| `GET /aoi/<aoi_id>/geometry/fixed` | Active reference line and transects |
| `GET /aoi/<aoi_id>/timeseries/status` | Dataset readiness and scene counts |

## Observation and forecast

| Route | Body (main fields) | Purpose |
| --- | --- | --- |
| `POST /aoi/<aoi_id>/extract/latest` | `force_refresh` | Search, screen and extract the newest usable scene; reuses a cached observation when nothing newer exists |
| `POST /aoi/<aoi_id>/history/context` | `years`, `month` | Historical context for the forecast page |
| `POST /aoi/<aoi_id>/history/coastlines` | `years` (default 5), `month` | Same-month vegetation-edge lines for the last N years and their net change |
| `POST /aoi/<aoi_id>/forecast/latest` | `run_id`, `profile` (`yearly` or `semiannual`), `steps_ahead` | Forecast from the latest observation. The horizon is 12 or 6 months times `steps_ahead` |

A forecast response for a MambaLSTM AOI includes the p10 / p50 / p90 GeoJSON lines, the uncertainty band, erosion-risk polygons and a `forecast_method` block with the base and correction components, the gate, the network horizon, interval calibration, the `horizon_extrapolated` flag and `model_skill` (the deployed model's test RMSE at this AOI).

## Validation

| Route | Body (main fields) | Purpose |
| --- | --- | --- |
| `POST /aoi/<aoi_id>/validation/history` | `run_id`, `lookback_years` (default 5), `preferred_case_count` | Find same-month historical cut-offs and suggest cases |
| `POST /aoi/<aoi_id>/validation/run` | `run_id`, `cases`, `lookback_years` | Run retrospective cases and score every method |
| `POST /aoi/<aoi_id>/validation/backfill-plan` | | Explain missing cut-off years and how to recover them |

## Digital twin, jobs and review

| Route | Purpose |
| --- | --- |
| `POST /jobs/assimilate` | Assimilate committed scene assets (`aoi_id`, `scene_asset_ids`, `idempotency_key`, `correlation_id`) |
| `POST /jobs/extract` | Extract the vegetation edge from given scenes |
| `POST /jobs/predict` | Queue a forecast |
| `POST /jobs/retrain` | Candidate training for per-AOI legacy forecasters |
| `GET /jobs/<job_id>` | Job status, phase events and result |
| `GET /aois/<aoi_id>/observations` | Observation history of an AOI |
| `POST /observations/<observation_id>/review` | Record a review decision (`decision`, `reason_codes`, `note`) |
| `GET /dt/bootstrap`, `POST /dt/predict` | Digital twin state summary and prediction |

## Model registry

| Route | Purpose |
| --- | --- |
| `GET /aois/<aoi_id>/model-status` | Active model, candidate and retraining eligibility |
| `POST /models/<model_version_id>/promote` | Promote a candidate and its dataset (administrator) |
| `POST /models/<model_version_id>/rollback` | Restore the previous model and dataset (administrator) |

## Results and downloads

| Route | Purpose |
| --- | --- |
| `GET /artifacts/<artifact_id>` | Resolve a committed artifact; redirects to a presigned link (`redirect=false` returns the reference) |
| `GET /results/<run_id>/summary` | Run summary |
| `GET /results/<run_id>/download` | Download exported run products |
| `GET /runs/<run_id>/artifacts/<path>` | A run's previews and exports |

## Diagnostics

A workflow that cannot complete returns a diagnostic rather than a partial result. Common causes:

- no recent scene passed AOI-level cloud screening or extraction;
- the imagery provider is unavailable;
- the AOI has no fixed geometry or not enough history;
- storage is unavailable (`503` from readiness and storage-backed routes);
- an administrator token is missing or wrong (`403` on `/models/*`).
