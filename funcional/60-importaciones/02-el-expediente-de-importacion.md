# El expediente de importación

Es el documento que acompaña a la mercancía desde que se compra hasta que entra
al almacén, y el sitio donde se mira en qué punto va cada embarque.

## 1. Abrir el expediente

**Importaciones → Información General → Importaciones.**

![La lista de expedientes](../img/imp-expedientes-listado.png)

**Nuevo** abre un expediente con folio propio —`IMP-00000001`, `IMP-00000002`…—
que asigna el sistema. El «Nuevo folio» que se ve mientras se llena es solo el
marcador de pantalla: el folio real aparece al guardar.

![El expediente, con su ruta, su naviera y sus contenedores](../img/imp-expediente-form.png)

Lo que hay que llenar:

| Campo | Notas |
|---|---|
| **Orden de compra** | Obligatorio: el expediente existe por una compra |
| **Agente aduanal** | Solo salen los contactos marcados como agente aduanal |
| **Rutas de importación** | Al elegirla, propone naviera, puertos y días libres |
| **Almacén** | Dónde se recibe la mercancía |
| **Fechas** | Embarque, despacho, llegada a puerto y retorno del vacío |
| **Etiquetas** | Para agrupar y filtrar («urgente», «perecedero») |

> **La naviera y los puertos no se escriben.** Son de solo lectura y los pone la
> ruta. Si hay que cambiarlos para un embarque concreto, se cambia la ruta o se
> crea otra.

## 2. Las pestañas

- **Bitácora** — el diario del expediente: fecha, novedad, descripción y quién
  la registró. Es donde queda por escrito la demora, la inspección o el cambio
  de buque.
- **Contenedores** — uno por línea, con su número y su tipo del catálogo. El
  contador del encabezado los cuenta solo.
- **Agente aduanal** — número y fecha del expediente aduanal.
- **Documentos** — hasta diez adjuntos con su descripción.

## 3. Cómo avanza

El expediente recorre sus etapas con el botón **Siguiente etapa**:

```
Nuevo → En Producción → Coordinando Embarque → Navegando → En puerto
      → Recibido → Folio cerrado
```

y **Cancelado** para el que no llega a puerto.

> **Un expediente solo se borra en «Nuevo».** En cuanto avanza —o en cuanto
> tiene un pedido confirmado o un albarán hecho— el sistema se niega: se
> archiva, no se borra. Es deliberado: el expediente es la trazabilidad de una
> compra internacional.

## 4. Los costos de destino

Cada expediente nace con su **costo de destino** asociado, que es donde se
juntan los gastos del viaje —flete, seguro, aduana, almacenaje, vigilancia— para
repartirlos después sobre el costo de la mercancía.

![La ficha de costos de destino](../img/imp-costo-destino-form.png)

Desde ahí se generan los **costos adicionales** de Odoo (`stock.landed.cost`),
que son los que suben el costo del producto recibido.

> **Esta parte no está comprobada en Odoo 20.** La pantalla abre y los gastos se
> registran, pero el reparto sobre la mercancía no se ha ejercitado todavía con
> una recepción real. Hasta que se compruebe, conviene revisar a mano el costo
> resultante del producto.

> **Hace falta el permiso «Inventario / Administrador»** para abrir esta ficha:
> enseña los costos adicionales, y ese modelo no lo lee un usuario de inventario
> corriente. Con permisos de usuario la pantalla se queda cargando sin decir por
> qué.
