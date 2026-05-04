---
name: sprint-close
description: Sprint关闭Skill — 执行归档、更新全局进度、记录版本，判断是否启动下一轮Sprint。
---

# Sprint 关闭

## 触发条件

- 复盘总结（RETROSPECTIVE）阶段完成后
- 用户在里程碑检查点选择中止Sprint

## 执行步骤

### Step 1: 生成变更记录

读取当前Sprint的所有产出，编写 `.sprint-workspace/versions/sprint-{N}/changes.md`：

```markdown
# Sprint {N} 变更记录

## 新增
- <新增的功能/文件>

## 修改
- <修改的内容>

## 修复
- <修复的问题>

## 移除
- <移除的功能/文件>
```

### Step 2: 记录技术决策

读取 `.sprint-workspace/sprint-decisions.md` 中本Sprint已确认的决策，写入 `.sprint-workspace/versions/sprint-{N}/decisions.md`。

### Step 3: 汇总指标

读取评估报告和任务文件，编写 `.sprint-workspace/versions/sprint-{N}/metrics.md`：

```yaml
sprint: {N}
evaluation_score: XX
test_pass_rate: XX%
feature_coverage: XX%
total_tasks: X
completed_tasks: Y
token_cost: XXXXX
duration_sec: XXXX
```

### Step 4: 更新变更日志索引

在 `.sprint-workspace/sprint-changelog.md` 顶部追加：

```markdown
## Sprint {N} ({日期}) — 已完成
详见 [.sprint-workspace/versions/sprint-{N}/changes.md](versions/sprint-{N}/changes.md)
```

### Step 5: 执行归档

将当前Sprint的临时文件移入归档目录：
```bash
mkdir -p .sprint-workspace/archive/sprint-{N}
mv .sprint-workspace/versions/sprint-{N}/evaluation.md .sprint-workspace/archive/sprint-{N}/
```

重置活跃文件：
- `.sprint-workspace/sprint-tasks.md` → 清空为下轮Sprint模板

### Step 6: 更新全局进度

更新 `.sprint-workspace/sprint-progress.md`：
- `status`: SPRINT_CLOSED
- `resource_usage.total_tokens` += 本Sprint消耗
- `resource_usage.total_duration_sec` += 本Sprint耗时
- `resource_usage.by_sprint.sprint-{N}`: 记录本Sprint数据
- `overall_progress`: 重新计算

### Step 7: 判断下一步

- 如果还有未完成的需求（`.sprint-workspace/sprint-requirements.md` 中有未实现的功能）：
  → 向用户确认是否启动下一轮Sprint
  → 用户确认后调用 `/sprint-start`
- 如果所有需求已实现且评估通过：
  → 标记项目完成，输出总结报告
