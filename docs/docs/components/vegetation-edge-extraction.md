# Coastal Vegetation Edge Indicator Extraction

This component measures the coastal **vegetation edge** (VE), the seaward limit of dune vegetation, in every accepted scene. It expresses the edge as a distance along fixed transects, so that every date is measured in the same frame and can feed the forecasting model directly.

```text
accepted scene -> vegetation classification -> per-transect edge candidates -> temporal prior -> VE line + transect distances
```

## Why the vegetation edge

The waterline moves with tide, waves and surge from one image to the next, by tens of metres within hours. The vegetation edge responds to erosion and accretion over months and years, which is the timescale coastal management cares about. It is visible in multispectral imagery and is a recognised indicator of dune coast change.

## Fixed geometry

Each AOI has one versioned **reference line** and a set of **transects** perpendicular to it, spaced 15 m apart.

- **Reference line from a temporal composite.** The line is traced on a temporal median composite of many clear scenes, not on a single date or a trained segmentation model. It sits at the long-term mean vegetation edge, works on coasts no model has seen, and gives every AOI a comparable alongshore grid.
- **Sign convention.** Transects are oriented so that distances are positive seaward and negative landward. An orientation audit checks this for every AOI.
- **Transect length** is set per AOI with a margin of about 100 m beyond the largest observed gap between the vegetation edge and the reference line.
- **Site-specific refinements** are recorded as new geometry versions, for example ending a line where a beach ends, or routing it round a hook so transects cross the edge instead of running along it. The measured spread on that hook fell from 52 m to 12 m.
- **Convex corners** such as spit tips, where transects fan out, get a corner allowance, so the apex is measured rather than smoothed away.

## Extraction method

1. **Vegetation classification.** A neural-network pixel classifier, ported from the COASTGUARD/VedgeSat toolkit, labels vegetation and non-vegetation from the five Sentinel-2 bands and spectral indices.
2. **Scene threshold.** A weighted-peaks threshold, placed at 0.2 x the lower mode + 0.8 x the upper mode of the non-vegetation and vegetation index distributions, adapts to each scene's illumination and phenology.
3. **Per-transect candidates.** Along each transect, candidate edge positions come from the threshold crossing, the traced contour and the index gradient, within a corridor around the reference line.
4. **Peak-preserving selection** (`peak_preserving_v3`, the default algorithm). Candidates are chosen jointly along the coast with a smoothness penalty, while preserving well-supported headlands and spits instead of blurring them.
5. **Temporal prior.** Each scene's result is compared with a leave-one-out rolling median of its nearest-in-time neighbouring scenes (8 neighbours, 2 passes). Because the prior follows the record, it tracks genuine movement and seasonality while overruling a run that is wrong in one scene only. This roughly halves extraction noise.
6. **Intersection and export.** The selected edge is intersected with every transect to give signed distances in metres, plus a map-ready VE line.

## Quality control

- **Per-scene metrics:** share of transects with an accepted edge, and the source of each value (threshold, contour or gradient).
- **Reach checks:** an edge farther from the reference line than the configured limit is not accepted.
- **Signal gate.** Before an AOI is used for forecasting, its record is tested for a forecastable signal: the variance of the true coastal state is estimated by subtracting the noise variance implied by month-to-month differences. AOIs with no recoverable signal are not forecast. Two candidate sites failed the gate as genuinely stable coasts.
- **Provenance.** Every observation carries a pipeline signature (code revision, container image, extraction algorithm and classifier checksum). Live extractions reuse the parameters pinned on the AOI's dataset version, so new observations stay comparable with the history the model was trained on.

## Outputs

- VE line (GeoJSON) and VE points on transects;
- per-transect signed distances in metres, stored as queryable measurements;
- scene-level and transect-level quality metrics;
- monthly composites for forecasting: the median of the month's accepted scenes, with alongshore outliers rejected.

## Public interfaces

- `GET /aoi/<aoi_id>/geometry/fixed`
- `POST /aoi/<aoi_id>/extract/latest`
- `POST /jobs/extract`
- `GET /aois/<aoi_id>/observations`
- `POST /observations/<observation_id>/review`

## Current status

Implemented and in operation. Histories from 2016 to 2026 have been extracted for 17 AOIs on the current pipeline; 15 pass the signal gate and are live in the forecast service. Two older AOIs from an earlier pipeline are retired.

## Planned work

- validate extraction on coastal types beyond Scottish dune coasts.
