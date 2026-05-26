---
name: auth-in-one-command
version: 1.0
description: "Wrapper de auth-scaffold con defaults pre-configurados para entrepreneur: JWT + Google OAuth, session-based, sin ABAC/OPA. Configura autenticación en un comando sin preguntar 6 opciones."
---

# /auth-in-one-command — Auth para Entrepreneur

Wrapper thin sobre `/auth-scaffold` con defaults opinados para el arco entrepreneur. Elimina la configuración manual — el entrepreneur obtiene autenticación funcional con JWT + Google OAuth en un solo comando.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/auth-essentials.md` | S-AUTH checks, OWASP auth rules | Yes | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio para mensajes al usuario | Yes | framework |

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] Stack no detectado Y usuario no puede especificarlo
- [ ] Proyecto tiene auth existente Y usuario no especificó acción (auditar/reemplazar/abortar)

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA solicitar CLIENT_SECRET, CLIENT_ID, o JWT_SECRET en el chat — solo guiar al usuario a `.env`
- NUNCA duplicar lógica de auth-scaffold — delegación por lectura únicamente
- NUNCA generar código con secrets hardcodeados

### REQUIRED OUTPUTS
- [ ] Autenticación configurada en el proyecto del usuario (via auth-scaffold)
- [ ] `.env.example` con placeholders
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (detect-auth) → Phase 2 (pre-configure)
             → Phase 3 (delegate-to-auth-scaffold) → Phase 4 (verify)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/auth-in-one-command` es standalone si no hay workflow activo.

---

## Fase 1: detect-auth

### GATE IN
- [ ] Knowledge injection completada (auth-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Detectar stack del proyecto: leer `.king/knowledge/stack.md` o `package.json`
2. [ ] Verificar si ya existe autenticación:
   - Buscar archivos de auth: `middleware/auth.*`, `src/auth/*`, `lib/auth.*`, `services/auth.*`
   - Buscar en `package.json`: dependencias `passport`, `passport-*`, `jsonwebtoken`, `next-auth`, `lucia`, `better-auth`, `clerk`
3. [ ] Reportar al usuario en lenguaje de negocio:
   - Sin auth: "Vamos a configurar el login con Google para tu app."
   - Auth completa detectada: "Ya tenés login configurado. ¿Querés agregarlo a una parte nueva?"
   - Auth parcial (solo login, sin signup/logout): "Tenés un login parcial. Voy a completarlo."

### CHECKPOINT
- [ ] Stack detectado o confirmado por el usuario
- [ ] Estado del auth existente documentado (none | partial | complete)

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Stack no detectado
Cause: Sin `.king/knowledge/stack.md` y sin `package.json` identificable.
Recovery:
  [ ] Preguntar al usuario: "¿Tu app está hecha con Node.js, Python o Go?" — no continuar sin respuesta.

---

## Fase 2: pre-configure

### GATE IN
- [ ] Fase 1 completada — stack y estado auth conocidos

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Establecer defaults pre-configurados para entrepreneur (sin preguntar opciones):

   | Parámetro | Valor default | Razón |
   |-----------|--------------|-------|
   | Strategy | `jwt` | Stateless, más simple para MVPs |
   | Provider | `google` | Provider más común en SaaS |
   | Authz | `rbac` | Sin ABAC/OPA — reduce complejidad |
   | Mode | `full` | Login + signup + logout + /me |

2. [ ] Si auth detectada en Fase 1 (parcial o completa) → presentar opciones al usuario:
   ```
   Encontré configuración de autenticación existente. ¿Qué querés hacer?
   A) Auditar    — revisar el estado actual sin cambiar nada
   B) Reemplazar — configurar desde cero con login Google (DESTRUCTIVO — sobreescribe archivos)
   C) Abortar    — dejarlo como está
   ```
   - Usuario elige `auditar`    → ir a Fase 2b (reporte + cero cambios) → COMPLETE
   - Usuario elige `reemplazar` → continuar con Mode=replace en Fase 3
   - Usuario elige `abortar`    → Execution Summary COMPLETE sin modificar nada
   - Sin respuesta del usuario  → abortar (opción segura — ver BLOCKING CONDITIONS)
3. [ ] Comunicar al usuario qué se va a hacer:
   ```
   Voy a configurar:
   - Login con Google
   - Tokens de acceso seguros
   - Roles básicos (admin / usuario)

   ¿Empezamos?
   ```
4. [ ] Esperar confirmación del usuario

### CHECKPOINT
- [ ] Defaults documentados internamente
- [ ] Usuario confirmó la configuración

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Auth detectada — usuario no especificó acción
Cause: Respuesta ambigua o sin respuesta a las opciones A/B/C.
Recovery:
  [ ] Abortar como opción segura — Execution Summary COMPLETE sin cambios.
  [ ] Informar al usuario: para personalizar todas las opciones, usar /auth-scaffold directamente.

---

## Fase 2b: auditar (solo si usuario eligió A en Fase 2)

### GATE IN
- [ ] Fase 2 completada — usuario eligió opción `auditar`

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar archivos de auth presentes vs esperados para el stack del proyecto:
   - Esperados (Node.js + Google + JWT + RBAC): `src/auth/config.ts`, `src/auth/oauth.ts`, `src/auth/jwt.ts`, `src/auth/routes.ts`, `rbac.matrix.ts`, `.env.example`, `.gitignore` con `.env`/`*.pem`
   - Para otros stacks: adaptar rutas (`pkg/auth/` en Go, `auth/` en Python)
2. [ ] Generar reporte de estado en lenguaje de negocio — tabla de componentes:
   ```
   | Componente             | Estado                            |
   |------------------------|-----------------------------------|
   | Login con Google       | ✅ Configurado / ⚠️ Parcial / ❌ Faltante |
   | Tokens seguros (JWT)   | ...                               |
   | Roles básicos (RBAC)   | ...                               |
   | Variables de entorno   | ...                               |
   | .gitignore             | ...                               |
   ```
3. [ ] Mostrar reporte al usuario y sugerir próximos pasos: reemplazar (opción B) o abortar (opción C).

### GARANTÍA — ABSOLUTA
- Cero archivos creados, modificados o eliminados durante Fase 2b.
- Solo operaciones de lectura del estado existente del proyecto.

### CHECKPOINT
- [ ] Reporte de estado generado y mostrado al usuario
- [ ] Cero archivos modificados verificado

### IF FAILS
```
ERROR: No se pueden leer archivos del proyecto
Cause: Permisos o rutas incorrectas.
Recovery:
  [ ] Reportar archivos no accesibles como ESTADO DESCONOCIDO en el reporte.
  [ ] Completar el reporte con los componentes accesibles y continuar.
```

---

## Fase 3: delegate-to-auth-scaffold

### GATE IN
- [ ] Fase 2 completada — defaults establecidos y confirmados

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Leer `skills/auth-scaffold/SKILL.md` con los siguientes defaults pre-aplicados:
   - `--stack {stack detectado en Fase 1}`
   - `--providers google`
   - `--strategy jwt`
   - `--authz rbac`
   - `--mode {full | extend según Fase 1}`
2. [ ] Ejecutar auth-scaffold con esos parámetros sin preguntar opciones adicionales al usuario
3. [ ] Los mensajes al usuario durante auth-scaffold deben mantenerse en lenguaje de negocio

### CHECKPOINT
- [ ] auth-scaffold ejecutado con los defaults del entrepreneur
- [ ] Archivos de auth generados en el proyecto del usuario

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: auth-scaffold no puede ejecutarse con los parámetros dados
Cause: Stack no soportado por auth-scaffold, o conflicto con auth existente.
Recovery:
  [ ] Mostrar mensaje en lenguaje de negocio con las opciones disponibles. Si el problema es el stack, redirigir al usuario a `/auth-scaffold` para configuración manual.

---

## Fase 4: verify

### GATE IN
- [ ] Fase 3 completada — auth-scaffold ejecutado

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Verificar S-AUTH checks críticos (de auth-essentials.md):
   - S-AUTH-1: Sin secrets hardcodeados ✓
   - S-AUTH-6: `.gitignore` contiene `.env` y `*.pem` ✓
2. [ ] Confirmar al usuario en lenguaje de negocio:
   ```
   ¡Listo! Tu app ya tiene login con Google.
   Los usuarios van a poder registrarse e iniciar sesión.
   Siguiente paso: /payments-in-one-command para agregar pagos.
   ```

### CHECKPOINT
- [ ] S-AUTH-1 y S-AUTH-6 verificados
- [ ] Confirmación en lenguaje de negocio enviada al usuario

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: S-AUTH check falló
Cause: auth-scaffold generó código con secrets o sin .gitignore.
Recovery:
  [ ] Volver a Fase 3 y re-ejecutar auth-scaffold. Si persiste, escalar a `/auth-scaffold` manual.

---

## FINAL CHECKPOINT

- [ ] Auth configurada correctamente en el proyecto del usuario
- [ ] Sin secrets solicitados ni persistidos en el chat
- [ ] Mensajes al usuario en lenguaje de negocio
- [ ] Delegación a auth-scaffold sin duplicar lógica

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS |
| Artifacts | Archivos de auth generados, `.env.example` |
| Next Recommended | Ver tabla de flujo |
| Risks | None si S-AUTH-1 y S-AUTH-6 PASS |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Auth configurada exitosamente | `/payments-in-one-command` |
| Usuario quiere personalizar | `/auth-scaffold` (opciones completas) |
| Auth ya existía y completa | `/payments-in-one-command` directamente |
