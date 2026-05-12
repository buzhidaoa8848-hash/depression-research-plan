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
- [x] 项目文档骨架（depression_screening.md）
- [ ] Phase 0：代码脚手架搭建 ← **就是现在**
- [ ] Phase 1：数据集解析与文件结构
- [ ] Phase 2：Baseline 实验
- [ ] Phase 3：主模型 / MIL
- [ ] Phase 4：增强实验与消融
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

## 重要目录结构
```
daic-depression-screening/
├── CLAUDE.md              ← 当前文件
├── EXECUTION_PLAN.md      ← 执行计划（逐阶段可验收）
├── requirements.txt       ← 依赖
├── data/                  ← 手动放置原始数据（不提交）
├── src/                   ← 源代码
│   ├── data/              ← 数据加载与预处理
│   ├── features/          ← 特征提取与聚合
│   ├── models/            ← 模型定义与训练
│   └── utils/             ← 工具函数
├── notebooks/             ← 分析用 Jupyter notebook
├── results/               ← 结果表格、图表（git-tracked）
├── outputs/               ← 实验产物（.gitignored）
└── docs/                  ← 交付文档、论文草稿
```

## 与 Hermes 的协作方式
- Hermes 负责：生成执行计划、拆解任务、提供 context
- Claude Code 负责：实际写代码、跑实验、输出结果
- 我（用户）负责：提供数据、确认方向、判断结果
