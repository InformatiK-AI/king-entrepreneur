# Landing Page — Vue Template

> Sub-archivo de `/landing-page-generate`. Cargado por PHASE ROUTER cuando target=vue.

---

## Estructura de componentes

```
src/
├── views/LandingPage.vue
└── components/landing/
    ├── LandingHero.vue
    ├── LandingFeatures.vue
    ├── LandingCTA.vue
    └── LandingFooter.vue
```

---

## Página principal

```vue
<!-- src/views/LandingPage.vue -->
<template>
  <main>
    <LandingHero />
    <LandingFeatures />
    <LandingCTA />
    <LandingFooter />
  </main>
</template>

<script setup lang="ts">
import LandingHero from '@/components/landing/LandingHero.vue';
import LandingFeatures from '@/components/landing/LandingFeatures.vue';
import LandingCTA from '@/components/landing/LandingCTA.vue';
import LandingFooter from '@/components/landing/LandingFooter.vue';
</script>
```

---

## Hero component

```vue
<!-- src/components/landing/LandingHero.vue -->
<template>
  <section class="flex flex-col items-center text-center py-24 px-6 max-w-4xl mx-auto">
    <h1 class="text-5xl font-bold leading-tight mb-6">{{ headline }}</h1>
    <p class="text-xl text-gray-600 mb-8">{{ subheadline }}</p>
    <a href="#cta" class="bg-blue-600 text-white px-8 py-4 rounded-xl text-lg hover:bg-blue-700 transition">
      {{ ctaText }}
    </a>
  </section>
</template>

<script setup lang="ts">
const headline = '{HEADLINE}';
const subheadline = '{SUBHEADLINE}';
const ctaText = '{CTA_TEXT}';
</script>
```

> **SEGURIDAD**: Vue escapa automáticamente el contenido de `{{ variable }}`.
> Usar siempre `{{ }}` para texto, nunca `v-html` con datos del usuario sin sanitizar.

---

## Features component

```vue
<!-- src/components/landing/LandingFeatures.vue -->
<template>
  <section class="py-20 px-6 bg-gray-50">
    <div class="max-w-6xl mx-auto">
      <h2 class="text-3xl font-bold text-center mb-12">¿Por qué {{ productName }}?</h2>
      <div class="grid md:grid-cols-3 gap-8">
        <div v-for="feature in features" :key="feature.title"
             class="bg-white p-6 rounded-xl shadow-sm">
          <span class="text-3xl block mb-4">{{ feature.icon }}</span>
          <h3 class="text-xl font-semibold mb-2">{{ feature.title }}</h3>
          <p class="text-gray-600">{{ feature.description }}</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
const productName = '{PRODUCT_NAME}';

// Static data — replace with actual content
const features = [
  { icon: '{ICON_1}', title: '{FEATURE_1_TITLE}', description: '{FEATURE_1_DESC}' },
  { icon: '{ICON_2}', title: '{FEATURE_2_TITLE}', description: '{FEATURE_2_DESC}' },
  { icon: '{ICON_3}', title: '{FEATURE_3_TITLE}', description: '{FEATURE_3_DESC}' },
];
</script>
```

---

## CTA component

```vue
<!-- src/components/landing/LandingCTA.vue -->
<template>
  <section id="cta" class="py-20 px-6 text-center">
    <h2 class="text-3xl font-bold mb-4">{{ ctaHeadline }}</h2>
    <p class="text-gray-600 mb-8">{{ ctaSubtext }}</p>
    <a :href="ctaUrl" class="bg-blue-600 text-white px-8 py-4 rounded-xl text-lg hover:bg-blue-700 transition">
      {{ ctaText }}
    </a>
  </section>
</template>

<script setup lang="ts">
const ctaHeadline = '{CTA_HEADLINE}';
const ctaSubtext = '{CTA_SUBTEXT}';
const ctaUrl = '{CTA_URL}';
const ctaText = '{CTA_TEXT}';
</script>
```
