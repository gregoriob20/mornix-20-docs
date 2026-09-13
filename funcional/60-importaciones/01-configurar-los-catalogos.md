# Cómo se configuran los catálogos

Un expediente de importación se apoya en catálogos que se dejan puestos **una
vez**: los puertos, la naviera, la ruta, el almacén, los aranceles y los
certificados. Todo vive en **Importaciones → Configuración**.

## El orden importa

Cada pieza necesita la anterior:

1. **Puertos** — de origen y de destino.
2. **Navieras** — cada una con su contacto.
3. **Rutas** — unen puertos y naviera, y llevan los gastos del viaje.
4. **Almacenes**, **aranceles**, **certificados** y **etiquetas** — se añaden
   cuando hagan falta.

## 1. Puertos

![Los puertos, con su referencia y su país](../img/imp-puertos-listado.png)

La lista se edita en línea: **Nuevo**, se escribe y se guarda sin abrir ficha.
La **referencia** es única por compañía —conviene el código del puerto, como
`VELAG` o `CNSHA`—.

## 2. Navieras

Cada naviera se apoya en un **contacto** de Odoo: si la naviera ya está en la
agenda, se enlaza; si no, se crea primero el contacto.

## 3. Rutas

**Importaciones → Configuración → Rutas de importación.**

![La ficha de una ruta, con sus puertos y sus gastos asociados](../img/imp-ruta-form.png)

La ruta es la que sabe por dónde va la mercancía y cuánto cuesta el viaje:

| Campo | Para qué sirve |
|---|---|
| **Puerto de origen / destino** | Los extremos del viaje |
| **Intermedio 1, 2 y 3** | Escalas de la naviera, si las hay |
| **Naviera** | Quién transporta |
| **Días libres** | Los que la naviera concede antes de cobrar almacenaje |
| **Tránsito estimado** | Días que se espera que tarde |
| **Concepto de flete** | El producto de servicio con el que se factura el flete |

> **La ruta no se guarda sin su línea de flete.** En la pestaña **Gastos
> asociados** tiene que haber una línea con el mismo producto que el **concepto
> de flete**; Odoo la propone al elegirlo, y si se borra, la ruta se niega a
> guardar con *«There must be at least one line related to the freight concept»*.

> El producto del flete tiene que ser **de tipo servicio**, con **«Puede ser
> costo en destino»** marcado y **tipo de concepto = Flete marítimo**. Si no
> aparece en la lista desplegable, es porque le falta alguna de las tres.

## 4. Aranceles

![Las partidas arancelarias, con su tasa](../img/imp-aranceles-listado.png)

Una partida por producto importado, con su **tasa** —entre 0 y 100; el sistema
no deja guardar otra cosa— y su descripción. La partida se enlaza luego en la
ficha del producto, pestaña de importaciones.

## 5. Contenedores

![El catálogo de contenedores, con sus medidas y su carga máxima](../img/imp-contenedores-listado.png)

Vienen **precargados** —dry de 20 y 40 pies, y los demás tipos habituales— con
sus medidas interiores, su capacidad cúbica, su tara y su carga máxima. La
instalación los copia a cada compañía de la base.

## 6. Certificados

![Los certificados, con su vigencia y su estado](../img/imp-certificados-listado.png)

Cada certificado tiene un **tipo**, una **vigencia** (desde / hasta), el
contacto que lo gestiona, el PDF y los **productos** que ampara.

El estado lo calcula el sistema a partir de la fecha de fin:

| Estado | Cuándo |
|---|---|
| **Vigente** | Faltan más de 30 días |
| **Por vencer** | Faltan 30 días o menos |
| **Vencido** | Ya pasó la fecha |

> **El aviso llega donde se compra.** En la línea del pedido de compra aparece
> un sello de color por producto: verde si el certificado está vigente, ámbar si
> está por vencer, rojo si venció y gris si el producto no tiene certificado.
> Es la razón de ser de este catálogo.

> Un cron recalcula los estados cada día. Sin él, un certificado vence y el
> sello sigue verde hasta que alguien abra la ficha.
