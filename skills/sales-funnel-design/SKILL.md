---
name: sales-funnel-design
version: 1.0
api_version: 1.0.0
description: "Diseña el funnel de ventas (Lead → MQL → SQL → Opportunity → Customer) con lead scoring automático, automation rules en el CRM y dashboard de conversión. Output: docs/sales/funnel-definition.md con criterios y thresholds por etapa."
model: sonnet
---

# /sales-funnel-design — Funnel de Ventas con Lead Scoring Automático

Skill que diseña, junto al founder, cómo se mueven los contactos desde que dejan su email hasta que pagan: define las etapas del funnel, asigna puntos automáticos por cada señal de interés (lead scoring), configura el aviso al founder cuando un lead está listo para vender, y arma el dashboard que muestra dónde se cae la gente y cuánto cuesta cada cliente.

## Knowledge Injection

Leer los siguientes archivos ANTES de Fase 1. Si un archivo no existe, registrar una advertencia y continuar — graceful degradation aplica.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/sales-crm-patterns.md` | Lead scoring, definiciones MQL/SQL, funnel metrics (CVR, CAC, LTV), attribution models | Yes | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio por persona, mensajes para el founder | Yes | framework |
| `.king/knowledge/secrets-provider.md` | Estado del secrets provider (M07) — detección de gate de seguridad | No | project |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar. Excepción: el gate de secrets provider (ver BLOCKING CONDITIONS) bloquea solo cuando el CRM exige credenciales y no hay provider — no es una degradación silenciosa.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente — no producir ningún output

- [ ] `/crm-integrate` NO está completo (no existe `docs/crm/field-mapping.md`, el artefacto de handoff que genera `/crm-integrate`) → dependencia DURA: el funnel se construye SOBRE el CRM ya integrado
- [ ] El producto maneja PII de leads y NO existe `.king/knowledge/secrets-provider.md` (M07) → verificación defensiva: normalmente `/crm-integrate` ya lo exigió aguas arriba; si falta, configurar el secrets provider antes de continuar
- [ ] El lead scoring se alimentaría de datos personales sin consentimiento registrado (GDPR/consent capturado por `/lead-magnets`) → no diseñar scoring sobre PII sin base legal; resolver el consent antes de continuar
- [ ] El usuario pega una API key o token real del CRM en el chat → DETENER, NO persistir en `docs/sales/` ni en la sesión, pedir que lo configure en `.env`

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA hardcodear secrets, API keys o tokens del CRM en archivos del repo — solo referenciar variables desde `.env`
- NUNCA escribir credenciales del CRM en `docs/sales/`, en el chat ni en la sesión
- NUNCA persistir input raw del usuario en `.king/sessions/` — solo outputs procesados; delimitar input persistido con `<!-- USER_INPUT_START -->` ... `<!-- USER_INPUT_END -->`
- NUNCA diseñar scoring que puntúe rasgos personales sensibles (salud, etnia, religión) — solo señales de intención de compra
- NUNCA producir output (`funnel-definition.md` ni automation rules) si una BLOCKING CONDITION sigue TRUE

### REQUIRED OUTPUTS
- [ ] Definición del funnel Lead → MQL → SQL → Opportunity → Customer con criterios de transición por etapa
- [ ] Tabla de lead scoring con puntos por acción y thresholds (MQL ≥ 30, SQL ≥ 60, ajustables)
- [ ] Automation rule configurada en el CRM: notificación al founder cuando un lead alcanza SQL (score ≥ 60)
- [ ] Dashboard de conversión: CVR por etapa, CAC por canal, tiempo promedio en cada etapa
- [ ] `docs/sales/funnel-definition.md` con criterios y thresholds por etapa
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (define-stages) → Phase 2 (configure-scoring)
             → Phase 3 (setup-automations) → Phase 4 (generate-dashboard)
             → Phase 5 (output)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

- **S (Security)**: tokens del CRM solo desde `.env`; secrets provider verificado; scoring sin PII sin consentimiento.
- **T (Testing)**: el threshold SQL y las automation rules son verificables (caso de 65 puntos dispara la notificación); el dashboard expone métricas reproducibles por etapa.

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/sales-funnel-design` es standalone si no hay workflow activo, pero SIEMPRE exige `/crm-integrate` completo (ver BLOCKING CONDITIONS).

---

## Fase 1: define-stages

### GATE IN
- [ ] Knowledge injection completada (sales-crm-patterns.md leído)
- [ ] BLOCKING CONDITIONS verificadas en FALSE — `/crm-integrate` completo y gate de secrets/consent resuelto

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Introducir el propósito en lenguaje de negocio:
   ```
   Vamos a diseñar el camino que recorre un contacto desde que deja su email
   hasta que te paga. Así vas a saber en qué etapa se te cae la gente y cuándo
   un lead está realmente listo para que le vendas.
   ```
2. [ ] Presentar el funnel canónico y validar las etapas con el founder (usar sales-crm-patterns.md):
   - **Lead**: dejó su email (formulario, lead magnet) y dio consentimiento
   - **MQL** (Marketing Qualified): muestra interés real — score ≥ 30
   - **SQL** (Sales Qualified): listo para que le vendas — score ≥ 60 o pidió una demo
   - **Opportunity**: negociación activa — propuesta enviada
   - **Customer**: pagó — suscripción o compra activa
3. [ ] Para cada etapa, acordar el **criterio de transición** en lenguaje de negocio. Preguntar:
   - "¿Qué tiene que hacer un contacto para que lo consideres 'interesado de verdad' (MQL)?"
   - "¿Y para considerarlo listo para venderle (SQL)?"
   - "¿Qué cuenta como 'oportunidad': una llamada agendada, una propuesta enviada?"
4. [ ] Confirmar que las etapas existen como `stage` en el CRM ya integrado (según `docs/crm/field-mapping.md` de `/crm-integrate`); si el CRM usa otros nombres, mapear los nombres canónicos a los del CRM.

### CHECKPOINT
- [ ] Las 5 etapas del funnel definidas y validadas con el founder
- [ ] Criterio de transición acordado para cada etapa (en lenguaje de negocio)
- [ ] Etapas mapeadas a los `stage` del CRM integrado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El founder no puede definir los criterios de transición
Cause: Proceso de venta poco claro o producto sin ciclo de venta definido.
Recovery:
  [ ] Option A: Usar los criterios por defecto de sales-crm-patterns.md (MQL ≥ 30, SQL ≥ 60, Opportunity = propuesta enviada) y marcarlos como "ajustar con datos reales".
  [ ] Option B: Si el producto es self-service sin venta humana, colapsar Opportunity dentro de SQL → Customer y documentarlo como funnel simplificado.
  [ ] Option C: Si las etapas del CRM no coinciden y no se pueden mapear, retornar al founder para revisar la integración con `/crm-integrate`.

---

## Fase 2: configure-scoring

### GATE IN
- [ ] Fase 1 completada — etapas y criterios de transición definidos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Presentar la tabla base de lead scoring (de sales-crm-patterns.md) y validar/ajustar los puntos con el founder:

   | Acción | Puntos |
   |--------|:------:|
   | Formulario enviado | +10 |
   | Email abierto | +2 |
   | Link de email clickeado | +5 |
   | Feature clave usada | +15 |
   | Pricing page visitada | +10 |
   | Demo solicitada | +30 |
   | Inactividad 14 días | −10 |

2. [ ] Acordar los **thresholds** de cada etapa (defaults: MQL ≥ 30, SQL ≥ 60). Explicar en lenguaje de negocio qué significa cada uno.
3. [ ] **Validar el caso de prueba del threshold SQL** con una suma CERRADA y reproducible (cada sumando sale de la tabla de arriba):
   ```
   Caso canónico SQL (= 65 puntos, ≥ 60 → SQL):
     formulario enviado    +10
     feature clave usada    +15
     pricing page visitada  +10
     demo solicitada        +30
     ──────────────────────────
     total                  65   → clasifica como SQL ✓
   ```
   > ⚠️ El threshold SQL es testeable: la suma de esas acciones da EXACTAMENTE 65 ≥ 60 → etapa SQL. Usar este mismo conjunto cerrado en el test (Fase 3), sin "+ señales" abiertas.
4. [ ] Confirmar que el campo `lead_score` existe en la entidad Contact del CRM (modelo canónico de sales-crm-patterns.md) y que las acciones tienen un evento/trigger que las alimente.
5. [ ] Documentar la fórmula de scoring y los thresholds para escribirlos en `funnel-definition.md` (Fase 5).

### CHECKPOINT
- [ ] Tabla de scoring acordada con puntos por acción
- [ ] Thresholds MQL y SQL definidos (numéricos)
- [ ] Caso de prueba verificado: formulario(+10) + feature(+15) + pricing(+10) + demo(+30) = 65 ≥ 60 → SQL
- [ ] Campo `lead_score` confirmado en el CRM y acciones con trigger asociado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El CRM no soporta scoring automático o no hay campo lead_score
Cause: Plan del CRM sin automation, o integración sin el campo mapeado.
Recovery:
  [ ] Option A: Crear el campo `lead_score` en el CRM como custom field y mapearlo en la integración existente.
  [ ] Option B: Si el CRM no permite reglas de scoring, documentar el cálculo de scoring como tabla y configurarlo via webhook/automation externa que actualice `lead_score`.
  [ ] Option C: Si ninguna opción es viable con el plan actual, marcar scoring como PARTIAL (manual) y documentar el upgrade necesario.

---

## Fase 3: setup-automations

### GATE IN
- [ ] Fase 2 completada — scoring y thresholds definidos, caso SQL verificado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Configurar la **automation rule principal** en el CRM:
   ```
   CUANDO  lead_score >= 60 (cruza a SQL)
   ENTONCES  mover el contacto a la etapa SQL del pipeline
   Y         notificar al founder (email / Slack / notificación in-CRM)
   ```
2. [ ] Verificar que la regla referencia credenciales del CRM SOLO via variables de entorno (`.env`) — NUNCA hardcodear tokens en la definición de la regla ni en `docs/sales/`.
3. [ ] Configurar automations secundarias acordadas con el founder (opcional según plan del CRM):
   - Lead cruza MQL (≥ 30) → asignar a secuencia de nurturing
   - Inactividad 14 días → restar −10 y, si baja de MQL, marcar para re-engagement
4. [ ] **Validar el escenario de aceptación end-to-end** (Gherkin escenario 1):
   ```
   Dado el funnel con threshold SQL en score >= 60
   Cuando un lead acumula 65 puntos (formulario +10, feature usada +15, pricing +10, demo +30)
   Entonces el CRM dispara la notificación al founder
   Y el lead aparece en el pipeline en etapa SQL
   ```
   Ejecutar el test de integración contra el sandbox del CRM si está disponible.

### CHECKPOINT
- [ ] Automation rule de SQL creada en el CRM (notifica al founder + mueve a etapa SQL)
- [ ] Credenciales del CRM referenciadas solo desde `.env` — sin secrets hardcodeados
- [ ] Escenario de aceptación verificado: lead de 65 puntos → notificación disparada + etapa SQL
- [ ] Automations secundarias configuradas o explícitamente diferidas

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: La automation rule no dispara la notificación al alcanzar SQL
Cause: Trigger mal configurado, credencial faltante en `.env`, o webhook del CRM sin firma/permiso.
Recovery:
  [ ] Option A: Revisar el trigger del CRM (condición `lead_score >= 60`) y reejecutar el test del sandbox con un lead de 65 puntos.
  [ ] Option B: Si falta la credencial, confirmar que existe en `.env` (NUNCA pedirla en el chat) y que la integración la lee correctamente.
  [ ] Option C: Si el CRM no permite la notificación nativa, configurar un webhook firmado que notifique al founder; si tampoco es viable, marcar la automation como PARTIAL y documentar el paso manual.

---

## Fase 4: generate-dashboard

### GATE IN
- [ ] Fase 3 completada — automation rule de SQL activa

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Definir las métricas del **dashboard de conversión** (de sales-crm-patterns.md), explicadas en lenguaje de negocio:
   - **CVR por etapa**: qué porcentaje pasa de una etapa a la siguiente — medir cada transición por separado:
     - Lead → MQL
     - MQL → SQL
     - SQL → Customer (o SQL → Opportunity → Customer si el funnel tiene Opportunity)
   - **CAC por canal**: cuánto cuesta conseguir un cliente, segmentado por canal de adquisición
   - **Tiempo promedio en cada etapa**: cuántos días tarda un contacto en avanzar de etapa
2. [ ] Definir el modelo de attribution para el CAC por canal (first-touch / last-touch / linear) según el ciclo de venta del producto.
3. [ ] **Validar el escenario de aceptación del dashboard** (Gherkin escenario 2):
   ```
   Dado que el dashboard está configurado
   Cuando el founder abre el dashboard de conversión
   Entonces ve la CVR de Lead→MQL, MQL→SQL y SQL→Customer
   Y ve el tiempo promedio de permanencia en cada etapa
   ```
4. [ ] Confirmar que las métricas se calculan desde datos del CRM (entidades Contact / Deal / Activity del modelo canónico), no desde valores inventados.

### CHECKPOINT
- [ ] Dashboard define CVR por etapa para las 3 transiciones (Lead→MQL, MQL→SQL, SQL→Customer)
- [ ] CAC por canal definido con modelo de attribution elegido
- [ ] Tiempo promedio en cada etapa incluido
- [ ] Escenario de aceptación verificado: el founder ve CVR por etapa + tiempo por etapa

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No hay datos suficientes para calcular las métricas del dashboard
Cause: Funnel recién configurado sin histórico, o el CRM no expone los datos de Activity/timestamps.
Recovery:
  [ ] Option A: Configurar el dashboard con las métricas definidas y marcar que se poblarán a medida que entren leads ("dashboard listo, esperando datos").
  [ ] Option B: Si el CRM no expone timestamps de transición, documentar el tiempo-por-etapa como "no disponible en este plan" y dejar CVR + CAC.
  [ ] Option C: Si el CRM no permite dashboards nativos, exportar las métricas definidas a `funnel-definition.md` como especificación para una herramienta externa (ej. Metabase/Looker).

---

## Fase 5: output

### GATE IN
- [ ] Fase 4 completada — dashboard definido y validado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Asegurar que el directorio existe:
   ```bash
   mkdir -p docs/sales/
   ```
2. [ ] Escribir `docs/sales/funnel-definition.md` con:
   - **Etapas del funnel** (Lead → MQL → SQL → Opportunity → Customer) y criterio de transición por etapa
   - **Tabla de lead scoring** con puntos por acción
   - **Thresholds por etapa** (MQL ≥ 30, SQL ≥ 60 — o los ajustados)
   - **Automation rules** configuradas (descripción de qué dispara la notificación SQL — SIN credenciales)
   - **Dashboard de conversión**: CVR por etapa, CAC por canal (con modelo de attribution), tiempo promedio por etapa
   - **Mapeo CRM**: qué `stage` y campos del CRM corresponden a cada etapa
3. [ ] Verificar que el documento NO contiene ningún secret, API key ni token del CRM — solo nombres de variables de `.env`.
4. [ ] Confirmar al founder dónde quedó el documento y resumir el funnel en 3 líneas de negocio.

### CHECKPOINT
- [ ] Archivo `docs/sales/funnel-definition.md` creado con todas las secciones
- [ ] Thresholds por etapa visibles y numéricos
- [ ] Sin secrets/credenciales del CRM en el documento
- [ ] Founder informado de la ubicación del documento

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede escribir docs/sales/funnel-definition.md
Cause: Permisos o path incorrecto.
Recovery:
  [ ] Option A: Crear el archivo en `docs/sales-funnel-definition.md` directamente.
  [ ] Option B: Imprimir el contenido completo en pantalla para guardado manual y registrarlo como PARTIAL.

---

## FINAL CHECKPOINT

- [ ] Funnel definido (5 etapas con criterios de transición)
- [ ] Lead scoring con tabla de puntos y thresholds (MQL ≥ 30, SQL ≥ 60)
- [ ] Automation rule de SQL activa (notifica al founder + mueve a etapa SQL)
- [ ] Dashboard de conversión con CVR por etapa, CAC por canal y tiempo por etapa
- [ ] `docs/sales/funnel-definition.md` creado
- [ ] Sin secrets/tokens del CRM hardcodeados — solo referencias a `.env`
- [ ] Sin scoring sobre PII sin consentimiento; secrets provider verificado si aplica

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS (secrets solo en `.env`, provider verificado), T: PASS (threshold SQL y dashboard verificables) |
| Artifacts | `docs/sales/funnel-definition.md` + automation rules en el CRM + dashboard de conversión |
| Next Recommended | Ver tabla de flujo |
| Risks | Scoring PARTIAL si el CRM no soporta automation nativa; dashboard "esperando datos" si funnel recién configurado |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

> **Origen DURO**: `← /crm-integrate` (requerido — el funnel se construye sobre el CRM y su mapeo de campos). **Origen típico en el flujo**: `← /email-sequences` (el funnel se conecta después de nutrir los leads).

| Condición | Próximo Skill |
|-----------|---------------|
| Funnel configurado y listo para escalar adquisición | `/go-to-market` (Bloque E — próximamente) |
| Funnel listo y se quiere mejorar la activación post-venta | `/customer-onboarding-flow` (Bloque E — próximamente) |
| `/crm-integrate` no estaba completo (BLOCKED) | Ejecutar `/crm-integrate` y volver |
| Scoring PARTIAL (CRM sin automation) | Upgrade del plan del CRM y re-ejecutar Fase 2-3 |

---

## REFERENCE

### Dependencia DURA con /crm-integrate
Este skill NO crea ni configura el CRM. Asume que `/crm-integrate` (M-88a) ya dejó: el CRM seleccionado, el mapeo de campos (incluido `lead_score` en Contact) en `docs/crm/field-mapping.md`, y el webhook de sincronización. El funnel, el scoring y las automations se diseñan SOBRE esa base. Si `docs/crm/field-mapping.md` no existe, es BLOCKING.

### Gate de seguridad (S)
- Los tokens/API keys del CRM viven SOLO en `.env`. El secrets provider de M07 (`.king/knowledge/secrets-provider.md`) es el mecanismo verificado para producción. Si el producto maneja PII y no hay provider → BLOCKED hasta resolverlo (configurar con M07 / `/tenancy-setup`).
- El lead scoring puntúa señales de **intención de compra** (acciones en el producto y en emails), NUNCA rasgos personales sensibles. Si los datos de leads incluyen PII sin consentimiento registrado (GDPR), no diseñar scoring sobre ellos hasta resolver la base legal.

### Vocabulario de negocio (para el founder)
- **Lead**: alguien que dejó su email. **MQL**: alguien interesado de verdad. **SQL**: alguien listo para que le vendas. **Opportunity**: ya estás negociando. **Customer**: te pagó.
- **CVR**: de cada 100 que entran a una etapa, cuántos pasan a la siguiente. **CAC**: cuánto te cuesta conseguir un cliente. **Lead scoring**: puntos automáticos que miden qué tan "caliente" está un contacto.

### Modelo de datos del CRM (canónico)
Contact (`contact_id`, `email`, `source`, `utm_*`, `lead_score`, `created_at`), Deal (`deal_id`, `contact_id`, `stage`, `amount`, `close_date`), Activity (`activity_id`, `contact_id`, `type`, `timestamp`). El scoring actualiza `lead_score` en Contact; el funnel mueve `stage` en Deal; el dashboard se calcula desde Activity y Deal. Ver `knowledge/_inject/sales-crm-patterns.md`.
