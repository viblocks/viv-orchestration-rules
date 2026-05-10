# Session Continuity Templates

## Welcome Back Prompt Template
When a user returns to continue work on an existing AI-DLC project, present this prompt:

```markdown
**Welcome back! I can see you have an existing AI-DLC project in progress.**

Based on your aidlc-state.md, here's your current status:
- **Project**: [project-name]
- **Current Phase**: [INCEPTION/CONSTRUCTION/VERIFICATION/DEPLOYMENT]
- **Current Stage**: [Stage Name]
- **Last Completed**: [Last completed step]
- **Next Step**: [Next step to work on]

**What would you like to work on today?**

A) Continue where you left off ([Next step description])
B) Review a previous stage ([Show available stages])

[Answer]: 
```

## MANDATORY: Session Continuity Instructions
1. **Always read aidlc-state.md first** when detecting existing project
2. **Parse current status** from the workflow file to populate the prompt
3. **MANDATORY: Load Previous Stage Artifacts** - Before resuming any stage, automatically read all relevant artifacts from previous stages:
   - **Reverse Engineering**: Read architecture.md, code-structure.md, api-documentation.md
   - **Requirements Analysis**: Read requirements.md, requirement-verification-questions.md
   - **User Stories**: Read stories.md, personas.md, story-generation-plan.md
   - **Application Design**: Read application-design artifacts (components.md, component-methods.md, services.md)
   - **Design (Units)**: Read unit-of-work.md, unit-of-work-dependency.md, unit-of-work-story-map.md
   - **Per-Unit Design**: Read functional-design.md, nfr-requirements.md, nfr-design.md, infrastructure-design.md
   - **Code Stages**: Read all code files, plans, AND all previous artifacts
   - **VERIFICATION Stages**: Read all verification artifacts (test-strategy.md, test-environment-design.md, production-infrastructure-blueprint.md, environment-setup-report.md, test-diagnosis.md, test-implementation-report.md, e2e-validation-report.md, ci-pipeline-report.md, production-readiness-verdict.md) + question answer files in plans/ + all Construction and Inception artifacts
   - **DEPLOYMENT Stages**: Read `aidlc-docs/deployment/deployment-strategy.md`, `aidlc-docs/deployment/cd-pipeline-diagnosis.md` (if exists), `aidlc-docs/deployment/cd-pipeline-design.md`, `aidlc-docs/deployment/cd-pipeline-impl-report.md`, `aidlc-docs/deployment/deployment-execution-report.md`, `aidlc-docs/deployment/post-deploy-validation-report.md`, all plan files in `aidlc-docs/deployment/plans/`, plus all previous phase artifacts (INCEPTION + CONSTRUCTION + VERIFICATION).
4. **Smart Context Loading by Stage**:
   - **Early Stages (Workspace Detection, Reverse Engineering)**: Load workspace analysis
   - **Requirements/Stories**: Load reverse engineering + requirements artifacts
   - **Design Stages**: Load requirements + stories + architecture + design artifacts
   - **Code Stages**: Load ALL artifacts + existing code files
   - **VERIFICATION Stages**: Load all Construction artifacts + all previous VERIFICATION stage artifacts + question answer files in plans/
   - **DEPLOYMENT Stages**: Load all VERIFICATION artifacts + all Construction and Inception artifacts + all previous DEPLOYMENT stage artifacts + plan files in aidlc-docs/deployment/plans/
5. **Adapt options** based on architectural choice and current phase
6. **Show specific next steps** rather than generic descriptions
7. **Domain skill re-invocation**: Domain skill invocation is session-scoped (IRON LAW). On resume, invoke the domain skill for the active unit's domain BEFORE any code edit or Agent dispatch — even if it was invoked in the prior session. One invocation covers the new session.
8. **Log the continuity prompt** in audit.md with timestamp
9. **Context Summary**: After loading artifacts, provide brief summary of what was loaded for user awareness
10. **Asking questions**: ALWAYS ask clarification or user feedback questions by placing them in .md files. DO NOT place the multiple-choice questions in-line in the chat session.

## Error Handling
If artifacts are missing or corrupted during session resumption, see [error-handling.md](error-handling.md) for guidance on recovery procedures. 