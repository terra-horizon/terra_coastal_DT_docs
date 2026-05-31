# API Surface

This page summarizes the public service surface documented for TERRA UGLA. The maintained OpenAPI source is available as [openapi.yaml](openapi.yaml).

## Health and Status

- `GET /healthz` returns service health.
- `GET /check_sentinel_hub_status` reports Sentinel Hub or imagery access availability.

## AOI and Geometry

- `GET /aoi` lists AOIs.
- `GET /aoi/prepared` lists prepared AOIs available to the UI.
- `POST /aoi` creates an AOI from polygon coordinates.
- `GET /aoi/<aoi_id>/timeseries/status` returns AOI dataset readiness.
- `GET /aoi/<aoi_id>/geometry/fixed` returns fixed reference line and transects.
- `POST /aoi/<aoi_id>/history/context` rebuilds and summarizes historical context.

## Runtime Extraction and Forecast

- `POST /aoi/<aoi_id>/extract/latest` extracts the latest usable vegetation-edge observation.
- `POST /aoi/<aoi_id>/forecast/latest` runs forecast inference from the latest runtime observation.

## Validation

- `POST /aoi/<aoi_id>/validation/history` builds strict same-month validation history and suggests default cases.
- `POST /aoi/<aoi_id>/validation/run` executes retrospective validation cases.
- `POST /aoi/<aoi_id>/validation/backfill-plan` diagnoses missing cutoff years and produces recovery guidance.

## Digital Twin and Jobs

- `GET /dt/bootstrap` loads digital twin bootstrap state.
- `POST /dt/predict` runs prediction from digital twin state.
- `POST /jobs/extract` queues extraction work.
- `POST /jobs/predict` queues prediction work.
- `POST /jobs/assimilate` triggers an assimilation cycle.
- `GET /jobs/<job_id>` queries job status.

## Results

- `GET /results/<run_id>/summary` returns run summary metadata.
- `GET /results/<run_id>/download` downloads exported artifacts.
- `GET /runs/<run_id>/artifacts/<path>` serves run artifacts such as previews and exports.
