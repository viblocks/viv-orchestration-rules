# Superpowers Integration Playbook

How typed agents specialize Superpowers (SP) skill bindings.

> Superpowers is an **external skill system**, not part of this strategy. See [ADR-002](../architecture/decisions/ADR-002-external-deps-referenced.md).

## Scope

This playbook applies when the consumer adopts Superpowers. Consumers without SP can ignore.

## The override principle

When the typed-agents strategy is active, **typed agents replace generic SP agents** in subagent-driven workflows:

| SP role | Typed-agent replacement |
|---|---|
| `implementer` | `<route.implementer>` (per routing-table) |
| `code-reviewer` | `<route.reviewer>` (per routing-table) |
| `spec-reviewer` | `general-purpose` (read-only; no domain coupling) |

Other SP roles (architect, planner, etc.) are unchanged.

## Skill bindings — overrides during typed-agent flows

Generic SP skill triggers ("use TDD when implementing any feature") are OVERRIDDEN inside typed-agent flows because the typed agent embeds the discipline.

| SP skill | Default trigger | Override (when typed agent active) |
|---|---|---|
| `subagent-driven-development` | when work warrants subagents | Always — but the subagents are typed, not generic |
| `test-driven-development` | "implementing any feature" | Embedded in implementer IRON LAW. Never invoked separately. |
| `verification-before-completion` | "at every action" | Only at chain stages with `kind: verification` |
| `writing-plans` | "before any implementation" | Only at the planning phase of the implementer's protocol |
| `systematic-debugging` | universal | Universal (no override; IRON LAW applies regardless) |
| `receiving-code-review` | when re-dispatched after review | Active during implementer re-dispatch loop in the post-implementation chain |
| `brainstorming` | "before any creative work" | Only at CONDITIONAL stages in the AI-DLC binding |

The first six skills are explicitly listed in [SPEC §8.2](https://github.com/viblocks/viv-typed-agents/blob/main/SPEC.md) as the integration surface. `brainstorming` is included because AI-DLC stages reference it, but it is not part of the SPEC §8.2 contract.

## When generic SP triggers ARE active

Generic SP behavior is unmodified when:

1. The change is in **Class B** paths (per `routing-table.json` `enforced: false`)
2. AI-DLC is NOT active AND the path doesn't require typed-agent dispatch
3. The work is exploration/research outside any code change

In short: typed agents override generic SP **only when typed agents are dispatched**. Outside the IRON LAW's scope, SP defaults apply.

## Subagent-driven development with typed agents

When using `superpowers:subagent-driven-development`, replace the generic agent slots:

```
implementer    → <typed-implementer-per-routing>
spec reviewer  → general-purpose (unchanged; spec review is read-only)
code reviewer  → <typed-reviewer-per-routing>
```

The typed implementer auto-loads its required skills (e.g. `nestjs-backend`, `crypto-backend`) so SP doesn't need to inject them.

## Behavioral hierarchy

| Layer | Authority |
|---|---|
| 1. User explicit instructions (CLAUDE.md, direct request) | Highest — always wins |
| 2. AI-DLC stage bindings (when AI-DLC is active) | Override generic SP triggers |
| 3. Typed-agent IRON LAW (when dispatching to a typed agent) | Override generic SP triggers within the agent |
| 4. Generic SP triggers | Lowest — apply when nothing else does |

If layer 1 says "skip TDD" and layer 3 says "TDD is mandatory," **layer 1 wins**. The user is in control.

## Why override at all

SP skills are designed for general-purpose use. Typed agents are designed for a specific stack/domain — their system prompts already embed the discipline SP would inject. Letting generic SP triggers fire alongside typed-agent dispatch causes:
- Duplicate skill activation (TDD invoked twice)
- Skill conflicts (generic vs. domain-specific guidance)
- Out-of-sequence invocation (e.g. brainstorming when the typed agent is already executing)

The override doctrine ensures **one source of guidance per phase**.

## Reference: where SP lives

A consumer adopting Superpowers installs it as a separate plugin/system. This strategy does not bundle SP content.
