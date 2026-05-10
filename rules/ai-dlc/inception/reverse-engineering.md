# Reverse Engineering

**Purpose**: Analyze existing codebase and generate comprehensive design artifacts

**Execute when**: Brownfield project detected (existing code found in workspace)

**Skip when**: Greenfield project (no existing code)

**Rerun behavior**: Rerun is controlled by workspace-detection.md. If existing reverse engineering artifacts are found and are still current, they are loaded and reverse engineering is skipped. If artifacts are stale (older than the codebase's last significant modification) or the user explicitly requests a rerun, reverse engineering executes again to ensure artifacts reflect current code state

## Step 1: Multi-Package Discovery

### 1.1 Scan Workspace
- All packages (not just mentioned ones)
- Package relationships via config files
- Package types: Application, CDK/Infrastructure, Models, Clients, Tests

### 1.2 Understand the Business Context
- The core business that the system is implementing overall
- The business overview of every package
- List of Business Transactions that are implemented in the system

### 1.3 Infrastructure Discovery
- CDK packages (package.json with CDK dependencies)
- Terraform (.tf files)
- CloudFormation (.yaml/.json templates)
- Deployment scripts

### 1.4 Build System Discovery
- Build systems: Brazil, Maven, Gradle, npm
- Config files for build-system declarations
- Build dependencies between packages

### 1.5 Service Architecture Discovery
- Lambda functions (handlers, triggers)
- Container services (Docker/ECS configs)
- API definitions (Smithy models, OpenAPI specs)
- Data stores (DynamoDB, S3, etc.)

### 1.5b UI Stack Discovery (only if frontend artifacts detected)

Detection signals:
- package.json has react/vue/svelte/solid/angular dependency
- Existence of: tailwind.config, postcss.config, theme files,
  src/components/, app/ (Next.js), pages/ (Next.js/Nuxt)
- Build output indicates client bundle (vite, webpack, rspack with browser target)

If detected, produce in `aidlc-docs/inception/reverse-engineering/`:

- `ui-stack-detected.md`
    Framework version, styling solution detected, component library used,
    list of files in `src/components/` (or equivalent), tailwind config /
    CSS variables file path / theme file path, routing system detected.

- `ui-component-scan.md`
    Best-effort scan of existing components (name, file, exports).
    Best-effort variant detection (cva calls, conditional classnames),
    flagged as "needs human review".

- `ui-tokens-detected.md`
    Manifest pointing at config file as source-of-truth.
    Inventory of token categories present (colors, spacing, typography, etc.)
    without duplicating values.

If not detected: do NOT produce these artifacts. Subsequent UI stages
(`frontend-design`) will skip silently.

These artifacts are inputs to `frontend-design` stage in INCEPTION (see
`rules/inception/frontend-design.md`). They are not directly approved here —
their consumer in `frontend-design` Step 2 invites the user to confirm via
`[Answer]` tags.

### 1.6 Code Quality Analysis
- Programming languages and frameworks
- Test coverage indicators
- Linting configurations
- CI/CD pipelines

### 1.7 Tooling Enforcement Audit (REQUIRED)

Reconcile **declared quality mechanisms** against **enforcers actually running them**. A declared mechanism without an enforcer is a *zombie script*: visible in source (e.g., `package.json` `scripts.lint`), but no CI workflow, pre-commit hook, or other automated process invokes it. Zombies are **worse than absence** — the visible artifact lies about the actual enforcement state, so reviewers stop scrutinizing the underlying invariant and drift accumulates silently.

For every declared quality mechanism (workspace `package.json` scripts, lint/format/typecheck configs, pre-commit hooks, CI workflow files, scripts under `scripts/`, platform equivalents like `pyproject.toml` / `Cargo.toml` / `Makefile` targets), classify each as exactly one of:

- **ENFORCED** — a specific identified enforcer runs it on every relevant change. Cite the enforcer (workflow file + job name, hook name, etc.).
- **OPT-OUT** — declared, intentionally not enforced, with documented rationale. Requires the OPT-OUT template (below).
- **ZOMBIE** — declared, no enforcer identified. **HIGH severity finding** — every zombie must be resolved before reverse engineering completes (wire an enforcer, convert to OPT-OUT, or remove the declaration).

The ZOMBIE severity is **HIGH, not Low or Medium**: the false sense of security a zombie creates is structurally worse than absence and not a tradeable risk. A zombie cannot be admitted as a Known Limitation.

#### OPT-OUT template

Each OPT-OUT entry MUST include three sections (omitting any invalidates the OPT-OUT and the mechanism reverts to ZOMBIE classification):

```markdown
### OPT-OUT — `<mechanism>`

**What's declared**: <package.json script / config file / hook etc., with literal command or path>

**Why it's not enforced**: <concrete rationale — not "we don't have time"; e.g., "developer-only convenience script for local profiling, never intended for CI">

**What mitigates the risk**: <compensating control — e.g., "the underlying invariant is enforced by <other ENFORCED mechanism>", or "no invariant is being declared, the script is a developer ergonomic">
```

#### Worked example (from `viblocks-ai`)

The audit applied to viblocks-ai before AI-DLC adoption would have produced (illustrative):

```markdown
### ZOMBIE — `services/ui/package.json` `scripts.lint`

**What's declared**: `"lint": "tsc --noEmit"` in `services/ui/package.json:18`

**Enforcer search**: grep across `.github/workflows/*.yml`, `.husky/`, `pre-commit-config.yaml`, `Makefile`, `package.json` of root and other workspaces.

**Result**: No workflow invokes `pnpm --filter @viblocks/ui run lint`. No pre-commit hook references it. `Makefile` has no target that calls it.

**Severity**: HIGH. The script is declared as the type-check gate but nothing runs it. Latent for ~30 days; 15 type errors silently accumulated (reference: `viblocks-ai#460`).

**Resolution required before RE complete**: Wire `pnpm --filter @viblocks/ui run lint` into the PR-merge CI gate, OR convert to OPT-OUT with the template above, OR remove the script.
```

#### Output

Append a `## Tooling Enforcement Audit` section to `aidlc-docs/inception/reverse-engineering/code-quality-assessment.md` (created in Step 9) listing every ENFORCED entry (table form: mechanism + enforcer), every OPT-OUT entry (using the template), and every ZOMBIE entry (using the format above). The Step 9 artifact MUST NOT be marked complete with any ZOMBIE entries unresolved.

## Step 2: Generate Business Overview Documentation

Create `aidlc-docs/inception/reverse-engineering/business-overview.md`:

```markdown
# Business Overview

## Business Context Diagram
[Mermaid diagram showing the Business Context]

## Business Description
- **Business Description**: [Overall Business description of what the system does]
- **Business Transactions**: [List of Business Transactions that the system implements and their descriptions]
- **Business Dictionary**: [Business dictionary terms that the system follows and their meaning]

## Component Level Business Descriptions
### [Package/Component Name]
- **Purpose**: [What it does from the business perspective]
- **Responsibilities**: [Key responsibilities]
```

## Step 3: Generate Architecture Documentation

Create `aidlc-docs/inception/reverse-engineering/architecture.md`:

```markdown
# System Architecture

## System Overview
[High-level description of the system]

## Architecture Diagram
[Mermaid diagram showing all packages, services, data stores, relationships]

## Component Descriptions
### [Package/Component Name]
- **Purpose**: [What it does]
- **Responsibilities**: [Key responsibilities]
- **Dependencies**: [What it depends on]
- **Type**: [Application/Infrastructure/Model/Client/Test]

## Data Flow
[Mermaid sequence diagram of key workflows]

## Integration Points
- **External APIs**: [List with purposes]
- **Databases**: [List with purposes]
- **Third-party Services**: [List with purposes]

## Infrastructure Components
- **CDK Stacks**: [List with purposes]
- **Deployment Model**: [Description]
- **Networking**: [VPC, subnets, security groups]
```

## Step 4: Generate Code Structure Documentation

Create `aidlc-docs/inception/reverse-engineering/code-structure.md`:

```markdown
# Code Structure

## Build System
- **Type**: [Maven/Gradle/npm/Brazil]
- **Configuration**: [Key build files and settings]

## Key Classes/Modules
[Mermaid class diagram or module hierarchy]

### Existing Files Inventory
[List all source files with their purposes - these are candidates for modification in brownfield projects]

**Example format**:
- `[path/to/file]` - [Purpose/responsibility]

## Design Patterns
### [Pattern Name]
- **Location**: [Where used]
- **Purpose**: [Why used]
- **Implementation**: [How implemented]

## Critical Dependencies
### [Dependency Name]
- **Version**: [Version number]
- **Usage**: [How and where used]
- **Purpose**: [Why needed]
```

## Step 5: Generate API Documentation

Create `aidlc-docs/inception/reverse-engineering/api-documentation.md`:

```markdown
# API Documentation

## REST APIs
### [Endpoint Name]
- **Method**: [GET/POST/PUT/DELETE]
- **Path**: [/api/path]
- **Purpose**: [What it does]
- **Request**: [Request format]
- **Response**: [Response format]

## Internal APIs
### [Interface/Class Name]
- **Methods**: [List with signatures]
- **Parameters**: [Parameter descriptions]
- **Return Types**: [Return type descriptions]

## Data Models
### [Model Name]
- **Fields**: [Field descriptions]
- **Relationships**: [Related models]
- **Validation**: [Validation rules]
```

## Step 6: Generate Component Inventory

Create `aidlc-docs/inception/reverse-engineering/component-inventory.md`:

```markdown
# Component Inventory

## Application Packages
- [Package name] - [Purpose]

## Infrastructure Packages
- [Package name] - [CDK/Terraform] - [Purpose]

## Shared Packages
- [Package name] - [Models/Utilities/Clients] - [Purpose]

## Test Packages
- [Package name] - [Integration/Load/Unit] - [Purpose]

## Total Count
- **Total Packages**: [Number]
- **Application**: [Number]
- **Infrastructure**: [Number]
- **Shared**: [Number]
- **Test**: [Number]
```

## Step 7: Generate Technology Stack Documentation

Create `aidlc-docs/inception/reverse-engineering/technology-stack.md`:

```markdown
# Technology Stack

## Programming Languages
- [Language] - [Version] - [Usage]

## Frameworks
- [Framework] - [Version] - [Purpose]

## Infrastructure
- [Service] - [Purpose]

## Build Tools
- [Tool] - [Version] - [Purpose]

## Testing Tools
- [Tool] - [Version] - [Purpose]
```

## Step 8: Generate Dependencies Documentation

Create `aidlc-docs/inception/reverse-engineering/dependencies.md`:

```markdown
# Dependencies

## Internal Dependencies
[Mermaid diagram showing package dependencies]

### [Package A] depends on [Package B]
- **Type**: [Compile/Runtime/Test]
- **Reason**: [Why dependency exists]

## External Dependencies
### [Dependency Name]
- **Version**: [Version]
- **Purpose**: [Why used]
- **License**: [License type]
```

## Step 9: Generate Code Quality Assessment

Create `aidlc-docs/inception/reverse-engineering/code-quality-assessment.md`:

```markdown
# Code Quality Assessment

## Test Coverage
- **Overall**: [Percentage or Good/Fair/Poor/None]
- **Unit Tests**: [Status]
- **Integration Tests**: [Status]

## Code Quality Indicators
- **Linting**: [Configured/Not configured]
- **Code Style**: [Consistent/Inconsistent]
- **Documentation**: [Good/Fair/Poor]

## Technical Debt
- [Issue description and location]

## Patterns and Anti-patterns
- **Good Patterns**: [List]
- **Anti-patterns**: [List with locations]

## Tooling Enforcement Audit
Populated by Step 1.7. MUST be empty of ZOMBIE entries before this artifact is marked complete.

### ENFORCED
| Mechanism | Enforcer |
|---|---|
| [package.json script / hook / config / scripts/ entry] | [workflow + job, hook name, or other concrete enforcer] |

### OPT-OUT
[One entry per OPT-OUT, using the template from Step 1.7. Each entry must have all three sections (What's declared / Why it's not enforced / What mitigates the risk).]

### ZOMBIE
[Must be empty at completion. If non-empty, resolve each entry per Step 1.7 (wire an enforcer, convert to OPT-OUT, or remove the declaration) before proceeding.]
```

## Step 10: Create Timestamp File

Create `aidlc-docs/inception/reverse-engineering/reverse-engineering-timestamp.md`:

```markdown
# Reverse Engineering Metadata

**Analysis Date**: [ISO timestamp]
**Analyzer**: AI-DLC
**Workspace**: [Workspace path]
**Total Files Analyzed**: [Number]

## Artifacts Generated
- [x] architecture.md
- [x] code-structure.md
- [x] api-documentation.md
- [x] component-inventory.md
- [x] technology-stack.md
- [x] dependencies.md
- [x] code-quality-assessment.md
```

## Step 11: Update State Tracking

Update `aidlc-docs/aidlc-state.md`:

```markdown
## Reverse Engineering Status
- [x] Reverse Engineering - Completed on [timestamp]
- **Artifacts Location**: aidlc-docs/inception/reverse-engineering/
```

## Step 12: Present Completion Message to User

```markdown
# 🔍 Reverse Engineering Complete

[AI-generated summary of key findings from analysis in the form of bullet points]

> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the reverse engineering artifacts at: `aidlc-docs/inception/reverse-engineering/`

> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the reverse engineering analysis if required
> ✅ **Approve & Continue** - Approve analysis and proceed to **Requirements Analysis**
```

## Step 13: Wait for User Approval

- **MANDATORY**: Do not proceed until user explicitly approves
- **MANDATORY**: Log user's response in audit.md with complete raw input

## Handoff to Behavioral Derivation

After this stage completes on a brownfield project, the next stage is **Behavioral Specification Derivation** (`behavioral-derivation.md`). It reads the artifacts produced here (`architecture.md`, `technology-stack.md`, `component-inventory.md`, `ui-stack-detected.md` if present) plus existing code, tests, and docs to derive five behavioral artifacts: `existing-stories.md`, `existing-personas.md`, `existing-screens-map.md`, `existing-flows.md`, and `existing-domain-baseline.md`.

Behavioral Derivation runs unconditionally on brownfield (not gated by UI detection). For backend-only projects, the stories and domain baseline artifacts inform the change-flow protocol even when no UI exists.

The handoff is artifact-based (Dependency Inversion): this stage produces files at known paths under `aidlc-docs/inception/reverse-engineering/`; Behavioral Derivation reads those paths. Neither stage depends on the other's rule internals.
