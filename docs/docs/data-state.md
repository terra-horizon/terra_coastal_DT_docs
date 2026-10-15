# Data & State Model

TERRA Coastal DT keeps the state of each coastal site as versioned records in MongoDB and immutable objects in MinIO. Every observation is expressed in the AOI's fixed measurement frame, so current, historical and forecast states are directly comparable.

## Measurement frame

- **AOI:** a registered coastal box with a name, coastal group, coordinate reference system and lifecycle status (active or retired).
- **Geometry version:** one reference line and its transects (15 m spacing). Changes create a new version rather than editing the old one.
- **Distance:** a vegetation-edge position is the signed distance in metres along a transect from the reference line, **positive seaward**.

## State records (MongoDB)

| Collection | Holds |
| --- | --- |
| `aois` | AOI geometry, bounding box, CRS and lifecycle status |
| `geometry_versions` | Versioned reference line and transect packages |
| `transects` | Per-transect documents |
| `scenes` | Satellite scene metadata and catalogue status |
| `scene_assets` | AOI-specific scene assets (the 5-band rasters) committed by acquisition |
| `scene_evaluations` | Accepted and rejected candidate scenes, with reasons |
| `observations` | Historical and live vegetation-edge observations, with pipeline signature and review status |
| `transect_measurements` | Per-transect VE distances of each observation (queryable) |
| `digital_twin_states` | Latest-state snapshots per AOI |
| `forecasts` | Forecast requests, signatures, provenance and output references |
| `forecast_measurements` | Per-transect p10 / p50 / p90 distances and erosion-risk flags |
| `dataset_versions` | Monthly dataset snapshots used for forecasting and training |
| `model_versions` | Model packages, lifecycle status (candidate, active, superseded) and evaluation |
| `training_runs` | Candidate training runs and their evaluation |
| `runs`, `run_events` | Job state and ordered phase events |
| `artifacts` | Registry of every MinIO object: checksum, state and lineage |
| `migration_ledger` | Record of storage migrations |

Common fields (`project_id`, `use_case_id`, `schema_version`, `created_at`, `updated_at`) scope and version every record. Collections have JSON-schema validators and unique indexes, created by an idempotent initialisation step.

## Objects (MinIO)

| Object family | Examples |
| --- | --- |
| Scene inputs | 5-band AOI rasters and previews |
| Run outputs | VE lines and points, intersections, time-series tables, forecast GeoJSON |
| Datasets | Monthly patch series per AOI |
| Models | MambaLSTM packages (weights, gates, calibration, validation report) |
| Classifier | The vegetation classifier, verified by checksum |

Objects are immutable: a changed product is a new object, registered in `artifacts` with its checksum. Users receive short-lived presigned links for downloads.

## Versioning and provenance

| What | How it is versioned |
| --- | --- |
| Geometry | New geometry version for any change; observations reference the version they were measured on |
| Observations | Pipeline signature: code revision, container image, extraction algorithm and classifier checksum. An identical request reuses the existing observation |
| Datasets | New dataset version per rebuild; extraction parameters are pinned on the version |
| Models | New model version per package; promotion and rollback switch the active version together with its dataset |
| Forecasts | Signature of observation, dataset and model; an identical forecast request reuses the stored result |

This makes every forecast reproducible: it can be traced to the exact observation, dataset, model and code that produced it.

## Public data boundary

This documentation describes data structures and products. Credentials, private infrastructure details and unpublished datasets are not part of the public site.
