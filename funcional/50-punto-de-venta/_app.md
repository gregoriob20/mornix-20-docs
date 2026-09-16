# Punto de venta

La caja del comercio venezolano: precios marcados en divisa y cobrados en
bolívares a la tasa del día, cobros mezclados —efectivo en dos monedas, punto
bancario, pago móvil, retención—, el IGTF sobre lo pagado en divisa, y la
factura por máquina fiscal que exige el SENIAT.

## Por qué esta caja no es la de Odoo «a secas»

El punto de venta de Odoo está pensado para un comercio que marca y cobra en
una sola moneda, sin máquina fiscal y sin impuestos sobre la forma de pago. En
Venezuela ninguna de las tres cosas se cumple:

| Realidad del comercio | Qué le añade la localización |
|---|---|
| Los precios se piensan en dólares porque el bolívar cambia a diario; se cobra en bolívares porque es la moneda de curso legal | **Doble moneda**: el precio se escribe en divisa, la caja lo convierte a la tasa del BCV y enseña las dos cifras en cada línea |
| El cliente paga con lo que tiene: unos bolívares en efectivo, unos dólares, el resto por punto o pago móvil | **Cobro mixto**: cada método sabe en qué moneda cobra y qué convierte |
| Pagar en divisa está gravado con el **IGTF** (3 %) | La caja calcula el IGTF sobre lo que se pagó en divisa y lo añade a la venta como una línea más |
| Un contribuyente especial retiene parte del IVA al pagar | **Métodos de retención**: descuentan en vez de cobrar, y dejan el rastro para el comprobante |
| La factura sale por **máquina fiscal**, con su número y su serial, y el día se cierra con el **reporte Z** | Servidor fiscal, impresoras registradas, reimpresión, reconciliación y el libro de ventas por Z |

Estas guías explican cada pieza en tres tiempos: **para qué sirve**, **por qué
está así** y **cómo se hace**, con un ejemplo numérico donde hace falta.

## Las guías

| Guía | Qué contesta |
|---|---|
| [Cómo se configura la caja](01-configurar-la-caja.md) | Productos, métodos de pago por caso, la caja, la máquina fiscal, los terminales y el IGTF |
| [Abrir la caja y vender](02-abrir-la-caja-y-vender.md) | El día a día: apertura con conteo, la venta línea a línea, el cliente, el descuento |
| [Cobrar en dos monedas y el IGTF](03-cobrar-en-dos-monedas-y-el-igtf.md) | Cómo se reparte un cobro entre métodos, cómo se calcula el IGTF, el vuelto en las dos monedas, la retención en caja |
| [La máquina fiscal, el cierre y los reportes](04-la-maquina-fiscal-el-cierre-y-los-reportes.md) | Factura y nota de crédito fiscal, reimprimir, reconciliar, el reporte Z, el cierre en dos monedas y el libro de ventas |

## Qué está comprobado en Odoo 20

> **La caja abre y vende; todavía no cobra.** Los productos, la doble moneda
> línea a línea, los métodos de pago, la caja, la máquina fiscal en modo prueba
> y los terminales **se configuran y se ven funcionar**. Al pulsar **Pago** la
> pantalla se queda en blanco: la lógica de IGTF y doble moneda del pago está a
> medio portar.
>
> Las guías 3 y 4 describen **cómo está diseñado** el cobro, la impresión fiscal
> y el cierre —leído del código que se migró— para que se sepa qué esperar y
> qué revisar cuando se termine de portar. Cada apartado dice si está comprobado
> en esta versión o no. El detalle técnico está en la ficha
> [Punto de venta](../../modulos/tpv.md).
