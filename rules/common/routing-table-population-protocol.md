# Routing Table Population Protocol

**Status**: ACTIVE
**Scope**: Project-agnostic and orchestrator-agnostic. Applicable to any project using typed agents, with or without an SDLC orchestrator (e.g. AI-DLC).

---

## 1. Problem

The routing table (`.claude/routing/routing-table.json`) maps service paths to typed agents. When a service has no row, the enforcement hook (Layer 2, PreToolUse) blocks dispatch — but there is no formal protocol for populating the table.

Two scenarios without coverage:
1. **Brownfield first adoption**: typed-agents is adopted in an existing project. All services are "unknown" to the routing table.
2. **New incremental service**: a new service is created during the project's lifecycle. The service has no row.

---

## 2. Solution

Two population moments, same action: detect service without a row, classify stack, add row.

### Ad-hoc adoption (no SDLC orchestrator) — use `/typedAgentSetup`

For projects adopting typed-agents **without** an SDLC orchestrator (no AI-DLC, no Reverse Engineering stage producing `component-inventory.md`), the recommended path is the `/typedAgentSetup` wizard shipped with `viv-typed-agents` (tier 3+).

After running `scripts/install.sh`, invoke the wizard in Claude Code:

```
> /typedAgentSetup
```

The wizard automates **Moment 1** for ad-hoc adoption:

1. Detects project state (greenfield vs brownfield)
2. Discovers candidate service folders (stack-agnostic)
3. Classifies each as `backend` / `frontend` / `ambiguous` via skill-declared `detection:` signatures
4. Asks for the project's business domain (Crypto, WaaS, Generic)
5. Resolves agents via `(domain, business_domain, type)` lookup
6. Writes `.claude/routing/routing-table.json` (idempotent, preserves hand-edits) plus the conditional tier-4 / tier-5 outputs

See `viv-typed-agents/architecture/specs/2026-05-09-typed-agent-setup.md` for the full design.

The manual protocol below remains canonical for:
- **Orchestrator-integrated adoption** — where an SDLC orchestrator (e.g. AI-DLC) wires Moment 1 as a post-RE step, consuming inventory + stack artifacts to drive population programmatically
- **Moment 2** — incremental population when a new service is introduced mid-lifecycle (the wizard targets initial setup, not per-service additions)
- **Edge cases** — projects with non-standard layouts the wizard's discovery doesn't cover

---

### Moment 1 — Bulk Population (brownfield first adoption)

**When**: When typed-agents is first adopted in an existing project. Orchestrators with a Reverse Engineering stage typically wire this in as a post-RE step (right after RE completes, before Requirements Analysis); ad-hoc adoption runs it once during initial setup.

**Trigger** (orchestrator-driven): an inventory of components and their technology stacks is available — either from an RE-stage artifact (e.g. `component-inventory.md` + `technology-stack.md`) or from manual project audit.

**Protocol**:

1. Read the component inventory — list of services with type (Application, Infrastructure, Shared, Test)
2. Read the technology stack mapping per service (NestJS, React, Python, etc.)
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
Bulk Population: Routing Table Generated

| Domain   | Paths                                     | Implementer              | Reviewer              |
|----------|-------------------------------------------|--------------------------|-----------------------|
| Backend  | services/api/** packages/shared/**        | <domain>-implementer     | <domain>-reviewer     |
| Frontend | services/web/**                           | <domain>-implementer     | <domain>-reviewer     |
| Unmatched| services/ml-engine/** (Python/FastAPI)    | -- (no typed agent)      | --                    |

! 1 service without typed agent. Will be resolved before any code generation.
```

### Moment 2 — Incremental Population (new service introduced)

**When**: a code-generation plan introduces paths in `services/*` or `packages/*` (or any enforced route in `routing-table.json`) without a row in the routing table. Orchestrators with a Code Generation stage typically catch this during plan review (Part 1); ad-hoc adoption catches it at the moment a typed agent is first dispatched to a new path.

**Trigger** (orchestrator-driven): a plan is generated and a path-classification check detects new unmapped paths.

**Protocol**:

1. Detect new paths in the plan without a row in the routing table
2. Classify domain based on already-completed design artifacts (NFR/stack documents, infrastructure design)
3. Add a "Step 0" (or equivalent leading task) to the plan:
   - Add entry to `.claude/routing/routing-table.json`
4. User approves the complete plan (including the routing-table update) at the appropriate approval gate

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

1. The bulk-population step (Moment 1) or the incremental-population step (Moment 2) marks it as "unmatched"
2. The orchestrator (or user, if no orchestrator) presents options:
   - **A)** Create a new typed agent for the stack (see [`common/typed-agent-mechanism.md`](typed-agent-mechanism.md) — "Extensibility — Adding a New Domain")
   - **B)** Use `general-purpose` for this service (no domain patterns, lower quality guarantee)
   - **C)** Defer — do not touch this service in this cycle
3. The decision is recorded in the routing table with the corresponding row
4. If option A is chosen, the new typed agent is created BEFORE any codegen targets that service

---

## 4. Safety Net

The PreToolUse hook in `settings.json` is maintained as Layer 2 enforcement:

- If the protocol failed (Step 0 not executed, Post-RE population skipped), the hook detects `services/[a-z]` without an explicit match → hard block (exit 2)
- The hook message references the protocol: "classify before dispatching"
- The hook does NOT replace the protocol — it is the last line of defense

---

## 5. Principle

The routing table is a **living registry** that grows with the project. It is not a static file configured once. The two population moments guarantee that every service — existing or new — receives the same typed agents, IRON LAW, and Post-Implementation Chain treatment before any line of code is written.
