# viv-orchestration-rules

> ⚠️ **Internal component of [viv-typed-agents](https://github.com/viblocks/viv-typed-agents).**
>
> The recommended install path is the typed-agents product, not this repo standalone:
> ```bash
> git clone https://github.com/viblocks/viv-typed-agents
> ./viv-typed-agents/scripts/install.sh /path/to/your-project --tier 5
> ```
>
> This repo is public for transparency and as a surgical-use escape hatch (`cp -r` a single playbook). See [ADR-RD-010](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-010-product-composition.md) for product composition rationale.

Behavioral orchestration rules for the typed-agents strategy. This is the **Tier 5** component (full system) that ties the other five together.

Per [ADR-RD-008](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-008-pure-descriptors.md), this repo ships only `.md`. No executable code, no JSON schemas — orchestration rules are **prose for the LLM**, not data for tooling.

## Contents

```
viv-orchestration-rules/
├── README.md
├── CLAUDE.template.md                    ← project-root CLAUDE.md template
├── playbooks/
│   ├── dispatch-protocol.md              ← typed agent dispatch entry point
│   ├── post-implementation-chain.md      ← chain orchestration entry point
│   ├── ai-dlc-integration.md             ← AI-DLC integration index
│   ├── superpowers-integration.md        ← SP integration index
│   ├── issue-driven-flow.md              ← autonomous change flow
│   ├── _common/                          ← 29 cross-cutting playbooks
│   │   ├── iron-law.md, typed-agent-mechanism.md, subagent-dispatch-contract.md
│   │   ├── post-implementation-chain.md, core-change-flow-protocol.md
│   │   ├── superpowers-integration.md (full bindings ~25KB), sp-precedence.md
│   │   ├── debugging-gate.md, overconfidence-prevention.md
│   │   ├── routing-table-population-protocol.md, code-quality-rules.md
│   │   ├── audit-and-logging.md, friction-reporting.md, session-continuity.md
│   │   ├── adaptive-execution.md, depth-levels.md, workflow-changes.md
│   │   ├── stage-structural-patterns.md, frontend-change-discipline.md
│   │   ├── enforcement-architecture.md, error-handling.md
│   │   ├── welcome-message.md, terminology.md, question-format-guide.md
│   │   ├── git-workflow.md, content-validation.md, ascii-diagram-standards.md
│   │   ├── process-overview.md, aidlc-docs-structure.md
│   ├── ai-dlc/                           ← per-stage AI-DLC rules (31 files)
│   │   ├── inception/    (9 stages)
│   │   ├── construction/ (6 stages)
│   │   ├── verification/ (8 stages)
│   │   ├── deployment/   (7 stages)
│   │   └── operations/   (1 placeholder)
│   └── extensions/                       ← opt-in extensions
│       ├── security/baseline/            (security-baseline.md + opt-in.md)
│       └── testing/property-based/       (property-based-testing.md + opt-in.md)
├── examples/
│   └── viblocks-style/
│       └── CLAUDE.example.md             ← concrete example mirroring viblocks-ai (rename to CLAUDE.md when vendoring; .example. suffix avoids hook-protected filename traps)
├── architecture/
│   └── decisions/
│       ├── ADR-001-template-not-prescription.md
│       ├── ADR-002-external-deps-referenced.md
│       └── ADR-003-iron-law-as-prose.md
└── migration/
    ├── from-viblocks.md
    └── preservation-audit.md
```

## Source attribution

The `_common/` and `ai-dlc/` content was extracted and sanitized from [`fabianyvidal/aidlc-orchestrator`](https://github.com/fabianyvidal/aidlc-orchestrator) per [ADR-RD-011](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-011-extend-from-aidlc-orchestrator.md) and local [ADR-004](architecture/decisions/ADR-004-extend-from-aidlc-orchestrator.md). Sanitization was mechanical and repo-wide:

- Stack-prefix agent renames (`nestjs-*`/`reactjs-*` → `backend-*`/`frontend-*` per viv-routing ADR-003)
- Routing path standardization (`.claude/context/routing-table.json` → `.claude/routing/routing-table.json`)
- Link path normalization to the new structure
- Architecture notes inserted where the eliminated `artifact-classifier.json` is referenced

## Five entry-point playbooks

This repo IS the Tier 5 component (per [composition/tiers.md](https://github.com/viblocks/viv-typed-agents/blob/main/composition/tiers.md)). Adopting it means you're at T5 by definition. The "Minimum dependencies" column lists the other components each playbook **assumes** the consumer has already vendored.

| Playbook | Purpose | Minimum dependencies |
|---|---|---|
| `dispatch-protocol.md` | The IRON LAW: typed agent dispatch by path | T3 (routing-table required) |
| `post-implementation-chain.md` | What runs after a typed implementer completes | T3 (workflow rules required) |
| `ai-dlc-integration.md` | How typed agents bind to AI-DLC stages | T5 + AI-DLC external |
| `superpowers-integration.md` | How typed agents specialize Superpowers skills | T5 + Superpowers external |
| `issue-driven-flow.md` | Autonomous change flow from issue tracker | T5 + issue tracker external |

A consumer who vendors this repo without `viv-routing` and `viv-workflows` cannot meaningfully execute `dispatch-protocol` or `post-implementation-chain` — the playbooks reference contracts published by those components. Without `viv-hooks` (T4), the playbooks describe behavior that is **advisory only**; structural enforcement requires the hooks layer.

## Quick start (consumer)

1. **Vendor**: `cp -r viv-orchestration-rules/ my-project/.claude/orchestration/` (and copy `CLAUDE.template.md` to `my-project/CLAUDE.md`)
2. **Customize** the template: replace placeholders with project name, paths, conventions
3. **Pick playbooks**: keep only those for your tier (T3 → drop AI-DLC and SP playbooks)
4. **Reference companion repos**: the template references `viv-routing`, `viv-workflows`, `viv-agents`, `viv-skills`, `viv-hooks`

## External dependencies (per ADR-002)

These are referenced but **NOT extracted** by this strategy:

- **AI-DLC** (Application Innovation & Development Lifecycle) — separate framework consumed via `.aidlc-rule-details/`
- **Superpowers** — separate skill system invoked via `superpowers:<skill-name>`
- **Issue tracker** — Linear, GitHub Issues, Jira — consumer-defined

The playbooks for these dependencies describe *integration*, not the dependencies themselves.

## Status

Initial extraction (2026-05-08). Not yet vendored back to viblocks-ai. Pending: `viv-hooks` extraction (last component).
