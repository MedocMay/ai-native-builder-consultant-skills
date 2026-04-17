---
name: ts-pipeline
description: 时序类任务（预测/异常检测/分类）的完整落地指南，涵盖数据准备、模型选择、训练评估到部署的全链条，基于 PaddleX 时序产线。适用于制造成本预测、设备异常检测、能耗预测等场景。
---

# TS Pipeline — 时序模型全链条落地

## 咨询链位置

**在新版 SOP 中：** 时序任务的具体落地 Skill，在 `dl-model-selection` 确认使用时序方向后启动。

**典型应用场景：**
- 制造成本趋势预测（材料成本、制造费用预测）
- 设备异常检测（传感器数据监控）
- 能耗预测（用电量、用水量预测）
- 质量指标预测（良率预测、缺陷率预测）

---

## 时序三类任务决策

```
业务问题
  ↓
"预测未来的值是多少？"
  → 时序预测（Time Series Forecasting）

"现在的数据是否异常？"
  → 时序异常检测（Time Series Anomaly Detection）

"这段时序属于哪个类别？"
  → 时序分类（Time Series Classification）
```

---

## Step 1：数据准备

### 标准数据格式

PaddleX 时序数据格式（CSV）：
```csv
date,target,feature1,feature2
2024-01-01 00:00:00,2162.0,25.3,0.8
2024-01-01 01:00:00,2835.0,25.1,0.9
2024-01-01 02:00:00,2764.0,24.9,0.85
```

**字段说明：**
- `date`：时间戳，必须单调递增，不能重复
- `target`：预测目标变量（预测任务必须）
- `feature1..N`：协变量（可选，用于多变量预测）

### 制造成本场景示例

```csv
date,material_cost,copper_price,aluminum_price,production_volume,month
2024-01-01,18500,68500,19200,1200,1
2024-02-01,19200,71000,19800,1350,2
2024-03-01,20100,72500,20100,1100,3
```

### 数据质量检查（必须通过）

```python
import pandas as pd

df = pd.read_csv("cost_data.csv", parse_dates=["date"])

# 检查1：时间戳是否单调递增
assert df["date"].is_monotonic_increasing, "时间戳必须单调递增"

# 检查2：是否有重复时间戳
assert df["date"].duplicated().sum() == 0, f"发现重复时间戳：{df[df['date'].duplicated()]}"

# 检查3：缺失值情况
print("缺失值统计：")
print(df.isnull().sum())

# 检查4：时间间隔是否均匀
gaps = df["date"].diff().dropna()
print(f"时间间隔：最小={gaps.min()}, 最大={gaps.max()}, 最多见={gaps.mode()[0]}")
```

### 缺失值处理策略

```python
# 时序数据缺失值处理（不能随意删除！）
# 方法1：线性插值（短期缺失，< 3个点）
df["target"] = df["target"].interpolate(method="linear")

# 方法2：前向填充（传感器数据，值变化慢）
df["target"] = df["target"].fillna(method="ffill")

# 方法3：标记缺失（缺失超过10%，保留标记）
df["target_missing"] = df["target"].isnull().astype(int)
df["target"] = df["target"].interpolate()
```

### 训练/验证/测试集划分（必须按时间，不能随机）

```python
n = len(df)
train_end = int(n * 0.7)
val_end = int(n * 0.85)

train = df.iloc[:train_end]
val = df.iloc[train_end:val_end]
test = df.iloc[val_end:]

# 保存
train.to_csv("./dataset/train.csv", index=False)
val.to_csv("./dataset/val.csv", index=False)
test.to_csv("./dataset/test.csv", index=False)
```

---

## Step 2：数据校验

```bash
python main.py -c paddlex/configs/modules/ts_forecast/DLinear.yaml \
    -o Global.mode=check_dataset \
    -o Global.dataset_dir=./dataset
```

校验通过输出：`Check dataset passed!`

---

## Step 3：模型选择和训练

### 模型快速体验

```python
from paddlex import create_model

# 时序预测
model = create_model("DLinear")
output = model.predict("./dataset/test.csv", batch_size=1)

# 时序异常检测
model = create_model("PatchTSAD")
output = model.predict("./dataset/test.csv", batch_size=1)
```

### 关键超参数配置

```yaml
# 时序预测配置（DLinear.yaml 关键参数）
Train:
  epochs_iters: 20      # 训练轮数，小数据集可用20-50
  batch_size: 16
  learning_rate: 0.0001

Predict:
  batch_size: 1

Global:
  input_len: 96    # 输入序列长度（用多长的历史预测未来）
  output_len: 96   # 预测序列长度（预测多远的未来）

  # 关键：制造成本月度预测场景
  # input_len: 12  # 用过去12个月
  # output_len: 3  # 预测未来3个月
```

### 启动训练

```bash
python main.py -c paddlex/configs/modules/ts_forecast/DLinear.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./dataset \
    -o Global.output_dir=./output \
    -o Train.epochs_iters=20 \
    -o Global.input_len=12 \
    -o Global.output_len=3
```

---

## Step 4：模型评估

```bash
python main.py -c paddlex/configs/modules/ts_forecast/DLinear.yaml \
    -o Global.mode=evaluate \
    -o Global.dataset_dir=./dataset
```

### 评估指标解读

**时序预测指标：**
| 指标 | 含义 | 目标 |
|------|------|------|
| MSE（均方误差） | 预测值与真实值的平方差均值 | 越小越好 |
| MAE（平均绝对误差） | 预测值与真实值的绝对差均值 | 越小越好 |
| MAPE（平均绝对百分比误差） | 相对误差，业务更直观 | < 10% 较好 |

**制造成本预测场景的业务指标转换：**
```python
# MAE = 500元 意味着：每月预测误差平均±500元
# 如果月成本是20000元，MAPE = 500/20000 = 2.5%
# 2.5% 的误差在成本预测中通常可接受
```

**时序异常检测指标：**
| 指标 | 含义 | 目标 |
|------|------|------|
| Precision（精确率） | 报警中真实异常的比例 | > 80%（避免误报） |
| Recall（召回率） | 真实异常中被发现的比例 | > 90%（不能漏报） |
| F1 | Precision 和 Recall 的综合 | > 85% |

---

## Step 5：与大模型集成

### 时序预测 + 大模型分析

```python
from paddlex import create_model
import json

# 小模型：时序预测
ts_model = create_model("DLinear")
forecast = ts_model.predict("latest_data.csv")

# 提取预测结果
predictions = {
    "next_3_months": [f.tolist() for f in forecast],
    "model_version": "DLinear_v1.2",
    "input_period": "2024-01 to 2024-12"
}

# 大模型：生成分析报告
prompt = f"""
你是制造成本分析专家。以下是时序预测模型对未来3个月制造成本的预测结果：

预测数据：{json.dumps(predictions, ensure_ascii=False)}
历史趋势：成本在过去12个月平均为 {historical_avg} 元，波动率 {volatility}%
市场因素：铜价近期 {copper_trend}，铝价 {aluminum_trend}

请分析：
1. 未来3个月的成本趋势判断（上升/下降/平稳）
2. 主要影响因素（2-3个）
3. 是否存在成本优化空间（具体指出哪个环节）
4. 风险提示

以结构化格式输出。
"""

llm_analysis = llm_client.chat(prompt)
```

### 时序异常检测 + 大模型告警

```python
# 小模型：异常检测
anomaly_model = create_model("PatchTSAD")
anomalies = anomaly_model.predict("sensor_data.csv")

# 过滤高置信度异常
high_conf_anomalies = [a for a in anomalies if a["anomaly_score"] > 0.8]

if high_conf_anomalies:
    # 大模型：生成告警说明
    prompt = f"""
    设备传感器监控系统检测到以下异常：
    {json.dumps(high_conf_anomalies, ensure_ascii=False)}
    
    设备信息：{equipment_info}
    最近维保记录：{maintenance_history}
    
    请判断：
    1. 异常严重程度（低/中/高/紧急）
    2. 可能的故障原因
    3. 建议处置方案（继续监控/计划维修/立即停机）
    4. 预计影响范围
    """
    alert_content = llm_client.chat(prompt)
    send_alert(alert_content)
```

---

## Step 6：部署

### 服务化部署

```bash
# 启动时序预测服务
paddlex --serve \
    --pipeline ts_forecast \
    --model_path ./output/best_model \
    --port 8081 \
    --device cpu  # 时序模型 CPU 即可，无需 GPU
```

### Python API 集成

```python
from paddlex import create_pipeline

# 创建时序预测产线
ts_pipeline = create_pipeline(pipeline="ts_forecast")
ts_pipeline.load_model("./output/best_model")

def predict_cost(recent_data_df):
    """
    输入：最近N个月的成本数据（DataFrame）
    输出：未来3个月预测值
    """
    result = ts_pipeline.predict(recent_data_df)
    return {
        "predictions": result.tolist(),
        "confidence_interval": calculate_ci(result)  # 可选
    }
```

---

## 制造成本预测场景的完整配置建议

```yaml
# 制造成本月度预测专项配置

数据要求：
  最少数据量: 24个月（2年）
  推荐数据量: 36个月以上
  采样频率: 月度（不混用日度数据）
  必须包含的字段:
    - date（月度时间戳）
    - total_material_cost（总材料成本）
    - copper_usage（铜用量）
    - aluminum_usage（铝用量）
  可选字段:
    - production_volume（产量，重要协变量）
    - copper_market_price（市场铜价）
    - aluminum_market_price（市场铝价）

模型配置:
  推荐模型: PatchTST（精度优先）或 DLinear（简单场景）
  input_len: 12  # 用过去12个月预测
  output_len: 3  # 预测未来3个月
  评估指标: MAPE < 8%（成本预测业务可接受范围）

再训练策略:
  触发条件: 实际成本连续3个月偏离预测 > 15%
  再训练方式: 增量训练（加入新数据，不从头训练）
  验证: 在最近3个月数据上评估，MAPE需 < 8%
```
