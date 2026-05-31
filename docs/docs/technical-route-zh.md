# VE Refline-Transect Technical Route

## 1. 文档目的

本文档用于恢复并整理当前项目中 VE 提取、固定 `refline/transect`、`RobustUNet`、历史重建与未来预测相关的完整技术路线，面向：

- 内部 webinar 与后续 PowerPoint 编写；
- 新加入项目、尚未了解该模块的同事；
- 需要从数字孪生（DT）模块角度理解该能力的人；
- 需要进一步追溯到代码、模型和脚本实现细节的人。

这份文档描述的是仓库当前主线的**实际实现方案**，不是抽象概念图，也不是纯研究路线推演。


### 1.1 这份文档想解决什么问题

如果只看系统演示，用户会觉得这个模块只是“选 AOI，然后系统给出当前 VE、历史 VE、未来 VE”。  
但在实际技术路线中，它包含了几层关键设计：

1. 为什么这个模块在 DT 中应被看作“海岸状态感知与预测模块”，而不是单次图像算法。
2. 为什么系统要先固定 `refline` 和 `transects`，再做提取和预测。
3. 为什么要训练一个 VE `RobustUNet` 来辅助自动初始化几何。
4. 为什么在线主线不是直接预测 VE mask，而是预测 transect distance。
5. 当前代码中这些设计分别落在哪些脚本、服务和数据产物里。


### 1.2 从数字孪生视角如何理解本模块

从数字孪生（DT）角度，这个模块不应被介绍成“一组模型和规则算法”。更准确的说法是：

**这是一个海岸状态感知、记忆和预测模块。**

它在 DT 中承担的职责是：

- 给每个海岸 AOI 建立稳定的空间参考框架；
- 把卫星影像转换成统一可量化的海岸状态变量；
- 把分散场景沉淀为可回放、可对比的历史记忆；
- 给出未来状态变化的预测与不确定性范围；
- 以地图可直接展示、可与其他 DT 模块联动的几何形式返回结果。

也就是说，DT 里需要的不只是“某一景影像上提取出一条线”，而是：

- 当前状态；
- 历史状态；
- 统一坐标系；
- 未来状态；
- 带不确定性的解释结果。

本模块正是在做这件事。


## 2. 一句话总述

如果给没有背景的听众一句话介绍当前模块，可以表述为：

**这是一个数字孪生中的海岸状态感知与预测模块，允许用户选择 AOI，自动建立测量参考框架，查看当前 VE、回看历史变化，并预览带不确定性的未来 VE 变化。**

从技术主线看，当前系统不是“影像直接分割 VE，再直接预测未来 VE mask”，而是：

1. 用户选择 AOI；
2. 系统为 AOI 固定一条 `refline` 和一组 `transects`；
3. 用 VE `RobustUNet` 为新 AOI 自动生成初始几何种子；
4. 在固定几何约束下，从当前影像中提取当前 VE；
5. 把 VE 表达成“每条 transect 上相对 refline 的距离”；
6. 用这个 transect-distance 时间序列预测未来 VE；
7. 再把预测距离重建回地图上的未来 VE 线和不确定性边界。

它的核心思想是：

**先固定测量坐标系，再做状态提取和时序建模。**


## 3. 这个 DT 模块在用户演示里表现为什么

对于 webinar 听众，最重要的不是先理解模型，而是先理解这个模块对用户展示出来的能力。

从用户视角，这个模块提供五类能力：

1. **状态初始化**
   - 用户选定 AOI 后，系统能自动建立该 AOI 的固定测量框架。
2. **当前状态感知**
   - 系统可以从最新卫星场景中提取当前 VE 状态。
3. **历史记忆回放**
   - 系统可以在同一套几何坐标下组织和展示历史状态变化。
4. **未来状态预测**
   - 系统可以给出未来 VE 的位置预测和不确定性范围。
5. **地图级输出**
   - 当前、历史、未来结果都能以 GeoJSON / map geometry 形式直接进入 DT 可视化界面。

因此，webinar 中建议把它介绍成：

**一个把卫星观测转成可解释海岸状态、并且把当前-历史-未来串联起来的 DT 功能模块。**


## 4. 当前在线服务的实际工作流

### 4.1 启动方式

当前服务通过：

```bash
python run.py
```

启动 Flask 应用，默认地址为 `http://localhost:5000`。

主入口代码主要在：

- `run.py`
- `src/terra_ugla/app.py`


### 4.2 前端主线交互

当前前端 VE 工作流主要在：

- `static/js/ve_workflow.js`

实际交互顺序是：

1. 用户选择或加载 AOI；
2. 前端检查该 AOI 是否已存在固定 `refline/transects`；
3. 系统抽取最新可用场景并生成当前 VE；
4. 系统补齐或确认历史场景，形成可预测的时序上下文；
5. 系统运行未来预测，返回 `p50 / p10 / p90` VE；
6. 前端在地图上显示当前线、历史线与未来线。

对应的核心后端接口包括：

- `GET /aoi/<aoi_id>/timeseries/status`
- `POST /aoi/<aoi_id>/extract/latest`
- `POST /aoi/<aoi_id>/history/context`
- `POST /run/<run_id>/predict/transect`


### 4.3 在线主线的真实技术含义

在线主线本质上是：

`AOI -> 固定几何 -> 最新场景 VE 提取 -> 历史上下文补齐 -> transect 距离预测 -> VE 几何重建`

这里要特别强调三点：

1. “当前 VE 提取”不是纯神经网络端到端输出。
   - 它是**固定几何约束下的规则提取 + 候选生成 + 候选评分**。
2. “未来预测”不是直接预测 polyline 或 mask。
   - 它是**预测 transect distance**。
3. “结果展示”不是抽象数值。
   - 预测结果会被重建为 GeoJSON 线与不确定性边界。


### 4.4 从 DT 视角看这个工作流为什么合理

DT 模块最怕的是：

- 每次处理逻辑都不一样；
- 历史与当前无法对齐；
- 输出不可解释；
- 无法稳定落到地图与后续业务层。

当前方案通过固定几何和距离化表示，把所有场景都映射到同一坐标系里，因此：

- 当前结果可解释；
- 历史结果可回放；
- 未来结果可重建；
- 结果可与其他 DT 图层联动。

这使它更像“状态模块”，而不是“单景算法调用器”。


## 5. 为什么主对象是 VE，而不是直接用水线

虽然系统也能提取 waterline，但当前主分析对象和主预测对象是 **VE（Vegetation Edge）**。主要原因是：

- 水线受潮位、波浪、短时水位变化影响更大；
- VE 在月尺度、年尺度下更稳定；
- VE 更适合构建跨时间可比的海岸状态时间序列；
- 对 DT 来说，VE 更像一个稳态状态变量，而不是受瞬时环境扰动较大的边界。

因此当前系统中：

- waterline 更多作为辅助提取与补充输出；
- VE 才是主状态变量。

从 DT 角度，这也是“状态变量选型”问题：  
系统优先选取一个更稳定、更适合比较、也更适合中期预测的海岸表征量。


## 6. 为什么 `refline` 和 `transects` 必须固定

这是整个方案最关键的设计点。

### 6.1 固定几何的本质

`refline` 不是某一时刻的“真值 VE 线”，而是某个 AOI 的**固定测量基准线**。  
`transects` 也不是临时可视化辅助线，而是这个 AOI 后续所有历史、当前、未来对比的**统一标尺**。

也就是说：

- `refline` 定义“从哪里开始量”；
- `transects` 定义“沿哪些剖面量”；
- `distance_m` 定义“量到哪里”。

如果每景影像都重新生成 `refline` 和 `transects`，那么每次的参考系都会变化，时间序列就无法保持可比性。

从 DT 角度看，固定几何的意义就是：  
把重复观测真正沉淀成可持续维护的状态记录。


### 6.2 如果不固定会出现什么问题

如果每次都根据当前场景重新生成几何，会产生：

1. 同一个 AOI 不同月份的距离值不在同一坐标系内；
2. 预测模型学到的是“海岸变化 + 几何漂移”的混合噪声；
3. 历史回放和未来重建无法稳定对齐；
4. 前端用户看到的未来 VE 线会因为参考框架漂移而失真；
5. 质量控制指标缺少统一基准，很难解释。


### 6.3 当前代码中如何体现“固定”

固定 AOI 几何由以下模块统一管理：

- `src/terra_ugla/services/aoi_geometry.py`

每个 AOI 在如下位置保存 canonical geometry：

- `data/aoi/<aoi_id>/refline.geojson`
- `data/aoi/<aoi_id>/refline_metadata.json`
- `data/aoi/<aoi_id>/transects.geojson`

代码中已经直接写明设计原则：

`Within one AOI, the reference line and transects must remain fixed across the full historical and operational workflow.`

也就是说：

- 历史抽取；
- 当前抽取；
- 在线预测；
- 未来重建；

全部共享同一套 AOI 几何基准。


### 6.4 当前默认几何参数

当前默认参数主要来自：

- `src/terra_ugla/services/aoi_geometry.py`
- `scripts/init_aoi_fixed_geometry.py`

核心参数包括：

- refline smoothing window: `200 m`
- refline resample step: `20 m`
- transect spacing: `25 m`
- transect length: `450 m`
- offshore ratio: `0.5`
- transect tangent smoothing window: `120 m`
- local max angle correction: `18°`
- tangent sample step: `10 m`

这些参数的工程意义是：

- refline 先平滑为稳定基线；
- transects 足够密以表达沿岸变化，但不能密到互相干扰；
- 局部转角修正避免复杂弯曲海岸处出现 transect 自交或方向突变。


## 7. `refline` 是怎么生成的

### 7.1 新 AOI 初始化主流程

新 AOI 固定几何初始化主要由以下组件协同完成：

- `scripts/init_aoi_fixed_geometry.py`
- `src/terra_ugla/services/extraction.py`
- `src/terra_ugla/services/aoi_geometry.py`

主流程大致是：

1. 选择一个 seed scene；
2. 用 VE `RobustUNet` 在该 seed scene 上生成初始 VE 线；
3. 对线进行排序、平滑、重采样；
4. 把结果保存为 AOI 的固定 `refline`；
5. 再从这条 refline 生成固定 `transects`。


### 7.2 为什么要用 VE `RobustUNet` 来做 geometry bootstrap

当前 COASTGUARD/VedgeSat 风格 VE 提取本身依赖 refline 约束。  
因此系统必须先拥有一个自动化方法，为 AOI 提供一条“足够合理的初始参考线”。

VE `RobustUNet` 在这里的角色不是“最终 VE 真值生成器”，而是：

- 自动初始化几何；
- 缩小后续规则提取的搜索空间；
- 替代人工手绘初始 refline 的步骤。

因此在 webinar 中，更准确的表达是：

**VE RobustUNet 是 geometry bootstrap model，而不是简单的 final VE detector。**


### 7.3 现有初始化资产中可以看到什么

例如某些 AOI 的 `refline_metadata.json` 中会记录：

- `source_scene_id`
- `source_date`
- `source_method`
- `source_model_checkpoint`
- `metric_crs`

其中主线初始化方式可见为：

- `source_method: ve_unet_seed`

这说明固定几何已经不是概念，而是真实落地在 AOI 资产中的模块产物。


### 7.4 几何初始化的容错思路

当前主线并不假设第一次初始化一定完美。工程上还包含几个重要思路：

- 如果 AOI 缺少固定几何，在线抽取流程可以触发 bootstrap；
- 如果当前固定几何与后续抽取结果严重不一致，系统可以通过 intersection/hit ratio 等指标发现异常；
- 当交点几乎为零或覆盖率异常低时，几何可能被判定为 stale，需要重新 bootstrap 或人工 QA。

这意味着当前方案的目标不是“几何永远正确”，而是：

**让几何成为可以初始化、复用、监控、必要时重建的 AOI 状态资产。**


## 8. 固定几何下，当前 VE 是如何提取的

### 8.1 当前主线不是单一算法，而是“候选生成 + 打分选择”

每景 VE 提取的主逻辑在：

- `src/terra_ugla/services/extraction.py`

处理步骤大致是：

1. 读取 5 波段 TIFF；
2. 构造 AOI mask 与 cloud mask；
3. 读取固定 `refline` 和固定 `transects`；
4. 用 COASTGUARD 分类器生成 vegetation / non-vegetation 分类；
5. 计算 NDVI；
6. 在 refline buffer 范围内生成 VE 候选；
7. 用 transects 和 refline 对候选进行重建与评分；
8. 选出该景的最优 VE 线。

也就是说，当前 VE 提取不是一个黑箱模型，而是一个：

**几何约束 + 遥感特征 + 规则候选 + QC 评分**

的组合流程。


### 8.2 当前候选 VE 线的主要来源

当前至少会考虑三类候选：

1. `reconstruct`
   - 从 contour 与固定 transects 的交点重建 VE 线；
2. `transect_first`
   - 沿每条 transect 直接搜索满足 NDVI/vegetation 条件的交点，再重建成线；
3. `smoothed_contour`
   - 从 contour 中选主线，再做平滑与规则化。

这一步的思想是：

- 不盲信单一路线；
- 把多种候选放进同一几何框架里比较；
- 让固定几何作为统一判别标准。


### 8.3 当前候选如何打分

候选分数主要受以下指标影响：

- `transect_reconstruct_hit_ratio`
- `transect_hit_ratio`
- `ref_cover_ratio`
- `ref_dist`
- buffer 扩大倍数惩罚
- 候选类型的先验加减分

这些指标的直观含义是：

- 与固定 transects 的交点越完整，候选越可信；
- 对固定 refline 的覆盖越充分，候选越可信；
- 候选离 refline 太远，可能抽到错误边界；
- 如果必须把搜索 buffer 放得很大，说明候选稳定性可能较差。


### 8.4 为什么固定几何会让规则提取更稳

固定几何在这里不是附属信息，而是**搜索约束**：

- `refline buffer` 决定“只在合理海岸带附近找边界”；
- `transects` 决定“候选必须能在统一剖面体系内被解释”；
- `ref_cover_ratio` 与 `transect_hit_ratio` 决定“错误候选更容易被过滤掉”。

因此固定几何的意义不只是“方便后续预测”，而是：

**直接提升当前场景 VE 提取质量。**


### 8.5 与 DT 模块定位之间的关系

对 DT 来说，这一提取链路非常关键，因为它保证当前状态不是一次不可解释的端到端输出，而是：

- 有稳定参考系；
- 有清晰候选来源；
- 有质量评分逻辑；
- 有几何可解释性。

这使得“当前状态感知”成为可审查、可维护、可持续运行的模块能力。


## 9. `RobustUNet` 是如何训练的

### 9.1 必须区分两种 `RobustUNet`

仓库里实际存在两套相关 UNet 体系：

1. **VE RobustUNet**
   - 定义在 `src/terra_ugla/models/ve_unet.py`
   - 训练脚本在 `scripts/train_ve_unet.py`
   - 用于 VE 线提取与 refline 初始化
2. **waterline RobustUNet**
   - 运行逻辑在 `src/terra_ugla/services/unet_segmentation.py`
   - 用于 waterline segmentation，并保留 COASTGUARD fallback

webinar 中必须明确：

**用来生成 refline 种子的，是 VE RobustUNet，不是 waterline UNet。**


### 9.2 VE `RobustUNet` 的架构设计

VE `RobustUNet` 在普通 UNet 基础上增加了更适合细长边界目标的结构：

- residual blocks
- channel attention
- spatial attention
- attention gates on skip connections
- dilated bottleneck
- Dropout2d regularisation

设计原因主要是：

- VE 是细线状目标，不是大块语义区域；
- 海岸背景复杂，纹理与光照变化大；
- 普通 encoder-decoder 容易淹没弱边界；
- attention 与 dilation 可以同时增强局部边界敏感性和大尺度上下文理解。


### 9.3 训练数据是如何构造的

训练脚本：

- `scripts/train_ve_unet.py`

主要流程：

1. 从 `data/labelme_work` 发现有效 LabelMe 标注；
2. 仅保留 label 为 `ve` 的 line strip；
3. 把 polyline 渲染成窄线宽二值 mask；
4. 先 pad 到正方形，再 resize 到 `512 x 512`；
5. 在训练集上进行几何和光谱增强。

关键点包括：

- 训练目标不是 polygon，而是**窄线带 mask**；
- pad-to-square 先于 resize，避免海岸线几何被拉伸；
- 增强包括翻转、轻微旋转、裁剪、亮度/对比度/饱和度变化、轻模糊。


### 9.4 当前训练配置与损失

当前脚本使用的大致配置包括：

- input size: `512`
- `base_channels=64`
- optimizer: `AdamW`
- loss: `BCEWithLogits + Dice`
- foreground weighting: `pos_weight`
- gradient clipping: `1.0`
- early stopping: 基于 validation IoU

不用 BCE 单独训练，而采用 BCE + Dice，原因是：

- VE 前景非常稀疏；
- 只用 BCE 容易过分偏向背景；
- Dice 更直接约束边界重叠质量。


### 9.5 当前已知训练结果

根据现有训练摘要，当前 VE UNet 已有如下级别结果：

- train samples: `503`
- val samples: `129`
- holdout samples: `14`
- best validation IoU: 约 `0.6404`
- holdout IoU: 约 `0.3548`
- holdout F1: 约 `0.5162`

这些结果的含义是：

- 它已经足够支撑 AOI geometry bootstrap；
- 但跨 AOI 泛化仍然有明显压力；
- 这也是为什么在线主线并不完全依赖“端到端网络直接给最终结果”。


### 9.6 从 DT 模块角度如何解释这个模型

对非技术听众，建议不要把它讲成“我们训练了一个分割网络”。更好的说法是：

**我们训练了一个用于自动建立 AOI 测量参考框架的模型。**

这更符合它在主线中的真实职责，也更符合 DT 模块讲述方式。


## 10. 为什么主线不是直接预测 VE mask，而是预测 transect distance

### 10.1 旧的 VE mask 预测链路仍然存在，但不是当前主线

仓库中仍保留了一套旧的 VE mask forecasting 路线：

- `src/terra_ugla/services/ve_inference.py`
- `scripts/train_ve_forecaster.py`

这套方法大致是：

- 用历史 VE mask 预测未来 VE mask；
- 用 Mamba-LSTM + MC Dropout 输出未来概率图。

它对研究和早期实验仍有价值，但：

**它不是当前在线 DT 演示主线。**


### 10.2 当前主线为什么切换到 transect-distance forecasting

当前主线之所以切换到 transect distance，主要因为它更适合工程落地：

1. 维度更低
   - mask 是二维场，transect distance 是一维剖面集合；
2. 可比性更强
   - 所有距离都相对同一条 refline、同一组 transects；
3. 物理意义更直接
   - 每个值代表某条剖面上 VE 相对基线前进或后退多少米；
4. 更容易做 QC
   - missing transects、hit ratio、ref coverage 都能直接使用；
5. 更容易表达不确定性
   - 直接输出 `p50/p10/p90` 距离，再重建成三条 VE 线。

从 DT 角度看，这条路线更有利，因为它产生的是一种：

- 可比较；
- 可存储；
- 可更新；
- 可解释；

的状态变量。


### 10.3 当前训练数据长什么样

训练集构建脚本：

- `scripts/build_transect_dataset.py`

会把运行结果整理成结构化数据表，字段包括：

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

这说明预测模型输入已经从图像空间切换到：

**几何一致的时空表空间。**


### 10.4 当前数据集规模与意义

现有 metadata 显示数据集已覆盖：

- `9` 个 AOI
- train rows: `159145`
- val rows: `26414`
- train scenes: `226`
- val scenes: `46`
- 时间跨度大致从 `2016-01` 到 `2026-03`

并且通过：

- `source_refline_version`
- `source_transect_version`

保证每条样本都绑定明确的几何版本。

这对 DT 模块很重要，因为它意味着：

- 有统一数据格式；
- 有跨 AOI、跨年份历史；
- 有与稳定几何版本绑定的数据闭环。


## 11. 当前 transect 预测模型如何设计，为什么这样设计

### 11.1 当前主模型与部署优先级

主训练脚本：

- `scripts/train_transect_forecaster.py`

运行时加载逻辑：

- `src/terra_ugla/services/prediction.py`

如果没有单独部署目录，系统会优先回退到：

- `data/models/transect_forecaster_v4/best.pth`

这说明当前在线主线优先使用的是 transect forecaster v4。


### 11.2 当前输入表示：从 native transects 到 normalized grid

不同 AOI 的真实 transect 数量并不一样。  
例如某 AOI 当前可能有：

- `61` 个 scene
- `236` 条 transects

为了让模型接受统一长度输入，当前方案是：

1. 保留 AOI 的 native transect layout；
2. 把 native distance 序列重采样到统一 `normalized_grid`；
3. 当前训练默认网格数为 `128`；
4. 推理后再插值回 native transect positions；
5. 最后从 native positions 重建真实 VE 几何。

这样做的好处是：

- 解决不同 AOI transect 数量不一致问题；
- 允许模型统一训练；
- 同时保留回到真实 AOI 几何的能力。


### 11.3 当前模型结构

当前 v2/v3/v4 主线不再是简单 LSTM，而是组合式时序结构，包含：

1. transect-distance encoder
2. relative-time embedding
3. calendar features
4. gap features
5. scene-quality covariates
6. horizon conditioning
7. Mamba-style temporal mixer
8. LSTM temporal memory
9. decoder 输出未来距离

其中显式加入了三类重要业务先验：

- **calendar features**
  - 月份正余弦、年内日期正余弦、长期趋势
- **gap features**
  - 观测间隔、相对样本 age
- **quality features**
  - `cloud_pct`
  - `ref_cover_ratio`
  - `transect_ve_hit_ratio`
  - `transect_reconstruct_hit_ratio`

这些设计不是为了单纯“增加复杂度”，而是为了让模型知道：

- 观测并不是等间隔；
- 海岸变化有季节性；
- 低质量场景不能与高质量场景同等对待。


### 11.4 为什么保留 Mamba + LSTM 组合

当前设计保留 Mamba-style mixer 与 LSTM 组合，而不是只用单一模块，原因包括：

- Mamba 类模块适合捕捉较长依赖与高效序列混合；
- LSTM 在中等规模数据下仍然稳定，且对局部时序记忆有效；
- 当前数据规模还没有到必须依赖更重 Transformer 才合理的程度；
- 项目目标是**稳定可部署的中等复杂度模型**，而不是追求论文式新颖性。

换句话说，这是一种偏工程稳健性的选型。


### 11.5 当前模型效果

现有 v4 metrics 显示：

- best epoch: `68`
- validation MAE: 约 `26.95 m`
- validation RMSE: 约 `40.52 m`
- 参数量：`104,617`
- normalized-grid transects: `128`
- 质量特征包括 `cloud_pct`, `ref_cover_ratio`, `transect_ve_hit_ratio`, `transect_reconstruct_hit_ratio`
- 支持 horizon bucket 大致为 `0.5` 到 `5` 年

按 horizon 分析，MAE 大致保持在 `25 m` 到 `29 m` 量级，意味着：

- 模型在不同预测年限下没有完全失控；
- 远期误差略高，但仍保持同一量级。


### 11.6 当前在线预测输出

当前在线预测最终返回：

- `forecast_VE_distance_m`
- `uncertainty_std_m`
- `p10_m`
- `p90_m`

随后重建为 GeoJSON：

- `p50` 未来 VE 主线
- `p10` 下边界
- `p90` 上边界

前端对应展示为：

- `Predicted VE (p50)`
- `Predicted VE (p10/p90 bounds)`

这意味着系统的不确定性不再只是抽象数值，而是：

**可直接在地图中表达的几何不确定性带。**


## 12. 在线历史上下文不是全量拉取，而是有策略地补齐

相关逻辑主要在：

- `src/terra_ugla/services/transect_online.py`

它并不是每次把所有历史都重新拉取，而是按业务使用价值补齐上下文。当前已知策略包括：

- minimum historical scene count: `12`
- target historical scene count: `16`
- historical search window: `6` years
- recent-history priority: `18` months
- same-season anchor window: `5` years

其背后思路是：

- 预测需要足够上下文，但不应每次做重型全量处理；
- 对 VE 来说，同季节历史很重要；
- 最近历史与同季节历史一起使用，比单纯时间最近更合理。

如果用 DT 的语言来讲，这部分相当于：

**模块的运行时记忆机制。**

并不是所有过去都同等重要，系统优先恢复那些最能帮助理解当前状态和预测未来状态的历史记忆。


## 13. 当前方案相对旧路线的改进

### 13.1 相比“每景直接分割 VE 再预测 VE mask”

当前方案的优势在于：

- 降低误差级联；
- 提升跨时间可比性；
- 把海岸变化表达成以米为单位的可解释量；
- 更容易做质量控制；
- 更容易把结果重建回地图几何。


### 13.2 相比“纯规则法 + 人工 refline”

当前方案的优势在于：

- 新 AOI 可以自动初始化；
- refline 不再需要每次人工重画；
- 规则法仍保留几何稳定性与物理解释性；
- 系统可以进入真正在线服务流程，而不是停留在离线科研处理。


### 13.3 从 DT 模块演进角度如何评价

这些改进的重要性不仅在于技术效果，而在于项目已经从：

- “研究脚本集合”

演进为：

- “可初始化、可更新、可回放、可预测、可可视化的 DT 功能模块”

也就是说，它越来越像一个真正可演示、可运维、可集成的状态模块。


## 14. webinar 中需要讲清楚的“不要混淆”点

### 14.1 不要把两套预测链路混为一谈

仓库中目前同时存在：

1. 旧的 VE mask forecaster
2. 当前主线的 transect-distance forecaster

webinar 中应明确：

- 当前在线 DT 主工作流使用的是 `fixed geometry + transect forecaster`
- VE mask forecaster 是保留的历史研究路线，不是现行主线


### 14.2 不要把 VE `RobustUNet` 和 waterline `RobustUNet` 混为一谈

应该明确区分：

- VE `RobustUNet`：负责 VE/refline 初始化
- waterline `RobustUNet`：负责水线分割，且仍可 fallback 到 COASTGUARD


### 14.3 `refline` 不是“某个月的真值 VE”

更准确的表述应该是：

**AOI 固定测量基准线**

而不是“初始 VE 真值”。


## 15. 当前方案的局限与下一步重点

### 15.1 VE `RobustUNet` 仍有跨 AOI 泛化压力

根据现有训练结果：

- validation IoU 高于 holdout IoU；
- 说明模型在新地貌、新背景、新 AOI 上仍存在 domain gap。


### 15.2 固定几何大多仍是自动生成后直接使用

例如现有 metadata 中常可见：

- `manual_qc_applied: false`

这说明系统已经自动化，但也意味着：

- 仍需建立更系统的 geometry QA 策略；
- 对复杂海岸和异常场景，仍应保留人工校核入口。


### 15.3 当前预测评估主要仍是距离误差

当前最稳定的评估指标是：

- per-transect MAE
- per-transect RMSE

如果后续更偏业务汇报，可以继续增加：

- reconstructed VE line 的几何误差；
- AOI 级别前进/后退统计；
- 更贴近海岸业务的 change hotspot 指标。


### 15.4 从 DT 模块演进看下一步

如果从 DT 模块角度看，下一步重点大致是：

1. 提升新 AOI / 新地貌泛化能力；
2. 把 geometry QA 制度化；
3. 把评估从距离误差扩展到状态解释与业务指标；
4. 持续提升在线部署稳定性与可维护性。


## 16. 建议的 PPT / webinar 讲述结构

如果要用这份文档做 webinar PPT，建议讲述顺序如下。

### Slide 1. DT 背景与模块价值

- 海岸数字孪生为什么需要这个模块
- 海岸状态监测与未来预判为什么重要
- 当前 demo 中用户已经能做什么


### Slide 2. 希望听众先记住什么

- 这是一个 DT 中的海岸状态感知与预测模块
- 它给出当前状态、历史记忆和未来趋势
- 输出可以直接进入地图展示与后续业务解释


### Slide 3. 当前在线主流程

- AOI
- 最新影像
- 当前 VE 提取
- 历史上下文补齐
- 未来 VE 预测
- 地图可视化输出


### Slide 4. 关键设计思想

- 每个 AOI 固定 `refline`
- 每个 AOI 固定 `transects`
- 用统一几何坐标系表达海岸状态


### Slide 5. 为什么固定 `refline/transect`

- 时间序列可比性
- 降低误差累积
- 支持规则法稳定提取
- 支持未来 VE 重建和不确定性表达


### Slide 6. VE `RobustUNet`

- 为什么需要它
- 它不是最终真值，而是 geometry bootstrap
- 架构亮点
- 训练数据和当前指标


### Slide 7. 当前 VE 提取链路

- fixed geometry + COASTGUARD/VedgeSat 风格规则提取
- contour / transect-first / reconstruct 多候选
- QC 和评分机制


### Slide 8. 预测模型

- 为什么不用直接 mask forecast 作为主线
- transect-distance 表达
- Mamba + LSTM + calendar/gap/quality features
- 当前验证指标


### Slide 9. 这个模块对 DT 的意义

- 提供可重复的海岸状态监测能力
- 在统一参考框架下连接当前、历史和未来
- 给出可解释且带不确定性的输出
- 可以自然接入更大的 DT 工作流


### Slide 10. 当前阶段结论与下一步

- 已实现端到端在线主流程
- 已具备可训练、可更新的数据闭环
- 下一步重点是泛化、QC、评估指标和部署稳定性


## 17. 最终结论

当前项目已经从“研究脚本集合”演进为一条比较清晰的在线技术主线：

- 用 VE `RobustUNet` 自动初始化 AOI 固定几何；
- 用固定 `refline/transects` 约束历史与当前 VE 提取；
- 用 transect-distance 作为统一的时序状态表示；
- 用带时间、季节、质量特征的时序模型预测未来距离；
- 再把距离重建为可展示的未来 VE 主线与不确定性边界。

从 webinar / DT 演示表达上，最值得强调的不是“我们用了很多模型”，而是：

**我们把海岸变化问题重组为一个几何稳定、可解释、可在线运行、可持续更新、可继续迭代的数据与模型闭环。**

如果要让听众把它记成一个数字孪生组件，那么最理想的印象应该是：

**这个模块为 DT 提供了可用的海岸状态记忆与预测能力，而不只是一次性的海岸线提取结果。**
