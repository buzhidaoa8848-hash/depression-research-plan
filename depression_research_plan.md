# DAIC-WOZ / E-DAIC 抑郁风险筛查 · 研究现状与技术方案

> 项目定位：基于访谈型多模态数据的抑郁风险辅助初筛模型（非临床诊断）
> 数据：DAIC-WOZ（189 sessions）+ E-DAIC（275 sessions）
> 标签：PHQ-8 分数 + 二分类（≥10 为抑郁）

---

## 第一部分：研究现状（2016-2026）

### 一、技术演进四阶段

| 阶段 | 时间 | 代表方法 | 特点 |
|:----|:----:|:---------|:-----|
| 传统特征 + ML | 2016-2018 | LIWC, eGeMAPS, OpenFace + SVM/RF | 可解释性强，指标有限 |
| 深度学习 | 2018-2021 | LSTM, CNN, 3D-CNN + 时序建模 | 能力提升，开始过拟合小样本 |
| 预训练模型 | 2021-2024 | wav2vec2, HuBERT, BERT, MentalBERT | 小样本跃升，主流方案 |
| LLM + 跨模态门控 | 2024-2026 | LLM 症状抽取, Cross-attention, MIL | 可解释性 + 多模态交互 |

### 二、各模态 SOTA 技术对比

#### 2.1 语音方向

| 技术 | 数据集 | 指标 | 是否 subject split | 备注 |
|:----|:------|:----:|:-----------------:|:-----|
| eGeMAPS(88d) + Logistic Regression | DAIC-WOZ | AUC=0.76 | ✅ | Baseline |
| COVAREP(444d) + SVM | DAIC-WOZ | Acc≈0.78 | ✅ | Baseline |
| OpenSMILE + LSTM | DAIC-WOZ | Acc≈0.85 | ⚠️ 部分有 leakage | 2018-2020 |
| **wav2vec2.0 微调** | DAIC-WOZ | Acc≈0.96 | ⚠️ 部分未严格 split | 需注意高估 |
| **HuBERT + CTC** | DAIC-WOZ | F1 显著提升 | ✅ | 2023-2024 |
| **Whisper embedding + 1D-CNN** | E-DAIC | 接近 SOTA | ✅ | 2024-2025 |

#### 2.2 面部行为方向

> ⚠️ **重要纠正**：DAIC-WOZ 视频 30fps，不足捕捉微表情（需 100-200fps），应称"面部行为分析 / AU分析"

| 技术 | 数据集 | 指标 | 备注 |
|:----|:------|:----:|:-----|
| OpenFace AU(17) + gaze + pose 统计特征 | DAIC-WOZ | 中等 | 最常用，几乎每篇必用 |
| TSNet-DD (3D CNN + 时间注意力) | E-DAIC | CCC=0.733 | 需 GPU |
| ResNet/ViT 面部视频编码 | DAIC-WOZ | 易过拟合 | 数据太小 |
| HOG + landmarks + SVM | DAIC-WOZ | Baseline | 过时 |

#### 2.3 文本方向

| 技术 | 数据集 | 指标 | 备注 |
|:----|:------|:----:|:-----|
| LIWC + LR | DAIC-WOZ | 可解释但低 | Baseline |
| BERT/RoBERTa 微调 | DAIC-WOZ | F1≈0.65-0.70 | 2020-2023 主流 |
| **MentalBERT/ClinicalBERT** | DAIC-WOZ | 比通用 BERT 好 2-5% | 2022-2024 |
| **双流 BERT+T5 + 1D-CNN** | E-DAIC | 提升准确率 | 2025 新工作 |

#### 2.4 多模态融合方向

| 技术 | 数据集 | 指标 | 备注 |
|:----|:------|:----:|:-----|
| Early fusion（拼接+LSTM） | DAIC-WOZ | 一般 | 维度爆炸 |
| Late fusion（独立+投票） | DAIC-WOZ | 稳定但上限低 | Benchmark |
| Cross-attention 跨模态注意力 | DAIC-WOZ | 比 early/late 好 | 2022-2024 |
| **SBT-Net（语义门控+张量注意力）** | DAIC-WOZ/EATD | SOTA | 2024 |
| TOAT（topic attention） | DAIC-WOZ | F1=0.647 | 2024 |
| **MIL + attention pooling** | DAIC-WOZ/E-DAIC | **F1≈0.86-0.88** | ⭐ 最适合本项目的架构 |

### 三、已知缺陷（所有已发表论文的通病）

| 缺陷 | 说明 | 影响 |
|:----|:-----|:-----|
| **Speaker leakage** | train/test 有同一个人 → 模型学说话人特征而非抑郁信号 | 准确率可能虚高 30%+ |
| **Segment leakage** | 同一人的不同片段分到 train/test | 高估严重 |
| **Ellie shortcut** | 模型靠识别 Ellie 提问句判断抑郁（准确率可以到 0.84 不看任何回答） | 最严重的伪高估 |
| **缺乏 subject-level split** | 很多论文不做人级划分 | 结果不可信 |
| **无置信区间** | 只报告单次运行指标 | 无法判断稳定性 |
| **无校准** | 概率输出不可靠 | 临床应用风险 |

### 四、研究空白（可发论文的创新点）

| 空白 | 说明 |
|:----|:-----|
| **LLM 结构化症状抽取 → 融合分类** | 用 LLM 按 PHQ-9 维度抽取症状特征后与传统特征融合，**几乎没人系统做过** |
| **端到端 Agent 部署** | 将模型封装为可交互的筛查 Agent，**没人做过** |
| **DAIC↔E-DAIC 严格跨域测试** | 很多论文只跑一个集，跨域泛化研究不足 |
| **不确定性估计 + 校准** | 绝大多数论文不做 |
| **症状级可解释性** | 多数只做二分类，不做症状维度分析 |

---

## 第二部分：推荐技术方案

### 一、核心技术路线

```
M0: 统计特征 + 传统ML baseline
  │
  ├─ 语音: eGeMAPS(88d) / COVAREP(444d) → 统计量
  ├─ 面部: OpenFace AU(17) + gaze(3) + pose(3) → 统计量  
  ├─ 文本: TF-IDF(5000d) / LIWC(93d)
  ├─ 模型: LR / SVM / RF / XGBoost
  ├─ 验证: 5-fold subject-level CV
  └─ 指标: AUC / F1 / MAE / RMSE
  │
  ▼
M1: MIL + 预训练特征 (主模型)
  │
  ├─ 语音: wav2vec2/HuBERT (冻结) → 768d/utterance
  ├─ 面部: OpenFace AU/gaze/pose 均值+方差 → 50d/utterance
  ├─ 文本: MentalBERT/RoBERTa (冻结) → 768d/utterance
  ├─ 融合: utterance-level 拼接 → Attention MIL → session-level
  ├─ 输出: PHQ-8 回归 + 二分类
  ├─ 可解释: attention 权重定位关键语句
  └─ 跨域: DAIC → E-DAIC / E-DAIC → DAIC
  │
  ▼
M2: 增强实验 (拉开差距)
  │
  ├─ LLM 症状特征: DeepSeek 从transcript抽9维PHQ症状 → 结构特征
  ├─ 多任务: PHQ-8 + 症状维度 + 情绪识别
  ├─ 校准: Temperature Scaling + ECE
  ├─ 置信区间: Bootstrap 95% CI
  ├─ 多种子: 5 seeds mean ± std
  └─ 消融: 单模态 vs 多模态 / 有LLM vs 无LLM

  ▼
M3: Agent Skill 蒸馏部署 (差异化)
  │
  ├─ 知识蒸馏: 大模型(wav2vec2+LLM+MIL) → 小模型(LightGBM)
  ├─ 封装: Hermes Agent Skill
  ├─ 输入: 访谈录音/转录
  ├─ 输出: 风险报告 + 关键症状 + 置信度
  └─ 演示: 答辩现场可运行
```

### 二、关键设计决策

| 决策点 | 建议 | 原因 |
|:------|:----|:------|
| **utterance 切分** | 按句子切，不按固定窗口 | 保持语义完整性，MIL instance 有意义 |
| **Ellie 处理** | 保留但标记 speaker 字段，训练时只取 participant | 删除会破坏对话结构 |
| **预训练策略** | **冻结特征提取器，只训分类头** | 小样本微调过拟合，wav2vec2 全参数微调必崩 |
| **模态融合时机** | Late fusion 起步 → M2 上 cross-attention | Late fusion 稳定可控 |
| **跨域顺序** | 先 DAIC→E-DAIC（大→小），再反向 | 源域数据多更稳定 |
| **LLM 角色** | 只做特征抽取，不做分类 | 防止 LLM 过拟合小样本，且可解释 |

### 三、算法对比与选择理由

| 算法 | 适合原因 | 风险 | 推荐度 |
|:----|:--------|:----|:------:|
| **MIL + Attention Pooling** | 天然适合 session→utterance 结构；可解释关键语句；已有 DAIC-WOZ 成功验证 | 需 careful tuning | ⭐⭐⭐⭐⭐ |
| **XGBoost (M0)** | 表格数据最强 baseline；对统计特征友好 | 无法建模时序 | ⭐⭐⭐⭐ |
| **LLM 症状抽取** | 创新点；可解释；与 PHQ-8 对齐 | LLM 可能幻觉 | ⭐⭐⭐⭐⭐ |
| **Supervised Contrastive Learning** | 小样本拉近同类 embedding | 超参敏感 | ⭐⭐⭐ |
| **Test-Time Training** | 推理时自适应，跨域有效 | 增加推理时间 | ⭐⭐⭐ |

### 四、不推荐的方案

| 方案 | 不推荐原因 |
|:----|:----------|
| **全参数微调 LLaMA** | 无 GPU，小样本必过拟合 |
| **GAN 生成面部数据** | 不稳定，失真，训练复杂 |
| **复杂图神经网络融合** | 需要大样本调参，DAIC-WOZ 太小 |
| **大规模 RLHF** | 数据量不够 |
| **扩散模型合成语音** | 可能改变抑郁相关声学特征 |

---

## 第三部分：推进路线（可执行版）

### 第一阶段：数据准备（第 1-2 周）

| 任务 | 负责人 | 交付物 |
|:----|:------|:-------|
| E-DAIC 数据申请 | 队员A | 申请表截图 |
| DAIC-WOZ 数据解析 | 项目负责人 | 文件目录 + 格式确认 |
| PHQ-8 标签表提取 | 项目负责人 | subject_label.csv |
| session→utterance 索引 | 队员A | utterance_index.csv |
| 缺失文件检查 | 队员B | missing_report.md |
| Ellie/Participant 分离 | 队员A | speaker_split.csv |

### 第二阶段：M0 Baseline（第 3-4 周）

| 任务 | 负责人 | 交付物 |
|:----|:------|:-------|
| OpenFace 统计特征 | 项目负责人 | openface_stats.csv |
| eGeMAPS/COVAREP 统计特征 | 队员B | audio_stats.csv |
| TF-IDF/LIWC 文本特征 | 队员A | text_features.csv |
| LR / RF / XGBoost 训练 | 项目负责人 | M0_results.md |
| 5-fold subject-level CV | 项目负责人 | M0_cv_results.csv |
| 可视化（分布/特征重要性） | 队员C | M0_plots/ |

### 第三阶段：M1 主模型（第 5-8 周）

| 任务 | 负责人 | 交付物 |
|:----|:------|:-------|
| wav2vec2/HuBERT 提取 | 项目负责人 | audio_embeddings.npy |
| MentalBERT 提取 | 项目负责人 | text_embeddings.npy |
| MIL 框架搭建 | 项目负责人 | mil_model.py |
| Attention Pooling 实现 | 项目负责人 | mil_attention.py |
| 全流程训练 + 评估 | 项目负责人 | M1_results.md |
| DAIC↔E-DAIC 跨域 | 项目负责人 | M1_cross_domain.md |
| 结果表 + 图表 | 队员C | results_table.xlsx |

### 第四阶段：M2 增强（第 9-12 周）

| 任务 | 负责人 | 交付物 |
|:----|:------|:-------|
| LLM 症状抽取 prompt 设计 | 项目负责人 | symptom_prompt.md |
| LLM 症状特征提取 | 项目负责人 | symptom_features.csv |
| 多任务学习 | 项目负责人 | M2_multitask.py |
| Temperature Scaling 校准 | 项目负责人 | M2_calibration.py |
| Bootstrap CI | 队员B | M2_bootstrap.csv |
| 5 seeds 跑全实验 | 项目负责人 | M2_multi_seed.csv |
| 消融实验 | 项目负责人 | M2_ablation.md |

### 第五阶段：Agent 部署 + 答辩（第 13-16 周）

| 任务 | 负责人 | 交付物 |
|:----|:------|:-------|
| 知识蒸馏大→小模型 | 项目负责人 | distilled_model.pkl |
| Hermes Skill 封装 | 项目负责人 | depression-screener skill |
| 论文/结项报告 | 项目负责人 | paper.md |
| 答辩 PPT | 队员C | defense.pptx |
| 简历项目描述 | 项目负责人 | resume_bullet.md |
| 答辩演练 | 全体 | 模拟答辩 |

---

## 第四部分：可写进论文/答辩的创新点表述

### 核心创新（2 个）

> **创新点 1：症状引导的多模态可解释融合框架**
>
> 针对现有抑郁筛查模型可解释性不足的问题，提出利用 LLM 从访谈文本中按 PHQ-9 维度抽取结构化症状特征，与传统语音/面部特征通过 MIL attention 机制融合，实现"症状级可解释"的抑郁风险预测——不仅能给出风险评分，还能明确指出哪些症状维度贡献最大、哪些语句触发了高风险判断。

> **创新点 2：面向抑郁筛查的 Agent 蒸馏部署方案**
>
> 针对现有模型停留在实验室指标、缺乏可用工具的问题，将训练后的筛查模型通过知识蒸馏压缩为轻量级版本，封装为 Hermes Agent Skill，实现"上传访谈录音 → 自动分析 → 输出风险报告"的端到端筛查流程，为早期抑郁风险筛查提供可交互的辅助工具。

### Baseline 描述

> 本研究采用严格 subject-level 5 折交叉验证，所有实验报告 5 个随机种子的均值±标准差，并通过 Bootstrap 方法计算 95% 置信区间。我们同时提供简单模型（均值 baseline）和传统机器学习 baseline（LR/RF/XGBoost）作为对照，确保结果不被高估。

### 局限性（答辩主动说）

> 本研究存在以下局限：(1) DAIC-WOZ/E-DAIC 样本量有限（约 460 人），模型泛化性需更大规模验证；(2) LLM 症状抽取受 prompt 设计和模型能力影响，存在幻觉风险；(3) Agent 部署版本经过知识蒸馏，性能与原模型存在差距。

---

## 第五部分：下一步行动（今天能做的）

| 优先级 | 任务 | 时间 | 产出 |
|:------:|:-----|:----:|:-----|
| **P0** | 队员提交 E-DAIC 申请 | 30min | 申请截图 |
| **P0** | 你解压 DAIC-WOZ 原始数据 | 1h | 数据目录 |
| **P1** | 你写 LLM 症状抽取 prompt 并跑一次 | 2h | 症状特征样例 |
| **P1** | 队员做 session→utterance 索引表 | 2h | utterance_index.csv |
| **P2** | 你跑 OpenFace 提取特征（如已有） | 3h | AU/gaze/pose 特征 |
| **P2** | 队员做缺失文件统计 | 1h | missing_report.md |

---

## 第六部分：论文发布后可追方向（2026-2027）

> 声明：以下方向基于截至 2025 年的公开文献检索与搜索引擎验证。标注「✅ 已有先例」的方向已在抑郁检测领域有相关工作，建议差异化切入；「⚠️ 极少先例」的方向可作为延伸创新点；「❌ 几乎空白」的方向可考虑作为独立新工作。

### 一、语音方向

#### 1.1 WavLM 替换 wav2vec2 做语音特征提取

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ✅ 已有类似思路（WavLM 在其他心理健康任务中有使用，但 DAIC-WOZ 上未见专门工作） |
| 与当前方案的关系 | 直接替换 M1 阶段 wav2vec2 特征提取器，预期对噪声鲁棒性更好 |
| 预期收益 | 边际提升（~1-3%），创新性有限 |
| 建议 | 可作为消融实验中的一个对比（wav2vec2 vs HuBERT vs WavLM），但不足以独立成文 |

#### 1.2 Whisper + LoRA 微调（🔥 推荐）

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ⚠️ 极少先例。现有工作是用 Whisper embedding（冻结）+ 1D-CNN（2024-2025），**未见有人用 LoRA 在 DAIC-WOZ 上微调 Whisper** |
| 与当前方案的关系 | 你方案中写「冻结特征提取器」，LoRA 是更好的折中——比冻结更强（适配下游任务），比全参数微调更稳（仅更新 0.1-1% 参数） |
| 预期收益 | 中等提升（2-5%），可作为论文中的一个对比实验 |
| 独特价值 | 证明 LoRA 在小样本抑郁检测场景下的有效性，可写进相关工作和消融实验 |
| 可落地性 | ⭐⭐⭐⭐ 代码改动小，已有 HuggingFace PEFT 库 |
| 注意事项 | Whisper 是 ASR 模型，语音 embedding 针对语音识别而非抑郁检测，LoRA 微调可缓解域偏移 |

#### 1.3 语音情感 embedding 时序建模

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ⚠️ 极少先例。少量工作用 emotion2vec / speech emotion recognition 辅助抑郁检测 |
| 思路 | 对每段 utterance 提取情感 embedding（开心/悲伤/焦虑等概率分布），建模访谈过程中的**情感轨迹** |
| 与当前方案的关系 | 可作为额外的模态分支，在 MIL 融合中加入情感特征 |
| 建议 | 可作为多任务学习的一部分（抑郁分类 + 情感识别），提升可解释性 |

### 二、面部方向

#### 2.1 ST-GCN 建模 AU 时空依赖（🔥 推荐）

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ❌ 几乎空白。现有工作将 OpenFace AU 作为统计特征（均值/方差）输入分类器，**未见将 AU 关键点建模为时空图网络的工作** |
| 核心思想 | 将 17 个 AU + gaze + pose 作为图节点，时序帧作为边，用 ST-GCN 同时建模 AU 之间的依赖关系和时序变化 |
| 与当前方案的关系 | 替换 M1 中 OpenFace 统计特征 → ST-GCN 产出动态 AU embedding，然后与语音/文本融合 |
| 独特价值 | 创新点明确：「面部行为动态图建模」，目前 DAIC-WOZ 上无人做过 |
| 可落地性 | ⭐⭐⭐ 需要一定 graph 网络知识，但 ST-GCN 实现已成熟（有开源代码） |
| 注意事项 | DAIC-WOZ 视频 30fps，单 session 约 20 分钟，ST-GCN 需要合理下采样 |

#### 2.2 VideoMAE / Video Vision Transformers

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ⚠️ 极少先例。视频预训练模型在抑郁检测中应用较少，主要受限于 DAIC-WOZ 仅 ~460 个视频 |
| 核心挑战 | 数据量太小，直接微调 ViT 必过拟合 |
| 可行路径 | 在大规模面部数据集（AffectNet, RAF-DB）上预训练 VideoMAE，然后在 DAIC-WOZ 上冻结/部分微调 |
| 建议 | 计算成本高，不建议作为主要方向，可作为长期探索 |

#### 2.3 AV-HuBERT 做面部-语音联合 Embedding

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ⚠️ 极少先例。AV-HuBERT 主要用于视听语音识别（AVSR），**未见用于抑郁检测** |
| 思路 | 利用 AV-HuBERT 在大型视听数据（LRS3）上的预训练，提取唇部运动+语音的联合 embedding，作为下游抑郁检测的输入 |
| 独特价值 | 首次将视听联合预训练模型应用于抑郁检测 |
| 注意 | DAIC-WOZ 的视频分辨率低、拍摄角度固定，唇部信息有限，可能效果不佳 |

### 三、多模态融合方向

#### 3.1 Mamba / State Space Model 替代 Attention Pooling（🔥🔥 最强推荐）

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ❌ 几乎空白。Mamba（2024 年 S6 架构）在抑郁检测领域 **至今未见应用** |
| 核心思想 | 当前 M1 方案用 attention pooling 聚合 utterance 特征（bag-of-utterances，忽略时序）。Mamba 能建模完整访谈的时序依赖关系，比 Transformer 更省显存（线性复杂度） |
| 具体做法 | session → utterance embeddings (T×d) → Mamba 块 → session 表征 |
| 独特价值 | ① 首次将 SSM 引入抑郁检测；② 证明时序建模比 bag-of-utterances 更好；③ 可发独立短文 |
| 可落地性 | ⭐⭐⭐⭐⭐ 开源实现齐全（mamba.py, mamba-minimal），训练成本低 |
| 预期收益 | 中等-high（如果时序信息确实重要，可能提升 3-10%） |

#### 3.2 Cross-modal Contrastive Learning 模态对齐

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ⚠️ 极少先例。对比学习在多模态抑郁检测中有零星工作 |
| 思路 | 在 MIL 融合之前，先做语音↔文本、面部↔语音的对比学习对齐——拉近相同抑郁状态的跨模态表达，推远不同状态 |
| 与当前方案的关系 | 在特征提取和融合之间插入对比对齐步骤 |
| 建议 | 可作为消融实验对比「有/无对比对齐」，适合放进 ablation table |

#### 3.3 Causal Multimodal Fusion（🔥 推荐）

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ❌ 几乎空白。因果推断在多模态抑郁检测中的应用 **几乎无人涉及** |
| 核心思想 | 不只是「语音特征 A 和面部特征 B 一起预测抑郁」，而是问：语音特征变化是否是抑郁的**原因**？面部 AU 的变化是否是情绪状态变化的**因果效应**？ |
| 具体方法 | ① 用因果图建模：抑郁 → 语音改变、抑郁 → 面部表情减少、抑郁 → 语言内容变化；② 用结构因果模型（SCM）或反事实推理做干预；③ 只保留有因果效应的模态特征 |
| 独特价值 | ① 方法论创新（目前无人做因果+抑郁检测）；② 天然可解释（因果图展示）；③ 可写进高水平论文 |
| 可落地性 | ⭐⭐ 因果推断方法有一定门槛（DoWhy 库、因果图先验） |
| 建议 | 适合作为论文的一个独立创新点，但需要时间学习因果推断方法 |

### 四、Agent / LLM 方向

#### 4.1 Multi-Agent Debate 抑郁筛查（🔥 推荐）

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ❌ 几乎空白。多智能体辩论（Multi-Agent Debate）在心理健康领域 **尚无应用** |
| 核心思想 | 多个 LLM Agent 各分析一个模态（Agent A 分析语音、Agent B 分析文本、Agent C 分析面部），然后通过多轮辩论达成抑郁风险判断共识 |
| 与当前方案的关系 | 直接衔接 M3（Agent 推理部署），是 M3 的自然升级——从单 Agent 推理 → 多 Agent 辩论 |
| 具体方案 | ① 每个 Agent 独立分析单一模态并给出初步判断及置信度；② 共享判断结果，进行 2-3 轮辩论（可质疑、可修正）；③ 最终投票/加权融合 |
| 独特价值 | ① 新颖（无人做过）；② 可解释（辩论过程透明）；③ 可控制（每个 Agent 输出可审查）；④ 天然适合答辩展示 |
| 可落地性 | ⭐⭐⭐ 需要调用 LLM API，辩论轮次需要一定 token 预算 |

#### 4.2 Multimodal Foundation Model 端到端筛查

| 项目 | 内容 |
|:----|:-----|
| 现状判断 | ⚠️ 极少先例。GPT-4V / Gemini 等有被试用于心理健康分析，但未见在 DAIC-WOZ 上的系统评估 |
| 思路 | 直接用多模态大模型（GPT-4V, Gemini, LLaVA-NeXT）吃入访谈视频+音频+文本，零样本或少样本做抑郁风险判断 |
| 注意 | ① API 成本高（长视频需极高 tokens）；② 零样本表现可能不及专用模型；③ 隐私问题（数据不能传国外 API） |
| 建议 | 作为基线对比或探索性实验，不建议作为主方案 |

### 五、综合推荐排序

| 优先级 | 方向 | 创新性 | 可落地性 | 预期提升 | 推荐原因 |
|:------:|:----|:------:|:--------:|:--------:|:---------|
| 🥇 | **Mamba 替代 Attention Pooling** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 3-10% | 最稳，时序建模真能改进 + 几乎无人做 |
| 🥇 | **Multi-Agent Debate 筛查** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 可解释性↑ | 最亮眼，答辩可展示 + 无人做过 |
| 🥈 | **Whisper + LoRA 微调** | ⭐⭐⭐ | ⭐⭐⭐⭐ | 2-5% | 改动小，收益明确，可做消融 |
| 🥈 | **ST-GCN 面部动态图** | ⭐⭐⭐⭐ | ⭐⭐⭐ | 3-8% | 面部方向最稳的创新点 |
| 🥈 | **Causal Multimodal Fusion** | ⭐⭐⭐⭐⭐ | ⭐⭐ | 可解释性↑↑ | 方法论创新最大的方向 |
| 🥉 | **Cross-modal Contrastive** | ⭐⭐⭐ | ⭐⭐⭐⭐ | 2-5% | 稳定但不够惊艳 |
| 🥉 | **语音情感 embedding 时序** | ⭐⭐⭐ | ⭐⭐⭐ | 1-3% | 可作为多任务分支 |

### 六、论文发表后行动计划

```
0-3 月：
  1. 跑 Mamba vs Attention Pooling 对比实验（最快出结果，3-10%提升潜力）
  2. 设计 Multi-Agent Debate 原型（最亮眼，适合答辩展示）
  
3-6 月：
  3. 跑 Whisper + LoRA 消融实验（补充数据）
  4. 尝试 ST-GCN 面部动态图建模（如果时间允许）

6-12 月：
  5. 探索 Causal Multimodal Fusion（长期方向，方法论创新）
  6. 考虑写短文 / workshop paper 投出去
```
