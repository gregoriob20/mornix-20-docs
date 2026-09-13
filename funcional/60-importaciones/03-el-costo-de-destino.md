# El costo de destino

Es donde los gastos del viaje —flete, seguro, aduana, almacenaje— acaban
sumados al costo de la mercancía. Cada expediente nace con su costo de destino
enlazado; esta guía es cómo se llena y cómo se reparte.

## El flujo

1. **Llegan las facturas** del viaje —la naviera, el agente de aduanas— y se
   registran **con el expediente puesto**. Los gastos entran solos al costo de
   destino.
2. **Se calcula**: el costo por unidad en puerto y en almacén.
3. **Se reparte**: se crea el costo adicional de Odoo, se calcula y se valida.
   El costo del producto sube y queda el asiento.

Así queda el ejemplo comprobado en la base de pruebas:

| Paso | Importe |
|---|---|
| Compra: 100 unidades a 18 USD | 1.800 USD = 65.700 Bs |
| Flete marítimo | 2.500 USD = 91.250 Bs |
| Gastos de aduana | 900 USD = 32.850 Bs |
| **Costo final de la mercancía** | **189.800 Bs — 1.898 Bs por unidad** |

## 1. Registrar la factura del viaje

Los gastos **no se escriben a mano** en el costo de destino. Entran al
**publicar la factura del proveedor** con el expediente puesto.

![La factura del flete, con la casilla marcada y su expediente](../img/imp-factura-form.png)

### Paso a paso

1. **Importaciones → Proveedores → Facturas → Nuevo** (es la pantalla de
   facturas de proveedor de Contabilidad).
2. **Proveedor**: la naviera, el agente de aduanas, la aseguradora.
3. Marque la casilla **Proceso de Importación**. Al marcarla aparece **Nro de
   Expediente de Importacion**: elija el expediente. **Es el campo que lo
   conecta todo**: sin él, la factura se contabiliza igual pero el costo de
   destino no se entera.
4. **Fecha de la factura**, **Número de Factura de Proveedor** y **Número de
   Control** (los pide la localización).
5. **Moneda**: la de la factura. Si viene en divisa, la **Tasa** se toma del
   día; el importe en bolívares lo calcula el sistema.
6. Pestaña **Líneas de factura → Agregar una línea**: el servicio (*Flete
   marítimo*, *Gastos de aduana*…), cantidad e importe. **Tienen que ser los
   productos de servicio marcados como costo en destino**: una línea con otro
   producto no llega al reparto.
7. **Confirmar.** La factura pasa a **Registrado**.
8. Resultado: en el costo de destino del expediente, pestaña **Costos
   Asociados**, aparece una línea por cada servicio de la factura, con su
   importe en bolívares y en divisa. Y el chatter del costo de destino registra
   qué factura la trajo.

> Si los gastos no aparecen, lo que falta casi siempre es el expediente en la
> factura —la casilla marcada pero el número sin elegir—, o que el producto del
> servicio no tenga marcado **Puede ser costo en destino**.

> Si la factura se devuelve a borrador (**Restablecer a borrador**), sus gastos
> asociados **se quitan** del costo de destino, y vuelven al confirmarla otra
> vez. No quedan gastos huérfanos.

## 2. Calcular el costo de destino

![La ficha de costos de destino, con las líneas de producto y los gastos](../img/imp-costo-destino-form.png)

### Paso a paso

1. Abra el expediente y pulse el botón **Costos Asociados** (arriba), o
   **Importaciones → Operaciones → Costo en destino**.
2. **Método de División**: cómo se reparten los gastos entre los productos.
   - *Cantidad*: a partes iguales por unidad.
   - *Peso* / *Volumen*: proporcional al peso o al volumen del producto (hay
     que tenerlos en la ficha).
   - *Capacidad del contenedor*: según cuánto ocupa cada producto en el
     contenedor; pide **Tipo de contenedor** y **Número de contenedores**.
3. **Tarifa Plana**: márquela solo si se paga un monto fijo por el viaje en
   lugar de los gastos detallados; entonces escriba el importe en divisa.
4. **Calcular Gastos.** El sistema lee las órdenes de compra del expediente y
   arma la pestaña **Líneas de costo**: una fila por producto con su cantidad y:

   | Columna | Qué es |
   |---|---|
   | **FOB** | El precio de compra por unidad, en bolívares y en divisa |
   | **CIF** | Lo que le toca de flete y seguro |
   | **Costo nacional** | Lo que le toca de aduana, almacenaje, flete terrestre y gastos generales |
   | **En puerto** | FOB + CIF |
   | **En almacén** | En puerto + costo nacional: el costo unitario final |
   | **Precio de venta** | En almacén más el % de utilidad de la ficha del producto |

5. Revise que **En almacén** cuadre con lo que espera. En el ejemplo: 657 de
   FOB + 912,50 de CIF + 328,50 nacional = **1.898 Bs**.
6. **Confirmar** deja el cálculo en firme (**Confirmado**). Si hay que
   retocar, **Restablecer a borrador**.

## 3. Repartir sobre la mercancía

El reparto es un **costo adicional** de Odoo (`Inventario → Operaciones →
Costos en destino`): sube el costo del producto recibido y deja el asiento.

![El costo adicional validado, con sus dos líneas de gasto](../img/imp-costo-adicional-form.png)

### Paso a paso

1. En el costo de destino, pulse **Crear costos de destino**. Necesita que la
   recepción esté validada (campo **Albaranes** lleno) y que haya gastos
   asociados; si falta algo, lo dice en un aviso y no crea nada.
2. Se abre el **costo adicional** ya armado: los **Traslados** (la recepción),
   una línea por gasto en **Costos adicionales**, con su **Cuenta**, su
   **Método de división** y su **Costo** en bolívares.
3. **Calcular.** La pestaña **Ajustes de valoración** enseña, por producto, el
   coste previo y lo que se le reparte.
4. **Validar.** El estado pasa a **Registrado**, se genera el **Asiento
   contable** (`STJ/…`) y el costo unitario del producto sube.
5. Compruebe en la ficha del producto: **Costo** debe ser el «En almacén» del
   paso 2 (1.898 Bs en el ejemplo), y **Inventario → Reportes → Valoración**
   debe sumar mercancía más gastos (189.800 Bs).

## 4. Lo que hay que saber

> **Ojo con el producto.** En Odoo 20 un producto de tipo «bienes» que **no
> esté marcado como almacenable** («Rastrear inventario» en su ficha) deja
> hacer todo el camino —se recibe, se reparte, el costo unitario sube— **sin
> generar un solo asiento contable**. Si el valor en inventario sale 0 con el
> costo unitario ya subido, es eso.

> **La contrapartida del gasto la elige el sistema**, y hoy toma la primera
> cuenta de gasto que encuentra —en el ejemplo, *Cost of Goods Sold*—, no la
> cuenta de la factura. Si su contabilidad necesita otra, hay que decirlo:
> está anotado como pendiente.

> **El detalle en divisa del reparto no está todavía.** El costo adicional se
> hace en bolívares; las columnas en divisa dependen de un módulo que no se ha
> migrado. El cálculo del costo de destino (paso 2) sí lleva las dos monedas.

> **Permisos.** Para abrir el costo de destino hace falta **Inventario /
> Administrador** (enseña los costos adicionales); para registrar la factura
> del viaje con la localización puesta, **Contabilidad / características
> completas**. Con menos permisos la pantalla se queda cargando o da «Error de
> acceso», sin decir por qué.
