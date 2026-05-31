# Deployment

This public repository is intended to publish documentation, not the private implementation service.

## Documentation Deployment

Documentation is built with MkDocs Material and follows the same pattern as the TERRA AAI documentation repository. The on-demand GitHub Actions workflow publishes versioned documentation through `mike` to the `gh-pages` branch.

The workflow expects the MkDocs project to live in:

```text
docs/
```

## Implementation Deployment

The implementation service should be deployed from the separate private code repository. That repository is responsible for runtime packaging, service images, operational secrets, model checkpoints, and environment-specific deployment instructions.

## Repository Boundary

Do not publish the following in this documentation repository:

- credentials or `config.json`;
- generated imagery, rasters, AOI private data, or model checkpoints;
- service source code;
- local test caches, bytecode, or private validation outputs.
