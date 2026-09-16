# Cobrar en dos monedas y el IGTF

Cómo la caja reparte un cobro entre varios métodos, cómo convierte lo que se
paga en divisa, cómo calcula el IGTF y cómo da el vuelto en las dos monedas.

> **Estado en Odoo 20: la pantalla de pago no abre todavía.** Lo que sigue es
> **cómo está diseñado** el cobro —leído del código que se migró y probado en
> la versión anterior—, para que se sepa qué esperar y qué revisar cuando se
> termine de portar. Los ejemplos son a **36,50 Bs por dólar**, la tasa de la
> caja de demostración.

## 1. Para qué sirve el cobro mixto

Un cliente rara vez paga una venta entera de una sola forma: trae unos dólares,
completa en bolívares con pago móvil y pide el vuelto en efectivo. La pantalla
de pago está hecha para eso: se van **añadiendo métodos** y cada uno toma una
parte del total, hasta que lo pagado cubre la venta. Cada método sabe **en qué
moneda cobra**, **si lleva IGTF** y **si convierte**.

## 2. Cómo se reparte el total entre métodos

La regla es simple y conviene conocerla porque explica lo que el cajero ve:

1. **El primer método que se toca toma el total completo** de la venta (salvo
   que sea un método de retención, que se explica más abajo). Si el cliente
   paga todo con ese método, ya está.
2. Si no, el cajero **corrige el importe** de esa línea a lo que el cliente da
   con ese método, y añade el siguiente: la caja propone para él **lo que
   falta**.
3. Cuando lo pagado alcanza el total, aparece **Validar**. Si lo pagado supera
   el total, la diferencia es el **vuelto**.

### Ejemplo A — todo en bolívares

Venta de **202,58 Bs** (café 83,95 + aceite 118,63; 5,55 $). El cliente paga en
efectivo con 250,00 Bs.

| Método | Importe |
|---|---:|
| Efectivo Bs (propuesto: 202,58; el cajero escribe lo entregado) | 250,00 |
| **Vuelto** | **47,42 Bs** (1,30 $) |

No hay IGTF: no se pagó nada en divisa.

## 3. Pagar en otra moneda

**Para qué**: que el cajero teclee lo que el cliente entrega **en la moneda en
que lo entrega** —«20 dólares»— y la caja anote los bolívares equivalentes,
sin que nadie multiplique de cabeza.

**Por qué está hecho así**: el sistema contabiliza en bolívares y Odoo exige que
todos los métodos de la caja estén en la moneda de la compañía. El método
«Efectivo USD» no es un método en dólares: es un método en bolívares que
**muestra y pide** dólares (**Cobra en otra moneda: Pago en otra moneda**,
**Moneda: USD**) y convierte con la tasa del día. En el pago quedan guardados
los tres datos —**Monto en divisa**, **Tasa** e importe en Bs— para que el
cierre y los reportes puedan decir cuántos dólares entraron de verdad.

### Ejemplo B — 20 $ en billetes y el resto en bolívares

Misma venta de **202,58 Bs**. El cliente da 20 $ y quiere el resto en
bolívares… pero pagar en divisa lleva IGTF, así que el total cambia. Se ve en el
apartado siguiente.

## 4. El IGTF

**Qué es**: el Impuesto a las Grandes Transacciones Financieras grava, al **3 %**,
los pagos hechos en moneda extranjera fuera del sistema financiero nacional
—los dólares en efectivo, el Zelle—. Lo paga quien paga en divisa; el comercio
lo recauda y lo declara.

**Para qué lo calcula la caja**: para que el cajero no tenga que saber la regla
ni hacer la cuenta, y para que el impuesto quede **dentro de la venta**, en la
factura fiscal y en la contabilidad, no anotado en un papel aparte.

**Por qué es una línea más de la venta**: la máquina fiscal solo imprime lo que
está en el tique. Si el IGTF no fuera una línea, no saldría en la factura ni se
contabilizaría. Por eso la localización lo añade como un producto —el
**Producto IGTF** de Contabilidad → Ajustes— con el importe calculado, y lo quita
si al final nadie pagó en divisa.

### Cómo se calcula

1. **Base**: la suma de lo pagado con métodos que tengan **Retener igtf**
   marcado (efectivo USD, Zelle…). La base **nunca supera el total de la
   venta**: si el cliente paga 100 $ por una venta de 60 $, el IGTF se calcula
   sobre 60, no sobre 100; lo demás es vuelto.
2. **IGTF** = base × **Tasa IGTF (%)** de la compañía (3 %).
3. La caja **añade la línea «IGTF»** por ese importe; el total sube, y lo que
   falta por pagar sube con él. Si el cajero cambia los pagos, la línea se
   recalcula; si quita el pago en divisa, la línea desaparece.

### Ejemplo B, resuelto

Venta de **202,58 Bs** a 36,50. El cliente da **20 $**.

| Paso | Bs | USD |
|---|---:|---:|
| Toca **Efectivo USD**: propone el total, el cajero escribe `20` | 730,00 | 20,00 |
| La caja ve que 730,00 supera la venta: la base del IGTF se limita al total | base 202,58 | |
| Añade la línea **IGTF** 3 % sobre 202,58 | + 6,08 | + 0,17 |
| **Nuevo total de la venta** | **208,66** | **5,72** |
| Pagado: 20 $ | 730,00 | 20,00 |
| **Vuelto** | **521,34 Bs** | **14,28 $** |

El cajero devuelve 14,28 $ si tiene divisa, o 521,34 Bs, o una mezcla: la caja
enseña las dos cifras (apartado 6).

### Ejemplo C — el cliente completa en bolívares

Venta de **3.650,00 Bs** (100 $). El cliente tiene **60 $** y paga el resto por
pago móvil.

| Paso | Bs |
|---|---:|
| **Efectivo USD**, el cajero escribe `60` | 2.190,00 |
| Base del IGTF = lo pagado en divisa (2.190,00, menor que la venta) | |
| Línea **IGTF** 3 % sobre 2.190,00 | + 65,70 |
| **Nuevo total** | **3.715,70** |
| Falta: 3.715,70 − 2.190,00 | 1.525,70 |
| **Pago móvil** (Referencia requerida: escribe el número de la operación) | 1.525,70 |
| **Vuelto** | 0,00 |

El IGTF recayó solo sobre los 60 $: lo pagado por pago móvil, que es sistema
financiero nacional, no lo lleva. Es exactamente la regla del impuesto.

> **Si el Producto IGTF no está configurado**, la caja avisa —*«Producto IGTF
> no configurado. Configúrelo en Punto de Venta > Configuración»*— y **no
> cierra la venta con pago en divisa**. Es deliberado: mejor no cobrar que
> cobrar sin el impuesto.

## 5. Cobrar con el punto bancario

**Para qué**: que el importe viaje de la caja al terminal y la respuesta
vuelva sola, sin teclear el monto dos veces ni cuadrar vouchers a mano.

**Por qué dos terminales**: cada banco o adquirente entrega el suyo. **VPOS** es
la terminal de pago con tarjeta (y sus variantes: anulación, vuelto en punto,
Cashea); **SITEF** es la plataforma que, además de la tarjeta en el PinPad,
valida pago móvil, transferencias, Zelle y Cashea.

1. El cajero toca **Punto de venta (VPOS)** o el método SITEF que corresponda.
2. La caja envía el importe al terminal y espera. El cliente pasa la tarjeta o
   confirma la operación en su teléfono.
3. Si el terminal **aprueba**, la línea de pago queda confirmada con la
   **referencia** y el **lote** que devolvió el banco (se guardan en el pago).
   Si **rechaza**, la línea se marca y se puede reintentar u otro método.
4. Con **Imprimir Voucher** activado en Ajustes, el comprobante del punto sale
   por el servicio de impresión.

Los pagos con punto **no llevan IGTF**: son en bolívares por el sistema
financiero nacional.

## 6. El vuelto en dos monedas

**Para qué**: cuando el cliente paga en dólares, el vuelto puede darse en
dólares, en bolívares o en parte y parte. La caja enseña el vuelto **en las dos
monedas** —vuelto en Bs y su equivalente en USD a la tasa de la caja— y el
cajero decide con qué billetes lo devuelve.

**Ejemplo.** Vuelto de **521,34 Bs** (ejemplo B) a 36,50 = **14,28 $**. El
cajero puede devolver **14 $ y 10,34 Bs** (14 × 36,50 = 511,00; 521,34 −
511,00 = 10,34), o los 521,34 Bs en billetes locales si no quiere soltar
divisa. Lo que se registra en la sesión es el vuelto en bolívares; los dólares
que salieron de la gaveta se reflejan en el conteo de cierre en USD.

## 7. La retención en caja

**Para qué**: un **contribuyente especial** está obligado a retener parte del
IVA de lo que compra (75 % o 100 %) y a entregar el comprobante de retención. Si
compra en la caja, paga **menos** que el total: lo retenido no lo paga a la
tienda, lo entera al SENIAT.

**Por qué es un método de pago**: para la caja, la retención es una forma de
«cubrir» parte del total sin recibir dinero, y así la venta queda cuadrada. El
método con **Cobra en otra moneda: Retención IVA** calcula el importe solo
—**% de retención** sobre el IVA de la venta— y lo registra contra la cuenta
de retenciones por cobrar, que después se cruza con el comprobante.

### Ejemplo D — cliente especial al 75 %

Venta de **1.160,00 Bs**: base 1.000,00 + IVA 160,00. El cliente es agente de
retención al 75 %.

| Paso | Bs |
|---|---:|
| Elige al cliente (con su RIF): la retención se calcula sobre él | |
| Toca **Retención IVA 75 %**: la caja calcula 75 % del IVA | 120,00 |
| Escribe la **referencia** (el número del comprobante de retención) | |
| Toca **Punto de venta (VPOS)**: propone lo que falta | 1.040,00 |
| **Pagado + retenido** | **1.160,00** |

La factura fiscal sale por 1.160,00; el comprobante del cliente, por 120,00; la
caja recibió 1.040,00. Con **Retención Base** el porcentaje se aplica sobre la
base imponible (1.000,00), que es como se retienen impuestos municipales o
ISLR: al 1 % serían 10,00.

> Los métodos de retención **nunca se proponen como primer método** con el total
> completo: siempre calculan su porcentaje, aunque se toquen antes que los
> demás.

## 8. Qué se guarda de cada cobro

Para que el cierre y los reportes cuadren en las dos monedas, cada pago
conserva:

| Dato | Para qué |
|---|---|
| **Importe** en Bs | Lo que suma a la venta y a la contabilidad |
| **Monto en divisa** y **Tasa** | Cuántos dólares entraron de verdad y a qué tasa se convirtieron |
| **Nro. de referencia de pago** | La operación del banco, del pago móvil o el comprobante de retención |
| **Lote** | El lote del terminal, para el cierre del punto |
| **Nro. de factura**, **Nro. de control**, **Nro. Reporte Z** | Los datos fiscales de la venta a la que pertenece |
| **Categoría de pago** | El grupo en el que se totaliza al cerrar (Efectivo, Punto de venta, Divisas) |

Con eso, el **reporte de cierre** puede decir cuánto entró por cada método en
bolívares **y en dólares**, y el **libro de ventas por reporte Z** puede
reconstruir el día factura por factura (guía siguiente).
