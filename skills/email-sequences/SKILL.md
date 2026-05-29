---
name: email-sequences
version: 1.0
api_version: 1.0.0
description: "Configura secuencias de email automatizadas (drip campaigns) para onboarding. Secuencia de bienvenida de 5 emails, triggers, A/B testing en subject lines y suppression list activa. Verifica DKIM/SPF/DMARC antes de permitir envíos. Providers: Resend, Mailgun, Postmark, Loops."
---

# /email-sequences — Secuencias de Email Automatizadas

Configura las secuencias de email que acompañan a tus usuarios desde que se registran: una serie de mensajes que se envían solos en los días correctos para que tus usuarios entiendan el valor de tu producto y se conviertan en clientes. Antes de enviar el primer email, el skill verifica que tu dominio esté correctamente configurado para que los emails lleguen al inbox y no a spam.

## Knowledge Injection

Leer los siguientes archivos ANTES de la Fase 1. Si un archivo no existe, registrar una advertencia y continuar — aplica graceful degradation.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio por persona, estructura de la secuencia de bienvenida | Yes | framework |
| `knowledge/_inject/sales-crm-patterns.md` | Patrones de deliverability, suppression list, attribution de emails | No | framework |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente — el skill NO produce output hasta resolverla

- [ ] El dominio de envío NO tiene DKIM/SPF/DMARC publicados y verificados → BLOQUEAR con "Configurá DKIM/SPF antes de enviar emails" + instrucciones DNS del provider
- [ ] No se puede detectar el stack del proyecto Y el usuario no puede especificarlo
- [ ] El usuario pega una API key real en el chat → DETENER y pedir que la configure en `.env`
  Patrones a detectar (escanear en cada mensaje del usuario):
  - Resend: `re_` seguido de caracteres alfanuméricos
  - Postmark: tokens UUID en contexto `POSTMARK_SERVER_TOKEN`
  - Mailgun: `key-` seguido de hex, o strings en contexto `MAILGUN_API_KEY`
  - Loops: strings en contexto `LOOPS_API_KEY`
  Acción al detectar: DETENER — NO persistir la key en ningún artefacto — mostrar: "Esa clave no debe compartirse en el chat. Configurala en tu archivo `.env` local."

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA habilitar el envío de emails si DKIM/SPF/DMARC no están verificados — el gate de entregabilidad es no omitible
- NUNCA solicitar `RESEND_API_KEY`, `MAILGUN_API_KEY`, `POSTMARK_SERVER_TOKEN`, `LOOPS_API_KEY` ni equivalentes en el chat
- NUNCA hardcodear API keys ni secrets — solo placeholders en `.env.example`
- NUNCA generar una secuencia sin suppression list ni link de unsubscribe — es obligatorio desde el primer envío
- NUNCA sobreescribir secuencias de email existentes sin confirmación del usuario

### REQUIRED OUTPUTS
- [ ] Secuencia de bienvenida de 5 emails (Day 0, Day 2, Day 5, Day 10, Day 15) generada en el proyecto
- [ ] Triggers configurados: `signup`, `trial_started`, `feature_not_used_7d`, `plan_upgraded`
- [ ] A/B testing en subject line de Day 0 y Day 15 (control/variant 50/50)
- [ ] Suppression list activa + link de unsubscribe en todos los emails
- [ ] `.env.example` con placeholders del provider
- [ ] `docs/email/sequences-map.md` con el diagrama del árbol de secuencias
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (configure-provider) → Phase 2 (define-sequence)
             → Phase 3 (configure-triggers) → Phase 4 (setup-ab-testing)
             → Phase 5 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

> S — DKIM/SPF/DMARC verificados antes del primer envío; sin secrets hardcodeados; suppression list + unsubscribe obligatorios.
> T — secuencia, triggers y A/B testing verificables; Day 0 entregado en sandbox antes de cerrar.

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/email-sequences` es standalone si no hay workflow activo.

---

## Fase 1: configure-provider

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Detectar el provider de email** — buscar en `package.json` y `.env.example`:
   - `resend` → Resend detectado (default si no hay ninguno)
   - `mailgun.js` o `mailgun-js` → Mailgun detectado
   - `postmark` → Postmark detectado
   - `@loops/sdk` o contexto `LOOPS_API_KEY` → Loops detectado
   - Ninguno → preguntar en lenguaje de negocio:
     ```
     ¿Con qué servicio querés enviar tus emails?
     A) Resend — el más simple para empezar (recomendado)
     B) Mailgun — robusto, ideal para alto volumen
     C) Postmark — enfocado en máxima entregabilidad
     D) Loops — pensado para SaaS, secuencias visuales
     ```
2. [ ] **Verificar DKIM/SPF/DMARC del dominio de envío** (GATE DE SEGURIDAD — no omitible):
   - Determinar el dominio de envío (preguntar al usuario si no está claro)
   - Verificar registros DNS:
     ```bash
     dig +short TXT _dmarc.{dominio}
     dig +short TXT {selector}._domainkey.{dominio}   # DKIM (selector del provider)
     dig +short TXT {dominio} | grep -i "v=spf1"        # SPF
     ```
   - Si CUALQUIERA falta → activar BLOCKING CONDITION (ver CHECKPOINT / IF FAILS)
3. [ ] **Crear o actualizar `.env.example`** con placeholders del provider (solo placeholders, nunca valores reales):
   - Resend: `RESEND_API_KEY=re_REPLACE_WITH_YOUR_VALUE`
   - Mailgun: `MAILGUN_API_KEY=REPLACE_WITH_YOUR_VALUE`, `MAILGUN_DOMAIN=REPLACE_WITH_YOUR_DOMAIN`
   - Postmark: `POSTMARK_SERVER_TOKEN=REPLACE_WITH_YOUR_VALUE`
   - Loops: `LOOPS_API_KEY=REPLACE_WITH_YOUR_VALUE`
   - Verificar que `.env` y `.env.local` están en `.gitignore`

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Provider identificado: `resend` | `mailgun` | `postmark` | `loops`
- [ ] DKIM, SPF y DMARC verificados y publicados para el dominio de envío
- [ ] `.env.example` con solo placeholders (sin valores reales)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: DKIM/SPF/DMARC no configurados — entregabilidad no garantizada
Cause: El dominio de envío no tiene los registros DNS necesarios; los emails irían a spam o serían rechazados.
Recovery:
  [ ] Option A: BLOQUEAR el skill y mostrar al usuario en lenguaje de negocio:
      ```
      Configurá DKIM/SPF antes de enviar emails. Sin esto, tus correos
      van a terminar en spam y tu dominio puede quedar marcado.

      En el dashboard de {provider} → Domains → Add domain, vas a obtener
      los registros DNS exactos. Agregalos en tu proveedor de DNS:
        - SPF:   un registro TXT en {dominio} con "v=spf1 include:{provider} ~all"
        - DKIM:  un registro TXT en {selector}._domainkey.{dominio}
        - DMARC: un registro TXT en _dmarc.{dominio} con "v=DMARC1; p=none; ..."

      Cuando {provider} marque el dominio como "Verified", volvé a ejecutar /email-sequences.
      ```
      NO generar ninguna secuencia ni habilitar envíos. Status: BLOCKED.
  [ ] Option B: Si el usuario confirma que ya configuró los registros pero la verificación DNS aún no propagó → esperar y re-verificar. No continuar con verificación pendiente.
  [ ] Option C: Si el usuario no puede indicar el dominio de envío → sin dominio no hay entregabilidad verificable. BLOQUEAR con `Status: BLOCKED` y explicar que sin un dominio propio configurado no se puede garantizar que los emails lleguen.

---

## Fase 2: define-sequence

### GATE IN
- [ ] Fase 1 completada — provider identificado y DKIM/SPF/DMARC verificados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Generar la secuencia de bienvenida de 5 emails** usando la estructura de onboarding-essentials.md:

   | Email | Día | Propósito (lenguaje de negocio) |
   |-------|-----|--------------------------------|
   | Day 0 | 0 | Bienvenida — confirmar registro, qué esperar, primer paso |
   | Day 2 | 2 | Value tip 1 — primer truco para sacarle provecho al producto |
   | Day 5 | 5 | Value tip 2 — segunda funcionalidad clave |
   | Day 10 | 10 | Case study — historia de un cliente que tuvo éxito |
   | Day 15 | 15 | CTA de conversión — invitación clara a pasar a pago |

2. [ ] **Incluir en CADA email** (obligatorio): link de unsubscribe funcional y registro automático en la suppression list al hacer unsubscribe.
3. [ ] **Generar el código de la secuencia** en el proyecto del usuario (carpeta de emails del provider o módulo dedicado), parametrizando `{PRODUCT_NAME}` y placeholders editables por el founder.
4. [ ] **Configurar la suppression list ACTIVA desde el primer envío**: un usuario que hace unsubscribe del Day 0 NO debe recibir Day 2, Day 5, Day 10 ni Day 15, y su email queda en la suppression list del provider.
5. [ ] Informar al usuario qué se generó, en lenguaje de negocio:
   ```
   Generé una secuencia de 5 emails que tus nuevos usuarios van a recibir
   automáticamente en sus primeros 15 días. Cada uno los acerca un poco más
   a convertirse en clientes. Todos incluyen un botón para darse de baja —
   si alguien se da de baja, no recibe ninguno de los siguientes.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Los 5 emails generados (Day 0, 2, 5, 10, 15) con propósito correcto
- [ ] Cada email tiene link de unsubscribe
- [ ] Suppression list activa: unsubscribe del Day 0 detiene Day 2/5/10/15 y registra el email
- [ ] Sin secrets hardcodeados en el código generado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pudo generar la secuencia o falta suppression list
Cause: Stack no soportado, conflicto con estructura del proyecto, o el provider no expone suppression list.
Recovery:
  [ ] Option A: Mostrar al usuario los archivos a crear manualmente con instrucciones en lenguaje de negocio, asegurando que la suppression list quede incluida.
  [ ] Option B: Si el provider no expone suppression list nativa → generar suppression list local (tabla/archivo) y filtrar destinatarios antes de cada envío. NUNCA omitir el unsubscribe.

---

## Fase 3: configure-triggers

### GATE IN
- [ ] Fase 2 completada — secuencia de 5 emails generada con suppression list

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Configurar los triggers** que disparan o ramifican la secuencia:

   | Trigger | Cuándo se dispara | Efecto |
   |---------|-------------------|--------|
   | `signup` | El usuario se registra | Inicia la secuencia → envía Day 0 |
   | `trial_started` | El usuario empieza un trial | Marca contexto para personalizar el CTA del Day 15 |
   | `feature_not_used_7d` | 7 días sin usar la feature clave | Ramifica a un recordatorio de activación |
   | `plan_upgraded` | El usuario pasa a pago | Saca al usuario de la secuencia de conversión (objetivo cumplido) |

2. [ ] **Garantizar que `signup` entregue el Day 0 rápidamente** (objetivo: < 5 minutos en sandbox del provider).
3. [ ] **Probar el trigger `signup` en sandbox**: simular el evento y verificar que el Day 0 llega al inbox del sandbox en menos de 5 minutos (con DKIM/SPF ya verificados).

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Los 4 triggers configurados (`signup`, `trial_started`, `feature_not_used_7d`, `plan_upgraded`)
- [ ] Trigger `signup` entrega el Day 0 en sandbox en < 5 min
- [ ] `plan_upgraded` saca al usuario de la secuencia de conversión

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El Day 0 no llega en sandbox o un trigger no dispara
Cause: API key del sandbox no configurada en `.env`, dominio sandbox sin verificar, o el provider no entregó.
Recovery:
  [ ] Option A: Verificar que el usuario configuró la API key del provider en su `.env` local (no en el chat) y que el dominio sandbox está verificado. Reintentar el envío de prueba.
  [ ] Option B: Revisar los logs del provider (sección Activity/Logs del dashboard) para ver si el email fue aceptado, diferido o rechazado, y mostrar el motivo al usuario.
  [ ] Option C: Si el provider está caído o el sandbox no responde → documentar el trigger como configurado-pendiente-de-prueba y marcar Status: PARTIAL.

---

## Fase 4: setup-ab-testing

### GATE IN
- [ ] Fase 3 completada — triggers configurados y Day 0 entregado en sandbox

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Configurar A/B testing en el subject line del Day 0 y del Day 15** (los dos emails de mayor impacto: apertura inicial y conversión):
   - Definir variante **control** y variante **variant** para cada uno
   - Reparto **50/50** de los destinatarios
   - Proponer subject lines distintos por variante (en lenguaje de negocio) y dejar al founder editarlos
2. [ ] **Definir la métrica de decisión**: open rate para Day 0, click/conversion rate para Day 15.
3. [ ] Explicar al usuario en lenguaje de negocio:
   ```
   Para los dos emails más importantes (el de bienvenida y el de conversión)
   armé dos versiones del asunto. La mitad de tus usuarios recibe una versión
   y la otra mitad la otra. Así vas a saber con datos cuál convierte mejor.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] A/B testing configurado en subject line de Day 0 (control/variant 50/50)
- [ ] A/B testing configurado en subject line de Day 15 (control/variant 50/50)
- [ ] Métrica de decisión definida por email

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El provider no soporta A/B testing nativo de subject lines
Cause: Provider con capacidades limitadas de experimentación.
Recovery:
  [ ] Option A: Implementar el split 50/50 a nivel de aplicación (asignar variante por hash del email/contact_id antes de enviar) y registrar la variante para medir resultados.
  [ ] Option B: Si no es viable medir → generar ambas variantes documentadas y dejar el A/B para activación manual posterior, marcando Status: PARTIAL.

---

## Fase 5: output

### GATE IN
- [ ] Fase 4 completada — A/B testing configurado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Asegurar que el directorio existe**:
   ```bash
   mkdir -p docs/email/
   ```
2. [ ] **Escribir `docs/email/sequences-map.md`** con el diagrama del árbol de secuencias: triggers → secuencia de 5 emails → ramificaciones (`feature_not_used_7d`, salida por `plan_upgraded`) → puntos de A/B testing → suppression list. Usar un diagrama legible (ASCII o Mermaid).
3. [ ] **Confirmar al usuario** en lenguaje de negocio dónde quedó todo y cuál es el siguiente paso:
   ```
   ¡Listo! Tus secuencias de email están configuradas.
   El mapa completo está en docs/email/sequences-map.md.
   Acordate de poner tu API key en el archivo .env antes de enviar en producción.
   Siguiente paso sugerido: /sales-funnel-design para conectar estos emails al funnel de ventas.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] `docs/email/sequences-map.md` creado con el árbol de secuencias
- [ ] Confirmación en lenguaje de negocio enviada
- [ ] Sin terminología técnica expuesta sin contexto al entrepreneur

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/email/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Crear `docs/email-sequences-map.md` directamente en `docs/`.
  [ ] Option B: Imprimir el diagrama en pantalla para guardado manual.

---

## FINAL CHECKPOINT

- [ ] DKIM/SPF/DMARC verificados ANTES de habilitar cualquier envío (gate de seguridad cumplido)
- [ ] Secuencia de 5 emails (Day 0, 2, 5, 10, 15) generada
- [ ] 4 triggers configurados (`signup`, `trial_started`, `feature_not_used_7d`, `plan_upgraded`)
- [ ] A/B testing en subject line de Day 0 y Day 15 (50/50)
- [ ] Suppression list activa + link de unsubscribe en todos los emails
- [ ] `.env.example` con solo placeholders — sin API keys solicitadas ni persistidas en el chat
- [ ] `docs/email/sequences-map.md` creado

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (DKIM/SPF verificados, sin secrets), T: PASS (Day 0 entregado en sandbox) |
| Artifacts | Secuencia 5 emails + triggers + A/B testing + `.env.example` + `docs/email/sequences-map.md` |
| Next Recommended | Ver tabla de flujo |
| Risks | BLOCKED si DKIM/SPF/DMARC no configurados; PARTIAL si A/B o sandbox quedaron pendientes |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

> **Origen típico**: `← /crm-integrate` — se ejecuta normalmente después de integrar el CRM, para que las secuencias se alimenten de los leads capturados.

| Condición | Próximo Skill |
|-----------|---------------|
| Secuencias configuradas exitosamente | `/sales-funnel-design` |
| DKIM/SPF/DMARC no configurados (BLOCKED) | Resolver registros DNS del provider y re-ejecutar `/email-sequences` |
| A/B testing o sandbox quedaron pendientes (PARTIAL) | Completar la prueba y re-ejecutar la fase pendiente |
| El usuario quiere emails transaccionales (recibos, reset de password) | `/transactional-email` |

---

## REFERENCE

### Providers soportados

| Provider | Default | Fortaleza | Verificación de dominio |
|----------|:-------:|-----------|-------------------------|
| Resend | ✓ | Simple, ideal para empezar | Dashboard → Domains (entrega SPF/DKIM/DMARC) |
| Mailgun | – | Alto volumen | Dashboard → Sending → Domains |
| Postmark | – | Máxima entregabilidad | Dashboard → Sender Signatures / Domains |
| Loops | – | Secuencias visuales para SaaS | Settings → Domains |

### Por qué DKIM/SPF/DMARC es un gate de BLOQUEO

Sin estos registros DNS, los proveedores de correo (Gmail, Outlook) no pueden verificar que los emails vienen realmente de tu dominio. El resultado es directo: tus correos van a spam, tu tasa de entrega se desploma y tu dominio puede quedar marcado como spammer de forma difícil de revertir. Por eso el skill NO habilita ningún envío hasta que la verificación pasa — es protección del activo más caro de un founder: su reputación de dominio.

- **SPF** (Sender Policy Framework): autoriza al provider a enviar en nombre de tu dominio.
- **DKIM** (DomainKeys Identified Mail): firma criptográfica que prueba que el email no fue alterado.
- **DMARC** (Domain-based Message Authentication): política de qué hacer si SPF/DKIM fallan.

### Suppression list y cumplimiento

La suppression list y el link de unsubscribe NO son opcionales. Son requisito legal (CAN-SPAM, GDPR) y de reputación: un usuario que se da de baja y sigue recibiendo emails puede reportarte como spam. La suppression list garantiza que un unsubscribe del Day 0 detiene toda la secuencia y registra al usuario para no volver a contactarlo.

### Escenarios cubiertos (Gherkin M-88b)

| Escenario | Fase / CHECKPOINT que lo cubre |
|-----------|--------------------------------|
| Day 0 entregado en < 5 min via Resend sandbox cuando se dispara `signup` (DKIM/SPF configurados) | Fase 3 (configure-triggers) — CHECKPOINT "Trigger `signup` entrega el Day 0 en sandbox en < 5 min" |
| `/email-sequences` bloquea si DKIM/SPF no están configurados (mensaje + instrucciones DNS) | BLOCKING CONDITION global + Fase 1 (configure-provider) CHECKPOINT + IF FAILS Option A |
| Suppression list activa: unsubscribe del Day 0 → no recibe Day 2/5/10/15 y queda en suppression list | Fase 2 (define-sequence) — CHECKPOINT "Suppression list activa: unsubscribe del Day 0 detiene Day 2/5/10/15 y registra el email" |
