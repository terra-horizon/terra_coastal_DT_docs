# Configuration

All configuration comes from environment variables, so the same image runs locally, in staging and in production.

## Secrets

Credentials are never stored in code, images or this documentation. Every secret can be passed either directly or as a file through a `<NAME>_FILE` variable (for example `MINIO_SECRET_KEY_FILE=/run/secrets/minio_secret_key`), which suits Docker and orchestrator secrets.

## Storage

| Variable | Purpose |
| --- | --- |
| `TERRA_STORAGE_BACKEND` | `mongo_minio` for deployments; `local` (files only) for development |
| `TERRA_PROJECT_ID`, `TERRA_USE_CASE_ID` | Scope of every stored record (`UC3`) |
| `MONGODB_URI`, `MONGODB_DATABASE` | MongoDB connection (secret) and database |
| `TERRA_MONGODB_COLLECTION_PREFIX` | Collection prefix; `uc3_` is required in a shared database |
| `MINIO_ENDPOINT`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY` | Object storage connection (keys are secrets) |
| `MINIO_BUCKET`, `MINIO_PREFIX` | Bucket and the UC3 object prefix within it |
| `MINIO_REGION` | Set it to avoid a bucket-location lookup that prefix-scoped keys may not be allowed |
| `MINIO_PUBLIC_ENDPOINT` | Address browsers use for presigned downloads, if different from the internal one |
| `MINIO_SECURE`, `MINIO_CERT_CHECK` | TLS settings |
| `TERRA_PRESIGNED_EXPIRY_SECONDS` | Lifetime of download links (900 s by default) |
| `TERRA_AUTO_INITIALIZE_STORAGE` | Create collections at start-up; refused in a shared database |

The service refuses to start against a shared database or bucket without the approved prefix.

## Imagery and extraction

| Variable | Purpose |
| --- | --- |
| `CDSE_CLIENT_ID`, `CDSE_CLIENT_SECRET` | Copernicus Data Space credentials for on-demand latest-scene search (secrets); leave empty to disable |
| `TERRA_LATEST_MAX_AOI_CLOUD_PCT` | AOI-level cloud limit for the latest scene |
| `TERRA_LATEST_OBSERVATION_CACHE_SECONDS` | How long a repeated latest-observation request reuses the stored result without a catalogue call |
| `TERRA_VE_EXTRACTION_ALGORITHM` | Extraction algorithm (`peak_preserving_v3` by default) |
| `TERRA_VE_CLASSIFIER_SHA256` | Expected checksum of the vegetation classifier loaded from storage |

## Provenance

| Variable | Purpose |
| --- | --- |
| `TERRA_CODE_REVISION`, `TERRA_CONTAINER_DIGEST` | Identity of the extraction pipeline, part of every observation's signature |

These describe the pipeline that produced the stored history, not merely the running image. Keeping them equal to the history's values means new observations get the same signature as the history; changing them would treat the new pipeline as different and duplicate observations.

## Model, jobs and administration

| Variable | Purpose |
| --- | --- |
| `TERRA_MODEL_ADMIN_TOKEN` | Bearer token for `/models/*` promotion and rollback (secret) |
| `TERRA_JOB_STALE_AFTER_SECONDS` | When an unfinished job from a stopped process is marked failed |
| `TERRA_DT_ENABLE_SCHEDULER` | In-process scheduler; off in staging and production, where Airflow schedules work |
| `TERRA_MATERIALIZED_CACHE_DIR` | Local cache for verified datasets and model packages (disposable) |

## Service

| Variable | Purpose |
| --- | --- |
| `TERRA_HOST`, `TERRA_PORT` | Bind address and port (5000 by default) |

## Effect on users

- Without imagery credentials, the latest-observation step reports the provider as unavailable; forecasts from stored observations still work.
- Without storage, readiness reports `503` and storage-backed routes return diagnostics.
- An AOI without an active model is not listed for forecasting.
