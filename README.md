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
├── rules/
│   ├── dispatch-protocol.md              ← typed agent dispatch entry point
│   ├── post-implementation-chain.md      ← chain orchestration entry point
│   ├── ai-dlc-integration.md             ← thin pointer to viblocks/aidlc-orchestrator
│   ├── superpowers-integration.md        ← typed-agents-only SP overrides
│   ├── issue-driven-flow.md              ← autonomous change flow
│   └── common/                           ← 9 typed-agents-core rules
│       ├── iron-law.md
│       ├── typed-agent-mechanism.md
│       ├── subagent-dispatch-contract.md
│       ├── post-implementation-chain.md
│       ├── routing-table-population-protocol.md
│       ├── code-quality-rules.md
│       ├── debugging-gate.md
│       ├── enforcement-architecture.md
│       └── git-workflow.md
├── examples/
│   └── viblocks-style/
│       └── CLAUDE.example.md             ← concrete example mirroring viblocks-ai
├── architecture/
│   └── decisions/
│       ├── ADR-001-template-not-prescription.md
│       ├── ADR-002-external-deps-referenced.md
│       ├── ADR-003-iron-law-as-prose.md
│       └── ADR-004-extend-from-aidlc-orchestrator.md
└── migration/
    ├── from-viblocks.md
    └── preservation-audit.md
```

> **Note (post-split):** earlier versions of this repo also hosted AI-DLC orchestration content. Per [ADR-RD-012](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-012-separate-aidlc-orchestrator.md) (which supersedes ADR-RD-011), that content was split into a separate repo [`viblocks/aidlc-orchestrator`](https://github.com/viblocks/aidlc-orchestrator) which depends on this one (DIP). This repo is now strictly the typed-agents-core orchestration: ~14 rule files instead of ~69.

## Companion product

| Repo | Role | Dependency |
|---|---|---|
| [viblocks/viv-typed-agents](https://github.com/viblocks/viv-typed-agents) (this network) | Typed-agents dispatch + quality enforcement (Tier 1-5) | (root) |
| [viblocks/aidlc-orchestrator](https://github.com/viblocks/aidlc-orchestrator) | AI-DLC SDLC orchestrator (Inception → Construction → Verification → Deployment) | depends on viv-typed-agents (DIP) |

The typed-agents network is **independently usable**; AI-DLC requires it as backbone.

## Source attribution

The 9 `rules/common/` files plus the 5 entry points were SOLID-redesigned for this network. AI-DLC content originally extracted from [`fabianyvidal/aidlc-orchestrator`](https://github.com/fabianyvidal/aidlc-orchestrator) was split out per ADR-RD-012 to `viblocks/aidlc-orchestrator`. See migration history in commit log + ADR-RD-011 (superseded) for the prior full-merge state.

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
