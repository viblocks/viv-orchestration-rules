
> **Architecture note (per [ADR-RD-004](https://github.com/viblocks/viv-typed-agents/blob/main/architecture/decisions/ADR-RD-004-classifier-folded.md)):** the legacy `.claude/context/artifact-classifier.json` no longer exists. Class A scope is derived from `routing-table.json` routes where `enforced: true`. Where this document references the classifier as a separate file, read it as `routing-table.json` with the `enforced` field.

## IRON LAW: Typed Agent Dispatch

**Toda creación, edición, corrección o mejora de código de aplicación se ejecuta EXCLUSIVAMENTE a través del typed agent correcto. Sin excepciones.**

La estrategia completa está en `_common/typed-agent-mechanism.md`.

### Routing Table

**Fuente de verdad unica**: `.claude/routing/routing-table.json` — contiene domain, paths, implementer, reviewer para cada dominio.

**Classifier PF21** (v3): `.claude/context/artifact-classifier.json` define la taxonomía binaria Clase A (enforcement aplica) vs Clase B (free edit). Todo flujo (F1-F4) respeta esta taxonomía — no hay carveouts.

El **spec reviewer** siempre es `general-purpose` — solo lee código, no necesita domain knowledge.

**Unknown service rule**: Cualquier `services/*` o `packages/*` que NO tenga fila explícita en la tabla → **STOP**. No asumir dominio. Ejecutar el Routing Table Population Protocol (ver `_common/routing-table-population-protocol.md`): (1) clasificar el stack desde artifacts de diseño (NFR/Infra) o RE (component-inventory/technology-stack), (2) agregar entrada a `.claude/routing/routing-table.json`, (3) si no existe typed agent para ese stack → escalar al usuario con opciones (crear agent, usar general-purpose, o diferir). Solo entonces despachar al typed agent correcto.

### Reglas de dispatch

1. **Detecta el dominio** por el path del código a modificar, no por keywords del problema. Si el path no tiene fila explícita en la routing table, aplica el **unknown service rule** — STOP y clasificar antes de despachar.
2. **Usa el typed implementer** del dominio — nunca `general-purpose` para código en `services/` o `packages/`.
3. **Usa el typed reviewer** del dominio — nunca `superpowers:code-reviewer` para `services/` o `packages/`.
4. **TDD embebido** — no invocar `superpowers:test-driven-development` por separado; el IRON LAW del typed agent lo incluye.
5. **Cross-domain** — dividir en el plan por dominio; `packages/shared` siempre es Task 1 (es el contrato).
6. **Blacklist domain** — antes de cualquier recomendación, fix o issue que cruce boundaries de servicio en el pipeline (pollers → enrichment → guardian → consumers), invocar `Skill(blacklist-monitoring)`. Sin excepciones. Aplica a: análisis de código, code review, propuesta de mejoras, apertura de issues.

### Integración con subagent-driven-development

Al usar `superpowers:subagent-driven-development`, los typed agents reemplazan a los agentes genéricos:

```
implementer    → backend-crypto-implementer | frontend-crypto-implementer
spec reviewer  → general-purpose (sin cambio)
code reviewer  → backend-crypto-reviewer   | frontend-crypto-reviewer
```

### Extensibilidad

Para agregar un nuevo dominio: crear los agentes en `.claude/agents/`, crear el skill en `.claude/skills/[domain]/`, agregar la entrada a `.claude/routing/routing-table.json`, y actualizar el hook en `settings.json`. Ver `_common/typed-agent-mechanism.md` sección "Extensibility — Adding a New Domain".
