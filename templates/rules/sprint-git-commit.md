---
name: sprint-git-commit
description: 每次迭代必须提交Git — 完成一个逻辑单元就提交，一个commit做一件事
---

# Rule: 每次迭代必提交 Git

## 触发条件

完成任何有意义的变更（功能完成、Bug修复、重构、文档更新）。

## 规则

1. 完成一个逻辑单元立即提交，不积攒多个不相关改动。
2. 一个 commit 做一件事。
3. 使用规范 commit message（`feat:` / `fix:` / `refactor:` / `docs:` / `chore:`）。
4. `.sprint-workspace/` 状态文件变更同样需提交。

## 判定标准

- 多个不相关改动在一次 commit 中 = 违规
- commit message 为 "update" / "wip" = 违规
- 状态文件更新后未提交 = 违规
