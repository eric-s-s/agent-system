# Agent System

## Language & Style

- Pure bash. No external deps beyond standard unix tools + git + uv + ruff.
- Scripts simple, readable. No clever tricks.
- `set -euo pipefail` at top of every script.

## Conventions

- `bin/` — user-facing commands. Each self-contained bash script.
- `lib/` — shared helpers sourced by `bin/` scripts.
- `prompts/` — system prompts for agents (markdown files).
- `.repos` — local config for git remotes (gitignored). `.repos.example` committed template.
- `.agent-session` — session marker written into ephemeral clones (gitignored).
- Per-repo config files (`.env.{repo}`, `.vscode.{repo}/`) live in agent-system root, gitignored.

## Git

- Prefer normal git commands. Avoid `git -C`.
- No tests in repo.

## Plans

- When working in `~/workspace/DataRobot`, write and commit all plans to `/home/eric-shaw/workspace/dr-repo-claude-plans` instead of `claude_plans`.