# DAIC-WOZ / E-DAIC 抑郁风险筛查大创项目

## 项目一句话
基于 DAIC-WOZ / E-DAIC 多模态临床访谈数据，构建抑郁风险辅助筛查模型（二分类 + PHQ-8 回归）。

## 团队结构
| 角色 | 负责人 | 能力 |
|:----:|:------:|:----:|
| 项目负责人 | 我（用户） | 技术路线、主干代码、老师沟通 |
| A 成员 | 文献与数据集 | 论文阅读、数据集对比 |
| B 成员 | 数据工程 | 文件整理、缺失检查、manifest |
| C 成员 | 实验记录与文档 | 实验日志、结果表、PPT |

## 当前阶段
- [x] 项目文档骨架（depression_research_plan.md）
- [x] Phase 0：代码脚手架搭建（本地目录 `f:\depression_project`，非 src/ 结构，等价完成）
- [x] Phase 1：数据集解析与入库（DAIC-WOZ 20.2GB SQLite，E-DAIC 1fps 抽帧，标签清洗完成）
- [x] Phase 2：M0/M1 视觉基线（OpenFace 特征，5-fold CV，同域 + 跨域评估完成）← **已完成**
- [ ] Phase 3：主模型 / MIL ← **下一步**
- [ ] Phase 4：增强实验与消融（LLM 症状抽取、多任务、校准）
- [ ] Phase 5：论文、结项、答辩

## 关键约束
- DAIC-WOZ 数据需要学术申请，不在仓库里。**代码必须设计成「数据放 data/ 目录就能跑」**
- 训练集约 100 样本，极易过拟合。所有实验必须 subject-level 5-fold CV
- PHQ-8 >= 10 = 抑郁阳性（二分类阈值）
- 所有结果必须可复现：固定 random seed、记录实验参数
- 所有结果表格必须用 markdown 输出到 `results/` 目录

## 代码规范
- Python 3.10+, type hints
- `requirements.txt` + 虚拟环境
- 每个脚本有 `argparse` 或 `main()` 入口
- 可复现：`SEED=42` 全局固定
- 执行产物写入 `outputs/` 目录（.gitignored）
- 核心结果写入 `results/` 目录（git-tracked）

## 实际目录结构（本地 `f:\depression_project`）

```
f:\depression_project\
├── data/
│   ├── raw/{daicwoz,edaicwoz}/          ← 原始数据（不入库）
│   ├── interim/{daicwoz,edaicwoz}/
│   ├── processed/{daicwoz,edaicwoz,frames_1fps}/
│   └── metadata/
│       ├── daicwoz_first_batch.sqlite   ← 20.2GB，已入库
│       └── {daicwoz,edaicwoz}/
├── db/{schema,sqlite,exports}/
├── scripts/
│   ├── ingestion/    ← 数据入库脚本（Phase 1 等价）
│   ├── inspection/   ← 数据质量检查
│   ├── labels/       ← 标签清洗
│   ├── baseline/     ← M0/M1 基线实验（Phase 2 等价）
│   ├── preprocessing/
│   └── features/
├── reports/
│   ├── inspection/   ← 标签清洗产物
│   └── baseline/     ← 特征矩阵 CSV
├── outputs/
│   ├── m1_baseline_daic/
│   ├── m1_baseline_edaic/
│   ├── m1_baseline_summary/
│   └── daicwoz_final_baseline/
├── processed/frames_1fps/               ← E-DAIC 1fps 抽帧
└── .venv/
```

> 注：`data/`、`outputs/`、`.venv/`、`*.sqlite` 不入 git。

## 与 Hermes 的协作方式
- Hermes 负责：生成执行计划、拆解任务、提供 context
- Claude Code 负责：实际写代码、跑实验、输出结果
- 我（用户）负责：提供数据、确认方向、判断结果
