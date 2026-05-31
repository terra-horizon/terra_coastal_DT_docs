# Configuration

TERRA Coastal DT relies on environment-specific configuration so that the same module can run in different deployment contexts.

## Main Configuration Areas

A deployment normally needs configuration for:

- service host, port, and runtime mode;
- imagery provider access or local imagery archives;
- AOI data locations;
- run output and artifact storage;
- model or forecast resource locations;
- job processing behavior;
- public URLs used by maps, APIs, and documentation.

## Credentials and Access

Some deployments require credentials for imagery services, storage, or platform integration. These values should be managed through the deployment environment and should not be exposed through public documentation or public client-side code.

## User-Facing Impact

Configuration affects what a user can do in the interface. For example:

- if imagery access is unavailable, latest-observation extraction may be limited;
- if historical AOI data is incomplete, validation cases may be unavailable;
- if forecast resources are not configured, forecast endpoints may return a diagnostic response rather than a prediction.

## Health Checks

Public health and status endpoints can help operators and integrators confirm whether the service is running and whether upstream imagery access is available.
