# Tasks — M14 Billing & Pro Landing

> Detalle largo en `mejora/planes-detallados/M14-business-model-monetization.md §6` (Bloques D, E).
> BLOCKED-BY: king-core m14-licensing-core (merged @ 931282b).

## Bloque D — Stripe billing extension (depende de license-check en develop de king-core)
- [x] D1 `payments-in-one-command/STRIPE-SUBSCRIPTIONS.md`: Checkout `mode: subscription` + price IDs + customer portal
- [x] D2 Handler `checkout.session.completed`: crea observation `king-framework/license` en Engram + email con key (esquema == king-core)
- [x] D3 Handlers `customer.subscription.updated` (actualiza tier) y `customer.subscription.deleted` (expira observation + email churn)
- [x] D4 Extender `payments-in-one-command/SKILL.md` Fase 2 (PHASE ROUTER): modo `king-billing` carga `STRIPE-SUBSCRIPTIONS.md` (aditivo)
- [x] D5 Extender `payments-in-one-command/SKILL.md` Fase 4 (.env.example): `STRIPE_PRICE_ID_PRO/TEAM`, `STRIPE_CUSTOMER_PORTAL_URL` (aditivo)

## Bloque E — King Pro landing (depende de D1)
- [x] E1 `skills/king-pro-landing/SKILL.md` Fase 0 + Fase 1 (load-pricing-data)
- [x] E2 Fase 2 (collect-testimonials) + Fase 3 (orquestar landing-page-generate)
- [x] E3 Fase 4 (inject-pricing-table: 3 tiers Core/Pro/Team + comparativa de features)
- [x] E4 Fase 5 (verify-checkout-links) + FINAL CHECKPOINT + Execution Summary
- [x] E5 `commands/king-pro-landing.md`

## Validación
- [x] V1 Contrato de observation en `checkout.session.completed` idéntico a king-core `license-management.md` (capa C — cross-repo) — 9/9 campos
- [x] V2 Los 3 webhooks tienen `stripe.webhooks.constructEvent` (firma obligatoria — capa S)
- [x] V3 `rg` sin keys reales (`sk_live_`/`whsec_`/price IDs reales); `.env.example` solo placeholders
- [x] V4 Extensiones a payments SKILL.md aditivas (+2 líneas funcionales; ver nota EOL en verify-report)
- [x] V5 Frontmatter king-pro-landing válido + anatomía v1.0 (coherente con el plugin)
