---
name: sprint-inbox-workflow
description: 用户问题 = 待办工作流 — 写入inbox→处理→归档→执行
---

# Rule: 用户问题 = 待办工作流

## 触发条件

用户在开发过程中提出问题、质疑、观察或改进建议。

## 规则

1. 即时写入 `.sprint-workspace/sprint-inbox.md`。
2. 用户说"处理inbox"时启动处理流程。
3. 逐条分析转化为 `.sprint-workspace/sprint-tasks.md` 中的正式任务。
4. 已处理内容归档到 `.sprint-workspace/archive/inbox/sprint-N.md`。
5. 调度对应 Agent 执行。
6. 验证问题已解决。
7. 更新状态文件。

## 判定标准

- 口头回答后跳过（未写入inbox）= 违规
- inbox条目未转化为正式任务 = 违规
