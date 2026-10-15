# AI-enabled Satellite Image Processing

This component turns an AOI request into screened, analysis-ready Sentinel-2 imagery for vegetation-edge extraction. It is the first runtime step of the product chain.

```text
AOI -> catalogue search -> AOI-level cloud screening -> 5-band download -> preflight checks -> scene asset
```

## Component role

- search the Copernicus Data Space Ecosystem (CDSE) catalogue for Sentinel-2 L2A scenes over an AOI;
- screen candidate scenes for cloud **over the AOI itself**, not just over the whole satellite tile;
- download the bands the vegetation classifier needs, clipped to the AOI;
- check every raster structurally and for coastal usability before it is used;
- record accepted and rejected scenes, with reasons, so every observation is traceable.

## Data source

| | |
| --- | --- |
| Mission | Sentinel-2, Level-2A surface reflectance |
| Provider | Copernicus Data Space Ecosystem (Sentinel Hub catalogue and processing APIs) |
| Bands | B02 (blue), B03 (green), B04 (red), B08 (near infrared), B11 (short-wave infrared), delivered as one 5-band reflectance raster |
| Screening layer | Scene Classification Layer (SCL) |

## AOI-level cloud screening

A Sentinel-2 tile covers about 110 x 110 km, while an AOI covers around 18 km² of coast, about 0.15% of the tile. The tile's cloud figure is therefore a weak guide: the same acquisition has been reported as 54% cloudy on one tile and 24% on its neighbour over identical AOI pixels. The component therefore uses the tile's cloud percentage only as a loose pre-filter, then requests a low-resolution (60 m) probe of the SCL layer over the AOI and computes the share of unusable pixels inside it: medium and high cloud, cloud shadow, thin cirrus, no data, saturated pixels, and snow or ice. Only scenes under the AOI-level limit are downloaded at full resolution, so rejected scenes cost one small request.

## Three acquisition paths

=== "Historical archive"

    Monthly pulls build each AOI's history from 2016 onwards: tile pre-filter at 80% cloud, AOI-level limit of 20%, and up to three scenes per month. Failures raise errors instead of falling back to synthetic data.

=== "Latest observation (on demand)"

    When a user asks for the latest state, the service searches a recent window (120 days by default), probes the newest candidates with AOI-level screening, and accepts the newest scene that passes screening and extraction. A deployment can tighten the AOI cloud limit (10% in operation). Repeated requests within a cache window reuse the accepted observation without another catalogue call.

=== "Scheduled ingestion (TERRA platform)"

    In the platform deployment, the TERRA ingestion service runs the UC3 acquisition profile `coastal-uc3-sentinel2` on a schedule, commits accepted scenes to shared storage, and asks the digital twin to assimilate them. See [Data Ingestion](../data-ingestion.md).

## Preflight checks

Before a raster is used, it must pass:

- **structural checks:** expected band count, data type, georeferencing and AOI coverage;
- **coastal checks:** enough valid pixels, and enough vegetation (NDVI) contrast for an edge to be measurable;
- **duplicate checks:** a scene acquired at the same time as one already observed is not processed twice (matched by acquisition time within 3 minutes, because one satellite pass can appear under two tile identifiers).

Rejected scenes are recorded with their reasons, so a missing month can be explained.

## Outputs

- accepted scene metadata and quality diagnostics;
- 5-band AOI rasters stored as immutable objects;
- preview images for the map;
- scene-evaluation records for accepted and rejected candidates.

## Public interfaces

- `GET /check_sentinel_hub_status`
- `GET /aoi/prepared`
- `POST /aoi/<aoi_id>/extract/latest`
- `POST /jobs/assimilate`
- `GET /artifacts/<artifact_id>`

## Current status

Implemented and in operation: catalogue search, AOI-level SCL screening, 5-band download, preflight checks, scene caching and duplicate detection, and the ingestion profile for the TERRA platform. The full 2016 to 2026 archive has been processed for all registry AOIs.

## Planned work

- run the ingestion profile under the platform scheduler for daily updates;
- add exogenous data (waves, surge, water level) alongside imagery as inputs for forecasting.
