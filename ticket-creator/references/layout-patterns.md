# Layout Patterns

Four canonical layout patterns are used across the application. Each has a **live mockup** under `/mockups/<template>` and follows specific structural rules.

## Decision Table

| Page type | Template | Fixed header? | Aside? | Chat input? | Example route |
|-----------|----------|:---:|:---:|:---:|---|
| Simple content page | **Base Layout** | Yes | No | No | Settings, single-form pages |
| List / table page | **Overview** | No | No | No | `/my-studio/courses`, `/institution/reading-lists` |
| Detail / entity page | **Detail (Subpage)** | Yes | Yes (sticky) | No | `/courses/[courseId]`, `/resources/[resourceId]` |
| Analytics + chat page | **Analytics** | Yes | Yes (sticky) | Yes (fixed bottom) | `/courses/[courseId]/analytics` |

## 1. Base Layout

**Mockup:** `/mockups/main-layout`
**Component:** `PageLayout` (`src/components/shared/page-layout.tsx`) — use directly when no aside is needed.

### Structure

```
Fixed header (sidebar-aware)
  +-- Breadcrumbs (left)
  +-- Action buttons (right)
Content container
  +-- Page content
```

### Key classes

```
Header:    fixed right-0 top-0 z-[5] flex h-16 items-center border-b border-gray-200
           bg-white shadow-sm transition-[left] duration-200 ease-linear
           style={{ left: sidebarWidth }}

Header inner: mx-auto flex h-full w-full max-w-[1400px] items-center justify-between
              gap-4 pl-12 pr-6

Content:   mx-auto max-w-[1400px] pb-6 pl-12 pr-6 pt-[88px]
```

### Sidebar coordination

```typescript
const { state: sidebarState } = useSidebar();
const sidebarWidth = sidebarState === 'expanded' ? '16rem' : '3rem';
```

The header uses `position: fixed` with a dynamic `left` value matching the sidebar width. This keeps the header visually aligned with the content area as the sidebar expands/collapses.

---

## 2. Overview Page

**Mockup:** `/mockups/overview`
**Real example:** `src/app/[locale]/(main)/my-studio/courses/page.tsx`

### Structure

```
Container (no fixed header -- scrolls with page)
  +-- Header -- icon + title + description + primary action button
  +-- Stats cards -- 4-column grid
  +-- Tabs -- status filters
  +-- Toolbar -- search input + filter button
  +-- Data table + pagination
```

### Key classes

```
Container: mx-auto w-full px-4 py-6 sm:px-6 lg:px-8

Header:    mb-6 flex flex-col gap-4 sm:mb-8 sm:flex-row sm:items-center sm:justify-between

Stats:     mb-6 grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4

Tab bar:   w-full justify-start rounded-none border-b bg-transparent p-0
Tab item:  rounded-none border-b-2 border-transparent bg-transparent px-4
           data-[state=active]:border-primary data-[state=active]:shadow-none
```

### Notes

- No `useSidebar` needed — the page sits inside `SidebarInset` and flows naturally.
- Stats cards use `Card` with icon in the header and a single metric in the content.
- Toolbar sits between tabs and the table, using `Input` with a search icon and a filter `Button`.

---

## 3. Detail Page (Subpage)

**Mockup:** `/mockups/subpage`
**Component:** `PageLayout` with `aside` prop (`src/components/shared/page-layout.tsx`)
**Real example:** `src/app/[locale]/(main)/courses/[courseId]/page.tsx`

### Structure

```
Fixed header (sidebar-aware)
  +-- Breadcrumbs
  +-- Action buttons
Content container
  +-- Two-column flex layout
       +-- Main (flex-1) -- info card + tabs
       +-- Aside (w-80, sticky) -- metadata + related items
```

### Key classes

```
Two-column: flex flex-col gap-6 lg:flex-row

Main:       min-w-0 flex-1
Main inner: space-y-6

Aside:       w-full shrink-0 lg:w-80
Aside inner: space-y-6 lg:sticky lg:top-header-sticky
```

### Notes

- `min-w-0` on main prevents flex children from overflowing (critical for tables/long text).
- `lg:top-header-sticky` is a custom Tailwind value that accounts for the fixed header height.
- The aside typically contains a details card (icon + label pairs separated by `Separator`) and a related-items card.

---

## 4. Analytics Page

**Mockup:** `/mockups/analytics`
**Real example:** Chat layout from `src/components/chat/chat-container.tsx`

### Structure

```
Outer wrapper (flex column, viewport height)
  +-- Fixed header (sidebar-aware)
  +-- Scrollable content (flex-1 overflow-y-auto)
  |    +-- Two-column layout (same as Detail Page)
  |         +-- Main -- info card + charts grid + tabs
  |         +-- Aside (sticky)
  +-- Fixed chat input (shrink-0 border-t)
```

### Key classes

```
Outer:     flex h-[calc(100vh-4rem)] flex-col

Scroll:    flex-1 overflow-y-auto

Chat bar:  shrink-0 border-t bg-background
Chat inner: mx-auto max-w-[1400px] px-12 py-4
```

### Key difference from Detail Page

The outer `h-[calc(100vh-4rem)]` flex column with `overflow-y-auto` on the middle section creates a viewport-locked layout where only the content scrolls, while the header and chat input stay pinned.

---

## Reusable Components

| Component | Path | Used in |
|-----------|------|---------|
| `PageLayout` | `src/components/shared/page-layout.tsx` | Base Layout, Detail Page |
| `PageBreadcrumb` | `src/components/shared/page-breadcrumb.tsx` | All pages with fixed headers |
| `useSidebar` | `@/components/ui/sidebar` | Pages with fixed headers (for `left` offset) |
| `Card` / `Badge` / `Button` | `@/components/ui/` | All templates |
| `Tabs` / `TabsList` / `TabsTrigger` / `TabsContent` | `@/components/ui/tabs` | Overview, Detail, Analytics |
| `Table` / `TableHeader` / `TableBody` / `TableRow` / `TableHead` / `TableCell` | `@/components/ui/table` | Overview |
| `Separator` | `@/components/ui/separator` | Detail, Analytics (aside cards) |

## Checklist

- [ ] Identified the correct template from the decision table
- [ ] Pages with fixed headers use `useSidebar` for `left` offset
- [ ] Two-column layouts use `min-w-0 flex-1` for main and `w-full shrink-0 lg:w-80` for aside
- [ ] Tabs use the underline style (`border-b-2`, no shadow, transparent bg)
- [ ] Content container is `max-w-[1400px]` with `pl-12 pr-6`
- [ ] Overview pages use `mx-auto w-full px-4 py-6 sm:px-6 lg:px-8` (no fixed header)
