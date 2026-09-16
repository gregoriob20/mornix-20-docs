# Importaciones

El expediente de una importación de punta a punta: la ruta y su naviera, los
contenedores, los certificados de la mercancía, los aranceles y los gastos que
acaban sumándose al costo.

## Por qué un módulo de importaciones y no solo Compras

Una compra nacional termina cuando llega la factura y la mercancía. Una
importación tiene un mes de vida entre las dos, y lo que pasa en ese mes decide
cuánto costó de verdad cada unidad:

| Realidad de la importación | Qué le añade el módulo |
|---|---|
| La mercancía viaja semanas y varios departamentos preguntan «¿dónde va?» | El **expediente** con sus etapas (en producción, navegando, en puerto…), su bitácora y sus contenedores |
| El precio de compra es la menor parte del costo: el flete, el seguro, la aduana y el almacenaje pueden sumar otro 50 % o más | El **costo de destino**: recoge cada gasto desde su factura y lo reparte sobre la mercancía hasta el costo unitario «en almacén» |
| Los gastos llegan en facturas distintas, de proveedores distintos, en fechas distintas | Cada factura se marca con su **expediente** y el gasto entra solo al reparto |
| Un contenedor devuelto tarde cuesta demora; un certificado vencido detiene la carga en aduana | **Rutas** con días libres y tarifas, **certificados** con vigencia y aviso en la compra |
| Se compra en dólares y se contabiliza en bolívares | Todo el cálculo en las dos monedas, con la tasa de cada documento |

Estas guías explican cada pieza en tres tiempos —**para qué sirve**, **por qué
está así** y **cómo se hace**— con la importación de la base de pruebas como
hilo: **100 unidades a 18 $** desde Shanghái a La Guaira, flete de 2.500 $ y
gastos de aduana de 900 $, a **36,50 Bs por dólar**.

## Las guías

| Guía | Qué contesta |
|---|---|
| [Cómo se configuran los catálogos](01-configurar-los-catalogos.md) | Puertos, navieras, servicios del viaje, rutas, almacenes, aranceles, contenedores, certificados |
| [El expediente de importación](02-el-expediente-de-importacion.md) | El proveedor extranjero, la compra, el expediente y sus etapas, la recepción |
| [El costo de destino](03-el-costo-de-destino.md) | Cómo entran los gastos, cómo se calcula el costo unitario, cómo se reparte y qué asiento deja |

> **Recién migrado a Odoo 20, y ya con el camino completo comprobado:** una
> compra en divisa entra al almacén y los gastos del viaje acaban sumados al
> costo de la mercancía, con su asiento. Lo que todavía no está es el detalle
> **en divisa** del reparto —depende de un módulo que no se ha migrado—; el
> reparto en bolívares sí funciona. El estado exacto, en la ficha técnica
> [Importaciones](../../modulos/importaciones.md).

Antes de empezar, cuatro cosas tienen que estar puestas en la base:

| Requisito | Si falta |
|---|---|
| **País** en la compañía | El módulo no se deja instalar |
| **Moneda activa y sincronizada** | La lista de expedientes no abre |
| **Idioma español activo** | Las pantallas del módulo salen en inglés |
| **«Inventario / Administrador»** para quien lleve costos | La ficha de costos de destino se queda cargando |
