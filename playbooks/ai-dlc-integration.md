# AI-DLC Integration Playbook

How typed agents bind to AI-DLC stages.

> AI-DLC (Application Innovation & Development Lifecycle) is an **external framework**, not part of this strategy. This playbook describes integration only. See [ADR-002](../architecture/decisions/ADR-002-external-deps-referenced.md).

## Scope

This playbook applies when the consumer adopts AI-DLC alongside the typed-agents strategy. Consumers without AI-DLC can ignore this playbook entirely.

## The four phases

AI-DLC defines four phases. Typed agents replace AI-DLC's generic implementer references.

### INCEPTION

Generic AI-DLC stages: Workspace Detection, Reverse Engineering (brownfield), Requirements Analysis, User Stories, Workflow Planning, Application Design, Units Generation.

**Typed agent integration:** none — Inception is design and analysis. No code is generated. Use AI-DLC's standard agents.

### CONSTRUCTION

Generic AI-DLC stages: Functional Design, NFR Requirements/Design, Infrastructure Design, Code Generation.

**Typed agent integration:**
- **Code Generation** stage dispatches **typed implementers** per the routing table — never `general-purpose`
- For multi-domain units, split per domain (per `playbooks/dispatch-protocol.md` cross-domain rules)
- Each typed implementer's IRON LAW (with TDD embedded) is the AI-DLC code-generation contract

### VERIFICATION

Generic AI-DLC stages: Test Strategy/Environment, Test Diagnosis, Test Implementation, E2E Validation, CI Pipeline Implementation, Production Readiness Gate.

**Typed agent integration:**
- **Test Implementation** dispatches typed implementers (TDD) and runs the post-impl chain per level
- **Test Environment Setup** dispatches `infra-devops-implementer`
- **CI Pipeline Implementation** dispatches `infra-devops-implementer` for workflow files
- E2E fixes during validation respect the IRON LAW — typed agents only

### DEPLOYMENT

Generic AI-DLC stages: Deployment Strategy, CD Pipeline Diagnosis/Design/Implementation, Deployment Execution, Post-Deploy Validation, Rollback Validation.

**Typed agent integration:**
- **CD Pipeline Implementation** dispatches `infra-devops-implementer`
- The post-impl chain runs after every infra change (verification + reviewer + security)

## Mid-Workflow Changes

When a mid-workflow change occurs (add/skip stages, change depth, restart), AI-DLC executes its workflow-changes protocol first. Typed-agent dispatch rules apply unchanged — the IRON LAW does not pause for mid-workflow adjustments.

## Skill invocation override

When AI-DLC is active, generic Superpowers triggers are OVERRIDDEN by AI-DLC stage bindings (see `playbooks/superpowers-integration.md`). Concrete:

| SP skill | Generic trigger | AI-DLC override |
|---|---|---|
| `brainstorming` | "before any creative work" | Only at stages marked CONDITIONAL in the binding |
| `test-driven-development` | "when implementing any feature" | Embedded in typed agent IRON LAW; never invoked separately |
| `writing-plans` | "before any implementation" | Only at Code Generation Part 1 |
| `verification-before-completion` | "at every action" | At gates defined in the binding |
| `systematic-debugging` | (no override; universal IRON LAW) | Same as default |

## Why this exists

AI-DLC stages have formal approval gates, audit trails, and sequential dependencies. Generic SP triggers firing out of sequence break orchestration. The IRON LAW provides the one-way constraint: typed agents replace generic implementers wherever AI-DLC requires implementation.

## Reference: where AI-DLC rule details live

A consumer adopting AI-DLC vendors its own rule details at `<.aidlc-rule-details/>` (or equivalent). This strategy does not bundle AI-DLC content.
