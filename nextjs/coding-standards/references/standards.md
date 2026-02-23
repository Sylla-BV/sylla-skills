# Full Coding Standards

Sylla-specific standards for this Next.js 16 App Router codebase. All rules below are settled and enforced in code review.

---

## TypeScript

### `interface` vs `type`

Use `interface` for all named object shapes — props, data models, function parameter objects. Use `type` for unions, intersections, and mapped/utility types that cannot be expressed as an interface.

The rationale: TypeScript compiles interfaces faster than type aliases because interfaces are cached by name. Type aliases are re-evaluated on every use.

```typescript
// ✅ interface for props and object shapes
interface StatusBadgeProps extends VariantProps<typeof statusBadgeVariants> {
  status: NonNullable<VariantProps<typeof statusBadgeVariants>['status']>;
  className?: string;
}

interface CoursePageData {
  course: CourseWithEnhancementStatus;
  permissions: CoursePermissions;
  isLocked: boolean;
  isDraft: boolean;
}

// ✅ type for unions and utility types
type Role = 'admin' | 'member' | 'viewer';
type SnakeCaseObject<T> = { [K in keyof T as SnakeCase<K>]: T[K] };
type Result<T> = Success<T> | Failure;

// ❌ type for plain object shapes
type ReadingListFormProps = {
  organizationalUnits: OrgUnit[];
  currentUserId: string;
};
```

### Avoid Unnecessary Type Declarations

Do not create a named `interface` or `type` for trivial, one-off data shapes — especially when the shape is only used in a single place and carries no semantic meaning beyond what the property names already convey.

If a value is simple enough to be expressed as an inline type annotation, leave it inline. Reserve named types for shapes that have real domain meaning, are reused in multiple places, or need documentation.

```typescript
// ✅ Inline — no ceremony needed for a simple local config
const NAVIGATION_ITEMS: { label: string; href: string }[] = [
  { label: 'Dashboard', href: '/dashboard' },
  { label: 'Settings', href: '/settings' },
];

// ✅ Inline — trivial return shape used once
const getPageMeta = (): { title: string; description: string } => ({
  title: 'Courses',
  description: 'Browse all courses',
});

// ❌ Named type for a trivial shape with no domain meaning
interface NavItem {
  label: string;
  href: string;
}
const NAVIGATION_ITEMS: NavItem[] = [...];

// ❌ Named type used in exactly one place
type PageMeta = { title: string; description: string };
const getPageMeta = (): PageMeta => ({ ... });

// ✅ Named interface — reused, has domain meaning
interface CoursePageData {
  course: CourseWithEnhancementStatus;
  permissions: CoursePermissions;
  isLocked: boolean;
  isDraft: boolean;
}
```

**The test:** before extracting a named type, ask "does this shape have a name that means something in the domain, or will I use it in more than one place?" If the answer is no to both, use an inline type.

### `React.FC<T>` — banned

Never use `React.FC<T>` or `React.FunctionComponent<T>`. It is a React 17-era pattern that implicitly includes `children` in props and obscures return types. Type props explicitly with `interface` and let TypeScript infer the JSX return.

```typescript
// ✅
interface StatusBadgeProps { ... }
const StatusBadge = ({ status, className }: StatusBadgeProps) => { ... };

// ❌
const StatusBadge: React.FC<StatusBadgeProps> = ({ status, className }) => { ... };
```

### Explicit Return Types

Required on all exported functions and server actions. Optional (inferred) on internal helpers and inline event handlers.

```typescript
// ✅ Explicit — exported server action
export const getCoursePageData = async (
  courseId: number,
  sessionClaims: CustomJwtSessionClaims
): Promise<CoursePageData | null> => {
  // ...
};

// ✅ Explicit — exported DB query
export const getUsersData = async ({
  page,
  pageSize
}: GetUsersDataParams): Promise<Result<{ users: UserWithStats[]; totalCount: number }>> => {
  // ...
};

// ✅ Inferred — internal handler
const highlight = (text: string, query: string) => {
  return text.replace(query, `<strong>${query}</strong>`);
};
```

### `null` vs `undefined`

Use `null` for intentional absence, especially DB returns. Use `undefined` for optional parameters (TypeScript's default for `?`).

```typescript
// ✅ DB query returns null when record not found
export const getCoursePageData = async (...): Promise<CoursePageData | null> => {
  if (!course || !canView) return null;
};

// ✅ Optional param — undefined is implicit
const getInitialData = (claims: Claims, searchParams?: SearchParams) => { ... };
```

Always use optional chaining and nullish coalescing over manual null checks:

```typescript
// ✅
const initialData = result?.data ?? null;
const showToast = result?.showToast ?? false;

// ❌
const initialData = result && result.data ? result.data : null;
```

---

## Functions & Components

### Arrow Functions — Always

All functions use arrow syntax. Never use the `function` keyword, including for React components, server actions, DB queries, and utilities.

```typescript
// ✅ Arrow — component
const StatusBadge = ({ status, className }: StatusBadgeProps) => {
  return <Badge className={cn(statusBadgeVariants({ status }), className)}>{status}</Badge>;
};

// ✅ Arrow — server action
export const getCoursePageData = async (
  courseId: number,
  sessionClaims: CustomJwtSessionClaims
): Promise<CoursePageData | null> => {
  // ...
};

// ✅ Arrow — DB query
export const getRecentChatsForUser = async ({
  userId,
  institutionId
}: GetChatsParams): Promise<Chat[]> => {
  // ...
};

// ❌ function keyword
export async function getCoursePageData(...) { ... }
export function StatusBadge(...) { ... }
```

**Exception:** Next.js file-convention exports (`page.tsx`, `layout.tsx`, `error.tsx`) use `export default function` because Next.js expects default exports for these and the `function` keyword is idiomatic there. This is the only exception.

```typescript
// ✅ Exception — Next.js file conventions only
export default function CoursesPage({ searchParams }: PageProps) { ... }
export default function RootLayout({ children }: { children: React.ReactNode }) { ... }
```

### Component Exports

A file containing a single component uses a default export. Files that export multiple utilities or components use named exports.

```typescript
// ✅ Single component file — default export
// src/components/adoptions/status-badge.tsx
const StatusBadge = ({ status, className }: StatusBadgeProps) => { ... };
export default StatusBadge;

// ✅ Multiple exports from one file — named exports
// src/components/data-tables/all-books/columns.tsx
export const nameColumn = ...;
export const statusColumn = ...;
export const actionsColumn = ...;
```

### Props Definition

For **short props lists (<4 props)**, destructure directly in the parameter signature:

```typescript
// ✅ Short — destructure in signature
const AdminGroup = ({ currentPath }: AdminGroupProps) => { ... };
const LoadingSpinner = ({ size = 'md', className }: LoadingSpinnerProps) => { ... };
```

For **long props lists (4+ props)**, receive as a named parameter and destructure in the first line of the body. This keeps the function signature scannable:

```typescript
// ✅ Long — destructure in body
const AddReadingListForm = (props: AddReadingListFormProps) => {
  const { organizationalUnits, currentUserId, courseId, onSuccess, initialValues } = props;
  // ...
};
```

**Props type naming:** Always `interface`, always suffixed with `Props`, always named after the component.

```typescript
// ✅
interface StatusBadgeProps { ... }
const StatusBadge = ({ status }: StatusBadgeProps) => { ... };

// ❌
type Props = { ... };
interface BadgeProps { ... }  // too generic
```

### Default Parameter Values

Always set defaults in the destructure in the function signature (parameter list), not inside the body.

```typescript
// ✅
const ReadingListBarChart = ({
  chartData,
  loading = false
}: ReadingListBarChartProps) => { ... };

// ❌
const ReadingListBarChart = ({ chartData, loading }: ReadingListBarChartProps) => {
  const isLoading = loading ?? false;
};
```

---

## Error Handling

### Never Throw — Always `tryCatch`

Never throw errors from any exported function (server actions, DB queries, utilities). Every operation that can fail must be wrapped in the `tryCatch` utility, which returns a `Result<T>`.

This applies at every layer — not just server actions. DB query functions called by server actions must also be wrapped.

```typescript
// ✅ Server action — wrapped in tryCatch
export const extractAndStoreCourseData = async ({
  base64Content,
  fileName,
  fileSize
}: ExtractCourseDataParams) => {
  return tryCatch(async () => {
    const { userId } = await getCurrentUser();
    const { data, error } = await extractCourseData({ buffer, fileName, fileSize, userId });
    if (!data) throw new Error(error?.message ?? 'Extraction failed');
    const key = `course:parsed:${userId}:${crypto.randomUUID()}`;
    await setCachedValue(key, data, { ttl: 1800 });
    return { importKey: key };
  });
};

// ✅ DB query function — also wrapped
export const getUsersInInstitution = async ({
  institutionId,
  page
}: GetUsersParams) => {
  return tryCatch(async () => {
    return db.query.users.findMany({
      where: (users, { eq }) => eq(users.institutionId, institutionId),
    });
  });
};

// ❌ Never throw from an exported function
export const getResource = async (id: number) => {
  const resource = await db.query.learningResources.findFirst(...);
  if (!resource) throw new Error('Resource not found'); // ❌
};
```

### `Result<T>` — Consuming at the Call Site

All callers of `tryCatch`-wrapped functions must handle both branches before accessing data.

```typescript
// ✅
const result = await getUsersData({ page, pageSize });
if (!result.data) {
  toast.error(result.error);
  return;
}
// result.data is now typed and safe
doSomethingWith(result.data.users);
```

### Client Component Error Handling

The pattern depends on how the server action is invoked.

**Calling server actions directly** — no `try/catch` needed on the client. The server action is already wrapped in `tryCatch` on the server side, so it returns a `Result<T>`. The client just handles the two branches:

```typescript
// ✅ Server action returns Result<T> — handle branches, no try/catch
const handleSubmit = async () => {
  const result = await createNote({ title, content });
  if (!result.data) {
    toast.error(result.error);
    return;
  }
  toast.success('Note created');
};
```

**React-query mutations** — handle errors via the `onError` callback, not `try/catch` inside `mutationFn`:

```typescript
// ✅ React-query — errors go in onError, not try/catch in mutationFn
const mutation = useMutation({
  mutationFn: (data: NoteInput) => createNote(data),
  onSuccess: () => toast.success('Note created'),
  onError: (error) => {
    toast.error('Failed to create note');
    captureException(error, { tags: { component: 'NoteForm' } });
  },
});

// ❌ Don't wrap mutationFn in try/catch — react-query handles the error flow
const mutation = useMutation({
  mutationFn: async (data: NoteInput) => {
    try {
      return await createNote(data);
    } catch (error) {
      toast.error('Failed');
    }
  },
});
```

---

## File Organization

### File Naming

kebab-case everywhere. Name files for what they do, not the library they use.

```
// ✅
src/actions/simsearch.ts        // function name
src/hooks/use-outside.ts
src/components/status-badge.tsx

// ❌
src/actions/qdrant.ts           // provider name
src/hooks/useOutside.ts         // camelCase
src/components/StatusBadge.tsx  // PascalCase
```

### Where Things Live

| What | Where |
|---|---|
| Reusable DB query | `src/db/queries/[domain].ts` |
| Route-specific server action | `src/app/.../[route]/_actions.ts` |
| Reusable server action (non-DB) | `src/actions/[feature].ts` |
| React component | `src/components/[domain]/` |
| Custom hook | `src/hooks/use-[name].ts` |
| Shared constants | `src/lib/constants.ts` |
| Importable domain types | `src/lib/types/[domain].ts` |
| Zod schemas | co-located `[name].types.ts`; AI schemas in `src/ai/schemas.ts` |

### Types Location

Three distinct places for types — know which to use:

- **`src/lib/types/[domain].ts`** — shared, importable domain types (interfaces and type aliases consumed across the codebase). Import from here freely.
- **`types/` directory** — ambient global declarations only (Clerk session claims augmentations, i18n message types, module augmentations). Never import directly from this directory; TypeScript picks them up automatically.
- **`'use server'` files** — must only export `async` server actions. Never export `interface` or `type` from a `'use server'` file; consumers importing that type will inadvertently pull the module into a server-only boundary. Move shared types to `src/lib/types/[domain].ts`.

```typescript
// ❌ Exporting a type from a 'use server' file
'use server';
export interface CreateBookParams { ... } // ❌ — move this out
export const createBook = async (params: CreateBookParams) => { ... };

// ✅ Type lives in src/lib/types/books.ts
// src/lib/types/books.ts
export interface CreateBookParams { ... }

// src/app/books/_actions.ts
'use server';
import type { CreateBookParams } from '@/lib/types/books';
export const createBook = async (params: CreateBookParams) => { ... };
```

### Barrel `index.ts` Files

**Default: no barrels.** Use path aliases (`@/components/status-badge`) to import directly from the source file.

Only introduce a barrel `index.ts` when both conditions are true:
1. The import ergonomics are genuinely painful (consumers are forced to know about internal file structure they shouldn't care about)
2. The module is provably isomorphic (safe to run in both server and client environments)

```typescript
// ✅ Default — import directly using alias
import StatusBadge from '@/components/adoptions/status-badge';
import { nameColumn } from '@/components/data-tables/all-books/columns';

// ✅ Barrel only when justified — e.g. a data-table module hiding deep internals
// src/components/data-tables/institutional-adoptions/index.ts
export { InstitutionalAdoptionsTable } from './data-table';
export type { AdoptionRow } from './types';
```

### `page-content.tsx` Pattern

`page-content.tsx` is a client page shell. Use this pattern when a page is conceptually a client-side experience (it has interactivity, local state, user-facing controls) but benefits from pre-fetched server data that doesn't need to be re-fetched on the client.

The split:
- `page.tsx` — server component, fetches initial data, handles auth, passes data as props
- `page-content.tsx` — `'use client'`, receives the pre-fetched data as props, owns all interactivity

```
// ✅ When to split: client-driven page that needs a one-time server data load
page.tsx
  └─ async, fetches courses + permissions
  └─ passes to <PageContent courses={courses} permissions={permissions} />

page-content.tsx
  └─ 'use client'
  └─ receives props, manages filters/sorting/modals
  └─ may use TanStack mutations
```

If a page is purely server-rendered with no significant client interactivity, keep it as a single `page.tsx` — no split needed.

---

## Database

### Query Style

Prefer `db.query` (Drizzle's relational API) with the callback pattern — it's readable, type-safe, and handles relations cleanly. Fall back to the query builder (`db.select()`, `db.insert()`, etc.) only when the relational API can't express the query, such as inner joins, complex aggregations, or subqueries. No raw SQL except in custom migrations.

**Decision rule:** Can `db.query` express this query? → use it. Does it require an inner join, aggregation, or subquery? → use `db.select()` with the query builder.

```typescript
// ✅ Prefer db.query for straightforward relational reads
const source = await db.query.learningResources.findFirst({
  where: (learningResources, { eq }) => eq(learningResources.id, resourceId),
  with: {
    children: {
      orderBy: (children, { asc, desc }) => [
        desc(children.currentVersion),
        asc(children.orderIndex)
      ]
    }
  }
});

// ✅ Use db.select() when db.query can't express it (e.g. inner join)
const results = await db
  .select({ id: courses.id, title: courses.title, userName: users.name })
  .from(courses)
  .innerJoin(users, eq(courses.ownerId, users.id))
  .where(eq(courses.institutionId, institutionId));

// ❌ Don't use db.select() for simple queries that db.query handles fine
const resource = await db.select().from(learningResources).where(eq(learningResources.id, id));
```

### Dynamic Conditions

When where conditions are dynamic (e.g. an optional filter), build a `conditions` array before the query rather than embedding inline ternaries inside the callback. This keeps queries readable and easy to extend.

```typescript
// ✅ Build conditions array first
const conditions = [eq(books.parentId, parentId)];
if (!override) {
  conditions.push(isNull(books.deletedAt));
}
const books = await db.query.books.findMany({
  where: and(...conditions),
});

// ❌ Inline ternary inside callback — hard to read and extend
const books = await db.query.books.findMany({
  where: (books, { eq, isNull, and }) =>
    and(eq(books.parentId, parentId), !override ? isNull(books.deletedAt) : undefined),
});
```

### Multiple Operations

**`db.batch()`** applies to raw Drizzle query objects (e.g. `db.query.*.findFirst(...)`, `db.update(...).set(...)`). These are sent together in a single round-trip.

**`Promise.all`** applies to independent async function calls (e.g. calling `insertClosedBook(item)`, `getAllOrganizationalUnits()`). These are not Drizzle queries — do not use `db.batch()` here.

```typescript
// ✅ db.batch() — raw Drizzle queries, single round-trip
const [course, topic] = await db.batch([
  db.query.courses.findFirst({ where: (c, { eq }) => eq(c.id, id) }),
  db.query.topics.findFirst({ where: (t, { eq }) => eq(t.courseId, id) })
]);

// ❌ Promise.all for raw Drizzle queries — use db.batch() instead
const [course, topic] = await Promise.all([
  db.query.courses.findFirst(...),
  db.query.topics.findFirst(...)
]);

// ✅ Promise.all — independent async function calls (not raw Drizzle queries)
const results = await Promise.all(items.map((item) => insertClosedBook(item)));

// ❌ Sequential for loop over independent async calls — slow, use Promise.all
const results = [];
for (const item of items) {
  results.push(await insertClosedBook(item));
}
```

### Multi-tenant Scoping

Every query touching user or institution data must include `institutionId`, unless the `isAdmin` flag has been checked via the `isSuperAdmin` utility (admin users can operate across institutions).

```typescript
// ✅
const courses = await db.query.courses.findMany({
  where: (courses, { eq }) => eq(courses.institutionId, institutionId)
});

// ✅ Admin bypass — isSuperAdmin confirmed before the query
const isAdmin = await isSuperAdmin(userId);
if (isAdmin) {
  return db.query.courses.findMany(); // cross-institution read is intentional
}

// ❌ missing institutionId (and no isSuperAdmin check)
const courses = await db.query.courses.findMany();
```

---

## Naming

### General Conventions

| Thing | Convention | Example |
|---|---|---|
| Files & folders | kebab-case | `reading-list-form.tsx` |
| Variables | camelCase | `isUserAuthenticated`, `totalRevenue` |
| Functions | camelCase, verb-first | `getUserById`, `computeEnhancementStatus` |
| React components | PascalCase | `StatusBadge`, `CourseCard` |
| Interfaces & types | PascalCase | `CoursePageData`, `ReadingListFormProps` |
| Hooks | camelCase, `use` prefix | `useCourseCollaborators`, `useOutside` |
| Boolean variables | `is`, `has`, `can` prefix | `isLoading`, `hasEmbeddings`, `canView` |

### Constants

All constants are `SCREAMING_SNAKE_CASE` — whether global or local, at module level or inside a function body.

```typescript
// ✅ Global — src/lib/constants.ts
export const FILTERS = {
  MIN_SCORE_THRESHOLD: 0.75,
  SCORE_THRESHOLD: 0.25,
} as const;

export const WORDS_PER_PAGE = 250 as const;

// ✅ Local — module-level constant in a component file
const NAVIGATION_ITEMS = [
  { label: 'Email', value: 'email', href: '/institution/settings/email' },
  { label: 'Integrations', value: 'integrations', href: '/institution/settings/integrations' },
] as const;

// ✅ Local — inside a function body
const handleUpload = async () => {
  const MAX_FILE_SIZE = 10 * 1024 * 1024;
  const ALLOWED_TYPES = ['application/pdf'] as const;
  // ...
};

// ❌ camelCase for constants
const navigationItems = [...];
const dropzoneConfig = { ... };
```

---

## React & Next.js Patterns

### Server vs Client Components

No `'use client'` = Server Component. Add the directive only when you need React hooks, browser APIs, or event listeners.

```typescript
// ✅ Server component — async, no directive
export const SimilarTopicsCard = async ({ topicId }: SimilarTopicsCardProps) => {
  const [t, topics] = await Promise.all([
    getTranslations('SimilarTopics'),
    getSimilarTopics(topicId),
  ]);
  return <div>...</div>;
};

// ✅ Client component — explicit directive
'use client';
const SearchInput = ({ value, onChange }: SearchInputProps) => {
  const [searchValue, setSearchValue] = useState(value);
  // ...
};
```

### Async Patterns

`async/await` always. Parallel execution with `db.batch()` for DB, `Promise.all` for non-DB.

```typescript
// ✅ Parallel — non-DB server fetches
const [orgUnits, initialData] = await Promise.all([
  getAllOrganizationalUnits(),
  getInitialData(sessionClaims, searchParams)
]);

// ❌ Sequential when parallel is possible
const orgUnits = await getAllOrganizationalUnits();
const initialData = await getInitialData(sessionClaims, searchParams);
```

### Loading & Error Boundaries

Every route with a top-level `await` in `page.tsx` must have a sibling `loading.tsx`. Route groups with user-triggered mutations should have an `error.tsx`.

```
src/app/courses/
├── page.tsx          ← has top-level await
├── loading.tsx       ← required
└── error.tsx         ← required (mutations possible)
```

### Early Returns

Prefer early returns over nested conditionals.

```typescript
// ✅
if (!user) return null;
if (!user.isAdmin) return <Unauthorized />;
if (!course) return <NotFound />;
return <CourseView course={course} />;

// ❌ Nested ternary hell
return user
  ? user.isAdmin
    ? course ? <CourseView course={course} /> : <NotFound />
    : <Unauthorized />
  : null;
```

### State Updates

Use the functional updater form whenever the next state depends on the previous state.

```typescript
// ✅
setCount(prev => prev + 1);
setItems(prev => [...prev, newItem]);

// ❌ — stale closure risk in async scenarios
setCount(count + 1);
```

---

## Code Smell Checklist

Review code for these before opening a PR:

- [ ] Function longer than ~40 lines — can it be split?
- [ ] More than 3 levels of nesting — use early returns
- [ ] Magic number or string inline — extract to a named `SCREAMING_SNAKE_CASE` constant
- [ ] `as any` anywhere — find the correct type
- [ ] `type` used for a named object shape — change to `interface`
- [ ] Named `interface`/`type` for a trivial shape used in one place — inline it
- [ ] `React.FC<T>` — remove, type props with `interface` directly
- [ ] `function` keyword (non–file-convention) — convert to arrow
- [ ] Named export for a single-component file — convert to default export
- [ ] `Promise.all` for DB queries — replace with `db.batch()`
- [ ] `throw` inside an exported function — wrap the whole body in `tryCatch`
- [ ] Missing `institutionId` in a DB query (and no `isSuperAdmin` check) — multi-tenant violation
- [ ] `middleware.ts` — must be `proxy.ts`
- [ ] Wrapper function that only calls through to another — delete it
- [ ] Barrel `index.ts` without clear justification — remove and use direct imports
