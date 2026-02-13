# Sylla Skills

Agent skills for the Sylla edtech stack.

A collection of agent skills for AI coding assistants, built for Sylla's stack. Compatible with Claude Code, Codex, Gemini CLI, Cursor, and other tools following the [agentskills.io](https://agentskills.io) spec.

## Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [ticket-creator](./ticket-creator/) | Creates well-structured Linear tickets from feature requests, bug reports, or technical tasks | `bunx skills add sylla-bv/sylla-skills/ticket-creator` |

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
