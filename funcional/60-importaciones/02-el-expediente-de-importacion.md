# El expediente de importación

Es el documento que acompaña a la mercancía desde que se compra hasta que entra
al almacén, y el sitio donde se mira en qué punto va cada embarque.

## El flujo

1. **El proveedor** queda marcado como extranjero.
2. **Se crea la orden de compra** al proveedor extranjero y se confirma.
3. **Se abre el expediente** y se le enlaza la compra.
4. **Avanza por etapas** mientras la mercancía viaja; la bitácora y los
   contenedores se registran por el camino.
5. **Se recibe** la mercancía en el almacén.
6. **Se reparte el costo** del viaje sobre la mercancía —eso es la guía
   siguiente, [El costo de destino](03-el-costo-de-destino.md).

## 1. El proveedor extranjero

El módulo distingue la compra internacional por el **tipo de persona** del
proveedor: solo las órdenes a un proveedor **no domiciliado** ofrecen el campo
del expediente.

### Paso a paso

1. **Contactos → abrir el proveedor** (o **Nuevo**).
2. **Tipo de contacto**: *Compañía*.
3. **Tipo de Persona compañía**: **PJND Persona Jurídica No Domiciliada**. Es lo
   que le dice al sistema que no lleva RIF venezolano y que sus compras son
   importaciones.
4. **País** del proveedor.
5. **Guardar.**

## 2. La orden de compra

![La orden de compra, con su «Nro de Expediente de Importacion»](../img/imp-compra-form.png)

### Paso a paso

1. **Importaciones → Compras → Solicitudes de cotización → Nuevo** (es la misma
   pantalla de Compras, ya filtrada a proveedores extranjeros).
2. **Proveedor**: el del paso 1. Al elegirlo aparece **Nro de Expediente de
   Importacion** en la cabecera. Déjelo vacío por ahora si el expediente aún no
   existe; se enlaza desde el expediente en el paso 3.
3. **Moneda**: la de la compra, normalmente **USD**. La **Tasa** se llena con la
   del día.
4. Pestaña **Productos → Agregar un producto**: la mercancía, cantidad y precio
   en divisa. En cada línea, el **sello del certificado** dice si el producto
   está amparado (verde), por vencer (ámbar), vencido (rojo) o sin certificado
   (gris).
5. Pestaña **Otra información**: **Incoterm**, y cuando la aduana los emita,
   **N° de Planilla de Importación**, **N° de Expediente de Importación**
   (el de la aduana, distinto del folio interno) y su fecha.
6. **Confirmar orden.** El estado pasa a **Orden de compra** y se crea la
   **Recepción** (botón arriba).

## 3. Abrir el expediente

**Importaciones → Información General → Importaciones.**

![La lista de expedientes](../img/imp-expedientes-listado.png)

### Paso a paso

1. **Importaciones → Información General → Importaciones → Nuevo.**
2. **Orden de compra**: elija la orden del paso 2. Solo salen las de
   proveedores extranjeros que no tengan ya expediente. Puede enlazar varias.
3. **Rutas de importación**: la ruta. Al elegirla se rellenan solos **Naviera**,
   **Puerto de Origen**, los **Intermedios**, **Puerto de Destino** y **Días
   libres**: son de solo lectura, los pone la ruta.
4. **Agente aduanal**: solo salen los contactos marcados como agente aduanal.
5. **Almacén**: dónde se recibe la mercancía.
6. Fechas: **Fecha de Embarque**, **Fecha de Despacho**, **Fecha Llegada a
   puerto**, **Fecha de Retorno Vacío**. Se van completando según se sepan.
7. **Incoterm** y **Etiquetas**.
8. **Guardar.** El expediente toma su folio —`IMP-00000001`— y nace con su
   **previsión** y su **costo de destino** enlazados (botones **Pronóstico** y
   **Costos Asociados**, arriba).

![El expediente, con su ruta, su naviera y sus contenedores](../img/imp-expediente-form.png)

> **La naviera y los puertos no se escriben.** Si hay que cambiarlos para un
> embarque concreto, se cambia la ruta o se crea otra.

## 4. Las pestañas: lo que se registra por el camino

### Contenedores

1. Pestaña **Contenedores → Agregar una línea**.
2. **Nro Container**: el número del contenedor (`MSCU-4412239`).
3. **Tipo de contenedor**: del catálogo.
4. El contador **Nro de contenedores** de la cabecera se actualiza solo.

### Bitácora

1. Pestaña **Bitácora → Agregar una línea**.
2. **Fecha**, **Novedad** (el tipo: demora, inspección, cambio de buque…),
   **Descripción**. El **Responsable** se pone solo.
3. Es el diario del expediente: lo que queda por escrito para cuando alguien
   pregunte qué pasó.

### Agente aduanal

Número y fecha del expediente aduanal, y los datos que el agente va entregando.

### Documentos

Hasta diez adjuntos con su descripción: BL, factura comercial, lista de empaque,
certificados, planilla.

## 5. Avanzar por etapas

El expediente recorre sus etapas con el botón **Siguiente etapa**:

```
Nuevo → En Producción → Coordinando Embarque → Navegando → En puerto
      → Recibo → Folio cerrado
```

### Paso a paso

1. Con la mercancía en fabricación: **Siguiente etapa** → **En Producción**.
2. Cuando se coordina el embarque: **Siguiente etapa** → **Coordinando
   Embarque**.
3. Zarpó: **Siguiente etapa** → **Navegando**. **Desde aquí el Agente aduanal
   y el Almacén son obligatorios**: si están vacíos, el sistema no deja
   avanzar.
4. Llegó: **Siguiente etapa** → **En puerto**. Aparece **Enviar por correo**
   para avisar.
5. Salió de aduana y entró al almacén: **Siguiente etapa** → **Recibo**
   (así está rotulado el estado «recibido»).
6. Con el costo repartido y todo cuadrado: **Cerrar importación** → **Folio
   cerrado**. Ya no se edita.
7. **Cancelado** en cualquier momento antes de cerrar. **Borrador** devuelve
   un expediente cerrado o cancelado al inicio, si hace falta corregir.

> **Un expediente solo se borra en «Nuevo».** En cuanto avanza —o en cuanto
> tiene un pedido confirmado o un albarán hecho— el sistema se niega: se
> archiva, no se borra. Es deliberado: el expediente es la trazabilidad de una
> compra internacional.

## 6. Recibir la mercancía

### Paso a paso

1. En el expediente, botón **Recibo** (arriba; también desde la orden de
   compra, **Recepción**, o en **Inventario → Recepciones**).
2. Abra el albarán. En cada línea, la **cantidad** recibida.
3. **Validar.** El albarán pasa a **Hecho**, la mercancía entra al almacén y el
   albarán aparece en el campo **Albaranes** del expediente.
4. Si el producto es **almacenable** (casilla **Rastrear inventario** en su
   ficha), la recepción queda valorada y el reparto del costo tendrá efecto
   contable. Si no lo es, ver el aviso de la guía siguiente.

Con la mercancía recibida, el camino sigue en
[El costo de destino](03-el-costo-de-destino.md).
