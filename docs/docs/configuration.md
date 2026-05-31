# Configuration

This page documents configuration concepts without exposing private runtime values.

## Runtime Configuration

The implementation service uses environment-specific configuration for host, port, debug mode, imagery access, data paths, and optional model locations. Public documentation should describe the configuration categories, but real credentials and operational secrets must remain outside this repository.

## Credentials

Sentinel Hub, CDSE, cloud storage, and deployment credentials must not be committed to the public documentation repository.

## Example Files

Configuration examples can be documented in prose or with redacted snippets. Full runtime templates belong in the private implementation repository unless they are explicitly reviewed for public release.

## Documentation Build

Local documentation preview uses:

```bash
cd docs
mkdocs serve
```

Versioned publication uses the on-demand GitHub Actions workflow in `.github/workflows/deploy-docs-on-demand.yml`.
