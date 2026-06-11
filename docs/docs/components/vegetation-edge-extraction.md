# Coastal Vegetation Edge Indicator Extraction

This component extracts the coastal Vegetation Edge (VE) as the primary shoreline-state indicator used by the TERRA Coastal DT workflow.

## Component Role

The component converts accepted satellite observations into comparable coastal state measurements. It uses stable AOI-specific reference lines and transects so that current, historical, and predicted vegetation-edge positions can be compared in the same measurement frame.

The component is responsible for:

- maintaining or loading fixed AOI reference lines and transects;
- extracting vegetation-edge candidates from prepared imagery;
- selecting and regularising the vegetation-edge line;
- intersecting VE outputs with fixed transects;
- exporting map-ready geometry and distance-based observations;
- producing quality-control metrics for interpretation and downstream filtering.

## Position in the Product Chain

This component follows satellite image preparation and provides the core observed coastal indicator for the digital twin.

```text
prepared satellite observation -> fixed AOI geometry -> VE extraction -> transect-distance observations
```

## Why VE Is Used

Vegetation Edge is used as the primary indicator because it is more stable than instantaneous waterline position at monthly and yearly timescales. Waterline observations can be strongly affected by tide, waves, and short-term water-level conditions. VE is better suited for monitoring medium-term coastal retreat and supporting forecast comparison.

## Inputs

- prepared satellite observation;
- AOI fixed reference line;
- AOI fixed transects;
- remote-sensing indicators such as vegetation contrast;
- extraction and quality-control parameters.

## Outputs

- vegetation-edge GeoJSON;
- vegetation-edge points on transects;
- transect-distance measurements in metres;
- scene-level and transect-level quality metrics;
- time-series-ready CSV and GeoJSON artifacts.

## Implementation Scope

In the current implementation, fixed-geometry VE extraction is implemented through the vegetation-edge extraction services, fixed-geometry services, intersection logic, and run export pipeline.

Relevant service areas include:

- fixed geometry management;
- vegetation-edge extraction under fixed transects;
- transect intersection and distance calculation;
- quality-control diagnostics;
- map-ready artifact generation.

## Public Interfaces

The component contributes to the following public service areas:

- `GET /aoi/<aoi_id>/geometry/fixed`
- `POST /aoi/<aoi_id>/extract/latest`
- `POST /jobs/extract`
- `GET /results/<run_id>/download`
- `GET /runs/<run_id>/artifacts/<path>`

## Current Status

Fixed-geometry VE extraction has been implemented, including reference-line and transect handling, VE line generation, transect intersection, time-series export, quality diagnostics, and map-ready outputs.

## Planned Work

Planned work focuses on improving extraction stability across additional AOIs, refining fixed-geometry quality control, improving diagnostics for weak scenes, and strengthening validation of VE extraction outputs.
