# Test Implementation

**Purpose**: Write missing tests, level by level, using the correct typed agents. Close gaps identified in Test Diagnosis.

**Condition**: Execute IF user selected PROCEED at Stage 4 gate (test-diagnosis.md Step 6). Skip IF user selected SKIP.

## Prerequisites
- Test Diagnosis gate selected PROCEED (per test-diagnosis.md Step 6)
- Gap report must identify actionable gaps
- Test environment must be running (Stage 3)

---

## Step 1: Load Gap Report

Read `aidlc-docs/verification/test-diagnosis.md` and extract:
- Missing scenarios (grouped by level)
- Dishonest tests needing rewrite (grouped by level)
- Broken tests needing fix
- Priority ordering from Test Strategy (Stage 1)

---

## Step 1b: Create Execution Plan

Create `aidlc-docs/verification/plans/test-implementation-plan.md` from the gap report:

```markdown
# Test Implementation — Execution Plan

## Gaps by Level

### L1 (if gaps exist)
- [ ] [Scenario name] — [boundary] — [action: write new / rewrite dishonest]
- [ ] [Scenario name] — ...
- [ ] Regression check after L1

### L2 (if gaps exist)
- [ ] [Scenario name] — [boundary] — [action]
- [ ] ...
- [ ] Regression check after L2 (re-run L1)

### L3-L5 (repeat pattern per level with gaps)
```

The plan is DYNAMIC — generated from the gap report content. Each scenario from the gap report becomes a checkbox. Only include levels that have gaps.

Update checkboxes immediately as each scenario is closed — same interaction.

---

## Step 2: Level-by-Level Execution Loop

Execute gaps **in level order** (L1 → L2 → L3 → L4 → L5). Each level must be fully closed before advancing to the next.

**Why L1 first**: A broken unit test can cause false positives at higher levels. Close L1 gaps before attempting L2+.

```
For each level with gaps (L1 → L2 → L3 → L4 → L5), in order:
  |
  +-- 2a. Select scenarios for current level from gap report
  |
  +-- 2b. Dispatch typed implementer (see dispatch rules below)
  |       Prompt includes:
  |       - Concrete scenario from test strategy (with example data)
  |       - Test environment configuration
  |       - Honesty Rule: "Write a test that FAILS if the other service is unavailable"
  |       - Integration sequence position (which service pair)
  |
  +-- 2c. Execute test in real test environment
  |       Pass? --> mark scenario as covered
  |       Fail? --> classify:
  |         +-- Test bug --> fix test
  |         +-- Code bug --> fix code (the test found a real bug!)
  |       **Escalation**: After 3 failed fix attempts for the same scenario,
  |       escalate to user with:
  |         - What was attempted (3 fix descriptions)
  |         - What specifically fails (error output)
  |         - Proposed alternatives (skip scenario, change approach, accept limitation)
  |       Do NOT continue iterating blindly on the same failure.
  |
  +-- 2d. Honesty check of newly written test
  |       Does it really cross the boundary with the real component?
  |       Does it fail if the other service is down?
  |       If dishonest --> reject, rewrite
  |
  +-- 2e. Update gap report -- mark as closed
  |
  +-- 2f. Regression check -- re-execute ALL previous levels
          If regression detected --> fix BEFORE advancing to next level
```

---

## Step 3: Typed Agent Dispatch Rules

**IRON LAW**: All test code is written by typed agents, never by the orchestrator.

| Tests for | Typed Agent | Notes |
|---|---|---|
| `services/core/**` | `backend-crypto-implementer` | Include honesty constraint in prompt |
| `services/bot/**` | `backend-crypto-implementer` | Include honesty constraint in prompt |
| `packages/shared/**` | `backend-crypto-implementer` | Shared contracts tested as part of consumer |
| `services/ui/**` | `frontend-crypto-implementer` | Include honesty constraint in prompt |
| Cross-service contracts | Agent for the service producing the test | `packages/shared` first (cross-domain rule) |
| E2E harness scripts | `backend-crypto-implementer` | Bash/shell scripts for orchestration |
| Docker/compose for tests | `infra-devops-implementer` | Environment extensions |

**Post-Implementation Chain applies**: After each typed implementer completes, the standard chain fires (verification → code review → security if applicable).

---

## Step 4: Handle Code Bugs Found by Tests

When a new test reveals a code bug:

**IRON LAW**: Invoke `superpowers:systematic-debugging` before each fix attempt. Root cause must be documented in the implementer prompt.

1. The test is correct — it exposed a real defect
2. Fix the code using the appropriate typed implementer
3. Post-Implementation Chain applies to the code fix
4. Re-run the test to confirm it now passes
5. Log the bug discovery in the test implementation report: "Test for [scenario] revealed bug in [file:line] — [description]"

This is a victory, not a problem. The test did its job.

---

## Step 5: Notes on L1 and L6

**L1 (unit tests)**: Already written during Code Generation with TDD. If diagnosis finds L1 gaps, they are closed FIRST in the level loop (before L2) because broken unit tests cause false positives at higher levels. L1 is not the primary focus of this stage.

**L6 (E2E browser)**: E2E scenarios are defined in the test strategy and executed in Stage 6 (E2E Validation). Stage 5 does NOT write L6 tests. If an L6 gap is discovered during E2E Validation (Stage 6), it is resolved within Stage 6.

---

## Step 6: Generate Test Implementation Report

Create `aidlc-docs/verification/test-implementation-report.md`:

```markdown
## Test Implementation Summary

### Gaps Closed
| Scenario | Level | Test File | Status |
|---|---|---|---|
| [scenario] | L[N] | [path] | Closed |

### Code Bugs Found
| Bug | Found by test | Fix | File:Line |
|---|---|---|---|
| [description] | [test file] | [fix description] | [location] |

### Regression Checks
| After Level | Previous Levels | Result |
|---|---|---|
| L2 | L1 | All pass |
| L3 | L1, L2 | All pass |

### Remaining Gaps (nice-to-have)
| Scenario | Level | Priority | Reason not closed |
|---|---|---|---|
```

---

## Step 7: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark Test Implementation as in-progress/complete

---

## Step 8: Present Completion Message

1. **Completion Announcement**:

```markdown
# ✅ Test Implementation Complete
```

2. **AI Summary**: Structured bullet points:
   - Gaps closed: N scenarios across L1-L5
   - Code bugs found and fixed: N
   - Regression checks: all pass / issues found
   - Remaining nice-to-have gaps: N
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the test implementation report at: `aidlc-docs/verification/test-implementation-report.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications or additional test coverage
> ✅ **Continue to Next Stage** - Approve test implementation and proceed to **E2E Validation**

---
```

## Step 9: Wait for Explicit Approval
- Gate: All scenarios prioritized as "must-have" in Stage 1 have honest tests passing

## Step 10: Record Approval and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## Test Implementation
**Timestamp**: [ISO timestamp]
**Status**: [Approved/Complete]
**Key Results**: Gaps closed: N scenarios (L1-L5); Code bugs found: N; Regressions: NONE/[details]
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- Mark Test Implementation as complete in aidlc-state.md
