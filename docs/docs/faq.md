# Frequently Asked Questions

## What is TERRA UGLA?

TERRA UGLA is a coastal digital twin module for vegetation-edge based coastline monitoring, forecasting, and retrospective validation.

## Who is the documentation for?

The documentation is for users, project partners, reviewers, and technical integrators who need to understand the module concepts, workflows, public API surface, and interpretation limits.

## What is an AOI?

An AOI is an Area of Interest. In TERRA UGLA, each prepared coastal AOI has a fixed reference line and transects so that observations can be compared across time.

## What is a transect?

A transect is a repeatable cross-shore measurement profile. TERRA UGLA uses transects to measure how far the vegetation edge is from the AOI reference line.

## Why does the module use fixed reference lines and transects?

Fixed geometry gives each AOI a stable measurement frame. This makes historical observations, current extraction, future prediction, and validation comparable across time.

## Why is vegetation edge the primary state variable?

Vegetation edge is generally more stable than the instantaneous waterline at monthly and yearly timescales, making it better suited for medium-term coastal-state monitoring and forecasting.

## What does the forecast show?

The forecast shows a likely future vegetation edge position and an uncertainty range. The uncertainty range helps users see where the future position is more or less certain.

## What does retrospective validation mean?

Retrospective validation reruns forecasts from historical dates and compares them with later observed vegetation edge positions. This helps users inspect how the forecast route behaved in past cases.

## Why might a validation case be missing?

A case may be missing because suitable imagery is unavailable, the scene did not pass quality checks, historical extraction has not been completed, or the selected AOI does not have enough same-month history.

## Why might the latest observation fail?

The latest-observation step may fail if recent satellite scenes are cloudy, contain too few valid pixels, are unavailable from the imagery provider, or do not produce a reliable vegetation edge under the AOI geometry.

## Are forecast outputs final decisions?

No. TERRA UGLA provides decision-support information. Forecasts and uncertainty bounds should be interpreted together with local knowledge, expert assessment, and other coastal evidence.

## Where is the OpenAPI description?

The public OpenAPI source is available as [openapi.yaml](openapi.yaml).
