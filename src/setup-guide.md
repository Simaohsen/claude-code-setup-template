# Claude Code Setup Guide

A step-by-step process for setting up Claude Code as a powerful, personalized
developer tool on Windows 11. Each step includes the goal, how to do it,
how to verify it, and its current status.

**Legend:** `[x]` done · `[ ]` todo · `[~]` in progress

---

## Understanding the Claude Ecosystem

Before diving in, it helps to know how the different Claude tools relate to each other.
They are not interchangeable — each has a distinct role.

| Tool | Interface | Best for |
|------|-----------|----------|
| **Claude Desktop App** | Windows/Mac GUI | General chat, quick tasks, visual access to Claude Code |
| **Claude Code tab** (inside Desktop) | GUI, no terminal needed | Coding: diffs, app preview, PR review, parallel sessions |
| **Claude Code CLI** | Terminal | Agentic coding: full shell, scripting, multi-agent, max power |
| **Claude Code — VS Code extension** | Inside VS Code | In-editor assistance, inline diffs, diagnostics access |
| **Cowork** (inside Desktop) | GUI, isolated VM | Multi-step desktop automation for non-coding tasks |

**Key fact:** The Claude Code tab in the Desktop app and the CLI share the same
underlying engine. That means `CLAUDE.md` files, the memory system, and all
configuration set up in this guide applies to both.

**This guide** focuses on the Claude Code tab (Desktop) and CLI. VS Code extension
and Cowork are covered in Phase 3.

---

---

## Windows Users: Choose Your Setup Path

If you are on **Windows**, make this decision before starting:

| | Native Windows | WSL 2 (Recommended) |
|---|---|---|
| **Shell** | Git Bash (MSYS2) | Real Linux bash |
| **MCP servers** | Needs `cmd /c` wrapper | Works out of the box |
| **PATH reliability** | Frequent issues | No issues |
| **File operations** | Junction bugs in tools | Clean symlinks |
| **Setup complexity** | Simpler | One extra step (install WSL) |
| **`/mcp` command** | Broken (upstream bug) | Works correctly |

**Recommendation: Use WSL 2.** It eliminates all known Windows shell reliability
issues. Claude Code was built for POSIX environments; WSL gives it that natively.

### If you choose WSL 2:
Follow this guide normally for configuration (CLAUDE.md, memory, MCP, commands) —
they are all shared via OneDrive. For the WSL-specific installation steps, see
**Phase 5** below. Full architecture details are in `docs/planning/hybrid-setup-plan.md`.

### If you choose Native Windows:
Continue with this guide as written. Be aware of the known limitations
documented in `knowledge/shell-gotchas.md`. Use PowerShell for all system
commands; never rely on Git Bash for PATH-sensitive operations.

---

## Phase 1 — Foundation

### [x] Step 1: Install Claude Code

**Goal:** Get Claude Code installed and runnable.

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:
```bash
claude --version
```

---

### [x] Step 2: Establish Root Folder Structure

**Goal:** Create a dedicated home for all Claude Code configuration, memory,
and projects — separate from any individual code project.

**Location:** `C:\Users\[YOUR-USERNAME]\OneDrive\Claude\Claude Code\`

```
Claude Code/
├── CLAUDE.md                 ← Global instructions (auto-loaded)
├── .claude/commands/         ← Custom slash commands
├── instructions/             ← Supplementary global instruction docs
├── memory/                   ← Manually curated global memory
├── knowledge/                ← Global reference knowledge
└── projects/
    ├── _template/            ← Copy this when starting a new project
    └── [project-name]/       ← Individual projects
        ├── CLAUDE.md
        ├── instructions/
        ├── memory/
        ├── knowledge/
        └── src/
```

**Why:** Keeps Claude configuration versioned, organized, and portable.
OneDrive location ensures it syncs and is backed up automatically.

Verify: `ls "C:/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/"` shows the structure above.

---

### [x] Step 3: Write Global CLAUDE.md

**Goal:** Define global behaviour, preferences, and conventions that apply
to every Claude Code session rooted at the Claude Code folder.

**File:** `C:\Users\[YOUR-USERNAME]\OneDrive\Claude\Claude Code\CLAUDE.md`

**Content:** Create this file with the following content. Adapt the auto-memory
path section to your own username and OneDrive path.

```markdown
# Global Claude Code Instructions

This file is automatically loaded by Claude Code in every session rooted here.
It defines global behaviour, preferences, and conventions.

---

## Folder Structure

This is the Claude Code root. The structure is:

​```
Claude Code/
├── CLAUDE.md                 ← This file (global instructions)
├── .claude/commands/         ← Custom slash commands
├── instructions/             ← Supplementary global instruction docs
├── memory/                   ← Manually curated global memory
├── knowledge/                ← Global reference knowledge
└── projects/
    ├── _template/            ← Template for new projects
    └── [project-name]/       ← Individual projects
        ├── CLAUDE.md         ← Project-specific instructions
        ├── instructions/     ← Project instruction details
        ├── memory/           ← Project memory/notes
        ├── knowledge/        ← Project knowledge base
        └── src/              ← Source code and documents
​```

---

## Global Preferences

### Communication
- Be concise. Avoid unnecessary filler or affirmations.
- Use plain technical language. No emojis unless asked.
- When uncertain, investigate before assuming.

### Code
- Prefer editing existing files over creating new ones.
- Do not add comments, docstrings, or type annotations to code you didn't change.
- Avoid over-engineering. Minimum complexity needed for the task.
- Never introduce security vulnerabilities.

### Workflow
- Use the TodoWrite tool to plan and track multi-step tasks.
- Always read a file before editing it.
- Ask before making architectural decisions.
- Use background parallel Task agents whenever work is independent — parallelise by default.

### Shell / Terminal

**First: detect the environment.** Run `uname -s` or check whether you are in a WSL shell.
- Returns `Linux` → **WSL 2** (Ubuntu, primary environment). Use native bash.
- Otherwise → **Native Windows** (VS Code extension, Desktop app). Use PowerShell.

#### WSL 2 (primary)
- Use native bash for all operations. Tools on PATH are available immediately after install.
- Access Windows files via `/mnt/c/`. Shared config lives on OneDrive via symlink at `~/.claude`.
- MCP servers use plain `npx` — no `cmd /c` wrapper needed.
- No junction/symlink EEXIST errors; Write/Edit tools work normally.

#### Native Windows (fallback)
- Use PowerShell (`powershell.exe -Command "..."`) for all system operations. Never default to Bash.
- After installing any tool, verify availability: `(Get-Command tool-name -ErrorAction SilentlyContinue)?.Source`.
- Never add `%VAR%`-style entries to the Windows User PATH — use literal paths only.
- PATH changes written to the registry are NOT visible in the current session; prepend to `$env:PATH` as a workaround.
- When a command fails with "not recognized": check with `Get-Command`, use full path, do NOT retry unchanged.
- MCP servers using `npx` must be wrapped with `cmd /c` in `~/.claude.json` (`npx` is a `.cmd` file).
- Write/Edit tools fail with EEXIST on paths with spaces or through the `~/.claude` junction — write to `%TEMP%` then `Copy-Item`.

### Commands
- Global slash commands (available everywhere) go in `~/.claude/commands/`.
- Project-specific commands go in `projects/[name]/.claude/commands/`.
- Rule: "Would this command make sense in a completely different project?" → Yes = global, No = project-level.

### Knowledge Files
- Every knowledge file should include a `Last reviewed:` date in its header.
- When loading a knowledge file, flag any file with a review date older than 90 days.

---

## Context System: Instructions, Memory, Knowledge

Claude Code uses three distinct types of persistent context. Each has a clear purpose.

### Instructions (`CLAUDE.md`)
Permanent behavioral conventions. How Claude should work, communicate, and make decisions.
Updated deliberately via `/update-instructions` only when a new convention is worth making permanent.

### Memory (`memory/`)
Session-derived context. Current state, decisions made, what was done, what's pending.
Answers: *"Where are we?"*. Updated regularly via `/update-memory`.
- **Auto-memory** — loaded automatically by Claude Code from the project's tracking folder
- **Manual memory** (`memory/` in root or project folders) — read via `/session-start`
- **Convention:** Do not overwrite memory entries from other sessions unless confirmed outdated.

#### Auto-Memory Path (WSL — adapt to your username)

The system prompt path `/home/[user]/.claude/projects/...` is incorrect — `~/.claude` is a
symlink to OneDrive, so that directory does not exist.

**Always write auto-memory to the real path:**
​```
/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/.claude/projects/[encoded-project-path]/memory/MEMORY.md
​```

To find the correct project folder name: `ls ~/.claude/projects/`

### Knowledge (`knowledge/`)
Reference material. Facts about tools, APIs, domains, and patterns that remain useful
across many sessions. Updated via `/research` when a domain changes.

| Question | Goes in |
|---|---|
| "How should Claude behave?" | Instructions |
| "Where are we / what happened?" | Memory |
| "How does X work?" | Knowledge |

---

## Project-Specific Instructions

Each project in `projects/[name]/` contains its own `CLAUDE.md` with instructions
specific to that project. Those are loaded automatically when working inside that folder.

When creating or updating a project `CLAUDE.md`, include a `/session-start` reminder
at the top — manual memory files are not auto-loaded.
```

Verify: Open a Claude Code session from the root. Claude should follow the
defined preferences without being told.

---

### [x] Step 4: Set Up the Memory System

**Goal:** Give Claude persistent memory across sessions, both automatic
(managed by Claude Code) and manual (curated by you).

**Auto-memory** (managed by Claude Code):
- Location: `~/.claude/projects/[encoded-path]/memory/MEMORY.md`
- Claude reads and writes this file to remember facts across sessions.

**Manual memory** (curated by you):
- Location: `Claude Code/memory/`
- Add `.md` files for topics you want Claude to always know.
- Reference them from `CLAUDE.md` so Claude loads them at session start.

Key file: `memory/environment.md` — system paths, setup status, shell info.

Verify: Start a new session. Claude should recall facts from `MEMORY.md`
without being told them again.

---

### [x] Step 5: Symlink .claude to OneDrive

**Goal:** Back up and sync the `~/.claude` folder (which stores all Claude
Code session history, memory, and config) to OneDrive.

**Why:** By default `~/.claude` is only local. Symlinking it to OneDrive
means it's backed up automatically and survives a machine rebuild.

**Steps (run in cmd.exe as Admin, with Claude Code closed):**
```cmd
rmdir /S /Q C:\Users\[YOUR-USERNAME]\.claude
mklink /D C:\Users\[YOUR-USERNAME]\.claude C:\Users\[YOUR-USERNAME]\OneDrive\Claude\.claude
```

**Prerequisite:** Windows Developer Mode must be enabled (allows symlink
creation without elevation).
`Settings → System → For Developers → Developer Mode: On`

Verify:
```bash
ls -la ~/  | grep .claude
# Should show: .claude -> /c/Users/[YOUR-USERNAME]/OneDrive/Claude/.claude
```

---

## Phase 2 — Personalization

### [x] Step 6: Configure .claude.json Settings

**Goal:** Tune Claude Code behaviour via its settings file.

**File:** `C:\Users\[YOUR-USERNAME]\.claude.json` (or `~/.claude.json`)

There are two separate settings files:

**`~/.claude/settings.json`** — shared config on OneDrive (applies everywhere).
This file controls the model, auto-approved tools, and hooks. The hooks here
reference the scripts created in Step 10.

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-6",
  "env": {
    "CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY": "1"
  },
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash",
      "Write",
      "Edit",
      "MultiEdit"
    ],
    "deny": []
  },
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File \"C:\\Users\\[YOUR-USERNAME]\\OneDrive\\Claude\\Claude Code\\hooks\\notify.ps1\""
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File \"C:\\Users\\[YOUR-USERNAME]\\OneDrive\\Claude\\Claude Code\\hooks\\block-dangerous.ps1\""
          }
        ]
      },
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File \"C:\\Users\\[YOUR-USERNAME]\\OneDrive\\Claude\\Claude Code\\hooks\\protected-files.ps1\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File \"C:\\Users\\[YOUR-USERNAME]\\OneDrive\\Claude\\Claude Code\\hooks\\session-context.ps1\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File \"C:\\Users\\[YOUR-USERNAME]\\OneDrive\\Claude\\Claude Code\\hooks\\format-on-stop.ps1\""
          }
        ]
      }
    ]
  }
}
```

**`~/.claude/settings.local.json`** — WSL-specific override (not on OneDrive,
created per-machine in WSL). Replaces PowerShell hook commands with bash equivalents:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/hooks/notify.sh\""
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/hooks/block-dangerous.sh\""
          }
        ]
      },
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/hooks/protected-files.sh\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/hooks/session-context.sh\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/hooks/format-on-stop.sh\""
          }
        ]
      }
    ]
  }
}
```

**`~/.claude.json`** (Windows) — MCP server configuration. Managed by `claude mcp add`,
but the relevant `mcpServers` section looks like this after setup:

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "cmd",
      "args": ["/c", "C:\\nvm4w\\nodejs\\mcp-server-sequential-thinking.cmd"]
    },
    "context7": {
      "command": "cmd",
      "args": ["/c", "C:\\nvm4w\\nodejs\\context7-mcp.cmd"]
    },
    "microsoft-learn": {
      "url": "https://learn.microsoft.com/api/mcp"
    }
  }
}
```

Note: The `.cmd` paths depend on your Node.js installation location. Adjust if
`nvm` installed Node elsewhere. Use `where node` in PowerShell to find the right path.

Verify: Open Claude Code and confirm the model shown in the header matches.
Run `/mcp` to confirm all 3 servers are connected.

---

### [x] Step 7: Create Custom Slash Commands

**Goal:** Add project-agnostic slash commands that speed up recurring workflows.

**Location:** `C:\Users\[YOUR-USERNAME]\OneDrive\Claude\Claude Code\.claude\commands\`

Each `.md` file in this folder becomes a `/command-name` slash command.

The full set of commands in this setup (create each as `[name].md`):

| Command | Purpose |
|---------|---------|
| `/commit` | Standardized git commit workflow |
| `/session-start` | Load memory and knowledge at the start of a session |
| `/update-memory` | Save session learnings to memory files |
| `/update-instructions` | Update CLAUDE.md with new conventions |
| `/review` | Code review checklist |
| `/health` | Environment diagnostic check |
| `/new-project` | Scaffold a new project from the template |
| `/handover` | Create a handover document for Claude Chat |
| `/research` | Web-research a topic and save it as a knowledge file |
| `/status` | Quick project status summary |
| `/changelog` | Generate a changelog from git history |
| `/snapshot` | Capture current project state |
| `/review-knowledge` | Check knowledge files for freshness |
| `/end-session` | End-of-session wrap-up |

**Key command file contents:**

`commit.md`:
```markdown
Prepare and create a git commit for the current changes. Follow this workflow:

1. Run `git status` to see what has changed.
2. Run `git diff` (staged and unstaged) to understand the changes in detail.
3. Stage relevant changed files. Skip secrets, generated files, build artifacts,
   and anything unrelated to the current logical change.
4. Write a concise commit message:
   - Subject line: imperative mood, under 72 characters ("Add X", "Fix Y", "Update Z")
   - Explain WHY, not just what — unless the change is self-evident
   - Add a body if the change is complex or has important context
5. Show me the staged files and commit message for confirmation before committing.
6. After confirmation, create the commit and show the final summary.

If anything looks unusual (untracked secrets, large diffs, mixed concerns), flag it
before proceeding.
```

`session-start.md`:
```markdown
Load session context before starting work: read memory files and relevant knowledge.

1. **Identify scope** — note the current working directory and whether a project
   CLAUDE.md is present.

2. **Read manual memory files** (auto-memory MEMORY.md is already loaded automatically):
   - Root: all `.md` files in `memory/`
   - Project (if in a project folder): all `.md` files in `projects/[name]/memory/`

3. **Load knowledge files:**
   - List available files in `knowledge/` (root) and `projects/[name]/knowledge/`
   - If total ≤ 5 files: read all of them
   - If total > 5: list the files and ask which are relevant to today's session

4. **Summarise what was loaded** — confirm current memory state and available
   knowledge in 3–5 bullet points.

5. Prompt the user: **"Context loaded. What are we working on today?"**
```

`update-memory.md`:
```markdown
Update the project memory based on what was accomplished in this session.

1. Review the conversation history — what was built, decided, discovered, or changed?
2. Read the current project memory file (check memory/MEMORY.md or the auto-memory
   at ~/.claude/projects/[path]/memory/MEMORY.md).
3. Add or update entries for:
   - Decisions made and the reasoning behind them
   - Important files discovered and what they do
   - Problems solved and how they were solved
   - Patterns or conventions established
   - Anything useful at the start of the next session
4. Remove or correct entries that are now stale or contradicted by new information.
5. Keep entries factual and durable — skip anything only relevant to today's session.

Do not duplicate what is already accurately captured. Be concise. Prefer bullet
points over prose.
```

Verify: In a Claude Code session, type `/` to see your commands listed.

---

### [x] Step 8: Set Up MCP Servers

**Goal:** Extend Claude Code with external tool integrations via the
Model Context Protocol (MCP).

**What MCP does:** Gives Claude access to external systems — live docs,
search, databases, APIs — as structured tools Claude can call directly.

**Installed servers:**
| Server | Package | Trust level |
|--------|---------|-------------|
| Sequential Thinking | `@modelcontextprotocol/server-sequential-thinking` | Official (Anthropic-backed), local only |
| Context7 | `@upstash/context7-mcp` | Third-party (Upstash), fetches live library docs |

**Sequential Thinking** — structures Claude's reasoning for complex problems.
No network calls. Invoke automatically or explicitly when planning architecture.

**Context7** — fetches up-to-date documentation for any library. Add
`"use context7"` to any prompt to pull live docs into the context window.
Example: *"Build a Next.js 14 API route — use context7"*

**Configuration location:** `~/.claude.json` under `"mcpServers"` (user scope,
applies to all projects). This is what `claude mcp add -s user` writes to.

**To add more servers later (via CLI):**
```bash
claude mcp add server-name -s user -- npx -y @package/server-name
```

**Other servers worth adding later:**
| Server | When useful |
|--------|-------------|
| `@modelcontextprotocol/server-github` | Once actively using GitHub |
| `@anthropic-ai/mcp-server-brave-search` | Web search (needs free API key) |
| `@modelcontextprotocol/server-postgres` | If using PostgreSQL |

Verify: In a Claude Code session, run `/mcp` to see connected servers and tools.

---

### [x] Step 9: Customize Keybindings

**Goal:** Rebind or add keyboard shortcuts to fit your workflow.

**File:** `~/.claude/keybindings.json`

**Decision:** Keeping all defaults. No file created — Claude Code works fine
without one. Add bindings later if a default shortcut becomes a pain point.

**To customize later:** Use the `/keybindings-help` skill in a Claude Code
session for guided setup. The skill reads the existing file, merges changes,
and validates against the full schema.

**Key defaults to know:**
- `Enter` — submit message
- `Ctrl+G` — open message in external editor
- `Ctrl+T` — toggle todo list
- `Ctrl+O` — toggle transcript
- `Ctrl+R` — search command history

---

### [x] Step 10: Configure Hooks

**Goal:** Run shell commands automatically in response to Claude Code events.

**Location:** `Claude Code/hooks/` — scripts are shared via OneDrive and
referenced from `settings.json` (Step 6). Create both `.ps1` (Windows) and
`.sh` (WSL) versions for each hook since both environments use the same OneDrive config.

**Hooks configured:**

| Hook event | Script | Purpose |
|------------|--------|---------|
| `Notification` | `notify.sh` / `notify.ps1` | Windows toast when Claude needs attention |
| `PreToolUse` (Bash) | `block-dangerous.sh` | Block destructive shell commands |
| `PreToolUse` (Write/Edit) | `protected-files.sh` | Block writes to `.env`, lock files, migrations |
| `SessionStart` | `session-context.sh` | Inject git status into Claude's context |
| `Stop` | `format-on-stop.sh` | Auto-format modified files on session end |

**Hook scripts (WSL `.sh` versions):**

`hooks/notify.sh` — Windows toast notification (works from WSL via `powershell.exe`):
```bash
#!/bin/bash
powershell.exe -NoProfile -NonInteractive -Command "
  Add-Type -AssemblyName System.Windows.Forms
  Add-Type -AssemblyName System.Drawing
  \$notify = New-Object System.Windows.Forms.NotifyIcon
  \$notify.Icon = [System.Drawing.SystemIcons]::Information
  \$notify.Visible = \$true
  \$notify.ShowBalloonTip(5000, 'Claude Code', 'Needs your attention', [System.Windows.Forms.ToolTipIcon]::Info)
  Start-Sleep -Seconds 6
  \$notify.Dispose()
" 2>/dev/null &

exit 0
```

`hooks/block-dangerous.sh` — blocks destructive Bash commands (exit 2 = block):
```bash
#!/bin/bash
input_json=$(cat)
command=$(echo "$input_json" | python3 -c "
import sys, json
d = json.load(sys.stdin)
print(d.get('tool_input', {}).get('command', ''))
" 2>/dev/null)

[ -z "$command" ] && exit 0

first_line=$(echo "$command" | head -1 | sed 's/^[[:space:]]*//')

# Git force operations
if echo "$first_line" | grep -qiE "git[[:space:]]+push[[:space:]].*--force"; then
    echo "BLOCKED: $first_line" >&2; exit 2
fi
if echo "$first_line" | grep -qiE "git[[:space:]]+reset[[:space:]]+--hard"; then
    echo "BLOCKED: $first_line" >&2; exit 2
fi

# rm -rf targeting root, home, or Windows-style root paths
if echo "$first_line" | grep -qE "^rm[[:space:]]" && \
   echo "$first_line" | grep -qE "\-rf|\-fr" && \
   echo "$first_line" | grep -qE "[[:space:]][/~\\\\]"; then
    echo "BLOCKED: $first_line" >&2; exit 2
fi

# SQL drops — only block when running a database client
first_token=$(echo "$first_line" | awk '{print $1}' | sed 's|.*/||')
db_clients="psql mysql sqlite3 sqlcmd isql"
if echo "$db_clients" | grep -qw "$first_token"; then
    if echo "$command" | grep -qiE "DROP[[:space:]]+(TABLE|DATABASE)|TRUNCATE[[:space:]]+TABLE"; then
        echo "BLOCKED: Destructive SQL statement" >&2; exit 2
    fi
fi

exit 0
```

`hooks/protected-files.sh` — blocks writes to sensitive files:
```bash
#!/bin/bash
input_json=$(cat)
file_path=$(echo "$input_json" | python3 -c "
import sys, json
d = json.load(sys.stdin)
print(d.get('tool_input', {}).get('file_path', ''))
" 2>/dev/null)

[ -z "$file_path" ] && exit 0

protected_patterns=(
    '\.env$'
    '\.env\.'
    'package-lock\.json$'
    'yarn\.lock$'
    'pnpm-lock\.yaml$'
    '\.git/'
    'migration.*\.sql$'
)

for pattern in "${protected_patterns[@]}"; do
    if echo "$file_path" | grep -qE "$pattern"; then
        echo "BLOCKED: '$file_path' is a protected file. Modify it manually if needed." >&2
        exit 2
    fi
done

exit 0
```

`hooks/session-context.sh` — injects git context into every new session:
```bash
#!/bin/bash
branch=$(git branch --show-current 2>/dev/null)
if [ -n "$branch" ]; then
    echo "## Git Context"
    echo "Branch: $branch"

    status=$(git status --short 2>/dev/null | head -10)
    if [ -n "$status" ]; then
        echo "Changed files:"
        echo "$status"
    fi

    commits=$(git log --oneline -5 2>/dev/null)
    if [ -n "$commits" ]; then
        echo "Recent commits:"
        echo "$commits"
    fi
fi

if [ -f "CLAUDE.md" ]; then
    echo ""
    echo "## Project"
    echo "CLAUDE.md found at: $(pwd)/CLAUDE.md"
fi
```

`hooks/format-on-stop.sh` — auto-formats modified files when Claude's session ends:
```bash
#!/bin/bash
modified=$(git diff --name-only 2>/dev/null)
staged=$(git diff --cached --name-only 2>/dev/null)
files=$(printf '%s\n%s\n' "$modified" "$staged" | sort -u | grep -v '^$')

[ -z "$files" ] && exit 0

while IFS= read -r file; do
    [ -f "$file" ] || continue
    ext="${file##*.}"
    case "$ext" in
        js|ts|jsx|tsx|json|css|md|html)
            if command -v prettier &>/dev/null; then
                prettier --write "$file" 2>/dev/null
            fi
            ;;
        py)
            if command -v ruff &>/dev/null; then
                ruff format "$file" 2>/dev/null
            fi
            ;;
    esac
done <<< "$files"

exit 0
```

**Windows `.ps1` versions:** Create equivalent `.ps1` scripts with PowerShell syntax
in the same `hooks/` folder. The `settings.json` (Step 6) references both — Windows
sessions use `.ps1`, WSL sessions use `.sh` via `settings.local.json`.

Make hook scripts executable in WSL:
```bash
chmod +x "/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/hooks/"*.sh
```

Verify: Start a Claude Code session and confirm the git context appears at the top.
Try editing a `.env` file — Claude should be blocked with a clear message.

---

## Phase 3 — Integration

### [x] Step 11: Configure the Dev Server Preview (Claude Desktop)

**Goal:** Enable Claude to launch, view, and interact with a local development
server directly from the Claude Code tab in the Desktop app.

**What it enables:** Claude can start your dev server, take screenshots of
the running app, click elements, inspect the DOM, and verify changes visually —
all without leaving the conversation.

**How it works:** A `.claude/launch.json` file in your project tells Claude how
to start the server. This is configured per project, not globally.

**File location:** `[project-root]/.claude/launch.json`

**Format:**
```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "dev",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 3000
    }
  ]
}
```

**Template file added:** `projects/_template/.claude/launch.json` is included in
the project template. Every new project scaffolded from the template has it ready
to fill in with the correct dev command and port.

Verify: Open a project in the Claude Code tab, ask Claude to start the dev
server, and confirm it can take a screenshot of the running app.

---

### [x] Step 12: IDE Integration (VS Code)

**Goal:** Use Claude Code from inside VS Code for in-editor assistance.

**Note:** This is the same Claude Code engine, now embedded in the editor.
Useful when you want to stay inside VS Code rather than switching to the Desktop app.

**Setup:**
1. Install the Claude Code extension from the VS Code marketplace.
2. Authenticate with your Anthropic account.
3. Open the Claude panel (sidebar).

**Key features:**
- Chat panel with full access to the open project
- Suggested edits appear as inline diffs you can accept/reject
- Claude sees open files, errors, and diagnostics directly
- Does NOT have autocomplete/copilot-style suggestions — works through chat
- MCP servers configured globally are available in VS Code sessions too

**Limitations vs CLI/Desktop:**
- No multi-agent orchestration
- No third-party API providers (Bedrock, Vertex) — CLI only
- No app preview / dev server integration

Verify: Open VS Code, open the Claude panel, and ask a question about an
open file. ✓ Done — this guide was completed using the VS Code extension.

---

### [x] Step 13: Shell Environment Setup

**Goal:** Ensure Node.js, npm, and the Claude Code CLI are available to Claude Code's shell.

**Shell decision:** Claude Code's embedded shell is Git Bash (MSYS2), but it runs
as a non-interactive process — `.bashrc` is not auto-sourced and Bash is unreliable
for Windows PATH resolution. **PowerShell is used for all shell operations.**

**What was set up:**
- Installed **nvm-windows** for Node.js version management
  - Location: `C:\Users\[YOUR-USERNAME]\AppData\Local\nvm` (non-standard; check with PowerShell if missing)
  - Node symlink: `C:\nvm4w\nodejs`
- Installed **Node.js v24.14.0 LTS** via nvm
- Installed **Claude Code CLI 2.1.68** via `npm install -g @anthropic-ai/claude-code`
- Set **PowerShell execution policy** to `RemoteSigned` (user scope) — required for npm/claude `.ps1` wrappers
- Fixed Windows User PATH: replaced `%NVM_HOME%`/`%NVM_SYMLINK%` references (not expanded by Git Bash)
  with literal paths via PowerShell registry write

**Created `~/.bashrc`** (for future Git Bash login shells):
```bash
export NVM_HOME="/c/Users/[YOUR-USERNAME]/AppData/Local/nvm"
export NVM_SYMLINK="/c/nvm4w/nodejs"
export PATH="$NVM_HOME:$NVM_SYMLINK:$PATH"
```

**Verify (PowerShell):**
```powershell
node --version   # v24.14.0
npm --version    # 11.9.0
claude --version # 2.1.68
```

---

## Phase 4 — Workflow

### [x] Step 14: Document the New Project Workflow

**Goal:** Define and document the repeatable process for starting any new
project with Claude Code.

See `instructions/new-project-workflow.md` for the full workflow.

**Summary:**
1. Run `/new-project` in a Claude Code session — scaffolds from `_template/`
2. Fill in `projects/[name]/CLAUDE.md` with project overview and conventions
3. Open Claude Code from `projects/[name]/` to scope the session
4. Run `/session-start` to load memory and knowledge
5. Work; Claude maintains project-scoped auto-memory automatically

---

### [x] Step 15: Seed the Knowledge Base

**Goal:** Give Claude persistent reference material it can draw on for any
project — without re-explaining things every session.

**Location:** `Claude Code/knowledge/`

**Seeded files:**
| File | Contents |
|------|---------|
| `windows-environment.md` | Paths, PowerShell patterns, PATH quirks, nvm-windows |
| `claude-code.md` | Settings, memory, MCP, hooks, commands, models |
| `python.md` | pyenv-win, venv, packaging, Azure/Graph SDKs, ruff |
| `typescript.md` | tsconfig, types, async, Azure Functions, SPFx, tooling |
| `powershell.md` | Syntax, env vars, Az module, Microsoft.Graph module |
| `bicep.md` | Bicep IaC syntax, deployment, modules, resource patterns |

**Microsoft Azure / M365 docs:** Covered by the `microsoft-learn` MCP server
(live, always current) rather than static files. Add `"use microsoft-learn"` to
any prompt to pull official docs into context.

**To add more knowledge later:** Use `/research` — Claude will web-search a topic,
synthesise it into a clean reference file, and save it to `knowledge/`.

---

## Phase 5 — WSL 2 Setup

> Skip this phase if you chose the Native Windows path.

### [x] Step 16: Install WSL 2 and Ubuntu

**Goal:** Get a real Linux shell running on Windows, which Claude Code can use as its native environment.

**From PowerShell (run as Administrator):**
```powershell
wsl --install -d Ubuntu
```

Restart when prompted. On first launch, create a Linux username and password.

**Configure WSL 2 resources** — create `C:\Users\[YOUR-USERNAME]\.wslconfig`:
```ini
[wsl2]
memory=8GB
processors=4
swap=2GB
localhostForwarding=true
```
Adjust to your machine. This caps WSL's RAM usage and enables `localhost` forwarding for dev servers.

**Verify:**
```powershell
wsl --list --verbose
# Ubuntu should show VERSION 2
```

---

### [x] Step 17: Install Development Tools in WSL

**Goal:** Set up Node.js and the Claude Code CLI inside WSL.

Open the Ubuntu terminal (Windows Terminal → Ubuntu tab):

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git curl wget

# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc

# Install Node.js LTS
nvm install --lts
nvm use --lts

# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Verify
node --version   # v24.x
npm --version
claude --version
```

---

### [x] Step 18: Link Claude Configuration from OneDrive

**Goal:** Share the same CLAUDE.md, memory, commands, and knowledge between WSL and
native Windows by symlinking WSL's `~/.claude` to the OneDrive-backed config.

```bash
ONEDRIVE="/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude"

# Link ~/.claude to the shared OneDrive config
ln -s "$ONEDRIVE/.claude" ~/.claude

# Verify
ls -la ~/.claude        # → symlink to /mnt/c/.../OneDrive/Claude/.claude
cat ~/.claude.json      # Should NOT exist yet — create it next
```

**Create a WSL-specific `~/.claude.json`** (not symlinked — WSL doesn't need `cmd /c`):

```json
{
  "model": "claude-opus-4-6",
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "microsoft-learn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    }
  }
}
```

**Note:** Unlike Windows, WSL uses plain `npx` — no `cmd /c` wrapper. The Windows
`~/.claude.json` (on OneDrive) keeps `cmd /c` for the native Windows fallback.

**Verify:**
```bash
claude  # Start Claude Code in WSL
# Run /mcp — all 3 servers should show as connected
```

**Authenticate:** On first run, Claude Code will open a browser window for login.
If the WSL browser doesn't open, copy the URL from the terminal to your Windows browser.

---

### [x] Step 19: Configure WSL Shell Shortcuts

**Goal:** Add a shell function to open any project in VS Code from WSL in one command,
with the correct folder and WSL connection already active.

Add the following to `~/.bashrc`:

```bash
# Claude Code — open a project in VS Code via Remote-WSL
cco() {
  local projects_dir="/mnt/c/Users/[YOUR-USERNAME]/OneDrive/Claude/Claude Code/projects"
  local projects=($(ls "$projects_dir" | grep -v '^_'))

  if [ -n "$1" ]; then
    cd "$projects_dir/$1" && code .
    return
  fi

  echo "Select a project:"
  select project in "${projects[@]}"; do
    [ -n "$project" ] && cd "$projects_dir/$project" && code . && break
  done
}
```

```bash
source ~/.bashrc
```

**Usage:**
- `cco` — shows a numbered list of all projects; type the number to open
- `cco claude-code-setup` — opens that project directly

**Why `code .` from WSL instead of opening VS Code first:**
When VS Code is launched via `code .` from a WSL terminal, it opens already connected
to WSL with the correct folder as the workspace root. This avoids the manual
"Reopen in WSL" step and ensures VS Code's git extension receives file change
events correctly.

**Note:** The `_template` folder is excluded from the picker automatically.

**VS Code settings for WSL reliability:**

Add to VS Code **user settings** (`C:\Users\[YOUR-USERNAME]\AppData\Roaming\Code\User\settings.json`):
```json
{
    "git.autoRefresh": true,
    "remote.WSL.fileWatcher.polling": true
}
```

Add to **workspace settings** (`.vscode/settings.json` in each project):
```json
{
    "git.autoRefresh": true,
    "files.watcherExclude": {}
}
```

`remote.WSL.fileWatcher.polling` forces VS Code to poll for file changes on `/mnt/c/`
paths rather than relying on inotify events, which don't propagate reliably across the
WSL/Windows boundary. Without this, the Source Control panel requires a manual refresh
after Claude Code edits files.

---

## Phase 6 — WSL Environment

### [x] Step 20: Python Environment in WSL

**Goal:** Set up a full Python development environment in WSL 2 with version
management via pyenv.

**Why pyenv over system Python:** Ubuntu ships Python 3.12. pyenv lets you install
any version per-project without touching the system Python.

#### Step 1: Install pip and venv for system Python

```bash
sudo apt update && sudo apt install -y python3-pip python3-venv
```

#### Step 2: Install pyenv build dependencies

```bash
sudo apt install -y libssl-dev libbz2-dev libreadline-dev libsqlite3-dev \
  libffi-dev liblzma-dev curl git
```

#### Step 3: Install pyenv

```bash
curl -fsSL https://pyenv.run | bash
```

#### Step 4: Add pyenv to ~/.bashrc

```bash
cat >> ~/.bashrc << 'EOF'

# pyenv
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
EOF
source ~/.bashrc
```

#### Step 5: Install Python and set as global default

```bash
pyenv install 3.13.2
pyenv global 3.13.2
python --version   # 3.13.2
pip --version      # 24.x
```

Note: A tkinter warning during install is harmless — GUI toolkit not needed in WSL.

#### Using venv per project (always do this, never install packages globally)

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
deactivate
```

**Versions installed:** Python 3.13.2, pip 24.3.1, pyenv 2.6.25

---
