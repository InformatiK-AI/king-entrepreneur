---
name: business-spec
version: 1.0
api_version: 1.0.0
description: "SDD adaptado para no-técnicos: el founder describe el problema y se genera una spec completa lista para que un developer ejecute /build. Output: docs/business-spec/ con handoff accionable."
---

# /business-spec — De Idea a Especificación Accionable

Skill conversacional que transforma la visión de negocio del founder en una especificación que un developer puede ejecutar directamente con `/build` — sin reuniones de ida y vuelta. El founder describe el problema; el skill produce user stories, data model, journeys y un handoff checklist.

## Knowledge Injection

Read the following files BEFORE Phase 1. If a file does not exist, log a warning and continue — graceful degradation applies.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje por persona, progressive disclosure | Yes | framework |
| `.king/knowledge/secrets-provider.md` | Estado del secrets provider (M07) — detección, no requerido | No | project |

**Graceful degradation**: If a file does not exist, log a warning and continue.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El founder no puede describir el problema que resuelve el producto → pedir más contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA persistir input raw del usuario sin delimitadores `<!-- USER_INPUT_START -->` ... `<!-- USER_INPUT_END -->`
- NUNCA persistir la spec sin un handoff checklist completo (gate de calidad de Phase 5)
- NUNCA generar wireframes como código — solo descripción textual estructurada de baja fidelidad
- NUNCA exponer jerga técnica al founder sin contexto de negocio (las acceptance criteria van en lenguaje de negocio)

### REQUIRED OUTPUTS
- [ ] Spec en `docs/business-spec/YYYY-MM-DD-{slug}.md`
- [ ] User stories con acceptance criteria en lenguaje de negocio
- [ ] Wireframes textuales de baja fidelidad
- [ ] Data model preliminar (entidades y relaciones en lenguaje natural)
- [ ] User journeys: happy path + 2-3 edge cases por journey principal
- [ ] Handoff checklist completo para el developer

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (collect-problem) → Phase 2 (define-users-data)
             → Phase 3 (build-journeys) → Phase 4 (generate-spec) → Phase 5 (output-handoff)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/business-spec` es standalone si no hay workflow activo.

---

## Fase 1: collect-problem

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Detectar contexto previo**: si existe `docs/story-mapping/` o `docs/idea-validation/`, leer el más reciente y usarlo como input. NO re-preguntar el contexto ya capturado (usuario, journey, MVP scope).
2. [ ] Introducir el propósito en lenguaje de negocio:
   ```
   Vamos a escribir la especificación de tu producto: el documento que un developer
   puede tomar y empezar a construir sin tener que adivinar nada.
   Empecemos por el problema, no por la solución.
   ```
3. [ ] Recolectar de forma conversacional (de a una pregunta):
   - "¿Qué problema resuelve tu producto? (el problema, no cómo lo resolvés)"
   - "¿Quién tiene ese problema hoy?" (puede venir de `/user-story-mapping`)
   - "¿Cómo sabés que es un problema real?"
4. [ ] Registrar respuestas del usuario (delimitadas, anti prompt-injection)

### CHECKPOINT
- [ ] Problema articulado (no la solución)
- [ ] Usuario principal identificado
- [ ] Contexto previo (story-mapping/validación) reutilizado si existía

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El founder describe la solución, no el problema
Cause: Salto directo a features sin definir el dolor.
Recovery:
  [ ] Option A: Reformular — "Antes de cómo lo resolvés: ¿qué le pasa hoy a tu usuario que esto vendría a arreglar?"
  [ ] Option B: Si insiste en features, documentar la solución propuesta pero anotar el problema inferido como riesgo a validar.

---

## Fase 2: define-users-data

### GATE IN
- [ ] Fase 1 completada — problema y usuario identificados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Definir los usuarios del producto y sus roles (ej: admin, usuario final, invitado).
2. [ ] Construir el **data model preliminar** en lenguaje natural: entidades principales y cómo se relacionan (ej: "Un usuario tiene muchos proyectos; un proyecto tiene muchas tareas"). Sin SQL ni tipos técnicos.
3. [ ] **Integración M07 (Security)** — preguntar explícitamente:
   ```
   ¿Tu producto maneja datos sensibles de usuarios (emails, pagos, información personal)?
   ```
   - Si **SÍ**: verificar si existe `.king/knowledge/secrets-provider.md`.
     - Si NO existe → agregar al handoff checklist (Phase 5) una nota:
       ```
       ⚠️ Tu producto maneja datos sensibles. Antes de `/build`, configurá tu secrets
       provider (ver M07 / secrets-management-essentials.md). Esto evita exponer
       credenciales accidentalmente.
       ```
     - **Esto es una ADVERTENCIA, no un bloqueo** — el skill continúa normalmente.
   - Si **NO**: continuar sin la nota.

### CHECKPOINT
- [ ] Usuarios y roles definidos
- [ ] Data model preliminar en lenguaje natural
- [ ] Pregunta de datos sensibles hecha; si aplica, nota de secrets provider registrada para el handoff

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El data model es ambiguo o contradictorio
Cause: Entidades sin relaciones claras o duplicadas.
Recovery:
  [ ] Option A: Dibujar las relaciones en texto con el founder ("¿un X puede tener varios Y, o solo uno?") hasta resolver la ambigüedad.
  [ ] Option B: Marcar las entidades dudosas con `[a confirmar con developer]` y continuar.

---

## Fase 3: build-journeys

### GATE IN
- [ ] Fase 2 completada — usuarios y data model definidos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Por cada journey principal (1-3), documentar:
   - **Happy path**: el recorrido ideal paso a paso
   - **2-3 edge cases**: qué pasa cuando algo sale mal (sin datos, error de pago, sesión expirada, etc.)
2. [ ] Construir los **wireframes textuales de baja fidelidad** por pantalla clave: descripción estructurada de qué elementos tiene cada pantalla (no código, no diseño visual).
3. [ ] Escribir las **user stories con acceptance criteria en lenguaje de negocio** (no Gherkin técnico):
   ```
   Como [rol], quiero [acción] para [beneficio].
   Listo cuando: [criterio de aceptación verificable en lenguaje de negocio]
   ```

### CHECKPOINT
- [ ] Journeys con happy path + 2-3 edge cases cada uno
- [ ] Wireframes textuales por pantalla clave
- [ ] User stories con acceptance criteria en lenguaje de negocio

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Los journeys no cubren los edge cases mínimos
Cause: Solo se documentó el happy path.
Recovery:
  [ ] Option A: Preguntar "¿Qué pasa si el usuario no tiene datos / falla el pago / pierde conexión?" para cada journey.
  [ ] Option B: Marcar los edge cases pendientes en el handoff checklist como "a definir antes de `/build`".

---

## Fase 4: generate-spec

### GATE IN
- [ ] Fase 3 completada — journeys, wireframes y user stories listos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Ensamblar la spec con el **schema canónico del handoff** (mismo formato siempre, para que `/mvp-accelerator` y `/build` lo parseen):
   ```markdown
   # Business Spec — {PRODUCT_NAME}
   ## Problema
   ## Usuarios y Roles
   ## Data Model (preliminar)
   ## User Stories (con acceptance criteria)
   ## Wireframes (baja fidelidad, textual)
   ## User Journeys (happy + edge cases)
   ## Handoff Checklist
   ```
2. [ ] Completar el **Handoff Checklist**: lo que el developer necesita antes de `/build` (stack sugerido, decisiones pendientes, nota de secrets provider si aplica de Phase 2).

### CHECKPOINT
- [ ] Spec ensamblada con el schema canónico completo
- [ ] Handoff checklist presente y completo
- [ ] Nota de secrets provider incluida si Phase 2 lo marcó

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: La spec queda ambigua y bloquearía al developer
Cause: Falta una sección o el handoff checklist está incompleto.
Recovery:
  [ ] Option A: Identificar la sección faltante y completarla con el founder antes de persistir.
  [ ] Option B: NO persistir una spec sin handoff checklist completo — es un gate de calidad. Volver a Phase 3 si falta información.

---

## Fase 5: output-handoff

### GATE IN
- [ ] Fase 4 completada — spec ensamblada con handoff checklist completo

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Asegurar que el directorio existe:
   ```bash
   mkdir -p docs/business-spec/
   ```
2. [ ] Escribir `docs/business-spec/YYYY-MM-DD-{slug}.md` con el schema canónico de Phase 4.
3. [ ] Confirmar al founder dónde quedó la spec y resumir en 2-3 líneas qué puede hacer ahora (pasarla a un developer o ejecutar `/mvp-accelerator`).

### CHECKPOINT
- [ ] Archivo `docs/business-spec/YYYY-MM-DD-{slug}.md` creado
- [ ] Schema canónico completo y parseable
- [ ] Handoff checklist visible

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/business-spec/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Verificar permisos y reintentar `mkdir -p docs/business-spec/`
  [ ] Option B: Crear el archivo en `docs/business-spec-YYYY-MM-DD.md` (sin subdirectorio)
  [ ] Option C: Imprimir el output en pantalla para guardado manual del founder

---

## FINAL CHECKPOINT

- [ ] Spec generada en `docs/business-spec/`
- [ ] User stories con acceptance criteria en lenguaje de negocio
- [ ] Data model preliminar + wireframes textuales + journeys con edge cases
- [ ] Handoff checklist completo (con nota de secrets provider si aplica)
- [ ] Sin input raw del usuario persistido sin delimitadores

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS (estructura verificable) |
| Artifacts | `docs/business-spec/YYYY-MM-DD-{slug}.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | Nota de secrets provider pendiente o secciones marcadas "a confirmar", si las hubo |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Spec lista, founder quiere lanzar el MVP | `/mvp-accelerator` (lee la spec automáticamente) |
| Founder quiere detallar las pantallas | `/wireframe-to-spec` |
| Developer va a implementar directamente | `/build` (con la spec como input) |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Handoff a `/mvp-accelerator` y `/build`

El schema canónico de Phase 4 es el contrato del handoff: SIEMPRE el mismo formato. `/mvp-accelerator` lee `PRODUCT_NAME` y el tipo de producto desde `docs/business-spec/` sin re-preguntar. Si el formato cambia, los consumidores se rompen — mantener el schema estable.

### Dependencia con M07 (Security)

Phase 2 solo **advierte** sobre el secrets provider; nunca bloquea. El bloqueo real ocurre en skills que tocan credenciales en runtime (ej: `/crm-integrate`). Aquí el objetivo es que el founder llegue a `/build` sabiendo que necesita configurar M07 si maneja datos sensibles.
