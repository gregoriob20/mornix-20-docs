# Inventario

Lo que la localización añade al inventario de Odoo: la **guía de despacho** que
acompaña a la mercancía, con su numeración y su formato, y el **costo en
divisa**, que dice cuánto costó cada unidad en dólares sin reconvertir
bolívares viejos a la tasa de hoy.

| Realidad del inventario venezolano | Qué le añade la localización |
|---|---|
| La mercancía viaja con un documento propio, con numeración de control y datos fiscales, no con el albarán de Odoo | La **guía de despacho**, con su secuencia, su formato y su facturación desde inventario |
| El costo en bolívares envejece en semanas y las decisiones se toman en dólares | Cada movimiento guarda su **valor en divisa** y su **tasa**; el producto tiene su **coste promedio $** derivado de la valoración |
| Los gastos de importación llegan en facturas a tasas distintas | Los **costos en destino** se reparten en las dos monedas, cada gasto con su tasa |

## Las guías

| Guía | Qué contesta |
|---|---|
| [Guías de despacho](01-guias-de-despacho.md) | Cómo se emite, numera e imprime la guía que acompaña la mercancía |
| [El costo en divisa](02-el-costo-en-divisa.md) | De dónde sale el valor $ de cada movimiento, el coste promedio $ del producto y los costos en destino en dos monedas |
