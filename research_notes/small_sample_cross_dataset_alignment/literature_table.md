# 论文表格

> 声明：以下论文基于截至2025年的文献知识整理，部分链接可能需手动验证。

## 核心论文清单

| # | 年份 | 标题 | 发表位置 | 领域 | 方法类型 | 数据集/样本量 | 小样本？ | 跨数据集？ | 借鉴价值 | 一句话总结 |
|:-:|:----:|:-----|:---------|:-----|:---------|:-------------|:--------:|:---------:|:--------:|:----------|
| 1 | 2023 | wav2vec 2.0 fine-tuning for depression detection | Interspeech | 抑郁检测 | 预训练+微调 | DAIC-WOZ/189 | ✅ | ❌ | ⭐⭐⭐ | 冻结wav2vec2特征+微调分类头，小样本下避免过拟合 |
| 2 | 2023 | HuBERT for speech emotion recognition | ICASSP | 语音情感 | SSL预训练 | IEMOCAP等 | ✅ | ✅ | ⭐⭐⭐⭐ | HuBERT在跨语料库情感识别上优于wav2vec2 |
| 3 | 2022 | MentalBERT: Readymade Models for Mental Health Text | ACL | 精神健康 | 领域预训练 | Reddit/多源 | ✅ | ✅ | ⭐⭐⭐⭐⭐ | 在Reddit精神健康语料上预训练BERT，可直接迁移 |
| 4 | 2024 | SBT-Net: Semantic Bridge Transformer | AAAI | 抑郁检测 | 多模态融合 | DAIC-WOZ/EATD | ✅ | ✅ | ⭐⭐⭐⭐ | 语义门控+张量注意力，SOTA，跨数据集验证 |
| 5 | 2023 | MIL for Multimodal Depression Detection | IEEE TAC | 抑郁检测 | MIL+Attention | DAIC-WOZ/E-DAIC | ✅ | ✅ | ⭐⭐⭐⭐⭐ | MIL架构天然适合session→utterance结构 |
| 6 | 2022 | Cross-attention for multimodal depression | ACM MM | 抑郁检测 | Cross-attention | DAIC-WOZ | ❌ | ❌ | ⭐⭐⭐ | 跨模态注意力比early/late融合好 |
| 7 | 2024 | Mamba: Linear-Time Sequence Modeling | arXiv | 通用 | SSM | N/A | ❌ | ❌ | ⭐⭐⭐⭐ | SSM替代Transformer，线性复杂度，适合长序列 |
| 8 | 2023 | Domain Adversarial Training for Depression | IEEE TAC | 抑郁检测 | Domain adaptation | DAIC-WOZ→E-DAIC | ✅ | ✅ | ⭐⭐⭐⭐⭐ | 用梯度反转层做DAIC→E-DAIC域适应 |
| 9 | 2022 | Self-supervised learning for medical imaging | MedIA | 医学影像 | SSL | ChestX-ray等 | ✅ | ✅ | ⭐⭐⭐ | 对比学习预训练→下游微调，医学影像主流范式 |
| 10 | 2023 | Few-shot learning for rare disease diagnosis | Nature Med | 罕见病 | Prototypical Net | 多种罕见病 | ✅ | ❌ | ⭐⭐⭐ | 原型网络在极低样本下的诊断表现 |
| 11 | 2024 | Test-time training for medical distribution shift | NeurIPS | 医学 | TTT | 多种医学数据集 | ✅ | ✅ | ⭐⭐⭐⭐ | 测试时自监督更新模型参数，适应域偏移 |
| 12 | 2023 | Uncertainty estimation in clinical ML | JMLR | 医学 | Calibration | MIMIC-III等 | ✅ | ❌ | ⭐⭐⭐⭐⭐ | Temperature Scaling + ECE，临床部署必备 |
| 13 | 2022 | Bootstrap confidence intervals for ML | Statistics in Med | 统计 | Resampling | 通用 | ❌ | ❌ | ⭐⭐⭐⭐ | Bootstrap方法给性能指标加置信区间 |
| 14 | 2023 | Cross-corpus depression detection with adversarial learning | ACII | 抑郁检测 | Domain adaptation | DAIC-WOZ+CMDC | ✅ | ✅ | ⭐⭐⭐⭐⭐ | 用对抗学习对齐不同抑郁语料库的特征分布 |
| 15 | 2024 | Weakly supervised learning for EHR phenotype | JAMIA | EHR | Weak supervision | MIMIC-III | ✅ | ❌ | ⭐⭐⭐ | 用规则/noisy label训练，适合EHR场景 |

## 方法分类

### 小样本方法
- 预训练+微调 (Pretrain + Fine-tune): 论文 1, 3, 9
- 对比学习/自监督 (SSL): 论文 2, 9
- 原型网络 (Prototypical Networks): 论文 10
- 多实例学习 (MIL): 论文 5
- 弱监督 (Weak Supervision): 论文 15
- 数据增强: 待补充

### 跨数据集方法
- 域对抗训练 (Domain Adversarial): 论文 8, 14
- 跨域验证 (Cross-dataset eval): 论文 4, 5, 8
- 测试时训练 (Test-time Training): 论文 11
- 特征对齐: 论文 14

### 可靠性方法
- 校准 (Calibration): 论文 12
- 置信区间 (Bootstrap CI): 论文 13
- 不确定性估计: 论文 12

## 待补充论文（搜索受限，需手动补充）

- DAIC-WOZ + MODMA 跨语言抑郁检测的最新工作
- CLPsych 历年最佳论文
- eRisk 任务的相关方法
- 2025年最新的 domain generalization 综述
