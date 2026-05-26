# Payments — Lemonsqueezy Integration Guide

> Sub-archivo de `/payments-in-one-command`. Cargado por PHASE ROUTER cuando provider=lemonsqueezy.
> Guía la generación de código de Fase 3 del skill padre.

---

## Inicialización del cliente

```ts
// lib/lemonsqueezy.ts
import { lemonSqueezySetup } from '@lemonsqueezy/lemonsqueezy-js';

if (!process.env.LEMONSQUEEZY_API_KEY) {
  throw new Error('LEMONSQUEEZY_API_KEY is not set');
}

lemonSqueezySetup({ apiKey: process.env.LEMONSQUEEZY_API_KEY });
```

---

## Webhook handler (firma HMAC-SHA256 obligatoria)

```ts
// app/api/webhooks/lemonsqueezy/route.ts (Next.js App Router)
import crypto from 'crypto';

export async function POST(req: Request) {
  const body = await req.text();
  const sig = req.headers.get('x-signature');

  if (!sig || !process.env.LEMONSQUEEZY_WEBHOOK_SECRET) {
    return new Response('Missing signature or secret', { status: 400 });
  }

  const expectedSig = crypto
    .createHmac('sha256', process.env.LEMONSQUEEZY_WEBHOOK_SECRET)
    .update(body)
    .digest('hex');

  if (!crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expectedSig))) {
    return new Response('Invalid signature', { status: 400 });
  }

  const payload = JSON.parse(body);
  const eventName = payload.meta?.event_name;

  switch (eventName) {
    case 'order_created':
      // Activar acceso del usuario
      break;
    case 'subscription_cancelled':
      // Desactivar acceso del usuario
      break;
    default:
      break;
  }

  return new Response('OK', { status: 200 });
}
```

> **REQUERIMIENTO**: La verificación HMAC-SHA256 con `timingSafeEqual` es obligatoria.
> Sin ella el endpoint acepta cualquier payload.

---

## Checkout URL

```ts
import { createCheckout } from '@lemonsqueezy/lemonsqueezy-js';

const { data } = await createCheckout(
  process.env.LEMONSQUEEZY_STORE_ID!,
  process.env.LEMONSQUEEZY_VARIANT_ID!,
  {
    checkoutOptions: { embed: false },
    checkoutData: {
      email: user.email,
      custom: { user_id: user.id },
    },
    productOptions: {
      redirectUrl: `${process.env.NEXT_PUBLIC_APP_URL}/success`,
    },
  }
);

const checkoutUrl = data?.data.attributes.url;
```

---

## Variables de entorno (.env.example)

```
LEMONSQUEEZY_API_KEY=REPLACE_WITH_YOUR_VALUE
LEMONSQUEEZY_WEBHOOK_SECRET=REPLACE_WITH_YOUR_VALUE
LEMONSQUEEZY_STORE_ID=REPLACE_WITH_YOUR_VALUE
LEMONSQUEEZY_VARIANT_ID=REPLACE_WITH_YOUR_VALUE
NEXT_PUBLIC_APP_URL=https://tu-dominio.com
```

---

## Instrucciones para el usuario (lenguaje de negocio)

Al completar la configuración de Lemonsqueezy, mostrar:

```
Para activar los pagos necesitás:
1. Ir a app.lemonsqueezy.com → Settings → API
   Crear una API key y pegarla como LEMONSQUEEZY_API_KEY
2. En Settings → Webhooks → Add webhook
   URL: https://tu-dominio.com/api/webhooks/lemonsqueezy
   Eventos: order_created, subscription_cancelled
   Copiar el "Signing secret" como LEMONSQUEEZY_WEBHOOK_SECRET
3. Ir a tu Store → copiar el Store ID como LEMONSQUEEZY_STORE_ID
4. Ir a tu producto → copiar el Variant ID como LEMONSQUEEZY_VARIANT_ID
```
