# 🎯 大创项目执行计划 — Claude Code 执行版

> 最后更新：2026-05-12
> 每个阶段有：目标 → 具体步骤 → 输出 → 验收标准 → 对你有什么用

---

## Phase 0: 代码脚手架搭建 ⚡

### 目标
搭好完整项目骨架，装好依赖，确保「数据放 data/ 就能跑实验」。

### 具体步骤

#### Step 0.1 — 目录初始化
```bash
# 你所在目录已经是 daic-depression-screening/
# 我已经创建了 CLAUDE.md 和 EXECUTION_PLAN.md
# 现在你帮我：
mkdir -p src/data src/features src/models src/utils
mkdir -p notebooks results outputs docs
```

#### Step 0.2 — 依赖文件
创建 `requirements.txt`：
```
# 核心
numpy>=1.24
pandas>=2.0
scikit-learn>=1.3
scipy>=1.10

# 可视化
matplotlib>=3.7
seaborn>=0.12

# 实验记录
pyyaml>=6.0

# 文本（后续用）
transformers>=4.30  # 可选，先注释掉
torch>=2.0          # 可选，先注释掉
```

#### Step 0.3 — Python 项目骨架

创建以下空文件（带注释骨架）：
- `src/__init__.py`
- `src/data/__init__.py`
- `src/features/__init__.py`
- `src/models/__init__.py`
- `src/utils/__init__.py`

创建 `src/config.py` — 全局配置：
```python
"""全局配置：路径、随机种子、模型参数"""
import os
from pathlib import Path

# 项目根目录
ROOT = Path(__file__).parent.parent

# 数据目录（手动放入原始数据）
DATA_DIR = ROOT / "data"
RAW_DIR = DATA_DIR / "raw"
PROCESSED_DIR = DATA_DIR / "processed"

# 输出目录
OUTPUTS_DIR = ROOT / "outputs"
RESULTS_DIR = ROOT / "results"
MODELS_DIR = OUTPUTS_DIR / "models"
LOGS_DIR = OUTPUTS_DIR / "logs"

# 随机性控制
SEED = 42

# 数据集配置
PHQ8_THRESHOLD = 10  # >=10 = 抑郁阳性
N_FOLDS = 5          # subject-level CV 折数

# 创建目录
for d in [RAW_DIR, PROCESSED_DIR, OUTPUTS_DIR, RESULTS_DIR, MODELS_DIR, LOGS_DIR]:
    d.mkdir(parents=True, exist_ok=True)
```

创建 `src/utils/seed.py` — 随机种子固定：
```python
"""全局随机种子固定"""
import random
import numpy as np

def set_seed(seed: int = 42):
    random.seed(seed)
    np.random.seed(seed)
```

#### Step 0.4 — gitignore
创建 `.gitignore`：
```
__pycache__/
*.pyc
*.egg-info/
.venv/
venv/
data/
outputs/
*.ipynb_checkpoints/
.DS_Store
```

#### Step 0.5 — 验证
运行 `python -c "from src.config import SEED; print(f'Config loaded, SEED={SEED}')"` 确认无报错。

### ✅ Phase 0 验收标准
- [ ] `python -c "from src.config import ..."` 无报错
- [ ] `requirements.txt` 完整
- [ ] 目录结构符合 CLAUDE.md 描述
- [ ] `pip install -r requirements.txt` 全绿

### 📦 产出
可执行的项目骨架 + 依赖环境

---

## Phase 1: DAIC-WOZ 数据解析管线 🗂️

### 目标
写代码解析 DAIC-WOZ 原始数据 → 统一的 participant × feature 矩阵，可喂给 sklearn。

### 前提
用户已将 DAIC-WOZ.zip 放入 `data/raw/` 目录。

### 具体步骤

#### Step 1.1 — DAIC-WOZ 原始结构分析
DAIC-WOZ 典型结构：
```
data/raw/
├── daic-woz/
│   ├── Train/
│   │   ├── 300/
│   │   │   ├── 300_AU.csv          # OpenFace Action Units
│   │   │   ├── 300_CLNF.txt        # OpenFace 追踪状态
│   │   │   ├── 300_COVAREP.csv     # 音频特征
│   │   │   ├── 300_TRANSCRIPT.csv  # 转录文本
│   │   │   └── 300_PHQ_AUX.csv     # PHQ-8 标签
│   │   └── 301/ ...
│   ├── Dev/ ...
│   └── Test/ ...
```

写 `src/data/daic_woz_parser.py`，功能：
1. 扫描目录结构，识别所有 participant session
2. 读取 PHQ_AUX.csv 提取 PHQ-8 分数
3. 读取 TRANSCRIPT.csv 提取语句（区分 Ellie vs participant）
4. 读取 AU.csv 提取 OpenFace 逐帧特征
5. 输出 participant 级汇总统计：缺失模态、文件大小、时长

#### Step 1.2 — Session Manifest 生成
写 `src/data/build_manifest.py`，输出 `results/session_manifest.csv`：
| participant_id | split | phq8_score | phq8_binary | has_audio | has_video | has_transcript | has_openface | has_covarep | notes |

#### Step 1.3 — 标签表
写 `src/data/build_labels.py`，输出 `results/phq8_labels.csv`：
| participant_id | split | phq8_score | phq8_binary |

#### Step 1.4 — 数据质量报告
写 `src/data/quality_report.py`，输出 `results/quality_report.md`：
- 总样本数
- 各 split 样本数
- 抑郁阳性率
- 缺失模态统计
- PHQ-8 分数分布（直方图）

#### Step 1.5 — 数据加载器
写 `src/data/loader.py`，提供：
```python
def load_labels() -> pd.DataFrame:      # 返回 label 表
def load_openface(pid: str) -> pd.DataFrame:  # 返回 AU 时间序列
def load_transcript(pid: str) -> pd.DataFrame: # 返回 transcript
def get_available_participants() -> List[str]: # 可用的 participant 列表
```

### ✅ Phase 1 验收标准
- [ ] `python src/data/build_manifest.py` → `results/session_manifest.csv` 生成成功
- [ ] `python src/data/build_labels.py` → `results/phq8_labels.csv` 生成成功
- [ ] `python src/data/quality_report.py` → `results/quality_report.md` 有实际内容
- [ ] 能在 Python 中 `from src.data.loader import load_labels` 并拿到 DataFrame

### 📦 产出
数据解析管线 + 3 个结果文件（manifest、标签表、质量报告）

---

## Phase 2: Baseline 实验 🧪

### 目标
用 OpenFace AU 聚合特征构建最简单的分类/回归模型。**目标是跑通全流程，不是刷分。**

### 具体步骤

#### Step 2.1 — 特征聚合器
写 `src/features/openface_aggregator.py`：
```python
def aggregate_openface(au_df: pd.DataFrame) -> dict:
    """将逐帧 AU 聚合为 participant 级特征向量"""
    # 对每类 AU 计算：mean, std, min, max, range, slope
    # 额外：gaze_angle 的 mean/std, pose_Rx/Ry/Rz 的 mean/std
    # 返回 {feature_name: value} dict
```

#### Step 2.2 — 特征矩阵构建
写 `src/features/build_feature_matrix.py`：
- 遍历所有 participant
- 对每个 participant 调用 `aggregate_openface()`
- 合并为统一的 feature × participant 矩阵
- 输出 `results/openface_feature_matrix.csv`

#### Step 2.3 — Baseline 分类器
写 `src/models/baseline_classifier.py`：
- 输入：特征矩阵 + 标签（二分类）
- 模型：LogisticRegression, RandomForestClassifier, SVC
- 评估：subject-level 5-fold CV
- 指标：Accuracy, Precision, Recall, F1, ROC-AUC
- 输出：`results/baseline_classification_results.md`

关键代码逻辑：
```python
from sklearn.model_selection import StratifiedKFold
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_auc_score

# StratifiedKFold(n_splits=5, shuffle=True, random_state=SEED)
# 在训练集内做 5-fold CV（不对 test set 做任何操作）
# 输出格式要求见验收标准
```

#### Step 2.4 — Baseline 回归器
写 `src/models/baseline_regressor.py`：
- 输入：特征矩阵 + 标签（PHQ-8 连续值）
- 模型：LinearRegression, RandomForestRegressor, SVR
- 评估：5-fold CV
- 指标：RMSE, MAE, R², Pearson r
- 输出：`results/baseline_regression_results.md`

#### Step 2.5 — 特征重要性分析
写 `src/models/feature_importance.py`：
- 用 RandomForest 的 feature_importances_
- 输出 top-20 特征排名
- 输出 AU04（皱眉）和 AU12（微笑）的具体数值
- 输出：`results/feature_importance.md`

#### Step 2.6 — 一键运行脚本
创建 `run_baseline.sh`：
```bash
#!/bin/bash
set -e
echo "=== Phase 2: Baseline Experiments ==="
echo "[1/4] Building feature matrix..."
python src/features/build_feature_matrix.py
echo "[2/4] Classification baseline..."
python src/models/baseline_classifier.py
echo "[3/4] Regression baseline..."
python src/models/baseline_regressor.py
echo "[4/4] Feature importance..."
python src/models/feature_importance.py
echo "=== Done ==="
```

### ✅ Phase 2 验收标准
命令行执行 `bash run_baseline.sh` 后，`results/` 下应有：
- [ ] `baseline_classification_results.md`: 包含每种模型的 Accuracy / Precision / Recall / F1 / ROC-AUC（5-fold mean ± std 格式）
- [ ] `baseline_regression_results.md`: 包含每种模型的 RMSE / MAE / R² / Pearson r
- [ ] `feature_importance.md`: top-20 特征 + AU04/AU12 具体值
- [ ] **所有结果使用相同 participant 划分，可复现**

### 📦 产出
完整可复现的 baseline 实验管线 + 结果报告

---

## Phase 3: 多模态扩展（后续执行）

只有在 Phase 2 完成后才进入此阶段：
1. 集成 COVAREP 音频特征（eGeMAPS）
2. 集成 BERT 文本 embedding
3. 特征级融合 baseline
4. 对比 single modality vs fusion

---

## 执行顺序总结

```
Phase 0: 脚手架 (15-30 min)
    ↓
Phase 1: 数据解析 (30-60 min, 需 data/ 有文件)
    ↓ 验收后
Phase 2: Baseline (30-60 min)
    ↓ 验收后
Phase 3+: 扩展实验
```

## 每个 checkpoint 的行为

每完成一个 Step，Claude Code 应该：
1. 输出该 step 执行摘要（做了什么 / 创建了什么文件）
2. 验证产物是否存在（文件检查）
3. 报告是否通过该 step 的验收标准
4. 问用户：继续下一步还是停下来检查？
