# Workspace Detection

**Purpose**: Determine workspace state and check for existing AI-DLC projects

## Step 1: Check for Existing AI-DLC Project

Check if `aidlc-docs/aidlc-state.md` exists:
- **If exists**: Resume from last phase (load context from previous phases)
- **If not exists**: Continue with new project assessment

## Step 2: Scan Workspace for Existing Code

**Determine if workspace has existing code:**
- Scan workspace for source code files (.java, .py, .js, .ts, .jsx, .tsx, .kt, .kts, .scala, .groovy, .go, .rs, .rb, .php, .c, .h, .cpp, .hpp, .cc, .cs, .fs, etc.)
- Check for build files (pom.xml, package.json, build.gradle, etc.)
- Look for project structure indicators
- Identify workspace root directory (NOT aidlc-docs/)

**Record findings:**
```markdown
## Workspace State
- **Existing Code**: [Yes/No]
- **Programming Languages**: [List if found]
- **Build System**: [Maven/Gradle/npm/etc. if found]
- **Project Structure**: [Monolith/Microservices/Library/Empty]
- **Workspace Root**: [Absolute path]
```

## Step 3: Determine Next Phase

**IF workspace is empty (no existing code)**:
- Set flag: `brownfield = false`
- Next phase: Requirements Analysis

**IF workspace has existing code**:
- Set flag: `brownfield = true`
- Check for existing reverse engineering artifacts in `aidlc-docs/inception/reverse-engineering/`
- **IF reverse engineering artifacts exist**:
    - Check if artifacts are stale (compare artifact timestamps against codebase's last significant modification)
    - **IF artifacts are current**: Load them, proceed to Behavioral Derivation check
    - **IF artifacts are stale**: Next phase is Reverse Engineering (rerun to refresh artifacts), followed by Behavioral Derivation
    - **IF user explicitly requests rerun**: Next phase is Reverse Engineering regardless of staleness, followed by Behavioral Derivation
- **IF no reverse engineering artifacts**: Next phase is Reverse Engineering, followed by Behavioral Derivation

**Behavioral Derivation check** (always evaluated post-A on brownfield):
- Check for existing artifacts in `aidlc-docs/inception/behavioral-derivation/`
- **IF artifacts exist and are not stale**: Load them, skip B
- **IF artifacts exist but are stale**: Re-run B silently (per D10 in the spec — auto-staleness re-derives without prompt)
- **IF no artifacts**: Next phase is Behavioral Derivation
- After B completes (or is skipped): Next phase is Requirements Analysis (or Frontend Design if UI detected and B has run)

## Step 4: Create Initial State File

Create `aidlc-docs/aidlc-state.md`:

```markdown
# AI-DLC State Tracking

## Project Information
- **Project Type**: [Greenfield/Brownfield]
- **Start Date**: [ISO timestamp]
- **Current Stage**: INCEPTION - Workspace Detection

## Workspace State
- **Existing Code**: [Yes/No]
- **Reverse Engineering Needed**: [Yes/No]
- **Workspace Root**: [Absolute path]

## Code Location Rules
- **Application Code**: Workspace root (NEVER in aidlc-docs/)
- **Documentation**: aidlc-docs/ only
- **Structure patterns**: See code-generation.md Critical Rules

## Stage Progress
[Will be populated as workflow progresses]
```

## Step 5: Present Completion Message

**For Brownfield Projects:**
```markdown
# 🔍 Workspace Detection Complete

Workspace analysis findings:
• **Project Type**: Brownfield project
• [AI-generated summary of workspace findings in bullet points]
• **Next Step**: Proceeding to **Reverse Engineering** to analyze existing codebase, then **Behavioral Derivation** to derive baseline behavioral spec...
```

**For Greenfield Projects:**
```markdown
# 🔍 Workspace Detection Complete

Workspace analysis findings:
• **Project Type**: Greenfield project
• **Next Step**: Proceeding to **Requirements Analysis**...
```

## Step 6: Plugin Auto-Bootstrap of `aidlc-docs/` (CONDITIONAL)

**Trigger condition** — execute ONLY if:
1. The `aidlc-orchestrator` plugin is detected: `rules/` symlink exists in the workspace root (created by the Setup hook when the plugin is installed)

**Skip condition** — skip if `rules/` does not exist.

**Purpose**: ensure the workflow can progress regardless of brownfield/greenfield. Applies to BOTH project types — brownfield repos benefit from the same scaffolding once Reverse Engineering is about to write artifacts under `aidlc-docs/inception/`.

> **Note on `CLAUDE.md`**: auto-creation of `CLAUDE.md` is handled by the Setup hook (`hooks/setup-install.sh`) at SessionStart, BEFORE Workspace Detection runs. By the time this rule executes, `CLAUDE.md` already exists (from the hook) or was already present in the consumer's repo. This rule no longer creates `CLAUDE.md` directly — the hook owns that responsibility because it fires earlier and unifies greenfield/brownfield paths.

### 6.1 Create `aidlc-docs/` structure

Create these directories and seed files if they do not already exist:

```
aidlc-docs/
  inception/
    plans/
    application-design/
  audit.md          ← seed with "# Audit Log\n"
```

This step is idempotent: existing files are preserved, missing ones are created.

### 6.2 Confirm to user

Inform the user that the auto-bootstrap ran:

```markdown
Plugin detected (aidlc-orchestrator). Auto-bootstrapped:
• aidlc-docs/ structure created
Proceeding to next phase...
```

(`CLAUDE.md` confirmation is emitted by the Setup hook earlier, in its `systemMessage`.)

## Step 7: Automatically Proceed

- **No user approval required** - this is informational only
- Automatically proceed to next phase:
  - **Brownfield**: Reverse Engineering (if needed), then Behavioral Derivation, then Requirements Analysis
  - **Greenfield**: Requirements Analysis
