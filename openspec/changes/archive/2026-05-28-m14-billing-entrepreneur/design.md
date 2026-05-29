# Design — M14 Billing & Pro Landing

> Fase: sdd-design · Change: m14-billing-entrepreneur
> Fuente de verdad: `mejora/planes-detallados/M14-business-model-monetization.md` §2 M-96/M-97.

## Decisiones de arquitectura

### D1 — king-billing como modo de Stripe, no provider nuevo

El modo `king-billing` extiende el provider Stripe con suscripciones recurrentes. NO es un provider nuevo
(no rompe la dicotomía stripe/lemonsqueezy del router). Se carga `STRIPE-SUBSCRIPTIONS.md` cuando el usuario
invoca `--mode king-billing`. Razón: reusar toda la infraestructura Stripe existente (cliente, firma de
webhook) y solo añadir lo específico de suscripciones.

### D2 — El webhook es el escritor del contrato de licencia (cross-repo)

`checkout.session.completed` es el extremo que **escribe** la observation `king-framework/license`;
`license-check` (king-core) es el que la **lee**. El esquema es la única fuente de acoplamiento entre los dos
repos y vive congelado en `king-core/.../license-management.md`. `STRIPE-SUBSCRIPTIONS.md` lo referencia
textualmente — NO lo redefine. Esto evita drift (riesgo R1).

```json
// Lo que el webhook escribe = lo que license-check lee
{
  "topic_key": "king-framework/license",
  "type": "policy", "scope": "project",
  "content": { "tier": "pro", "key": "...", "activated_at": "...", "expires_at": "...", "seats": 1, "email": "..." }
}
```

### D3 — Tres webhooks, un ciclo de vida de licencia

| Webhook | Efecto en la licencia |
|---------|----------------------|
| `checkout.session.completed` | CREA la observation (tier del metadata) + envía email con key de activación |
| `customer.subscription.updated` | ACTUALIZA el tier (upgrade/downgrade) |
| `customer.subscription.deleted` | EXPIRA la observation (`expires_at` al pasado) + email de churn |

Los tres validan firma con `stripe.webhooks.constructEvent` ANTES de procesar (riesgo S-PAY-1). Un evento sin
firma válida → HTTP 400, sin tocar Engram.

### D4 — king-pro-landing orquesta, no duplica

`king-pro-landing` reusa `landing-page-generate` como motor de generación (Fase 3). Lo precede con fases de
producto (pricing data, feature matrix, testimonios) y lo sucede con la inyección de la pricing table y la
verificación de links de checkout. Razón: DRY — la lógica de detección de framework y generación
HTML/React/Vue ya existe y está probada; duplicarla sería deuda.

### D5 — Seguridad: sin secrets, links verificados

- `.env.example` solo placeholders (`STRIPE_PRICE_ID_PRO=price_REPLACE_WITH_YOUR_VALUE`, etc.).
- Firma de webhook obligatoria en los 3 handlers.
- La landing verifica que los CTA de checkout NO sean 404/localhost/placeholders (Fase 5).
- Ningún ejemplo contiene keys reales.

## CASTLE

`_·_·S·T·_·_` — coherente con el skill base. S: firma de webhook + sin secrets. T: links de checkout
verificados + contrato de observation consistente.

## Alternativas descartadas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| `king-billing` como provider nuevo (3er sub-archivo paralelo a stripe/ls) | Duplica la infra Stripe; king-billing ES Stripe con suscripciones |
| Duplicar la generación de landing en king-pro-landing | Deuda técnica; landing-page-generate ya resuelve framework detection + generación |
| Redefinir el esquema de observation en STRIPE-SUBSCRIPTIONS.md | Drift garantizado con king-core; se referencia textual |
| Webhook sin validación de firma "para simplificar el ejemplo" | Vulnerabilidad crítica (activación de licencias falsas); firma SIEMPRE obligatoria |
