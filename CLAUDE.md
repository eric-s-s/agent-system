# Agent System

## Language & Style

- Pure bash. No external dependencies beyond standard unix tools + git + uv + ruff.
- Keep scripts simple and readable. No clever tricks.
- `set -euo pipefail` at the top of every script.

## Conventions

- `bin/` — user-facing commands. Each is a self-contained bash script.
- `lib/` — shared helpers sourced by `bin/` scripts.
- `prompts/` — system prompts for agents (markdown files).
- `.repos` — local config for git remotes (gitignored). `.repos.example` is the committed template.
- `.agent-session` — session marker written into ephemeral clones (gitignored).
- Per-repo config files (`.env.{repo}`, `.vscode.{repo}/`) live in the agent-system root and are gitignored.

## Git

- Prefer normal git commands. Avoid `git -C`.
- No tests in this repo.

## Plans

- When working in `~/workspace/DataRobot`, write and commit all plans to `/home/eric-shaw/workspace/dr-repo-claude-plans` instead of `claude_plans`.
