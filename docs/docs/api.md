# API Surface

This page summarizes the public service surface documented for TERRA Coastal DT. The maintained OpenAPI source is available as [openapi.yaml](openapi.yaml).

The API is intended for web interfaces, platform integration, workflow automation, and result retrieval. Endpoint availability may depend on deployment configuration, imagery access, prepared AOI data, and enabled processing services.

## General Conventions

Most workflow endpoints return JSON payloads with status fields, metadata, and map-ready geometry references or GeoJSON content. Longer-running operations may return a job identifier that can be polled through the job status endpoint.

Typical responses may include:

- selected AOI metadata;
- scene or observation metadata;
- quality indicators;
- GeoJSON layers for map display;
- forecast lines and uncertainty bounds;
- validation metrics;
- diagnostic messages when a workflow cannot continue.

Authentication and authorization are deployment-specific. Public documentation describes the API surface, while operational deployments define who can access AOIs, trigger jobs, and download artifacts.

## Typical Forecast Sequence

A client usually follows this sequence:

1. list prepared AOIs;
2. inspect AOI timeseries and fixed-geometry readiness;
3. extract or load the latest usable observation;
4. rebuild or load historical context;
5. run the latest forecast;
6. display returned map layers and result metadata;
7. download or inspect result artifacts where enabled.

## Typical Validation Sequence

A validation workflow usually follows this sequence:

1. select an AOI;
2. build validation history for the target month;
3. review available historical cutoff cases;
4. run validation;
5. compare map overlays and summary metrics;
6. request a backfill plan if expected historical cases are missing.

## Health and Status

- `GET /healthz` returns service health.
- `GET /check_sentinel_hub_status` reports imagery access availability where that check is enabled.

## AOI and Geometry

- `GET /aoi` lists AOIs.
- `GET /aoi/prepared` lists prepared AOIs available to the UI.
- `POST /aoi` creates an AOI from polygon coordinates where AOI creation is enabled.
- `GET /aoi/<aoi_id>/timeseries/status` returns AOI dataset readiness.
- `GET /aoi/<aoi_id>/geometry/fixed` returns fixed reference line and transects.
- `POST /aoi/<aoi_id>/history/context` rebuilds or summarizes historical context.

## Runtime Extraction and Forecast

- `POST /aoi/<aoi_id>/extract/latest` extracts the latest usable vegetation-edge observation.
- `POST /aoi/<aoi_id>/forecast/latest` runs forecast inference from the latest AOI state.

## Validation

- `POST /aoi/<aoi_id>/validation/history` builds same-month validation history and suggests available cases.
- `POST /aoi/<aoi_id>/validation/run` executes retrospective validation cases.
- `POST /aoi/<aoi_id>/validation/backfill-plan` diagnoses missing cutoff years and produces recovery guidance.

## Digital Twin and Jobs

- `GET /dt/bootstrap` loads digital twin bootstrap state.
- `POST /dt/predict` runs prediction from digital twin state.
- `POST /jobs/extract` queues extraction work.
- `POST /jobs/predict` queues prediction work.
- `POST /jobs/assimilate` triggers an assimilation cycle where enabled.
- `GET /jobs/<job_id>` queries job status.

## Results

- `GET /results/<run_id>/summary` returns run summary metadata.
- `GET /results/<run_id>/download` downloads exported artifacts where downloads are enabled.
- `GET /runs/<run_id>/artifacts/<path>` serves run artifacts such as previews and exports.

## Diagnostics and Errors

Users and integrators should expect diagnostic responses when a workflow cannot complete. Common causes include:

- no usable recent satellite scene;
- cloud or valid-pixel quality failure;
- missing AOI fixed geometry;
- insufficient historical observations;
- unavailable imagery provider access;
- disabled job or download capability in the current deployment.

These diagnostics are part of normal workflow interpretation and should be surfaced clearly in user interfaces.
