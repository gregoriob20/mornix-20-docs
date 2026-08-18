# Cambios que rompen: v16 / v18 -> v20 (master)

Bitacora de roturas encontradas durante la migracion. Cada entrada anota en que
version se introdujo el cambio y como se resolvio, para no re-investigar lo mismo
en el siguiente modulo.

## Estado de la version destino

`master` declara internamente:

```python
version_info = (19, 5, 0, ALPHA, 1, '')   # odoo/release.py
```

Es decir, **la serie que reporta el runtime es `19.5`, no `20.0`**. Trabajamos
sobre master tratandolo como v20 por decision del proyecto, porque la rama `20.0`
todavia no existe en `odoo/odoo` (solo hay `16.0`, `17.0`, `18.0`, `19.0` y
`master`).

Consecuencias a re-verificar cuando Odoo publique la rama `20.0`:

- [ ] Codigo del cliente que compare contra `odoo.release.version_info` vera `19.5`.
- [x] El campo `version` de los manifests: resuelto usando la forma corta, que
      Odoo prefija con la serie vigente. Ver mas abajo.
- [ ] APIs de master pueden cambiar antes del release: revisar todo lo marcado
      como `RIESGO-MASTER` en este archivo.

## Requisitos de plataforma

| | v16 | v18 | v20 (master) |
|---|---|---|---|
| Python minimo | 3.7 | 3.10 | **3.12** |
| PostgreSQL minimo | 10 | 12 | **16** (`MIN_PG_VERSION`) |
| Addons en el core | 487 | 626 | 632 |

## Dependencias Python

Cambios de `requirements.txt` entre 18.0 y master:

- Se eliminaron todos los pines condicionados a `python_version < '3.12'`; el
  archivo quedo mucho mas corto porque master solo soporta Python >= 3.12.
- **Entran**: `python-magic`, `h11==0.16.0`, `tzdata` (solo win32).
- **Sale `python-ldap`**: ya no esta en `requirements.txt`, quedo unicamente en
  `Recommends` de `debian/control`. El `Dockerfile.v20` lo instala aparte.
  - [ ] Confirmar si el cliente usa `auth_ldap`.

## `http_interface` ahora escucha solo en localhost

**Confirmado en codigo, no supuesto.** Default de `--http-interface`:

| version | default |
|---|---|
| 16.0 | `''` (todas las interfaces) |
| 18.0 | `''` (todas las interfaces) |
| master | `127.0.0.1` |

`odoo/tools/config.py` en master fuerza `127.0.0.1` cuando el valor viene vacio.
Cualquier despliegue en contenedor o detras de un proxy reverso deja de responder
al actualizar, sin ningun error en el log: Odoo arranca normal y reporta
`HTTP service running on 127.0.0.1:8069`.

Solucion aplicada en `docker/odoo.conf.tmpl`: `http_interface = 0.0.0.0`. La
exposicion real la limita compose, que publica los puertos solo en el localhost
del host.

- [ ] Revisar la configuracion de produccion del cliente antes de subir a v20.

## La version del manifest ahora se valida (y decide si el modulo existe)

**El hallazgo mas importante hasta ahora. Afecta a los 226 modulos.**

v20 valida el campo `version` del manifest contra la serie que corre. Si no
coincide, marca el modulo `installable=False`:

```
WARNING: The module l10n_ve_nimetrix has an incompatible version,
         setting installable=False
```

`odoo/modules/module.py:453`. **No existe en 18.0.**

### Por que duele

El modo de fallo es silencioso. Odoo:

- registra un **WARNING**, no un error;
- **termina con codigo de salida 0**;
- simplemente no instala el modulo.

Un `-i mi_modulo` sobre un modulo con la version vieja "funciona": el proceso
termina bien y no instala nada. Si uno mira solo el codigo de salida, parece
exito. Nos paso en el primer intento del piloto.

### Que version poner

`check_version()` exige que la version empiece con la serie. Y **la serie de
master es `19.5`, no `20.0`**:

| Version en el manifest | Resultado |
|---|---|
| `18.0.0.11.0` | rechazada, no instala |
| `20.0.1.0.0` | **rechazada tambien** — la serie es 19.5, no 20.0 |
| `19.5.1.0.0` | aceptada, pero queda obsoleta cuando salga la rama 20.0 |
| `1.0.0` | **aceptada**: con 2 o 3 partes, `adapt_version()` le antepone la serie vigente |

**Convencion del proyecto: usar la forma corta `x.y.z`.** Es la unica que
sobrevive al release de v20 sin tener que tocar 226 manifests otra vez.

Detalle en `adapt_version()`: solo antepone la serie si la version tiene 3
partes o menos. Con 4 o 5 partes la deja tal cual, y entonces tiene que empezar
por la serie a mano.

- [ ] Al migrar cada modulo, cambiar `version` a la forma corta.

## `--without-demo=all` ya no acepta valores

Menor, pero rompe los comandos de siempre:

```
WARNING: option --without-demo: since 19.0, invalid boolean value: 'all',
         assume True
```

Desde 19.0 es un booleano. `--without-demo=all` sigue funcionando por
interpretacion benevola, pero lo correcto es `--without-demo` a secas.

## La validacion del `vat` cambio de sitio y ahora normaliza

**Doble cambio en `base_vat`.**

### Donde se valida

| version | mecanismo |
|---|---|
| 18.0 | `@api.constrains('vat', 'country_id') def check_vat` en `base_vat` |
| master | inverse del campo: `vat` -> `_inverse_vat` -> `_check_vat` -> `_run_vat_checks` |

Un modulo que sobreescriba `check_vat` y llame a `super().check_vat()` lanza
`AttributeError: 'super' object has no attribute 'check_vat'`. El override debe
pasar a `_check_vat(self, validation="error")`.

### Que se guarda

`_check_vat` escribe de vuelta el vat **normalizado**: `_format_vat_number`
aplica `stdnum` y elimina los separadores, asi que `J-99800001-6` queda
`J998000016`.

En 18.0 no pasaba para Venezuela: `_fix_vat_number` comparaba el prefijo del vat
contra el codigo de pais (`j-` contra `ve`), no coincidia, y devolvia el valor
sin tocar.

**Consecuencia para las localizaciones que guardan el identificador con
formato**: los reportes, TXT y XML que tomen el numero de `vat` lo veran sin
separadores. Un modulo que exima a su pais del chequeo estandar (como hace
`l10n_ve_mornix`) conserva el formato, porque la reescritura vive dentro del
metodo que se esta salteando.

- [ ] Al migrar cada modulo, revisar si compara `vat` contra un patron con
      separadores.

## «Mi cuenta» del portal se rehizo: las tarjetas son registros

Hasta v19, una tarjeta en `/my/home` se añadia heredando la plantilla:

```xml
<template inherit_id="portal.portal_my_home">
    <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
        <t t-call="portal.portal_docs_entry">
            <t t-set="title">Mis cosas</t>
            <t t-set="url" t-value="'/my/cosas'"/>
        </t>
    </xpath>
</template>
```

En v20 esa herencia **se procesa sin error y la tarjeta no sale**.
`portal.portal_docs_entry` ya no lee variables sueltas: recorre `portal_cards`,
una lista de registros del modelo nuevo `portal.entry`, y lee de cada uno
`entry.url`, `entry.name`, `entry.image` y `entry.should_show_portal_card()`.

La forma correcta es declarar un registro:

```xml
<record id="mi_entrada" model="portal.entry">
    <field name="name">Mis cosas</field>
    <field name="description">Lo que sea</field>
    <field name="url">/my/cosas</field>
    <field name="placeholder_count">mi_contador</field>
    <field name="category">vendor_category</field>
    <field name="sequence" eval="130"/>
</record>
```

El contador sigue viniendo de `_prepare_home_portal_values`, igual que antes.
`category` reparte la tarjeta entre los bloques de cliente y de proveedor.

- [ ] Cualquier modulo del cliente que añada apartados al portal hay que
      revisarlo por esto: no da error, solo desaparece la tarjeta.

## `t-esc` desaparecio de QWeb — y no avisa

**El peor de los encontrados hasta ahora**, porque no da error: el nodo se
renderiza **vacio**.

```
<span t-esc="valor"/>  ->  <span></span>
<span t-out="valor"/>  ->  <span>HOLA</span>
```

Comprobado renderizando las dos directivas con el mismo valor en la instancia.
`t-esc` quedo obsoleto en v17 en favor de `t-out`, y en v20 el compilador de
QWeb ya no lo trata: `_compile_directive_esc` no existe. Como QWeb ignora los
atributos que no conoce, la plantilla se procesa sin queja y el PDF sale.

Consecuencia en este proyecto: **280 usos en `l10n_ve_mornix`**, repartidos por
los nueve reportes. Los comprobantes de retencion de IVA e ISLR, el ARC, la guia
de despacho, el libro de inventario y los libros fiscales salian con los campos
en blanco. Se detecto por casualidad, al ver que la pestaña del navegador de la
documentacion no tenia titulo.

Se sustituyeron los 310 usos (los 280 mas los de `mornix_docs`) por `t-out`, que
escapa igual. `t-raw`, eliminado antes, tambien se sustituye por `t-out`.

`scripts/verificar_estandar.py` lo comprueba desde ahora.

- [ ] **Cualquier modulo que se migre a partir de aqui**: barrer `t-esc` antes
      de dar por bueno un reporte. Que salga el PDF no significa que lleve
      datos.

## v20 paso de OWL 2 a OWL 3

**Lo de mayor alcance para los modulos con JavaScript.** Verificado en
`addons/web/static/lib/owl/` y en `addons/web/static/src/owl2/owl3_compatibility_layer.js`,
que el manifest de `web` carga por defecto.

### Que dice la propia capa de compatibilidad de Odoo

```
1. Directivas de plantilla:
   t-portal -> t-custom-portal
   t-model  -> t-custom-model
2. Hooks:
   useEffect -> useLayoutEffect
3. defaultProps ya no se respeta: los valores por defecto van en el esquema
   de props (useProps con t.string().optional(...))
```

### Hooks que expone OWL 3

`useApp`, `useById`, `useConfig`, `useEffect`, `useListener`, `useOnChange`,
`usePlugin`, `useProps`, `useScope`.

**`useRef` ya no existe.** Un `import { useRef } from "@odoo/owl"` resuelve a
`undefined` y falla al ejecutarse con `TypeError: useRef is not a function`.

### Por que es peor que otras roturas

Un componente de la barra de navegacion que falle **deja toda la interfaz en
blanco**, no solo su widget. Y no lo detecta ninguna prueba de Python: hay que
abrir el navegador.

Nos paso con el widget de tasa de cambio, dos veces seguidas: primero por pedir
un servicio inexistente, despues por `useRef`.

- [ ] Inventariar el JavaScript de los 226 modulos. El volumen de JS por modulo
      es el mejor predictor del costo de migrarlo.

## Servicios de enterprise que no existen en community

`useService("home_menu")` lanza `Service home_menu is not available`: ese
servicio lo aporta enterprise. Si el componente vive en la barra de navegacion,
el error deja la interfaz en blanco.

Conviene revisar cada `useService` contra los servicios registrados en el core
antes de dar por buena una vista.

## La API de consultas SQL se rehizo entera

Afecta a cualquier modulo que construya SQL a mano a partir de un dominio —
tipico en reportes y libros. Todo esto **desaparecio** en v20:

| v18 | v20 |
|---|---|
| `check_access_rights('read')` | `check_access('read')` — sobre un recordset vacio comprueba el permiso a nivel de modelo, que es lo que hacia el metodo viejo |
| `check_access_rule(...)` | absorbido por `check_access` |
| `_where_calc(domain)` | `_search(domain)`, que ya devuelve un `Query` |
| `_apply_ir_rules(query)` | lo aplica `_search` por dentro |
| `sql.code` / `sql.params` | **los objetos `SQL` son opacos**: no exponen el texto ni los parametros |

Antes se concatenaban cadenas; ahora se componen objetos `SQL` y se pasa el
resultado entero al cursor:

```python
# v18
query = am._where_calc(domain)
am._apply_ir_rules(query)
cr.execute(f"SELECT ... FROM {query.from_clause.code} WHERE {query.where_clause.code}",
           query.where_clause.params)

# v20
from odoo.tools import SQL
query = am._search(domain)
cr.execute(SQL("SELECT %s FROM %s WHERE %s",
               SQL(select_clause, *select_params),
               query.from_clause,
               query.where_clause))
```

En `SQL(code, *args)`, los argumentos que son objetos `SQL` se insertan como
SQL; el resto viaja como parametro. Es lo que evita la inyeccion.

**Modo de fallo:** el modulo instala perfecto y revienta al generar el reporte.
Nada en la instalacion toca ese camino.

## `ir.config_parameter.get_param` reemplazado por accesores tipados

| v18 | v20 |
|---|---|
| `get_param(key, default)` | `get_str`, `get_bool`, `get_int`, `get_float` |
| `set_param(key, value)` | `set_str`, `set_bool`, `set_int`, `set_float` |

Mismo modo de fallo silencioso: solo aparece cuando se ejecuta el codigo que lo
usa.

## `_uid` ya no existe en los recordsets

En 18.0, `odoo/models.py` definia `_uid = property(lambda self: self.env.uid)`.
En master no existe. Rompe sobre todo en valores por defecto:

```python
# v18
default=lambda s: s._uid
# v20
default=lambda s: s.env.uid
```

Falla en el momento de calcular el default, no al importar, asi que aparece
tarde: el modulo instala bien y revienta al abrir el formulario o al crear un
registro en un test.

- 6 archivos del codigo del cliente lo usan.

## `SingleTransactionCase` eliminada

**Confirmado en codigo.** `odoo/tests/common.py`:

| version | tiene `SingleTransactionCase` |
|---|---|
| 16.0 | si |
| 18.0 | si |
| master | **no** |

En v20 quedan `BaseCase`, `TransactionCase` y `HttpCase`.

El reemplazo es `TransactionCase`, pero el aislamiento cambia:
`SingleTransactionCase` compartia una unica transaccion entre todos los tests de
la clase, `TransactionCase` da una por test con rollback. Los tests que dependian
de datos creados por un test anterior van a fallar, y ese fallo es correcto.

El codigo actual del cliente no la usa (verificado con grep sobre
`/opt/odoo-client/`), asi que hoy no bloquea nada. Queda anotado por si aparece
en codigo que todavia no hemos revisado.

## `base_vat` no existe como modulo en v20

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Al actualizar el modulo |

`l10n_ve_mornix` declara `base_vat` en `depends`. En v20 la actualizacion aborta:

```
odoo.exceptions.UserError: You try to upgrade the module l10n_ve_mornix that
depends on the module: base_vat.
```

**El directorio sigue estando, y eso despista.** `/opt/odoo/addons/base_vat/`
existe en el arbol de master, pero dentro solo queda la carpeta `i18n`: no hay
`__manifest__.py` ni modelos. Odoo no lo registra como modulo — `update_list()`
deja el conteo igual (674 antes, 674 despues) y `base_vat` no aparece. O sea:
comprobar que la carpeta existe **no basta** para dar la dependencia por buena.

La validacion de RIF/VAT que aportaba hay que buscarla en `base`, o asumirla en
la propia localizacion.

- [ ] Quitar `base_vat` de `depends` y verificar que la validacion del
      identificador venezolano sigue en pie sin el. Hay un apaño local en el
      arbol de trabajo (`__manifest__.py` y `views/res_partner.xml`, rotulado
      «DESACTIVADO SOLO EN LOCAL PARA LA PRUEBA») que **no esta confirmado**:
      sirve para levantar el entorno, no como solucion.

## Motor de reportes PDF

master incorpora el addon **`base_report_paper_muncher`** ("Report Engine: Paper
Muncher"), el motor de render propio de Odoo (https://odoo.github.io/paper-muncher/).
No existe en 18.0. `wkhtmltopdf` sigue referenciado en varios addons del core.

- [ ] Definir con que motor se renderizan los reportes del cliente en v20.

**La decision ya tiene consecuencias concretas.** Los seis formatos de retencion
(1.34.0) estan disenados y verificados contra `wkhtmltopdf`, y tres detalles de
su hoja de estilos son rodeos a limitaciones del Qt WebKit que lleva dentro:
nada de `linear-gradient` (no lo pinta), pildoras con relleno en vez de borde
(no pinta el borde de un `inline-block` dentro de una celda) y filetes de `.6pt`
como minimo (a `.5pt` desaparecen al rasterizar a 110 dpi). Paper Muncher no
tiene ninguna de las tres limitaciones, asi que en principio los rodeos siguen
funcionando — pero **hay que regenerar los seis y mirarlos** antes de dar el
cambio por bueno. Detalle en `docs/modulos/l10n_ve_mornix.md`, seccion 6.22.
### El wkhtmltopdf de Ubuntu 24.04 exige display y no admite pies de pagina

Ubuntu 24.04 empaqueta `wkhtmltopdf 0.12.6-2build2` **sin el Qt parcheado**
(upstream no publica build para Noble). **Verificado en ejecucion**, con dos
consecuencias y ambas silenciosas:

1. **Exige un display.** Sin el falla con `QPainter::begin(): Returned false`,
   **devuelve codigo de salida 0** y no escribe el archivo. Resuelto en
   `docker/Dockerfile.v20` con un envoltorio que lo corre bajo `xvfb`.
2. **No admite encabezados ni pies de pagina.** Con cualquier opcion
   `--header-*` o `--footer-*` termina sin generar nada, tambien con codigo 0.

Afecta a **todos** los reportes PDF de Odoo, no solo a los del cliente: sin el
envoltorio el criterio A7 no se puede cumplir, y el fallo no deja rastro en el
codigo de salida.

- [ ] Si el cliente necesita encabezados o pies en sus reportes, hay que
      compilar el wkhtmltopdf parcheado o pasar a `base_report_paper_muncher`.

## `res.users.groups_id` pasa a llamarse `group_ids`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

```python
# v18
usuario.groups_id
# v20
usuario.group_ids          # grupos asignados explicitamente
usuario.all_group_ids      # incluye los implicados
```

`odoo/addons/base/models/res_users.py:249`. Cualquier codigo que asigne o
consulte grupos por el nombre viejo revienta con
`ValueError: Invalid field 'groups_id' in 'res.users'`.

- [ ] Al migrar cada modulo, buscar `groups_id` en el codigo Python. En las
      vistas y en los manifests el atributo `groups=` no cambia.

## Un `--` dentro de un comentario XML aborta el arranque

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | **Enmascarado** |

No es un cambio de v20 —el XML nunca lo permitio— pero merece estar aqui por
como se manifiesta: Odoo no dice que archivo es ni que el problema es el
comentario. Aborta la carga entera del registro con un traceback de lxml:

```
lxml.etree.XMLSyntaxError: Double hyphen within comment
odoo.registry: Failed to load registry
```

Aparece al redactar explicaciones largas en las vistas, que es justo lo que uno
hace al migrar. `scripts/verificar_estandar.py` lo detecta antes de intentar la
actualizacion.

## Python 3.12 elimino `base64.encodestring`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

```python
# v16/v18 sobre Python <3.9
out = base64.encodestring(fp.getvalue())
# v20 (Python 3.12)
out = base64.b64encode(fp.getvalue()).decode()
```

Se renombro a `encodebytes` en Python 3.1 y se elimino en 3.9. v20 exige 3.12,
asi que el nombre viejo lanza `AttributeError`. Aparece en el codigo que genera
adjuntos y reportes, que suele ser el mas viejo del modulo.

## Un campo Binary rechaza `bytes`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

```
TypeError: <campo>: use BinaryValue instead of bytes
```

`odoo/orm/fields_binary.py:98`. En v20 un campo Binary acepta una **cadena**
base64, que decodifica sola, o un `BinaryValue`. El unico campo al que se le
pueden pasar bytes crudos es `raw` (el de `ir.attachment`), porque ahi Odoo sabe
que no vienen codificados.

Al **leer** pasa lo simetrico: el campo devuelve un `BinaryValue`, no base64.
El contenido esta en `.content`:

```python
# v18
datos = base64.b64decode(registro.campo)
# v20
datos = registro.campo.content
```

- [ ] Al migrar cada modulo, revisar todo `write` sobre un campo Binary y toda
      lectura que asuma base64.

## Las listas de facturas por tipo ahora heredan de la generica

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | **Silencioso** |

En v18, `account.view_out_invoice_tree`, `view_in_invoice_tree` y sus hermanas
eran vistas **independientes**: extender `account.view_invoice_tree` no llegaba
a ellas, asi que un modulo que quisiera una columna en todas las listas tenia
que extender cada una por separado.

En v20 **heredan** de la generica (`inherit_id = account.view_invoice_tree`,
modo `primary`). Comprobado sobre la base.

Consecuencia: el modulo que siga extendiendolas una por una vera sus columnas
**por duplicado** -- dos "N° Control", dos "N° Factura" --. No da error: solo
salen columnas repetidas.

La forma correcta en v20 es extender solo la generica. Para que una columna
aparezca unicamente donde tiene sentido, el propio core usa el contexto de la
accion, no la fila:

```xml
<field name="x_numero_cliente"
       column_invisible="context.get('default_move_type') in ('in_invoice', 'in_refund')"/>
```

- [ ] Al migrar cada modulo, buscar extensiones de las vistas de lista por tipo
      y quedarse solo con la de `account.view_invoice_tree`.
- [ ] Recordar que Odoo **no borra** un `ir.ui.view` porque su registro
      desaparezca del XML: se queda en la base y sigue aplicandose. Hay que
      eliminarlo con un script `pre-`.

## `precompute=True` sobre un campo que depende de un `default`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | **Silencioso** |

No es un cambio de v20, pero se destapo migrando y merece estar aqui por como
se manifiesta: **no falla nunca, simplemente el numero es otro**.

Un campo calculado y almacenado con `precompute=True` se calcula **antes** del
INSERT. Si depende de otro campo que recibe su valor por `default=`, puede
leerlo todavia vacio y guardar el resultado de esa lectura. Y como la
dependencia solo se dispara al **escribir** el campo del que depende, un
`default` no la vuelve a activar: el valor mal calculado se queda para siempre.

El caso: `nx_currency_ref_rate = 1 / (nx_rate or 1)`, con `precompute=True`.
`nx_rate` llega por `default`, asi que el calculo veia 0 y guardaba **1**. Los
totales en divisa de la factura salian identicos a los de bolivares. Medido
sobre la base: **36 de 52 facturas**.

Se resuelve quitando el `precompute`: el calculo pasa a correr despues del
create, con el valor ya puesto.

- [ ] Al migrar cada modulo, revisar los campos `precompute=True` que dependan
      de campos con `default=`. Un `default` no dispara la dependencia.
- [ ] Y arreglar lo ya guardado: quitar el precompute no recalcula lo viejo.
      Hace falta `env.add_to_compute(...)` en una migracion.

## El asistente de pagos guarda los lotes como ids, no como recordsets

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

```python
# v18
liquidity_lines = wizard.batches[0]['lines']      # recordset
# v20
liquidity_lines = wizard._get_batches()[0]['lines']
```

`account/wizard/account_payment_register.py:406` guarda ahora
`batch['lines'] = batch['lines'].ids` para preparar el prefetch, y el core anade
`_get_batches()` para volver a leerlas. La linea no existe en 18.0.

Quien siga leyendo el atributo directo recibe una **lista de enteros** y revienta
al pulsar "Registrar pago":

```
AttributeError: 'list' object has no attribute 'mapped'
```

## `exchange_move_id` se movio de la conciliacion completa a la parcial

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

| | v18 | v20 |
|---|---|---|
| `account.full.reconcile.exchange_move_id` | existe | **eliminado** |
| `account.partial.reconcile.exchange_move_id` | — | existe |

Crear la conciliacion completa con esa clave lanza:

```
ValueError: Invalid field 'exchange_move_id' in 'account.full.reconcile'
```

El enlace se deriva ahora de las parciales, via
`account.move.line.exchange_move_ids`. El core simplemente **omite la clave** al
crear la conciliacion completa; un modulo que sobreescriba `_reconcile_plan` o
similar tiene que hacer lo mismo.

- [ ] Al migrar cada modulo, buscar `exchange_move_id` sobre full.reconcile y
      lecturas de `wizard.batches`.

## `stock` — la valoracion salio de `stock.valuation.layer`

| Cambio | Estado | Como falla |
|---|---|---|
| **`stock.valuation.layer` eliminado** | VERIFICADO | Instalacion |
| La valoracion vive ahora en `stock.move`: `value`, `quantity`, `is_in`, `is_out` | VERIFICADO | — |
| **`stock.move.name` eliminado** (en 18.0 era obligatorio) | VERIFICADO | Ejecucion |
| `stock.move.is_valued` es calculado **sin almacenar**: no se puede usar en un dominio | VERIFICADO | Ejecucion |

Cuidado con **`product.value`**, que por el nombre parece el sustituto y no lo
es: su propio docstring dice que guarda las revaluaciones **manuales**, no cada
movimiento. Quien migre por ahi obtendra un reporte casi vacio, sin error.

| v18 | v20 |
|---|---|
| `stock.valuation.layer.value` | `stock.move.value` |
| `stock.valuation.layer.quantity` | `stock.move.quantity` |
| signo de `value` para saber si entra o sale | `is_in` / `is_out`, explicitos |

Buscar por `is_valued` lanza:

```
ValueError: Field has no SQL representation because it is not stored
```

Sirve `('value', '!=', 0)`, que es lo mismo y si esta almacenado.

- [ ] Al migrar cada modulo, buscar `stock.valuation.layer` y `stock.move.name`.

## Roturas por modulo

_(Se va llenando durante la migracion.)_

| Modulo | Sintoma | Version que lo introdujo | Solucion |
|---|---|---|---|
| | | | |
