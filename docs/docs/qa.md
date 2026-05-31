# Quality Assurance

Quality assurance for TERRA UGLA focuses on whether the module produces traceable, interpretable, and consistent coastal state outputs.

## Workflow Checks

Key workflows should be checked end to end:

- selecting a prepared AOI;
- loading fixed geometry;
- extracting or loading the latest vegetation edge observation;
- displaying current state on the map;
- loading historical context;
- producing forecast layers and uncertainty bounds;
- running retrospective validation cases;
- downloading or inspecting result artifacts where enabled.

## Data Quality Checks

The module depends on the quality of source imagery and AOI geometry. Important checks include:

- cloud and valid-pixel screening;
- vegetation edge extraction coverage;
- transect hit ratio and missing-transect indicators;
- consistency of fixed geometry across dates;
- availability of same-month historical observations for validation.

## Forecast Review

Forecast outputs should be reviewed with both map overlays and summary metrics. Users should inspect the central forecast line, uncertainty bounds, and any areas where the forecast suggests possible landward exposure.

## Interpretation

Quality checks support interpretation but do not remove the need for expert judgement. Coastal systems can be affected by storms, engineering works, sediment changes, vegetation dynamics, and data gaps that may not be fully captured by an automated forecast.
