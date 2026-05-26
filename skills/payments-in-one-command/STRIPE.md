# Payments — Stripe Integration Guide

> Sub-archivo de `/payments-in-one-command`. Cargado por PHASE ROUTER cuando provider=stripe.
> Guía la generación de código de Fase 3 del skill padre.

---

## Inicialización del cliente

```ts
// lib/stripe.ts
import Stripe from 'stripe';

if (!process.env.STRIPE_SECRET_KEY) {
  throw new Error('STRIPE_SECRET_KEY is not set');
}

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2023-10-16',
});
```

---

## Webhook handler (firma obligatoria)

```ts
// app/api/webhooks/stripe/route.ts (Next.js App Router)
import { stripe } from '@/lib/stripe';
import { headers } from 'next/headers';

export async function POST(req: Request) {
  const body = await req.text();
  const sig = headers().get('stripe-signature');

  if (!sig || !process.env.STRIPE_WEBHOOK_SECRET) {
    return new Response('Missing signature or secret', { status: 400 });
  }

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(body, sig, process.env.STRIPE_WEBHOOK_SECRET);
  } catch {
    return new Response('Invalid signature', { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      // Activar acceso del usuario
      break;
    case 'customer.subscription.deleted':
      // Desactivar acceso del usuario
      break;
    default:
      break;
  }

  return new Response('OK', { status: 200 });
}
```

> **REQUERIMIENTO**: `constructEvent` es obligatorio. Sin esta verificación el endpoint
> acepta cualquier payload y es explotable para fraude.

---

## Checkout session

```ts
// Crear sesión de pago
const session = await stripe.checkout.sessions.create({
  payment_method_types: ['card'],
  mode: 'subscription',
  line_items: [{ price: process.env.STRIPE_PRICE_ID, quantity: 1 }],
  success_url: `${process.env.NEXT_PUBLIC_APP_URL}/success`,
  cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
  customer_email: user.email,
});
```

---

## Customer portal (gestión de suscripción)

```ts
const portalSession = await stripe.billingPortal.sessions.create({
  customer: stripeCustomerId,
  return_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard`,
});
```

---

## Variables de entorno (.env.example)

```
STRIPE_SECRET_KEY=sk_live_REPLACE_WITH_YOUR_VALUE
STRIPE_PUBLISHABLE_KEY=pk_live_REPLACE_WITH_YOUR_VALUE
STRIPE_WEBHOOK_SECRET=whsec_REPLACE_WITH_YOUR_VALUE
STRIPE_PRICE_ID=price_REPLACE_WITH_YOUR_VALUE
NEXT_PUBLIC_APP_URL=https://tu-dominio.com
```

---

## Instrucciones para el usuario (lenguaje de negocio)

Al completar la configuración de Stripe, mostrar:

```
Para activar los pagos necesitás:
1. Ir a dashboard.stripe.com → Developers → API Keys
2. Copiar la "Secret key" y pegarla en tu .env como STRIPE_SECRET_KEY
3. Crear un webhook en Stripe → Developers → Webhooks → Add endpoint
   URL: https://tu-dominio.com/api/webhooks/stripe
   Eventos: checkout.session.completed, customer.subscription.deleted
4. Copiar el "Signing secret" del webhook como STRIPE_WEBHOOK_SECRET
```
