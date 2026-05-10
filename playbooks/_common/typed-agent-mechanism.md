# Typed Agent Mechanism

Universal pattern for typed agents in AI-DLC + Superpowers orchestration.

---

## 1. Mental Model

```
┌─────────────────────────────────────────────────────────────────┐
│  AI-DLC  — ORCHESTRATOR                                         │
│  WHAT to build · Why · Approval gates · Audit trail             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Superpowers  — EXECUTION ENGINE                        │   │
│  │  HOW to build it · With what discipline                 │   │
│  │  writing-plans · subagent-driven-development · TDD      │   │
│  │                                                         │   │
│  │  ┌──────────────────────────────────────────────────┐  │   │
│  │  │  Typed Agents  — SPECIALIZED EXECUTORS           │  │   │
│  │  │  Domain knowledge embedded · IRON LAW active     │  │   │
│  │  │                                                   │  │   │
│  │  │  BACKEND   <domain>-implementer                  │  │   │
│  │  │            <domain>-reviewer                     │  │   │
│  │  │                                                   │  │   │
│  │  │  FRONTEND  <domain>-implementer                  │  │   │
│  │  │            <domain>-reviewer                     │  │   │
│  │  │                                                   │  │   │
│  │  │  SECURITY  security-reviewer                     │  │   │
│  │  │                                                   │  │   │
│  │  │  INFRA     infra-devops-implementer              │  │   │
│  │  │            infra-devops-reviewer                  │  │   │
│  │  └──────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Structural enforcement (settings.json):
  deny list  → main session cannot write to domain service paths
  hook Agent → blocks dispatch of wrong agent for a domain
```

**Golden rule**: AI-DLC is the sole orchestrator. Superpowers executes with discipline. Typed agents are the only ones that write application code.

---

## 2. Why Typed Agents

### The problem they solve

`subagent-driven-development` dispatches `general-purpose` agents by default. A `general-purpose` agent starts with empty context — it does not know the domain patterns, established conventions, or project-specific constraints.

**Without typed agents**:
```
Plan approved → dispatch general-purpose → code without patterns → drift → audit findings → CRs
```

**With typed agents**:
```
Plan approved → dispatch typed agent → patterns guaranteed → correct code from the start
```

### Why not inject patterns into the dispatch prompt

Injecting all patterns in full into every dispatch = fixed token overhead per task.

With routing on-demand, the typed agent loads only the relevant patterns for the specific task — substantially lower token cost per dispatch.

---

## 3. Architecture of a Typed Agent

All typed agents follow the same structure. What varies is the domain.

```
.claude/agents/<domain>-<role>.md
│
├── Frontmatter — tools restricted to the domain
│
├── Identity
│   "You are the <role> for <domain> in this project.
│    You never write code without reading the applicable patterns."
│
├── Pre-code protocol (MANDATORY)
│   1. Read routing table → identify applicable patterns
│   2. Read pattern files (Read tool, on-demand)
│   3. Only then write code
│
├── IRON LAW
│   TDD: failing test first, minimal implementation after
│   Verification: tests pass before reporting DONE
│   Commit: after each completed task
│
└── Pattern index (routing table)
    Points to files in .claude/skills/<domain>/patterns/
```

### Backend implementer example

```
Tools: Read, Write, Edit, Glob, Grep
       Bash(pnpm *), Bash(git *), Bash(npm run *), Bash(npx vitest *)
       TodoWrite

Patterns in: .claude/skills/<domain>/patterns/[01-N].md

Routing on-demand (examples):
  New message consumer     → load patterns for messaging, idempotency, error handling
  New API endpoint         → load patterns for architecture, validation, error handling
  New module from scratch  → load patterns for architecture, observability, graceful shutdown, testing
```

### Frontend implementer example

```
Tools: Read, Write, Edit, Glob, Grep
       Bash(pnpm *), Bash(git *), Bash(npm run *), Bash(npx vitest *)
       Bash(npx playwright *)
       TodoWrite

Patterns in: .claude/skills/<domain>/patterns/[01-N].md

Routing on-demand (examples):
  New component with data display  → load patterns for state management, error boundaries, testing
  Real-time feed                   → load patterns for state management, streaming, degradation
  New view/screen                  → load patterns for state management, error boundaries, observability, testing
```

### Infra & DevOps (universal — ships with plugin)

```
Tools (implementer): Read, Write, Edit, Glob, Grep
       Bash(make *), Bash(docker *), Bash(docker compose *), Bash(gh *)
       Bash(pnpm *), Bash(git *), Bash(npm run *)
       TodoWrite

Tools (reviewer): Read, Glob, Grep, Bash(git diff *), Bash(git log *), Bash(wc *)

Skills: docker-patterns, infra-cloud, ci-cd-patterns

Scope: Dockerfiles, docker-compose*.yml, .github/workflows/**, Makefile, scripts/**, terraform/**

Used in VERIFICATION: Stages 3 (environment setup), 6 (environment fixes), 7 (CI pipelines)
```

### Security reviewer (universal — ships with plugin)

```
Tools: Read, Glob, Grep, Bash(git diff *), Bash(git log *), Bash(wc *)

Checklist: .claude/skills/owasp-security/SKILL.md

Role: conditional reviewer in Post-Implementation Chain.
  Invoked only if changed files match trigger criteria.
  The domain typed agent writes security-relevant code;
  security-reviewer audits it against the OWASP + domain checklist.

Conditional dispatch:
  Changed files match trigger criteria → dispatch security-reviewer
  No match → skip, log "security review: N/A"
  CRITICAL/HIGH → BLOCK
  MEDIUM/LOW → ADVISORY
```

### Component/Module Reuse Validation

Both typed reviewers enforce a **reuse-before-reinvent** check:

- **Frontend reviewer**: triggers on new component files — finding is MEDIUM (potential duplication); escalates to HIGH if an existing component clearly covers the use case and no "existing components evaluated" rationale exists in the design spec.
- **Backend reviewer**: triggers on new service/module files — same escalation logic.

Integration with DESIGN PATH: the design spec section "Existing components evaluated" provides the rationale that prevents escalation.

---

## 4. Routing Table — Central Decision Table

**Single source of truth**: `<project-root>/.claude/routing/routing-table.json`

The JSON contains `domain`, `paths`, `implementer`, `reviewer` per domain, and optionally a `note` field documenting design decisions (why an implementer or reviewer is `null`, what the reviewer actually covers, etc.). This file is the only source — do not duplicate in any other document.

**Unknown service rule**: Any `services/*` or `packages/*` that has NO explicit entry in `routing-table.json` → **STOP**. Do not assume domain. Identify the service's stack, add an entry to `routing-table.json`, only then dispatch. If no typed agent exists for that stack, escalate to the user.

**Notes**:
- **Spec reviewer** is always `general-purpose` — only reads code, needs no domain knowledge to compare spec vs implementation.
- **Security** is an additional reviewer on top of the domain reviewer, not a replacement.
- **packages/shared** is always its own task in the plan — it is the contract, goes first.

---

## 5. Integration with subagent-driven-development

`subagent-driven-development` defines three roles per task. Typed agents are **specializations** of the generic Superpowers roles:

```
Superpowers default           Project-specific typed
────────────────────────      ────────────────────────────────────────
implementer → general-purpose  →  <domain>-implementer (backend)
                                   <domain>-implementer (frontend)
                                   infra-devops-implementer (infra/CI/CD)

spec reviewer → general-purpose  →  general-purpose (no change)

code reviewer → superpowers:code-reviewer  →  <domain>-reviewer (backend)
                                               <domain>-reviewer (frontend)
                                               infra-devops-reviewer (infra/CI/CD)
                                               dev-testing-strategy-reviewer (testing)
```

### Complete flow — backend task

```
Plan task: "Implement <feature> in services/<backend-service>/"

1. Main session: detect domain → routing-table lookup → Backend
2. Dispatch implementer: <domain>-implementer
   └── Identify task type
   └── Route to applicable patterns
   └── Read pattern files (on-demand)
   └── Write failing test (TDD)
   └── Implement until test passes
   └── Self-review → commit → DONE signal

3. Dispatch spec reviewer: general-purpose
   └── Verify code matches exact spec
   └── ✅ Spec compliant / ❌ Issues with file:line

4. If ✅ → Dispatch code reviewer: <domain>-reviewer
   └── Audit against domain patterns
   └── ✅ Approved / Issues with severity

5. If ✅ → task complete → next plan task
```

### Complete flow — frontend task

```
Plan task: "Implement <feature> in services/<frontend-service>/"

1. Main session: detect domain → routing-table lookup → Frontend
2. Dispatch implementer: <domain>-implementer
   └── Identify task type
   └── Route to applicable patterns
   └── Read pattern files (on-demand)
   └── Write failing test (TDD)
   └── Implement until test passes
   └── Self-review → commit → DONE signal

3. Dispatch spec reviewer: general-purpose → ✅ / ❌
4. If ✅ → Dispatch code reviewer: <domain>-reviewer → ✅ / Issues
5. If ✅ → task complete → next plan task
```

### Security review — conditional gate

```
After typed code reviewer passes:

1. Main session: evaluate changed files against trigger criteria
   └── Controllers, DTOs, guards, auth, frontend forms, config, package.json, pipeline/enrichment paths
2. If match → Dispatch security-reviewer
   └── Loads owasp-security skill
   └── Classifies files → applicable categories
   └── File-by-file review → cross-cutting checks
   └── Structured report
3. If CRITICAL/HIGH → resolve → back to verification step
4. If PASS → proceed to commit
5. If no match → skip, log N/A → proceed to commit
```

### Complete flow — cross-domain task

```
CR: "Fix event contract between service A and B, update UI types"

Plan divided (mandatory):
  Task 1: Update shared schema    → packages/shared/  → <backend-domain>-implementer
  Task 2: Update service A        → services/<A>/     → <backend-domain>-implementer
  Task 3: Update service B        → services/<B>/     → <backend-domain>-implementer
  Task 4: Update UI types         → services/<ui>/    → <frontend-domain>-implementer

Rule: packages/shared is ALWAYS Task 1 — it is the contract, defines the rest.
Execution: Task 1 → Tasks 2 + 3 (parallel possible) → Task 4
```

### TDD — not invoked separately

`superpowers:test-driven-development` **is not invoked as an additional skill** when typed agents are used. TDD is embedded in the IRON LAW of every typed implementer:

```
IRON LAW (all typed implementers):
  Write the failing test first.
  Minimal implementation to make it pass.
  Never report DONE without passing tests.
```

Dispatching a typed implementer = TDD guaranteed, no additional instruction needed.

### Post-Implementation Chain — MANDATORY

After any typed implementer completes, see `_common/post-implementation-chain.md`.

---

## 7. Extensibility — Adding a New Domain

**4-step pattern. Nothing else changes.**

### Step 1 — Create the domain skill

```
.claude/skills/<domain>/
├── SKILL.md                     ← pattern index + routing table
└── patterns/
    ├── 01-<pattern-name>.md
    ├── 02-<pattern-name>.md
    └── ...
```

### Step 2 — Create the agents

```
.claude/agents/<domain>-implementer.md   ← if the domain produces new code
.claude/agents/<domain>-reviewer.md      ← for domain auditing
```

For domains that only audit existing code (like security), only a reviewer is needed. The domain implementer (backend or frontend) still writes the code.

### Step 3 — Add entry to `.claude/routing/routing-table.json`

Add an object to the `domains` array with `name`, `paths`, `implementer`, `reviewer`, and optional `skill`. The schema matches what `bootstrap-enforcement.sh` writes (`templates/routing-table.json.template`).

### Step 4 — Update the hook allowlist in settings.json

Add the new agent to the Layer 2 hook allowlist.

**Nothing else changes.** AI-DLC, subagent-driven-development, and TDD work the same way.

### Routing Table Population Protocol

The routing table is a living registry that grows with the project. It is populated at two moments:

1. **Post-RE (brownfield first adoption)**: After Reverse Engineering, before Requirements Analysis. Read `component-inventory.md` + `technology-stack.md` to generate bulk rows.
2. **Pre-Dispatch (new service incremental)**: In Code Generation Part 1, Step 0 of the plan adds the row before dispatching typed agents.

Services with a stack that has no existing typed agent are marked "unmatched" and resolved at Workflow Planning with a user decision.

---

## 8. What Does Not Change

- The AI-DLC flow (approval gates, audit.md, aidlc-state.md)
- The `subagent-driven-development` skill — used the same way, with typed agents as executors
- The pattern files (`.claude/skills/<domain>/patterns/`)

---

## 9. Design Decisions

| Decision | Discarded alternative | Reason |
|---|---|---|
| Patterns in routing table + on-demand | Full patterns embedded in system prompt | Substantially lower token cost per dispatch |
| Deny list blocks main session | Advisory hook on Edit/Write | Advisory can be ignored; deny list cannot |
| Typed agents with restricted tools | Full tool set | Smaller blast radius if agent deviates from scope |
| Spec reviewer remains general-purpose | Typed spec reviewer | Only reads code — no domain knowledge needed to compare spec vs implementation |
| Cross-domain = split in plan, never in execution | Multi-domain agent | Atomic commits, simple rollback, correct typed agent per task |
| Security as additional reviewer on top of domain | Separate security implementer | Code remains backend or frontend — same implementer, additional reviewer |
| 4 steps to add a domain | Centralized configuration | Low coupling — each domain is autonomous, does not modify central logic |

---

## 10. Change Flow Integration

The change flow is a parallel operating mode alongside greenfield for reactive changes (GitHub / Linear issues). Typed agents are used the same way — the difference is the entry point.

### Triage → Path → Typed Agent

```
Issue arrives → Triage gate (4 questions) → Path selection → Typed agent dispatch
```

The 4 paths reuse the same routing table (§4) and Post-Implementation Chain (§5 / `_common/post-implementation-chain.md`). No new agents or patterns are needed.

### Reference

- Agnostic strategy: `_common/core-change-flow-protocol.md`
- Full typed-agent mechanism doc: this file (`_common/typed-agent-mechanism.md`)
