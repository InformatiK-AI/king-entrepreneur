# STRIPE-SUBSCRIPTIONS — Modo king-billing (sub-archivo de payments-in-one-command)

> Sub-archivo cargado por el PHASE ROUTER (Fase 2) cuando se invoca `/payments-in-one-command --mode king-billing`.
> Milestone M14 (M-97). Extiende el provider Stripe con **suscripciones recurrentes**, **customer portal** y
> **3 webhooks** que sincronizan la licencia King con Engram.
> Hereda TODAS las reglas de seguridad del skill base: firma de webhook obligatoria, sin keys en chat,
> `.env.example` solo con placeholders.

---

## 1. Diferencia con el modo Stripe base (`STRIPE.md`)

| Feature base (`STRIPE.md`) | Feature king-billing (este archivo) |
|----------------------------|-------------------------------------|
| Stripe Checkout one-time | Stripe Checkout con `mode: 'subscription'` |
| Webhook básico | + `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted` |
| `.env.example` base | + `STRIPE_PRICE_ID_PRO`, `STRIPE_PRICE_ID_TEAM`, `STRIPE_CUSTOMER_PORTAL_URL` |
| Sin customer portal | Stripe Billing Portal (`stripe.billingPortal.sessions.create`) |

---

## 2. Contrato de la licencia (cross-repo — NO redefinir)

Los webhooks sincronizan la observation `king-framework/license` que **`/license-check` (king-core) lee**. El
esquema es la **única fuente de acoplamiento** entre king-entrepreneur y king-core y vive congelado en
`king-core/knowledge/universal/license-management.md`. Reproducido aquí **solo como referencia** — si cambia,
cambialo en king-core, no acá:

```json
{
  "topic_key": "king-framework/license",
  "type": "policy",
  "scope": "project",
  "content": {
    "tier": "pro",                         // "core" | "pro" | "team" | "enterprise"
    "key": "KF-PRO-XXXX-XXXX",
    "activated_at": "2026-05-28T00:00:00Z",
    "expires_at": "2026-06-28T00:00:00Z",
    "seats": 1,
    "email": "user@example.com"
  }
}
```

> **Cómo llega la licencia al Engram del developer**: Engram es memoria **local** del entorno del developer. Un
> webhook server-side NO escribe directo en el Engram local del cliente. El flujo correcto es:
> 1. El webhook `checkout.session.completed` valida el pago, genera la `key` (`KF-<TIER>-XXXX-XXXX`) y la
>    **envía por email** + la registra en el backend de validación de King.
> 2. El developer ejecuta `/license-check activate <key>` (king-core), que valida la key contra ese backend y
>    hace el `mem_save` de la observation en SU Engram local.
> En setups con Engram compartido/remoto (tier Enterprise, multi-user), el webhook PUEDE escribir la observation
> directamente. El esquema es el mismo en ambos casos.

---

## 3. Stripe Checkout — modo subscription

```ts
// checkout.ts — Stripe Checkout para suscripción King Pro/Team
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);   // NUNCA hardcodear

export async function createSubscriptionCheckout(tier: 'pro' | 'team', email: string) {
  const priceId = tier === 'pro'
    ? process.env.STRIPE_PRICE_ID_PRO!
    : process.env.STRIPE_PRICE_ID_TEAM!;

  return stripe.checkout.sessions.create({
    mode: 'subscription',                                    // ← clave: recurrente, no one-time
    line_items: [{ price: priceId, quantity: 1 }],
    customer_email: email,
    metadata: { tier, email },                               // ← el webhook lee tier/email de acá
    success_url: `${process.env.APP_URL}/welcome?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/pricing`,
  });
}
```

---

## 4. Customer Portal (gestión de plan)

```ts
// portal.ts — el cliente gestiona/cancela su suscripción
export async function createPortalSession(customerId: string) {
  return stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${process.env.APP_URL}/account`,
  });
}
```

> `STRIPE_CUSTOMER_PORTAL_URL` puede apuntar a un portal sin login (Stripe-hosted) como fallback.

---

## 5. Webhook handlers (3 eventos) — firma SIEMPRE obligatoria

> **S-PAY-1 (no negociable)**: validar `stripe.webhooks.constructEvent` ANTES de procesar. Sin firma válida →
> HTTP 400, sin tocar Engram. Esto evita que un actor active licencias falsas.

```ts
// webhook.ts — Express handler para los 3 eventos king-billing
import express from 'express';
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export const webhook = express.Router();

// raw body es obligatorio para verificar la firma
webhook.post('/stripe/king-billing', express.raw({ type: 'application/json' }), async (req, res) => {
  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      req.body,
      req.header('Stripe-Signature')!,                       // sin header → throw → 400
      process.env.STRIPE_WEBHOOK_SECRET!,
    );
  } catch {
    return res.status(400).send('Invalid signature');        // NO se procesa, NO se toca Engram
  }

  switch (event.type) {
    case 'checkout.session.completed': {
      const s = event.data.object as Stripe.Checkout.Session;
      const tier = s.metadata?.tier ?? 'pro';
      const email = s.metadata?.email ?? s.customer_email!;
      const key = generateLicenseKey(tier);                  // KF-<TIER>-XXXX-XXXX
      await registerLicense({                                // backend de validación + envío de email
        tier, key, email,
        activated_at: new Date().toISOString(),
        expires_at: addMonths(new Date(), 1).toISOString(),  // ciclo mensual
        seats: tier === 'team' ? 5 : 1,
      });
      await sendActivationEmail(email, key);                 // el developer activará con /license-check activate <key>
      break;
    }
    case 'customer.subscription.updated': {
      const sub = event.data.object as Stripe.Subscription;
      await updateLicenseTier(sub);                          // upgrade/downgrade → nuevo tier
      break;
    }
    case 'customer.subscription.deleted': {
      const sub = event.data.object as Stripe.Subscription;
      await expireLicense(sub);                              // expires_at al pasado
      await sendChurnEmail(sub);                             // oferta de reactivación
      break;
    }
  }
  res.status(200).json({ received: true });                  // NUNCA loggear la key en claro
});
```

> `registerLicense`/`updateLicenseTier`/`expireLicense` construyen el objeto con el **esquema exacto** de la
> §2. En setups con Engram compartido, estas funciones hacen `mem_save`/`mem_update` de la observation
> `king-framework/license`; en el flujo estándar, registran en el backend de validación que `/license-check
> activate` consultará.

| Evento | Efecto en la licencia |
|--------|----------------------|
| `checkout.session.completed` | CREA licencia (tier del metadata) + email con key |
| `customer.subscription.updated` | ACTUALIZA `tier` (upgrade/downgrade) |
| `customer.subscription.deleted` | EXPIRA (`expires_at` al pasado) + email de churn |

---

## 6. Variables de entorno (Fase 4 — solo placeholders)

Añadir a `.env.example` (NUNCA valores reales):

```
STRIPE_PRICE_ID_PRO=price_REPLACE_WITH_YOUR_VALUE
STRIPE_PRICE_ID_TEAM=price_REPLACE_WITH_YOUR_VALUE
STRIPE_CUSTOMER_PORTAL_URL=https://billing.stripe.com/REPLACE_WITH_YOUR_VALUE
```

Las del modo base (`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, etc.) siguen aplicando.

---

## 7. Checklist de seguridad (S-PAY king-billing)

- [ ] Los 3 webhooks validan firma con `stripe.webhooks.constructEvent` antes de procesar
- [ ] Sin firma válida → HTTP 400, sin modificar Engram/backend
- [ ] La `key` NUNCA se loggea en claro ni se devuelve en la respuesta del webhook
- [ ] `.env.example` solo placeholders (incluidos los 3 nuevos price/portal)
- [ ] El esquema de la observation coincide con `king-core/knowledge/universal/license-management.md`
- [ ] `mode: 'subscription'` (no one-time) para cobro recurrente
