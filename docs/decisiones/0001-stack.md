# 0001 · Stack: Next.js + FastAPI + Supabase en un solo proyecto de Vercel

- **Estado**: aceptada
- **Fecha**: 2026-11-26
- **Decide**: Sergio Muñoz Marín, con el equipo

## Contexto

Cinco integrantes, cada uno con su propio Claude, tienen que entregar 16 HU en tres sprints cortos (27 nov – 15 dic 2026). El sistema es un CRUD con reglas de negocio de dinero (cuota inicial, deudas, abonos), generación de PDF y envío por WhatsApp. El equipo ya tiene cuentas de Supabase y Vercel.

## Decisión

- **Frontend**: Next.js (App Router) con TypeScript, Tailwind y shadcn/ui.
- **Backend**: FastAPI (Python 3.12) con Pydantic y SQLAlchemy. Concentra las reglas de negocio y el acceso a datos.
- **Datos y login**: Supabase (Postgres, Auth y RLS). Esquema versionado con migraciones SQL de la Supabase CLI.
- **Despliegue**: un solo proyecto de Vercel. Next.js sirve la interfaz y FastAPI corre como función Python en `api/index.py`; `/api/*` se enruta a FastAPI.

## Consecuencias

- Las reglas viven en un solo lugar (Python) y se prueban con pytest sin levantar la interfaz.
- Un solo deploy y un solo dominio: sin CORS en producción y con un preview por PR que incluye front y back.
- Hay dos ecosistemas (pnpm y uv). `pnpm dev` levanta ambos para que nadie tenga que recordarlo.
- Las funciones Python de Vercel no mantienen conexiones abiertas: se usa el *pooler* de Supabase en modo transacción y sesiones cortas de SQLAlchemy.
- Se descarta Alembic: las migraciones son SQL de Supabase para poder declarar RLS y vistas en el mismo archivo.
- Riesgo: el enrutamiento Next.js + Python en el mismo proyecto debe validarse en la tarea SETUP con un deploy de preview. Si falla, la alternativa es desplegar FastAPI como un segundo proyecto de Vercel sobre el mismo repositorio.

## Alternativas consideradas

- **Solo Next.js** (Server Actions + Supabase): menos piezas, pero el equipo prefiere Python para el backend.
- **FastAPI en otro proveedor** (Render, Railway): agrega otra cuenta, otro deploy y CORS.
