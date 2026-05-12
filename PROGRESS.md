# 进度报告

> 最后更新：2026-05-12

## 当前阶段：M1 视觉基线 ✅ 已完成

---

## 已完成工作

### 数据入库与清洗
- DAIC-WOZ 原始数据入库 → `daicwoz_first_batch.sqlite`（20.2 GB，失败率 ≈ 0）
- E-DAIC-WOZ 视频抽帧 1fps 完成
- 标签清洗 → `reports/inspection/daic_label_master_clean.csv`（189 条）

### M1 视觉单模态基线（OpenFace 特征）

| 场景 | 最佳 config | Acc | AUC | F1 |
|------|------------|-----|-----|----|
| DAIC 同域 | pose_only | 0.660 | 0.712 | 0.529 |
| DAIC → eDAIC 跨域 | pose_only | 0.712 | 0.599 | 0.364 |
| eDAIC 同域 | pose_only | 0.614 | 0.624 | 0.414 |

- 6 种 feature config 对比：`gaze_only / pose_only / au_only / au_pose / au_gaze / all_visual`
- 5-fold CV 最终基线：`pose_only` AUC 0.647 ± 0.197
- 跨域泛化明显下降（AUC 0.712 → 0.599），符合预期

---

## 待完成

- [ ] M2 音频单模态基线（COVAREP / FORMANT 特征）
- [ ] M2 文本单模态基线（transcript TF-IDF / 轻量 encoder）
- [ ] 多模态融合（early / late fusion）
- [ ] 深度学习 / 预训练模型实验
- [ ] 最终报告撰写
