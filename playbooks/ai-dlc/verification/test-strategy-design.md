# Test Strategy Design

**Purpose**: Design the test plan based on what was actually built — not theory, not templates.

## Prerequisites
- Code Generation must be complete for all units
- Build & Compile must pass (code compiles, unit tests pass)
- All construction artifacts available (functional design, NFR, application design, code generation plans)

---

## Step 1: Load Construction Artifacts

Read ALL of the following (skip any that don't exist):
- `aidlc-docs/construction/{unit-name}/functional-design/` — for each unit
- `aidlc-docs/construction/{unit-name}/nfr-design/` — for each unit
- `aidlc-docs/inception/application-design/` — component architecture
- `aidlc-docs/construction/plans/` — code generation plans
- `aidlc-docs/inception/reverse-engineering/` — if brownfield (component inventory, architecture)

**Output**: Mental model of what was built — components, boundaries, data flows.

---

## Step 2: Boundary Mapping

Read the actual generated code (not just docs) and identify every point where two components communicate:

For each boundary found, document:
- **Producer** → **Consumer** (which component initiates, which receives)
- **Transport** (Kafka, REST, WebSocket, SSE, shared DB, file system)
- **Data contract** (message type, DTO, event schema)
- **Criticality** (does this boundary move money, critical data, or user-facing state?)

**Output**: Boundary map table in the test strategy document.

---

## Step 2b: Clarifying Questions

Present boundary map summary to the user, then generate clarifying questions.

**Adaptive depth** — determine rigor by preliminary greenfield/brownfield assessment (quick glob for `**/*.spec.ts`, `**/*.test.ts`):

| State | Rigor | Behavior |
|---|---|---|
| `greenfield` (no test files found) | Full | Resolve all ambiguities before proceeding |
| `brownfield-tests` (test files but no test infra) | Standard | Detect, report, follow-up if ambiguous |
| `brownfield-full` (test files + test infra exist) | Light | Detect, report, proceed if user accepts |

**Question categories to evaluate** (ask only where the answer isn't obvious from artifacts):

- **Risk priorities** — Which boundaries are most critical to the business? Are there boundaries that move money, affect compliance, or have SLA implications the model might not see from code alone?
- **Testing philosophy** — What coverage level is expected? Is there a preference between test pyramid (many unit, few E2E) vs diamond (heavy integration)? Any organizational testing standards?
- **Known problem areas** — Has the system failed at specific boundaries before? Are there integration points that have historically been fragile or error-prone?
- **Time and resource constraints** — Is there a deadline or budget that should influence prioritization? Should the strategy optimize for speed (test critical paths only) or thoroughness?
- **CI/automation philosophy** — Gate by coverage %? Which test levels run on PR vs main? (This answer feeds Stage 7 — CI Pipeline Implementation — directly)

**Question format** (per `_common/question-format-guide.md`):
- NEVER ask questions in chat — save to .md file
- Multiple choice: A, B, C options + X) Other (MANDATORY as last option)
- [Answer]: tag after each question
- Only include meaningful, mutually exclusive options

Save questions to `aidlc-docs/verification/plans/test-strategy-questions.md`.
Wait for user to complete all [Answer]: tags before proceeding.

**After answers received:** Evaluate all responses for ambiguity. Watch for vague signals: "depends", "maybe", "not sure", "mix of", "somewhere between". Apply rigor per adaptive depth table above. If ambiguities found at Full/Standard rigor, create `aidlc-docs/verification/plans/test-strategy-clarification-questions.md` and resolve before proceeding to Step 3.

---

## Step 3: Scenario Generation

For each boundary in the map, generate concrete test scenarios:

| Category | Examples |
|---|---|
| Happy path | Normal data flows through the boundary correctly |
| Error handling | Producer sends malformed data, consumer rejects gracefully |
| Edge cases | Empty payload, maximum size payload, special characters, BigInt boundaries |
| Timing | What happens if consumer is slow, unavailable, or restarts mid-processing? |
| Ordering | Does message order matter? What if messages arrive out of order? |

Each scenario must include **example data** — not "test with valid input" but the actual input values.

**Output**: Scenario list per boundary.

---

## Step 4: Honesty Classification

Classify each scenario by its real test level. Apply the **Honesty Rule**:

> **A test that mocks the boundary between services is a unit test, not an integration test — regardless of its file name, folder, or declared classification.**

```
Does the test cross the boundary with the REAL component on the other side?
  |
  +-- YES (real Kafka, real DB, real HTTP) --> Integration test (L2+)
  |
  +-- NO (mock, stub, fake at the boundary) --> Unit test (L1)
        This test is misclassified if it claims to be "integration"
```

**Test Level Classification:**

| Level | Name | What it tests | Infrastructure needed |
|---|---|---|---|
| L1 | Unit | Single component, mocked dependencies | None |
| L2 | Integration | Two components, real boundary | Testcontainers or docker-compose |
| L3 | Replay / Golden Files | Recorded real-world data replayed through component | Recorded data files + component |
| L4 | Contract | API contract between producer and consumer | Contract testing framework |
| L5 | System | Multiple services as a complete subsystem | Docker-compose with all services |
| L6 | E2E Browser | Full stack including UI | Playwright + full backend |

**Output**: Scenarios classified by level.

---

## Step 5: Integration Sequence

Define the incremental order in which services are integrated and tested. Criteria:

1. **Dependency**: producers before consumers
2. **Stability**: integrate with the most stable service first
3. **Validation complexity**: start with the easiest boundary to verify
4. **Critical path**: the main business flow integrates first

**Output**: Numbered integration sequence (e.g., "Step 1: core solo → Step 2: core+bot → Step 3: core+bot+UI").

---

## Step 6: Greenfield/Brownfield Detection

Search the codebase for existing tests and test infrastructure:

| What to check | How | Classification |
|---|---|---|
| Test files | Glob for `**/*.spec.ts`, `**/*.test.ts`, `**/*.e2e-spec.ts` | Exist? → brownfield-tests |
| Test infrastructure | Glob for `docker-compose.test.yml`, `docker-compose.dev.yml`, `jest.config*`, `vitest.config*`, `playwright.config*` | Exist? → brownfield-infra |
| E2E harnesses | Glob for `e2e-*.sh`, `test/system/**`, `test/e2e/**` | Exist? → brownfield-e2e |

**Possible states:**
- **greenfield** — no tests, no infrastructure
- **brownfield-tests** — tests exist but no dedicated test environment
- **brownfield-full** — both tests and test infrastructure exist

**Output**: Detection state recorded in test strategy document.

---

## Step 7: Gap Analysis (Brownfield Only)

**Skip if greenfield.**

Compare the scenarios from Step 3 against existing tests:

1. For each scenario: does a test exist? At what level?
2. Apply honesty audit to existing "integration" tests — are they honest?
3. Reclassify dishonest tests (mock at boundary → L1, not L2)
4. Produce delta: what's missing, what needs rewriting, what's correct

**Output**: Gap analysis table (scenario → existing test → honest? → action needed).

---

## Step 8: Prioritization

### Critical Boundary List (deterministic derivation — mandatory)

A boundary is tagged `critical` if and only if it appears in the Critical Boundary List, derived per the rules below. This is NOT a subjective call.

1. **Brownfield projects**: the list is derived from the Reverse Engineering artifacts. A boundary is critical if it appears in (a) an Interaction Diagram labeled as a money/data-moving transaction, (b) the Component Inventory with a `trust-boundary` or `external-integration` marker, or (c) a Technology Stack entry flagged as a single point of failure.
2. **Greenfield projects**: the list is derived from the Construction Infrastructure Design + NFR Requirements. A boundary is critical if the Infrastructure Design names it as a cross-trust-zone edge, or NFR Requirements mark it as carrying money, PII, or auth material.
3. **No artifact path available** (no reverse-engineering AND no infrastructure design — e.g., a very small change flow): this stage PAUSES and emits a blocker requiring the user to either produce a minimal Critical Boundary List inline (using the 3 heuristics below as an elicitation prompt) or justify an empty list. The stage does NOT default to an empty list silently.

The three heuristics — trust-boundary crossing; money/data loss or duplication; pipeline SPOF — serve as the elicitation prompt for rule 3 only, not as a subjective filter elsewhere.

A **critical service** is any endpoint of ≥1 critical boundary. Referenced by the observability criterion in `production-readiness-gate.md`.

### Prioritization

Order all scenarios by risk:

| Priority | Criteria |
|---|---|
| P1 — Critical | Boundaries that move money, critical data, or affect system integrity |
| P2 — High | Main business flow happy paths |
| P3 — Medium | Error handling and edge cases on critical boundaries |
| P4 — Low | Non-critical boundaries, cosmetic flows |

Mark each scenario as **must-have** (P1-P2) or **nice-to-have** (P3-P4) for the production readiness gate.

### P3 Elevation on Critical Boundaries

- P3 (error handling) on a critical boundary → **must-have**.
- P3 on a non-critical boundary → nice-to-have (unchanged).
- P4 (edge cases) → nice-to-have everywhere (unchanged).

**Output**: Prioritized scenario list.

---

## Step 9: Domain Hook

**MANDATORY**: invoke `superpowers:dev-testing-strategy`. If the routing table (`.claude/routing/routing-table.json`) declares a domain skill applicable to the paths under test, invoke it as well (e.g., `Skill(blacklist-monitoring)` for viblocks-ai pipeline work).

Integrate skill recommendations into the strategy without duplicating or contradicting prior steps.

---

## Step 10: Generate Test Strategy Artifact

Create `aidlc-docs/verification/test-strategy.md` containing:

1. Boundary Map (Step 2)
2. Scenarios per boundary with example data (Step 3)
3. Honesty classification per scenario (Step 4)
4. Integration Sequence (Step 5)
5. Greenfield/Brownfield state (Step 6)
6. Gap Analysis — if brownfield (Step 7)
7. Prioritized plan (Step 8)
8. Production Readiness Criteria (derived from P1-P2 scenarios — these become the gate in Stage 8)

---

## Step 11: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Add VERIFICATION Phase section with all 8 stages as checkboxes
- Mark Test Strategy Design as in-progress

---

## Step 12: Present Completion Message

Present completion message in this structure:

1. **Completion Announcement** (mandatory):

```markdown
# 🧪 Test Strategy Design Complete
```

2. **AI Summary** (mandatory): Structured bullet-point summary:
   - Number of boundaries mapped
   - Number of scenarios generated per level (L1-L6)
   - Integration sequence summary
   - Greenfield/brownfield state
   - Gap analysis summary (if brownfield)
   - Number of must-have vs nice-to-have scenarios
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message** (mandatory):

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the test strategy at: `aidlc-docs/verification/test-strategy.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the test strategy based on your review
> ✅ **Continue to Next Stage** - Approve test strategy and proceed to **Test Environment Design**

---
```

---

## Step 13: Wait for Explicit Approval
- Do not proceed until the user explicitly approves the test strategy
- If user requests changes, update the strategy and repeat the approval process

## Step 14: Record Approval and Update Progress
- Log approval in audit.md with timestamp and complete raw user input
- Mark Test Strategy Design as complete in aidlc-state.md

---

## Step 15: Log Interaction

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## Test Strategy Design
**Timestamp**: [ISO timestamp]
**Boundaries Mapped**: [N]
**Scenarios Generated**: [N] (L1: X, L2: X, L3: X, L4: X, L5: X, L6: X)
**Integration Sequence**: [Summary]
**State**: [greenfield / brownfield-tests / brownfield-full]
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```
