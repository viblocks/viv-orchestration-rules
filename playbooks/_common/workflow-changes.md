# Mid-Workflow Changes and Stage Management

## Overview

Users may request changes to the execution plan or stage execution during the workflow. This document provides guidance on handling these requests safely and effectively.

---

## Types of Mid-Workflow Changes

### 1. Adding a Skipped Stage

**Scenario**: User wants to add a stage that was originally skipped

**Example**: "Actually, I want to add user stories even though we skipped that stage"

**Handling**:
1. **Confirm Request**: "You want to add User Stories stage. This will create user stories and personas. Confirm?"
2. **Check Dependencies**: Verify all prerequisite stages are complete
3. **Update Execution Plan**: Add stage to `execution-plan.md` with rationale
4. **Update State**: Mark stage as "PENDING" in `aidlc-state.md`
5. **Execute Stage**: Follow normal stage execution process
6. **Log Change**: Document in `audit.md` with timestamp and reason

**Considerations**:
- May need to update later stages that could benefit from new artifacts
- Existing artifacts may need revision to incorporate new information
- Timeline will be extended

---

### 2. Skipping a Planned Stage

**Scenario**: User wants to skip a stage that was planned to execute

**Example**: "Let's skip the NFR Design stage for now"

**Handling**:
1. **Confirm Request**: "You want to skip NFR Design. This means no NFR patterns or logical components will be incorporated. Confirm?"
2. **Warn About Impact**: Explain what will be missing and potential consequences
3. **Get Explicit Confirmation**: User must explicitly confirm understanding of impact
4. **Update Execution Plan**: Mark stage as "SKIPPED" with reason
5. **Update State**: Mark stage as "SKIPPED" in `aidlc-state.md`
6. **Adjust Later Stages**: Note that later stages may need manual setup
7. **Log Change**: Document in `audit.md` with timestamp and reason

**Considerations**:
- Later stages may fail or require manual intervention
- User accepts responsibility for missing artifacts
- Can be added back later if needed

---

### 3. Restarting Current Stage

**Scenario**: User is unhappy with current stage results and wants to redo it

**Example**: "I don't like these user stories. Can we start over?"

**Handling**:
1. **Understand Concern**: "What specifically would you like to change about the stories?"
2. **Offer Options**:
   - **Option A**: Modify existing artifacts (faster, preserves some work)
   - **Option B**: Complete restart (clean slate, more time)
3. **If Restart Chosen**:
   - Archive existing artifacts: `{artifact}.backup.{timestamp}`
   - Reset stage checkboxes in plan file
   - Mark stage as "IN PROGRESS" in `aidlc-state.md`
   - Clear stage completion status
   - Re-execute from beginning
4. **Log Change**: Document reason for restart and what will change

**Considerations**:
- Existing work will be lost (but backed up)
- May need to redo dependent stages
- Timeline will be extended

---

### 4. Restarting Previous Stage

**Scenario**: User wants to go back and redo a completed stage

**Example**: "I want to change the architectural decision we made earlier"

**Handling**:
1. **Assess Impact**: Identify all stages that depend on the stage to be restarted
2. **Warn User**: "Restarting Application Design will require redoing: Units Planning, Units Generation, per-unit design (all units), Code Generation. Confirm?"
3. **Get Explicit Confirmation**: User must understand full impact
4. **If Confirmed**:
   - Archive all affected artifacts
   - Reset all affected stages in `aidlc-state.md`
   - Clear checkboxes in all affected plan files
   - Return to the stage to restart
   - Re-execute from that point forward
5. **Log Change**: Document full impact and reason for restart

**Considerations**:
- Significant rework required
- All dependent stages must be redone
- Timeline will be significantly extended
- Consider if modification is better than restart

---

### 5. Changing Stage Depth

**Scenario**: User wants to change the depth level of current or upcoming stage

**Example**: "Let's do a comprehensive requirements analysis instead of standard"

**Handling**:
1. **Confirm Request**: "You want to change Requirements Analysis from Standard to Comprehensive depth. This will be more thorough but take longer. Confirm?"
2. **Update Execution Plan**: Change depth level in `workflow-planning.md`
3. **Adjust Approach**: Follow comprehensive depth guidelines for the stage
4. **Update Estimates**: Inform user of new timeline estimate
5. **Log Change**: Document depth change and reason

**Considerations**:
- More depth = more time but better quality
- Less depth = faster but may miss details
- Can only change before or during stage, not after completion

---

### 6. Pausing Workflow

**Scenario**: User needs to pause and resume later

**Example**: "I need to stop for now and continue tomorrow"

**Handling**:
1. **Complete Current Step**: Finish the current step in progress if possible
2. **Update Checkboxes**: Mark all completed steps with [x]
3. **Update State**: Ensure `aidlc-state.md` reflects current status
4. **Log Pause**: Document pause point in `audit.md`
5. **Provide Resume Instructions**: "When you return, I'll detect your existing project and offer to continue from: [current stage, current step]"

**On Resume**:
1. **Detect Existing Project**: Check for `aidlc-state.md`
2. **Load Context**: Read all artifacts from completed stages
3. **Show Status**: Display current stage and next step
4. **Offer Options**: Continue where left off or review previous work
5. **Log Resume**: Document resume point in `audit.md`

---

### 7. Changing Architectural Decision

**Scenario**: User wants to change from monolith to microservices (or vice versa)

**Example**: "Actually, let's do microservices instead of a monolith"

**Handling**:
1. **Assess Current Progress**: Determine how far into workflow
2. **Explain Impact**: 
   - If before Units Planning: Minimal impact, just update decision
   - If after Units Planning: Must redo Units Planning, Units Generation, all per-unit design
   - If after Code Generation: Significant rework required
3. **Recommend Approach**:
   - Early in workflow: Restart from Application Design stage
   - Late in workflow: Consider if modification is feasible vs. restart
4. **Get Confirmation**: User must understand full scope of change
5. **Execute Change**: Follow restart procedures for affected stages

**Considerations**:
- Architectural changes have cascading effects
- Earlier in workflow = easier to change
- Later in workflow = consider cost vs. benefit

---

### 8. Adding/Removing Units

**Scenario**: User wants to add or remove units after Units Generation

**Example**: "We need to split the Payment unit into Payment and Billing"

**Handling**:
1. **Assess Impact**: Determine which units have completed design/code
2. **Explain Consequences**:
   - Adding unit: Need to do full design and code for new unit
   - Removing unit: Need to redistribute functionality to other units
   - Splitting unit: Need to redo design and code for both resulting units
3. **Update Unit Artifacts**:
   - Modify `unit-of-work.md`
   - Update `unit-of-work-dependency.md`
   - Revise `unit-of-work-story-map.md`
4. **Reset Affected Units**: Mark affected units as needing redesign
5. **Execute Changes**: Follow normal unit design and code process for affected units

**Considerations**:
- Affects all downstream stages for those units
- May affect other units if dependencies change
- Timeline impact depends on how many units affected

---

## General Guidelines for Handling Changes

### Before Making Changes

1. **Understand the Request**: Ask clarifying questions about what user wants to change and why
2. **Assess Impact**: Identify all affected stages, artifacts, and dependencies
3. **Explain Consequences**: Clearly communicate what will need to be redone and timeline impact
4. **Offer Alternatives**: Sometimes modification is better than restart
5. **Get Explicit Confirmation**: User must understand and accept the impact

### During Changes

1. **Archive Existing Work**: Always backup before making destructive changes
2. **Update All Tracking**: Keep `aidlc-state.md`, plan files, and `audit.md` in sync
3. **Communicate Progress**: Keep user informed about what's happening
4. **Validate Changes**: Ensure changes are consistent across all artifacts
5. **Test Continuity**: Verify workflow can continue smoothly after changes

### After Changes

1. **Verify Consistency**: Check that all artifacts are aligned with changes
2. **Update Documentation**: Ensure all references are updated
3. **Log Completely**: Document full change history in `audit.md`
4. **Confirm with User**: Verify changes meet user's expectations
5. **Resume Workflow**: Continue with normal execution from new state

---

## Change Request Decision Tree

```
User requests change
    |
    ├─ Is it current stage?
    |   ├─ Yes: Can modify or restart current stage
    |   └─ No: Go to next question
    |
    ├─ Is it a completed stage?
    |   ├─ Yes: Assess impact on dependent stages
    |   |   ├─ Low impact: Modify and update dependents
    |   |   └─ High impact: Recommend restart from that stage
    |   └─ No: Go to next question
    |
    ├─ Is it adding a skipped stage?
    |   ├─ Yes: Check prerequisites, add to plan, execute
    |   └─ No: Go to next question
    |
    ├─ Is it skipping a planned stage?
    |   ├─ Yes: Warn about impact, get confirmation, skip
    |   └─ No: Go to next question
    |
    └─ Is it changing depth level?
        ├─ Yes: Update plan, adjust approach
        └─ No: Clarify request with user
```

---

## Logging Requirements

### Change Request Log Format

```markdown
## Change Request - [Stage Name]
**Timestamp**: [ISO timestamp]
**Request**: [What user wants to change]
**Current State**: [Where we are in workflow]
**Impact Assessment**: [What will be affected]
**User Confirmation**: [User's explicit confirmation]
**Action Taken**: [What was done]
**Artifacts Affected**: [List of files changed/reset]

---
```

---

## VERIFICATION Phase — Specific Scenarios

### Skipping Test Implementation (Stage 5)

**Scenario**: Test Diagnosis (Stage 4) shows complete and honest coverage — no gaps exist.

**Handling**:
1. **Verify Condition**: Confirm gap report shows zero missing scenarios and zero dishonest tests for must-have priorities (P1-P2)
2. **Mark as Skipped**: Update `aidlc-state.md` — mark Test Implementation as "SKIPPED (no gaps found in diagnosis)"
3. **Log**: Document in `audit.md` with timestamp: "Test Implementation skipped — Test Diagnosis shows complete honest coverage for all must-have scenarios"
4. **Proceed**: Continue directly to E2E Validation (Stage 6)

**Note**: This is the standard conditional path — Test Implementation is CONDITIONAL per CLAUDE.md. Skipping when no gaps exist is expected behavior, not an exception.

---

### Restarting a VERIFICATION Stage

**Scenario**: User wants to re-run a VERIFICATION stage (e.g., re-run Test Diagnosis after fixing environment issues, or re-run Test Strategy after scope change).

**Handling**:
1. **Confirm Request**: "You want to restart [Stage Name]. This will archive current artifacts and re-execute from Step 1. Confirm?"
2. **Archive**: Rename current stage artifacts with `-v[N]` suffix (e.g., `test-diagnosis.md` → `test-diagnosis-v1.md`)
3. **Cascade Check**: Determine which subsequent stages must also reset:
   - Restarting Stage 1 (Test Strategy) → reset ALL subsequent stages (2-8)
   - Restarting Stage 2 (Environment Design) → reset Stages 3-8
   - Restarting Stage 3 (Environment Setup) → reset Stages 4-8
   - Restarting Stage 4 (Diagnosis) → reset Stages 5-8
   - Restarting Stage 5 (Implementation) → reset Stages 6-8
   - Restarting Stage 6 (E2E) → reset Stages 7-8
   - Restarting Stage 7 (CI Pipeline) → reset Stage 8
   - Restarting Stage 8 (Readiness Gate) → no cascade
4. **Update State**: Reset all affected stage checkboxes in `aidlc-state.md` to `[ ]`
5. **SP Adjustment**: Per `_common/superpowers-integration.md` Mid-Workflow Changes — re-evaluate skill bindings for all reset stages
6. **Re-execute**: Start the target stage from Step 1
7. **Log**: Document cascade and rationale in `audit.md`

**Considerations**:
- Artifacts from stages prior to the restarted one are retained
- If the restart is due to a scope change, the user should confirm that prior stage artifacts (e.g., Test Strategy) are still valid

---

### Returning from NO-GO Verdict (Stage 8)

**Scenario**: Production Readiness Gate emits NO-GO. User chooses to return to a specific stage to address blockers.

**Handling**:
1. **Identify Blocking Stage**: The NO-GO verdict specifies which criteria failed and recommends the stage to return to:
   - Test gaps → return to Stage 5 (Test Implementation)
   - E2E failures → return to Stage 6 (E2E Validation)
   - Environment issues → return to Stage 3 (Test Environment Setup)
   - CI failures → return to Stage 7 (CI Pipeline Implementation)
2. **Reset**: Apply the cascade logic from "Restarting a VERIFICATION Stage" above — reset the blocking stage and all subsequent stages
3. **Re-execute**: Start from the blocking stage
4. **Re-evaluate Verdict**: After the blocking stage and all subsequent stages complete, Stage 8 re-executes and emits a new verdict
5. **Log**: Document NO-GO → return → re-execution path in `audit.md`

**Note**: The user may also choose to override NO-GO and proceed to DEPLOYMENT (the "Accept & Continue" option). This is documented as a risk acceptance, not a fix.

---

## Best Practices

1. **Always Confirm**: Never make destructive changes without explicit user confirmation
2. **Explain Impact**: Users need to understand consequences before deciding
3. **Offer Options**: Sometimes there are multiple ways to handle a change
4. **Archive First**: Always backup before making destructive changes
5. **Update Everything**: Keep all tracking files in sync
6. **Log Thoroughly**: Document all changes for audit trail
7. **Validate After**: Ensure workflow can continue smoothly
8. **Be Flexible**: Workflow should adapt to user needs, not force rigid process

---

## DEPLOYMENT Phase — Specific Scenarios

### Adding DEPLOYMENT to an in-progress project post-VERIFICATION

If the project completed VERIFICATION before DEPLOYMENT phase was introduced, the phase can be added retroactively:

1. Check VERIFICATION verdict — must be GO or CONDITIONAL-GO.
2. Execute Stage 1 Phase Bootstrap (Step 0): create aidlc-state section, create aidlc-docs/deployment/ directory.
3. Proceed through 7 stages normally. Brownfield detection at Stage 1 determines if Stage 2 Diagnosis executes.

---

### Re-entering DEPLOYMENT via Change Flow

Per `core-change-flow-protocol.md`, re-entry is triggered by:
- Pipeline-code-only change → re-execute Stages 3+4
- Deployment-strategy change → re-execute Stages 1+3+4+5
- Production-topology change → full re-entry Stages 1–7

The re-entry uses Path DESIGN (for (b)+(c)) or Path CROSS-DOMAIN (when change spans services).

---

### Transitioning from DEPLOYMENT to OPERATIONS (future)

DEPLOYMENT Stage 7 Sign-off produces `operations-handoff.md` — the input contract for the future OPERATIONS phase. Until OPERATIONS phase exists, the handoff is informational; user manually manages SLOs/dashboards/alerts/runbooks per the handoff inventory.
