---
name: lead-magnets
version: 1.0
api_version: 1.0.0
description: "Genera formularios de captura de leads embebibles en la landing. Providers: Tally (default), Typeform, Formspree. Output: componente React/HTML con UTM tracking, GDPR consent obligatorio, anti-spam honeypot e integración al CRM via webhook."
model: sonnet
---

# /lead-magnets — Captura de Leads para tu Landing

Skill que genera un formulario de captura embebible en tu landing page. El formulario trackea de dónde viene cada visitante (UTM), pide el consentimiento legal obligatorio para guardar sus datos (GDPR), filtra bots automáticamente y envía cada nuevo contacto a tu CRM. Es el paso que llena tu base de contactos antes de conectarla con tu CRM.

## Knowledge Injection

Leer los siguientes archivos ANTES de la Fase 1. Si un archivo no existe, registrar una advertencia y continuar — aplica graceful degradation.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio por persona, progressive disclosure | Yes | framework |
| `knowledge/_inject/sales-crm-patterns.md` | Modelo de datos del Contact, campos `utm_*`, contexto de leads (custom: este skill alimenta el CRM) | No | framework |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El usuario rechaza o no acepta incluir el checkbox de GDPR consent → NO generar formulario, mostrar "El consent GDPR es obligatorio", no escribir ningún archivo
- [ ] El usuario no puede indicar a qué CRM/destino enviar los leads (ni siquiera "lo conecto después") → pedir contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA generar un formulario sin el checkbox de GDPR consent obligatorio (opt-in con timestamp + IP)
- NUNCA marcar el checkbox de consent como pre-tildado o como opcional — debe ser una acción explícita del usuario final
- NUNCA poner el webhook secret del CRM en el componente cliente (ni vía variables del bundle: las vars del cliente son públicas) — el secret vive SOLO server-side; el formulario postea a un endpoint propio del backend
- NUNCA confiar el filtrado anti-spam (honeypot) ni la captura de `consent_ip` al cliente — ambos se evalúan/sellan server-side; el honeypot CSS del cliente es solo defensa en profundidad
- NUNCA mostrar un error visible al bot cuando el honeypot se dispara — el descarte es silencioso

### REQUIRED OUTPUTS
- [ ] Componente del formulario (`src/components/LeadForm.{tsx,jsx,html}` según stack) con UTM tracking, GDPR consent obligatorio, honeypot oculto y POST al webhook del CRM
- [ ] `docs/leads/gdpr-consent-log.md` con el protocolo de registro de consentimiento (qué se guarda, dónde, por cuánto tiempo)
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (collect-config) → Phase 2 (generate-form)
             → Phase 3 (integrate-utm) → Phase 4 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

> **S (Security)**: GDPR consent verificado (BLOQUEA si falta), honeypot anti-spam, secrets solo desde `.env`.
> **T (Testing)**: estructura verificable de los 3 escenarios (consent obligatorio, UTM trackeados, honeypot descarta).

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/lead-magnets` es standalone si no hay workflow activo.

---

## Fase 1: collect-config

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Iniciar con contexto y propósito en lenguaje de negocio:
   ```
   Vamos a crear el formulario que captura los contactos de tu landing.
   Cada persona que lo complete va directo a tu base de contactos (CRM),
   con la fuente de dónde vino y su permiso legal para guardar sus datos.
   ```
2. [ ] Detectar el provider de formulario. Default: **Tally**. Alternativas: **Typeform**, **Formspree**. Preguntar solo si el usuario quiere cambiar el default:
   - "Por defecto uso Tally (gratis y rápido). ¿Lo dejamos así o preferís Typeform o Formspree?"
3. [ ] Preguntar de forma conversacional (de a una, no todas juntas):
   - "¿Qué datos querés pedir? (mínimo: email; opcional: nombre, empresa)"
   - "¿A qué CRM o destino van los contactos? (HubSpot, Pipedrive, Notion... o 'lo conecto después')"
4. [ ] Confirmar EXPLÍCITAMENTE el GDPR consent en lenguaje de negocio (este es el paso que BLOQUEA):
   ```
   Por ley (GDPR), antes de guardar el email de alguien necesitás su permiso explícito.
   Voy a incluir un checkbox que la persona DEBE marcar, y vamos a registrar
   la fecha/hora y su IP como prueba del consentimiento. Esto no es opcional.
   ¿Confirmás que lo incluimos?
   ```
5. [ ] Si el usuario confirma → continuar. Si rechaza o pide quitarlo → entrar en IF FAILS (BLOCKING).

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Provider de formulario definido (Tally por default)
- [ ] Campos del formulario definidos (email como mínimo)
- [ ] Destino del CRM/webhook identificado (puede ser "conectar después")
- [ ] GDPR consent CONFIRMADO por el usuario (sin esto, no se avanza)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: GDPR consent no confirmado
Cause: El usuario rechaza incluir el checkbox de consentimiento o pide marcarlo como opcional.
Recovery:
  [ ] Option A: Explicar el riesgo de negocio: "Sin el consent, cualquier email que captures es un riesgo de multa regulatoria (hasta 4% de la facturación anual). No puedo generar un formulario que te exponga a eso." Volver a pedir confirmación.
  [ ] Option B: Si el usuario insiste en omitirlo → DETENER. Mostrar "El consent GDPR es obligatorio" y retornar `Status: BLOCKED`. No escribir ningún archivo.
  [ ] Option C: Si el problema es de comprensión legal → ofrecer dejar el flujo de consent con el texto por defecto y que el usuario lo revise con su asesor legal después. El checkbox se incluye igual.

---

## Fase 2: generate-form

### GATE IN
- [ ] Fase 1 completada — GDPR consent confirmado y provider/campos definidos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Detectar el framework del proyecto (React/Vue/HTML) para elegir el formato del componente. Si no se detecta → generar HTML embebible.
2. [ ] **Arquitectura cliente/servidor (CRÍTICO de seguridad)**: el componente cliente NO postea directo al CRM ni porta el webhook secret. Postea a un **endpoint propio del backend** (ruta pública sin secret, ej. `POST /api/leads`). Ese endpoint server-side es quien (a) añade la firma/secret al reenviar al CRM, (b) sella `consent_ip` y `consent_timestamp` autoritativos desde la request, (c) evalúa el honeypot de forma autoritativa. Lo provee `/crm-integrate` (o se deja como placeholder de ruta).
3. [ ] Generar el componente del formulario con TODOS estos elementos:
   - **Campos visibles**: los definidos en Fase 1 (email obligatorio).
   - **Checkbox de GDPR consent**: NO pre-tildado, obligatorio para enviar. Texto claro tipo "Acepto que [Producto] guarde mis datos para contactarme. [Ver política de privacidad]".
   - **Campo honeypot oculto**: input adicional (ej. `name="website"`) escondido con CSS (`position:absolute;left:-9999px`), `tabindex="-1"` + `autocomplete="off"`. Defensa en profundidad — el descarte AUTORITATIVO ocurre server-side.
   - **Lógica de submit**: POST al endpoint propio del backend con email + campos + flag de consent + `consent_timestamp` informativo del cliente. NO incluir secret alguno en el cliente.
4. [ ] **Server-side** (en el endpoint propio / handler de `/crm-integrate`): si el honeypot trae valor → descartar silenciosamente (200, no reenviar al CRM, no error); sellar `consent_ip` (de `X-Forwarded-For` / `req.ip`) y `consent_timestamp` autoritativo; firmar y reenviar al CRM leyendo el secret desde `.env` (NUNCA expuesto al cliente).
5. [ ] NO escribir aún el archivo si quedan dudas de consent — el archivo se persiste en Fase 4 (output).

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Componente generado con email + checkbox GDPR obligatorio (no pre-tildado)
- [ ] Honeypot CSS en el cliente + descarte AUTORITATIVO server-side
- [ ] `consent_ip` y `consent_timestamp` autoritativos sellados server-side (no desde el browser)
- [ ] El componente postea a un endpoint propio (sin secret); el secret del CRM vive solo server-side desde `.env`

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Componente incompleto o con secret hardcodeado
Cause: Falta el honeypot, el consent quedó opcional, o se escribió un valor sensible inline.
Recovery:
  [ ] Option A: Si falta el honeypot o el descarte silencioso → regenerar el bloque de submit con la guardia honeypot antes del POST al CRM.
  [ ] Option B: Si el componente cliente porta el secret (inline o vía var de bundle) → moverlo server-side: el cliente postea al endpoint propio sin secret, y el handler añade la firma desde `.env`.
  [ ] Option C: Si el consent quedó pre-tildado u opcional → corregirlo a obligatorio y no pre-tildado. Es un requisito de la capa S; no se puede relajar.

---

## Fase 3: integrate-utm

### GATE IN
- [ ] Fase 2 completada — componente con consent y honeypot generado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Agregar la captura automática de los 5 parámetros UTM desde la URL al cargar el formulario: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`.
   - Leerlos de `window.location.search` (o equivalente del framework) y guardarlos en campos ocultos del formulario.
   - Si un UTM no está presente en la URL → enviarlo vacío o como `null`, nunca inventar valores.
2. [ ] Incluir los 5 UTM en el payload que el componente envía al endpoint propio del backend, junto a los datos del contacto y el flag de consent. (El `consent_ip` y el `consent_timestamp` autoritativos los sella el servidor, no el cliente.)
3. [ ] Mapear los campos al modelo de datos canónico del Contact (ver `sales-crm-patterns.md`): `email`, `name`, `source`, `utm_source/medium/campaign/content/term`, `consent_timestamp`, `consent_ip`, `created_at`. Nota: `consent_timestamp`, `consent_ip` y `utm_content/term` son extensiones que `/crm-integrate` debe agregar a su field mapping.
4. [ ] Verificar que el payload de ejemplo con `?utm_source=twitter&utm_campaign=launch` incluya `utm_source=twitter` y `utm_campaign=launch` en el body del POST.

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Los 5 UTM se capturan de la URL y viajan en el submission al CRM
- [ ] El flag de consent viaja en el submission; IP + timestamp autoritativos se sellan server-side
- [ ] Mapeo alineado al modelo de Contact canónico
- [ ] Caso de prueba `?utm_source=twitter&utm_campaign=launch` produce esos valores en el payload

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: UTM o consent no llegan al CRM
Cause: Los parámetros no se leen de la URL, o no se incluyen en el body del POST.
Recovery:
  [ ] Option A: Revisar la lectura de `window.location.search` y los campos ocultos — confirmar que se hidratan al montar el componente.
  [ ] Option B: Inspeccionar el body del POST al webhook — agregar los UTM y el consent al objeto enviado si faltan.
  [ ] Option C: Si el provider de formulario (Tally/Typeform/Formspree) no soporta campos ocultos por URL → usar hidden fields nativos del provider y documentar el mapeo en `docs/leads/`.

---

## Fase 4: output

### GATE IN
- [ ] Fase 3 completada — UTM y consent integrados al payload del CRM

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Asegurar que los directorios existen:
   ```bash
   mkdir -p docs/leads/
   ```
2. [ ] Escribir el componente del formulario en `src/components/LeadForm.{tsx,jsx,html}` (según el stack detectado), listo para insertar en la landing.
3. [ ] Escribir `docs/leads/gdpr-consent-log.md` con el protocolo de registro de consentimiento:
   - **Qué se registra**: email, `consent_timestamp` (ISO 8601, sellado server-side), `consent_ip` (capturado server-side desde la request), versión del texto de consent.
   - **Dónde se almacena**: en el CRM como campos del Contact (no en logs de aplicación en texto plano). El sellado de IP/timestamp es responsabilidad del endpoint server-side, no del browser.
   - **Retención y derechos**: cuánto tiempo se guarda y cómo el usuario puede solicitar borrado (derecho al olvido GDPR).
   - **Texto exacto del checkbox** mostrado al usuario final.
4. [ ] Si se referenciaron variables nuevas (webhook URL/secret), agregarlas a `.env.example` con valores placeholder.
5. [ ] Confirmar al usuario en lenguaje de negocio qué se generó, dónde insertarlo, y que el siguiente paso es conectar el CRM:
   ```
   Listo. Tu formulario está en src/components/LeadForm — insertálo en tu landing.
   Cada contacto va a llegar con su fuente (UTM) y su permiso legal registrado.
   El siguiente paso es conectar tu CRM con /crm-integrate para que los reciba.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Componente del formulario escrito en `src/components/LeadForm.*`
- [ ] `docs/leads/gdpr-consent-log.md` creado con el protocolo completo
- [ ] `.env.example` actualizado si se agregaron variables
- [ ] Mensaje de cierre en lenguaje de negocio con el próximo paso (`/crm-integrate`)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pudo escribir el componente o el consent log
Cause: Permisos, path incorrecto o disco lleno.
Recovery:
  [ ] Option A: Verificar permisos del directorio: `ls -ld src/components docs/leads`
  [ ] Option B: Escribir el componente en la raíz del proyecto (`LeadForm.html`) e indicar al usuario dónde moverlo.
  [ ] Option C: Si `docs/leads/` no es escribible → imprimir el protocolo de consent en pantalla para guardado manual. NO marcar COMPLETE sin que el consent log quede registrado en algún lugar.

---

## FINAL CHECKPOINT

- [ ] Componente del formulario generado con GDPR consent obligatorio (no pre-tildado)
- [ ] Campo honeypot oculto presente con descarte silencioso de bots
- [ ] Los 5 UTM + consent (timestamp + IP) viajan en el submission al CRM
- [ ] `docs/leads/gdpr-consent-log.md` creado con el protocolo de registro
- [ ] Sin secrets/API keys/webhook secret hardcodeados — solo referencias a `.env`
- [ ] Si el GDPR consent no fue confirmado → ningún archivo fue escrito y Status = BLOCKED

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (consent + honeypot + secrets desde .env), T: PASS (3 escenarios verificables) |
| Artifacts | `src/components/LeadForm.*`, `docs/leads/gdpr-consent-log.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | BLOCKED si no hay GDPR consent — sin consent no se genera formulario |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

> **Dependencia downstream**: `/lead-magnets` DEBE completarse ANTES de `/crm-integrate` — genera los formularios que alimentan el CRM.

| Condición | Próximo Skill |
|-----------|---------------|
| Formulario generado (COMPLETE) | `/crm-integrate` — conectar el CRM que recibe los leads |
| Consent rechazado (BLOCKED) | Re-ejecutar `/lead-magnets` aceptando el GDPR consent obligatorio |
| Falta el provider del CRM | Definir destino y re-ejecutar `/lead-magnets` |
| Formulario listo, falta tráfico | `/seo-foundations` o `/growth-loops` para traer visitantes |

---

## REFERENCE

### Providers de formulario

| Provider | Default | Notas |
|----------|:-------:|-------|
| Tally | ✓ | Gratis, soporta hidden fields por URL para UTM, embebible. Recomendado para arrancar. |
| Typeform | – | UX conversacional, plan pago para volumen. Soporta hidden fields. |
| Formspree | – | Endpoint de captura para formularios HTML propios. Útil si querés control total del markup. |

### Anatomía del honeypot anti-spam

El honeypot es un campo de formulario invisible para humanos pero visible para bots que rellenan todo automáticamente. Reglas:

1. Nombre que tiente al bot (`website`, `company_url`, `phone2`).
2. Ocultarlo con CSS (`position:absolute;left:-9999px`), NUNCA con `type="hidden"` (los bots lo detectan).
3. `tabindex="-1"` y `autocomplete="off"` para que un humano nunca lo toque por accidente.
4. El descarte AUTORITATIVO es server-side: el endpoint que recibe la submission, si el honeypot trae valor → responde 200 y no reenvía al CRM (un bot que postea directo al endpoint igual queda filtrado). El honeypot CSS del cliente es defensa en profundidad, no la única guardia.

### GDPR consent — por qué bloquea

Capturar y almacenar el email de una persona sin su consentimiento explícito viola el GDPR (y leyes equivalentes como CCPA). La sanción puede llegar al 4% de la facturación anual global. Por eso el consent NO es opcional en este skill: es la capa S del CASTLE. El checkbox debe ser una acción afirmativa del usuario (no pre-tildado), y se registra timestamp + IP como prueba auditable.

### Integraciones

- **Upstream**: `/landing-page-generate` — el formulario se inserta en la landing generada.
- **Downstream**: `/crm-integrate` (M-88a) — provee el endpoint server-side (`/api/leads` → CRM) que recibe el POST del formulario: añade la firma con el secret desde `.env`, sella `consent_ip`/`consent_timestamp`, evalúa el honeypot y reenvía al CRM. El componente cliente NUNCA porta el secret. `/crm-integrate` debe extender su field mapping con `consent_timestamp`, `consent_ip` y los 5 UTM.
- **Knowledge**: `sales-crm-patterns.md` define el modelo de datos del Contact (`utm_*`, `lead_score`, `source`) al que se mapean los campos del formulario.
