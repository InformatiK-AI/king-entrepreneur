# MVP Accelerator — Acceptance Testing

> Escenarios BDD del issue #61 para verificación manual del skill.

---

## Escenario 1: Happy Path — MVP SaaS desde cero (AC-1)

```gherkin
Given un entrepreneur con una idea de SaaS validada llamada "TaskFlow"
  And el proyecto está inicializado con /genesis (stack: Node.js + Next.js)
  And no existe auth, pagos ni deploy configurados
When ejecuta /mvp-accelerator
Then el orquestador muestra el plan de 4 pasos en lenguaje de negocio
  And ejecuta /auth-in-one-command → Login con Google configurado
  And ejecuta /payments-in-one-command → Stripe configurado con webhook
  And ejecuta /deploy-in-one-command → App configurada para Vercel
  And ejecuta /landing-page-generate → Landing page React generada
  And muestra el Execution Summary con los 4 componentes en COMPLETE
  And no solicita ninguna API key ni token en el chat en ningún momento
```

**Criterio de PASS**: El orquestador completa los 4 pasos sin preguntar sobre
técnicas de implementación. El usuario solo necesita confirmar y proveer
credenciales en sus respectivos dashboards (no en el chat).

---

## Escenario 2: Independencia de Skills (AC-2)

```gherkin
Given un proyecto existente con auth ya configurada (Next-Auth con Google)
When el entrepreneur ejecuta solo /payments-in-one-command
Then el skill detecta que no hay pagos configurados
  And pregunta por el provider (Stripe o Lemonsqueezy)
  And configura pagos sin preguntar sobre el auth existente
  And el webhook handler generado incluye validación de firma obligatoria
  And el .env.example solo tiene placeholders, sin valores reales
  And /auth-in-one-command NO fue invocado ni es necesario
```

**Criterio de PASS**: El BLOCKING CONDITION de payments-in-one-command no
menciona auth como prerequisito.

---

## Escenario 3: Fallo parcial — recovery sin abort (AC-1 + robustez)

```gherkin
Given un entrepreneur ejecutando /mvp-accelerator
  And el proyecto no tiene CLI de Vercel instalado
When /deploy-in-one-command retorna BLOCKED (CLI no instalado)
Then el orquestador PAUSA en Fase 4 (no aborta)
  And muestra el motivo del blocker en lenguaje de negocio:
    "El deploy necesita tu atención: el CLI de Vercel no está instalado"
  And ofrece opciones concretas de recovery
  And espera confirmación del usuario antes de continuar
  And el Execution Summary refleja: deploy=BLOCKED, landing=PENDING
```

**Criterio de PASS**: El usuario puede resolver el blocker y retomar desde
la fase 4 sin necesidad de reiniciar todo el flujo desde el principio.

---

## Escenario 4: Lenguaje de negocio (AC-3)

```gherkin
Given un entrepreneur sin experiencia técnica
When ejecuta /mvp-accelerator y pasa por todas las fases
Then ningún mensaje al usuario contiene las palabras:
  JWT, OAuth, PKCE, webhook, DNS, token (como dato técnico), endpoint,
  middleware, payload, hydration, bundle, CORS, SSL certificate
And cada paso se describe en términos de resultado de negocio:
  "Login con Google" (no "OAuth2+PKCE flow")
  "Sistema de pagos" (no "Stripe webhooks with HMAC-SHA256")
  "Publicar la app" (no "deploy con CI/CD pipeline")
  "Landing page" (no "SSR Next.js page with Tailwind")
```

**Criterio de PASS**: Un entrepreneur sin experiencia técnica puede seguir
el flujo completo sin necesitar explicaciones adicionales sobre los términos.

---

## Escenario 5: Regresión — auth-scaffold no afectado

```gherkin
Given un developer que usa /auth-scaffold (no /auth-in-one-command)
When otro usuario usa /auth-in-one-command en el mismo repo framework
Then /auth-scaffold sigue funcionando con todas sus opciones disponibles
  And /auth-in-one-command delega a /auth-scaffold sin modificarlo
  And las BLOCKING CONDITIONS de /auth-scaffold no cambian
  And los templates de auth-scaffold no son modificados por ningún skill S12
```

**Criterio de PASS**: `git diff develop skills/auth-scaffold/` no muestra
cambios en PR-B.

---

## Checklist de verificación manual

### Estructura (grep automatizable)
- [ ] mvp-accelerator/SKILL.md tiene BLOCKING CONDITIONS, PHASES OVERVIEW, FINAL CHECKPOINT
- [ ] Las 6 fases tienen GATE IN, MUST DO, CHECKPOINT, IF FAILS
- [ ] CASTLE declara `_·A·S·T·_·_`

### Security
- [ ] Ninguna fase solicita API keys, tokens o secrets
- [ ] ABSOLUTE RESTRICTIONS incluye prohibición de PHASE ROUTER no-lazy
- [ ] IF FAILS de auth/payments/deploy menciona recovery en lenguaje de negocio

### Orquestación
- [ ] Fases 2-5 cargan ONE sub-skill at a time (PHASE ROUTER lazy)
- [ ] GATE IN de Fase 3 requiere Fase 2 COMPLETE (no solo ejecutado)
- [ ] Fase 6 (summary) ejecuta sin importar el estado de las fases anteriores
