---
name: transactional-email
version: 1.0
api_version: 1.0.0
description: "Genera emails transaccionales listos para producción: bienvenida, reset de contraseña, recibo de pago, renovación y recuperación de conversión. i18n español + inglés, dark mode automático y entregabilidad verificada (DKIM/SPF). Usable junto a /email-sequences y /crm-integrate."
model: sonnet
---

# /transactional-email — Emails Transaccionales para tu Producto

Genera los emails que tu producto envía automáticamente cuando pasa algo importante: alguien se registra, pide recuperar su contraseña, paga, renueva su suscripción o está a punto de irse. Cada template viene en español e inglés, se adapta solo al modo claro y oscuro del lector, y solo se genera cuando tu dominio está configurado para no caer en spam.

## Knowledge Injection

Leer los siguientes archivos ANTES de la Fase 1. Si un archivo no existe, registrar una advertencia y continuar — aplica graceful degradation.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio por persona, progressive disclosure | Yes | framework |
| `knowledge/_inject/sales-crm-patterns.md` | Email deliverability (DKIM/SPF/DMARC), suppression list (custom: este skill verifica entregabilidad) | No | framework |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El dominio de envío NO tiene DKIM ni SPF configurados (registros DNS publicados) → DETENER. Sin entregabilidad verificada, los emails caen en spam. Mostrar instrucciones para el provider DNS del usuario y no generar nada hasta resolverlo.
- [ ] El usuario pega una API key real del proveedor de email en el chat → DETENER y pedir que la configure en `.env`
  Patrones a detectar (escanear en cada mensaje del usuario):
  - Resend: `re_` seguido de 20+ caracteres
  - Postmark / Mailgun: tokens de 32+ caracteres alfanuméricos en contexto de variable de email (`RESEND_API_KEY`, `POSTMARK_SERVER_TOKEN`, `MAILGUN_API_KEY`)
  Acción al detectar: DETENER inmediatamente — NO persistir la key en ningún artefacto — mostrar: "Esa clave no debe compartirse en el chat. Configurala en tu archivo `.env` local."

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA generar un solo template antes de verificar DKIM/SPF — la entregabilidad es prerequisito, no un paso posterior
- NUNCA hardcodear API keys, tokens ni el dominio de envío con secret embebido — todo desde `.env`
- NUNCA generar `.env` con valores reales — solo `.env.example` con `KEY=REPLACE_WITH_YOUR_VALUE`
- NUNCA generar el template `password-reset` sin expiración del link (máximo 1 hora)
- NUNCA omitir la media query `prefers-color-scheme: dark` en ningún template
- NUNCA sobreescribir templates existentes del usuario sin confirmación

### REQUIRED OUTPUTS
- [ ] Templates transaccionales generados en el proyecto (los seleccionados por el usuario)
- [ ] Cada template con versión español + inglés (i18n) y media query dark mode
- [ ] `.env.example` actualizado con las variables del proveedor (solo placeholders)
- [ ] `docs/email/transactional-templates.md` con el inventario de templates y su trigger
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (select-templates) → Phase 2 (configure-provider)
             → Phase 3 (generate-templates) → Phase 4 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

> **S (Security)**: DKIM/SPF verificados antes de generar, sin secrets hardcodeados, link de reset con expiración.
> **T (Testing)**: cada template verificable en modo claro y oscuro; criterio de aceptación visual + entregabilidad (§4 de la estrategia de testing de M08).

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/transactional-email` es standalone si no hay workflow activo.

---

## Fase 1: select-templates

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Iniciar con contexto en lenguaje de negocio:
   ```
   Vamos a crear los correos que tu producto envía solo cuando pasa
   algo importante: alguien se registra, paga, o necesita recuperar
   su contraseña. Vienen en español e inglés y se ven bien en modo
   claro y oscuro.
   ```
2. [ ] Presentar el catálogo de templates en lenguaje de negocio y pedir cuáles necesita (puede elegir varios):
   ```
   ¿Cuáles necesitás? (podés elegir varios)
   A) Bienvenida        — cuando alguien se registra, con los próximos pasos
   B) Recuperar acceso  — link para resetear la contraseña (vence en 1 hora)
   C) Recibo de pago    — confirmación con el detalle de lo que pagó
   D) Renovación        — aviso de renovación automática con la próxima fecha
   E) Recuperación      — carrito abandonado / prueba por vencer (recuperar conversión)
   ```
3. [ ] Mapear la selección del usuario a los templates técnicos:
   - A → `welcome` · B → `password-reset` · C → `receipt` · D → `subscription-renewed` · E → `abandoned-cart` y/o `trial-expiring`
4. [ ] Preguntar por idiomas: por defecto español + inglés. Si necesita idiomas extra, registrarlos (flag `--lang`).
5. [ ] Si ya existen templates transaccionales en el proyecto, detectarlos e informar: "Ya tenés algunos correos creados. Voy a completar los que falten sin pisar los tuyos." — no sobreescribir sin confirmación.

### CHECKPOINT
- [ ] Al menos un template seleccionado y mapeado a su nombre técnico
- [ ] Idiomas definidos (mínimo español + inglés)
- [ ] Estado previo documentado (none | partial | complete)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Selección de templates vacía o ambigua
Cause: El usuario no indicó qué correos necesita o eligió fuera de las opciones.
Recovery:
  [ ] Option A: Re-preguntar con un ejemplo concreto: "Casi todos los productos arrancan con Bienvenida (A) y Recuperar acceso (B). ¿Empezamos por esos?"
  [ ] Option B: Si el usuario sigue sin decidir, proponer el set base `welcome` + `password-reset` y confirmar antes de continuar.

---

## Fase 2: configure-provider

### GATE IN
- [ ] Fase 1 completada — al menos un template seleccionado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar la entregabilidad del dominio ANTES de cualquier generación (gate de seguridad S):
   - Detectar el proveedor de email del proyecto (`package.json`: `resend`, `postmark`, `mailgun.js`). Si ya existe `/email-sequences`, reutilizar su provider SI es uno de los soportados acá (Resend/Postmark/Mailgun); si `/email-sequences` usó Loops, preguntar un provider transaccional de esta lista (Loops no se usa para transaccionales acá).
   - Verificar que el dominio de envío tiene **DKIM y SPF** publicados en DNS. Comprobar registros TXT del dominio (selector DKIM del provider y registro SPF).
   - Si faltan DKIM/SPF → activar BLOCKING CONDITION: DETENER, no generar templates, mostrar instrucciones específicas para el provider DNS del usuario:
     ```
     Antes de enviar correos necesitás verificar tu dominio para no caer en spam.
     En el panel de tu proveedor de email vas a encontrar los registros DKIM y SPF
     que tenés que copiar en tu DNS. Cuando estén publicados, volvé y seguimos.
     ```
2. [ ] Si no hay provider detectado, preguntar en lenguaje de negocio:
   ```
   ¿Con qué servicio vas a enviar los correos?
   A) Resend    — el más simple para empezar
   B) Postmark  — enfocado en transaccionales, alta entregabilidad
   C) Mailgun   — flexible, buen volumen
   ```
3. [ ] Confirmar el dominio remitente (`from`) y el nombre del producto para usar en los templates. No pedir credenciales — solo el dominio público.

### CHECKPOINT
- [ ] Provider de email identificado (o reutilizado de `/email-sequences`)
- [ ] DKIM y SPF verificados como publicados (S-EMAIL gate PASS)
- [ ] Dominio remitente y nombre de producto capturados (sin secrets)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: DKIM/SPF no verificados
Cause: El dominio no tiene los registros DNS publicados o no se pudieron comprobar.
Recovery:
  [ ] Option A: Mostrar los registros DKIM/SPF exactos del provider del usuario y pausar — esperar a que confirme que los publicó, luego re-verificar. Mantener `Status: BLOCKED`; NO generar ningún template con el gate fallido.
  [ ] Option B: Si los registros ya se publicaron pero falta propagación DNS → esperar y re-verificar; mantener `Status: BLOCKED` hasta que la verificación pase. No generar borradores enviables.
  [ ] Option C: Si el usuario ya va a configurar secuencias, `/email-sequences` Fase 1 (configure-provider) verifica el mismo DKIM/SPF; al volver acá el estado se reutiliza. Pero los templates transaccionales se generan SOLO con `/transactional-email` una vez resuelto el DNS.

---

## Fase 3: generate-templates

### GATE IN
- [ ] Fase 2 completada — provider identificado y DKIM/SPF verificados (S-EMAIL gate PASS)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar cada template seleccionado con su contenido y trigger correspondiente:
   - `welcome` — bienvenida post-signup con los próximos pasos del producto (CTA al primer uso)
   - `password-reset` — link de reset con **expiración de 1 hora** (texto explícito de vencimiento, token desde variable, nunca embebido)
   - `receipt` — confirmación de pago con detalle de la transacción (monto, items, fecha, número de referencia)
   - `subscription-renewed` — aviso de renovación automática con la **próxima fecha** de cobro
   - `abandoned-cart` / `trial-expiring` — recuperación de conversión con CTA de retorno
2. [ ] Cada template DEBE incluir:
   - Versión **español + inglés** (i18n) — y los idiomas extra solicitados en Fase 1
   - Media query `@media (prefers-color-scheme: dark)` con colores legibles en fondo oscuro
   - Estructura de email compatible (tablas/inline CSS, no depender de CSS externo)
   - Variables interpoladas desde datos de runtime (nombre, monto, fecha, link) — nunca valores reales hardcodeados
3. [ ] Generar el código de envío del provider que renderiza el template y lo dispara con el `from` configurado, leyendo la API key SOLO desde `process.env`.
4. [ ] Informar al usuario en lenguaje de negocio:
   ```
   Listo los correos. Cada uno se ve bien en modo claro y oscuro, viene
   en español e inglés, y el de recuperar acceso vence en 1 hora por
   seguridad.
   ```

### CHECKPOINT
- [ ] Cada template seleccionado generado con versión español + inglés
- [ ] Cada template tiene la media query `prefers-color-scheme: dark`
- [ ] `password-reset` (si fue seleccionado) tiene expiración de 1 hora explícita
- [ ] Sin API keys ni tokens hardcodeados en ningún archivo generado
- [ ] Código de envío lee la key desde `.env` únicamente

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Un template no cumple un requisito obligatorio (dark mode, i18n o expiración)
Cause: El render generado omitió la media query, un idioma, o la expiración del link de reset.
Recovery:
  [ ] Option A: Regenerar el template específico agregando el requisito faltante — no continuar con templates incompletos.
  [ ] Option B: Si el stack del proyecto no soporta el formato de email generado, mostrar el template como archivo independiente y dar instrucciones de integración en lenguaje de negocio.
  [ ] Option C: Si se detecta un secret hardcodeado, eliminarlo, moverlo a `.env.example` como placeholder y re-verificar el CHECKPOINT.

---

## Fase 4: output

### GATE IN
- [ ] Fase 3 completada — templates generados y verificados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Crear o actualizar `.env.example` con las variables del provider (solo placeholders):
   - Resend: `RESEND_API_KEY=re_REPLACE_WITH_YOUR_VALUE`, `EMAIL_FROM=hola@tudominio.com`
   - Postmark: `POSTMARK_SERVER_TOKEN=REPLACE_WITH_YOUR_VALUE`, `EMAIL_FROM=hola@tudominio.com`
   - Mailgun: `MAILGUN_API_KEY=REPLACE_WITH_YOUR_VALUE`, `MAILGUN_DOMAIN=tudominio.com`, `EMAIL_FROM=hola@tudominio.com`
2. [ ] Verificar que `.env` y `.env.local` están en `.gitignore`.
3. [ ] Asegurar que el directorio existe y escribir el inventario:
   ```bash
   mkdir -p docs/email/
   ```
   `docs/email/transactional-templates.md` con: lista de templates generados, su trigger, idiomas soportados, y nota de dark mode + entregabilidad verificada.
4. [ ] Confirmar al usuario y proponer el siguiente paso:
   ```
   ¡Listo! Tus correos transaccionales están creados y tu dominio está
   verificado para que no caigan en spam.
   Siguiente paso sugerido: /email-sequences para los correos de seguimiento,
   o /crm-integrate para conectar tus leads.
   ```

### CHECKPOINT
- [ ] `.env.example` tiene solo placeholders (sin valores reales)
- [ ] `.gitignore` verificado
- [ ] `docs/email/transactional-templates.md` creado con el inventario
- [ ] Confirmación en lenguaje de negocio enviada

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede escribir el inventario o actualizar `.gitignore`
Cause: Directorio no creable o permisos.
Recovery:
  [ ] Option A: Crear `docs/email-transactional-templates-YYYY-MM-DD.md` directamente, o imprimir el inventario en pantalla para guardado manual.
  [ ] Option B: Si no existe `.gitignore`, crearlo con las entradas `.env` y `.env.local`, e informar al usuario.

---

## FINAL CHECKPOINT

- [ ] DKIM y SPF verificados antes de generar (BLOCKING CONDITION resuelta — S gate PASS)
- [ ] Templates seleccionados generados con español + inglés y dark mode
- [ ] `password-reset` con expiración de 1 hora (si fue seleccionado)
- [ ] `.env.example` con solo placeholders — sin secrets hardcodeados ni solicitados en el chat
- [ ] `docs/email/transactional-templates.md` con inventario y trigger por template

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (DKIM/SPF verificados, sin secrets), T: PASS (dark + light verificable) |
| Artifacts | Templates transaccionales + `.env.example` + `docs/email/transactional-templates.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | None si DKIM/SPF verificados y CHECKPOINTs PASS. BLOCKED si entregabilidad sin resolver. |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Templates generados — falta automatizar seguimiento | `/email-sequences` |
| Templates generados — falta conectar los leads al CRM | `/crm-integrate` |
| DKIM/SPF sin resolver (BLOCKED) | Resolver DNS y re-ejecutar `/transactional-email` (`/email-sequences` Fase 1 verifica el mismo DKIM/SPF) |
| Usuario quiere más idiomas | Re-ejecutar `/transactional-email --lang=<idioma>` |

---

## REFERENCE

> Contexto no accionable. Casos borde e integraciones.

### Relación con otros skills del stack V3 Sales
- **`/email-sequences`**: comparte el gate de entregabilidad (DKIM/SPF; `/email-sequences` además verifica DMARC) y, si coincide, el provider. Los transaccionales son disparados por eventos del producto (signup, pago); las secuencias son drip campaigns de marketing — `/email-sequences` solo reutiliza la verificación DNS, NO genera los transaccionales. Si `/email-sequences` configuró Loops (no soportado acá para transaccionales), este skill pregunta un provider de su lista (Resend/Postmark/Mailgun).
- **`/crm-integrate`**: los datos del contacto (nombre, email) que alimentan los templates suelen venir del CRM. No hay dependencia dura, pero el flujo natural es CRM → datos → emails transaccionales.

### Criterio de testing (M08 §4)
Tipo: **Visual + deliverability**. Criterio de éxito: cada template renderiza correctamente en **modo claro y oscuro**, y **DKIM está verificado** en el dominio de envío. Probar contra el sandbox del provider, nunca producción.

### Catálogo de templates y triggers
| Template | Trigger | Requisito específico |
|----------|---------|----------------------|
| `welcome` | `signup` completado | CTA a primer uso, próximos pasos |
| `password-reset` | solicitud de reset | link con expiración ≤ 1 hora |
| `receipt` | pago confirmado | detalle de transacción (monto, items, fecha) |
| `subscription-renewed` | renovación automática | próxima fecha de cobro visible |
| `abandoned-cart` | carrito sin completar | CTA de retorno |
| `trial-expiring` | prueba por vencer | CTA de conversión + fecha de vencimiento |

### Anti-patrones
- Generar templates "para después configurar el DNS": NO. La entregabilidad es prerequisito de bloqueo, no un TODO.
- Embeber el token de reset en el HTML estático: NO. Siempre variable de runtime con expiración.
- Un solo idioma "porque el usuario habla español": el default es español + inglés; los idiomas extra se agregan, no se quitan los base.
