# Sylla Skills

Agent skills for the Sylla edtech stack.

A collection of agent skills for AI coding assistants, built for Sylla's stack. Compatible with Claude Code, Codex, Gemini CLI, Cursor, and other tools following the [agentskills.io](https://agentskills.io) spec.

## Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [ticket-creator](./ticket-creator/) | Creates well-structured Linear tickets from feature requests, bug reports, or technical tasks | `bunx skills add sylla-bv/sylla-skills/ticket-creator` |
| [coding-standards (Python)](./python/coding-standards/) | Enforces Sylla conventions in Python data-engine code | `bunx skills add sylla-bv/sylla-skills/python/coding-standards` |
| [code-quality](./python/code-quality/) | Reviews Python files against coding standards and quality principles | `bunx skills add sylla-bv/sylla-skills/python/code-quality` |
| [update-docstrings](./python/update-docstrings/) | Audits and updates Python docstrings to match implementations | `bunx skills add sylla-bv/sylla-skills/python/update-docstrings` |
| [update-tests](./python/update-tests/) | Audits test coverage and generates pytest-based tests | `bunx skills add sylla-bv/sylla-skills/python/update-tests` |

## Installation

Install all skills:

```sh
bunx skills add sylla-bv/sylla-skills
```

Or install individually:

```sh
bunx skills add sylla-bv/sylla-skills/ticket-creator
```

## Contributing

1. Each skill lives in its own directory with a `SKILL.md` file
2. Follow the [agentskills.io](https://agentskills.io) spec
3. Keep skill files under 200 lines
4. Be specific and opinionated — encode Sylla's actual conventions

## License

MIT
