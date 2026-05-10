# Behavioral Specification Derivation

**Purpose**: Derive a project's behavioral specification from existing code, tests, and docs. Produces five artifacts (`existing-stories.md`, `existing-personas.md`, `existing-screens-map.md`, `existing-flows.md`, `existing-domain-baseline.md`) plus a consolidated `REVIEW.md` work queue and (if multi-package) a `packages-manifest.md`.

**Execute when**: Brownfield project detected (`**Project Type**: Brownfield` in `aidlc-docs/aidlc-state.md`) AND `reverse-engineering` is complete. Runs unconditionally on brownfield — applies to backend-only projects too (stories and domain baseline are useful even without UI).

**Skip when**: Greenfield project (greenfield projects produce stories/personas/etc. via `requirements-analysis` → `user-stories` → `application-design`, not via derivation).

**Rerun behavior**: Same staleness rule as `reverse-engineering` (controlled by `workspace-detection`). Auto-trigger via staleness re-derives **silently** — preserves zero-friction adoption. Explicit user-invoked re-runs prompt for confirmation:

> ⚠️ Re-running behavioral derivation will regenerate all artifacts.
> N items in REVIEW.md will be re-derived. Manual edits to artifact bodies will be lost.
> Confirm: [yes/no]

## Prerequisites

- `workspace-detection` complete; `aidlc-state.md` exists and contains `**Project Type**: Brownfield`
- `reverse-engineering` complete; outputs available under `aidlc-docs/inception/reverse-engineering/`
- `aidlc-docs/inception/behavioral-derivation/` does not exist OR is stale per the rerun rule

## Output directory

`aidlc-docs/inception/behavioral-derivation/`. Layout depends on the package count detected in Step 1.

## Step 1: Detect package layout

Read `aidlc-docs/inception/reverse-engineering/architecture.md` (and any package-level summaries A produced). Count packages with **distinct behavioral surface** — defined as:

- Independent routing tree (separate `*Router*`, `pages/`, `app/` for each), OR
- Independent user-facing flows (separate e2e suites per package), OR
- Independent domain entity set (separate `src/types/`, schemas, models)

A package counts toward the multi-package layout even if it is backend-only when its API contract is distinct (separate OpenAPI/GraphQL surface, separate handler tree).

**Single-package layout** applies when:
- A reports 0 or 1 packages, OR
- A reports multiple packages that share a single behavioral surface (e.g., a frontend + a thin BFF wrapping the same domain).

In single-package layout, write artifacts directly to `aidlc-docs/inception/behavioral-derivation/`.

**Multi-package layout** applies when ≥2 packages have distinct behavioral surfaces. In this case:
- Create `aidlc-docs/inception/behavioral-derivation/packages-manifest.md` listing the discovered packages
- Write per-package artifacts to `aidlc-docs/inception/behavioral-derivation/packages/<pkg-name>/`
- `REVIEW.md` stays at the root (`aidlc-docs/inception/behavioral-derivation/REVIEW.md`), consolidated cross-package

**Edge case**: 2+ frontend packages backed by a shared monorepo backend → multi-package on the frontend dimension. Each frontend gets its own per-package directory; the shared backend's `existing-domain-baseline.md` is duplicated per frontend (acceptable redundancy: each frontend's domain consumption may differ).

## Step 2: Source priority and confidence rule

For each artifact, scan sources in priority order. Items derived from a `high` source are accepted without flagging; items derived from `low` sources are also produced but flagged in `REVIEW.md`.

| Artifact | `high` source (actively executing) | `low` source (inferred from patterns) |
|---|---|---|
| `existing-screens-map.md` | Routing files: React Router config, Next.js `pages/` or `app/`, Remix routes, TanStack Router route trees | Component naming patterns, README routing references |
| `existing-flows.md` | Playwright/Cypress e2e specs that run in the test suite (not `test.skip`) | Routing chains, in-app navigation links in code |
| `existing-stories.md` | e2e `test()` / `it()` names + `describe()` blocks (running in the test suite); Storybook stories matching `.storybook/main.ts` glob whose component import resolves | Component names, route names, prop documentation; Storybook stories whose component is missing or excluded by config |
| `existing-personas.md` | Auth guards / middleware with role checks invoked at runtime | Naming patterns (`AdminPanel`, `UserDashboard`), README mentions |
| `existing-domain-baseline.md` | TypeScript interfaces / Zod schemas / Prisma models actively imported; Storybook `args` / `argTypes` when they reference imported types | API response shapes inferred from fetch calls, prop drilling |

### Strict rule for `high` classification

The source must be **actively executing in the project's normal run**:

- Tests included in the test runner config (jest, vitest, playwright, cypress) and not marked `.skip`
- Routes mounted in the app's router (commented-out routes → `low`)
- Types imported by code that compiles (unimported / dead types → `low`)
- Storybook stories matching the glob in `.storybook/main.ts` AND whose component import resolves to an existing module (stories of deleted/renamed components → `low`)

CI presence is not required; the criterion is "this would run if you ran the project today".

### Skip-marker patterns to detect

Treat as `low` (skipped/dead): `test.skip(...)`, `it.skip(...)`, `describe.skip(...)`, `xtest(...)`, `xit(...)`, `xdescribe(...)`. For environment-gated tests (e.g., `if (process.env.CI) test(...)`), default to `low` unless the gate is unconditionally true.

### Storybook detection

Read `.storybook/main.ts` (or `.storybook/main.js`) to extract the `stories` glob. Enumerate files matching the glob. For each story file, parse exported story objects (CSF v3 default; auto-fallback to CSF v2 if detected via `export default { title: ... }` shape). Stories that fail to compile (broken TypeScript, missing args) are **skipped silently** — do not emit a `low` entry; do not pollute `REVIEW.md` with build errors.

### Storybook scope

Storybook contributes ONLY to `existing-stories.md` (each story export = one component-level state) and `existing-domain-baseline.md` (each `args` / `argTypes` referencing imported types = a domain entity shape signal). Storybook does NOT contribute to `existing-personas.md`, `existing-screens-map.md`, or `existing-flows.md` — those concerns are routing/auth/e2e territory.

### Fallback rules

- **Missing `high` source for an artifact**: produce the artifact with banner:
  > ⚠️ No actively executing source found. All items derived from code inference — full human review required.
- **Missing all sources**: produce the artifact as a stub:
  > ⚠️ No derivation sources found. This artifact is a stub for manual entry.

## Step 3: Generate `existing-screens-map.md`

For each package (or the single root in single-package layout), produce a screens map.

### Inputs

Routing files matching one of:
- React Router: files importing `createBrowserRouter`, `RouterProvider`, or `<Routes>` JSX
- Next.js: filesystem-based routing under `pages/` or `app/`
- Remix: `routes/` directory under `app/`
- TanStack Router: `routes/` tree, files with `createRoute` / `createFileRoute`

### Heuristics

For each route node:
- **Route**: the path string
- **Screen Name**: derive from the component name; remove suffixes like "Page", "Screen", "View"
- **Auth Required**: yes if the route is wrapped in a guard component (`<RequireAuth>`, `<ProtectedRoute>`, etc.) OR if it's inside a layout that includes such a guard
- **Linked From**: scan code for `<Link to="X">`, `navigate("X")`, `redirect("X")` referring to this route
- **Confidence**: `high` if the route is mounted in an actively-running router; `low` if inferred (commented out, in a non-imported file, or named like a screen but not in any router)

### Output schema

```markdown
# Existing Screens Map
> Derived from: <list of routing files used>
> Confidence: <high|partial|stub>
> Pending review: <N> items — see REVIEW.md

| Route | Screen Name | Auth Required | Linked From | Confidence | Source |
|---|---|---|---|---|---|
| /dashboard | Dashboard | yes (AdminGuard) | /login, /home | high | src/router/index.tsx:45 |
```

### Per-row low-confidence reasons

When a row is `low`, also emit a `REVIEW.md` entry with the `Reason` column populated. Common reasons: "Role inferred from naming, no explicit guard found", "Route in non-imported file", "Auth guard component named ambiguously".

### When to skip

If no routing files are detected, produce the artifact with the "No actively executing source found" banner and emit only `low` entries inferred from component naming. If neither routing nor named screen components exist, produce the stub artifact.

## Step 4: Generate `existing-flows.md`

> Execution scope: per-package in multi-package layout, single root in single-package layout (per Step 1).

### Inputs

E2E spec files matching one of:
- Playwright: files importing `@playwright/test`, typically under `e2e/`, `tests/e2e/`, `tests/`
- Cypress: files under `cypress/e2e/` or `cypress/integration/`

### Heuristics

For each `test()` / `it()` block:
- **Flow name**: the test title, normalized (e.g., "user can log in" → "User Login")
- **Source**: file path + line number of the `test(` opening
- **Confidence**: `high` if the test is in the runner config and not skipped; `low` if `test.skip` / `xtest` / env-gated
- **Steps**: extract page interactions in order — `page.goto`, `page.fill`, `page.click`, `expect(...)` assertions. Render as numbered list.

### Output schema

```markdown
# Existing User Flows
> Derived from: <list of e2e spec files>
> Confidence: <high|partial|stub>
> Pending review: <N> items — see REVIEW.md

## Flow: User Login
- **Source**: `e2e/auth.spec.ts:14` — `test('user can log in')`
- **Confidence**: high
- **Steps**:
  1. Navigate to /login
  2. Enter credentials in email/password fields
  3. Click submit button
  4. Verify redirect to /dashboard

## Flow: Checkout (skipped)
- **Source**: `e2e/checkout.spec.ts:34` — `test.skip('user can complete checkout')`
- **Confidence**: low
- **Note**: Test is skipped — flow may be incomplete or broken
```

### Per-row low-confidence reasons

When a flow is `low`, emit a `REVIEW.md` entry. Common reasons: "Test marked `.skip`", "Test gated on environment variable", "Steps inferred from routing chain — no e2e coverage".

### When to skip

If no e2e specs are detected, produce the artifact with the "No actively executing source found" banner and emit only `low` entries inferred from routing chains and in-app navigation links. If neither e2e nor navigation patterns exist, produce the stub artifact.

## Step 5: Generate `existing-stories.md`

> Execution scope: per-package in multi-package layout, single root in single-package layout (per Step 1).

This artifact captures behavioral facts about the system. Two `high` sources contribute at different granularities:

- **e2e specs** capture user flows ("user with no data sees empty dashboard")
- **Storybook stories** capture component-level states ("Dashboard / Empty State")

Both flow into the artifact as complementary entries — not duplicates. Each entry cites its source.

### Inputs

- E2E spec files (same set used in Step 4)
- Storybook stories: files matching the glob in `.storybook/main.ts` (or `.storybook/main.js`)

### Heuristics for e2e-derived stories

For each `test()` block:
- Extract Given/When/Then from the test body:
  - **Given** clause: from `test.beforeEach` setup, fixtures, or page state at start
  - **When** clause: the principal user action (`page.click`, `page.fill` followed by `submit`, etc.)
  - **Then** clause: the assertion (`expect(...).toBe(...)`, `expect(page).toHaveURL(...)`)
- **Confidence**: `high` for active tests; `low` for skipped
- **Source**: `e2e/<file>:<line>`

### Heuristics for Storybook-derived stories

For each story export (one entry per `export const Variant: Story = {...}`):
- **Story title**: `<Component> / <Variant>` (e.g., "Dashboard / Empty State")
- **Behavioral fact**: derive Given/When/Then from the story name and args:
  - **Given**: the component's setup (props from `args`)
  - **When**: the component mounts
  - **Then**: the variant renders (e.g., "the empty-state placeholder is rendered with 'No data yet' copy")
- **Confidence**: `high` if story file matches glob AND component import resolves; `low` if component is missing/renamed
- **Source**: `<story-file>:<line>` — Storybook story `<Variant name>`

### Output schema

```markdown
# Existing Behavioral Stories
> Derived from: <list of e2e files + storybook files>
> Confidence: <high|partial|stub>
> Pending review: <N> items — see REVIEW.md

## Story: User can view dashboard
- **Given** the user is authenticated
- **When** they navigate to /dashboard
- **Then** the dashboard screen is displayed with their data
- **Confidence**: high
- **Source**: `e2e/dashboard.spec.ts:14` (e2e)

## Story: Dashboard renders empty state when there is no data
- **Given** the user is authenticated and has no records
- **When** the dashboard mounts
- **Then** the empty-state placeholder is rendered with "No data yet" copy
- **Confidence**: high
- **Source**: `src/components/Dashboard.stories.tsx:42` — Storybook story `Empty State`
```

### Per-row low-confidence reasons

When a story is `low`, emit a `REVIEW.md` entry. Common reasons: "Test marked `.skip`", "Storybook story references missing/renamed component", "Story inferred from component + route names — no e2e or Storybook coverage", "Storybook stories file outside `.storybook/main.ts` glob".

### Overlap handling

When a Storybook story and an e2e test describe the same screen+state combo (e.g., Storybook "Dashboard / Empty State" and e2e "user with no data sees empty dashboard"):
- Keep BOTH entries with their distinct source citations
- Add an entry to `REVIEW.md` flagging the overlap with reason: "Same screen+state combo as <other entry>; manual review needed to dedup or merge"

Do not attempt automatic dedup — similarity detection is fragile and out of scope.

### When to skip

If neither e2e nor Storybook sources are detected, produce the artifact with the "No actively executing source found" banner and emit only `low` entries inferred from component + route names. If no naming signal exists either, produce the stub artifact.

## Step 6: Generate `existing-personas.md`

> Execution scope: per-package in multi-package layout, single root in single-package layout (per Step 1).

### Inputs

- Auth guards / middleware: files containing role-check patterns (`role === 'admin'`, `requireRole('admin')`, `<RequireRole role="admin">`)
- Route guards detected in Step 3
- Naming patterns (component names containing role hints): `AdminPanel`, `UserDashboard`, `GuestView`

### Heuristics

For each distinct role identifier found:
- **Persona name**: derive from the role identifier (`'admin'` → "Admin", `'user'` → "Regular User")
- **Role identifier**: the literal string used in the code
- **Access level**: enumerate routes/components gated by this role (use Step 3 output for cross-reference)
- **Confidence**: `high` if the role is invoked in middleware/guards at runtime; `low` if inferred from naming only
- **Source**: `<file>:<line>` of the principal guard/middleware definition

### Output schema

```markdown
# Existing Personas
> Derived from: <list of auth/middleware files>
> Confidence: <high|partial|stub>
> Pending review: <N> items — see REVIEW.md

| Persona | Role Identifier | Access Level | Confidence | Source |
|---|---|---|---|---|
| Admin | `role: 'admin'` | Full access — all routes + user management | high | src/middleware/auth.ts:42 |
| Regular User | `role: 'user'` | Standard routes — no admin paths | high | src/middleware/auth.ts:42 |
| Guest | (no role) | Read-only — public marketing routes | low | src/router/index.tsx:88 |
```

### Per-row low-confidence reasons

Common reasons: "No explicit role found; inferred from unauthenticated routes", "Role mentioned in naming but no guard implementation", "Access level inferred from route names only".

### When to skip

If no auth guards or middleware exist, produce a `low` entry "Anonymous User" inferred from public routes, with the missing-source banner. If no public routes exist either, produce the stub artifact.

## Step 7: Generate `existing-domain-baseline.md`

> Execution scope: per-package in multi-package layout, single root in single-package layout (per Step 1).

### Inputs

- TypeScript interfaces and types: `interface X {}`, `type X = {...}`
- Zod schemas: `z.object({...})`
- Prisma models: `model X {...}` in `schema.prisma`
- Storybook `args` / `argTypes` referencing imported types

### Heuristics

For each entity definition:
- **Entity name**: the interface/type/model name
- **Source**: file path + line; note if "active" (imported elsewhere) or "dead" (no importers)
- **Confidence**: `high` if actively imported by compiling code; `low` if unimported or imported only by skipped/dead modules
- **Shape**: enumerate fields with types

For Storybook contributions:
- When `args` or `argTypes` reference an imported domain type, treat the type usage as **corroborating evidence** that the entity is real and used. Add a note "Also referenced in Storybook story `<file>`" to the entity entry. Do not create separate Storybook-only entities.

### Output schema

```markdown
# Existing Domain Baseline
> Derived from: TypeScript interfaces in src/types/, Zod schemas in src/schemas/, Storybook args/argTypes referencing imported types
> Confidence: <high|partial|stub>
> Pending review: <N> items — see REVIEW.md

## Entity: User
- **Source**: `src/types/user.ts` (active — imported in 12 files)
- **Confidence**: high
- **Shape**:
  - `id: string`
  - `email: string`
  - `role: 'admin' | 'user'`
  - `createdAt: Date`
- **Also referenced in**: `src/components/UserCard.stories.tsx` (Storybook args)

## Entity: Order
- **Source**: `src/schemas/order.ts` (Zod schema, imported in 5 files)
- **Confidence**: high
- **Shape**:
  - `id: string`
  - `userId: string`
  - `items: OrderItem[]`
  - `status: 'pending' | 'paid' | 'shipped'`
```

### Per-row low-confidence reasons

Common reasons: "Type defined but no importers found", "Schema imported only by skipped tests", "Shape inferred from API response — no compiled definition".

### When to skip

If no type/schema definitions exist anywhere, produce the artifact with the "No actively executing source found" banner and emit only `low` entries inferred from API call shapes and prop drilling. If no inferrable shapes exist, produce the stub artifact.

## Step 8: Generate `REVIEW.md`

After Steps 3-7 produce all five artifacts, walk every artifact and collect every per-item entry whose `Confidence` is `low`. For each, emit a row in `REVIEW.md`.

### Output location

`aidlc-docs/inception/behavioral-derivation/REVIEW.md` — always at the root, regardless of single- or multi-package layout. In multi-package, `REVIEW.md` is consolidated cross-package: rows reference the originating package via the `Artifact` column (e.g., `packages/admin-app/existing-personas.md`).

### Output schema

```markdown
# Behavioral Derivation — Pending Review

> N items require human review before these artifacts are used as spec inputs.
> Remove each row once reviewed and corrected/accepted in the source artifact.

| Artifact | Item | Reason | Source |
|---|---|---|---|
| existing-personas.md | Guest persona | No explicit role found; inferred from unauthenticated routes | src/router/index.tsx:88 |
| existing-screens-map.md | /admin/users | Role 'admin' inferred from naming, no explicit guard found | src/router/index.tsx:112 |
| existing-stories.md | Admin bulk-delete | Component exists but no e2e coverage | src/components/AdminTable.tsx:201 |
| existing-stories.md | Dashboard / Empty State (overlap) | Same screen+state combo as e2e 'user with no data sees empty dashboard'; manual review needed to dedup or merge | src/components/Dashboard.stories.tsx:42 |
```

### Empty case

If no `low` items exist, produce `REVIEW.md` with a single line:
```markdown
# Behavioral Derivation — Pending Review

> 0 items require review. Behavioral derivation is fully reviewed and ready as spec input.
```

## Step 9: Generate `packages-manifest.md` (multi-package only)

Skip this step in single-package layout.

### Output location

`aidlc-docs/inception/behavioral-derivation/packages-manifest.md`

### Output schema

```markdown
# Behavioral Derivation — Packages Manifest

> N frontend/backend packages discovered. Each has its own derived artifacts under `packages/<name>/`.

| Package | Path | Framework | Has e2e? | Derivation Status |
|---|---|---|---|---|
| admin-app | apps/admin | React 19 + Vite | yes | high (12 stories from actively executing e2e) |
| customer-app | apps/customer | Next.js 14 | no | partial (stories from routes only) |
```

The `Derivation Status` column summarizes each package's aggregate confidence (`high`, `partial`, or `stub` — same scale as artifact headers).

## Step 10: Update `aidlc-state.md`

Append or update the following section in `aidlc-docs/aidlc-state.md`:

```markdown
## Stage Progress
- **Behavioral Derivation**: complete (<ISO timestamp>)
  - Layout: <single-package|multi-package>
  - Packages: <N> (multi-package only)
  - Pending review: <N> items in REVIEW.md
```

The `behavioral-derivation: complete` marker signals downstream stages (frontend-design, change-flow, future Workflow E) that B's artifacts are present and valid for consumption.

## Completion message

After Step 10, present to the user:

```
# 🧠 Behavioral Derivation Complete

Generated artifacts in aidlc-docs/inception/behavioral-derivation/:
• <N> stories from <M> e2e specs and <P> Storybook stories
• <N> personas from auth guards
• <N> screens from <framework> routing
• <N> flows from e2e specs
• <N> domain entities from types/schemas

Pending review: <N> items in REVIEW.md
Next step: <Frontend Design (UI detected) | Requirements Analysis (no UI)>
```
