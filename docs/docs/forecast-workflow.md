# Forecast Workflow

The primary user workflow is AOI-first. A user selects a prepared coastal AOI, extracts the latest usable vegetation edge observation, reviews the historical context, and generates a future forecast.

## Step 1: Select AOI

The user selects a prepared AOI from the available list. The selected AOI determines the fixed reference line, transects, historical observations, and forecast context used by the workflow.

## Step 2: Latest Observation and Current VE

The service searches for the newest usable satellite observation, applies quality checks, extracts the current vegetation edge, and displays the accepted observation on the map.

## Step 3: Historical Context

The module loads or rebuilds the relevant AOI history. Historical vegetation edge positions are expressed under the same fixed geometry so that they can be compared with the current observation.

## Step 4: Forecast

The forecast model predicts future vegetation edge distances along the fixed transects. The service then reconstructs those distances into map-ready layers:

- current vegetation edge;
- forecast median line;
- uncertainty bounds;
- uncertainty band;
- optional areas indicating possible landward exposure where the uncertainty range suggests erosion risk.

## Why Fixed Geometry Matters

The workflow fixes the measurement coordinate system before extraction and forecasting. This makes current state, historical state, and predicted future state comparable within the selected AOI.
