# Data & State Model

TERRA UGLA uses fixed AOI geometry and per-run artifacts to keep coastal state comparable across time.

## AOI Geometry

Each AOI has one canonical geometry set. The key artifacts are:

```text
data/aoi/<aoi_id>/refline.geojson
data/aoi/<aoi_id>/refline_metadata.json
data/aoi/<aoi_id>/transects.geojson
```

The reference line defines the measurement baseline, and transects define the cross-shore profiles used to convert vegetation edge position into a distance time series.

## Runtime Runs

Runtime extraction and forecast outputs are grouped by run identifier:

```text
data/runs/<run_id>/
```

Typical run outputs include imagery, previews, transects, intersections, vegetation-edge exports, waterline exports, selection traces, run summaries, manifests, and forecast outputs.

## Fixed-Geometry VE History

The fixed-geometry extraction history is the bridge between imagery and forecasting. It stores scene-level vegetation edge observations in a representation that can be compared across years and AOIs.

## Digital Twin State

AOI-level digital twin state is maintained separately from single-run output. It supports bootstrap, assimilation, and prediction paths and can be updated as new observations become available.

## Public Repository Boundary

This documentation repository should not contain private data, credentials, model checkpoints, generated imagery, or implementation code. Those assets belong in the private implementation repository or secured operational storage.
