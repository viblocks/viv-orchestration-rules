### Post-Implementation Chain (MANDATORY)

After any typed implementer agent completes:

1. **Dispatch `superpowers:verification-before-completion`** — tests + build must pass.
2. **Dispatch typed code reviewer** for the domain:
   - Backend (`services/core`, `services/bot`, `packages/shared`) → `backend-crypto-reviewer`
   - Frontend (`services/ui`) → `frontend-crypto-reviewer`
3. If the reviewer reports issues → resolve and return to step 1.
4. **Dispatch `security-reviewer` IF changed files match trigger criteria** (see owasp-security skill):
   - Controllers, DTOs, guards, auth, frontend forms, config with secrets/CORS, package.json, chain/enrichment paths
   - If no files match → skip, log `security review: N/A — no security-sensitive paths`
   - If CRITICAL/HIGH findings → resolve and return to step 1
5. Only after all gates pass: **commit automático** (sin preguntar al usuario) + close issue/CR.
6. **Push NUNCA automático** — solo cuando el usuario lo pide explícitamente.

### Post-Chain Output (MANDATORY)

After the chain completes, present this structured output to the user **AND append the same content as a structured entry to `aidlc-docs/audit.md`** (per `audit-and-logging.md` schema). The append is what makes the chain auditable post-mortem and enables the L4b commit-time chain-evidence gate (#34/2) — a chain run with no audit.md entry is indistinguishable from a chain that never ran.

#### Format (strictly parseable — contract for the L4b gate parser)

```markdown
## Post-Implementation Chain (<unit-name> / <issue-id>)
**Timestamp**: <ISO 8601>
**Verification**: PASS — N/N tests, build OK
**Code Review**: <reviewer-agent-name> — PASS
**Security Review**: <security-reviewer | N/A> — <PASS | FAIL | N/A> — <findings summary | "no security-sensitive paths">
**Files Changed**: <comma-separated paths>
**Commit**: <auto-committing as VI-XXX | escalating to user — reason>

---
```

#### Field grammar (each line must match exactly)

| Field | Leading enum | Free-text suffix |
|---|---|---|
| `**Verification**:` | `PASS` \| `FAIL` | ` — N/N tests, build OK\|FAIL` |
| `**Code Review**:` | `<agent-name>` (no spaces) | ` — PASS \| FAIL \| N issues (severity)` |
| `**Security Review**:` | `<agent-name>` \| `N/A` | ` — PASS \| FAIL \| N/A — <details>` |
| `**Files Changed**:` | comma-separated path list | (none) |
| `**Commit**:` | `auto-committing` \| `escalating` \| `manual` | ` — <reason or trailer>` |

The leading enum is what the gate parses; free-text suffix is preserved for human review but not validated.

#### Example (canonical)

```markdown
## Post-Implementation Chain (api / VI-101)
**Timestamp**: 2026-05-04T14:30:00Z
**Verification**: PASS — 42/42 tests, build OK
**Code Review**: backend-crypto-reviewer — PASS
**Security Review**: security-reviewer — PASS — controllers + DTOs reviewed, no critical findings
**Files Changed**: services/api/src/users.controller.ts, services/api/src/users.service.ts
**Commit**: auto-committing as VI-101

---
```

All 5 fields are required. Do NOT summarize as "code review limpio" — name the reviewer agent explicitly. Do NOT invent values: if a step did not run (e.g., orchestrator skipped security review when it should have), the entry must reflect that honestly (`Security Review: N/A` only when no security-sensitive paths were touched).

This chain is NOT optional. Do not commit with CR closure language or close Linear issues until the chain completes AND the entry is appended to audit.md.
