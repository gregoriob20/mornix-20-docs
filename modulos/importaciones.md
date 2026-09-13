# Importaciones — un módulo, 29 modelos y 12 pruebas nuevas

> Estado: **instala, las 18 pantallas abren y las 12 pruebas pasan**. Lo que no
> está probado es el negocio: ningún expediente ha recorrido todavía el camino
> completo de compra → recepción → reparto del costo.
> Origen: `mornix-tech/nx-import`, rama `main`, commit `3857102`.
> Destino: `gregoriob20/mornix_importaciones`, versión `1.1.0` (Odoo la prefija
> con la serie vigente → `19.5.1.1.0`).

## 1. Qué hace

Lleva el expediente de una importación: la ruta marítima con su naviera y sus
puertos, los contenedores, el agente de aduanas, los certificados con su
vencimiento, los aranceles por partida, y los gastos de destino que acaban
repartidos sobre el costo de la mercancía (`stock.landed.cost`).

| Área | Modelos |
|---|---|
| Expediente | `ft.import`, `ft.import.container`, `ft.binnacle` |
| Catálogos | `ft.ports`, `ft.duty`, `ft.agreement`, `ft.containers`, `ft.storage`, `ft.import.routes`, `ft.label`, `ft.shipping.companies` |
| Certificados | `ft.certificate`, `ft.certificate.type` |
| Costos | `ft.destination.cost`, `ft.associated.expenses`, `ft.delay.fee`, `ft.vigilance.fee` |
| Previsión | `ft.forecast`, `ft.forecast.product` |

## 2. Tamaño

| | |
|---|---:|
| Módulos | 1 |
| Python | 2.892 líneas |
| XML | 2.417 líneas |
| Modelos declarados | 29 |
| Vistas | 26 archivos |
| Pruebas al llegar | **0** |
| Pruebas ahora | **12** |

## 3. La dependencia que habría bloqueado todo

El manifest de v18 declaraba `stock_enterprise`, que **no existe en
Community**: con esa línea, el módulo no se instala en esta migración y punto.

Antes de quitarla se comprobó que no se usa: ni un import, ni un modelo
heredado, ni una vista que herede de las suyas. Estaba declarada y nada más.

Las otras dos dependencias propias eran las de siempre, ya migradas:

| v18 | v20 |
|---|---|
| `l10n_ve_nimetrix` | `l10n_ve_mornix` |
| `nimetrix_dual_currency` | `mornix_dual_currency` |

## 4. Lo que v20 rompió

Nada de JavaScript —el módulo no tiene— y nada de las roturas clásicas de v17:
ya venía con `<list>`, sin `attrs`, sin `name_get`, sin `t-esc`. Lo que sí hubo
que cambiar:

| Qué | Dónde | Qué pasaba |
|---|---|---|
| `ir.model.access.csv` + `ir.rule` → `ir.access.csv` | 26 accesos y 12 reglas por compañía | El módulo no carga |
| `res.groups.category_id` → `res.groups.privilege` | 4 grupos | El módulo no carga |
| `res.groups.users` → `user_ids` | grupo de administrador | El módulo no carga |
| `_sql_constraints` → `models.Constraint` | **15 restricciones en 12 modelos** | Las restricciones no llegan a la base |
| `account.account.deprecated` | búsqueda de la cuenta de gasto | El campo ya no existe |
| `<filter name="categ_id">` en la búsqueda de productos | `search_views.xml` | v20 rehizo la vista: el xpath no encuentra nada y **aborta la instalación** |
| `<group expand="1" string="...">` en vistas de búsqueda | 2 vistas propias y 2 heredadas | v20 no admite esos atributos; la vista queda inválida |
| `target="inline"` en una acción | pantalla de ajustes | El valor ya no existe en `ir.actions.act_window` |
| `states={...}` en un campo | 2 campos del expediente | Se ignora en silencio; el «solo lectura» ya lo hace la vista |
| `digits=` en un `Monetary` | `flat_rate_dif` | Aviso en el arranque; el Monetary toma los decimales de su moneda |
| `default=_('New folio')` | 3 modelos | Se evalúa al importar el módulo, cuando todavía no hay idioma: v20 lo avisa. Ahora es `lambda self: _(...)` |

## 5. Un defecto propio que v20 destapó

El gancho de instalación crea dos impuestos de la casa —**TSA** (tasa de
servicio aduanero) y **TSS** (tasa del SENIAT)— y para ello buscaba un grupo de
impuestos así:

```python
tax_group = env['account.tax.group'].search([('company_id', 'in', [company.id, False])], limit=1)
```

El primero que hubiera. En v20 el grupo y el impuesto **tienen que compartir
país**, y con un grupo de otra localización la instalación abortaba entera con
*«The tax group must have the same country_id as the tax using it»*. Ahora el
grupo se busca por país y, si hay que crearlo, se crea con el de la compañía.

> **Consecuencia práctica:** la compañía necesita **país configurado** antes de
> instalar. Sin él, el módulo se planta con *«The company does not have a
> configured country»* — es una guarda suya, no un fallo: sin país no sabe con
> qué país crear los impuestos.

## 6. Lo que el instalador no ve

Instalar en verde no significa que las pantallas abran. Se abrieron **las 18**
en un navegador de verdad, con la consola escuchando:

```
ok expedientes · costos-destino · previsión · certificados · contenedores
ok puertos · aranceles · acuerdos · navieras · rutas · almacenes · etiquetas
ok tipos-certificado · tipos-novedad · solicitudes · pedidos · productos · ajustes
```

y los formularios nuevos de los nueve modelos que tienen formulario propio.
Cero diálogos de error, cero errores de consola.

> **Lo que sí falló, y no era el módulo:** el servidor en marcha expande
> `addons_path` **al arrancar**. Con la carpeta `addons/importaciones/` recién
> creada, el servidor la ignoraba y daba *«module foreing_trade_import: not
> installable, skipped»*: todas las pantallas del módulo salían con un «Oops».
> Un reinicio del contenedor y las 18 abrieron. Ver
> [Roturas entre versiones](../BREAKING-CHANGES.md).

## 7. Las pruebas

12, escritas en la migración —el módulo llegó sin ninguna—:

| Qué prueba | Por qué |
|---|---|
| El expediente toma su folio de la secuencia | `New folio` es solo el marcador de pantalla |
| El expediente nace con su previsión y su costo de destino | Los crea el `create`; sin ellos las pestañas van vacías |
| El expediente solo se borra en el estado inicial | Una vez en marcha se archiva |
| El arancel rechaza una tasa fuera de 0–100 | La restricción tiene que llegar a la base, no quedarse en Python |
| El código del arancel es único por compañía | Igual |
| El certificado avisa antes de vencer y marca el vencido | Es lo que pinta los semáforos del pedido |
| El certificado rechaza fechas invertidas | |
| El cron recalcula el estado de los certificados | El estado está almacenado: sin cron, vence y nadie se entera |
| La instalación creó TSA y TSS | El gancho que abortaba |
| El impuesto y su grupo comparten país | El defecto del punto 5, con prueba que lo fija |
| Los impuestos de la casa no se borran | Se archivan |
| La instalación dejó el catálogo de contenedores | |

```bash
docker compose run --rm odoo20 odoo -d import_test -u foreing_trade_import \
    --test-enable --test-tags /foreing_trade_import --stop-after-init
# 0 failed, 0 error(s) of 12 tests
```

## 8. Lo que hay que saber antes de montarlo

- **La compañía necesita país** (punto 5).
- **Una moneda activa y «sincronizar» en verdadero**, o la lista de expedientes
  aborta al calcular la tasa del día: la comprobación es de la localización
  (`mornix_currency_rate`), no de este módulo.
- **Los costos de destino exigen «Inventario / Administrador»**: la ficha
  enseña los costos adicionales (`stock.landed.cost`) y ese modelo no lo lee un
  usuario de inventario corriente. Con el permiso de usuario, la pantalla se
  queda cargando.
- **El idioma es_VE hay que activarlo**: el módulo trae sus 593 entradas
  traducidas, pero si el idioma no está activo en la base, la pantalla sale en
  inglés y la traducción no se usa.

## 9. Lo que queda abierto

- **El negocio no está probado.** Instala y abre; nadie ha llevado todavía un
  expediente de compra a recepción y reparto del costo con números reales.
- **El nombre del módulo lleva una errata de origen**: `foreing_trade_import`.
  No se corrige a la ligera: renombrar un módulo obliga a migrar datos en las
  bases de los clientes, igual que se decidió con `nimetrix.*` en la
  localización.
- **Campos duplicados con la localización**: `account.move.nx_people_type_company`
  existe aquí y en `l10n_ve_mornix` (`nx_people_type_company1`). Odoo lo avisa
  como dos campos con la misma etiqueta. Hay que decidir cuál sobra, y eso toca
  datos del cliente.
- **Etiquetas repetidas dentro del propio módulo**: diez `Document`, diez
  `Description`, tres `Flat rate`. Es cosmético, pero en una vista de búsqueda
  el usuario ve varias veces la misma palabra y no sabe cuál elegir.
- **Sin guía de usuario todavía**: la funcional está en
  [Importaciones](../funcional/60-importaciones/_app.md).
