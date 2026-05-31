---
name: crm-integrate
version: 1.0
api_version: 1.0.0
description: "Integra tu CRM en un comando. Auto-detecta o configura el provider (HubSpot default, Pipedrive, Attio, Salesforce, Notion CRM), genera el field mapping canónico y un webhook handler con verificación de firma por provider. Un lead del formulario llega al CRM en menos de 30 segundos."
model: sonnet
---

# /crm-integrate — Conectá tu CRM en un comando

Configura la integración con tu CRM para que cada lead capturado en tus formularios aparezca automáticamente en el CRM en menos de 30 segundos. Detecta o configura el provider, define un esquema de datos único y consistente, y genera un webhook handler que verifica la firma de cada notificación antes de procesarla — para que nadie pueda inyectar datos falsos.

## Knowledge Injection

Leer los siguientes archivos ANTES de la Fase 1. Si un archivo no existe, registrar una advertencia y continuar — aplica graceful degradation.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio por persona para los mensajes al founder | Yes | framework |
| `knowledge/_inject/sales-crm-patterns.md` | CRM data model canónico, lead scoring, funnel (custom: este skill define el field mapping del CRM) | No | framework |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar.

> **Nota — `.king/knowledge/secrets-provider.md`**: NO es un archivo de inyección de conocimiento. Es un GATE de seguridad BLOQUEANTE verificado en la Fase 0. Si no existe, el skill se DETIENE (ver BLOCKING CONDITIONS y Fase 0).

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] No existe `.king/knowledge/secrets-provider.md` (prerequisito de M07) → BLOQUEAR con: "Configurá tu secrets provider antes de integrar el CRM" — NO continuar hasta que el archivo EXISTA (verificado con `test -f`). La confirmación verbal del founder NO basta: debe materializarse el archivo.
- [ ] El usuario pega una API key o token real del CRM en el chat → DETENER y pedir que lo configure en `.env`
  Patrones a detectar (escanear en cada mensaje del usuario):
  - HubSpot: `pat-na1-`, `pat-eu1-`, tokens de 36+ caracteres en contexto de `HUBSPOT_*`
  - Pipedrive: `PIPEDRIVE_API_TOKEN`, strings de 40 caracteres hex
  - Attio / Salesforce / Notion: strings largos en contexto de `*_API_KEY`, `*_TOKEN`, `*_WEBHOOK_SECRET`, `secret_` (Notion)
  Acción al detectar: DETENER — NO persistir el valor en ningún artefacto — mostrar: "Ese token no debe compartirse en el chat. Configuralo en tu archivo `.env` local."

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA continuar sin verificar que existe `.king/knowledge/secrets-provider.md` (es BLOQUEO real, no advertencia)
- NUNCA solicitar tokens, API keys ni webhook secrets del CRM en el chat
- NUNCA hardcodear secrets/API keys/tokens — solo referenciarlos desde `.env`
- NUNCA generar el webhook handler sin verificación de firma específica del provider
- NUNCA generar `.env` con valores reales — solo `.env.example` con `KEY=REPLACE_WITH_YOUR_VALUE`
- NUNCA sobreescribir una integración de CRM existente sin confirmación del usuario

### REQUIRED OUTPUTS
- [ ] Field mapping canónico documentado en `docs/crm/field-mapping.md` (`contact_id`, `email`, `name`, `company`, `source`, `utm_source/medium/campaign`, `created_at`, `lead_score`)
- [ ] Webhook handler en `/api/crm/webhook` con verificación de firma por provider (no omitible)
- [ ] Sync de leads formulario → CRM en < 30 segundos + webhook handler entrante con firma verificada
- [ ] `.env.example` con placeholders del provider seleccionado
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load + GATE secrets-provider) → Phase 1 (select-provider)
             → Phase 2 (configure-field-mapping) → Phase 3 (setup-webhook)
             → Phase 4 (test-sync) → Phase 5 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

> **S** — webhook con firma verificada por provider, secrets nunca hardcodeados.
> **T** — test de integración contra sandbox: lead de formulario aparece en el CRM en < 30 seg.

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/crm-integrate` es standalone si no hay workflow activo.

### GATE IN
- [ ] Conocimiento inyectado (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Verificar secrets provider (BLOQUEO crítico)** — comprobar que existe `.king/knowledge/secrets-provider.md`:
   ```bash
   test -f .king/knowledge/secrets-provider.md && echo "OK" || echo "MISSING"
   ```
2. [ ] Si el archivo NO existe → DETENER el skill y mostrar al founder en lenguaje de negocio:
   ```
   Configurá tu secrets provider antes de integrar el CRM.

   Tu CRM maneja datos de tus usuarios y requiere tokens de acceso.
   Para evitar exponer esas credenciales por accidente, primero configurá
   tu secrets provider con /tenancy-setup (módulo de seguridad M07, vive en el plugin king-infra).

   Cuando esté listo, volvé a ejecutar /crm-integrate.
   ```
   NO avanzar a la Fase 1 hasta que el founder confirme explícitamente que el provider está configurado.
3. [ ] Verificar que `/lead-magnets` ya corrió (existe `docs/leads/` o un componente de formulario generado). Si no hay formularios → advertir: "Todavía no tenés formularios que alimenten el CRM. Ejecutá /lead-magnets primero."

### CHECKPOINT
- [ ] `.king/knowledge/secrets-provider.md` existe (re-verificado con `test -f`; la confirmación verbal NO sustituye al archivo)
- [ ] Si faltan formularios / lead magnets → advertencia mostrada (no bloquea; `/lead-magnets` es el upstream recomendado)
- [ ] Conocimiento de dominio cargado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Secrets provider ausente
Cause: No existe `.king/knowledge/secrets-provider.md` (M07 no ejecutado).
Recovery:
  [ ] Option A: Redirigir al founder a `/tenancy-setup` (M07) para configurar el secrets provider, y NO continuar — re-ejecutar `/crm-integrate` después
  [ ] Option B: Si el founder confirma que ya tiene un secrets provider externo equivalente, documentarlo y pedir que cree `.king/knowledge/secrets-provider.md` describiéndolo antes de continuar
  [ ] Option C: Si no hay forma de resolverlo → `Status: BLOCKED`, surface al usuario. NO producir ningún output.

---

## Fase 1: select-provider

### GATE IN
- [ ] Fase 0 completada — secrets provider verificado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Buscar en `package.json` y `.env.example` señales del provider ya en uso:
   - `@hubspot/api-client` o `HUBSPOT_*` → HubSpot
   - `pipedrive` o `PIPEDRIVE_*` → Pipedrive
   - `attio` o `ATTIO_*` → Attio
   - `jsforce` / `@salesforce/*` o `SALESFORCE_*` → Salesforce
   - `@notionhq/client` o `NOTION_*` → Notion CRM
2. [ ] Si no hay señal, preguntar al founder con opciones en lenguaje de negocio (HubSpot es el default recomendado):
   ```
   ¿Qué CRM querés conectar?
   A) HubSpot      — el más popular, plan gratis generoso (recomendado)
   B) Pipedrive    — simple y orientado a ventas
   C) Attio        — moderno y flexible
   D) Salesforce   — para equipos grandes / Enterprise
   E) Notion CRM   — si ya usás Notion como base de datos
   ```
3. [ ] Si ya existe una integración de CRM parcial → informar: "Ya tenés [provider] conectado parcialmente. Voy a completar lo que falta." NO sobreescribir sin confirmación.

### CHECKPOINT
- [ ] Provider seleccionado: `hubspot` | `pipedrive` | `attio` | `salesforce` | `notion`
- [ ] Estado previo documentado (none | partial | complete)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Provider no seleccionado
Cause: Respuesta ambigua o fuera de las opciones.
Recovery:
  [ ] Option A: Re-preguntar con contexto: "¿En qué CRM ya tenés tus contactos hoy?" — usar la respuesta para elegir
  [ ] Option B: Si el founder no tiene ninguno → recomendar HubSpot (plan gratis) y continuar con ese default

---

## Fase 2: configure-field-mapping

### GATE IN
- [ ] Fase 1 completada — provider seleccionado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Definir el field mapping canónico — el esquema único que viaja del formulario al CRM. Usar el CRM data model de `sales-crm-patterns.md` como base:

   | Campo canónico | Tipo | Origen | Property en el CRM |
   |----------------|------|--------|--------------------|
   | `contact_id` | string | generado por el CRM | id nativo |
   | `email` | string | formulario | email |
   | `name` | string | formulario | firstname/lastname |
   | `company` | string | formulario | company |
   | `source` | string | lead magnet | lead_source |
   | `utm_source` | string | querystring | utm_source |
   | `utm_medium` | string | querystring | utm_medium |
   | `utm_campaign` | string | querystring | utm_campaign |
   | `created_at` | datetime | servidor | createdate |
   | `lead_score` | number | scoring inicial | hs_lead_score / equivalente |

2. [ ] Mapear cada campo canónico a la property nativa del provider seleccionado (los nombres varían entre HubSpot, Pipedrive, Attio, Salesforce y Notion — usar la nomenclatura del provider).
3. [ ] Asegurar que el directorio existe y escribir el schema:
   ```bash
   mkdir -p docs/crm/
   ```
   Escribir `docs/crm/field-mapping.md` con la tabla canónica + el mapeo específico del provider.
4. [ ] Explicar al founder en lenguaje de negocio:
   ```
   Definí cómo viajan los datos de cada lead desde tu formulario hasta tu CRM:
   email, nombre, empresa, de dónde vino (campaña, canal) y su puntaje inicial.
   Esto mantiene tu CRM ordenado y te deja medir qué canal trae mejores leads.
   ```

### CHECKPOINT
- [ ] `docs/crm/field-mapping.md` creado con los 10 campos canónicos
- [ ] Cada campo canónico mapeado a la property nativa del provider
- [ ] Incluye los 3 campos UTM (`utm_source`, `utm_medium`, `utm_campaign`)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pudo escribir el field mapping
Cause: Directorio `docs/crm/` no se puede crear, o property del provider desconocida.
Recovery:
  [ ] Option A: Crear el archivo en `docs/crm-field-mapping.md` directamente, o imprimirlo en pantalla para guardado manual
  [ ] Option B: Si una property nativa no existe en el CRM del founder → documentarla como custom property a crear en el dashboard del CRM e indicarlo en el archivo

---

## Fase 3: setup-webhook

### GATE IN
- [ ] Fase 2 completada — field mapping documentado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar el webhook handler en `/api/crm/webhook` (ruta según el stack: `app/api/crm/webhook/route.ts`, `pages/api/crm/webhook.ts`, etc.).
2. [ ] El handler DEBE verificar la firma de cada notificación **según el provider** — este paso NO es omitible:
   - **HubSpot**: validar el header `X-HubSpot-Signature-v3` (HMAC-SHA256 sobre método + URI + body + timestamp, usando el client secret)
   - **Pipedrive**: validar Basic Auth del webhook o el token configurado
   - **Attio**: validar el header de firma HMAC con el webhook secret
   - **Salesforce**: validar la firma del Platform Event / Outbound Message
   - **Notion**: validar el token del integration / verification token del webhook
3. [ ] El handler usa el field mapping de la Fase 2 para transformar el payload entrante al esquema canónico antes de procesarlo.
4. [ ] Configurar el sync de salida: cuando un lead llega del formulario (vía `/lead-magnets`), enviarlo al CRM usando el token desde `.env` — objetivo de latencia < 30 segundos.
5. [ ] Crear o actualizar `.env.example` con placeholders del provider (sin valores reales):
   - HubSpot: `HUBSPOT_ACCESS_TOKEN=REPLACE_WITH_YOUR_VALUE`, `HUBSPOT_CLIENT_SECRET=REPLACE_WITH_YOUR_VALUE`
   - (equivalentes `*_API_TOKEN`, `*_WEBHOOK_SECRET` para los demás providers)
   Verificar que `.env` y `.env.local` están en `.gitignore`.
6. [ ] Explicar al founder en lenguaje de negocio:
   ```
   Generé el conector que recibe avisos de tu CRM y el que manda cada
   lead nuevo a tu CRM. Antes de procesar cualquier aviso, el sistema
   verifica que sea legítimo — así nadie puede inyectar contactos falsos.
   ```

### CHECKPOINT
- [ ] Webhook handler creado en `/api/crm/webhook`
- [ ] Verificación de firma presente y específica del provider seleccionado
- [ ] Sync de salida formulario → CRM configurado (token desde `.env`)
- [ ] `.env.example` con solo placeholders; `.gitignore` verificado
- [ ] Sin secrets/tokens hardcodeados en ningún archivo generado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Webhook sin verificación de firma
Cause: El código generado no incluye la validación de firma del provider.
Recovery:
  [ ] Option A: Regenerar el handler incluyendo la validación de firma obligatoria del provider — sin ella NO se considera completo
  [ ] Option B: Si el provider no documenta firma de webhook (caso raro) → exigir secret compartido en el header y validarlo, nunca aceptar el webhook sin autenticación
  [ ] Option C: Si no se puede generar para el stack → documentar los archivos a crear manualmente con las instrucciones de firma. NO marcar COMPLETE sin firma.

---

## Fase 4: test-sync

### GATE IN
- [ ] Fase 3 completada — webhook handler con firma generado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Ejecutar el test de integración contra el **sandbox / cuenta de prueba** del CRM (nunca producción):
   - Simular un submission del formulario con datos de prueba + parámetros UTM
   - Verificar que el contacto aparece en el CRM con los campos canónicos correctos
   - Medir la latencia desde el submission hasta la aparición en el CRM
2. [ ] Verificar los S-checks de seguridad:
   - S-CRM-1: Sin tokens/secrets hardcodeados ✓
   - S-CRM-2: Webhook handler verifica la firma del provider ✓
   - S-CRM-3: `.env.example` solo tiene placeholders ✓
3. [ ] Confirmar el criterio de aceptación: **el lead aparece en el CRM en menos de 30 segundos**.
4. [ ] Reportar al founder el resultado del test en lenguaje de negocio:
   ```
   Probé la conexión: envié un lead de prueba desde tu formulario y
   apareció en [provider] en {N} segundos. Todo funcionando.
   ```

### CHECKPOINT
- [ ] Test contra sandbox ejecutado (no producción)
- [ ] Lead de prueba aparece en el CRM con todos los campos canónicos (incluido UTM)
- [ ] Latencia medida < 30 segundos
- [ ] S-CRM-1, S-CRM-2, S-CRM-3 verificados

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El lead no llegó al CRM o tardó > 30 seg
Cause: Token inválido en `.env`, firma rechazada, field mapping incorrecto, o sync lento.
Recovery:
  [ ] Option A: Si la firma se rechaza → revisar que `*_WEBHOOK_SECRET` en `.env` coincide con el configurado en el dashboard del CRM
  [ ] Option B: Si el contacto llega incompleto → volver a la Fase 2 y corregir el mapeo de la property faltante
  [ ] Option C: Si tarda > 30 seg → revisar el path de sync (cola, reintentos) y optimizar; documentar como riesgo si no se resuelve en la sesión

---

## Fase 5: output

### GATE IN
- [ ] Fase 4 completada — test de sync pasó

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar que `docs/crm/field-mapping.md` está completo y refleja el provider final usado.
2. [ ] Confirmar al founder el cierre de la integración:
   ```
   ¡Listo! Tu CRM ([provider]) ya está conectado.
   Cada lead que entre por tus formularios va a aparecer automáticamente
   en menos de 30 segundos, con su origen y campaña.

   Acordate de tener tus tokens cargados en el archivo .env antes de salir a producción.
   Siguiente paso: /email-sequences para nutrir esos leads automáticamente.
   ```
3. [ ] Listar los artefactos generados (handler, field mapping, `.env.example`).

### CHECKPOINT
- [ ] `docs/crm/field-mapping.md` final verificado
- [ ] Confirmación en lenguaje de negocio enviada
- [ ] Próximo paso indicado (`/email-sequences`)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Falta un artefacto requerido al cerrar
Cause: El field mapping o el handler no se generaron correctamente.
Recovery:
  [ ] Option A: Identificar el artefacto faltante y volver a la fase que lo produce
  [ ] Option B: Si no se puede completar → `Status: PARTIAL`, listar lo pendiente y el path para retomar

---

## FINAL CHECKPOINT

- [ ] `.king/knowledge/secrets-provider.md` verificado en Fase 0 (BLOQUEO resuelto)
- [ ] `docs/crm/field-mapping.md` creado con los 10 campos canónicos
- [ ] Webhook handler en `/api/crm/webhook` con verificación de firma del provider (no omitible)
- [ ] Sync formulario → CRM probado contra sandbox en < 30 segundos
- [ ] S-CRM-1, S-CRM-2, S-CRM-3 verificados
- [ ] Sin tokens/API keys solicitados ni persistidos en el chat

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS |
| Artifacts | Webhook handler `/api/crm/webhook` + `docs/crm/field-mapping.md` + `.env.example` |
| Next Recommended | Ver tabla de flujo |
| Risks | None si S-CRM checks PASS y sync < 30 seg. BLOCKED si falta secrets-provider.md |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

> **Origen requerido**: `← /lead-magnets` — debe estar COMPLETE antes (formularios que alimentan el CRM). Si no hay formularios, este skill advierte en la Fase 0.

| Condición | Próximo Skill |
|-----------|---------------|
| CRM conectado, nutrir los leads | `/email-sequences` |
| CRM conectado, diseñar el funnel de ventas / lead scoring | `/sales-funnel-design` |
| Falta secrets provider (M07) | `/tenancy-setup` (king-infra) y luego re-ejecutar `/crm-integrate` |
| Faltan formularios que alimenten el CRM | `/lead-magnets` y luego re-ejecutar `/crm-integrate` |

---

## REFERENCE

### Providers soportados

| Provider | Default | Verificación de firma de webhook | Nota |
|----------|:-------:|----------------------------------|------|
| HubSpot | ✓ | `X-HubSpot-Signature-v3` (HMAC-SHA256 con client secret) | Plan gratis generoso, recomendado |
| Pipedrive | – | Basic Auth / token de webhook | Orientado a ventas |
| Attio | – | HMAC con webhook secret | Moderno y flexible |
| Salesforce | – | Firma de Platform Event / Outbound Message | Enterprise |
| Notion CRM | – | Verification token del integration | Si ya usa Notion como DB |

### Field mapping canónico (esquema de referencia)

`contact_id` · `email` · `name` · `company` · `source` · `utm_source` · `utm_medium` · `utm_campaign` · `created_at` · `lead_score`

Detalle completo y el mapeo a properties nativas del provider en `docs/crm/field-mapping.md` (generado en la Fase 2). El CRM data model canónico (Contact / Company / Deal / Activity) vive en `knowledge/_inject/sales-crm-patterns.md`.

### Por qué la firma del webhook es obligatoria (capa S)

Un webhook sin verificación de firma es una puerta abierta: cualquiera que conozca la URL `/api/crm/webhook` podría inyectar contactos falsos, manipular lead scores o disparar automatizaciones. Verificar la firma del provider en cada request garantiza que la notificación viene realmente del CRM. Por eso este paso NO es omitible y se valida en S-CRM-2.

### Integraciones con otros skills del stack V3

- **Aguas arriba** — `/lead-magnets` genera los formularios con UTM tracking y GDPR consent que alimentan este CRM.
- **Aguas abajo** — `/email-sequences` consume los leads sincronizados para nutrirlos; `/sales-funnel-design` usa el `lead_score` para disparar notificaciones al cruzar el threshold SQL.
