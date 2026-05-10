
> **Architecture note (per [ADR-RD-004](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-004-classifier-folded.md)):** the legacy `.claude/context/artifact-classifier.json` no longer exists. Class A scope is derived from `routing-table.json` routes where `enforced: true`. Where this document references the classifier as a separate file, read it as `routing-table.json` with the `enforced` field.

## MANDATORY: Plan-Level Checkbox Enforcement

### MANDATORY RULES FOR PLAN EXECUTION
1. **NEVER complete any work without updating plan checkboxes**
2. **IMMEDIATELY after completing ANY step described in a plan file, mark that step [x]**
3. **This must happen in the SAME interaction where the work is completed**
4. **NO EXCEPTIONS**: Every plan step completion MUST be tracked with checkbox updates

### Two-Level Checkbox Tracking System
- **Plan-Level**: Track detailed execution progress within each stage
- **Stage-Level**: Track overall workflow progress in aidlc-state.md
- **Update immediately**: All progress updates in SAME interaction where work is completed

## Prompts Logging Requirements
- **MANDATORY**: Log EVERY user input (prompts, questions, responses) with timestamp in audit.md
- **MANDATORY**: Capture user's COMPLETE RAW INPUT exactly as provided (never summarize)
- **MANDATORY**: Log every approval prompt with timestamp before asking the user
- **MANDATORY**: Record every user response with timestamp after receiving it
- **CRITICAL**: ALWAYS append changes to EDIT audit.md file, NEVER use tools and commands that completely overwrite its contents
- **CRITICAL**: NEVER use file writing tools and commands that overwrite the entire contents of audit.md, as this causes duplication
- Use ISO 8601 format for timestamps (YYYY-MM-DDTHH:MM:SSZ)
- Include stage context for each entry

### Audit Log Schema

Every entry in `aidlc-docs/audit.md` MUST satisfy:

| Required field | Form | Validator check |
|---|---|---|
| Heading | Single line starting with `## ` | Format |
| Timestamp | `**Timestamp**: <ISO 8601>` (within first 5 non-empty lines after heading) | Format + monotonic (≥ previous entry) |

Beyond those two required fields, the entry's body is free-form. Common entry types and their additional fields:

**Prompt-logging entry** (most common — every user/AI exchange):
```markdown
## [Stage Name or Interaction Type]
**Timestamp**: [ISO 8601]
**User Input**: "[Complete raw user input - never summarized]"
**AI Response**: "[AI's response or action taken]"
**Context**: [Stage, action, or decision made]

---
```

**Plugin Enforcement Bootstrap entry** (Phase A → Phase B transition — written by `bootstrap-enforcement.sh`):
```markdown
## Plugin Enforcement Bootstrap (Greenfield Phase A → Phase B)
**Timestamp**: [ISO 8601]
**Trigger**: Units Generation user approval (Step 17)
**Units mapped to agents**: <count>
...
```

**Post-Implementation Chain entry** (one per chain run, after typed implementer + chain steps complete — written by orchestrator before commit):
```markdown
## Post-Implementation Chain (<unit-name> / <issue-id>)
**Timestamp**: [ISO 8601]
**Verification**: PASS — N/N tests, build OK
**Code Review**: <reviewer-agent-name> — PASS
**Security Review**: <agent-name | N/A> — <PASS | FAIL | N/A> — <details>
**Files Changed**: <comma-separated paths>
**Commit**: <auto-committing as <id> | escalating to user — reason>
```

Format contract (strict) defined in `post-implementation-chain.md` "Post-Chain Output" section. The L4b commit-time gate (#34/2, ships next) parses these entries to verify the chain actually ran for staged Class A files; an entry must exist with `Timestamp` ≥ the most recent Class A modification. Required leading-enum values per field — free-text suffix is preserved for humans but not gate-validated. Without this entry, an L4 trailer alone is insufficient evidence: the commit gate falls back to soft (warn).

**Append-only invariant**: New entries MUST be appended to the end. The byte-prefix of the file MUST equal the previous committed version. This is the most important invariant and is enforced by `scripts/validate-audit-md.sh` (run in pre-commit and/or CI).

### Cross-trail invariant (audit.md ↔ ad-hoc-audit/*.jsonl)

The two audit trails must stay consistent:

- **Every commit recorded in `ad-hoc-audit/*.jsonl` `commits[]` MUST be referenced (hash mention) somewhere in `audit.md`.** Catches the case where an F4 ad-hoc commit lands structurally but never narratively.
- **Every JSONL row's `paths[]` MUST be Class B** (per the consumer's `artifact-classifier.json`). F4 is for Class B ad-hoc changes; a Class A path in JSONL means the wrong trail was used.
- **(Soft)** Hashes mentioned in `audit.md` that don't appear in any JSONL row are fine in general (CROSS-DOMAIN, DESIGN, REVERT commits don't go in JSONL), but suspicious if the surrounding context says F4 — emitted as advisory.

Enforced by `scripts/validate-audit-cross.sh`.

### Correct Tool Usage for audit.md

✅ CORRECT:

1. Read the audit.md file
2. Append/Edit the file to make changes

❌ WRONG:

1. Read the audit.md file
2. Completely overwrite the audit.md with the contents of what you read, plus the new changes you want to add to it
