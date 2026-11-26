# Maga Make Up · Gestión de pedidos y estados de cuenta

Software para que Maga Make Up registre los pedidos que llegan por llamada o WhatsApp, controle el inventario y la compra al proveedor, y lleve la cartera de sus clientes: deudas, abonos, estados de cuenta y el cobro quincenal por WhatsApp.

Lo desarrolla el **Equipo 2.6** (Ingeniería de Software, UNAL Medellín) en tres sprints, del 27 de noviembre al 15 de diciembre de 2026.

> **¿Eres nuevo en el proyecto, o eres Claude?** Empieza por [docs/CONTEXTO.md](docs/CONTEXTO.md): el negocio, el problema, la solución, los roles, el ciclo del pedido, las reglas y el glosario.

## Stack

| Capa | Tecnología |
| --- | --- |
| Frontend | [Next.js](https://nextjs.org/) (App Router) con TypeScript, Tailwind CSS y [shadcn/ui](https://ui.shadcn.com/) |
| Backend (API) | [FastAPI](https://fastapi.tiangolo.com/) con Python 3.12, Pydantic y SQLAlchemy |
| Base de datos y login | [Supabase](https://supabase.com/): Postgres, Auth y Row Level Security |
| Despliegue | **Un solo proyecto de [Vercel](https://vercel.com/)**: Next.js para la interfaz y FastAPI como función Python bajo `/api`. Producción desde `main` y un preview por cada PR |
| WhatsApp | [Cloud API de Meta](https://developers.facebook.com/docs/whatsapp/cloud-api) para estados de cuenta y notificaciones ([ADR 0002](docs/decisiones/0002-whatsapp-cloud-api.md)) |
| Pruebas | pytest (reglas y endpoints) y Vitest (utilidades del frontend) |

El porqué de cada elección está en [docs/decisiones/0001-stack.md](docs/decisiones/0001-stack.md).

## Documentación

| Documento | Para qué sirve |
| --- | --- |
| [docs/CONTEXTO.md](docs/CONTEXTO.md) | Todo el contexto del negocio y del proyecto (fuente: carpeta de Drive y Notion) |
| [docs/reglas-negocio.md](docs/reglas-negocio.md) | Reglas RN01 a RN08 con valores exactos y fórmulas |
| [docs/arquitectura.md](docs/arquitectura.md) | Estructura de carpetas, modelo de datos, estados del pedido y API |
| [docs/backlog.md](docs/backlog.md) | Las 16 HU por sprint, con dependencias y enlaces a Notion |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Cómo tomar una HU, crear la rama, hacer commits y abrir el PR |
| [CLAUDE.md](CLAUDE.md) | Instrucciones que lee automáticamente el Claude de cada integrante |
| [docs/decisiones/](docs/decisiones/) | Registro de decisiones técnicas (ADR) |
| [docs/skills.md](docs/skills.md) | Skills de Claude útiles para el proyecto y cuándo usarlas |

La fuente de verdad del avance es el tablero **PRODUCT BACKLOG** en Notion (espacio Maga Make Up).

## Puesta en marcha local

> El esqueleto de Next.js y FastAPI se crea en la tarea **SETUP** del sprint 1 (ver [docs/backlog.md](docs/backlog.md)). Hasta entonces el repositorio solo tiene documentación y estos comandos son el objetivo de esa tarea.

Requisitos: Node.js 20 LTS o superior, [pnpm](https://pnpm.io/), Python 3.12, [uv](https://docs.astral.sh/uv/), Git y Docker (para correr Supabase en local).

```bash
git clone https://github.com/sergio666xd/Maga_Makeup_Management_Software.git
cd Maga_Makeup_Management_Software
pnpm install                 # dependencias del frontend
uv sync                      # dependencias del backend (crea .venv)
cp .env.example .env.local   # pide los valores al equipo, nunca los subas al repo
pnpm supabase start          # Postgres + Auth locales con Docker
pnpm supabase db reset       # aplica migraciones y datos de prueba
pnpm dev                     # Next.js en :3000 y FastAPI en :8000 al mismo tiempo
```

En local, Next.js reenvía `/api/*` a FastAPI (`http://127.0.0.1:8000`). En Vercel, `/api/*` llega directo a la función Python. La documentación interactiva de la API queda en `http://localhost:8000/api/docs`.

Comandos habituales:

```bash
pnpm lint && pnpm typecheck       # frontend
uv run ruff check . && uv run pytest   # backend
pnpm build                        # build de producción del frontend
```

## Roles del sistema

- **Dueña**: registra clientes, pedidos y proveedores; gestiona cuota inicial, pagos de contado y despacho.
- **Contador**: registra la llegada de productos, consulta deudas y estados de cuenta, registra abonos y envía los estados de cuenta.
