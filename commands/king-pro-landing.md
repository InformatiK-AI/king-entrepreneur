---
name: king-pro-landing
description: "Genera la landing de conversión de King Pro: Hero + social proof + feature matrix + pricing table de 3 tiers (Core $0 / Pro $29 / Team $99) + FAQ + Footer, con CTAs a Stripe Checkout. Orquesta landing-page-generate (no duplica su lógica). M14 M-96."
argument-hint: "[--target html|react|vue] (auto-detectado si se omite)"
allowed-tools: [Read, Write, Edit, Skill]
---

# /king-pro-landing

Genera la landing page específica para convertir visitantes en suscriptores King Pro. Reutiliza
`landing-page-generate` como motor (detección de framework + generación + design system) y le añade lo
específico de producto: pricing table de 3 tiers, feature matrix comparativa, FAQ y CTAs que apuntan a Stripe
Checkout. Alimenta **CASTLE S** (sin price IDs/keys hardcodeados) y **T** (links de checkout verificados).

## Instrucciones

1. Invocar el skill `king-pro-landing` usando la herramienta Skill
2. Argumento opcional:
   - `--target <html|react|vue>`: framework de salida (si se omite, `landing-page-generate` lo auto-detecta del `package.json`)
3. Seguir las fases del skill en orden: load-pricing-data → collect-testimonials → generate-landing (orquesta `landing-page-generate`) → inject-pricing-table → verify-checkout-links
4. IMPORTANTE: NUNCA hardcodear price IDs ni keys; los CTAs de Pro/Team apuntan a URLs de Stripe Checkout
   (configuradas por `/payments-in-one-command --mode king-billing`); si no existen, se marcan con placeholder
   explícito; NUNCA duplicar la lógica de `landing-page-generate` — orquestarla

La pricing data oficial (Core $0 / Pro $29 / Team $99) proviene de `king-core/knowledge/universal/business-model.md`
§2; si no es accesible en runtime, el skill usa los valores oficiales embebidos.

## Secciones de la landing generada

1. **Hero**: "Build production-ready software in hours, not weeks" + CTA "Start free" + CTA "See pricing"
2. **Social proof**: logos/quotes de early adopters (placeholder si no hay)
3. **Feature matrix**: tabla comparativa Core vs Pro vs Team vs Enterprise
4. **Pricing table**: 3 cards (Core/Pro/Team) con CTA "Get Pro"/"Get Team" → Stripe Checkout
5. **FAQ**: 5 preguntas (BSL, cancelación, reembolso, equipos, enterprise)
6. **Footer**: links a docs, GitHub, Twitter, email

## Ejemplos

### Generar la landing (auto-detecta framework)

```
/king-pro-landing
```

### Forzar salida React

```
/king-pro-landing --target react
```

## Relación con otros skills

| Necesidad | Skill |
|-----------|-------|
| Configurar el billing al que apuntan los CTAs | `/payments-in-one-command --mode king-billing` |
| Publicar la landing | `/deploy-in-one-command` |
| Lanzar todo junto | `/mvp-accelerator` |
| Motor de generación base (orquestado) | `/landing-page-generate` |
