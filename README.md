# ECRO-Forecasting
# Private Power-System Dataset Description

本仓库中的私有电力系统数据集用于支持论文中的事件条件残差算子学习实验。该部分数据围绕电力系统中的两类典型预测任务构建：一类是用户侧用电量预测，包括居民用电量数据集（REC）和农业用电量数据集（AEC）；另一类是电力市场价格预测，包括日前电价数据集（DEP）和实时电价数据集（REP）。

这些数据集均来源于某省电力系统真实业务记录，能够反映用户侧用电需求、电力市场价格以及外部事件扰动下的非平稳变化特征。与此同时，我们针对上述时间序列构建了对应的事件文本数据集，用于刻画政策调控、经济活动、极端天气、节假日和市场运行状态等外部因素对电力系统未来演化的影响。

本仓库主要提供数据集说明、字段定义、处理流程、事件文本构建规则和匿名化样例，以便读者理解实验数据的构成方式和预处理逻辑。

---

## 1. Dataset Overview

私有电力系统数据集共包含四个子数据集，具体统计信息如下。

| Dataset | Full Name | Frequency | Target Variable | Timespan | Number of Samples | Data Type |
|---|---|---|---|---|---:|---|
| REC | Residential Electricity Consumption | Daily | Residential electricity consumption | 2023-01-01 to 2025-12-31 | 1096 | Private |
| AEC | Agricultural Electricity Consumption | Daily | Agricultural electricity consumption | 2023-01-01 to 2025-12-31 | 1096 | Private |
| DEP | Day-ahead Electricity Price | Hourly | Day-ahead electricity price | 2024-01-01 to 2025-06-15 | 12624 | Private |
| REP | Real-time Electricity Price | Hourly | Real-time electricity price | 2024-01-01 to 2025-06-15 | 12624 | Private |

其中，REC 和 AEC 用于刻画不同用户类型的日级用电需求变化，DEP 和 REP 用于刻画电力市场中的小时级价格波动。四个数据集均具有明显的时间依赖性、周期性和非平稳性，并可能受到政策、天气、节假日、经济活动和市场供需变化等外部事件影响。

---
## 2. Numerical Time-Series Data

### 2.1 Electricity Price Datasets: DEP and REP

DEP 和 REP 分别对应日前电价数据集和实时电价数据集，均来源于某省电力市场运行记录。两个数据集采用相同的数据组织方式和预处理流程，用于刻画电力市场价格在不同时间段、不同市场状态和外部事件影响下的动态变化。

#### 1. 数据来源与采集

DEP 数据集来源于某省日前电价记录，REP 数据集来源于某省实时电价记录。两个数据集均覆盖 2024 年 1 月 1 日至 2025 年 6 月 15 日，以 1 小时为采样频率记录，共计 12624 个小时级样本。原始数据由电力交易相关业务系统生成，能够连续反映电力市场在不同日期、不同时段和不同运行状态下的价格变化。

其中，DEP 反映日前交易环节形成的电价变化，REP 反映实际运行过程中受供需状态、市场交易、负荷波动和运行约束影响形成的实时价格变化。两个数据集均为小时级单变量时间序列，适用于电力市场价格预测任务。

#### 2. 数据特征

DEP 和 REP 均只包含一个核心目标变量，即对应时刻的电价。每条记录对应一个小时级时间戳及其电价观测值。

| Field | Description |
|---|---|
| `timestamp` | Hourly timestamp |
| `price` | Electricity price at the corresponding hour |

两个电价数据集时间序列完整、连续，能够反映电力市场价格的日内波动、周周期变化、季节性变化以及异常市场波动。数据覆盖工作日、周末、节假日、季节性过渡期和电力需求高峰期，因此能够较好地反映不同时间段下的电力市场动态。

#### 3. 数据处理与质量控制

为了保证数据的可用性和研究价值，DEP 和 REP 均经过统一的数据预处理和质量控制。首先，对原始时间字段进行标准化处理，并按照小时频率构建连续时间索引；其次，对重复时间戳、缺失小时和异常记录进行检查，保证每个小时仅对应一个有效电价观测值。

对于极端高价或低价点，不进行简单删除，而是结合电力市场运行特征进行核查。若异常波动具有合理市场含义，例如供需紧张、交易波动、节假日负荷变化或极端天气影响，则予以保留，用于后续建模时反映市场异常波动和非平稳变化。经过上述处理后，DEP 和 REP 保持了较好的连续性和数据质量，可用于小时级电价预测实验。

---

### 2.2 Electricity Consumption Datasets: REC and AEC

REC 和 AEC 分别对应居民用电量数据集和农业用电量数据集，均来源于某省电力公司用户侧用电统计记录。两个数据集采用相同的数据组织方式和预处理流程，用于刻画不同用户类型在季节变化、节假日、天气条件和社会活动影响下的日级用电需求变化。

#### 1. 数据来源与采集

REC 数据集记录某省居民用户的日用电量情况，AEC 数据集记录某省农业用户的日用电量情况。两个数据集均来源于该省电力公司用户侧用电统计记录，并结合气象部门公开温度数据进行整理。数据覆盖 2023 年 1 月 1 日至 2025 年 12 月 31 日，以 1 天为采样频率记录，共计 1096 条日级样本。

REC 能够连续反映居民用户在不同时间段的电力消费变化，AEC 能够连续反映农业生产相关用电需求的变化。结合公开温度信息后，两个数据集可用于分析气象因素、季节变化和节假日因素对用户侧用电需求的影响。

#### 2. 数据特征

REC 和 AEC 均包含一个核心目标变量，即日用电量。REC 的目标变量为居民日用电量，用于描述居民用户每日的电力消费水平；AEC 的目标变量为农业日用电量，用于描述农业生产相关用户每日的用电需求。

| Field | Description |
|---|---|
| `date` | Daily date |
| `electricity_consumption` | Daily electricity consumption |
| `temperature` | Daily temperature information from public meteorological data |

由于数据跨越多个完整年度，REC 能够反映居民用电在季节变化、节假日、气温变化和居民生活行为变化下的波动规律；AEC 能够反映农业用电在季节性生产周期、天气条件、灌溉需求、农事活动和相关政策影响下的变化规律。

#### 3. 数据处理与质量控制

为了保证数据的可用性和可靠性，对 REC 和 AEC 进行了必要的数据预处理和质量控制。首先，对原始日期字段进行标准化处理，并按照自然日构建连续时间索引；其次，对时间序列完整性进行检查，确保每个自然日均有对应观测值；再次，对重复日期记录和明显异常记录进行识别与核查，避免重复样本或录入错误影响后续建模。

对于个别异常用电波动，不进行机械删除，而是结合节假日、极端天气、温度变化和业务背景进行判断。若异常波动具有合理现实含义，则予以保留，用于反映真实用电需求中的非平稳变化。经过上述处理后，REC 和 AEC 保持了较高的数据质量，可用于日级用电量预测实验。

## 3. Event Text Data

为了支持文本事件驱动的非平稳时间序列预测研究，我们为 REC、AEC、DEP 和 REP 构建了对应的事件文本数据集。相关新闻、公告和公开信息主要来源于人民网、山东省人民政府、中国山东网等公开渠道，内容覆盖可能影响电力系统运行和电力需求变化的外部事件。

事件文本主要包括以下类型：

- 政策调控事件，例如能源政策、电力市场规则调整、节能降耗政策等；
- 经济活动事件，例如产业生产、消费活动、区域经济运行变化等；
- 极端天气事件，例如高温、寒潮、强降雨、台风和气象预警等；
- 节假日事件，例如春节、劳动节、国庆节、端午节等；
- 社会活动事件，例如大型活动、出行高峰和公共事件等；
- 能源供需事件，例如电力供需变化、能源结构调整和市场交易波动等。

原始文本并不直接输入预测模型，而是经过文本到事件映射流程转换为结构化事件变量。该处理过程可以降低开放文本中的噪声和重复信息，使外部事件能够与时间序列预测窗口建立更清晰的语义对应关系。

---

## 4. Text-to-Event Structuring

每条有效事件被转换为结构化事件变量，主要字段如下。

| Field | Description |
|---|---|
| `title` | Normalized event title |
| `publish_time` | Publication time of the event |
| `direction` | Potential impact direction on the target variable |
| `intensity` | Potential impact intensity |
| `duration` | Potential impact duration |
| `factor` | Main event factor |
| `rationale` | Causal explanation of the event impact |

字段含义说明如下：

- `direction` 表示事件对目标变量的可能影响方向，通常包括 `positive` 和 `negative`；
- `intensity` 表示事件可能引发扰动的相对强度，通常包括 `high`、`medium` 和 `low`；
- `duration` 表示事件影响的持续周期，通常包括 `short-term`、`medium-term` 和 `long-term`；
- `factor` 表示事件影响目标变量的主要因素，例如 `policy`、`weather`、`holiday`、`economy`、`market` 等；
- `rationale` 用于说明事件影响目标变量的因果路径。

---

## 5. Event Text Processing Pipeline

事件文本处理流程包括以下步骤。

### Step 1: Event collection

从公开新闻、政府公告和权威媒体报道中收集可能影响电力需求或电力价格的事件文本。文本来源包括人民网、山东省人民政府、中国山东网等公开渠道。

### Step 2: Task relevance filtering

根据预测目标、发布时间、区域范围和事件内容筛除明显无关文本。仅保留可能通过用户行为、市场状态、气象条件、政策调整或经济活动影响目标序列的文本。

### Step 3: Event clustering and deduplication

开放文本中常存在重复报道、转载新闻和相似公告。为避免同一事件被重复输入模型，我们对候选文本进行聚类和去重，将多条相似文本合并为一个候选事件，并选择信息较完整、表达较清晰的文本作为代表文本。

### Step 4: Semantic structuring

对候选事件进行语义结构化，提取影响方向、影响强度、影响周期、影响因素和因果解释等字段。该步骤的目标不是抽取孤立关键词，而是将开放文本转化为可用于事件响应建模的结构化语义条件。

### Step 5: Temporal alignment

根据事件发布时间，将结构化事件对齐到对应预测起点之前。模型只允许使用预测时刻之前已经公开的事件信息，从而避免未来信息泄露。

---

## 6. Examples of Event Structuring

以下示例用于说明原始事件文本如何被转换为结构化事件变量。示例均为匿名化展示，仅用于说明处理逻辑，不包含完整原始数据。

### Example 1: Extreme Weather Event for Residential Electricity Consumption

Raw text:

> 山东省气象部门发布高温预警，预计未来多地将持续出现高温天气，居民制冷需求可能明显增加。

Structured event:

```json
{
  "title": "山东多地发布持续高温预警",
  "publish_time": "2024-07-10 09:00:00",
  "direction": "positive",
  "intensity": "high",
  "duration": "short-term",
  "factor": "weather",
  "rationale": "持续高温可能提升居民空调用电需求，从而增加短期居民用电量。"
}

由于原始数据涉及真实电力运行、市场交易和用户侧统计信息，出于隐私考虑，完整原始数据暂不公开。通过联系通讯作者可获得去识别化样本，仅限于学术研究和重复研究。
