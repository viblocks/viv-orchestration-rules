# ADR-004 — Extend orchestration content from aidlc-orchestrator

**Status:** Accepted
**Date:** 2026-05-09
**Category:** viv-orchestration-rules local

## Context

This repo's initial extraction (commit `45a87a1`) shipped 5 playbooks (~25KB) — a thin behavioral surface plus an example.

In parallel, [`fabianyvidal/aidlc-orchestrator`](https://github.com/fabianyvidal/aidlc-orchestrator) extracted the same content from viblocks-ai with a different model: a single-repo Claude Code plugin with deep per-stage rules (~250KB across 60+ files), explicit issue-analysis discipline, overconfidence prevention, friction reporting, and other operational disciplines that viblocks-ai's monolithic CLAUDE.md did NOT have.

The user requested 100% extraction of those rules into our network, sanitized to align with the SOLID architecture (per ADR-RD-009).

## Decision

Extract the full `rules/` tree from aidlc-orchestrator into `viv-orchestration-rules/playbooks/`, sanitized as follows:

### Structural mapping

| aidlc-orchestrator path | viv-orchestration-rules path |
|---|---|
| `rules/common/` | `playbooks/_common/` |
| `rules/inception/` | `playbooks/ai-dlc/inception/` |
| `rules/construction/` | `playbooks/ai-dlc/construction/` |
| `rules/verification/` | `playbooks/ai-dlc/verification/` |
| `rules/deployment/` | `playbooks/ai-dlc/deployment/` |
| `rules/operations/` | `playbooks/ai-dlc/operations/` |
| `rules/extensions/` | `playbooks/extensions/` |

### Sanitizations applied (mechanical, repo-wide)

1. **Agent name renames** (legacy framework-prefix → stack-prefix per viv-routing ADR-003):
   - `nestjs-*` → `backend-*` (with crypto/waas tier preserved)
   - `reactjs-*` → `frontend-*` (with crypto/waas tier preserved)
2. **Routing-table path**: `.claude/context/routing-table.json` → `.claude/routing/routing-table.json`
3. **Markdown link path renames**: `common/X.md`, `inception/X.md`, etc. → corresponding `_common/X.md`, `ai-dlc/inception/X.md`
4. **Repo identity**: GitHub URLs pointing to `fabianyvidal/aidlc-orchestrator` updated to our repos where ownership applies
5. **Architecture note auto-insertion**: any file mentioning `.claude/context/artifact-classifier.json` (eliminated per ADR-RD-004) gains a top-of-file note clarifying that Class A scope is now derived from `routing-table.json`

### Existing playbooks preserved as entry points

The 5 original playbooks (`dispatch-protocol.md`, `post-implementation-chain.md`, `ai-dlc-integration.md`, `superpowers-integration.md`, `issue-driven-flow.md`) remain as **entry-point summaries** linking to the comprehensive `_common/` and `ai-dlc/` content. Specifically:

- `ai-dlc-integration.md` → reframed as **index** for the per-phase tree
- `superpowers-integration.md` → thin pointer to `_common/superpowers-integration.md` (the 25KB canonical version)
- `dispatch-protocol.md`, `post-implementation-chain.md`, `issue-driven-flow.md` → kept as SOLID-redesigned entry points; cross-link to the comprehensive `_common/` files

This avoids replacing our SOLID-redesigned surface while landing the full content depth.

### Boundary update — ADR-002 revised

ADR-002 ("external deps referenced not extracted") had stated AI-DLC and Superpowers content stays external. This ADR revises that boundary: the **integration content** (how typed agents bind to AI-DLC stages, how SP skills override) is now hosted here. AI-DLC and Superpowers as **systems** remain external — we don't ship the AI-DLC framework runtime nor the SP skill packages.

## Rationale

| Concern | How this satisfies |
|---|---|
| Content depth | Operational disciplines (issue-analysis, overconfidence, friction, etc.) viblocks-ai never had are now part of the strategy |
| SRP at file level | 60+ files each with one reason to change; same SOLID hygiene as the rest of the network |
| Drift prevention | Sanitization is mechanical — re-running the script reproduces the deployment |
| Attribution preserved | Source attribution to `fabianyvidal/aidlc-orchestrator` retained in README and at content boundaries |
| Stack-prefix consistency | Agent name renames align with viv-routing ADR-003 (no `nestjs-*`/`reactjs-*` survives) |
| Routing path consistency | All references use `.claude/routing/routing-table.json` per viv-routing convention |

## Consequences

### What changes

- `playbooks/` grows from 5 files to 69 files (~25KB → ~580KB)
- Each AI-DLC stage now has its own playbook in `ai-dlc/<phase>/` with depth selection, question generation, approval gate format
- `_common/` provides 29 cross-cutting playbooks including operational disciplines (issue-analysis, overconfidence, friction, audit-and-logging) absent from viblocks-ai
- `extensions/` provides opt-in extensions (security baseline, property-based testing) with `*.opt-in.md` opt-in prompts

### What does NOT change

- Our 5 original playbooks remain as entry points (SOLID-redesigned)
- The 6+1 component decomposition stays intact (ADR-RD-009)
- Pure descriptors pattern preserved (ADR-RD-008) — no executable code added
- Cross-component contracts (routing, workflows, agents) unchanged
- ADR-RD-010 product composition (typed-agents IS the installable) unchanged

### Cross-component effects

- `viv-typed-agents` adds [ADR-RD-011](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-011-extend-from-aidlc-orchestrator.md) documenting the extension at the system-wide layer
- `viv-typed-agents/MANIFEST.yaml` bumps `viv-orchestration-rules` SHA to the post-extension commit so the installer deploys the extended content

## Alternatives considered

- **Reference aidlc-orchestrator as external dependency** (no extraction): rejected — user explicitly requested 100% extraction so consumers of viv-typed-agents don't depend on a parallel plugin
- **Replace the 5 SOLID-redesigned playbooks with the aidlc-orch versions wholesale**: rejected — loses SOLID-redesign work; entry-point + canonical-detail layering is cleaner
- **Bring only the per-stage AI-DLC rules, skip operational disciplines (`overconfidence-prevention`, `friction-reporting`, etc.)**: rejected — the operational disciplines ARE the differentiator vs viblocks-ai's original; bringing them is the whole point of the extension

## Related

- ADR-RD-009 (preserve viblocks objectives, redesign per SOLID)
- ADR-RD-008 (pure descriptors — preserved by extension)
- ADR-RD-010 (product composition — installer deploys extended content)
- ADR-002 local (external deps — revised by this ADR)
- viv-routing ADR-003 (stack-prefix naming — applied during sanitization)
