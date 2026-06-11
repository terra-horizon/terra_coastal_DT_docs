# Coastal Change Time-Series Analysis and Forecasting

This component analyses historical Vegetation Edge observations and predicts future coastal change using transect-based time-series representation.

## Component Role

The component transforms repeated VE observations into AOI-level coastal change histories and future scenarios. It represents shoreline state as distance along fixed transects, which makes observations comparable across dates and suitable for forecasting.

The component is responsible for:

- rebuilding or loading AOI vegetation-edge time series;
- preparing transect-distance histories for forecasting;
- predicting future vegetation-edge position;
- producing uncertainty-aware forecast outputs;
- supporting retrospective validation against later observed targets.

## Position in the Product Chain

This component follows vegetation-edge extraction and provides the predictive part of the coastal monitoring workflow.

```text
VE transect observations -> historical context -> probabilistic forecast -> validation and reporting outputs
```

## Forecast Representation

The operational workflow forecasts VE movement as distances along fixed transects rather than direct image masks. This keeps the state representation compact, interpretable, and aligned with AOI geometry.

Forecast outputs can be reconstructed into map-ready layers such as:

- current VE line;
- forecast median line;
- lower and upper uncertainty bounds;
- uncertainty band;
- erosion or retreat indicators derived from landward movement and uncertainty.

## Inputs

- AOI fixed geometry;
- historical VE transect-distance records;
- latest observed VE state;
- scene and extraction quality covariates;
- forecast horizon.

## Outputs

- forecast VE distance table;
- predicted VE GeoJSON layers;
- uncertainty bounds;
- validation metrics where retrospective cases are run;
- diagnostic information for unavailable or weak historical context.

## Implementation Scope

In the current implementation, this capability is implemented through the transect dataset builder, transect forecaster, validation service, forecast packaging logic, and API routes for latest forecasting and retrospective validation.

Relevant service areas include:

- AOI time-series status;
- historical context reconstruction;
- transect-based forecasting;
- uncertainty output generation;
- retrospective validation and backfill diagnostics.

## Public Interfaces

The component contributes to the following public service areas:

- `GET /aoi/<aoi_id>/timeseries/status`
- `POST /aoi/<aoi_id>/history/context`
- `POST /aoi/<aoi_id>/forecast/latest`
- `POST /aoi/<aoi_id>/validation/history`
- `POST /aoi/<aoi_id>/validation/run`
- `POST /aoi/<aoi_id>/validation/backfill-plan`

## Current Status

Transect-based historical context reconstruction, probabilistic VE forecasting, uncertainty output generation, and same-month retrospective validation workflows have been implemented.

## Planned Work

Planned work focuses on extending validation across additional AOIs, improving forecast generalisation, refining uncertainty interpretation, and developing clearer erosion and vegetation-retreat indicators for reporting.
