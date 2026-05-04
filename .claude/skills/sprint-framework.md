---
name: sprint-framework
description: 自迭代Sprint框架 — 通过命令参数控制工作流（init/start/next/close/milestone）
---

# Sprint Framework

根据命令参数执行对应工作流。

## 命令分发

读取 Skill 的 args 参数，按以下规则分发：

- **无参数 或 `init`** → 执行「框架初始化」（见下文）
- **`start`** → 执行「启动 Sprint」
- **`next`** → 执行「阶段流转」
- **`close`** → 执行「Sprint 关闭」
- **`milestone`** → 执行「里程碑检查」

## 框架初始化（init）

将通用的自迭代工作流框架部署到当前项目。框架包含 7 个专职 Agent、1 个工作流 Skill、5 个行为规则，以及基于文件的状态机。

### 前置检查

1. **Git 仓库**：运行 `git rev-parse --is-inside-work-tree` 确认。如果不是，提示用户先初始化。
2. **Claude Code**：确认 `.claude/` 目录存在（如不存在则创建）。

### 交互收集信息

使用 AskUserQuestion 收集：

| 信息 | 默认值 | 用途 |
|------|--------|------|
| 项目名称 | 当前目录名 | 填入状态机文件，记为 `{{PROJECT_NAME}}` |
| 项目描述 | "待定" | 填入 progress.md，记为 `{{PROJECT_DESC}}` |

### 冲突检测

生成文件前检测所有目标文件。已存在的文件**跳过**并记录到冲突列表。不存在的目录用 `mkdir -p` 创建。

### 文件生成

框架模板存放在仓库的 `templates/` 目录下。读取模板文件，使用 Write 工具写入目标位置。冲突列表中的文件跳过。

#### Agent 文件 — `templates/agents/` → `.claude/agents/`

逐个读取 `templates/agents/` 下所有 `.md` 文件，写入 `.claude/agents/` 同名位置。共 7 个文件：
sprint-pm / sprint-researcher / sprint-architect / sprint-developer / sprint-qa / sprint-evaluator / sprint-knowledge-manager

#### Rule 文件 — `templates/rules/` → `.claude/rules/`

逐个读取写入。共 5 个文件：sprint-git-commit / sprint-doc-output / sprint-test-coverage / sprint-inbox-workflow / sprint-agent-dispatch

#### Workspace 文件 — `templates/workspace/` → `.sprint-workspace/`

逐个读取写入。写入前将 `{{PROJECT_NAME}}` 和 `{{PROJECT_DESC}}` 替换为交互收集的值。共 9 个文件：
sprint-state / sprint-progress / sprint-tasks / sprint-inbox / sprint-changelog / sprint-decisions / sprint-brainstorm / sprint-requirements / sprint-architecture

#### 目录结构

```bash
mkdir -p .sprint-workspace/research/sprint-1
mkdir -p .sprint-workspace/versions/sprint-1
mkdir -p .sprint-workspace/archive
```

### 初始化后指导

所有文件生成完成后，输出汇总：

```
Sprint Framework 初始化完成
已创建：
  - <列出实际创建的文件数量和类型>
  - .sprint-workspace/ 工作目录
已跳过（文件已存在）：
  - <列出跳过的文件>
下一步：
  1. 编辑 .sprint-workspace/sprint-brainstorm.md，写下你的项目想法
  2. 运行 /sprint-framework start 启动第一个 Sprint
进阶定制：
  - 添加自定义 Agent：在 .claude/agents/ 中创建新文件
  - 添加项目特定规则：在 .claude/rules/ 中创建新文件
  - 修改评估权重：编辑 references/evaluation-model.md
```

生成完成后执行 git commit：

```bash
git add .sprint-workspace/ .claude/agents/ .claude/skills/ .claude/rules/
git commit -m "feat: 初始化 Sprint Framework（由 /sprint-framework 自动生成）"
```

## 启动 Sprint（start）

### 触发条件

- 用户提出新的项目想法或需求
- 上一轮Sprint已关闭，需要启动下一轮
- `.sprint-workspace/sprint-state.md` 不存在或 `current_phase` 为空

### 执行步骤

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

## 阶段流转（next）

### 触发条件

- 任何Agent完成当前阶段工作后调用
- 用户重启后恢复执行时调用
- 自动修复循环后重新评估时调用

### 执行步骤

#### Step 1: 读取当前状态

读取 `.sprint-workspace/sprint-state.md`，获取 `current_phase` 和 `phase_history`。

#### Step 2: 阶段流转决策

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
  → 分数 >= 80: 更新 current_phase 为 RETROSPECTIVE，调度 Knowledge Manager
  → 分数 60-79: 启动修复循环（最多3轮）
    - 调度 Developer（根据评估报告修复）
    - 调度 QA（验证修复）
    - 调度 Evaluator（重新评估）
    - 如第3轮仍 < 80: 触发 milestone-check
  → 分数 < 60: 触发 milestone-check（等用户决策）

当前阶段为 RETROSPECTIVE 且已完成:
  → 执行「Sprint 关闭」（close 命令）
```

#### Step 3: 更新状态文件

每次流转时更新 `.sprint-workspace/sprint-state.md`：
- 更新 `current_phase`
- 在 `phase_history` 中追加已完成阶段的记录（含 agent、token_cost、duration_sec、output）
- 清除已处理的 `checkpoints_pending`

#### Step 4: 调度下一个Agent

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

#### Step 5: 恢复逻辑

如果是重启恢复场景：
1. 读取 `.sprint-workspace/sprint-state.md` 获取 `current_phase`
2. 如果 `phase_history` 最后一项的 phase 等于 `current_phase`，说明该阶段已完成，直接流转到下一阶段
3. 如果 `phase_history` 最后一项的 phase 不等于 `current_phase`，说明当前阶段尚未完成，重新调度对应Agent继续工作

## Sprint 关闭（close）

### 触发条件

- 复盘总结（RETROSPECTIVE）阶段完成后
- 用户在里程碑检查点选择中止Sprint

### 执行步骤

#### Step 1: 生成变更记录

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

#### Step 2: 记录技术决策

读取 `.sprint-workspace/sprint-decisions.md` 中本Sprint已确认的决策，写入 `.sprint-workspace/versions/sprint-{N}/decisions.md`。

#### Step 3: 汇总指标

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

#### Step 4: 更新变更日志索引

在 `.sprint-workspace/sprint-changelog.md` 顶部追加：

```markdown
## Sprint {N} ({日期}) — 已完成
详见 [.sprint-workspace/versions/sprint-{N}/changes.md](versions/sprint-{N}/changes.md)
```

#### Step 5: 执行归档

将当前Sprint的临时文件移入归档目录：
```bash
mkdir -p .sprint-workspace/archive/sprint-{N}
mv .sprint-workspace/versions/sprint-{N}/evaluation.md .sprint-workspace/archive/sprint-{N}/
```

重置活跃文件：
- `.sprint-workspace/sprint-tasks.md` → 清空为下轮Sprint模板

#### Step 6: 更新全局进度

更新 `.sprint-workspace/sprint-progress.md`：
- `status`: SPRINT_CLOSED
- `resource_usage.total_tokens` += 本Sprint消耗
- `resource_usage.total_duration_sec` += 本Sprint耗时
- `resource_usage.by_sprint.sprint-{N}`: 记录本Sprint数据
- `overall_progress`: 重新计算

#### Step 7: 判断下一步

- 如果还有未完成的需求（`.sprint-workspace/sprint-requirements.md` 中有未实现的功能）：
  → 向用户确认是否启动下一轮Sprint
  → 用户确认后执行「启动 Sprint」（start 命令）
- 如果所有需求已实现且评估通过：
  → 标记项目完成，输出总结报告

## 里程碑检查（milestone）

### 触发条件

- 需求分析完成后（用户确认需求范围）
- 架构设计完成后（用户确认技术方案）
- 质量评估分数 < 60时（用户决定是否继续迭代）
- 自动修复3轮后仍未通过时

### 执行步骤

#### Step 1: 确定检查点类型

读取 `.sprint-workspace/sprint-state.md` 获取触发原因：

- `REQUIREMENTS` 完成 → 需求确认检查点
- `ARCHITECTURE` 完成 → 架构确认检查点
- `EVALUATION` 分数不足 → 质量决策检查点

#### Step 2: 汇总待决策事项

读取 `.sprint-workspace/sprint-decisions.md`，收集所有 `状态: 待确认` 的事项。

#### Step 3: 向用户呈现汇总

根据检查点类型，向用户展示：

**需求确认检查点**：
```
需求确认检查点 — Sprint {N}
已完成需求分析，请审阅 .sprint-workspace/sprint-requirements.md
待决策事项：
{列出 .sprint-workspace/sprint-decisions.md 中的待确认项}
请确认：
1. 需求范围是否OK？
2. 以上决策项请逐一确认或给出你的选择
```

**架构确认检查点**：
```
架构确认检查点 — Sprint {N}
已完成架构设计，请审阅 .sprint-workspace/sprint-architecture.md
待决策事项：
{列出 .sprint-workspace/sprint-decisions.md 中的待确认项}
请确认：
1. 技术方案是否OK？
2. 以上决策项请逐一确认或给出你的选择
```

**质量决策检查点**：
```
质量决策检查点 — Sprint {N}
本轮评估得分: {分数}/100
已尝试修复: {N}轮
评估报告: .sprint-workspace/versions/sprint-{N}/evaluation.md
请选择：
1. 继续迭代（追加1-2轮修复）
2. 接受当前质量，进入复盘阶段
3. 回退到架构/规划阶段重新设计
```

#### Step 4: 处理用户回复

- 用户确认通过 → 清除 `.sprint-workspace/sprint-decisions.md` 中已确认的事项，执行「阶段流转」（next 命令）继续流转
- 用户提出修改 → 将修改意见追加到对应的文档，调度相关Agent进行修改
- 用户选择中止 → 记录决策，执行「Sprint 关闭」（close 命令）结束Sprint

#### Step 5: 更新状态文件

更新 `.sprint-workspace/sprint-state.md`：
- 清除 `checkpoints_pending` 中已处理的项
- 在 `phase_history` 中记录检查点处理结果
