# Maintenance

## Keeping the digital twin current

New observations need no manual step in a platform deployment: the daily workflow acquires new scenes and assimilates them, and the next forecast refits the statistical base on the updated record. The MambaLSTM weights do not need to change for new months to count.

## Adding an AOI

`config/aois.json` in the source repository is the single source of truth for AOI extents, names, coastal groups and status. To add a site:

1. **Register** the AOI with its bounding box and coastal group.
2. **Pull history:** monthly Sentinel-2 scenes from 2016 with AOI-level cloud screening.
3. **Build geometry:** a reference line from the temporal composite, an orientation check, and transects every 15 m. Check the largest gap between the vegetation edge and the reference line first, and set transect reach with about 100 m of margin.
4. **Extract** the vegetation edge for every scene, with the temporal prior.
5. **Run the signal gate.** If the record carries no forecastable signal, stop: the AOI should not be forecast.
6. **Build the monthly dataset.**
7. **Retrain** the MambaLSTM correction on all AOIs, and compare the test report with the active model's (see [Model Lifecycle](ai-model/model-lifecycle.md)).
8. **Publish** the AOI, geometry, history, dataset and model to the stores.

## Correcting geometry

When a reference line is wrong for a stretch of coast (it runs past the end of a beach, or cuts across a hook so transects run along the edge), the correction is made as a new geometry version: the line is trimmed or rerouted, the affected history is re-extracted, the dataset rebuilt and the model retrained. Past geometry versions are kept, so earlier observations stay traceable.

## Retiring an AOI

Set its status to `retired`. It disappears from the web interface and the platform's AOI list, while its records stay in storage.

## Retraining the model

Retrain the shared MambaLSTM correction when several months of new history have accumulated across AOIs, when AOIs are added, or after geometry corrections. Always compare the new rolling-origin report with the active one before promoting, and keep the previous package for rollback.

## Operational monitoring

| Signal | Where |
| --- | --- |
| Service and dependency health | `GET /health/live`, `GET /health/ready` |
| Job failures and stale jobs | `GET /jobs/<job_id>`, job phase events |
| Scene rejections and their reasons | Scene evaluations, observation history |
| Gaps in an AOI's record | Time-series status, validation backfill plan |
| Model state | `GET /aois/<aoi_id>/model-status` |

Diagnostics separate system errors from missing imagery and from low-quality observations; only the first needs operator action.

## Documentation updates

Update this site whenever user workflows, the public API, the model or deployment assumptions change, and publish it as a new version with the **Deploy Docs** workflow.
