# Technical Approach

This page explains the main technical ideas behind TERRA Coastal DT in public-facing terms. It focuses on why the module uses fixed AOI geometry, vegetation edge state extraction, transect-distance forecasting, and retrospective validation.

## Coastal State as a Digital Twin Variable

TERRA Coastal DT treats the coastline as an evolving state rather than as a single line extracted from one image. For each AOI, the module aims to connect:

- the latest observed vegetation edge;
- historical vegetation edge movement;
- a stable measurement frame;
- future forecast scenarios;
- uncertainty information that can be shown on a map.

This lets the wider digital twin reason about current coastal condition, past change, and possible future movement in one workflow.

## Vegetation Edge as the Primary Indicator

The vegetation edge is used as the primary shoreline-state indicator because it is generally more stable than the instantaneous waterline at monthly and yearly timescales. Waterline position can change quickly with tide, waves, and short-term water level. Vegetation edge is better suited to medium-term monitoring and forecast comparison.

Waterline and other layers may still be useful as supporting information, but the main state variable in this workflow is vegetation edge position.

## Fixed Reference Line and Transects

A central design choice is to give each AOI a stable reference line and a stable set of transects. The reference line acts as the baseline, while transects act as repeatable cross-shore measurement profiles.

This matters because observations from different dates must be measured in the same spatial frame. If every observation used a newly generated baseline, the system would mix real shoreline change with changes in the measurement frame. Fixed geometry avoids that problem and makes historical comparison, forecasting, and validation easier to interpret.

## Current-State Extraction

For a selected AOI, the module searches for a usable satellite observation and extracts the current vegetation edge under the fixed geometry. The extraction process combines:

- scene-quality checks;
- remote-sensing indicators such as vegetation contrast;
- candidate vegetation edge generation;
- geometric consistency checks against the reference line and transects;
- quality metrics that help identify unreliable cases.

The result is a map-ready vegetation edge line and a set of transect-distance observations.

## Forecast Representation

Instead of forecasting a future image mask directly, the operational workflow forecasts vegetation edge movement as distances along the fixed transects.

This representation has several advantages:

- each value has a clear physical meaning in meters;
- the same AOI geometry is used for history, current state, and future state;
- missing or low-quality transects can be inspected directly;
- uncertainty can be expressed as lower, median, and upper forecast lines;
- forecast results can be reconstructed back into map geometry.

## Uncertainty

Forecast outputs are intended to show both a central estimate and an uncertainty range. In the map workflow, this can appear as a median forecast line and surrounding bounds. The uncertainty band helps users interpret where future vegetation edge position is more or less certain.

## Retrospective Validation

Retrospective validation compares historical cutoff forecasts against a later observed target state. This is useful because it turns forecast quality into something visible and testable: users can compare map overlays and summary errors for different historical cases.

Same-month validation is preferred where possible because it reduces seasonal mismatch.

## Current Limitations

The module should be interpreted as decision-support technology, not as a replacement for expert coastal assessment. Important limitations include:

- forecast quality depends on imagery availability and scene quality;
- complex coastlines may require additional review of fixed geometry;
- vegetation edge behavior can differ across AOIs and geomorphological settings;
- uncertainty bounds are informative but do not capture every possible future driver.

## Summary

The main technical idea behind TERRA Coastal DT is to turn repeated satellite observations into a stable, comparable, and forecastable coastal state record. Fixed geometry provides the measurement frame, vegetation edge extraction provides the observed state, transect-distance forecasting provides future scenarios, and retrospective validation helps users inspect forecast credibility.
