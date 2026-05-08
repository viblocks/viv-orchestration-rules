# ADR-001 — CLAUDE.md is a template, not a prescription

**Status:** Accepted
**Date:** 2026-05-08
**Category:** viv-orchestration-rules local

## Context

viblocks-ai's `CLAUDE.md` is a single concrete document with project-specific paths, agent names, and project policies (Spanish + English mixed, Linear-specific issue IDs, blacklist-domain advisor). Extracting it as-is would force consumers to either:
- accept viblocks-specific content irrelevant to them, or
- delete sections by hand, risking accidental removal of essential rules.

## Decision

Ship `CLAUDE.template.md` with **explicit placeholders** (`<PROJECT_NAME>`, `<ISSUE_PREFIX>`, `<INFRA_PATHS>`, etc.) and **explicit drop points** (sections marked optional per tier).

A consumer:
1. Copies the template to `<project-root>/CLAUDE.md`
2. Replaces placeholders with project-specific values
3. Drops sections for tiers they don't adopt (T3 consumer drops AI-DLC and SP playbook references)
4. Adds project-specific extensions in the dedicated extension block

The companion `examples/viblocks-style/CLAUDE.md` shows a fully-populated example for reference.

## Rationale

| Concern | How this satisfies |
|---|---|
| Reusability | A greenfield project gets the structure without inheriting viblocks specifics |
| Tier-friendliness | T3 consumers don't see AI-DLC content they won't use |
| Discoverability | Placeholders make it obvious what needs filling in |
| Reference availability | Concrete example exists for consumers who want to see the filled-in form |

## Consequences

- The template uses angle-bracketed placeholders. Tooling MAY scan for unfilled `<...>` tokens and warn during validation.
- The example mirrors viblocks-ai but lives in `examples/`, NOT at the repo root, so consumers don't accidentally vendor it.
- Adding new sections to the template (e.g. for a new tier or extension) is OCP-safe: existing consumers continue to work.

## Alternatives considered

- **Ship the viblocks CLAUDE.md verbatim:** rejected — couples consumers to viblocks specifics
- **Ship multiple per-tier templates:** rejected — duplication; sections drop more naturally than templates branch
- **Ship a CLI generator that asks questions and produces CLAUDE.md:** rejected — code-in-data-repo violates ADR-RD-008

## Related

- ADR-RD-008 (pure descriptors)
- viv-routing routing-table.template.json (same pattern: template + placeholders + concrete example)
