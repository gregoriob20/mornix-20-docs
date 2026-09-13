# Importaciones

El expediente de una importación de punta a punta: la ruta y su naviera, los
contenedores, los certificados de la mercancía, los aranceles y los gastos que
acaban sumándose al costo.

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
