# Functional Design

## Purpose
**Detailed business logic design per unit**

Functional Design focuses on:
- Detailed business logic and algorithms for the unit
- Domain models with entities and relationships
- Detailed business rules, validation logic, and constraints
- Technology-agnostic design (no infrastructure concerns)

**Note**: This builds upon high-level component design from Application Design (INCEPTION phase)

## Prerequisites
- Units Generation must be complete
- Unit of work artifacts must be available
- Application Design recommended (provides high-level component structure)
- Execution plan must indicate Functional Design stage should execute

## Overview
Design detailed business logic for the unit, technology-agnostic and focused purely on business functions.

## Steps to Execute

### Step 1: Analyze Unit Context
- Read unit definition from `aidlc-docs/inception/application-design/unit-of-work.md`
- Read assigned stories from `aidlc-docs/inception/application-design/unit-of-work-story-map.md`
- Understand unit responsibilities and boundaries

### Step 2: Create Functional Design Plan
- Generate plan with checkboxes [] for functional design
- Focus on business logic, domain models, business rules
- Each step should have a checkbox []

### Step 3: Generate Context-Appropriate Questions
**DIRECTIVE**: Thoroughly analyze the unit definition and functional design artifacts to identify ALL areas where clarification would improve the functional design. Be proactive in asking questions to ensure comprehensive understanding.

**CRITICAL**: Default to asking questions when there is ANY ambiguity or missing detail that could affect functional design quality. It's better to ask too many questions than to make incorrect assumptions.

- EMBED questions using [Answer]: tag format
- Focus on ANY ambiguities, missing information, or areas needing clarification
- Generate questions wherever user input would improve functional design decisions
- **When in doubt, ask the question** - overconfidence leads to poor designs

**Question categories to consider** (evaluate ALL categories):
- **Business Logic Modeling** - Ask about core entities, workflows, data transformations, and business processes
- **Domain Model** - Ask about domain concepts, entity relationships, data structures, and business objects
- **Business Rules** - Ask about decision rules, validation logic, constraints, and business policies
- **Data Flow** - Ask about data inputs, outputs, transformations, and persistence requirements
- **Integration Points** - Ask about external system interactions, APIs, and data exchange
- **Error Handling** - Ask about error scenarios, validation failures, and exception handling
- **Business Scenarios** - Ask about edge cases, alternative flows, and complex business situations
- **Frontend Components** (if applicable) - Ask about UI component structure, user interactions, state management, and form handling

### Step 4: Store Plan
- Save as `aidlc-docs/construction/plans/{unit-name}-functional-design-plan.md`
- Include all [Answer]: tags for user input

### Step 5: Collect and Analyze Answers
- Wait for user to complete all [Answer]: tags
- **MANDATORY**: Carefully review ALL responses for vague or ambiguous answers
- **CRITICAL**: Add follow-up questions for ANY unclear responses - do not proceed with ambiguity
- Look for responses like "depends", "maybe", "not sure", "mix of", "somewhere between"
- Create clarification questions file if ANY ambiguities are detected
- **Do not proceed until ALL ambiguities are resolved**

### Step 6: Generate Functional Design Artifacts
- Create `aidlc-docs/construction/{unit-name}/functional-design/business-logic-model.md`
- Create `aidlc-docs/construction/{unit-name}/functional-design/business-rules.md`
- Create `aidlc-docs/construction/{unit-name}/functional-design/domain-entities.md`
- If unit includes frontend/UI: Create `aidlc-docs/construction/{unit-name}/functional-design/frontend-components.md`
  - (a) Component hierarchy and structure
  - (b) Props and state definitions for each component
  - (c) User interaction flows
  - (d) Form validation rules
  - (e) API integration points (which backend endpoints each component uses)

  **Sub-steps f-i — REQUIRED if `frontend-design` ran in INCEPTION** (detect by checking
  existence of `aidlc-docs/inception/frontend-design/ui-stack.md`):

  f. **Screen layout** — textual wireframe (ASCII or markdown structure).
     Each region uses layout primitives from `ui-stack.md`. Each block
     references components by name from `component-inventory.md`. New
     components require revision pass adding to inventory.

  g. **State matrix** — table with columns:
     **State | Trigger | Rendered components | Data | Copy**.
     Derived from THEN clauses of G/W/T AC of stories mapped to this unit.
     Must cover all mandatory states from `ui-stack.md` (loading, empty,
     error, success/populated, partial).
     - **Data** column: shape consumed in that state (binding name from
       data-bindings table or sub-shape, e.g., `error.message`,
       `items[0..N]`, `—` if not applicable).
     - **Copy** column: exact user-facing strings (labels, error messages,
       CTAs). Eliminates lorem ipsum and generic placeholder text.

     Example row (state=error):
     `error | fetch failed | ErrorBanner, RetryButton | error.message | "Couldn't load items. Retry."`

  h. **Data bindings** — table mapping `Component.prop → source` where
     source is a symbol in `domain-entities.md`, `components.md`, or
     `component-methods.md`. Orphan bindings → follow-up.

  i. **Snapshot test contracts** — declarative list, one entry per state
     matrix row. Each entry enumerates inputs the typed agent's TDD loop
     consumes for the three verification levels declared in
     `skills/snapshot-driven-ui/SKILL.md`:
     - For Level A (mandatory): `fixture`, `expectedTestIds`, `expectedCopy`
     - For Level B (if Storybook): `storyName`
     - For Level C (if Playwright): `route`

     Example entry:
     ```yaml
     - state: error
       fixture: fixtures/alert-feed/error.json
       expectedTestIds: [error-banner, retry-button]
       expectedCopy: ["Couldn't load items. Retry."]
       storyName: AlertFeed.Error
       route: /alerts?simulate=error
     ```

  **Schema co-ownership note**: The state matrix (g), data bindings (h), and
  snapshot contract (i) schemas are co-owned by this rule and
  `rules/common/frontend-change-discipline.md` (Workflow D). Workflow D edits
  these artifacts in-place during change-flow when UI files are touched.
  Schema changes here require updating D's rule too. The yaml example above
  is the canonical schema source — D copies this format without inventing
  fields.

  If `frontend-design` did NOT run in INCEPTION (no `ui-stack.md` present):
  sub-steps f-i are skipped and `frontend-components.md` retains its
  current shape (sub-steps a-e only).

### Step 7: Present Completion Message
- Present completion message in this structure:
     1. **Completion Announcement** (mandatory): Always start with this:

```markdown
# 🔧 Functional Design Complete - [unit-name]
```

     2. **AI Summary** (optional): Provide structured bullet-point summary of functional design
        - Format: "Functional design has created [description]:"
        - List key business logic models and entities (bullet points)
        - List business rules and validation logic defined
        - Mention domain model structure and relationships
        - DO NOT include workflow instructions ("please review", "let me know", "proceed to next phase", "before we proceed")
        - Keep factual and content-focused
     3. **Formatted Workflow Message** (mandatory): Always end with this exact format:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the functional design artifacts at: `aidlc-docs/construction/[unit-name]/functional-design/`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the functional design based on your review  
> ✅ **Continue to Next Stage** - Approve functional design and proceed to **[next-stage-name]**

---
```

### Step 8: Wait for Explicit Approval
- Do not proceed until the user explicitly approves the functional design
- Approval must be clear and unambiguous
- If user requests changes, update the design and repeat the approval process

### Step 9: Record Approval and Update Progress
- Log approval in audit.md with timestamp
- Record the user's approval response with timestamp
- Mark Functional Design stage complete in aidlc-state.md
