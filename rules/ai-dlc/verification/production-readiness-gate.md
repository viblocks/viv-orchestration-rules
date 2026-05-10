# Production Readiness Gate

**Purpose**: Emit the final GO/NO-GO verdict with all accumulated evidence. This is the last gate before DEPLOYMENT.

## Prerequisites
- All previous VERIFICATION stages complete
- CI Pipeline running (Stage 7)
- E2E Validation passed (Stage 6)

---

## Step 1: Evidence Collection

Compile results from ALL previous stages into a single evidence package:

| Source | What to collect |
|---|---|
| Stage 1 (Test Strategy) | Total scenarios planned, levels defined, must-have vs nice-to-have |
| Stage 2 (Environment Design) | Fidelity table — SIMULATED components and their risks |
| Stage 3 (Environment Setup) | Environment health report, smoke test result |
| Stage 4 (Diagnosis) | Honest test inventory, reclassified tests, original gap count |
| Stage 5 (Implementation) | Gaps closed, code bugs found, remaining nice-to-have gaps |
| Stage 6 (E2E Validation) | Integration sequence results, known limitations |
| Stage 7 (CI Pipeline) | Pipeline architecture, gate configuration, CI validation result |

---

## Step 1b: Final Verification & Domain Hook

**MANDATORY**: Run `verification-before-completion` — execute full test suite + build on current codebase to capture final state as evidence. Do not rely solely on prior stage results if code was modified in Stages 5-7. Include results in evidence package.

**CONDITIONAL**: Invoke `owasp-security` if security-sensitive paths (controllers, DTOs, guards, auth, frontend forms, config, package.json, chain/enrichment) were not already covered by `security-reviewer` in Post-Implementation Chain stages during this VERIFICATION phase. Review the final codebase state for any gaps.

---

## Step 2: Production Readiness Criteria Evaluation

Evaluate against the criteria established in Stage 1 (test strategy) and enriched during the phase:

### Objective Criteria Schema

Two related but distinct markdown tables, both parseable:

**(a) Criterion Definition Schema** — catalog of criteria, lives in this file. Rows define what each criterion *is*:

```
| id | type | admissibility | description | source | gate_rule | notes |
```

`type` ∈ `{binary, severity}`. `admissibility` ∈ `{admissible, non-admissible}` — **required for `binary` criteria only** (severity criteria have built-in override paths via H-override and M-KL; severity rows MAY leave the column blank or `—`). See Non-admissible criteria table below.

**(b) Criterion Evaluation Schema** — per-verdict record, lives in each `production-readiness-verdict.md`. Rows record the *outcome* of evaluating the catalog against the current unit:

```
| id | type | gate rule | result | severity / KL-id (if fail) | notes |
```

The evaluation table references criteria by `id`; the CI validator parses the definition schema (a) and is not concerned with verdict tables (b).

**Known Limitation id format**: `KL-NN` where `NN` is a zero-padded two-digit sequence number unique within a single verdict. Example: `KL-01`, `KL-02`. The risk-assessment path for H-severity KLs is `aidlc-docs/verification/risk-assessments/KL-NN.md`.

- `binary`: result is `PASS` or `FAIL`. `gate_rule: PASS required for GO`.
- `severity`: result carries level `H / M / L`. Default `gate_rule`: `0 H without override; 0 M without Known Limitation; L always admitted`. Individual criteria may tighten this default (e.g., `0 H; 0 M` with no admission path) but cannot loosen it.

### Severity semantics

- `H` — blocks GO. Override path: explicit user approval recorded as Known Limitation H-override, backed by a written risk assessment at `aidlc-docs/verification/risk-assessments/KL-NN.md` with the required sections below.

  **Risk assessment minimum structure** (omitting any section invalidates the override):

  ```markdown
  # Risk Assessment — KL-NN — <title>

  ## Failure mode
  What breaks and under what conditions (observable symptoms, trigger events).

  ## Blast radius
  Scope of impact — users affected, data at risk, downstream services, reversibility.

  ## Probability
  Estimated likelihood (with reasoning). Qualitative band — low / medium / high — tied to concrete triggers.

  ## Compensating controls
  Monitoring, alerts, runbooks, feature flags, or rollback procedures that contain the risk if it materializes.

  ## Accepted by
  Name, email, timestamp. Must match the Stakeholder Approval in the verdict.
  ```
- `M` — negotiable. Requires Known Limitation entry (description + mitigation plan + mitigation deadline) and stakeholder approval.
- `L` — recorded in verdict for auditability. Does not gate GO; no approval required.

### Known-limitation is the unified escape

A Known Limitation is not a criterion in itself — it is the mechanism by which a failed criterion (binary FAIL or severity ≥ M) is admitted into a CONDITIONAL-GO verdict.

**Binary-via-known-limitation rule**:

> A `binary` criterion may result `FAIL` and still yield `CONDITIONAL-GO` if and only if: (a) a Known Limitation exists with category matching the criterion, severity `M`, mitigation plan, and deadline; (b) the user accepts it in Stakeholder Approval; and (c) the criterion is not tagged `non-admissible`. A binary `FAIL` without a matching accepted Known Limitation → automatic `NO-GO`. A `non-admissible` criterion that fails → automatic `NO-GO` with no KL escape.

**Non-admissible criteria** (closed list — cannot be bypassed via KL-M):

| id | reason for non-admissibility |
|---|---|
| `security-scans` (CRITICAL findings only) | Known CRITICAL security issues in production are not a negotiable trade-off. HIGH findings remain admissible via KL-M. |
| `migrations-rollback` (when change scope contains migrations) | Shipping a migration without a proven rollback path risks unrecoverable data loss. `N/A` remains valid when no migrations are in scope. |
| `no-dishonest-tests` | Dishonest tests invalidate the entire verdict by mis-signaling coverage. |
| `script-workflow-alignment` | Zombies cannot be admitted as a Known Limitation. There is no scenario in which shipping with known zombies is a defensible trade-off: either the mechanism enforces something (wire it) or it doesn't (remove it or allow-list with rationale). The visible-but-unwired declaration creates a false sense of security worse than absence — see issue #71 and the `viblocks-ai#460`/`#470` empirical case. |

All other `binary` criteria (including `observability-minimum`, `e2e-happy-path`) remain admissible via the binary-via-KL-M rule. New criteria added in future must declare `admissible` or `non-admissible` explicitly — omitting the tag is a schema error.

### Iteration Cap (cross-cutting)

All VERIFICATION approval gates that present a "Request Changes" or equivalent rework option are capped at **3 iterations**. On the 4th, the stage must either accept the current state (registering remaining issues as Known Limitations) or explicitly pause the phase pending user intervention. This rule is referenced by every stage's completion message template.

### Stakeholder Approval Flow

1. Stage 8 emits a verdict draft with Known Limitations enumerated.
2. Each KL carries: `id`, `category`, `severity`, `description`, `mitigation plan`, `mitigation deadline`.
3. The user responds at the approval gate with accept/reject per KL id (e.g., `accept KL-01, KL-03; reject KL-02`).
4. For each H-severity KL accepted, the user provides the path to a risk-assessment doc.
5. The final verdict records: `accepted-by: <name>`, `email`, `timestamp`, `accepted-ids: [...]`, `rejected-ids: [...]`, `h-overrides: [...]`.
6. Any rejected KL whose underlying criterion is required → verdict becomes NO-GO. Rejected KLs whose criterion is `L` or optional → registered and verdict unaffected.

### Criteria Catalog

| id | type | admissibility | description | source | gate_rule | notes |
|---|---|---|---|---|---|---|
| must-have-pass | binary | admissible | All must-have P1-P2 scenarios have honest tests passing | Stage 5 + 6 | PASS required for GO | — |
| integration-sequence | binary | admissible | All service pairs validated | Stage 6 | PASS required for GO | — |
| no-dishonest-tests | binary | non-admissible | 0 tests reclassified as dishonest in Stage 4 that remain unfixed | Stage 4 + 5 | PASS required for GO | non-admissible per §3.3 |
| env-reproducible | binary | admissible | Test env recreatable from committed files | Stage 3 | PASS required for GO | — |
| ci-gate-coverage | binary | admissible | CI enforces test levels at gates | Stage 7 | PASS required for GO | — |
| e2e-happy-path | binary | admissible | Complete pipeline happy path passes | Stage 6 | PASS required for GO | — |
| security-scans | binary | non-admissible | 0 CRITICAL findings | Post-Implementation Chain | PASS required for GO; N/A if no security-sensitive paths | non-admissible per §3.3 |
| e2e-failure-coverage | severity | — | ≥1 failure/recovery scenario passing per critical boundary | e2e-validation-report.md (6b) | 0 H; M admitted via Known Limitation | L not permitted |
| observability-minimum | binary | admissible | ≥1 metric + ≥1 active alert per critical service | provisioning manifests + alert routing config | PASS required for GO | tool-agnostic |
| migrations-rollback | binary | non-admissible | Rollback executed in test env for all pending migrations in change scope (up → down → up) | migration-rollback-report.md | PASS required for GO; N/A if no migrations in change scope | scope-claim validated in CI |
| security-scans-catalog | binary | non-admissible | Dependency scan + secrets scan green, 0 CRITICAL | security-scan-report.md | PASS required for GO | HIGH findings recorded; complements Post-Impl Chain security-reviewer |
| script-workflow-alignment | binary | non-admissible | 0 zombie scripts: every workspace `package.json` `scripts.*` entry (or platform equivalent: `pyproject.toml`, `Cargo.toml`, `Makefile` targets, `scripts/`) is either invoked by ≥1 CI workflow step OR explicitly allow-listed with rationale per `script-workflow-allowlist.md` | Stage 7 (CI Pipeline Implementation Step 1a) | PASS required for GO | non-admissible per §3.3 — zombies create false sense of security and are not a tradeable risk |

### Observability Minimum
- Critical service scope inherits from the Critical Boundary List in test-strategy-design.md Step 8.
- Verification: existence of metric export + alert with active routing. No requirement that the alert has ever fired.
- Tool-agnostic: Grafana, CloudWatch, Datadog, Prometheus equivalents all satisfy.

### Migrations Rollback
- Scope is the current change only, not all historical migrations.
- Rollback executed as `up → down → up` in the test environment regardless of tool (Prisma, TypeORM, Flyway, Alembic, raw SQL, etc.).
- Non-admissible: FAIL → automatic NO-GO, no KL-M escape. Irreversible migrations must be re-scoped before shipping.
- `N/A` is a first-class outcome when the diff contains no migration files. Stage 7 CI validates the scope claim against the diff.

### Security Scans
- Example tools: `npm audit` / Trivy (dep), `gitleaks` / TruffleHog (secrets). Any equivalent satisfies.
- CRITICAL findings are non-admissible: ≥1 CRITICAL → NO-GO. Must be remediated before verdict.
- HIGH findings do NOT fail this binary (it only gates on CRITICAL). Record HIGH in the report; if deferring remediation is desired, enter a separate KL-M record — does NOT flip this criterion to FAIL.
- Complements, does not replace, the per-PR security-reviewer in the Post-Implementation Chain.

### No Dishonest Tests
- Source: test-diagnosis.md Honesty Audit section.
- A dishonest test claims a level it does not meet (e.g., labeled "integration" but boundaries are mocked).
- Non-admissible: FAIL → NO-GO. Admitting via KL-M would defeat the verdict's integrity signal.

### Script-Workflow Alignment
- Source: Stage 7 `check-invariants.<ext>` job (defined in `ci-pipeline-implementation.md` Step 1a).
- A zombie is a declared quality mechanism (`package.json` `scripts.*`, `Makefile` target, `pyproject.toml` script, `Cargo.toml` alias, file under `scripts/`) with no enforcer running it and no entry in the OPT-OUT allow-list (`aidlc-docs/verification/script-workflow-allowlist.md`).
- Detection: the `check-invariants.<ext>` CI job enumerates declared mechanisms per stack and verifies each has either an enforcer match in workflow files OR an allow-list entry. Exit non-zero on any zombie.
- Non-admissible: FAIL → NO-GO. Zombies create a false sense of security worse than absence (declared artifact lies about enforcement state); admitting them via KL-M would re-introduce the exact failure mode the criterion exists to close.
- Empirical basis: `viblocks-ai#460` (UI: `tsc --noEmit` declared, never invoked by any CI; ~30 days latent; 15 type errors silently accumulated) and `viblocks-ai#470` (backend: `eslint` declared in 3 services + root with no `.eslintrc*` config anywhere). Both are project-level fixes — this criterion is the methodology-level correction so the same class doesn't recur in any consumer.

---

## Step 3: Known Limitations Review

List everything that could NOT be validated and why:

| Limitation | Reason | Risk | Mitigation |
|---|---|---|---|
| SIMULATED components (from Fidelity Table) | Cannot replicate in test env | [specific risk] | [mitigation] |
| Non-reproducible edge cases | Depend on external state | [specific risk] | Monitoring/alerting |
| Performance under production load | Test env != production scale | [specific risk] | Staging validation |
| Nice-to-have scenarios not covered | Time/scope constraints | [specific risk] | Post-launch iteration |

---

## Step 4: Verdict

Based on criteria evaluation and known limitations, emit ONE of three verdicts:

| Verdict | Criteria | User action |
|---|---|---|
| **GO** | All must-have criteria PASS. Known limitations are acceptable (low risk or mitigated). | Deliver artifacts to DEPLOYMENT. |
| **CONDITIONAL-GO** | All must-have criteria PASS. BUT significant known limitations that the user must explicitly accept (high-risk SIMULATED components, missing failure/recovery tests, etc.) | Present limitations clearly. User must explicitly accept each one. |
| **NO-GO** | One or more must-have criteria FAIL. | Present specific blockers. Recommend returning to the stage that can resolve them (Stage 5 for test gaps, Stage 6 for E2E failures, Stage 3 for environment issues). |

---

## Step 5: Delivery Package (Output Contract)

Assemble the output artifacts for DEPLOYMENT:

1. **Complete Test Suite** — all tests passing, honestly classified by level
2. **Test Environment Definition** — reproducible, versioned (`docker-compose.test.yml` or equivalent)
3. **E2E Evidence Report** — evidence that the complete flow worked with real data in test environment
4. **Production Readiness Verdict** — GO/CONDITIONAL-GO with detail
5. **CI Pipeline Configuration** — workflows that run the test suite automatically
6. **Production Infrastructure Blueprint** — reconciled infra design ready for DEPLOYMENT
7. **Known Limitations** — what was NOT validated and why

---

## Step 6: Generate Verdict Artifact

Create `aidlc-docs/verification/production-readiness-verdict.md`:

```markdown
# Production Readiness Verdict — <unit-name> — YYYY-MM-DD

## Verdict
**<GO | CONDITIONAL-GO | NO-GO>**

## Objective Criteria Evaluation

The evaluation table omits the `admissibility` column — admissibility is inherited from the Definition Schema in `production-readiness-gate.md` via `id` lookup (see §3.1). Verdict generators MUST NOT duplicate the admissibility tag here; when a binary FAIL appears with a KL-id, the reader resolves admissibility against the definition catalog.

| id | type | gate rule | result | severity / KL-id (if fail) | notes |
|---|---|---|---|---|---|
| e2e-happy-path | binary | PASS required | PASS | — | 100% integration sequence green |
| e2e-failure-coverage | severity | 0 H | PASS | — | Critical boundaries ≥1 failure scenario |
| observability-minimum | binary | PASS required | FAIL | KL-01 | guardian alert missing |
| migrations-rollback | binary | PASS required or N/A | N/A | — | No migrations in change scope |
| security-scans | binary | PASS required | PASS | — | 0 CRITICAL |
| no-dishonest-tests | binary | PASS required | PASS | — | Honesty audit clean |

## Known Limitations

| id | category | severity | description | mitigation plan | deadline | risk-assessment (if H) |
|---|---|---|---|---|---|---|
| KL-01 | observability | M | No alert routing for guardian service | Add guardian alert to Grafana provisioning | 2026-05-15 | — |

## Stakeholder Approval

- **Name**: <user name>
- **Email**: <user email>
- **Timestamp**: <ISO 8601>
- **Accepted KL ids**: [KL-01]
- **Rejected KL ids**: []
- **H-severity overrides**: []

## Evidence

- Test strategy: `aidlc-docs/verification/test-strategy.md`
- E2E validation: `aidlc-docs/verification/e2e-validation-report.md`
- Security scans: `aidlc-docs/verification/security-scan-report.md` (or N/A)
- Migration rollback: `aidlc-docs/verification/migration-rollback-report.md` (or N/A)
- Risk assessments (if any): `aidlc-docs/verification/risk-assessments/`

## Delivery Package
<retain existing>
```

Forward-only migration: existing verdicts (e.g., 2026-04-14 CONDITIONAL-GO) are not rewritten retroactively.

---

## Step 7: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark Production Readiness Gate as complete
- Record verdict

---

## Step 8: Present Verdict

1. **Completion Announcement**:

```markdown
# 🏁 Production Readiness Gate — [VERDICT]
```

2. **AI Summary**: Structured bullet points:
   - Verdict: GO / CONDITIONAL-GO / NO-GO
   - Must-have coverage: N/M scenarios
   - Known limitations: N (list critical ones)
   - For CONDITIONAL-GO: list each limitation requiring user acceptance
   - For NO-GO: list specific blockers and recommended return stage
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

For **GO**:
```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the verdict at: `aidlc-docs/verification/production-readiness-verdict.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Dispute the verdict or request additional validation
> ✅ **Accept Verdict** - Accept GO verdict and proceed to **DEPLOYMENT** (placeholder)

---
```

For **CONDITIONAL-GO**:
```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the verdict at: `aidlc-docs/verification/production-readiness-verdict.md`
>
> **⚠️ CONDITIONAL-GO requires explicit acceptance of known limitations listed above.**



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Address limitations before accepting, or return to a previous stage
> ✅ **Accept Limitations** - Explicitly accept all listed limitations and proceed to **DEPLOYMENT**

---
```

For **NO-GO**:
```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the verdict at: `aidlc-docs/verification/production-readiness-verdict.md`
>
> **🚫 NO-GO — blockers must be resolved before proceeding.**



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Return to a previous stage to address the specific blockers identified
> ✅ **Accept & Continue** - Override NO-GO verdict and proceed to **DEPLOYMENT** (not recommended)

---
```

## Step 9: Wait for Explicit Approval
- For GO: standard approval
- For CONDITIONAL-GO: user must explicitly accept each limitation
- For NO-GO: user decides (return to stage or accept risk)

## Step 10: Record Verdict and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## Production Readiness Gate
**Timestamp**: [ISO timestamp]
**Status**: [GO / CONDITIONAL-GO / NO-GO]
**Key Results**: Verdict: [verdict]; Must-have coverage: N/M; Known limitations: N; Accepted limitations: N
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- If CONDITIONAL-GO: log each accepted limitation
- Mark Production Readiness Gate as complete in aidlc-state.md
- Mark VERIFICATION Phase as complete
