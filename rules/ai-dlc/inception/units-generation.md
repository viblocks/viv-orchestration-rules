# Units Generation - Detailed Steps

> **Architecture note (per [ADR-RD-004](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-004-classifier-folded.md)):** the legacy `.claude/context/artifact-classifier.json` no longer exists. Class A scope is derived from `routing-table.json` routes where `enforced: true`. Where this document references the classifier as a separate file, read it as `routing-table.json` with the `enforced` field.


## Overview
This stage decomposes the system into manageable units of work through two integrated parts:
- **Part 1 - Planning**: Create decomposition plan with questions, collect answers, analyze for ambiguities, get approval
- **Part 2 - Generation**: Execute approved plan to generate unit artifacts

**DEFINITION**: A unit of work is a logical grouping of stories for development purposes. For microservices, each unit becomes an independently deployable service. For monoliths, the single unit represents the entire application with logical modules.

**Terminology**: Use "Service" for independently deployable components, "Module" for logical groupings within a service, "Unit of Work" for planning context.

## Prerequisites
- Workspace Detection must be complete
- Requirements Analysis recommended (provides functional scope)
- User Stories recommended (stories map to units)
- Application Design stage REQUIRED (determines components, methods, and services)
- Execution plan must indicate Design stage should execute

---

# PART 1: PLANNING

## Step 1: Create Unit of Work Plan
- Generate plan with checkboxes [] for decomposing system into units of work
- Focus on breaking down the system into manageable development units
- Each step and sub-step should have a checkbox []

## Step 2: Include Mandatory Unit Artifacts in Plan
**ALWAYS** include these mandatory artifacts in the unit plan:
- [ ] Generate `aidlc-docs/inception/application-design/unit-of-work.md` with unit definitions and responsibilities
- [ ] Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md` with dependency matrix
- [ ] Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md` mapping stories to units
- [ ] **Greenfield only**: Document code organization strategy in `unit-of-work.md` (see code-generation.md for structure patterns)
- [ ] Validate unit boundaries and dependencies
- [ ] Ensure all stories are assigned to units

## Step 3: Generate Context-Appropriate Questions
**DIRECTIVE**: Thoroughly analyze the requirements, stories, and application design to identify ALL areas where clarification would improve unit decomposition quality. Be proactive in asking questions to ensure comprehensive coverage of decomposition concerns.

**CRITICAL**: Default to asking questions when there is ANY ambiguity or missing detail that could affect unit boundaries or decomposition quality. It's better to ask too many questions than to make incorrect assumptions about how the system should be decomposed.

**MANDATORY**: Evaluate ALL of the following categories by asking targeted questions about each. For each category, determine applicability based on evidence from the requirements, stories, and application design -- do not skip categories without explicit justification:

- EMBED questions using [Answer]: tag format
- Focus on ANY ambiguities, missing information, or areas needing clarification
- Generate questions wherever user input would improve decomposition decisions
- **When in doubt, ask the question** - overconfidence leads to poor unit boundaries

**Question categories to evaluate** (consider ALL categories):
- **Story Grouping** - Ask about grouping strategy, story affinity, and logical clustering approaches
- **Dependencies** - Ask about integration approach, shared resources, and inter-unit communication patterns
- **Team Alignment** - Ask about team structure, ownership boundaries, and collaboration models
- **Technical Considerations** - Ask about scalability/deployment requirements that may differ across units
- **Business Domain** - Ask about domain boundaries, bounded contexts, and business capability alignment
- **Code Organization (Greenfield multi-unit only)** - Ask about deployment model and directory structure preferences
- **UI Affinity** (only if `aidlc-docs/inception/frontend-design/screen-affinity-map.md` exists) -
  Ask whether stories that share a UI screen should be grouped in the same unit.
  Tradeoff:
    - Same unit: preserves screen cohesion at codegen time, simpler
      `frontend-components.md` per unit
    - Split by backend bounded context: preserves backend cohesion, but
      screen-spec fragments across units
  Default safe: ask explicitly; do not infer.

## Step 4: Store UOW Plan
- Save as `aidlc-docs/inception/plans/unit-of-work-plan.md`
- Include all [Answer]: tags for user input
- Ensure plan covers all aspects of system decomposition

## Step 5: Request User Input
- Ask user to fill [Answer]: tags directly in the plan document
- Emphasize importance of decomposition decisions
- Provide clear instructions on completing the [Answer]: tags

## Step 6: Collect Answers
- Wait for user to provide answers to all questions using [Answer]: tags in the document
- Do not proceed until ALL [Answer]: tags are completed
- Review the document to ensure no [Answer]: tags are left blank

## Step 7: ANALYZE ANSWERS (MANDATORY)
Before proceeding, you MUST carefully review all user answers for:
- **Vague or ambiguous responses**: "mix of", "somewhere between", "not sure", "depends"
- **Undefined criteria or terms**: References to concepts without clear definitions
- **Contradictory answers**: Responses that conflict with each other
- **Missing generation details**: Answers that lack specific guidance
- **Answers that combine options**: Responses that merge different approaches without clear decision rules

## Step 8: MANDATORY Follow-up Questions
If the analysis in step 7 reveals ANY ambiguous answers, you MUST:
- Add specific follow-up questions to the plan document using [Answer]: tags
- DO NOT proceed to approval until all ambiguities are resolved
- Examples of required follow-ups:
  - "You mentioned 'mix of A and B' - what specific criteria should determine when to use A vs B?"
  - "You said 'somewhere between A and B' - can you define the exact middle ground approach?"
  - "You indicated 'not sure' - what additional information would help you decide?"
  - "You mentioned 'depends on complexity' - how do you define complexity levels?"

## Step 9: Request Approval
- Ask: "**Unit of work plan complete. Review the plan in aidlc-docs/inception/plans/unit-of-work-plan.md. Ready to proceed to generation?**"
- DO NOT PROCEED until user confirms

## Step 10: Log Approval
- Log prompt and response in audit.md with timestamp
- Use ISO 8601 timestamp format
- Include complete approval prompt text

## Step 11: Update Progress
- Mark Units Planning complete in aidlc-state.md
- Update the "Current Status" section
- Prepare for transition to Units Generation

---

# PART 2: GENERATION

## Step 12: Load Unit of Work Plan
- [ ] Read the complete plan from `aidlc-docs/inception/plans/unit-of-work-plan.md`
- [ ] Identify the next uncompleted step (first [ ] checkbox)
- [ ] Load the context and requirements for that step

## Step 13: Execute Current Step
- [ ] Perform exactly what the current step describes
- [ ] Generate unit artifacts as specified in the plan
- [ ] Follow the approved decomposition approach from Planning
- [ ] Use the criteria and boundaries specified in the plan

## Step 14: Update Progress
- [ ] Mark the completed step as [x] in the unit of work plan
- [ ] Update `aidlc-docs/aidlc-state.md` current status
- [ ] Save all generated artifacts

## Step 15: Continue or Complete
- [ ] If more steps remain, return to Step 12
- [ ] If all steps complete, verify units are ready for design stages
- [ ] Mark Units Generation stage as complete

## Step 16: Present Completion Message

```markdown
# 🔧 Units Generation Complete

[AI-generated summary of units and decomposition created in bullet points]

> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the units generation artifacts at: `aidlc-docs/inception/application-design/`

> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the units generation if required
> ✅ **Approve & Continue** - Approve units and proceed to **CONSTRUCTION PHASE**
```

## Step 17: Wait for Explicit Approval
- Do not proceed until the user explicitly approves the units generation
- Approval must be clear and unambiguous
- If user requests changes, update the units and repeat the approval process

## Step 18: Record Approval Response
- Log the user's approval response with timestamp in `aidlc-docs/audit.md`
- Include the exact user response text
- Mark the approval status clearly

## Step 19: Update Progress
- Mark Units Generation stage complete in `aidlc-docs/aidlc-state.md`
- Update the "Current Status" section
- Prepare for transition to CONSTRUCTION PHASE

## Step 20: Plugin Enforcement Bootstrap (MANDATORY POST-COMPLETION, CONDITIONAL)

**Trigger condition** — execute this step ONLY if BOTH:
1. The project consumes the `aidlc-orchestrator` plugin: `rules/` symlink exists in the project root, AND
2. `routing-table.json` has `"domains": []` (Z2 auto-detect state — bootstrap not yet done)

**Skip condition** — skip this step if:
- `rules/` does not exist (project does not use the plugin), OR
- `routing-table.json` already has populated `domains` (brownfield re-run or prior bootstrap)

**Purpose**: This is the **transition trigger ★** from Phase A → Phase B. Runs `scripts/bootstrap-enforcement.sh` to render typed-agent templates, populate enforcement config, and flip enforcement to hard. Construction MUST NOT begin until this step succeeds.

**INVARIANT**: The full protocol is implemented in `scripts/bootstrap-enforcement.sh`. This step invokes the script — it does NOT re-implement the protocol in prose.

### 20.1 Invoke bootstrap-enforcement.sh

**Compute `has_ui` per unit** (per spec `docs/superpowers/specs/2026-05-05-aidlc-frontend-design-gap-design.md` §8.4) before invoking the script. The bootstrap forwards this boolean to `render-template.sh` as `--set has_ui=<true|false>` so that conditional `{{#has_ui}}...{{/has_ui}}` blocks in the typed-implementer / typed-reviewer templates render correctly.

For each unit U in `aidlc-docs/inception/application-design/unit-of-work.md`:

```
If aidlc-docs/inception/frontend-design/screen-affinity-map.md exists:
  has_ui[U] = true if any story assigned to U appears in screen-affinity-map.md
            else false
Else if aidlc-docs/inception/frontend-design/ui-stack.md exists:
  # Single-screen fallback — see Q5 in frontend-design.md Open implementation questions.
  has_ui[U] = true
Else:
  has_ui[U] = false
```

The bootstrap script computes this internally and passes the per-unit value to `render-template.sh`; the orchestrator does not need to pass `has_ui` on the command line.

Run the script from the plugin's `scripts/` directory (resolved via the `.aidlc-rule-details` symlink parent):

```bash
bash "${PLUGIN_ROOT}/scripts/bootstrap-enforcement.sh" \
  --project-root "${PROJECT_ROOT}" \
  --plugin-root "${PLUGIN_ROOT}"
```

Where:
- `PLUGIN_ROOT` = parent of the `.aidlc-rule-details` symlink target (e.g. `readlink .aidlc-rule-details` → `<plugin>/rules/` → parent is `<plugin>/`)
- `PROJECT_ROOT` = workspace root (where `aidlc-docs/` lives)

The script handles: per-unit `has_ui` computation, template rendering, routing-table.json, artifact-classifier.json, enforcement flip, validation, and audit log entry. On success it exits 0. On failure it exits non-zero with a diagnostic message — do not proceed until it exits 0.

### 20.2 Block Construction until bootstrap succeeds

**CRITICAL**: Construction phase stages (Functional Design, NFR Requirements, NFR Design, Infrastructure Design, Code Generation) MUST NOT begin until `bootstrap-enforcement.sh` exits 0. If Construction is requested before this step completes:

1. Halt the Construction stage immediately.
2. Emit: *"Cannot start [Construction Stage]: Plugin Enforcement Bootstrap (Step 20) must complete first. Run bootstrap-enforcement.sh and confirm it exits 0."*
3. Resume only after bootstrap succeeds.

---

## Critical Rules

### Planning Phase Rules
- Generate ONLY context-relevant questions
- Use [Answer]: tag format for all questions
- Analyze all answers for ambiguities before proceeding
- Resolve ALL ambiguities with follow-up questions
- Get explicit user approval before generation

### Generation Phase Rules
- **NO HARDCODED LOGIC**: Only execute what's written in the unit of work plan
- **FOLLOW PLAN EXACTLY**: Do not deviate from the step sequence
- **UPDATE CHECKBOXES**: Mark [x] immediately after completing each step
- **USE APPROVED APPROACH**: Follow the decomposition methodology from Planning
- **VERIFY COMPLETION**: Ensure all unit artifacts are complete before proceeding

## Completion Criteria
- All planning questions answered and ambiguities resolved
- User approval obtained for the plan
- All steps in unit of work plan marked [x]
- All unit artifacts generated according to plan:
  - `unit-of-work.md` with unit definitions
  - `unit-of-work-dependency.md` with dependency matrix
  - `unit-of-work-story-map.md` with story mappings
- Units verified and ready for per-unit design stages
