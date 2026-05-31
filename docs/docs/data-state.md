# Data & State Model

TERRA UGLA keeps coastal observations comparable by expressing them in an AOI-specific measurement frame.

## AOI Geometry

Each AOI uses one stable reference line and one stable set of transects. The reference line defines the baseline, and transects define the cross-shore profiles used to measure vegetation edge position.

This fixed geometry lets the module compare current observations, historical observations, and forecast results in the same spatial frame.

## Observations and Runs

An accepted observation can produce several map and data products:

- scene preview layers;
- extracted vegetation edge geometry;
- optional waterline or auxiliary layers;
- transect intersections and distance measurements;
- run summary and quality information;
- forecast geometries and uncertainty bounds.

These outputs support both the interactive map workflow and later audit or validation tasks.

## Vegetation Edge History

The vegetation edge history stores repeated observations under the same AOI geometry. This history is the input for forecasting and for retrospective validation.

## Digital Twin State

Digital twin state represents the evolving coastal condition for an AOI. It can be updated as new observations become available and can support future prediction, state review, and operational decision support.

## Public Data Boundary

Public documentation describes the data model and outputs at a conceptual level. Sensitive credentials, private datasets, unpublished imagery, and model artifacts are not part of the public documentation site.
