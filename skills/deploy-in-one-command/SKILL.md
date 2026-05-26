---
name: deploy-in-one-command
version: 1.0
description: "Deploy automatizado a Vercel, Railway o Fly.io en un comando. Auto-detecta la plataforma por archivos del proyecto. Las credenciales nunca pasan por el chat — se usan los CLIs de cada plataforma."
---

# /deploy-in-one-command — Deploy para Entrepreneur

Publica tu app online con un comando. Detecta automáticamente si tu proyecto está configurado para Vercel, Railway o Fly.io, y te guía paso a paso para publicarlo. Las credenciales de las plataformas se configuran via sus propios CLIs — nunca se piden en el chat.

## Knowledge Injection

| File | Purpose | Required | Source |
|------|---------|----------|--------|
| `knowledge/_inject/devops-essentials.md` | GitFlow, deployment patterns | Yes | framework |
| `knowledge/_inject/onboarding-essentials.md` | Lenguaje de negocio | Yes | framework |

## QUICK REFERENCE

### BLOCKING CONDITIONS
> ⛔ Si alguna es TRUE, DETENER inmediatamente

- [ ] No hay repositorio git inicializado en el proyecto
- [ ] No hay git remote configurado → DETENER y guiar creación antes de continuar
- [ ] Las credenciales de plataforma son pedidas directamente en el chat → DETENER y redirigir al CLI

### ABSOLUTE RESTRICTIONS
> 🚫 Comportamientos absolutamente prohibidos — sin excepciones

- NUNCA solicitar tokens de Vercel, Railway o Fly.io en el chat
- NUNCA pedir credenciales de plataforma de ningún tipo en el chat
- Las credenciales de deployment NUNCA pasan por el contexto LLM — siempre via CLI de la plataforma

### REQUIRED OUTPUTS
- [ ] Configuración de deployment generada (vercel.json, railway.toml, fly.toml según plataforma)
- [ ] Variables de entorno documentadas para la plataforma elegida
- [ ] Instrucciones de deploy en lenguaje de negocio
- [ ] Session document creado (via session-management Phase N+1) — solo si workflow activo

### PHASES OVERVIEW
```
Phase 0 (Load) → Phase 1 (detect-platform) → Phase 2 (PHASE ROUTER)
             → Phase 3 (configure-platform) → Phase 4 (setup-env) → Phase 5 (deploy-guide)
```

---

## CASTLE: _·_·S·T·_·_ — Capas S y T activas

---

## Fase 0: Load Context

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase 0
> Fast-path activo: `/deploy-in-one-command` es standalone si no hay workflow activo.

---

## Fase 1: detect-platform

### GATE IN
- [ ] Knowledge injection completada (devops-essentials.md leído)

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Detectar plataforma por archivos del proyecto (en orden de preferencia):

   | Archivo detectado | Plataforma |
   |-------------------|-----------|
   | `vercel.json` | Vercel |
   | `railway.json` o `railway.toml` | Railway |
   | `fly.toml` | Fly.io |
   | `Dockerfile` (sin config de plataforma) | Railway (default contenedores) |
   | `next.config.*` sin archivos de plataforma | Vercel (default Next.js) |
   | Ninguno detectado | → preguntar al usuario (paso 2) |

2. [ ] Si no se detecta plataforma, preguntar en lenguaje de negocio:
   ```
   ¿Dónde querés publicar tu app?
   A) Vercel — ideal para apps web y Next.js, muy fácil de usar
   B) Railway — ideal si tu app tiene una base de datos o servicios en background
   C) Fly.io — ideal si necesitás servidores en múltiples regiones del mundo
   ```
3. [ ] Verificar si existe repositorio remoto (git remote):
   - Si no existe → BLOCKING: detener inmediatamente y mostrar instrucción de resolución:
     ```
     Antes de hacer el deploy necesitás publicar tu código en GitHub.
     Seguí estos pasos y volvé a invocar /deploy-in-one-command:
     1. Crear un repositorio en github.com/new
     2. git remote add origin https://github.com/TU-USUARIO/TU-REPO.git
     3. git push -u origin main
     ```
     No avanzar a Fase 2 hasta que el usuario confirme que el remote está configurado.
     Cero archivos de configuración generados hasta resolver.

### CHECKPOINT
- [ ] Plataforma identificada: `vercel` | `railway` | `fly`
- [ ] Estado del repositorio remoto documentado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede determinar la plataforma
Cause: Usuario no respondió la pregunta.
Recovery:
  [ ] Recomendar Vercel como default para apps web. Confirmar con el usuario.

---

## Fase 2: PHASE ROUTER

### GATE IN
- [ ] Fase 1 completada — plataforma identificada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Cargar el sub-archivo de la plataforma:
   - `vercel` → leer `skills/deploy-in-one-command/VERCEL.md`
   - `railway` → leer `skills/deploy-in-one-command/RAILWAY.md`
   - `fly` → leer `skills/deploy-in-one-command/FLY.md`

### CHECKPOINT
- [ ] Sub-archivo de plataforma cargado

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: Sub-archivo de plataforma no encontrado
Cause: `VERCEL.md`, `RAILWAY.md` o `FLY.md` no accesible.
Recovery:
  [ ] Verificar instalación del skill. Si persiste, guiar al usuario con los comandos básicos de la plataforma desde `knowledge/_inject/devops-essentials.md` directamente.

---

## Fase 3: configure-platform

### GATE IN
- [ ] Fase 2 completada — sub-archivo de plataforma disponible

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Generar o verificar archivo de configuración de la plataforma (usando el sub-archivo)
2. [ ] Si el archivo ya existe → verificar que las configuraciones recomendadas estén presentes
3. [ ] Si no existe → crear con la configuración mínima recomendada

### CHECKPOINT
- [ ] Archivo de configuración de la plataforma presente y con configuración correcta

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede generar configuración para la plataforma
Cause: Estructura del proyecto incompatible o framework no soportado.
Recovery:
  [ ] Mostrar al usuario la configuración mínima manual según el sub-archivo y guiar la creación.

---

## Fase 4: setup-env

### GATE IN
- [ ] Fase 3 completada — configuración de plataforma generada

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Identificar las variables de entorno necesarias para la plataforma
2. [ ] Verificar que existen en `.env.example` (creado por /auth-in-one-command y /payments-in-one-command si corrieron antes)
3. [ ] Instruir al usuario cómo configurar env vars en la plataforma:
   - Vercel: "En tu proyecto de Vercel → Settings → Environment Variables"
   - Railway: "En tu proyecto de Railway → Variables"
   - Fly.io: "Con el comando `fly secrets set NOMBRE=VALOR`"
4. [ ] Recordar al usuario: NUNCA pegar las keys reales aquí — solo en el dashboard de la plataforma

### CHECKPOINT
- [ ] Variables de entorno documentadas para la plataforma
- [ ] Sin credenciales solicitadas en el chat

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se puede crear o actualizar .env.example o .gitignore
Cause: Permisos de archivo o path incorrecto.
Recovery:
  [ ] Crear `.env.example` manualmente listando las variables requeridas para la plataforma con placeholders. Verificar que `.gitignore` existe y contiene `.env`.

---

## Fase 5: deploy-guide

### GATE IN
- [ ] Fase 4 completada — env vars documentadas

### MUST DO
> ⚠️ All actions are MANDATORY

1. [ ] Proporcionar comandos de deploy según la plataforma (del sub-archivo)
3. [ ] Confirmar al usuario:
   ```
   ¡Todo listo para publicar tu app!
   Seguí los pasos de arriba y en unos minutos tu app va a estar online.
   Siguiente paso: /landing-page-generate para crear tu página de ventas.
   ```

### CHECKPOINT
- [ ] Comandos de deploy proporcionados
- [ ] Instrucciones en lenguaje de negocio

### IF FAILS
> ❌ What to do when CHECKPOINT fails

ERROR: No se pueden generar comandos de deploy
Cause: Plataforma no inicializada o CLI no instalado.
Recovery:
  [ ] Guiar al usuario a instalar el CLI de la plataforma y autenticarse. Mostrar link de descarga.

---

## FINAL CHECKPOINT

- [ ] Plataforma detectada y configurada
- [ ] Sin credenciales de plataforma solicitadas en el chat
- [ ] Variables de entorno documentadas (no valores reales)
- [ ] Instrucciones de deploy en lenguaje de negocio

---

## Execution Summary

| Field | Value |
|-------|-------|
| Status | `COMPLETE` \| `PARTIAL` \| `BLOCKED` |
| CASTLE Verdict | S: PASS, T: PASS |
| Artifacts | Configuración de plataforma + guía de deploy |
| Next Recommended | Ver tabla de flujo |
| Risks | None si credenciales nunca pasaron por el chat |

---

## Fase N+1: Write Session

> Seguir instrucciones de `skills/session-management/SKILL.md` → Phase N+1
> Si skill standalone (sin workflow activo), omitir registro de sesión.

## Tabla de Flujo

| Condición | Próximo Skill |
|-----------|---------------|
| Deploy configurado exitosamente | `/landing-page-generate` |
| Usuario ya hizo el deploy | `/landing-page-generate` directamente |
| Problemas con la plataforma | Revisar documentación de la plataforma + reintentar |
