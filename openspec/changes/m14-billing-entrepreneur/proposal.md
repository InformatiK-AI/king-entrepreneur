# Proposal — M14 Billing & Pro Landing (Entrepreneur)

> Fase: sdd-propose · Change: m14-billing-entrepreneur · Backend: openspec · Milestone: M14 (mitad king-entrepreneur)

## Why

Es la **mitad king-entrepreneur** del milestone M14. La mitad king-core (`m14-licensing-core`) ya está en
develop (@ 931282b): define `license-check` y el contrato de la observation `king-framework/license`. Este
cambio cierra el círculo: el **billing** que cobra suscripciones recurrentes con Stripe y, al recibir el pago,
**escribe la observation** que `license-check` (king-core) lee — más la **landing King Pro** que convierte
visitantes en suscriptores apuntando a Stripe Checkout.

**BLOCKED-BY**: `king-core m14-licensing-core`. El handler `checkout.session.completed` debe escribir
exactamente el esquema que `license-check` lee; ese contrato vive en
`king-core/knowledge/universal/license-management.md` y NO debe divergir.

## What Changes

- **Extensión** del skill `payments-in-one-command` con un modo `king-billing`: suscripciones recurrentes
  (no solo one-time), customer portal, y 3 webhook handlers que sincronizan la licencia en Engram.
- **Skill nuevo** `king-pro-landing` que orquesta `landing-page-generate` para producir una landing de
  conversión con pricing table de 3 tiers y checkout integrado.

Fuente de verdad: `mejora/planes-detallados/M14-business-model-monetization.md` (§2 M-96/M-97, §6 Bloques D·E, §7 Gherkin).

## Capabilities (contrato para sdd-spec)

| # | Capability | Item(s) | Artefactos |
|---|------------|---------|------------|
| 1 | `stripe-subscriptions` | M-97 | `payments-in-one-command/STRIPE-SUBSCRIPTIONS.md` (nuevo) + extensiones aditivas a `payments-in-one-command/SKILL.md` (ROUTER king-billing + .env vars) |
| 2 | `king-pro-landing` | M-96 | skill `skills/king-pro-landing/SKILL.md` + command `commands/king-pro-landing.md` |

**Total**: 1 skill nuevo, 1 command nuevo, 1 sub-archivo nuevo, 2 extensiones aditivas.

## Scope

- **In scope**: `STRIPE-SUBSCRIPTIONS.md` (checkout mode subscription + customer portal + 3 webhooks);
  extensión aditiva al PHASE ROUTER (Fase 2) y a env-setup (Fase 4) de `payments-in-one-command/SKILL.md`;
  skill `king-pro-landing` (5 fases que orquestan `landing-page-generate`) + su command. Anatomía v1.0 (estilo
  del plugin huésped). Contrato de la observation idéntico al de king-core.
- **Out of scope**:
  - Despliegue real de un webhook Stripe en vivo / cuenta Stripe real (los skills GENERAN el código/guidance;
    no configuran Stripe en producción en este cambio).
  - Ejecución de `king-pro-landing` contra un proyecto externo (M14 entrega el SKILL.md generador, no una
    landing renderizada).
  - El endpoint de validación de keys (vive en king-core / infra externa).

## Affected modules

`king-entrepreneur/skills/payments-in-one-command/` (1 sub-archivo nuevo + SKILL.md extendido),
`king-entrepreneur/skills/king-pro-landing/` (nuevo), `king-entrepreneur/commands/king-pro-landing.md` (nuevo),
`king-entrepreneur/.gitignore` (añadir `.worktrees/` y `.king/`).

## Delivery

- **single-pr** con `size:exception`. Worktree `feature/m14-billing-entrepreneur` desde develop → un `/merge`.
- Depende de que `m14-licensing-core` esté en develop de king-core (GATE cross-repo) — satisfecho.

## Rollback plan

- Trabajo aislado en el worktree/branch `feature/m14-billing-entrepreneur`. Abortar antes del merge:
  `git worktree remove` + borrar branch → develop intacto.
- Las 2 extensiones a `payments-in-one-command/SKILL.md` son **aditivas** (verificable por git diff): revertir =
  quitar las líneas añadidas. El skill conserva sus modos `stripe`/`lemonsqueezy` intactos.
- `STRIPE-SUBSCRIPTIONS.md`, `king-pro-landing/` y el command son archivos nuevos — revertir = borrarlos.
