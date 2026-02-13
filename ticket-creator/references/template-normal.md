# Normal Ticket Template

Use this template for all non-bug work: features, improvements, refactors, new pages, new endpoints, new actors, etc.

The template has a common structure with conditional sections that activate based on the domain (UI, Backend, Data-Engine).

---

## Template Structure

```markdown
## Summary

[1-3 sentences: What is being built/changed and why.]

## Context

[Why this is needed. Link to the parent project if applicable.]

**Project:** [Link to Linear project if applicable]

**Dependencies:**
- Blocked by: [ticket IDs]
- Blocks: [ticket IDs]

---

## Requirements

[Bullet list of what the implementation must do. Keep each requirement atomic and verifiable.]

- Requirement 1
- Requirement 2
- ...

---

## Technical Context

**Repo:** [nextjs | data-engine | both]

**Existing pattern to follow:** [FILE PATH — this is the MOST IMPORTANT field]
[Brief explanation of what to copy/adapt from the referenced pattern]

**Systems involved:** [e.g., Qdrant, OpenFGA, NeonDB, Voyage AI, SQS/Dramatiq, S3]

**New/modified files:**
- `path/to/new-file.ts` — [purpose]
- `path/to/modified-file.ts` — [what changes]

---

<!-- CONDITIONAL: Include for Backend work (server actions, API, DB, jobs) -->
## Backend Specification

**Function signatures:**
```typescript
// Input/output contracts for new functions
export async function myFunction(input: InputType): Promise<OutputType> {}
```

**Schema changes (if any):**
- Table: `table_name`
- New columns: `column_name` (type, nullable, default, index?)
- Backfill strategy: [what happens to existing rows]

**Side effects:**
- [Which other tables/queries/jobs are affected by this change]
- [Cascading impacts to surface explicitly]

**Integration point:**
- [Where is this called from? Route? Job? Cron? Server action?]
- [Permissions/FGA checks required]
<!-- END CONDITIONAL: Backend -->

---

<!-- CONDITIONAL: Include for UI work (pages, components, layouts) -->
## UI Specification

**Page URL / Route:** `/path/to/page`

**Component type:** [Server component | Client component ("use client")]

**Layout reference:** [Which existing page is this most similar to? File path.]

**Existing components to reuse:**
- `ComponentName` from `@/components/path` — [how it's used]

**Data source:** [Existing query? Server action? Mock? Props from parent?]

**Data shape:**
```typescript
type PageData = {
  // TypeScript type of the data this component renders
}
```

**Query parameters:** [e.g., `?tab=adopted&sort=date`]

**Suspense/loading strategy:** [Skeleton? Suspense boundary? Lazy load?]
<!-- END CONDITIONAL: UI -->

---

<!-- CONDITIONAL: Include for Data-Engine work (harvesters, actors, pipelines) -->
## Data-Engine Specification

**Actor type:** [Harvester | Processing | Ingestion | other]

**Base pattern to follow:** [File path to base class or similar actor]

**Source URL / API endpoint:** [The external service URL if applicable]

**Example API response:**
```json
// ACTUAL JSON response, not just schema — this is critical
{
  "field": "value"
}
```

**Fields to extract:**
| Source Field | Maps To | Notes |
|-------------|---------|-------|
| `response.field` | `Model.field` | |

**External API documentation:** [Link or inline relevant docs]

**Library/version:** [Which library and API version to use]
<!-- END CONDITIONAL: Data-Engine -->

---

## Acceptance Criteria

- [ ] Criterion 1 (specific, verifiable)
- [ ] Criterion 2
- [ ] ...
- [ ] `bun typecheck` passes (nextjs) / `mypy` passes (data-engine)

---

## Verification

[Lightweight — 2-4 items max. Nudge towards testable work, don't over-specify.]

- [ ] [Most valuable test or manual check]
- [ ] `bun typecheck` passes

---

## Implementation Toolkit

skills: [discovered skills listed here during pattern discovery]

---

## Out of Scope

- [What is explicitly NOT included in this ticket]
```

---

## Field Priority (for gathering information)

When gathering ticket information, prioritize in this order:

1. **Critical**: Existing pattern to follow, What & Why, File paths
2. **High**: Function signatures / data shapes, Acceptance criteria
3. **Medium**: Schema changes, Side effects, Integration points, Dependencies
4. **Nice-to-have**: Out of scope, Verification hints

For small tickets (translations, config changes, simple fixes), only the critical fields are needed. The tighter the scope, the less context required.
