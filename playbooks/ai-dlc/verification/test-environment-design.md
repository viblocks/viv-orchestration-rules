# Test Environment Design

**Purpose**: Design a test environment that faithfully replicates production topology.

## Prerequisites
- Test Strategy Design (Stage 1) must be approved
- Infrastructure Design from Construction (if exists)
- NFR Design from Construction (if exists)

---

## Step 1: Import Infrastructure Design

Read infrastructure artifacts from Construction:
- `aidlc-docs/construction/{unit-name}/infrastructure-design/` — for each unit
- `aidlc-docs/construction/{unit-name}/nfr-design/` — for each unit
- `aidlc-docs/construction/shared-infrastructure.md` — if exists

If no Infrastructure Design was executed in Construction, note this — reconciliation in Step 2 becomes more extensive.

**Output**: Infrastructure baseline summary.

---

## Step 1b: Clarifying Questions

Present infrastructure baseline summary to the user, then generate clarifying questions.

**Adaptive depth** — use greenfield/brownfield state from Stage 1 (Test Strategy Design) Step 12 detection:

| State | Rigor | Behavior |
|---|---|---|
| `greenfield` | Full | Resolve all ambiguities before proceeding |
| `brownfield-tests` | Standard | Detect, report, follow-up if ambiguous |
| `brownfield-full` | Light | Detect, report, proceed if user accepts |

**Question categories to evaluate** (ask only where the answer isn't obvious from artifacts):

- **Infrastructure constraints** — What's available for the test environment? Local Docker only, or cloud resources available? Memory/CPU limits on the development machine?
- **Fidelity tolerance** — Which components MUST be exact replicas vs acceptable as simulations? Are there components where simulation is unacceptable (e.g., "we must test against real Kafka, not Redpanda")?
- **Data requirements** — What seed data is needed? Are there data sensitivity constraints (no production data in test env)? Minimum dataset size for meaningful tests?
- **Environment sharing** — Solo developer or team? Persistent environment or ephemeral per test run?
- **External service access** — Are test/sandbox credentials available for external services? Are there rate limits or cost concerns with external service usage in tests?

**Question format** (per `_common/question-format-guide.md`):
- NEVER ask questions in chat — save to .md file
- Multiple choice: A, B, C options + X) Other (MANDATORY as last option)
- [Answer]: tag after each question
- Only include meaningful, mutually exclusive options

Save questions to `aidlc-docs/verification/plans/test-environment-questions.md`.
Wait for user to complete all [Answer]: tags before proceeding.

**After answers received:** Evaluate all responses for ambiguity. Watch for vague signals: "depends", "maybe", "not sure", "mix of", "somewhere between". Apply rigor per adaptive depth table above. If ambiguities found at Full/Standard rigor, create `aidlc-docs/verification/plans/test-environment-clarification-questions.md` and resolve before proceeding to Step 2.

---

## Step 2: Reconciliation & Deepening

Evaluate if the infra design from Construction is concrete enough to replicate in a test environment:

| Scenario | Action |
|---|---|
| Infra Design complete and detailed | Quick validation — confirm all components are accounted for, proceed |
| Infra Design exists but superficial | Deepen only what's needed for test replication (missing external services, networking, data stores) |
| Infra Design was skipped | Perform mini Infrastructure Design focused on production topology — just enough to know what to replicate |
| Greenfield, no infra at all | Full production topology design — this becomes the de facto Infrastructure Design |

### Completeness Check (MANDATORY before declaring "no deepening needed")

Before classifying as "complete and detailed", ALL of the following must be true:

1. **System-wide coverage**: Does the infra design cover ALL services/units, not just one? If only one unit has infrastructure design, the topology is incomplete — classify as "exists but superficial".
2. **Pinned versions**: Are all container images version-pinned (no `latest` tags)? If not, deepen to resolve versions from actual deployment files (Dockerfiles, docker-compose).
3. **Communication map**: Is there a service-to-service communication map including message broker topics, REST endpoints, reverse proxy routing, and external APIs? If not, deepen.
4. **External services catalog**: Are external service dependencies catalogued with their access method (RPC URL pattern, API key env var, circuit breaker params, fail-safe behavior)? If not, deepen.
5. **Reverse proxy / ingress**: If a reverse proxy (nginx, Traefik, etc.) fronts application services, are its routing rules documented? If not, deepen.
6. **Networking**: Are ports, protocols, and Docker internal DNS names documented for all inter-service communication? If not, deepen.

**If ANY check fails** → scenario is "exists but superficial", not "complete and detailed". Deepen and produce `production-infrastructure-blueprint.md`.

**Key questions to resolve:**
- What external services does the system depend on? (databases, message brokers, third-party APIs, blockchain RPCs)
- How do services discover and communicate with each other? (DNS, env vars, service mesh)
- What data needs to exist for the system to function? (seed data, configuration, credentials)

**Output**: Reconciled infrastructure design. If gaps were found and deepened, update `aidlc-docs/verification/production-infrastructure-blueprint.md`.

---

## Step 3: Production Topology Map

Define the complete production topology:

```
For each component:
  - Service name
  - Type (application service, database, message broker, external API, cache, etc.)
  - Communication (who talks to whom, via what protocol)
  - Data persistence (what data does it store/manage)
  - External dependencies (third-party services, blockchain networks, etc.)
```

**Output**: Production topology diagram (ASCII or Mermaid — validate syntax per `_common/content-validation.md`).

---

## Step 4: Test Environment Design

Design the test environment as a replica of production. For each production component, decide:

| Decision | Options |
|---|---|
| **EXACT replica** | Same technology, same version (e.g., PostgreSQL 15 in production → PostgreSQL 15 via Testcontainers) |
| **EQUIVALENT substitute** | Same technology family, slightly different form (e.g., AWS MSK → Redpanda local) |
| **SIMULATED** | Mock, stub, or simplified version (e.g., mainnet RPC → Anvil fork, Telegram API → mock server) |

Design decisions must be justified — why EQUIVALENT instead of EXACT? What risk does SIMULATED introduce?

**Output**: Test environment architecture document.

---

## Step 5: Fidelity Assessment

Create the Fidelity Table — makes risk explicit for each component:

```markdown
| Component | Production | Test Environment | Fidelity | Risk |
|---|---|---|---|---|
| [component] | [prod tech] | [test tech] | EXACT/EQUIVALENT/SIMULATED | [risk description] |
```

For each SIMULATED component, document:
- What behavior is lost in simulation
- What bugs could hide behind the simulation
- Mitigation strategy (if any)

**Output**: Fidelity table in the test environment design document.

---

## Step 6: Local vs Cloud Decision

Based on environment complexity and resource requirements:

| Factor | Local | Cloud |
|---|---|---|
| All services run in Docker | Preferred | Overkill |
| External managed services needed (RDS, MSK) | Cannot replicate | Required |
| Team size > 1 needs shared environment | Inconsistent | Preferred |
| Resource-intensive (GPU, large datasets) | May not fit | Required |

**Output**: Recommendation with justification.

---

## Step 7: Domain Hook

Invoke applicable domain skills:
- **`docker-patterns`** — MANDATORY: docker-compose.test.yml design decisions
- **`infra-cloud`** — MANDATORY: production topology design and reconciliation (cloud services, IaC, networking)
- **`brainstorming`** — CONDITIONAL: when multiple viable topology options exist (local vs cloud, tech choices, fidelity tradeoffs)

Integrate recommendations into the environment design.

---

## Step 8: Generate Artifacts

Create:
- `aidlc-docs/verification/test-environment-design.md` — complete environment design with topology, fidelity table, local/cloud decision
- `aidlc-docs/verification/production-infrastructure-blueprint.md` — reconciled infra design ready for DEPLOYMENT (only if reconciliation deepened the original design)

---

## Step 9: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark Test Environment Design as in-progress

---

## Step 10: Present Completion Message

1. **Completion Announcement**:

```markdown
# 🏗️ Test Environment Design Complete
```

2. **AI Summary**: Structured bullet points:
   - Production components mapped
   - Fidelity breakdown (N EXACT, N EQUIVALENT, N SIMULATED)
   - Key risks from SIMULATED components
   - Local vs Cloud recommendation
   - Whether reconciliation deepened the infra design
   - DO NOT include workflow instructions
   - Include extension compliance summary per CLAUDE.md Extensions Enforcement rules (compliant/non-compliant/N/A per enabled extension)

3. **Formatted Workflow Message**:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the test environment design at: `aidlc-docs/verification/test-environment-design.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the environment design based on your review
> ✅ **Continue to Next Stage** - Approve environment design and proceed to **Test Environment Setup**

---
```

## Step 11: Wait for Explicit Approval
- Do not proceed until the user explicitly approves

## Step 12: Record Approval and Update Progress

**MANDATORY**: Log the stage in `aidlc-docs/audit.md`:

```markdown
## Test Environment Design
**Timestamp**: [ISO timestamp]
**Status**: [Approved/Complete]
**Key Results**: Fidelity breakdown: N EXACT, N EQUIVALENT, N SIMULATED; Local/Cloud: [recommendation]
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[Action taken]"

---
```

- Mark Test Environment Design as complete in aidlc-state.md
