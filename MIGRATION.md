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
| v18 | `mornix-tech/nx-import` | 1 |
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

## La nomina va aparte

`mornix-tech/nx-nomina` (v18, 55 modulos) **no entra en el inventario de arriba**:
se migra en su propio repositorio, `gregoriob20/mornix_nomina_20`, y Odoo lo lee
desde `addons/nomina/`.

| | Modulos |
|---|---|
| Migrados e instalando | **35** |
| Motor de nomina (OCA adaptado) | 1 |
| Bloqueados por Enterprise o terceros | 20 |
| **Total del repositorio de origen** | **55** |

Los 35 se instalan de una vez sobre una base vacia y las **102 pruebas pasan**.

Se separo por dos razones, las dos de fondo:

- **El motor no es nuestro.** 42 de los 55 modulos dependian de `hr_payroll`, que
  es de Enterprise. Se sustituyo por `OCA/payroll` adaptado a v20, que tiene su
  propio ciclo de vida: cuando OCA saque rama 20.0 habra que rebasar los nueve
  cambios que le hicimos.
- **Un bloqueo que no depende de nosotros.** `res.bank` desaparecio de v20 y
  arrastra 9 modulos de banca y pagos. La decision esta aplazada hasta ver que
  hace Odoo 20 cuando salga.

Detalle en [modulos/nomina.md](modulos/nomina.md); lista de bloqueados, en
`BLOQUEADOS.txt` del repositorio de la nomina.

## El TPV y las importaciones, tambien en su propio repositorio

Mismo patron que la nomina: repositorio propio, y Odoo lo lee desde su carpeta
de `addons/`, que no se versiona en este repositorio.

| Area | Origen (v18) | Destino | Punto de montaje | Estado |
|---|---|---|---|---|
| Punto de venta | `mornix-tech/nx_point_of_sale` (12 modulos) | `gregoriob20/mornix_pos_20` | `addons/point_of_sale/` | Instala, el TPV abre y se arma una venta; **la pantalla de pago no funciona** |
| Importaciones | `mornix-tech/nx-import` (1 modulo) | `gregoriob20/mornix_importaciones` | `addons/importaciones/` | Instala, las 18 pantallas abren, 13 pruebas y **el camino completo cuadrado**: compra, recepcion y reparto del costo |

En importaciones, lo que habria bloqueado la instalacion entera estaba en una
sola linea del manifest: `stock_enterprise`, declarado y **sin usar**. Se
comprobo antes de quitarlo.

Lo que quedo abierto ahi fue una dependencia que nadie habia declarado:
`create_landed_cost()` escribia dos campos —`currency_price_unit` y
`nx_rate_ref`— que ponia `nimetrix_stock_cost_usd`, del repositorio de doble
moneda, escrito sobre `stock.valuation.layer`, el modelo que v20 elimino.
**Cerrado el 16 de septiembre de 2026**: el costo en divisa se reescribio
dentro de `l10n_ve_mornix` (1.40.0) sobre la valoracion de v20, con los mismos
nombres de campo (ver la seccion siguiente).

Detalle en [modulos/tpv.md](modulos/tpv.md) y
[modulos/importaciones.md](modulos/importaciones.md).

## La doble moneda: que queda de `nx_dual_currency`

Revisado el 16 de septiembre de 2026 contra `mornix-tech/nx_dual_currency`
(`main`, 79748b0; 10 modulos, 12.400 lineas de Python). Lo que hay en v20 y lo
que falta, modulo por modulo:

| Modulo v18 | Lineas | En v20 | Estado |
|---|---:|---|---|
| `nimetrix_dual_currency` | 4.397 | `mornix_dual_currency` 1.8.0 | **Migrado**; mismos 11 modelos y los mismos campos (v20 añade `nx_diferencial_ref`, `nx_balance_ref_forzado`, `nx_full_reconcile_ref_id`). Faltan por revisar los ultimos commits de v18: totales USD en las listas de facturas, `nx_costo_unit_bs` para cargar en Bs sin borrar el precio, propagacion de `nx_rate_custome` del pago al asiento |
| `nimetrix_currency_rate` (repo `nx_localizacion`) | — | `mornix_currency_rate` 0.8.0 | **Migrado** |
| `nimetrix_stock_cost_usd` | 1.741 | `l10n_ve_mornix` 1.40.0, `nx_cost_*` | **Reescrito** sobre `stock.move.value`: valor $ y tasa por movimiento, coste promedio $ por producto, costos en destino en dos monedas, `Costo $` sincronizado. Fuera: fabricacion, subcontratacion, ajuste $ al facturar (ver la ficha, §6.34) |
| `nimetrix_cost_usd_property` | 241 | — | **No hace falta**: en v20 `standard_price_usd` ya es un campo normal por compañia |
| `nimetrix_list_price_property` | 501 | `l10n_ve_mornix` (`list_price_usd`, `list_price_vat`, `list_price_vat_usd`) | **Absorbido** desde el piloto |
| `nimetrix_dual_pricelist` | 118 | `mornix_dual_currency` (bases `standard_price_usd`, `list_price_usd`, `nx_last_cost_usd`) | **Absorbido**; queda por comprobar `nx_price_fixed_usd` (precio fijo en $ por regla) y la actualizacion masiva de tarifas al cambiar la tasa (`mornix_currency_rate.update_product_pricelist`) |
| `nimetrix_igtf_dual` | 34 | — | **Pendiente, trivial**: 34 lineas que dependen de `nimetrix_igtf`; va con el IGTF del TPV |
| `mornix_iva_islr_edit_total_bs_fix` | 905 | — | **Por revisar**: corrige retenciones IVA/ISLR de facturas en USD con «Editar total Bs»; la retencion en v20 se reescribio (ficha de la localizacion §6.4) y hay que ver si el caso sigue existiendo |
| `nimetrix_account_reports_dual` | 2.446 | — | **Bloqueado por Enterprise**: extiende `account_reports`. El libro diario en divisa ya se hizo sin Enterprise (§6.32); los demas informes (balance, mayor, antigüedad, flujo de caja) en divisa necesitarian el mismo camino |
| `nimetrix_customer_statement` | 696 | — | **Bloqueado por Enterprise** (`account_reports`). Estado de cuenta del cliente en divisa: se puede rehacer como informe QWeb propio |
| `nimetrix_mrp_dual_currency` | 1.335 | — | **Fuera de alcance**: fabricacion (`mrp_account`, `mrp_workorder` Enterprise). Se retoma si el cliente fabrica |

**Consecuencia:** de las diez piezas, cinco estan cubiertas, una no hace
falta, una es trivial, una hay que revisar y dos dependen de Enterprise o de
MRP. El costo en divisa —la que bloqueaba importaciones— ya esta.

## La base del cliente: `auromin`

Creada el 15 de septiembre de 2026 con todo lo migrado, en tres pasos que se
repiten con dos scripts: `base` solo → `configurar_auromin.py` (compañía en
Venezuela, bolívares, es_VE) → los 53 módulos de una vez →
`configurar_auromin_post.py` (moneda de referencia, tasa del BCV, caja y
métodos de pago, permisos del administrador).

| | |
|---|---|
| Módulos de la casa instalados | **54** (53 + `l10n_ve`), 152 en total |
| Plan contable | venezolano (`ve`), 276 cuentas, 8 diarios más el de efectivo |
| Idioma | es_VE activo y por defecto; zona horaria America/Caracas |
| Monedas | VES de la compañía; USD activa, sincronizada con el BCV, tasa del día cargada (842,21 Bs) |
| Caja | «Caja principal» con doble moneda y tres métodos: Efectivo Bs, VPOS, SITEF |
| Pantallas comprobadas en navegador | 11 de 11 (nómina, TPV, importaciones, contabilidad, productos) |
| Acceso | https://auromin.migracion.mornix.tech (el certificado se emite al primer acceso) |

Lo que **no** se inventó y queda para el cliente: RIF y dirección de la
compañía, la contraseña del administrador (sigue la de fábrica: **cambiarla**),
los usuarios, los productos, las cuentas bancarias y la estructura de nómina.

Dos cosas salieron de crearla y ya están corregidas: la guarda de impuestos
de importaciones tumbaba la instalación en una base nueva (ver su ficha), y el
plan venezolano no trae diario de efectivo, así que el script lo crea.

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

0. ~~Piloto~~ — hecho: `l10n_ve_mornix` migrado, 189 pruebas en verde.
   Ver [modulos/l10n_ve_mornix.md](modulos/l10n_ve_mornix.md).

   Siete modulos sueltos quedaron **absorbidos** dentro de el, porque no eran
   funcionalidad aparte sino piezas de la misma localizacion:

   | Modulo origen | Repo | Version | Que aporto |
   |---|---|---|---|
   | `mornix_dual_currency` (parcial) | `nx_dual_currency` | 1.26.0 | Saldos por cobrar/pagar en divisa |
   | `nimetrix_stock_account_report` | `nx_localizacion` | 1.27.0 | Libro de inventario |
   | `nimetrix_iva_resumen_report` | `nx_tools` | 1.28.0 | Resumen de Ventas y Compras completo |
   | `nimetrix_retencion_municipal` | `nx_localizacion` | 1.29.0 | Retencion municipal |
   | `nimetrix_restrictions` | `nx_localizacion` | 1.30.0 | Restricciones de borrado, cancelacion y Studio |
   | `nimetrix_report_arc` | `nx_localizacion` | 1.30.0 | Comprobante anual de retenciones ISLR |
   | `l10n_ve_stock_account` | `nx_localizacion` | 1.31.0 | Guias de despacho y facturacion desde inventario |

   Ninguno se instala ya por separado. Al absorberlos se rompio ademas la
   dependencia circular `l10n_ve_mornix -> mornix_dual_currency ->
   mornix_currency_rate -> l10n_ve_mornix`.
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
