---
name: sprint-milestone-check
description: 里程碑检查Skill — 在关键节点暂停工作流，汇总待决策事项，等待用户确认后继续。
---

# 里程碑检查

## 触发条件

- 需求分析完成后（用户确认需求范围）
- 架构设计完成后（用户确认技术方案）
- 质量评估分数 < 60时（用户决定是否继续迭代）
- 自动修复3轮后仍未通过时

## 执行步骤

### Step 1: 确定检查点类型

读取 `.sprint-workspace/sprint-state.md` 获取触发原因：

- `REQUIREMENTS` 完成 → 需求确认检查点
- `ARCHITECTURE` 完成 → 架构确认检查点
- `EVALUATION` 分数不足 → 质量决策检查点

### Step 2: 汇总待决策事项

读取 `.sprint-workspace/sprint-decisions.md`，收集所有 `状态: 待确认` 的事项。

### Step 3: 向用户呈现汇总

根据检查点类型，向用户展示：

**需求确认检查点**：
```
📋 需求确认检查点 — Sprint {N}

已完成需求分析，请审阅 .sprint-workspace/sprint-requirements.md

待决策事项：
{列出 .sprint-workspace/sprint-decisions.md 中的待确认项}

请确认：
1. 需求范围是否OK？
2. 以上决策项请逐一确认或给出你的选择
```

**架构确认检查点**：
```
🏗️ 架构确认检查点 — Sprint {N}

已完成架构设计，请审阅 .sprint-workspace/sprint-architecture.md

待决策事项：
{列出 .sprint-workspace/sprint-decisions.md 中的待确认项}

请确认：
1. 技术方案是否OK？
2. 以上决策项请逐一确认或给出你的选择
```

**质量决策检查点**：
```
⚠️ 质量决策检查点 — Sprint {N}

本轮评估得分: {分数}/100
已尝试修复: {N}轮

评估报告: .sprint-workspace/versions/sprint-{N}/evaluation.md

请选择：
1. 继续迭代（追加1-2轮修复）
2. 接受当前质量，进入复盘阶段
3. 回退到架构/规划阶段重新设计
```

### Step 4: 处理用户回复

- 用户确认通过 → 清除 `.sprint-workspace/sprint-decisions.md` 中已确认的事项，通知 sprint-phase-next 继续流转
- 用户提出修改 → 将修改意见追加到对应的文档，调度相关Agent进行修改
- 用户选择中止 → 记录决策，调用 sprint-close 结束Sprint

### Step 5: 更新状态文件

更新 `.sprint-workspace/sprint-state.md`：
- 清除 `checkpoints_pending` 中已处理的项
- 在 `phase_history` 中记录检查点处理结果
