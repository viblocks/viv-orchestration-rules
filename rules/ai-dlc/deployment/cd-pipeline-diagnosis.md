# CD Pipeline Diagnosis

**Purpose**: Map what CI/CD already exists before designing — avoid reimplementing ad-hoc deploys and preserve operational knowledge embedded in existing automation.

**Stage type**: ANALYSIS (reads current state, reports diagnosis — no Question DNA required)

**Skip IF**: Greenfield OR no existing CI/CD workflows detected. On skip, log reason in audit.md and mark stage `[x]` with SKIPPED in aidlc-state.md.

## Prerequisites
- Stage 1 Deployment Strategy Design must be complete
- Brownfield detection must be `true` (prior CD workflows OR prior production deploys OR live services)

---

## Step 1: Inventory Existing Automation

Enumerate all deployment-related artifacts in the repo:

| Location | What to look for |
|---|---|
| `.github/workflows/**` (or `.gitlab-ci.yml`, etc.) | Workflow files related to build/deploy |
| `scripts/deploy*`, `scripts/release*` | Deploy shell scripts |
| `Makefile` | Deploy-related targets |
| IaC files (`*.tf`, `*.yml`, `cdk/**`, `pulumi/**`) | Infrastructure deployment specs |
| Dockerfiles + `docker-compose*` | Container deployment |
| Team documentation | Deploy runbooks, READMEs |

Output: Inventory table in artifact.

---

## Step 2: Classification

For each artifact, classify:
- **CI**: build + test only
- **CD**: actually deploys to an environment
- **Mixed**: both

---

## Step 3: Ad-Hoc Deploy Detection

Review commit history, team-provided documentation, and explicit user input for undocumented deploys that are not codified. Ask the user explicitly:

> "Does your team perform any deploys that are NOT in the codified workflows above? (e.g., manual SSH to servers, cloud-console manual deploys, ad-hoc scripts). If yes, describe."

Save response to artifact as "Ad-Hoc Deploy List".

---

## Step 4: Gap vs Stage 1 Strategy

Compare existing CD against the strategy selected in Stage 1. For each artifact from Step 1, identify:
- **Keep**: aligns with strategy
- **Refactor**: aligns in purpose but needs adjustment
- **Replace**: misaligned, must be rewritten
- **Deprecate**: no longer needed

Output: Gap report.

---

## Step 5: Migration Path

Recommend incremental migration:
- Order of changes
- Which ad-hoc deploys to codify first (Priority 1)
- Which existing workflows can stay as-is (Priority 3)
- Transition timeline (before Stage 5 Execution, or accepted as Known Limitation)

---

## Step 6: Generate Diagnosis Artifact

Generate `aidlc-docs/deployment/cd-pipeline-diagnosis.md` with inventory, classification, ad-hoc list, gap report, and migration path.

---

## Step 7: Completion Message

### Part 1 — Announcement

```markdown
# 🟡 CD Pipeline Diagnosis Complete
```

### Part 2 — AI Summary

Bullet summary of: artifacts inventoried count, CI/CD/mixed breakdown, ad-hoc deploys discovered, gap count, recommended migration priority.

### Part 3 — Workflow Message

```markdown
> **📋 REVIEW REQUIRED:**
> Please examine the diagnosis at: `aidlc-docs/deployment/cd-pipeline-diagnosis.md`

> **🚀 WHAT'S NEXT?**
>
> 🔧 **Request Changes** - Ask for modifications to the diagnosis
> ✅ **Continue to Next Stage** - Approve Diagnosis and proceed to **CD Pipeline Design**

---
```

---

## Step 8: Approval Gate

PROCEED / AMEND (max 3 cycles per Pattern 3.4). On PROCEED, record approval, mark stage complete in aidlc-state.md.

---

## Extension Compliance Hooks

Diagnosis is read-only; most extensions are N/A. If security-baseline is enabled, include a compliance summary line noting `N/A (diagnosis stage, no artifacts produced that fall under security review)`.
