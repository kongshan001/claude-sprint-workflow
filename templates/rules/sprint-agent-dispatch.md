---
name: sprint-agent-dispatch
description: 项目工作流必须使用项目Agent — 禁止用内置类型替代
---

# Rule: 项目工作流必须使用项目 Agent

## 触发条件

执行项目工作流的任何阶段。

## 规则

1. 每个阶段必须通过 `.claude/agents/` 中定义的 Agent 执行。
2. 使用 Agent 工具的 subagent_type 参数指定 Agent 类型名。
3. Sprint 关闭时，检查阶段历史确认全部 Agent 角色至少被调度一次。
4. 即使某角色无实质工作，也需在复盘中明确记录原因。

| 阶段 | Agent | subagent_type |
|------|-------|---------------|
| 头脑风暴 | PM | sprint-pm |
| 调研 | Researcher | sprint-researcher |
| 需求分析 | PM | sprint-pm |
| 架构设计 | Architect | sprint-architect |
| 任务规划 | PM | sprint-pm |
| 编码实现 | Developer | sprint-developer |
| 测试评审 | QA | sprint-qa |
| 质量评估 | Evaluator | sprint-evaluator |
| 复盘总结 | Knowledge Manager | sprint-knowledge-manager |

## 判定标准

- 使用 general-purpose 替代项目 Agent = 违规
- 跳过阶段导致 Agent 从未被调度 = 违规
