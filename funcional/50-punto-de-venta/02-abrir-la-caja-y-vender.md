# Abrir la caja y vender

El día a día del cajero: abrir la caja declarando lo que hay en la gaveta,
armar la venta viendo cada importe en las dos monedas, ponerla a nombre del
cliente cuando toca y aplicar descuentos sin romper la factura fiscal.

## 1. Abrir la caja

**Punto de venta → Tablero.** Cada caja es una tarjeta.

![El tablero del TPV, con la caja y su botón](../img/tpv-tablero.png)

### Para qué sirve la apertura

La **sesión** es el turno de la caja: todo lo que se venda hasta cerrarla
cuelga de ella. Al abrirla el cajero declara **cuánto efectivo hay en la
gaveta**; al cerrarla declara cuánto quedó, y el sistema compara: efectivo
inicial + cobros en efectivo − vueltos debe dar lo contado. La diferencia es el
sobrante o el faltante del turno, y queda registrada con nombre y hora.

### Por qué se cuenta en dos monedas

En un comercio que recibe dólares en efectivo, la gaveta tiene dos montones.
Contar solo los bolívares deja los dólares fuera del arqueo: nadie sabe si los
20 $ que entraron por la mañana siguen ahí. Por eso la caja puede pedir el
efectivo **en Bs y en USD** al abrir y al cerrar (**Apertura/Cierre en USD** en
la caja), y cada moneda cuadra por separado.

El botón de la tarjeta dice lo que va a pasar:

- **Abrir caja registradora** — no hay sesión abierta: se abre una nueva.
- **Seguir vendiendo** — ya hay una sesión abierta y se vuelve a ella.

### Paso a paso: abrir la caja al empezar el día

1. **Punto de venta → Tablero → Abrir caja registradora** en la tarjeta de la
   caja.
2. Espere. **El TPV tarda en abrir**: no es una pantalla más del sistema, es
   una aplicación aparte que se descarga entera la primera vez, con sus
   productos y su sesión. Veinte segundos con la pantalla en blanco son
   normales; el segundo arranque es rápido.
3. **Solo la primera vez**, Odoo pregunta cómo se exhiben los precios. Elija
   **Impuestos incluidos**: en el comercio venezolano el precio marcado ya
   lleva el IVA dentro, y así es como lo espera la máquina fiscal. No vuelve a
   preguntar.
4. Aparece el **control de apertura**: escriba el **efectivo inicial** que hay
   en la gaveta —y en divisa, si la caja tiene «Apertura/Cierre en USD»— y
   pulse **Abrir caja registradora**.
5. Resultado: la pantalla de venta, con los productos a la derecha y el
   carrito vacío a la izquierda. En la cabecera, la tasa del día (`USD: 36.5`).

![El TPV abierto, con los productos](../img/tpv-productos.png)

**Ejemplo.** La gaveta amanece con 500,00 Bs en sencillo y 40 $ de fondo para
dar vuelto en divisa. El cajero abre con **500,00** en Bs y **40,00** en USD.
Durante el turno cobra 2.100,00 Bs en efectivo y 65 $ en billetes, y da 12 $ de
vuelto. Al cerrar debe contar **2.600,00 Bs** y **93 $**. Si cuenta 90 $, el
cierre marca −3 $ en la columna de divisa y 0,00 en la de bolívares: se sabe
**en qué moneda** falta, no solo que falta.

> **Sin tasa del día no hay precios.** Si la cabecera dice `USD: 0` y los
> productos salen a 0,00 Bs, Contabilidad no tiene cargada la tasa de hoy. La
> caja lee la tasa **fechada hoy**; la de ayer no le sirve. Ver la guía de
> configuración, apartado de productos.

### Paso a paso: volver a una caja ya abierta

1. **Punto de venta → Tablero → Seguir vendiendo.**
2. Entra directo a la pantalla de venta, sin control de apertura. Si la caja
   tiene **Iniciar sesión como empleado**, pide el PIN del cajero.

## 2. Armar la venta

![Dos productos en el carrito, con el importe en divisa en cada línea](../img/tpv-venta.png)

### Por qué cada línea enseña dos importes

El vendedor y el cliente piensan en dólares —«el aceite está a 3,25»— y la
factura es en bolívares. Si la pantalla solo mostrara bolívares, el cajero
haría la cuenta de cabeza y el cliente no sabría si le cobraron lo que vio en
el anaquel. Por eso cada línea lleva el importe en Bs y, debajo en rojo, el
mismo importe en divisa, siempre con **la misma tasa** que se ve en la
cabecera: no hay dos tasas en una venta.

### Paso a paso: una venta

1. **Toque el producto** en la cuadrícula de la derecha: entra al carrito con
   cantidad 1. Tocarlo otra vez suma uno más.
2. Para cambiar la cantidad: seleccione la línea en el carrito, pulse **Cant.**
   en el teclado y escriba el número. En un producto **por pesar**, el peso con
   decimales.
3. Para un descuento en la línea: **%** y el porcentaje. Para cambiar el
   precio: **Precio** y el nuevo importe (en bolívares).
4. **+/-** invierte el signo (una devolución en la misma venta); **⌫** borra el
   último dígito y, con la cantidad en 0, quita la línea.
5. **Cliente** (botón sobre el teclado) para poner la venta a nombre de alguien:
   se busca por nombre, **cédula o RIF**.
6. **Nota** añade un comentario a la línea seleccionada.

### Ejemplo, línea a línea

Tasa del día: **36,50 Bs por dólar**. Precios con IVA (16 %) incluido.

| Acción | Línea | Bs | USD |
|---|---|---:|---:|
| Toca «Café molido 500 g» | 1 × 83,95 | 83,95 | 2,30 |
| Lo toca otra vez | 2 × 83,95 | 167,90 | 4,60 |
| Toca «Aceite de girasol 1 L» | 1 × 118,63 | 118,63 | 3,25 |
| Selecciona el aceite, **%**, `10` | 1 × 118,63 −10 % | 106,77 | 2,93 |
| **Pie** | Impuestos incluidos | 37,89 | 1,04 |
| **Total** | | **274,67** | **7,53** |

Lo que se lee de ahí: el IVA no se suma al final —ya está dentro de cada
precio— y el importe en divisa de cada línea es el de bolívares entre 36,50,
redondeado a dos decimales. Si mañana la tasa es 37,00, el café sigue costando
2,30 $ y pasará a 85,10 Bs sin que nadie toque el producto.

### El cliente: cuándo hace falta y cómo se busca

**Para qué**: la factura fiscal sale a «Consumidor final» si no se indica nadie.
Hay que poner cliente cuando:

- el cliente lo pide (necesita la factura a su nombre para su contabilidad),
- es **contribuyente especial** y va a retener IVA (la retención se calcula
  sobre él y sale en su comprobante),
- la venta va a **cuenta de cliente** (fiado) o financiada,
- va a haber **nota de crédito**: una devolución fiscal exige el documento y el
  cliente de la factura original.

**Por qué se busca por cédula o RIF**: es el dato que el cliente dicta y el que
imprime la máquina fiscal. El buscador de clientes de la caja encuentra por el
documento —`V12345678`, `J-12345678-9`— además de por nombre.

1. Pulse **Cliente**.
2. Escriba la cédula o el RIF (o el nombre) y elija.
3. Si no existe, **Crear**: nombre, **Tipo Documento** (V, E, J, P…) y el
   número. Con eso basta para la factura fiscal; el resto se completa después
   en Contactos.

### El descuento al total

**Para qué**: rebajar toda la venta de una vez —«te dejo todo con 10 %»— sin ir
línea por línea.

**Por qué está hecho así**: Odoo, por defecto, añade una línea negativa con un
producto «Descuento». En Venezuela eso no sirve: la máquina fiscal exige que el
descuento vaya **dentro de cada línea**, y la localización no admite líneas con
precio negativo. El botón **Descuento al total** de esta caja reparte el
porcentaje **a todas las líneas** como descuento de línea; el tique fiscal sale
con cada artículo y su rebaja, que es lo que el SENIAT espera.

1. Con productos en el carrito, pulse **Descuento al total**.
2. Escriba el porcentaje (0 a 100) y confirme.
3. Todas las líneas quedan con ese descuento. Si alguna ya tenía uno, **se
   reemplaza**, no se acumula.

Sin productos, avisa: *«Agregue productos a la orden antes de aplicar el
descuento»*. Quién puede hacerlo se controla con los permisos de gerente de la
caja.

**Ejemplo.** La venta anterior, con **Descuento al total** al 10 % en vez del
descuento solo en el aceite: café 2 × 83,95 −10 % = 151,11 Bs; aceite 106,77
Bs; total **257,88 Bs** (7,07 $). En el tique fiscal salen dos artículos, cada
uno con su «−10 %», y ninguna línea negativa.

### No vender lo que no hay

**Para qué**: que la caja no cobre un producto que el almacén no tiene.

**Por qué**: en una tienda con varias cajas o con venta por catálogo, el
cajero no ve el anaquel. La caja consulta **en línea** las existencias del
almacén de su tipo de operación al añadir un producto de tipo *Bienes*; si es
cero, no lo añade y avisa **Sin stock**: *«El producto seleccionado no posee
existencias»*. Los servicios no se validan.

No hay nada que hacer en la venta; lo que hay que cuidar es que la caja apunte
al almacén correcto (guía de configuración, apartado 1).

El recorrido entero, del tablero al carrito:

![Abrir la caja y añadir dos productos](../media/recorrido-tpv-venta.webm)

## 3. Dónde se corta hoy

**Al pulsar «Pago» la caja se queda en blanco.** No es una pantalla vacía: se
cae el TPV entero y hay que recargar el navegador. La venta que estuviera
armada se pierde.

### Qué hacer si pasa

1. Recargue la página del navegador (F5).
2. La caja vuelve a la pantalla de venta con el carrito vacío.
3. No repita el intento de cobro: fallará igual.

Es un fallo conocido y localizado —la lógica de IGTF y doble moneda del pago
llama a métodos que Odoo 20 renombró—, no un problema de configuración ni de
permisos. Está en la ficha técnica
[Punto de venta](../../modulos/tpv.md), con el detalle de qué falta.

Por lo tanto, **en Odoo 20 esta caja todavía no factura**. Cómo está diseñado
el cobro —y qué esperar cuando funcione— está en la guía siguiente,
[Cobrar en dos monedas y el IGTF](03-cobrar-en-dos-monedas-y-el-igtf.md).
