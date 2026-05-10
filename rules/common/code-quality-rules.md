## Regla: Principios SOLID — Bajo Acoplamiento / Alta Cohesión

**Todo diseño, implementación y code review DEBE aplicar los principios SOLID.** Esta regla no es opcional — violaciones son blocking findings en el Post-Implementation Chain.

### Principios obligatorios

| Principio | Regla concreta |
|---|---|
| **S** — Single Responsibility | Cada clase/módulo tiene UNA razón para cambiar. God objects → dividir. |
| **O** — Open/Closed | Extender via composición/herencia, no modificar código existente probado. |
| **L** — Liskov Substitution | Subtipos son sustituibles por su tipo base sin romper comportamiento. |
| **I** — Interface Segregation | Interfaces pequeñas y específicas. No forzar dependencias no usadas. |
| **D** — Dependency Inversion | Depender de abstracciones, no de implementaciones concretas. |

### Bajo acoplamiento / Alta cohesión (complementarios)

- **Bajo acoplamiento**: un cambio en módulo A no debe obligar cambios en B. Si lo hace → rediseñar boundary.
- **Alta cohesión**: todo lo que cambia junto, vive junto. Mezclar responsabilidades distintas en un módulo = señal de rediseño.

### Cuándo aplicar

1. **Code generation**: el plan de código generado por el typed implementer debe justificar la división de responsabilidades antes de generar archivos.
2. **Code review (Post-Implementation Chain)**: el typed reviewer verifica SOLID explícitamente. Violaciones S, D o acoplamiento alto = finding HIGH mínimo.
3. **Diseño (any design phase, regardless of orchestrator)**: si el diseño propuesto acopla módulos que cambian por razones distintas → STOP, rediseñar antes de continuar.
4. **Triage de issues**: si el root cause de un bug es una violación SOLID (e.g., god class, tight coupling) → el fix debe corregir el diseño, no solo parchear el síntoma.

### Anti-patrones bloqueantes

Estos patrones son incompatibles con la regla y requieren rediseño explícito:
- God class / God module (más de una responsabilidad principal)
- Acoplamiento directo entre servicios (`services/core` importando directamente de `services/bot`, etc.)
- Constructores/funciones con más de 5 dependencias sin fachada
- Lógica de negocio mezclada con lógica de infraestructura en el mismo módulo

---

## Regla: Desacoplamiento Despliegue ↔ Código de Aplicación

Cuando se detectan errores al correr `docker build`, `docker compose up`, o cualquier operación de contenedores:

✅ **SIEMPRE** buscar solución en la capa de despliegue primero:
- `Dockerfile`, `docker-compose.yml`, `docker-compose.dev.yml`
- Variables de entorno (`NODE_PATH`, `ENV`, build args)
- Volúmenes, redes, configuración de build

❌ **NUNCA** modificar código de servicios (`src/`) como solución a un problema de despliegue.

Si se agota la capa de despliegue sin solución:
→ Parar, explicar qué se intentó y por qué no funcionó, proponer el cambio de código necesario, y **esperar decisión explícita del usuario** antes de editar cualquier archivo de aplicación.
