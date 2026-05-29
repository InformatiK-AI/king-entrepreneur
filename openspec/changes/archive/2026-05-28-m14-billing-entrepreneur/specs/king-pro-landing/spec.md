# king-pro-landing — Delta Spec (M-96)

> Reusa Gherkin de `mejora/planes-detallados/M14-business-model-monetization.md §7 M-96`.

## ADDED Requirements

### Requirement: Landing de conversión King Pro

El skill `/king-pro-landing` MUST generar una landing page con Hero + pricing table de 3 tiers + FAQ + Footer,
orquestando `landing-page-generate` (sin duplicar su lógica). Los CTA de Pro/Team MUST apuntar a URLs de Stripe
Checkout válidas (no 404, no localhost, no placeholders).

#### Scenario: Generación exitosa de landing
- **GIVEN** se invoca `/king-pro-landing` en un proyecto React
- **AND** existe `design-essentials.md` con paleta de marca
- **WHEN** el skill completa todas las fases
- **THEN** genera landing con Hero + pricing table 3 tiers + FAQ + Footer
- **AND** usa la paleta de la marca (no defaults genéricos)
- **AND** guarda en `src/pages/landing.tsx` (o equivalente React)

#### Scenario: Pricing table con 3 tiers visibles
- **GIVEN** la landing fue generada
- **WHEN** un auditor hace review manual
- **THEN** encuentra 3 cards: Core ($0), Pro ($29/mes), Team ($99/mes)
- **AND** cada card lista los features incluidos
- **AND** los CTAs de Pro y Team apuntan a URLs de Stripe Checkout

#### Scenario: Generación sin design system disponible
- **GIVEN** no existe `design-essentials.md`
- **WHEN** el skill orquesta `landing-page-generate` (Fase inject-design)
- **THEN** usa defaults seguros (azul #2563EB, Inter, Tailwind)
- **AND** la landing se genera igualmente (esta fase es graceful)

#### Scenario: Links de checkout verificados
- **GIVEN** la landing fue generada con URLs de Stripe
- **WHEN** la Fase 5 (verify-checkout-links) verifica los links
- **THEN** todos los href de CTA son URLs válidas (no localhost, no placeholders)
- **AND** el skill confirma al usuario el path del archivo generado

#### Scenario: Orquesta sin duplicar landing-page-generate
- **GIVEN** el skill `king-pro-landing`
- **WHEN** se revisa su Fase 3
- **THEN** invoca `landing-page-generate` como motor de generación base
- **AND** no reimplementa la detección de framework ni la generación HTML/React/Vue
