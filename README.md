# Agent Coding System

## The Big Idea

Ephemeral repos. Cattle not pets. Every feature gets its own clone, its own workspace, its own agent. When the feature merges, the repo gets nuked. The remote is the only source of truth — everything local is disposable.

Works with any Python repo that uses [uv](https://docs.astral.sh/uv/) for dependency management.

## Setup

1. Clone this repo alongside your project repos:
   ```
   workspace/
   ├── agent-system/      # this repo
   ├── my-app-feature-1/  # ephemeral clone (created by bin/agent-clone)
   └── my-app-feature-2/  # another ephemeral clone
   ```

2. Configure git remotes:
   ```bash
   cp .repos.example .repos
   # Edit .repos — set your default remote and any per-repo overrides:
   # default=git@github.com:myorg
   # special-repo=git@github.com:other-org
   ```

3. Add `bin/` to your PATH so agent tools are available globally:
   ```bash
   export PATH="$PATH:/path/to/agent-system/bin"
   ```
   Add this to your `~/.bashrc` or `~/.zshrc` to persist across sessions.

4. (Optional) Add env vars and VS Code settings for your repos:
   - `.env.my-app` — copied into clones as `.env`
   - `.vscode.my-app/` — copied into clones as `.vscode/`

4. Your target repo should have a `CLAUDE.md` with codebase conventions (framework patterns, naming, formatting rules, etc.). The agents read this to understand how to write code that fits.

## The Agents

### Plan Agent
- Interactive session where the human and agent plan a feature together
- Reads the codebase to ask informed questions about boundaries and integrations
- Derives the plan filename from the branch: `<yyyy-mm-dd>-<ticket>-<feature-name>.md`
- Branch must follow the format `<user>/<ticket>/<feature-name>` or the agent will refuse to start
- Plans directory is passed as an argument (relative or absolute); prompts with tab completion if omitted
- Gets peer review from a separate Claude instance before committing
- `bin/agent-plan [plans-dir]`

### Build Agent
- Gets a plan describing what to build
- **Acts on feedback** — from the peer reviewer or the human. The build agent is the one who fixes the code.
- **Keeps the plan up to date** — if the design shifts during the build, the plan gets updated
- Builds incrementally — commits after each meaningful chunk
- Does NOT push — the human reviews commits and pushes when ready
- Test-first with fakes, then implementation. Must pass `ruff check` + `ruff format` + `pytest`
- `bin/agent-build [code-dir] <plan-file>`

### Hack Agent
- For exploratory work, debugging, UI iteration, and small changes that don't need a plan doc
- Interactive conversation to design the approach, then autonomous build
- `bin/agent-hack <clone-dir>`

### Peer Reviewer (Codex)
- A different frontier model — different strengths, different blind spots, that's the point
- Invoked as `codex review --base main` by the build agent or plan agent
- Reviews code with a fresh perspective, asks for revisions
- The build agent acts on them, or escalates to the human if there's a disagreement

## The Workflow

### 1. Clone
```bash
bin/agent-clone <repo-name> [feature-name]
```
- Clones into `<repo-name>-<feature-name>/` (or `<repo-name>-1/`, `<repo-name>-2/` if no feature name)
- Stays on main
- Runs `uv sync` if `pyproject.toml` exists
- Copies `.env` from `agent-system/.env.<repo-name>` if it exists
- Copies `.vscode` settings from `agent-system/.vscode.<repo-name>/` if they exist

### 2. Plan
```bash
bin/agent-plan [plans-dir]
```
- Run from inside the feature branch
- Branch must follow `<user>/<ticket>/<feature-name>` (e.g. `eric-shaw/PROJ-123/add-oauth-login`)
- `plans-dir` is the directory to write the plan into — relative or absolute. If omitted, you'll be prompted with tab completion starting from the current directory.
- Plan is written to `<plans-dir>/<yyyy-mm-dd>-<ticket>-<feature-name>.md`
- A separate Claude instance reviews the plan before commit

### 3. Build
```bash
bin/agent-build [code-dir] <plan-file>
```
- `code-dir` is optional — defaults to the current directory if omitted
- `plan-file` is required — prompted with tab completion if omitted
- Confirms directory, branch, and plan file with you before starting
- Build agent reads the plan, reads the codebase, builds the thing
- Commits incrementally, gets peer review via `agent-code-review --claude`
- Human reviews commits when done: `git log --oneline main..HEAD`
- Human pushes when satisfied: `git push`

### 4. Status
```bash
bin/agent-status
```
- Shows all active ephemeral clones, their agent state, and progress

### 5. Nuke
```bash
rm -rf <clone-dir>
```

## Permissions

No `--dangerously-skip-permissions`. Agents get explicit `--allowedTools`:

- **File ops:** `Edit`, `Write`, `Read`, `Glob`, `Grep`
- **Git:** specific commands only — `add`, `commit`, `diff`, `log`, `status`, `branch`, `checkout`, `show`, `stash`, `rev-parse`. No `push`, no `reset`, no `remote`.
- **Build tools:** `uv`, `ruff`, `python`, `pytest`
- **Peer review:** `codex review` only
- **Shell:** `ls`, `cat`, `head`, `tail`, `grep`, `find`, `tree`, `sed`, `awk`, etc.

Tool definitions live in `lib/allowed-tools.sh`.

## File Structure

```
agent-system/
├── bin/
│   ├── clone               # Clone repo, setup env and vscode
│   ├── plan                # Launch plan agent
│   ├── build               # Launch build agent
│   ├── hack                # Launch hack agent
│   ├── status              # Show active features dashboard
│   ├── agent-plan-review   # Review a plan doc for design completeness
│   └── agent-code-review   # Review code changes vs main/master
├── lib/
│   ├── allowed-tools.sh    # Shared tool permission definitions
│   └── agent-session.sh    # Session tracking helpers
├── prompts/
│   ├── plan.md             # Plan agent system prompt
│   ├── build-agent.md      # Build agent system prompt
│   └── hack-agent.md       # Hack agent system prompt
├── .repos.example          # Example remote config (copy to .repos)
└── .gitignore
```

## Key Principles

- **The human is not the bottleneck.** Agents consult Codex before escalating.
- **Different models for different perspectives.** Cross-model review catches what same-model review misses.
- **Accept imperfect, focus on integrations.** If the boundaries are right, the internals are fixable. If the integrations are wrong, that's expensive.
- **The plan is the handoff, not a perfect spec.** The conversation produces shared understanding. The plan captures the key decisions and boundaries. The agent figures out the rest.
- **Everything local is disposable.** The remote is the source of truth. Repos are cattle, not pets.
- **Codebase conventions belong in the target repo.** The agent prompts handle process and behavior. Your repo's `CLAUDE.md` handles how to write code that fits.
