# La máquina fiscal, el cierre y los reportes

Qué pasa con la venta cuando el cajero pulsa **Validar**: la factura sale por la
máquina fiscal, la caja guarda el número que la impresora devolvió, el día se
cierra con el reporte Z y contabilidad reconstruye el libro de ventas a partir
de esos números.

> **Estado en Odoo 20.** La máquina fiscal en modo **Prueba** está comprobada
> (la caja abre y cierra con ella). La impresión real, la reimpresión, la
> reconciliación y el reporte Z dependen de la pantalla de pago, que todavía no
> abre; están descritos **como están diseñados en el código migrado** y se
> señalan como *no comprobado en v20*. El libro de ventas por reporte Z sí se
> puede abrir y probar con datos manuales.

## 1. Por qué una máquina fiscal

En Venezuela el comercio al detal no factura con un PDF: la Providencia 0071 del
SENIAT obliga a emitir la factura por una **máquina fiscal** homologada, que
lleva su propia numeración correlativa, guarda una memoria fiscal que nadie
puede editar y cierra cada jornada con un **reporte Z** que totaliza el día. La
factura de Odoo, sola, no tiene validez fiscal.

Lo que hace la localización es **conectar la caja con esa máquina** y traer de
vuelta lo que la máquina asignó —número de factura, serial, número de Z— para
que la venta de Odoo y el tique fiscal sean el mismo documento.

### Cómo se conecta

La caja no habla con la impresora directamente: habla con un **servidor de
impresión** (`fp_server`) que corre en la PC de la caja y tiene el driver de la
marca. Por eso en la caja se configura una **URL** (`http://localhost:5000`) y
no un puerto serie. El servidor puede ser:

| Servidor | Para qué |
|---|---|
| **Prueba** | No imprime nada ni exige servidor. La caja funciona igual y no pide datos fiscales. Es el modo de las bases de demostración y de la capacitación. |
| **HK** | Impresoras The Factory HKA (las más comunes: HKA80, HKA112, Tally). |
| **PNP** | Impresoras PNP / Bematech. |

Los detalles de configuración —registrar el serial, apuntar la caja al servidor,
las opciones de impresión— están en la guía de configuración, apartado 4.

## 2. La factura fiscal al cobrar

**Para qué**: que el tique fiscal salga solo, con los datos correctos, sin que el
cajero teclee nada en la impresora.

**Cómo funciona** (*no comprobado en v20*):

1. El cajero pulsa **Validar** en la pantalla de pago.
2. La caja arma la factura fiscal con lo que hay en la venta: el cliente y su
   **cédula o RIF**, cada línea con su precio, su descuento y su tasa de IVA,
   la línea de IGTF si la hubo, y cada pago con su método.
3. La envía al servidor de impresión. La impresora imprime y **devuelve tres
   datos**: el **número de factura fiscal**, su **serial** y el **número del
   último Z**.
4. La caja los guarda en la venta —**Número Factura Fiscal**, **Serial Fiscal**,
   **Reporte Z Fiscal**— y los copia a la factura contable, donde son el número
   de factura y el número de control.
5. Pasa a la pantalla de recibo. Si algo falló, aparece **Imprimir Factura
   Fiscal** para reintentar; si ya se imprimió, **Reimprimir Fiscal**.

### Qué comprueba antes de imprimir, y por qué

| Comprobación | Mensaje | Por qué |
|---|---|---|
| Hay cliente | *Debe seleccionar un cliente* | La factura fiscal lleva siempre un receptor, aunque sea el consumidor final genérico |
| El cliente tiene cédula o RIF | *El cliente no tiene un número de identificación fiscal* | Es el dato que identifica al comprador ante el SENIAT |
| Ninguna línea a precio cero | *No se puede imprimir una factura con precio cero* | La máquina fiscal rechaza ítems sin importe; un regalo se factura con precio y 100 % de descuento |
| La venta no está ya impresa | *Esta orden ya está impresa fiscalmente* | La numeración fiscal es correlativa: imprimir dos veces sería emitir dos facturas |

**Ejemplo.** La venta del ejemplo B de la guía anterior (café y aceite, 20 $ con
IGTF, total 208,66 Bs). Al validar, la impresora HKA devuelve
`nroFiscal: 2417`, `serial: Z7C7037796`, `ultimoZ: 445`. La venta queda con
**Número Factura Fiscal 00002417**, **Serial Fiscal Z7C7037796** y **Reporte Z
Fiscal 00000445**; la factura contable toma el 00002417 como número. El 445 es
el Z **anterior**: esta factura entrará en el 446, el que se emita al cerrar.

### El descuento y el IGTF en el tique

- El descuento de cada línea sale **en la línea**, nunca como un ítem negativo;
  por eso el **Descuento al total** se reparte a las líneas (guía 2). Con **No
  mostrar descuento en la factura** activo, el tique lleva el precio ya rebajado.
- El IGTF viaja como **un ítem más** del tique y como su propio pago, para que
  la memoria fiscal lo totalice aparte.

## 3. Cuando la impresora no contesta: «Pendiente de reconciliar»

**El problema**: una impresora HKA con la cola llena, o con un Z pendiente, puede
tardar más de lo que la caja espera. La venta **ya se cobró** y la factura de
Odoo **ya se generó**, pero la respuesta con el número fiscal no llegó.

**Por qué no se anula**: el tique físico **sí salió** casi siempre; anular la
venta dejaría un tique fiscal sin venta, que es peor que una venta sin número.
Así que la caja marca la venta como **Pendiente de reconciliar**, avisa al
cajero de que revise la impresora, y sigue.

### Paso a paso: reconciliar (*no comprobado en v20*)

Lo hace un administrador, con el tique físico en la mano.

1. **Punto de venta → Pedidos → Pedidos**, filtre las ventas con **Pendiente de
   reconciliar**.
2. Abra la venta y pulse **Reconciliar fiscal**.
3. Copie del tique los tres datos: **Serial impresora fiscal** (`Z7C7037796`),
   **Nº Factura Fiscal** (`2417`; la caja lo completa a `00002417`) y **Nº
   Reporte Z** (`445`).
4. **Reconciliar**. La venta deja de estar pendiente, los datos pasan a la
   factura contable y queda una nota en el chatter con quién lo hizo y qué
   escribió.

Si la venta ya tenía número fiscal, el asistente se niega: *«Esta orden ya tiene
número fiscal … edita los campos directamente con un usuario administrador»*.
Es a propósito: corregir un número fiscal a mano es una decisión que debe dejar
rastro.

## 4. Nota de crédito fiscal

**Para qué**: una devolución no se «borra»: se emite una **nota de crédito** que
apunta a la factura original. La máquina fiscal exige el número de la factura
afectada y su serial.

**Cómo funciona** (*no comprobado en v20*):

1. En la caja, **Pedidos** → seleccione la venta → **Reembolso**: Odoo crea una
   venta con las líneas en negativo enlazadas a la original.
2. Al validar, la caja detecta que es un reembolso y arma una **nota de
   crédito** en vez de una factura: manda el **número de factura afectada** y
   el **serial** de la venta original, y las líneas devueltas.
3. La impresora devuelve el número de la nota; se guarda igual que el de una
   factura.

**Ejemplo.** El cliente devuelve el aceite (106,77 Bs) de la factura 00002417.
La devolución sale como nota de crédito por 106,77 Bs afectando a la 00002417,
serial Z7C7037796. La 00002417 no se toca: en el libro de ventas quedan las dos,
la factura por 208,66 y la NC por −106,77.

> Si la venta original no tiene número fiscal (está pendiente de reconciliar),
> primero se reconcilia y después se devuelve. Sin factura afectada no hay nota
> de crédito.

## 5. Reimprimir

**Para qué**: el papel se atascó, el cliente perdió el tique, contabilidad
necesita una copia de un rango de facturas. La memoria fiscal permite reimprimir
cualquier documento por número.

Hay tres caminos (*no comprobados en v20*):

| Desde | Cuándo |
|---|---|
| Pantalla de recibo → **Reimprimir Fiscal** | El tique de la venta que se acaba de cobrar |
| Menú de la caja (☰) → **Reimpresión Fiscal** | Cualquier documento o rango, sin salir de la caja |
| Contabilidad → factura → **Reimprimir Documento** | Desde la oficina, para una factura concreta |

### Paso a paso: reimprimir un rango

1. En la caja, menú ☰ → **Reimpresión Fiscal**.
2. **Documento**: Factura, Nota de Crédito, Nota de Débito o Reporte Z.
3. **Desde** y **Hasta**: los números fiscales. Para uno solo, el mismo en los
   dos. Si Hasta es menor que Desde, avisa: *«El Número Hasta no puede ser
   menor al Número Desde»*.
4. **Imprimir**. Confirma: *«Documento fiscal reimpreso exitosamente»*.

**Ejemplo.** Contabilidad pide copia de las facturas de la mañana: Documento
**Factura**, Desde `2410`, Hasta `2417`. Salen ocho tiques marcados como copia.

Con **Generar copia automática** en la caja, cada factura sale por duplicado
sin pedirlo: la caja lanza la reimpresión un segundo después de imprimir.

## 6. Cerrar la caja

### Para qué sirve el cierre

Cierra el turno: el cajero **cuenta** lo que hay en la gaveta, la caja lo
**compara** con lo que debería haber, y contabiliza la sesión —ventas, cobros por
método, diferencias—. Después del cierre la sesión no admite ventas.

### Por qué en dos monedas

Con **Apertura/Cierre en USD** en la caja, el control de cierre pide dos
conteos: **bolívares** y **dólares** (**Conteo en USD**, con la tasa al lado).
Cada método de pago muestra su total en Bs y, debajo, su **equivalente en
divisa** a la tasa de la caja. Así el cierre dice no solo si falta plata, sino
**en qué moneda** falta: un faltante de 3 $ con los bolívares cuadrados apunta a
un vuelto en divisa mal dado, no a un error de cobro.

### Paso a paso: cerrar (*el cierre con conteo está comprobado; el Z no*)

1. En la caja, menú ☰ → **Cerrar caja registradora**.
2. Se ve el resumen de la sesión: total vendido, y cada método con su importe
   —efectivo con lo esperado y lo contado; los demás, con lo esperado—. En rojo,
   el equivalente en USD de cada cifra.
3. Cuente la gaveta y escriba **lo contado** en bolívares (con **Monedas y
   billetes** para desglosar) y en **Conteo en USD**.
4. Si la diferencia no es cero, escriba el motivo en la nota.
5. **Cerrar sesión**. Si el servidor fiscal no es *Prueba* y la caja está en
   **Reporte Z al cerrar: Automático**, en ese momento **se imprime el reporte
   Z** y después se cierra la sesión.

**Ejemplo.** Sesión con la apertura del ejemplo de la guía 2 (500,00 Bs y 40 $)
y estas ventas:

| Método | Esperado Bs | En USD (÷ 36,50) |
|---|---:|---:|
| Efectivo Bs (500,00 + 2.100,00 − vueltos 0,00) | 2.600,00 | 71,23 |
| Efectivo USD (65 $ cobrados − 12 $ de vuelto = 53 $ × 36,50) | 1.934,50 | 53,00 |
| Punto de venta (VPOS) | 4.380,00 | 120,00 |
| Pago móvil | 1.525,70 | 41,80 |
| **Total sesión** | **10.440,20** | **286,03** |

Conteo: 2.600,00 Bs → diferencia 0,00. **Conteo en USD**: 90 $ (40 de fondo + 53
de ventas = 93 esperados) → **−3 $**. El cierre queda con la nota «vuelto en
divisa por revisar», y el faltante está localizado en la gaveta de dólares.

### Reporte X y reporte Z

Los dos son totales de la máquina fiscal, pero no son lo mismo:

| | Reporte X | Reporte Z |
|---|---|---|
| Qué es | Un **corte parcial**: cuánto lleva vendido la máquina hoy | El **cierre fiscal del día**: totaliza y **reinicia** los acumuladores |
| Cuántas veces | Las que haga falta | **Una** por jornada; queda grabado en la memoria fiscal |
| Botón | **ReporteX** en el cierre | **ReporteZ** en el cierre (solo en modo Manual) |

**Por qué hay modo Automático y Manual**: si una tienda tiene **dos cajas con
la misma impresora**, el Z se emite una vez al final, no al cerrar cada caja; ahí
se pone la caja en **Manual** y el encargado pulsa **ReporteZ** cuando cierra la
última. Con una impresora por caja, **Automático** evita que se olvide. Si el
servidor no está configurado, avisa: *«Servidor no configurado: entra en las
configuraciones de la caja y valida el servidor de impresión»*.

## 7. El libro de ventas por reporte Z

**Para qué**: el libro de ventas de IVA de un comercio con máquina fiscal se
declara **por reporte Z**, no factura por factura: cada Z es un renglón con su
base, su IVA y su exento, más las notas de crédito. Armarlo a mano supone cruzar
los tiques Z con las sesiones de la caja.

**Por qué funciona**: cada venta guardó su **Reporte Z Fiscal** al facturarse
(apartado 2). El libro agrupa las ventas por ese número y por serial de
impresora, y suma. Las ventas sin número fiscal se listan aparte, para que se
vea qué falta por reconciliar.

### Paso a paso (*comprobado en v20 con datos cargados a mano*)

1. **Contabilidad → Informes → Venezuela → Libro de ventas Reportes Z → Nuevo.**
2. **Desde** y **Hasta**: el período. **Descripción**: por ejemplo «Septiembre
   2026, caja principal».
3. **Caja(s)**: elija las cajas; se cargan solas las **Sesiones POS** cerradas
   dentro del período, y con ellas las **Ventas POS**. También puede añadir
   **Diarios** de venta para incluir facturas emitidas desde Contabilidad por la
   misma máquina.
4. Revise los totales calculados: **Total Base Imponible POS**, **Total IVA
   POS**, **Total Exento POS**, y los mismos tres para las notas de crédito
   (**… NC**).
5. La pestaña **Detalle de ventas POS** enseña cada Z con sus facturas.
   **Validar** lo pasa a **Hecho**; **Pasar a borrador** lo reabre. El archivo
   **XLSX** se descarga con el formato del libro.

**Ejemplo.** Un día con dos Z, 445 y 446:

| Z | Serial | Facturas | Base imponible | IVA 16 % | Exento | NC base | NC IVA |
|---|---|---|---:|---:|---:|---:|---:|
| 445 | Z7C7037796 | 00002401–00002409 | 6.250,00 | 1.000,00 | 380,00 | 0,00 | 0,00 |
| 446 | Z7C7037796 | 00002410–00002417 | 8.741,55 | 1.398,65 | 0,00 | 92,04 | 14,73 |
| **Total** | | | **14.991,55** | **2.398,65** | **380,00** | **92,04** | **14,73** |

El renglón del 446 incluye la factura 00002417 del ejemplo: de sus 208,66 Bs,
los 202,58 de mercancía son base 174,64 + IVA 27,94, y los 6,08 de IGTF **no
entran** en el libro de IVA (no son base ni impuesto de IVA; van a su propia
declaración). También incluye la nota de crédito del aceite (106,77 = base
92,04 + IVA 14,73). Ese cuadro, Z por Z, es lo que va al libro de ventas.

## 8. El recibo en PDF

**Para qué**: cuando la venta **no** va por máquina fiscal —una caja en modo
Prueba, una venta interna, un cliente que pide un comprobante para su control—,
la caja puede emitir un **recibo completo en PDF** con la leyenda de que no es
un documento fiscal.

Se activa por caja: **Informe PDF recibo completo** en la ficha de la caja.
Desde **Pedidos**, en la venta, el botón de impresión saca el PDF con las líneas,
los impuestos, los pagos en las dos monedas y la tasa del día. No lleva número
fiscal: si lo llevara, sería la factura.

## 9. Resumen: qué documento sale en cada caso

| Situación | Documento | Quién lo emite |
|---|---|---|
| Venta normal en caja con servidor HK/PNP | **Factura fiscal** | La máquina fiscal, al validar el pago |
| Devolución | **Nota de crédito fiscal**, afectando a la factura | La máquina fiscal, al validar el reembolso |
| Impresora no contestó | Venta **Pendiente de reconciliar** | El administrador completa los datos con el tique |
| Copia de un tique | **Reimpresión** | La máquina fiscal, por número |
| Fin de jornada | **Reporte Z** | La máquina fiscal, al cerrar (automático) o con el botón (manual) |
| Declaración de IVA | **Libro de ventas por Z** (XLSX) | Contabilidad, desde el informe |
| Caja en modo Prueba o comprobante interno | **Recibo PDF** no fiscal | Odoo |
