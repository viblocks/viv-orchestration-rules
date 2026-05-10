# Subagent Dispatch Contract

**Purpose**: Formal contract for every Agent() call during AI-DLC workflow stages that creates or modifies a code artifact. Ensures consistent typed agent dispatch, chain enforcement, and observable evidence — the same formalism as the issue-driven change flow.

**Loaded by**: `common/superpowers-integration.md` global rule. Do not load independently.

---

## Core Rule

**Every creation or modification of a code artifact during AI-DLC workflow stages MUST be executed by a subagent — never inline by the main session.**

The main session orchestrates only:
1. Generates the `[DISPATCH CONTRACT]` just-in-time before each dispatch
2. Calls `Agent()` with the contract embedded
3. Verifies the returned evidence (completeness + no explicit failures)
4. Marks the plan step complete and advances

The main session NEVER directly creates or edits code artifacts.

---

## Trigger — Code Artifact Taxonomy

The dispatch contract is required when an `Agent()` call will create or modify any of the following:

| Category | Glob patterns | Typed agent |
|---|---|---|
| Application code | `services/*/src/**/*.ts`, `packages/*/src/**/*.ts`, `services/ui/src/**/*.tsx` | backend-crypto-implementer, frontend-crypto-implementer |
| Tests | `**/*.spec.ts`, `**/*.test.ts`, `e2e/**/*.ts` | backend-crypto-implementer, frontend-crypto-implementer |
| Schema | `**/prisma/schema.prisma` | backend-crypto-implementer |
| Infrastructure/CI | `Dockerfile*`, `docker-compose*.yml`, `.github/workflows/*.yml`, `Makefile` | infra-devops-implementer |

**Boundary rule**: Does a machine execute/interpret it? → contract required. Does a human read it? → no contract.

Artifacts that do NOT trigger the contract: `aidlc-docs/**/*.md`, `docs/**/*.md`, spec files, plan files, the dispatch contract itself.

**Precondition**: A dispatch contract can only be generated when the file paths of the task are known. If paths are unknown, the task is not ready to dispatch — complete analysis/planning first.

---

## Contract Generation — Just-in-Time

The contract is generated **immediately before calling each `Agent()`** — not during planning.

At dispatch time, the main session:
1. Reads the task definition (from the plan step)
2. Resolves the domain from the task's file paths using `routing-table.json`
3. Resolves typed implementer + typed reviewer from the routing table
4. Generates the `[DISPATCH CONTRACT]` block
5. Calls `Agent()` with the contract as the first section of the prompt

**Note**: The domain skill is NOT declared in the contract. The typed implementer already knows which domain skill to invoke as part of its own definition.

---

## Contract Template

Place this block at the **start** of every qualifying `Agent()` prompt:

```
[DISPATCH CONTRACT]
Typed implementer : <agent from routing-table.json for the paths in this task>
Typed reviewer    : <reviewer agent from routing-table.json>
Security gate     : CONDITIONAL
  → After implementation: if changed files include controllers, DTOs, guards,
    auth, frontend forms, config with secrets/CORS, package.json, or
    chain/enrichment paths → dispatch security-reviewer
  → If no files match → log "N/A — no security-sensitive paths"
  → If CRITICAL/HIGH findings → escalate to main session before commit
Chain             : TDD → implement → verify →
                    typed-review → [security-reviewer if gate applies] → commit
Acceptance criteria:
  - <specific, verifiable criteria for this task>

[TASK]
<clear description of what to implement — file paths, behavior, constraints>

[EVIDENCE REQUIRED]
Respond with this exact format when done:
  Verification : [PASS/FAIL] — N/N tests, build [OK/FAIL]
  Code review  : [agent name] — [PASS / N issues (severity)]
  Security     : [security-reviewer — PASS/FAIL] or [N/A — reason]
  Commits      : [hash(es)]
```

---

## Evidence Verification — Main Session

When the subagent returns, the main session runs two checks only:

| Check | Rule |
|---|---|
| **Completeness** | All 4 evidence fields present (Verification, Code review, Security, Commits) |
| **No explicit failures** | No field reports FAIL |

**If both pass** → mark plan step `[x]`, advance to next step.

**If either fails:**
- Missing field → treat as FAIL, escalate to user
- Explicit FAIL in any field → escalate to user

**Rationale**: The subagent's IRON LAW already ran `verification-before-completion`, typed reviewer, and security gate before generating evidence. The main session is the recipient of evidence, not the re-validator of the chain.
