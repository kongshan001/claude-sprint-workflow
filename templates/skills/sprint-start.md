---
name: sprint-start
description: 启动新的Sprint迭代。读取全局进度，初始化状态机，调度PM Agent开始头脑风暴。
---

# Sprint 启动

## 触发条件

- 用户提出新的项目想法或需求
- 上一轮Sprint已关闭，需要启动下一轮
- `.sprint-workspace/sprint-state.md` 不存在或 `current_phase` 为空

## 执行步骤

1. **读取全局进度**
   - 读取 `.sprint-workspace/sprint-progress.md`
   - 如果文件不存在，创建初始版本：

```yaml
project: ""
total_sprints: 0
current_sprint: 0
status: NOT_STARTED
started_at: ""
overall_progress: "0%"

milestones: []

resource_usage:
  total_tokens: 0
  total_duration_sec: 0
  by_sprint: {}
```

2. **递增Sprint编号**
   - `current_sprint = total_sprints + 1`
   - `total_sprints = current_sprint`

3. **创建Sprint版本目录**
   - 创建 `.sprint-workspace/versions/sprint-{N}/`
   - 创建 `.sprint-workspace/versions/sprint-{N}/changes.md`（空模板）
   - 创建 `.sprint-workspace/versions/sprint-{N}/decisions.md`（空模板）
   - 创建 `.sprint-workspace/versions/sprint-{N}/metrics.md`（空模板）

4. **初始化状态机**
   - 写入 `.sprint-workspace/sprint-state.md`：

```yaml
project: "<从sprint-brainstorm.md提取项目名>"
current_sprint: {N}
current_phase: BRAINSTORM
sprint_goal: ""
phase_history: []
checkpoints_pending: []
human_decisions: []
```

5. **创建当前Sprint任务文件**
   - 写入 `.sprint-workspace/sprint-tasks.md`：

```yaml
sprint: {N}
sprint_goal: ""
tasks: []
```

6. **更新全局进度**
   - 更新 `.sprint-workspace/sprint-progress.md` 的 `current_sprint` 和 `total_sprints`

7. **调度 PM Agent**
   - 使用 Agent 工具，`subagent_type="sprint-pm"`（对应 `.claude/agents/sprint-pm.md`）
   - PM Agent 将读取 `.sprint-workspace/sprint-brainstorm.md` 开始头脑风暴
   - **禁止使用 general-purpose 或其他内置类型替代**
