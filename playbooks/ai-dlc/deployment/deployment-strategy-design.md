# Deployment Strategy Design

**Purpose**: Decide deployment strategy and pre-prod mechanism before designing the pipeline. Also verify that observability inputs exist; block phase execution if they do not.

**Stage type**: DESIGN (user judgment shapes outputs — requires Question DNA)

## Prerequisites
- VERIFICATION phase complete — `aidlc-docs/verification/production-readiness-verdict.md` exists with verdict GO or CONDITIONAL-GO (blocking)
- `aidlc-docs/verification/production-infrastructure-blueprint.md` must exist (blocking)
- `aidlc-docs/verification/ci-pipeline-report.md` must exist (blocking)
- `aidlc-docs/construction/build-and-test/build-instructions.md` must exist (blocking)
- NFR design docs per-unit recommended (provides strategy rationale)
- RE `technology-stack.md` (if brownfield) recommended

---

## Step 0: Phase Bootstrap

**MANDATORY**: If this is the first DEPLOYMENT execution for this project:

1. Create section in `aidlc-docs/aidlc-state.md`:
   ```markdown
   ### DEPLOYMENT PHASE
   - [~] Stage 1: Deployment Strategy Design (IN PROGRESS)
   - [ ] Stage 2: CD Pipeline Diagnosis (CONDITIONAL — brownfield)
   - [ ] Stage 3: CD Pipeline Design
   - [ ] Stage 4: CD Pipeline Implementation
   - [ ] Stage 5: Deployment Execution
   - [ ] Stage 6: Post-Deploy Validation
   - [ ] Stage 7: Rollback Validation + Sign-off Gate
   ```
2. Create directory `aidlc-docs/deployment/` with `plans/` subdirectory.
3. Log bootstrap in `aidlc-docs/audit.md`.

---

## Step 1: Load Inputs

Read all prior phase artifacts (cumulative per session-continuity):
- All Inception artifacts
- All Construction artifacts
- All Verification artifacts

**MANDATORY**: Verify the 4 blocking prerequisites exist. If any missing, abort with explicit error listing the missing artifacts and escalate to user.

---

## Step 2: Observability Readiness Input Check

**MANDATORY**: Verify three conditions before proceeding:

| Condition | How to verify |
|---|---|
| (a) NFR Design declares observability requirements | Read per-unit `aidlc-docs/construction/{unit}/nfr-design/` — must contain an observability section with declared metrics/logs/traces |
| (b) Generated code instruments the required signals | Inspect code (spot check based on NFR Design claims) |
| (c) Infrastructure Design provisions the observability stack | Read per-unit `aidlc-docs/construction/{unit}/infrastructure-design/` — must reference an observability backend as IaC |

If any condition fails, **BLOCK phase execution** and escalate:
- (a) missing → escalate back to NFR Requirements / NFR Design
- (b) missing → escalate back to Code Generation
- (c) missing → escalate back to Infrastructure Design

Output: `aidlc-docs/deployment/plans/observability-readiness-report.md` (PASS / BLOCK with gap list).

---

## Step 3: Create Plan File

Save to `aidlc-docs/deployment/plans/deployment-strategy-design-plan.md`:

```markdown
# Deployment Strategy Design Plan

## Execution Steps
- [ ] Step 1: Load Inputs
- [ ] Step 2: Observability Readiness Input Check
- [ ] Step 3: Service shape classification
- [ ] Step 4: Strategy selection
- [ ] Step 5: Pre-prod mechanism selection
- [ ] Step 6: Rollback strategy selection
- [ ] Step 7: Release cadence declaration
- [ ] Step 8: Extension compliance check
- [ ] Step 9: Generate deployment-strategy.md artifact

## Questions

### Question 1: Service shape classification
[question about stateless/stateful/traffic pattern — see Question DNA below]
```

---

## Step 4: Question DNA (MANDATORY — 7-step micro-pattern)

**DIRECTIVE**: Thoroughly analyze loaded artifacts to identify ALL areas where clarification would improve deployment strategy quality.

**CRITICAL**: Default to asking questions when there is ANY ambiguity or missing detail.

**MANDATORY**: Evaluate ALL of the following categories by asking targeted questions. For each category, determine applicability based on evidence from artifacts — do not skip categories without explicit justification.

Categories (minimum 5, maximum 8):

1. **Service shape** — stateless vs stateful, traffic pattern, cold-start tolerance, data migration constraints
2. **Deployment strategy preference** — blue/green, canary, rolling, recreate, preview-branch, serverless
3. **Pre-prod mechanism** — staging env, canary slice, shadow traffic, preview branches
4. **Rollback strategy** — version-pin revert, artifact revert, snapshot restore, feature flag toggle
5. **Release cadence expected** — on-merge, on-tag, scheduled, manual
6. **Extensions active** — security-baseline, compliance, etc.

Question file format per `_common/question-format-guide.md`:
- NEVER ask in chat — save to `.md` file
- Multiple choice A, B, C, D + X) Other (MANDATORY as last option)
- `[Answer]:` tag after each question
- Minimum 2 mutually exclusive options + Other

Save to `aidlc-docs/deployment/plans/deployment-strategy-design-questions.md`.

Wait for user to complete all `[Answer]:` tags before proceeding.

**After answers received**: evaluate all responses for vague signals ("depends", "maybe", "not sure", "mix of"). If ambiguity detected, create `aidlc-docs/deployment/plans/deployment-strategy-design-clarification-questions.md`. Resolve before proceeding.

---

## Step 5: Generate Strategy Artifact

Based on answers, generate `aidlc-docs/deployment/deployment-strategy.md` covering:
- Service shape profile
- Deployment strategy selected (with rationale)
- Pre-prod mechanism selected (with rationale)
- Rollback strategy selected
- Release cadence
- Extension compliance summary

---

## Step 6: Completion Message

Present the 3-part completion message per `stage-structural-patterns.md` Pattern 2.4:

### Part 1 — Announcement

```markdown
# 🟡 Deployment Strategy Design Complete
```

### Part 2 — AI Summary

Structured bullet summary of: strategy chosen, pre-prod mechanism, rollback strategy, release cadence, extension compliance status.

### Part 3 — Workflow Message

```markdown
> **📋 REVIEW REQUIRED:**
> Please examine the strategy artifact at: `aidlc-docs/deployment/deployment-strategy.md`

> **🚀 WHAT'S NEXT?**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the deployment strategy
> ✅ **Continue to Next Stage** - Approve Strategy Design and proceed to **CD Pipeline Diagnosis** (if brownfield) or **CD Pipeline Design** (if greenfield)

---
```

---

## Step 7: Approval Gate

Wait for explicit user approval. DO NOT PROCEED until confirmed. Record approval in `audit.md` with timestamp and raw user input. Update `aidlc-state.md` to mark Stage 1 complete.

---

## Escalation

If the Observability Readiness Input Check blocks the phase, escalate per `_common/error-handling.md` with:
- Which condition failed (a/b/c)
- Which upstream phase owns the gap
- Recommended recovery path

---

## Extension Compliance Hooks

Evaluate enabled extensions from `aidlc-state.md` Extension Configuration:
- `security-baseline` — applicable (secret handling in strategy artifact): verify no secrets inline, all via Parameter Store/Secrets Manager references
- Other extensions — evaluate applicability per stage purpose

Include compliance summary in Part 2 of completion message.
