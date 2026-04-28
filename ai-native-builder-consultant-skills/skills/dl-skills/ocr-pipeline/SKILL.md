---
name: ocr-pipeline
description: OCR 类任务（文字识别/表格识别/文档信息抽取）的完整落地指南，基于 PaddleX PP-OCRv4 和 PP-ChatOCRv3/v4 产线，涵盖从数据准备到大小模型集成的全链条。适用于票据识别、工业标签识别、文档信息抽取等场景。
---

# OCR Pipeline — OCR 全链条落地

## 咨询链位置

**在新版 SOP 中：** OCR 任务的具体落地 Skill，在 `dl-model-selection` 确认 OCR 方向后启动。

**典型场景：**
- 工业标签/铭牌识别（设备编号、物料编码）
- 票据/单据信息抽取（采购单、质检报告）
- 文档结构化（合同、技术规范扫描件）
- 表格识别（BOM 表格、成本报表图片化）
- 印章/手写内容识别

---

## OCR 产线选择

### 产线决策树

```
输入是什么？
├── 图片中有文字，只需要识别出来
│   └── 通用 OCR（PP-OCRv4）
├── 图片中有表格，需要识别表格结构和内容
│   └── 通用表格识别
├── 文档图片，需要提取特定字段（如"合同金额""有效期"）
│   └── 文档场景信息抽取（PP-ChatOCRv3/v4）← 大小模型连用
└── 特殊场景
    ├── 印章识别 → PP-ChatOCRv3（含印章模块）
    ├── 公式识别 → UniMERNet
    └── 手写中文 → PP-OCRv4 微调
```

### 各产线说明

**通用 OCR（PP-OCRv4）：**
- 小模型完成文字检测 + 识别
- 输出：文字内容 + 位置框 + 置信度
- 适合：通用印刷体，速度快

**PP-ChatOCRv3/v4（大小模型连用）：**
- PP-OCRv4 小模型负责检测和识别（感知层）
- 文心大模型负责字段理解和结构化（理解层）
- 输出：结构化 JSON（字段: 值）
- 适合：需要"理解"文档内容，提取特定字段

---

## Step 1：快速体验

```bash
# 通用 OCR
paddlex --pipeline OCR \
        --input your_image.jpg \
        --use_doc_orientation_classify False \
        --save_path ./output \
        --device gpu:0

# 表格识别
paddlex --pipeline table_recognition \
        --input table_image.jpg \
        --save_path ./output \
        --device gpu:0
```

```python
# Python API 体验
from paddlex import create_pipeline

# 通用 OCR
ocr_pipeline = create_pipeline(pipeline="OCR")
result = ocr_pipeline.predict("input.jpg")
for res in result:
    print(res["rec_texts"])     # 识别的文字列表
    print(res["rec_scores"])    # 对应置信度

# 文档信息抽取（PP-ChatOCRv3）
chat_ocr = create_pipeline(pipeline="PP-ChatOCRv3-doc")
result = chat_ocr.predict(
    "invoice.jpg",
    query=["合同金额", "有效期", "甲方名称", "乙方名称"]
)
```

---

## Step 2：评估预训练模型 gap

**评估维度：**

| 维度 | 检查方法 | 合格标准 |
|------|---------|---------|
| 字符识别准确率 | 在20-50张真实业务图片上测试 | > 95%（印刷体）/ > 85%（手写）|
| 字段提取准确率 | 手工核对提取的字段值 | > 90% |
| 误检率 | 背景被识别为文字的比例 | < 5% |
| 漏检率 | 文字未被检测到的比例 | < 5% |

**常见需要微调的场景：**
- 工业现场特殊字体（点阵字体、LCD显示）
- 手写体（签名、手填表格）
- 特殊符号（化学式、单位符号）
- 图像质量差（模糊、强光、阴影）
- 竖排文字

---

## Step 3：数据准备（需要微调时）

### 文本识别标注格式

```
# 标注文件格式（txt）
# 每行：图片路径\t文字内容
./images/img001.jpg	发动机型号EV200-A
./images/img002.jpg	变更单号EWO-2024-0315
./images/img003.jpg	物料编码M-Cu-4N-001
```

### 标注工具和流程

```bash
# 推荐使用 PPOCRLabel 进行标注
pip install PPOCRLabel
PPOCRLabel --lang ch  # 启动中文标注工具
```

**标注质量要求：**
- 标注框紧贴文字边缘（不要留太多空白）
- 文字内容完全准确（包括标点、大小写）
- 模糊/遮挡严重的文字可跳过，不要强行标注

### 数据增强（OCR 专项）

```python
# PaddleX OCR 常用数据增强配置
Train:
  transforms:
    - type: RecResizeImg    # 统一高度到32像素
      image_shape: [3, 32, 320]
    - type: RecAug          # OCR专用增强（旋转、透视变换）
      use_tia_distort: true
      use_tia_stretch: true
      use_tia_perspective: true
```

**合成数据扩充（适用于文本识别）：**
```python
# 用 text_renderer 合成工业场景文字图片
# 将自定义字体渲染到工业背景图上，快速扩充数据
# 合成数据：真实数据 = 3:1 的比例效果较好
```

---

## Step 4：模型训练

```bash
# 文本识别微调
python main.py -c paddlex/configs/modules/text_recognition/PP-OCRv4_server_rec.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./dataset \
    -o Train.epochs_iters=20 \
    -o Train.batch_size=128 \
    -o Train.learning_rate=0.001

# 文本检测微调
python main.py -c paddlex/configs/modules/text_detection/PP-OCRv4_server_det.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./dataset \
    -o Train.epochs_iters=100
```

---

## Step 5：PP-ChatOCRv3/v4 大小模型集成

这是 OCR 场景中最重要的大小模型连用架构。

### 架构说明

```
文档图片
  ↓ [小模型层]
  PP-OCRv4：文字检测 + 识别
  版面分析：理解文档结构（标题/段落/表格/印章）
  表格识别：解析表格结构
  ↓ [结构化中间结果]
  {文字: 位置, 文字块: 内容, 表格: [[行][列]]}
  ↓ [大模型层]
  文心大模型：字段理解 + 关联 + 结构化输出
  ↓ [最终输出]
  {字段1: 值1, 字段2: 值2, ...}
```

### 代码示例（采购订单信息抽取）

```python
from paddlex import create_pipeline

# 创建 PP-ChatOCRv3 产线
pipeline = create_pipeline(pipeline="PP-ChatOCRv3-doc")

# 定义需要提取的字段
query_fields = [
    "供应商名称",
    "订单编号",
    "物料名称",
    "物料编码",
    "采购数量",
    "单价",
    "总金额",
    "交货日期",
    "合同编号"
]

# 执行提取
result = pipeline.predict(
    "purchase_order.jpg",
    query=query_fields
)

# 结构化输出
structured_data = result["chat_res"]
print(structured_data)
# 输出：{"供应商名称": "XX铜业", "订单编号": "PO-2024-001", ...}
```

### 微调 PP-ChatOCRv3 中的小模型

```bash
# 只需微调 OCR 小模型部分，大模型（文心）不需要微调
# 微调检测模型（针对特殊版面）
python main.py -c paddlex/configs/modules/text_detection/PP-OCRv4_server_det.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./custom_dataset

# 微调识别模型（针对特殊字体）
python main.py -c paddlex/configs/modules/text_recognition/PP-OCRv4_server_rec.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./custom_dataset

# 将微调后的模型集成回 PP-ChatOCRv3 产线
pipeline = create_pipeline(
    pipeline="PP-ChatOCRv3-doc",
    text_det_model_dir="./output/custom_det",
    text_rec_model_dir="./output/custom_rec"
)
```

---

## Step 6：部署

```bash
# 服务化部署（OCR API）
paddlex --serve \
    --pipeline OCR \
    --port 8080 \
    --device gpu:0

# 高性能推理（生产环境）
paddlex --pipeline OCR \
    --input image.jpg \
    --use_hpip \
    --device gpu:0
```

### 输出后处理

```python
def post_process_ocr(ocr_result: dict, confidence_threshold: float = 0.8) -> dict:
    """
    OCR 结果后处理：过滤低置信度、格式化输出
    """
    filtered_texts = []
    for text, score, bbox in zip(
        ocr_result["rec_texts"],
        ocr_result["rec_scores"],
        ocr_result["rec_boxes"]
    ):
        if score >= confidence_threshold:
            filtered_texts.append({
                "text": text,
                "confidence": round(score, 3),
                "bbox": bbox.tolist()
            })

    return {
        "total_detected": len(ocr_result["rec_texts"]),
        "high_confidence": len(filtered_texts),
        "low_confidence_rate": 1 - len(filtered_texts) / len(ocr_result["rec_texts"]),
        "results": filtered_texts
    }
```

---

## 工业场景特殊处理

### 图像预处理（工厂环境常见问题）

```python
import cv2
import numpy as np

def preprocess_industrial_image(image_path: str) -> np.ndarray:
    """
    工业场景图像预处理：去噪、增强对比度、矫正
    """
    img = cv2.imread(image_path)

    # 去噪（高斯模糊去除传感器噪点）
    img = cv2.GaussianBlur(img, (3, 3), 0)

    # 对比度增强（CLAHE，适合光照不均匀场景）
    lab = cv2.cvtColor(img, cv2.COLOR_BGR2LAB)
    l, a, b = cv2.split(lab)
    clahe = cv2.createCLAHE(clipLimit=3.0, tileGridSize=(8, 8))
    l = clahe.apply(l)
    img = cv2.cvtColor(cv2.merge([l, a, b]), cv2.COLOR_LAB2BGR)

    # 倾斜矫正（如果文字倾斜）
    # PaddleX 的 use_doc_unwarping=True 可自动处理

    return img
```

---

## 评估指标和上线红线

```markdown
### OCR 上线红线

文本识别：
- [ ] 字符级准确率（Acc）≥ 95%（印刷体）
- [ ] 低置信度比例（score < 0.8）< 10%

文档信息抽取：
- [ ] 关键字段提取准确率 ≥ 90%
- [ ] 关键字段召回率 ≥ 85%（不漏字段）

性能：
- [ ] 单张图片推理时间 p95 < 3秒（服务化部署）
- [ ] 批量处理速度满足业务吞吐需求
```
