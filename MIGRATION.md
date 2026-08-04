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

## Acoplamiento que los manifests no declaran

Medido con `scripts/dependencias_no_declaradas.py` sobre los 86 módulos de v18:
**42 campos usados sin declarar la dependencia, en 9 módulos.**

El caso que lo destapó: `l10n_ve_nimetrix` genera su libro fiscal con SQL que
referencia `account_move.amount_exempt_bs`, campo que define
`nimetrix_dual_currency`. El módulo instala limpio y falla recién al generar el
reporte.

Los módulos de los que más se depende sin declararlo:

| Proveedor oculto | Módulos que lo necesitan |
|---|---:|
| `nimetrix_dual_currency` | 7 |
| `nx_pos_dual_currency` | 6 |
| `nimetrix_currency_rate` | 5 |
| `l10n_ve_nimetrix` | 4 |
| `nimetrix_stock_cost_usd` | 2 |
| `nimetrix_report_base` | 2 |
| `nimetrix_cost_usd_property` | 2 |
| resto (7 módulos) | 1 c/u |

**Consecuencia para el orden de trabajo:** la doble moneda no es un área
paralela a la localización, es su cimiento. `nimetrix_dual_currency`,
`nimetrix_currency_rate` y `nx_pos_dual_currency` hay que migrarlos **antes o a
la par** de lo demás, no después.

Cómo verificarlo en cualquier momento:

```bash
python3 scripts/dependencias_no_declaradas.py /opt/odoo-client/v18
python3 scripts/dependencias_no_declaradas.py /opt/odoo-client/v18 --modulo <modulo> --verbose
```

Límites del análisis, para no leer de más: solo detecta **campos** (no métodos
ni modelos), y solo los que siguen las convenciones del cliente — prefijo `nx_`
o sufijo `_bs`/`_usd`. El acoplamiento real puede ser mayor.

- [ ] Declarar estas dependencias en los manifests a medida que se migra cada
      módulo. Un manifest correcto es lo que hace que Odoo instale en el orden
      correcto solo.

## Orden de trabajo sugerido

0. ~~Piloto~~ — hecho: `l10n_ve_mornix` migrado, 113 pruebas en verde.
   Ver [modulos/l10n_ve_mornix.md](modulos/l10n_ve_mornix.md).
1. **Doble moneda primero.** El analisis de acoplamiento lo puso arriba de la
   lista: `nimetrix_dual_currency`, `nimetrix_currency_rate` y
   `nx_pos_dual_currency` son la base oculta de la que cuelgan 7, 5 y 6 modulos
   respectivamente, incluido el piloto ya migrado.
2. **Clasificar los 7 modulos sin autor** y confirmar la lista de terceros.
3. **Resto de la localizacion**, que ya tiene su raiz migrada.
4. **Reportes y frontend al final**: son los mas afectados por OWL y por el
   cambio de motor PDF.

> **Aviso para quien migre `nimetrix_iva_resumen_report`**: su
> `prior_period_dates` leia `res.partner.nx_contribuyente_seniat`, campo que se
> elimino de `l10n_ve_mornix` en la version 1.5.0 por decision del cliente. Con
> el valor 'especial' ese modulo calcula el periodo QUINCENAL del libro resumen
> de IVA; sin el, calculara siempre el mensual. Hay que decidir de donde sale el
> dato antes de darlo por migrado.

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
