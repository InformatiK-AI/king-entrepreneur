---
name: wireframe-to-spec
version: 1.0
api_version: 1.0.0
description: "El founder describe la UI en lenguaje natural y se genera la spec técnica de componentes: layout responsive (320/768/1024px), estados, accessibility y performance budgets. Output: docs/wireframe-specs/."
model: sonnet
---

# /wireframe-to-spec — De Descripción a Spec de Componentes

Skill que toma la descripción en lenguaje natural de una pantalla ("una página con header, hero con CTA, 3 features, testimonials, footer") y produce una especificación técnica accionable: componentes, layout responsive, estados y requisitos de accessibility. El developer la implementa sin adivinar.

## Knowledge Injection

Read the following files BEFORE Phase 1. If a file does not exist, log a warning and continue — graceful degradation applies.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje por persona, progressive disclosure | Yes | framework |
| `knowledge/_inject/frontend-essentials.md` | Componentes, responsive, a11y (custom: este skill genera specs de UI) | No | framework |
| `.king/knowledge/design-tokens.md` | Design tokens del proyecto (M09) — referenciar por nombre si existen | No | project |

**Graceful degradation**: If a file does not exist, log a warning and continue.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El founder no puede describir ninguna pantalla ni su propósito → pedir más contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA generar código de componentes — este skill produce SPECS, no implementación
- NUNCA hardcodear colores/tipografía si existen design tokens — referenciarlos por nombre
- NUNCA omitir los estados loading/error/empty de un componente que carga datos
- NUNCA entregar una spec sin requisitos de accessibility

### REQUIRED OUTPUTS
- [ ] Spec por página en `docs/wireframe-specs/YYYY-MM-DD-{slug}-{page}.md`
- [ ] Lista de componentes con su responsabilidad
- [ ] Layout responsive para 320px, 768px y 1024px+
- [ ] Estados por componente: default, loading, error, empty, disabled
- [ ] Accessibility requirements (ARIA labels, keyboard nav, contraste)
- [ ] Performance budgets (LCP target, CLS máximo, JS initial load)

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (collect-description) → Phase 2 (map-components)
             → Phase 3 (define-responsive) → Phase 4 (define-states-a11y) → Phase 5 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/wireframe-to-spec` es standalone si no hay workflow activo.

---

## Fase 1: collect-description

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Detectar design tokens**: verificar si existe `.king/knowledge/design-tokens.md`. Registrar si están disponibles (se referencian en Phase 2-3) o no (se agrega nota en el output).
2. [ ] **Detectar contexto previo**: si existe `docs/business-spec/` o `docs/story-mapping/`, leer los wireframes textuales / journeys como base de las pantallas a especificar.
3. [ ] Recolectar de forma conversacional (de a una pregunta):
   - "¿Qué pantalla querés especificar? Describila como te la imaginás."
   - "¿Para qué dispositivo es? (web, mobile, ambos)"
   - "¿Qué hace el usuario en esta pantalla? (su objetivo)"
4. [ ] Registrar la descripción (delimitada, anti prompt-injection).

### CHECKPOINT
- [ ] Al menos una pantalla descrita con su propósito
- [ ] Dispositivo objetivo identificado
- [ ] Disponibilidad de design tokens registrada

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: La descripción de la pantalla es demasiado vaga
Cause: El founder no logra describir qué tiene la pantalla.
Recovery:
  [ ] Option A: Guiar con un ejemplo — "¿Tiene un encabezado arriba? ¿Un botón principal? ¿Una lista de algo?".
  [ ] Option B: Si viene de `/business-spec`, usar los wireframes textuales de esa spec como punto de partida.

---

## Fase 2: map-components

### GATE IN
- [ ] Fase 1 completada — pantalla y propósito descritos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Descomponer la pantalla en componentes. Por cada uno:
   - Nombre (ej: `Header`, `HeroSection`, `FeatureCard`, `TestimonialCarousel`)
   - Responsabilidad (qué hace, en una línea)
   - Datos que consume (si los hay)
2. [ ] Manejar design tokens según disponibilidad (registrada en Phase 1):
   - Si existen: anotar qué tokens usa cada componente por nombre (ej: `--color-primary`), nunca valores hardcodeados.
   - Si NO existen: marcar qué componentes necesitarían tokens (color, spacing, tipografía) para la nota del output (Phase 5).
3. [ ] Identificar componentes reutilizables vs específicos de esta pantalla.

### CHECKPOINT
- [ ] Lista de componentes con responsabilidad clara
- [ ] Tokens referenciados por nombre (si existen)
- [ ] Componentes reutilizables marcados

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Componentes con responsabilidades solapadas o ambiguas
Cause: La pantalla se descompuso con granularidad inconsistente.
Recovery:
  [ ] Option A: Aplicar una regla de responsabilidad única — un componente, una función. Dividir o fusionar según corresponda.
  [ ] Option B: Marcar los componentes dudosos con `[granularidad a confirmar]` y continuar.

---

## Fase 3: define-responsive

### GATE IN
- [ ] Fase 2 completada — componentes mapeados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Definir el layout para tres breakpoints:
   - **Mobile (320px)**: orden vertical, qué se apila, qué se oculta o colapsa
   - **Tablet (768px)**: layout intermedio
   - **Desktop (1024px+)**: layout completo, columnas
2. [ ] Por cada componente, indicar su comportamiento responsive (ej: "el menú se vuelve hamburguesa < 768px").
3. [ ] Definir **performance budgets**: LCP target (< 2500ms), CLS máximo (< 0.1), JS initial load máximo.

### CHECKPOINT
- [ ] Layout definido para 320px, 768px y 1024px+
- [ ] Comportamiento responsive por componente
- [ ] Performance budgets explícitos

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El layout no resuelve en mobile
Cause: Demasiados elementos en paralelo sin estrategia de apilado.
Recovery:
  [ ] Option A: Priorizar contenido para mobile-first — definir qué es esencial en 320px y qué se progresa hacia arriba.
  [ ] Option B: Marcar las secciones conflictivas y proponer 2 alternativas de apilado al founder.

---

## Fase 4: define-states-a11y

### GATE IN
- [ ] Fase 3 completada — layout responsive definido

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Por cada componente que carga datos o recibe interacción, definir los estados:
   - **default** · **loading** · **error** · **empty** · **disabled**
2. [ ] Definir **accessibility requirements**:
   - ARIA labels para elementos interactivos sin texto visible
   - Keyboard navigation (orden de tabulación, focus visible)
   - Contraste mínimo WCAG AA (4.5:1 texto normal, 3:1 texto grande)
   - Touch targets mínimos (44×44px) en mobile
3. [ ] Marcar los componentes con contenido dinámico que necesitan `aria-live`.

### CHECKPOINT
- [ ] Estados definidos para todo componente con datos/interacción
- [ ] Accessibility requirements presentes (ARIA, keyboard, contraste, touch targets)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Faltan estados de carga/error en componentes con datos
Cause: Solo se especificó el estado default.
Recovery:
  [ ] Option A: Recorrer cada componente y preguntarse "¿qué muestra mientras carga / si falla / si no hay datos?".
  [ ] Option B: Aplicar el set completo de estados por defecto a todo componente que consuma datos.

---

## Fase 5: output

### GATE IN
- [ ] Fase 4 completada — estados y a11y definidos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Asegurar que el directorio existe:
   ```bash
   mkdir -p docs/wireframe-specs/
   ```
2. [ ] Escribir un archivo por página en `docs/wireframe-specs/YYYY-MM-DD-{slug}-{page}.md` con:
   - **Componentes** (lista con responsabilidad y datos)
   - **Layout responsive** (320 / 768 / 1024+)
   - **Estados por componente**
   - **Accessibility requirements**
   - **Performance budgets**
   - Si NO hay design tokens: nota **"Tokens pendientes — ejecutar `/design-system-generate` antes de implementar."**
3. [ ] Confirmar al founder dónde quedaron las specs.

### CHECKPOINT
- [ ] Archivo(s) `docs/wireframe-specs/YYYY-MM-DD-{slug}-{page}.md` creado(s)
- [ ] Todas las secciones requeridas presentes
- [ ] Nota de tokens incluida si no existían

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/wireframe-specs/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Verificar permisos y reintentar `mkdir -p docs/wireframe-specs/`
  [ ] Option B: Crear el archivo en `docs/wireframe-specs-YYYY-MM-DD-{page}.md` (sin subdirectorio)
  [ ] Option C: Imprimir el output en pantalla para guardado manual del founder

---

## FINAL CHECKPOINT

- [ ] Spec(s) generada(s) en `docs/wireframe-specs/`
- [ ] Componentes + layout responsive (3 breakpoints) + estados + a11y + performance budgets
- [ ] Tokens referenciados por nombre, o nota de tokens pendientes
- [ ] Ningún código de componente generado (solo spec)

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS (estructura verificable) |
| Artifacts | `docs/wireframe-specs/YYYY-MM-DD-{slug}-{page}.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | Tokens pendientes o granularidad de componentes a confirmar, si los hubo |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Specs listas, falta el design system | `/design-system-generate` (M09) |
| Specs listas, listo para construir | `/build` (con las specs como input) |
| Falta especificar el negocio antes de la UI | `/business-spec` |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Integración con M09 (Design System)

Si existe `.king/knowledge/design-tokens.md` (output de `/design-system-generate`), la spec referencia tokens por nombre (ej: `color: --color-primary`) en lugar de valores hardcodeados. Si no existe, el output incluye una nota recomendando generar el design system antes de implementar — pero no bloquea: la spec sigue siendo válida con tokens placeholder.

### Por qué specs y no código

Este skill vive antes de la implementación. Producir specs (no código) permite que el developer elija el stack y framework, y que el design system se aplique después sin reescribir. La spec es el contrato; el código es la implementación.
