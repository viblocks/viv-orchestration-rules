# CI Pipeline Implementation

**Purpose**: Implement CI/CD pipelines that automate the test suite so that DEPLOYMENT can gate on them. Without this, tests exist but have no enforcement.

## Prerequisites
- Complete test suite passing (Stages 4-6)
- Test Environment Definition (Stage 3)
- Existing CI workflows (if brownfield)

---

## Step 1: CI Inventory (Brownfield)

If existing CI workflows exist, analyze them:

```bash
# Find CI workflow files
ls -la .github/workflows/
# Read each workflow
cat .github/workflows/*.yml
```

For each existing workflow:
- What tests does it run?
- What level are those tests? (L1? L2? All?)
- Does it use the test environment from Stage 3?
- What gates PR merge? What gates release?

**Output**: CI gap report — what's covered, what's missing, what needs updating.

**Skip if greenfield** — proceed to Step 1a.

---

## Step 1a: Script-Workflow Alignment (REQUIRED — greenfield AND brownfield)

Reconcile **declared quality mechanisms** against **enforcers actually running them**. The same audit performed in `reverse-engineering.md` Step 1.7 is repeated here against the current state of the codebase, because new declarations may have been added during INCEPTION/CONSTRUCTION and brownfield zombies may not have been resolved if RE was skipped or stale.

For every workspace `package.json` `scripts.*` entry (or platform equivalent: `pyproject.toml` script targets, `Cargo.toml` `[[bin]]` / aliases, `Makefile` targets, scripts under `scripts/`), require exactly one of:

- **ENFORCED** by ≥1 CI workflow step (cite workflow file + job name); OR
- **OPT-OUT** with the rationale template from `reverse-engineering.md` Step 1.7 recorded in an allow-list at `aidlc-docs/verification/script-workflow-allowlist.md`; OR
- **REMOVED** from the source declaration.

Zombies (declared, no enforcer, no OPT-OUT entry) MUST be resolved before this stage is marked complete. Severity is HIGH — non-admissible per `production-readiness-gate.md` §3.3.

### Mechanical enforcer (REQUIRED CI gate)

The audit IS a CI gate, not a procedural recommendation. The pipeline plan MUST include a check-invariants job that:

1. Enumerates declared mechanisms across the codebase (per-stack: `jq` on `package.json`, `python -m tomllib` on `pyproject.toml`, `make -pn` on `Makefile`, etc.).
2. For each, searches workflow files + the OPT-OUT allow-list for an enforcer match.
3. Fails the job (exit non-zero) on any zombie — declared mechanism with no enforcer and no allow-list entry.
4. Runs on every PR (cannot be skipped via path filters).

Each consumer ships their own implementation under `scripts/check-invariants.*` (per-stack — `.sh` for shell-driven Node/JS monorepos, `.py` for Python projects, etc.). The plugin does not provide a reference implementation because the per-stack inventory grammar varies (`package.json` `scripts.*` is JSON, `Cargo.toml` aliases are TOML, `Makefile` targets parse differently). The invariant is stack-independent; the implementation is consumer-specific.

### Output

Append to the CI gap report (Step 1) or create if greenfield:

```markdown
## Script-Workflow Alignment

| Mechanism | Status | Enforcer / OPT-OUT rationale / removal |
|---|---|---|
| <package.json script / Makefile target / etc.> | ENFORCED | <workflow + job, e.g. `.github/workflows/test-unit.yml::lint`> |
| <…> | OPT-OUT | See `aidlc-docs/verification/script-workflow-allowlist.md#<id>` |
| <…> | REMOVED | Removed from `<file>:<line>` per Step 1a; rationale: <reason> |

## check-invariants Implementation

- Path: `scripts/check-invariants.<ext>`
- CI job: `.github/workflows/<file>.yml::<job-name>`
- Runs on: every PR
- Exit code on zombie: non-zero (blocks merge)
```

This step would have caught the `viblocks-ai` zombies (`services/ui` `tsc --noEmit` and `services/{core,bot}` + `packages/shared` `eslint` × 3) during the first VERIFICATION phase after AI-DLC adoption. The fact that it didn't is the evidence backing this step — see issue #71 for the empirical case.

---

## Step 1b: Domain Hook

Invoke applicable skills BEFORE pipeline design:
- **`ci-cd-patterns`** — MANDATORY: CI pipeline design and workflow implementation
- **`dev-testing-strategy`** (pattern 06) — FALLBACK: if `ci-cd-patterns` skill doesn't exist yet

Integrate recommendations into the pipeline design (Step 2).

---

## Step 2: Pipeline Design

Design the CI pipeline architecture:

```markdown
On Commit (local + CI):
  +-- L1: Unit Tests

On PR (CI gate -- REQUIRED to merge):
  +-- L1: Unit Tests
  +-- L2: Integration Tests (Testcontainers or docker-compose)
  +-- L3: Replay / Golden Files (if applicable)
  +-- L4: Contract Tests (if applicable)
  +-- Performance Gate (if applicable)

On Merge to Main (CI gate -- REQUIRED for release):
  +-- All PR gates
  +-- L5: System Tests (Docker Compose full stack)

Nightly (scheduled):
  +-- All above
  +-- L6: E2E Browser (Playwright + real backend)
```

Adapt based on project reality — not all levels exist in every project.

### Required jobs (tool-agnostic)

The pipeline plan MUST include the following jobs regardless of chosen tooling:

- **Dependency scan** — any of `npm audit`, Trivy, Snyk, OWASP dep-check. Severity CRITICAL blocks.
- **Secrets scan** — any of `gitleaks`, TruffleHog. Any leak blocks.
- **Script-workflow alignment** (`check-invariants.<ext>`) — fails on any zombie (declared quality mechanism with no enforcer and no allow-list entry). See Step 1a. Runs on every PR; cannot be skipped via path filters.
- **Migration rollback validation** (only when change scope contains migrations): execute `up → down → up` in the test environment, regardless of migration tool (Prisma, TypeORM, Flyway, Alembic, raw SQL). Reversibility failure → FAIL.
- **Scope-claim validation**: if the verdict claims `migrations-rollback = N/A`, a CI step MUST verify the diff contains no migration files. Mismatch → FAIL with a schema error.

**Key principle**: Every CI job has a corresponding Makefile target. CI commands must match local commands (SS2 compliance from `dev-testing-strategy` pattern 06).

---

## Step 3: Workflow Implementation

**IRON LAW**: Dispatch `infra-devops-implementer` for all CI workflow implementation.

For each pipeline stage, create or update GitHub Actions workflows:

- `.github/workflows/test-unit.yml` — L1 tests, runs on every push
- `.github/workflows/test-integration.yml` — L2-L4 tests, runs on PR
- `.github/workflows/test-system.yml` — L5 tests, runs on merge to main
- `.github/workflows/test-e2e.yml` — L6 tests, runs nightly (scheduled)

Or consolidate into fewer workflows with job dependencies — depends on project preference.

**Typed agent prompt must include:**
- Pipeline design from Step 2
- Test environment files from Stage 3
- Existing workflows to extend (if brownfield)
- Makefile targets to reference
- Security: use SHA-pinned actions, not version tags

### Report artifacts + Post-Impl Chain

The `infra-devops-implementer` MUST ensure the workflows emit the following report artifacts (accessible from the verdict stage):

- `aidlc-docs/verification/security-scan-report.md` — dep + secrets scan output, severity summary, 0 CRITICAL proof.
- `aidlc-docs/verification/migration-rollback-report.md` — per-migration rollback log (or explicit `N/A — no migrations in diff`).

**MANDATORY: Execute Post-Implementation Chain after the implementer completes.** The chain and its 5-field output template are defined in `CLAUDE.md` → "Post-Implementation Chain (MANDATORY)" and "Post-Chain Output (MANDATORY)". Stage files MUST NOT redefine the template inline.

---

## Step 4: Makefile Targets

Ensure every CI job has a corresponding Makefile target:

```makefile
# Test targets (CI mirrors these exactly)
test-unit:          ## Run L1 unit tests
test-integration:   ## Run L2-L4 integration tests
test-system:        ## Run L5 system tests
test-e2e:           ## Run L6 E2E browser tests
test-all:           ## Run all test levels sequentially
```

**Output**: Updated Makefile with test targets.

---

## Step 5: Gate Configuration

Configure which jobs are required to pass:

| Gate | Required Jobs | When |
|---|---|---|
| PR merge | test-unit, test-integration | Every PR |
| Release | test-unit, test-integration, test-system | Merge to main |
| Nightly | All including E2E | Scheduled |

If the project uses branch protection rules, update them. Otherwise, document the intended gate configuration for manual enforcement.

---

## Step 6: CI Validation

Run the CI pipeline on the current branch to verify it works:

```bash
# Trigger CI locally or push to verify
git push origin HEAD
# Check CI status
gh run list --limit 5
gh run view <run-id>
```

Verify:
- All jobs start and complete
- Correct tests run in each job
- Jobs use the test environment from Stage 3
- Failed tests actually block the pipeline (not just warnings)

**If CI pipeline fails:**
1. Diagnose: read CI logs (`gh run view <run-id> --log-failed`), identify failing job and root cause
2. Fix: update workflow file, Makefile target, or test configuration as needed
3. Re-trigger: push fix and verify CI re-runs
4. **Escalation**: After 3 failed CI fix attempts, escalate to user with:
   - Failing job name and error output
   - What was attempted
   - Proposed alternatives (simplify pipeline, split into phases, address infra issue)
5. Do NOT advance to Stage 8 with a failing CI pipeline

---

## Step 7: Generate CI Pipeline Report

Create `aidlc-docs/verification/ci-pipeline-report.md`:

```markdown
## CI Pipeline Implementation Report

### Pipeline Architecture
[Pipeline design from Step 2]

### Workflows Created/Updated
| File | Trigger | Tests | Gate |
|---|---|---|---|
| .github/workflows/[name].yml | [push/PR/merge/nightly] | [L1/L2/...] | [PR merge / release / none] |

### Report Artifacts
| Artifact | Status |
|---|---|
| aidlc-docs/verification/security-scan-report.md | present / N/A |
| aidlc-docs/verification/migration-rollback-report.md | present / N/A |

### Makefile Targets
| Target | Command | Maps to CI job |
|---|---|---|
| test-unit | pnpm --filter ... run test | test-unit.yml |

### CI Validation
- Pipeline run: [link to CI run]
- Result: [all jobs pass / specific failures]

### Gate Configuration
| Gate | Required | Configured |
|---|---|---|
| PR merge | [jobs] | [yes/no/manual] |
| Release | [jobs] | [yes/no/manual] |
```

---

## Step 8: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark CI Pipeline Implementation as in-progress/complete

---

## Step 9: Present Completion Message

1. **Completion Announcement**:

```markdown
# 🔄 CI Pipeline Implementation Complete
```

2. **AI Summary**: Structured bullet points:
   - Workflows created/updated: N
   - Test levels automated: L1-LN
   - Gates configured: PR merge, release, nightly
   - CI validation result
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the CI pipeline report at: `aidlc-docs/verification/ci-pipeline-report.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the CI pipeline based on your review
> ✅ **Continue to Next Stage** - Approve CI pipeline and proceed to **Production Readiness Gate**

---
```

## Step 10: Wait for Explicit Approval
- Gate: CI pipeline runs successfully on current branch. All test levels execute in their designated stage.

## Step 11: Record Approval and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## CI Pipeline Implementation
**Timestamp**: [ISO timestamp]
**Status**: [Approved/Complete]
**Key Results**: Workflows created/updated: N; Levels automated: L1-LN; CI validation: PASS/FAIL
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- Mark CI Pipeline Implementation as complete in aidlc-state.md
