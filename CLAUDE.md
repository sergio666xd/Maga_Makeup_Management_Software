# CLAUDE.md

Contexto para Claude cuando trabaja en este repositorio. Cinco personas del equipo usan su propio Claude sobre el mismo código: sigue estas reglas para que el trabajo de todos sea consistente.

## El proyecto

Software de gestión de pedidos, inventario y cartera para **Maga Make Up**, una microempresa de maquillaje de Medellín que vende por llamada y WhatsApp, de contado y a crédito. Usuarios: la **Dueña** y el **Contador**. Las clientas no usan el sistema; reciben su estado de cuenta por WhatsApp.

**Lee [docs/CONTEXTO.md](docs/CONTEXTO.md) al empezar cada sesión.** Ahí están el negocio, los roles, el ciclo del pedido, cuándo nace una deuda, el glosario y los temas abiertos.

Responde y escribe la documentación en **español**.

## Antes de escribir código

1. Identifica la HU (rama `feat/HUxx-...`, issue de GitHub o lo que diga la persona). Lee sus escenarios en el issue o en [docs/backlog.md](docs/backlog.md), y sus dependencias.
2. Lee [docs/reglas-negocio.md](docs/reglas-negocio.md) y [docs/arquitectura.md](docs/arquitectura.md).
3. Propón un plan corto (endpoints, migraciones, pantallas, pruebas por escenario) y espera la aprobación de la persona.

Si un escenario es ambiguo, contradice las reglas o toca un tema abierto de CONTEXTO.md, pregunta en lugar de suponer, y anota la duda en el PR.

## Stack y comandos

- **Frontend**: Next.js (App Router) + TypeScript + Tailwind + shadcn/ui, en `src/`. Paquetes con **pnpm**.
- **Backend**: FastAPI + Pydantic + SQLAlchemy (Python 3.12), en `backend/`, con punto de entrada `api/index.py`. Paquetes con **uv**.
- **Base de datos y login**: Supabase (Postgres, Auth, RLS). Migraciones SQL en `supabase/migrations/`.
- **Despliegue**: un solo proyecto de Vercel. Todas las rutas de FastAPI empiezan por `/api`.

```bash
pnpm dev                               # Next (:3000) + FastAPI (:8000)
pnpm lint && pnpm typecheck            # frontend
uv run ruff check . && uv run ruff format --check . && uv run pytest   # backend
pnpm supabase migration new <nombre>   # nueva migración
pnpm supabase db reset                 # reaplica migraciones + seed en local
```

Antes de dar una tarea por terminada, corre el lint, el typecheck y las pruebas de la parte que tocaste.

## Dónde va cada cosa

- **Reglas de negocio** (umbral de $100.000, rango de cuota, cálculo de saldos, quién paga el envío, cierre de deuda): **solo** en `backend/reglas/`, como funciones puras con constantes nombradas y pruebas en `backend/tests/reglas/`. El frontend nunca decide una regla; a lo sumo muestra un cálculo provisional.
- **Endpoints**: `backend/routers/<modulo>.py`, delgados. Validan con esquemas Pydantic de `backend/schemas/` y llaman a `backend/servicios/`.
- **Servicios**: `backend/servicios/<modulo>.py`. Combinan reglas y base de datos dentro de **una transacción** cuando una acción cambia varias tablas (por ejemplo: pagar la cuota inicial → cambiar estado → crear deuda → actualizar lista de deudores).
- **Frontend**: páginas en `src/app/(panel)/<modulo>/`, componentes en `src/components/`, cliente de la API en `src/lib/api.ts`. El frontend no consulta tablas de negocio directamente con supabase-js; usa la API. Supabase en el frontend es **solo para el login**.

## Skills y herramientas

- [docs/skills.md](docs/skills.md) lista las skills útiles y cuándo usarlas (diseño de interfaz, FastAPI, pruebas, revisión, seguridad, PDF). **No todos los integrantes las tienen instaladas**: si una no está disponible, sigue el enfoque que describe ese documento con las reglas de este repo. Ninguna skill reemplaza CLAUDE.md ni `docs/`; si chocan, mandan las reglas del repo.
- Usa `security-review` antes de pedir review de cambios en login, roles, el webhook de WhatsApp o datos de clientas.
- Si existe `graphify-out/graph.json`, consulta el grafo (`graphify query "..."`, `graphify explain "..."`) antes de buscar archivo por archivo. Ver [CONTRIBUTING.md](CONTRIBUTING.md#9-graphify-mapa-del-repositorio).

## WhatsApp

Se usa la **Cloud API de Meta** para estados de cuenta y notificaciones de pedido ([ADR 0002](docs/decisiones/0002-whatsapp-cloud-api.md)). Todo pasa por `backend/whatsapp/`; ningún otro módulo llama a Meta. Solo se envían **plantillas aprobadas** de categoría utilidad. Las pruebas nunca llaman a Meta: usan un doble de `backend/whatsapp/cliente.py`. No envíes mensajes a teléfonos reales desde desarrollo.

## Convenciones

- **Idioma del código**: conceptos del negocio en español y sin tildes (`pedido`, `clienta`, `cuota_inicial`, `abono`, `saldo_pendiente`). Lo técnico puede ir en inglés. Textos de la interfaz en español con tildes.
- **Nombres**: Python y tablas en `snake_case` (tablas en plural: `pedidos`, `detalle_pedido`); TypeScript en `camelCase`; componentes React en `PascalCase`. La API responde JSON en `snake_case`.
- **Dinero**: pesos colombianos **enteros** (`int` en Python, `number` entero en TS, `bigint` en Postgres). Nunca `float` ni `numeric` con decimales. En la interfaz: `Intl.NumberFormat('es-CO', { style: 'currency', currency: 'COP', maximumFractionDigits: 0 })`.
- **Estados**: usa exactamente los valores de [docs/arquitectura.md](docs/arquitectura.md#estados-del-pedido). No inventes estados sin actualizar ese documento.
- **Errores**: la API responde `422` con el campo y un mensaje en español para validaciones (los mensajes que piden los escenarios, por ejemplo *"La cuota inicial debe estar entre $10.000 y $100.000"*), y `409` para conflictos (clienta o proveedor duplicado).
- **Base de datos**: solo con migraciones nuevas. Nunca edites una migración ya fusionada en `main`. Toda tabla nueva lleva RLS activado.
- **Pruebas**: cada escenario "Dado / Cuando / Entonces" de la HU tiene una prueba cuyo nombre cita la HU y el escenario: `test_hu03_e3_credito_desde_100000_requiere_cuota`.
- **Secretos**: nunca en el código ni en commits. `DATABASE_URL` y la `service_role` key solo existen en el backend.

## Límites

- Trabaja solo en lo que pide la HU actual. Si ves algo que arreglar fuera de ella, menciónalo en lugar de cambiarlo.
- No modifiques el código de la HU de otra persona sin que lo pidan. Si necesitas cambiar código compartido (`backend/reglas/`, `backend/db.py`, `backend/auth.py`, `src/components/ui/`, migraciones), dilo en la descripción del PR.
- No hagas push a `main`, no fusiones PR y no cambies la configuración de Supabase o Vercel de producción.
- No agregues dependencias nuevas sin mencionarlo; si afectan a todo el equipo, propón un ADR en `docs/decisiones/`.

## Al terminar

- Commits con [Conventional Commits](https://www.conventionalcommits.org/es/): `feat(HU01): ...`.
- Si aclaraste una regla, un estado o una decisión, actualiza `docs/` en el mismo PR para que el Claude de los demás lo sepa.
- Llena la plantilla del PR indicando qué escenarios quedaron cubiertos y cómo se probó cada uno.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- When graphify-out/graph.json exists, run `graphify query "<question>"` before your first grep, search or source-file read for a codebase question, and put this rule in every subagent prompt that explores code. Read raw files only for lines the graph or an earlier read already located. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. When the answer is under a Markdown heading, `graphify section "<heading>"` prints that section's text, so the .md need not be opened. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
- A committed graphify-out/graph.json is rewritten whole on every build, so a large diff to it in a pull request is expected. Do not flag it as churn or suggest skipping the graph rebuild or its hooks.
