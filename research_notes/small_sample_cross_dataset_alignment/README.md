# 小样本医学多模态学习与跨数据集对齐/泛化

> 大创项目：DAIC-WOZ / E-DAIC 多模态抑郁风险筛查
> 研究模块：小样本学习 + 跨数据集泛化

## 模块定位

本项目主模型为 MIL + Attention Pooling + LLM 症状特征抽取（M0→M3）。
本模块作为**论文发表后的延伸研究方向**，解决 DAIC-WOZ/E-DAIC 小样本场景下的核心瓶颈：
- 样本量有限（~460 sessions）导致模型泛化性存疑
- 多数据集间模态对齐困难（不同语言、采集协议、标签体系）
- 缺乏跨数据集验证的标准评估框架

## 目录结构

```
research_notes/small_sample_cross_dataset_alignment/
├── README.md                          ← 本文件：总览
├── literature_table.md                ← 完整论文表格
├── medical_small_sample_methods.md    ← 医学小样本学习方法综述
├── cross_dataset_alignment_methods.md ← 跨数据集对齐与泛化方法
├── depression_dataset_alignment.md    ← 抑郁症数据集对齐分析
├── experiment_design.md               ← 可落地实验设计
└── weekly_tracking.md                 ← 后续追踪记录
```

## 核心问题

1. **数据少**：DAIC-WOZ 仅 189 sessions，E-DAIC 275 sessions，总计 ~460
2. **模态多**：文本+音频+视频，每个模态都需要单独的处理 pipeline
3. **域偏移**：不同数据集语言不同（英/中）、采集协议不同、标签定义不同
4. **评估难**：缺乏大规模外部验证集，难以评估真实泛化能力

## 推荐技术路线（优先级排序）

| 优先级 | 路线 | 预期收益 | 可落地性 |
|:------:|:----|:--------:|:--------:|
| 🥇 | 多数据集 pooled 训练 + 跨域验证 | 3-8% AUC 提升 | ⭐⭐⭐⭐⭐ |
| 🥇 | Self-supervised pretraining + fine-tuning | 5-10% 小样本提升 | ⭐⭐⭐⭐ |
| 🥈 | Domain adversarial training | 2-5% 跨域提升 | ⭐⭐⭐ |
| 🥈 | Calibration + uncertainty estimation | 可解释性提升 | ⭐⭐⭐⭐⭐ |
| 🥉 | Test-time adaptation | 推理时自适应 | ⭐⭐ |

## 声明

本文献综述基于截至 2025 年的公开文献。部分搜索工具限流，如有遗漏欢迎补充。
方法推荐均以"本科大创阶段可落地"为第一原则。
