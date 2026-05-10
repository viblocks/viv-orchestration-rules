# Friction Reporting During AI-DLC Execution

**When this rule applies**: any AIDLC stage. The model reads this when it encounters unexpected behavior, an error, or rough UX during workflow execution.

---

## What counts as friction

- A plugin hook blocked something the AIDLC stage required
- A rule file (this directory tree, `rules/`) gave unclear or contradictory instruction
- A typed-agent template rendered with leftover `{{placeholders}}` or had wrong-domain prose
- A plugin-grade skill (`root-cause-discipline`, `owasp-security`, etc.) lacked a pattern needed for the task
- A plugin script errored or produced wrong output
- The consumer's own files (project agents, skills, routing-table, classifier, CLAUDE.md, product code) need a fix

NOT friction (don't report):
- The AIDLC workflow asked for explicit user input you don't have yet — that's normal flow, ask the user.
- A typed agent's review found valid issues in the implementer's output — that's the chain working correctly.
- A test failed during `verify` — that's information for the implementer, not a tracker issue.

---

## Mandatory routing decision

Before opening any issue, classify the source per [`<plugin>/docs/issue-routing.md`](../../docs/issue-routing.md). Decision tree summarized:

| If friction is in... | Tracker |
|---|---|
| Plugin hook, rule, template, skill, script, doc | **Plugin tracker**: `fabianyvidal/aidlc-orchestrator` |
| Consumer's own files or product code | **Consumer project tracker** (current repo) |

When in doubt → plugin tracker with `type:friction` label and `Routing question:` title prefix.

## Explicit `--repo` flag

When using `gh issue create` from inside a consumer project session, you MUST pass `--repo <owner>/<name>` if reporting to the plugin tracker. Without it, gh defaults to the consumer's remote — wrong destination.

```bash
# Plugin issue (correct)
gh issue create --repo fabianyvidal/aidlc-orchestrator --title "..." ...

# Consumer issue (correct, no --repo)
gh issue create --title "..." ...
```

## Required fields when reporting to plugin tracker

Use the appropriate template at https://github.com/viblocks/viv-orchestration-rules/issues/new/choose:

- `bug-report.md` — broken behavior
- `friction-report.md` — worked but awkward
- `feature-request.md` — net-new capability

Fields to fill (minimum):
1. **Plugin version** — `claude plugin list | grep aidlc-orchestrator`
2. **Consumer project** — name of current project (e.g. `video-vault`)
3. **AIDLC stage when surfaced** — pick from the checklist
4. **Severity** — your assessment
5. **Symptom verbatim** — quote the exact error or unexpected behavior
6. **Reproduction steps** — minimum to reproduce
7. **Workaround applied** — if any (this is valuable signal)

## When to report mid-execution vs at the end

**Report immediately** if:
- Friction blocks the current stage (you can't proceed)
- Workaround is destructive or trades correctness for speed
- The friction looks like a security/data-integrity bug

**Defer to end-of-stage** if:
- Friction is a minor UX wart that doesn't block progress
- Multiple frictions of the same type — batch into one issue with all instances

**Defer to end-of-AIDLC-phase** if:
- Friction surfaces a pattern across multiple stages (better as one architectural issue than N tactical ones)

## Workaround documentation

When you apply a workaround to keep AIDLC moving, document it in two places:
1. **The issue body** — under "Workaround applied"
2. **`aidlc-docs/audit.md`** — append a "Workaround log" entry with timestamp + issue link + brief description

This ensures future maintenance (or re-execution) can recognize the workaround when the underlying issue is fixed.

## Linking issues to AIDLC artifacts

If the friction relates to a specific AIDLC artifact (e.g. a unit definition, a typed-agent rendering), include the path in the issue body:

```
Affected artifact: aidlc-docs/inception/application-design/unit-of-work.md (line 42)
Affected typed agent: .claude/agents/backend-implementer.md
```

This helps the maintainer understand the context without asking.

## When the friction is the consumer's own file

If the routing decision says "consumer tracker" and the consumer is the user you're working with:
- For solo projects (single-developer): you can ask the user for permission to fix in-line (no separate issue) since the consumer = user
- For team projects: still open the consumer-side issue for team visibility, but you may also fix in-line if the user authorizes

## Don't lose findings

The worst outcome is friction surfaces during AIDLC execution and gets forgotten. If you're unsure whether something is plugin-side or consumer-side, **err toward over-reporting to the plugin tracker**. Maintainer triage is cheap; lost findings are expensive.

## See also

- [`<plugin>/docs/issue-routing.md`](../../docs/issue-routing.md) — full decision tree with examples
- [`<plugin>/docs/tracking-bugs-and-improvements.md`](../../docs/tracking-bugs-and-improvements.md) — process, labels, triage
- [`<plugin>/docs/known-limitations.md`](../../docs/known-limitations.md) — already-known issues; check before reporting duplicates
