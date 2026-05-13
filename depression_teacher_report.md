# DAIC-WOZ / E-DAIC 抑郁风险筛查 · 老师汇报材料

> 项目定位：基于访谈型多模态数据的抑郁风险辅助初筛模型（非临床诊断）
> 数据源：DAIC-WOZ（189 sessions）+ E-DAIC（275 sessions）
> 标签体系：PHQ-8 分数 + 二分类（≥10 为抑郁）

---

## 一、数据集介绍

### 1.1 DAIC-WOZ（Distress Analysis Interview Corpus - Wizard of Oz）

| 项目 | 内容 |
|:----|:------|
| 来源 | USC Institute for Creative Technologies |
| 规模 | 189 个访谈 session（其中 47 抑郁 / 142 非抑郁） |
| 模态 | 音频（44.1kHz）、视频（30fps）、文本转录 |
| 标签 | PHQ-8 分数（0-24）、二分类（≥10 = 抑郁） |
| 访谈形式 | 虚拟助手 Ellie 与受试者对话，平均时长 ~20 分钟 |
| 语言 | 英语 |
| 获取方式 | 签署 EULA 后从 USC ICT 申请下载 |
| 引用 | Valstar et al., "AVEC 2016: Depression, Mood, and Emotion Recognition Workshop and Challenge" |

### 1.2 E-DAIC（Expanded DAIC）

| 项目 | 内容 |
|:----|:------|
| 来源 | USC ICT 后续扩展 |
| 规模 | 275 个 session |
| 模态 | 音频 + 文本转录（部分有视频） |
| 标签 | 同 DAIC-WOZ：PHQ-8 + 二分类 |
| 获取方式 | 从 USC ICT 申请，比 DAIC-WOZ 更严格 |
| 引用 | Gratch et al., "E-DAIC: A New Corpus for Depression Detection" / AVEC 2019 |

### 1.3 数据集特点与注意事项

| 特点 | 详细说明 |
|:----|:---------|
| ✅ 学术标准数据集 | DAIC-WOZ/E-DAIC 是抑郁检测领域最广泛使用的基准 |
| ✅ 多模态同步 | 音频/视频/文本时间对齐，天然适合多模态研究 |
| ✅ 临床标签 | PHQ-8 是临床验证的抑郁筛查工具 |
| ⚠️ 样本有限 | 总计 ~460 人，小样本限制深度模型使用 |
| ⚠️ 类别不均衡 | 抑郁比例约 25%，需要处理不平衡 |
| ⚠️ 单语种 | 仅英语，跨语言泛化待验证 |
| ❌ 隐私限制 | 数据不可公开，仅学术用途，不可上传公共 API |

---

## 二、技术演进与现状

### 2.1 四阶段演进总览

| 阶段 | 时间 | 代表方法 | 代表论文 |
|:----|:----:|:---------|:---------|
| 传统特征 + ML | 2016-2018 | LIWC, eGeMAPS, OpenFace + SVM/RF | Valstar 2016, Williamson 2016 |
| 深度学习 | 2018-2021 | LSTM, CNN, 3D-CNN | Alhanai 2018, Ray 2019 |
| 预训练模型 | 2021-2024 | wav2vec2, HuBERT, BERT, MentalBERT | Yang 2022, Ma 2023, Dumpala 2022 |
| LLM + 跨模态门控 | 2024-2026 | LLM 症状抽取, Cross-attention, MIL | Muzi 2024, SBT-Net 2024 |

### 2.2 各模态 SOTA 论文索引

#### 语音方向

| 技术 | 论文 | 年份 | 数据集 | 指标 |
|:----|:----|:----:|:------|:----:|
| eGeMAPS + LR | Valstar et al., AVEC 2016 | 2016 | DAIC-WOZ | AUC=0.76 |
| COVAREP + SVM | Williamson et al., Interspeech 2016 | 2016 | DAIC-WOZ | Acc≈0.78 |
| OpenSMILE + LSTM | Ray et al., Interspeech 2019 | 2019 | DAIC-WOZ | Acc≈0.85 |
| wav2vec2.0 微调 | Yang et al., Interspeech 2022 | 2022 | DAIC-WOZ | Acc≈0.96 |
| HuBERT + CTC | Ma et al., ICASSP 2023 | 2023 | DAIC-WOZ | F1 显著提升 |
| Whisper embedding + 1D-CNN | Venkatesh et al., 2024 | 2024 | E-DAIC | 接近 SOTA |

#### 面部行为方向

| 技术 | 论文 | 年份 | 数据集 | 备注 |
|:----|:----|:----:|:------|:-----|
| OpenFace AU + 统计特征 | Valstar 2016, 几乎所有论文 | 2016 | DAIC-WOZ | 通用 baseline |
| TSNet-DD (3D CNN) | Du et al., ACII 2023 | 2023 | E-DAIC | CCC=0.733 |
| HOG + landmarks + SVM | Jaiswal et al., 2019 | 2019 | DAIC-WOZ | 过时 |

#### 文本方向

| 技术 | 论文 | 年份 | 数据集 | 备注 |
|:----|:----|:----:|:------|:-----|
| LIWC + LR | Valstar 2016 baseline | 2016 | DAIC-WOZ | Baseline |
| BERT/RoBERTa 微调 | Dumpala et al., Interspeech 2022 | 2022 | DAIC-WOZ | F1≈0.65-0.70 |
| MentalBERT | Ji et al., ACL 2022 | 2022 | 通用 | 比通用 BERT 好 2-5% |
| ClinicalBERT | Huang et al., arXiv 2020 | 2020 | MIMIC-III | 临床领域预训练 |

#### 多模态融合方向

| 技术 | 论文 | 年份 | 数据集 | 备注 |
|:----|:----|:----:|:------|:-----|
| Early/Late fusion | Alhanai et al., 2018 | 2018 | DAIC-WOZ | Benchmark |
| Cross-attention | Zhao et al., ACM MM 2022 | 2022 | DAIC-WOZ | 比 early/late 好 |
| SBT-Net | Muzi et al., AAAI 2024 | 2024 | DAIC-WOZ/EATD | SOTA |
| TOAT | Li et al., EMNLP 2024 | 2024 | DAIC-WOZ | F1=0.647 |
| MIL + attention pooling | Wang et al., IEEE TAC 2023 | 2023 | DAIC-WOZ/E-DAIC | F1≈0.86-0.88 |

### 2.3 已知缺陷（论文通病）

1. **Speaker leakage** — train/test 有同一人，准确率虚高 30%+
2. **Segment leakage** — 同一人片段跨 split，严重高估
3. **Ellie shortcut** — 模型靠识别 Ellie 提问句判断抑郁（可达 0.84 不看回答）
4. **缺乏 subject-level split** — 很多论文不做人级划分
5. **无置信区间** — 只报告单次运行
6. **无校准** — 概率输出不可靠

---

## 三、论文发布后可追方向（2026-2027）

### 3.1 各方向核心论文与任务关联分析

| 序号 | 方向 | 核心论文 | 为什么对抑郁检测任务有用 | 创新点潜力 |
|:---:|:----|:---------|:------------------------|:----------:|
| **1** | **Mamba 替代 Attention Pooling** | Gu & Dao, "Mamba: Linear-Time Sequence Modeling with Selective State Spaces", arXiv 2024 | 当前 MIL 是 bag-of-utterances（忽略时序顺序），而抑郁症状在访谈中是有时间模式的——开场拘谨、中期放松、后期情绪低落。Mamba 能线性复杂度建模完整时序，比 Transformer 省显存 | ⭐⭐⭐⭐⭐ |
| **2** | **Multi-Agent Debate 抑郁筛查** | Du et al., "Improving Factuality and Reasoning in LLMs through Multi-Agent Debate", arXiv 2024 | 每个模态独立分析后辩论：语音 Agent 说"语速变慢"，面部 Agent 说"微笑减少"，文本 Agent 说"消极词汇增多"——辩论过程天然可解释，适合临床场景 | ⭐⭐⭐⭐⭐ |
| **3** | **Whisper + LoRA 微调** | Radford et al., "Robust Speech Recognition via Large-Scale Weak Supervision", OpenAI 2023; Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models", ICLR 2022 | Whisper 通用语音表征与抑郁声学特征有域偏移，LoRA 微调可自适应调整（仅更新 0.1% 参数），比冻结更强、比全微调更稳 | ⭐⭐⭐⭐ |
| **4** | **ST-GCN 面部 AU 动态图** | Yan et al., "Spatial Temporal Graph Convolutional Networks for Skeleton-Based Action Recognition", AAAI 2018 | OpenFace 提取的 AU 不是独立的——AU4（皱眉）和 AU7（眼睑收紧）常同时出现。ST-GCN 能建模 AU 间的时空依赖，比统计量更 Rich | ⭐⭐⭐⭐ |
| **5** | **Causal Multimodal Fusion** | Pearl, "Causality" (书); Nogueira et al., "Causal Inference in Machine Learning" | 现有模型只做相关性（语音A→抑郁），不问因果（抑郁是否真的导致语音A变化？）。因果图可以过滤掉虚假相关、提升泛化 | ⭐⭐⭐⭐⭐ |
| **6** | **Cross-modal Contrastive Learning** | Radford et al., "CLIP: Learning Transferable Visual Models from Natural Language Supervision", ICML 2021 | 语音↔文本对比对齐，拉近相同抑郁状态的跨模态表示，推远不同状态。在融合前先做对齐，理论上提升融合效果 | ⭐⭐⭐ |
| **7** | **WavLM 替换 wav2vec2** | Chen et al., "WavLM: Large-Scale Self-Supervised Pre-Training for Full Stack Speech Processing", IEEE JSTSP 2023 | WavLM 在语音去噪和说话人表征上优于 wav2vec2，对访谈环境的背景噪声更鲁棒 | ⭐⭐ |
| **8** | **AV-HuBERT** | Hsu et al., "AV-HuBERT: Audio-Visual HuBERT", arXiv 2023 | 视听联合预训练模型，可从大型数据（LRS3）学到音画一致的表示，直接提取音频+面部联合 embedding | ⭐⭐⭐ |
| **9** | **VideoMAE** | Tong et al., "VideoMAE: Masked Autoencoders are Data-Efficient Learners for Self-Supervised Video Pre-Training", NeurIPS 2022 | 视频自监督预训练，但 DAIC-WOZ 仅 460 个视频，需要先在大规模面部数据集预训练再迁移 | ⭐⭐ |
| **10** | **语音情感 embedding** | Ma et al., "emotion2vec: Self-Supervised Pre-Training for Speech Emotion Representation", ACL 2024 | 将每段 utterance 的情感概率分布作为额外特征，建模访谈全程的"情感轨迹"——从开心到低落的转变本身就是一个抑郁指标 | ⭐⭐⭐ |

### 3.2 综合推荐排序（给老师版）

| 优先级 | 方向 | 创新性 | 可落地性 | 预计产出 | 直接收益 |
|:------:|:----|:------:|:--------:|:---------|:--------:|
| 🥇 | **Mamba 替代 Attention Pooling** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 1 篇短文/期刊 | 时序建模 vs bag-of-utterances，明确 ablation |
| 🥇 | **Multi-Agent Debate 筛查** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 大创加分+答辩亮点 | 展示效果好，可交互演示 |
| 🥈 | **Whisper + LoRA 微调** | ⭐⭐⭐ | ⭐⭐⭐⭐ | 消融实验数据 | 代码改动小，对比明确 |
| 🥈 | **ST-GCN 面部动态图** | ⭐⭐⭐⭐ | ⭐⭐⭐ | 1 个创新点 | 面部方向唯一的结构化创新 |
| 🥈 | **Causal Multimodal Fusion** | ⭐⭐⭐⭐⭐ | ⭐⭐ | 方法论论文 | 目前无人做，空白大 |
| 🥉 | **Cross-modal Contrastive** | ⭐⭐⭐ | ⭐⭐⭐⭐ | 消融实验 | 作为 Ablation 中的一行 |
| 🥉 | **语音情感轨迹** | ⭐⭐⭐ | ⭐⭐⭐ | 多任务分支 | 可作为多任务的一部分 |

---

## 四、可用模型与技术详细说明

### 4.1 语音模型

#### WavLM (Microsoft 2023)
- **架构**：Transformer with gated relative position bias
- **预训练数据**：94k hours 多语种语音
- **参考实现**：HuggingFace `microsoft/wavlm-base-plus`
- **在项目中的角色**：替换 wav2vec2，做 utterance-level 语音特征提取
- **输入/输出**：原始音频 → 768d embedding/utterance

#### Whisper (OpenAI 2023)
- **版本推荐**：Whisper small/medium（平衡效果和资源）
- **参考实现**：`openai/whisper` 或 HuggingFace `openai/whisper-small`
- **在项目中的角色**：提取通用语音 embedding + LoRA 微调适配抑郁检测
- **LoRA 配置**：`rank=8, alpha=16, target_modules=["q_proj", "v_proj"]`

#### emotion2vec (ACL 2024)
- **功能**：专为语音情感识别设计的情感表征模型
- **在项目中的角色**：提取每段 utterance 的情感概率分布（开心/悲伤/愤怒/中性等）

### 4.2 面部模型

#### OpenFace (免费学术工具)
- **输出**：17 个 AU 强度 + gaze 方向 + head pose
- **在项目中的角色**：M0 baseline 特征提取
- **注意**：30fps，不可用于微表情分析

#### ST-GCN (AAAI 2018)
- **架构**：时空图卷积，空间维建模 AU 关系，时间维建模帧间变化
- **输入**：T×V 的时空图（T=时间帧, V=AU+pose 节点）
- **输出**：session-level 面部动态 embedding
- **开源实现**：`microsoft/ST-GCN` (GitHub)

#### VideoMAE (NeurIPS 2022)
- **架构**：Video Masked Autoencoder（ViT backbone）
- **推荐预训练数据集**：AffectNet, RAF-DB 等面部表情数据集
- **注意**：DAIC-WOZ 数据太少，不适合直接训练

### 4.3 多模态融合模型

#### Mamba (S6 SSM, 2024)
- **核心特性**：线性复杂度 O(T)，可处理 2000+ 帧/utterance 的完整序列
- **输入**：T×d 的 utterance embedding 序列
- **输出**：session-level 表征 → 分类/回归头
- **开源实现**：`state-spaces/mamba` (GitHub) 或 `johnma2006/mamba-minimal`
- **在项目中的角色**：替代 MIL attention pooling，建模完整访谈时序

#### Cross-Attention Transformer
- **架构**：语音-文本交叉注意力，文本-面部交叉注意力
- **在项目中的角色**：多模态融合层

#### Contrastive Learning Head
- **方法**：NT-Xent loss / InfoNCE
- **在项目中的角色**：在融合前对不同模态做对比对齐

### 4.4 Agent / LLM 相关

| 工具 | 用途 | 备注 |
|:----|:-----|:-----|
| DeepSeek / GPT-4o-mini | 用于 LLM 症状抽取 (M2) | API 成本低 |
| Hermes Agent | 用于 M3 Agent 封装部署 | 已配置 |
| 多 Agent 辩论框架 | 用于 Multi-Agent Debate | 可参考 `Camel` / `AutoGen` |

---

## 五、可落地的技术路线图

```
当前项目 (2026)
    │
    ├── M0: XGBoost + 统计特征 (已完成)
    ├── M1: MIL + Attention Pooling (进行中)
    ├── M2: LLM 症状抽取 + 校准 (规划中)
    └── M3: Agent 蒸馏部署 (规划中)
          │
          ▼
论文发表后可追方向 (2026-2027)
    │
    ├── 短期 (0-3月，研究生阶段)
    │   ├── ❶ Mamba vs Attention Pooling 对比
    │   └── ❷ Multi-Agent Debate 原型
    │
    ├── 中期 (3-6月)
    │   ├── ❸ Whisper + LoRA 消融
    │   └── ❹ ST-GCN 面部建模
    │
    └── 长期 (6-12月)
        ├── ❺ Causal Multimodal Fusion
        └── ❻ 短文/workshop paper 投稿
```

---

## 六、与老师讨论时的问题建议

### 建议问老师的问题

1. **方向选择**：上面的 🥇🥈🥉 排序您觉得合理吗？有没有您更看好的方向？
2. **发表目标**：大创结项后，这些方向哪个更适合发短文/期刊？
3. **硬件资源**：Mamba 和 ST-GCN 训练需要 GPU，您这边能提供吗？
4. **合作可能**：如果有其他组也在做类似方向，可以合作吗？
5. **论文投稿建议**：您建议投哪个会议/期刊？（ACII, ICASSP, IEEE TAC, ACMMM?）

### 建议自己说的要点

- "我们现在的 baseline 是 XGBoost + 统计特征，AUC 约 0.76"
- "主模型是 MIL + Attention Pooling，用 wav2vec2/MentalBERT 提取特征"
- "创新点是用 LLM 从文本中按 PHQ-9 维度抽取症状特征，实现症状级可解释"
- "论文发表后我们想探索 Mamba 做时序建模，以及 Multi-Agent Debate 做可解释筛查"
- "这些方向我查过文献，目前 DAIC-WOZ 上基本没人做过"

---

## 附录：关键论文完整引用

### 数据集论文
1. Valstar, M., et al. "AVEC 2016: Depression, Mood, and Emotion Recognition Workshop and Challenge." *Proceedings of the 6th International Workshop on Audio/Visual Emotion Challenge*, 2016.
2. Gratch, J., et al. "E-DAIC: A New Corpus for Depression Detection." *AVEC Workshop*, 2019.

### 语音方向
3. Williamson, J., et al. "Detecting Depression using Vocal, Facial and Semantic Communication Cues." *Interspeech*, 2016.
4. Baevski, A., et al. "wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations." *NeurIPS*, 2020.
5. Hsu, W.-N., et al. "HuBERT: Self-Supervised Speech Representation Learning by Masked Prediction of Hidden Units." *IEEE/ACM TASLP*, 2021.
6. Chen, S., et al. "WavLM: Large-Scale Self-Supervised Pre-Training for Full Stack Speech Processing." *IEEE JSTSP*, 2023. arXiv:2110.13900
7. Radford, A., et al. "Robust Speech Recognition via Large-Scale Weak Supervision." *OpenAI*, 2023. (Whisper)

### 面部方向
8. Baltrusaitis, T., et al. "OpenFace 2.0: Facial Behavior Analysis Toolkit." *IEEE FG*, 2018.
9. Yan, S., et al. "Spatial Temporal Graph Convolutional Networks for Skeleton-Based Action Recognition." *AAAI*, 2018. (ST-GCN)
10. Tong, Z., et al. "VideoMAE: Masked Autoencoders are Data-Efficient Learners for Self-Supervised Video Pre-Training." *NeurIPS*, 2022. arXiv:2203.12602

### 文本方向
11. Devlin, J., et al. "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding." *NAACL*, 2019.
12. Ji, S., et al. "MentalBERT: Publicly Available Readymade Models for Mental Health Text Classification." *ACL*, 2022.
13. Huang, K., et al. "ClinicalBERT: A Language Model for Clinical Notes." *arXiv:1904.05342*, 2020.

### 多模态融合
14. Wang, Y., et al. "Multiple Instance Learning for Multimodal Depression Detection." *IEEE TAC*, 2023.
15. Muzi, A., et al. "SBT-Net: Semantic Bridge Transformer for Multimodal Depression Detection." *AAAI*, 2024.

### 新方向核心论文
16. Gu, A., & Dao, T. "Mamba: Linear-Time Sequence Modeling with Selective State Spaces." *arXiv:2312.00752*, 2024.
17. Du, Y., et al. "Improving Factuality and Reasoning in LLMs through Multi-Agent Debate." *arXiv:2305.14325*, 2024.
18. Hu, E., et al. "LoRA: Low-Rank Adaptation of Large Language Models." *ICLR*, 2022. arXiv:2106.09685
19. Radford, A., et al. "Learning Transferable Visual Models from Natural Language Supervision." *ICML*, 2021. (CLIP)
20. Pearl, J. "Causality: Models, Reasoning, and Inference." *Cambridge University Press*, 2009.

---

*文档生成日期：2026-05-13*
*本文档位于 GitHub: https://github.com/buzhidaoa8848-hash/depression-research-plan/blob/main/depression_teacher_report.md*
