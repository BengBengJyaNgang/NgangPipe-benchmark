# NgangPipe

**NgangPipe** 是首个面向城市排水管道 CCTV 影像的大规模公开目标检测数据集，提供边界框标注，并附带基准测试：21 个 YOLO 系列检测器在统一的逐类别协议下训练（共计 378 次训练运行），为智能管道检测领域的算法研究与可复现对比提供公共基础。

- 📄 论文：*NgangPipe: A Dataset and Evaluation Benchmark for CCTV Defect Detection in Urban Drainage Pipelines*（WACV 2027 UrbanState Workshop，Datasets Track，审稿中）
- 🧭 数据集主页（基准代码 + 权重）：https://github.com/BengBengJyaNgang/NgangPipe-benchmark
- 🤗 数据集镜像：https://huggingface.co/datasets/bengbengJyaNgang/NgangPipe-benchmark
- [English README](README.md)

## 亮点

| | |
|---|---|
| 缺陷图像（去重后） | **25,267** 张 |
| 边界框标注 | **34,370** 个 |
| 缺陷类别 | **18** 类（按管材 × 视觉形态组织） |
| 管材类型 | HDPE 波纹管 & 混凝土管 |
| 检测里程 | 120 km CCTV 检测视频 |
| 基准模型 | 21 个 YOLO 变体（YOLOv5–YOLO26，共六个代际） |
| 训练运行次数 | 378 次（逐类别单类协议） |
| 总体平均 mAP50 | **0.8304** |

其中 8 个类别已接近饱和（最佳逐类 mAP50 ≥ 0.92），而丝状树根、混凝土管接口错位、渗漏等类别仍有明显提升空间——**漏检**是最主要的失效模式。逐类结果、失效分析、混淆矩阵与数据规模消融实验详见论文。

## 类别体系

18 个类别沿两个维度组织——管材与视觉形态——并保持与 CJJ 181-2012 的工程语义映射（完整映射表见论文）。分组图例：**B** = HDPE 波纹管特有，**T** = 两种管材共有，**H** = 混凝土管特有。DSW = 依据 CJJ 181-2012 等级推导的缺陷严重度权重。

| 编码 | 类别名称 | 分组 | 图像数 | 框数 | DSW |
|---|---|---|---:|---:|---:|
| 0BXX | 心形内凹 | B | 919 | 914 | 0.80 |
| 1BGK | 接口环异常 | B | 913 | 1,491 | 0.40 |
| 2BTZ | 条状异常 | B | 5,170 | 7,464 | 0.50 |
| 3BJD | 接口分离 | B | 1,579 | 2,439 | 0.90 |
| 4BPD | HDPE 波纹管破洞 | B | 969 | 966 | 0.90 |
| 5BTL | HDPE 波纹管坍塌 | B | 972 | 982 | 1.00 |
| 6TZA | 障碍物 | T | 2,360 | 2,434 | 0.20 |
| 7TCJ | 沉积物 | T | 2,587 | 2,460 | 0.20 |
| 8TSL | 渗漏 | T | 706 | 785 | 0.50 |
| 9TAJ | 私接支管 | T | 1,636 | 1,667 | 0.20 |
| 10TQF | 积水 | T | 894 | 847 | 0.10 |
| 11BLZ | 颗粒状异常 | B | 619 | 660 | 0.30 |
| 12HCK | 混凝土管接口错位 | H | 683 | 1,321 | 0.50 |
| 13HLF | 混凝土管裂缝 | H | 3,932 | 5,345 | 0.60 |
| 14HPD | 混凝土管破洞 | H | 663 | 646 | 0.90 |
| 15HTL | 填缝料脱落 | H | 534 | 575 | 0.10 |
| 16TSS | 丝状树根 | T | 1,433 | 2,429 | 0.40 |
| 17TTS | 团状树根 | T | 594 | 945 | 0.50 |

&gt; 逐类计数基于分解后的单缺陷训练图像统计；含多类缺陷的帧在其所属的每个类别中各计一次。发布时额外混入约 7% 的无缺陷背景帧至训练集以抑制误检，这些帧**不计入**上表。

## 数据集结构

发布包以 `NGANGPipe-dataset` 为根目录，按缺陷类别组织。每个类别子集以 YOLO 格式发布，采用 8:1:1 的训练/验证/测试划分。其中 10 个类别保留原始（`_original`）数据。

```text
NGANGPipe-dataset/
├── {class code}/
│   ├── {class code}_original/      # raw data, stored flat (10 classes only)
│   └── {class code}_Processed/     # release & training data
│       ├── images/{train, val, test}/
│       └── labels/{train, val, test}/   # YOLO TXT, single-class (class 0)
├── 18HNM/                          # background-frame banks (defect-free)
├── 19BNM/
└── weights/                        # benchmark model weights
```

- 标注遵循标准 YOLO 格式（`class x_center y_center width height`，归一化坐标）。标签在**每个子集内为单类别**（每个框的类别均为 `0`）；子集目录名即类别标识。18 类映射见 `classes.yaml`。
- 提供数据规模消融子集（训练图像的 100% / 75% / 50% / 25% / 12%），用于参考模型 YOLO11-m。
- 多缺陷帧被分解为单缺陷训练图像，并对非目标区域做选择性高斯模糊，保证每张训练图像只含一个边界框和一个无歧义标签。

## 快速开始

基准测试使用的环境（任何兼容的 PyTorch/Ultralytics 环境均可）：

```bash
pip install ultralytics==8.4.138   # Python 3.9, PyTorch 1.13.1, CUDA 11.7
```

使用发布划分训练单个类别子集（示例：`13HLF`）：

```bash
yolo detect train \
  data=13HLF/13HLF_Processed/13HLF.yaml \
  model=yolo11m.pt \
  epochs=500 patience=100 batch=8 lr0=0.01
```

论文采用的基准协议：所有模型从 COCO 预训练权重初始化，**逐类别**训练（单类别检测头），共 500 个 epoch，早停耐心值 100，SGD 优化器（初始学习率 0.01，动量/权重衰减取框架默认值），批大小 8，默认数据增强，全部 378 次运行使用相同超参数。验证指标在验证集上计算；测试集保留，不用于训练或调参。

## 基准协议与指标

- **逐类别训练协议。** 18 个类别子集各自独立训练，避免视觉相似类别在共享分类头上互相干扰。
- **指标。** IoU 0.5 下的逐类 AP50，以及 18 类宏平均 mAP50。数据集同样支持 mAP50:95、精确率、召回率和 F1。
- **参考结果。** 合并训练与逐类协议的对比（YOLO11-m 基线）及逐类最佳模型见论文（表 5–6）；基准权重位于 `weights/` 目录。

## 许可与署名

- 数据集中自行采集的部分（影像与标注）以 **CC BY-NC 4.0** 许可发布：允许自由使用、修改与再分发，但需署名且**仅限非商业用途**。
- 基准代码以 **MIT** 许可发布。
- 外部来源的混凝土管缺陷影像单独打包，遵循原始提供方的许可，**不**以 CC BY-NC 4.0 重新授权。
- 所有图像均嵌入了不可见水印以追踪未授权传播。请勿尝试反推匿名编码对应的真实管道位置。
- 数据集按**“现状”**提供，不附带对特定用途适用性的任何担保。

**基于本数据集发表工作时，请引用论文。**

```bibtex
@inproceedings{ngangpipe2027,
  title   = {NgangPipe: A Dataset and Evaluation Benchmark for CCTV Defect Detection in Urban Drainage Pipelines},
  author  = {Anonymous},
  booktitle = {WACV 2027 UrbanState Workshop, Datasets Track},
  note    = {Under review},
  year    = {2027}
}
```

## 更新日志

数据集版本变更记录见论文附录 C.1。

## 问题反馈

如有数据集问题、标注错误样本或基准复现问题，请在本仓库提交 Issue。
