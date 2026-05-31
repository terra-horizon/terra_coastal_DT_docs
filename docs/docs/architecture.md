# Service Architecture

TERRA UGLA is organized as a coastal digital twin module. Its purpose is to maintain a repeatable coastal state representation for each AOI and expose workflows for current-state extraction, historical context, forecasting, and validation.

## Core Components

### AOI and Fixed Geometry

Each prepared AOI has a stable measurement frame made of a reference line and a set of transects. This fixed geometry is the common reference system used to compare observations across dates and to reconstruct forecast results back onto the map.

### Observation Ingestion

The module searches candidate satellite scenes for the selected AOI, applies quality checks, and prepares the accepted observation for vegetation edge extraction. Scene availability depends on imagery access, cloud coverage, and data quality.

### Vegetation Edge Extraction

Vegetation edge extraction converts an accepted satellite observation into a coastal state line. The process combines remote-sensing indicators, geometry constraints, candidate selection, and quality checks. The result is a map-ready vegetation edge together with transect-distance observations.

### Forecasting

The forecast workflow predicts future vegetation edge position as distances along the AOI transects. These predicted distances are reconstructed into map-ready forecast lines and uncertainty bounds.

### Retrospective Validation

The validation workflow compares historical cutoff forecasts against a later observed target state. This helps users understand whether recent forecasts are behaving consistently and how forecast error changes across different lookback periods.

### Digital Twin State

AOI-level state can be updated as new observations become available. This state layer lets the module connect current observations, historical memory, future scenarios, and validation evidence.

## Interfaces

The module exposes user-facing workflows and API endpoints for AOI discovery, fixed-geometry retrieval, latest-observation extraction, forecasting, validation, digital twin prediction, job status, and result access.
