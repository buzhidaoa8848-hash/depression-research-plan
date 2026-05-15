# 可落地实验设计

> 设计原则：本科大创阶段，以"有限资源做最有价值的实验"为第一原则。

## 一、实验主线

### M0: 单数据集 Baseline

| 实验编号 | 训练集 | 测试集 | 模态 | 模型 | 预期基线 |
|:-------:|:------|:------|:----|:----|:--------:|
| M0a | DAIC-WOZ | DAIC-WOZ (5-fold CV) | 语音+文本+面部 | XGBoost | AUC ~0.76 |
| M0b | E-DAIC | E-DAIC (5-fold CV) | 语音+文本 | XGBoost | AUC ~0.75 |

### M1: 单数据集训练，跨数据集测试

| 实验编号 | 训练集 | 测试集 | 意义 |
|:-------:|:------|:------|:-----|
| M1a | DAIC-WOZ | E-DAIC | 同协议、同语言的域内泛化 |
| M1b | E-DAIC | DAIC-WOZ | 反向验证 |
| M1c | DAIC-WOZ+E-DAIC | DAIC-WOZ+E-DAIC CV | pooled 训练效果上限 |

### M2: 跨域实验（如果有 MODMA/CMDC）

| 实验编号 | 训练集 | 测试集 | 意义 |
|:-------:|:------|:------|:-----|
| M2a | DAIC-WOZ | MODMA | 跨语言泛化（英→中） |
| M2b | DAIC-WOZ+E-DAIC | MODMA | 多源训练→跨语言 |
| M2c | DAIC-WOZ+MODMA | E-DAIC | 跨语言辅助训练能否提升英语泛化？ |

### M3: Domain Adaptation 实验

| 实验编号 | 方法 | 训练/验证 | 预期提升 |
|:-------:|:----|:---------|:--------:|
| M3a | CORAL 特征对齐 | DAIC-WOZ→E-DAIC | +1-3% |
| M3b | DANN 对抗适应 | DAIC-WOZ→E-DAIC | +3-5% |
| M3c | 只微调 BN 层 | DAIC-WOZ→E-DAIC | +0-2% |

### M4: Calibration + 不确定性

| 实验编号 | 方法 | 评估指标 |
|:-------:|:----|:--------|
| M4a | Temperature Scaling | ECE (Expected Calibration Error) |
| M4b | Bootstrap 95% CI | AUC 置信区间宽度 |
| M4c | 5 seeds mean±std | 稳定性报告 |

### M5: Ablation

| 实验编号 | 模态组合 | 说明 |
|:-------:|:--------|:-----|
| M5a | 仅语音 | wav2vec2 |
| M5b | 仅文本 | MentalBERT |
| M5c | 仅面部 | OpenFace AU |
| M5d | 语音+文本 | 双模态 |
| M5e | 语音+文本+LLM症状 | 三模态+LLM增强 |
| M5f | 语音+文本+面部+LLM | 全模态 |

## 二、实验优先级排序

```
第一优先级（立即开始，2周内完成）：
  M0a, M0b    — baseline 基线
  M1a, M1b    — 跨数据集验证（重要发现）
  M4a, M4c    — 校准 + 多种子（论文底线）

第二优先级（论文发表前完成）：
  M5a-M5f     — 完整消融实验
  M4b         — Bootstrap CI
  M1c         — Pooled 训练

第三优先级（论文发表后追）：
  M3a, M3b    — Domain adaptation
  M2a-M2c     — 跨语言验证（依赖外数据集）
```

## 三、实验表格模板（每轮实验填写）

```
实验编号: M1a
日期: YYYY-MM-DD
训练集: DAIC-WOZ (n=189)
测试集: E-DAIC (n=275)
模态: 语音+文本+面部
模型: MIL + Attention Pooling
种子: [42, 123, 456, 789, 1111]
结果: AUC = 0.XX ± 0.XX
校准 ECE: 0.XX
Bootstrap 95% CI: [0.XX, 0.XX]
备注: 
```

## 四、最推荐的三条技术路线

### 🥇 路线1：跨数据集验证 + Pooled 训练
**理由**：零代码更改，只需要改数据加载路径。跑出结果就能回答"模型在 E-DAIC 上泛化如何？"
**行动**：今天就能开始的实验。

### 🥇 路线2：MIL + Domain Adversarial Training
**理由**：DANN 的梯度反转层实现简单（~20行代码），域对齐的效果已经被抑郁检测文献验证过。
**行动**：在 M1 模型上加一个域分类器 + 梯度反转层，源域和目标域同时训练。

### 🥈 路线3：Calibration + Uncertainty
**理由**：虽然不直接提升 AUC，但这是论文审稿人最常质疑的地方。做了就是良心论文。
**行动**：加 Temperature Scaling 校准 + Bootstrap Confidence Interval。

## 五、最适合写进论文的创新点

1. **症状引导的多模态可解释融合 + 跨数据集验证**
   - 核心：LLM 抽取 PHQ-9 症状特征 + MIL attention
   - 验证：在 DAIC-WOZ 和 E-DAIC 上做 cross-validation

2. **域对抗训练 + 抑郁症症状特征对齐**
   - 核心：在症状特征空间做域对齐（而不是原始特征空间）
   - 可解释：展示对齐前后症状分布的变化

3. **小样本医学多模态的严格评估框架**
   - 核心：subject-level split + 多种子 + Bootstrap CI + 校准
   - 价值：绝大多数论文不做这个，做了就是差异化

## 六、风险与坑点

| 风险 | 影响 | 应对 |
|:----|:----|:-----|
| DAIC→E-DAIC 域适应效果有限 | 收益不大 | 先跑 M1 看 baseline 差距 |
| MODMA 申请不下来 | 跨语言实验搁置 | 先用 AVEC 测试集替代（同语言） |
| 算力不足跑 DANN | 实验周期拉长 | 先做特征级对齐（CORAL） |
| 论文审稿人质疑跨域设置 | 需要更充分解释 | 在论文里充分讨论域偏移的原因 |
| 强化过拟合 | 域适应反而变差 | 严格 subject-level split + 早停 |

## 七、时间线

```
第1-2周：M0a, M0b, M1a, M1b（跨域验证基线）
第3-4周：M1c, M5a-M5f（pooled + 消融）
第5-6周：M4a-M4c（校准 + 统计）
第7-8周：M3a, M3b（域适应，可选）
第9-10周：论文写作 + 补充实验
```
