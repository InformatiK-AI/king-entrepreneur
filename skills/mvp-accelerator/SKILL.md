---
name: mvp-accelerator
version: 1.0
description: "Orquestador principal del arco entrepreneur. Coordina auth-in-one-command + payments-in-one-command + deploy-in-one-command + landing-page-generate en secuencia. PHASE ROUTER lazy — carga un sub-skill a la vez."
---

# /mvp-accelerator — Lanzá tu MVP en 1 semana

Orquestador que coordina los 4 skills de ejecución del arco entrepreneur en orden. Auth → Pagos → Deploy → Landing. Cada skill se ejecuta solo cuando el anterior fue completado exitosamente.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio para mensajes | Yes | framework |

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] El proyecto no fue inicializado con `/genesis` (no existe `.king/knowledge/stack.md`)
- [ ] El usuario no puede articular el nombre del producto

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA cargar más de un sub-skill a la vez (PHASE ROUTER lazy — token budget)
- NUNCA continuar al siguiente sub-skill sin confirmar que el anterior completó exitosamente
- NUNCA solicitar credenciales de ningún tipo en el chat
- NUNCA abortar silenciosamente si un sub-skill falla — siempre pausar y mostrar recovery

### REQUIRED OUTPUTS
- [ ] Los 4 componentes del MVP configurados: auth + payments + deploy + landing
- [ ] Execution Summary con estado de cada componente
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (pre-check) → Phase 2 (auth)
             → Phase 3 (payments) → Phase 4 (deploy) → Phase 5 (landing)
             → Phase 6 (summary)
```

---

## CASTLE: _·A·S·T·_·_ — Capas A, S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path: si no hay workflow activo, crear uno para este MVP.

---

## Fase 1: pre-check

### GATE IN
- [ ] Knowledge injection completada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar que el proyecto fue inicializado (existe `.king/knowledge/stack.md`)
2. [ ] Preguntar el nombre del producto (si no es obvio del contexto):
   ```
   ¡Empezamos! Para lanzar tu MVP necesito una cosa:
   ¿Cómo se llama tu producto?
   ```
3. [ ] Mostrar el plan de lo que se va a hacer:
   ```
   Perfecto. Vamos a configurar {PRODUCT_NAME} en 4 pasos:
   1. Login con Google  — para que tus usuarios puedan registrarse
   2. Sistema de pagos  — para que puedan pagar
   3. Publicar la app   — para que esté online
   4. Landing page      — para que la gente te encuentre

   ¿Empezamos?
   ```
4. [ ] Esperar confirmación

### CHECKPOINT
- [ ] Proyecto inicializado verificado
- [ ] Nombre del producto definido
- [ ] Usuario confirmó el plan

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Proyecto no inicializado
Cause: No existe `.king/knowledge/stack.md`.
Recovery:
  [ ] Redirigir: "Primero necesitás inicializar tu proyecto con `/genesis`. Una vez listo, volvé acá."

---

## Fase 2: auth

### GATE IN
- [ ] Fase 1 completada — proyecto verificado y usuario confirmó plan

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 1/4: Configurando el login con Google...
   ```
2. [ ] Leer y ejecutar `skills/auth-in-one-command/SKILL.md` con los defaults pre-configurados
3. [ ] Al completar, reportar estado:
   - COMPLETE: "✅ Login configurado"
   - BLOCKED: "⚠️ El login necesita atención" + mostrar recovery del sub-skill

### CHECKPOINT
- [ ] auth-in-one-command ejecutado
- [ ] Estado reportado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: auth-in-one-command retornó BLOCKED
Cause: Stack no soportado, auth existente con conflicto, o error de configuración.
Recovery:
  [ ] Pausar: "El paso de login necesita tu atención. [mostrar recovery de auth-in-one-command]"
  [ ] Esperar al usuario. No continuar automáticamente.
  [ ] Cuando el usuario confirme que está resuelto, continuar a Fase 3.

---

## Fase 3: payments

### GATE IN
- [ ] Fase 2 completada — auth con Status: COMPLETE

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 2/4: Configurando el sistema de pagos...
   ```
2. [ ] Leer y ejecutar `skills/payments-in-one-command/SKILL.md`
3. [ ] Al completar, reportar estado

### CHECKPOINT
- [ ] payments-in-one-command ejecutado
- [ ] Estado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: payments-in-one-command retornó BLOCKED
Cause: Provider no detectado, webhook sin firma, o error de configuración.
Recovery:
  [ ] Pausar y mostrar recovery. No continuar hasta confirmación del usuario.

---

## Fase 4: deploy

### GATE IN
- [ ] Fase 3 completada — payments con Status: COMPLETE

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 3/4: Publicando tu app...
   ```
2. [ ] Leer y ejecutar `skills/deploy-in-one-command/SKILL.md`
3. [ ] Al completar, reportar estado

### CHECKPOINT
- [ ] deploy-in-one-command ejecutado
- [ ] Estado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: deploy-in-one-command retornó BLOCKED
Cause: Sin repositorio remoto, plataforma no configurada, o CLI no instalado.
Recovery:
  [ ] Pausar: "⚠️ El paso de publicación necesita tu atención. [mostrar recovery de deploy-in-one-command]"
  [ ] Presentar opciones al usuario:
      A) Resolver ahora — seguir las instrucciones de arriba y confirmar cuando esté listo → continuar a Fase 5
      B) Continuar sin publicar — configurar el deploy después con /deploy-in-one-command → marcar como SKIPPED y continuar a Fase 5
  [ ] Si usuario elige A → cuando confirme que resolvió, continuar a Fase 5 con deploy Status: COMPLETE.
  [ ] Si usuario elige B → marcar deploy como SKIPPED, continuar a Fase 5 con status PARTIAL.
  [ ] Si no responde → esperar (no abortar el pipeline ni continuar automáticamente).

---

## Fase 5: landing

### GATE IN
- [ ] Fase 4 ejecutada — deploy con Status: COMPLETE o SKIPPED (usuario eligió continuar sin deploy)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Anunciar el paso:
   ```
   Paso 4/4: Creando tu landing page...
   ```
2. [ ] Leer y ejecutar `skills/landing-page-generate/SKILL.md`
3. [ ] Al completar, reportar estado

### CHECKPOINT
- [ ] landing-page-generate ejecutado
- [ ] Estado: COMPLETE | BLOCKED

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: landing-page-generate retornó BLOCKED
Cause: Usuario no puede articular tagline/CTA, o framework no soportado.
Recovery:
  [ ] Pausar y mostrar recovery. La landing es opcional — el MVP funciona sin ella. Ofrecer hacerla después con `/landing-page-generate`.

---

## Fase 6: summary

### GATE IN
- [ ] Fases 2-5 ejecutadas (con cualquier estado)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar Execution Summary con estado de los 4 componentes:

   **{PRODUCT_NAME} — MVP completado**

   | Componente | Estado |
   |------------|--------|
   | Login con Google | ✅ CONFIGURADO / ⚠️ NECESITA ATENCIÓN |
   | Sistema de pagos | ✅ CONFIGURADO / ⚠️ NECESITA ATENCIÓN |
   | App publicada | ✅ ONLINE / ⚠️ NECESITA ATENCIÓN / ⏭️ PENDIENTE |
   | Landing page | ✅ LISTA / ⚠️ NECESITA ATENCIÓN |

   Para componentes BLOCKED: mostrar el motivo y el recovery path específico de PHASES.md.
   Para componentes SKIPPED (⏭️ PENDIENTE): mostrar "Ejecutar /deploy-in-one-command cuando tengas el repositorio listo."

2. [ ] Si todos están COMPLETE, agregar mensaje de cierre:
   "¡Tu MVP está listo. A conseguir los primeros usuarios!"
2b. [ ] Si deploy está SKIPPED y el resto COMPLETE, agregar:
   "¡Casi listo! Cuando publiques la app, tu MVP estará completo. A conseguir los primeros usuarios."

### CHECKPOINT
- [ ] Summary generado con estado de los 4 componentes
- [ ] Mensajes en lenguaje de negocio

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Summary no se puede generar — estado de fases no disponible
Cause: Una o más fases no reportaron su estado al completar.
Recovery:
  [ ] Construir el summary con la información disponible — marcar fases sin estado como UNKNOWN. No omitir el summary.

---

## FINAL CHECKPOINT

- [ ] Los 4 sub-skills ejecutados en orden (deploy puede tener estado SKIPPED si usuario eligió continuar sin publicar)
- [ ] Estado de cada componente documentado (COMPLETE | BLOCKED | SKIPPED)
- [ ] Summary presentado al usuario en lenguaje de negocio
- [ ] Sin credenciales solicitadas ni persistidas en el chat

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | A: PASS, S: PASS, T: PASS |
| Artifacts | Auth + Payments + Deploy + Landing configurados |
| Next Recommended | Ver tabla de flujo |
| Risks | Ver componentes BLOCKED en summary si los hay |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| 4/4 componentes COMPLETE | Lanzar a usuarios — iterar con feedback |
| Algún componente BLOCKED | Resolver blocker y re-ejecutar el sub-skill específico |
| Solo landing necesita ajuste | Re-ejecutar `/landing-page-generate` |
