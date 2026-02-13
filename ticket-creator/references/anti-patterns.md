# Anti-Pattern Detection Rules

Evaluate every ticket against these 6 rules before compiling the final ticket. Surface warnings for any that trigger, and ask the mitigation question. Present all warnings together rather than one at a time.

---

## Rule 1: No Pattern Referenced

**Trigger:** The ticket has no pattern reference, or user said "I don't know" when asked for a reference.

**Warning:**
> No base pattern specified — the implementing agent may invent conventions that diverge from the codebase.

**Why this matters:** Tickets referencing an existing pattern (e.g., "use OAPEN harvester as base") almost always succeed in one shot, while tickets without pattern references require multiple iteration passes.

**Mitigation:** Ask: *"Which existing file or implementation should this follow?"*

Use `Task` with subagent_type `Explore` to search the codebase for candidates and suggest them.

**Severity:** High — #1 predictor of ticket failure.

---

## Rule 2: Schema-Only for External API

**Trigger:** Data-Engine or Backend ticket that describes field mappings or schemas but includes no actual JSON response example.

**Warning:**
> Only a schema was provided, no example API response — high risk of wrong field mapping.

**Why this matters:** Multiple harvester tickets (S-1119, S-1128) required refactoring because the return structure was implemented incorrectly without a concrete data sample.

**Mitigation:** Ask: *"Can you provide an actual example JSON response from this API? Schema alone causes field mapping errors."*

**Severity:** High.

---

## Rule 3: UI Without Layout Reference

**Trigger:** UI ticket with no layout template selected and no existing page referenced as "similar to."

**Warning:**
> No layout reference — the implementing agent may create inconsistent UI that doesn't match existing pages.

**Mitigation:** Ask: *"Which layout template should this page use?"* Present the 4 options from `references/layout-patterns.md`:
1. Base Layout (simple content page)
2. Overview (list/table page)
3. Detail/Subpage (entity page with aside)
4. Analytics (charts + chat input)

Or ask: *"Which existing page is this most similar to?"*

**Severity:** Medium.

---

## Rule 4: Ticket Touches Multiple Concerns

**Trigger:** The ticket mentions changes across 3 or more of: UI components, server actions, database schema, background jobs, external APIs, authorization/FGA.

**Warning:**
> This ticket touches multiple concerns and may have hidden side effects.

**Why this matters:** Tickets that span multiple concerns without explicitly listing side effects lead to incomplete implementations — the primary change is handled but secondary effects are missed.

**Mitigation:** Ask: *"What other tables, queries, components, or jobs does this change affect?"*

Consider suggesting the ticket be split into focused, independently implementable tickets.

**Severity:** Medium.

---

## Rule 5: First-Time Pattern

**Trigger:** Codebase search found no similar implementation, and the user confirmed nothing similar exists.

**Warning:**
> No prior implementation exists in the codebase — extra specification is needed to prevent pattern divergence.

**Mitigation:** Require these additional details:
- Detailed TypeScript/Python types (input and output)
- Folder location and file naming
- Naming conventions for functions/components
- Which existing patterns/conventions should be respected even though no direct example exists

Optionally, if the scope is large, suggest creating a spike ticket first to design the approach — but do not push this if the user has enough detail to specify the pattern inline.

**Severity:** High.

---

## Rule 6: Vague Acceptance Criteria

**Trigger:** Acceptance criteria contain subjective or unmeasurable language: "works properly", "is fast", "looks good", "handles errors", "is secure".

**Warning:**
> Acceptance criteria are too vague for AI implementation — specific, testable conditions are needed.

**Why this matters:** Every successful ticket in evaluation had concrete, checkable criteria. Vague criteria like "works properly" cannot be verified by the implementing agent or reviewer.

**Mitigation:** Ask: *"Can you make this more specific? For example, instead of 'handles errors properly', say 'returns Result<T> with error message when user is not authorized'."*

Good examples from real tickets:
- "Warm Lambda invocations targeting a different environment correctly re-initialise the broker" (S-1222)
- "Size validation: minimum 1 MB, maximum 100 MB" (S-1156)
- "`validate_isbn()` correctly validates ISBN-10 and ISBN-13 with checksum" (S-1148)

**Severity:** High.

---

## Evaluation Process

1. For each rule, check if the trigger condition matches the gathered ticket information
2. For **any** rule that triggers, show the warning and ask the mitigation question
3. Wait for user response before continuing
4. Multiple rules can trigger simultaneously — present all warnings together
5. If **no** rules trigger, proceed silently to the next stage
