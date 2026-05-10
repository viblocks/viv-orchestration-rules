## Directory Structure

```text
<WORKSPACE-ROOT>/                   # ⚠️ APPLICATION CODE HERE
├── [project-specific structure]    # Varies by project (see code-generation.md)
│
├── aidlc-docs/                     # 📄 DOCUMENTATION ONLY
│   ├── inception/                  # 🔵 INCEPTION PHASE
│   │   ├── plans/
│   │   ├── reverse-engineering/    # Brownfield only
│   │   ├── requirements/
│   │   ├── user-stories/
│   │   └── application-design/
│   ├── construction/               # 🟢 CONSTRUCTION PHASE
│   │   ├── plans/
│   │   ├── {unit-name}/
│   │   │   ├── functional-design/
│   │   │   ├── nfr-requirements/
│   │   │   ├── nfr-design/
│   │   │   ├── infrastructure-design/
│   │   │   └── code/               # Markdown summaries only
│   │   └── build-and-test/
│   ├── verification/               # 🔬 VERIFICATION PHASE
│   │   ├── test-strategy.md
│   │   ├── test-environment-design.md
│   │   ├── production-infrastructure-blueprint.md
│   │   ├── environment-setup-report.md
│   │   ├── test-diagnosis.md
│   │   ├── test-implementation-report.md
│   │   ├── e2e-validation-report.md
│   │   ├── ci-pipeline-report.md
│   │   └── production-readiness-verdict.md
│   ├── deployment/                 # 🟡 DEPLOYMENT PHASE
│   │   ├── deployment-strategy.md
│   │   ├── cd-pipeline-diagnosis.md     # conditional — brownfield only
│   │   ├── cd-pipeline-design.md
│   │   ├── cd-pipeline-impl-report.md
│   │   ├── deployment-execution-report.md
│   │   ├── post-deploy-validation-report.md
│   │   └── rollback-validation-signoff.md
│   ├── aidlc-state.md
│   └── audit.md
```

**CRITICAL RULE**:
- Application code: Workspace root (NEVER in aidlc-docs/)
- Documentation: aidlc-docs/ only
- Project structure: See code-generation.md for patterns by project type
