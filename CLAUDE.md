# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin repository that provides **project boundary protection** — preventing Claude from modifying files outside the current project directory. It uses a two-layer approach: hard enforcement via PreToolUse hooks and soft guardrails via AI skill documentation.

The primary language for user-facing documentation is Korean. Shell scripts use English comments.

## Architecture

```
marketplace.json                    # Plugin registry for distribution
project-boundary-guard/
├── .claude-plugin/plugin.json      # Plugin metadata (name, version)
├── hooks/hooks.json                # PreToolUse hook configuration
├── scripts/
│   ├── guard-file-boundary.sh      # Guards Write/Edit tools — blocks external file paths
│   └── guard-bash-boundary.sh      # Guards Bash tool — blocks modify commands, redirects, global installs
└── skills/
    ├── brainstorming/SKILL.md      # Design-first development process
    ├── frontend-design/SKILL.md    # UI/UX aesthetic guidelines
    ├── handoff/SKILL.md            # Agent-to-agent handoff documentation
    └── project-boundary/SKILL.md   # Soft guardrails for patterns hooks can't detect
```

**Hook system:** `hooks.json` registers two PreToolUse hooks. When Claude invokes Edit/Write, `guard-file-boundary.sh` checks if the target file is inside `$CLAUDE_PROJECT_DIR`. When Claude invokes Bash, `guard-bash-boundary.sh` parses the command for modification commands (`rm`, `mv`, `cp`, etc.), output redirects (`>`, `>>`, `tee`), and global installs (`npm -g`, `pip` without venv). Both scripts read JSON from stdin, normalize paths with `realpath`, and output a JSON permission decision or exit silently to allow.

**Key design detail:** PreToolUse hooks run even under `--dangerously-skip-permissions`, making this plugin effective regardless of permission mode.

**Soft guardrails layer:** `project-boundary/SKILL.md` documents indirect execution patterns that hooks cannot catch (e.g., `bash -c "rm /path"`, `xargs rm`, `find -exec`, `eval`, variable indirection, Python/Node file operations). These require AI awareness rather than script-level enforcement.

## Build / Test / Lint

There is no build system, test suite, or linter. The repository consists of static shell scripts, JSON configuration, and markdown documentation. No compilation or dependency installation is required.

## Development Notes

- Hook scripts use inline Node.js (`node -e`) for JSON parsing instead of `jq` to avoid external dependencies.
- `guard-bash-boundary.sh` uses a whitelist approach: read-only commands (`cat`, `ls`, `grep`, `git log`, etc.) are explicitly allowed on external paths; everything else is checked.
- Both hook scripts have a 10-second timeout configured in `hooks.json`.
- Path checking uses prefix matching after `realpath` normalization against `$CLAUDE_PROJECT_DIR` (defaults to `$PWD`).
