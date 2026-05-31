# VE Refline-Transect Technical Route

## 1. Purpose of This Document

This document restores and consolidates the full technical route behind VE extraction, fixed `refline/transect`, `RobustUNet`, historical reconstruction, and future prediction in the current project. It is intended for:

- internal webinar and PowerPoint preparation;
- colleagues who are new to the module;
- audiences who need to understand it as a digital twin (DT) capability;
- readers who want implementation-level traceability back to code, models, scripts, and data products.

This document describes the **actual implemented mainline workflow** in the repository, not an abstract concept diagram and not only a research idea.


### 1.1 What This Document Is Trying to Clarify

If one only watches the system demo, the module may appear to be simply:

"select an AOI, get the current VE, inspect history, and preview future VE."

But the real technical route includes several important design decisions:

1. Why this capability should be presented as a **coastal-state DT module**, not just a set of image algorithms.
2. Why the system fixes `refline` and `transects` first, and only then performs extraction and forecasting.
3. Why a VE `RobustUNet` is trained to support automatic geometry bootstrap.
4. Why the operational forecasting mainline is based on transect distance rather than direct VE-mask forecasting.
5. Where these ideas are implemented in the current scripts, services, and data assets.


### 1.2 How to Understand the Module from a Digital Twin Perspective

From a digital twin point of view, this module should not be introduced as "a collection of extraction and prediction models." A more accurate description is:

**a coastal state perception, memory, and forecasting module.**

Inside the DT, its role is to:

- establish a stable spatial reference frame for a coastal AOI;
- convert satellite imagery into a measurable shoreline-state variable;
- accumulate multi-temporal outputs into an interpretable historical memory;
- forecast future state evolution together with uncertainty bounds;
- return current state, historical context, and future scenarios in a map-ready form that can be integrated with the wider DT interface.

In other words, the DT does not only need "a line extracted from one scene." It needs:

- current state;
- historical state;
- one stable coordinate system;
- future state;
- uncertainty-aware outputs.

That is what this module is designed to provide.


## 2. One-Sentence Summary

For an audience with no technical background, the module can be described in one sentence as:

**a digital twin coastal-state perception and forecasting module that lets a user choose an AOI, automatically establish a measurement frame, inspect current VE, review historical change, and preview future VE movement with uncertainty.**

Technically, the current system is not:

"segment VE directly from imagery, then directly predict future VE masks."

Instead, the implemented mainline is:

1. the user selects an AOI;
2. the system fixes one `refline` and one set of `transects` for that AOI;
3. VE `RobustUNet` is used to generate an initial geometry seed for a new AOI;
4. under the fixed geometry, the system extracts the current VE from the latest imagery;
5. VE is represented as the distance from the refline along each transect;
6. that transect-distance time series is used to predict future VE;
7. the predicted distances are reconstructed back into future VE lines and uncertainty bounds on the map.

The core idea is:

**fix the measurement coordinate system first, then perform state extraction and temporal modeling.**


## 3. What This DT Module Looks Like in the User Demonstration

For webinar listeners, the most important thing is not to begin with the models. It is to understand what capability the module provides to the user.

From the user point of view, the module offers five classes of capability:

1. **State initialization**
   - once an AOI is selected, the system can establish a usable fixed measurement framework for it;
2. **Current-state perception**
   - the latest satellite scene can be turned into a current VE state;
3. **Historical memory playback**
   - historical scenes can be organized into a consistent state history under the same geometry;
4. **Future-state forecasting**
   - the system can estimate how VE may move in the future, together with uncertainty bounds;
5. **Map-ready delivery**
   - current, historical, and future outputs are returned as geometries that can be displayed directly in the DT interface.

For webinar communication, this module is best described as:

**a DT capability that turns satellite observations into an interpretable coastal state and connects current, past, and future into one coherent operational flow.**


## 4. Actual Online Service Workflow

### 4.1 How the Service Starts

The current service is started through:

```bash
python run.py
```

This launches the Flask application, typically at `http://localhost:5000`.

The main entry points are:

- `run.py`
- `src/terra_ugla/app.py`


### 4.2 Front-End Main Interaction Flow

The current front-end VE workflow is mainly implemented in:

- `static/js/ve_workflow.js`

The actual interaction sequence is:

1. the user selects or loads an AOI;
2. the front end checks whether the AOI already has fixed `refline/transects`;
3. the system extracts the latest available scene and produces the current VE;
4. the system restores or completes historical scenes to build a prediction-ready temporal context;
5. the system runs future prediction and returns `p50 / p10 / p90` VE;
6. the front end displays current, historical, and future outputs on the map.

The key backend endpoints include:

- `GET /aoi/<aoi_id>/timeseries/status`
- `POST /aoi/<aoi_id>/extract/latest`
- `POST /aoi/<aoi_id>/history/context`
- `POST /run/<run_id>/predict/transect`


### 4.3 What the Online Mainline Really Means Technically

The online mainline is essentially:

`AOI -> fixed geometry -> latest-scene VE extraction -> historical context completion -> transect-distance forecasting -> VE geometry reconstruction`

Three points are especially important:

1. "Current VE extraction" is not a purely end-to-end neural output.
   - It is **rule-based extraction plus candidate generation and candidate scoring under fixed geometry**.
2. "Future prediction" is not direct polyline or mask prediction.
   - It is **transect-distance prediction**.
3. "Final delivery" is not an abstract numeric table.
   - Predicted distances are reconstructed into GeoJSON lines and uncertainty bounds.


### 4.4 Why This Workflow Fits a DT Module

A DT module becomes hard to trust if:

- every run uses a different reference system;
- history cannot be aligned with the present;
- outputs are hard to interpret;
- results cannot be consistently delivered back into map space.

The current route avoids that by mapping all scenes into one common geometric measurement system. As a result:

- current outputs are interpretable;
- historical outputs are replayable;
- future outputs are reconstructable;
- the results can be combined with other DT layers.

This makes the system behave like a **state module**, not only a one-scene algorithm caller.


## 5. Why VE Is the Main Target Instead of Directly Using the Waterline

Although the system can also extract the waterline, the main analysis target and forecasting target is **VE (Vegetation Edge)**. The main reasons are:

- waterline is much more sensitive to tide, wave condition, and short-term water-level variation;
- VE is more stable at monthly and yearly timescales;
- VE is better suited for temporally comparable shoreline-state time series;
- from a DT perspective, VE behaves more like a stable state variable than a highly transient boundary.

Therefore, in the current system:

- waterline is used more as an auxiliary extraction and output layer;
- VE is the primary state variable.

This is also a **state-variable design choice** from the DT perspective: the system prefers a shoreline representation that is more stable, more comparable, and more useful for medium-term monitoring and forecasting.


## 6. Why `refline` and `transects` Must Be Fixed

This is the most important design decision in the whole route.

### 6.1 What Fixed Geometry Actually Means

The `refline` is not "the true VE line of one date." It is the **fixed measurement baseline** for one AOI.  
The `transects` are not temporary visualization helpers either. They are the **common ruler** used for all historical, current, and future comparisons in that AOI.

That means:

- `refline` defines where measurement starts;
- `transects` define along which profiles measurement is performed;
- `distance_m` defines how far the target boundary is from the baseline.

If every scene generated a new `refline` and new `transects`, the reference frame would change every time, and the time series would no longer be comparable.

From the DT perspective, fixed geometry is what turns repeated observation into a maintainable state record.


### 6.2 What Goes Wrong If Geometry Is Not Fixed

If geometry were regenerated from each current scene, several problems would follow:

1. distance values from different months would not be expressed in the same coordinate system;
2. the forecasting model would learn mixed noise from shoreline change and geometry drift;
3. historical replay and future reconstruction would be difficult to align consistently;
4. future VE lines in the UI could become visually misleading because the reference frame drifts;
5. QC metrics would lose a single baseline and become harder to interpret.


### 6.3 How the Current Code Enforces "Fixed"

Fixed AOI geometry is centrally managed by:

- `src/terra_ugla/services/aoi_geometry.py`

Each AOI stores one canonical geometry set in:

- `data/aoi/<aoi_id>/refline.geojson`
- `data/aoi/<aoi_id>/refline_metadata.json`
- `data/aoi/<aoi_id>/transects.geojson`

The design rule is already stated in code:

`Within one AOI, the reference line and transects must remain fixed across the full historical and operational workflow.`

This means that:

- historical extraction;
- current extraction;
- online forecasting;
- future reconstruction;

all share the same AOI measurement baseline.


### 6.4 Current Default Geometry Parameters

The default parameters mainly come from:

- `src/terra_ugla/services/aoi_geometry.py`
- `scripts/init_aoi_fixed_geometry.py`

The key values include:

- refline smoothing window: `200 m`
- refline resample step: `20 m`
- transect spacing: `25 m`
- transect length: `450 m`
- offshore ratio: `0.5`
- transect tangent smoothing window: `120 m`
- local max turning-angle correction: `18°`
- tangent sample step: `10 m`

The engineering meaning of these parameters is:

- the refline is first smoothed into a stable baseline;
- transects must be dense enough to describe alongshore variation, but not so dense that they interfere with each other;
- local turning-angle correction helps prevent transect self-intersection and abrupt direction changes around highly curved coastlines.


## 7. How the `refline` Is Generated

### 7.1 Main Initialization Flow for a New AOI

Fixed-geometry initialization for a new AOI is mainly handled by:

- `scripts/init_aoi_fixed_geometry.py`
- `src/terra_ugla/services/extraction.py`
- `src/terra_ugla/services/aoi_geometry.py`

The main process is:

1. select a seed scene;
2. use VE `RobustUNet` to generate an initial VE line from that scene;
3. order, smooth, and resample the line;
4. save the result as the AOI's fixed `refline`;
5. generate fixed `transects` from that refline.


### 7.2 Why VE `RobustUNet` Is Used for Geometry Bootstrap

The current COASTGUARD/VedgeSat-style VE extraction itself depends on a refline constraint.  
So the system first needs an automated way to provide a "reasonable initial reference line" for a new AOI.

In that role, VE `RobustUNet` is not the final provider of VE truth. It is used to:

- automate geometry initialization;
- reduce the search space of downstream rule-based extraction;
- replace manual drawing of an initial refline.

That is why, in the webinar, it should be described more accurately as a:

**geometry bootstrap model**, not simply a final VE detector.


### 7.3 What Existing Geometry Metadata Shows

In AOI-level `refline_metadata.json`, one typically finds fields such as:

- `source_scene_id`
- `source_date`
- `source_method`
- `source_model_checkpoint`
- `metric_crs`

In the mainline, the recorded initialization mode is typically:

- `source_method: ve_unet_seed`

This shows that fixed geometry is already a real AOI-level asset in the project, not just a conceptual placeholder.


### 7.4 Geometry Initialization as a Managed Asset, Not a One-Off Step

The mainline does not assume that the first geometry bootstrap is always perfect. Several engineering ideas are already part of the route:

- if an AOI has no fixed geometry, the online extraction flow can trigger bootstrap;
- if the existing geometry becomes strongly inconsistent with downstream extraction, intersection and hit-ratio indicators can expose that mismatch;
- if intersections collapse or coverage becomes abnormally low, the geometry may be treated as stale and may require re-bootstrap or manual QA.

So the goal is not "geometry is always correct forever." The goal is:

**geometry is an AOI asset that can be initialized, reused, monitored, and rebuilt when necessary.**


## 8. How Current VE Is Extracted Under Fixed Geometry

### 8.1 The Current Mainline Is Not a Single Method, but "Candidate Generation + Scored Selection"

The per-scene VE extraction logic is mainly implemented in:

- `src/terra_ugla/services/extraction.py`

The workflow is roughly:

1. read the 5-band TIFF;
2. construct the AOI mask and cloud mask;
3. load the fixed `refline` and fixed `transects`;
4. use the COASTGUARD classifier to generate vegetation / non-vegetation classes;
5. compute NDVI;
6. generate VE candidates inside the refline buffer;
7. use transects and refline to reconstruct and score those candidates;
8. select the best VE line for that scene.

So current VE extraction is not a black-box segmentation-only method. It is a combination of:

**geometry constraints + remote-sensing features + rule-based candidates + QC scoring**


### 8.2 Main Candidate Sources for VE Lines

At least three candidate routes are considered:

1. `reconstruct`
   - reconstruct a VE line from contour and fixed-transect intersections;
2. `transect_first`
   - search directly along each transect for points satisfying NDVI/vegetation conditions, then reconstruct a line;
3. `smoothed_contour`
   - choose the main contour first, then smooth and regularize it.

The important design idea is:

- do not trust one route blindly;
- compare multiple candidate routes under one shared geometric framework;
- use fixed geometry as the common decision standard.


### 8.3 How Candidates Are Scored

Candidate scores are mainly influenced by:

- `transect_reconstruct_hit_ratio`
- `transect_hit_ratio`
- `ref_cover_ratio`
- `ref_dist`
- penalties for larger buffer expansion
- prior bonuses or penalties for candidate types

Their intuitive meaning is:

- the more complete the intersections with fixed transects, the more trustworthy the candidate is;
- the better its coverage relative to the fixed refline, the more trustworthy it is;
- if it lies too far from the refline, it may correspond to the wrong boundary;
- if the search buffer has to be enlarged too much, extraction stability is likely lower.


### 8.4 Why Fixed Geometry Makes Rule-Based Extraction More Stable

Fixed geometry is not auxiliary metadata here. It acts as a **search constraint**:

- `refline buffer` means "only search near the plausible shoreline zone";
- `transects` mean "the candidate must be explainable under one common cross-sectional system";
- `ref_cover_ratio` and `transect_hit_ratio` make wrong candidates easier to reject.

So the role of fixed geometry is not only to support later forecasting. It also:

**directly improves current-scene VE extraction quality.**


### 8.5 Why This Matters for the DT Module Framing

For the DT module, this extraction chain is important because the current state is not an opaque one-shot output. Instead, it is:

- tied to a stable measurement frame;
- traceable to explicit candidate-generation logic;
- filtered by quality and coverage criteria;
- geometrically interpretable.

This makes "current-state perception" a reviewable, maintainable, and operational capability.


## 9. How `RobustUNet` Is Trained

### 9.1 Two Different `RobustUNet` Variants Must Be Distinguished

The repository contains two relevant UNet-based lines:

1. **VE RobustUNet**
   - defined in `src/terra_ugla/models/ve_unet.py`
   - trained by `scripts/train_ve_unet.py`
   - used for VE-line extraction and refline initialization
2. **waterline RobustUNet**
   - runtime logic in `src/terra_ugla/services/unet_segmentation.py`
   - used for waterline segmentation with COASTGUARD fallback

For the webinar, one point must be explicit:

**the model used to provide the refline seed is VE RobustUNet, not the waterline UNet.**


### 9.2 VE `RobustUNet` Architecture Design

VE `RobustUNet` extends a standard UNet with structures better suited for thin boundary targets:

- residual blocks
- channel attention
- spatial attention
- attention gates on skip connections
- dilated bottleneck
- Dropout2d regularization

The main reasons for this architecture are:

- VE is a thin line-like target rather than a large region;
- coastal backgrounds are complex, with strong texture and illumination variation;
- a plain encoder-decoder can easily wash out weak boundaries;
- attention and dilation help combine local boundary sensitivity and larger contextual understanding.


### 9.3 How the Training Data Is Prepared

The main training script is:

- `scripts/train_ve_unet.py`

Its workflow is roughly:

1. discover valid LabelMe annotations under `data/labelme_work`;
2. keep only line strips labeled as `ve`;
3. render each polyline into a narrow binary mask;
4. pad to square first, then resize to `512 x 512`;
5. apply geometric and spectral augmentation on the training set.

Important details include:

- the training target is not a polygon, but a **thin-line mask**;
- pad-to-square happens before resize to avoid geometric distortion of shoreline lines;
- augmentation includes flip, light rotation, crop, brightness/contrast/saturation variation, and light blur.


### 9.4 Current Training Configuration and Loss

The current script uses a configuration roughly including:

- input size: `512`
- `base_channels=64`
- optimizer: `AdamW`
- loss: `BCEWithLogits + Dice`
- foreground weighting: `pos_weight`
- gradient clipping: `1.0`
- early stopping based on validation IoU

Why not BCE only?

- VE foreground is very sparse;
- BCE alone easily over-favors background;
- Dice more directly constrains boundary overlap quality.


### 9.5 Current Known Training Results

Based on the training summary, the VE UNet currently reaches a level roughly including:

- train samples: `503`
- val samples: `129`
- holdout samples: `14`
- best validation IoU: about `0.6404`
- holdout IoU: about `0.3548`
- holdout F1: about `0.5162`

The interpretation is:

- it is already usable for AOI geometry bootstrap;
- but cross-AOI generalization is still a real challenge;
- this is also why the production mainline does not rely only on an end-to-end segmentation model.


### 9.6 How to Explain This Model in DT Language

For non-specialist listeners, it is better not to say only:

"we trained a segmentation network."

A better explanation is:

**we trained a model that helps the system automatically establish the AOI measurement reference frame.**

That is much closer to its real role in the operational route and much more aligned with DT storytelling.


## 10. Why the Mainline Is Not Direct VE-Mask Forecasting

### 10.1 The Older VE-Mask Forecasting Route Still Exists, but It Is Not the Current Mainline

The repository still contains an older VE-mask forecasting route:

- `src/terra_ugla/services/ve_inference.py`
- `scripts/train_ve_forecaster.py`

That route roughly:

- predicts future VE masks from historical VE masks;
- uses Mamba-LSTM + MC Dropout to output future probability maps.

It is still valuable for research and earlier experimentation, but:

**it is not the current online DT mainline.**


### 10.2 Why the Operational Mainline Switched to Transect-Distance Forecasting

The current mainline uses transect distance mainly because it is better suited for engineering deployment:

1. lower dimensionality
   - a mask is a 2D field, while transect distance is a 1D set of profiles;
2. stronger comparability
   - all distances are defined against the same refline and the same transect set;
3. clearer physical meaning
   - each value means how many meters VE is seaward or landward relative to the baseline along one transect;
4. easier QC
   - missing transects, hit ratio, and ref coverage can be directly used;
5. easier uncertainty handling
   - the system can predict `p50/p10/p90` distances first and then reconstruct three VE lines.

From the DT perspective, this route is preferable because it produces a state variable that is:

- comparable;
- storable;
- updateable;
- explainable.


### 10.3 What the Current Training Data Looks Like

The dataset construction script:

- `scripts/build_transect_dataset.py`

organizes runtime outputs into a structured table containing fields such as:

- `aoi_id`
- `run_id`
- `scene_id`
- `datetime`
- `transect_id`
- `order_idx`
- `VE_distance_m`
- `WL_distance_m`
- `is_valid`
- `cloud_pct`
- `transect_hit_ratio`
- `transect_ve_hit_ratio`
- `transect_reconstruct_hit_ratio`
- `ref_cover_ratio`
- `missing_ve_transect_ratio`
- `source_refline_version`
- `source_transect_version`

This shows that the forecasting input has already moved from image space into:

**a geometry-consistent spatiotemporal table space.**


### 10.4 Current Dataset Scale and Why It Matters

Existing metadata indicates a dataset roughly covering:

- `9` AOIs
- train rows: `159145`
- val rows: `26414`
- train scenes: `226`
- val scenes: `46`
- time span approximately from `2016-01` to `2026-03`

Each sample is also tied to geometry versioning through:

- `source_refline_version`
- `source_transect_version`

This matters for the DT module because it means there is already:

- one unified dataset format;
- multi-AOI, multi-year history;
- a training loop tightly bound to stable geometry versions.


## 11. How the Current Transect Forecasting Model Is Designed, and Why

### 11.1 Current Main Model and Deployment Priority

The main training script is:

- `scripts/train_transect_forecaster.py`

The runtime loading logic is in:

- `src/terra_ugla/services/prediction.py`

If no dedicated deployment directory is available, the system falls back first to:

- `data/models/transect_forecaster_v4/best.pth`

So the operational mainline effectively prioritizes transect forecaster v4.


### 11.2 Current Input Representation: From Native Transects to a Normalized Grid

Different AOIs do not share the same real transect count.  
For example, one AOI may currently contain:

- `61` scenes
- `236` transects

To make the model see a fixed-length input, the current strategy is:

1. keep each AOI's native transect layout;
2. resample native distance sequences onto a common `normalized_grid`;
3. use a default training grid size of `128`;
4. after inference, interpolate predictions back to native transect positions;
5. reconstruct the actual VE geometry from those native positions.

This design:

- solves cross-AOI inconsistency in transect count;
- allows one unified model input shape;
- preserves the ability to map predictions back to real AOI geometry.


### 11.3 Current Model Structure

The v2/v3/v4 mainline is no longer a simple LSTM. It includes:

1. transect-distance encoder
2. relative-time embedding
3. calendar features
4. gap features
5. scene-quality covariates
6. horizon conditioning
7. Mamba-style temporal mixer
8. LSTM temporal memory
9. decoder for future distance outputs

Three especially important classes of prior information are explicitly injected:

- **calendar features**
  - month sine/cosine, day-of-year sine/cosine, long-term trend
- **gap features**
  - observation interval and relative sample age
- **quality features**
  - `cloud_pct`
  - `ref_cover_ratio`
  - `transect_ve_hit_ratio`
  - `transect_reconstruct_hit_ratio`

This is not just complexity for its own sake. It tells the model:

- observations are not equally spaced in time;
- shoreline behavior has seasonality;
- low-quality observations should not be treated the same as high-quality ones.


### 11.4 Why the Mamba + LSTM Combination Is Kept

The design keeps a Mamba-style mixer together with LSTM instead of using only one temporal block. The reasons include:

- Mamba-style modules are useful for longer dependencies and efficient sequence mixing;
- LSTM remains stable and effective for local memory on medium-scale data;
- the current data scale does not yet force a heavier pure-Transformer route;
- the project needs a **stable, deployable, medium-complexity model**, not a novelty-driven architecture.

In short, this is a model choice optimized for engineering robustness.


### 11.5 Current Model Performance

Existing v4 metrics indicate:

- best epoch: `68`
- validation MAE: about `26.95 m`
- validation RMSE: about `40.52 m`
- parameter count: `104,617`
- normalized-grid transects: `128`
- quality covariates include `cloud_pct`, `ref_cover_ratio`, `transect_ve_hit_ratio`, `transect_reconstruct_hit_ratio`
- supported horizon buckets are roughly `0.5` to `5` years

Across forecast horizons, MAE stays roughly in the `25 m` to `29 m` range, suggesting:

- the model does not collapse completely at different forecast lengths;
- long-horizon error is somewhat higher, but still within the same order of magnitude.


### 11.6 Current Online Prediction Outputs

The current online prediction finally returns:

- `forecast_VE_distance_m`
- `uncertainty_std_m`
- `p10_m`
- `p90_m`

These are then reconstructed into GeoJSON:

- `p50` future VE mainline
- `p10` lower bound
- `p90` upper bound

The front end displays them as:

- `Predicted VE (p50)`
- `Predicted VE (p10/p90 bounds)`

So uncertainty is no longer only an abstract number. It becomes:

**a map-visible geometric uncertainty band.**


## 12. Online Historical Context Is Not Fully Reloaded Each Time, but Strategically Restored

The relevant logic is mainly in:

- `src/terra_ugla/services/transect_online.py`

It does not reload the full history every time. Instead, it strategically restores the most useful context. The known settings include:

- minimum historical scene count: `12`
- target historical scene count: `16`
- historical search window: `6` years
- recent-history priority: `18` months
- same-season anchor window: `5` years

The rationale is:

- prediction needs enough context, but should not require heavy full-history processing every time;
- for VE, same-season history matters;
- combining recent history with same-season history is more meaningful than using only the temporally nearest scenes.

In DT language, this acts as the module's:

**runtime memory restoration mechanism.**

Not all past data is equally useful. The system preferentially restores the history most informative for understanding the current state and forecasting the future.


## 13. Improvements Over the Older Routes

### 13.1 Compared with "Direct VE Segmentation per Scene + Future VE Mask Forecasting"

The current route has several advantages:

- reduced error compounding;
- improved temporal comparability;
- shoreline change expressed as an interpretable quantity in meters;
- easier quality control;
- easier reconstruction back into map geometry.


### 13.2 Compared with "Pure Rule-Based Method + Manual Refline"

The current route also improves by:

- enabling automatic initialization for new AOIs;
- removing the need to redraw the refline manually each time;
- keeping the geometric stability and physical interpretability of rule-based extraction;
- allowing the system to operate as a real online service rather than only an offline research pipeline.


### 13.3 What This Means as a DT-Module Evolution

The significance of these improvements is not only better technical performance. It is that the project has evolved from:

- "a collection of research scripts"

into:

- "a DT functional module that can be initialized, updated, replayed, forecast, and visualized"

In other words, it increasingly behaves like a real operational state module.


## 14. Key "Do Not Confuse" Points for the Webinar

### 14.1 Do Not Mix Up the Two Forecasting Routes

The repository currently contains both:

1. the older VE mask forecaster
2. the current transect-distance forecaster

The webinar should make it explicit:

- the current online DT workflow uses `fixed geometry + transect forecaster`
- the VE mask forecaster is a retained historical research route, not the operational mainline


### 14.2 Do Not Mix Up VE `RobustUNet` and waterline `RobustUNet`

The distinction should be explicit:

- VE `RobustUNet`: used for VE/refline initialization
- waterline `RobustUNet`: used for waterline segmentation, with fallback to COASTGUARD


### 14.3 `refline` Is Not "the True VE of Some Month"

The more accurate wording is:

**the fixed AOI measurement baseline**

not "the initial VE ground truth."


## 15. Current Limitations and Next Priorities

### 15.1 VE `RobustUNet` Still Faces Cross-AOI Generalization Pressure

From the existing training results:

- validation IoU is higher than holdout IoU;
- this indicates a domain gap on new geomorphology, new backgrounds, and new AOIs.


### 15.2 Most Fixed Geometry Is Still Used Directly After Automatic Generation

Current metadata often still shows:

- `manual_qc_applied: false`

This means the system is already automated, but it also implies:

- a more systematic geometry QA strategy is still needed;
- for complex coastlines and abnormal scenes, a human review path should remain available.


### 15.3 Forecast Evaluation Is Still Mainly Distance Error

The most stable current evaluation metrics are:

- per-transect MAE
- per-transect RMSE

If later reporting becomes more business-facing, it would be useful to add:

- geometric error of reconstructed VE lines;
- AOI-level advance/retreat statistics;
- coastal change-hotspot indicators.


### 15.4 Next Steps from the DT-Module Perspective

From a DT-module evolution perspective, the main next steps are:

1. improve generalization to new AOIs and new geomorphologies;
2. systematize geometry QA;
3. extend evaluation from distance error to state interpretation and business-facing indicators;
4. continue improving online deployment stability and maintainability.


## 16. Suggested PPT / Webinar Structure

If this document is used to prepare the webinar PPT, the recommended sequence is:

### Slide 1. DT Context and Module Value

- why the coastal digital twin needs this module
- why shoreline-state monitoring and future outlook matter
- what the user can already do in the current demo


### Slide 2. What the Audience Should Remember First

- this is a DT coastal-state perception and forecasting module
- it provides current state, historical memory, and future outlook
- its outputs are ready for map display and downstream interpretation


### Slide 3. Current Online Main Workflow

- AOI
- latest imagery
- current VE extraction
- historical context completion
- future VE prediction
- map-based visualization output


### Slide 4. Key Design Idea

- each AOI has a fixed `refline`
- each AOI has fixed `transects`
- shoreline state is represented under one unified geometric coordinate system


### Slide 5. Why `refline/transect` Must Be Fixed

- time-series comparability
- reduced accumulated error
- support for stable rule-based extraction
- support for future VE reconstruction and uncertainty expression


### Slide 6. VE `RobustUNet`

- why it is needed
- it is not final ground truth, but geometry bootstrap
- architecture highlights
- training data and current metrics


### Slide 7. Current VE Extraction Route

- fixed geometry + COASTGUARD/VedgeSat-style rule-based extraction
- contour / transect-first / reconstruct as multiple candidates
- QC and scoring logic


### Slide 8. Forecasting Model

- why direct mask forecasting is not the operational mainline
- transect-distance representation
- Mamba + LSTM + calendar/gap/quality features
- current validation metrics


### Slide 9. What This Module Means for the DT

- repeatable coastal-state monitoring
- one reference frame linking current, historical, and future states
- interpretable outputs with uncertainty
- natural integration into wider DT workflows


### Slide 10. Current Conclusion and Next Steps

- an end-to-end online mainline is already in place
- a trainable and updateable data loop already exists
- the next priorities are generalization, QC, evaluation metrics, and deployment stability


## 17. Final Conclusion

The current project has evolved from "a collection of research scripts" into a fairly clear online technical mainline:

- VE `RobustUNet` is used to initialize fixed AOI geometry automatically;
- fixed `refline/transects` constrain historical and current VE extraction;
- transect distance acts as the unified temporal state representation;
- a temporal model with time, seasonality, and quality features predicts future distances;
- those distances are reconstructed into a future VE mainline and uncertainty bounds that can be displayed directly.

For webinar and DT demonstration purposes, the most important message is not:

"we used many models."

The more important message is:

**we reorganized shoreline change into a geometry-stable, interpretable, online-operable, continuously updateable, and iteratively improvable closed loop of data and models.**

If the audience should remember it as a digital twin component, the ideal impression is:

**this module gives the DT a usable coastal-state memory and forecasting capability, rather than only a one-off shoreline extraction result.**
