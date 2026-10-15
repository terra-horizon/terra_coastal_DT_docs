# Deployment

TERRA Coastal DT is delivered as a Docker image and runs as one container next to MongoDB and MinIO. This page describes what a deployment needs and how the pieces fit; environment-specific values belong in the deployment's private configuration, not here.

## Service image

| | |
| --- | --- |
| Registry | GitHub Container Registry, published from version tags of the source repository |
| Base | Python 3.10 (slim), CPU build of PyTorch |
| Process | gunicorn, 1 worker with 8 threads, 600 s request timeout |
| Port | 5000 |
| User | Non-root |
| Contents | Code, web interface and AOI registry only; no data, models or credentials |

Release images should be built from a commit (for example from `git archive`), so file permissions inside the image are normalised and the image matches a recorded code revision.

## Requirements

- **MongoDB** (replica set) with an account that has read-write access to the UC3 collections.
- **MinIO** (or S3-compatible storage) with a bucket, or a prefix of a shared bucket, for UC3 objects.
- **Copernicus Data Space Ecosystem** credentials, if the service should fetch the latest scenes on demand.
- **Published data:** AOIs, geometry versions, observation history, datasets, the active MambaLSTM model package and the vegetation classifier, loaded into the stores before the service starts.
- **A reverse proxy or gateway** that authenticates users and routes to the service. The web interface uses absolute paths, so it needs its own host name or port rather than a sub-path, and long extraction requests need a read timeout of about 600 s.

## Deployment steps

1. **Initialise storage.** Run the initialisation tool in plan mode (read-only), then apply it. It creates missing collections, validators and indexes, skips what already matches, and stops without writing if an existing collection differs. In a shared database it touches only the `uc3_`-prefixed collections.
2. **Publish domain data.** Load AOIs, geometry, history, datasets and the model package into MongoDB and MinIO. Publishing is idempotent.
3. **Configure the container** through environment variables and secret files (see [Configuration](configuration.md)), including the provenance values of the extraction pipeline the history was built with.
4. **Start the container** and check `GET /health/ready`. It reports `200` only when MongoDB, the UC3 collections and MinIO are reachable.
5. **Smoke test:** list prepared AOIs, load fixed geometry, run a forecast and download an artifact.

## Deployment patterns

| Pattern | Use |
| --- | --- |
| Local TERRA node | Development and demonstration against a local MongoDB and MinIO in Docker |
| Platform staging | Integration with the TERRA Airflow and ingestion services |
| Shared platform storage | Production next to the TERRA shared data layer: UC3 collections under the `uc3_` prefix and objects under their own bucket prefix, isolated from other use cases |

The service's in-process scheduler stays disabled in staging and production; Airflow is the control plane.

## Moving between stores

A migration tool moves an existing UC3 store into a shared one: it inventories source and target, dumps the source database, restores it under the new collection prefix, copies objects, rewrites object references and verifies counts and checksums. Pipeline signatures are preserved, so migrated history stays consistent with new observations.

## Documentation site

This documentation is built with MkDocs Material and versioned with mike. The on-demand **Deploy Docs** workflow in the documentation repository publishes a version and moves the `latest` alias:

```text
https://terra-horizon.github.io/terra_coastal_DT_docs/latest/
```
