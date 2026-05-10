# AI-DLC Structural Patterns

**Purpose**: Definitive catalog of structural patterns at both the phase and stage level. Use this document as the compliance checklist when creating new phases, new stages, or auditing existing ones.

**Source**: Cross-analysis of 21 rule files across Inception (7), Construction (6), and Verification (8) phases, validated against CLAUDE.md enforcement rules and common rule files.

**Date**: 2026-04-13

**Enforcement context**: This document covers *structural* patterns for defining phases and stages. For *enforcement* mechanisms (SRP hooks, Class A/B artifact classifier, `AIDLC_ENFORCEMENT_MODE`), see `_common/enforcement-architecture.md`.

---

## How to Use This Document

When creating a **new AI-DLC phase**:

1. Read Section 0 (Phase-Level Patterns) completely
2. Define all 7 phase-level elements
3. Create the phase's stages, each following Sections 1-3
4. Run the phase compliance checklist (Section 5.1) and stage checklists (Section 5.2+)

When creating a **new AI-DLC stage** within an existing phase:

1. Classify the new stage by type (Section 1)
2. Apply all UNIVERSAL stage patterns (Section 2) — no exceptions
3. Apply CONDITIONAL stage patterns based on stage type (Section 3)
4. Run the stage compliance checklist (Section 5.2+) before submitting for review

When **auditing** an existing phase or stage:

1. Run the appropriate compliance checklist (Section 5) against the rule files
2. Any UNIVERSAL pattern missing = structural gap — must fix
3. Any CONDITIONAL pattern missing = evaluate if the condition applies

---

## 0. Phase-Level Patterns

A phase is the top-level grouping in the AI-DLC workflow. Each phase contains one or more stages executed in sequence. These patterns define the structural contract that every phase must satisfy.

### 0.1 Phase Definition in CLAUDE.md

Every phase has a definition block in CLAUDE.md with this exact structure:

```markdown
# [Emoji] [PHASE NAME] PHASE

**Purpose**: [What this phase accomplishes]

**Focus**: [The core question this phase answers]

**Stages in [PHASE NAME] PHASE**:
- Stage Name (ALWAYS / CONDITIONAL — trigger condition)
- Stage Name (ALWAYS / CONDITIONAL — trigger condition)
```

After the definition block, each stage gets its own section with:
- Execution conditions (ALWAYS EXECUTE / Execute IF / Skip IF)
- Numbered step sequence referencing the stage's rule file
- MANDATORY audit logging directive
- Approval gate (Wait for Explicit Approval — DO NOT PROCEED until user confirms)

**Example** (from CONSTRUCTION):
```markdown
# 🟢 CONSTRUCTION PHASE

**Purpose**: Detailed design, NFR implementation, and code generation

**Focus**: Determine HOW to build it

**Stages in CONSTRUCTION PHASE**:
- Functional Design (CONDITIONAL, per-unit)
- NFR Requirements (CONDITIONAL, per-unit)
- ...
```

---

### 0.2 Phase Contract (Inter-Phase Deliverables)

Every phase has a formal contract defining what it receives and delivers.

**Structure** (maintained in the phase's design spec):

```markdown
| Phase | Receives from | Delivers to |
|---|---|---|
| [PREVIOUS] | ... | [deliverables] → [THIS PHASE] |
| [THIS PHASE] | [deliverables from previous] | [deliverables] → [NEXT PHASE] |
| [NEXT] | [deliverables from this phase] | ... |
```

**Rules:**
- Deliverables are specific artifacts, not vague concepts
- If a deliverable is missing when the phase starts, the phase must generate it (e.g., VERIFICATION reconciliation when production topology was absent)
- The phase contract is the authority for what the previous phase MUST produce before handoff
- Build & Compile (or equivalent handoff stage) includes a pre-flight checklist to verify deliverables exist

---

### 0.3 Phase Section in aidlc-state.md

Every phase has its own section in `aidlc-state.md` with checkboxes for all its stages.

**Structure:**
```markdown
### [PHASE NAME] PHASE
- [x] Stage Name (COMPLETED [date] — [summary])
- [~] Stage Name (IN PROGRESS — [current step])
- [ ] Stage Name
- [ ] Stage Name (CONDITIONAL — [trigger])
```

**Rules:**
- Added when the phase starts (first stage creates the section)
- All stages listed, including CONDITIONAL ones (marked with trigger condition)
- Updated immediately as stages progress (same interaction where work completes)
- Phase marked complete when all non-CONDITIONAL stages are `[x]` and all CONDITIONAL stages are either `[x]` or explicitly skipped

---

### 0.4 Phase Directory in aidlc-docs/

Every phase has its own directory under `aidlc-docs/`.

**Structure:**
```
aidlc-docs/
├── inception/
│   ├── plans/
│   ├── reverse-engineering/
│   ├── requirements/
│   ├── user-stories/
│   └── application-design/
├── construction/
│   ├── plans/
│   ├── {unit-name}/
│   │   └── {stage-name}/
│   ├── build-and-test/
│   └── shared-infrastructure.md
├── verification/
│   ├── plans/
│   └── [stage artifacts]
└── deployment/
    ├── plans/
    └── [stage artifacts: deployment-strategy.md, cd-pipeline-{diagnosis,design,impl-report}.md, deployment-execution-report.md, post-deploy-validation-report.md, rollback-validation-signoff.md]
```

**Rules:**
- Phase directory is created when the phase starts
- `plans/` subdirectory holds plan files and question files for all stages in the phase
- Stage-specific artifacts go in stage-named subdirectories or as top-level files (phase decides)
- Application code NEVER goes in aidlc-docs/ — only documentation and design artifacts

---

### 0.5 SP Integration Section

Every phase has its own section in `_common/superpowers-integration.md` defining which SP skills bind to which stages.

**Structure:**
```markdown
## [PHASE NAME] PHASE — Skill Invocations

> **Scope**: [When these bindings apply]

### Stage Name
- **`skill-name`** — [MANDATORY/CONDITIONAL]: [purpose]
- **Post-Implementation Chain** — MANDATORY: [trigger condition]
```

**Rules:**
- MANDATORY skills fire unconditionally when the stage executes
- CONDITIONAL skills fire only when the stated condition is true
- IRON LAW skills (TDD, verification-before-completion, systematic-debugging) apply universally — no per-phase override needed
- Each phase section lists only phase-specific bindings; IRON LAW skills are listed once in the universal section

---

### 0.6 Session Continuity Entries

Every phase needs entries in `_common/session-continuity.md` defining what artifacts to load when resuming the phase in a new session.

**Structure:**
```markdown
- **[Phase Name] Stages**: Read [artifact list from this phase] + all previous phase artifacts
```

**Rules:**
- Artifact loading is cumulative: later phases load ALL earlier phase artifacts
- The list must be specific (file names or directory paths), not vague ("all relevant artifacts")
- The "Welcome back" prompt shows: current phase, current stage, last completed step, next step

---

### 0.7 Phase Transition (Handoff Gate)

The transition between phases follows a specific pattern.

**Outgoing phase (last stage):**
1. Verify all deliverables in the phase contract exist
2. Pre-flight checklist (if defined) — block if any check fails
3. Approval gate: "**[Phase] complete. Ready to proceed to [NEXT PHASE]?**"
4. Log in audit.md

**Incoming phase (first stage):**
1. Load all artifacts from previous phases
2. Create phase section in aidlc-state.md with all stage checkboxes
3. Begin first stage execution

**The user sees a seamless transition:**
```
INCEPTION ✅ → CONSTRUCTION ✅ → VERIFICATION (current) → DEPLOYMENT (next)

Stage: Test Strategy Design
```

---

## 1. Stage Classification

Every AI-DLC stage falls into one of three types. The type determines which patterns are mandatory vs conditional.

| Type | Definition | Examples | Question DNA | Plan File |
|---|---|---|---|---|
| **DESIGN** | User judgment shapes the output. The stage produces artifacts that require human input to be correct. | Requirements Analysis, Functional Design, NFR Requirements, NFR Design, Infrastructure Design, Application Design, User Stories, Units Generation, Test Strategy Design, Test Environment Design | MANDATORY | MANDATORY |
| **EXECUTION** | Acts on already-approved designs. The model implements what was decided in a prior DESIGN stage. | Code Generation, Build & Compile, Test Environment Setup, Test Implementation, E2E Validation, CI Pipeline Implementation | NOT REQUIRED | CONDITIONAL (for complex multi-step execution) |
| **ANALYSIS** | Reads and reports. The model examines the current state and produces a diagnostic, not a design. | Workspace Detection, Reverse Engineering, Test Diagnosis, Production Readiness Gate | NOT REQUIRED | NOT REQUIRED |

**Classification rule**: If the stage produces an artifact where a wrong assumption by the model leads to rework → it's DESIGN. If the stage implements an already-approved plan → it's EXECUTION. If the stage reads and reports without changing anything → it's ANALYSIS.

**Classification note — Production Readiness Gate**: Classified as ANALYSIS with an enhanced approval gate (3-verdict format: GO/CONDITIONAL-GO/NO-GO). See Pattern 2.4 defined exceptions. The enhanced gate does not change the stage's classification — the stage reads evidence and emits a factual verdict without requiring human input to shape the output.

---

## 2. UNIVERSAL Patterns (All Stages, No Exceptions)

These patterns appear in every AI-DLC stage regardless of type. A stage missing any of these has a structural gap.

### 2.1 Prerequisites Section

Every stage begins with a formal prerequisites block listing dependencies and readiness conditions.

**Structure:**
```markdown
## Prerequisites
- [Blocking dependency] must be complete
- [Recommended dependency] recommended (provides [benefit])
- [Conditional dependency] (if [condition])
```

**Rules:**
- "must be complete" = blocking — stage CANNOT start without this
- "recommended" = helpful but not blocking
- "if [condition]" = conditional on project type (greenfield/brownfield)

**Exception**: The workflow entry point (Workspace Detection) has no Prerequisites section since it is the first stage with no dependencies. When creating a new phase that follows another, the first stage of that phase MUST have prerequisites referencing the previous phase's deliverables.

**Cross-reference**: Prerequisites link to specific artifact paths using the standard path conventions (see Pattern 2.7).

---

### 2.2 State Tracking Steps (Two-Level Checkbox System)

AI-DLC uses a two-level progress tracking system. Both levels are mandatory.

**Level 1 — Stage-Level** (in `aidlc-state.md`):
```markdown
### [PHASE] PHASE
- [x] Stage Name (COMPLETED [date] — [summary])
- [~] Stage Name (IN PROGRESS — [current step])
- [ ] Stage Name
```

**Level 2 — Plan-Level** (in the stage's plan file):
```markdown
## [Stage] Plan
- [x] Step 1: [Description]
- [x] Step 2: [Description]
- [ ] Step 3: [Description]
```

**Timing rules:**
- Mark "in-progress" (`[~]`) in aidlc-state.md at stage START
- Mark checkboxes `[x]` in plan file IMMEDIATELY after completing each step — same interaction
- Mark "complete" (`[x]`) in aidlc-state.md at stage END, with date and summary
- NEVER batch checkbox updates — update in the same interaction where the work completes

**CLAUDE.md enforcement:**
> "NEVER complete any work without updating plan checkboxes. IMMEDIATELY after completing ANY step described in a plan file, mark that step [x]. This must happen in the SAME interaction where the work is completed. NO EXCEPTIONS."

---

### 2.3 Audit Logging Step

Every stage logs interactions to `aidlc-docs/audit.md` with ISO 8601 timestamps.

**What gets logged:**
- Stage start (timestamp + context)
- Every user input (COMPLETE RAW INPUT — never summarized)
- Approval prompts (before presenting to user)
- Approval responses (after receiving from user)
- Stage completion (timestamp + summary)

**Format:**
```markdown
## [Stage Name] — [Event Type]
**Timestamp**: 2026-04-13T14:30:00Z
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"
**Context**: [Stage, action, or decision made]

---
```

**Rules:**
- ALWAYS append to audit.md — NEVER overwrite
- Use ISO 8601 format for all timestamps
- Log BEFORE asking approval (the prompt itself) and AFTER receiving response
- Never summarize or paraphrase user input

---

### 2.4 Completion Message Structure (3-Part)

Every stage presents completion in exactly 3 parts. No emergent behavior.

**Part 1 — Completion Announcement** (mandatory):
```markdown
# [Emoji] [Stage Name] Complete [- Context]
```

The `- [Context]` suffix is included only when the stage operates per-unit (e.g., `- [unit-name]`) or has other disambiguating context. Stages that execute once per workflow (e.g., Build & Compile, Test Strategy Design) omit the suffix.

**Part 2 — AI Summary** (mandatory for DESIGN/EXECUTION, optional for ANALYSIS):
- Structured bullet-point summary of key outcomes
- Include extension compliance summary (compliant/non-compliant/N/A per enabled extension)
- STRICTLY NO workflow instructions ("please review", "let me know", "before we proceed")
- Factual and content-focused only

**Part 3 — Formatted Workflow Message** (mandatory):
```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**
> Please examine [artifacts] at: `[path]`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - [Ask for modifications based on review]
> ✅ **Continue to Next Stage** - Approve [stage] and proceed to **[Next Stage Name]**

---
```

**Anti-pattern — NO EMERGENT BEHAVIOR:**
> "Construction phases MUST use standardized 2-option completion messages as defined in their respective rule files. DO NOT create 3-option menus or other emergent navigation patterns."

**Defined exceptions** (these are NOT emergent — they are explicitly defined in their rule files):

1. **INCEPTION conditional 3rd options**: Three INCEPTION stages include a conditional option guarded by `[IF condition:]` to allow adding skipped stages:
   - Requirements Analysis: `📝 **Add User Stories**` (when User Stories is skipped)
   - Application Design: `📝 **Add Units Generation**` (when Units Generation is skipped)
   - Workflow Planning: `📝 **Add Skipped Stages**` (when any stages are marked SKIP)
2. **Production Readiness Gate**: Uses a 3-verdict format (GO / CONDITIONAL-GO / NO-GO) with verdict-specific approval wording.

The 2-option base format remains the minimum. New conditional options MUST be explicitly defined in the rule file with a conditional guard — never emergent.

---

### 2.5 Approval Gate Step

Every stage (except ANALYSIS stages with informational output) implements a 2-option approval gate.

**Gate structure:**
1. Present 3-part completion message (Pattern 2.4)
2. Wait for explicit user approval — DO NOT PROCEED until user confirms
3. If user requests changes → update artifacts → repeat from step 1
4. Record approval in audit.md with timestamp and raw user input
5. Update aidlc-state.md to mark stage complete

**Exact wording (standardized):**
```
🔧 **Request Changes** - Ask for modifications to the [artifact] based on your review
✅ **Continue to Next Stage** - Approve [stage] and proceed to **[Next Stage]**
```

**Wording variants by phase:**
- INCEPTION stages may use `✅ **Approve & Continue**` — both forms are equivalent
- CONSTRUCTION and VERIFICATION stages consistently use `✅ **Continue to Next Stage**`
- Production Readiness Gate uses verdict-specific wording (`Accept Verdict`, `Accept Limitations`, `Accept & Continue`)

---

### 2.6 Directive Language Hierarchy

All stages use hierarchical directive language with specific meanings.

| Level | Keyword | Meaning | Override |
|---|---|---|---|
| Guidance | **DIRECTIVE** | Best practice, follow with professional judgment | Can adapt to context |
| Constraint | **CRITICAL** | Important safety constraint, must follow in almost all cases | Requires explicit justification to skip |
| Absolute | **MANDATORY** | Non-negotiable requirement, no exceptions | Cannot be overridden |
| External | **IRON LAW** | Hard requirement from external authority (e.g., typed agent strategy) | Cannot be overridden |

**Usage pattern:**
```markdown
**DIRECTIVE**: Thoroughly analyze [context] to identify ALL areas where [improvement].

**CRITICAL**: Default to asking questions when there is ANY ambiguity.

**MANDATORY**: Evaluate ALL of the following categories.
```

---

### 2.7 Artifact Path Conventions

All AI-DLC artifacts follow a consistent path structure.

**Phase-level:**
```
aidlc-docs/[phase]/
├── plans/                          # Plan files (with checkboxes and questions)
├── [stage-name]/                   # Stage-specific artifacts
│   └── [artifact].md
└── [cross-stage-artifact].md       # Shared artifacts
```

**Per-unit (Construction):**
```
aidlc-docs/construction/
├── plans/
│   └── {unit-name}-{stage-name}-plan.md
├── {unit-name}/
│   └── {stage-name}/
│       └── [artifact].md
└── shared-infrastructure.md
```

**VERIFICATION:**
```
aidlc-docs/verification/
├── plans/
│   └── {stage-name}-questions.md   # Question files
├── test-strategy.md
├── test-environment-design.md
├── production-infrastructure-blueprint.md
├── environment-setup-report.md
├── test-diagnosis.md
├── test-implementation-report.md
├── e2e-validation-report.md
├── ci-pipeline-report.md
└── production-readiness-verdict.md
```

**Rule**: Application code goes in workspace root, NEVER in aidlc-docs/. Documentation goes in aidlc-docs/ only.

---

### 2.8 Extension Compliance

Every stage evaluates enabled extensions and includes a compliance summary in its completion message.

**Process:**
1. Check `aidlc-docs/aidlc-state.md` → `Extension Configuration` table for enabled/disabled status
2. For each enabled extension: evaluate if its rules are applicable to this stage
3. Applicable + compliant → `compliant`
4. Applicable + non-compliant → `non-compliant` (BLOCKING finding — resolve before completion)
5. Not applicable → `N/A` with brief rationale
6. Disabled extensions → skip, log skip in audit.md

**Summary format** (in AI Summary, Part 2 of completion message):
```
Extension compliance: Security Baseline: compliant | PBT: N/A (no test code generated at this stage)
```

---

### 2.9 Sequential Step Numbering

All stages use sequential step numbering starting at 1.

**Conventions:**
- Main steps: Step 1, Step 2, Step 3, ...
- Sub-steps (inserted between existing steps): Step 2b, Step 6b
- Two-part stages: Part 1 steps 1-N, Part 2 steps N+1-M (continuous numbering)
- Every step that produces work should have a corresponding checkbox in the plan file

---

## 3. CONDITIONAL Patterns (Apply Based on Stage Type)

### 3.1 Question DNA Micro-Pattern (DESIGN Stages Only)

The 7-step micro-pattern for gathering and validating user input. MANDATORY for all DESIGN stages.

**Position in step sequence:** After initial analysis/plan creation, BEFORE artifact generation.

**The 7 steps:**

```
Step A: Create Plan
  → Generate plan with checkboxes [] for the stage
  → Each step should have a checkbox []

Step B: Generate Context-Appropriate Questions
  → DIRECTIVE: "Thoroughly analyze [context] to identify ALL areas
    where clarification would improve [quality]"
  → CRITICAL: "Default to asking questions when there is ANY ambiguity
    or missing detail. It's better to ask too many questions than to
    make incorrect assumptions."
  → MANDATORY: "Evaluate ALL of the following categories by asking
    targeted questions about each. For each category, determine
    applicability based on evidence — do not skip categories without
    explicit justification."
  → Questions embedded using [Answer]: tag format
  → 5-8 domain-specific categories per stage
  → "When in doubt, ask the question"

Step C: Store Plan
  → Save to: aidlc-docs/[phase]/plans/{context}-{stage-name}-plan.md
  → Plan includes both checkboxes AND [Answer]: tags

Step D: Collect Answers
  → Wait for user to complete all [Answer]: tags
  → Do not proceed until all questions answered

Step E: Analyze for Ambiguity
  → MANDATORY: Carefully review ALL responses for vague or ambiguous answers
  → CRITICAL: Watch for: "depends", "maybe", "not sure", "mix of",
    "somewhere between", "standard", "typical"
  → Check for contradictions between answers
  → Check for undefined terms or criteria

Step F: Follow-Up Questions (if ambiguities found)
  → Create clarification file: {stage-name}-clarification-questions.md
  → Use same [Answer]: tag format
  → "Do not proceed until ALL ambiguities are resolved"

Step G: Gate → Proceed to artifact generation
```

**Question format compliance** (per `_common/question-format-guide.md`):
- NEVER ask questions in chat — always in .md files
- Multiple choice: A, B, C, D options + X) Other (MANDATORY as last option)
- [Answer]: tag after each question
- Minimum: 2 meaningful options + Other
- Options must be mutually exclusive
- After answers: detect contradictions → create clarification file if needed → resolve before proceeding

**Adaptive depth for ambiguity rigor:**

| Project State | Rigor Level | Behavior |
|---|---|---|
| Greenfield / no prior work | Full | "Do not proceed until ALL ambiguities resolved" — mandatory follow-up |
| Brownfield / partial coverage | Standard | Detect and report, follow-up if ambiguous |
| Brownfield-full / mature project | Light | Detect and report, proceed if user accepts |

The driver for adaptive depth is stage-specific (e.g., VERIFICATION uses greenfield/brownfield detection from Test Strategy Design Stage 1 Step 6).

**Phase-level structural variant — VERIFICATION DESIGN stages:**

VERIFICATION DESIGN stages (Test Strategy Design, Test Environment Design) use a lighter structural variant of the Question DNA:

| Element | INCEPTION / CONSTRUCTION | VERIFICATION |
|---|---|---|
| Questions location | Embedded in plan file | Separate file: `plans/{stage-name}-questions.md` |
| Plan file with checkboxes | Yes (contains both plan + questions) | No — step sequences in rule files serve as progress tracker |
| Question categories | 5-8 formal categories per stage | Fewer, targeted to the stage's decision space |
| Ambiguity analysis | Formal Step E/F with clarification file | Implicit — always evaluate answers before proceeding |
| Steps A-C (plan+questions+store) | Full sequence | Collapsed into a single "clarifying questions" sub-step |

**Why the difference**: VERIFICATION stages operate on already-approved designs from Construction. Questions target specific gaps (risk priorities, fidelity tolerance, constraints), not broad requirements discovery. The formal plan-with-checkboxes pattern adds overhead without proportional value because VERIFICATION rule files already have explicit step sequences.

**When creating a new VERIFICATION DESIGN stage**: Use the lighter variant. Questions go to `aidlc-docs/verification/plans/{stage-name}-questions.md`. Do not create a separate plan file with checkboxes — the rule file's step sequence IS the plan.

---

### 3.2 Plan File with Checkboxes (DESIGN + Complex EXECUTION Stages)

MANDATORY for all DESIGN stages. CONDITIONAL for EXECUTION stages with multi-step complex work.

**When EXECUTION stages need plan files:**
- Stage involves dispatching multiple typed agents
- Stage has iterative loops (e.g., level-by-level test implementation)
- Stage spans multiple sessions
- Stage has 5+ distinct steps that need progress tracking

**Plan file structure:**
```markdown
# [Stage Name] Plan

## Context
[Brief summary of what this plan covers]

## Execution Steps
- [ ] Step 1: [Description]
- [ ] Step 2: [Description]
  - [ ] Sub-step 2a: [Detail]
  - [ ] Sub-step 2b: [Detail]
- [ ] Step 3: [Description]

## Questions (DESIGN stages only)

### Question 1
[Question text]

A) [Option]
B) [Option]
X) Other (please describe after [Answer]: tag below)

[Answer]:
```

**Naming convention:** `aidlc-docs/[phase]/plans/{context}-{stage-name}-plan.md`

---

### 3.3 Two-Part Structure: Planning / Generation (Complex DESIGN + EXECUTION Stages)

Used when a stage has a natural split between planning (what to do) and generation (doing it).

**Part 1 — Planning:**
- Create plan with checkboxes
- Generate questions (if DESIGN)
- Collect and validate answers
- Approval gate: user approves the PLAN before execution

**Part 2 — Generation:**
- Execute approved plan step-by-step
- Update checkboxes as each step completes
- Approval gate: user approves the OUTPUT

**Examples:**
- Code Generation (Part 1: plan → Part 2: code)
- User Stories (Part 1: plan + questions → Part 2: story generation)

**When to use:** If the plan itself needs approval before execution — i.e., getting the plan wrong would waste significant effort.

---

### 3.4 Error Handling / Escalation (EXECUTION Stages)

EXECUTION stages that run commands, tests, or builds must define escalation rules.

**Pattern:**
```markdown
If [failure condition]:
1. Apply `systematic-debugging` (IRON LAW) — diagnose before fixing
2. Fix and retry
3. After [N] failed iterations, escalate to user with:
   - What was attempted
   - What failed and why
   - Proposed solutions
4. Do NOT continue iterating blindly
5. Do NOT advance with broken [artifact/environment]
```

**Default iteration limit:** 3 attempts before escalation (per Test Environment Setup Stage 3).

---

### 3.5 Domain Hooks / Skill Invocations

Stages that produce artifacts benefiting from domain-specific knowledge invoke domain skills.

**Invocation timing:** BEFORE artifact generation, AFTER loading context.

**Pattern:**
```markdown
### Step N: Domain Hook

Invoke applicable domain skills:
- **`[skill-name]`** — for [specific purpose]
- **`[skill-name]`** — for [specific purpose]

Integrate recommendations into the [artifact] without duplicating or contradicting prior steps.
```

**Binding authority:** `_common/superpowers-integration.md` is the sole authority for which skills bind to which stages. Generic skill triggers are OVERRIDDEN by the bindings.

**Phase-level structural variant:**
- **VERIFICATION stages** formalize domain hooks as explicit named steps in their step sequence (e.g., `## Step N: Domain Hook`). This makes the invocation point unambiguous within the rule file.
- **INCEPTION / CONSTRUCTION stages** invoke domain skills implicitly via the superpowers-integration.md bindings without a dedicated step in the rule file. The invocation is triggered by the binding, not by a step.

When creating a new VERIFICATION stage with domain skill dependencies, add a dedicated `## Step N: Domain Hook` step. For INCEPTION/CONSTRUCTION stages, rely on the binding in superpowers-integration.md.

---

### 3.6 Session Continuity (Multi-Session Stages)

Stages that may span multiple sessions must support resumption.

**Resume protocol:**
1. Read `aidlc-state.md` — determine current stage and progress
2. Load all previous stage artifacts (per `_common/session-continuity.md`)
3. Load the current stage's plan file — find last completed checkbox
4. Present "Welcome back" prompt with current status
5. Log continuity in audit.md
6. Resume from the first uncompleted step

**Artifact loading by phase** (cumulative — later phases load earlier ones):
- INCEPTION: workspace analysis → RE artifacts → requirements → stories → design
- CONSTRUCTION: all Inception artifacts + per-unit design + code files + plans
- VERIFICATION: all Construction artifacts + all previous Verification stage artifacts

---

### 3.7 Content Validation (Stages Producing Diagrams)

Stages that produce Mermaid diagrams, ASCII art, or structured content must validate before writing.

**Per `_common/content-validation.md`:**
- ASCII diagrams: count characters per line, same width, only `+|-|^|v|<|>` and spaces
- Mermaid diagrams: validate syntax, escape special characters, provide text fallback
- Embedded code blocks: validate syntax
- Markdown: check correctness

**Pattern:**
```markdown
**Output**: [Diagram type] (ASCII or Mermaid — validate syntax per `_common/content-validation.md`).
```

---

## 4. Anti-Patterns (What NOT to Do)

### 4.1 Asking Questions in Chat
Questions go in .md files with [Answer]: tags. NEVER in chat.

### 4.2 Emergent Navigation
Do not create 3-option menus, custom approval formats, or navigation patterns not defined in the rule file.

### 4.3 Overwriting audit.md
Always append. Never overwrite.

### 4.4 Skipping Checkbox Updates
Never batch. Update in the same interaction where the work completes.

### 4.5 Summarizing User Input
Always capture complete raw input in audit.md. Never paraphrase.

### 4.6 Proceeding Without Approval
Every DESIGN and EXECUTION stage has an approval gate. Wait for explicit user confirmation.

### 4.7 Skipping Question Categories Without Justification
If the Question DNA applies, evaluate ALL categories. Skip only with explicit justification based on evidence from artifacts.

### 4.8 Assumptions Over Questions
"It's better to ask too many questions than to make incorrect assumptions." — Universal directive across all DESIGN stages.

---

## 5. Compliance Checklists

### 5.1 Phase-Level Checklist

Use when creating a new AI-DLC phase or auditing an existing one.

- [ ] **Phase definition in CLAUDE.md** — Purpose, Focus, Stages list, per-stage execution sections
- [ ] **Phase contract** — receives/delivers table in the phase's design spec, deliverables are specific artifacts
- [ ] **Phase section in aidlc-state.md** — all stages listed with checkboxes, CONDITIONAL stages marked with trigger
- [ ] **Phase directory** — `aidlc-docs/[phase]/` created with `plans/` subdirectory
- [ ] **SP integration section** — `_common/superpowers-integration.md` has section with MANDATORY/CONDITIONAL bindings per stage
- [ ] **Session continuity entries** — `_common/session-continuity.md` lists specific artifacts to load on phase resume
- [ ] **Phase transition — outgoing** — last stage has pre-flight checklist verifying all deliverables exist
- [ ] **Phase transition — incoming** — first stage loads all prior phase artifacts and creates phase section in aidlc-state.md
- [ ] **Stage rule files** — every stage has its own rule file in `playbooks/[phase]/`
- [ ] **Process overview updated** — `_common/process-overview.md` Mermaid diagram and phase descriptions include the new phase
- [ ] **Welcome message updated** — `_common/welcome-message.md` user-facing phase overview includes the new phase
- [ ] **SP Override table updated** — CLAUDE.md "SP Skill Invocation Override" table includes the new phase's mode and authority
- [ ] **Workflow changes protocol** — `_common/workflow-changes.md` includes scenarios for mid-workflow changes involving the new phase (if applicable)
- [ ] **Each stage passes its stage-level checklist** (5.2+)

### 5.2 Stage-Level Checklists

Use when creating or auditing any AI-DLC stage.

#### UNIVERSAL (All Stages)

- [ ] **Prerequisites section** — lists dependencies with blocking/recommended/conditional classification
- [ ] **State tracking — stage-level** — updates aidlc-state.md with in-progress and complete markers
- [ ] **Audit logging** — logs stage start, user inputs (raw), approval prompts, responses, completion
- [ ] **Completion message** — 3-part structure: announcement + AI summary + workflow message
- [ ] **Approval gate** — 2-option format (Request Changes / Continue to Next Stage)
- [ ] **Record approval** — logs approval in audit.md with timestamp and raw user input
- [ ] **Directive language** — uses DIRECTIVE/CRITICAL/MANDATORY/IRON LAW hierarchy correctly
- [ ] **Artifact paths** — follows `aidlc-docs/[phase]/` conventions
- [ ] **Extension compliance** — checks enabled extensions, includes compliance summary in completion message
- [ ] **Step numbering** — sequential, with sub-steps as needed

### DESIGN Stages (Additional)

- [ ] **Plan file with checkboxes** — creates `plans/{context}-{stage-name}-plan.md` with `[ ]` per step
- [ ] **State tracking — plan-level** — updates plan checkboxes immediately on completion
- [ ] **Question DNA** — 7-step micro-pattern: create plan → questions → store → collect → analyze ambiguity → follow-up → gate
- [ ] **Question format** — multiple choice A/B/C + X) Other, [Answer]: tags, in .md file (NEVER chat)
- [ ] **Ambiguity analysis** — detects vague signals, creates clarification file, resolves before proceeding
- [ ] **Adaptive depth** — adjusts question count and ambiguity rigor based on project state
- [ ] **Domain hooks** — invokes applicable skills per superpowers-integration.md bindings

### EXECUTION Stages (Additional)

- [ ] **Error handling / escalation** — defines failure protocol with iteration limit and user escalation
- [ ] **Plan file** (if complex) — creates plan with checkboxes for multi-step execution
- [ ] **Domain hooks** — invokes applicable skills per superpowers-integration.md bindings

### ANALYSIS Stages (Additional)

- [ ] **Approval gate** — may be informational (no blocking gate) if explicitly designed as such
- [ ] **No question requirement** — ANALYSIS stages do not require Question DNA

### Content Stages (Additional)

- [ ] **Content validation** — validates Mermaid/ASCII/code before writing files
- [ ] **Text fallback** — provides text alternative for complex visual content

### Multi-Session Stages (Additional)

- [ ] **Session continuity** — supports resume via aidlc-state.md + plan file checkboxes
- [ ] **Artifact loading** — loads all prior stage artifacts on resume
- [ ] **Welcome back prompt** — presents status and next steps on session resume
