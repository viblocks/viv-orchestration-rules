# CLAUDE.md — viblocks-ai (post-redesign example)

> Filled-in example showing what `CLAUDE.template.md` looks like for a project mirroring viblocks-ai's stack and conventions, AFTER applying the redesigned naming and component model. Filename uses `.example.md` suffix because some hook setups protect literal `CLAUDE.md` files; rename when vendoring.

## IRON LAW: Typed Agent Dispatch

**All creation, edit, fix, or improvement of application code executes EXCLUSIVELY through the correct typed agent. No exceptions.**

### Routing Table

**Single source of truth**: `.claude/routing/routing-table.json`. The `enforced` field determines Class A vs Class B.

### Dispatch rules

1. Detect domain by path. Unknown service → STOP and apply unknown-service rule.
2. Use the typed implementer for the domain — never `general-purpose` for Class A.
3. Use the typed reviewer for the domain.
4. TDD is embedded in the implementer's IRON LAW.
5. Cross-domain work splits per domain; `packages/shared` is Task 1.
6. Before any recommendation crossing service boundaries in the blacklist pipeline (pollers → enrichment → guardian → consumers), invoke `Skill(blacklist-monitoring)`.

## Post-Implementation Chain (MANDATORY)

After any typed implementer:

1. Verification — `superpowers:verification-before-completion`
2. Domain code review — paired per routing
3. Security review — conditional on path triggers (controllers, DTOs, guards, auth, frontend forms, config, package.json, chain/enrichment paths)
4. Auto-commit when gates pass; push manual

## Post-Chain Output (MANDATORY)

```
***** Post-Implementation Chain Complete *****
- Verification: [PASS/FAIL] — [N/N] tests, build [OK/FAIL]
- Code review: [agent name] — [PASS / N issues (severity)]
- Security review: [security-reviewer — PASS/FAIL/N/A] — [findings or "no security-sensitive paths"]
- Files changed: [list]
- Commit: [auto-committing / escalating to user — reason]
```

## PR Overlap Check (before `gh pr create`)

Before creating PRs, verify no file/commit overlap with open PRs (sequential workflows).

## Commit Policy

- Auto-commit when verification PASS + code review PASS.
- Escalate on verification fail, CRITICAL/HIGH review findings, or scope change.
- Push manual.

## Change Flow — Issue-Driven (Linear: `VI-XXX`)

When user references a Linear issue:

1. Read the issue: `linear issue view VI-XXX`
2. Triage Q1-Q4. First match wins.
3. Execute the path (DIRECT/CROSS-DOMAIN/DESIGN/REVERT).
4. Close with evidence per `evidence-schema.json`.

Autonomous: DIRECT, REVERT. Supervised: DESIGN, CROSS-DOMAIN.

## Full Workflow — AI-DLC + Superpowers integration

AI-DLC stages bind typed agents:
- **Code Generation** → `backend-crypto-implementer` / `frontend-crypto-implementer` per routing
- **Test Implementation** → typed implementer with TDD
- **Test Environment Setup** → `infra-devops-implementer`
- **CI Pipeline Implementation** → `infra-devops-implementer`
- **CD Pipeline Implementation** → `infra-devops-implementer`

SP override:
- `test-driven-development` embedded in implementer; never invoked separately
- `verification-before-completion` only at chain verification stages
- `brainstorming` only at AI-DLC CONDITIONAL stages
- `systematic-debugging` universal (no override)

## Project-specific extensions

### Worktree Bootstrap

Always create worktrees with `make worktree NAME=<branch>` — never with `git worktree add` directly.

### Worktree Session Hygiene

Start `claude` from the worktree where you'll work, not from main repo. Marker registry resolves role via `lib/role-detection.sh`.

### Blacklist Domain Skill Trigger

Before any recommendation, fix, or issue that crosses service boundaries in the blacklist pipeline, invoke `Skill(blacklist-monitoring)`. Applies to: code analysis, code review, improvement proposals, issue creation.

### docs/ folder migrated to Notion

Do NOT read or reference `docs/` files (except `docs/superpowers/` which is local).

### Linear is the source of truth for issues

Use `linear issue ...` commands. Never `gh issue` for the change flow.

### Naming migration completed

Pre-redesign: `nestjs-crypto-implementer`, `reactjs-crypto-implementer`. Post-redesign: `backend-crypto-implementer`, `frontend-crypto-implementer`. References updated in routing-table and CLAUDE.md.

---

This CLAUDE.md follows the `viv-orchestration-rules` template structure with viblocks-specific extensions in the dedicated section.
