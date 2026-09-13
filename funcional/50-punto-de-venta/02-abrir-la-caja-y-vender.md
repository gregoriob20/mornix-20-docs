# Abrir la caja y vender

El día a día: abrir la caja, armar la venta y ver el importe en las dos monedas.

## 1. Abrir la caja

**Punto de venta → Tablero.** Cada caja es una tarjeta.

![El tablero del TPV, con la caja y su botón](../img/tpv-tablero.png)

El botón de la tarjeta dice lo que va a pasar:

- **Abrir caja registradora** — no hay sesión abierta: se abre una nueva.
- **Seguir vendiendo** — ya hay una sesión abierta y se vuelve a ella.

La primera vez que se abre, Odoo pregunta cómo se exhiben los precios. En el
comercio venezolano se exhiben **con el IVA incluido**. Se responde una vez y no
vuelve a preguntar.

> **El TPV tarda en abrir.** No es una pantalla más del sistema: es una
> aplicación aparte que se descarga entera la primera vez, con sus productos y
> su sesión. Veinte segundos con la pantalla en blanco son normales; el segundo
> arranque es rápido.

![El TPV abierto, con el control de apertura y los productos](../img/tpv-productos.png)

## 2. Armar la venta

Se toca el producto y entra al carrito. La cantidad se cambia con el teclado
numérico de la derecha, y **Cant.**, **%** y **Precio** dicen sobre qué actúa lo
que se teclea.

![Dos productos en el carrito, con el importe en divisa en cada línea](../img/tpv-venta.png)

La doble moneda está a la vista en tres sitios:

- **En la cabecera**, la tasa del día: `USD: 36.5`.
- **En cada línea**, el importe en bolívares y, debajo en rojo, el mismo importe
  en divisa.
- **En el pie**, los impuestos y el total.

El recorrido entero, del tablero al carrito:

![Abrir la caja y añadir dos productos](../media/recorrido-tpv-venta.webm)

## 3. Dónde se corta hoy

**Al pulsar «Pago» la caja se queda en blanco.** No es una pantalla vacía: se
cae el TPV entero y hay que recargar el navegador. La venta que estuviera
armada se pierde.

Es un fallo conocido y localizado —la lógica de IGTF y doble moneda del pago
llama a métodos que Odoo 20 renombró—, no un problema de configuración ni de
permisos. Está en la ficha técnica
[Punto de venta](/documentacion/20.0/desarrollador/punto-de-venta-12-modulos-el-40-en-javascript),
con el detalle de qué falta.

Por lo tanto, **en Odoo 20 esta caja todavía no factura**. Lo que va después del
carrito —cobrar, imprimir en la máquina fiscal, el voucher del punto, el
reporte Z— no está comprobado en esta versión y no debe darse por bueno.
