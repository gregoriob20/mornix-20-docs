# Estado de la migracion

Inventario de modulos del cliente y su avance hacia v20. Se llena cuando tengamos
acceso a los repos origen.

## Leyenda de estado

| Estado | Significado |
|---|---|
| `pendiente` | Sin tocar |
| `en curso` | Migracion en progreso |
| `instala` | Instala en v20 sin error, sin probar funcionalmente |
| `probado` | Probado contra el flujo real del cliente |
| `descartado` | No se migra (obsoleto o cubierto por el core de v20) |

## Inventario

Detalle modulo por modulo en [INVENTARIO.md](INVENTARIO.md). Resumen:

**226 modulos, 282.491 lineas** (py + xml + js) en 7 repositorios.

| Origen | Modulos | Lineas | Que implica |
|---|---:|---:|---|
| Cliente (Nimetrix / Oasis / Mornix) | 157 | 182.586 | Migracion a mano. Es el trabajo real. |
| Terceros de pago | 50 | 85.205 | **No se migran**: hay que comprar/pedir la version v20 al vendor. |
| OCA | 12 | 8.258 | Se toman del upstream cuando exista rama `20.0`. |
| Sin autor declarado | 7 | 6.442 | Clasificar a mano. |

Repos de origen:

| Version | Repo | Modulos |
|---|---|---:|
| v16 | `nimetrix/l10n_ve_odoo_16` (rama `PRE-PROD`) | 119 |
| v16 | `nimetrix/l10n_ve_sucursales` | 21 |
| v18 | `nx-desarrollo/nx_tools` | 39 |
| v18 | `nx-desarrollo/nx_localizacion` | 18 |
| v18 | `nx-desarrollo/nx_point_of_sale` | 12 |
| v18 | `nx-desarrollo/nx_dual_currency` | 9 |
| v18 | `nx-desarrollo/nx_sucursales` | 8 |

**19 modulos existen en v16 y v18 a la vez.** Para esos se parte de la version
v18, que ya trae dos saltos de version resueltos:

`bi_branch_budget_ent`, `bi_branch_pos`, `bi_branch_scrap_order`,
`bi_odoo_mrp_multi_branch`, `bi_odoo_multi_branch_hr`, `branch`,
`branch_analytic_account`, `fiscal_lock_days`, `l10n_ve_dpt`, `l10n_ve_stock`,
`l10n_ve_stock_account`, `nimetrix_detailed_sale_report`,
`nimetrix_general_sale_report`, `nimetrix_report_arc`, `nimetrix_restrictions`,
`nimetrix_standard_report_invoice`, `stock_no_negative`, `unidades_permitidas`,
`unidades_presentacion`.

## Consolidacion: v16 y v18 son del mismo cliente

Los repos `nimetrix/` (v16) y `nx-desarrollo/` (v18) pertenecen a la misma
empresa (Nimetrix, hoy Mornix). El destino es **una sola base de codigo v20**,
no dos.

Lo que eso obliga a resolver:

- **La localizacion venezolana existe en dos generaciones**: `l10n_ve_full` (v16,
  21.580 lineas) y `l10n_ve_nimetrix` (v18, 14.788 lineas). Son la misma
  funcionalidad reescrita. Se toma la linea v18 como base.
  - [ ] Verificar que funcionalidad de `l10n_ve_full` **no** quedo en
        `l10n_ve_nimetrix`. Si algo se perdio en el camino v16 -> v18 y el
        cliente todavia lo usa, hay que reponerlo, no asumir que sobraba.
- **Los 19 modulos duplicados** se unifican en uno solo, partiendo de la
  version v18.
- **Sucursales tambien esta duplicado**: `l10n_ve_sucursales` (v16, 21 modulos)
  contra `nx_sucursales` (v18, 8 modulos). La v16 tiene 13 modulos que no
  existen en v18.
  - [ ] Definir cuales de esos 13 siguen en uso.

## Dependencias externas: el riesgo mayor del proyecto

85.205 lineas (30% del total) son modulos de terceros. **No dependen de nosotros**
y ninguno tendra version v20 antes de que Odoo libere v20:

| Vendor | Modulos | Lineas |
|---|---:|---:|
| INM & LDR Soluciones | 2 | 18.998 |
| Emipro Technologies | 2 | 16.877 |
| BrowseInfo | 23 | 15.760 |
| Terabits Technolab | 2 | 9.367 |
| Teqstars | 1 | 6.633 |
| binaural-dev | 2 | 5.352 |
| TechKhedut | 1 | 3.614 |
| resto (15 vendors) | 17 | 8.604 |

Lo mismo aplica a los 12 modulos OCA: los repos de OCA no abren rama `20.0`
hasta despues del release oficial.

- [ ] Decidir por cada vendor: esperar su v20, portarlo nosotros (revisar licencia),
      o reemplazar la funcionalidad.

## Orden de trabajo sugerido

1. **Clasificar los 7 modulos sin autor** y confirmar la lista de terceros.
2. **Modulos base del cliente** primero (los que otros heredan), porque su API
   condiciona al resto: la localizacion venezolana es la raiz de casi todo.
3. **Un modulo piloto de tamano medio** antes de estimar el resto, para medir
   cuanto duele realmente el salto con codigo de este cliente en la mano.
4. **Reportes y frontend al final**: son los mas afectados por OWL y por el
   cambio de motor PDF.

## Puntos de atencion conocidos v16 -> v20

Los saltos v16->v17->v18->v19->v20 acumulan cuatro releases. Lo que suele romper:

- **`attrs` y `states` en vistas**: eliminados desde v17. Se reemplazan por
  atributos directos (`invisible`, `readonly`, `required`) con expresion Python.
- **Bundles de assets**: se declaran en el manifest (`assets`), ya no en XML.
- **OWL**: los widgets viejos de JS ya no existen.
- **Vistas `tree` renombradas a `list`** en v18.
- **`name_get()` reemplazado por `_compute_display_name`** en v17.

Cada una se confirma y se anota en [BREAKING-CHANGES.md](BREAKING-CHANGES.md) a
medida que la encontremos en codigo real; la lista de arriba es lo esperado, no
lo verificado.
