---
name: landing-page-generate
version: 1.0
api_version: 1.0.0
description: "Genera landing page con conversion optimization. Auto-detecta target (HTML, React, Vue). Integra S13 Design Intelligence si disponible. Sin inserción dinámica de HTML sin sanitización."
---

# /landing-page-generate — Landing Page para Entrepreneur

Genera una landing page lista para publicar con el contenido de tu producto. Detecta el framework de tu proyecto y genera la estructura correcta. Si el sistema de diseño S13 está disponible, usa la paleta y tipografía de tu marca.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/design-essentials.md` | Design tokens S13, paletas, tipografía | No (graceful) | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio | Yes | framework |

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El usuario no puede articular ni el nombre ni el tagline del producto

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA generar inserción directa de HTML no sanitizado con variables dinámicas
- Siempre usar `textContent` o el mecanismo de escape automático del framework
- NUNCA usar funciones de evaluación de código arbitrario en el código generado

### REQUIRED OUTPUTS
- [ ] Landing page generada en el target detectado (HTML | React | Vue)
- [ ] Archivo guardado en `public/landing/` o equivalente del proyecto
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (detect-target) → Phase 2 (inject-design)
             → Phase 3 (collect-content) → Phase 4 (generate-landing) → Phase 5 (performance-check)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/landing-page-generate` es standalone si no hay workflow activo.

---

## Fase 1: detect-target

### GATE IN
- [ ] Knowledge injection completada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Leer `package.json` para detectar el framework:
   - `react`, `next` → React (TSX)
   - `vue` → Vue (SFC)
   - Sin `package.json` o sin framework → HTML puro
2. [ ] Informar al usuario:
   ```
   Voy a crear la landing page para tu app.
   La voy a hacer en [React/Vue/HTML] para que encaje con tu proyecto.
   ```

### CHECKPOINT
- [ ] Target identificado: `html` | `react` | `vue`

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede detectar el framework
Cause: `package.json` ilegible o sin dependencias claras.
Recovery:
  [ ] Preguntar al usuario: "¿Tu app está hecha con React, Vue, o es una página web simple?"

---

## Fase 2: inject-design

### GATE IN
- [ ] Fase 1 completada — target identificado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Intentar cargar `knowledge/_inject/design-essentials.md`
2. [ ] Si existe → usar paleta, tipografía y estilos del sistema S13
3. [ ] Si NO existe → usar defaults seguros:
   - Paleta: azul (#2563EB) + blanco + gris oscuro (#1F2937)
   - Tipografía: system-ui, Inter, sans-serif
   - Espaciado: Tailwind defaults

### CHECKPOINT
- [ ] Sistema de diseño cargado o defaults definidos

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: design-essentials.md no encontrado y defaults no aplicables
Cause: Error de lectura del archivo de knowledge o ruta incorrecta.
Recovery:
  [ ] Continuar con los defaults seguros del paso 3. Esta fase es graceful por diseño — la ausencia de S13 no bloquea la generación.

---

## Fase 3: collect-content

### GATE IN
- [ ] Fase 2 completada — diseño definido

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Preguntar de a uno en lenguaje de negocio:
   - "¿Cómo se llama tu producto?"
   - "¿En una frase: qué hace tu producto?"  (tagline)
   - "¿Cuáles son las 3 cosas más importantes que hace?" (features)
   - "¿Qué querés que hagan los visitantes?" (CTA: registrarse / contratar / ver demo)
   - "¿Tenés testimonios de clientes o números de tracción?" (opcional — si no, omitir)
2. [ ] Registrar respuestas con delimitadores: `<!-- USER_INPUT_START -->` ... `<!-- USER_INPUT_END -->`

### CHECKPOINT
- [ ] Nombre, tagline, 3 features y CTA recopilados
- [ ] Testimonios opcionales documentados

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Usuario no puede articular el tagline o el CTA
Cause: Propuesta de valor poco clara.
Recovery:
  [ ] Sugerir: "¿Podés completar esta frase? 'Mi app ayuda a [QUIÉN] a [QUÉ] sin [DOLOR]'" Usar la respuesta como tagline.

---

## Fase 4: generate-landing

### GATE IN
- [ ] Fase 3 completada — contenido recopilado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Cargar el sub-archivo del target:
   - `html` → leer `skills/landing-page-generate/HTML.md`
   - `react` → leer `skills/landing-page-generate/REACT.md`
   - `vue` → leer `skills/landing-page-generate/VUE.md`
2. [ ] Generar el código de la landing page con:
   - Secciones: Hero, Features (3), CTA, Testimonios (si hay), Footer
   - Diseño responsivo (mobile-first)
   - Sin inserción dinámica de HTML — solo contenido estático o escapado por el framework
3. [ ] Guardar en el proyecto del usuario (path según target y estructura del proyecto)

### CHECKPOINT
- [ ] Landing page generada con Hero + Features + CTA
- [ ] Sin inserción dinámica de HTML sin sanitización
- [ ] Archivo guardado en el proyecto

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Landing page no se puede guardar en el proyecto
Cause: Path no determinado o estructura de proyecto inusual.
Recovery:
  [ ] Guardar en `public/landing.html` (HTML) o `src/pages/landing.tsx` (React) como fallback.

---

## Fase 5: performance-check

### GATE IN
- [ ] Fase 4 completada — landing page generada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar si aplica el performance gate (el proyecto tiene frontend con bundler)
2. [ ] Si aplica → revisar según `rules/performance-gate.md`:
   - Sin imágenes sin optimizar (usar `next/image` en React, `<img loading="lazy">` en HTML)
   - Sin CSS bloqueante no crítico
   - Sin scripts síncronos en `<head>`
3. [ ] Confirmar al usuario:
   ```
   ¡Tu landing page está lista!
   La podés ver en [path generado].

   Cuando subas tu app, tu landing page va a estar en tu dominio.
   Ahora podés continuar con /mvp-accelerator para lanzar todo junto.
   ```

### CHECKPOINT
- [ ] Performance gate verificado o documentado como no aplicable
- [ ] Confirmación en lenguaje de negocio enviada

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Performance gate no se puede ejecutar o no hay reglas aplicables
Cause: `rules/performance-gate.md` no encontrado, o el proyecto no tiene frontend detectable.
Recovery:
  [ ] Documentar "Performance gate: skipped — reglas no encontradas" en el reporte de sesión. Continuar — la landing ya fue generada y es el artefacto principal.

---

## FINAL CHECKPOINT

- [ ] Landing page generada en el target correcto
- [ ] Sin inserción dinámica de HTML sin sanitización
- [ ] Contenido del usuario persistido con delimitadores
- [ ] Performance gate verificado

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS |
| Artifacts | Landing page en `public/landing/` o equivalente |
| Next Recommended | Ver tabla de flujo |
| Risks | None si sin dominio: artefacto entregado igualmente |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Landing generada + MVP completo | `/mvp-accelerator` para orquestar todo |
| Solo necesita landing page | Publicar manualmente via deploy |
| Sin dominio aún | Hacer deploy primero, luego la landing es pública |
