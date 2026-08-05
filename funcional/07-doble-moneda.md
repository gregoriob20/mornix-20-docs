# Doble moneda

Cómo conviven el bolívar y la divisa en las facturas y en los saldos.

## La moneda referencial

Se configura una vez por compañía. Es la divisa en la que se llevan los importes
en paralelo al bolívar, normalmente el dólar.

## En la factura

Cada factura muestra su desglose de IVA por alícuota **siempre en bolívares**,
sea cual sea la moneda en la que se emitió. Junto a cada base va su impuesto, y
al final el total del documento.

La tasa aplicada se guarda **en la propia factura**: es la que se usó al
asentarla, y no cambia aunque la tasa del día cambie después. Todo lo que se
calcule sobre esa factura —retenciones, libros, reportes— usa esa misma tasa.

## Saldos del contacto en divisa

En la ficha del contacto, dos botones muestran el **total por cobrar** y el
**total por pagar** convertidos a la moneda referencial. Al pulsarlos se abren
los documentos que siguen abiertos.

El cálculo va apunte por apunte: los que ya están en la divisa se suman
directamente, y los demás se convierten con la tasa de su fecha. No es una
conversión del total al cambio de hoy.
