# Libro de inventario

Existencias y valoración por producto en un período, como lo pide la normativa.

## Generarlo

En **Contabilidad → Informes de Venezuela → Libro de Inventario**:

| Campo | Qué poner |
|---|---|
| Fecha desde / hasta | El período a listar |
| Compañía | La que corresponda |
| Filtrar por | Todos los productos, un producto concreto o una categoría |

Sale un PDF con, por cada producto: existencia inicial y su costo, entradas del
período, salidas del período y existencia final, todo valorado.

## Qué entra y qué no

**Solo los productos con movimiento en el período.** Un producto con existencia
pero sin entradas ni salidas en esas fechas no aparece en el listado.

- [ ] Pendiente de confirmar con el cliente si el libro debe incluir también
      los productos que no se movieron pero tienen existencia. Cambiar eso es
      un ajuste pequeño; lo que hace falta es la decisión.

La existencia inicial se calcula con todos los movimientos **anteriores** a la
fecha desde, no con una foto guardada: si se corrige un movimiento antiguo, el
libro de un período posterior cambia en consecuencia.

## Sobre la valoración

El libro lee la valoración que Odoo lleva en cada movimiento de inventario. Eso
significa que depende de cómo esté configurada la valoración de cada categoría
de producto (estándar, precio medio o FIFO). Si una categoría no tiene
valoración automática, sus productos saldrán en cero.
