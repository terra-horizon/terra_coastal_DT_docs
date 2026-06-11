# AI-enabled Satellite Image Processing

This component prepares satellite observations for the coastal digital twin workflow. It turns an AOI request into usable remote-sensing inputs for downstream coastal indicator extraction.

## Component Role

The component is responsible for:

- searching and selecting satellite scenes for a selected AOI;
- checking scene usability based on cloud, valid-pixel, and data-quality conditions;
- preparing raster and preview products for processing and map display;
- exposing observation metadata and diagnostics to the rest of the workflow.

It provides the observation layer used by vegetation-edge extraction, time-series construction, validation, and forecast workflows.

## Position in the Product Chain

This is the first runtime step of the Digital Twin for Monitoring Coastal Change from Satellite Remote Sensing product chain.

```text
AOI selection -> satellite scene search -> quality screening -> imagery preparation -> coastal indicator extraction
```

The component does not describe model training or data augmentation workflows. Those activities are part of the supporting model lifecycle and remain outside the runtime product-chain execution.

## Inputs

- AOI identifier and geometry;
- configured imagery provider access or prepared imagery archives;
- scene search window and quality thresholds;
- deployment-specific storage paths.

## Outputs

- accepted scene metadata;
- prepared raster artifacts;
- preview images for map display;
- quality diagnostics;
- references to imagery artifacts used by downstream components.

## Implementation Scope

In the current implementation, this capability is implemented through the AOI runtime extraction services, imagery access utilities, quality checks, run artifact storage, and API endpoints that support latest-observation extraction.

Relevant service areas include:

- AOI-first runtime extraction;
- satellite imagery search and loading;
- scene quality screening;
- preview and artifact generation.

## Public Interfaces

The component contributes to the following public service areas:

- `GET /check_sentinel_hub_status`
- `GET /aoi/prepared`
- `POST /aoi/<aoi_id>/extract/latest`
- `GET /results/<run_id>/summary`
- `GET /runs/<run_id>/artifacts/<path>`

## Current Status

Satellite imagery access, AOI-based scene selection, quality screening, preview generation, and runtime image preparation have been implemented and connected to the coastal monitoring workflow.

## Planned Work

Planned work focuses on improving scene-selection robustness, cloud and quality filtering, diagnostics for missing or unsuitable imagery, and integration with platform-level workflow execution.
