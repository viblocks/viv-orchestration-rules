# Preservation audit

What was extracted into viv-orchestration-rules vs. what stayed behind in viblocks-ai.

## Extracted (lives in viv-orchestration-rules)

| Content | Source | Where it lives now |
|---|---|---|
| IRON LAW dispatch protocol | viblocks `CLAUDE.md` "IRON LAW: Typed Agent Dispatch" | `CLAUDE.template.md` + `rules/dispatch-protocol.md` |
| Routing dispatch rules (1-6) | viblocks `CLAUDE.md` "Reglas de dispatch" | `rules/dispatch-protocol.md` step-by-step |
| Unknown service rule | viblocks `CLAUDE.md` "Unknown service rule" | `rules/dispatch-protocol.md` step 3 |
| Post-Implementation Chain stages | viblocks `CLAUDE.md` "Post-Implementation Chain (MANDATORY)" | `rules/post-implementation-chain.md` |
| Post-Chain Output template | viblocks `CLAUDE.md` "Post-Chain Output (MANDATORY)" | `rules/post-implementation-chain.md` + `CLAUDE.template.md` |
| AI-DLC stage bindings | viblocks `.aidlc-rule-details/common/` content (referenced) | `rules/ai-dlc-integration.md` |
| SP skill invocation override | viblocks `CLAUDE.md` "SP Skill Invocation Override (CRITICAL)" | `rules/superpowers-integration.md` |
| Subagent-driven dev with typed agents | viblocks `CLAUDE.md` "Integración con subagent-driven-development" | `rules/superpowers-integration.md` |
| Issue-Driven flow triage gate (Q1-Q4) | viblocks `CLAUDE.md` + issue-driven-change-flow-design.md | `rules/issue-driven-flow.md` |
| Four flow paths (DIRECT/CROSS-DOMAIN/DESIGN/REVERT) | viblocks `core-change-flow-protocol.md` | `rules/issue-driven-flow.md` |
| Commit policy + push manual | viblocks `CLAUDE.md` "Commit Policy" | `CLAUDE.template.md` "Commit Policy" |
| PR Overlap Check | viblocks `CLAUDE.md` "PR Overlap Check (MANDATORY...)" | `CLAUDE.template.md` "PR Overlap Check" |
| Behavioral hierarchy | viblocks `CLAUDE.md` (implicit + SP override section) | `rules/superpowers-integration.md` table |

## Stayed behind (viblocks-ai-specific)

| Content | Why it stays |
|---|---|
| Worktree Bootstrap rule | Project-specific operational discipline (`make worktree NAME=...`) |
| Worktree Session Hygiene | Project-specific (marker registry, AIDLC_ENFORCEMENT_MODE env var) |
| Bajo Acoplamiento / Alta Cohesión section | SOLID guidance — already covered by ADRs in viv-typed-agents and per-agent system prompts |
| Desacoplamiento Despliegue ↔ Código rule | Project-specific deployment policy |
| Linear-specific commands (`linear issue view`, `linear issue create`) | Linear is consumer choice; consumer maps to their tracker |
| Blacklist-domain detector keywords | Project-specific (blockchain blacklist monitoring product) |
| `aidlc-docs/`, `.aidlc-rule-details/` content | AI-DLC framework files; not part of typed-agents strategy |
| MANDATORY: Plan-Level Checkbox Enforcement | AI-DLC content, not strategy content |
| MANDATORY: Custom Welcome Message | AI-DLC content |
| Adaptive Workflow Principle | AI-DLC content |
| `Co-Authored-By: Claude Sonnet 4.6` specific attribution | Consumer policy |
| "Carpeta docs/ Migrada a Notion" rule | Project-specific |
| V3 Post-Deployment Checklist | Project-specific deployment artifact |
| Project Context (`.promptops/project-context.json`) | Project-specific tooling integration |

## Knowledge loss check

- [x] IRON LAW preserved verbatim (template + playbook)
- [x] All 6 dispatch rules preserved (step-by-step in playbook)
- [x] Unknown-service rule preserved (3 escalation options A/B/C)
- [x] Post-Impl Chain stage order preserved (verify → review → security → commit)
- [x] Post-Chain Output 5 fields preserved (Verification, Code review, Security review, Files, Commit)
- [x] AI-DLC integration tier mapping preserved (4 phases × typed agent integration)
- [x] SP override hierarchy preserved (4 layers; user > AI-DLC > IRON LAW > generic SP)
- [x] Issue-driven flow Q1-Q4 triage preserved
- [x] All 4 flow paths preserved (DIRECT/CROSS-DOMAIN/DESIGN/REVERT)
- [x] Autonomous-vs-supervised distinction preserved (DIRECT/REVERT autonomous; CROSS-DOMAIN/DESIGN supervised)
- [x] Fix-intent gate behavior preserved (root cause required; references viv-workflows)
- [x] Commit auto + push manual policy preserved
- [x] PR Overlap Check sequential-only scope preserved

## Verification

Reproduce viblocks-ai's behavior using viv-orchestration-rules:

| viblocks-ai behavior | viv-orchestration-rules equivalent |
|---|---|
| User says "fix VI-42" → triage Q1-Q4 → DIRECT path | `rules/issue-driven-flow.md` triage + DIRECT |
| User says "implement feature X" in services/core → backend-crypto-implementer | `rules/dispatch-protocol.md` steps 1-7 |
| Implementer completes → verification → reviewer → security (conditional) → commit | `rules/post-implementation-chain.md` stage execution |
| AI-DLC active + Code Generation stage → typed implementer (not general-purpose) | `rules/ai-dlc-integration.md` Construction section |
| SP `subagent-driven-development` invoked → typed agents replace generic agents | `rules/superpowers-integration.md` "Subagent-driven development with typed agents" |
| User adds scope mid-flow → escalate, re-enter triage | `rules/issue-driven-flow.md` "Failure modes" |

All viblocks orchestration behaviors representable. No behavioral coverage loss.

## Identified during post-extraction review

- **MIN-1 (resolved):** the viblocks-style example was named `CLAUDE.md`, which collided with hook protections on root-level `CLAUDE.md` filenames in self-hosting consumers. Renamed to `CLAUDE.example.md`. Consumers rename when vendoring.

## Post-publication SPEC alignment review (2026-05-08)

A second review aligned this repo against the SPEC (Goals §1.1, ADR Index §5, Inter-Component Contracts §6, Composition Tiers §7, External Dependencies §8, Apéndice B invariants). Findings and resolutions:

- **MED-1 (resolved):** README "Five playbooks" tier table conflated two semantics. Per `composition/tiers.md`, this repo IS the T5 component. The table now reads "Minimum dependencies" (other components the playbook assumes), not "Required tier". Clarification added that without T3 dependencies, the playbooks are not meaningfully executable.
- **LOW-1 (resolved):** `rules/ai-dlc-integration.md` hardcoded `infra-devops-implementer` agent name in three places, violating SPEC Apéndice B invariant 7. Replaced with "the typed implementer for the `infra-devops` domain (per `routing-table.json`)" abstraction.
- **LOW-2 (resolved):** `rules/superpowers-integration.md` skill bindings table omitted `receiving-code-review` (listed in SPEC §8.2). Added with the "active during implementer re-dispatch loop" semantic. `brainstorming` retained but explicitly marked as not part of SPEC §8.2 contract.
- **LOW-3 (resolved):** `CLAUDE.template.md` "Enforcement is structural" section did not introduce the four hook types from ADR-RD-006. Added a brief enumeration table (deny / advisory / refinement / lifecycle) so consumers reading the template understand what the hooks layer provides.
