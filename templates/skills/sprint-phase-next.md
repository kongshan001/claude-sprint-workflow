---
name: sprint-phase-next
description: 核心编排Skill — 在每个阶段完成后流转到下一阶段，调度对应Agent，处理检查点和恢复逻辑。
---

# 阶段流转（核心编排）

## 触发条件

- 任何Agent完成当前阶段工作后调用
- 用户重启后恢复执行时调用
- 自动修复循环后重新评估时调用

## 执行步骤

### Step 1: 读取当前状态

读取 `.sprint-workspace/sprint-state.md`，获取 `current_phase` 和 `phase_history`。

### Step 2: 阶段流转决策

根据 `current_phase` 确定下一步：

```
当前阶段为 BRAINSTORM 且已完成:
  → 更新 current_phase 为 RESEARCH
  → 调度 Researcher Agent（读取 sprint-brainstorm.md，进行调研）

当前阶段为 RESEARCH 且已完成:
  → 更新 current_phase 为 REQUIREMENTS
  → 调度 PM Agent（读取 sprint-brainstorm.md + research/，编写 sprint-requirements.md）
  → 标记 checkpoints_pending: ["requirements"]

当前阶段为 REQUIREMENTS 且已完成:
  → 触发 milestone-check（需求确认检查点）
  → 等待用户确认后继续

用户已确认需求:
  → 更新 current_phase 为 ARCHITECTURE
  → 调度 Architect Agent（读取 sprint-requirements.md + research/，编写 sprint-architecture.md）
  → 标记 checkpoints_pending: ["architecture"]

当前阶段为 ARCHITECTURE 且已完成:
  → 触发 milestone-check（架构确认检查点）
  → 等待用户确认后继续

用户已确认架构:
  → 更新 current_phase 为 PLANNING
  → 调度 PM Agent（读取 sprint-requirements.md + sprint-architecture.md，编写 sprint-tasks.md）

当前阶段为 PLANNING 且已完成:
  → 更新 current_phase 为 EXECUTION
  → 调度 Developer Agent（读取 sprint-tasks.md + sprint-architecture.md，逐个实现任务）

当前阶段为 EXECUTION 且已完成:
  → 更新 current_phase 为 REVIEW
  → 调度 QA Agent（读取代码 + sprint-tasks.md，编写测试并执行）

当前阶段为 REVIEW 且已完成:
  → 更新 current_phase 为 EVALUATION
  → 调度 Evaluator Agent（读取代码 + sprint-test-report.md + sprint-tasks.md，评估打分）

当前阶段为 EVALUATION 且已完成:
  → 读取评估分数
  → 分数 ≥ 80: 更新 current_phase 为 RETROSPECTIVE，调度 Knowledge Manager
  → 分数 60-79: 启动修复循环（最多3轮）
    - 调度 Developer（根据评估报告修复）
    - 调度 QA（验证修复）
    - 调度 Evaluator（重新评估）
    - 如第3轮仍 < 80: 触发 milestone-check
  → 分数 < 60: 触发 milestone-check（等用户决策）

当前阶段为 RETROSPECTIVE 且已完成:
  → 调用 sprint-close Skill
```

### Step 3: 更新状态文件

每次流转时更新 `.sprint-workspace/sprint-state.md`：
- 更新 `current_phase`
- 在 `phase_history` 中追加已完成阶段的记录（含 agent、token_cost、duration_sec、output）
- 清除已处理的 `checkpoints_pending`

### Step 4: 调度下一个Agent

**必须使用 `Agent` 工具，通过 `subagent_type` 参数指定 `.claude/agents/` 中定义的 Agent 类型名：**

```
当前阶段 → 下一阶段     → subagent_type 值         → 输入文件
BRAINSTORM → RESEARCH   → "sprint-researcher"      → sprint-brainstorm.md
RESEARCH → REQUIREMENTS → "sprint-pm"              → sprint-brainstorm.md + research/
REQUIREMENTS → ARCHITECTURE → "sprint-architect"   → sprint-requirements.md + research/
ARCHITECTURE → PLANNING → "sprint-pm"              → sprint-requirements.md + sprint-architecture.md
PLANNING → EXECUTION    → "sprint-developer"       → sprint-tasks.md + sprint-architecture.md
EXECUTION → REVIEW      → "sprint-qa"              → src/ + sprint-tasks.md
REVIEW → EVALUATION     → "sprint-evaluator"       → src/ + sprint-test-report.md + sprint-tasks.md
EVALUATION → RETROSPECTIVE → "sprint-knowledge-manager" → 全部产出物
```

调度示例：
```
Agent(subagent_type="sprint-developer", description="编码实现", prompt="读取 sprint-tasks.md 和 sprint-architecture.md...")
```

传入上下文：
- 告知 Agent 当前 Sprint 编号和阶段
- 指示 Agent 读取哪些文件作为输入
- 指示 Agent 产出哪些文件
- 指示 Agent 完成后如何更新状态文件

**禁止使用 `general-purpose` 或外部工具链替代项目 Agent。**

### Step 5: 恢复逻辑

如果是重启恢复场景：
1. 读取 `.sprint-workspace/sprint-state.md` 获取 `current_phase`
2. 如果 `phase_history` 最后一项的 phase 等于 `current_phase`，说明该阶段已完成，直接流转到下一阶段
3. 如果 `phase_history` 最后一项的 phase 不等于 `current_phase`，说明当前阶段尚未完成，重新调度对应Agent继续工作
