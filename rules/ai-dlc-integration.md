# AI-DLC Integration — Pointer

> AI-DLC integration content lives in a separate product: [`viblocks/aidlc-orchestrator`](https://github.com/viblocks/aidlc-orchestrator).
>
> Per [ADR-RD-012](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-012-separate-aidlc-orchestrator.md), the typed-agents strategy and the AI-DLC orchestrator are separate products linked by **dependency injection** — AI-DLC depends on typed-agents for codegen; typed-agents has zero knowledge of AI-DLC.

## What this means for you

| If you adopt | You install | Where to read |
|---|---|---|
| Typed agents only | `git clone viv-typed-agents && ./install.sh ~/proj --tier 5` | This repo (the 5 entry points + `rules/common/` typed-agents core) |
| AI-DLC + typed agents | (above) **plus** `git clone aidlc-orchestrator` | aidlc-orchestrator's `rules/ai-dlc/` per-stage rules |

## What stays in this repo

The typed-agents strategy needs **its own** orchestration:

- `rules/dispatch-protocol.md` — IRON LAW operational
- `rules/post-implementation-chain.md` — chain orchestration
- `rules/issue-driven-flow.md` — autonomous change flow
- `rules/common/iron-law.md` — IRON LAW formal statement
- `rules/common/typed-agent-mechanism.md` — typed agents mechanics
- `rules/common/subagent-dispatch-contract.md` — dispatch contract
- `rules/common/post-implementation-chain.md` — chain rule definition
- `rules/common/routing-table-population-protocol.md` — routing population
- `rules/common/code-quality-rules.md` — SOLID enforcement
- `rules/common/debugging-gate.md` — L5 hook contract
- `rules/common/enforcement-architecture.md` — hook layers
- `rules/common/git-workflow.md` — git conventions

These are **typed-agents-core** — independent of any external SDLC orchestrator.

## What moved to aidlc-orchestrator

- All `rules/ai-dlc/<phase>/` per-stage rules (Inception → Construction → Verification → Deployment → Operations)
- AI-DLC workflow disciplines: `adaptive-execution`, `depth-levels`, `workflow-changes`, `stage-structural-patterns`, `process-overview`
- AI-DLC consumer experience: `welcome-message`, `aidlc-docs-structure`, `audit-and-logging`, `session-continuity`, `terminology`
- AI-DLC artifact conventions: `content-validation`, `ascii-diagram-standards`, `question-format-guide`, `error-handling`
- AI-DLC quality disciplines: `overconfidence-prevention`, `friction-reporting`, `frontend-change-discipline`
- Comprehensive Superpowers integration (per-stage SP bindings): `superpowers-integration`, `sp-precedence`
- AI-DLC change flow: `core-change-flow-protocol` (the F1-F4 paths AI-DLC adds on top of `issue-driven-flow`)
- Opt-in extensions: `extensions/security/baseline/`, `extensions/testing/property-based/`

## Why this split

AI-DLC is a **superset orchestrator** that uses typed-agents for codegen. Coupling AI-DLC content into typed-agents would:

- Force consumers wanting only typed-agents to vendor 60+ AI-DLC files
- Give `viv-orchestration-rules` two reasons to change (typed-agents AND AI-DLC) — SRP violation
- Invert the dependency direction (typed-agents shouldn't know about AI-DLC; AI-DLC SHOULD know about typed-agents)

The fix follows DIP: AI-DLC injects typed-agents at codegen stages. See ADR-RD-012.
