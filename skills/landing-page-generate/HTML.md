# Landing Page — HTML Template

> Sub-archivo de `/landing-page-generate`. Cargado por PHASE ROUTER cuando target=html.

---

## Estructura de componentes

```
index.html
├── <head> — meta tags, title, Tailwind CDN
├── <nav> — logo + CTA button
├── <section id="hero"> — headline, subheadline, CTA button
├── <section id="features"> — 3 feature cards
├── <section id="testimonials"> — 2-3 testimonials (si hay)
├── <section id="cta"> — CTA final
└── <footer> — links básicos
```

---

## Template base

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{PRODUCT_NAME} — {TAGLINE}</title>
  <meta name="description" content="{TAGLINE}">
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="font-sans text-gray-900">

  <!-- Navigation -->
  <nav class="flex items-center justify-between p-6 max-w-6xl mx-auto">
    <span class="text-xl font-bold">{PRODUCT_NAME}</span>
    <a href="#cta" class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition">
      {CTA_TEXT}
    </a>
  </nav>

  <!-- Hero -->
  <section id="hero" class="text-center py-24 px-6 max-w-4xl mx-auto">
    <h1 class="text-5xl font-bold leading-tight mb-6">{HEADLINE}</h1>
    <p class="text-xl text-gray-600 mb-8">{SUBHEADLINE}</p>
    <a href="#cta" class="bg-blue-600 text-white px-8 py-4 rounded-xl text-lg hover:bg-blue-700 transition">
      {CTA_TEXT}
    </a>
  </section>

  <!-- Features -->
  <section id="features" class="py-20 px-6 bg-gray-50">
    <div class="max-w-6xl mx-auto">
      <h2 class="text-3xl font-bold text-center mb-12">¿Por qué {PRODUCT_NAME}?</h2>
      <div class="grid md:grid-cols-3 gap-8">
        <!-- Feature 1 -->
        <div class="bg-white p-6 rounded-xl shadow-sm">
          <div class="text-3xl mb-4">{ICON_1}</div>
          <h3 class="text-xl font-semibold mb-2">{FEATURE_1_TITLE}</h3>
          <p class="text-gray-600">{FEATURE_1_DESC}</p>
        </div>
        <!-- Feature 2 -->
        <div class="bg-white p-6 rounded-xl shadow-sm">
          <div class="text-3xl mb-4">{ICON_2}</div>
          <h3 class="text-xl font-semibold mb-2">{FEATURE_2_TITLE}</h3>
          <p class="text-gray-600">{FEATURE_2_DESC}</p>
        </div>
        <!-- Feature 3 -->
        <div class="bg-white p-6 rounded-xl shadow-sm">
          <div class="text-3xl mb-4">{ICON_3}</div>
          <h3 class="text-xl font-semibold mb-2">{FEATURE_3_TITLE}</h3>
          <p class="text-gray-600">{FEATURE_3_DESC}</p>
        </div>
      </div>
    </div>
  </section>

  <!-- CTA Final -->
  <section id="cta" class="py-20 px-6 text-center">
    <h2 class="text-3xl font-bold mb-4">{CTA_HEADLINE}</h2>
    <p class="text-gray-600 mb-8">{CTA_SUBTEXT}</p>
    <a href="{CTA_URL}" class="bg-blue-600 text-white px-8 py-4 rounded-xl text-lg hover:bg-blue-700 transition">
      {CTA_TEXT}
    </a>
  </section>

  <footer class="text-center py-8 text-gray-400 text-sm">
    &copy; 2025 {PRODUCT_NAME}. Todos los derechos reservados.
  </footer>

</body>
</html>
```

> **SEGURIDAD**: Todos los `{PLACEHOLDERS}` deben ser contenido estático — nunca interpolar
> datos dinámicos del usuario usando `.innerHTML` o `.insertAdjacentHTML()` sin sanitizar.
> Usar siempre `.textContent` para texto dinámico.
