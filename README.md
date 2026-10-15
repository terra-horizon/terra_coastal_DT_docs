# TERRA Coastal DT

This repository hosts the public documentation for the TERRA Coastal DT module.

TERRA Coastal DT is the coastal digital twin of the TERRA platform (use case 3). It measures the coastal vegetation edge of Scottish dune coasts from Sentinel-2 imagery on fixed transects, keeps a monthly state history for each site, and forecasts the edge 1 to 24 months ahead with calibrated uncertainty using a hybrid forecaster whose core AI model is MambaLSTM. The documentation is intended for users, reviewers, project partners and technical integrators.

Documentation site:

https://terra-horizon.github.io/terra_coastal_DT_docs/

## Local Documentation Preview

```bash
cd docs
pip install mkdocs mkdocs-material mike neoteroi-mkdocs pymdown-extensions
mkdocs serve
```

The documentation is built with MkDocs Material, versioned with mike, and published to GitHub Pages by the on-demand **Deploy Docs** workflow.
