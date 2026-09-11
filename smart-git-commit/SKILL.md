---
name: smart-git-commit
description: Plan and execute Git commits in a default smart mode that separates frontend and backend changes by business module, or in a strict one-logical-file-per-commit mode, then push each successful commit to its remote branch. Use when the user asks to analyze, plan, edit, commit, push, or resume Git changes with commands such as "分析", "智能分析", "单文件分析", "提交", "继续提交", "修改第N条", "跳过第N条", "analyze", or "commit".
---

# Smart Git Commit

## Overview

按用户口令分两个阶段处理 Git 变更：先生成可审阅的提交计划，再在用户确认后提交。默认按前后端层级和业务模块智能分组；仅在用户明确要求时切换为一文件一提交。

始终遵守以下原则：

- 未收到分析口令时，不主动扫描变更。
- 未收到提交口令时，不执行暂存或提交。
- 不使用 `git add .` 或 `git add -A`。
- 不把计划外文件带入提交。
- 每个本地提交成功并验证后立即推送到当前分支的远程跟踪分支。
- 使用用户当前主要语言生成提交描述，并在计划生成后冻结该语言。

## Command Triggers

| 用户口令 | 行为 |
| --- | --- |
| `分析`、`智能分析`、`analyze` | 使用默认智能模式重新生成计划，不提交。 |
| `重新分析`、`reanalyze` | 丢弃当前计划，使用当前模式重新分析。 |
| `单文件分析`、`analyze one file per commit` | 使用一文件一提交模式重新生成计划，不提交。 |
| `提交`、`commit` | 执行最近一次有效计划，并在每个提交成功后推送到远程分支；没有计划时要求先分析。 |
| `继续提交`、`continue committing` | 从最近失败条目继续；没有失败记录时说明无法继续。 |
| `修改第N条`、`edit item N` | 修改指定提交信息并重新输出完整计划。 |
| `跳过第N条`、`skip item N` | 移除指定条目，重新编号并输出完整计划。 |

将未明确指定模式的分析请求视为智能模式。不要让一次 `单文件分析` 永久改变后续新请求的默认模式。

## Phase 1: Analyze

### 1. Establish Context

1. 用 `git rev-parse --show-toplevel` 确认当前目录属于 Git 仓库。
2. 从环境上下文识别操作系统和 Shell；需要命令差异时读取 [platform-commands.md](references/platform-commands.md)。
3. 根据用户触发本次分析时使用的主要语言确定提交描述语言；中英混合且无法判断时简短询问。
4. 记录本次模式、语言、文件状态和差异，作为提交前漂移检查基线。

### 2. Collect Every Change

先执行且只用以下命令收集三类文件列表：

```text
git diff --cached --name-status
git diff --name-status
git ls-files --others --exclude-standard
```

然后逐文件读取实际变更：

- 已暂存：`git diff --cached -- "<path>"`。
- 未暂存：`git diff -- "<path>"`。
- 同时暂存和未暂存：同时检查两份差异，并用 `git diff HEAD -- "<path>"` 理解提交时的完整最终内容。
- 未跟踪文本：按 UTF-8 读取完整内容。
- 二进制或不可读文件：依据路径、类型和相邻文件保守分类，并在计划中标明。

去重时让每个普通文件只出现一次。将一次重命名视为一个逻辑文件，并同时保留源路径和目标路径。纳入删除和未跟踪文件，不自动纳入被忽略文件。

若没有任何变更，只输出与用户语言一致的“当前没有检测到任何变更文件”，然后停止。

发现 `.env`、私钥、令牌、凭据、生产配置或疑似敏感内容时，不把它们直接加入计划；先报告路径和风险并等待用户确认。

### 3. Build The Plan

智能模式必须读取 [classification-rules.md](references/classification-rules.md)，按其中规则确定层级、模块、修改目的和依赖顺序。不得仅按扩展名分组，也不得把前端和后端文件放进同一提交。

单文件模式遵守：

- 每条计划只包含一个普通文件或一个重命名逻辑文件。
- 配置、文档、测试和未跟踪文件同样逐文件规划。
- 重命名条目显示为 `<source> -> <destination>`，提交时同时传入两个路径。

同一文件包含多个无法安全拆分的无关目的时，停止并要求用户先拆分文件内容；不要擅自执行交互式补丁暂存。

## Commit Message Rules

只允许以下类型：

- `feat`: 新功能
- `fix`: 修复问题
- `refactor`: 重构
- `perf`: 性能优化
- `docs`: 文档修改
- `style`: 格式调整
- `test`: 测试相关
- `chore`: 构建、配置或杂项

严格使用 `type: 描述`，不要添加 scope。提交描述必须：

- 跟随生成计划时用户使用的主要语言。
- 不超过 20 个字符。
- 简洁说明该组变更的主要意图。
- 不包含作者、署名、多余说明或工具归因。
- 不出现 `AI`、`Claude`、`ChatGPT` 等字样。

配置文件使用独立计划，通常使用 `chore`。同一依赖的清单文件和锁文件可以作为一个不可分割的配置组。

## Plan Output

智能模式输出：

```text
📋 智能提交计划

[1] feat: 新增认证接口
    模块：后端 / 认证
    📁 src\Api\AuthController.cs
    📁 tests\Api\AuthControllerTests.cs

[2] feat: 新增登录表单
    模块：前端 / 认证
    📁 src\Web\pages\login.tsx
    📁 src\Web\pages\login.test.tsx
```

单文件模式输出：

```text
📋 单文件提交计划

[1] feat: 新增用户登录
    📁 src\auth\login.ts

[2] chore: 更新依赖配置
    📁 package.json
```

输出完整计划后停止，等待 `提交`、`修改第N条`、`跳过第N条` 或 `重新分析`。不要在计划后附加未请求的提交操作。

## Plan Edits

处理 `修改第N条`：

- 用户提供有效的 `type: 描述` 时直接替换。
- 用户未提供新信息时询问一次，不提交。
- 校验类型、语言和长度后重新输出完整计划。

处理 `跳过第N条`：

- 从计划移除该条，但不修改工作区或索引。
- 重新编号并输出完整计划。

## Phase 2: Commit

收到提交口令后按以下顺序执行：

1. 重跑三类文件扫描，并逐文件复核差异。
2. 将当前状态与计划基线比较。任何计划文件新增、删除、路径变化或内容变化都会使计划失效；停止并要求重新分析。
3. 在第一个提交前确认当前分支和远程：`git branch --show-current` 必须有结果，且 `git remote get-url origin` 必须成功。若当前分支已有 upstream，保存该远程跟踪分支并使用 `git push`；若没有 upstream，保存 `git push -u origin "<current-branch>"` 作为第一次推送命令。
4. 按计划顺序一次处理一个提交组。
5. 对当前组使用显式路径暂存：`git add -- "<path-1>" "<path-2>"`。单文件模式除重命名外，每次只能传入一个路径。
6. 用 `git diff --cached --name-status` 检查索引。若存在当前组之外的已暂存路径，使用路径限定提交，不能让它们进入当前提交。
7. 执行 `git commit -m "type: 描述" -- "<path-1>" "<path-2>"`。单文件模式除重命名外只能传入一个路径。
8. 检查退出码，再用 `git diff-tree --no-commit-id --name-status -r -M HEAD` 验证最新提交只包含计划路径。
9. 立即推送刚刚验证的提交：已有 upstream 时执行 `git push`；没有 upstream 时首次执行 `git push -u origin "<current-branch>"`，成功后沿用该 upstream。检查推送退出码和输出，确认远程分支已更新。
10. 记录提交哈希、推送结果和远程分支，成功后再处理下一组。

禁止使用裸 `git commit` 提交未核对的索引。不要修改全局 Git 配置，不要自动跳过 hooks，不要使用 `--no-verify`，不要使用 `git push --force` 或 `--force-with-lease`，不要在失败后继续后续条目。

全部完成后执行 `git status --short --branch`，并核对当前本地 HEAD 与远程跟踪分支一致；输出每条提交信息、短哈希、推送目标、成功状态和剩余工作区状态。

## Failure And Resume

任一 `git add`、`git commit`、hook、提交内容验证或 `git push` 失败时：

1. 立即停止。
2. 保存失败条目、原计划、已完成条目和错误文本。
3. 输出失败序号、错误信息以及“处理后发送继续提交”。

如果本地提交已经成功但推送失败，标记该条目为“待推送”，不要再次执行 `git commit`。收到 `继续提交` 后，先重新检查失败条目、当前 HEAD、远程跟踪分支和剩余计划是否漂移；未漂移时先重试该提交的 `git push`，推送成功后再处理下一组；已漂移时要求重新分析。

## Safety Checklist

- 覆盖已暂存、未暂存、未跟踪、删除和重命名文件。
- 不覆盖或丢弃用户已有索引状态。
- 不暂存计划外文件或被忽略文件。
- 不把敏感文件静默提交。
- 不混合前端、后端或无关业务模块。
- 不在用户只要求分析时提交。
- 不在没有有效计划时提交。
