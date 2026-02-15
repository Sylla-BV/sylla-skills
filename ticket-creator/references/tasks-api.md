# Claude Code Tasks API — Reference for Sylla

Compiled from official docs, community guides, and DeepWiki analysis. Feb 2026.

## TL;DR

Claude Code has two task management systems. The old one (TodoWrite) is session-scoped
and ephemeral. The new one (Tasks API, default since v2.1.19) persists to disk, supports
dependencies, and works across sessions/terminals. Neither is triggered by special syntax
in SKILL.md — they're internal tools Claude Code uses during execution.

For our ticket-creator skill: The `- [ ]` checkboxes in acceptance criteria are just
markdown. Claude Code reads them and builds its own task plan using TaskCreate. No special
syntax needed.

## System Comparison

| Feature | TodoWrite (Legacy) | Tasks API (v2.1.16+) |
|---|---|---|
| Persistence | Session memory only | Disk: `~/.claude/tasks/` |
| Survives | Nothing — lost on `/compact`, exit, crash | Everything — sessions, reboots, `/compact` |
| Multi-session | No | Yes — resume from any terminal |
| Dependencies | No — manual ordering | Yes — DAG via `blockedBy` field |
| Multi-agent | No — single agent | Yes — shared state across terminals |
| Status states | `pending` / `in_progress` / `completed` | `pending` / `in_progress` / `completed` / `failed` |
| Toggle | `CLAUDE_CODE_ENABLE_TASKS=false` | Default since v2.1.19 |

## Tasks API — Four Operations

### 1. TaskCreate

Creates a new task with optional dependencies and metadata.

```
TaskCreate(
  subject: "Implement login endpoint",        # Short title (visible in TaskList)
  description: "Create POST /auth/login...",   # Detailed notes (NOT visible in TaskList!)
  blockedBy: [],                               # Task IDs this depends on
  metadata: { priority: "high", files: [...] } # Custom fields (NOT visible in TaskList!)
)
```

### 2. TaskList

Returns high-level overview of ALL tasks. Critical limitation: only shows a subset of
fields.

- **Visible:** `id`, `subject`, `status`, `owner`, `blockedBy`
- **Hidden:** `description`, `metadata`, `activeForm` (requires TaskGet per task)

### 3. TaskGet

Fetches ALL fields for a specific task. Costs 1 API call per task.

To see all details for 20 tasks = 1 TaskList + 20 TaskGet = 21 API calls (the "1+N
problem").

### 4. TaskUpdate

Changes status, metadata, or dependencies of an existing task.

```
TaskUpdate(taskId: "task-1", status: "completed")
TaskUpdate(taskId: "task-2", status: "in_progress")
TaskUpdate(taskId: "task-3", addBlockedBy: ["task-1"])
```

## Task Schema (Full Fields)

| Field | Type | In TaskList? | Description |
|---|---|---|---|
| `id` | string | Yes | Unique identifier |
| `subject` | string | Yes | Short title (50–100 chars recommended) |
| `status` | enum | Yes | `pending` / `in_progress` / `completed` / `failed` |
| `owner` | string | Yes | Agent or user identifier |
| `blockedBy` | string[] | Yes | Task IDs this depends on |
| `description` | string | No | Detailed implementation notes |
| `metadata` | object | No | Custom fields (priority, estimates, files) |
| `activeForm` | string | No | Progress spinner text |

## Task Lifecycle

```
pending ──→ in_progress ──→ completed
  │              │
  │              └──→ failed ──→ pending (retry)
  └──→ failed (invalid)
```

## Dependencies (DAG Execution)

Tasks can declare blocking relationships:

```
Authentication System (parent)
├── 1. Login endpoint         (no deps → starts immediately)
├── 2. Token refresh          (blockedBy: [1])
├── 3. Logout endpoint        (blockedBy: [1])
└── 4. Integration tests      (blockedBy: [2, 3])
```

Execution order is enforced: task 4 won't start until both 2 and 3 complete.

## Multi-Session Coordination

Set `CLAUDE_CODE_TASK_LIST_ID` to share tasks across terminals:

```bash
# Session 1: Planning
export CLAUDE_CODE_TASK_LIST_ID="auth-system-v2"
claude
> "Create task hierarchy for JWT authentication..."

# Session 2: Resume next day
export CLAUDE_CODE_TASK_LIST_ID="auth-system-v2"
claude
> "TaskList — show me where we left off"
```

Storage: `~/.claude/tasks/auth-system-v2/tasks.json`

> **Warning:** Use repo-specific IDs to avoid cross-project contamination.

## Known Limitations & Workarounds

### The 1+N Problem

TaskList hides `description` and `metadata`. To see all details for N tasks, you need N+1
API calls. This matters for cost/token consumption.

**Workarounds:**

1. **Hybrid approach (recommended):** Use Tasks API for status/deps, markdown files for
   detailed context
2. **Subject-as-summary:** Pack critical info into the `subject` field (visible in
   TaskList)
3. **Selective fetching:** Only TaskGet for the task you're about to work on

### Tasks Don't Persist Automatically

You must set `CLAUDE_CODE_TASK_LIST_ID` for persistence. Without it, task state may not
survive restarts reliably.

### Dependencies Not Always Enforced

Claude sometimes executes blocked tasks before dependencies complete. Workaround:
explicitly remind Claude to check task dependencies before starting work.

## Relevance to Our Ticket-Creator Skill

No changes needed to our skill. Here's why:

1. The `- [ ]` checkbox syntax in our acceptance criteria and verification sections is
   standard markdown. Claude Code reads it and translates into its own TaskCreate calls
   when planning.

2. There's no special SKILL.md syntax that auto-creates Tasks. The skill's job is to
   produce a well-structured ticket description — Claude Code's job (at implementation
   time) is to parse that into a task plan.

3. The Tasks API is an implementation-time concern, not a ticket-creation-time concern.
   Our skill creates the input that Claude Code later consumes.

**Potential future enhancement:** We could add a note in the skill telling the
implementing agent to use `CLAUDE_CODE_TASK_LIST_ID` for complex multi-step tickets. But
that's a CLAUDE.md concern, not a ticket template concern.
