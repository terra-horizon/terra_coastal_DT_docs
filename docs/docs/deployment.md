# Deployment

This page describes deployment concepts for users and integrators. It does not prescribe one specific operational environment.

## Documentation Site

The public documentation is published through GitHub Pages:

```text
https://terra-uglateam.github.io/terra_coast_DT_docs/
```

The site is built from the MkDocs source in this repository.

## Service Deployment Context

TERRA UGLA is expected to run as part of a wider platform or project environment where satellite imagery access, AOI data, processing resources, storage, and user-facing interfaces are available.

A typical deployment needs:

- access to required satellite imagery services or prepared imagery archives;
- storage for AOI geometry, run outputs, validation outputs, and digital twin state;
- compute resources suitable for geospatial processing and forecast inference;
- configuration for environment-specific paths and service endpoints;
- monitoring of health, job status, and data availability.

## Publication and Access

Public users generally interact with TERRA UGLA through the deployed documentation, maps, API endpoints, or project demonstrations. Operational deployment details such as credentials, private datasets, and infrastructure-specific secrets are managed outside the public documentation site.
