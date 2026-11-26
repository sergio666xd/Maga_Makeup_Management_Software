# Arquitectura

Decisión de stack: [decisiones/0001-stack.md](decisiones/0001-stack.md). Contexto del negocio: [CONTEXTO.md](CONTEXTO.md).

## Vista general

```
                     Un solo proyecto de Vercel
 ┌──────────────────────────────────────────────────────────────────┐
 │                                                                  │
 │  Navegador ──▶ Next.js (src/)  ──fetch /api/* + JWT──▶ FastAPI    │
 │   Dueña /       páginas, formularios,                (api/index.py│
 │   Contador      login con Supabase Auth               + backend/) │
 │                                                          │       │
 └──────────────────────────────────────────────────────────┼───────┘
                │ login (supabase-js)                       │ SQL (SQLAlchemy + psycopg)
                ▼                                           ▼
          Supabase Auth  ─────────────────────────▶  Supabase Postgres
                                                     (migraciones en supabase/migrations)
```

- **Next.js** solo pinta pantallas y llama a la API. Usa Supabase únicamente para iniciar y cerrar sesión.
- **FastAPI** tiene toda la lógica: valida, aplica las reglas de negocio y escribe en la base de datos. Verifica el JWT de Supabase en cada petición y lee el rol (`duena` o `contador`) de la tabla `perfiles`.
- **Postgres** guarda los datos. RLS está activo en todas las tablas y sin políticas para `anon`, de modo que nadie puede leer datos de negocio con la llave pública; el backend entra con su propia conexión.

### Enrutamiento

| Entorno | `/` y páginas | `/api/*` |
| --- | --- | --- |
| Local | `next dev` en `:3000` | `next.config.ts` reescribe a `http://127.0.0.1:8000/api/*` (uvicorn) |
| Vercel | Next.js | `vercel.json` envía `/api/(.*)` a la función Python `api/index.py` |

Por eso **todas** las rutas de FastAPI empiezan con `/api` y Next.js no define rutas propias bajo `/api`. La tarea SETUP valida este enrutamiento con un deploy de preview.

## Estructura de carpetas

```
.
├── api/
│   └── index.py              # Punto de entrada para Vercel: expone `app` de backend.main
├── backend/                  # Paquete Python (FastAPI)
│   ├── main.py               # Crea la app, CORS, incluye routers con prefijo /api
│   ├── config.py             # Variables de entorno (pydantic-settings)
│   ├── db.py                 # Engine y sesión de SQLAlchemy
│   ├── auth.py               # Verifica JWT de Supabase y rol (dependencias de FastAPI)
│   ├── modelos/              # Tablas mapeadas con SQLAlchemy
│   ├── schemas/              # Entradas y salidas Pydantic por módulo
│   ├── reglas/               # Reglas de negocio puras (sin base de datos)
│   │   ├── constantes.py
│   │   ├── pedido.py         # total, cuota inicial (RN01)
│   │   ├── inventario.py     # disponibilidad y faltantes
│   │   ├── envio.py          # RN02, RN03
│   │   └── cartera.py        # deuda, abonos, cierre (RN05, RN07, RN08)
│   ├── servicios/            # Casos de uso: reglas + base de datos en una transacción
│   ├── whatsapp/             # Cloud API de Meta: cliente, plantillas, webhook (ADR 0002)
│   ├── routers/              # Endpoints por módulo
│   │   ├── clientas.py  pedidos.py  inventario.py  proveedores.py
│   │   ├── compras.py   despacho.py  cartera.py   estados_cuenta.py
│   └── tests/
│       ├── reglas/           # pruebas unitarias por HU y escenario
│       └── api/              # pruebas de endpoints
├── src/                      # Next.js
│   ├── app/
│   │   ├── login/
│   │   └── (panel)/          # rutas protegidas
│   │       ├── pedidos/  clientas/  inventario/  proveedores/
│   │       ├── compras/  despacho/  cartera/
│   ├── components/
│   │   └── ui/               # componentes shadcn/ui (compartidos)
│   └── lib/
│       ├── api.ts            # cliente de la API con el token de sesión
│       ├── formato.ts        # formato de pesos y fechas
│       └── supabase/         # clientes de Supabase solo para Auth
├── supabase/
│   ├── config.toml
│   ├── migrations/           # SQL versionado (única forma de cambiar el esquema)
│   └── seed.sql              # datos ficticios para desarrollo
├── docs/
├── next.config.ts  package.json  pyproject.toml  uv.lock  vercel.json
└── .env.example
```

## Modelo de datos (propuesta inicial)

Cada HU crea las tablas que necesita en su migración. Esta es la forma acordada para que no choquen. Dinero en `bigint` (pesos), fechas en `timestamptz`.

| Tabla | Columnas principales | HU |
| --- | --- | --- |
| `perfiles` | `user_id` (FK auth.users), `nombre`, `rol` (`duena` \| `contador`) | SETUP |
| `clientas` | `id`, `nombre`, `telefono` (único, obligatorio), `centro_comercial`, `local`, `direccion`, `acepta_whatsapp`, `creado_en` | HU01, HU13 |
| `productos` | `id`, `referencia` (única), `nombre`, `valor_venta`, `existencia` (≥ 0) | HU02, HU07 |
| `proveedores` | `id`, `nombre`, `telefono` (único), `creado_en` | HU06 |
| `proveedor_productos` | `proveedor_id`, `producto_id` | HU06 |
| `pedidos` | `id`, `consecutivo` (serial), `clienta_id`, `modalidad_pago` (`contado` \| `credito`), `estado`, `estado_pago`, `total`, `cuota_inicial`, `requiere_compra`, `modalidad_entrega` (`domicilio` \| `bodega`), `direccion_entrega`, `fecha_entrega_acordada`, `costo_envio`, `mensajero_id`, `creado_en` | HU01–HU05, HU08, HU09 |
| `detalle_pedido` | `pedido_id`, `producto_id`, `cantidad` (> 0), `valor_unitario` | HU01, HU03 |
| `pagos` | `id`, `pedido_id`, `tipo` (`cuota_inicial` \| `contado`), `valor`, `fecha`, `medio`, `comprobante`, `registrado_por` | HU04, HU09 |
| `compras` | `id`, `proveedor_id`, `pedido_id` (opcional), `fecha`, `estado` | HU05, HU07 |
| `detalle_compra` | `compra_id`, `producto_id`, `cantidad_comprada`, `cantidad_recibida`, `calidad` (`aprobada` \| `rechazada`), `observaciones`, `precio_compra` | HU07 |
| `mensajeros` | `id`, `nombre`, `telefono` | HU08 |
| `deudas` | `id`, `pedido_id` (único), `clienta_id`, `valor_total`, `saldo`, `estado` (`abierta` \| `cerrada`), `creada_en`, `cerrada_en` | HU10, HU16 |
| `abonos` | `id`, `deuda_id`, `valor` (> 0), `fecha`, `medio`, `comprobante`, `registrado_por` | HU14 |
| `envios_estado_cuenta` | `id`, `clienta_id`, `fecha`, `saldo_enviado`, `whatsapp_message_id`, `estado_entrega` (`sent` \| `delivered` \| `read` \| `failed`) | HU12 |
| `notificaciones` | `id`, `clienta_id`, `pedido_id`, `plantilla`, `whatsapp_message_id`, `estado_entrega`, `creado_en` | ADR 0002 |

Vistas derivadas (no se guardan a mano, así se cumple RN07):

- `lista_deudores`: clientas con suma de `saldo` de deudas abiertas > 0, con teléfono, local y saldo (HU13, HU16).
- `movimientos_estado_cuenta`: unión de deudas (cargos) y abonos por clienta, ordenada por fecha (HU11, HU15).

`medio` admite: `efectivo`, `transferencia_bancolombia`, `transferencia_nequi`. Si es transferencia, `comprobante` es obligatorio (RN04).

## Estados del pedido

`pedidos.estado` (flujo del pedido):

| Valor | Significado | Se llega cuando |
| --- | --- | --- |
| `registrado` | Pedido guardado | Se crea un pedido que no requiere cuota inicial (HU01) |
| `esperando_cuota_inicial` | Falta pagar la cuota | Se crea un pedido a crédito con total ≥ $100.000 y cuota válida (HU04) |
| `cuota_inicial_abonada` | Cuota pagada completa | Se registra el pago de la cuota; se crea la deuda (HU04, HU10) |
| `autorizado` | Compra al proveedor autorizada | Requiere compra y (no requiere cuota, o ya está abonada) (HU05) |
| `listo_para_recoger` | Listo en bodega | Pasa la verificación de despacho con modalidad bodega (HU08) |
| `despachado` | Salió con el mensajero | Pasa la verificación de despacho con modalidad domicilio (HU08) |
| `entregado` | La clienta lo recibió | Se registra la entrega o la recogida; si es crédito < $100.000, se crea la deuda (HU10) |
| `cerrado` | Terminado | Contado entregado y pagado, o su deuda llegó a $0 (HU16) |

`pedidos.estado_pago` (solo pedidos de contado): `pendiente_de_pago` → `pagado` (HU09). En pedidos a crédito es `null`.

En la interfaz se muestran con su nombre legible: "Registrado", "Esperando cuota inicial", "Cuota Inicial Abonada", "Autorizado", "Listo para recoger", "Despachado", "Entregado", "Cerrado", "Pendiente de pago", "Pagado".

## Autenticación y permisos

1. La persona inicia sesión en Next.js con Supabase Auth (correo y contraseña). No hay registro público: las cuentas de la Dueña y el Contador se crean desde el panel de Supabase.
2. `src/lib/api.ts` envía `Authorization: Bearer <access_token>` en cada llamada.
3. `backend/auth.py` verifica el token con las llaves de Supabase y carga el perfil. Dependencias: `usuario_actual`, `requiere_rol("duena")`, `requiere_rol("contador")`, `requiere_rol("duena", "contador")`.

| Módulo | Dueña | Contador |
| --- | --- | --- |
| Clientas, pedidos, cuota inicial, pago de contado, despacho | ✔ | consulta |
| Proveedores y compras | ✔ | consulta |
| Llegada de productos (HU07) | ✔ | ✔ |
| Deudas, estados de cuenta, lista de deudores | consulta | ✔ |
| Abonos (HU14) | ✔ | ✔ |
| Envío de estados de cuenta (HU12) | — | ✔ |

## Entornos y variables

| Variable | Dónde | Para qué |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | Front | URL del proyecto de Supabase |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Front | Llave pública (publishable) para el login |
| `SUPABASE_URL` | Back | Verificar JWT con las llaves públicas del proyecto |
| `DATABASE_URL` | Back | Conexión a Postgres. En Vercel, la del *pooler* en modo transacción (puerto 6543) |
| `SUPABASE_SERVICE_ROLE_KEY` | Back | Solo si se usa Storage para los PDF |
| `WHATSAPP_*` | Back | Cloud API de Meta, ver [ADR 0002](decisiones/0002-whatsapp-cloud-api.md) |

Entornos: **local** (Supabase con Docker), **preview** (un deploy de Vercel por PR) y **producción** (rama `main`). Preview y producción usan el proyecto de Supabase en la nube; la configuración de Vercel y Supabase la administra Sergio.

## Decisiones pendientes

- **PDF del estado de cuenta** (HU11): generarlo en el backend con una librería pura de Python (`fpdf2` o `reportlab`) para poder reutilizarlo en el envío por WhatsApp.
- **WhatsApp**: decidido, Cloud API de Meta con plantillas *utility* para estados de cuenta y notificaciones de pedido ([ADR 0002](decisiones/0002-whatsapp-cloud-api.md)). Código en `backend/whatsapp/`.
