---
name: king-pro-landing
version: 1.0
api_version: 1.0.0
description: "Genera la landing page de conversión de King Pro: Hero + social proof + feature matrix + pricing table de 3 tiers (Core/Pro/Team) + FAQ + Footer, con CTAs que apuntan a Stripe Checkout. Orquesta landing-page-generate (no duplica su lógica) y le añade la pricing data y la verificación de links de checkout. Usar cuando se necesite la landing de suscripción de King Pro. M14 M-96."
---

# /king-pro-landing — Landing de Conversión King Pro

Genera una landing page específica para convertir visitantes en suscriptores King Pro. A diferencia de una
landing genérica, ésta incluye pricing table con 3 tiers, comparativa de features y checkout integrado que
redirige a Stripe. **Reutiliza** `landing-page-generate` como motor de generación (detección de framework +
generación HTML/React/Vue) y lo precede con fases de producto (pricing data, testimonios) y lo sucede con la
inyección de la pricing table y la verificación de links de checkout.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/design-essentials.md` | Design tokens S13, paleta y tipografía de marca | No (graceful) | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio para copy de conversión | Yes | framework |

> Los tiers y precios oficiales (Core $0 / Pro $29 / Team $99) son la fuente de la pricing table. Fuente de
> verdad del pricing: `king-core/knowledge/universal/business-model.md` (§2 tiers). Si no es accesible en
> runtime, usar los valores oficiales embebidos en este skill (Fase 1).

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] No se puede detectar el framework del proyecto destino Y el usuario no puede especificarlo
- [ ] No hay price IDs de Stripe ni URLs de checkout configurables (la pricing table quedaría sin CTA funcional) y el usuario no acepta generar con placeholders marcados

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA generar inserción directa de HTML no sanitizado con variables dinámicas (heredado de landing-page-generate)
- NUNCA hardcodear price IDs reales ni secret keys en la landing — usar variables de entorno / URLs de checkout configurables
- NUNCA dejar CTAs de checkout apuntando a `localhost`, `#`, o placeholders sin marcarlos explícitamente como pendientes
- NUNCA duplicar la lógica de generación de `landing-page-generate` — orquestarla

### REQUIRED OUTPUTS
- [ ] Landing King Pro generada con: Hero + social proof + feature matrix + pricing table (3 tiers) + FAQ (5) + Footer
- [ ] Pricing table con 3 cards (Core $0 / Pro $29 / Team $99), cada una con sus features y CTA
- [ ] CTAs de Pro/Team apuntando a URLs de Stripe Checkout (verificadas, no 404/localhost)
- [ ] Archivo guardado en el proyecto (`public/landing/king-pro/` o equivalente del framework)
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (load-pricing-data) → Phase 2 (collect-testimonials)
             → Phase 3 (generate-landing via landing-page-generate)
             → Phase 4 (inject-pricing-table) → Phase 5 (verify-checkout-links)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

> S: sin hardcodeo de price IDs/keys; T: links de checkout verificados (no 404).

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/king-pro-landing` es standalone si no hay workflow activo.

---

## Fase 1: load-pricing-data

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Cargar la pricing data oficial de King (de `king-core/knowledge/universal/business-model.md` §2 si accesible; si no, usar estos valores oficiales embebidos):
   - **King Core** — $0/mes — 5 skills core + 3 agentes + CASTLE S+T + Chronicle
   - **King Pro** — $29/mes — King Core + 40+ skills + Jarvis Mode + /brand-identity + soporte prioritario
   - **King Team** — $99/mes (hasta 5 devs) — King Pro × equipo + CASTLE 6 capas + Engram first-class + AI Audit Ledger
2. [ ] Resolver las URLs de checkout: leer `STRIPE_CUSTOMER_PORTAL_URL` y los checkout endpoints del proyecto (generados por `/payments-in-one-command --mode king-billing`). Si no existen, marcar los CTA como pendientes con placeholder explícito
3. [ ] Construir la feature matrix Core vs Pro vs Team vs Enterprise (qué incluye cada tier)

### CHECKPOINT
- [ ] `PRICING_TIERS[]` cargado (Core/Pro/Team con precio + features)
- [ ] `CHECKOUT_URLS{}` resuelto (o marcado pendiente con placeholder explícito)
- [ ] `FEATURE_MATRIX` construido

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pudo cargar la pricing data o las URLs de checkout
Cause: business-model.md no accesible y/o `/payments-in-one-command --mode king-billing` no corrió aún.
Recovery:
  [ ] Option A: usar los valores oficiales embebidos (paso 1) para el pricing
  [ ] Option B: si no hay URLs de checkout, generar los CTA con placeholder `{{STRIPE_CHECKOUT_URL_PRO}}` y avisar al usuario que corra `/payments-in-one-command --mode king-billing` primero

---

## Fase 2: collect-testimonials

### GATE IN
- [ ] Fase 1 completada — pricing data cargada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Preguntar en lenguaje de negocio si hay social proof disponible:
   - "¿Tenés logos/quotes de early adopters o números de tracción para mostrar?"
2. [ ] Si hay → registrar con delimitadores `<!-- USER_INPUT_START -->` ... `<!-- USER_INPUT_END -->`
3. [ ] Si NO hay → usar placeholders de social proof (sección presente con copy "Trusted by builders" + slots vacíos) — la sección NO se omite, se deja lista para llenar

### CHECKPOINT
- [ ] Social proof recopilado o placeholders definidos (sección siempre presente)

### IF FAILS
ERROR: Usuario no responde sobre social proof
Cause: no hay early adopters aún.
Recovery:
  [ ] Continuar con placeholders — esta sección es graceful, no bloquea la generación

---

## Fase 3: generate-landing (orquesta landing-page-generate)

### GATE IN
- [ ] Fase 2 completada — contenido base recopilado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Invocar `landing-page-generate`** como motor de generación base (NO reimplementar su lógica):
   - Le provee: nombre ("King Framework"), tagline ("Build production-ready software in hours, not weeks"),
     CTA primario ("Start free"), CTA secundario ("See pricing")
   - `landing-page-generate` detecta el framework (HTML/React/Vue), inyecta el design system (S13 o defaults)
     y genera la estructura base (Hero + Features + CTA + Footer)
2. [ ] Recibir el path del archivo base generado por `landing-page-generate`

### CHECKPOINT
- [ ] `landing-page-generate` invocado y completado (no se duplicó su lógica)
- [ ] Estructura base generada con Hero + CTA + Footer
- [ ] `LANDING_PATH` recibido

### IF FAILS
ERROR: landing-page-generate no disponible o falló
Cause: skill no instalado o framework no detectable.
Recovery:
  [ ] Option A: verificar que `landing-page-generate` está instalado en king-entrepreneur
  [ ] Option B: si el framework no se detecta, dejar que landing-page-generate caiga a HTML puro (su fallback)

---

## Fase 4: inject-pricing-table

### GATE IN
- [ ] Fase 3 completada — landing base generada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Inyectar en la landing base las secciones específicas de King Pro:
   - **Feature matrix**: tabla comparativa Core vs Pro vs Team vs Enterprise
   - **Pricing table**: 3 cards (Core $0 / Pro $29 / Team $99), cada una con sus features y un CTA:
     - Core → "Start free"
     - Pro → "Get Pro" → `CHECKOUT_URLS.pro`
     - Team → "Get Team" → `CHECKOUT_URLS.team`
   - **FAQ**: 5 preguntas (BSL, cancelación, reembolso, equipos, enterprise) — fuente: business-model.md FAQ
2. [ ] Usar el escape automático del framework para todo contenido dinámico (sin innerHTML crudo)
3. [ ] Mantener la paleta/tipografía que `landing-page-generate` ya aplicó (no romper el design system)

### CHECKPOINT
- [ ] Pricing table con 3 tiers visibles, cada uno con features + CTA
- [ ] Feature matrix Core/Pro/Team/Enterprise presente
- [ ] FAQ con 5 preguntas presente
- [ ] Sin inserción de HTML sin sanitizar; price IDs no hardcodeados

### IF FAILS
ERROR: No se pudo inyectar la pricing table
Cause: estructura de la landing base inesperada.
Recovery:
  [ ] Localizar el contenedor principal de la landing y añadir las secciones como bloques independientes antes del Footer

---

## Fase 5: verify-checkout-links

### GATE IN
- [ ] Fase 4 completada — pricing table inyectada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar que cada CTA de checkout (Pro/Team) apunta a una URL válida:
   - NO `localhost`, NO `#`, NO `{{PLACEHOLDER}}` sin marcar
   - Si las URLs reales no están disponibles, los CTA deben tener un placeholder EXPLÍCITO y un aviso al usuario
2. [ ] Verificar que above-the-fold (viewport 1440px) muestra el Hero + CTA visible sin scroll
3. [ ] Confirmar al usuario en lenguaje de negocio:
   ```
   ¡Tu landing de King Pro está lista en [LANDING_PATH]!
   Tiene la tabla de precios (Core/Pro/Team) con botones que llevan al checkout.
   Acordate de configurar tus price IDs de Stripe (corré /payments-in-one-command --mode king-billing si no lo hiciste).
   ```

### CHECKPOINT
- [ ] CTAs de Pro/Team verificados (no 404/localhost/placeholder sin marcar)
- [ ] Hero + CTA above-the-fold
- [ ] Confirmación en lenguaje de negocio enviada

### IF FAILS
ERROR: CTAs apuntan a URLs inválidas
Cause: price IDs / checkout URLs no configurados.
Recovery:
  [ ] Marcar los CTA con placeholder explícito (`{{STRIPE_CHECKOUT_URL_PRO}}`) y documentar en el reporte que falta correr `/payments-in-one-command --mode king-billing`. La landing se entrega igualmente con el aviso.

---

## FINAL CHECKPOINT

- [ ] Landing King Pro generada con Hero + social proof + feature matrix + pricing table (3 tiers) + FAQ (5) + Footer
- [ ] Pricing table con Core $0 / Pro $29 / Team $99, cada uno con features + CTA
- [ ] CTAs de Pro/Team a URLs de Stripe Checkout (verificadas o placeholder explícito)
- [ ] `landing-page-generate` orquestado (lógica no duplicada)
- [ ] Sin price IDs/keys hardcodeados; sin HTML sin sanitizar
- [ ] Archivo guardado en el proyecto

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (sin secrets, CTAs verificados), T: PASS (links no 404) |
| Artifacts | Landing King Pro en `public/landing/king-pro/` o equivalente del framework |
| Next Recommended | Ver tabla de flujo |
| Risks | CTAs con placeholder si no se corrió payments king-billing; o None |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Landing lista pero falta billing | `/payments-in-one-command --mode king-billing` (configurar Stripe subscriptions + price IDs) |
| Landing + billing listos | `/deploy-in-one-command` para publicar |
| Parte de un lanzamiento completo | `/mvp-accelerator` para orquestar todo |
| Falta identidad visual | `/brand-identity` (si está disponible) antes de regenerar la landing |
