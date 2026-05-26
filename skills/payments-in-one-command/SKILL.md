---
name: payments-in-one-command
version: 1.0
description: "Configura sistema de pagos en un comando. Auto-detecta Stripe o Lemonsqueezy. Genera inicialización, webhook con firma obligatoria y .env.example. Usable independientemente de otros skills S12."
---

# /payments-in-one-command — Pagos para Entrepreneur

Configura el sistema de pagos de tu app SaaS en un comando. Detecta automáticamente si usás Stripe o Lemonsqueezy, genera el código de integración con validación de webhooks incluida, y crea el `.env.example` con placeholders seguros.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/payments-essentials.md` | Stripe/Lemonsqueezy patterns, S-PAY checks, webhook security | Yes | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio para mensajes | Yes | framework |

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] No se puede detectar el stack del proyecto Y el usuario no puede especificarlo
- [ ] El usuario pega una API key real en el chat → DETENER y pedir que la configure en `.env`
  Patrones a detectar (escanear en cada mensaje del usuario):
  - Stripe: `sk_live_`, `sk_test_`, `pk_live_`, `pk_test_`, `whsec_`, `rk_live_`, `rk_test_`
  - Lemonsqueezy: strings de 40+ caracteres alfanuméricos en contexto de variable de pago (LEMONSQUEEZY_API_KEY, LEMONSQUEEZY_WEBHOOK_SECRET)
  Acción al detectar: DETENER inmediatamente — NO persistir la key en ningún artefacto — mostrar: "Esa clave no debe compartirse en el chat. Configurala en tu archivo `.env` local."

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA solicitar `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` ni equivalentes en el chat
- NUNCA generar código de webhook sin validación de firma (`stripe.webhooks.constructEvent` para Stripe, HMAC-SHA256 para Lemonsqueezy)
- NUNCA generar `.env` con valores reales — solo `.env.example` con `KEY=REPLACE_WITH_YOUR_VALUE`
- NUNCA sobreescribir configuración de pagos existente sin confirmación del usuario

### REQUIRED OUTPUTS
- [ ] Código de inicialización del provider en el proyecto del usuario
- [ ] Webhook handler con validación de firma (no omitible)
- [ ] `.env.example` con placeholders
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (detect-provider) → Phase 2 (PHASE ROUTER)
             → Phase 3 (generate-config) → Phase 4 (env-setup) → Phase 5 (verify-setup)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/payments-in-one-command` es standalone si no hay workflow activo.

---

## Fase 1: detect-provider

### GATE IN
- [ ] Knowledge injection completada (payments-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Buscar en `package.json` dependencias de pagos:
   - `stripe` → Stripe detectado
   - `@lemonsqueezy/lemonsqueezy-js` o `lemonsqueezy-node` → Lemonsqueezy detectado
   - Ambas → preguntar al usuario cuál usar
   - Ninguna → preguntar con opciones en lenguaje de negocio:
     ```
     ¿Qué sistema de pagos querés usar?
     A) Stripe — el más popular para SaaS internacionales
     B) Lemonsqueezy — más simple, incluye manejo de IVA automático
     ```
2. [ ] Si el proveedor ya está parcialmente configurado:
   - Detectar si existe webhook handler, cliente inicializado, o `.env.example` con vars de pagos
   - Si existe → informar: "Ya tenés [Stripe/LS] configurado parcialmente. Voy a completar lo que falta."
   - No sobreescribir lo existente sin confirmación

### CHECKPOINT
- [ ] Provider identificado: `stripe` | `lemonsqueezy`
- [ ] Estado previo documentado (none | partial | complete)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Provider no detectado y usuario no respondió
Cause: Respuesta ambigua o fuera de opciones.
Recovery:
  [ ] Re-preguntar con contexto: "¿Ya tenés una cuenta de Stripe o Lemonsqueezy?" — usar la respuesta para elegir.

---

## Fase 2: PHASE ROUTER

### GATE IN
- [ ] Fase 1 completada — provider identificado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Según el provider, cargar el sub-archivo correspondiente:
   - `stripe` → leer `skills/payments-in-one-command/STRIPE.md`
   - `lemonsqueezy` → leer `skills/payments-in-one-command/LEMONSQUEEZY.md`
2. [ ] El sub-archivo guía la generación de código de la Fase 3

### CHECKPOINT
- [ ] Sub-archivo del provider cargado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Sub-archivo del provider no encontrado
Cause: `STRIPE.md` o `LEMONSQUEEZY.md` no accesible en `skills/payments-in-one-command/`.
Recovery:
  [ ] Verificar que el skill está correctamente instalado. Buscar el sub-archivo con ruta absoluta. Si persiste, continuar usando los patrones de `knowledge/_inject/payments-essentials.md` directamente.

---

## Fase 3: generate-config

### GATE IN
- [ ] Fase 2 completada — sub-archivo del provider disponible

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar archivo de inicialización del cliente de pagos (usando patterns del sub-archivo)
2. [ ] Generar webhook handler con validación de firma — **este paso NO es omitible**
3. [ ] Generar tipos/interfaces si el proyecto usa TypeScript
4. [ ] Informar al usuario en lenguaje de negocio qué se generó:
   ```
   Generé el código para recibir pagos y para que tu app sea
   notificada cuando alguien pague. El sistema verifica que las
   notificaciones son legítimas antes de procesarlas.
   ```

### CHECKPOINT
- [ ] Cliente de pagos inicializado
- [ ] Webhook handler generado con validación de firma presente
- [ ] Sin API keys hardcodeadas en ningún archivo generado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pudo generar código de pagos para el stack
Cause: Stack no soportado o conflicto con estructura del proyecto.
Recovery:
  [ ] Mostrar al usuario qué archivos necesitan crearse manualmente con instrucciones en lenguaje de negocio.

---

## Fase 4: env-setup

### GATE IN
- [ ] Fase 3 completada — código generado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Crear o actualizar `.env.example` con las variables requeridas (placeholders únicamente):
   - Stripe: `STRIPE_SECRET_KEY=sk_live_REPLACE_WITH_YOUR_VALUE`, `STRIPE_PUBLISHABLE_KEY=pk_live_REPLACE_WITH_YOUR_VALUE`, `STRIPE_WEBHOOK_SECRET=whsec_REPLACE_WITH_YOUR_VALUE`
   - Lemonsqueezy: `LEMONSQUEEZY_API_KEY=REPLACE_WITH_YOUR_VALUE`, `LEMONSQUEEZY_WEBHOOK_SECRET=REPLACE_WITH_YOUR_VALUE`, `LEMONSQUEEZY_STORE_ID=REPLACE_WITH_YOUR_VALUE`
2. [ ] Verificar que `.env` y `.env.local` están en `.gitignore`
3. [ ] Instruir al usuario cómo obtener las keys:
   ```
   Para que los pagos funcionen, necesitás agregar tus claves al archivo .env.
   Las encontrás en el Dashboard de [Stripe/Lemonsqueezy] → Developers → API Keys.
   Nunca compartas esas claves con nadie.
   ```

### CHECKPOINT
- [ ] `.env.example` tiene solo placeholders (sin valores reales)
- [ ] `.gitignore` verificado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede actualizar .gitignore
Cause: Archivo no existe o permisos.
Recovery:
  [ ] Crear `.gitignore` con las entradas necesarias. Informar al usuario.

---

## Fase 5: verify-setup

### GATE IN
- [ ] Fase 4 completada — env configurado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Ejecutar S-PAY checks (de payments-essentials.md):
   - S-PAY-1: Sin secrets hardcodeados ✓
   - S-PAY-2: Webhook handler tiene verificación de firma ✓
   - S-PAY-3: `.env.example` solo tiene placeholders ✓
2. [ ] Confirmar al usuario:
   ```
   ¡Listo! Tu app ya puede recibir pagos.
   Acordate de agregar tus claves al archivo .env antes de probar.
   Siguiente paso: /deploy-in-one-command para publicar tu app.
   ```

### CHECKPOINT
- [ ] S-PAY-1, S-PAY-2, S-PAY-3 verificados
- [ ] Confirmación en lenguaje de negocio enviada

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: S-PAY-2 falló — webhook sin verificación de firma
Cause: El código generado no incluye validación de firma.
Recovery:
  [ ] Volver a Fase 3 y regenerar el webhook handler incluyendo la validación obligatoria del sub-archivo del provider.

---

## FINAL CHECKPOINT

- [ ] Provider detectado y código generado
- [ ] Webhook handler con validación de firma (no omitible)
- [ ] `.env.example` con solo placeholders
- [ ] S-PAY-1, S-PAY-2, S-PAY-3 verificados
- [ ] Sin API keys solicitadas ni persistidas en el chat

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS |
| Artifacts | Código de pagos + webhook handler + `.env.example` |
| Next Recommended | Ver tabla de flujo |
| Risks | None si S-PAY checks PASS |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Pagos configurados exitosamente | `/deploy-in-one-command` |
| Provider ya estaba completo | `/deploy-in-one-command` directamente |
| Usuario quiere personalización avanzada | Configurar manualmente según sub-archivo del provider |
