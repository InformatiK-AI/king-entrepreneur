# Deploy — Railway Guide

> Sub-archivo de `/deploy-in-one-command`. Cargado por PHASE ROUTER cuando platform=railway.

---

## Archivo de configuración (railway.toml)

```toml
[build]
builder = "nixpacks"

[deploy]
startCommand = "npm start"
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "on_failure"
restartPolicyMaxRetries = 3
```

---

## Comandos de deploy

```bash
# 1. Instalar CLI (solo la primera vez)
npm i -g @railway/cli

# 2. Autenticarse (abre el navegador)
railway login

# 3. Inicializar proyecto (solo la primera vez)
railway init

# 4. Deploy
railway up

# 5. Abrir la app
railway open
```

---

## Variables de entorno en Railway

```bash
# Configurar variables desde el .env.example
railway variables set DATABASE_URL="$DATABASE_URL"
railway variables set STRIPE_SECRET_KEY="$STRIPE_SECRET_KEY"

# Verificar variables configuradas
railway variables
```

O configurar en el Dashboard Railway → Project → Variables (pegar los valores del `.env`).

**Importante**: Los valores reales van en el Dashboard de Railway. Nunca en el código.

---

## Health check endpoint

Railway verifica `/health` por defecto. Agregar en el servidor:

```ts
// Node.js / Express
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});
```

---

## Instrucciones en lenguaje de negocio

Al completar el deploy a Railway:

```
¡Tu app está online en Railway!
URL: https://tu-servicio.railway.app

Railway va a redesplegar automáticamente cada vez que hagas push a GitHub.
Para agregar tu dominio personalizado:
- Railway → Project → Settings → Networking → Add Custom Domain
```
