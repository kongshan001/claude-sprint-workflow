---
name: sprint-doc-output
description: 每个工作流阶段必须有文档产出 — 文档缺失等于阶段未完成
---

# Rule: 每个阶段必须有文档产出

## 触发条件

工作流阶段完成时。

## 规则

1. 每个阶段完成后必须产出对应文档（参见下表）。
2. 阶段完成 = 文档产出 + 状态文件更新 + Git 提交，三者缺一不可。
3. 文档缺失 = 阶段未完成，不允许进入下一阶段。

| 阶段 | 产出文件 |
|------|----------|
| 头脑风暴 | .sprint-workspace/sprint-brainstorm.md |
| 调研 | .sprint-workspace/research/sprint-N/*.md |
| 需求分析 | .sprint-workspace/sprint-requirements.md |
| 架构设计 | .sprint-workspace/sprint-architecture.md |
| 任务规划 | .sprint-workspace/sprint-tasks.md |
| 编码实现 | 可运行代码 + 单元测试 |
| 测试评审 | 测试报告 |
| 质量评估 | .sprint-workspace/versions/sprint-N/evaluation.md |
| 复盘总结 | .sprint-workspace/versions/sprint-N/retrospective.md |

## 判定标准

- 阶段完成但无对应文档 = 违规
- 有文档但未更新状态文件 = 违规
- 文档存在但未提交 Git = 违规
