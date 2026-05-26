# Deploy — Vercel Guide

> Sub-archivo de `/deploy-in-one-command`. Cargado por PHASE ROUTER cuando platform=vercel.

---

## Archivo de configuración (vercel.json)

```json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "installCommand": "npm install",
  "regions": ["iad1"]
}
```

---

## Comandos de deploy

```bash
# 1. Instalar CLI (solo la primera vez)
npm i -g vercel

# 2. Autenticarse (abre el navegador)
vercel login

# 3. Vincular proyecto (solo la primera vez)
vercel link

# 4. Deploy a producción
vercel --prod
```

---

## Variables de entorno en Vercel

Configurar en Vercel Dashboard → Project → Settings → Environment Variables:

| Variable | Entorno |
|----------|---------|
| `DATABASE_URL` | Production |
| `NEXTAUTH_SECRET` | Production |
| `STRIPE_SECRET_KEY` | Production |
| Resto del `.env.example` | Production |

---

## Domain setup

```bash
# Agregar dominio personalizado
vercel domains add tu-dominio.com

# Verificar configuración DNS (mostrar registros requeridos)
vercel domains inspect tu-dominio.com
```

---

## Health check post-deploy

```bash
# Verificar que el deploy fue exitoso
curl -I https://tu-app.vercel.app

# Ver logs de la última función
vercel logs --follow
```

---

## Instrucciones en lenguaje de negocio

Al completar el deploy a Vercel:

```
¡Tu app está online en Vercel!
URL: https://tu-app.vercel.app

Para conectar tu dominio personalizado:
1. En tu proveedor de dominio, agregar un registro CNAME que apunte a cname.vercel-dns.com
2. En Vercel → Project → Settings → Domains → Add Domain

Cada vez que hagas cambios y los subas a GitHub, Vercel los publica automáticamente.
```
