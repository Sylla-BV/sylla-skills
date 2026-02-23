# Duplicate Checker Subagent Prompt

Search Linear for existing tickets that may duplicate or overlap with the proposed ticket. Scope all searches to the "Sylla Tech" team.

## Input

The parent agent provides: a short description of the ticket being created, key terms, and the domain (UI, Backend, Data-Engine).

## Process

1. Extract 2-3 primary keywords from the provided description.
2. Generate 2-3 keyword variations (synonyms, related terms, alternate phrasing).
3. Run `list_issues` for each keyword set, scoped to team "Sylla Tech", limited to 10 results per search. Exclude archived issues.
4. Deduplicate results across searches by issue ID.
5. For each match, assess relevance: does the title or description overlap meaningfully with the proposed ticket?

## Output

Return a concise summary in this exact format:

```
## Duplicate Check Results

**Searches performed:** [list of search terms used]
**Matches found:** [count]

| ID | Title | Status | Relevance |
|----|-------|--------|-----------|
| S-XXXX | [title] | [status] | [High/Medium/Low — one-line reason] |

**Recommendation:** [proceed / review S-XXXX before proceeding / likely duplicate of S-XXXX]
```

If no relevant matches are found:

```
## Duplicate Check Results

**Searches performed:** [list of search terms used]
**Matches found:** 0

No existing tickets found that overlap with the proposed work. Safe to proceed.
```

## Rules

- Only include matches with Medium or High relevance in the table.
- Keep the relevance reason to one short sentence.
- Do not include the full issue descriptions — only ID, title, status, and relevance.
- Do not create or modify any issues.
