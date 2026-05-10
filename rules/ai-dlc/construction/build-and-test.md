# Build & Compile

**Purpose**: Build all units and validate that code compiles and unit tests pass. Integration, E2E, and production readiness testing have moved to the VERIFICATION phase.

## Prerequisites
- Code Generation must be complete for all units
- All code artifacts must be generated
- Project is ready for build validation

---

## Step 1: Analyze Build Requirements

Analyze the project to determine build strategy:
- **Build tool**: What build system is used? (npm, pnpm, make, etc.)
- **Dependencies**: Are all dependencies resolved?
- **Unit tests**: Tests generated during Code Generation with TDD

---

## Step 2: Generate Build Instructions

Create `aidlc-docs/construction/build-and-test/build-instructions.md`:

```markdown
# Build Instructions

## Prerequisites
- **Build Tool**: [Tool name and version]
- **Dependencies**: [List all required dependencies]
- **Environment Variables**: [List required env vars]
- **System Requirements**: [OS, memory, disk space]

## Build Steps

### 1. Install Dependencies
\`\`\`bash
[Command to install dependencies]
\`\`\`

### 2. Configure Environment
\`\`\`bash
[Commands to set up environment]
\`\`\`

### 3. Build All Units
\`\`\`bash
[Command to build all units]
\`\`\`

### 4. Verify Build Success
- **Expected Output**: [Describe successful build output]
- **Build Artifacts**: [List generated artifacts and locations]

## Troubleshooting
[Common build issues and solutions]
```

---

## Step 3: Generate Unit Test Execution Instructions

Create `aidlc-docs/construction/build-and-test/unit-test-instructions.md`:

```markdown
# Unit Test Execution

## Run Unit Tests

### 1. Execute All Unit Tests
\`\`\`bash
[Command to run all unit tests]
\`\`\`

### 2. Review Test Results
- **Expected**: [X] tests pass, 0 failures
- **Test Coverage**: [Expected coverage percentage]
- **Test Report Location**: [Path to test reports]

### 3. Fix Failing Tests
If tests fail:
1. Review test output
2. Identify failing test cases
3. Fix code issues
4. Rerun tests until all pass
```

---

## Step 4: Execute Build and Unit Tests

**This step actually executes** (not just generates documentation):

1. Run the build command
2. Verify build succeeds
3. Run all unit tests (L1)
4. Verify all unit tests pass
5. If failures: apply `systematic-debugging` (IRON LAW), fix, retry

---

## Step 5: Generate Build Summary

Create `aidlc-docs/construction/build-and-test/build-and-test-summary.md`:

```markdown
# Build & Compile Summary

## Build Status
- **Build Tool**: [Tool name]
- **Build Status**: [Success/Failed]
- **Build Artifacts**: [List artifacts]
- **Build Time**: [Duration]

## Unit Test Results
- **Total Tests**: [X]
- **Passed**: [X]
- **Failed**: [X]
- **Coverage**: [X]%
- **Status**: [Pass/Fail]

## Overall Status
- **Build**: [Success/Failed]
- **Unit Tests**: [Pass/Fail]
- **Ready for VERIFICATION**: [Yes/No]

## Next Steps
[If all pass]: Ready to proceed to VERIFICATION phase
[If failures]: Address failing tests and rebuild
```

---

## Step 5b: Pre-flight for VERIFICATION Handoff

**MANDATORY**: Before proceeding to VERIFICATION, verify all deliverables exist:

- [ ] `aidlc-docs/construction/shared-infrastructure.md` or equivalent system-wide production topology exists
- [ ] Topology covers ALL deployed services (not just one unit)
- [ ] All container/runtime images are version-pinned (no floating tags)
- [ ] Inter-service communication map is documented

If any check fails, generate the missing artifact before proceeding to VERIFICATION. For single-unit projects, derive the topology from the unit's infrastructure-design artifacts or from actual deployment files (Dockerfiles, docker-compose).

---

## Step 6: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:
- Mark Build & Compile stage as complete (or failed)

---

## Step 7: Present Results to User

Present completion message in this structure:

1. **Completion Announcement** (mandatory):

```markdown
# 🔨 Build & Compile Complete
```

2. **AI Summary** (optional): Structured bullet-point summary:
   - Build status and artifacts
   - Unit test results (count, pass/fail, coverage)
   - DO NOT include workflow instructions
   - Keep factual and content-focused

3. **Formatted Workflow Message** (mandatory):

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the build summary at: `aidlc-docs/construction/build-and-test/build-and-test-summary.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the build configuration based on your review
> ✅ **Approve & Continue** - Approve build results and proceed to **VERIFICATION**

---
```

---

## Step 7b: Branch Completion

After user approves and all tests pass, invoke `finishing-a-development-branch` (MANDATORY per SP integration). This handles merge/PR decisions for the completed unit work.

---

## Step 8: Log Interaction

**MANDATORY**: Log the stage completion in `aidlc-docs/audit.md`:

```markdown
## Build & Compile Stage
**Timestamp**: [ISO timestamp]
**Build Status**: [Success/Failed]
**Unit Test Status**: [X pass, X fail]
**Files Generated**:
- build-instructions.md
- unit-test-instructions.md
- build-and-test-summary.md

---
```
