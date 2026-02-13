# Example: UI Ticket (Page Creation)

> Based on real ticket S-1163 (Chat Page Route & Layout), enhanced with layout reference, component reuse, and data shapes to prevent open-ended UI decisions.

---

## Summary

Create the chat page route at `/chat/[chatId]` with server-side data loading, authorization, and a layout that follows the existing detail page pattern.

## Context

The AI Discovery Agent needs a chat interface where users can converse with the agent about course resources. This page is the main interaction point for the discovery feature.

**Project:** [Chat UI v0 — Mocked Proof of Concept](https://linear.app/sylla/project/chat-ui-v0)

**Dependencies:**
- Blocked by: S-1161 (Database queries for chat operations — needs the queries to load data)

---

## Requirements

- Create the chat page route with server-side data loading
- Verify the user has access to the course associated with the chat
- Load chat metadata, course info, and message history server-side
- Pass loaded data to client components for rendering
- Return 404 if chat doesn't exist or user is unauthorized

---

## Technical Context

**Repo:** nextjs

**Existing pattern to follow:**
- Page structure: `src/app/[locale]/(main)/courses/[courseId]/page.tsx` — follow this pattern for server-side data loading + auth check + passing props to client components
- Server actions: `src/app/[locale]/(main)/courses/[courseId]/_actions.ts` — follow for mutation patterns
- Authorization: use `checkRelation()` from existing FGA patterns

**Systems involved:** NeonDB (via Drizzle), OpenFGA (authorization)

**New/modified files:**
- `src/app/[locale]/(main)/chat/[chatId]/page.tsx` — Server component page (NEW)
- `src/app/[locale]/(main)/chat/[chatId]/_actions.ts` — Server actions for chat mutations (NEW)

---

## UI Specification

**Page URL / Route:** `/chat/[chatId]`

**Component type:** Server component (page.tsx is server, child components may be client)

**Layout reference:** Detail Page (Subpage) — follow `src/app/[locale]/(main)/courses/[courseId]/page.tsx`:
- Server-side data loading in page.tsx
- Auth check before rendering
- Props passed down to client components

**Existing components to reuse:**
- `PageBreadcrumb` from `@/components/shared/page-breadcrumb` — for navigation
- `PageHeader` from `@/components/shared/page-header` — for consistent header
- `getCurrentUser()` from `@/lib/auth` — for auth
- `checkRelation()` from `@/lib/fga` — for course access verification

**Data source:** Server-side Drizzle queries (from S-1161)

**Data shape:**
```typescript
interface ChatPageData {
  chat: {
    id: string;
    courseId: string;
    title: string;
    createdAt: Date;
  };
  course: {
    id: string;
    title: string;
  };
  messages: Array<{
    id: string;
    role: 'user' | 'assistant';
    content: string;
    createdAt: Date;
  }>;
}
```

**Suspense/loading strategy:** Skeleton loading state for the message list area. Server component handles initial data — no client-side fetch on load.

---

## Acceptance Criteria

- [ ] Route created at `/chat/[chatId]`
- [ ] Authorization checks user can access the course
- [ ] Chat metadata, course info, and messages loaded server-side
- [ ] Server actions file created for future mutations
- [ ] Returns 404 if chat doesn't exist or unauthorized
- [ ] Page renders with breadcrumb: Home > Courses > [Course Title] > Chat
- [ ] `bun typecheck` passes

---

## Verification

- [ ] Page renders correctly with sidebar open and closed
- [ ] `bun typecheck` passes

---

## Out of Scope

- Message sending/receiving (separate ticket)
- Real-time message streaming (separate ticket)
- Chat creation flow (separate ticket)
