# Post-Deploy Validation

**Purpose**: Confirm the production deploy runs correctly — independent of whether rollback works (that is Stage 7).

**Stage type**: ANALYSIS (informational report, no Question DNA)

## Prerequisites
- Stage 5 Deployment Execution complete with prod deploy successful (blocking)

---

## Step 0: Stage Bootstrap

**MANDATORY**: Mark Stage 6 as in-progress in `aidlc-docs/aidlc-state.md` (`[~]` marker on Stage 6 row). Log stage start in `aidlc-docs/audit.md` with ISO 8601 timestamp.

---

## Step 1: Execute Smoke Suite

Run the smoke suite implemented in Stage 4 against production. Capture all results.

## Step 2: Observation Window

Observe metrics and logs for a bounded window. Default by service profile from Stage 1:
- Stateless low-traffic: 15 min
- Stateful or high-traffic: 30 min

## Step 3: Baseline Comparison

Branching logic:
- **Subsequent deploys (not first)**: capture key metrics (latency, error rate, throughput) and compare against pre-deploy baseline derived from last N release windows.
- **Greenfield first deploy**: baseline = NFR targets from Construction (no historical baseline exists). Flag any metric outside NFR bounds.

## Step 4: Regression Detection

Flag any metric significantly worse than baseline. Threshold driven by NFRs where possible.

## Step 5: Evidence Package

Consolidate smoke output, observation logs, and metric snapshots with timestamps into `aidlc-docs/deployment/post-deploy-validation-report.md`.

---

## Step 6: Completion Message + Approval Gate

3-part per Pattern 2.4.

**Gate**: smoke PASS + no significant regression.

If regression detected:
- User chooses: accept as Known Limitation (admissible — continue to Stage 7), trigger manual rollback, or escalate.

Next stage: **Rollback Validation + Sign-off Gate**.

**MANDATORY**: Log the approval prompt in `aidlc-docs/audit.md` BEFORE presenting to user. On approval, log the user's raw response with timestamp. Mark Stage 6 complete (`[x]`) in `aidlc-docs/aidlc-state.md` with date and one-line summary.

---

## Extension Compliance Hooks

security-baseline: N/A (informational stage).
