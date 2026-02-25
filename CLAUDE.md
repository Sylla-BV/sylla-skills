# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A collection of AI coding assistant **skills** for the Sylla edtech stack, distributed via `bunx skills add sylla-bv/sylla-skills`. Skills are self-contained instruction sets consumed by Claude Code, Cursor, Gemini CLI, and similar tools.

## Skill Structure

Every skill follows this layout:

```
<domain>/<skill-name>/
├── SKILL.md              # Main skill file with YAML frontmatter
├── references/           # Templates, guidelines, anti-patterns
└── examples/             # Real-world ticket/output examples
```

**SKILL.md frontmatter** defines: `name`, `description`, `version`, `allowed_tools`. Keep SKILL.md under 200 lines — offload detail to `references/`.

## Skills in This Repo

| Skill | Location | Purpose |
|-------|----------|---------|
| ticket-creator | `ticket-creator/` | Create Linear tickets with pattern-first methodology |
| coding-standards | `nextjs/coding-standards/` | Enforce Sylla conventions in Next.js 16 App Router |
| brainstorming | `brainstorming/` | Design exploration before any implementation |
| agent-browser | `nextjs/agent-browser/` | Browser automation CLI for AI agents (navigation, forms, screenshots) |
| verify | `nextjs/verify/` | Multi-stage quality gate before opening a PR |
| coding-standards (Python) | `python/coding-standards/` | Enforce Sylla conventions in Python data-engine code |
| verify (Python) | `python/verify/` | Code quality gate for Python files |
| code-upkeep | `python/code-upkeep/` | Audit and update Python docstrings and test coverage |

## Commit Conventions

Conventional Commits are enforced (see `.claude/rules/conventional-commits.md`):

```
feat(ticket-creator): add anti-pattern detection
fix(coding-standards): correct tryCatch example
chore(brainstorming): update checklist order
```

- Scope = the skill directory name being modified
- Lowercase, imperative mood, under 72 chars on line 1
- Always use a HEREDOC when committing from the CLI

## Key Design Principles

**Pattern-first**: Tickets must reference an existing implementation. No open-ended architectural decisions left for the implementing agent.

**Anti-pattern gates**: `ticket-creator/references/anti-patterns.md` defines 6 rules that block ticket creation (e.g., no pattern referenced, vague acceptance criteria).

**Multi-tenant safety**: All DB queries in the codebase must scope by `institutionId` — this is enforced by the coding-standards skill.

**No type escapes**: `as any` is forbidden; `interface` over `type` for object shapes.

## MCP & Tool Permissions

`.claude/settings.json` grants:
- Linear MCP for issue management
- `gh` CLI and `git push` for GitHub operations
- `WebSearch` and `WebFetch`

When adding new skills that require additional tools, update `allowed_tools` in the skill's YAML frontmatter and document any new MCP requirements.
