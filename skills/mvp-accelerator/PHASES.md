# MVP Accelerator — Phases Detail

> Sub-archivo de `/mvp-accelerator`. Detalle de transiciones entre fases, estado inter-fase y recovery completo.

---

## Estado inter-fase

El orquestador mantiene el estado de cada componente conforme avanza:

```
Estado inicial:
  auth:     PENDING
  payments: PENDING
  deploy:   PENDING
  landing:  PENDING

Después de Fase 2 (auth):
  auth:     COMPLETE | BLOCKED
  payments: PENDING
  deploy:   PENDING
  landing:  PENDING

... y así sucesivamente
```

---

## GATE IN detallado por transición

| De Fase | A Fase | Condición GATE IN |
|---------|--------|-------------------|
| 1 → 2 | pre-check → auth | Proyecto inicializado ✓, nombre definido ✓, usuario confirmó ✓ |
| 2 → 3 | auth → payments | auth-in-one-command Status: COMPLETE |
| 3 → 4 | payments → deploy | payments-in-one-command Status: COMPLETE |
| 4 → 5 | deploy → landing | deploy-in-one-command Status: COMPLETE |
| 5 → 6 | landing → summary | landing-page-generate ejecutado (cualquier estado) |

> **Regla**: Si el GATE IN de una transición NO se cumple porque el sub-skill anterior está BLOCKED,
> el orquestador PAUSA en esa fase y espera confirmación del usuario antes de continuar.
> No aborta — el usuario puede resolver el blocker y confirmar para reanudar.

---

## IF FAILS — Recovery completo

### auth-in-one-command BLOCKED

```
⚠️ Paso 1 — Login necesita atención

Causa probable: [mostrar el motivo que retornó auth-in-one-command]

Opciones:
A) Si tu proyecto usa un framework diferente → ejecutar /auth-scaffold directamente
B) Si ya tenés auth pero está incompleta → confirmame qué parte falta y la completo
C) Si querés omitir el login por ahora → puedo pasar directo a los pagos (no recomendado)

¿Qué hacemos?
```

### payments-in-one-command BLOCKED

```
⚠️ Paso 2 — Pagos necesitan atención

Causa probable: [mostrar el motivo que retornó payments-in-one-command]

Opciones:
A) Si no tenés cuenta de Stripe todavía → crear en stripe.com (gratis, 5 minutos)
B) Si ya tenés cuenta pero no encontramos el package → ejecutar: npm install stripe
C) Si querés usar Lemonsqueezy en lugar de Stripe → decime y lo configuro

¿Qué hacemos?
```

### deploy-in-one-command BLOCKED

```
⚠️ Paso 3 — Deploy necesita atención

Causa probable: [mostrar el motivo que retornó deploy-in-one-command]

Opciones:
A) Sin repositorio en GitHub → crear en github.com/new y hacer el primer push
B) Sin CLI de la plataforma → instalar vercel/railway/fly CLI y autenticarse
C) Sin dominio todavía → el deploy funciona igual, el dominio lo agregás después

¿Qué hacemos?
```

### landing-page-generate BLOCKED

```
⚠️ Paso 4 — Landing page necesita atención

La landing es opcional para lanzar. El MVP funciona sin ella.
Opciones:
A) Completar ahora: decime el nombre y tagline de tu producto
B) Hacerlo después: podés ejecutar /landing-page-generate cuando estés listo

¿Qué hacemos?
```

---

## FINAL CHECKPOINT detallado

| Check | Criterio |
|-------|---------|
| auth ejecutado | auth-in-one-command fue invocado (cualquier estado) |
| payments ejecutado | payments-in-one-command fue invocado (cualquier estado) |
| deploy ejecutado | deploy-in-one-command fue invocado (cualquier estado) |
| landing ejecutado | landing-page-generate fue invocado (cualquier estado) |
| summary generado | Estado de los 4 componentes documentado y comunicado |
| sin secrets en chat | Ninguna fase solicitó tokens ni API keys |

> **MVP COMPLETO** = los 4 componentes en Status: COMPLETE
> **MVP PARCIAL** = al menos 1 componente en BLOCKED (documentado en summary)
