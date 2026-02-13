# Example: Backend Ticket (Server Action + API Integration)

> Based on real ticket S-1176 (searchResources tool), enhanced to prevent open-ended implementation decisions.

---

## Summary

Implement the `searchResources` agent tool that combines query generation, embedding creation, and vector search into a single atomic operation for the AI Discovery Agent.

## Context

This tool is the primary search mechanism for the AI Discovery Agent. Instead of having separate tools for query generation, embedding, and search (which would always be called in sequence), we consolidate them into one tool for efficiency.

**Project:** [AI Discovery Agent Backend](https://linear.app/sylla/project/ai-discovery-agent-backend-85462f077f73)

**Dependencies:**
- Blocked by: S-1175 (LangGraph agent core — must be merged first)

---

## Requirements

- Generate search queries from user intent + course context
- Create embeddings from generated queries using Voyage AI
- Create BM25 queries from the same generated queries
- Search Qdrant with raw embedding vectors (hybrid search)
- Return array of learning resource sections with relevance scores
- Apply multi-tenant filter (institutionId scoping)
- Capture LangSmith traces for query generation and search steps

---

## Technical Context

**Repo:** nextjs

**Existing patterns to follow:**
- Query generation: `src/ai/gen.ts:349-390` (`generateTopicSearchQueries`) — copy this pattern for generating queries from user intent
- Embeddings: `src/actions/embeddings.ts:26-28` (`embedTexts`) — use directly, don't reimplement
- Simsearch structure: `src/actions/simsearch.ts` — follow the existing Qdrant query patterns

**Systems involved:** Voyage AI, Qdrant, LangSmith

**New/modified files:**
- `src/agents/discovery/tools/search-resources.ts` — Tool implementation (NEW)
- `src/actions/simsearch.ts` — Add `searchWithVectors()` function (MODIFY)

---

## Backend Specification

**Function signatures:**

```typescript
// New function in simsearch.ts
export async function searchWithVectors(
  vectors: number[][],
  filter: QdrantFilter,
  limit: number
): Promise<SearchResult[]>

// Tool input schema
interface SearchResourcesInput {
  userIntent: string;
  courseContext: {
    courseId: string;
    title: string;
    outcomes: string[];
  };
}

// Tool output
interface SearchResourcesOutput {
  resources: Array<{
    id: string;
    title: string;
    bookTitle: string;
    summary: string;
    score: number;
  }>;
}
```

**Side effects:**
- None — this is a read-only search tool
- LangSmith traces are written as a side effect but don't affect functionality

**Integration point:**
- Called by the LangGraph discovery agent as a tool
- Results stored as `ToolResultPart` in the message's `content` column:

```typescript
{
  role: 'assistant',
  content: [
    { type: 'text', text: 'Here are relevant resources for your course:' },
    {
      type: 'tool-result',
      toolCallId: 'call_123',
      toolName: 'searchResources',
      output: { resources: [{ id: 'uuid-1', title: 'Psychology 2e', score: 0.95 }] }
    }
  ]
}
```

See S-1173 for the full message format mapping layer.

---

## Acceptance Criteria

- [ ] `searchWithVectors()` function added to `simsearch.ts`
- [ ] Tool generates relevant search queries from user intent
- [ ] Embeddings created via Voyage AI (`embedTexts`)
- [ ] BM25 queries created via `bm25.ts`
- [ ] Qdrant searched with raw vectors (hybrid: dense + BM25)
- [ ] Results include section metadata (title, summary, bookTitle, score)
- [ ] Multi-tenant filter applied (institutionId scoping)
- [ ] LangSmith traces capture query generation and search steps
- [ ] `bun typecheck` passes

---

## Verification

- [ ] Unit test: `searchWithVectors()` returns correctly shaped `SearchResult[]`
- [ ] `bun typecheck` passes

---

## Out of Scope

- Frontend rendering of search results (separate ticket)
- Reranking of search results (future improvement)
- Caching of embeddings (premature optimization)
