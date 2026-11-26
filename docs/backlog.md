# Backlog por sprint

Fuente: tablero **PRODUCT BACKLOG** de Notion y *Product_Backlog_Mejorado-Equipo_2.6* (Drive). Los escenarios completos (Dado / Cuando / Entonces) se copian al issue de cada HU con la plantilla *Historia de usuario*.

Responsables: se asignan en Notion (columna *Responsable(s)*) y en el issue de GitHub.

## Sprint 1 · 27 nov – 2 dic 2026

| ID | Historia | Rol | Depende de | Notion |
| --- | --- | --- | --- | --- |
| SETUP | Esqueleto Next.js + FastAPI, Supabase local, login, `perfiles`, CI y deploy en Vercel | Equipo | — | — |
| HU01 | Registrar pedido (clienta con teléfono obligatorio, productos, cantidades, modalidad de pago) | Dueña | SETUP | [abrir](https://app.notion.com/p/424ba7e1c16e81caa1d9c48ae8909234) |
| HU02 | Verificar inventario automáticamente al guardar el pedido | Dueña | HU01 (tabla `productos`) | [abrir](https://app.notion.com/p/424ba7e1c16e8139804afa240a190033) |
| HU03 | Calcular total y si requiere cuota inicial (RN01) | Dueña | HU01 | [abrir](https://app.notion.com/p/424ba7e1c16e819a99b0dcfb8267d25b) |
| HU10 | Generar deuda automáticamente según el valor del pedido | Contador | HU01, HU03 | [abrir](https://app.notion.com/p/424ba7e1c16e81c5a631ff1c3c1ca07a) |
| HU11 | Consultar estado de cuenta y exportarlo a PDF | Contador | HU10 | [abrir](https://app.notion.com/p/424ba7e1c16e81298c92e161241a1ef3) |

Orden sugerido: SETUP primero (día 1). HU01 define `clientas`, `productos`, `pedidos` y `detalle_pedido`; HU02 y HU03 pueden avanzar en paralelo con las funciones de `backend/reglas/` mientras HU01 termina la migración. HU10 y HU11 empiezan con las reglas y vistas, y se conectan cuando HU01 esté en `main`.

> HU10 necesita los estados *Entregado* y *Cuota Inicial Abonada*, que llegan en el sprint 2 (HU04, HU08). En el sprint 1 se implementan el servicio `generar_deuda` y sus pruebas, y un disparador temporal para probarlo; en el sprint 2 se conecta a esas transiciones.

## Sprint 2 · 3 – 9 dic 2026

| ID | Historia | Rol | Depende de | Notion |
| --- | --- | --- | --- | --- |
| HU04 | Registrar cuota inicial ($10.000–$100.000, pago único) | Dueña | HU03, HU10 | [abrir](https://app.notion.com/p/424ba7e1c16e818ebf7ad854f1f5a79b) |
| HU05 | Autorizar compra al proveedor automáticamente (RN01) | Dueña | HU02, HU04 | [abrir](https://app.notion.com/p/424ba7e1c16e81d298b2df2952c69afa) |
| HU06 | Registrar proveedor (nombre, teléfono, productos) | Dueña | SETUP | [abrir](https://app.notion.com/p/424ba7e1c16e819abf54e7d2e61c5b47) |
| HU07 | Registrar llegada de productos (faltantes y calidad) | Contador | HU05, HU06 | [abrir](https://app.notion.com/p/424ba7e1c16e81549ce7d3c136340a91) |
| HU08 | Verificar despacho (existencias, pagos, condiciones de entrega, envío RN03) | Dueña | HU02, HU04, HU09 | [abrir](https://app.notion.com/p/424ba7e1c16e8151a43bfa132ff444ef) |
| HU09 | Registrar pago de contado | Dueña | HU01 | [abrir](https://app.notion.com/p/424ba7e1c16e8106b2abca0e566e8666) |

## Sprint 3 · 10 – 15 dic 2026

| ID | Historia | Rol | Depende de | Notion |
| --- | --- | --- | --- | --- |
| HU12 | Enviar estado de cuenta por WhatsApp cada quincena (Cloud API de Meta) | Contador | HU11, [ADR 0002](decisiones/0002-whatsapp-cloud-api.md) | [abrir](https://app.notion.com/p/424ba7e1c16e813f8a06e34d7e868d72) |
| HU13 | Agregar deudor automáticamente | Contador | HU10 | [abrir](https://app.notion.com/p/424ba7e1c16e819e8f8dd99902807d57) |
| HU14 | Registrar abono (RN04, RN05) | Contador | HU10 | [abrir](https://app.notion.com/p/424ba7e1c16e81898f71fce60a84570c) |
| HU15 | Actualizar estado de cuenta automáticamente (RN07) | Contador | HU11, HU14 | [abrir](https://app.notion.com/p/424ba7e1c16e819989c0fd68bfc134f4) |
| HU16 | Cerrar deuda automáticamente (RN08) | Contador | HU13, HU14 | [abrir](https://app.notion.com/p/424ba7e1c16e8192a6a2dc88b641bc1d) |

## Fuera del backlog actual

- **Notificaciones de pedido por WhatsApp** (cuota inicial pendiente, listo para recoger, despachado, abono registrado, deuda saldada): decididas en el [ADR 0002](decisiones/0002-whatsapp-cloud-api.md), sin HU todavía.
- **Preparación de Meta** (no es código, pero bloquea HU12): cuenta de Meta Business, número dedicado y plantillas enviadas a aprobación antes del 5 de diciembre.
- **Devoluciones y entregas fallidas** (RN02): están en el modelo verbal pero no tienen HU. Ver temas abiertos en [CONTEXTO.md](CONTEXTO.md).
