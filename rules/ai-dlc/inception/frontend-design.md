# Frontend Design - Detailed Steps

## Purpose
**Formal UI/UX design step that produces machine-actionable frontend design artifacts before codegen runs**

AI-DLC has historically been backend-flavored: it produces detailed artifacts for components, methods, services, and domain entities — but no formal UI/UX design step. The result is UI codegen that proceeds without visual design input, producing components that fail to render well and require post-hoc redesign + recodification.

Frontend Design closes that gap. It runs in INCEPTION between Application Design and Units Generation. It produces three mandatory globals (`ui-stack.md`, `component-inventory.md`, `design-tokens.md`) and one conditional global (`screen-affinity-map.md`, only when the project has more than one screen). Per the **derivation principle**, every artifact has an explicit input source — derived from existing AI-DLC outputs (preferred) or elicited from the user via `[Answer]` tags. Pure invention by the agent is prohibited.

The stage activates proportionally: backend-only projects skip silently with zero overhead; UI-bearing greenfield projects elicit via `[Answer]` tags; brownfield projects with detected UI derive most artifacts from the `reverse-engineering` outputs.

## Prerequisites
- Workspace Detection must be complete
- Reverse Engineering must be complete (if brownfield) — provides `ui-stack-detected.md`, `ui-component-scan.md`, `ui-tokens-detected.md` when UI is detected
- Application Design must be complete — provides `component-methods.md` for data bindings derivation in CONSTRUCTION
- User Stories must be complete — provides acceptance criteria for the AC quality precheck and downstream state matrix derivation
- Execution plan must indicate Frontend Design stage should execute

## Activation

| Scenario | `frontend-design` runs? | Input source |
|---|---|---|
| Greenfield + UI declared | Yes | `[Answer]` elicitation |
| Greenfield + no UI | Skip silently | — |
| Brownfield + UI detected | Yes | `reverse-engineering` extension + `[Answer]` for gaps |
| Brownfield + no UI detected | Skip silently | — |
| Any + force flag | Yes | According to greenfield/brownfield context |

Single point of decision: `frontend-design` Step 1.

## Step 1 — Detect intent

The two branches below depend on workspace context. Read `aidlc-docs/aidlc-state.md` to determine whether the project is greenfield or brownfield (set by Workspace Detection), then execute the matching branch.

### 1.A Greenfield path (only enters here when aidlc-state.md confirms greenfield)

1.1 Read `aidlc-docs/aidlc-state.md`. If `overrides.force_frontend_design: true`, jump to Step 2.
1.2 Emit `[Answer]` tag:

> **[Answer]**: Does this project include frontend/UI components?
> Options:
>  - **Yes — full** web/mobile UI
>  - **Yes — minimal** admin/dashboard only
>  - **No** — backend / API / CLI only
>  - **Unsure** — proceed with frontend-design and skip if irrelevant

1.3 Branch on response:
- **No** → mark stage SKIPPED in `aidlc-docs/aidlc-state.md`, exit.
- **Yes — full** / **Yes — minimal** / **Unsure** → proceed to Step 1.5 (AC quality precheck), then Step 2. ("Unsure" is treated as Yes because the cost of running frontend-design and discovering it's irrelevant is bounded — the user can abandon at the approval gate. The cost of skipping when UI was actually needed is high.)

### 1.B Brownfield path

1.1 Read `aidlc-docs/aidlc-state.md`. If `overrides.force_frontend_design: true`, jump to Step 2.
1.2 Check for `ui-*-detected.md` files in `aidlc-docs/inception/reverse-engineering/`.
1.3 If absent: skip silently. Mark stage SKIPPED in `aidlc-docs/aidlc-state.md`, exit.
1.4 Else: proceed to Step 1.5 (AC quality precheck), then Step 2 with the brownfield mode-aware derivation flow.

## Step 1.5 — AC quality precheck (mandatory before Step 2)

Once `frontend-design` is activated (any path) and before Step 2 (artifact generation), run a precheck on `aidlc-docs/inception/user-stories/stories.md`:

```
For each story S in stories.md:
  If S is in scope for UI (i.e., its persona interacts with UI, OR
  it is in a unit alongside stories whose persona has UI interactions,
  per `unit-of-work-story-map.md` if available):
    Read its acceptance criteria.
    For each AC:
      - Check that observable states are enumerable (G/W/T THEN clauses,
        or explicit "user sees X" / "system shows Y" statements).
      - Vague AC ("user can log in", "users get errors") → fail this check.
    If precheck fails for any AC:
      Emit [Answer] tag asking the user to enrich the AC, citing the
      problematic story and offering G/W/T as recommended format.
      Block Step 2 until [Answer] resolved.
```

**Why this matters**: state matrix derivation in CONSTRUCTION assumes AC enumerate observable states. Vague AC produce a degenerate state matrix that defeats the entire visual-spec discipline. The precheck converts a silent upstream failure into an explicit gate.

**Coherence with G/W/T recommendation**: the precheck is the enforcement mechanism that gives the recommendation in `user-stories.md` teeth. If the user chose free-form AC and the AC turn out to be vague, the precheck forces enrichment — at which point G/W/T becomes the natural answer.

**Skip condition**: if no UI-scoped stories exist (rare given that `frontend-design` was activated, but possible — backend-heavy projects with one trivial UI), the precheck completes trivially.

## Step 2 — Mode selection + artifact generation

**Routing**: agents arriving from path 1.A (greenfield) execute Step 2.A. Agents arriving from path 1.B (brownfield) execute Step 2.B. Each path is mutually exclusive — do not execute both.

### Mode Selection — applies to the three mandatory globals

Each of the three mandatory globals (`ui-stack.md`, `component-inventory.md`, `design-tokens.md`) operates in one of four modes, selected per-file via `[Answer]` tag in the brownfield flow (greenfield is always Synthesis).

### Pre-execution check — Behavioral Derivation artifacts

Before presenting Mode Selection prompts, check for `aidlc-docs/inception/behavioral-derivation/`:

- **If the directory exists AND contains all 5 artifacts** (`existing-stories.md`, `existing-personas.md`, `existing-screens-map.md`, `existing-flows.md`, `existing-domain-baseline.md`) plus `REVIEW.md`: B has run successfully. Activate **derived-mode pre-population** for relevant `[Answer]` tags throughout Step 2. High-confidence items from B's artifacts become `[Derived: <value>]`; low-confidence items become `[Answer]` with the derived value as a hint. The user confirms, adjusts, or overrides each.
- **If the directory exists but artifacts are incomplete** (missing one or more files): treat as if B has not run; use standard `[Answer]` elicitation. Emit a warning: `⚠️ Behavioral derivation directory exists but is incomplete — falling back to manual elicitation. Run B explicitly to populate.`
- **If the directory does not exist** (greenfield, or brownfield without B yet): use the standard `[Answer]` elicitation as today.

Read `REVIEW.md` (root of the directory) for pending-review counts. If non-empty, show this banner before any prompt:

> ⚠️ Behavioral derivation has N pending reviews in REVIEW.md.
> High-confidence items below are pre-populated. Low-confidence items appear as [Answer] with suggested values.

### Worked example: persona elicitation with B's artifacts

Without B (greenfield):
> [Answer]: List the main user personas for this project

With B (brownfield, `existing-personas.md` available):
> [Derived: Admin (full access), Regular User (standard access)]
> Source: existing-personas.md — 2 high-confidence personas
> 1 additional persona requires review (Guest) — see REVIEW.md
> Confirm, adjust, or override:

The same pattern applies to ui-stack questions (sourced from A's `ui-stack-detected.md`), domain entity references (sourced from B's `existing-domain-baseline.md`), screen counts (sourced from B's `existing-screens-map.md`), and any other elicitation that has a B-side artifact.

**Scope note**: Mode Selection does NOT apply to the conditional 4th global `screen-affinity-map.md`. That artifact is always derived from `stories.md` + screen identification (it has no in-repo source-of-truth equivalent — there is no "existing screen affinity map" in any project). It is generated only when the project has more than one screen and is omitted otherwise.

| Mode | Meaning | When to choose |
|---|---|---|
| **Synthesis** | Full content lives inside `aidlc-docs/`. The artifact IS the source of truth. | Greenfield (always); brownfield with no existing design system |
| **Index** | The artifact is a pointer/manifest to the in-repo source of truth (e.g., `tailwind.config.ts`, `src/components/`). The repo's existing files remain authoritative. | Brownfield with mature design system already in code |
| **Hybrid** | Per-file mode mix (e.g., `design-tokens.md` Index, `component-inventory.md` Synthesis if no docs exist for it) | Brownfield with partial coverage |
| **External** | Design system lives outside the repo (npm package like `@company/design-system`, separate repo). The artifact is an opaque pointer with package name / version / URL. | Brownfield consuming external design system as a dependency |

**Index-mode artifacts** include a validation footer instructing the agent to read the in-repo source-of-truth file directly. The pointer file is a navigation aid, not a copy.

**External-mode artifacts** include the package name, version, and a link to its design tokens / component documentation.

### 2.A Greenfield execution path

Mode is always **Synthesis** in greenfield (no in-repo source-of-truth exists yet, so Index/External modes do not apply). Proceed directly to Step 3 (plan generation) with all three mandatory globals in Synthesis mode.

### 2.B Brownfield execution path (mode-aware)

2.B.1 Report scan findings from `aidlc-docs/inception/reverse-engineering/ui-*-detected.md`, citing concrete numbers. Example:

> "Found: `tailwind.config.ts` with 12 custom colors and 8 spacings, 14 components in `src/components/ui/`, `@radix-ui/react-*` in dependencies. No Storybook detected."

2.B.2 Emit `[Answer]` tag with the four-option mode selector, **per-artifact**:

> **[Answer]**: Select mode per artifact (Synthesis | Index | Hybrid | External)
>  - `ui-stack.md` → ?
>  - `component-inventory.md` → ?
>  - `design-tokens.md` → ?

The user can mix modes (e.g., Index for tokens, Synthesis for inventory, External for stack if Radix is the system).

2.B.3 For each artifact, the mode-specific generation will execute in Step 4:
- **Synthesis**: derive first cut from `ui-*-detected.md`; fields without source (a11y baseline, motion, mandatory states) → `[Answer]`.
- **Index**: generate manifest pointing at the in-repo source-of-truth (config file, components dir, etc.); only project-policy fields declared inline (a11y baseline, mandatory states).
- **Hybrid**: per-section mode mix; declare per-section which fields are Index vs Synthesis.
- **External**: declare package name, version, and link to the external design system docs; only project-specific additions inline.

2.B.4 The user fills `[Answer]` tags for whatever the chosen mode requires (Synthesis: gaps; Index: project-policy fields; External: package metadata).

2.B.5 **MANDATORY**: invoke the same "Analyze answers" + "MANDATORY follow-up questions" sub-steps defined in Step 3 (greenfield path). Brownfield-specific responses to mode selectors (e.g., "Hybrid" without specifying which sections) and gap-filling [Answer] tags must pass the same ambiguity gate before proceeding to Step 4. Do NOT proceed if any [Answer] response is vague.

The 4th conditional global (`screen-affinity-map.md`) is always derived from `stories.md` regardless of the modes chosen above (see Step 4.5).

## Step 3 — Generate plan with [Answer] tags

- Generate a frontend-design plan with checkboxes `[ ]` for each artifact and decision.
- Embed all `[Answer]:` tags collected in Step 2 (mode selection per artifact, Synthesis content gaps, Index source-of-truth confirmations, External package metadata).
- Include questions covering:
  - **Framework UI** — which framework + version
  - **Styling solution** — Tailwind, CSS Modules, vanilla-extract, styled-components
  - **Component library** — shadcn/ui, Radix, MUI, Chakra, headless, custom
  - **Layout primitives** — Stack, Cluster, Grid, Sidebar, Switcher
  - **A11y baseline** — declared WCAG level + concrete checklist
  - **Mandatory states** — closed list every screen must cover (loading, empty, error, success/populated, partial)
  - **Responsive strategy** — mobile-first vs desktop-first; explicit breakpoints
  - **Motion principles** — standard duration, curves, what is and isn't animated
  - **Testing strategy** — Level A runtime (Vitest/Jest); Level B/C tools auto-detected
  - **Tokens** — color, spacing, typography, radius, shadow, breakpoints, z-index scale
- Save the plan as `aidlc-docs/inception/plans/frontend-design-plan.md`.
- Request user input by asking the user to fill `[Answer]:` tags directly in the plan document.
- Wait for ALL `[Answer]:` tags to be completed. Do not proceed if any remain blank.

### Analyze answers (MANDATORY)
Before proceeding, you MUST carefully review all user answers for:
- **Vague or ambiguous responses**: "mix of", "somewhere between", "not sure", "depends"
- **Undefined criteria or terms**: References to concepts without clear definitions
- **Contradictory answers**: Responses that conflict with each other
- **Missing design details**: Answers that lack specific guidance
- **Answers that combine options**: Responses that merge different approaches without clear decision rules

### MANDATORY follow-up questions
If the analysis above reveals ANY ambiguous answers, you MUST:
- Add specific follow-up `[Answer]:` tags to the plan document
- DO NOT proceed to artifact generation (Step 4) until all ambiguities are resolved

## Step 4 — Generate artifacts

Per the modes selected in Step 2, generate to `aidlc-docs/inception/frontend-design/`:

- `ui-stack.md` (mandatory) — Single source for stack and principles. Sections: Framework UI (with version), Styling solution, Component library, Layout primitives, Routing, State management, Form handling, A11y baseline (declared WCAG level + concrete checklist: focus rings visible, contrast ≥4.5:1 for normal text, all interactive elements with accessible name, logical keyboard nav, ARIA only when native HTML insufficient), Mandatory states (closed list: loading, empty, error, success/populated, partial — a component omitting one is a bug), Responsive strategy, Motion principles, Testing strategy.
  - **Synthesis** mode: file is the source of truth; greenfield elicits 100% via `[Answer]`. Brownfield with no design system: ~80% derived from `ui-stack-detected.md`, `[Answer]` fills gaps.
  - **Index** mode: file is a navigation manifest pointing at `package.json`, framework config, etc.; complementary section captures decisions not in code (a11y baseline, mandatory states, motion).
  - **External** mode: file points at the external design system package; only project-specific decisions declared inline.

- `component-inventory.md` (mandatory) — Catalog of UI primitives. Each entry declares: Source (e.g., `shadcn:button` or `custom:src/components/Button.tsx`), Variants, Sizes, States required (must include all mandatory states from `ui-stack.md`), A11y notes, and the literal anchor line `Do NOT create new "Button-ish" components. Compose this one.` The "Do NOT create new" line is literal text — it is the anchor the agent reads at unit start.
  - Hard rule: each component declares **all** mandatory states from `ui-stack.md`. Missing state → forced follow-up.
  - **Synthesis**: full inventory inline. Greenfield imports library base + `[Answer]` for custom primitives. Brownfield: best-effort scan of `src/components/` produces first cut, flagged "needs human review" for `cva()` calls and polymorphic props; `[Answer]` confirmation required.
  - **Index**: navigation manifest. Each entry points at the source component file; the "Do NOT create new" anchor and mandatory-states declaration still live inline because they are project policies.
  - **External**: pointer to the external library's component documentation; only project-specific *additions* are inventoried.
  - **Important**: no `Used by:` field in any mode. Forward-population from CONSTRUCTION would cross phase boundary.

- `design-tokens.md` (mandatory) — Tokens for color, spacing, typography, radius, shadow, breakpoints, z-index scale.
  - **Synthesis**: tokens declared inline; codegen derives `tailwind.config.js` / CSS vars from this file.
  - **Index**: file points at existing `tailwind.config.*` / theme files as source-of-truth; complementary section captures tokens that don't live in code (motion durations, formal contrast minima, z-index scale if informal).
  - **Hybrid**: e.g., color/spacing/typography Index from existing config; motion/contrast Synthesis.
  - **External**: pointer to external design system package; tokens are imported, not declared.

- `screen-affinity-map.md` (conditional, only if more than 1 screen — see Step 4.5).

## Step 4.5 — Screen identification

Before generating `screen-affinity-map.md`, identify screens by reading `aidlc-docs/inception/user-stories/stories.md` and `aidlc-docs/inception/user-stories/personas.md`. A screen is a distinct user-facing surface (e.g., `/login`, `/dashboard`, `/settings`).

**Brownfield**: cross-reference with detected routes from `aidlc-docs/inception/reverse-engineering/ui-stack-detected.md`.

**Branch**:
- If **more than 1 screen** identified: generate `aidlc-docs/inception/frontend-design/screen-affinity-map.md` with one section per screen listing persona, mapped stories, components used, and notes about state-sharing across stories. Example entry:

  ```markdown
  ## Screen: /alerts (Live Alert Feed)
  - Persona: Platform Operator
  - Stories: STORY 1.1, STORY 1.2, STORY 1.6
  - Components: AlertFeed, AlertCard, FilterBar, NetworkBadge
  - Notes: Stories 1.1 and 1.2 share state — keep in same unit.
  ```

- If **only 1 screen** identified: omit the file. (Single-screen apps fall through to the `{{has_ui}}` fallback in `units-generation` per the typed-implementer template.)

Record the screen count in `aidlc-docs/aidlc-state.md` (used by the `{{has_ui}}` detection algorithm at typed-agent bootstrap).

## Step 5 — Approval gate

### Log approval prompt
- Log the approval prompt with timestamp in `aidlc-docs/audit.md` (ISO 8601).
- Include the complete approval prompt text.

### Present completion message

```markdown
# 🎨 Frontend Design Complete

[AI-generated bullet summary of artifacts created and modes chosen — e.g., "ui-stack.md (Synthesis), component-inventory.md (Index → src/components/), design-tokens.md (Index → tailwind.config.ts), screen-affinity-map.md (3 screens)"]

> **📋 <u>**REVIEW REQUIRED:**</u>**
> Please examine the frontend design artifacts at: `aidlc-docs/inception/frontend-design/`

> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the frontend design if required
> ✅ **Approve & Continue** - Approve design and proceed to **Units Generation**
```

### Wait for explicit approval
- Do not proceed until the user explicitly approves the frontend design.
- Approval must be clear and unambiguous.
- If the user requests changes, update the artifacts and repeat the approval process.

### Record approval response
- Log the user's approval response with timestamp in `aidlc-docs/audit.md`.
- Include the exact user response text.

### Update progress
- Mark Frontend Design stage complete in `aidlc-docs/aidlc-state.md`.
- Update the "Current Status" section.
- Prepare for transition to Units Generation.

## Idempotence and re-runs

`frontend-design` follows the same rerun pattern as `reverse-engineering`:

- **First run**: produce all artifacts. Mark COMPLETE.
- **Re-run, artifacts current**: existing artifacts are loaded; stage skipped.
- **Re-run, artifacts stale or user-forced**: regenerate as needed.

**Staleness criterion**: an artifact is stale if either:

1. The artifact's mtime is older than the most recent change to its inputs. **Inputs depend on the artifact's mode**:
   - **`ui-stack.md`, `component-inventory.md`, `design-tokens.md`**:
     - **Synthesis** mode: input is the original `[Answer]` document (greenfield) or `aidlc-docs/inception/reverse-engineering/ui-*-detected.md` (brownfield).
     - **Index** mode: input is the in-repo source-of-truth file the artifact points at (e.g., `tailwind.config.ts` for tokens, `src/components/` directory mtime for inventory, `package.json` for stack).
     - **External** mode: input is the external package version recorded in the artifact. Staleness is detected when the package version in `package.json` differs from the version recorded; the user controls when to update.
   - **`screen-affinity-map.md`**: input is `aidlc-docs/inception/user-stories/stories.md` and `aidlc-docs/inception/user-stories/personas.md`.
2. The user explicitly requests rerun via the same mechanism `reverse-engineering` already supports (no new mechanism introduced). The escape hatch flag (see Escape hatch below) does NOT trigger re-execution of a COMPLETE stage — it only activates a non-COMPLETE stage.

**Refresh strategy when stale**: targeted regeneration of the stale artifacts, not full stage rerun. Same pattern as `reverse-engineering`.

## Escape hatch

Flag in `aidlc-docs/aidlc-state.md`:

```yaml
overrides:
  force_frontend_design: true
  force_frontend_design_reason: "Adding admin dashboard to existing API service"
  force_frontend_design_set_at: "2026-05-05T14:30:00Z"
```

This rule file declares the contract. The **mechanism for setting the flag** (slash command, CLI flag, manual edit) is deferred to implementation execution — see Open implementation questions below. Any of the three honors the contract.

Lifecycle:

- **Set**: by user or command.
- **Read**: `frontend-design` Step 1.1 (both greenfield and brownfield branches), before any other detection.
- **Effect**: skip detection, enter Step 2 with the appropriate (greenfield or brownfield) flow.
- **Reset**: flag persists as audit trail; not removed automatically.
- **Re-run after stage COMPLETE**: idempotence rules (above) take precedence over the flag. Once `aidlc-docs/aidlc-state.md` records `frontend-design` as COMPLETE, subsequent re-runs check freshness against inputs (per the staleness criterion) regardless of the flag's value. The flag is consulted only when activating the stage from a non-COMPLETE state.

## Open implementation questions (deferred)

These are flagged in the source spec and resolved during execution, not in this rule file:

- **Q1**: concrete syntax of the escape hatch setter (slash command vs CLI flag vs manual edit) — resolved in implementation plan.
- **Q3**: initial baseline approval policy for snapshot tests (PR review required? auto-approve with flag?) — resolved in implementation plan + the `snapshot-driven-ui` skill.
- **Q4**: formal schema of `aidlc-docs/aidlc-state.md` with its `overrides` section — declared inline in this rule file when set, until a template is added.
- **Q5**: single-screen app fallback for `{{has_ui}}` detection at typed-agent bootstrap — current MVP fallback uses presence of `ui-stack.md` to mark all units `has_ui=true`; a more discriminating signal may be needed if a project has both UI and non-UI units in a single-screen scope.
