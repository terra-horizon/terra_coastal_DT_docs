# Intelligent Coastal Digital Twin Orchestration

This component coordinates the coastal monitoring product chain as an operational digital twin workflow.

## Component Role

The component connects satellite observation processing, coastal indicator extraction, time-series analysis, forecasting, validation, state management, and user-facing visualisation. It provides the orchestration layer that makes the individual processing capabilities behave as one coastal digital twin workflow.

The component is responsible for:

- maintaining AOI-specific coastal digital twin state;
- coordinating the main forecast workflow;
- coordinating retrospective validation;
- exposing API endpoints and web interfaces;
- managing asynchronous extraction, prediction, and assimilation jobs;
- returning map-ready outputs and result artifacts to users and platform services.

## Position in the Product Chain

This component is the runtime coordinator for the full Coastal DT product chain.

```text
AOI state -> observation update -> VE extraction -> history update -> forecast -> validation/reporting output
```

It is distinct from training-oriented digital twin activities. Model training, retraining, and data augmentation may support future model lifecycle improvements, but they are not treated as core runtime steps of this product chain.

## Inputs

- AOI selection and state;
- prepared imagery and extraction results;
- historical VE time series;
- forecast and validation requests;
- workflow and job configuration.

## Outputs

- workflow status and diagnostics;
- current coastal state;
- historical context summaries;
- forecast and uncertainty layers;
- validation results;
- downloadable run artifacts;
- API responses for platform integration.

## Implementation Scope

In the current implementation, the orchestration layer is implemented through the Flask application, web templates, static front-end logic, digital twin services, job services, and result retrieval routes.

Relevant service areas include:

- main AOI-first web workflow;
- validation web workflow;
- digital twin bootstrap and prediction;
- asynchronous job handling;
- map-layer and artifact delivery;
- deployment and health endpoints.

## Public Interfaces

The component contributes to the following public service areas:

- `GET /`
- `GET /validation`
- `GET /dt/bootstrap`
- `POST /dt/predict`
- `POST /jobs/extract`
- `POST /jobs/predict`
- `POST /jobs/assimilate`
- `GET /jobs/<job_id>`
- `GET /healthz`

## Current Status

The web workflow, digital twin bootstrap and prediction endpoints, asynchronous job handling, result retrieval, and map-based interfaces for forecast and validation have been implemented.

## Planned Work

Planned work focuses on improving automated assimilation, report generation, platform integration, operational deployment, and clearer links between Coastal DT outputs and wider TERRA product-chain execution.
