# viv-orchestration-rules

Behavioral orchestration rules for the typed-agents strategy. This is the **Tier 5** component (full system) that ties the other five together.

Per [ADR-RD-008](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-008-pure-descriptors.md), this repo ships only `.md`. No executable code, no JSON schemas — orchestration rules are **prose for the LLM**, not data for tooling.

## Contents

```
viv-orchestration-rules/
├── README.md
├── CLAUDE.template.md                    ← project-root CLAUDE.md template
├── playbooks/
│   ├── dispatch-protocol.md              ← typed agent dispatch (the IRON LAW)
│   ├── post-implementation-chain.md      ← orchestration of chain stages
│   ├── ai-dlc-integration.md             ← typed agents in AI-DLC stages
│   ├── superpowers-integration.md        ← typed agents in SP skill bindings
│   └── issue-driven-flow.md              ← autonomous change flow
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

## Five playbooks

| Playbook | Purpose | Required tier |
|---|---|---|
| `dispatch-protocol.md` | The IRON LAW: typed agent dispatch by path | T2+ |
| `post-implementation-chain.md` | What runs after a typed implementer completes | T3+ (uses workflows) |
| `ai-dlc-integration.md` | How typed agents bind to AI-DLC stages | T5 (with AI-DLC consumer) |
| `superpowers-integration.md` | How typed agents specialize Superpowers skills | T5 (with SP consumer) |
| `issue-driven-flow.md` | Autonomous change flow from issue tracker | T5 (with issue tracker) |

A consumer at Tier 3 needs only the first two. Tier 5 consumers (AI-DLC + Superpowers) consume all five.

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
