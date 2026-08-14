# git-skills

[简体中文](README.md) | [English](README.en.md)

Git commit skills for modular frontend and backend development. The current `smart-git-commit` skill plans commits by layer and business module by default, while also supporting strict one-file-per-commit workflows.

## Features

- Two phases: review a plan first, commit only after confirmation
- Smart mode by default: separate frontend and backend, then group by business module and intent
- One-file mode: each commit contains one regular file or one logical rename
- Covers staged, unstaged, untracked, deleted, and renamed files
- Keeps tests with their implementation while planning configuration and documentation separately
- Writes commit descriptions in the user's current language
- Supports Windows PowerShell, macOS Bash/zsh, and Linux Bash/zsh
- Stops on failure and resumes from the failed item

## Installation

You can ask Codex to install the skill directly:

```text
Install the smart-git-commit skill from https://github.com/turtoncarllyle/git-skills/tree/main/smart-git-commit.
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

Restart Codex or open a new task after installation so the skill can be discovered.

## Usage

Use smart mode by default:

```text
Use $smart-git-commit to analyze the current Git changes.
```

The skill returns a commit plan without changing the repository. After reviewing the plan, send:

```text
commit
```

Use strict one-file-per-commit mode:

```text
Use $smart-git-commit to analyze one file per commit.
```

Edit the plan:

```text
edit item 2 to fix: repair login state
skip item 3
reanalyze
```

After resolving an external cause of a failed commit:

```text
continue committing
```

## How It Works

Smart mode inspects the repository structure, then classifies changes as shared contracts, backend modules, frontend modules, configuration, or documentation. Frontend and backend changes never share a commit, and unrelated business modules are not merged merely because their files have similar extensions.

Commit messages keep the `type: description` format and allow only `feat`, `fix`, `refactor`, `perf`, `docs`, `style`, `test`, and `chore`. Descriptions are limited to 20 characters and contain no author or tool attribution.

The skill rescans the working tree before committing. If a file or its content changed after the plan was generated, the plan becomes invalid and must be regenerated.

## Platform Notes

- Windows paths use backslashes and text is read as UTF-8.
- macOS and Linux use Bash/zsh-compatible commands and slash-separated paths.
- If `rg` cannot start from a WindowsApps environment, the skill uses native PowerShell commands or `git grep`.
- The skill does not change global Git configuration or bypass Git hooks.

## Repository Layout

```text
git-skills/
├── README.md
├── README.en.md
├── LICENSE
└── smart-git-commit/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
        ├── classification-rules.md
        └── platform-commands.md
```

## License

Released under the [MIT License](LICENSE).
