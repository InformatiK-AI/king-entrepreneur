# Archive Report — M14 Billing & Pro Landing

> Fase: sdd-archive · Change: m14-billing-entrepreneur · Fecha: 2026-05-28 · Veredicto: FORTIFIED

## Resumen

Mitad king-entrepreneur del milestone M14 (Business Model & Monetization). Entrega el billing de suscripciones
recurrentes con Stripe y la landing de conversión King Pro. Cierra M14 junto con `king-core m14-licensing-core`.

## Lineage

| Artefacto | Estado |
|-----------|--------|
| config.yaml (infra SDD nueva en entrepreneur) · proposal · exploration · design · specs (2) · tasks · verify-report | completed |
| Bloques implementados | D (Stripe subscriptions), E (king-pro-landing) |

## Entregables (en develop @ 3b29e10)

- `skills/payments-in-one-command/STRIPE-SUBSCRIPTIONS.md` — checkout `mode: subscription` + customer portal + 3 webhooks (con firma obligatoria) que sincronizan la observation `king-framework/license`
- `skills/payments-in-one-command/SKILL.md` — +modo `king-billing` en PHASE ROUTER + 3 env vars (aditivo)
- `skills/king-pro-landing/SKILL.md` + `commands/king-pro-landing.md` — landing conversión 3 tiers, orquesta `landing-page-generate` (anatomía v1.0)
- `.gitignore` — +`.worktrees/` +`.king/`
- `openspec/` — infra SDD inicializada en king-entrepreneur

## Verificación

- specs: 2/2 COMPLIANT (12 escenarios) · CASTLE C·A·S·T en verde
- **Contrato cross-repo: 9/9 campos del esquema de observation idénticos a king-core `license-management.md`**
- Firma `constructEvent` en los 3 webhooks · secrets scan clean (solo placeholders)
- Nota EOL: `payments SKILL.md` renormalizado CRLF→LF (cumple `.gitattributes eol=lf`); cambio funcional = +2 líneas

## Cierre del milestone M14

| Repo | Cambio | Merge |
|------|--------|-------|
| king-core | m14-licensing-core | 931282b |
| king-entrepreneur | m14-billing-entrepreneur | 3b29e10 |

Flujo end-to-end: pago Stripe → webhook escribe observation `king-framework/license` (mismo esquema) →
`/license-check` (king-core) la lee → skills premium habilitados. Landing King Pro apunta a Stripe Checkout.

## Commits

- Feature: `f3e09de` · Merge a develop: `3b29e10` (pusheado a origin/develop)
