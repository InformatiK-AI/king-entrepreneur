---
name: go-to-market
version: 1.0
api_version: 1.0.0
description: "Orquestador one-stop-shop del arco go-to-market. Encadena validate-idea + landing-page-generate + lead-magnets + crm-integrate + email-sequences + seo-foundations en secuencia. PHASE ROUTER lazy — carga un sub-skill a la vez, salta lo que ya existe, pausa en BLOCKED."
---

# /go-to-market — Sacá tu producto al mercado de punta a punta

Orquestador que coordina los 6 skills del arco de salida al mercado en orden. Validación de idea → Landing → Lead magnets → CRM → Email sequences → SEO. Cada skill se ejecuta solo cuando el anterior fue completado exitosamente, y se salta automáticamente si su artefacto ya existe.

## Knowledge Injection

Leer los siguientes archivos ANTES de la Fase 1. Si un archivo no existe, registrar una advertencia y continuar — aplica graceful degradation.

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio para los mensajes al founder | Yes | framework |

**Graceful degradation**: Si un archivo no existe, registrar una advertencia y continuar.

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El proyecto no fue inicializado con `/genesis` (no existe `.king/knowledge/stack.md`)
- [ ] El usuario no puede articular el nombre del producto

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA cargar más de un sub-skill a la vez (PHASE ROUTER lazy — token budget)
- NUNCA continuar al siguiente sub-skill sin confirmar que el anterior retornó `Status: COMPLETE`
- NUNCA solicitar credenciales, API keys, tokens ni webhook secrets de ningún tipo en el chat
- NUNCA abortar silenciosamente si un sub-skill retorna BLOCKED — siempre pausar y mostrar el recovery del sub-skill, esperar al founder

### REQUIRED OUTPUTS
- [ ] Los 6 componentes del go-to-market ejecutados: validación de idea + landing + lead magnets + CRM + email sequences + SEO
- [ ] Execution Summary con estado de los 6 componentes (COMPLETE | SKIPPED | BLOCKED)
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (pre-check + detect-existing)
             → Phase 2 (validate-idea)   → Phase 3 (landing-page-generate)
             → Phase 4 (lead-magnets)     → Phase 5 (crm-integrate)
             → Phase 6 (email-sequences)  → Phase 7 (seo-foundations)
             → Phase 8 (summary)
```

---

## CASTLE: _·A·S·T·_·_ — Capas A, S y T activas

> **A (Architecture)**: orquestación lazy y orden de dependencias del arco — un sub-skill a la vez, sin avanzar sin COMPLETE del anterior.
> **S (Security)**: ningún sub-skill recibe credenciales por el chat; los gates de seguridad de cada sub-skill (GDPR, firma de webhook, DKIM/SPF) se respetan.
> **T (Testing)**: cada componente reporta un estado verificable (COMPLETE | SKIPPED | BLOCKED) que el summary consolida.

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path: si no hay workflow activo, crear uno para este go-to-market (orquesta múltiples sub-skills en secuencia).

---

## Fase 1: pre-check + detect-existing

### GATE IN
- [ ] Knowledge injection completada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar que el proyecto fue inicializado (existe `.king/knowledge/stack.md`)
2. [ ] Determinar el nombre del producto (`PRODUCT_NAME`) — si existe `docs/business-spec/` o `docs/idea-validation/`, leer el artefacto más reciente y tomarlo del encabezado **Nombre del producto** (lectura semántica, no una clave literal). NO re-preguntar lo ya respondido. Si no hay artefacto, preguntar:
   ```
   ¡Vamos a sacar tu producto al mercado! Para empezar necesito una cosa:
   ¿Cómo se llama tu producto?
   ```
3. [ ] Detectar qué componentes ya existen (para saltarlos más adelante):
   ```bash
   test -d docs/idea-validation && echo "VALIDATE: existe" || echo "VALIDATE: falta"
   ```
   Para la landing, detectar según el stack (no solo `public/landing/`): revisar `public/landing*`, `app/(marketing)/`, `src/pages/landing*`, o leer la ruta real del artefacto de `/landing-page-generate` si existe. Registrar: si `docs/idea-validation/` existe → Validación = SKIPPED candidate; si se detecta una landing en cualquiera de esas rutas → Landing = SKIPPED candidate (no regenerar para no pisar el trabajo del founder).
4. [ ] Mostrar el plan del arco completo en lenguaje de negocio:
   ```
   Perfecto. Vamos a configurar el go-to-market de {PRODUCT_NAME} en 6 pasos:
   1. Validar la idea       — confirmar que hay mercado (lo salto si ya lo hiciste)
   2. Landing page          — tu página pública (la salto si ya existe)
   3. Captura de leads       — el formulario que llena tu base de contactos
   4. Conectar el CRM        — para que cada contacto llegue automático
   5. Emails automáticos     — para nutrir a esos contactos
   6. SEO                    — para que te encuentren en Google y buscadores con IA

   Voy paso a paso: termino uno antes de empezar el siguiente.
   ¿Empezamos?
   ```
5. [ ] Esperar confirmación del founder

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Proyecto inicializado verificado (`.king/knowledge/stack.md`)
- [ ] Nombre del producto definido (`PRODUCT_NAME`)
- [ ] Componentes existentes detectados (Validación / Landing → COMPLETE preexistente o ausente)
- [ ] Founder confirmó el plan

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Proyecto no inicializado
Cause: No existe `.king/knowledge/stack.md`.
Recovery:
  [ ] Option A: Redirigir: "Primero necesitás inicializar tu proyecto con `/genesis`. Una vez listo, volvé a ejecutar `/go-to-market`."
  [ ] Option B: Si el founder no puede articular el nombre del producto → pedir una frase de lo que hace y derivar un nombre provisional; sin un nombre mínimo, DETENER (Status: BLOCKED).

---

## Fase 2: validate-idea

### GATE IN
- [ ] Fase 1 completada — proyecto verificado y founder confirmó el plan

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Si `docs/idea-validation/` YA existe (detectado en Fase 1) → marcar Validación = SKIPPED (ya existía), informar "Tu idea ya está validada, salto este paso" y avanzar a Fase 3 SIN cargar el sub-skill.
2. [ ] Si NO existe → anunciar el paso:
   ```
   Paso 1/6: Validando tu idea...
   ```
3. [ ] Leer y ejecutar `skills/validate-idea/SKILL.md` (cargar UN sub-skill a la vez — no precargar los siguientes)
4. [ ] Al completar, leer el `Status` del Execution Summary del sub-skill y reportar:
   - COMPLETE: "✅ Idea validada"
   - BLOCKED: "⚠️ La validación necesita tu atención" + mostrar el recovery del sub-skill

### CHECKPOINT
> ✅ Verify before continuing

- [ ] validate-idea ejecutado (o SKIPPED si `docs/idea-validation/` ya existía)
- [ ] Estado registrado: COMPLETE | SKIPPED | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: validate-idea retornó BLOCKED
Cause: El founder no puede articular el problema/segmento, o el mercado no es identificable.
Recovery:
  [ ] Option A: Pausar y mostrar el recovery de validate-idea. NO avanzar a Fase 3 hasta resolución.
  [ ] Option B: Si la recomendación fue DESCARTAR/PIVOTAR → preguntar al founder si quiere ajustar la idea (re-ejecutar) o detener el arco aquí.
  [ ] Option C: Si no se resuelve → registrar Validación = BLOCKED, esperar confirmación del founder (no abortar el pipeline automáticamente).

---

## Fase 3: landing-page-generate

### GATE IN
- [ ] Fase 2 completada — validate-idea con Status: COMPLETE o SKIPPED

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Si la landing YA existe (detectado en Fase 1: `public/landing/` o equivalente) → marcar Landing = SKIPPED (ya existía), informar "Tu landing ya existe, salto este paso" y avanzar a Fase 4 SIN cargar el sub-skill.
2. [ ] Si NO existe → anunciar el paso:
   ```
   Paso 2/6: Creando tu landing page...
   ```
3. [ ] Leer y ejecutar `skills/landing-page-generate/SKILL.md`
4. [ ] Al completar, leer el `Status` del Execution Summary del sub-skill y reportar estado

### CHECKPOINT
> ✅ Verify before continuing

- [ ] landing-page-generate ejecutado (o SKIPPED si la landing ya existía)
- [ ] Estado registrado: COMPLETE | SKIPPED | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: landing-page-generate retornó BLOCKED
Cause: El founder no puede articular nombre/tagline/CTA, o el framework no es soportado.
Recovery:
  [ ] Option A: Pausar y mostrar el recovery del sub-skill. NO avanzar a Fase 4 hasta resolución.
  [ ] Option B: Esperar la confirmación del founder de que está resuelto → continuar a Fase 4 con Landing = COMPLETE.
  [ ] Option C: Si no se resuelve → registrar Landing = BLOCKED y esperar (no abortar el pipeline). La landing es upstream de lead-magnets y SEO; advertir que esos pasos dependen de ella.

---

## Fase 4: lead-magnets

### GATE IN
- [ ] Fase 3 completada — landing-page-generate con Status: COMPLETE o SKIPPED

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 3/6: Configurando la captura de leads...
   ```
2. [ ] Leer y ejecutar `skills/lead-magnets/SKILL.md`
3. [ ] Al completar, leer el `Status` del Execution Summary del sub-skill y reportar estado

### CHECKPOINT
> ✅ Verify before continuing

- [ ] lead-magnets ejecutado
- [ ] Estado registrado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: lead-magnets retornó BLOCKED
Cause: El founder rechaza el GDPR consent obligatorio, o no puede indicar el destino del CRM.
Recovery:
  [ ] Option A: Pausar y mostrar el recovery del sub-skill. NO avanzar a Fase 5 hasta que lead-magnets sea COMPLETE.
  [ ] Option B: Explicar al founder por qué el consent es obligatorio (riesgo regulatorio) y esperar su confirmación para re-ejecutar.
  [ ] Option C: Si no se resuelve → registrar Lead magnets = BLOCKED y esperar confirmación del founder. NO continuar a crm-integrate sin formularios.

---

## Fase 5: crm-integrate

### GATE IN
- [ ] Fase 4 completada — lead-magnets con Status: COMPLETE

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 4/6: Conectando tu CRM...
   ```
2. [ ] Leer y ejecutar `skills/crm-integrate/SKILL.md`
3. [ ] Al completar, leer el `Status` del Execution Summary del sub-skill y reportar estado

### CHECKPOINT
> ✅ Verify before continuing

- [ ] crm-integrate ejecutado
- [ ] Estado registrado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: crm-integrate retornó BLOCKED
Cause: Falta `.king/knowledge/secrets-provider.md` (M07), falta la API key del CRM, o el webhook no verifica firma.
Recovery:
  [ ] Option A: Pausar la orquestación y mostrar el recovery del sub-skill bloqueado (ej: configurar secrets provider con `/tenancy-setup`, o cargar la API key en `.env` — nunca en el chat).
  [ ] Option B: Esperar la confirmación del founder de que el blocker está resuelto. NO avanzar a Fase 6 (email-sequences) hasta que crm-integrate sea COMPLETE.
  [ ] Option C: Si no se resuelve → registrar CRM = BLOCKED y esperar. NO abortar el pipeline automáticamente ni continuar a la siguiente fase.

---

## Fase 6: email-sequences

### GATE IN
- [ ] Fase 5 completada — crm-integrate con Status: COMPLETE

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 5/6: Configurando los emails automáticos...
   ```
2. [ ] Leer y ejecutar `skills/email-sequences/SKILL.md`
3. [ ] Al completar, leer el `Status` del Execution Summary del sub-skill y reportar estado

### CHECKPOINT
> ✅ Verify before continuing

- [ ] email-sequences ejecutado
- [ ] Estado registrado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: email-sequences retornó BLOCKED
Cause: El dominio de envío no tiene DKIM/SPF/DMARC verificados, o no se puede determinar el dominio.
Recovery:
  [ ] Option A: Pausar y mostrar el recovery del sub-skill (instrucciones DNS del provider). NO avanzar a Fase 7 hasta que email-sequences sea COMPLETE.
  [ ] Option B: Esperar a que el founder configure los registros DNS y confirme → re-ejecutar el sub-skill.
  [ ] Option C: Si no se resuelve → registrar Email sequences = BLOCKED y esperar confirmación. NO abortar el pipeline.

---

## Fase 7: seo-foundations

### GATE IN
- [ ] Fase 6 completada — email-sequences con Status: COMPLETE
> Nota: la landing ya está garantizada COMPLETE/SKIPPED — si hubiera quedado BLOCKED en Fase 3, el pipeline ya estaría pausado ahí y nunca alcanzaría esta fase.

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 6/6: Optimizando el SEO de tu landing...
   ```
2. [ ] Leer y ejecutar `skills/seo-foundations/SKILL.md`
3. [ ] Al completar, leer el `Status` del Execution Summary del sub-skill y reportar estado

### CHECKPOINT
> ✅ Verify before continuing

- [ ] seo-foundations ejecutado
- [ ] Estado registrado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: seo-foundations retornó BLOCKED
Cause: El proyecto no tiene nombre, descripción ni URL canónica identificables.
Recovery:
  [ ] Option A: Pausar y mostrar el recovery del sub-skill. Capturar nombre/descripción/URL canónica con el founder y re-ejecutar.
  [ ] Option B: Si falta la URL canónica o la descripción del producto → capturarla con el founder y re-ejecutar (la landing ya está disponible por el orden del arco).
  [ ] Option C: Si no se resuelve → registrar SEO = BLOCKED y avanzar al summary con ese estado (es el último paso del arco).

---

## Fase 8: summary

### GATE IN
- [ ] Fases 2-7 ejecutadas (con cualquier estado: COMPLETE | SKIPPED | BLOCKED)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar el Execution Summary con el estado de los 6 componentes en lenguaje de negocio:

   **{PRODUCT_NAME} — Go-to-Market completado**

   | Componente | Estado |
   |------------|--------|
   | Validación de idea | ✅ COMPLETE / ⏭️ SKIPPED (ya existía) / ⚠️ BLOCKED |
   | Landing page | ✅ COMPLETE / ⏭️ SKIPPED (ya existía) / ⚠️ BLOCKED |
   | Lead magnets | ✅ COMPLETE / ⚠️ BLOCKED |
   | CRM integrado | ✅ COMPLETE / ⚠️ BLOCKED |
   | Email sequences | ✅ COMPLETE / ⚠️ BLOCKED |
   | SEO foundations | ✅ COMPLETE / ⚠️ BLOCKED |

   - Para componentes SKIPPED: indicar "ya existía, no se regeneró".
   - Para componentes BLOCKED: mostrar el motivo y el recovery path específico del sub-skill (el founder debe re-ejecutar ese sub-skill para resolverlo).

2. [ ] Si los 6 están COMPLETE o SKIPPED → mensaje de cierre:
   ```
   ¡Tu go-to-market está listo! Ya tenés todo para conseguir, capturar y
   nutrir a tus primeros clientes. A traer tráfico.
   ```
2b. [ ] Si algún componente quedó BLOCKED → cierre parcial:
   ```
   ¡Casi listo! Resolvé el paso marcado con ⚠️ ejecutando su comando, y tu
   go-to-market queda completo.
   ```

### CHECKPOINT
> ✅ Verify before continuing

- [ ] Summary generado con el estado de los 6 componentes
- [ ] Mensajes en lenguaje de negocio (sin jerga técnica)
- [ ] Componentes BLOCKED muestran su recovery path

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Summary no se puede generar — estado de fases no disponible
Cause: Una o más fases no reportaron su estado al completar.
Recovery:
  [ ] Option A: Construir el summary con la información disponible — marcar fases sin estado como UNKNOWN. NO omitir el summary.
  [ ] Option B: Re-leer el último estado reportado por cada fase y completar la tabla.

---

## FINAL CHECKPOINT

- [ ] Los 6 sub-skills procesados en orden (validate-idea, landing-page-generate, lead-magnets, crm-integrate, email-sequences, seo-foundations)
- [ ] Validación de idea y Landing: COMPLETE, o SKIPPED si su artefacto ya existía
- [ ] Estado de cada componente documentado (COMPLETE | SKIPPED | BLOCKED)
- [ ] Ningún sub-skill avanzó sin COMPLETE/SKIPPED del anterior; los BLOCKED pausaron el pipeline
- [ ] Summary presentado al founder en lenguaje de negocio
- [ ] Sin credenciales solicitadas ni persistidas en el chat

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | A: PASS, S: PASS, T: PASS |
| Artifacts | Validación + Landing + Lead magnets + CRM + Email sequences + SEO configurados |
| Next Recommended | Ver tabla de flujo |
| Risks | Ver componentes BLOCKED en el summary si los hay |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1

## Tabla de Flujo

> **Dependencias**: `/go-to-market` orquesta sub-skills que deben existir en el plugin. El orden respeta sus dependencias: lead-magnets antes de crm-integrate; landing antes de seo-foundations.

| Condición | Próximo Skill |
|-----------|---------------|
| 6/6 componentes COMPLETE/SKIPPED | Traer tráfico — `/analytics-setup` o `/growth-loops` |
| Algún componente BLOCKED | Resolver el blocker y re-ejecutar el sub-skill específico marcado en el summary |
| CRM bloqueado por secrets provider | `/tenancy-setup` (king-infra) y luego re-ejecutar `/crm-integrate` |
| Email bloqueado por DKIM/SPF | Configurar registros DNS del provider y re-ejecutar `/email-sequences` |

---

## REFERENCE

> 📚 Contexto adicional. Esta sección NO contiene acciones.

### Sub-skills orquestados (arco go-to-market)

| Orden | Sub-skill | Artefacto principal | Estado posible |
|-------|-----------|---------------------|----------------|
| 1 | `validate-idea` | `docs/idea-validation/` | COMPLETE / SKIPPED |
| 2 | `landing-page-generate` | `public/landing/` o equivalente | COMPLETE / SKIPPED |
| 3 | `lead-magnets` | `src/components/LeadForm.*`, `docs/leads/` | COMPLETE / BLOCKED |
| 4 | `crm-integrate` | `/api/crm/webhook`, `docs/crm/field-mapping.md` | COMPLETE / BLOCKED |
| 5 | `email-sequences` | secuencia 5 emails, `docs/email/sequences-map.md` | COMPLETE / BLOCKED |
| 6 | `seo-foundations` | `<head>` metadata, `public/sitemap.xml`, `docs/seo/audit-*.md` | COMPLETE / BLOCKED |

### Comportamiento lazy (PHASE ROUTER)

Igual que `/mvp-accelerator`: este orquestador carga UN sub-skill a la vez para respetar el token budget. NO precarga los siguientes. Antes de avanzar de una fase a la siguiente, verifica que el sub-skill anterior retornó `Status: COMPLETE` (o `SKIPPED` para validate-idea y landing si su artefacto ya existía). Si un sub-skill retorna `BLOCKED`, el orquestador PAUSA, muestra el recovery del sub-skill bloqueado, y espera la confirmación del founder antes de continuar — nunca avanza al siguiente sub-skill ni aborta el pipeline silenciosamente.

### Por qué SKIPPED y no re-ejecutar

`validate-idea` y `landing-page-generate` producen artefactos costosos de regenerar (un reporte de mercado, una landing customizada). Si el founder ya los tiene (`docs/idea-validation/` o `public/landing/`), regenerarlos pisaría su trabajo. Por eso el orquestador los detecta en Fase 1 y los marca SKIPPED en vez de volver a ejecutarlos. Los demás sub-skills (lead-magnets, crm-integrate, email-sequences, seo-foundations) se ejecutan siempre porque configuran integraciones, no documentos sobreescribibles.

### Relación con `/mvp-accelerator`

`/mvp-accelerator` orquesta el arco de CONSTRUCCIÓN del MVP (auth + pagos + deploy + landing). `/go-to-market` orquesta el arco de SALIDA AL MERCADO (validación + landing + captura + CRM + emails + SEO). Comparten el patrón PHASE ROUTER lazy y la landing como punto de contacto. Un founder típico corre `/mvp-accelerator` para construir y luego `/go-to-market` para vender.
