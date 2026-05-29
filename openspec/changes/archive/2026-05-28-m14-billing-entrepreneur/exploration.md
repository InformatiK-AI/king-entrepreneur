# Exploration — M14 Billing & Pro Landing

> Fase: sdd-explore · Change: m14-billing-entrepreneur

## Hallazgos

### H1 — PHASE ROUTER de payments-in-one-command

`payments-in-one-command/SKILL.md` Fase 2 (PHASE ROUTER, líneas 109-112) despacha por provider:
`stripe` → `STRIPE.md`, `lemonsqueezy` → `LEMONSQUEEZY.md`. El modo `king-billing` se añade como un **caso
especial de Stripe** (suscripciones recurrentes para King Pro): cuando se invoca con `--mode king-billing`,
el router carga `STRIPE-SUBSCRIPTIONS.md` además de/en lugar de `STRIPE.md`. **Extensión aditiva**: se añade una
entrada al despacho, sin tocar las existentes.

### H2 — env-setup (Fase 4)

La Fase 4 (env-setup) genera `.env.example` con placeholders (`sk_live_REPLACE_WITH_YOUR_VALUE`, etc., línea
169). D5 añade 3 vars del modo king-billing: `STRIPE_PRICE_ID_PRO`, `STRIPE_PRICE_ID_TEAM`,
`STRIPE_CUSTOMER_PORTAL_URL` — todas como placeholders. Extensión aditiva a la lista de vars.

### H3 — landing-page-generate como base de king-pro-landing

`landing-page-generate/SKILL.md` tiene 5 fases (detect-target → inject-design → collect-content →
generate-landing → performance-check) y despacha a `HTML.md`/`REACT.md`/`VUE.md`. `king-pro-landing` NO
duplica esa lógica: la **orquesta**. Precede la generación con fases de producto (pricing data, feature matrix,
testimonios de framework) y usa `landing-page-generate` para la generación base, luego inyecta la pricing table.

### H4 — Anatomía v1.0 del plugin

Los skills de king-entrepreneur son **version: 1.0** (más simple que king-core v2.0): Knowledge Injection,
QUICK REFERENCE (BLOCKING/ABSOLUTE/REQUIRED/PHASES OVERVIEW), línea CASTLE, Fases con
GATE IN/MUST DO/CHECKPOINT/IF FAILS, FINAL CHECKPOINT, Execution Summary, Fase N+1 (Write Session),
Tabla de Flujo. **NO** hay Phase N+2 separada. `king-pro-landing` sigue este estilo (coherencia con el huésped).

### H5 — Contrato cross-repo (CRÍTICO)

El handler `checkout.session.completed` MUST escribir la observation `king-framework/license` con EXACTAMENTE
el esquema de `king-core/knowledge/universal/license-management.md`:
`{ tier, key, activated_at, expires_at, seats, email }`, `topic_key: king-framework/license`, `type: policy`,
`scope: project`. license-check (king-core) lo lee. Cualquier divergencia rompe el flujo pago→activación.

### H6 — Seguridad de webhooks (S-PAY)

El skill base ya exige `stripe.webhooks.constructEvent` (firma obligatoria) y prohíbe keys en chat / `.env` con
valores reales. Los 3 webhooks nuevos del modo king-billing heredan estas reglas sin excepción.

## Decisiones que entran a design/spec

- `king-billing` es un modo de Stripe (no un provider nuevo) → extensión aditiva al ROUTER.
- El contrato de la observation se referencia textual desde king-core (no se redefine).
- `king-pro-landing` orquesta `landing-page-generate`, no lo duplica.
- Firma de webhook obligatoria en los 3 handlers nuevos.
