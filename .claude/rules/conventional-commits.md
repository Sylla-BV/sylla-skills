---
paths:
  - "**/*"
---

# Conventional Commits

All commit messages in this repository MUST follow the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification.

## Format

```
<type>(<optional scope>): <description>

[optional body]

[optional footer(s)]
```

## Types

| Type | When to use |
|------|-------------|
| `feat` | New functionality or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `chore` | Maintenance tasks (dependencies, config, tooling) |
| `test` | Adding or updating tests |
| `ci` | CI/CD pipeline changes |
| `perf` | Performance improvement |
| `style` | Formatting, whitespace, semicolons (no logic change) |
| `build` | Build system or external dependency changes |

## Scope

Use a short noun describing the area of the codebase in parentheses:

- `feat(ticket-creator): add anti-pattern detection rules`
- `fix(layout-patterns): correct sidebar width calculation`
- `docs(guidelines): clarify verification section`

Scope is optional but encouraged when the change targets a specific skill or component.

## Rules

- The description MUST be lowercase and imperative mood ("add feature" not "Added feature" or "Adds feature")
- The description MUST NOT end with a period
- Keep the first line (type + scope + description) under 72 characters
- Use the body for context on the "why", not the "what" — the diff shows the what
- Breaking changes MUST include `BREAKING CHANGE:` in the footer or `!` after the type/scope
- Always pass the commit message via HEREDOC for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
feat(ticket-creator): add anti-pattern detection rules

Combine best detection rules from seba and renato variants.
Ground each rule in real ticket failure evidence.
EOF
)"
```
