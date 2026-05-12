# Claude Code 启动提示词

将此提示词粘贴给 Claude Code（在 `portfolio-workbench/02_Projects/daic-depression-screening/` 目录下执行 `claude` 后粘贴）。

---

你当前在 DAIC-WOZ / E-DAIC 抑郁风险筛查大创项目的根目录。我已读入 CLAUDE.md 和 EXECUTION_PLAN.md。

## 你的任务

严格按照 `EXECUTION_PLAN.md` 的 Phase 0 → Phase 1 → Phase 2 顺序执行，**不要跳阶段**。

## 执行规则

1. **每步先检查** — 跑之前先看产物是否已存在，已存在的跳过
2. **每步后报告** — 做了什么、创建了什么文件、是否通过验收
3. **遇到问题就修** — 报错了先看错误信息，自行修复后继续
4. **验收后再继续** — 每个 Phase 完成后问我"继续下一个 Phase？"等我确认
5. **不要瞎猜数据路径** — data/ 目录目前为空，所有数据相关代码必须兼容"data/ 下有 DAIC-WOZ 目录"的情况
6. **所有结果写 markdown ** — 表格都输出到 results/ 目录

## 开始执行

先从 Phase 0 Step 0.1 开始：创建 src/ 子目录结构。
每完成一步告诉我：这一步做了什么、输出了什么、下一步准备做什么。
