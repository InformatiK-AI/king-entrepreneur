# Deploy — Fly.io Guide

> Sub-archivo de `/deploy-in-one-command`. Cargado por PHASE ROUTER cuando platform=fly.

---

## Archivo de configuración (fly.toml)

```toml
app = "tu-app-nombre"
primary_region = "iad"

[build]

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true

[[vm]]
  memory = "512mb"
  cpu_kind = "shared"
  cpus = 1
```

---

## Comandos de deploy

```bash
# 1. Instalar CLI (solo la primera vez)
curl -L https://fly.io/install.sh | sh

# 2. Autenticarse
fly auth login

# 3. Inicializar app (primera vez, genera fly.toml)
fly launch

# 4. Deploy
fly deploy

# 5. Verificar status
fly status
```

---

## Variables de entorno en Fly.io

```bash
# Configurar secrets (valores del .env)
fly secrets set DATABASE_URL="postgresql://..."
fly secrets set STRIPE_WEBHOOK_SECRET="whsec_..."
fly secrets set NEXTAUTH_SECRET="$(openssl rand -base64 32)"

# Listar secrets configurados
fly secrets list
```

---

## Scaling básico

```bash
# Ver máquinas activas
fly machines list

# Escalar a más regiones
fly regions add lhr syd

# Ajustar memoria
fly scale memory 1024
```

---

## Instrucciones en lenguaje de negocio

Al completar el deploy a Fly.io:

```
¡Tu app está online en Fly.io!
URL: https://tu-app-nombre.fly.dev

Para agregar tu dominio personalizado:
- fly certs add tu-dominio.com
- Fly te va a mostrar los registros DNS que necesitás configurar en tu dominio

Fly.io escala automáticamente según el tráfico. Empezás con 0 servidores cuando no hay usuarios,
y levanta instancias cuando llegan visitantes.
```
