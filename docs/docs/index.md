# TERRA Coastal DT

TERRA Coastal DT is a coastal digital twin module for vegetation-edge based coastline monitoring, forecasting, and retrospective validation. It is part of the wider [TERRA](https://terra-horizon.eu/) platform and contributes coastal-state perception and forecasting capability to the platform digital twin.

The module helps users work with coastal Areas of Interest (AOIs) by turning satellite observations into comparable coastal state information. It combines fixed AOI geometry, vegetation edge extraction, transect-based state history, probabilistic forecasting, and validation workflows.

The main documentation entry points are:

- [AI-enabled Satellite Image Processing](components/satellite-image-processing.md), for satellite scene search, screening, preparation, and observation artifacts.
- [Coastal Vegetation Edge Indicator Extraction](components/vegetation-edge-extraction.md), for fixed-geometry VE extraction and transect-distance observations.
- [Coastal Change Time-Series Analysis and Forecasting](components/time-series-forecasting.md), for historical context, probabilistic forecasting, and retrospective validation.
- [Intelligent Coastal Digital Twin Orchestration](components/digital-twin-orchestration.md), for the runtime workflow coordinator, state management, jobs, APIs, and user-facing interfaces.
- [Architecture](architecture.md), for the module structure and responsibilities.
- [Data & State Model](data-state.md), for AOI geometry, run outputs, and digital twin state concepts.
- [Forecast Workflow](forecast-workflow.md), for the AOI-first user workflow.
- [Retrospective Validation](validation.md), for historical forecast testing.
- [Technical Approach](technical-route.md), for the main technical ideas behind fixed geometry, vegetation edge extraction, and transect forecasting.

The documentation focuses on public concepts, user workflows, service interfaces, and integration-level information.
