# ADR-003 — IRON LAW expressed as prose, not as YAML/JSON rule

**Status:** Accepted
**Date:** 2026-05-08
**Category:** viv-orchestration-rules local

## Context

viv-routing and viv-workflows ship rule data as JSON. A tempting parallel would be to express the IRON LAW (typed-agent dispatch protocol) as JSON too:

```json
{
  "rule": "iron-law",
  "applies_to": "class_a",
  "required_action": "dispatch typed agent",
  "violations": ["general-purpose for class A", ...]
}
```

This would give tooling something machine-readable.

## Decision

**The IRON LAW and the playbooks are prose** in Markdown. They are **for the LLM**, not for tooling.

Rule data (routing-table, workflow rules) IS for tooling — schemas validate, hooks consume. Orchestration rules are **behavioral guidance for the LLM** — the LLM reads the prose and acts accordingly.

Hooks in `viv-hooks` enforce the **structural** consequences of the IRON LAW (block wrong dispatches at Edit/Write time) — but they enforce against the routing-table rule data, not against this prose.

## Rationale

| Concern | How this satisfies |
|---|---|
| LLM as primary audience | Prose with examples and red flags engages LLM reasoning more effectively than JSON |
| Conditional reasoning | "if A then B unless C" is hard to capture cleanly in JSON; trivial in prose |
| Onboarding | Human contributors read CLAUDE.md and the playbooks; JSON would require docs anyway |
| Separation of concerns | Tooling consumes routing/workflows JSON. Behavior consumes orchestration prose. Clean SRP. |

## Consequences

- Playbooks are reviewed for clarity, completeness, and behavioral accuracy — not validated by schema
- Drift between playbook prose and rule JSON is possible (e.g. playbook says "verify then review" but workflow JSON says "review then verify"). Caught by code review, not tooling.
- A consumer in a different language (Spanish-only team) translates the prose — the typed agents themselves remain in English (per viv-agents convention)

## Alternatives considered

- **Encode IRON LAW as JSON state machine:** rejected — captures the surface but loses the rationale + red flags + edge cases that the LLM uses to generalize
- **Encode the IRON LAW as a SP skill:** rejected — would couple the strategy to SP; this strategy must work without SP at lower tiers
- **Mix prose + structured data (e.g. frontmatter on each playbook):** considered — useful future extension if tooling needs to discover playbooks; not required for the initial extraction

## Related

- viv-workflows ADR-001 (gate-vs-hook boundary; same principle: rules describe, hooks enforce)
- ADR-RD-008 (pure descriptors)
