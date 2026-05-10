# E2E Validation

**Purpose**: Validate that the complete system works end-to-end with data as close to production as possible. This is the moment of truth.

## Prerequisites
- Test Environment running (Stage 3)
- Tests L2-L5 passing (Stage 5, or Stage 4 if Stage 5 was skipped)
- Test Strategy with E2E scenarios and Integration Sequence (Stage 1)
- Fidelity table (Stage 2) — to know which SIMULATED components carry risk

---

## Step 1: Load Integration Sequence

Read the Integration Sequence from `aidlc-docs/verification/test-strategy.md`.

E2E Validation follows this sequence exactly. Each step adds one more service to the validated set. If a step fails, the error is isolated to the newly added boundary.

---

## Step 2: Execute Integration Sequence

For each step in the Integration Sequence:

```
Step N: E2E of [service pair description]
  |
  +-- 2a. Start required services (subset of test environment)
  |
  +-- 2b. Execute E2E scenarios for this pair:
  |       - Happy path FIRST
  |       - Then edge cases
  |       - Then failure/recovery scenarios
  |
  +-- 2c. Result?
        +-- All pass --> Next integration step
        +-- Fail --> Sandwich pattern (Step 3)
```

---

## Step 3: Sandwich Pattern (on failure)

When an E2E scenario fails at integration step N:

```
1. Identify blocker(s) from the failure
2. Classify each blocker:
   +-- Test bug --> fix test (wrong assertion, incorrect setup)
   +-- Code bug --> fix code (dispatch typed agent for the affected service)
   +-- Environment gap --> fix environment (dispatch infra-devops-implementer)
   +-- Missing L2-L5 test --> back to Stage 5 (a gap was missed)
3. Fix the blockers
4. Re-attempt ONLY failed scenarios (not the full suite)
5. Loop until all pass OR blockers documented as known limitations
6. **Escalation guard**: After 3 fix-retry cycles for the same blocker,
   escalate to user with:
   - What was attempted (3 fix descriptions)
   - What specifically fails
   - Proposed alternatives (document as known limitation, change approach, defer)
   Do NOT continue iterating blindly. This aligns with the default
   escalation limit (Pattern 3.4, 3 attempts).
```

**Typed agent dispatch for fixes:**
- Code bugs in `services/core`, `services/bot` → `backend-crypto-implementer`
- Code bugs in `services/ui` → `frontend-crypto-implementer`
- Environment fixes → `infra-devops-implementer`
- Post-Implementation Chain applies to all code fixes

**Debugging:** If an E2E failure is unclear, invoke `superpowers:systematic-debugging` before proposing a fix.

---

## Step 4: Sub-gates 6a / 6b / 6c

Each integration step in the sequence is evaluated against three sub-gates. Stage 6 has no local terminal failure — the final verdict lives at Stage 8.

### 6a. Happy path (binary)
All integration steps of the complete pipeline must PASS. Binary PASS required for GO — recorded as criterion `e2e-happy-path` in the verdict.

### 6b. Failure & recovery — critical boundaries only (severity-typed)
Run ≥1 failure/recovery scenario per critical boundary (per Critical Boundary List from `test-strategy-design.md` Step 8). A failed scenario becomes a Known Limitation with severity assigned by this rule (not user opinion):

- **H** — failure mode involves irreversible data or money loss, silent duplication, or security invariant violation.
- **M** — failure mode is recoverable (retries, idempotency keys, manual remediation) within a bounded time window and with no persistent loss.
- **L** is NOT permitted in 6b — every tracked 6b failure sits on a critical boundary by definition and is at least M.

The user may challenge the classification during the approval flow; any deviation from the rule must be noted in the verdict.

### 6c. Edge cases (optional, not gated)
Edge-case scenarios (including SIMULATED-boundary validation and extreme-data cases) are recorded for the verdict but do not gate Stage 6 exit. Reference Fidelity Table from Stage 2 when selecting SIMULATED-boundary scenarios.

### Stage 6 exit

No local terminal failure. Any 6b scenario that cannot be made to pass after the 3-cycle sandwich loop is registered as a Known Limitation in `e2e-validation-report.md`, and Stage 6 continues to Stage 7. The final GO / CONDITIONAL-GO / NO-GO decision lives in Stage 8.

---

## Step 5: E2E Evidence Collection

For each executed scenario, collect evidence:
- **Command or action** that triggered the scenario
- **Expected result** vs **actual result**
- **Logs** — relevant log lines from each service involved
- **Data state** — database records, message broker state, UI state
- **Screenshots** — for UI scenarios (if applicable)

Evidence is the foundation of the Production Readiness Gate (Stage 8).

---

## Step 6: Generate E2E Validation Report

Create `aidlc-docs/verification/e2e-validation-report.md`:

```markdown
## E2E Validation Report

### Integration Sequence Results

| Step | Service Pair | Happy Path | Edge Cases | Failure/Recovery | Overall |
|---|---|---|---|---|---|
| 1 | [pair] | PASS | PASS | PASS | PASS |
| 2 | [pair] | PASS | PASS (1 known limitation) | PASS | PASS |

### Scenario Details

#### Step 1: [service pair]

**Scenario: [name]**
- Trigger: [what was done]
- Expected: [what should happen]
- Actual: [what happened]
- Evidence: [logs, data, screenshots]
- Status: PASS / FAIL / KNOWN LIMITATION

### Blockers Found and Resolved

| Blocker | Type | Fix | Resolved |
|---|---|---|---|
| [description] | Code bug / Test bug / Env gap | [what was done] | Yes / Known limitation |

### Known Limitations

| Limitation | Why | Risk | Mitigation |
|---|---|---|---|
| [description] | [explanation] | [impact] | [what reduces the risk] |
```

---

## Step 7: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Update Integration Sequence Position checkboxes
- Mark E2E Validation as in-progress/complete

---

## Step 8: Present Completion Message

1. **Completion Announcement**:

```markdown
# 🎯 E2E Validation Complete
```

2. **AI Summary**: Structured bullet points:
   - Integration steps completed: N/N
   - Scenarios executed: N (pass: N, known limitations: N)
   - Code bugs found and fixed: N
   - Known limitations: N (list briefly)
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the E2E validation report at: `aidlc-docs/verification/e2e-validation-report.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for additional E2E scenarios or dispute known limitations
> ✅ **Continue to Next Stage** - Approve E2E validation and proceed to **CI Pipeline Implementation**

---
```

## Step 9: Wait for Explicit Approval
- Gate: Happy path of the complete pipeline passes. Edge cases and failure scenarios classified as pass/known-limitation.

## Step 10: Record Approval and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## E2E Validation
**Timestamp**: [ISO timestamp]
**Status**: [Approved/Complete]
**Key Results**: Integration steps: N/N; Scenarios: N pass, N known limitations; Code bugs fixed: N
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- Mark E2E Validation as complete in aidlc-state.md
