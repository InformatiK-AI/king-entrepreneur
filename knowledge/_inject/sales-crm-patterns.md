---
inject: sales-crm-patterns
version: 1.0
scope: universal
---

# Sales & CRM Patterns — Funnel, Lead Scoring y Deliverability

> Knowledge slim para inyección. Consumido por `/sales-funnel-design`, `/crm-integrate`, `/email-sequences`.

## Funnel canónico

```
Lead → MQL → SQL → Opportunity → Customer
```

| Etapa | Definición | Criterio de transición típico |
|-------|-----------|-------------------------------|
| Lead | Contacto capturado (formulario, lead magnet) | tiene email + consent |
| MQL | Marketing Qualified — muestra interés | lead score ≥ 30 |
| SQL | Sales Qualified — listo para venta | lead score ≥ 60 o demo solicitada |
| Opportunity | Negociación activa | propuesta enviada |
| Customer | Pago confirmado | suscripción/compra activa |

## Lead Scoring

Sumar puntos por señal de intención. Threshold ajustable por producto.

| Acción | Puntos |
|--------|:------:|
| Formulario enviado | +10 |
| Email abierto | +2 |
| Link de email clickeado | +5 |
| Feature clave usada | +15 |
| Pricing page visitada | +10 |
| Demo solicitada | +30 |
| Inactividad 14 días | −10 |

**MQL** = score ≥ 30 · **SQL** = score ≥ 60. Notificar al founder cuando un lead cruza SQL.

## Funnel Metrics

- **CVR por etapa**: conversiones / entradas a la etapa. Medir cada transición por separado.
- **CAC** = gasto de adquisición / nuevos clientes. Segmentar por canal.
- **LTV** = ARPU × margen bruto × lifespan promedio.
- **Ratios saludables SaaS**: LTV/CAC ≥ 3 · CAC payback < 12 meses · NRR > 100%.

## Attribution Models

| Modelo | Asigna crédito a | Cuándo usar |
|--------|------------------|-------------|
| First-touch | Primer contacto | medir qué canal genera awareness |
| Last-touch | Último contacto antes de convertir | medir qué cierra |
| Linear | Reparte entre todos los touchpoints | ciclos de venta largos y multi-touch |

## Email Deliverability (prerequisito de envío)

- **DKIM + SPF + DMARC** publicados en DNS antes del primer envío masivo. Sin esto → spam folder.
- **Suppression list** activa desde el envío 1: unsubscribe, bounces duros, quejas de spam.
- **Bounce handling**: soft bounce → reintentar; hard bounce → suprimir permanentemente.
- Mantener bounce rate < 2% y complaint rate < 0.1%.

## CRM Data Model canónico

| Entidad | Campos mínimos |
|---------|----------------|
| Contact | `contact_id`, `email`, `name`, `source`, `utm_*`, `lead_score`, `created_at` |
| Company | `company_id`, `name`, `domain`, `size` |
| Deal | `deal_id`, `contact_id`, `stage`, `amount`, `close_date` |
| Activity | `activity_id`, `contact_id`, `type`, `timestamp` |
