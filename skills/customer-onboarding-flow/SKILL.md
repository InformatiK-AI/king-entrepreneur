---
name: customer-onboarding-flow
version: 1.0
api_version: 1.0.0
description: "Configura el onboarding de clientes post-conversión: secuencia de 3 emails en 7 días, checklist in-app de getting started con progreso visual, métricas de activación importadas de analytics, y una señal de churn automática en el CRM cuando el usuario no activa la feature clave en 7 días. Depende de /crm-integrate (field mapping del CRM existente)."
---

# /customer-onboarding-flow — Activá a tus nuevos clientes en sus primeros 7 días

Configura el flujo que acompaña a cada cliente nuevo desde que se convierte hasta que se vuelve un usuario activo: una secuencia de 3 emails en los primeros 7 días, un checklist visible dentro de tu producto que les muestra los pasos para arrancar, las métricas que definen "usuario activado" y una alerta automática en tu CRM cuando alguien no llega a usar la función clave en una semana. El objetivo es simple: que tus clientes nuevos lleguen rápido al momento en que ven el valor de tu producto, antes de que se enfríen.

## Knowledge Injection

Leer los siguientes archivos ANTES de la Fase 1. Si un archivo no existe, registrar una advertencia y continuar — aplica graceful degradation.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio por persona, estructura del onboarding y los emails de activación | Yes | framework |
| `knowledge/_inject/sales-crm-patterns.md` | Contexto del CRM y de activación: data model, criterios de señal/alerta (custom: este skill crea una regla de churn en el CRM) | No | framework |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] No existe `docs/crm/field-mapping.md` (`/crm-integrate` no fue completado) → BLOQUEAR con "Conectá tu CRM primero con /crm-integrate" — la señal de churn necesita el field mapping del CRM
- [ ] El founder no puede articular cuál es la feature clave que define "usuario activado" → pedir contexto antes de continuar
- [ ] El usuario pega una API key o token real del CRM/email en el chat → DETENER y pedir que lo configure en `.env`
  Patrones a detectar (escanear en cada mensaje del usuario):
  - CRM: `pat-na1-`, `pat-eu1-`, `PIPEDRIVE_API_TOKEN`, `secret_` (Notion), strings largos en contexto `*_API_KEY` / `*_TOKEN` / `*_WEBHOOK_SECRET`
  - Email: `re_` (Resend), tokens en contexto `POSTMARK_SERVER_TOKEN` / `MAILGUN_API_KEY` / `LOOPS_API_KEY`
  Acción al detectar: DETENER — NO persistir el valor en ningún artefacto — mostrar: "Esa clave no debe compartirse en el chat. Configurala en tu archivo `.env` local."

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA redefinir eventos de analytics que otro skill ya creó (`/analytics-setup`) — importar de `analytics/events.*` y extender solamente
- NUNCA solicitar tokens, API keys ni webhook secrets del CRM o del proveedor de email en el chat — los secrets viven solo en `.env`
- NUNCA hardcodear secrets/API keys/tokens — solo placeholders en `.env.example`
- NUNCA reconfigurar DKIM/SPF/DMARC ni el proveedor de email — el onboarding HEREDA la entregabilidad ya configurada por `/email-sequences` y `/transactional-email`
- NUNCA generar la secuencia de onboarding sin link de unsubscribe ni respetar la suppression list existente
- NUNCA crear la señal de churn fuera del CRM — es una regla EN el CRM, no una automatización paralela

### REQUIRED OUTPUTS
- [ ] Secuencia de onboarding de 3 emails (Day 0 activación del account, Day 3 primera feature clave, Day 7 caso de éxito) generada en el proyecto
- [ ] Componente de checklist in-app "getting started" con milestones y progreso visual (React/HTML según el stack)
- [ ] Métricas de activación documentadas: eventos que definen "usuario activado" para el tipo de producto (importados/extendidos de `analytics/events.*`, no redefinidos)
- [ ] Regla de churn risk en el CRM: dispara cuando el usuario NO activa la feature clave en 7 días
- [ ] `docs/onboarding/onboarding-flow.md` con el mapa del flujo (emails + milestones + activación + señal de churn)
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load + GATE crm field-mapping) → Phase 1 (define-milestones)
             → Phase 2 (generate-checklist-ui) → Phase 3 (configure-email-sequence)
             → Phase 4 (setup-churn-signal) → Phase 5 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

> **S** — secrets del CRM y del email solo desde `.env`; nunca hardcodeados ni pedidos en el chat; señal de churn como regla dentro del CRM (sin exponer credenciales).
> **T** — flujo verificable end-to-end: el checklist UI renderiza y un milestone se marca como completo al dispararse el evento de activación correspondiente.

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/customer-onboarding-flow` es standalone si no hay workflow activo.

### GATE IN
- [ ] Knowledge injection completada (onboarding-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Verificar la dependencia del CRM (BLOQUEO)** — comprobar que `/crm-integrate` fue completado y existe el field mapping:
   ```bash
   test -f docs/crm/field-mapping.md && echo "OK" || echo "MISSING"
   ```
2. [ ] Si el archivo NO existe → DETENER el skill y mostrar al founder en lenguaje de negocio:
   ```
   Para configurar el onboarding necesito tu CRM conectado primero.

   La alerta de "cliente que no arranca" se crea como una regla dentro de tu CRM,
   y para eso necesito saber cómo están mapeados tus datos de contacto.

   Conectá tu CRM con /crm-integrate y volvé a ejecutar /customer-onboarding-flow.
   ```
   NO avanzar a la Fase 1.
3. [ ] **Detectar el contexto existente, sin re-preguntar lo ya configurado**:
   - Leer `docs/crm/field-mapping.md` → identificar el provider del CRM y los campos disponibles.
   - Si existe `analytics/events.*` (de `/analytics-setup`) → leer los eventos disponibles para usarlos como métricas de activación (NO redefinir).
   - Si existe `docs/analytics/tracking-plan.md` → leer el tipo de producto y el criterio de disparo de cada evento.
   - Si existe `docs/email/sequences-map.md` (de `/email-sequences`) → identificar el proveedor de email y el dominio ya verificado (DKIM/SPF heredado).

### CHECKPOINT
> ✅ Verify before continuing

- [ ] `docs/crm/field-mapping.md` existe (re-verificado con `test -f`)
- [ ] Provider del CRM identificado desde el field mapping
- [ ] Eventos de activación existentes detectados (de `analytics/events.*` si está) o anotado que se definirán nuevos
- [ ] Proveedor de email heredado identificado si `docs/email/sequences-map.md` existe

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: CRM no integrado — falta `docs/crm/field-mapping.md`
Cause: `/crm-integrate` no fue ejecutado o no completó; la señal de churn no tiene a dónde escribirse.
Recovery:
  [ ] Option A: Redirigir al founder a `/crm-integrate` y NO continuar — re-ejecutar `/customer-onboarding-flow` cuando el CRM esté conectado
  [ ] Option B: Si el founder confirma que tiene un CRM conectado por fuera del framework, pedirle que genere `docs/crm/field-mapping.md` con el esquema de sus campos antes de continuar
  [ ] Option C: Si no hay forma de resolverlo → `Status: BLOCKED`, surface al usuario. NO producir ningún output.

---

## Fase 1: define-milestones

### GATE IN
- [ ] Fase 0 completada — CRM verificado y contexto cargado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Identificar la feature clave de activación** preguntando en lenguaje de negocio (de a una pregunta, no todas juntas):
   ```
   ¿Cuál es la primera acción que, cuando un cliente la hace, te demuestra
   que entendió el valor de tu producto? (ej: "crea su primer proyecto",
   "invita a un compañero", "conecta su cuenta de banco")
   ```
2. [ ] **Definir los milestones de getting started** (3 a 5) — los pasos que llevan al cliente desde el registro hasta la feature clave. Ejemplo para SaaS:
   - Completar el perfil / setup inicial del account
   - Llegar a la feature clave (el momento "aha")
   - Invitar a un compañero o conectar una integración
3. [ ] **Mapear cada milestone a un evento de activación**:
   - Si existe `analytics/events.*` → reutilizar los eventos existentes (ej: SaaS usa `first_feature_used`, `email_verified`). IMPORTAR, no redefinir.
   - Si falta algún evento de activación → anotarlo como evento NUEVO a agregar en la Fase 5 (extiende `analytics/events.*`, no lo reescribe).
4. [ ] Confirmar con el founder la lista de milestones y cuál de ellos es "usuario activado".

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Feature clave de activación articulada por el founder
- [ ] 3 a 5 milestones de getting started definidos
- [ ] Cada milestone mapeado a un evento de activación (existente o nuevo)
- [ ] Definido cuál milestone marca "usuario activado"

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede definir la feature clave de activación
Cause: El founder no logra articular qué acción demuestra que el cliente entendió el valor.
Recovery:
  [ ] Option A: Preguntar concreto: "¿Qué hace un cliente tuyo que SÍ se queda, que uno que se va nunca llega a hacer?" — usar esa acción como feature clave
  [ ] Option B: Usar el evento de activación por defecto del tipo de producto (SaaS → `first_feature_used`) y documentarlo como provisional para ajustar después
  [ ] Option C: Si no hay tipo de producto claro → usar SaaS como base y dejar los milestones como editables por el founder

---

## Fase 2: generate-checklist-ui

### GATE IN
- [ ] Fase 1 completada — milestones definidos y mapeados a eventos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Detectar el stack** de `.king/knowledge/stack.md` (o `package.json`) para generar el componente en el lenguaje correcto: React (`.tsx`), Vue, o HTML según el stack del proyecto.
2. [ ] **Generar el componente de checklist "getting started"** con:
   - La lista de milestones de la Fase 1, cada uno con su estado (pendiente / completo)
   - Progreso visual (barra o contador "X de N completados")
   - Cada milestone se marca como completo al recibir su evento de activación (no por click manual)
3. [ ] **Conectar el checklist a los eventos de activación**: el componente escucha/consulta los eventos de `analytics/events.*` y marca el milestone cuando el evento correspondiente se dispara. Importar la definición tipada del evento, no inventar una nueva.
4. [ ] **Dejar el componente listo para insertar** en el dashboard del producto, parametrizando `{PRODUCT_NAME}` y los textos editables por el founder.
5. [ ] Explicar al founder en lenguaje de negocio:
   ```
   Generé un checklist que tus clientes nuevos ven dentro de tu producto:
   les muestra los pasos para arrancar y se va marcando solo a medida que
   los completan. Es la forma más efectiva de guiarlos hasta el momento
   en que ven el valor de lo que ofrecés.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Componente de checklist generado en el lenguaje del stack (React/Vue/HTML)
- [ ] Muestra los milestones con progreso visual (X de N)
- [ ] Cada milestone se marca al dispararse su evento de activación (no por click manual)
- [ ] El componente importa los eventos de `analytics/events.*` (no los redefine)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pudo generar el componente de checklist para el stack
Cause: Stack no detectado, o el componente no logra conectarse a los eventos de activación.
Recovery:
  [ ] Option A: Preguntar el framework al founder (Next.js, React, Vue, etc.) y generar para ese, conectando los eventos por su nombre canónico
  [ ] Option B: Si los eventos no existen aún → generar el componente con los eventos NUEVOS definidos en la Fase 1 y dejar el binding documentado para conectar tras la Fase 5
  [ ] Option C: Si no se puede generar para el stack → entregar una versión HTML agnóstica con instrucciones de dónde insertarla y cómo conectar cada milestone a su evento

---

## Fase 3: configure-email-sequence

### GATE IN
- [ ] Fase 2 completada — checklist UI generado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Generar la secuencia de onboarding de 3 emails en 7 días** (distinta de la secuencia de bienvenida de `/email-sequences`: esta es post-conversión y se enfoca en activación):

   | Email | Día | Propósito (lenguaje de negocio) |
   |-------|-----|--------------------------------|
   | Day 0 | 0 | Activación del account — bienvenida como cliente, primer paso concreto del checklist |
   | Day 3 | 3 | Primera feature clave — guía directa para llegar al momento "aha" |
   | Day 7 | 7 | Caso de éxito — historia de un cliente que activó y obtuvo resultado, refuerzo de hábito |

2. [ ] **Heredar la entregabilidad ya configurada**: usar el mismo proveedor de email y el dominio verificado de `/email-sequences` / `/transactional-email` (DKIM/SPF/DMARC heredado). NO reconfigurar DNS ni el provider. Si no hay proveedor configurado todavía → ver IF FAILS.
3. [ ] **Incluir en CADA email** (obligatorio): link de unsubscribe funcional y respeto de la suppression list existente (un usuario en la suppression list NO recibe estos emails).
4. [ ] **Conectar el trigger de inicio** a la conversión (evento de cliente nuevo) y alinear el contenido con los milestones del checklist: el Day 3 empuja la feature clave que también es el milestone central.
5. [ ] Explicar al founder en lenguaje de negocio:
   ```
   Armé 3 emails que tus clientes nuevos reciben en su primera semana:
   el primero los recibe y les marca el primer paso, el segundo los lleva
   a la función más importante, y el tercero les muestra el resultado que
   pueden lograr. Usan tu mismo dominio de email ya configurado, así que
   llegan al inbox.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Los 3 emails generados (Day 0, Day 3, Day 7) con su propósito correcto
- [ ] Proveedor de email y dominio heredados de `/email-sequences` / `/transactional-email` (sin reconfigurar DKIM/SPF)
- [ ] Cada email tiene link de unsubscribe y respeta la suppression list existente
- [ ] Trigger de inicio conectado a la conversión; Day 3 alineado con la feature clave
- [ ] Sin secrets de email hardcodeados

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No hay proveedor de email con entregabilidad verificada para heredar
Cause: `/email-sequences` y `/transactional-email` no fueron ejecutados; no hay dominio con DKIM/SPF verificado.
Recovery:
  [ ] Option A: Generar las plantillas de los 3 emails igual, pero marcar el envío como pendiente y redirigir: "Configurá tu email con /email-sequences (verifica tu dominio) antes de activar esta secuencia." Status: PARTIAL.
  [ ] Option B: Si el founder confirma un proveedor ya configurado por fuera → reutilizar ese dominio verificado y NUNCA reconfigurar DKIM/SPF desde este skill
  [ ] Option C: Si el provider no expone suppression list → reutilizar la suppression list ya creada por `/email-sequences`; nunca generar una secuencia sin unsubscribe

---

## Fase 4: setup-churn-signal

### GATE IN
- [ ] Fase 3 completada — secuencia de onboarding generada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Definir la regla de churn risk en el CRM** (no una automatización paralela): cuando el usuario NO dispara el evento de activación canónico (el de la feature clave definido en Fase 1, ej. `first_feature_used` de `analytics/events.*`) en 7 días desde la conversión, marcarlo como "en riesgo de churn".
2. [ ] **Usar el field mapping del CRM** (`docs/crm/field-mapping.md`) para escribir la señal en el campo correcto del contacto:
   - Reutilizar el provider y los campos ya mapeados (ej: `lead_score`, o una property de estado del contacto del CRM).
   - **Fuente de verdad**: el criterio se ancla al MISMO evento de activación de `analytics/events.*` que usan el checklist (Fase 2) y el milestone (Fase 1) — ausencia de ESE evento en 7 días. El trigger `feature_not_used_7d` de `/email-sequences` es un trigger DERIVADO de la misma condición lógica (comparten la condición, no el evento) — NO redefinir ni duplicar el evento de analytics.
3. [ ] **Configurar la regla EN el CRM** según el provider del field mapping (HubSpot workflow / Pipedrive automation / Attio / Salesforce flow / Notion), leyendo cualquier token necesario desde `.env` (nunca del chat).
4. [ ] **Documentar la acción de la señal**: cuando se marca el riesgo de churn, notificar al founder y/o disparar un email de re-activación (reutilizando el proveedor heredado, no uno nuevo).
5. [ ] Explicar al founder en lenguaje de negocio:
   ```
   Configuré una alerta en tu CRM: si un cliente nuevo no llega a usar la
   función clave en sus primeros 7 días, queda marcado como "en riesgo".
   Así sabés a quién darle una mano antes de que se vaya — recuperar un
   cliente a tiempo es mucho más barato que conseguir uno nuevo.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Regla de churn risk creada DENTRO del CRM (no como automatización paralela)
- [ ] Criterio: ausencia del evento de activación canónico (Fase 1, de `analytics/events.*`) en 7 días → marcado en riesgo
- [ ] El criterio referencia el MISMO evento de activación que el checklist (Fase 2) y el milestone (Fase 1) — sin redefinirlo
- [ ] La señal escribe en el campo correcto según `docs/crm/field-mapping.md`
- [ ] Token del CRM leído desde `.env`, nunca del chat ni hardcodeado
- [ ] Acción de la señal definida (notificación al founder y/o email de re-activación)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede crear la regla de churn en el CRM
Cause: El provider no expone automations nativas, o el field mapping no tiene un campo de estado disponible.
Recovery:
  [ ] Option A: Si el provider no tiene workflows nativos → documentar una regla equivalente vía webhook/cron que escriba en el CRM usando el field mapping existente (sigue siendo una señal EN el CRM, no por fuera)
  [ ] Option B: Si falta un campo de estado en el field mapping → indicar la custom property a crear en el dashboard del CRM y registrarla en `docs/crm/field-mapping.md`
  [ ] Option C: Si no se puede configurar en esta sesión → documentar la regla y su criterio en `docs/onboarding/onboarding-flow.md` y marcar Status: PARTIAL con el path para retomar

---

## Fase 5: output

### GATE IN
- [ ] Fase 4 completada — señal de churn configurada en el CRM

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Asegurar que el directorio existe**:
   ```bash
   mkdir -p docs/onboarding/
   ```
2. [ ] **Si la Fase 1 definió eventos de activación NUEVOS** → agregarlos a `analytics/events.*` extendiendo el archivo (importar lo existente, no redefinir). Documentar su criterio de disparo.
3. [ ] **Escribir `docs/onboarding/onboarding-flow.md`** con el mapa completo del flujo: milestones del checklist → eventos de activación → secuencia de 3 emails (Day 0/3/7) → criterio de "usuario activado" → regla de churn risk en el CRM. Usar un diagrama legible (ASCII o Mermaid).
4. [ ] **Confirmar al founder** en lenguaje de negocio dónde quedó todo y el siguiente paso:
   ```
   ¡Listo! Tu onboarding de clientes está configurado:
   - Un checklist dentro de tu producto que los guía hasta la función clave
   - 3 emails en su primera semana que los acompañan a activarse
   - Una alerta en tu CRM si alguno no arranca en 7 días

   El mapa completo está en docs/onboarding/onboarding-flow.md.
   Acordate de tener tus tokens en el archivo .env antes de producción.
   Siguiente paso sugerido: /sales-funnel-design para conectar la activación al funnel de ventas.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] `docs/onboarding/onboarding-flow.md` creado con el mapa del flujo
- [ ] Eventos de activación nuevos (si los hubo) agregados a `analytics/events.*` sin redefinir los existentes
- [ ] Confirmación en lenguaje de negocio enviada
- [ ] Sin terminología técnica expuesta sin contexto al entrepreneur

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Directorio docs/onboarding/ no existe y no se puede crear
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Crear `docs/onboarding-flow.md` directamente en `docs/`
  [ ] Option B: Imprimir el mapa del flujo en pantalla para guardado manual del founder

---

## FINAL CHECKPOINT

- [ ] `docs/crm/field-mapping.md` verificado en Fase 0 (dependencia de `/crm-integrate` resuelta)
- [ ] Secuencia de onboarding de 3 emails (Day 0, Day 3, Day 7) generada con unsubscribe y suppression list heredada
- [ ] Componente de checklist in-app con milestones y progreso visual generado
- [ ] Milestones mapeados a eventos de activación de `analytics/events.*` (importados, no redefinidos)
- [ ] Regla de churn risk creada DENTRO del CRM (feature clave no activada en 7 días)
- [ ] DKIM/SPF/DMARC heredados de `/email-sequences` — no reconfigurados
- [ ] `docs/onboarding/onboarding-flow.md` creado
- [ ] Sin tokens/API keys del CRM o email solicitados ni persistidos en el chat

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (secrets solo en `.env`, señal de churn dentro del CRM), T: PASS (checklist renderiza, milestone se marca al disparar el evento de activación) |
| Artifacts | Secuencia 3 emails + componente checklist UI + regla de churn en CRM + `docs/onboarding/onboarding-flow.md` (+ extensión de `analytics/events.*` si aplica) |
| Next Recommended | Ver tabla de flujo |
| Risks | BLOCKED si falta `docs/crm/field-mapping.md`; PARTIAL si el email o la regla de churn quedaron pendientes de proveedor |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

> **Origen requerido**: `← /crm-integrate` — debe estar COMPLETE antes (existe `docs/crm/field-mapping.md`). Típicamente se ejecuta también después de `/sales-funnel-design`, ya post-lanzamiento (P2).

| Condición | Próximo Skill |
|-----------|---------------|
| Onboarding configurado, conectar la activación al funnel de ventas | `/sales-funnel-design` |
| Falta el CRM (BLOCKED) | `/crm-integrate` y luego re-ejecutar `/customer-onboarding-flow` |
| Falta proveedor de email con dominio verificado (PARTIAL) | `/email-sequences` (verificar dominio) y luego completar la secuencia de onboarding |
| Quiere medir mejor la activación antes de optimizar | `/analytics-setup` (definir/ampliar eventos) |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Por qué depende de `/crm-integrate`

La señal de churn risk se crea como una regla DENTRO del CRM, escribiendo en el campo correcto del contacto. Para saber a qué campo escribir, el skill necesita el field mapping canónico que `/crm-integrate` genera en `docs/crm/field-mapping.md` (`contact_id`, `email`, `source`, `lead_score`, etc.). Sin ese archivo no hay forma de localizar el contacto ni de marcar su estado de riesgo — por eso es una BLOCKING CONDITION.

### Qué comparte con otros skills (y qué NO redefine)

| Recurso | Skill dueño | Cómo lo usa este skill |
|---------|-------------|------------------------|
| Eventos de activación (`analytics/events.*`) | `/analytics-setup` | IMPORTA y extiende — nunca redefine (evita double-counting en los funnels) |
| Dominio + DKIM/SPF/DMARC del email | `/email-sequences`, `/transactional-email` | HEREDA la entregabilidad ya verificada — no reconfigura DNS ni el provider |
| Suppression list + unsubscribe | `/email-sequences` | Reutiliza la suppression list existente; cada email de onboarding incluye unsubscribe |
| Field mapping del CRM | `/crm-integrate` | Lee `docs/crm/field-mapping.md` para escribir la señal de churn en el CRM |
| Trigger `feature_not_used_7d` | `/email-sequences` | Misma semántica de "feature clave no activada en 7 días" — la señal de churn se alinea, no duplica |

### Onboarding de bienvenida vs. onboarding de cliente

`/email-sequences` genera la secuencia de **bienvenida** (5 emails en 15 días, foco en convertir un registro en cliente). Este skill genera la secuencia de **onboarding de cliente** (3 emails en 7 días, foco en activar a quien YA convirtió). Son secuencias distintas con objetivos distintos: la primera vende, la segunda activa.

### Escenarios cubiertos (criterio de testing §4 — E2E)

| Escenario | Fase / CHECKPOINT que lo cubre |
|-----------|--------------------------------|
| El checklist UI renderiza dentro del producto | Fase 2 (generate-checklist-ui) — CHECKPOINT "Componente de checklist generado en el lenguaje del stack" + "muestra los milestones con progreso visual" |
| Un milestone se marca como completo al dispararse el evento de activación | Fase 2 (generate-checklist-ui) — CHECKPOINT "Cada milestone se marca al dispararse su evento de activación (no por click manual)" |
| El milestone usa el evento real de analytics, sin redefinirlo | Fase 1 (define-milestones) CHECKPOINT "Cada milestone mapeado a un evento de activación" + Fase 2 CHECKPOINT "importa los eventos de `analytics/events.*`" |
