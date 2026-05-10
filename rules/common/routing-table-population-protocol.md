# Routing Table Population Protocol

**Status**: ACTIVE
**Scope**: Project-agnostic. Applicable to any project using AI-DLC + Superpowers with typed agents.

---

## 1. Problem

The routing table (`.claude/routing/routing-table.json`) maps service paths to typed agents. When a service has no row, the enforcement hook (Layer 2, PreToolUse) blocks dispatch — but there is no formal protocol for populating the table.

Two scenarios without coverage:
1. **Brownfield first adoption**: AI-DLC is adopted in an existing project. All services are "unknown" to the routing table.
2. **New incremental service**: AI-DLC creates a new service during Code Generation. The service has no row.

---

## 2. Solution

Two population moments, same action: detect service without a row, classify stack, add row.

### Moment 1 — Post-RE Population (brownfield first adoption)

**When**: Right after Reverse Engineering completes, before Requirements Analysis.

**Trigger**: RE generates `component-inventory.md` and `technology-stack.md`.

**Protocol**:

1. Read `component-inventory.md` — list of services with type (Application, Infrastructure, Shared, Test)
2. Read `technology-stack.md` — stack per service (NestJS, React, Python, etc.)
3. For each Application or Shared service:
   - Classify domain by stack:
     - NestJS/Express/Fastify → Backend
     - React/Vue/Angular → Frontend
     - Other → Unmatched
   - If stack has existing typed agent → generate row
   - If not → mark as "unmatched"
4. Add entries to `.claude/routing/routing-table.json`
5. Update hook allowlist in `settings.json` if new domains require specific gates
6. User approves as part of the RE approval gate (no new gate added)

**Example output**:

```
Post-RE: Routing Table Generated

| Domain   | Paths                                     | Implementer              | Reviewer              |
|----------|-------------------------------------------|--------------------------|-----------------------|
| Backend  | services/api/** packages/shared/**        | <domain>-implementer     | <domain>-reviewer     |
| Frontend | services/web/**                           | <domain>-implementer     | <domain>-reviewer     |
| Unmatched| services/ml-engine/** (Python/FastAPI)    | -- (no typed agent)      | --                    |

! 1 service without typed agent. Will be resolved in Workflow Planning.
```

### Moment 2 — Pre-Dispatch Population (new incremental service)

**When**: Code Generation Part 1 (Planning) produces a plan with paths in `services/*` or `packages/*` without a row in the routing table.

**Trigger**: `writing-plans` generates the plan and the orchestrator detects new paths.

**Protocol**:

1. Detect new paths in the plan without a row in the routing table
2. Classify domain based on already-completed design artifacts (NFR Requirements define stack, Infra Design defines deployment)
3. Add Step 0 to the plan:
   - Add entry to `.claude/routing/routing-table.json`
4. User approves the complete plan (including Step 0) at the Code Gen Part 1 approval gate

**Example output**:

```
Step 0: Routing Table Update
  - New service: services/analytics/
  - Stack: NestJS (defined in NFR Requirements)
  - Add entry to routing-table.json:
    { "domain": "backend", "paths": ["services/analytics/**"], "implementer": "<domain>-implementer", "reviewer": "<domain>-reviewer" }
```

---

## 3. Unmatched Services

When a service uses a stack without an existing typed agent:

1. Post-RE or Step 0 marks it as "unmatched"
2. In Workflow Planning, the orchestrator presents options:
   - **A)** Create a new typed agent for the stack (see `common/typed-agent-mechanism.md` — "Extensibility — Adding a New Domain")
   - **B)** Use `general-purpose` for this service (no domain patterns, lower quality guarantee)
   - **C)** Defer — do not touch this service in this cycle
3. The decision is recorded in the routing table with the corresponding row
4. If option A is chosen, the new typed agent is created BEFORE Code Generation

---

## 4. Safety Net

The PreToolUse hook in `settings.json` is maintained as Layer 2 enforcement:

- If the protocol failed (Step 0 not executed, Post-RE population skipped), the hook detects `services/[a-z]` without an explicit match → hard block (exit 2)
- The hook message references the protocol: "classify before dispatching"
- The hook does NOT replace the protocol — it is the last line of defense

---

## 5. Principle

The routing table is a **living registry** that grows with the project. It is not a static file configured once. The two population moments guarantee that every service — existing or new — receives the same typed agents, IRON LAW, and Post-Implementation Chain treatment before any line of code is written.
