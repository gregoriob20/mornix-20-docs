# Cómo se configura la caja

Lo que hay que dejar puesto **una vez** para que el TPV abra: los productos que
se venden en caja, los métodos de pago y la moneda de exhibición.

## 1. Los productos que salen en caja

Un producto aparece en el TPV solo si lleva marcado **Disponible en el punto de
venta** (en la ficha del producto, pestaña *Ventas*). No basta con que exista ni
con que tenga precio.

> **El precio en bolívares no se escribe a mano.** En esta localización sale del
> precio en divisa por la tasa del día. Si los productos salen todos a 0,00 en la
> caja, lo que falta es la tasa, no el precio. Ver
> [Doble moneda](../10-contabilidad/06-doble-moneda.md).

## 2. Los métodos de pago

**Punto de venta → Configuración → Métodos de pago.**

![Los métodos de pago de la caja: efectivo y los dos terminales](../img/tpv-metodos-pago.png)

Cada método dice por dónde entra el dinero:

| Campo | Qué decide |
|---|---|
| **Tipo** | Si el importe entra como efectivo, por banco o a cuenta del cliente |
| **Diario** | El diario contable donde se registra el cobro |
| **Proveedor de pago** | El terminal, cuando el método es un punto bancario |
| **Punto de venta** | En qué cajas se ofrece el método |

En la demo hay tres: **Efectivo Bs** contra el diario de efectivo, y los dos
puntos bancarios de la casa —**VPOS** y **SITEF**— contra el diario de banco.

> **Dos cosas cambiaron en Odoo 20.** El **Tipo** es ahora obligatorio: un
> método sin tipo no se puede guardar. Y el terminal ya no se elige en «Usar
> terminal de pago», sino en **Proveedor de pago**. Un método traído de la
> versión anterior llega sin ninguno de los dos y hay que ponérselos.

## 3. La caja

**Punto de venta → Configuración → Punto de venta → Caja principal.**

![Los ajustes de la caja, con la doble moneda](../img/tpv-caja-ajustes.png)

Aquí, además de los ajustes de Odoo, están los de la localización:

| Ajuste | Para qué sirve |
|---|---|
| **Show dual currency** | Añade a cada producto y a cada línea su precio en la otra moneda |
| **Currency** | La moneda de exhibición. En Venezuela, USD |
| **Rate** | La tasa vigente de esa moneda, tal como está en Contabilidad |
| **Apertura/Cierre en USD** | Cuenta la caja en divisa al abrir y al cerrar |
| **Servidor Fiscal** | La máquina fiscal a la que se manda la impresión |

> **La tasa se ve aquí, pero no se pone aquí.** El campo **Rate** es la tasa de
> la moneda en Contabilidad, y se muestra como Odoo la guarda: dólares por
> bolívar (`0,0273972…`), no bolívares por dólar. En la caja se ve del derecho
> —`USD: 36.5` en la cabecera—. Para cambiarla se carga la tasa del día en
> Contabilidad; tocarla aquí no es el camino.

> **Con la caja abierta no se cambian los ajustes.** Odoo avisa en una franja
> amarilla: *«Hay una sesión abierta para este PdV. Antes de cambiar algunos
> ajustes debe cerrar la sesión»*. Es de Odoo, no de la localización.

> **Varios rótulos salen en inglés** —«Show dual currency», «Currency», «Rate»—:
> son campos de la localización que todavía no tienen traducción al español de
> Venezuela. Es cosmético, y está anotado.

## 4. El IGTF

**Contabilidad → Configuración → Ajustes → IGTF**: la tasa del impuesto y el
producto con el que se cobra.

> El IGTF **no está portado a Odoo 20 todavía**. La configuración se guarda,
> pero no se aplica: el cobro es justo la parte que no funciona. No se use esta
> caja para cobrar en divisa hasta que se cierre esa parte.
