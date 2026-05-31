# Service Architecture

TERRA UGLA is organized as a coastal digital twin module rather than a single image-processing script. Its purpose is to maintain a repeatable state representation for each coastal AOI and expose current-state extraction, historical context, future forecasting, and validation workflows.

## Core Components

### AOI and Fixed Geometry

Each prepared AOI has a stable measurement frame:

- a fixed reference line;
- a fixed set of transects;
- metadata describing the source scene, method, model checkpoint, and metric CRS.

The fixed geometry is the common reference system used by extraction, historical reconstruction, forecasting, and validation.

### Observation Ingestion

The runtime workflow searches recent candidate satellite scenes, applies scene-quality checks, downloads usable imagery where credentials are available, and prepares preview and raster assets for the selected AOI.

### Vegetation Edge Extraction

Vegetation edge extraction combines remote-sensing features, fixed-geometry constraints, candidate generation, and quality scoring. The output is a map-ready VE geometry plus per-transect distance observations.

### Forecasting

The operational forecasting route predicts future vegetation edge position as transect distances against the fixed AOI geometry. Forecast distances are reconstructed into p50 and uncertainty-bound geometries for map display.

### Retrospective Validation

The validation workflow treats the latest accepted observation as current ground truth, builds same-month historical cutoff cases, reruns forecasts from historical dates, and compares predicted state against the target observation.

### Digital Twin State

The backend can maintain AOI-level state that supports bootstrap, assimilation cycles, prediction, and future retraining workflows. This state layer is what makes the module a persistent digital twin capability instead of a one-off processing endpoint.

## Main Interfaces

The module exposes:

- human-facing web pages for forecast, validation, and geometry preparation workflows;
- AOI and fixed-geometry endpoints;
- latest-observation extraction and forecast endpoints;
- validation history, run, and backfill planning endpoints;
- digital twin bootstrap and prediction endpoints;
- job and result-artifact endpoints.
