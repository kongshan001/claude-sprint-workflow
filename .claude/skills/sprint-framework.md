---
name: sprint-framework
description: 自迭代Sprint框架初始化 — 一键部署9阶段工作流、7个专职Agent和文件状态机到任意项目。
---

# Sprint Framework 初始化

将通用的自迭代工作流框架部署到当前项目。框架包含 7 个专职 Agent、4 个工作流 Skill、5 个行为规则，以及基于文件的状态机。

## 前置检查

1. **Git 仓库**：运行 `git rev-parse --is-inside-work-tree` 确认。如果不是，提示用户先初始化。
2. **Claude Code**：确认 `.claude/` 目录存在（如不存在则创建）。

## 交互收集信息

使用 AskUserQuestion 收集：

| 信息 | 默认值 | 用途 |
|------|--------|------|
| 项目名称 | 当前目录名 | 填入状态机文件，记为 `{{PROJECT_NAME}}` |
| 项目描述 | "待定" | 填入 progress.md，记为 `{{PROJECT_DESC}}` |

## 冲突检测

生成文件前检测所有目标文件。已存在的文件**跳过**并记录到冲突列表。不存在的目录用 `mkdir -p` 创建。

## 文件生成

框架模板存放在仓库的 `templates/` 目录下。读取模板文件，使用 Write 工具写入目标位置。冲突列表中的文件跳过。

### Agent 文件 — `templates/agents/` → `.claude/agents/`

逐个读取 `templates/agents/` 下所有 `.md` 文件，写入 `.claude/agents/` 同名位置。共 7 个文件：
sprint-pm / sprint-researcher / sprint-architect / sprint-developer / sprint-qa / sprint-evaluator / sprint-knowledge-manager

### Skill 文件 — `templates/skills/` → `.claude/skills/`

逐个读取写入。共 4 个文件：sprint-start / sprint-phase-next / sprint-close / sprint-milestone-check

### Rule 文件 — `templates/rules/` → `.claude/rules/`

逐个读取写入。共 5 个文件：sprint-git-commit / sprint-doc-output / sprint-test-coverage / sprint-inbox-workflow / sprint-agent-dispatch

### Workspace 文件 — `templates/workspace/` → `.sprint-workspace/`

逐个读取写入。写入前将 `{{PROJECT_NAME}}` 和 `{{PROJECT_DESC}}` 替换为交互收集的值。共 9 个文件：
sprint-state / sprint-progress / sprint-tasks / sprint-inbox / sprint-changelog / sprint-decisions / sprint-brainstorm / sprint-requirements / sprint-architecture

### 目录结构

```bash
mkdir -p .sprint-workspace/research/sprint-1
mkdir -p .sprint-workspace/versions/sprint-1
mkdir -p .sprint-workspace/archive
```

## 初始化后指导

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
  2. 运行 /sprint-start 启动第一个 Sprint

进阶定制：
  - 添加自定义 Agent：在 .claude/agents/ 中创建新文件
  - 添加项目特定规则：在 .claude/rules/ 中创建新文件
  - 修改评估权重：编辑 .claude/agents/sprint-evaluator.md
```

生成完成后执行 git commit：

```bash
git add .sprint-workspace/ .claude/agents/ .claude/skills/ .claude/rules/
git commit -m "feat: 初始化 Sprint Framework（由 /sprint-framework 自动生成）"
```
