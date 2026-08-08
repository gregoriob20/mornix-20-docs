# Qué no deja hacer el sistema

Restricciones puestas a propósito. Conviene conocerlas para no perder tiempo
buscando un fallo donde hay una decisión.

## Documentos contables

| No se puede | Motivo |
|---|---|
| Borrar un asiento que no esté en borrador | La contabilidad no se borra: se anula |
| Cancelar una factura de otro período | Cerrado el mes, cambiarlo descuadra lo ya declarado |
| Modificar un documento declarado | Ver [Declaraciones y bloqueo](08-declaraciones-y-bloqueo.md) |

Para cancelar un documento ya sellado con hash inalterable hay un botón
**Cancelar** propio, que pide confirmación. Solo aparece dentro del período en
curso.

![Así se ve un documento declarado: aviso arriba y cinta en la esquina. Los campos no se pueden editar](img/comprobante-iva-bloqueado.png)

## Configuración

| No se puede | Motivo |
|---|---|
| Instalar Studio | La localización se mantiene desde el repositorio, no desde la interfaz |
| Editar vistas desde la interfaz | Lo mismo: un cambio hecho ahí no queda en el repositorio y se pierde en la siguiente actualización |

**Instalar y actualizar módulos sí funciona con normalidad.** La restricción
solo afecta a editar vistas a mano.

## Datos obligatorios

| Campo | Dónde | Si falta |
|---|---|---|
| RIF | Compañía y contactos | No se puede emitir ni declarar |
| Tipo de persona | Contactos | El ISLR se calcula con la tarifa equivocada |
| Motivo del traslado | Traslados internos | No se puede guardar el albarán |
| Fecha de factura | Facturas | Obligatoria |

## Precios

No se admiten precios en cero en los pedidos de venta. Si hace falta entregar
algo sin cobro, se usa el motivo de traslado **donación**, que es lo que la
normativa espera.
