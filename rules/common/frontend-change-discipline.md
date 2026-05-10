# Frontend Change Discipline (Workflow D)

**Purpose**: Enforce spec coherence on UI changes within the change-flow protocol. When any change modifies files in `ui_bearing` domains (per `routing-table.json`), the corresponding state matrix rows + data bindings + snapshot contracts in the per-unit `frontend-components.md` (or per-screen fallback `aidlc-docs/screens/{screen-name}.md`) MUST be updated in the same commit.

**Execute when**: Any change-flow path (DIRECT, CROSS-DOMAIN, DESIGN, REVERT) touches files in domains marked `ui_bearing: true` in `.claude/routing/routing-table.json`.

**Skip when**:
- No `routing-table.json` exists OR no domain has `ui_bearing: true` (consumer hasn't declared UI surface yet).
- Modified files do NOT match any `ui_bearing: true` domain paths.
- Pure refactor declared in DESIGN path's `brainstorming-spec.md` per §"Refactor declaration override" below.

**Cross-cutting**: D is NOT a step in any single path-protocol. It is an invariant declared in IRON LAW (`rules/common/core-change-flow-protocol.md` §3.10) that applies across all paths. The rule body below describes the operational mechanics.

**Schema co-ownership**: D and `rules/construction/functional-design.md` Step 6 sub-pasos g+h+i co-own the schema of `frontend-components.md`. Schema changes in either rule require dual-update enforced via PR review. Sub-paso (i) example yaml in `functional-design.md` is the canonical source of truth for snapshot contract entries.

## Prerequisites

D's lookup logic depends on artifacts from Workflow B (brownfield) and/or Workflow C (UI inception):

- **Greenfield + UI declared**: C provides `screen-affinity-map.md` → use it for screen → unit lookup
- **Brownfield + UI detected (B + C ran)**: same as greenfield (C consumed B's outputs)
- **Brownfield without UI declared but ui-domain present**: C did not run; B's `existing-screens-map.md` provides screen list (per-screen fallback)
- **Brownfield without B nor C** (inconsistent state — UI declared but inception incomplete): D fails explicitly with guidance "run inception:behavioral-derivation first"

## Output artifacts (consumer-side, edited or created)

```
aidlc-docs/
├── construction/
│   ├── issue-{N}/
│   │   └── brainstorming-spec.md     ← DESIGN path: extended with `## UI Spec Delta`
│   └── {unit-name}/
│       └── functional-design/
│           └── frontend-components.md  ← edited in-place (sub-pasos g+h+i)
├── screens/
│   └── {screen-name}.md              ← brownfield-without-units fallback
└── spec-coverage-debt.md             ← project-level debt registry
```

## Step 1: Layer B — Text analysis hint at triage

During triage of an issue (Step 1 of any change-flow path), the agent runs heuristic analysis of issue title + body + labels searching for UI signals. This is **informational, non-blocking**.

**UI signals to detect**:
- Mention of screen / page / route / view
- Mention of UI component (button, input, modal, card, table, drawer, etc.)
- Mention of visual state (loading, empty, error, success, partial)
- Mention of copy / text / label / message / placeholder
- Mention of layout / color / spacing / responsive / dark mode
- Mention of user-facing event (click, hover, focus, scroll, keypress)
- Match of any screen name in `aidlc-docs/inception/behavioral-derivation/existing-screens-map.md` (Workflow B output) when present

If any signal detected → annotate planning to include state matrix considerations from the start.
If NO signals → standard planning, no UI overhead.

## Step 2: Layer A — Path-glob enforcement at PR

Before creating the PR (during Post-Implementation Chain of any path), execute:

```
1. List modified files in the current branch's commit set
   (e.g., `git diff main..HEAD --name-only`)

2. Load ui-domain paths from .claude/routing/routing-table.json:
   - Iterate ALL domains where `ui_bearing` field is true
   - If field absent on a domain, treat as false (backward compat)
   - If no domain has ui_bearing true → SKIP D entirely (Layer A passes silently)

3. For each modified file, check if it matches any ui-domain path glob (OR semantics).

4. If ANY modified file matches:
   for each screen touched (may be >1):
     a. Identify unit via `aidlc-docs/inception/frontend-design/screen-affinity-map.md`
        (Workflow C output)
     b. If unit identified: verify
        `aidlc-docs/construction/{unit-name}/functional-design/frontend-components.md`
        has commits in this branch covering affected rows (trinity g+h+i — see Step 3)
     c. If no unit (brownfield-without-units): fall back to
        `aidlc-docs/inception/behavioral-derivation/existing-screens-map.md` (Workflow B output)
        to identify screen-name; verify `aidlc-docs/screens/{screen-name}.md`
        has corresponding updates
     d. If neither unit nor screen-name lookup works (brownfield without B nor C):
        FAIL explicitly with: "Run inception:behavioral-derivation first to enable D"

5. If verification fails for any screen → block PR, present what's missing per screen.
```

### Skip vs Fail — when each applies

- **Skip silently** is correct when **no UI surface is declared** — consumer hasn't set `ui_bearing: true` on any domain in `routing-table.json`. Nothing to enforce.
- **Fail explicitly** is correct when **UI is declared but prerequisite spec artifacts are missing** — consumer set `ui_bearing: true` AND modified ui-domain files AND brownfield project lacks both B's `existing-screens-map.md` and C's `screen-affinity-map.md`.

The distinction matters: silent skip preserves backward compatibility for consumers not yet adopting D; explicit fail surfaces inception incompleteness so the user can fix workflow ordering.

## Step 3: Definition of "affected row" (trinity g+h+i)

A row of the state matrix is **affected** by a change if any of these conditions hold:

- Change modifies the code of the component rendering that state
- Change modifies copy / labels / messages appearing in that state
- Change adds a new state that did not exist
- Change removes a state that existed (NOT applicable to mandatory states — see below)
- Change modifies the data binding source for that state

For each affected row, the **trinity** must be updated coherently in the same commit:
- (g) state matrix row: `state | trigger | rendered components | data | copy`
- (h) data binding entry: `Component.prop → source` for that row
- (i) snapshot contract entry: yaml entry with `fixture`, `expectedTestIds`, `expectedCopy`, optionally `storyName` and `route`

The trinity is one unit, not three independent updates. A state matrix row without its data binding is declaration without source; without its snapshot contract is contract without verification.

### Affected vs unaffected — pre-existing rows

A row is **unaffected** if the change does NOT touch its component code, copy, or data binding. Unaffected rows are out of scope for the current change — D does NOT require updating them, even if they are incomplete (missing fields). See `## Spec coverage debt` below for how pre-existing gaps are tracked.

## Step 4: Mandatory states default

When `aidlc-docs/inception/frontend-design/ui-stack.md` is present (Workflow C ran), D reads the mandatory states list from there.

When absent (consumer without C), D uses the **hardcoded default**:

```
[loading, empty, error, populated, partial]
```

This matches the v1 recommendation in C's spec. Mandatory states cannot be removed by a change (`removed` action does not apply to mandatory states); they can only be modified. Custom non-mandatory states declared in `ui-stack.md` may be added or removed.

## Step 5: `## UI Spec Delta` section in `aidlc-docs/construction/issue-{N}/brainstorming-spec.md`

The DESIGN path already writes `brainstorming-spec.md` with audit table. D appends this section at the end:

```markdown
## UI Spec Delta

> Records the spec changes this issue applied. Read alongside the touched
> frontend-components.md files to understand what shifted.

### Files modified

| Path | Type | Reason |
|---|---|---|
| `aidlc-docs/construction/<unit-name>/functional-design/frontend-components.md` | per-unit | <reason> |

### Affected rows (per file)

#### `aidlc-docs/construction/<unit-name>/functional-design/frontend-components.md`

| State | Action | Note |
|---|---|---|
| <state> | <added|modified|removed> | <one-line description> |

### Refactor declaration (if applicable)

> Skip this subsection unless the change is a pure refactor with no behavior delta.

- **Pure refactor**: <yes|no>
- **Justification**: <one line>
- **Verification**: <how confirmed identical render>

### Unknown rows (if applicable)

> Skip this subsection unless the agent could not determine behavior of an affected row.
> Each unknown row is also flagged in the matrix with `unknown — needs human input`.

| Row state | Reason agent could not determine | Action requested from reviewer |
|---|---|---|
| <state> | <one-line> | <one-line> |
```

The Action column accepts ONLY `added`, `modified`, or `removed`. `unchanged` rows are NOT included.

## Step 6: `aidlc-docs/spec-coverage-debt.md`

Project-level file tracking screens with incomplete state matrix (missing mandatory states):

```markdown
# Spec Coverage Debt

> Screens whose state matrix is incomplete (missing one or more mandatory states).
> This file is automatically updated by Workflow D on every UI change-flow.
> Address as dedicated cleanup issues — not blocking individual changes.
>
> Entries may be stale if matrices were edited outside change-flow.
> Run periodic cleanup (e.g., dedicated `spec-coverage-debt` labeled issue).

| Screen | Location | Missing states | First detected | Last touched issue |
|---|---|---|---|---|
| <screen-name> | <path/to/frontend-components.md> | <comma-separated list> | <YYYY-MM-DD> | issue-<N> |
```

**Empty case** (no debt): file with header + line `> 0 entries — no debt`.

**Update protocol when D fires**:
1. Identify touched screen
2. Load matrix from its `frontend-components.md` (per-unit) or `aidlc-docs/screens/{screen-name}.md` (fallback)
3. Has all mandatory states (per `ui-stack.md` or hardcoded default)?
   - YES → no-op in debt file. If screen WAS in debt file, REMOVE its row (cleanup natural).
   - NO → upsert row in debt file with: missing states + last touched issue + first-detected date if new entry.

**File creation**: D creates `aidlc-docs/spec-coverage-debt.md` lazily on first fire if it does not exist (with header + empty-case line).

## Step 7: `aidlc-docs/screens/{screen-name}.md` (brownfield-without-units fallback)

When no unit decomposition exists for a touched screen, D writes to `aidlc-docs/screens/{screen-name}.md`. Same schema as per-unit `frontend-components.md` sub-pasos g+h+i:

```markdown
# {screen-name} — Frontend Components Spec

> Source: per-screen fallback for brownfield-without-units.
> Schema mirrors functional-design Step 6 sub-pasos g+h+i.

## State matrix
<table per Step 6 sub-paso g — see rules/construction/functional-design.md>

## Data bindings
<table per Step 6 sub-paso h>

## Snapshot contracts
<yaml entries per Step 6 sub-paso i>
```

**Directory creation**: D creates `aidlc-docs/screens/` lazily on first fire if it does not exist.

**Screen-name source**: derived from `existing-screens-map.md` (Workflow B output) — use the `Screen Name` column normalized to kebab-case for filename.

## Step 8: Refactor declaration override (D10)

When a change touches ui-domain files but is a pure refactor (no behavior delta), the agent declares this explicitly to bypass Layer A enforcement.

### Detection at triage gate

At Step 1 of any path-protocol, the agent classifies the issue. **Refactor signals**:
- Issue text mentions "refactor", "rename", "extract", "consolidate", "cleanup" without behavior change description
- No mention of new functionality, bug fix, or visible UI change

If refactor signals detected AND the change touches ui-domain files → route to **DESIGN path** instead of DIRECT. DIRECT/CROSS-DOMAIN paths do NOT support refactor declaration (see below).

### Mid-work escalation

If the agent classified as DIRECT but mid-work realizes the change is a pure refactor of UI files:
- Close the DIRECT branch
- Open a new DESIGN-path issue referencing the original
- Resume work in the new branch

### Declaration format (DESIGN path only)

In `aidlc-docs/construction/issue-{N}/brainstorming-spec.md`, within the `## UI Spec Delta` section (per Step 5), populate the `### Refactor declaration (if applicable)` sub-section:

```markdown
### Refactor declaration (if applicable)

- **Pure refactor**: yes
- **Justification**: <one line — why this is a refactor, not a behavior change>
- **Verification**: <how the agent confirmed identical render — e.g., diff of pre/post component output, snapshot test pass without contract changes, screenshot comparison>
```

When this section is present, Layer A passes without verifying state matrix updates. Abuse-protection via PR review humano.

### DIRECT/CROSS-DOMAIN constraint

Refactor declaration is ONLY valid in DESIGN path. Refactors entering via DIRECT or CROSS-DOMAIN MUST escalate to DESIGN per the mid-work escalation rule above. Reason: those paths do NOT produce `brainstorming-spec.md`, so there is no document where the declaration can be recorded.

## Step 9: Unknown-row escape hatch (D17)

When an affected row's behavior cannot be determined from code/tests/B-derived artifacts, the agent flags the row as unknown rather than stalling.

### When to flag

Use unknown flag when:
- The component imports the data indirectly (e.g., via context provider chains) and the agent cannot trace it
- E2E coverage doesn't exercise the affected state
- The state's behavior depends on runtime data the agent cannot inspect

### How to flag

In the `frontend-components.md` matrix, set the affected column to literal value `unknown — needs human input`:

```markdown
| State | Trigger | Components | Data | Copy |
|---|---|---|---|---|
| error | API failure | ErrorBanner, RetryButton | error.message | unknown — needs human input |
```

In `aidlc-docs/construction/issue-{N}/brainstorming-spec.md`, add to the `## UI Spec Delta` section:

```markdown
### Unknown rows

| Row state | Reason agent could not determine | Action requested from reviewer |
|---|---|---|
| error | E2E specs don't exercise error state for this filter | Confirm copy text for error state |
```

PR review humano resolves. Layer A passes — the row IS updated (with explicit unknown), not skipped.

## Step 10: REVERT path special handling (D18)

REVERT path is autonomous (no supervision) and mechanically restores previous state. When a REVERT touches UI files, special rules apply.

### Layer A relaxation for REVERT

Layer A does NOT block the REVERT PR even if the per-unit `frontend-components.md` is not updated to match the reverted code state. Reason: REVERT is typically emergency rollback; adding spec-update friction undermines its purpose.

### Mandatory debt registration

D registers the touched screen in `aidlc-docs/spec-coverage-debt.md` with a special marker:

```markdown
| Screen | Location | Missing states | First detected | Last touched issue |
|---|---|---|---|---|
| <screen-name> | <path> | pending re-derivation post-revert: <issue-N> | <YYYY-MM-DD> | issue-<N> |
```

The `pending re-derivation post-revert` marker in the Missing states column indicates that the screen's spec needs human re-sync, distinct from "matrix is incomplete" debt.

### Cleanup pass

A dedicated cleanup pass (separate issue, typically `spec-coverage-debt` labeled) consumes the post-revert markers and re-derives the affected screens' specs in line with the now-reverted code.
