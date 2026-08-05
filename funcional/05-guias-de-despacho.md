# Guías de despacho

El documento que acompaña la mercancía cuando sale del almacén sin factura.

## Cómo se decide que un envío lleva guía

De dos maneras:

- **Desde el pedido de venta**: el campo **Documento** puede valer «Factura» o
  «Guía de despacho». El valor por defecto sale del contacto, del campo
  «Documento por defecto» en la pestaña Ventas y Compras.
- **Desde el albarán**, marcando **Es guía de despacho**. Es lo habitual en los
  traslados internos.

## Cuándo se numera

**Al confirmar el pedido de venta.** El albarán sale ya con su número de guía,
antes de validarse, que es cuando se necesita para que acompañe la mercancía.

En los traslados internos marcados a mano, que no vienen de un pedido, el
número se asigna al validar el albarán.

> En un almacén que entregue en varios pasos, el número lo lleva **solo el
> albarán que sale del almacén**, no los pasos intermedios de preparación y
> empaquetado.

## Motivo del traslado

Los traslados internos exigen indicar el motivo: venta, donación, consignación,
exportación o autoconsumo. El motivo condiciona qué se puede hacer después con
ese albarán.

## Imprimir la guía

Con el albarán validado aparece el botón **Guía de despacho**, que saca el PDF.

## Facturar desde el albarán

Un albarán despachado con guía se factura después. Los botones del albarán
crean la factura, la factura de proveedor o la nota de crédito según el tipo de
operación. También se pueden facturar **varios albaranes a la vez** desde la
lista.

La columna **Estado de la guía** indica si el albarán ya se facturó o sigue
pendiente.
