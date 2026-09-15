# Importaciones — un módulo, 29 modelos y 12 pruebas nuevas

> Estado: **instala, las 18 pantallas abren, las 14 pruebas pasan y el camino
> completo —compra, recepción y reparto del costo— está recorrido y cuadrado**.
> Lo que falta es la columna en divisa del reparto: depende de un módulo que no
> está migrado (punto 7).
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
| Pruebas ahora | **14** |

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

### 5 bis. Un defecto que solo salió al crear una base de cliente

Al montar `auromin` desde cero —`base` primero, la compañía en Venezuela y
después los 53 módulos de una vez— la instalación llegó al final y se cayó
entera con *«the tax: TSA cannot be removed from the model, it can only be
archived»*. El orden de los hechos: el gancho de este módulo crea TSA y TSS,
y **después** Odoo carga el plan contable venezolano de la compañía nueva
borrando con `force_delete` los impuestos que hubiera para recrearlos desde
la plantilla. La guarda de `unlink`, pensada para que una persona no borre el
impuesto, se interponía al sistema.

Ahora la guarda deja pasar el contexto `force_delete` —que solo pone Odoo— y
sigue frenando el borrado a mano. Con prueba. No apareció en `import_test`
porque allí el plan ya estaba cargado cuando el módulo se instaló.

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

## 7. El camino completo, recorrido

Instalar y abrir no es funcionar. Se recorrió el flujo entero en `import_test`
con `scripts/probar_flujo_importacion.py`, y cuadra:

| Paso | Qué se hizo | Resultado |
|---|---|---|
| Compra | 100 unidades a 18 USD, proveedor no domiciliado, expediente enlazado | 1.800 USD = **65.700 Bs** |
| Recepción | Albarán validado | 100 unidades en existencias |
| Factura del viaje | Flete 2.500 USD + aduana 900 USD, con el expediente puesto | **gastos asociados creados solos** al publicarla |
| Cálculo del costo | Reparto por cantidad | 657 FOB + 912,50 CIF + 328,50 nacional = **1.898 Bs/unidad** |
| Reparto | Costo adicional validado | asiento `STJ/2026/09/0001`, 124.100 Bs a valoración |
| Comprobación | Valor en inventario | **189.800 Bs** y **1.898 Bs/unidad** |

Las dos cifras se calculan por caminos distintos —el módulo por un lado, la
valoración de Odoo por otro— y coinciden: 65.700 + 91.250 + 32.850 = 189.800.

![El reparto ya validado, con sus dos líneas de costo](../funcional/img/imp-costo-adicional-form.png)

**Tres roturas más de v20 aparecieron aquí**, y ninguna se ve instalando:

| Qué | Dónde | Síntoma |
|---|---|---|
| `purchase.order.line.product_uom` → **`uom_id`** | cálculo del costo de destino y de la previsión | `AttributeError` al calcular: el reparto no llega a empezar |
| `stock.valuation.layer` **eliminado** | la valoración vive ahora en `stock.move.value` y `product.total_value` | Cualquier comprobación de valor escrita contra SVL revienta |
| `currency_price_unit` y `nx_rate_ref` en el costo adicional | `create_landed_cost()` | `ValueError: Invalid field` — **el reparto entero se cae** |

Las dos primeras están corregidas. La tercera es de fondo y está en el punto 9.

> **La trampa que más caro sale:** un producto de tipo «bienes» **sin
> `is_storable`** deja hacer todo el camino —se recibe, se reparte, el costo
> unitario sube— y **no genera un solo asiento contable**. En la primera pasada
> el reparto salió «validado» con la contabilidad en blanco. Lo que lo delata es
> que el valor en inventario sea 0 con el costo unitario ya subido.

> **La tasa del FOB es la del día anterior.** El cálculo del costo de destino
> convierte las órdenes de compra con `_convert`, y v20 toma para una fecha la
> tasa **anterior** a ella (`name < fecha`, ver Roturas entre versiones). La
> factura del viaje, en cambio, usa la tasa del día que fija la localización.
> Si la tasa cambió ese día, FOB y gastos no comparten tasa: hay que saberlo al
> cuadrar.

> **La contrapartida del reparto es arbitraria.** `create_landed_cost()` busca
> «la primera cuenta de gasto que encuentre» y con ella carga las dos líneas;
> en la prueba salió *Cost of Goods Sold*. El gasto asociado **ya guarda la
> cuenta de la factura** (`product_invoice_account_id`) y no se usa. No se ha
> cambiado: toca a la contabilidad del cliente y es decisión suya, no de la
> migración.

## 8. Las pruebas

14, escritas en la migración —el módulo llegó sin ninguna—:

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
| **El costo repartido llega al producto** | El camino entero en una prueba: compra, recepción, factura del flete, cálculo y reparto. Fija las dos roturas de v20 del punto 7 |
| **La recarga del plan contable sí puede borrar los impuestos** | La guarda de TSA/TSS tumbaba la instalación en una base nueva (punto 5 bis) |

```bash
docker compose run --rm odoo20 odoo -d import_test -u foreing_trade_import \
    --test-enable --test-tags /foreing_trade_import --stop-after-init
# 0 failed, 0 error(s) of 14 tests
```

## 9. Lo que hay que saber antes de montarlo

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
- **La factura del viaje exige «Contabilidad / características completas»**
  (`account.group_account_user`): con la localización puesta, la factura de
  proveedor lee los libros fiscales (`nimetrix.fiscal.book`) y sin ese grupo no
  abre — «Error de acceso», sin más pistas. Apareció al tomar las capturas con
  el usuario de documentación, que solo tenía facturación.

## 10. Lo que queda abierto

- **La columna en divisa del reparto.** `create_landed_cost()` escribía
  `currency_price_unit` y `nx_rate_ref` en las líneas del costo adicional. Esos
  campos no son de Odoo: los pone `nimetrix_stock_cost_usd`, del repositorio de
  doble moneda, **que no está migrado** — y no es un porte mecánico: sus 1.741
  líneas están escritas sobre `stock.valuation.layer`, el modelo que v20
  eliminó, así que hay que rehacerlo. Mientras tanto el reparto se hace **en
  bolívares** y las columnas en divisa se escriben solo si el modelo las tiene.
- **Usar la cuenta de la factura** como contrapartida del reparto, en vez de la
  primera cuenta de gasto que aparezca (punto 7). Es una decisión contable del
  cliente.
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
- La guía de usuario está en [Importaciones](../funcional/60-importaciones/_app.md):
  catálogos, expediente y costo de destino, con el paso a paso de cada proceso
  verificado en pantalla.

## 11. Traducciones

Se revisó **todo lo que sale en pantalla** de los módulos de la casa —campos,
selecciones, menús, vistas y mensajes de Python— exportando las traducciones
es_VE de cada base (`odoo i18n export`) y separando lo que de verdad se ve en
inglés de lo que simplemente no tiene `msgstr` porque ya está escrito en
español en el código. De 8.064 términos, **~50 salían en inglés**;
quedan **0**, y ninguno es una etiqueta: son fragmentos de código,
HTML y expresiones de plantilla que no se traducen.

Las traducciones viven en `scripts/i18n/*_es_VE.json` del repositorio de
migración y las aplica `scripts/completar_traducciones.py`, que reescribe el
`i18n/es_VE.po` de cada módulo a partir de la exportación —con sus referencias,
que es lo que el importador necesita— y se cargan con `odoo i18n import -w`.

> **Tres trampas de v20 que costaron una tarde**, anotadas en
> [Roturas entre versiones](../BREAKING-CHANGES.md): la exportación ya no es
> `--i18n-export` sino el subcomando `odoo i18n export`; un término compartido
> por varios módulos sale como `#. modules: a, b` y se pierde si solo se lee el
> singular; y las traducciones de código Python **exigen la marca
> `#. odoo-python`** en el `.po`, sin la cual el `_()` sigue en inglés aunque
> el `msgstr` esté lleno.

Lo que sigue en inglés no es nuestro: «Load a Template» en la ficha del
empleado y «Connect printers to your PoS» en la caja son huecos de la
traducción es_VE del propio Odoo.
