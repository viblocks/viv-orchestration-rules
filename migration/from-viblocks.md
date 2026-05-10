# Migration from viblocks-ai

How viv-orchestration-rules was extracted from viblocks-ai's CLAUDE.md and AI-DLC integration files.

## Source artifacts (viblocks-ai)

1. `<project-root>/CLAUDE.md` (viblocks-ai's project CLAUDE.md, ~600 lines mixed Spanish/English)
2. `.aidlc-rule-details/common/core-change-flow-protocol.md`
3. `.aidlc-rule-details/common/superpowers-integration.md`
4. `docs/superpowers/specs/2026-04-14-issue-driven-change-flow-design.md`

## Transformations applied

### 1. Decomposed the monolithic CLAUDE.md

viblocks' CLAUDE.md mixes:
- Worktree hygiene rules (project-specific operational discipline)
- IRON LAW dispatch protocol (typed-agents strategy)
- Post-Implementation Chain (workflows)
- AI-DLC integration (external system binding)
- Superpowers binding (external system binding)
- Issue-driven flow (autonomous change flow)
- Project-specific extensions (Linear, blacklist domain, runner migration, paths-filter)

Decomposition:
- IRON LAW + Routing + Post-Impl Chain → `CLAUDE.template.md` (core)
- Dispatch protocol detail → `rules/dispatch-protocol.md`
- Post-Impl Chain orchestration → `rules/post-implementation-chain.md`
- AI-DLC integration → `rules/ai-dlc-integration.md`
- SP integration → `rules/superpowers-integration.md`
- Issue-driven flow → `rules/issue-driven-flow.md`
- Worktree hygiene + project extensions → STAY in viblocks-ai

### 2. Sanitized project-specific tokens

| viblocks-ai token | Template placeholder |
|---|---|
| `viblocks-ai` | `<PROJECT_NAME>` |
| `services/core`, `services/bot`, `services/ui`, `packages/shared` | `<INFRA_PATHS>` (refer to routing-table) |
| `VI-` (Linear prefix) | `<ISSUE_PREFIX>` |
| `linear issue ...` (Linear CLI) | `<ISSUE_TRACKER>` |
| `Skill(blacklist-monitoring)` | `<PROJECT_DOMAIN_SKILL_TRIGGERS>` |
| `.aidlc-rule-details/common/...` | `<AIDLC_RULE_DETAILS_PATH>` (referenced in playbook) |
| Spanish + English mixed | English in template; Spanish preserved in `examples/viblocks-style/CLAUDE.md` |

### 3. Promoted "Capa 1-8" to playbook prose

viblocks' CLAUDE.md describes 8 enforcement layers (`Capa 1-8`) in a table. The redesigned strategy uses 5 tiers (per `viv-typed-agents/composition/tiers.md`). The viblocks layer detail (specific hooks, marker registry) belongs to `viv-hooks`, not to orchestration rules.

The orchestration playbooks reference the **structural enforcement** as a guarantee (Tier 4+) but don't enumerate individual hooks.

### 4. Promoted "SP Skill Invocation Override" to a dedicated playbook

viblocks' CLAUDE.md has a critical section explaining SP skill overrides during AI-DLC. Promoted to `rules/superpowers-integration.md` with structured authority hierarchy table.

### 5. Issue-driven flow extracted from Linear coupling

viblocks' issue-driven flow uses Linear (`linear issue view VI-XXX`). Generalized to `<ISSUE_TRACKER>` placeholder; `<ISSUE_ID>` format consumer-defined. Linear-specific commands stay in viblocks-ai.

### 6. Project-specific advisors NOT extracted

These stay in viblocks-ai because they're project-specific:
- Worktree hygiene rules (Regla: Worktree Bootstrap, Regla: Worktree Session Hygiene)
- Blacklist domain detector
- AI-DLC enforcement mode env var (`AIDLC_ENFORCEMENT_MODE`)
- "Carpeta docs/ Migrada a Notion" rule
- PR Overlap Check (kept as a generic gate in CLAUDE.template; viblocks-specific scope notes stay)
- "Adaptive Workflow Principle", "MANDATORY: Rule Details Loading", "MANDATORY: Extensions Loading" — these are AI-DLC content, not strategy content

## Sanitization checklist

- [x] No `viblocks-ai` references in template (placeholder used)
- [x] No `VI-` prefix in template (placeholder used)
- [x] No Spanish-only sections in template (English; viblocks-style preserves Spanish)
- [x] No `services/core` / `services/bot` paths in template
- [x] No Linear CLI commands in template
- [x] No blacklist-domain content in template
- [x] All AI-DLC references go through `<AIDLC_RULE_DETAILS_PATH>` placeholder or playbook abstraction
- [x] All SP references go through `superpowers:<skill>` convention; no SP content bundled

## Re-vendor plan (future)

When viv-orchestration-rules is vendored back into viblocks-ai:

1. Replace `viblocks-ai/CLAUDE.md` with the template, fill placeholders with viblocks values
2. Vendor playbooks to `viblocks-ai/.claude/orchestration/playbooks/`
3. Re-add viblocks-specific extensions (worktree hygiene, blacklist detector) in the dedicated extension block
4. Update references from `nestjs-*`/`reactjs-*` to `backend-*`/`frontend-*` (per viv-routing NAMING.md)
5. Verify dispatch behavior unchanged (run a sample DIRECT path on a small fix)

This re-vendor is gated on viv-hooks extraction.
