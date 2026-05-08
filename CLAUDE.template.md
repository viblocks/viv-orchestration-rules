# CLAUDE.md — `<PROJECT_NAME>`

> Template from `viv-orchestration-rules`. Replace `<PLACEHOLDERS>` with project-specific values. Drop sections for tiers you don't adopt.

## IRON LAW: Typed Agent Dispatch

**All creation, edit, fix, or improvement of application code executes EXCLUSIVELY through the correct typed agent. No exceptions.**

The full strategy is in `<TYPED_AGENT_STRATEGY_DOC>`.

### Routing Table

**Single source of truth**: `.claude/routing/routing-table.json` — see [viv-routing](https://github.com/viblocks/viv-routing).

The `enforced` field on each route determines Class A (typed-agent dispatch enforced) vs Class B (free edit). There is no separate classifier — see [ADR-RD-004](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-004-classifier-folded.md).

### Dispatch rules

1. **Detect domain by path**, not by problem keywords. If the path has no explicit route, **STOP** and apply the unknown-service rule (see `playbooks/dispatch-protocol.md`).
2. **Use the typed implementer** — never `general-purpose` for Class A code.
3. **Use the typed reviewer** — never a generic code-reviewer for Class A code.
4. **TDD is embedded** — don't invoke a separate TDD skill; the typed agent's IRON LAW includes it.
5. **Cross-domain work** is split per domain in the plan; shared packages are Task 1.
6. **Project-specific domain skills** — invoke before recommendations or fixes that cross service boundaries in domain-critical pipelines (see `<PROJECT_DOMAIN_SKILL_TRIGGERS>`).

See `playbooks/dispatch-protocol.md` for the full protocol.

## Post-Implementation Chain (MANDATORY)

After any typed implementer completes, the chain runs per `.claude/workflows/post-implementation-chain.json` (see [viv-workflows](https://github.com/viblocks/viv-workflows)):

1. **Verification** — tests + build must pass
2. **Domain code review** — typed reviewer paired by routing
3. **Security review** — conditional on path triggers
4. **Commit** — automatic when all preceding gates pass; push is never automatic

See `playbooks/post-implementation-chain.md` for stage-by-stage orchestration.

## Post-Chain Output (MANDATORY)

After the chain completes, present this structured output before committing:

```
***** Post-Implementation Chain Complete *****
- Verification: [PASS/FAIL] — [N/N] tests, build [OK/FAIL]
- Code review: [agent name] — [PASS / N issues (severity)]
- Security review: [security-reviewer — PASS/FAIL/N/A] — [findings or "no security-sensitive paths"]
- Files changed: [list]
- Commit: [auto-committing / escalating to user — reason]
```

All 5 fields are required. Do NOT summarize as "code review passed" — name the reviewer agent explicitly.

## PR Overlap Check (before `gh pr create`)

Before creating a PR, verify the local diff and commits aren't already in flight in another open PR.

- Files overlap → STOP. Present conflicting PRs to the user.
- Commit-SHA duplicates → STOP. Reference existing PR.
- No overlap → proceed.

Sequential workflows only. Concurrent multi-agent coordination is out of scope.

## Commit Policy

- **Auto-commit** when verification PASS + code review PASS (both automatic gates).
- **Escalate** when verification fails, review reports CRITICAL/HIGH, or scope changed mid-task.
- **Push is manual.** Never push without explicit user request.

## Change Flow — Issue-Driven (post-production autonomous)

When the user references an issue (`<ISSUE_PREFIX>-XXX`), execute the issue-driven flow per `playbooks/issue-driven-flow.md`:

1. **Read the issue** from `<ISSUE_TRACKER>`
2. **Triage** (4 questions: new abstraction? cross-domain? revert? scoped fix?). First match wins.
3. **Execute the path** — DIRECT, CROSS-DOMAIN, DESIGN, or REVERT
4. **Close with evidence** per `evidence-schema.json`

Autonomous: DIRECT, REVERT. Supervised: DESIGN, CROSS-DOMAIN.

## Full Workflow — AI-DLC + Superpowers integration

When executing the full AI-DLC workflow (Inception → Construction → Verification → Deployment), Superpowers skills bind to specific stages per `playbooks/superpowers-integration.md` and `playbooks/ai-dlc-integration.md`.

**Critical override:** during AI-DLC execution, the operative bindings are the **sole authority** for SP skill invocation. Generic SP triggers ("use brainstorming before any creative work") are OVERRIDDEN.

| Mode | Authority for SP invocation |
|---|---|
| AI-DLC full workflow | `playbooks/superpowers-integration.md` |
| AI-DLC change flow | `playbooks/issue-driven-flow.md` |
| Issue-Driven autonomous | `playbooks/issue-driven-flow.md` |
| Outside AI-DLC + Class B | Default SP behavior |
| Outside AI-DLC + Class A | `playbooks/dispatch-protocol.md` |

## Enforcement is structural, not behavioral

The IRON LAW is enforced by `viv-hooks` (when adopted at Tier 4+). Behavioral compliance is the first line; structural hooks are the safety net.

```
Behavioral: this CLAUDE.md → LLM dispatches typed agents
Structural: viv-hooks → wrong dispatch is blocked at Edit/Write time
```

`viv-hooks` ships four explicit hook types (per [ADR-RD-006](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-006-three-hook-types.md)):

| Type | Role |
|---|---|
| **deny** | Hard block on contract violation (routing, secrets, self-mod, isolation, commit trailer) |
| **advisory** | Warn-and-allow with `additionalContext` injection (post-impl chain, evidence, fix-intent, skill-installed) |
| **refinement** | Positive override that allows what `deny` would block (fast-lane for Class A non-app edits) |
| **lifecycle** | Marker register/cleanup; no allow/deny decision |

If your project does not adopt `viv-hooks`, the IRON LAW is **advisory** — the LLM follows it because it's documented, not because it's enforced. Quality drops accordingly.

## Project-specific extensions

`<INSERT_PROJECT_EXTENSIONS_HERE>` — Project-specific rules that don't generalize (domain advisors, deploy gates, etc.) live below. Keep generic content above; project drift below.

---

**This template is part of the typed-agents strategy.** See `viv-orchestration-rules` for the canonical playbooks.
