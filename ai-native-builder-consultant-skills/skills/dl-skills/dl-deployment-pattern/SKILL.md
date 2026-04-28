---
name: dl-deployment-pattern
description: 为深度学习模型选择部署方式（高性能推理/服务化部署/端侧部署），设计与大模型的集成接口，规划上线后的模型版本管理和再训练机制。基于 PaddleX 三种部署模式设计。
---

# DL Deployment Pattern — 深度学习模型部署范式

## 咨询链位置

**在新版 SOP 中：** 深度学习范式的 Module 4+5，在模型训练和评估完成后使用。

**联动：**
- `dl-model-selection` — 部署决策基于模型选型结果
- `launch-risk-review` — DL 系统也需要上线风险评审，关注点不同
- `iteration-flywheel` — DL 系统的迭代是数据+模型双飞轮

---

## PaddleX 三种部署模式

### 模式 1：高性能推理（High-Performance Inference）

**适用场景：**
- 云端服务器部署，追求极致推理速度
- 对延迟有严格要求（< 100ms）
- 有 GPU 且希望充分利用 GPU 性能

**核心能力：**
- 自动进行模型量化（FP32 → FP16/INT8）
- TensorRT 加速（NVIDIA GPU）
- 端到端流程优化（前处理 + 推理 + 后处理）

```python
from paddlex import create_pipeline

pipeline = create_pipeline(
    pipeline="image_classification",
    device="gpu",
    use_hpip=True  # 启用高性能推理
)
output = pipeline.predict("input.jpg")
```

**安装：**
```bash
paddlex --install hpi-gpu  # GPU版（包含CPU版功能）
paddlex --install hpi-cpu  # 仅CPU版
```

---

### 模式 2：服务化部署（Serving）

**适用场景：**
- 多个业务系统需要调用同一个模型
- 需要负载均衡和水平扩展
- 与现有 API 体系集成

**架构：**
```
客户端（业务系统）
    ↓ HTTP/gRPC
推理服务（PaddleX Serving）
    ↓
模型推理
    ↓
结构化 JSON 结果返回
```

**启动服务：**
```bash
paddlex --serve --pipeline image_classification --device gpu:0 --port 8080
```

**调用示例：**
```python
import requests, base64

with open("input.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

response = requests.post(
    "http://localhost:8080/image-classification",
    json={"image": img_b64}
)
result = response.json()
```

---

### 模式 3：端侧部署（On-Device）

**适用场景：**
- 工厂现场，网络不稳定或无网络
- 实时性要求极高（< 30ms）
- 摄像头、工控机、嵌入式设备

**支持设备：**
- Android 手机/平板
- 工控机（x86 CPU / ARM）
- 昆仑芯 XPU / 华为昇腾 NPU

**关键步骤：**
1. 模型导出为推理格式（Paddle Inference / ONNX）
2. 模型量化（INT8，减小模型体积 4x）
3. 平台适配（TensorRT / Paddle Lite）

```bash
# 导出推理模型
python main.py -c config.yaml -o Global.mode=export -o Global.save_inference_dir=./inference_model

# 模型量化（可选，大幅减小体积）
paddlex --pipeline image_classification --export --quant_type int8
```

---

## 大小模型集成接口设计

**核心原则：小模型输出必须是结构化的，大模型才能稳定消费。**

### 标准集成接口

```python
# 小模型（感知层）输出格式
cv_result = {
    "task_id": "inspect_20260413_001",
    "input_path": "product_001.jpg",
    "timestamp": "2026-04-13T10:30:00",
    "detections": [
        {
            "label": "crack",           # 缺陷类别
            "score": 0.94,             # 置信度
            "bbox": [120, 80, 200, 160] # 位置
        }
    ],
    "model_version": "defect_v2.1",
    "inference_time_ms": 45
}

# 大模型（理解层）输入 Prompt 模板
prompt_template = """
你是工业质检分析专家。以下是视觉检测系统对产品 {product_id} 的检测结果：

检测发现：
{detections}

请基于以下上下文进行分析：
- 产品类型：{product_type}
- 生产工序：{process_stage}
- 历史缺陷记录：{history}

请输出：
1. 缺陷严重程度判断（轻微/中等/严重）
2. 可能的产生原因（2-3个）
3. 建议处置方案
4. 是否需要停线检查（是/否，原因）

以 JSON 格式输出。
"""
```

### 接口稳健性设计

```python
def integrate_cv_with_llm(cv_result: dict, llm_client) -> dict:
    """
    小模型 → 大模型集成，包含完整错误处理
    """
    # 1. 验证小模型输出格式
    if not cv_result.get("detections"):
        return {"status": "no_defect", "action": "pass"}

    # 2. 过滤低置信度结果（阈值由业务决定，建议 > 0.7）
    high_conf = [d for d in cv_result["detections"] if d["score"] > 0.7]
    if not high_conf:
        return {"status": "uncertain", "action": "manual_review"}

    # 3. 调用大模型
    try:
        prompt = format_prompt(cv_result, high_conf)
        llm_response = llm_client.chat(prompt)
        result = parse_json_response(llm_response)
        return {"status": "analyzed", "result": result}
    except Exception as e:
        # 大模型失败时降级到规则判断
        return fallback_rule_judgment(high_conf)
```

---

## 模型版本管理和再训练机制

### 版本管理规范

```
模型命名规范：
{任务名}_{版本}_{训练日期}
示例：defect_detection_v2.1_20260413

目录结构：
models/
  ├── production/          # 当前生产模型
  │   └── defect_v2.1/
  ├── staging/             # 待上线模型
  │   └── defect_v2.2/
  ├── archive/             # 历史模型（保留6个月）
  │   └── defect_v1.x/
  └── registry.json        # 版本注册表（模型→精度→上线时间）
```

### 再训练触发条件

**主动触发（定期）：**
- 每积累 500 条新标注数据，评估是否需要再训练
- 每季度至少做一次模型性能评审

**被动触发（事件驱动）：**

| 信号 | 触发条件 | 行动 |
|------|---------|------|
| 精度下降 | 验证集精度连续3天低于基线5% | 立即启动数据调查 + 再训练 |
| 分布偏移 | 新数据与训练数据分布差异显著 | 增量标注新分布数据 + 微调 |
| 新类别出现 | 业务新增缺陷类型 | 采集新类别数据 + 增量训练 |
| 部署环境变化 | 摄像头更换、光照条件改变 | 在新环境下采集数据 + 微调 |

### AB 测试发布策略

```
新模型上线前：
1. 在 Staging 环境用历史数据评估（精度 ≥ 生产模型）
2. 影子模式运行 48 小时（新模型并行推理，不影响生产）
3. 对比新旧模型的不一致案例，人工判断哪个对
4. 新模型胜出率 ≥ 60% → 切换；< 60% → 继续优化
```

---

## 上线风险检查清单（DL 版本）

```markdown
### 模型行为
- [ ] 验证集精度 ≥ 目标值（见模型选型文档）
- [ ] 边界案例单独测试通过（容易混淆的类别）
- [ ] 对抗样本测试（模糊图片、极端光照、遮挡）
- [ ] 推理延迟 p95 满足业务要求

### 部署稳定性
- [ ] 服务化部署健康检查通过
- [ ] 并发压力测试通过（模拟峰值负载）
- [ ] 内存/显存占用在安全范围内
- [ ] 模型文件版本和校验和记录

### 大小模型集成
- [ ] 小模型低置信度时的降级处理已验证
- [ ] 大模型调用超时的 fallback 已验证
- [ ] 接口格式变更的向后兼容性确认

### 监控
- [ ] 推理延迟监控已就位
- [ ] 低置信度比例监控已就位（>20% 触发告警）
- [ ] 模型版本标记在每条推理日志中

### 回滚
- [ ] 上一个版本的生产模型已备份
- [ ] 回滚操作已测试（< 10 分钟完成切换）
```

---

## 输出格式

```markdown
## DL 部署方案

**产线：** [产线名] **模型版本：** [版本号] **部署日期：** [日期]

### 部署模式选择
- 主要部署方式：高性能推理 / 服务化 / 端侧
- 理由：[基于延迟要求、环境约束、并发量]
- 硬件：[GPU/CPU/NPU型号]
- 目标延迟：p95 < [X] ms

### 大小模型集成
- 小模型输出格式：[JSON Schema]
- 与大模型的接口：[Prompt 模板摘要]
- 降级策略：[大模型不可用时的 fallback]

### 再训练计划
- 数据积累触发条件：[N 条新标注数据]
- 精度下降触发条件：[精度低于基线 X%]
- 再训练周期预估：[X 天]

### 版本管理
- 模型存储路径：[路径]
- 版本命名规范：[规范]
- 回滚时间目标：< [X] 分钟
```
