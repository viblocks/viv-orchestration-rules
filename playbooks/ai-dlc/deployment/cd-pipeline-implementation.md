# CD Pipeline Implementation

**Purpose**: Materialize the Stage 3 pipeline design as code via typed agent dispatch.

**Stage type**: EXECUTION (complex multi-step, typed agent dispatch, Post-Implementation Chain)

## Prerequisites
- Stage 3 CD Pipeline Design complete with approved design artifact (blocking)
- `infra-devops-implementer` typed agent available (blocking per IRON LAW routing)

---

## Step 0: Stage Bootstrap

**MANDATORY**: Mark Stage 4 as in-progress in `aidlc-docs/aidlc-state.md` (`[~]` marker on Stage 4 row). Log stage start in `aidlc-docs/audit.md` with ISO 8601 timestamp.

---

## Step 1: Create Execution Plan

Save to `aidlc-docs/deployment/plans/cd-pipeline-implementation-plan.md`:

```markdown
# CD Pipeline Implementation Plan

## Execution Steps
- [ ] Step 1: Load Stage 3 design artifact
- [ ] Step 2: Dispatch typed implementer for workflow files
- [ ] Step 3: Post-Implementation Chain (verification + reviewer + security)
- [ ] Step 4: Sandwich on failure (max 3 cycles)
- [ ] Step 5: Generate impl report
- [ ] Step 6: Completion message + approval
```

---

## Step 2: Dispatch Typed Implementer

**IRON LAW** (per CLAUDE.md routing table): paths touched are Class A (`.github/workflows/**`, `scripts/**`, `Makefile`, `Dockerfile*`, `docker-compose*`). Main session CANNOT edit. Dispatch via `Agent` tool:

- **Agent**: `infra-devops-implementer`
- **Context**: Full `cd-pipeline-design.md` content + spec file path reference
- **Activities**:
  - Write pre-prod deploy workflow
  - Write prod deploy workflow
  - Write rollback workflow/script
  - Wire secret references via Parameter Store / Secrets Manager (NEVER values)
  - Add Makefile targets (`make deploy-preprod`, `make deploy-prod`, `make rollback`)
  - Configure prod approval gate mechanism per Stage 3 Step 6
  - Configure concurrency control per Stage 3 Step 7
  - Implement smoke test suite per Stage 3 Step 12
  - Update README with CD pointers

---

## Step 3: Post-Implementation Chain (MANDATORY per `rules/common/post-implementation-chain.md`)

Run in order:

1. **`superpowers:verification-before-completion`** — workflow syntax validates (`actionlint` if available), Makefile targets lint, smoke tests dry-run succeed.
2. **`infra-devops-reviewer`** — typed code review.
3. **`security-reviewer`** — TRIGGERED IF changes touch secrets config, auth, CORS, new dependencies, privileged infra ops. (Per `common/owasp-security` skill criteria.)
4. On PASS: commit automatic with descriptive message + spec reference.

If any gate returns CRITICAL/HIGH findings → return to Step 2 with findings as input.

---

## Step 4: Escalation

Per Pattern 3.4 (EXECUTION stages):

If implementation fails after 3 sandwich cycles:
1. Apply `systematic-debugging` IRON LAW
2. Escalate to user with:
   - What was attempted
   - What failed and why
   - Proposed solutions
3. Do NOT advance to Stage 5 with broken pipeline.

---

## Step 5: Generate Impl Report

Generate `aidlc-docs/deployment/cd-pipeline-impl-report.md` summarizing:
- Files created/modified (list from agent result)
- Post-Implementation Chain gate results
- Any deviations from Stage 3 design (with justification)

---

## Step 6: Completion Message + Approval Gate

3-part per Pattern 2.4. Next stage: **Deployment Execution**. 2-option approval gate.

**MANDATORY**: Log the approval prompt in `aidlc-docs/audit.md` BEFORE presenting to user. On approval, log the user's raw response with timestamp. Mark Stage 4 complete (`[x]`) in `aidlc-docs/aidlc-state.md` with date and one-line summary.

---

## Extension Compliance Hooks

- `security-baseline`: enforced by security-reviewer in Post-Implementation Chain.
- Other extensions: evaluate applicability.
