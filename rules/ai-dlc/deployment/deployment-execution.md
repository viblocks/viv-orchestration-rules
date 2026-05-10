# Deployment Execution

**Purpose**: First real deploy using the pipeline created in Stage 4. Split execution: agent deploys pre-prod, user triggers prod.

**Stage type**: EXECUTION (multi-step, cross-session, split agent/human authority)

## Prerequisites
- Stage 4 CD Pipeline Implementation complete with all Post-Implementation Chain gates PASS (blocking)
- `aidlc-docs/verification/production-readiness-verdict.md` verdict remains GO/CONDITIONAL-GO (re-verify freshness)
- Main branch green (blocking)
- No active incident (blocking — check team-provided status)

---

## Step 0: Stage Bootstrap

**MANDATORY**: Mark Stage 5 as in-progress in `aidlc-docs/aidlc-state.md` (`[~]` marker on Stage 5 row). Log stage start in `aidlc-docs/audit.md` with ISO 8601 timestamp.

---

## Step 1: Create Execution Plan (supports session continuity)

Save to `aidlc-docs/deployment/plans/deployment-execution-plan.md`:

```markdown
# Deployment Execution Plan

## Execution Steps
- [ ] Step 1: Pre-flight checks
- [ ] Step 2: Pre-prod deploy (agent)
- [ ] Step 3: Pre-prod smoke (agent)
- [ ] Step 4: Pre-prod evidence capture
- [ ] Step 5: Prod trigger preparation
- [ ] Step 6: Prod deploy (user-triggered, agent observes)
- [ ] Step 7: Prod deploy evidence capture
- [ ] Step 8: Completion + approval
```

---

## Step 2: Pre-Flight Checks

**MANDATORY**: Verify all prerequisites. Abort if any fail.

| Check | How to verify |
|---|---|
| Verdict GO fresh | Verdict file modified within the acceptable freshness window (default: no expiration; re-check only if >30 days) |
| Main green | CI latest run on main is PASS |
| No active incident | Ask user explicitly: "Any active incident I should be aware of?" |
| Artifact versioned + pushed | Per Stage 3 versioning scheme; artifact exists in registry |

---

## Step 3: Pre-Prod Deploy (Agent-executed)

Dispatch `infra-devops-implementer` with instructions:
- Trigger the pre-prod deploy workflow (`make deploy-preprod` OR `gh workflow run deploy-preprod.yml`)
- Tail logs
- Capture full log output to evidence file

Sandwich pattern on failure: classify blocker (app / infra / config) → fix → retry. Max 3 cycles before escalation.

---

## Step 4: Pre-Prod Smoke (Agent-executed)

Execute smoke suite from Stage 4 against pre-prod. Capture:
- Smoke pass/fail per test
- Latency / error rate metrics
- Any anomalies

Output: evidence saved to `aidlc-docs/deployment/plans/preprod-smoke-evidence.md`.

---

## Step 5: Pre-Prod Evidence Capture

Consolidate into `aidlc-docs/deployment/deployment-execution-report.md` (partial — pre-prod section):
- Timestamps ISO 8601
- Artifact version deployed
- Deploy duration
- Smoke result summary

---

## Step 6: Prod Trigger Preparation

**CRITICAL**: Agent does NOT auto-deploy to prod.

1. Present the exact trigger command to the user. Example (format depends on Stage 3 design):
   ```
   gh workflow run deploy-prod.yml --ref v1.0.0
   ```
   or
   ```
   git tag -a v1.0.0 -m "Release v1.0.0" && git push origin v1.0.0
   ```
2. Wait for user to confirm: "Triggered" / "Abort".
3. If session pauses here (user leaves mid-stage), session continuity per Pattern 3.6: on resume, re-verify pre-prod still healthy (no time-based decay), then re-present trigger command.

---

## Step 7: Prod Deploy (User-triggered, agent observes)

After user confirms trigger:
- Agent observes logs via `gh run watch` or equivalent
- Capture full log output
- Monitor for failure signals (per Stage 3 failure branches)

On prod failure: auto-rollback is triggered by pipeline (Stage 3 Step 8 design). Phase state transitions to FAILED. Escalate to user. Re-entry via Change Flow (not stage retry in-place).

---

## Step 8: Prod Evidence Capture

Append to `deployment-execution-report.md` (prod section):
- Prod timestamp
- Artifact version confirmed in prod
- Deploy duration
- Any anomalies observed

---

## Step 9: Completion Message + Approval Gate

3-part per Pattern 2.4. Gate: pre-prod smoke PASS + prod deploy completed without errors. Next stage: **Post-Deploy Validation**.

**MANDATORY**: Log the approval prompt in `aidlc-docs/audit.md` BEFORE presenting to user. On approval, log the user's raw response with timestamp. Mark Stage 5 complete (`[x]`) in `aidlc-docs/aidlc-state.md` with date and one-line summary.

---

## Session Continuity

This stage may pause at Step 6 (user prod trigger). Resume protocol:
1. Read `aidlc-state.md` → stage is `[~]` in Stage 5
2. Read `deployment-execution-plan.md` — find last checked step
3. Re-verify pre-prod is still healthy (run smoke again)
4. Present the prod trigger command again
5. Continue

If pre-prod has degraded during pause, restart from Step 3.

---

## Extension Compliance Hooks

- `security-baseline`: N/A (no artifacts produced that fall under security review — artifacts are already deployed via reviewer-approved workflows).
