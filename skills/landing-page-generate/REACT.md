# Landing Page — React Template

> Sub-archivo de `/landing-page-generate`. Cargado por PHASE ROUTER cuando target=react.

---

## Estructura de componentes

```
src/
├── app/landing/page.tsx       — Página principal (Next.js App Router)
└── components/landing/
    ├── LandingHero.tsx
    ├── LandingFeatures.tsx
    ├── LandingCTA.tsx
    └── LandingFooter.tsx
```

---

## Página principal

```tsx
// src/app/landing/page.tsx
import { LandingHero } from '@/components/landing/LandingHero';
import { LandingFeatures } from '@/components/landing/LandingFeatures';
import { LandingCTA } from '@/components/landing/LandingCTA';

export const metadata = {
  title: '{PRODUCT_NAME} — {TAGLINE}',
  description: '{TAGLINE}',
};

export default function LandingPage() {
  return (
    <main>
      <LandingHero />
      <LandingFeatures />
      <LandingCTA />
    </main>
  );
}
```

---

## Hero component

```tsx
// src/components/landing/LandingHero.tsx
export function LandingHero() {
  return (
    <section className="flex flex-col items-center text-center py-24 px-6 max-w-4xl mx-auto">
      <h1 className="text-5xl font-bold leading-tight mb-6">
        {HEADLINE}
      </h1>
      <p className="text-xl text-gray-600 mb-8">{SUBHEADLINE}</p>
      <a
        href="#cta"
        className="bg-blue-600 text-white px-8 py-4 rounded-xl text-lg hover:bg-blue-700 transition"
      >
        {CTA_TEXT}
      </a>
    </section>
  );
}
```

---

## Features component

```tsx
// src/components/landing/LandingFeatures.tsx

// Static data — replace placeholders with actual content
const features = [
  { icon: '{ICON_1}', title: '{FEATURE_1_TITLE}', description: '{FEATURE_1_DESC}' },
  { icon: '{ICON_2}', title: '{FEATURE_2_TITLE}', description: '{FEATURE_2_DESC}' },
  { icon: '{ICON_3}', title: '{FEATURE_3_TITLE}', description: '{FEATURE_3_DESC}' },
];

export function LandingFeatures() {
  return (
    <section className="py-20 px-6 bg-gray-50">
      <div className="max-w-6xl mx-auto">
        <h2 className="text-3xl font-bold text-center mb-12">
          ¿Por qué {PRODUCT_NAME}?
        </h2>
        <div className="grid md:grid-cols-3 gap-8">
          {features.map((f) => (
            <div key={f.title} className="bg-white p-6 rounded-xl shadow-sm">
              <span className="text-3xl block mb-4">{f.icon}</span>
              <h3 className="text-xl font-semibold mb-2">{f.title}</h3>
              <p className="text-gray-600">{f.description}</p>
            </div>
          ))}
        </div>
      </div>
    </section>
  );
}
```

> **SEGURIDAD**: JSX escapa automáticamente el contenido de `{f.title}` y `{f.description}`.
> Nunca usar el prop de inyección de HTML crudo de React para datos que vienen del usuario.
> Mantener los datos de features como constantes estáticas en el archivo.

---

## CTA component

```tsx
// src/components/landing/LandingCTA.tsx
export function LandingCTA() {
  return (
    <section id="cta" className="py-20 px-6 text-center">
      <h2 className="text-3xl font-bold mb-4">{CTA_HEADLINE}</h2>
      <p className="text-gray-600 mb-8">{CTA_SUBTEXT}</p>
      <a
        href="{CTA_URL}"
        className="bg-blue-600 text-white px-8 py-4 rounded-xl text-lg hover:bg-blue-700 transition"
      >
        {CTA_TEXT}
      </a>
    </section>
  );
}
```
