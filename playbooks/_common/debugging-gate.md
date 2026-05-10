### L5 Debugging Gate — Grammar Contract

**Purpose**: structural backstop for the debugging discipline already enforced behaviorally by the `superpowers:systematic-debugging` skill (CONDITIONAL on repeat failure) and the local `root-cause-discipline` skill (four-questions gate per `core-change-flow-protocol.md:209`). When those skills are invoked the discipline holds; L5 catches the case where they're not invoked — typically ad-hoc Agent dispatches outside the typed-implementer flow.

This document defines the **grammar contract** that the L5 hook (PR #35/2) parses. Behavioral skills enforce the *spirit*; this document defines what the *structural enforcer* recognizes.

#### Status

- **L5 contract** (this document): defined in PR #35/1
- **L5 hook implementation**: ships in PR #35/2 (rolls out warn-mode first, then hard)
- **Until PR #35/2 ships**: enforcement remains behavioral via skills

---

### 1. Trigger criteria — when L5 fires

L5 fires on **PreToolUse Agent** dispatches (subagent invocation) where the prompt expresses **debugging intent**: a problem-statement seeking diagnosis or symptom resolution.

#### 1.1 Positive triggers (L5 enforces `Root cause:`)

The prompt contains AT LEAST ONE of:

**Verb family** (case-insensitive, word-boundary):
- `fix`, `fixing`, `fixed`
- `debug`, `debugging`
- `diagnose`, `diagnosing`
- `troubleshoot`, `troubleshooting`
- `investigate <bug|issue|regression|failure>`
- `resolve <bug|issue|regression|failure>`

**Symptom-shaped phrasing**:
- `not working`, `doesn't work`, `stopped working`
- `broken`, `is broken`, `keeps breaking`
- `fails`, `failing`, `throws`, `crashes`
- `returns wrong`, `returns incorrect`, `returns null`
- `regression`
- `same X keeps breaking`, `repeat failure`
- `intermittent`, `flaky`, `racy`

#### 1.2 Negative guards (L5 does NOT fire even if a positive trigger matches)

The prompt also matches one of these — explicit fix-intent **with already-known root cause**, or **non-debugging context**:

- **Already-known fix**: `implement the fix from <ref>`, `apply the fix described in <PR|issue>`, `port the fix from <branch>`, `as described in #<num>`
- **Mechanical edit**: `fix the typo`, `fix lint`, `fix formatting`, `fix indentation`, `fix the comment`
- **Test-for-bug**: `add (a |regression |unit |integration )?test for`, `cover the bug with a test` — verification, not debugging
- **Documentation**: `fix the docs`, `fix the README`, `fix the changelog`

**Why guards exist**: trivial mechanical edits and known-fix applications don't require root-cause analysis (it's already done elsewhere). False positives erode trust in the gate; the negative guards make L5's domain precise.

#### 1.3 Boundary cases — explicit decisions

| Prompt | L5 fires? | Reason |
|---|---|---|
| "fix the auth bug where login returns 500" | ✅ Yes | debug intent + symptom |
| "fix the typo in README.md" | ❌ No | mechanical-edit guard |
| "implement the fix described in PR #123" | ❌ No | already-known-fix guard |
| "add a regression test for the login bug" | ❌ No | test-for-bug guard |
| "investigate why the build fails on CI" | ✅ Yes | investigate + failure |
| "fix lint errors in services/api" | ❌ No | mechanical-edit guard (lint) |
| "the same OAuth bug keeps breaking — find the root cause" | ✅ Yes | repeat-failure phrasing |
| "refactor the auth module" | ❌ No | no debug intent |
| "the API is slow under load" | ✅ Yes | symptom-shaped phrasing |
| "fix the docs to explain X" | ❌ No | documentation guard |

The boundary cases are **calibration targets** for the regex in PR #35/2. Any case that the regex misclassifies relative to this table is a defect to fix.

---

### 2. `Root cause:` declaration — what counts

When L5 fires, the prompt must contain a `Root cause:` declaration matching this contract.

#### 2.1 Required form

- **Literal string**: `Root cause:` (case-sensitive, with the colon, with the space-after-colon optional)
- **Followed by content**: at minimum 3 non-whitespace tokens
- **Position**: anywhere in the prompt (header line, inline, end-of-prompt — implementation-defined)

Examples that **pass**:
- `Root cause: race condition in services/auth/src/login.ts:42 — token refreshed before validation`
- `Root cause: missing null check on user.session in the OAuth callback handler — fails when session is undefined on first login`
- `Root cause:\nIncorrect timezone handling — DB returns UTC, frontend assumes local time, off-by-one on day boundaries.`

#### 2.2 Rejection list — placeholders that fail

- Empty: `Root cause:` (nothing after the colon)
- Single token: `Root cause: TBD`, `Root cause: ?`, `Root cause: ???`, `Root cause: unknown`, `Root cause: tbd`, `Root cause: n/a`, `Root cause: na`, `Root cause: TODO`
- Question form: `Root cause: <three or more tokens but ending in ?>`
- Disclaimer: `Root cause: not sure`, `Root cause: still investigating`, `Root cause: pending`

The token-count threshold (3) is **v1, calibrate against real prompt corpus during PR #35/2 review**. The threshold is parameterizable; consumer override mechanism is out of scope for #35 (separate enhancement if needed).

#### 2.3 What is NOT validated

- **Quality of the hypothesis** — L5 does not assess whether the stated root cause is correct, complete, or sound. That's the typed reviewer's job during the chain.
- **File-line specificity** — `Root cause: a bug in the validator` passes the contract even though it's vague. Vagueness is bad practice but not structurally enforceable; the reviewer catches it semantically.
- **Single vs multi-cause** — multi-line root-cause descriptions pass as long as the literal `Root cause:` marker is present and 3+ tokens follow.

The hook is **structural**, not semantic. Semantic quality is the behavioral skills' (and reviewers') domain.

---

### 3. Enforcement modes

| Mode | L5 behavior |
|---|---|
| `disabled` | bypass — no enforcement (consistent with other layers) |
| `warn` | log advisory to stderr, allow Agent dispatch |
| `hard` (default) | block Agent dispatch, return exit 2 with diagnostic |

PR #35/2 ships in **`warn`** mode for the first release window so consumers can adapt prompts. Promotion to `hard` happens in a subsequent release after observed signal.

---

### 4. Cross-references

- **Behavioral counterpart**: `superpowers:systematic-debugging` skill (CONDITIONAL — fires on repeat failure patterns per `superpowers-integration.md:245`)
- **Local skill**: `root-cause-discipline` — four-questions gate per `core-change-flow-protocol.md:209`
- **Implementer template**: `agents/_shared/typed-implementer.template.md:50` — REQUIRED skill invocation for bug-fix/regression tasks
- **IRON LAW**: universal trigger on any failure (`superpowers-integration.md:282` R-07)
- **Architecture row**: `enforcement-architecture.md` L5 (corrected #35)

L5 does **NOT replace** the behavioral skills — it's a structural backstop for the cases where the skills aren't invoked (ad-hoc Agent dispatches, prompts that bypass the typed-implementer flow).

---

### 5. Out of scope (for this contract document)

- **Implementation details** of the regex / parser — that's PR #35/2
- **Validating the quality** of the Root cause declaration (semantic correctness)
- **Extending the gate to non-Agent tools** (Bash with fix intent, etc.) — Agent dispatch is the high-leverage gate point
- **Per-consumer threshold override** — `3 tokens` is plugin-default; if needed, a separate enhancement
- **L3 chain enforcement** — separate concern (#34)
