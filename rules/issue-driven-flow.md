# Issue-Driven Flow Playbook

Autonomous change flow triggered by an issue tracker reference.

## Trigger

User says: "fix `<ISSUE_ID>`", "work on `<ISSUE_ID>`", or pastes an issue URL.

`<ISSUE_ID>` format is consumer-defined — `VI-123` (Linear), `gh-42` (GitHub), `JIRA-PROJ-9` (Jira), etc.

## The triage gate

Read the issue. Evaluate four questions in order. **First match wins.**

### Q1 — Does this require a new abstraction?

If the change introduces a new public API, new module boundary, new service, or new domain concept: **DESIGN path**.

DESIGN is **supervised**: pause for the user to approve the design before implementing.

### Q2 — Does this change span multiple domains?

If the change touches paths in two or more `routing-table.json` domains: **CROSS-DOMAIN path**.

CROSS-DOMAIN is **supervised**: generate a multi-domain plan first, get user approval, then dispatch per domain.

### Q3 — Is this a revert?

If the change is undoing a recent commit/PR (no new logic, just removing or reverting): **REVERT path**.

REVERT is **autonomous**: revert + verify + commit + close issue.

### Q4 — Is this a scoped fix within one domain?

If the change is bug fix, small feature, or refactor within one domain's paths: **DIRECT path**.

DIRECT is **autonomous**: dispatch typed implementer + run chain + commit + close issue.

If none of Q1-Q4 match cleanly, escalate to the user.

## The four paths

### DIRECT (autonomous)

1. Read the issue (description + acceptance criteria)
2. Identify the target paths
3. Resolve the implementer per routing-table
4. Dispatch the typed implementer with:
   - Issue context
   - Acceptance criteria
   - **`Root cause:`** field if this is a fix (per `fix-intent-pattern.json`)
5. Run the post-implementation chain
6. Commit with `Audit-Trail: <ISSUE_ID>` trailer
7. Close the issue with evidence per `evidence-schema.json`

### CROSS-DOMAIN (supervised)

1. Read the issue
2. Identify all touched domains
3. Generate a per-domain plan (shared packages first)
4. **PAUSE** for user approval of the plan
5. For each domain in order:
   - Dispatch typed implementer
   - Run chain
   - Commit
6. Close the issue with evidence aggregating all commits

### DESIGN (supervised)

1. Read the issue
2. Run an Application Design / Functional Design phase (AI-DLC stages, or equivalent)
3. **PAUSE** for user approval of the design
4. Decompose the design into per-domain implementation tasks
5. Execute each task as a DIRECT or CROSS-DOMAIN sub-flow
6. Close with evidence

### REVERT (autonomous)

1. Read the issue (must reference the commit/PR being reverted)
2. Identify the target paths from the original change
3. Resolve the implementer for those paths (revert is still typed-agent dispatch)
4. Dispatch with revert intent (the implementer applies the reverse diff)
5. Run the chain (verification ensures the revert doesn't break anything)
6. Commit with `Audit-Trail: <ISSUE_ID>` and clear "Revert" subject
7. Close the issue

## Issue close evidence

Per `evidence-schema.json`, the close comment MUST include:

```markdown
**Verification**: PASS — N/N tests, build OK
**Code review**: <reviewer-agent> — PASS
**Security review**: PASS|FAIL|N/A -- <justification when N/A>
**Commits**: <SHA-list>
```

A close attempted without all four markers is **blocked** by the consumer hook.

## Fix-intent gate

If the issue is a fix (Q4 path with bug-fix language):

The implementer dispatch prompt MUST include `Root cause:` (or `Causa raíz:`/`Causa raiz:`) followed by an articulated root-cause statement.

A dispatch missing this token (when intent matches `fix|bug|...`) is **blocked** by the fix-intent gate hook (per `fix-intent-pattern.json`).

The gate exists because patches without root-cause analysis recur. See `superpowers:systematic-debugging` for the discipline this enforces.

## Autonomous-vs-supervised summary

| Path | Mode | Pause point |
|---|---|---|
| DIRECT | autonomous | none — full execution |
| REVERT | autonomous | none — full execution |
| CROSS-DOMAIN | supervised | after plan generation |
| DESIGN | supervised | after design approval |

## Failure modes

| Failure | Response |
|---|---|
| Issue tracker unreachable | Escalate. Don't proceed without the issue context. |
| Issue has no acceptance criteria | Escalate. The implementer needs a spec. |
| Multiple matches in Q1-Q4 | Take the first. The triage is intentionally ordered. |
| User reverses scope mid-flow ("actually do X too") | Escalate. The flow re-enters triage with the new scope. |

## Why this flow exists

Issue-driven autonomous change is the **highest** capability tier (Tier 5). It requires:
- A populated routing table (Tier 3)
- Workflow rules (Tier 3)
- Hooks enforcement (Tier 4)
- This playbook (Tier 5)

Without these, the flow degrades to user-supervised execution at every step. With them, an issue closes itself end-to-end with audit trail and evidence.

## Foundations (read for depth)

This page is the **autonomous flow entry point**. The comprehensive treatment lives in `common/`:

| Topic | Reference |
|---|---|
| Core change flow protocol (DIRECT/CROSS-DOMAIN/DESIGN/REVERT detail) | [`common/core-change-flow-protocol.md`](common/core-change-flow-protocol.md) |
| Issue analysis discipline (Phase 1 + Phase 2 with fast-pass) | [`common/overconfidence-prevention.md`](common/overconfidence-prevention.md) |
| Debugging gate (Root cause: contract for fix-intent) | [`common/debugging-gate.md`](common/debugging-gate.md) |
| Audit-and-logging (issue close evidence schema) | [`common/audit-and-logging.md`](common/audit-and-logging.md) |
| Friction reporting (when issues block, where to file) | [`common/friction-reporting.md`](common/friction-reporting.md) |
| Routing table population (when an issue references new paths) | [`common/routing-table-population-protocol.md`](common/routing-table-population-protocol.md) |
