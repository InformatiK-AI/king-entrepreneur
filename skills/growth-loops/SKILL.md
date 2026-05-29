---
name: growth-loops
version: 1.0
api_version: 1.0.0
description: "Identifica e implementa loops de crecimiento orgánico (viral, referral, content, network): backend + frontend + analytics, con rate-limiting anti-fraude. Deja el código listo para revisar (PR-ready). Importa eventos de /analytics-setup."
---

# /growth-loops — Implementá tu Motor de Crecimiento Orgánico

Skill que recomienda el loop de crecimiento con mayor potencial para el producto del founder y lo implementa de punta a punta: backend de tracking/atribución, componentes frontend (share/invite/feed) y eventos de analytics. Deja el código listo para revisar (PR-ready) — código real, no solo recomendaciones; el developer lo integra al repo.

## Knowledge Injection

Read the following files BEFORE Phase 1. If a file does not exist, log a warning and continue — graceful degradation applies.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/growth-patterns.md` | Growth engineering, viral loops, K-factor, anti-fraude | Yes | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje por persona, progressive disclosure | Yes | framework |
| `.king/knowledge/stack.md` | Stack del proyecto para generar backend/frontend en el lenguaje correcto | No | project |
| `.king/knowledge/design-tokens.md` | Design tokens (M09) — componentes del loop por nombre de token | No | project |

**Graceful degradation**: If a file does not exist, log a warning and continue.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El founder no puede indicar el tipo de producto ni su segmento → pedir contexto antes de continuar

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA generar un loop de referral sin rate-limiting anti-fraude (por IP + cap de redenciones por código)
- NUNCA redefinir eventos que ya existen en `analytics/events.*` — IMPORTAR y EXTENDER
- NUNCA otorgar reward de referral sin activación real del referido (no solo signup)
- NUNCA hardcodear secrets/API keys — solo desde `.env`
- NUNCA exponer jerga técnica del loop sin contexto de negocio al founder

### REQUIRED OUTPUTS
- [ ] Loop recomendado/seleccionado con justificación (K-factor potencial)
- [ ] Backend del loop (tracking/atribución/referral/social graph según tipo)
- [ ] Componentes frontend del loop (share buttons / invite UI / feed / widget)
- [ ] Eventos del loop agregados a `analytics/events.*` (sin redefinir existentes)
- [ ] Rate-limiting anti-fraude en endpoints sensibles (referral)
- [ ] Instrucciones de integración al stack existente (PR listo para revisar)

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (identify-best-loop) → Phase 2 (implement-viral-referral)
             → Phase 3 (implement-content-network) → Phase 4 (output)
```

---

## CASTLE: _·A·S·T·_·_ — Capas A, S y T activas
> Capa A: el agente `developer` revisa el backend del loop antes de cerrar (ver `skills/_shared/castle-capas.md`).

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/growth-loops` es standalone si no hay workflow activo.

---

## Fase 1: identify-best-loop

### GATE IN
- [ ] Knowledge injection completada (growth-patterns.md leído — sección viral loops / K-factor)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Detectar contexto previo**: si existe `docs/business-spec/` o `docs/idea-validation/`, leer tipo de producto, segmento y SAM para no re-preguntarlos.
2. [ ] Leer el `loop` objetivo si el founder lo especificó: `viral` | `referral` | `content` | `network` | `all`.
3. [ ] Si NO se especificó, recomendar usando la tabla de decisión por tipo de producto y señales de mercado:

   | Loop | Mejor para | Señal |
   |------|-----------|-------|
   | Viral | Producto compartible, output visible | El uso genera artefactos que se comparten |
   | Referral | Alto LTV, incentivo costeable | El valor justifica pagar por cada referido |
   | Content | SEO/UGC, long tail | Los usuarios generan contenido indexable |
   | Network | Más valor por cada usuario que se suma | Efecto de red real (colaboración/social) |

4. [ ] Recomendar el loop con mayor **K-factor potencial** para ese contexto, con justificación en lenguaje de negocio.

### CHECKPOINT
- [ ] Tipo de producto y segmento identificados
- [ ] Loop seleccionado (por el founder o recomendado con justificación K-factor)
- [ ] Si recomendado: motivo explicado en lenguaje de negocio

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede recomendar un loop
Cause: Falta el tipo de producto o el patrón de uso.
Recovery:
  [ ] Option A: Preguntar "¿Tu producto genera algo que el usuario quiera mostrar, invita a otros, crea contenido, o gana valor con cada usuario nuevo?" para mapear al loop.
  [ ] Option B: Recomendar Referral como opción de menor riesgo (aplicable a casi todo SaaS) y documentar el supuesto.

---

## Fase 2: implement-viral-referral

### GATE IN
- [ ] Fase 1 completada — loop seleccionado es `viral`, `referral` o `all`

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Viral** (si aplica):
   - Backend: share tracking con **attribution links únicos** por usuario/contenido
   - Frontend: share buttons (Twitter/X, WhatsApp, email) + **OG tags dinámicos** por página compartida
   - Analytics: agregar `share_initiated`, `share_converted` a `analytics/events.*` (importar, no redefinir) + cálculo de viral coefficient
2. [ ] **Referral** (si aplica):
   - Definir con el founder qué cuenta como **activación real** en su producto (ej: onboarding completado, feature clave usada, pago confirmado — nunca solo signup)
   - Backend: generación de referral code + distribución de reward (credit/discount) SOLO tras esa activación real del referido
   - Frontend: invite friends UI + referral dashboard + reward status
   - Analytics: agregar `referral_sent`, `referral_activated`, `reward_granted`
   - **Anti-fraude (OBLIGATORIO)**: rate-limiting por IP en el endpoint de redención + máximo configurable de redenciones por código
3. [ ] Si el loop es `content` o `network`, saltar a Fase 3 (esta fase no aplica).

### CHECKPOINT
- [ ] Backend del loop generado (attribution links / referral codes según tipo)
- [ ] Frontend viral (si aplica): share buttons (Twitter/X, WhatsApp, email) + OG tags dinámicos
- [ ] Frontend referral (si aplica): invite UI + referral dashboard + reward status
- [ ] Eventos agregados a `analytics/events.*` sin redefinir existentes
- [ ] Referral: rate-limiting anti-fraude presente + reward solo tras activación real

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El loop no se puede implementar sobre el stack actual
Cause: Stack no detectado o sin backend para tracking/referral.
Recovery:
  [ ] Option A: Detectar el stack (`.king/knowledge/stack.md` o inspección) y generar para ese framework.
  [ ] Option B: Generar el backend en formato agnóstico con instrucciones de integración, marcando lo que el developer debe conectar.

---

## Fase 3: implement-content-network

### GATE IN
- [ ] Fase 1 completada — loop seleccionado es `content`, `network` o `all`

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] **Content** (si aplica):
   - Frontend: páginas UGC SEO-optimizadas + testimonial widget
   - Analytics: agregar `ugc_created`, `ugc_viewed` + fuente de tráfico orgánico
   - (Sin backend dedicado — apoyarse en el CMS/DB existente)
2. [ ] **Network** (si aplica):
   - Backend: social graph (follow/connect) + queries de activity feed
   - Frontend: activity feed component + social proof ("X amigos usan esto")
   - Analytics: agregar `connection_made`, `network_activated` + DAU/MAU
3. [ ] Si el loop es `viral` o `referral` puro, esta fase no aplica.
4. [ ] Si hay `design-tokens.md`, los componentes frontend referencian tokens por nombre.

### CHECKPOINT
- [ ] Componentes content/network generados según el loop
- [ ] Eventos agregados a `analytics/events.*` sin redefinir existentes
- [ ] Tokens referenciados por nombre si existen

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El producto no soporta el loop content/network
Cause: No hay CMS/DB para UGC o no hay relación entre usuarios para social graph.
Recovery:
  [ ] Option A: Proponer el mínimo viable (testimonial widget para content; social proof estático para network) y documentar lo que falta.
  [ ] Option B: Recomendar cambiar a un loop viral/referral que sí encaje con el producto actual.

---

## Fase 4: output

### GATE IN
- [ ] Fase 2 y/o Fase 3 completadas — loop implementado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar las **instrucciones de integración** al stack existente: dónde montar el backend, dónde insertar los componentes frontend, qué variables `.env` configurar.
2. [ ] Resumir el loop implementado, los eventos agregados y cómo medir el K-factor.
3. [ ] Dejar el cambio listo para revisar (PR-ready): backend + frontend + analytics en pasos verificables.
4. [ ] **CASTLE capa A**: invocar al agente `developer` para revisar el backend del loop (tracking/atribución/anti-fraude) antes de cerrar.

### CHECKPOINT
- [ ] Instrucciones de integración generadas
- [ ] Resumen del loop + cómo medir K-factor
- [ ] Backend revisado por el agente `developer` (capa A)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: El agente developer marca un problema en el backend del loop
Cause: Riesgo de fraude, race condition en reward, o atribución incorrecta.
Recovery:
  [ ] Option A: Aplicar la corrección señalada por el agente y re-verificar.
  [ ] Option B: Marcar el hallazgo como riesgo en el output y NO otorgar rewards automáticos hasta resolverlo.

---

## FINAL CHECKPOINT

- [ ] Loop seleccionado/recomendado con justificación K-factor
- [ ] Backend + frontend + analytics del loop generados
- [ ] Eventos importados de `analytics/events.*` sin redefinir
- [ ] Referral con rate-limiting anti-fraude + reward solo tras activación real
- [ ] Backend revisado por agente `developer` (CASTLE capa A)
- [ ] Instrucciones de integración (PR listo para revisar)

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | A: PASS (backend revisado), S: PASS (anti-fraude + sin secrets), T: PASS (eventos tipados) |
| Artifacts | Backend del loop, componentes frontend, `analytics/events.*` extendido, instrucciones de integración |
| Next Recommended | Ver tabla de flujo |
| Risks | Fraude de referral si el anti-fraude se desactiva; K-factor real sin medir hasta producción |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Loop implementado, medir resultados | `/analytics-setup` (si aún no está) o revisar dashboards |
| Loop listo, falta capturar leads del tráfico | `/lead-magnets` |
| Loop implementado, validar el backend | `/review` |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Dependencia con `/analytics-setup`

`/growth-loops` corre DESPUÉS de `/analytics-setup` (Bloque B) porque comparte `analytics/events.*`. Este skill IMPORTA los eventos existentes y AGREGA solo los del loop (`share_initiated`, `referral_sent`, etc.) — nunca los redefine, para evitar double-counting en los funnels. Ver naming convention en `analytics-setup`.

### Anti-fraude en referral (growth-patterns.md)

El reward de referral es el vector de fraude más común. Mitigaciones obligatorias (de `growth-patterns.md`): rate-limiting por IP en el endpoint de redención, cap configurable de redenciones por código, verificación de email del referido, y reward SOLO tras activación real (no signup). Sin esto, el loop se explota con cuentas falsas.

### K-factor

K = invitaciones por usuario × tasa de conversión de la invitación. K > 1 = crecimiento viral autosostenido. El skill instrumenta los eventos necesarios para calcularlo; el valor real solo se mide en producción con tráfico real.
