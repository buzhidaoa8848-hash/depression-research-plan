# DAIC-WOZ / E-DAIC 抑郁风险筛查 · 数据收集与整理指南

---

## 一、核心数据集

### 1.1 DAIC-WOZ（主训练集）

| 属性 | 说明 |
|:----|:------|
| **来源** | USC Institute for Creative Technologies, AVEC 2016 Challenge |
| **获取方式** | 填写学术使用申请 → https://dcapswoz.ict.usc.edu/ |
| **样本量** | 189 次访谈（107 训练 / 35 验证 / 47 测试） |
| **模态** | 视频、音频、转录文本、OpenFace 特征（2D/3D面部标记、眼动、头姿、HOG、AU）、COVAREP 声学特征 |
| **标签** | PHQ-8 分数 + 二分类标签（≥10 为抑郁） |
| **访谈形式** | 虚拟助手 Ellie 驱动的 semi-structured 访谈，平均 15-30 分钟 |
| **推荐用途** | 主训练集 + Baseline 实验 |

### 1.2 E-DAIC（扩展训练 + 跨域验证）

| 属性 | 说明 |
|:----|:------|
| **来源** | USC ICT, AVEC 2019 Challenge（DAIC-WOZ 扩展版） |
| **获取方式** | 同 DAIC-WOZ，提供特征包和分割列表 |
| **样本量** | 275 名受访者（163 训练 / 56 开发 / 56 测试） |
| **模态** | 头姿、AU、眼动、eGeMAPS 音频特征(88维)、MFCC、深度视觉特征 |
| **标签** | PHQ-8 + PCL-C 分数 |
| **与 DAIC-WOZ 兼容性** | **同源，可直接合并训练**（推荐） |
| **推荐用途** | 扩展训练集 + 跨域测试（DAIC→E-DAIC / E-DAIC→DAIC） |

---

## 二、外部辅助数据集（供对比/预训练/跨域验证）

### 2.1 可直接获取的（A级推荐）

| 数据集 | 模态 | 样本量 | 标签 | 获取方式 | 推荐用途 |
|:------|:----|:------:|:----:|:---------|:---------|
| **CMDC** | 音频+视频+文本 | 78 人 | PHQ-8 + 二分类 | IEEE DataPort 签EULA | 跨语言验证（中文） |
| **MODMA** | EEG+音频 | 52 人 | PHQ-9 | 官网注册签EULA | 多模态预训练（备选） |
| **Androids** | 音频 | 116 人 | 自陈诊断 | 公开下载（同意研究用途） | 声学预训练 |

### 2.2 可做文献对比的（B级推荐）

| 数据集 | 模态 | 不直接合并的原因 | 推荐用途 |
|:------|:----|:-----------------|:---------|
| **EATD-Corpus** | 音频+文本(短问答) | 非访谈型，仅3个情绪问答 | 文本预训练对比 |
| **eRisk 2017/2018** | 纯文本(Reddit) | 社交媒体非结构化，噪音大 | 文献对比/相关工作 |
| **CLPsych 2015** | 纯文本(Twitter) | 弱标签+域偏移 | 文献对比 |

### 2.3 不建议直接合并的数据集

| 数据集 | 不建议原因 |
|:------|:-----------|
| **D-Vlog / LMVD** | YouTube vlog，非访谈场景，弱标签噪声大 |
| **Pittsburgh Voice Diary** | 德语自然语音周记，域差异大 |
| **AViD-Corpus** | 朗读/计数/自述等任务，缺少转录文本 |

---

## 三、数据申请操作指南（发给队员）

### 3.1 E-DAIC 申请步骤

```
1. 打开 https://dcapswoz.ict.usc.edu/
2. 点击 "Request Access"
3. 填写学术申请表：
   - 项目名称：Multimodal Depression Risk Screening Based on DAIC-WOZ/E-DAIC
   - 机构：上海财经大学
   - 指导老师：[填写指导老师姓名]
   - 用途：非商业学术研究，抑郁风险辅助筛查
4. 提交后等待邮件回复（通常 1-2 周）
5. 如果被拒绝 → 使用公开的 E-DAIC Feature Pack（无需审批）
6. 拿到数据后解压并确认文件完整性
```

### 3.2 CMDC 申请步骤

```
1. 前往 IEEE DataPort
2. 搜索 "CMDC" 或 "Chinese Multimodal Depression Corpus"
3. 签署 EULA（终端用户协议）
4. 获取访问密码 → 下载数据
5. 确认包含：音频文件 + 视频文件 + 转录文本 + PHQ-8 标签
```

### 3.3 数据申请验收标准

```
□ E-DAIC 申请已提交（截图证明）
□ E-DAIC Feature Pack 已下载（作为备选）
□ CMDC 已获取访问权限
□ 数据解压后能列出文件结构和基本信息
□ 统计数据量：多少个 session，多少条 utterance
```

---

## 四、数据目录结构（建议）

```
depression-data/
├── DAIC-WOZ/
│   ├── train/
│   │   ├── 300/          # session_id
│   │   │   ├── 300_AUDIO.wav
│   │   │   ├── 300_VIDEO.mp4
│   │   │   ├── 300_TRANSCRIPT.csv
│   │   │   ├── OpenFace/
│   │   │   │   ├── 300.csv
│   │   │   │   └── 300_of_details.txt
│   │   │   └── COVAREP/
│   │   │       └── 300.csv
│   │   ├── 301/
│   │   └── ...
│   ├── dev/
│   └── test/
│
├── E-DAIC/
│   ├── train/
│   ├── dev/
│   └── test/
│
├── metadata/
│   ├── phq8_labels.csv
│   ├── session_manifest.csv
│   ├── utterance_index.csv
│   └── split_list.csv
│
└── external_datasets/
    ├── CMDC/
    ├── MODMA/
    └── Androids/
```

---

## 五、数据预处理流程

```
原始数据
    │
    ▼
Step 1: 解压 & 目录整理
    ├── DAIC-WOZ: 拆成 session→train/dev/test
    ├── E-DAIC: 同上
    └── 输出: 结构化目录
    │
    ▼
Step 2: 标签表提取
    ├── 从 PHQ_PHQ8_NoNa 提取 score + binary label
    ├── 每个 subject 一行
    └── 输出: phq8_labels.csv
    │
    ▼
Step 3: session → utterance 分段
    ├── 解析转录 CSV（Speaker, Value, StartTime, EndTime）
    ├── 过滤只保留 Participant（去掉 Ellie）
    ├── 每行一个 utterance
    └── 输出: utterance_index.csv
    │
    ▼
Step 4: 文件完整性检查
    ├── 确认每个 session 有音频/视频/OpenFace/COVAREP
    ├── 标记缺失文件
    └── 输出: missing_files_report.md
    │
    ▼
Step 5: 特征提取
    ├── wav2vec2/HuBERT → 768d/utterance（语音）
    ├── OpenFace AU/gaze/pose → 统计量（面部）
    ├── MentalBERT → 768d/utterance（文本）
    └── 输出: feature_embeddings/
    │
    ▼
Step 6: LLM 症状特征（可选，M2）
    ├── DeepSeek prompt → 9 维症状打分
    └── 输出: symptom_features.csv
    │
    ▼
Step 7: 数据就绪
    ├── 确认 subject-level split
    ├── 确认无 speaker/segment leakage
    └── 可以开始实验
```

---

## 六、数据核查清单

每次实验前必须检查：

- [ ] subject-level split：同一个人的所有数据在同一 fold
- [ ] 无 segment leakage：同一个人的 utterance 不会跨 train/test
- [ ] 已过滤 Ellie 语句：model 只看到 participant 的回答
- [ ] 标签分布：train/dev/test 的抑郁比接近
- [ ] 缺失文件已记录：不影响实验结论
- [ ] 特征维度已对齐：所有 session 特征维度一致

---

## 七、数据风险与应对

| 风险 | 应对 |
|:----|:------|
| E-DAIC 申请被拒 | 使用公开 Feature Pack，特征向量已提取 |
| 部分 session 缺少 OpenFace | 标记缺失，只用可用 session |
| 部分 session 缺少 COVAREP | 用 eGeMAPS 补充 |
| 音频/视频不同步 | 用转录时间戳对齐 |
| 标签不均衡（抑郁少） | class weight / focal loss / SMOTE |
| DAIC ↔ E-DAIC 域偏移 | 各自训练 → 交叉验证 |
