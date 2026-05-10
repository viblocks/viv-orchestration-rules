# Test Diagnosis

**Purpose**: Establish the real state of testing with evidence, not assumptions. Answers: "what do I have, what's missing, and what's lying?"

## Prerequisites
- Test Environment Setup (Stage 3) must pass (smoke test green)
- Test Strategy (Stage 1) must be approved
- Test environment must be running

---

## Step 1: Test Inventory

Find all test files in the codebase:

```bash
# Find all test files
find . -name "*.spec.ts" -o -name "*.test.ts" -o -name "*.e2e-spec.ts" | sort
```

For each test file, record:
- File path
- Service it belongs to
- Declared type (unit, integration, e2e — based on file name, folder, or describe block)
- Dependencies (what does it import? Does it mock anything? Does it start infrastructure?)

**Output**: Raw test inventory table.

---

## Step 2: Execute All Tests

Run the entire existing test suite in the **real test environment** (not with mocks):

```bash
# Start test environment
make test-env-up

# Run all tests per service
pnpm --filter <service> run test
# Repeat for each service

# Run integration/e2e tests if they exist
pnpm --filter <service> run test:integration  # if script exists
pnpm --filter <service> run test:e2e          # if script exists
```

Record per test: **pass / fail / skip** with error message if failed.

**Output**: Test execution results table.

---

## Step 3: Honesty Audit

For each test classified as "integration" or higher (L2+), apply the honesty check:

**Dishonesty signals** (test is actually L1):
- Imports a mock/stub/fake of the service on the other side of the boundary
- Uses `jest.mock()` or `vi.mock()` on communication modules (Kafka producer/consumer, HTTP client, DB client)
- The `beforeAll`/`beforeEach` doesn't start real infrastructure (no Testcontainers, no Docker, no external connection)
- The test passes even when the test environment is NOT running

**Honesty signals** (test is genuinely L2+):
- Uses Testcontainers or connects to real service from test environment
- Data crosses real serialization/deserialization (JSON.stringify → network → JSON.parse)
- The test fails if the other service is unavailable

For each dishonest test, reclassify:
- Move to L1 (unit test) in the classification
- Document evidence: "This test mocks [X] at the boundary — it's a unit test"

**Output**: Honesty audit results — reclassified tests with evidence.

---

## Step 4: Coverage Mapping

Map existing tests against the Test Strategy plan (Stage 1):

For each scenario in the test strategy:
- Is there an existing test? → Link to test file
- At what level? → Compare declared vs honest level
- Does it cover the scenario adequately? → Full/partial/none

```markdown
| Scenario | Expected Level | Existing Test | Honest Level | Coverage |
|---|---|---|---|---|
| core→kafka happy path | L2 | core/kafka.spec.ts | L1 (mocked) | None (dishonest) |
| bot receives event | L2 | None | — | Missing |
| UI displays balance | L6 | ui/balance.e2e.ts | L6 | Full |
```

**Output**: Coverage matrix.

---

## Step 5: Gap Report

Produce the final diagnosis combining all previous steps:

```markdown
## Diagnosis Summary

### What's Correct
- [List of scenarios with honest, adequate coverage]

### What's Missing
- [List of scenarios with no test at all]
- Priority: [P1/P2/P3/P4 from test strategy]

### What Needs Rewriting
- [List of dishonest tests that need to be rewritten at the correct level]
- Current: [file path] classified as [declared level]
- Should be: [honest level]
- Evidence: [why it's dishonest]

### What's Broken
- [List of tests that fail in the real test environment]
- Error: [error message]
- Likely cause: [diagnosis]
```

**Output**: `aidlc-docs/verification/test-diagnosis.md`

---

## Step 6: Determine Stage 5 Execution (Formal Gate)

Present the user with three options and wait for explicit choice:

- **PROCEED** — execute Stage 5. Gap report identifies actionable gaps.
- **SKIP** — Stage 5 is N/A. Gap report is empty (full honest coverage). Workflow advances to Stage 6. Log in audit.md.
- **AMEND** — correct the gap report. Used when the user identifies misclassified tests, false-positive gaps, or an environment problem that prevented execution. Specific corrections (test ids, gap entries) are applied to the artifact, the gate re-presents. If the cause was a broken test environment, the AMEND response records a requirement to re-enter Stage 3 before re-executing Stage 4.

### Iteration Cap

AMEND is capped at **3 cycles**. On the 4th attempt, the stage escalates: the user must either PROCEED with the current gap report (accepting its imperfections as Known Limitations at Stage 8) or declare the diagnosis blocked and pause the phase. This cap mirrors the cross-cutting iteration cap declared in `production-readiness-gate.md`.

### No Autonomous Default

VERIFICATION is user-supervised by design. Autonomous change flows (DIRECT / REVERT per CLAUDE.md) do not run full VERIFICATION and therefore cannot reach this gate.

### Downstream

- PROCEED → Stage 5 prerequisites satisfied.
- SKIP → Stage 5 skipped; advance to Stage 6.
- AMEND → re-enter this step after applying corrections (and Stage 3 if env was broken).

---

## Step 7: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark Test Diagnosis as complete

**Note**: Stage 4 is a formal gate (see Step 6). User chooses PROCEED / SKIP / AMEND — the choice must be recorded in audit.md along with the gap report contents that were presented at the time of the decision.

---

## Step 8: Present Diagnosis Results

1. **Completion Announcement**:

```markdown
# 🔍 Test Diagnosis Complete
```

2. **AI Summary**: Structured bullet points:
   - Total tests found
   - Tests executed: pass/fail/skip counts
   - Honesty audit: N tests reclassified (with specific examples)
   - Coverage: N/M must-have scenarios covered honestly
   - Gaps: N missing, N need rewriting, N broken
   - Stage 5 recommendation: EXECUTE / SKIP / USER DECIDES
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the diagnosis at: `aidlc-docs/verification/test-diagnosis.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> ▶️ **PROCEED** - Execute Stage 5 (Test Implementation)
> ⏭️ **SKIP** - Stage 5 is N/A; advance to Stage 6 (E2E Validation)
> 🔁 **AMEND** - Apply corrections to the gap report and re-present (max 3 cycles)

---
```

## Step 9: Record and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## Test Diagnosis
**Timestamp**: [ISO timestamp]
**Status**: [Complete]
**Key Results**: Tests found: N; Reclassified: N; Gaps: N missing, N dishonest, N broken; Stage 5: EXECUTE/SKIP
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- Mark Test Diagnosis as complete in aidlc-state.md
