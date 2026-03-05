# Claude Code Setup

A complete, reproducible setup guide for [Claude Code](https://www.anthropic.com/claude-code) as a
developer tool on Windows 11 — documented, reproducible, and version-controlled.

---

## What This Is

This repository documents the complete process of configuring Claude Code from
scratch into a powerful, personalized developer assistant. It covers every
decision made: what was set up, why, and how to verify it.

The goal is a setup that can be understood, rebuilt, or extended at any point.
A reusable reference for setting up Claude Code on any Windows machine.

---

## The Setup

All 15 steps are documented in [`src/setup-guide.md`](src/setup-guide.md).

**Legend:** `[x]` done · `[~]` deferred · `[ ]` todo

| Phase | Steps | Status |
|-------|-------|--------|
| **1 — Foundation** | Install, folder structure, CLAUDE.md, memory, symlink | All done |
| **2 — Personalization** | Settings, slash commands, MCP servers, keybindings, hooks | Done (hooks deferred) |
| **3 — Integration** | Dev server preview, VS Code extension, shell env | All done |
| **4 — Workflow** | New project workflow, knowledge base | All done |

---

## Key Outcomes

**What was configured:**

- **Root folder** at `C:\Users\[YOUR-USERNAME]\your cloud sync folder\Claude\Claude Code\` — all config,
  memory, and projects in one place, synced via your cloud sync folder.

- **Global CLAUDE.md** — persistent instructions loaded in every session: tone,
  code conventions, shell rules, workflow preferences.

- **Memory system** — auto-memory (Claude-managed) plus manual `memory/` files
  for environment state, decisions, and session context.

- **`.claude` symlinked to your cloud sync folder** — session history, memory, and settings
  are backed up and survive a machine rebuild.

- **MCP servers** — Sequential Thinking (reasoning), Context7 (live library
  docs), Microsoft Learn (Azure/M365 docs).

- **Custom slash commands** — `/commit`, `/review`, `/session-start`,
  `/update-memory`, `/update-instructions`, `/new-project`, `/handover`,
  `/research`, `/simplify`.

- **Project template** — `projects/_template/` scaffolds new projects with the
  right structure instantly via `/new-project`.

- **Knowledge base** — 6 reference files covering Windows environment,
  Claude Code, Python, TypeScript, PowerShell, and Bicep.

- **VS Code extension** — installed and active; same engine as CLI, with inline
  diffs and diagnostic access.

---

## Project Structure

```
claude-code-setup/
├── README.md              ← This file
├── CLAUDE.md              ← Project-specific Claude instructions
├── src/
│   └── setup-guide.md     ← Full step-by-step setup documentation
├── memory/
│   └── progress.md        ← Step status and decisions log
├── knowledge/             ← Project-specific reference (minimal — global KB used)
└── instructions/          ← Additional instruction docs for Claude
```

---

## Decisions Log

Key technical decisions made during setup. Full context in `memory/progress.md`.

| Decision | Rationale |
|----------|-----------|
| **PowerShell over Bash** | Git Bash (MSYS2) doesn't expand `%VAR%` Windows PATH entries. PowerShell is reliable for Node, npm, claude. |
| **Microsoft Learn MCP** instead of static azure.md | Live, official, always current. No maintenance needed. |
| **nvm-windows** for Node.js | Same pattern as pyenv-win for Python; easy version switching. |
| **pyenv-win** for Python | Consistent version management; Python 3.13.2 global. |
| **your cloud sync folder for all config** | Automatic sync + backup. No separate backup tooling needed. |
| **No hooks yet** | Manual `/update-memory` is sufficient. Add hooks when a recurring trigger is clear. |
| **Single setup-guide.md** | One file is easier to navigate and maintain than many small ones. |

---

## Related Files

- `C:\Users\[YOUR-USERNAME]\your cloud sync folder\Claude\Claude Code\CLAUDE.md` — global instructions
- `C:\Users\[YOUR-USERNAME]\your cloud sync folder\Claude\Claude Code\knowledge\` — cross-project reference
- `C:\Users\[YOUR-USERNAME]\your cloud sync folder\Claude\Claude Code\projects\_template\` — new project scaffold
