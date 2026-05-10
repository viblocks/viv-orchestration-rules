# Superpowers Integration — Index

How typed agents specialize Superpowers (SP) skill bindings.

> Superpowers is an **external skill system**, not part of this strategy. Per [ADR-002](../architecture/decisions/ADR-002-external-deps-referenced.md) (revised), the **integration content** is hosted here; SP itself remains external.

## Where the canonical content lives

The full bindings (~25KB, per-stage SP skill activation rules) live in:

**[`common/superpowers-integration.md`](common/superpowers-integration.md)**

This page is a thin index pointing there. It exists for two reasons:
1. Quick discovery from the playbook root
2. Backward compatibility with consumers that linked here before the extension

## The override principle

When the typed-agents strategy is active, **typed agents replace generic SP agents** in subagent-driven workflows:

| SP role | Typed-agent replacement |
|---|---|
| `implementer` | `<route.implementer>` (per routing-table) |
| `code-reviewer` | `<route.reviewer>` (per routing-table) |
| `spec-reviewer` | `general-purpose` (read-only; no domain coupling) |

## Skill binding overrides (summary)

| SP skill | Default trigger | Override (when typed agent active) |
|---|---|---|
| `subagent-driven-development` | when work warrants subagents | Always — but the subagents are typed |
| `test-driven-development` | "implementing any feature" | Embedded in implementer IRON LAW. Never invoked separately. |
| `verification-before-completion` | "at every action" | Only at chain stages with `kind: verification` |
| `writing-plans` | "before any implementation" | Only at Code Generation Part 1 |
| `systematic-debugging` | universal | Universal (no override; IRON LAW applies) |
| `receiving-code-review` | when re-dispatched after review | Active during implementer re-dispatch loop |
| `brainstorming` | "before any creative work" | Only at CONDITIONAL stages in AI-DLC bindings |

The first six skills are listed in [SPEC §8.2](https://github.com/viblocks/viv-typed-agents/blob/main/SPEC.md) as the integration surface. `brainstorming` is included because AI-DLC stages reference it.

## Behavioral hierarchy

| Layer | Authority |
|---|---|
| 1. User explicit instructions (CLAUDE.md, direct request) | Highest — always wins |
| 2. AI-DLC stage bindings (when AI-DLC is active) | Override generic SP triggers |
| 3. Typed-agent IRON LAW (when dispatching to a typed agent) | Override generic SP triggers within the agent |
| 4. Generic SP triggers | Lowest — apply when nothing else does |

If layer 1 says "skip TDD" and layer 3 says "TDD is mandatory," **layer 1 wins**. The user is in control.

## Per-phase SP skill activation

For the canonical activation matrix per AI-DLC phase, read:

- [`common/superpowers-integration.md`](common/superpowers-integration.md) — full bindings
- [`common/sp-precedence.md`](common/sp-precedence.md) — precedence table

## Subagent-driven development with typed agents

When using `superpowers:subagent-driven-development`, replace the generic agent slots:

```
implementer    → <typed-implementer-per-routing>
spec reviewer  → general-purpose (unchanged; spec review is read-only)
code reviewer  → <typed-reviewer-per-routing>
```

The typed implementer auto-loads its required skills (declared in viv-agents frontmatter) so SP doesn't need to inject them.

## When generic SP triggers ARE active

Generic SP behavior is unmodified when:

1. The change is in **Class B** paths (per `routing-table.json` `enforced: false`)
2. AI-DLC is NOT active AND the path doesn't require typed-agent dispatch
3. The work is exploration/research outside any code change

In short: typed agents override generic SP **only when typed agents are dispatched**.
