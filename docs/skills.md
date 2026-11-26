# Skills de Claude para este proyecto

Las *skills* son instrucciones empaquetadas que Claude carga cuando la tarea lo amerita. Esta es la lista de las que Sergio tiene instaladas y sirven para el desarrollo de Maga Make Up.

> **Importante**: cada integrante tiene su propia cuenta de Claude, así que **es posible que no tengas estas skills**. Si no aparecen en tu Claude, no pasa nada: las reglas de este repositorio (CLAUDE.md y `docs/`) son suficientes. Puedes pedirle a Claude que siga el mismo enfoque descrito aquí, o instalar la skill si está disponible en tu plan. Las skills con prefijo `engineering:` vienen del plugin *engineering*; las `anthropic-skills:` del catálogo de Anthropic; las que no tienen prefijo vienen con Claude Code.

## Cuándo usar cada una

### Planear y diseñar

| Skill | Úsala para |
| --- | --- |
| `engineering:architecture` | Escribir un ADR nuevo en `docs/decisiones/` (por ejemplo, cómo generar el PDF). |
| `engineering:system-design` | Diseñar un módulo antes de una HU grande (cartera, despacho) o revisar el modelo de datos. |
| `anthropic-skills:fastapi-clean-architecture` | Estructurar routers, servicios, esquemas y pruebas del backend. Adáptala a la estructura de [arquitectura.md](arquitectura.md); no agregues capas que no estén allí. |
| `engineering:testing-strategy` | Convertir los escenarios Dado / Cuando / Entonces de una HU en un plan de pruebas pytest. |

### Construir la interfaz

| Skill | Úsala para |
| --- | --- |
| `anthropic-skills:impeccable` | Diseñar o mejorar pantallas: formularios de pedido, listas, estados vacíos, errores. |
| `anthropic-skills:anti-ui-slop` | Evitar interfaces genéricas; definir los estados obligatorios de cada pantalla. |
| `anthropic-skills:make-interfaces-feel-better` | Pulir detalles: números tabulares en montos, foco, hover, sombras. |
| `anthropic-skills:web-design-guidelines` | Revisar accesibilidad y buenas prácticas antes de pedir review. |
| `anthropic-skills:transitions-dev` / `transitions-polish` | Solo si hace falta animar algo (modales, toasts). Con moderación. |
| `dataviz` | Gráficas o tarjetas de resumen (cartera total, deudas por quincena). |

### Revisar y depurar

| Skill | Úsala para |
| --- | --- |
| `code-review` | Revisar tu diff o el PR de un compañero antes de aprobarlo. |
| `security-review` | Antes de fusionar algo que toque login, roles, webhook de WhatsApp o datos de clientas. |
| `simplify` | Limpiar código que funciona pero quedó enredado. |
| `engineering:debug` | Errores que no se entienden ("funciona local y no en el preview"). |
| `engineering:code-review` | Alternativa del plugin engineering para revisar seguridad, rendimiento y casos borde. |
| `engineering:tech-debt` | Al final de cada sprint, listar deuda técnica a atender. |

### Documentar, entregar y operar

| Skill | Úsala para |
| --- | --- |
| `engineering:documentation` | Actualizar `docs/`, escribir guías para la dueña o el contador. |
| `engineering:deploy-checklist` | Antes de llevar un sprint a producción, sobre todo si hay migraciones. |
| `engineering:standup` | Resumen diario de lo hecho (commits y PR) para el equipo. |
| `run` | Levantar la app y comprobar en el navegador que la HU funciona. |
| `anthropic-skills:pdf` | Revisar el PDF del estado de cuenta generado (HU11). |
| `anthropic-skills:xlsx` | Leer los Excel actuales de la dueña o el backlog para importar datos de prueba. |

### WhatsApp (Cloud API de Meta)

No hay una skill específica para Maga Make Up. Sergio tiene `anthropic-skills:whatsapp-despacho-am`, pero **es de otro negocio** (un despacho de abogados): no la uses tal cual. Su método sí sirve como referencia para redactar las plantillas de Meta: mensajes cortos, claros, con el dato concreto y aprobables como *utilidad*. Las plantillas y reglas de Maga están en [decisiones/0002-whatsapp-cloud-api.md](decisiones/0002-whatsapp-cloud-api.md).

## Herramientas del repositorio que no son skills

- **graphify**: grafo de conocimiento del repositorio para que Claude encuentre rápido cómo se conectan reglas, HU, tablas y código. Ver [CONTRIBUTING.md](../CONTRIBUTING.md#9-graphify-mapa-del-repositorio).
