# Forecast Workflow

The primary operator workflow is AOI-first and follows a fixed sequence from AOI selection to forecast visualization.

## Step 1: Select AOI

The operator selects one prepared AOI. The runtime path should preserve that selected AOI and should not remap the request to another AOI.

## Step 2: Latest Observation and Current VE

The service searches for the newest usable scene, runs scene-quality checks, extracts the current vegetation edge, and renders accepted scene preview layers on the map.

## Step 3: Historical Context

The system rebuilds or loads the AOI transect history used by the forecaster. Historical context is restored strategically rather than blindly loading every past scene for every request.

## Step 4: Transect Forecast

The forecast model predicts future vegetation edge distances along fixed transects. The service reconstructs these distances into map-ready layers:

- current vegetation edge;
- forecast p50 line;
- uncertainty band;
- p10 and p90 uncertainty bounds;
- derived erosion-risk hint areas where the landward uncertainty side indicates possible exposure.

## Design Rationale

The workflow fixes the measurement coordinate system before extraction and forecasting. This makes the current state, historical state, and predicted future state comparable under one AOI-specific reference frame.
