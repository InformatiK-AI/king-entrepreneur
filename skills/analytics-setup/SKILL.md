---
name: analytics-setup
version: 1.0
api_version: 1.0.0
description: "Configura analytics de negocio con eventos predefinidos por tipo de producto, tipado estricto, funnels y cohorts desde el día 0 (PostHog/Amplitude/Mixpanel). Output: analytics/events + docs/analytics/tracking-plan.md."
model: sonnet
---

# /analytics-setup — Analytics de Negocio desde el Día 0

Skill que configura el tracking de producto con eventos predefinidos según el tipo de producto, tipado estricto, dashboards de activación/retención y un tracking plan documentado. El founder mide el funnel desde el primer usuario, sin armar nada a mano.

## Knowledge Injection

Read the following files BEFORE Phase 1. If a file does not exist, log a warning and continue — graceful degradation applies.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje por persona, progressive disclosure | Yes | framework |
| `.king/knowledge/stack.md` | Stack del proyecto para generar events file en el lenguaje correcto | No | project |

**Graceful degradation**: If a file does not exist, log a warning and continue.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El founder no puede indicar el tipo de producto (SaaS/Marketplace/Mobile/API/E-commerce) → pedir contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA hardcodear API keys del provider — solo leer desde `.env`
- NUNCA generar eventos sin tipado estricto de sus propiedades
- NUNCA redefinir eventos que otro skill ya creó (ej: `/growth-loops`) — importar y extender
- NUNCA exponer jerga técnica del SDK sin contexto de negocio al founder

### REQUIRED OUTPUTS
- [ ] Configuración del SDK sin credenciales hardcodeadas (vía `.env`)
- [ ] Archivo `analytics/events.{ts|js|...}` con eventos predefinidos + tipado estricto
- [ ] Dashboards definidos: funnel de activación, retención D1/D7/D30, cohort
- [ ] `docs/analytics/tracking-plan.md` con cada evento, propiedades y criterio de disparo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (collect-provider-type) → Phase 2 (configure-sdk)
             → Phase 3 (generate-events-file) → Phase 4 (setup-dashboards)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/analytics-setup` es standalone si no hay workflow activo.

---

## Fase 1: collect-provider-type

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Preguntar el provider: **PostHog (default)** | Amplitude | Mixpanel.
2. [ ] Preguntar el tipo de producto: SaaS | Marketplace | Mobile | API-only | E-commerce.
3. [ ] Presentar la tabla de eventos predefinidos que se van a configurar según el tipo:

   | Tipo | Eventos predefinidos |
   |------|----------------------|
   | SaaS | `signup`, `email_verified`, `first_feature_used`, `plan_upgraded`, `subscription_cancelled` |
   | Marketplace | `search_performed`, `listing_viewed`, `transaction_started`, `transaction_completed` |
   | Mobile | `app_opened`, `onboarding_completed`, `feature_used`, `push_opted_in`, `session_ended` |
   | API-only | `api_key_created`, `first_call`, `rate_limit_hit`, `plan_upgraded` |
   | E-commerce | `product_viewed`, `add_to_cart`, `checkout_started`, `purchase_completed` |

### CHECKPOINT
- [ ] Provider elegido
- [ ] Tipo de producto definido
- [ ] Set de eventos predefinidos confirmado con el founder

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Tipo de producto ambiguo
Cause: El producto no encaja claro en una categoría.
Recovery:
  [ ] Option A: Preguntar "¿Tu usuario paga una suscripción, hace transacciones, o usa una app?" para clasificar.
  [ ] Option B: Usar el set SaaS como base (el más común) y permitir agregar eventos custom.

---

## Fase 2: configure-sdk

### GATE IN
- [ ] Fase 1 completada — provider y tipo definidos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar la configuración del SDK del provider elegido, leyendo la API key desde `.env` (nunca hardcodeada).
2. [ ] Agregar las variables necesarias a `.env.example` (sin valores reales).
3. [ ] Inicializar el cliente de analytics en el punto de entrada del stack (auto-detectar de `stack.md` si existe).

### CHECKPOINT
- [ ] SDK configurado leyendo credenciales desde `.env`
- [ ] `.env.example` actualizado con las variables del provider
- [ ] Ninguna API key hardcodeada en el código

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Stack no detectado para inicializar el SDK
Cause: No existe `.king/knowledge/stack.md` ni se puede inferir el framework.
Recovery:
  [ ] Option A: Preguntar al founder el framework (Next.js, React, Vue, etc.) y configurar para ese.
  [ ] Option B: Generar la config en formato agnóstico con instrucciones de dónde inicializar el cliente.

---

## Fase 3: generate-events-file

### GATE IN
- [ ] Fase 2 completada — SDK configurado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Detectar eventos existentes**: si ya existe `analytics/events.*` (de una configuración previa o manual), IMPORTAR y EXTENDER — nunca redefinir. Nota: `/growth-loops` corre DESPUÉS de este skill y extenderá este mismo archivo.
2. [ ] Generar `analytics/events.{ts|js|...}` con los eventos predefinidos del tipo de producto, cada uno con **tipado estricto de propiedades** (ej: `signup: { method: 'email' | 'google'; plan: string }`).
3. [ ] Documentar inline el criterio de disparo de cada evento.

### CHECKPOINT
- [ ] `analytics/events.*` creado o extendido (sin duplicar eventos existentes)
- [ ] Cada evento con tipado estricto de propiedades
- [ ] Naming convention canónica respetada

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Conflicto de eventos con un archivo existente
Cause: `analytics/events.*` ya define eventos con el mismo nombre.
Recovery:
  [ ] Option A: Importar los existentes y agregar solo los nuevos, evitando double-counting.
  [ ] Option B: Si hay colisión de nombres con semántica distinta, renombrar el nuevo con prefijo y documentar.

---

## Fase 4: setup-dashboards

### GATE IN
- [ ] Fase 3 completada — events file generado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Definir los dashboards predefinidos:
   - **Funnel de activación**: signup → activation → conversion
   - **Retención**: D1 / D7 / D30
   - **Cohort analysis** semanal
2. [ ] Generar la configuración para CREARLOS en el provider elegido (config importable o instrucciones paso-a-paso): PostHog (dashboard + insights JSON), Amplitude (charts + cohorts), Mixpanel (funnels/retention reports).
3. [ ] Asegurar que el directorio existe:
   ```bash
   mkdir -p docs/analytics/
   ```
4. [ ] Escribir `docs/analytics/tracking-plan.md` con: cada evento, sus propiedades tipadas y el criterio de disparo.
5. [ ] Confirmar al founder dónde quedó el tracking plan y cómo activar los dashboards en el provider.

### CHECKPOINT
- [ ] Dashboards definidos (funnel, retención, cohort)
- [ ] Config/instrucciones de dashboards generadas para el provider
- [ ] `docs/analytics/tracking-plan.md` creado con todos los eventos
- [ ] Founder sabe cómo ver los dashboards

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/analytics/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Verificar permisos y reintentar `mkdir -p docs/analytics/`
  [ ] Option B: Crear el archivo en `docs/analytics-tracking-plan-YYYY-MM-DD.md` (sin subdirectorio)
  [ ] Option C: Imprimir el tracking plan en pantalla para guardado manual del founder

---

## FINAL CHECKPOINT

- [ ] SDK configurado sin credenciales hardcodeadas (vía `.env`)
- [ ] `analytics/events.*` con eventos predefinidos + tipado estricto
- [ ] Dashboards (funnel, retención D1/D7/D30, cohort) definidos
- [ ] `docs/analytics/tracking-plan.md` generado
- [ ] Eventos existentes importados, no redefinidos

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (sin secrets hardcodeados), T: PASS (events tipados) |
| Artifacts | `analytics/events.*`, `docs/analytics/tracking-plan.md`, `.env.example` |
| Next Recommended | Ver tabla de flujo |
| Risks | Provider sandbox sin verificar en runtime, si aplica |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Analytics listo, implementar loops de crecimiento | `/growth-loops` (importa events existentes) |
| Analytics listo, optimizar conversión de pricing | `/pricing-strategy` |
| Analytics listo, medir SEO/tráfico | `/seo-foundations` |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Naming convention canónica (anti double-counting)

`analytics/events.*` es la fuente de verdad de eventos. `/growth-loops` IMPORTA y extiende este archivo (agrega `share_initiated`, `referral_sent`, etc.), nunca lo redefine. Esto evita double-counting en los funnels cuando ambos skills corren sobre el mismo proyecto.

### Por qué tipado estricto

El tipado estricto de propiedades por evento previene eventos con datos inconsistentes (el mayor enemigo de un analytics confiable). Un `signup` siempre con `{ method, plan }` tipados es analizable; uno con propiedades libres, no.
