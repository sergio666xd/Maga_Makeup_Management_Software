# Cómo trabajamos

Este repositorio es el canal compartido del equipo. Cada integrante trabaja con su propio Claude, así que **todo lo que el equipo necesite saber tiene que quedar aquí**: en el código, en `docs/` o en la descripción del PR. Lo que solo quede en una conversación de Claude se pierde para los demás.

Antes de tu primera HU, lee [docs/CONTEXTO.md](docs/CONTEXTO.md).

## 1. Tomar una historia de usuario

1. En Notion (PRODUCT BACKLOG), ponte como **Responsable** de la HU y cámbiala a **En desarrollo**.
2. Crea un issue en GitHub con la plantilla *Historia de usuario* (o reutiliza el existente) y asígnatelo. Copia ahí los escenarios de aceptación, para que tu Claude los tenga a mano.
3. Revisa en [docs/backlog.md](docs/backlog.md) de qué otras HU depende la tuya y coordina en el grupo antes de empezar.

Una persona por HU. Si dos HU tocan la misma tabla, quien llegue primero crea la migración y el otro la reutiliza.

## 2. Ramas

`main` siempre está desplegable y está protegida: nadie hace push directo.

| Tipo | Formato | Ejemplo |
| --- | --- | --- |
| Historia de usuario | `feat/HUxx-nombre-corto` | `feat/HU01-registrar-pedido` |
| Corrección | `fix/descripcion-corta` | `fix/total-con-descuento` |
| Infraestructura, docs, configuración | `chore/descripcion-corta` | `chore/ci-pytest` |

```bash
git checkout main && git pull
git checkout -b feat/HU01-registrar-pedido
```

Actualiza tu rama con `main` a menudo (`git pull --rebase origin main`) para evitar conflictos grandes al final del sprint.

## 3. Commits

[Conventional Commits](https://www.conventionalcommits.org/es/) en español, con la HU entre paréntesis cuando aplique:

```
feat(HU01): registrar pedido con consecutivo y estado Registrado
fix(HU03): recalcular total al quitar un producto
test(HU04): rango de cuota inicial entre 10.000 y 100.000
chore: configurar pytest en CI
docs: aclarar RN02 en reglas de negocio
```

Commits pequeños y que pasen las pruebas. No subas `.env.local`, llaves ni datos reales de clientas.

## 4. Pull requests

1. Abre el PR contra `main` en cuanto tengas algo que mostrar; márcalo como *Draft* mientras no esté listo.
2. Llena la plantilla: HU, escenarios cubiertos, migraciones y capturas.
3. Requisitos para fusionar:
   - CI en verde (lint, tipos, pruebas y build).
   - **Una aprobación** de otro integrante.
   - Cada escenario de aceptación cubierto por una prueba o verificado en el preview de Vercel.
   - Si cambiaste una regla, un estado o el modelo de datos, actualizaste `docs/`.
4. Fusiona con **Squash and merge**. El título del PR queda como commit en `main`, así que úsalo con el formato de commits.
5. Al fusionar, pasa la HU a **Implementada** en Notion y cierra el issue.

### Revisar el PR de otra persona

- Abre el preview de Vercel y prueba los escenarios de la HU.
- Revisa que las reglas usen `backend/reglas/` y no números escritos a mano en endpoints o pantallas.
- Revisa que las migraciones no borren datos ni modifiquen migraciones ya fusionadas.
- Si tienes la skill, pídele a tu Claude un `code-review` del PR (ver [docs/skills.md](docs/skills.md)).

## 5. Base de datos

- Todo cambio de esquema es una migración en `supabase/migrations/` creada con `pnpm supabase migration new <nombre>`. **Nunca** cambies tablas desde el panel de Supabase de producción.
- Las migraciones fusionadas no se editan: si algo está mal, se crea una migración nueva.
- Los datos de prueba van en `supabase/seed.sql` y son ficticios (nada de teléfonos reales).
- Toda tabla nueva tiene Row Level Security activado en la misma migración.
- El modelo acordado está en [docs/arquitectura.md](docs/arquitectura.md#modelo-de-datos-propuesta-inicial). Si necesitas otra columna o tabla, actualízalo en el mismo PR.

## 6. Pruebas

- Las reglas de negocio son funciones puras en `backend/reglas/` con pruebas pytest en `backend/tests/reglas/`. Cada escenario "Dado / Cuando / Entonces" que dependa de una regla tiene su prueba.
- Los endpoints se prueban en `backend/tests/api/` con el `TestClient` de FastAPI.
- Nombra las pruebas con la HU y el escenario: `test_hu03_e3_credito_desde_100000_requiere_cuota`.
- Las pruebas nunca llaman a Meta (WhatsApp) ni a la base de datos de producción.

```bash
uv run pytest                 # todo el backend
uv run pytest -k hu03         # solo una HU
```

## 7. Trabajar con Claude

Cada integrante usa su propio Claude, que lee [CLAUDE.md](CLAUDE.md) al abrir el repo. Para que todos los Claude trabajen igual:

- Pídele que lea la HU (issue o Notion), [docs/CONTEXTO.md](docs/CONTEXTO.md) y [docs/reglas-negocio.md](docs/reglas-negocio.md) antes de escribir código.
- Pídele un plan corto y revísalo antes de que empiece.
- Revisa tú el diff antes de hacer commit. Tú eres responsable del código, no Claude.
- Si Claude descubre algo que el equipo debe saber (una decisión, una regla aclarada), que lo agregue a `docs/` en el mismo PR.
- Las skills útiles están en [docs/skills.md](docs/skills.md). No todos las tienen; no son obligatorias.

## 8. Decisiones técnicas

Las decisiones que afectan a todo el equipo (librerías nuevas, cambios de estructura, integraciones) se registran como ADR en `docs/decisiones/NNNN-titulo.md` con la [plantilla](docs/decisiones/0000-plantilla.md) y se aprueban en un PR como cualquier otro cambio.

## 9. graphify: mapa del repositorio

[graphify](https://github.com/Graphify-Labs/graphify) construye un grafo de conocimiento del repositorio (código, docs y cómo se conectan) para que Claude responda preguntas como "¿qué toca la regla RN01?" o "¿qué depende de la tabla `deudas`?" sin leer archivo por archivo.

El repo ya trae la skill y los hooks en `.claude/` (instalados con `graphify install --project`). Cada integrante hace esto **una vez**:

```bash
uv tool install graphifyy      # el paquete se llama graphifyy, con doble y
graphify hook install          # reconstruye el grafo del código en cada commit y checkout
```

Luego, en Claude Code dentro del repo:

```
/graphify .                    # primera vez: grafo completo (código + docs)
/graphify . --update           # después de cambios en docs/ (el hook solo cubre código)
```

Y para consultar, tú o Claude:

```bash
graphify query "cuándo se genera la deuda"
graphify explain "RN01"
graphify path "HU04" "deudas"
```

Notas:

- La carpeta `graphify-out/` **no se sube al repo** (está en `.gitignore`): cada quien tiene su grafo local, así no hay conflictos entre ramas.
- El grafo del código se construye sin modelo de IA y es gratis. El de `docs/` lo hace tu Claude con `/graphify`, y gasta tokens de tu plan; hazlo cuando cambien los documentos, no en cada sesión.
- Después de `git pull`, corre `graphify update .` (el hook no cubre los pull).
- Si no instalas graphify, los hooks de `.claude/settings.json` no hacen nada y todo funciona igual.
