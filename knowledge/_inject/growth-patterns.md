---
inject: growth-patterns
version: 1.0
scope: universal
---

# Growth Patterns — Growth Engineering, SEO y Loops Virales

> Knowledge slim para inyección. Consumido por `/seo-foundations`, `/growth-loops`.

## Growth Engineering

| Etapa | Tácticas concretas | Métrica clave |
|-------|--------------------|---------------|
| Acquisition | SEO orgánico, referral, content, paid (último recurso) | CAC por canal |
| Activation | Onboarding guiado, time-to-value < 5 min, aha-moment temprano | Activation rate |
| Retention | Hábito (trigger→acción→reward), email lifecycle, re-engagement | D1/D7/D30 retention |
| Referral | Incentivo bilateral (referidor + referido), fricción mínima | K-factor |
| Revenue | Expansion (upsell/cross-sell), reducir churn | NRR, LTV |

**Regla**: optimizar retención ANTES que acquisition. Un balde con agujeros no se llena con más agua.

## SEO Essentials (technical)

- **Core Web Vitals targets**: LCP < 2500 ms · CLS < 0.1 · INP < 200 ms. Medir en campo (CrUX), no solo lab.
- **Indexabilidad**: `robots.txt` correcto, `sitemap.xml` regenerado por build, canonical URLs en todas las páginas, `hreflang` si multilingüe.
- **Metadata mínima por página**: `<title>` único (≤60 chars), `meta description` (≤155 chars), Open Graph + Twitter Card.
- **Structured data (JSON-LD) prioritario**: `Organization`, `WebSite`, `Product`/`Service`, `FAQPage`, `BreadcrumbList`. Validar con Google Rich Results Test (0 errores).

## AI Search Optimization

- Objetivo: ser citado por SGE (Google), Perplexity, ChatGPT Search.
- **`llms.txt`** (spec v0.1) en la raíz `/public/llms.txt`: nombre del producto, descripción de 1-2 líneas, features principales, URL canónica, links a docs. Markdown plano.
- Contenido respondible: encabezados como preguntas, respuestas directas en el primer párrafo, datos citables (tablas, listas).

## Loops Virales

| Concepto | Definición | Señal de salud |
|----------|-----------|----------------|
| Viral coefficient (K) | invitaciones × tasa de conversión por usuario | K > 1 = crecimiento viral |
| Cycle time | tiempo entre que un usuario entra y genera el próximo | menor = más rápido compuesto |
| Anti-fraude | rate-limit por IP, cap de redenciones por código, verificación de email del referido | bloquea abuso de rewards |

**Tipos de loop**: viral (share) · referral (reward) · content (UGC indexable) · network (más valor por usuario que se suma).

**Implementación mínima (referral)**: generar código único → tracking de atribución → reward solo tras activación real del referido (no solo signup) → rate-limiting anti-fraude en el endpoint de redención.
