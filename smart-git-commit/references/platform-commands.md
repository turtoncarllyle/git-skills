# 跨平台命令约定

仅在需要处理操作系统、Shell、路径或编码差异时读取本文件。Git 命令尽量保持跨平台一致，Shell 只负责安全读取文件和检查退出状态。

## Windows PowerShell

优先采用环境上下文判断 Windows 和 PowerShell。需要确认时可读取 `$PSVersionTable` 和 `$env:OS`，不要修改系统或全局环境变量。

使用反斜杠显示 Windows 路径，并始终引用包含空格、中文或特殊字符的路径：

```powershell
git diff --cached -- "src\用户\Profile.cs"
git add -- "src\用户\Profile.cs"
git commit -m "feat: 新增用户资料" -- "src\用户\Profile.cs"
```

按 UTF-8 读取未跟踪文本：

```powershell
Get-Content -LiteralPath "src\用户\Profile.cs" -Raw -Encoding UTF8
```

每个 Git 命令后检查 `$LASTEXITCODE`。不要把 PowerShell 的 `$?` 当作唯一 Git 成功证据。

若当前位于 WindowsApps 包路径或 `rg.exe` 启动被拒绝，使用以下替代方式：

- `Get-ChildItem` 枚举文件。
- `Select-String` 搜索文本。
- `git grep` 搜索已跟踪内容。

不要为了运行 `rg` 修改执行策略、权限或系统目录。

## macOS And Linux

用现有环境上下文判断 Bash 或 zsh；需要确认时使用 `uname -s`。确保当前 locale 支持 UTF-8，不要更改用户全局 Shell 配置。

使用正斜杠并引用路径：

```bash
git diff --cached -- "src/user/profile.ts"
git add -- "src/user/profile.ts"
git commit -m "feat: add user profile" -- "src/user/profile.ts"
```

读取文本时使用支持 UTF-8 的现有 Shell 工具，并在路径前使用 `--`（工具支持时）：

```bash
sed -n '1,240p' -- "src/user/profile.ts"
```

每个 Git 命令后检查退出码。命令失败时立即停止，不使用 `set +e` 隐藏错误。

## Shared Git Rules

- 使用 Git 输出的精确路径，不手工拼接未验证的路径。
- 路径参数前使用 `--`，避免把以连字符开头的文件名解释为选项。
- 不使用未解析的 glob 暂存文件。
- 不使用 `git add .`、`git add -A` 或裸 `git commit` 处理未核对索引。
- 不修改 `core.autocrlf`、`core.quotepath`、`i18n.commitEncoding` 等全局配置。
- 将 LF/CRLF 警告与命令失败区分；以退出码和提交结果为准。
- Git 提交消息和技能文档使用 UTF-8。
