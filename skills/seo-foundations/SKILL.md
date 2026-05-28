---
name: seo-foundations
version: 1.0
api_version: 1.0.0
description: "SEO técnico completo + optimización para AI search (SGE/Perplexity/ChatGPT) desde el primer deploy: metadata, structured data, sitemap, robots.txt, llms.txt y auditoría. Output: cambios al <head> + public/ + docs/seo/."
---

# /seo-foundations — SEO Técnico y AI Search desde el Día 0

Skill que deja el proyecto con SEO técnico sólido y optimizado para motores de búsqueda con IA. Audita la landing existente, completa la metadata, genera structured data, sitemap, robots.txt y `llms.txt`, y produce un reporte de auditoría. Aplica cambios quirúrgicos — no regenera lo que el founder customizó.

## Knowledge Injection

Read the following files BEFORE Phase 1. If a file does not exist, log a warning and continue — graceful degradation applies.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/growth-patterns.md` | SEO essentials, Core Web Vitals, structured data, AI search / llms.txt | Yes | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje por persona, progressive disclosure | No | framework |

**Graceful degradation**: If a file does not exist, log a warning and continue.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El proyecto no tiene nombre, descripción ni URL canónica identificables → pedir contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA regenerar completa una landing existente — aplicar cambios quirúrgicos al `<head>`, preservar customizaciones del founder
- NUNCA generar structured data con errores en Google Rich Results Test
- NUNCA omitir `llms.txt` cuando hay nombre, descripción y URL canónica disponibles
- NUNCA exponer jerga técnica de SEO sin contexto de negocio al founder

### REQUIRED OUTPUTS
- [ ] Metadata completa en el `<head>` de las páginas
- [ ] `public/sitemap.xml` (regenerado en cada build si Next.js/Astro)
- [ ] `public/robots.txt` con configuración correcta
- [ ] `public/llms.txt` con descripción, features y URL canónica
- [ ] Structured data JSON-LD (Organization + WebSite + Product/Service)
- [ ] `docs/seo/audit-YYYY-MM-DD.md` con estado e items pendientes

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (audit-existing) → Phase 2 (generate-metadata)
             → Phase 3 (generate-structured-data) → Phase 4 (generate-sitemap-robots-llmstxt)
             → Phase 5 (output-audit-report)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas (Lighthouse SEO 95+ verificable en CI)

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/seo-foundations` es standalone si no hay workflow activo.

---

## Fase 1: audit-existing

### GATE IN
- [ ] Knowledge injection completada (growth-patterns.md leído — sección SEO)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Detectar si existe una landing generada (ej: por `/landing-page-generate`) e inspeccionar su `<head>`, sitemap, robots.txt actuales.
2. [ ] Identificar el stack (Next.js, Astro, HTML estático, etc.) de `.king/knowledge/stack.md` o por inspección.
3. [ ] Capturar los datos base del producto: nombre, descripción, URL canónica, features principales (de `docs/business-spec/` o `docs/idea-validation/` si existen).
4. [ ] Listar los gaps de SEO encontrados (metadata faltante, sin structured data, sin sitemap, etc.).

### CHECKPOINT
- [ ] Estado SEO actual de la landing inspeccionado (o "no hay landing aún")
- [ ] Stack identificado
- [ ] Datos base del producto capturados (nombre, descripción, URL canónica)
- [ ] Lista de gaps SEO generada

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No hay landing ni datos base del producto
Cause: El proyecto no tiene página ni nombre/descripción definidos.
Recovery:
  [ ] Option A: Capturar nombre, descripción y URL conversacionalmente con el founder.
  [ ] Option B: Sugerir ejecutar `/landing-page-generate` primero y luego volver a `/seo-foundations`.

---

## Fase 2: generate-metadata

### GATE IN
- [ ] Fase 1 completada — datos base capturados y gaps identificados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Leer el `<head>` existente (si hay landing) y detectar qué metadata ya está presente.
2. [ ] Aplicar la metadata por **merge QUIRÚRGICO** (nunca regenerar el archivo):
   - Si falta: agregar (`<title>` ≤60 chars, `meta description` ≤155 chars, Open Graph, Twitter Card, `hreflang` si multilingüe)
   - Si ya existe y está customizada por el founder: PRESERVAR
   - `canonical` URL: agregar o actualizar siempre a la URL canónica actual
3. [ ] Definir Core Web Vitals targets (LCP < 2500ms, CLS < 0.1, INP < 200ms) como baseline en el reporte.

### CHECKPOINT
- [ ] Metadata completa aplicada sin sobreescribir customizaciones
- [ ] Canonical URLs presentes en todas las páginas
- [ ] CWV targets definidos

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: La landing tiene metadata customizada que se pisaría
Cause: El founder ya editó títulos/descripciones a mano.
Recovery:
  [ ] Option A: Hacer merge quirúrgico — completar solo los campos faltantes, preservar los customizados.
  [ ] Option B: Listar los conflictos y pedir al founder cuáles preservar.

---

## Fase 3: generate-structured-data

### GATE IN
- [ ] Fase 2 completada — metadata aplicada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar structured data JSON-LD para la página principal: **Organization + WebSite + Product/Service**.
2. [ ] Agregar structured data adicional donde aplique: `BreadcrumbList`, `FAQPage`, `Product`.
3. [ ] Asegurar que el JSON-LD tiene todos los campos requeridos por schema.org para que pase Google Rich Results Test con 0 errores (verificable en CI / externamente — no solo a ojo).

### CHECKPOINT
- [ ] JSON-LD Organization + WebSite + Product/Service generado
- [ ] Structured data adicional donde aplica
- [ ] Sin errores de campos requeridos (Rich Results Test)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Faltan campos requeridos en el structured data
Cause: Datos del producto incompletos (ej: sin logo para Organization).
Recovery:
  [ ] Option A: Capturar el dato faltante con el founder (logo URL, nombre legal, etc.).
  [ ] Option B: Generar el schema con los campos disponibles y marcar los faltantes como pendientes en el reporte.

---

## Fase 4: generate-sitemap-robots-llmstxt

### GATE IN
- [ ] Fase 3 completada — structured data generado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Detectar el directorio de assets estáticos del stack (`public/` en Next.js/Astro/Vite, `static/` en Hugo/SvelteKit, etc.) usando `.king/knowledge/stack.md`. Asegurar que existe:
   ```bash
   mkdir -p {ASSETS_DIR}/
   ```
2. [ ] Generar `{ASSETS_DIR}/sitemap.xml` (con regeneración por build si el stack es Next.js/Astro).
3. [ ] Generar `{ASSETS_DIR}/robots.txt` con configuración correcta (permitir crawl, referenciar sitemap).
4. [ ] Generar `{ASSETS_DIR}/llms.txt` (AI search): nombre del producto, descripción de 1-2 líneas, features principales, URL canónica, links a docs.

### CHECKPOINT
- [ ] `{ASSETS_DIR}/sitemap.xml` generado (en el directorio de assets del stack)
- [ ] `{ASSETS_DIR}/robots.txt` generado y referencia el sitemap
- [ ] `{ASSETS_DIR}/llms.txt` con descripción, features y URL canónica

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio public/ no existe y no se puede crear
Cause: Estructura del proyecto distinta (ej: `static/` en Astro/Hugo).
Recovery:
  [ ] Option A: Detectar el directorio de assets estáticos del stack (`static/`, `public/`, `assets/`) y usar ese.
  [ ] Option B: Generar los archivos en la raíz e indicar al founder dónde moverlos según su framework.

---

## Fase 5: output-audit-report

### GATE IN
- [ ] Fase 4 completada — sitemap/robots/llms.txt generados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Asegurar que el directorio existe:
   ```bash
   mkdir -p docs/seo/
   ```
2. [ ] Escribir `docs/seo/audit-YYYY-MM-DD.md` con estas secciones:
   - **Estado antes**: gaps SEO encontrados en Fase 1
   - **Cambios aplicados**: metadata, structured data, sitemap, robots.txt, llms.txt
   - **Core Web Vitals targets**: LCP < 2500ms, CLS < 0.1, INP < 200ms
   - **Items pendientes**: campos requeridos sin datos, si los hay
   - **Cómo verificar en CI**: comando Lighthouse (threshold SEO ≥ 95) + Rich Results Test (0 errores)
3. [ ] Confirmar al founder qué se aplicó y qué queda pendiente.

### CHECKPOINT
- [ ] `docs/seo/audit-YYYY-MM-DD.md` creado con estado e items pendientes
- [ ] Reporte indica cómo verificar Lighthouse SEO 95+
- [ ] Founder sabe qué se cambió

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/seo/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Verificar permisos y reintentar `mkdir -p docs/seo/`
  [ ] Option B: Crear el archivo en `docs/seo-audit-YYYY-MM-DD.md` (sin subdirectorio)
  [ ] Option C: Imprimir el reporte en pantalla para guardado manual del founder

---

## FINAL CHECKPOINT

- [ ] Metadata completa en el `<head>` (cambios quirúrgicos, sin pisar customizaciones)
- [ ] `public/sitemap.xml` + `public/robots.txt` + `public/llms.txt` generados
- [ ] Structured data JSON-LD sin errores (Organization + WebSite + Product/Service)
- [ ] `docs/seo/audit-YYYY-MM-DD.md` con estado e items pendientes
- [ ] llms.txt presente cuando hay nombre, descripción y URL canónica

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS (Lighthouse SEO 95+ verificable en CI) |
| Artifacts | `<head>` metadata, `public/sitemap.xml`, `public/robots.txt`, `public/llms.txt`, `docs/seo/audit-*.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | CWV reales no medidos hasta el deploy; items pendientes del audit, si los hay |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| SEO listo, falta medir tráfico | `/analytics-setup` |
| SEO listo, implementar content loop | `/growth-loops` (loop content) |
| No había landing — generarla primero | `/landing-page-generate` |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Integración con `/landing-page-generate`

`/landing-page-generate` ya genera "SEO ready" básico (metadata, JSON-LD, sitemap, robots.txt). `/seo-foundations` lo EXTIENDE: audita lo existente, corrige gaps, agrega structured data adicional (BreadcrumbList, FAQPage), `llms.txt` y canonical URLs. Lee el archivo existente y aplica cambios quirúrgicos — nunca regenera completo para no pisar lo que el founder customizó.

### AI Search y llms.txt

`llms.txt` (spec v0.1) es el equivalente a `robots.txt` para motores con IA (SGE, Perplexity, ChatGPT Search): describe el producto en Markdown plano para que las IAs lo citen con precisión. Detalle en `growth-patterns.md` (sección AI Search Optimization).
