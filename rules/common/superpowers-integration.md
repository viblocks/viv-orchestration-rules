# Superpowers Integration — Skill-to-Stage Binding Rules

**Purpose**: Operative rules for invoking Superpowers skills at the correct AI-DLC stages.

**PRECEDENCE RULE**: During AI-DLC workflow execution, THIS FILE is the sole authority for SP skill invocation. Generic SP skill triggers (from `using-superpowers`, `brainstorming`, `test-driven-development`, etc.) are OVERRIDDEN by the bindings below. A skill not listed at a stage = do NOT invoke it at that stage, regardless of what the skill's own trigger says. See CLAUDE.md "SP Skill Invocation Override" for the full override table.

**Model mental**: AI-DLC = orchestrator + single artifact store. SP = stateless execution engine.
No AI-DLC stage is replaced. SP skills implement specific stages at higher quality.
SP artifacts are persisted in aidlc-docs/ following AI-DLC structure — SP has no separate store.

---

## FASE 0 — Pre-Inception (condicional)

**Trigger**: Feature ambigua con múltiples enfoques de diseño viables, ANTES de formalizar requisitos.

| Acción | SP skill |
|---|---|
| Explorar 2-3 enfoques → usuario elige → resultado alimenta AI-DLC Inception | `brainstorming` |

> Phase 0 es DISTINTA de Requirements Analysis. Phase 0 ocurre antes de que Inception inicie formalmente. Si el feature llega con diseño claro, Phase 0 se salta y se entra directo a Inception.

---

## IRON LAW Skills (universales)

Estos skills son non-negotiable en toda sesión de construcción. No hay excepciones.

| Skill | Cuándo | Tipo |
|---|---|---|
| `test-driven-development` | Dentro de cada subagente implementador | Embebido en typed agent, no invocado por separado |
| `verification-before-completion` | Antes de cada approval gate AI-DLC y antes de marcar cualquier tarea completa | Gate |
| `systematic-debugging` | Antes de proponer cualquier fix para cualquier falla | Gate |
| **Domain skill invocation** | Antes de la primera edición de código en una sesión de CR | Gate |

**Domain skill invocation**: al inicio de cualquier sesión donde se tocará código, invocar el skill de dominio correspondiente ANTES del primer edit o Agent dispatch. Una invocación cubre la sesión completa.

---

## DISPATCH CONTRACT Rule (universal)

**Every creation or modification of a code artifact during AI-DLC workflow stages MUST be executed by a subagent — never inline by the main session.**

Every `Agent()` call that creates or modifies a code artifact MUST include a `[DISPATCH CONTRACT]` section at the start of the prompt, generated just-in-time before calling each Agent. The main session orchestrates (generate contract → dispatch → verify evidence) and never directly edits code artifacts.

**Full spec**: `common/subagent-dispatch-contract.md` — contract template, code artifact taxonomy, JIT generation steps, evidence verification rules.

---

## Artifact Store Rule

SP skill outputs are saved where AI-DLC expects them:

| SP Skill | Output | AI-DLC Path |
|---|---|---|
| `brainstorming` | Design spec | Depends on invocation context — see INCEPTION/CONSTRUCTION sections for exact paths |
| `writing-plans` | TDD execution plan | `aidlc-docs/construction/plans/{unit-name}-code-generation-plan.md` |
| `subagent-driven-development` | Code + tests | workspace root (per code-generation.md structure) |
| `requesting-code-review` | Review report | `aidlc-docs/construction/{unit-name}/code/code-review-report.md` |

SP never creates `docs/superpowers/plans/` for AI-DLC-managed units.

---

## INCEPTION PHASE — Skill Invocations

> **Scope**: These bindings apply to BOTH greenfield and brownfield projects.
> Brownfield activates Reverse Engineering; greenfield skips it. All other stages use the same bindings regardless of project type.

> Note: No SP skills apply to **Workspace Detection**, **User Stories**, or **Workflow Planning**. AI-DLC handles these natively.
> For **Requirements Analysis**: `brainstorming` applies conditionally (see below); all other SP skills do not apply.

### Requirements Analysis
- **`brainstorming`** — CONDITIONAL: when the feature has multiple viable design approaches before requirements are formalized. Output: `aidlc-docs/inception/requirements/brainstorming-spec.md`

### Reverse Engineering (BROWNFIELD ONLY)

> This stage executes ONLY when `brownfield = true` in aidlc-state.md. In greenfield projects, these bindings do not apply — skip to Requirements Analysis.

- **`dispatching-parallel-agents`** — CONDITIONAL: independent subsystems to analyze in parallel (multi-package scan, parallel artifact generation for 8 independent documentation outputs).
- **`systematic-debugging`** — CONDITIONAL: unexpected behavior found in existing code during analysis.

#### Post-RE: Routing Table Population (MANDATORY for first adoption)

After RE completes and before Requirements Analysis, if the routing table in CLAUDE.md is empty or incomplete:

1. Read `component-inventory.md` + `technology-stack.md` from RE artifacts
2. For each service of type Application or Shared: classify domain by stack (NestJS/Express/Fastify → Backend, React/Vue/Angular → Frontend, other → Unmatched)
3. Generate routing table rows in CLAUDE.md and mirror in `typed-agent-strategy.md`
4. Services marked "Unmatched" (no typed agent for their stack) → resolved in Workflow Planning with user decision: create new typed agent, use general-purpose, or defer
5. User approves as part of RE approval gate (no additional gate)

See `common/routing-table-population-protocol.md` for full spec.

### Application Design
- **`brainstorming`** — CONDITIONAL: architectural decision with multiple viable approaches (components, services, integration patterns). Output: `aidlc-docs/inception/application-design/brainstorming-spec.md`
- **`dispatching-parallel-agents`** — CONDITIONAL: multiple independent services to design in parallel.

### Units Generation
- **`dispatching-parallel-agents`** — CONDITIONAL: units with no dependencies between them — parallel analysis.

---

## CONSTRUCTION PHASE — Skill Invocations

### At the Start of Each Unit (MANDATORY before any code)
- **`using-git-worktrees`** — MANDATORY, unless already working in an isolated worktree/branch.

### Functional Design
- **`brainstorming`** — CONDITIONAL: domain design decision with multiple viable approaches. Output: `aidlc-docs/construction/{unit-name}/functional-design/brainstorming-spec.md`

### NFR Requirements
- **`brainstorming`** — CONDITIONAL: tech stack selection or patterns with multiple comparable options. Output: `aidlc-docs/construction/{unit-name}/nfr-requirements/brainstorming-spec.md`
  > CRITICAL for blockchain/AI agents: timing, ordering, and atomicity constraints are irreversible.
  > The `POLL_BATCH_SIZE=1` edge case (fast-execution ordering) was discovered in Build&Test by skipping this step.
- **`dispatching-parallel-agents`** — CONDITIONAL: independent concerns (security + performance + scalability) in parallel.

### NFR Design
- **`brainstorming`** — CONDITIONAL: multiple viable implementation patterns (circuit breaker, retry, ordering, atomicity). Output: `aidlc-docs/construction/{unit-name}/nfr-design/brainstorming-spec.md`

### Infrastructure Design
- **`brainstorming`** — CONDITIONAL: multiple infrastructure options with significant trade-offs. Output: `aidlc-docs/construction/{unit-name}/infrastructure-design/brainstorming-spec.md`

### Code Generation Part 1 — Planning (MANDATORY)
- **`writing-plans`** — MANDATORY: produces the bite-sized TDD plan from FD + NFR + Infra.
- **Routing Table Population** — If the plan includes paths in `services/*` or `packages/*` without a row in the routing table, the plan MUST include **Step 0: update routing table** before any typed agent dispatch. Classify domain from NFR/Infra artifacts, add row to `.claude/routing/routing-table.json`. User approves as part of the plan approval gate. See `common/routing-table-population-protocol.md`.
- **`verification-before-completion`** — IRON LAW: before the AI-DLC approval gate.

### Code Generation Part 2 — Execution (MANDATORY)
- **`subagent-driven-development`** — MANDATORY: executes the plan; fresh subagent/task + 2-stage review.
  - Inside each subagent → **`test-driven-development`** — IRON LAW, no exceptions.
  - **`verification-before-completion`** — IRON LAW before marking each task complete.
  - **`systematic-debugging`** — IRON LAW on any failure before proposing a fix.
  - **`requesting-code-review`** — integrated into the loop after each task (spec + quality review).

> Note: `requesting-code-review` in Code Generation is mandatory and per-task (built into the subagent-driven-development loop). In Build and Test, it is an additional optional invocation for a final whole-unit review before sign-off — both invocations serve different purposes.

### Post-Implementation Chain (MANDATORY, after every typed implementer completes)

The chain fires after Code Generation Part 2 tasks AND after any standalone typed implementer dispatch:

1. **`verification-before-completion`** — tests + build must pass (IRON LAW)
2. **Typed code reviewer** of the domain — dispatched by orchestrator, not by the implementer
3. If reviewer reports issues → resolve and return to step 1
4. **`security-reviewer`** IF changed files match trigger criteria (controllers, DTOs, guards, auth, frontend forms, config, package.json, chain/enrichment) — see owasp-security skill. If no files match → skip, log N/A. If CRITICAL/HIGH → resolve and return to step 1
5. Only after all gates pass: commit final + close issue/CR

**Completion Signal Protocol**: When a typed implementer finishes, its final output MUST include:

    IMPLEMENTER COMPLETE — ORCHESTRATOR: dispatch verification + code review before commit.

The implementer does NOT commit with closure language ("closes #", "fixes #"). The orchestrator handles closure.

**New Abstraction Gate**: If a task requires creating a new event type, aggregate, entity, or abstraction that does not exist in the codebase, the implementer STOPS and returns:

    DESIGN REQUIRED — this task introduces [description].
    Execute superpowers:brainstorming before implementing.

Only modifications to existing abstractions proceed without this gate.

### At Unit Close (MANDATORY)
- **`finishing-a-development-branch`** — MANDATORY.

---

## BUILD AND TEST — Skill Invocations

| Moment | SP Skill | Type |
|---|---|---|
| Before asserting any test passes | `verification-before-completion` | IRON LAW |
| When any test fails | `systematic-debugging` | IRON LAW |
| Multiple independent failures | `dispatching-parallel-agents` | Conditional |
| Final review before sign-off | `requesting-code-review` | Conditional |
| After everything passes | `finishing-a-development-branch` | MANDATORY |
| After all units pass and branch is ready | Audit periodic (see project profile) | Recommended |

> Note: `finishing-a-development-branch` fires twice in the full workflow — once per unit when Code Generation Part 2 completes, and again after Build and Test passes for all units. These are distinct invocations at different points.

---

## VERIFICATION PHASE — Skill Invocations

> **Scope**: These bindings apply when the AI-DLC workflow enters the VERIFICATION phase (after CONSTRUCTION completes).

### Stage 1: Test Strategy Design
- **`dev-testing-strategy`** — MANDATORY: enrich plan with project testing patterns (pyramid, level definitions)
- **Domain skill** (e.g., `crypto-backend`, `react-crypto-frontend`) — MANDATORY: domain-specific test knowledge

### Stage 2: Test Environment Design
- **`docker-patterns`** — MANDATORY: docker-compose.test.yml design
- **`infra-cloud`** — MANDATORY: production topology design and reconciliation
- **`brainstorming`** — CONDITIONAL: when environment design has multiple viable approaches (local vs cloud, tech choices)

### Stage 3: Test Environment Setup
- **`docker-patterns`** — MANDATORY: container implementation
- **`infra-cloud`** — CONDITIONAL: if environment includes cloud components
- **Post-Implementation Chain** — MANDATORY: after `infra-devops-implementer` completes environment files

### Stage 4: Test Diagnosis
- No SP skills specific to this stage. IRON LAW skills apply universally.

### Stage 5: Test Implementation
- **`test-driven-development`** — IRON LAW: embedded in typed agents (not invoked separately)
- **`verification-before-completion`** — IRON LAW: after each test batch
- **Post-Implementation Chain** — MANDATORY: after each typed implementer completes test code
- **`systematic-debugging`** — IRON LAW: if tests fail unexpectedly

### Stage 6: E2E Validation
- **`systematic-debugging`** — IRON LAW: if E2E fails — structured diagnosis before proposing fixes
- **Post-Implementation Chain** — MANDATORY: for any code fixes dispatched during E2E

### Stage 7: CI Pipeline Implementation
- **`ci-cd-patterns`** — MANDATORY: CI pipeline design and workflow implementation
- **`dev-testing-strategy`** (pattern 06) — FALLBACK: if `ci-cd-patterns` skill doesn't exist yet
- **Post-Implementation Chain** — MANDATORY: after `infra-devops-implementer` completes CI workflows

### Stage 8: Production Readiness Gate
- **`verification-before-completion`** — MANDATORY: final verification before verdict
- **`owasp-security`** — CONDITIONAL: if security-sensitive paths not covered in prior stages

---

## DEPLOYMENT PHASE — Skill Invocations

> **Scope**: These bindings apply when the AI-DLC workflow enters the DEPLOYMENT phase (after VERIFICATION completes). Generic SP triggers are OVERRIDDEN by these bindings per the SP Skill Invocation Override rule in CLAUDE.md.

### Stage 1: Deployment Strategy Design
- **`infra-cloud`** — CONDITIONAL (cloud-native stacks): applicability check for cloud-native deployment strategies
- No other skill bindings — stage is rule-driven

### Stage 2: CD Pipeline Diagnosis
- None — inventory + gap analysis is rule-driven

### Stage 3: CD Pipeline Design
- **`ci-cd-patterns`** — MANDATORY: CD pattern library for pipeline design decisions
- **`infra-cloud`** — CONDITIONAL (cloud-native stacks)

### Stage 4: CD Pipeline Implementation
- **`docker-patterns`** — CONDITIONAL (container-based deployments)
- **Post-Implementation Chain** — MANDATORY: `superpowers:verification-before-completion` → `infra-devops-reviewer` → `security-reviewer` (conditional on security-sensitive paths)
- TDD embedded via typed agent IRON LAW

### Stage 5: Deployment Execution
- **`superpowers:systematic-debugging`** — IRON LAW (universal, fires on any deploy anomaly)
- **`superpowers:root-cause-discipline`** — CONDITIONAL (fires on repeat failure patterns)

### Stage 6: Post-Deploy Validation
- None — informational stage

### Stage 7: Rollback Validation + Sign-off Gate
- **`superpowers:verification-before-completion`** — MANDATORY at rollback validation step

---

## META — writing-skills

- **`writing-skills`** — CONDITIONAL: repeatable emerging pattern (2+ appearances) or non-obvious high-impact knowledge. Project profiles may list specific candidates.

---

## Skills NOT Invoked in This Workflow

| Skill | Reason |
|---|---|
| `executing-plans` | Superseded by `subagent-driven-development` (superior: same session, no batching) |
| `receiving-code-review` | Internal to the `subagent-driven-development` loops |

---

## Reglas de Activación (tabla consolidada)

### MANDATORY (no hay sesión de construcción sin ellos)

| ID | SP skill | Trigger |
|---|---|---|
| R-01 | `using-git-worktrees` | Inicio de Code Generation de cualquier unidad |
| R-02 | `writing-plans` | AI-DLC abre Code Generation Part 1 |
| R-03 | `subagent-driven-development` | AI-DLC abre Code Generation Part 2 |
| R-04 | `test-driven-development` | Dentro de cada subagente implementador (IRON LAW) |
| R-05 | `verification-before-completion` | Antes de cada approval gate y antes de marcar cualquier tarea completa (IRON LAW) |
| R-06 | `finishing-a-development-branch` | Al completar Build and Test de cada unidad |
| R-07 | `systematic-debugging` | Cualquier falla, cualquier error inesperado, en cualquier fase (IRON LAW) |
| R-08 | Post-Implementation Chain | Después de que cualquier typed implementer completa (IRON LAW) |
| R-08b | `security-reviewer` (dentro de R-08) | Condicional dentro del chain: solo si archivos cambiados matchean trigger criteria (ver owasp-security skill) |
| R-13 | `dev-testing-strategy` | VERIFICATION Stage 1 — Test Strategy Design |
| R-14 | `docker-patterns` | VERIFICATION Stages 2-3 — Test Environment Design and Setup |
| R-15 | `infra-cloud` | VERIFICATION Stage 2 — Test Environment Design reconciliation |
| R-16 | `ci-cd-patterns` | VERIFICATION Stage 7 — CI Pipeline Implementation |

### CONDITIONAL

| ID | SP skill | Trigger |
|---|---|---|
| R-09 | `brainstorming` | Decisión ambigua con múltiples enfoques viables en cualquier etapa de diseño |
| R-10 | `dispatching-parallel-agents` | 2+ tareas o fallas independientes sin estado compartido |
| R-11 | `requesting-code-review` | Final de Build and Test antes del sign-off (review adicional al del chain) |
| R-12 | `writing-skills` | Patrón repetible emergente (2+ apariciones) o conocimiento no-obvio de alto impacto |
| R-17 | `infra-cloud` | VERIFICATION Stage 3 — si el environment incluye componentes cloud |
| R-18 | `owasp-security` | VERIFICATION Stage 8 — si security-sensitive paths no cubiertos en stages previos |

### Skills que NO aplican

| SP skill | Razón |
|---|---|
| `executing-plans` | Reemplazado por `subagent-driven-development` |
| `receiving-code-review` | Interno a los loops de `subagent-driven-development` |

---

## Extensions Integration

AI-DLC v0.1.7 introduces **extensions** (opt-in/opt-out modules loaded at Requirements Analysis). Extensions are AI-DLC constraints — they do NOT replace SP skills nor introduce new ones.

### How extensions interact with SP

| Extension | Interaction with SP | Notes |
|---|---|---|
| **security-baseline** | Adds validation constraints to Code Generation and Build & Test | Typed implementers enforce security rules as part of their IRON LAW self-review. No new SP skill needed — `verification-before-completion` validates compliance. |
| **property-based-testing** | Extends TDD scope inside typed implementers | When enabled, the TDD IRON LAW in typed implementers includes property-based tests alongside example-based tests. No separate SP skill — the implementer reads the extension rules. |
| _(future extensions)_ | Same pattern: AI-DLC constraint → typed implementer enforces | If an extension requires a NEW SP skill (e.g., a dedicated security audit skill), add it to the IRON LAW or Reglas de Activación tables above. |

### Principle

Extensions are **constraints on execution quality**, not new execution modes. SP skills remain the same — extensions raise the bar that typed implementers must clear during their IRON LAW cycle.

### When to escalate

If an extension introduces requirements that cannot be satisfied by existing typed implementers or SP skills:
1. Document the gap in the extension's rule file
2. Evaluate if a new typed agent or SP skill is needed (per Extensibility in `typed-agent-strategy.md` §7)
3. Do NOT silently skip extension rules — they are blocking constraints when enabled

---

## Mid-Workflow Changes — SP Integration

When a mid-workflow change occurs, AI-DLC executes the protocol in `common/workflow-changes.md` (archive, update state, adjust plan). **After AI-DLC resolves the meta-level**, apply the SP rules below to adjust skill invocations.

### 1. Adding a Skipped Stage

After AI-DLC adds the stage to the execution plan:
- **Evaluate SP bindings** for the added stage using the INCEPTION/CONSTRUCTION binding tables above
- If the stage has CONDITIONAL bindings (e.g., Application Design → `brainstorming`), evaluate the condition — do NOT auto-skip because the stage was originally skipped
- If the stage is in Construction and Code Generation has already run for the unit, the added stage's output must feed back into a **plan revision** (`writing-plans` re-invoked for affected unit)

### 2. Skipping a Planned Stage

After AI-DLC marks the stage as SKIPPED:
- **Archive SP artifacts** if the stage had already produced them (e.g., `brainstorming-spec.md`)
- **Assess downstream impact**: if a skipped stage had a `brainstorming` binding that was about to fire, note in audit.md: "SP brainstorming skipped at [stage] — design decision not formally explored"
- **No SP skill cleanup needed** if stage hadn't started — bindings simply don't fire

### 3. Restarting Current Stage

After AI-DLC archives and resets the stage:
- **Archive SP artifacts** for this stage (brainstorming-spec.md, plan files if Code Gen Part 1)
- **Re-evaluate SP bindings** from scratch — CONDITIONAL skills may trigger differently with new context
- If restarting Code Generation Part 2: `subagent-driven-development` starts fresh (new subagents, no carry-over from previous attempt)
- If the stage had an active worktree: `finishing-a-development-branch` to clean up, then `using-git-worktrees` for new attempt

### 4. Restarting Previous Stage (cascade)

After AI-DLC identifies cascade and resets all affected stages:
- **Archive ALL SP artifacts** for ALL affected stages in the cascade
- **SP skills re-fire for each stage** as if executing for the first time — use the binding tables above
- **Worktree management**: if any unit in the cascade had an active worktree, close it (`finishing-a-development-branch`), then create new worktree for re-execution
- **Post-Implementation Chain resets**: any completed chain for affected units is invalidated — must re-run after new Code Generation

### 5. Changing Stage Depth

After AI-DLC updates the depth level:
- **Re-evaluate CONDITIONAL bindings**: depth changes can affect whether a skill triggers
  - `brainstorming` is MORE likely at Comprehensive depth (more viable approaches surface)
  - `dispatching-parallel-agents` is MORE likely at Comprehensive depth (more independent concerns)
- **No re-invocation needed** if changing depth of a future stage — bindings evaluate naturally when the stage executes
- **Re-invocation needed** if changing depth of current in-progress stage AND a CONDITIONAL skill should now fire that didn't before

### 6. Pausing and Resuming Workflow

After AI-DLC logs the pause point:
- **No SP action needed on pause** — SP skills are stateless, state is in AI-DLC artifacts

On resume (new session):
- **Domain skill invocation MUST re-fire** — it is session-scoped (IRON LAW). The domain skill for the active unit's domain must be invoked before any code edit or Agent dispatch
- **Re-load `superpowers-integration.md`** — it's in the auto-load list, but verify it's in context before continuing execution
- **Worktree state check**: if a worktree was active at pause, verify it still exists and is on the correct branch before resuming Code Generation

### 7. Changing Architectural Decision

After AI-DLC identifies impact and user confirms restart:
- **`brainstorming` is MANDATORY** (not CONDITIONAL) for the architectural re-decision — this is a design decision with multiple viable approaches by definition
- **All SP artifacts for affected stages cascade**: archive brainstorming-specs, plans, code review reports for all affected units
- **Worktrees for affected units**: close all (`finishing-a-development-branch`), create new ones on re-execution
- Output of brainstorming: `aidlc-docs/inception/application-design/brainstorming-spec.md` (overwrite archived version)

### 8. Adding/Removing Units

After AI-DLC updates unit artifacts:

**Adding a unit**:
- New unit gets FULL SP treatment: `using-git-worktrees` → SP bindings per stage → `writing-plans` → `subagent-driven-development` → Post-Implementation Chain → `finishing-a-development-branch`
- Existing units: no SP change unless dependencies are affected (in which case, re-evaluate their plans)

**Removing a unit**:
- Close the unit's worktree if active (`finishing-a-development-branch`)
- SP artifacts in `aidlc-docs/construction/{unit-name}/` are archived by AI-DLC — no additional SP action
- If removed unit had completed Post-Implementation Chain, its commits remain (revert is a separate REVERT PATH via core-change-flow-protocol.md)

**Splitting a unit**:
- Treat as: remove original + add 2 new units
- Both new units get full SP treatment
- Original unit's worktree and SP artifacts are archived

### Logging

All SP adjustments during mid-workflow changes MUST be logged in `audit.md` with:
```markdown
## SP Adjustment — [Change Type]
**Timestamp**: [ISO timestamp]
**Trigger**: [What workflow change occurred]
**SP artifacts archived**: [List of files]
**SP skills re-invoked**: [List of skills and stages]
**SP skills skipped**: [List with reason]
**Domain skill re-invocation**: [Yes/No — session-scoped check]
```

---

## Priority When Skills Conflict

1. Process skills first: `brainstorming`, `systematic-debugging`
2. Implementation skills after: `writing-plans`, `subagent-driven-development`
