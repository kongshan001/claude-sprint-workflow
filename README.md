# Claude Sprint Workflow

一个通用的自迭代 Claude Code 工作流框架，基于文件状态机协调多个专职 Agent，自主驱动从头脑风暴到复盘总结的全流程。

## 快速开始

### 1. 复制框架到你的项目

```bash
# 复制编排器 Skill
mkdir -p your-project/.claude/skills
cp .claude/skills/sprint-framework.md your-project/.claude/skills/

# 复制模板目录（编排器会从中读取模板生成文件）
cp -r templates/ your-project/templates/
```

### 2. 在项目中初始化框架

在 Claude Code 中打开你的项目，输入：

```
/sprint-framework
```

框架会交互式收集项目信息，读取 `templates/` 下的模板，自动生成所有必要文件。

### 3. 启动第一个 Sprint

编辑 `.sprint-workspace/sprint-brainstorm.md` 写下你的项目想法，然后运行：

```
/sprint-start
```

> 初始化完成后可删除 `templates/` 目录，模板已写入 `.claude/` 对应位置。

## 工作流概览

```
头脑风暴 → 调研 → 需求分析 → 架构设计 → 任务规划
→ 编码实现 → 测试评审 → 质量评估 → 复盘总结
```

每个阶段由专职 Agent 负责，通过文件状态机（`.sprint-workspace/sprint-state.md`）协调流转。

## 包含内容

### 7 个专职 Agent

| Agent | 职责 | 阶段 |
|-------|------|------|
| PM | 需求分析、头脑风暴、任务拆解 | 头脑风暴 / 需求分析 / 任务规划 |
| Researcher | 外部调研、技术选型 | 调研 |
| Architect | 架构设计、接口定义 | 架构设计 |
| Developer | 编码实现、Bug 修复 | 编码实现 |
| QA | 测试编写、执行、Bug 报告 | 测试评审 |
| Evaluator | 质量评估、加权评分 | 质量评估 |
| Knowledge Manager | 复盘总结、知识沉淀 | 复盘总结 |

### 4 个工作流 Skill

| Skill | 作用 |
|-------|------|
| sprint-start | 启动新的 Sprint 迭代 |
| sprint-phase-next | 核心编排 — 阶段流转引擎 |
| sprint-close | Sprint 关闭、归档、版本记录 |
| sprint-milestone-check | 里程碑检查点，暂停等用户确认 |

### 5 个行为规则

| Rule | 作用 |
|------|------|
| sprint-git-commit | 每次迭代必提交 Git |
| sprint-doc-output | 每个阶段必须有文档产出 |
| sprint-test-coverage | 代码修改必须补充单元测试 |
| sprint-inbox-workflow | 用户反馈 → 待办工作流 |
| sprint-agent-dispatch | 工作流必须使用项目 Agent |

## 设计特点

- **渐进式披露** — 编排器仅 87 行，模板按需加载，每个文件职责单一
- **语言无关** — 不绑定任何编程语言
- **只增不覆盖** — 检测已有文件，跳过不破坏
- **规则独立** — 使用 `.claude/rules/` 文件，不侵入 CLAUDE.md
- **独立工作空间** — 使用 `.sprint-workspace/`，不与 `docs/` 冲突
- **统一前缀** — 所有文件 `sprint-` 前缀，避免命名冲突

## 生成的文件结构

```
.claude/
├── agents/sprint-*.md         # 7 个 Agent
├── skills/sprint-*.md         # 4 个 Skill + 初始化 Skill
└── rules/sprint-*.md          # 5 个 Rule

.sprint-workspace/
├── sprint-state.md            # Sprint 状态机
├── sprint-progress.md         # 全局进度
├── sprint-tasks.md            # 任务列表
├── sprint-inbox.md            # 用户反馈
├── sprint-brainstorm.md       # 头脑风暴
├── sprint-requirements.md     # 需求文档
├── sprint-architecture.md     # 架构文档
├── research/sprint-N/         # 调研报告
├── versions/sprint-N/         # 版本记录
└── archive/sprint-N/          # 归档
```

## 里程碑检查点

框架在以下节点暂停，等待用户确认后继续：

1. **需求分析完成后** — 确认需求范围
2. **架构设计完成后** — 确认技术方案
3. **质量评估分数 < 60** — 决定是否继续迭代

## License

MIT
