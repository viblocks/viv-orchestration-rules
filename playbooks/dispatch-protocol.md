# Dispatch Protocol

The step-by-step protocol the LLM follows for every code change.

## The IRON LAW

> All creation, edit, fix, or improvement of **application code** (Class A paths per `routing-table.json`) executes EXCLUSIVELY through the correct typed agent. No exceptions.

Application code = any path where `enforced: true` in the routing table.
Non-application code = any path where `enforced: false` (free edit).

## Step-by-step

### 1. Identify the target paths

Before any Edit/Write, list the paths the change will touch.

### 2. Resolve the domain per path

For each path, look up `routing-table.json`:
- Find the route(s) matching the path (longest-match wins per [viv-routing ADR-001](https://github.com/viblocks/viv-routing/blob/main/architecture/decisions/ADR-001-longest-match-wins.md))
- The matched route's `domain` is the dispatch domain

### 3. Apply the unknown-service rule

If a path has no matching route, or has `domain: null` / placeholder unfilled:

**STOP.** Do not dispatch. Escalate to the user with options:
- (A) classify the path and add a route to `routing-table.json`
- (B) declare it Class B (`enforced: false`) and proceed without typed agent
- (C) defer the change

Never silently fall back to `general-purpose` for unmatched Class A paths.

### 4. Resolve the implementer

`route.implementer` is the typed agent for writes.

If `route.implementer` is `null` (e.g. testing, docs), the change is written by **another domain's implementer** as part of their TDD or by direct edit (consumer policy).

### 5. Resolve the reviewer

`route.reviewer` is the typed agent for review. The pairing is consumed via `.claude/workflows/implementer-reviewer-pairings.json` (see [viv-workflows ADR-003](https://github.com/viblocks/viv-workflows/blob/main/architecture/decisions/ADR-003-pairings-derived-from-routing.md)).

### 6. Dispatch the implementer

Invoke `Agent` with `subagent_type: <route.implementer>`.

The implementer's system prompt embeds:
- Required skills (auto-loaded)
- Critical Constraints
- IRON LAW with TDD embedded
- Behavior contract (modifies meta? modifies source? uses network? destructive git?)

### 7. Run the Post-Implementation Chain

After the implementer returns, follow `playbooks/post-implementation-chain.md`.

### 8. Cross-domain changes

If the change spans multiple domains:

1. Split the plan by domain.
2. Shared packages (e.g. `packages/shared`) are **Task 1** — they define contracts the others consume.
3. Sequential dispatch: domain A → domain A chain → commit → domain B → domain B chain → commit.
4. Do NOT batch multi-domain changes into a single dispatch.

## Behavioral red flags

These thoughts signal a violation about to happen — STOP:

| Thought | Why it's a red flag |
|---|---|
| "I'll just fix this one line directly" | Class A is Class A. One line still requires dispatch. |
| "general-purpose can handle this" | If the path is enforced, no it can't. |
| "It's only a config file" | Config files in `<INFRA_PATHS>` are Class A. |
| "TDD doesn't apply for this small change" | TDD is in the implementer's IRON LAW. The implementer decides, not you. |
| "I'll dispatch first and figure out the domain after" | Dispatch is downstream of domain resolution. |

## Why this protocol exists

Dispatch by path (not by keyword) is **deterministic**. Any LLM following this protocol on the same routing table reaches the same dispatch decision. That's what makes structural enforcement (viv-hooks) viable: the hook can verify the LLM's decision against the same rule.

See [ADR-RD-007](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-007-defer-validation.md) for the rationale on Edit/Write-time validation.
