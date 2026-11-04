# 香港道路交通事故严重程度预测：数据包说明

## 推荐研究题目

**Interpretable Prediction of Severe Road Traffic Accidents in Hong Kong: A Temporal Validation Study (2024–2025)**

中文：**香港严重道路交通事故的可解释预测：基于 2024–2025 年数据的时间外验证**

研究问题：在事故发生时已经能够观察到的道路、环境、碰撞和车辆信息，能否识别最终造成严重或致命后果的事故？哪些因素与较高的严重事故风险相关？

## 数据来源

- 香港运输署 2024 年道路交通事故统计：https://www.td.gov.hk/en/road_safety/road_traffic_accident_statistics/2024/index.html
- 香港运输署 2025 年道路交通事故统计：https://www.td.gov.hk/en/road_safety/road_traffic_accident_statistics/2025/index.html

`raw_xls/` 保存运输署原始下载文件。`converted_xlsx/` 是为方便 Python、Excel 和其他分析工具读取而生成的 XLSX 副本。

每年的主数据库都包含三张表：

- `accidentYYYY`：一行代表一宗事故。
- `vehicleYYYY`：一行代表一辆涉事车辆，同一事故可能有多行。
- `casualtyYYYY`：一行代表一名伤亡者，同一事故可能有多行。

代码表 `YYcodee` 用来把数字或字母代码翻译成实际类别。

## 初步数据审计

| 年份 | 事故 | 涉事车辆记录 | 伤亡者记录 | 致命 | 严重 | 轻微 |
|---|---:|---:|---:|---:|---:|---:|
| 2024 | 18,354 | 31,315 | 23,656 | 83 (0.5%) | 751 (4.1%) | 17,520 (95.5%) |
| 2025 | 17,320 | 29,595 | 21,815 | 92 (0.5%) | 467 (2.7%) | 16,761 (96.8%) |

事故表没有缺失值。车辆表的 `Year of Manufacture` 在 2024 年缺失 7.3%，2025 年缺失 8.3%。伤亡者表的 `Pedestrian Action` 只适用于行人，因此大量空值属于结构性缺失，不应直接当成数据质量错误。

## 建议的建模定义

- 分析单位：事故。
- 目标变量：`severe = 1` 当 `Severity` 为 1（Fatal）或 2（Serious）；否则为 0。
- 训练与验证：在 2024 年数据内划分训练集和验证集。
- 最终测试：完整保留 2025 年数据作为时间外测试集。
- 基线模型：始终预测轻微事故、加权逻辑回归。
- 候选模型：随机森林或梯度提升树。
- 主要评价指标：PR-AUC、严重事故 Recall、F1、Balanced Accuracy；不要把 Accuracy 作为主要指标。

## 可直接使用的事故层面特征

- District Council District
- Hit and Run
- Weather
- Rain
- Natural Light
- Junction Control
- Road Classification
- Vehicle Movements
- Type of Collision

车辆表可以按 `Serial` 聚合后加入驾驶者年龄、性别、车辆类别及车龄等特征。

## 必须避免的数据泄漏

如果目标是预测严重程度，下列字段会直接或间接暴露事故结果，主模型应排除：

- `vehicleYYYY` 中的 `Severity of Accident`
- `casualtyYYYY` 中的 `Degree of Injury`
- `casualtyYYYY` 中的 `Location of Injury`
- `accidentYYYY` 中的 `No. of Casualties`，若研究定位为事故早期风险识别

伤亡者表适合做结果描述和亚组分析，不宜直接作为严重程度预测的输入。

## 下一步

1. 读取事故表和代码表。
2. 将代码翻译成类别名称。
3. 创建二元目标 `severe`。
4. 按事故编号聚合车辆表。
5. 将车辆特征左连接到事故表。
6. 处理未知值与类别不平衡。
7. 先完成 EDA 和逻辑回归基线，再训练树模型。

