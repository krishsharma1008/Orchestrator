# Agent Orchestrator

Agent Orchestrator is an open-source system for running multiple AI coding agents in parallel. Each agent gets its own git worktree, branch, and PR. The system handles CI failures and review comments automatically; you only get involved when human judgment is needed.

**What it does:** You spawn agents (e.g. `ao spawn my-project 123`). Each agent works in isolation on a branch, opens a PR, and reacts to CI and review feedback. You supervise from one dashboard and step in when decisions are required.

**Design:** Agent-agnostic (Claude Code, Codex, Aider), runtime-agnostic (tmux, Docker), and tracker-agnostic (GitHub, Linear). Built as a pluggable system—every piece (runtime, agent, workspace, tracker, notifier) is swappable.

---

## Quick Start

**From a repo URL:**

```bash
git clone https://github.com/krishsharma1008/Orchestrator.git
cd Orchestrator && bash scripts/setup.sh

ao start https://github.com/your-org/your-repo
```

**From an existing local repo:**

```bash
cd ~/your-project && ao init --auto
ao start
```

Spawn an agent:

```bash
ao spawn my-project 123    # issue/ticket number or ad-hoc
```

Dashboard: `http://localhost:3000`. CLI overview: `ao status`.

---

## How It Works

1. **Workspace** — Creates an isolated git worktree and feature branch.
2. **Runtime** — Starts a tmux session (or Docker/process).
3. **Agent** — Runs Claude Code, Codex, or Aider with issue context.
4. Agent works autonomously: reads code, writes tests, opens PR.
5. **Reactions** — Auto-handle CI failures and review comments.
6. **Notifier** — Alerts you when your input is needed.

### Plugin Slots

| Slot     | Default     | Alternatives        |
|----------|-------------|---------------------|
| Runtime  | tmux        | docker, process     |
| Agent    | claude-code | codex, aider        |
| Workspace| worktree    | clone               |
| Tracker  | github      | linear              |
| Notifier | desktop     | slack, webhook      |
| Terminal | iterm2      | web                 |

Interfaces live in `packages/core/src/types.ts`. A plugin implements one interface and exports a `PluginModule`.

---

## Configuration

Example `agent-orchestrator.yaml`:

```yaml
port: 3000

defaults:
  runtime: tmux
  agent: claude-code
  workspace: worktree
  notifiers: [desktop]

projects:
  my-app:
    repo: owner/my-app
    path: ~/my-app
    defaultBranch: main

reactions:
  ci-failed:
    auto: true
    action: send-to-agent
    retries: 2
  changes-requested:
    auto: true
    action: send-to-agent
  approved-and-green:
    auto: false
    action: notify
```

See `agent-orchestrator.yaml.example` for the full reference.

---

## CLI

```bash
ao status                    # Overview of all sessions
ao spawn <project> [issue]   # Spawn an agent
ao send <session> "..."      # Send instructions
ao session ls                # List sessions
ao session kill <session>    # Kill a session
ao session restore <session> # Restore a crashed agent
ao dashboard                 # Open web dashboard
```

---

## Prerequisites

- Node.js 20+
- Git 2.25+
- tmux (default runtime)
- `gh` CLI (for GitHub)

---

## Development

```bash
pnpm install && pnpm build
pnpm test
pnpm dev    # Web dashboard
```

See [CLAUDE.md](CLAUDE.md) for architecture and conventions.

---

## Docs

- [Setup Guide](SETUP.md)
- [Examples](examples/) — Config templates
- [CLAUDE.md](CLAUDE.md) — Plugin pattern and types
- [Troubleshooting](TROUBLESHOOTING.md)

---

## License

MIT
