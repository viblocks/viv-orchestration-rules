# Rollback Validation + Sign-off Gate

**Purpose**: Prove rollback works via proactive exercise after a successful production deploy (not a reaction to failure — Stage 5 handles failed deploys via auto-rollback). Then emit the final phase verdict. Also produce the observability handoff inventory for OPERATIONS.

**Stage type**: ANALYSIS (enhanced — Pattern 2.4 defined exception: 3-verdict format GO / CONDITIONAL-GO / NO-GO, like Production Readiness Gate)

## Prerequisites
- Stage 6 Post-Deploy Validation complete with smoke PASS (blocking)

---

## Step 0: Stage Bootstrap

**MANDATORY**: Mark Stage 7 as in-progress in `aidlc-docs/aidlc-state.md` (`[~]` marker on Stage 7 row). Log stage start in `aidlc-docs/audit.md` with ISO 8601 timestamp.

---

## Step 1: Rollback Exercise

**Default (canary-capable stacks)**:
1. Deploy a fresh canary slice of the already-deployed release.
2. Trigger rollback against just that slice.
3. Verify the rollback path works on real prod infra.
4. Discard the canary. The main release remains untouched.

**Fallback (stacks that cannot canary — serverless, preview-branch-only, single-instance)**:
1. Exercise rollback on pre-prod.
2. Declare Known Limitation: "Prod rollback is simulated via pre-prod exercise; real prod rollback unverified until an actual incident forces it."

## Step 2: Rollback Health Check

Verify the slice (or pre-prod in fallback) is healthy on the prior version after rollback.

## Step 3: Canary Cleanup

Discard the canary slice used for the test. No re-deploy of the main release is needed — it was never rolled back.

## Step 4: Observability Handoff Inventory

Enumerate:
- Services instrumented (list, per-unit)
- Metrics baseline captured (from Stage 6)
- Stacks provisioned (Prometheus / Datadog / etc.)
- Dashboards present vs missing
- Alert rules present vs missing
- SLO formalization status (typically missing — OPERATIONS concern)

Output: `aidlc-docs/deployment/operations-handoff.md`.

## Step 5: Known Limitations Aggregation

Consolidate all KLs from Stages 1–7. Classify each:
- **Admissible**: can degrade GO → CONDITIONAL-GO with user accept
- **Non-Admissible**: forces NO-GO

**Non-Admissible KL list for DEPLOYMENT** (inherits from PR #259 pattern):
- Observability Readiness Input Check failed at Stage 1
- Prod smoke failed at Stage 6 (auto-rollback should have triggered; if present here, something is inconsistent)
- Rollback exercise not executed in any form (neither canary nor pre-prod fallback)
- security-reviewer CRITICAL findings from Stage 4 unresolved

**Admissible KL examples**:
- Greenfield first-deploy "rollback to void" (intrinsic)
- Pre-prod-only rollback exercise (declared fallback)
- Brownfield ad-hoc deploys not fully codified
- Observability gaps handed off to OPERATIONS

## Step 6: Pre-Flight Checklist (Phase Exit per Pattern 0.7)

Verify all phase deliverables exist:
- [ ] `aidlc-docs/deployment/deployment-strategy.md`
- [ ] `aidlc-docs/deployment/cd-pipeline-diagnosis.md` (if brownfield)
- [ ] `aidlc-docs/deployment/cd-pipeline-design.md`
- [ ] `aidlc-docs/deployment/cd-pipeline-impl-report.md`
- [ ] `aidlc-docs/deployment/deployment-execution-report.md`
- [ ] `aidlc-docs/deployment/post-deploy-validation-report.md`
- [ ] `aidlc-docs/deployment/operations-handoff.md`

If any missing, BLOCK verdict emission.

## Step 7: Verdict Emission

Apply decision logic:

| Condition | Verdict |
|---|---|
| Any Non-Admissible KL present | **NO-GO** (forced) |
| No KLs | **GO** |
| Only Admissible KLs | **CONDITIONAL-GO** |

Write to `aidlc-docs/deployment/deployment-verdict.md`.

## Step 8: Delivery Package

Assemble handoff package (all phase artifacts listed in Step 6 + verdict + handoff inventory).

## Step 9: Phase Close

Per Pattern 0.7 outgoing transition:

1. Present 3-part Completion Message + Approval Gate (enhanced — 3-verdict format):
   - **GO**: `✅ **Accept Verdict**` + transition to OPERATIONS (placeholder)
   - **CONDITIONAL-GO**: `✅ **Accept Limitations & Continue**` + user must explicitly accept each Admissible KL
   - **NO-GO**: `🔧 **Return to Stage N**` / `⚠️ **Accept Risk**`

2. On approval, mark `### DEPLOYMENT PHASE` complete in `aidlc-docs/aidlc-state.md`.
3. **MANDATORY**: Log the approval prompt in `aidlc-docs/audit.md` BEFORE presenting to user. On approval, log the user's raw response with timestamp. Mark Stage 7 complete (`[x]`) in `aidlc-docs/aidlc-state.md` with date and one-line summary.

---

## Extension Compliance Hooks

- `security-baseline`: applicable to final verdict — enforce that no secrets leaked in handoff inventory.
