### PR Overlap Check (MANDATORY antes de `gh pr create` — workflows secuenciales)

> **Scope**: válido para un agente ejecutando a la vez. No es un mecanismo de coordinación para agentes concurrentes (race condition). El diseño multi-agente de coordinación concurrente (VI-155) fue cancelado; si aparece un caso real, abrir issue nuevo con contexto post-cancel.

Antes de crear cualquier PR, verificar que los archivos y commits del diff local no están ya en flight en otro PR abierto:

```bash
# Archivos del diff local
git diff $(git merge-base HEAD main)...HEAD --name-only

# PRs abiertos con sus archivos
gh pr list --state open --json number,title,headRefName,files \
  --jq '.[] | {number,title,branch:.headRefName,files:[.files[].path]}'

# Commits locales (para detección de duplicados)
git log $(git merge-base HEAD main)...HEAD --format="%H %s"
```

- **Overlap de archivos detectado** → STOP. Presentar PRs conflictivos al usuario. Esperar decisión: (A) rebase/consolidar, (B) separar commits independientes, (C) justificar independencia con explicación del usuario.
- **Commits duplicados (mismo SHA en PR abierto)** → STOP. Referenciar PR existente, no crear PR nuevo.
- **Sin overlap** → continuar con `gh pr create`.

Detalle completo: `_common/core-change-flow-protocol.md` step 4.5 (DIRECT/REVERT) y step 7.5 (DESIGN).

### Commit Policy

- **Commit automático**: cuando verification PASS + code review PASS (ambos gates automáticos). No esperar confirmación humana — el humano no debe ser cuello de botella.
- **Escalar al humano**: si verification falla, code review reporta CRITICAL/HIGH, o la tarea cambió de scope durante ejecución.
- **Push manual**: commit local es bajo riesgo (revertible). Push a remote es irreversible en shared branches — siempre esperar instrucción del usuario.

### Auto-commit on meaningful changes

**Do this automatically — do NOT wait for the user to ask.**

A "meaningful change" is any completed action: executing a correction prompt, generating artifacts, updating state files, applying corrections, stage transitions, etc.

### Commit message format

- Subject line: imperative, max 72 chars, describes WHAT changed
- Body (optional): bullet points for multiple changes
- Always add: `Co-Authored-By: <model-name> <noreply@anthropic.com>`
  - Replace `<model-name>` with the active model (e.g. `Claude Sonnet 4.6`, `Claude Opus 4.7`). Project-specific typed agents should use the model they are configured for.
