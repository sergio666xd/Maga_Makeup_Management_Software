# 0002 · WhatsApp con la Cloud API de Meta

- **Estado**: aceptada
- **Fecha**: 2026-11-26
- **Decide**: Sergio Muñoz Marín, con el equipo

## Contexto

HU12 pide que **el sistema envíe** por WhatsApp el estado de cuenta en PDF cada quincena. La revisión del backlog advirtió que abrir WhatsApp con un mensaje listo (`wa.me`) no es lo mismo que un envío del sistema. Además, el equipo quiere avisar a los clientes de cambios en su pedido.

## Decisión

Usar la **WhatsApp Cloud API de Meta** (Graph API) desde el backend FastAPI para:

1. **Estados de cuenta** (HU12): plantilla de categoría *utility* con encabezado de documento (el PDF) y el saldo en el cuerpo.
2. **Notificaciones de pedido**: plantillas *utility* para cambios de estado relevantes para el cliente.

Todo el código de WhatsApp vive en `backend/whatsapp/` y ningún otro módulo llama a Meta directamente.

```
backend/whatsapp/
├── cliente.py       # POST /{phone_number_id}/messages y subida de media
├── plantillas.py    # nombres, idioma (es_CO o es) y armado de parámetros
├── webhook.py       # router GET/POST /api/whatsapp/webhook
└── tests/
```

## Cómo funciona

- **Plantillas aprobadas**: fuera de la ventana de 24 horas desde el último mensaje del cliente, Meta solo deja enviar plantillas aprobadas. Todos los envíos del sistema son plantillas. Se crean en el WhatsApp Manager con categoría **utilidad** (no marketing): mensajes cortos, sobre una transacción concreta del cliente y sin promociones.
- **PDF**: se sube a la Cloud API como media (o se publica con un enlace firmado de corta duración) y se envía como encabezado `document` de la plantilla.
- **Teléfono**: se envía en formato E.164 (`57` + número). `backend/reglas/` normaliza el teléfono guardado del cliente.
- **Estados de entrega**: Meta avisa por webhook si el mensaje fue `sent`, `delivered`, `read` o `failed`. Se guardan en `envios_estado_cuenta` / `notificaciones` para cumplir HU12 E3 ("fecha del último envío").
- **Webhook**: `GET /api/whatsapp/webhook` responde el reto con `WHATSAPP_VERIFY_TOKEN`; `POST` valida la firma `X-Hub-Signature-256` con `WHATSAPP_APP_SECRET` antes de procesar nada.
- **Pruebas**: en CI y en local nunca se llama a Meta; `cliente.py` se reemplaza por un doble de prueba. Solo el número de prueba de Meta se usa en desarrollo.

## Plantillas propuestas (borrador, a aprobar en Meta)

| Nombre | Cuándo | Parámetros |
| --- | --- | --- |
| `estado_cuenta_quincenal` | Envío quincenal (HU12) | PDF, nombre, saldo, fecha de corte |
| `estado_cuenta_actualizado` | Envío individual o después de un abono | PDF, nombre, saldo |
| `pedido_cuota_inicial` | Pedido a crédito ≥ $100.000 queda esperando cuota | nombre, n.º de pedido, valor de la cuota |
| `pedido_listo_recoger` | Pedido pasa a *Listo para recoger* | nombre, n.º de pedido, fecha y hora acordadas |
| `pedido_despachado` | Pedido pasa a *Despachado* | nombre, n.º de pedido, fecha de entrega |
| `abono_registrado` | Se registra un abono | nombre, valor del abono, nuevo saldo |
| `deuda_saldada` | Deuda cerrada (saldo $0) | nombre, n.º de pedido |

Reglas de redacción: frases cortas, tuteo cordial, datos concretos (número de pedido, valor, fecha), sin emojis de más y sin lenguaje promocional, para que Meta las apruebe como utilidad.

## Variables de entorno (solo backend)

| Variable | Qué es |
| --- | --- |
| `WHATSAPP_ACCESS_TOKEN` | Token permanente de un usuario del sistema de Meta Business |
| `WHATSAPP_PHONE_NUMBER_ID` | ID del número que envía |
| `WHATSAPP_BUSINESS_ACCOUNT_ID` | ID de la cuenta de WhatsApp Business (WABA) |
| `WHATSAPP_APP_SECRET` | Para validar la firma del webhook |
| `WHATSAPP_VERIFY_TOKEN` | Texto acordado para verificar el webhook |
| `WHATSAPP_API_VERSION` | Versión de la Graph API, fijada en un solo lugar |

## Consecuencias

- Requiere una cuenta de **Meta Business**, una app con el producto WhatsApp y un número dedicado. Un número que ya se usa en la app WhatsApp Business del celular necesita migrarse (o usar la coexistencia que ofrezca Meta). Sergio/el Product Owner lo gestionan con la dueña.
- Las plantillas tardan en aprobarse: se envían a revisión **antes del sprint 3** (meta: 5 de diciembre).
- Meta cobra por mensaje de plantilla según la categoría y el país; el volumen quincenal es bajo, pero la dueña debe conocer el costo.
- Los clientes deben haber aceptado recibir mensajes por WhatsApp (opt-in). Se agrega `acepta_whatsapp` a `clientes`.
- Las notificaciones de pedido no están en ninguna HU: se proponen como HU nueva o como tarea técnica del sprint 3 si alcanza el tiempo.

## Alternativas consideradas

- **Enlace `wa.me`**: gratis e inmediato, pero el envío lo hace una persona; no cumple HU12 tal como está escrita.
- **Proveedores intermedios** (Twilio y similares): agregan costo y otra cuenta sin ventaja para este volumen.
