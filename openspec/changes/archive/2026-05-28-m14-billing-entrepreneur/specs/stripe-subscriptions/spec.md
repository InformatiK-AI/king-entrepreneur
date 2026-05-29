# stripe-subscriptions — Delta Spec (M-97)

> Reusa Gherkin de `mejora/planes-detallados/M14-business-model-monetization.md §7 M-97`.

## ADDED Requirements

### Requirement: Billing de suscripciones King con sincronización de licencia

El modo `king-billing` de `payments-in-one-command` MUST generar Stripe Checkout en modo `subscription`,
customer portal, y 3 webhook handlers que sincronizan la observation `king-framework/license` en Engram. Todos
los handlers MUST validar la firma con `stripe.webhooks.constructEvent` antes de procesar. El esquema de la
observation escrita MUST ser idéntico al que `license-check` (king-core) lee.

#### Scenario: Webhook activación post-pago
- **GIVEN** un request POST con firma Stripe válida
- **AND** el evento es `checkout.session.completed`
- **AND** metadata contiene `{ tier: "pro", email: "user@example.com" }`
- **WHEN** el webhook handler lo procesa
- **THEN** crea observation `king-framework/license` en Engram del customer con el esquema de king-core
- **AND** retorna HTTP 200
- **AND** registra el evento en logs (sin exponer la key)

#### Scenario: Webhook sin firma válida rechazado
- **GIVEN** un request POST sin header `Stripe-Signature`
- **WHEN** el handler lo recibe
- **THEN** retorna HTTP 400 inmediatamente
- **AND** no procesa el evento
- **AND** no modifica ningún estado en Engram

#### Scenario: Suscripción cancelada revoca licencia
- **GIVEN** existe observation `king-framework/license` con tier "pro"
- **AND** llega webhook `customer.subscription.deleted` con firma válida
- **WHEN** el handler lo procesa
- **THEN** actualiza la observation marcando `expires_at` en el pasado
- **AND** el próximo `/license status` muestra licencia expirada

#### Scenario: Upgrade/downgrade de suscripción
- **GIVEN** llega webhook `customer.subscription.updated` con firma válida
- **WHEN** el handler lo procesa
- **THEN** actualiza el `tier` en la observation (upgrade o downgrade)

#### Scenario: Variables de entorno solo placeholders
- **GIVEN** el modo king-billing generó `.env.example`
- **WHEN** se lo inspecciona
- **THEN** contiene `STRIPE_PRICE_ID_PRO`, `STRIPE_PRICE_ID_TEAM`, `STRIPE_CUSTOMER_PORTAL_URL`
- **AND** todos con placeholders (`REPLACE_WITH_YOUR_VALUE`), ningún valor real

### Requirement: Extensión aditiva al PHASE ROUTER

La extensión a `payments-in-one-command/SKILL.md` MUST ser aditiva — los modos `stripe` y `lemonsqueezy`
existentes MUST permanecer intactos.

#### Scenario: El router conserva los providers existentes
- **GIVEN** el `payments-in-one-command/SKILL.md` extendido
- **WHEN** se hace `git diff`
- **THEN** muestra solo contenido añadido (despacho a `STRIPE-SUBSCRIPTIONS.md` para modo king-billing)
- **AND** las entradas `stripe` → `STRIPE.md` y `lemonsqueezy` → `LEMONSQUEEZY.md` permanecen sin cambios
