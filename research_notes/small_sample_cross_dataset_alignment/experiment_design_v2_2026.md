# 实验路线 V2 · 4 周执行计划

> 基于 2026 年最新趋势更新。每个实验标注：目的、输入、输出、指标、成功标准、论文价值。

---

## 第 1 周：跨域 Baseline

### 目标
建立 DAIC-WOZ ↔ E-DAIC 跨域基线，知道"跨域掉多少分"。

### 实验列表

| 编号 | 实验 | 训练集 | 测试集 | 目的 | 评价指标 |
|:----:|:----|:------|:------|:-----|:--------|
| W1-1 | DAIC→DAIC (5-fold CV) | DAIC-WOZ | DAIC-WOZ | 域内baseline | AUC / F1 / MAE |
| W1-2 | E-DAIC→E-DAIC (5-fold CV) | E-DAIC | E-DAIC | 域内baseline | AUC / F1 / MAE |
| **W1-3** | **DAIC→E-DAIC** | **DAIC-WOZ** | **E-DAIC** | **跨域验证🔑** | AUC / F1 / MAE |
| **W1-4** | **E-DAIC→DAIC** | **E-DAIC** | **DAIC-WOZ** | **反向跨域🔑** | AUC / F1 / MAE |
| W1-5 | Pooled→Pooled CV | DAIC+E-DAIC | DAIC+E-DAIC | 合并训练上限 | AUC / F1 / MAE |
| W1-6 | 单模态(T) | DAIC-WOZ | E-DAIC | 仅文本跨域 | AUC |
| W1-7 | 单模态(A) | DAIC-WOZ | E-DAIC | 仅音频跨域 | AUC |
| W1-8 | 单模态(V) | DAIC-WOZ | E-DAIC | 仅面部跨域 | AUC |

### 模型
- LogReg / SVM / XGBoost / LightGBM / MLP
- 每个模型 5 seeds

### 输入文件
- DAIC-WOZ: `processed/openface_stats.csv`, `audio_embeddings.npy`, `text_embeddings.npy`
- E-DAIC: 同上结构

### 输出文件
- `results/cross_domain_baseline.csv`
- `results/cross_domain_baseline_summary.md`

### 成功标准
- 跨域 AUC 比域内低多少？< 5% 算不错，> 10% 说明域偏移严重
- Pooled 训练是否优于单数据集？

### 论文价值
✅ 直接写进论文——这是审稿人第一个要问的问题

### 即使失败也有价值
- 跨域掉分严重 → 说明需要域适应方法 → 创新点有依据
- Pooled 不提升 → 说明两数据集分布不同 → 需要更精细的对齐

---

## 第 2 周：特征与标签对齐

### 目标
解决跨数据集最大的工程障碍：特征维度不一致、标签体系不同。

### 实验列表

| 编号 | 实验 | 内容 | 输出 |
|:----:|:----|:-----|:-----|
| W2-1 | Dataset-wise 标准化 | 每个数据集特征独立 Z-score | `features_normalized/` |
| W2-2 | 标签映射 | 统一为 binary: PHQ-8≥10, PHQ-9≥10, SDS≥40 | `labels_unified.csv` |
| W2-3 | 公共特征框架 | 定义 text/audio/visual 公共特征维 | `common_feature_schema.json` |
| W2-4 | 模态可用性矩阵 | 每个数据集的模态可用情况 | `modality_availability.csv` |
| W2-5 | 缺失模态处理 | zero-padding / mean / learnable mask | 对比实验 |

### 对齐后的实验

| 编号 | 实验 | 说明 |
|:----:|:----|:-----|
| W2-6 | 对齐后 DAIC→E-DAIC | 特征标准化后再做跨域 |
| W2-7 | 对齐后 Pooled | 标准化+统一标签后合并训练 |

### 成功标准
- 特征标准化后跨域 AUC 是否提升
- 模态缺失补全后能否保留性能

### 论文价值
✅ 工程贡献，可写进方法章节

---

## 第 3 周：症状引导特征

### 目标
LLM 症状抽取 → 症状向量 → 单独建模 → 与传统特征融合 → 跨域测试。

### 实验列表

| 编号 | 实验 | 内容 |
|:----:|:----|:-----|
| W3-1 | Prompt 设计 | 设计 5 版 prompt，对比抽取质量 |
| W3-2 | 人工验证 | 人工标注 50 条，评估抽取准确率 |
| W3-3 | 症状向量建模 | 只用 symptom vector 训练分类器 |
| W3-4 | 症状+传统特征融合 | symptom vector + wav2vec2/AU/BERT 拼接 |
| **W3-5** | **症状跨域稳定性** | **DAIC 训练症状模型→E-DAIC 测试🔑** |
| W3-6 | 症状可视化 | 两组抑郁/非抑郁的症状向量分布对比 |

### 模型
- 症状向量: LogReg / XGBoost
- 融合: MIL / MLP

### 输入
- DAIC-WOZ transript → LLM → `symptom_features.csv` (每行9维)
- E-DAIC transcript → 同上

### 输出
- `results/symptom_modeling.md`
- `results/symptom_fusion.md`
- `results/symptom_cross_domain.md`

### 成功标准
- 症状向量单独 AUC > 0.70？症状本身就有预测力！
- 融合后比纯传统特征 AUC 提升？
- 症状向量跨域稳定性：训练域名→测试域，症状特征分布是否一致？

### 论文价值
⭐⭐⭐⭐⭐ 核心创新——"症状引导的可解释多模态融合"

---

## 第 4 周：轻量对齐与可信输出

### 目标
域对齐方法 + 统计学严谨性。

### 实验列表

| 编号 | 实验 | 内容 |
|:----:|:----|:-----|
| W4-1 | CORAL 对齐 | 对齐源域目标域协方差 |
| W4-2 | MMD 对齐 | 最小化最大均值差异 |
| W4-3 | Supervised Contrastive | 对比学习拉近同类 |
| W4-4 | Temperature Scaling | 校准模型概率输出 |
| W4-5 | Bootstrap CI | 重采样 1000 次算 AUC 95% CI |
| W4-6 | Multi-seed 结果表 | 5 seeds mean ± std |
| W4-7 | Calibration curve | 校准曲线 + ECE |

### 完整结果表

```
实验编号 | 方法 | AUC | F1 | MAE | ECE | 95% CI | 种子数
W1-3    | DAIC→E-DAIC | 0.XX±0.XX | 0.XX | X.XX | 0.XX | [0.XX,0.XX] | 5
W4-1    | +CORAL      | 0.XX±0.XX | 0.XX | X.XX | 0.XX | [0.XX,0.XX] | 5
W4-2    | +MMD        | 0.XX±0.XX | 0.XX | X.XX | 0.XX | [0.XX,0.XX] | 5
W3-5    | +症状特征   | 0.XX±0.XX | 0.XX | X.XX | 0.XX | [0.XX,0.XX] | 5
```

### 成功标准
- CORAL/MMD 是否提升跨域 AUC
- Calibration 是否降低 ECE
- Bootstrap CI 宽度是否合理

### 论文价值
⭐⭐⭐⭐ 审稿人最看重的"实验严谨性"就靠这些

---

## 4 周时间线概览

```
第1周                     第2周                     第3周                     第4周
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ W1-1 域内baseline│    │ W2-1 特征标准化  │    │ W3-1 Prompt设计  │    │ W4-1 CORAL对齐  │
│ W1-2 域内baseline│    │ W2-2 标签映射    │    │ W3-2 人工验证    │    │ W4-2 MMD对齐    │
│ W1-3 DAIC→E-DAIC│    │ W2-3 特征框架    │    │ W3-3 症状向量    │    │ W4-3 SupCon     │
│ W1-4 E-DAIC→DAIC│    │ W2-4 模态矩阵    │    │ W3-4 症状融合    │    │ W4-4 Scaling    │
│ W1-5 Pooled     │    │ W2-5 缺失模态    │    │ W3-5 跨域稳定    │    │ W4-5 Bootstrap  │
│ W1-6~8 单模态   │    │ W2-6 对齐后跨域  │    │ W3-6 可视化     │    │ W4-6 多种子     │
│                 │    │ W2-7 对齐后Pooled│    │                 │    │ W4-7 校准曲线   │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 关键决策点

| 决策点 | 时间 | 判断标准 | 后续路径 |
|:------|:----|:---------|:---------|
| 跨域掉分 > 10%？ | 第1周结束 | DAIC→E-DAIC AUC | 严重→优先域适应；不严重→直接pooled |
| 症状向量 AUC > 0.70？ | 第3周中 | 单独症状建模 | 高→融合作为主推方法；低→放弃融合 |
| CORAL 提升 > 2%？ | 第4周初 | 对齐前后对比 | 有效→写进论文；无效→换方法 |
