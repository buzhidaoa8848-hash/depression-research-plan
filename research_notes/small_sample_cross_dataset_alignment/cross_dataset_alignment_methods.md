# 跨数据集对齐与泛化方法

## 一、问题框架

当你有多数据集，但来源不同，面临的核心问题是<strong>域偏移 (Domain Shift)</strong>：

```
训练数据 (源域)                   测试数据 (目标域)
DAIC-WOZ (英语, 美国)     vs     E-DAIC (英语, 美国, 扩展)
DAIC-WOZ                  vs     MODMA (中文, 中国)
DAIC-WOZ + E-DAIC         vs     CMDC (中文, 中国)
DAIC-WOZ                  vs     AVEC (英语, 跨年份)
```

## 二、方法体系

### 2.1 域适应 (Domain Adaptation)

**设定**：源域有标签，目标域有或无标签。

#### 2.1.1 对抗性域适应 (Adversarial Domain Adaptation)

```
核心思路：训练特征提取器同时骗过域分类器
          → 学到"域不变"的特征表示
```

**代表方法**：
- DANN (Ganin & Lempitsky, 2015) — 梯度反转层
- CDAN (Long et al., 2018) — 条件域对抗
- 在抑郁检测上的应用：论文8, 14

**对你大创的适用性**：
- ✅ DAIC-WOZ → E-DAIC：域偏移较小，预期效果好
- ⚠️ DAIC-WOZ → MODMA/CMDC：语言+文化双重偏移，效果有限
- ⚠️ 需要两数据集特征维度一致（MLP 投影到统一维度）

#### 2.1.2 统计特征对齐 (Statistical Feature Alignment)

```
核心思路：显式匹配源域和目标域的统计分布
```

**代表方法**：
- CORAL (Sun & Saenko, 2016) — 对齐协方差矩阵
- MMD (Maximum Mean Discrepancy) — 最小化分布差异
- 深度 CORAL — 在深度特征层做协方差对齐

**对你大创的适用性**：
- ✅ 简单有效，适合 high-level 特征的分布匹配
- ✅ 适合 ML baseline (M0) 的统计特征（eGeMAPS, OpenFace AU）

#### 2.1.3 特征归一化 (Feature Harmonization)

```
核心思路：在特征层面做标准化，消除数据集间的系统偏差
```

**对你大创的适用性**：
- 不同数据集音频采样率不同、视频帧率不同 → 先统一预处理
- 特征层面做 ComBat 协调（原用于基因表达数据，已被引入神经影像）

### 2.2 域泛化 (Domain Generalization)

**设定**：在多个源域上训练，在<strong>完全未见</strong>的目标域上测试。

**对你大创的意义**：更现实的场景——模型要在从未见过的临床环境中使用。

| 方法 | 核心思想 | 你的适用性 |
|:----|:---------|:----------|
| IRM (Invariant Risk Minimization) | 学习跨域不变的特征-标签关系 | ⭐⭐ 理论强但实现难 |
| GroupDRO | 最差组风险最小化 | ⭐⭐⭐ 实现简单 |
| Meta-learning (MAML) | 学会快速适应新域 | ⭐⭐ 计算成本高 |
| Data augmentation (域随机化) | 在训练中模拟域变化 | ⭐⭐⭐⭐ 简单有效 |

### 2.3 测试时适应 (Test-time Adaptation/TTT)

**设定**：推理时用目标域数据自身的信息自适应调整模型。

| 方法 | 核心思想 | 你的适用性 |
|:----|:---------|:----------|
| TTT (Test-Time Training) | 测试时自监督更新 | ⭐⭐ 增加推理时间 |
| TENT | 测试时仅更新 BN 参数 | ⭐⭐⭐ 极轻量 |
| MEMO | 单样本边缘熵最小化 | ⭐⭐ 不确定 |

### 2.4 多源域适应 (Multi-source Domain Adaptation)

**设定**：多个有标签源域，一个或多个无标签目标域。

**对你的启示**：如果你最终拿到 DAIC-WOZ + E-DAIC + MODMA 三个源域，可以考虑。

### 2.5 对齐的实践路线

```
M0 baseline:
  DAIC-WOZ train → DAIC-WOZ test (within-domain baseline)
  
M1 cross-dataset:
  DAIC-WOZ train → E-DAIC test (cross-dataset)
  
M1.5 harmonization:
  统一特征维度 → 数据标准化 → 再做跨域
  
M2 domain adaptation:
  DANN / CORAL → DAIC-WOZ train → E-DAIC test
  
M3 domain generalization:
  DAIC-WOZ + E-DAIC → MODMA test (unseen)
  
M4 calibration:
  跨域后的概率校准 + Bootstrap CI
```

## 三、推荐程度

| 方法 | 效果 | 成本 | 创新度 | 推荐 |
|:----|:----|:----|:------|:----:|
| 跨域验证 (基础) | ★★ | ★ | ★★ | ✅ 必做 |
| 特征归一化 | ★★★ | ★★ | ★★ | ✅ 必做 |
| DANN 对抗适应 | ★★★★ | ★★★ | ★★★★★ | 🔥 创新点 |
| CORAL 对齐 | ★★★ | ★★ | ★★★ | ✅ 推荐 |
| 域随机化增强 | ★★★ | ★★ | ★★★ | ✅ 推荐 |
| 测试时适应 | ★★ | ★★★★ | ★★★★★ | ⚠️ 可选 |
