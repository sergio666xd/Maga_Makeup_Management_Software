# Reglas de negocio

Fuente: documento *Entrega Proyecto Empresa* (Drive), actualizado con las decisiones del *Product_Backlog_Mejorado* (26 nov 2026). Cuando hay diferencia, manda el backlog mejorado.

Todas las reglas se implementan **una sola vez, en el backend**, como funciones puras de Python en `backend/reglas/` usando las constantes de abajo. Ni los endpoints, ni el frontend, ni las consultas SQL repiten estos valores a mano. El frontend puede mostrar cálculos provisionales (por ejemplo, el total mientras se arma el pedido), pero el valor que se guarda siempre lo calcula el backend.

## Constantes

```python
# backend/reglas/constantes.py
UMBRAL_CUOTA_INICIAL = 100_000   # RN01, inclusive (>=)
CUOTA_INICIAL_MIN = 10_000       # RN01, inclusive
CUOTA_INICIAL_MAX = 100_000      # RN01, inclusive
MIN_PRODUCTOS_ENVIO_EMPRESA = 2  # RN02
MEDIOS_PAGO = ("efectivo", "transferencia_bancolombia", "transferencia_nequi")  # RN04
```

Todos los valores de dinero son **pesos colombianos enteros**.

## RN01 · Autorización por monto

- Un pedido **a crédito** con total **≥ $100.000** requiere cuota inicial. Un pedido de contado nunca la requiere, sin importar su valor.
- La Dueña fija la cuota entre **$10.000 y $100.000** (ambos incluidos). Fuera de ese rango el pedido no se guarda y se muestra: *"La cuota inicial debe estar entre $10.000 y $100.000"*.
- No se valida que la cuota sea proporcional al total.
- La cuota se paga **en un solo pago** por su valor completo. Un pago menor se rechaza.
- Al pagarse, el pedido pasa a *Cuota Inicial Abonada*, se genera la deuda por *total − cuota* y la compra se autoriza.
- Pedidos que no requieren cuota se autorizan de inmediato si necesitan compra al proveedor.

```python
def requiere_cuota_inicial(modalidad_pago: str, total: int) -> bool:
    return modalidad_pago == "credito" and total >= UMBRAL_CUOTA_INICIAL

def cuota_inicial_valida(cuota: int) -> bool:
    return CUOTA_INICIAL_MIN <= cuota <= CUOTA_INICIAL_MAX
```

HU relacionadas: HU03, HU04, HU05, HU08, HU10.

## RN02 · Costo de envío en devoluciones

Cuando el producto regresa a la bodega:

| Situación | Quién paga el envío |
| --- | --- |
| Nadie recibió el pedido | Cliente |
| El cliente devuelve y compró **menos de 2** productos | Cliente |
| El cliente devuelve y compró **2 o más** productos | Empresa |

> Ninguna de las 16 HU actuales implementa devoluciones. Ver tema abierto en [CONTEXTO.md](CONTEXTO.md#abiertos-confirmar-con-el-product-owner-antes-de-implementar-lo-que-afecten).

## RN03 · Costo de envío regular

En entregas a domicilio, el cliente paga el envío y su costo se suma al pedido al verificar el despacho (HU08). Recoger en bodega no tiene costo de envío.

## RN04 · Medios de pago

- Modalidades: **contado** o **crédito**.
- Medios: **efectivo** o **transferencia** (Bancolombia o Nequi).
- Toda transferencia exige el **número de comprobante**.
- No hay integración bancaria: la Dueña o el Contador confirman cada pago a mano.

HU relacionadas: HU04, HU09, HU14.

## RN05 · Flexibilidad de abonos

- El cliente abona cualquier monto, sin cifra fija.
- Válido si `0 < abono <= saldo`. Un abono de $0, negativo o mayor al saldo se rechaza mostrando el motivo.
- `saldo_nuevo = saldo_anterior − abono`.

HU relacionadas: HU14, HU15, HU16.

## RN06 · Tiempos de cobro

- La difusión de cobro (estado de cuenta por WhatsApp) se envía **el día de la quincena** a los clientes con saldo pendiente.
- El cobro presencial es **al día siguiente**.
- Los clientes con saldo $0 no reciben envío.

HU relacionadas: HU12, HU13.

## RN07 · Estado de cuenta

- Existe desde la primera deuda del cliente y se actualiza **automáticamente** con cada deuda nueva o abono. Nadie lo edita a mano.
- Muestra nombre, teléfono, fecha de consulta y cada compra con su valor, abonos y saldo, ordenados por fecha.
- Se exporta a PDF con la misma información de la pantalla.

HU relacionadas: HU11, HU12, HU15.

## RN08 · Cierre de deuda

Cuando el saldo de una deuda llega a $0, el sistema automáticamente:

1. Cierra la deuda.
2. Marca su pedido como **Cerrado**.
3. Retira al cliente de la lista de deudores **si no le quedan otras deudas abiertas**. Si le quedan, sigue en la lista con el saldo de las demás.

Si después se le genera una deuda nueva, vuelve a la lista.

HU relacionadas: HU13, HU16.

## Otras reglas de los criterios de aceptación

- **Teléfono del cliente** obligatorio y único (HU01).
- **Proveedor** único por teléfono; nombre y teléfono obligatorios (HU06).
- **Total del pedido** = Σ cantidad × valor unitario; se recalcula con cada cambio (HU03).
- **Inventario**: existencia ≥ cantidad pedida → *Disponible*; si no, se muestran las unidades que faltan y el pedido requiere compra (HU02).
- **Llegada de productos**: solo se suman al inventario las unidades recibidas que pasan la revisión de calidad; los faltantes y rechazos quedan registrados en la compra; cantidades y valores deben ser > 0 (HU07).
- **Pedido de contado** queda *Pendiente de pago* hasta que la Dueña registra el pago (HU09).
