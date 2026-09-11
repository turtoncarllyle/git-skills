# git-skills

[简体中文](README.md) | [English](README.en.md)

面向前后端模块化开发的 Git 提交技能集合。当前提供 `smart-git-commit`：默认按前端、后端和业务模块智能规划提交，也支持严格的一文件一提交。

## 功能

- 两阶段流程：先分析并输出计划，确认后再提交
- 默认智能模式：前后端分层，再按业务模块和修改目的分组
- 单文件模式：每个提交只包含一个普通文件或一个重命名逻辑文件
- 覆盖已暂存、未暂存、未跟踪、删除和重命名文件
- 测试跟随对应实现，配置和文档独立规划
- 每个提交成功后自动推送到当前分支的远程跟踪分支
- 提交描述跟随用户当前使用的语言
- Windows PowerShell、macOS Bash/zsh、Linux Bash/zsh 兼容
- 失败即停止，支持从失败条目继续

## 安装

也可以直接告诉 Codex：

```text
请从 https://github.com/turtoncarllyle/git-skills/tree/main/smart-git-commit 安装 smart-git-commit skill。
```

### Windows PowerShell

```powershell
$clonePath = Join-Path $env:TEMP ("git-skills-" + [guid]::NewGuid().ToString("N"))
$skillRoot = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $env:USERPROFILE ".codex" }
$targetPath = Join-Path $skillRoot "skills\smart-git-commit"

git clone --depth 1 "https://github.com/turtoncarllyle/git-skills.git" $clonePath
New-Item -ItemType Directory -Force -Path $targetPath | Out-Null
Copy-Item -Path (Join-Path $clonePath "smart-git-commit\*") -Destination $targetPath -Recurse -Force
```

### macOS / Linux

```bash
clone_dir="$(mktemp -d)"
skill_root="${CODEX_HOME:-$HOME/.codex}"
target_dir="$skill_root/skills/smart-git-commit"

git clone --depth 1 "https://github.com/turtoncarllyle/git-skills.git" "$clone_dir/git-skills"
mkdir -p "$target_dir"
cp -R "$clone_dir/git-skills/smart-git-commit/." "$target_dir/"
```

安装后重新启动 Codex 或开启新任务，使技能被重新发现。

## 使用方法

默认使用智能模式：

```text
使用 $smart-git-commit 分析当前 Git 变更。
```

技能只输出提交计划。确认计划后发送 `提交`，每个提交组成功并验证后会立即推送到远程分支：

```text
提交
```

严格一文件一提交：

```text
使用 $smart-git-commit 单文件分析当前 Git 变更。
```

调整计划：

```text
修改第2条为 fix: 修复登录状态
跳过第3条
重新分析
```

提交失败并完成外部修复后：

```text
继续提交
```

如果本地提交已成功但推送失败，`继续提交` 会先重试该提交的推送，不会重复创建本地提交。已有远程跟踪分支时使用该分支；当前分支没有 upstream 时，首次推送到 `origin/<当前分支>` 并建立跟踪关系。

## 工作方式

智能模式先识别仓库结构，再按公共契约、后端模块、前端模块、配置和文档分类。前端与后端不会进入同一提交，不同业务模块也不会仅因文件类型相同而合并。

提交信息保持 `type: 描述`，只允许 `feat`、`fix`、`refactor`、`perf`、`docs`、`style`、`test`、`chore`。描述不超过 20 个字符，不包含作者或工具归因。

提交前技能会重新扫描工作区。若计划生成后文件或内容发生变化，原计划立即失效，需要重新分析。执行提交前还会确认当前分支和 `origin` 远程，提交后核对本地 HEAD 与远程跟踪分支一致。

## 平台说明

- Windows 路径以反斜杠显示，文本按 UTF-8 读取。
- macOS 和 Linux 使用 Bash/zsh 兼容命令及正斜杠路径。
- WindowsApps 环境无法启动 `rg` 时，改用 PowerShell 原生命令或 `git grep`。
- 技能不会修改用户的全局 Git 配置，也不会跳过 Git hooks。

## 仓库结构

```text
git-skills\
├── README.md
├── README.en.md
├── LICENSE
└── smart-git-commit\
    ├── SKILL.md
    ├── agents\openai.yaml
    └── references\
        ├── classification-rules.md
        └── platform-commands.md
```

## 许可

本仓库采用 [MIT License](LICENSE) 发布。
