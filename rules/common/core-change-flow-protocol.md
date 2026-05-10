# Core Change Flow Protocol (AI-DLC + Superpowers)

**Purpose**: Operative rules for the **core change flow protocol** — shared by two distinct flows that reuse it:

- **AI-DLC Change Flow** (dev-time, interactive) — trigger: "basado en ai dlc change flow issue #XXX". Aplica durante Inception → Construction → Verification → Deployment.
- **Issue-Driven Change Flow** (post-production, autonomous) — trigger: auto-dispatch via runner or label. Consumer projects configure their own trigger mechanism.

Este protocolo define QUÉ hacer con un cambio (triage, paths, agents, chain). Los dos flujos solo difieren en el trigger y el execution environment.

**PRECEDENCE RULE**: During change flow execution (either flow), THIS FILE is the sole authority for SP skill invocation. Generic SP skill triggers (from `using-superpowers`, `brainstorming`, `test-driven-development`, etc.) are OVERRIDDEN by the path protocols and IRON LAW below. A skill not listed in the path protocol or IRON LAW section = do NOT invoke it, regardless of what the skill's own trigger says. See CLAUDE.md "SP Skill Invocation Override" for the full override table.

**Triggers**:
- AI-DLC flow: user prompt references a GitHub issue (e.g. "issue #XXX", "change flow issue #XXX")
- Issue-Driven flow: label `ready-for-agent` aplicado al issue (por triager automático o humano)

**Batch trigger** (aplica solo al AI-DLC flow): Multiple issues (e.g. "issues #XXX, #YYY, #ZZZ") → triage each individually, detect file conflicts, group into parallel/sequential batches, execute, close each with individual comment.

---

## One-PR-Per-Issue Rule (MANDATORY)

**Un issue Linear = una rama = un PR. Sin excepciones.**

| Rule | Detail |
|---|---|
| One branch per issue | Branch name: `claude/VI-XXX` — never reuse a branch for a second issue |
| One PR per issue | PR title/body references exactly one Linear issue (`Fixes VI-XXX`) |
| Rework isolation | If rework is needed after a PR is ready-for-review, approved, or merged: open a NEW issue + NEW branch + NEW PR. Do NOT add commits to the previous branch |
| Rework linkage | The rework PR links to the NEW issue (e.g., `VI-118`), not the original (`VI-117`) |
| No piggyback commits | A second issue's commits must NEVER appear in the branch of a first issue that is already approved or merged |

**Why**: One branch/PR per issue guarantees: (a) atomic revertibility — `git revert` of a PR undoes exactly one issue, (b) traceability — PR title = issue title = commit context, (c) no silent scope creep in open PRs.

**Enforcement gate** (CI): The PR body must contain a single `Fixes VI-\d+` reference. Multiple issue references in one PR body → CI warning (grace period) → CI block (post-grace).

---

## Batch Protocol

When multiple issues arrive:
1. **Triage each** individually (Q1-Q4)
2. **Detect file conflicts** — issues touching same files → sequential group
3. **Group**: independent issues → parallel (`dispatching-parallel-agents`), conflicting → sequential. DESIGN/CROSS-DOMAIN → always sequential
4. **Execute + close** each issue individually. Issues sharing files may share commits but each gets its own close comment

---

## Triage Gate

Evaluate these 4 questions IN ORDER. First match determines the path.

| # | Question | YES → Path |
|---|---|---|
| Q1 | Introduces new abstraction? (new event type, aggregate, entity, schema, concept not in codebase) | DESIGN PATH |
| Q2 | Crosses service boundaries? (2+ services, or shared/contracts with downstream impact) | CROSS-DOMAIN PATH |
| Q3 | Reverts a previous implementation? | REVERT PATH |
| Q4 | Bug fix, audit remediation, or improvement scoped to 1 service/module? | DIRECT PATH |

**Precedence**: Q1 > Q2 > Q3 > Q4. If uncertain between DIRECT and DESIGN, escalate to DESIGN.

**Project override (viblocks-ai)**: Any issue that modifies on-chain event semantics in `services/core/src/chain/` or `services/core/src/enrichment/` → DESIGN PATH mandatory, even if not explicitly a "new abstraction".

---

## Modes

| Mode | Paths | Human interaction |
|---|---|---|
| **AUTONOMOUS** | DIRECT, REVERT | None — agent completes end-to-end |
| **SUPERVISED** | DESIGN, CROSS-DOMAIN | DESIGN: approval gate on brainstorming. CROSS-DOMAIN: plan visible before execution |

---

## Path Protocols

### DIRECT PATH (autonomous)

1. Triage → classify DIRECT
2. Domain detection → routing table → typed agent
3. Invoke domain skill if applicable
4. Dispatch typed implementer (issue = spec, IRON LAW applies)
5. Post-Implementation Chain (verification + typed code reviewer)
6. Create PR on dedicated branch `claude/VI-XXX` (One-PR-Per-Issue Rule): `gh pr create` → report PR URL to user
7. Close: `gh issue close --comment` with triage + evidence (include PR URL in Commits field)

### CROSS-DOMAIN PATH (supervised)

1. Triage → classify CROSS-DOMAIN
2. Invoke `writing-plans` → divide by domain (shared/contracts = Task 1)
3. Execute per task → typed implementer per domain
4. Tasks without dependencies → parallel (`dispatching-parallel-agents`)
5. Post-Implementation Chain per task
6. Integration verification (full build, E2E if relevant)
7. Create PR: `gh pr create` → report PR URL to user
8. Close: commit(s) + close comment with triage + tasks + evidence (include PR URL)

### DESIGN PATH (supervised)

1. Triage → classify DESIGN
2. Invoke `brainstorming` → explore 2-3 approaches → APPROVAL GATE
3. **Design System Audit** (MANDATORY before writing-plans):
   - Determine audit scope from the consumer project's `.claude/routing/routing-table.json`:
     the scope is the union of `paths` for the domain(s) relevant to the issue (matched
     via the same domain-detection logic the routing enforcement hook uses). The audit
     globs/greps inside that scope only — never against hardcoded directory layouts.
   - If the issue spans multiple domains (CROSS-DOMAIN that escalated to DESIGN), audit
     the union of all affected domains' `paths`.
   - Examples (illustrative — actual scope comes from `routing-table.json`):
     - viblocks-style layout: `services/ui/src/components/**`, `services/core/src/**`, `packages/shared/src/**`
     - monorepo with `apps/`+`libs/`: `apps/<app>/src/**`, `libs/<lib>/src/**`
     - flat `src/` layout: `src/<domain>/**`
   - Add a **"Componentes existentes evaluados"** table to the brainstorming spec:
     | Component/Service | Reused? | Rationale |
     |---|---|---|
     | ExistingFoo | Yes/No | why |
   - Approval gate: this table MUST reflect a real audit of the resolved scope. If the
     scope yields zero candidates, state that explicitly with one row "(none found in
     scope `<paths>`)" — an absent table or a table with no audit evidence still blocks
     the gate.
4. Save design spec (with audit table) to `aidlc-docs/construction/issue-{N}/brainstorming-spec.md`
5. Invoke `writing-plans` with approved design
6. Execute like DIRECT or CROSS-DOMAIN depending on scope
7. Post-Implementation Chain
8. Create PR: `gh pr create` → **PAUSE: notify user with PR URL before closing** (user may want to review implementation against approved design)
9. Evaluate skill update (`writing-skills` if non-obvious domain knowledge)
10. Close: commit(s) + close comment with triage + design decision + evidence (include PR URL)
    - Evidence MUST include `**Design spec**: aidlc-docs/construction/issue-{N}/brainstorming-spec.md`

### REVERT PATH (autonomous)

1. Triage → classify REVERT
2. Impact analysis: read original commits, identify ALL files, check downstream deps
3. If clean revert: git revert or typed implementer. If downstream deps: escalate to CROSS-DOMAIN
4. Residue check: no orphaned imports, types, tests, docs
5. Post-Implementation Chain
6. Create PR: `gh pr create` → report PR URL to user
7. Close: close comment with triage + original issue + files reverted (include PR URL)

---

## Escalation Rules

Agent MUST escalate to user if:
- Triage ambiguous
- Tests fail after 2 fix attempts
- Code reviewer reports CRITICAL issue
- Revert has non-trivial downstream deps
- Fix requires paths outside issue scope

---

## Spec-Driven Artifact Rule

| Design decision needed? | Store | aidlc-docs/ artifacts |
|---|---|---|
| No (DIRECT, REVERT) | GitHub issue | None |
| No (CROSS-DOMAIN simple) | GitHub issue + close comment | None |
| Yes (DESIGN, CROSS-DOMAIN with design) | `aidlc-docs/construction/issue-{N}/` | `brainstorming-spec.md`, `plan.md` |

---

## Close Comment Format

```
## Triage
**Path**: [DIRECT | CROSS-DOMAIN | DESIGN | REVERT]
**Reason**: [one line]
**Typed agent**: [agent(s)]

## Execution
[varies by path]

## Evidence
**Verification**: tests + build [pass/fail]
**Code review**: [reviewer agent] [result — approved or issues found]
**Security review**: [security-reviewer — PASS/FAIL/N/A] [findings summary or "no security-sensitive paths"]
**Commits**: [hash(es)]
**PR**: [URL]
```

**MANDATORY**: All 5 Evidence fields are required. Do NOT close an issue without:
- `Verification` showing explicit pass/fail for tests and build
- `Code review` naming the reviewer agent used and its result
- `Security review` naming the result or N/A with reason (see owasp-security skill for trigger criteria)
- If no typed reviewer applies (e.g. scripts, docs), state: "N/A — [reason]"
- `PR` with the URL — DESIGN path pauses here for user confirmation before closing

---

## Skill Update Loop

After close: did the fix reveal non-obvious domain knowledge?
- No → done
- Yes, already in skill → done
- Yes, 1st occurrence → note in close comment: "Candidate for skill update"
- Yes, 2nd+ occurrence → invoke `writing-skills`, update skill, commit

In autonomous mode: register candidate only. Do not invoke writing-skills autonomously.

---

## IRON LAW (all paths)

| Skill | When |
|---|---|
| `test-driven-development` | Inside every typed implementer |
| `verification-before-completion` | Before every close |
| `systematic-debugging` | Any failure, before proposing fix |
| `root-cause-discipline` | Complex/layered/env-dependent symptoms — run the four-questions gate before acting on any candidate fix (complements `systematic-debugging`) |
| Domain skill invocation | Before first dispatch |

- **UI changes update spec**: if any modified file overlaps the ui-domain
  paths declared in `.claude/routing/routing-table.json` (domains with
  `ui_bearing: true`), the corresponding state matrix rows + data bindings +
  snapshot test contracts in
  `aidlc-docs/construction/{unit-name}/functional-design/frontend-components.md`
  (per-unit) or `aidlc-docs/screens/{screen-name}.md` (brownfield-without-units
  fallback) MUST be updated in the same commit. Pre-existing gaps in the matrix
  for unaffected mandatory states are tracked in
  `aidlc-docs/spec-coverage-debt.md`, not enforced in this change.
  See `rules/common/frontend-change-discipline.md` for delta semantics,
  refactor declaration (DESIGN path only — refactors entering via
  DIRECT/CROSS-DOMAIN MUST escalate to DESIGN), unknown-row escape hatch,
  and REVERT path handling.

---

## Phase Re-Entry Rules

### DEPLOYMENT Phase Re-Entry

DEPLOYMENT is one-time establishment. Subsequent releases use the pipeline without re-entering the phase. Re-entry is triggered by deployment topology changes:

| Trigger | Stages re-executed | Change Flow path |
|---|---|---|
| Pipeline-code-only change (workflow file edit, Makefile target change) | Stages 3 + 4 only | DESIGN |
| Deployment-strategy change (canary↔blue/green, add staging env) | Stages 1 + 3 + 4 + 5 | DESIGN |
| Production-topology change (new region, new cluster, new service added) | Full re-entry Stages 1–7 | DESIGN (or CROSS-DOMAIN if change spans services) |

Re-entry creates a new execution in `aidlc-state.md` under `### DEPLOYMENT PHASE` with the re-executing stages marked `[~]`.
