---
name: dl-model-selection
description: 基于 PaddleX 全链条范式，为 CV/OCR/时序/语音任务选择合适的模型和产线。核心决策是：用预训练模型直接部署、微调，还是从头训练？大模型和小模型如何分工？
---

# DL Model Selection — 深度学习模型选型

## 咨询链位置

**在新版 SOP 中：** 深度学习范式的核心决策模块，在数据策略（`dl-data-strategy`）确认后使用。

**联动：**
- `cv-pipeline` — CV 任务的具体实施
- `ocr-pipeline` — OCR 任务的具体实施
- `ts-pipeline` — 时序任务的具体实施
- `dl-deployment-pattern` — 模型选定后的部署决策

---

## PaddleX 产线选型地图

### CV 类产线

| 业务需求 | PaddleX 产线 | 推荐模型（精度优先） | 推荐模型（速度优先） |
|---------|------------|-----------------|-----------------|
| 图像属于哪个类别 | 通用图像分类 | PP-HGNetV2-B6 | PP-LCNet_x1_0 |
| 图中有什么东西、在哪里 | 通用目标检测 | PicoDet-L | PicoDet-S |
| 像素级分类（区域分割） | 通用语义分割 | PP-LiteSeg-T | PP-LiteSeg-T |
| 工业缺陷检测（无异常样本） | 图像异常检测 | STFPM | STFPM |
| 小目标检测 | 小目标检测 | PP-YOLOE_plus_SOD-L | PP-YOLOE_plus_SOD-S |
| 人脸/人体检测 | 人脸检测/人体检测 | BlazeFace | BlazeFace |

### OCR 类产线

| 业务需求 | PaddleX 产线 | 核心模型 |
|---------|------------|--------|
| 通用文字识别（印刷体） | 通用OCR | PP-OCRv4_server |
| 手写体识别 | 通用OCR（微调） | PP-OCRv4_server_rec |
| 表格内容识别 | 通用表格识别 | SLANet_plus |
| 文档版面分析+信息抽取 | 文档场景信息抽取v3/v4 | PP-ChatOCRv3/v4 |
| 印章识别 | 文档场景信息抽取 | PP-ChatOCRv3 |
| 公式识别 | 公式识别 | UniMERNet |

**PP-ChatOCRv3/v4 的大小模型连用架构：**
```
图像输入
  ↓ 小模型（PP-OCRv4）
文字检测 + 识别
  ↓ 大模型（文心）
结构化信息提取（字段理解、关联）
  ↓
结构化 JSON 输出
```

### 时序类产线

| 业务需求 | PaddleX 产线 | 推荐模型 | 特点 |
|---------|------------|--------|------|
| 未来值预测（销量/成本/用电量） | 时序预测 | PatchTST | 高精度，兼顾局部和全局 |
| 未来值预测（简单场景） | 时序预测 | DLinear | 轻量，效率高，易部署 |
| 检测时序中的异常点 | 时序异常检测 | PatchTSAD | — |
| 对时序片段分类 | 时序分类 | TimesNet | 多周期分析 |

**时序预测模型选择决策树：**
```
数据量 < 1000 条？
  → 是 → DLinear（轻量，不易过拟合）
  → 否 → 序列有明显周期性？
            → 是 → TimesNet（多周期分析）
            → 否 → 需要长期预测（>96步）？
                      → 是 → PatchTST
                      → 否 → DLinear 或 TiDE
```

---

## 框架选择：PaddleX vs 其他框架

**核心原则：框架跟着任务和硬件走，不是跟着平台走。**

### 优先选 PaddleX 的条件

- 国产硬件部署（昆仑芯XPU / 华为昇腾NPU）— PaddleX 原生支持，其他框架需额外适配
- 任务在 PaddleX 覆盖范围内（CV/OCR/时序预测/异常检测）
- 团队已有 PaddlePaddle 经验
- 需要一站式全链条工具（训练+部署+服务化）

### 需要其他框架的场景

| 场景 | 推荐框架 | 核心模型 |
|------|---------|---------|
| 语音识别（中文） | FunASR / PaddleSpeech | Paraformer / Conformer |
| 语音识别（多语言/高精度） | OpenAI Whisper | Whisper large-v3（MIT授权）|
| 声纹识别 | SpeechBrain | ECAPA-TDNN |
| 目标检测（PyTorch生态首选） | Ultralytics | YOLOv8 / YOLOv11 / RT-DETR |
| NLP 文本分类/NER | HuggingFace | BERT / RoBERTa / ERNIE 3.0 |
| 工业异常检测（更多选择） | PyTorch | PatchCore / FastFlow / EfficientAD |
| 医疗影像分析 | MONAI | nnUNet / TransUNet |
| 多模态理解 | HuggingFace | CLIP / BLIP2 / InternVL2 |
| 时序预测（更多模型） | PyTorch | Informer / iTransformer / TimeMixer |
| 视频理解 | PyTorch | TimeSformer / Video Swin Transformer |

### 非 PaddleX 框架的通用工作流

```bash
# 以 HuggingFace 为例（PyTorch 生态通用模式）

# 1. 安装
pip install transformers datasets torch

# 2. 快速体验预训练模型
from transformers import pipeline
classifier = pipeline("image-classification", model="microsoft/resnet-50")
result = classifier("your_image.jpg")

# 3. 微调（以图像分类为例）
from transformers import AutoFeatureExtractor, AutoModelForImageClassification, TrainingArguments, Trainer
# ... 标准 HuggingFace 微调流程

# 4. 导出为 ONNX（部署通用格式）
model.export("model.onnx", format="onnx")

# 5. 用 ONNXRuntime 部署
import onnxruntime
session = onnxruntime.InferenceSession("model.onnx")
```

### 混用策略（常见场景）

```
场景：工厂质检系统（国产昇腾NPU + 语音指令）

质检模块：PaddleX（原生支持昇腾NPU）
  → PP-YOLOE 目标检测
  → 直接在昇腾NPU上推理

语音指令模块：FunASR（支持昇腾适配）
  → Paraformer 语音识别
  → 转换为文本后送给调度逻辑

结论生成：本地部署LLM（文心/Qwen）
  → 分析质检结果，生成处置建议

三个模块，三套框架，各负其责。
```

---

## 大小模型分工决策框架

**核心问题：这个任务的哪部分需要"理解"，哪部分需要"感知"？**

```
感知任务（小模型负责）          理解任务（大模型负责）
─────────────────────          ─────────────────────
图像中有没有缺陷                为什么出现这个缺陷
文档里有哪些文字                这些文字意味着什么业务结论
时序数据是否异常                异常可能的原因和处理建议
声音是什么指令                  这个指令的上下文意图
```

**典型的大小模型连用架构：**

```
[原始输入] → [小模型感知层] → [结构化中间结果] → [大模型理解层] → [业务输出]

示例1（工业质检）：
摄像头图像 → CV小模型（缺陷检测） → {缺陷位置,类型,置信度} → LLM → 质检报告+处置建议

示例2（财务票据）：
票据图片 → OCR小模型（文字识别） → {字段:值} → LLM → 结构化财务数据+异常标注

示例3（设备监控）：
传感器数据 → 时序小模型（异常检测） → {异常时间点,分数} → LLM → 异常分析报告
```

---

## 模型选型五步法（基于PaddleX）

### Step 1：快速体验预训练模型

```bash
# 一行命令体验，不需要任何配置
paddlex --pipeline [产线名] --input [测试图片/数据] --device gpu:0
```

**评估标准：**
- 精度是否满足业务要求？（不是 benchmark，是你的真实数据）
- 速度是否满足实时性要求？
- 输出格式是否符合下游使用？

### Step 2：确认是否需要微调

| 预训练模型效果 | 建议 |
|-------------|------|
| 满足要求（精度 gap < 5%） | 直接部署，无需微调 |
| 基本可用（gap 5%-20%） | 少量数据微调（50-500条） |
| 不满足（gap > 20%） | 收集业务数据，全量微调 |
| 完全无关（场景差异极大） | 考虑从头训练 |

### Step 3：微调 vs 从头训练决策

**选微调（Transfer Learning）的条件：**
- 预训练模型的场景与目标场景有一定相似性
- 数据量有限（< 10000 条）
- 时间窗口紧（< 4 周）

**选从头训练的条件：**
- 场景极度特殊（预训练模型完全无效）
- 数据量充足（> 10000 条，CV场景）
- 对极致精度要求高，可以承受更长训练周期

### Step 4：模型规格选择（精度 vs 速度）

**选 Server 端大模型（精度优先）：**
- 离线批处理（不需要实时）
- 精度要求极高（质量报告、财务分析）
- 有 GPU 服务器

**选 Mobile/轻量模型（速度优先）：**
- 实时推理（< 100ms）
- 边缘端部署（工控机、摄像头）
- 无 GPU 或 GPU 资源有限

### Step 5：验证模型选择

```bash
# 训练后评估
python main.py -c config.yaml -o Global.mode=evaluate

# 关注指标（按任务类型）：
# 分类：Accuracy, Top-1 Accuracy
# 检测：mAP, AP50
# 分割：mIoU
# OCR识别：字符级准确率（Acc）
# 时序预测：MSE, MAE
# 时序异常：Precision, Recall, F1
```

---

## 硬件选型指引

PaddleX 支持多种硬件，产业落地时的硬件选型原则：

| 场景 | 推荐硬件 | 说明 |
|------|---------|------|
| 云端训练 | NVIDIA A100/V100 | 训练速度最快 |
| 云端推理（高并发） | NVIDIA T4/A10 | 推理性价比高 |
| 边缘推理（工厂现场） | 昆仑芯XPU / 华为昇腾NPU | 国产化替代，PaddleX原生支持 |
| 嵌入式/摄像头端 | ARM CPU + NPU | 轻量模型 + INT8量化 |
| 纯 CPU 服务器 | x86 CPU | 小模型，低并发场景 |

**国产硬件适配（PaddleX 支持）：**
```bash
# 昆仑芯 XPU
paddlex --pipeline image_classification --device xpu:0

# 华为昇腾 NPU
paddlex --pipeline image_classification --device npu:0
```

---

## 输出格式

```markdown
## DL 模型选型决策

**任务类型：** [分类/检测/分割/OCR/时序预测/时序异常]
**PaddleX 产线：** [产线名称]
**阶段：** MVP / 生产

### 模型选择
| 层级 | 模型 | 理由 | 预期精度 |
|------|------|------|---------|
| 主模型 | [模型名] | [为什么选这个] | [指标目标] |
| 备选 | [模型名] | [成本/速度权衡] | — |

### 大小模型分工
- 小模型负责：[感知任务]
- 大模型负责：[理解/生成任务]
- 接口：[小模型输出格式 → 大模型输入格式]

### 微调策略
- 预训练模型基线：[体验后的实际精度]
- 业务要求精度：[目标]
- Gap：[差距]
- 微调数据量：[N 条]
- 预计微调周期：[X 周]

### 硬件配置
- 训练：[GPU型号]
- 推理：[硬件 + 推理模式（高性能/服务化/端侧）]
- 推理延迟目标：[X ms]

### 快速体验命令
paddlex --pipeline [产线] --input [测试数据] --device [设备]

### 上线前精度红线
- [ ] 验证集精度 ≥ [目标值]
- [ ] 推理延迟 p95 ≤ [目标值] ms
- [ ] 在边界案例上单独评估通过
```
