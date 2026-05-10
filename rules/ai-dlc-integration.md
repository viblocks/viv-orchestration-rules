# AI-DLC Integration — Index

How typed agents bind to AI-DLC stages. This page is the **entry point**; per-phase detail lives in `rules/ai-dlc/`.

> AI-DLC (Application Innovation & Development Lifecycle) is the workflow framework. Per [ADR-002](../architecture/decisions/ADR-002-external-deps-referenced.md) (revised) the **integration content** is now hosted here; AI-DLC the framework itself remains an external dependency.

## The four phases

```
INCEPTION → CONSTRUCTION → VERIFICATION → DEPLOYMENT → OPERATIONS
```

| Phase | Purpose | Per-stage rules |
|---|---|---|
| **Inception** | What to build and why | [9 stages](ai-dlc/inception/) |
| **Construction** | How to build it (designs + codegen) | [6 stages](ai-dlc/construction/) |
| **Verification** | Does it work? Is it ready? | [8 stages](ai-dlc/verification/) |
| **Deployment** | Ship it safely | [7 stages](ai-dlc/deployment/) |
| **Operations** | Run it (placeholder) | [1 file](ai-dlc/operations/) |

## Cross-cutting foundations (`common/`)

| File | Purpose |
|---|---|
| [`common/process-overview.md`](common/process-overview.md) | Full AI-DLC workflow narrative |
| [`common/iron-law.md`](common/iron-law.md) | IRON LAW formal statement |
| [`common/typed-agent-mechanism.md`](common/typed-agent-mechanism.md) | How typed agents work mechanically |
| [`common/post-implementation-chain.md`](common/post-implementation-chain.md) | Chain rule definition |
| [`common/superpowers-integration.md`](common/superpowers-integration.md) | Full SP skill bindings per stage |
| [`common/sp-precedence.md`](common/sp-precedence.md) | SP override hierarchy |
| [`common/core-change-flow-protocol.md`](common/core-change-flow-protocol.md) | Change flow protocol (DIRECT / CROSS-DOMAIN / DESIGN / REVERT) |
| [`common/adaptive-execution.md`](common/adaptive-execution.md) | Minimal/standard/comprehensive depth selection |
| [`common/depth-levels.md`](common/depth-levels.md) | What each depth level means per stage |
| [`common/workflow-changes.md`](common/workflow-changes.md) | Mid-workflow change protocol |
| [`common/stage-structural-patterns.md`](common/stage-structural-patterns.md) | Patterns common to every stage (questions, approval, audit) |
| [`common/audit-and-logging.md`](common/audit-and-logging.md) | `audit.md` + JSONL audit trails |
| [`common/welcome-message.md`](common/welcome-message.md) | Workflow activation banner |
| [`common/session-continuity.md`](common/session-continuity.md) | Resuming interrupted sessions |
| [`common/question-format-guide.md`](common/question-format-guide.md) | A/B/C/D/E format + ambiguity resolution |

## Critical override doctrine

When AI-DLC is active, generic Superpowers triggers are **OVERRIDDEN** by stage bindings in [`common/superpowers-integration.md`](common/superpowers-integration.md). Concrete:

| SP skill | Default trigger | AI-DLC override |
|---|---|---|
| `brainstorming` | "before any creative work" | Only at CONDITIONAL stages in the binding |
| `test-driven-development` | "implementing any feature" | Embedded in typed agent IRON LAW; never separate |
| `writing-plans` | "before any implementation" | Only at Code Generation Part 1 |
| `verification-before-completion` | "at every action" | At gates defined in the binding |
| `systematic-debugging` | universal | Universal (no override) |

See [`common/sp-precedence.md`](common/sp-precedence.md) for the full precedence table.

## Typed agent integration per phase (high-level)

| Phase | Where typed agents are dispatched |
|---|---|
| Inception | None (design only; no code) |
| Construction | **Code Generation** dispatches typed implementers per routing-table |
| Verification | **Test Implementation** dispatches typed implementers; **Test Environment Setup** + **CI Pipeline** dispatch the infra-devops domain implementer |
| Deployment | **CD Pipeline Implementation** dispatches the infra-devops domain implementer; chain post-impl after every infra change |

> Per [SPEC Appendix B invariant 7](https://github.com/viblocks/viv-typed-agents/blob/main/SPEC.md), agent names are not hardcoded in playbooks. Resolve via routing-table.

## Mid-workflow changes

When workflow changes occur (add/skip stages, restart, scope change), execute [`common/workflow-changes.md`](common/workflow-changes.md) first (meta-level), then the SP bindings adjust per [`common/superpowers-integration.md`](common/superpowers-integration.md) "Mid-Workflow Changes".

## Skills, agents, hooks references

| Concern | Where it lives |
|---|---|
| Knowledge content | [viv-skills](https://github.com/viblocks/viv-skills) |
| Agent identities | [viv-agents](https://github.com/viblocks/viv-agents) |
| Routing | [viv-routing](https://github.com/viblocks/viv-routing) |
| Workflow rules (data) | [viv-workflows](https://github.com/viblocks/viv-workflows) |
| Structural enforcement | [viv-hooks](https://github.com/viblocks/viv-hooks) |

## Source attribution

Most `common/` and `ai-dlc/` content was extracted and sanitized from [`fabianyvidal/aidlc-orchestrator`](https://github.com/fabianyvidal/aidlc-orchestrator) per [ADR-RD-011](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-011-extend-from-aidlc-orchestrator.md). Stack-prefix agent names and routing-table path adapted to our SOLID architecture.
