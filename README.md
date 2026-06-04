# ECRO-Forecasting
# Private Power-System Dataset Description

本仓库中的私有电力系统数据集用于支持论文中的事件条件残差算子学习实验。该部分数据围绕电力系统中的两类典型预测任务构建：一类是用户侧用电量预测，包括居民用电量数据集（REC）和农业用电量数据集（AEC）；另一类是电力市场价格预测，包括日前电价数据集（DEP）和实时电价数据集（REP）。

这些数据集均来源于某省电力系统真实业务记录，能够反映用户侧用电需求、电力市场价格以及外部事件扰动下的非平稳变化特征。与此同时，我们针对上述时间序列构建了对应的事件文本数据集，用于刻画政策调控、经济活动、极端天气、节假日和市场运行状态等外部因素对电力系统未来演化的影响。

由于原始数据涉及真实电力运行、市场交易和用户侧统计信息，完整原始数据暂不公开。本仓库主要提供数据集说明、字段定义、处理流程、事件文本构建规则和匿名化样例，以便读者理解实验数据的构成方式和预处理逻辑。

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

DEP 和 REP 分别对应日前电价和实时电价数据集，来源于某省电力市场运行记录。数据覆盖时间范围为 2024 年 1 月 1 日至 2025 年 6 月 15 日，以 1 小时为采样频率记录，共包含 12624 个小时级样本。

DEP 数据集以日前电价为目标变量，用于反映电力市场在日前交易环节形成的价格变化；REP 数据集以实时电价为目标变量，用于反映电力系统在实际运行过程中受供需状态、市场交易、负荷波动和运行约束影响而形成的动态价格变化。

两个电价数据集均只包含一个核心目标变量，即对应时刻的电价。每条记录对应一个小时级时间戳及其电价观测值。数据覆盖工作日、周末、节假日、季节转换期和电力需求高峰期，能够反映日内波动、周周期变化、季节性变化以及市场异常波动等特征。

#### Data fields

| Field | Description |
|---|---|
| `timestamp` | Hourly timestamp |
| `price` | Electricity price at the corresponding hour |

#### Data processing

对于 DEP 和 REP，数据处理流程主要包括以下步骤：

1. **时间戳标准化**  
   将原始时间字段统一转换为标准时间戳格式，并按照小时频率构建连续时间索引。

2. **重复记录检查**  
   检查并删除重复时间戳记录，确保每个小时仅对应一个有效电价观测值。

3. **缺失值检查**  
   检查缺失小时记录。若发现缺失时间点，则结合原始业务记录进行核查；无法可靠恢复的缺失记录不参与实验窗口构建。

4. **异常值核查**  
   对极端高价或低价点进行识别和人工核查。对于具有合理市场含义的异常波动，例如供需紧张、交易波动、节假日负荷变化或极端天气影响，不进行简单删除，而是保留为反映非平稳市场状态的重要样本。

5. **时间顺序划分**  
   所有样本按照时间顺序划分训练集、验证集和测试集，避免未来信息泄露。

---

### 2.2 Electricity Consumption Datasets: REC and AEC

REC 和 AEC 分别对应居民用电量和农业用电量数据集，来源于某省电力公司用户侧用电统计记录。数据覆盖时间范围为 2023 年 1 月 1 日至 2025 年 12 月 31 日，以 1 天为采样频率记录，共包含 1096 条日级样本。

REC 数据集记录居民用户每日用电量，用于描述居民侧电力消费水平。居民用电通常受到气温变化、节假日、居民生活方式、极端天气和社会活动等因素影响。AEC 数据集记录农业用户每日用电量，用于描述农业生产相关用电需求。农业用电通常受到季节性生产周期、天气条件、灌溉需求、农事活动和相关政策等因素影响。

两个用电量数据集均以日级用电量作为核心目标变量。由于数据跨越多个完整年度，因此能够反映季节变化、节假日效应、天气变化以及不同年份之间的用电需求差异。

#### Data fields

| Field | Description |
|---|---|
| `date` | Daily date |
| `electricity_consumption` | Daily electricity consumption |

#### Data processing

对于 REC 和 AEC，数据处理流程主要包括以下步骤：

1. **日期标准化**  
   将原始日期字段统一转换为标准日期格式，并按照自然日构建连续时间索引。

2. **完整性检查**  
   检查每个自然日是否存在对应观测值，保证日级时间序列的连续性。

3. **重复记录检查**  
   对重复日期记录进行核查和去重，确保每个日期仅保留一个有效用电量观测值。

4. **异常波动核查**  
   对明显异常的用电跳变进行识别，并结合节假日、极端天气、气温变化和业务记录进行核查。对于具有合理业务含义的波动样本予以保留。

5. **时间顺序划分**  
   所有样本按照时间顺序划分训练集、验证集和测试集，确保模型训练和测试过程不引入未来信息。

---

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
