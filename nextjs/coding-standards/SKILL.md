---
name: coding-standards
description: >-
  This skill should be used when the user asks about "coding standards",
  "code style", "how should I write this", "is this correct style",
  "what's the convention for", "interface vs type", "should I use React.FC",
  "how do we handle errors", "tryCatch pattern", "db query style",
  "arrow functions", "named vs default export", "how to structure this file",
  "barrel files", "institutionId", "multi-tenant", "page-content pattern",
  "naming conventions", "SCREAMING_SNAKE_CASE", "reviewing a PR",
  "does this pass code review", or wants to know whether
  code follows Sylla's Next.js 16 repository standards.
---

# Coding Standards

Sylla-specific standards for the Next.js 16 App Router codebase. All rules are settled and enforced in code review. Full rationale and code examples for every rule are in **`references/standards.md`**.

## How to Apply This Skill

- For quick lookups (rule name, convention, where a file belongs), use the Quick Reference tables below.
- For rationale, code examples, or edge cases, consult `references/standards.md`.
- When reviewing code, run through the Code Smell Checklist at the bottom of this file.

---

## Code Quality Principles

### 1. Readability First

- Code is read far more than it is written
- Clear variable and function names over short ones
- Self-documenting code preferred over comments
- Consistent formatting throughout

### 2. KISS (Keep It Simple, Stupid)

- Simplest solution that works
- Avoid over-engineering
- No premature optimization
- Easy to understand beats clever

### 3. DRY (Don't Repeat Yourself)

- Extract common logic into functions
- Create reusable components
- Share utilities across modules
- Avoid copy-paste programming

### 4. YAGNI (You Aren't Gonna Need It)

- Don't build features before they're needed
- Avoid speculative generality
- Add complexity only when a real requirement demands it
- Start simple, refactor when needed

### 5. Security & Safety First

- No `as any` — strictly forbidden, find the correct type
- Multi-tenant scoping always — every DB query must include `institutionId` (unless `isAdmin` is confirmed via `isSuperAdmin`)
- Server Components by default — only add `'use client'` when you need interactivity or browser APIs

---

## Quick Reference

### TypeScript

| Rule | Standard |
|------|----------|
| Object shapes, props, data models | `interface` |
| Unions, intersections, mapped types | `type` |
| `React.FC<T>` | Banned — type props with `interface` directly |
| Named type for trivial one-off shape | Banned — use an inline type annotation instead |
| Return types on exported functions | Required |
| Return types on internal helpers | Inferred |
| Intentional absence (DB returns) | `null` |
| Optional params | `undefined` (use `?`) |
| Null guards | Optional chaining + nullish coalescing |

### Functions & Components

| Rule | Standard |
|------|----------|
| Function syntax | Arrow functions always |
| `function` keyword | Banned — except Next.js file-convention `export default function` |
| Single-component file | `export default` |
| Multi-export file | Named exports |
| Props type name | `interface ComponentNameProps` |
| Short props list (<4) | Destructure in signature |
| Long props list (4+) | Named `props` param, destructure in body |
| Default values | In destructure signature, not body |

### Error Handling

| Rule | Standard |
|------|----------|
| Exported functions that can fail | Wrap entire body in `tryCatch`, return `Result<T>` |
| `throw` from exported functions | Banned |
| Consuming `Result<T>` | Check `result.data` branch before use |
| Calling server actions from client | No `try/catch` needed — server action is wrapped in `tryCatch`; handle `Result<T>` branches |
| React-query mutations | Handle errors via `onError` callback, not `try/catch` in `mutationFn` |

### Database

| Rule | Standard |
|------|----------|
| Query syntax | Prefer `db.query.*` (relational API); use `db.select()` for inner joins / aggregations |
| Raw SQL | Banned (except custom migrations) |
| Multiple DB operations | `db.batch()` — not `Promise.all` |
| `institutionId` on user/institution queries | Required unless `isSuperAdmin` confirms admin |

### File & Naming

| Thing | Convention |
|-------|-----------|
| Files & folders | kebab-case |
| Variables | camelCase |
| Functions | camelCase, verb-first |
| Components | PascalCase |
| Interfaces & types | PascalCase |
| Hooks | `use` prefix |
| Boolean vars | `is`, `has`, `can` prefix |
| Constants (all scopes) | `SCREAMING_SNAKE_CASE` |
| Barrel `index.ts` | Avoid — direct imports via alias |

### Where Things Live

| What | Where |
|------|-------|
| Reusable DB query | `src/db/queries/[domain].ts` |
| Route-specific server action | `src/app/.../[route]/_actions.ts` |
| Reusable server action (non-DB) | `src/actions/[feature].ts` |
| React component | `src/components/[domain]/` |
| Custom hook | `src/hooks/use-[name].ts` |
| Shared constants | `src/lib/constants.ts` |
| Zod schemas / types | co-located `[name].types.ts` |

### React & Next.js

| Rule | Standard |
|------|----------|
| Async | `async/await` always |
| Parallel non-DB fetches | `Promise.all` |
| Parallel DB fetches | `db.batch()` |
| Conditionals | Early returns over nested ternaries |
| State that depends on previous | Functional updater `setX(prev => ...)` |
| `page-content.tsx` | Use when page is client-driven but needs a one-time server data load |
| Routes with top-level `await` | Must have sibling `loading.tsx` |

---

## Code Smell Checklist

Before opening a PR, check:

- [ ] Function >~40 lines — split it
- [ ] >3 levels of nesting — use early returns
- [ ] Magic number/string — extract to `SCREAMING_SNAKE_CASE` constant
- [ ] `as any` — find the correct type
- [ ] `type` for a named object shape — change to `interface`
- [ ] Named `interface`/`type` for a trivial shape used in one place — inline it
- [ ] `React.FC<T>` — remove it
- [ ] `function` keyword (non–file-convention) — convert to arrow
- [ ] Named export on single-component file — convert to default export
- [ ] `Promise.all` for DB queries — replace with `db.batch()`
- [ ] `throw` inside exported function — wrap in `tryCatch`
- [ ] Missing `institutionId` in DB query (and no `isSuperAdmin` check) — multi-tenant violation
- [ ] `middleware.ts` — must be `proxy.ts`
- [ ] Wrapper function that only calls through — delete it
- [ ] Barrel `index.ts` without justification — remove, use direct imports

