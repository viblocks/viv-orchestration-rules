# Test Environment Setup

**Purpose**: Implement the environment designed in Stage 2 and validate it works before running any tests.

## Prerequisites
- Test Environment Design (Stage 2) must be approved
- Test Strategy (Stage 1) must be approved

---

## Step 1: Evaluate Existing Infrastructure (Brownfield)

If brownfield, evaluate existing environment files:
- `docker-compose.dev.yml`, `docker-compose.yml` — can we extend? Fork? Replace?
- Existing bash harnesses (`e2e-*.sh`, `scripts/test-*`) — reusable?
- Existing Makefiles targets — compatible?

Decision: reuse, extend, or replace. Document rationale.

**Skip if greenfield** — proceed directly to Step 2.

---

## Step 1b: Domain Hook

Invoke applicable domain skills before dispatching the typed agent:
- **`docker-patterns`** — MANDATORY: container configuration, compose patterns, health checks
- **`infra-cloud`** — CONDITIONAL: if the environment includes cloud components (managed DBs, message brokers, external APIs)

Integrate domain skill recommendations into the typed agent prompt.

---

## Step 1c: Create Execution Plan

Create `aidlc-docs/verification/plans/test-environment-setup-plan.md`:

```markdown
# Test Environment Setup — Execution Plan

## Steps
- [ ] Evaluate existing infrastructure (brownfield) or skip (greenfield)
- [ ] Invoke domain hooks (docker-patterns, infra-cloud if applicable)
- [ ] Implement test environment (dispatch infra-devops-implementer)
- [ ] Validate service health
- [ ] Validate boundary connectivity
- [ ] Load seed data
- [ ] Execute smoke test
```

Update checkboxes immediately as each step completes — same interaction.

---

## Step 2: Implement Test Environment

**IRON LAW**: Dispatch `infra-devops-implementer` for all environment implementation work.

Implementation deliverables:
- `docker-compose.test.yml` (or extension of existing compose file) — all services for test environment
- Setup scripts — initialization, seed data loading, health wait loops
- Makefile targets — `make test-env-up`, `make test-env-down`, `make test-env-health`
- `.env.test` — test-specific environment variables
- Seed data files — minimum data needed for smoke test

**Typed agent prompt must include:**
- The Test Environment Design document (Stage 2)
- The Fidelity Table (what's EXACT, EQUIVALENT, SIMULATED)
- Existing infrastructure files to reference/extend (if brownfield)
- The Honesty Rule ("tests must use real services, not mocks at boundaries")

**MANDATORY: Execute Post-Implementation Chain.** Chain steps + 5-field output template are defined in `CLAUDE.md` → "Post-Implementation Chain (MANDATORY)" and "Post-Chain Output (MANDATORY)". Do NOT redefine inline — single source of truth.

---

## Step 3: Service Health Validation

Start the environment and verify each service is healthy:

```bash
make test-env-up
# Wait for health checks
make test-env-health
```

For each service:
- Is it running? (container status)
- Is it healthy? (health check endpoint or process check)
- Can it accept connections? (port check)

**Output**: Health check report — all services green or specific failures documented.

---

## Step 4: Connectivity Validation

Verify that boundaries between services work:

For each boundary in the Test Strategy boundary map:
- Can service A reach service B via the expected transport?
- Does the connection use the correct credentials/configuration?
- Can a simple message/request traverse the boundary?

This is NOT a functional test — it's a connectivity check. "Can A talk to B?" not "Does A's message produce the correct result in B?"

**Output**: Connectivity report — all boundaries reachable or specific failures documented.

---

## Step 5: Seed Data Setup

Load test data that enables realistic scenario execution:

- Database seed: minimum records needed for test scenarios
- Message broker: topics/queues created with correct configuration
- External services: mock servers configured with expected responses (for SIMULATED components)
- Configuration: all services have correct env vars for test environment

**Output**: Seed data scripts committed to repository.

---

## Step 6: Smoke Test

Execute 1 simple end-to-end operation to confirm the environment works as a unit:

Pick the simplest happy-path scenario from the Test Strategy (Stage 1) and run it manually:

1. Trigger the operation (e.g., send a test event, call an API endpoint)
2. Trace it through the system (logs, database changes, message delivery)
3. Verify the expected outcome occurred

**Escalation rule**: If the smoke test fails, iterate to fix. **After 3 failed iterations**, escalate to the user with:
- What was attempted
- What specifically fails
- Diagnosis of why
- Proposed solutions (if any)

Do NOT continue iterating blindly. Do NOT advance to Stage 4 with a broken environment.

---

## Step 7: Generate Setup Report

Create `aidlc-docs/verification/environment-setup-report.md`:
- Environment files created/modified (with paths)
- Health check results per service
- Connectivity validation results per boundary
- Seed data summary
- Smoke test result (pass/fail with evidence)

---

## Step 8: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark Test Environment Setup as in-progress

---

## Step 9: Present Completion Message

1. **Completion Announcement**:

```markdown
# ⚙️ Test Environment Setup Complete
```

2. **AI Summary**: Structured bullet points:
   - Environment files created (list with paths)
   - Services running and healthy (count)
   - Boundaries validated (count)
   - Smoke test result
   - Makefile targets available
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the setup report at: `aidlc-docs/verification/environment-setup-report.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the environment setup based on your review
> ✅ **Continue to Next Stage** - Approve environment setup and proceed to **Test Diagnosis**

---
```

## Step 10: Wait for Explicit Approval
- Do not proceed until the user explicitly approves
- Smoke test MUST pass before approval is valid

## Step 11: Record Approval and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## Test Environment Setup
**Timestamp**: [ISO timestamp]
**Status**: [Approved/Complete]
**Key Results**: Smoke test: PASS/FAIL; Services healthy: N/N; Boundaries validated: N/N
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- Mark Test Environment Setup as complete in aidlc-state.md
