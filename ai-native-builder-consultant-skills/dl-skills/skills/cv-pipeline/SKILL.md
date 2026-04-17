---
name: cv-pipeline
description: CV 类任务（图像分类/目标检测/语义分割/异常检测）的完整落地指南，基于 PaddleX 全链条开发范式，涵盖从数据准备、模型选择、训练评估到部署的全流程。适用于工业质检、缺陷检测、目标计数等场景。
---

# CV Pipeline — 计算机视觉全链条落地

## 咨询链位置

**在新版 SOP 中：** CV 任务的具体落地 Skill，在 `dl-model-selection` 确认 CV 方向后启动。

**典型场景：**
- 工业质检：产品外观缺陷检测（划痕、裂纹、异物）
- 目标检测：生产线上零件计数、位置检测
- 图像分类：产品类别自动识别、不良品分类
- 异常检测：无标注异常样本的质检（用正常样本训练）
- 语义分割：缺陷区域精确定位

---

## Step 1：CV 任务类型选择

### 任务决策树

```
业务问题
├── "这张图是哪个类别？"（整图分类）
│   → 图像分类
├── "图中有什么物体，在哪里？"（检测+定位）
│   → 目标检测
├── "图中每个像素属于什么类别？"（精确区域）
│   → 语义分割 / 实例分割
└── "图中有没有异常？"（质检场景）
    ├── 异常样本稀少、类型多变 → 图像异常检测（无监督）
    └── 异常类型固定、样本充足 → 目标检测 / 图像分类（有监督）
```

### 工业质检场景的关键选择

**无监督异常检测（STFPM）优先使用条件：**
- 只有正常样本，异常样本极少
- 异常类型多样、不可预期
- 对精确定位要求不高（知道"有异常"即可）

**有监督检测优先使用条件：**
- 有足够异常样本（每类 > 100 张）
- 需要精确定位异常位置和类别
- 异常类型固定且可预期

---

## Step 2：快速体验预训练模型

```bash
# 图像分类
paddlex --pipeline image_classification \
        --input product.jpg \
        --device gpu:0

# 目标检测
paddlex --pipeline object_detection \
        --input scene.jpg \
        --device gpu:0

# 异常检测（质检场景）
paddlex --pipeline anomaly_detection \
        --input product.jpg \
        --save_path ./output \
        --device gpu:0
```

```python
# Python API
from paddlex import create_pipeline

# 异常检测
pipeline = create_pipeline(pipeline="anomaly_detection")
result = pipeline.predict("product.jpg")
for res in result:
    print(f"异常分数: {res['anomaly_score']}")  # 越高越异常
    # res['pred_mask'] 是像素级异常热力图
```

---

## Step 3：数据准备

### 图像分类数据格式

```
dataset/
├── train/
│   ├── normal/        # 正常品
│   │   ├── img001.jpg
│   │   └── img002.jpg
│   └── defect_crack/  # 裂纹缺陷
│       ├── img003.jpg
│       └── img004.jpg
├── val/
│   ├── normal/
│   └── defect_crack/
└── label.txt  # 类别定义：0 normal\n1 defect_crack
```

### 目标检测数据格式（COCO）

```json
{
  "images": [{"id": 1, "file_name": "img001.jpg", "width": 640, "height": 480}],
  "annotations": [
    {
      "id": 1,
      "image_id": 1,
      "category_id": 1,
      "bbox": [120, 80, 60, 40],
      "area": 2400,
      "iscrowd": 0
    }
  ],
  "categories": [{"id": 1, "name": "crack"}, {"id": 2, "name": "scratch"}]
}
```

### 异常检测数据格式（只需正常样本）

```
dataset/
├── train/
│   └── good/          # 只放正常样本
│       ├── img001.jpg
│       └── img002.jpg
└── test/
    ├── good/          # 测试集正常样本
    └── defect/        # 测试集异常样本（评估用）
        └── crack/
            └── img003.jpg
```

### 数据采集要点（工业场景）

```
摄像头固定要求：
✅ 固定拍摄角度和距离（避免透视变化）
✅ 稳定光源（LED环形灯，避免自然光变化）
✅ 固定曝光和白平衡参数
✅ 分辨率统一（建议 640×640 或 1024×1024）

数据覆盖要求：
✅ 正常样本：覆盖不同批次、不同材质颜色变化
✅ 异常样本（有监督）：每种缺陷类型至少100张
✅ 边界案例：轻微缺陷（人工难以判断的）单独归类

禁止事项：
❌ 不要用合成的缺陷图替代真实缺陷图（纹理不真实）
❌ 不要在异常样本极少时强行做有监督分类
❌ 不要用不同型号摄像头拍摄的图像混在一起训练
```

### 标注工具

```bash
# LabelMe（目标检测/语义分割）
pip install labelme
labelme  # 启动标注工具

# CVAT（支持团队协作，多人标注）
# 推荐在线使用：app.cvat.ai

# PaddleX 支持的标注格式转换
python main.py -c config.yaml \
    -o Global.mode=check_dataset \
    -o CheckDataset.convert.enable=True \
    -o CheckDataset.convert.src_dataset_type=LabelMe  # 从 LabelMe 格式转换
```

---

## Step 4：模型训练

### 图像分类

```bash
python main.py -c paddlex/configs/modules/image_classification/PP-HGNetV2-B4.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./dataset \
    -o Train.epochs_iters=100 \
    -o Train.batch_size=32 \
    -o Train.learning_rate=0.001 \
    -o Global.output_dir=./output
```

### 目标检测

```bash
python main.py -c paddlex/configs/modules/object_detection/PicoDet-L.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./dataset \
    -o Train.epochs_iters=300 \
    -o Train.batch_size=16 \
    -o Train.learning_rate=0.001
```

### 异常检测（STFPM）

```bash
python main.py -c paddlex/configs/modules/image_anomaly_detection/STFPM.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./dataset \
    -o Train.epochs_iters=100 \
    -o Train.batch_size=4
```

### 训练技巧

```yaml
# 小数据集（< 500张）的训练配置建议
Train:
  epochs_iters: 50        # 减少训练轮数，避免过拟合
  learning_rate: 0.0001   # 降低学习率
  batch_size: 8           # 小 batch size
  # 增强数据增强来弥补数据量不足
  transforms:
    - type: RandomFlip
    - type: RandomRotate
      max_rotation: 15
    - type: RandomDistort
    - type: MixupImage    # MixUp 数据增强（分类任务有效）

# 迁移学习（从预训练模型微调）
Global:
  pretrain_weights: "IMAGENET"  # 使用 ImageNet 预训练权重
```

---

## Step 5：模型评估

```bash
python main.py -c config.yaml \
    -o Global.mode=evaluate \
    -o Global.dataset_dir=./dataset
```

### 业务指标对应

**质检场景的指标优先级：**

```
Recall（召回率）> Precision（精确率）

理由：漏检（将缺陷品误判为正常品）的代价 >> 误检（将正常品误判为缺陷品）的代价

实际业务阈值设定：
- Recall ≥ 99%（核心红线，不能漏检缺陷品）
- Precision ≥ 80%（控制误检率，避免过多正常品被误拦）
- 如果两者难以兼顾，先保 Recall，再优化 Precision
```

**如何调整阈值：**
```python
# 通过调整置信度阈值来平衡 Precision 和 Recall
# 降低阈值 → Recall 升高，Precision 下降
# 升高阈值 → Recall 下降，Precision 升高

pipeline = create_pipeline(pipeline="object_detection")
pipeline.threshold = 0.3  # 降低阈值，提高召回率（质检场景推荐）
```

---

## Step 6：与大模型集成

### 质检场景集成示例

```python
from paddlex import create_pipeline

# 小模型：缺陷检测
cv_pipeline = create_pipeline(pipeline="object_detection")
cv_pipeline.threshold = 0.3  # 低阈值，宁可误报不漏报

def inspect_product(image_path: str, product_info: dict) -> dict:
    """完整的质检流程：小模型感知 + 大模型分析"""

    # Step 1: 小模型检测
    cv_result = cv_pipeline.predict(image_path)[0]

    defects = [
        {
            "type": box["label"],
            "confidence": round(box["score"], 3),
            "location": box["coordinate"],
            "severity": estimate_severity(box)  # 基于面积估算严重程度
        }
        for box in cv_result["boxes"]
        if box["score"] > 0.3
    ]

    # Step 2: 无缺陷，直接通过
    if not defects:
        return {"decision": "PASS", "defects": [], "analysis": None}

    # Step 3: 大模型分析（仅在发现缺陷时调用，控制成本）
    prompt = f"""
    你是工业质检专家。产品 {product_info['id']}（{product_info['type']}）的视觉检测结果如下：

    检测到的缺陷：
    {json.dumps(defects, ensure_ascii=False, indent=2)}

    产品技术标准：{product_info['spec']}
    当前工序：{product_info['process']}

    请判断：
    1. 整体质量等级（合格/轻微不合格/严重不合格）
    2. 每个缺陷的性质（外观缺陷/功能性缺陷/安全缺陷）
    3. 处置建议（直接放行/返工/报废/升级检验）
    4. 可能的产生原因（帮助工艺改进）

    以 JSON 格式输出。
    """

    analysis = llm_client.chat(prompt, response_format="json")

    return {
        "decision": analysis["quality_grade"],
        "defects": defects,
        "analysis": analysis,
        "cv_model_version": "defect_v2.1"
    }
```

---

## 上线红线检查清单

```markdown
### CV 模型上线红线

精度指标：
- [ ] 核心缺陷类别 Recall ≥ 99%
- [ ] 整体 Precision ≥ 80%（避免过多误报）
- [ ] 边界案例（轻微缺陷）单独评估通过

推理性能：
- [ ] 单张图片推理延迟 p95 < [业务要求] ms
- [ ] GPU 显存占用 < 显存上限的 80%

数据质量：
- [ ] PaddleX check_dataset 校验通过
- [ ] 测试集包含 ≥ 20% 边界案例
- [ ] 标注一致率 ≥ 85%（双人标注验证）

集成稳定性：
- [ ] 低置信度降级处理已验证（score < 阈值时转人工）
- [ ] 图像预处理异常处理（模糊、过暗、超大图片）
- [ ] 大模型不可用时 fallback 到仅小模型判断

监控：
- [ ] 低置信度比例监控（> 20% 触发告警，可能出现分布偏移）
- [ ] 误报率日监控（连续3天误报率 > 15% 触发再训练评估）
```
