---
name: validate-idea
version: 1.0
api_version: 1.0.0
description: "Análisis de viabilidad de idea de SaaS: TAM/SAM/SOM conversacional, competidores, diferenciadores. Output: reporte Markdown con recomendación clara (PROCEDER / PIVOTAR / DESCARTAR)."
---

# /validate-idea — Validación de Idea de Producto

Skill conversacional que guía al entrepreneur a través de un análisis de mercado estructurado antes de invertir tiempo y dinero en construir algo.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Market Analysis Framework, lenguaje por persona | Yes | framework |

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El usuario no tiene una idea mínimamente articulada (solo "quiero hacer algo de tech") → pedir más contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA persistir input raw del usuario en `.king/sessions/` — solo outputs procesados
- Input del usuario persistido delimitado: `<!-- USER_INPUT_START -->` ... `<!-- USER_INPUT_END -->`
- NUNCA generar recomendación de PROCEDER para ideas con mercado no identificable

### REQUIRED OUTPUTS
- [ ] Reporte de validación en `docs/idea-validation/YYYY-MM-DD-{nombre-idea}.md`
- [ ] Recomendación explícita: PROCEDER | PIVOTAR | DESCARTAR con justificación
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (collect-idea) → Phase 2 (market-analysis)
             → Phase 3 (viability-decision) → Phase 4 (output-report)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/validate-idea` es standalone si no hay workflow activo.

---

## Fase 1: collect-idea

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Iniciar con contexto y propósito en lenguaje de negocio:
   ```
   Vamos a analizar si tu idea tiene potencial de mercado antes de construirla.
   Esto te va a ahorrar semanas de trabajo si hay que ajustar el enfoque.
   ```
2. [ ] Hacer las siguientes preguntas de forma conversacional (de a una, no todas juntas):
   - "¿Cómo se llama tu producto o idea?"
   - "¿Qué problema resuelve? ¿Quién lo tiene?"
   - "¿A quién le venderías? (tipo de empresa, persona, industria)"
3. [ ] Si la idea es vaga o confusa → hacer preguntas de clarificación antes de avanzar:
   - "¿Podés darme un ejemplo concreto de cómo usaría tu producto alguien?"
4. [ ] Registrar respuestas del usuario (delimitadas, anti prompt-injection)

### CHECKPOINT
- [ ] Nombre de la idea capturado
- [ ] Problema y segmento mínimamente identificados
- [ ] Si idea vaga: preguntas de clarificación hechas y respondidas antes de continuar

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Idea demasiado vaga para analizar
Cause: El usuario no puede articular el problema o el segmento.
Recovery:
  [ ] Option A: Hacer una pregunta más concreta: "¿Quién pagaría por esto hoy?" y esperar respuesta — no continuar sin al menos un segmento identificado
  [ ] Option B: Si el usuario sigue sin poder articular el problema, documentarlo como riesgo ALTO y proceder con advertencia explícita en el reporte

---

## Fase 2: market-analysis

### GATE IN
- [ ] Fase 1 completada — idea articulada con problema y segmento identificados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Guiar el análisis TAM/SAM/SOM usando preguntas del Market Analysis Framework (onboarding-essentials.md):
   - "¿Cuántas personas en el mundo tienen este problema?"
   - "¿A cuántas podés llegar con tu modelo de distribución en 2 años?"
   - "¿Cuántas van a pagar en el primer año? ¿Cuánto por mes?"
2. [ ] Identificar 2-3 competidores directos:
   - "¿Cómo resuelven esto hoy las personas que tienen el problema?"
   - "¿Qué herramientas o soluciones existen ya?"
3. [ ] Identificar diferenciador principal:
   - "¿Qué hace que tu solución sea distinta o mejor a lo que existe?"
4. [ ] Sintetizar el análisis en lenguaje de negocio (sin fórmulas ni jerga de VC)

### CHECKPOINT
- [ ] TAM/SAM/SOM estimados (pueden ser aproximados — está bien)
- [ ] Al menos 2 competidores identificados
- [ ] Diferenciador articulado por el usuario

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede estimar el mercado
Cause: Idea demasiado nueva o mercado inexistente.
Recovery:
  [ ] Option A: Documentar TAM como "mercado emergente / no cuantificable aún" — es una señal de riesgo, no un blocker
  [ ] Option B: Si no hay competidores → puede ser mercado sin demanda o innovación real — documentar ambas posibilidades

---

## Fase 3: viability-decision

### GATE IN
- [ ] Fase 2 completada — TAM/SAM/SOM estimados, competidores y diferenciador identificados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Evaluar la viabilidad con estos criterios:

   | Criterio | Señal positiva | Señal negativa |
   |----------|---------------|----------------|
   | Mercado | SAM > 10K personas con poder adquisitivo | SAM < 1K o no cuantificable |
   | Problema | Dolor real y frecuente | Problema "nice to have" |
   | Diferenciador | Claro y defendible | "Igual pero más barato" |
   | Competencia | Mercado activo pero fragmentado | Monopolio establecido o sin demanda |

2. [ ] Generar recomendación en lenguaje de negocio:
   - **PROCEDER**: al menos 3 criterios positivos. Indicar el path → `/lean-canvas` o `/user-story-mapping` → `/mvp-accelerator`
   - **PIVOTAR**: 1-2 criterios negativos pero idea salvageable. Indicar qué ajustar.
   - **DESCARTAR**: 3+ criterios negativos o problema no validado. Explicar por qué con respeto.

3. [ ] Presentar recomendación al usuario ANTES de escribir el reporte

### CHECKPOINT
- [ ] Recomendación generada (PROCEDER | PIVOTAR | DESCARTAR)
- [ ] Justificación basada en criterios documentados
- [ ] Recomendación en lenguaje de negocio, sin jerga técnica

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Criterios insuficientes para recomendar
Cause: Información de mercado demasiado incompleta.
Recovery:
  [ ] Option A: Identificar qué criterio falta, hacer la pregunta específica y retomar
  [ ] Option B: Generar recomendación "CONDICIONAL" — proceder solo si el usuario puede validar el criterio faltante con usuarios reales

---

## Fase 4: output-report

### GATE IN
- [ ] Fase 3 completada — recomendación generada y presentada al usuario

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Asegurar que el directorio existe:
   ```bash
   mkdir -p docs/idea-validation/
   ```
2. [ ] Escribir reporte en `docs/idea-validation/YYYY-MM-DD-{slug-idea}.md` con:
   - **Nombre del producto**
   - **Problema y segmento**
   - **TAM / SAM / SOM** (estimados, con fuente si hay)
   - **Competidores** (2-3 con una línea de comparación)
   - **Diferenciador**
   - **Recomendación**: PROCEDER | PIVOTAR | DESCARTAR
   - **Justificación**: 1 párrafo en lenguaje de negocio
   - **Próximos pasos** (si PROCEDER → `/lean-canvas`; si PIVOTAR → qué ajustar; si DESCARTAR → por qué)
3. [ ] Confirmar al usuario que el reporte fue generado y dónde está

### CHECKPOINT
- [ ] Archivo `docs/idea-validation/YYYY-MM-DD-{slug}.md` creado
- [ ] Reporte tiene todas las secciones requeridas
- [ ] Recomendación visible al inicio del reporte
- [ ] Sin terminología técnica de negocio compleja (DCF, CAGR, etc.) a menos que el usuario la use

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/idea-validation/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Crear en `docs/idea-validation-YYYY-MM-DD.md` directamente, o imprimir en pantalla para guardado manual.

---

## FINAL CHECKPOINT

- [ ] Reporte de validación generado en `docs/idea-validation/`
- [ ] Recomendación explícita: PROCEDER | PIVOTAR | DESCARTAR
- [ ] Sin input raw del usuario persistido sin delimitadores
- [ ] Sin terminología técnica expuesta sin contexto al entrepreneur

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS (estructura verificable) |
| Artifacts | `docs/idea-validation/YYYY-MM-DD-{slug}.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | None si recomendación generada con criterios completos |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Recomendación = PROCEDER (definir modelo de negocio) | `/lean-canvas` |
| Recomendación = PROCEDER (mapear journey / ir a spec) | `/user-story-mapping` |
| Recomendación = PIVOTAR | Re-ejecutar `/validate-idea` con ajustes |
| Recomendación = DESCARTAR | Fin del flujo. Sugerir nueva idea. |
