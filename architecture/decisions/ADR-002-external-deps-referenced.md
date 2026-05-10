# ADR-002 — External dependencies (AI-DLC, Superpowers) referenced, not extracted

**Status:** Accepted
**Date:** 2026-05-08
**Category:** viv-orchestration-rules local

## Context

viblocks-ai's CLAUDE.md tightly references two external systems:
- **AI-DLC** rule details at `.aidlc-rule-details/`
- **Superpowers** skills invoked as `superpowers:<skill-name>`

A naive extraction could:
- vendor AI-DLC content into this strategy (out of scope; AI-DLC is its own framework)
- vendor SP skills (out of scope; SP is its own plugin)
- ignore them entirely (loses real integration that consumers need)

## Decision

The typed-agents strategy **references** AI-DLC and Superpowers as external systems but does **NOT bundle** them.

`viv-orchestration-rules` ships:
- `rules/ai-dlc-integration.md` — describes how typed agents bind to AI-DLC stages
- `rules/superpowers-integration.md` — describes how typed agents override SP skill bindings

These playbooks are **integration documentation**. They tell the consumer: "if you use AI-DLC, here's how it interacts with this strategy." They do NOT redistribute AI-DLC's rule details or SP's skills.

## Rationale

| Concern | How this satisfies |
|---|---|
| Scope clarity | This strategy is about typed agents + dispatch + workflows. AI-DLC and SP are orthogonal systems. |
| Versioning independence | AI-DLC and SP can update without forcing viv-orchestration-rules updates |
| Vendoring simplicity | Consumers vendor what they need; AI-DLC/SP follow their own distribution |
| Tier-friendliness | T3 consumers (no AI-DLC/SP) skip these playbooks entirely |

## Consequences

- The integration playbooks are **stable contracts** with AI-DLC and SP — they describe protocol, not content. If AI-DLC renames a stage, the playbook reference becomes stale (review-time concern).
- Consumers who don't use AI-DLC or SP simply don't load those playbooks. The CLAUDE.md template marks the relevant sections as optional.
- A consumer adopting both this strategy AND AI-DLC needs to maintain both vendor relationships independently.

## Alternatives considered

- **Bundle AI-DLC rule snippets relevant to typed agents:** rejected — AI-DLC's authority is upstream; bundling fragments creates drift risk
- **Define a parallel "typed-agents AI-DLC" alongside this strategy:** rejected — duplicates AI-DLC for marginal benefit
- **Avoid mentioning AI-DLC/SP entirely:** rejected — viblocks-ai's real integration would be lost; consumers reproducing this setup would have to rediscover the protocol

## Related

- README.md (declares external deps explicitly)
- CLAUDE.template.md (sections for AI-DLC and SP marked optional per tier)
