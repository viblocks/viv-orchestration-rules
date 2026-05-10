# Post-Implementation Chain Playbook

Orchestration of the chain stages after a typed implementer completes.

The **rule** lives in `.claude/workflows/post-implementation-chain.json` (see [viv-workflows](https://github.com/viblocks/viv-workflows)). This playbook describes how to **execute** that rule.

## Stage execution

For each stage in the chain JSON, in order:

### `kind: verification`

Run project-defined verification. Common implementations:
- Invoke `superpowers:verification-before-completion` skill
- Run `make verify`, `npm test && npm run build`, or equivalent

If `blocks_on_failure: true` (default) and verification fails: STOP. Resolve the failure, re-dispatch the implementer, re-run the chain from stage 1.

### `kind: domain-review`

Resolve the reviewer per `implementer-reviewer-pairings.json`:
- `default_rule: "from-routing-table"` → look up `route.reviewer` in `routing-table.json` for the implementer that just ran
- Check `overrides` for explicit pairings that win over the default

Dispatch the reviewer via `Agent`. If the reviewer reports issues:
- CRITICAL/HIGH → STOP. Re-dispatch implementer with the findings, re-run chain.
- LOW/MEDIUM → optional resolve based on consumer policy.

### `kind: security-review`

Evaluate the stage's `condition`:
- `kind: paths-match` — check if any changed file matches `condition.paths` (glob)
- `kind: always` — always run
- `kind: never` — skip

If condition NOT satisfied: SKIP. Log `security review: N/A — <reason>` for the post-chain output.

If condition satisfied: dispatch the agent named in the stage's `agent` field (resolved at consumer install time; in the viblocks-style example this is the security-domain reviewer declared in `viv-agents`). CRITICAL/HIGH findings block.

> Per [SPEC Appendix B invariant 7](https://github.com/viblocks/viv-typed-agents/blob/main/SPEC.md), agent names are not hardcoded in playbooks. The canonical source is the `agent` field in `post-implementation-chain.json` (consumer-customizable).

### `kind: commit`

Auto-commit when all preceding stages PASS. Build the commit message:
- Subject (imperative, ≤72 chars)
- Body (optional bullet list)
- Trailers per `.claude/workflows/audit-trail-pattern.json` (e.g. `Audit-Trail: <ID>`)
- Co-author attribution (consumer policy)

**Push is NEVER automatic.** Stop after commit; await user instruction.

## Post-Chain Output

After the chain completes (or stops with an escalation), present this exact format:

```
***** Post-Implementation Chain Complete *****
- Verification: [PASS/FAIL] — [N/N] tests, build [OK/FAIL]
- Code review: [reviewer-agent-name] — [PASS / N issues (severity)]
- Security review: [security-reviewer — PASS/FAIL/N/A] — [findings or "no security-sensitive paths"]
- Files changed: [list]
- Commit: [auto-committing / escalating to user — reason]
```

All five fields required. Never summarize as "review passed" — name the agent.

## Loop semantics

The chain is **not** a single pass. If a downstream stage fails AND a fix is generated:

1. Re-dispatch the implementer with the failure context
2. Re-run the chain from stage 1 (verification)
3. Continue until all stages pass OR the loop is escalated to the user

A consumer may bound the loop count to prevent runaway iteration (e.g. max 3 iterations).

## Issue-tracker integration

If the change is associated with an issue (`<ISSUE_PREFIX>-XXX`):

- The Audit-Trail trailer carries the issue ID
- The issue is closed with evidence per `evidence-schema.json`
- The close comment includes the four required markers

This is orchestrated by `playbooks/issue-driven-flow.md`.

## Failure modes

| Failure | Response |
|---|---|
| Verification fails 3 times in a row | Escalate; the implementer is stuck. Don't keep retrying. |
| Reviewer reports same issue 3 iterations | Escalate; the fix isn't addressing the finding. |
| Security review N/A but consumer policy requires explicit reviewer | Escalate. |
| Audit-Trail value pattern doesn't match | Hook blocks the commit. Fix the trailer; don't bypass. |

## Why a playbook, not code

The chain orchestration is **behavioral** — the LLM reads the rule JSON and executes the stages. A code implementation is also possible (in `viv-hooks/lib/` for example), but the LLM-driven flow allows escalation, scope adjustment, and conversational recovery that pure automation can't provide.

See [ADR-003](../architecture/decisions/ADR-003-iron-law-as-prose.md) for the rationale on prose vs. code.

## Foundations (read for depth)

This page is the **orchestration entry point**. The comprehensive treatment lives in `_common/`:

| Topic | Reference |
|---|---|
| Chain rule definition (consumed by hooks) | [`_common/post-implementation-chain.md`](_common/post-implementation-chain.md) |
| Enforcement architecture (5 layers) | [`_common/enforcement-architecture.md`](_common/enforcement-architecture.md) |
| Audit-and-logging discipline | [`_common/audit-and-logging.md`](_common/audit-and-logging.md) |
| Git workflow (commit conventions) | [`_common/git-workflow.md`](_common/git-workflow.md) |
| Debugging gate (Root cause: contract) | [`_common/debugging-gate.md`](_common/debugging-gate.md) |
| Verification-before-completion (SP integration) | [`_common/superpowers-integration.md`](_common/superpowers-integration.md) |
