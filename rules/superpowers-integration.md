# Superpowers Integration — Typed-Agents Core

How **typed agents** specialize Superpowers (SP) skill bindings.

> Comprehensive per-stage SP bindings for AI-DLC live in [`viblocks/aidlc-orchestrator`](https://github.com/viblocks/aidlc-orchestrator) `rules/common/superpowers-integration.md`. This page is the typed-agents-only contract.

## The override principle (typed-agents core)

When a typed agent is dispatched, **typed agents replace generic SP roles** in subagent-driven workflows:

| SP role | Typed-agent replacement |
|---|---|
| `implementer` | `<route.implementer>` (per routing-table) |
| `code-reviewer` | `<route.reviewer>` (per routing-table) |
| `spec-reviewer` | `general-purpose` (read-only; no domain coupling) |

## Skill overrides (typed-agents core)

These overrides apply **whenever a typed agent is active**, independent of any SDLC orchestrator:

| SP skill | Default trigger | Override (typed agent active) |
|---|---|---|
| `subagent-driven-development` | when work warrants subagents | Always — but the subagents are typed |
| `test-driven-development` | "implementing any feature" | Embedded in implementer IRON LAW. Never invoked separately. |
| `verification-before-completion` | "at every action" | At chain stages with `kind: verification` |
| `systematic-debugging` | universal | Universal (no override; IRON LAW applies) |
| `receiving-code-review` | when re-dispatched after review | Active during implementer re-dispatch loop |

These are **typed-agents-only** overrides. They hold whether or not AI-DLC is in the picture.

## Behavioral hierarchy (typed-agents core)

| Layer | Authority |
|---|---|
| 1. User explicit instructions | Highest |
| 2. Typed-agent IRON LAW (when dispatching) | Override generic SP triggers within the agent |
| 3. Generic SP triggers | Lowest — apply when nothing else does |

## When AI-DLC is also active

If the consumer adopts the AI-DLC orchestrator, **additional** stage-bindings come into play:

- `brainstorming` → only at CONDITIONAL AI-DLC stages
- `writing-plans` → only at AI-DLC Code Generation Part 1
- `dev-testing-strategy` → only at AI-DLC VERIFICATION Stage 1
- `docker-patterns` → only at AI-DLC VERIFICATION Stages 2-3
- (etc., per-stage matrix)

The full matrix is in [`viblocks/aidlc-orchestrator/rules/common/superpowers-integration.md`](https://github.com/viblocks/aidlc-orchestrator/blob/main/rules/common/superpowers-integration.md). When AI-DLC is active, the AI-DLC bindings take precedence over generic SP triggers (per AI-DLC `sp-precedence` rule).

## Why split

AI-DLC's per-stage bindings are valuable but they're an AI-DLC concern, not a typed-agents concern. A consumer using typed-agents without AI-DLC needs the typed-agents core overrides; they don't need AI-DLC stage bindings. See [ADR-RD-012](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-012-separate-aidlc-orchestrator.md).
