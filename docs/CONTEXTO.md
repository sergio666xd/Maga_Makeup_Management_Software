# Contexto del proyecto Maga Make Up

Este documento reúne todo lo que el equipo sabe del negocio y del proyecto, a partir de los archivos de la carpeta de Drive **Planeacion_Maga_Make_Up** y del tablero de Notion. Es la lectura obligatoria antes de tomar cualquier HU, para personas y para Claude.

Si algo de aquí contradice el backlog vigente, manda el backlog (ver [Fuentes y vigencia](#11-fuentes-y-vigencia)). Si descubres algo nuevo o una regla cambia, actualiza este archivo en tu PR.

---

## 1. El negocio

**Maga Make Up** es una microempresa de Medellín que compra y distribuye productos de maquillaje y cuidado personal para un público mayoritariamente femenino. Tiene una **bodega pequeña en el centro de Medellín**, que sirve de almacén y punto de despacho de pedidos bajo demanda. Lo que no se vende se promociona en redes sociales.

La dueña construyó su clientela visitando personalmente locales de los centros comerciales más populares de Medellín, ofreciendo productos con posibilidad de pago a crédito. Con esa base recurrente empezó a vender de forma virtual. Hoy:

- Los clientes piden por **llamada o WhatsApp Business** (también es el canal de atención).
- Se vende **de contado o a crédito**.
- Si el cliente quiere envío, se contrata un **mensajero** de una red cercana a la bodega y el envío lo paga el cliente. También puede **recoger en la bodega**.
- Hay **garantía de devolución**: si el producto no le gusta, el mismo mensajero lo regresa a la bodega.
- Cada cliente recibe un **estado de cuenta** justo después de comprar, con el nombre y valor de cada producto y la deuda total. Hoy se hace a mano en **Excel** (libros organizados por centro comercial y local) y se envía por WhatsApp Business.

### Cobro quincenal

- El **día de la quincena** se envía un mensaje de cobro a todos los clientes en mora, para que quien no quiera tener efectivo al día siguiente pague por transferencia.
- El **día siguiente a la quincena**, la dueña recorre los locales donde tiene clientes con deuda (con una lista hecha a mano en papel) y cobra en persona.
- El cliente paga en **efectivo** o por **transferencia (Bancolombia o Nequi)**. **No hay cuota fija**: abona lo que quiera.
- La dueña anota los pagos en papel y luego se los pasa al **contador**, que los transcribe en Excel. Ahí están los errores y la doble digitación.

### Misión, visión y objetivos (resumen)

- **Misión**: facilitar a las mujeres de Medellín y alrededores el acceso a productos de maquillaje y cuidado personal, con atención confiable y oportuna.
- **Visión 2028**: empresa reconocida en Medellín y el Valle de Aburrá, con varios puntos de venta, línea de productos propia y **procesos de inventario y estados de cuenta automatizados**.
- **Objetivos**: corto plazo, aumentar ventas y confianza; mediano, ampliar proveedores y márgenes; largo, marca propia y nuevos locales.
- Estrategias DOFA relevantes para el software: reemplazar papel y Excel por una app de cartera e inventario; recordatorios automáticos de cobro; definir política de crédito; proteger los datos personales de los clientes y verificar transferencias.

## 2. El problema

Los procesos de control de existencias, cálculo de deudas y cobranza son **manuales** (papel, Excel individual y WhatsApp uno a uno). Consecuencias:

- **Inventario**: no hay trazabilidad en tiempo real; las compras y devoluciones no actualizan existencias; no hay alertas de faltantes ni visibilidad de mercancía estancada.
- **Cartera**: registro fragmentado (dueña en papel → contador en Excel), errores de digitación, estados de cuenta hechos uno por uno y envío manual por WhatsApp.
- **Causas raíz**: no existe un software integrado; se depende de herramientas informales no compartidas; las reglas de negocio no están automatizadas.
- **Impacto**: riesgo de cartera vencida y descuadres de dinero, sobrecarga operativa para la dueña y el contador, y un negocio que no puede crecer hacia la visión 2028.

## 3. La solución (modelo verbal de la solución)

Un sistema de información que **controla cada pedido desde que el cliente lo solicita hasta que su deuda queda saldada**. **No incluye nómina ni contabilidad general.** Tampoco hay integración bancaria: los pagos se confirman a mano.

1. **Pedido y compra al proveedor.** La Dueña registra al cliente (teléfono obligatorio, es su identificador) y los productos que pide. El sistema verifica el inventario y calcula el total. Si el pedido es a crédito desde $100.000, queda esperando cuota inicial y solo pasa a compra cuando la cuota se paga. La compra al proveedor se registra con la revisión de calidad, cambios de precio y faltantes.
2. **Bodega, despacho y entrega.** Al llegar los productos, el inventario se actualiza. Para despachar, el sistema verifica condiciones de entrega acordadas, unidades disponibles y pagos requeridos, y se asigna un mensajero (puede llevar varios pedidos de destinos cercanos) o se deja listo para recoger en bodega. Se registra el resultado de la entrega; si nadie recibe o el cliente devuelve productos, vuelven a la bodega y el sistema decide quién paga el envío.
3. **Ventas, cartera y cobro.** El sistema genera la deuda automáticamente en el momento que corresponde, agrega al cliente a la lista de deudores, mantiene el estado de cuenta actualizado y lo envía por WhatsApp cada quincena. Se registran los abonos (efectivo o transferencia) y, cuando el saldo llega a $0, el sistema cierra la deuda, cierra el pedido y retira al cliente de la lista de deudores.

Datos que maneja: clientes, proveedores, productos, mensajeros, pedidos, compras, entregas, devoluciones, deudas, abonos y estados de cuenta.

## 4. Actores y roles

| Actor | ¿Usa el sistema? | Qué hace |
| --- | --- | --- |
| **Dueña** | Sí (rol `duena`) | Registra clientes, pedidos, proveedores, cuota inicial, pagos de contado, despacho y entregas. También puede registrar abonos y consultar la cartera. |
| **Contador** | Sí (rol `contador`) | Registra la llegada de productos a bodega, consulta deudas y estados de cuenta, registra abonos y envía los estados de cuenta. Asume las tareas de los antiguos "gestores". |
| Cliente | No | Pide, paga, abona, recibe su estado de cuenta por WhatsApp. Se identifica por su teléfono. |
| Proveedor | No (es un dato) | Vende los productos a la empresa. |
| Mensajero | No (es un dato) | Entrega pedidos y regresa devoluciones. |
| Sistema | — | Verifica inventario, calcula totales, autoriza compras, genera y cierra deudas, actualiza la lista de deudores y el estado de cuenta. |

**Decisión del equipo**: los roles "Gestor de Inventario", "Gestor de Finanzas", "Gestor de Cobranzas" y "Gestor de Estados de Cuenta" de la Entrega 1 **ya no se usan**. Sus tareas quedaron en el **Contador**. En código y documentos se usan solo **Dueña** y **Contador**.

## 5. Ciclo de vida de un pedido

```
Registrado ──(crédito ≥ $100.000)──▶ Esperando cuota inicial ──(pago completo)──▶ Cuota Inicial Abonada
    │                                                                              │
    └──(contado o < $100.000, y faltan productos)──▶ Autorizado ◀──(faltan productos)┘
                                                          │
                                      compra al proveedor + llegada a bodega
                                                          ▼
                        Verificar despacho (existencias + pagos + condiciones de entrega)
                              │                                     │
                       Domicilio: Despachado                 Bodega: Listo para recoger
                              └──────────────▶ Entregado ◀──────────┘
                                                   │
                                (deuda con saldo $0 o pedido de contado pagado)
                                                   ▼
                                                Cerrado
```

El detalle de estados y transiciones está en [arquitectura.md](arquitectura.md#estados-del-pedido).

### Cuándo nace la deuda

| Tipo de pedido | ¿Cuota inicial? | ¿Deuda? | Momento |
| --- | --- | --- | --- |
| Contado, cualquier valor | No | **Nunca** | El pago se confirma a mano (HU09) |
| Crédito, total < $100.000 | No | Sí, por el total | Cuando el pedido pasa a **Entregado** |
| Crédito, total ≥ $100.000 | Sí, $10.000–$100.000 | Sí, por total − cuota | Cuando se registra el **pago completo de la cuota inicial**, aunque el pedido no se haya entregado |

## 6. Reglas de negocio

El detalle con fórmulas y constantes está en [reglas-negocio.md](reglas-negocio.md). Resumen:

| Código | Regla |
| --- | --- |
| RN01 | Pedidos a crédito **desde $100.000 (inclusive)** exigen cuota inicial; el resto se autoriza de inmediato. |
| RN02 | Envío en devoluciones: lo paga el cliente si nadie recibió o si compró menos de 2 productos; la empresa si compró 2 o más. |
| RN03 | En entregas normales a domicilio, el cliente paga el envío. |
| RN04 | Medios de pago: efectivo o transferencia (Bancolombia o Nequi). Transferencia exige número de comprobante. |
| RN05 | Abonos libres, sin cifra fija: saldo = saldo anterior − abono. |
| RN06 | Cobro: difusión el día de la quincena, cobro presencial al día siguiente. |
| RN07 | El estado de cuenta se genera al comprar y se actualiza con cada deuda o abono. |
| RN08 | Saldo $0 → se cierra la deuda, se cierra el pedido y el cliente sale de la lista de deudores. |

## 7. Backlog y sprints

16 historias de usuario en 3 sprints. La lista completa con dependencias está en [backlog.md](backlog.md).

| Sprint | Fechas | Historias | Tema |
| --- | --- | --- | --- |
| 1 | 27 nov – 2 dic 2026 | HU01, HU02, HU03, HU10, HU11 | Pedido, inventario, total, deuda y estado de cuenta |
| 2 | 3 – 9 dic 2026 | HU04, HU05, HU06, HU07, HU08, HU09 | Cuota inicial, compra, proveedores, bodega, despacho y pago de contado |
| 3 | 10 – 15 dic 2026 | HU12, HU13, HU14, HU15, HU16 | WhatsApp, deudores, abonos, actualización y cierre |

Seguimiento del estado de cada HU: tablero **PRODUCT BACKLOG** en Notion (Definida → En desarrollo → Implementada).

## 8. Glosario

| Término | Significado en el sistema |
| --- | --- |
| Cliente | Persona que compra. Se identifica por su **teléfono** (único y obligatorio). Suele estar ubicado en un local de un centro comercial. |
| Pedido | Solicitud de productos de un cliente, con número consecutivo, modalidad de pago y de entrega. |
| Modalidad de pago | **Contado** o **crédito**. |
| Modalidad de entrega | **Domicilio** (con mensajero y costo de envío) o **recoger en bodega** (sin envío). |
| Cuota inicial (anticipo) | Pago único que exige un pedido a crédito desde $100.000, entre $10.000 y $100.000, fijado por la Dueña. |
| Autorizar compra | Paso automático que habilita comprarle al proveedor los productos que faltan para un pedido. |
| Compra | Lo que la empresa le compra a un proveedor para surtir pedidos. Registra cantidades, faltantes, calidad y precios. |
| Existencia | Unidades disponibles de un producto en la bodega. |
| Despacho | Salida del pedido de la bodega con un mensajero, o su paso a "Listo para recoger". |
| Deuda | Lo que un cliente debe por **un pedido** a crédito. Tiene valor total, saldo pendiente y estado (abierta/cerrada). |
| Abono | Pago parcial o total de una deuda. Cualquier valor mayor a $0 y no mayor al saldo. |
| Saldo pendiente | Lo que falta por pagar de una deuda. El saldo del cliente es la suma de sus deudas abiertas. |
| Estado de cuenta | Resumen por cliente de compras, abonos y saldo, ordenado por fecha. Exportable a PDF. |
| Lista de deudores | Clientes con saldo > $0: nombre, teléfono, local y saldo. Se usa para el cobro quincenal. |
| Quincena / día de cobro | Días 15 y último de cada mes (difusión) y el día siguiente (cobro presencial). |
| Devolución | Producto que regresa a la bodega porque nadie recibió o el cliente no quedó satisfecho. |
| Mensajero (repartidor, domiciliario) | Persona externa contratada para entregas. |

## 9. Decisiones confirmadas y temas abiertos

### Confirmadas (no se discuten de nuevo)

- Teléfono del cliente obligatorio y es su identificador.
- Umbral de cuota inicial: crédito **desde $100.000 inclusive**. Pedidos de contado nunca piden cuota ni generan deuda, sin importar el valor.
- Cuota inicial entre **$10.000 y $100.000** (inclusive), fijada por la Dueña, sin validar proporción con el total. Se paga **en un solo pago** por su valor completo.
- La autorización de compra al proveedor es **automática**.
- Lo que no pasa la revisión de calidad **no entra al inventario**.
- Antes de despachar se verifican fecha/hora acordada, existencias, pago de contado registrado y cuota inicial abonada.
- El pago de contado **no** se marca como pagado automáticamente; la Dueña lo confirma.
- La deuda se genera **automáticamente** (momentos de la sección 5).
- Estado de cuenta exportable a **PDF** y enviado por **WhatsApp cada quincena** por el Contador, solo a clientes con saldo pendiente.
- WhatsApp se integra con la **Cloud API de Meta** (envío real desde el sistema, con plantillas aprobadas), también para notificaciones de pedido ([ADR 0002](decisiones/0002-whatsapp-cloud-api.md)).
- Actualización del estado de cuenta, cierre de deuda, cierre del pedido y retiro de la lista de deudores son **automáticos**.

### Abiertos (confirmar con el Product Owner antes de implementar lo que afecten)

1. **Envío y deuda**: ¿el costo de envío (RN03) se suma a la deuda del pedido a crédito o se cobra aparte? ¿Cuenta para el umbral de $100.000? Propuesta: no cuenta para el umbral y sí se suma a lo que el cliente debe.
2. **Devoluciones (RN02)**: el modelo verbal las incluye, pero **ninguna de las 16 HU** las cubre (entrega fallida, producto rechazado, costo de envío, reingreso a inventario). Hay que decidir si entran como HU nueva o quedan fuera de este semestre.
3. **Abono con varias deudas abiertas**: ¿el cliente abona a una deuda concreta o a su saldo total? Propuesta: el abono se registra contra una deuda y la interfaz sugiere la más antigua.
4. **Notificaciones de pedido por WhatsApp**: se decidió usar la Cloud API de Meta para estados de cuenta **y** notificaciones de cambios de pedido ([ADR 0002](decisiones/0002-whatsapp-cloud-api.md)), pero las notificaciones no tienen HU. Falta decidir si entran como HU nueva en el sprint 3.
5. **Quién registra abonos**: el backlog dice Contador; el modelo verbal dice Dueña. Se decidió que **ambos roles** pueden registrarlos.
6. **"Local" del cliente (HU13)**: se guarda centro comercial y local como datos opcionales del cliente.

## 10. El equipo

**Equipo 2.6**, curso de Ingeniería de Software (Universidad Nacional de Colombia, sede Medellín).

- Magdiel Santiago Gil Castillo
- Alejandro Avendaño Serna (**Product Owner**, contacto directo con la dueña)
- Sergio Muñoz Marín
- Santiago Montoya Naranjo
- Boris Oliver Quintero

Lecciones de la retrospectiva de la Entrega 1 que aplican al desarrollo:

- Trabajar **en el repositorio** con ramas, commits por artefacto y pull requests (antes solo había un commit inicial).
- Usar **un solo nombre** para cada actor y concepto (este glosario).
- **Corte interno** dos días antes de cada entrega y **revisión cruzada** antes de entregar.
- Tablero de tareas con responsable y fecha (Notion).
- No mantener versiones paralelas del mismo archivo ("v2", "final").
- No tener el repositorio dentro de una carpeta sincronizada de Google Drive (genera `desktop.ini` y conflictos con Git).

## 11. Fuentes y vigencia

Orden de prioridad cuando dos fuentes no coinciden (la más reciente manda):

1. Tablero **PRODUCT BACKLOG** de Notion y **Product_Backlog_Mejorado-Equipo_2.6** (Drive, 26 nov 2026). Incluye la hoja "Cambios y pendientes" frente a la V1.
2. **Modelo verbal de la solución** y **reglas de negocio** del documento *Entrega Proyecto Empresa* (Drive).
3. *conversacion.txt* (carpeta *prompt clarita*): decisiones confirmadas durante la revisión del backlog.
4. *Informe_revision_backlogs_Maga_Make_Up.docx*: observaciones de la revisión de la V1.
5. *Retrospectiva_Entrega_1_Maga_Make_Up.docx*.
6. Versiones anteriores, solo como historia: *Product Backlog V1*, *Product Backlog Proyecto.xlsx*, *Entrega_1_Maga_Make_Up.docx*.

Diagramas en Drive (no se copian al repo): proceso actual (*DIAGRAMA_PROCESOS_Maga_Make_Up.drawio*, *Maga Make Up (final).bpmn*, *Maga Make Up.png*), proceso de la solución (*Maga Make Up DP_SOLUCION.pdf* / *.bpm*), esquema preconceptual y modelo del dominio (entidades: Cliente, Pedido, Producto, Proveedor, Mensajero, Venta, Deuda, Abono, Estado de cuenta, Lista de deudores).

Material del curso que **no** describe el negocio: *instrucciones.pdf*, *Ejemplos.pdf*, *referencias.pdf*, el artículo *Quality User Story*, el caso de uso de ejemplo *RAPIZZA* (*Caso de Uso Registrar Pedidos.docx*) y la carpeta *Práctica*.

> **Corrección importante**: la regla RN01 original decía "más de $100.000". El backlog mejorado la corrigió a **"desde $100.000, inclusive"**. Un pedido a crédito de exactamente $100.000 **sí** requiere cuota inicial.
