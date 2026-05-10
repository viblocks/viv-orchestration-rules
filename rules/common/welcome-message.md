# AI-DLC Welcome Message

**Purpose**: This file contains the user-facing welcome message that should be displayed ONCE at the start of any AI-DLC workflow.

---

# 👋 Welcome to AI-DLC (AI-Driven Development Life Cycle)! 👋

I'll guide you through an adaptive software development workflow that intelligently tailors itself to your specific needs.

## What is AI-DLC?

AI-DLC is a structured yet flexible software development process that adapts to your project's needs. Think of it as having an experienced software architect who:

- **Analyzes your requirements** and asks clarifying questions when needed
- **Plans the optimal approach** based on complexity and risk
- **Skips unnecessary steps** for simple changes while providing comprehensive coverage for complex projects
- **Documents everything** so you have a complete record of decisions and rationale
- **Guides you through each phase** with clear checkpoints and approval gates

## The Four-Phase Lifecycle

```
                         User Request
                              |
                              v
        +---------------------------------------+
        |     INCEPTION PHASE                   |
        |     Planning & Application Design     |
        +---------------------------------------+
        | * Workspace Detection (ALWAYS)        |
        | * Reverse Engineering (COND)          |
        | * Requirements Analysis (ALWAYS)      |
        | * User Stories (CONDITIONAL)          |
        | * Workflow Planning (ALWAYS)          |
        | * Application Design (CONDITIONAL)    |
        | * Units Generation (CONDITIONAL)      |
        +---------------------------------------+
                              |
                              v
        +---------------------------------------+
        |     CONSTRUCTION PHASE                |
        |     Design, Implementation & Test     |
        +---------------------------------------+
        | * Per-Unit Loop (for each unit):      |
        |   - Functional Design (COND)          |
        |   - NFR Requirements Assess (COND)    |
        |   - NFR Design (COND)                 |
        |   - Infrastructure Design (COND)      |
        |   - Code Generation (ALWAYS)          |
        | * Build and Test (ALWAYS)             |
        +---------------------------------------+
                              |
                              v
        +---------------------------------------+
        |     VERIFICATION PHASE                |
        |     Validation & Production Readiness |
        +---------------------------------------+
        | * Test Strategy Design (ALWAYS)       |
        | * Test Environment Design (ALWAYS)    |
        | * Test Environment Setup (ALWAYS)     |
        | * Test Diagnosis (ALWAYS)             |
        | * Test Implementation (COND)          |
        | * E2E Validation (ALWAYS)             |
        | * CI Pipeline Implementation (ALWAYS) |
        | * Production Readiness Gate (ALWAYS)  |
        +---------------------------------------+
                              |
                              v
        +---------------------------------------+
        |     DEPLOYMENT PHASE                  |
        |     Ship to Production Safely         |
        +---------------------------------------+
        | * Deployment Strategy Design (ALWAYS) |
        | * CD Pipeline Diagnosis (COND)        |
        | * CD Pipeline Design (ALWAYS)         |
        | * CD Pipeline Implementation (ALWAYS) |
        | * Deployment Execution (ALWAYS)       |
        | * Post-Deploy Validation (ALWAYS)     |
        | * Rollback Validation + Sign-off      |
        |   Gate (ALWAYS)                       |
        +---------------------------------------+
                              |
                              v
                          Complete
```

### Phase Breakdown:

**INCEPTION PHASE** - *Planning & Application Design*
- **Purpose**: Determines WHAT to build and WHY
- **Activities**: Understanding requirements, analyzing existing code (if any), planning the approach
- **Output**: Clear requirements, execution plan, decisions on the number of units of work for parallel development
- **Your Role**: Answer questions, review plans, approve direction

**CONSTRUCTION PHASE** - *Detailed Design, Implementation & Test*
- **Purpose**: Determines HOW to build it
- **Activities**: Detailed design (when needed), code generation, comprehensive testing
- **Output**: Working code, tests, build instructions
- **Your Role**: Review designs, approve implementation plans, validate results

**VERIFICATION PHASE** - *Validation & Production Readiness*
- **Purpose**: Validate the system works as built and is ready to deploy
- **Activities**: Test strategy design, environment setup, test diagnosis and implementation, E2E validation, CI pipeline implementation, production readiness evaluation
- **Output**: Complete test suite, reproducible test environment, CI pipeline, production readiness verdict
- **Your Role**: Approve test strategy, review environment design, accept or reject production readiness verdict

**DEPLOYMENT PHASE** - *Ship to Production Safely and Reversibly*
- **Purpose**: Establish production deployment capability
- **Focus**: How do we SHIP it to production safely and reversibly?
- **Activities**: Deployment strategy design, CD pipeline diagnosis (brownfield), CD pipeline design and implementation, deployment execution, post-deploy validation, rollback validation and sign-off
- **Output**: Working CD pipeline, deployment playbook, post-deploy validation evidence, rollback procedure validated
- **Your Role**: Approve deployment strategy, review pipeline design, sign off on production readiness after deployment

## Key Principles:

- ⚡ **Fully Adaptive**: Each stage independently evaluated based on your needs
- 🎯 **Efficient**: Simple changes execute only essential stages
- 📋 **Comprehensive**: Complex changes get full treatment with all safeguards
- 🔍 **Transparent**: You see and approve the execution plan before work begins
- 📝 **Documented**: Complete audit trail of all decisions and changes
- 🎛️ **User Control**: You can request stages be included or excluded

## What Happens Next:

1. **I'll analyze your workspace** to understand if this is a new or existing project
2. **I'll gather requirements** and ask clarifying questions if needed
3. **I'll create an execution plan** showing which stages I propose to run and why
4. **You'll review and approve** the plan (or request changes)
5. **We'll execute the plan** with checkpoints at each major stage
6. **You'll get working code** with complete documentation and tests

The AI-DLC process adapts to:
- 📋 Your intent clarity and complexity
- 🔍 Existing codebase state
- 🎯 Scope and impact of changes
- ⚡ Risk and quality requirements

Let's begin!
