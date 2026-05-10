### SP Skill Invocation Override (CRITICAL)

**During AI-DLC workflow execution (full workflow or change flow), the operative extracts are the SOLE AUTHORITY for SP skill invocation.** Generic SP skill triggers (e.g., "use brainstorming before any creative work", "use TDD when implementing any feature") are OVERRIDDEN by the precise stage bindings.

| Mode | Authority for SP invocation | Generic SP triggers |
|---|---|---|
| **AI-DLC full workflow** (greenfield OR brownfield: Inception → Construction → B&C → Verification) | `_common/superpowers-integration.md` | OVERRIDDEN — only invoke skills where the binding says to |
| **AI-DLC change flow** (interactive, invoked via "basado en ai dlc change flow VI-XXX") | `_common/core-change-flow-protocol.md` | OVERRIDDEN — only invoke skills where the path protocol says to |
| **Issue-Driven change flow** (autonomous, post-production) | `_common/core-change-flow-protocol.md` | OVERRIDDEN — only invoke skills where the path protocol says to |
| **AI-DLC VERIFICATION phase** | `_common/superpowers-integration.md` (VERIFICATION section) | OVERRIDDEN — only invoke skills where the binding says to |
| **AI-DLC DEPLOYMENT phase** | `_common/superpowers-integration.md` (DEPLOYMENT section) | OVERRIDDEN — only invoke skills where the binding says to |
| **Outside AI-DLC + path Class B** | Default SP behavior (`using-superpowers`) | ACTIVE |
| **Outside AI-DLC + path Class A** | `_common/core-change-flow-protocol.md` | OVERRIDDEN — apply same formalism as F1-F3 per PF20 |

**Concrete overrides**:
- `brainstorming` → ONLY at stages marked CONDITIONAL in the binding, NOT "before any creative work"
- `test-driven-development` → embedded in typed agent IRON LAW, NEVER invoked as a separate skill
- `writing-plans` → ONLY at Code Generation Part 1, NOT "before any implementation"
- `verification-before-completion` → at gates defined in the binding, NOT at every action
- `systematic-debugging` → IRON LAW applies universally (no override — same behavior)
- `dev-testing-strategy` → ONLY at VERIFICATION Stage 1, NOT generically
- `docker-patterns` → ONLY at VERIFICATION Stages 2-3, NOT generically
- `ci-cd-patterns` → ONLY at VERIFICATION Stage 7, NOT generically
- `infra-cloud` → ONLY at VERIFICATION Stage 2 (MANDATORY) and Stage 3 (CONDITIONAL)
- `ci-cd-patterns` → MANDATORY at DEPLOYMENT Stage 3 (CD Pipeline Design), NOT generically
- `docker-patterns` → CONDITIONAL at DEPLOYMENT Stage 4 (CD Pipeline Implementation) for container-based deployments

**Why**: AI-DLC stages have formal approval gates, audit trails, and sequential dependencies. Generic SP triggers firing out of sequence would break orchestration — e.g., brainstorming at Workspace Detection, or writing-plans at Requirements Analysis.

Core protocol: `_common/core-change-flow-protocol.md` (shared by AI-DLC flow + Issue-Driven flow)
