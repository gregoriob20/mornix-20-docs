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

## El wkhtmltopdf del entorno no admite cabeceras ni pies

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Silenciosamente, al imprimir |

Ya estaba anotado mas abajo que Ubuntu 24.04 empaqueta wkhtmltopdf sin el Qt
parcheado. Lo que faltaba es **el alcance**: afecta a todo informe que use
`div.header` o `div.footer`.

Odoo no dibuja esos bloques dentro del documento: los recorta del HTML y se los
pasa a wkhtmltopdf como archivos aparte. El binario sin parchear los descarta y
**avisa por consola**, aviso que Odoo se traga:

```
The switch --header-html, is not support using unpatched qt, and will be ignored.
```

Comprobado en el contenedor de v20 con una cabecera de prueba: el texto no
aparece en el PDF. El de la instancia de 18 sí la pinta, porque es
`0.12.6.1 (with patched qt)`.

**Consecuencia practica:** no basta con revisar que el informe "sale". Si el
titulo, el logo o los datos de identificacion viven en `div.header`, el PDF sale
**sin ellos y sin error**. En la guia de despacho eso dejaba fuera el titulo, el
numero de guia, el destinatario y las cuatro casillas de firma; en el libro
fiscal, el logo, el RIF y el titulo del libro.

En 1.35.0 esos bloques se movieron al cuerpo. Se pierde la repeticion por
pagina, que es lo que se recupera cuando el entorno tenga wkhtmltopdf parcheado
o Paper Muncher.

- [ ] Al arreglar el motor, devolver a `div.header` la identificacion del libro
      fiscal y del ARC, y a `div.footer` las firmas de la guia.

## `web.minimal_layout` es quien aporta el `meta charset`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Acentos rotos en el PDF |

Un informe cuyo contenido NO va dentro de `div.article` no pasa por
`web.minimal_layout`, y el HTML llega a wkhtmltopdf sin declaracion de
codificacion. El motor asume Latin-1 y los acentos salen mal: «GuÃ­a de
Despacho», «CÃ³DIGO».

`div.article` no es decoracion: es lo que dispara ese envoltorio.

## `noupdate="1"` se guarda en la base, no en el archivo

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | El cambio no llega |

El indicador vive en `ir_model_data.noupdate`. Cambiar `noupdate` en el XML
**no** desbloquea un registro ya creado: Odoo consulta el valor almacenado y se
salta la actualizacion, en silencio.

Aparecio con `dispatch_guide_paperformat`, que reservaba 130 mm de margen
superior para una cabecera que no se pinta. Corregido el XML, la guia seguia
saliendo a media pagina en cualquier base ya instalada.

- [ ] Para que los margenes corregidos lleguen a las bases existentes hace falta
      un script de migracion que ponga `noupdate = false` en ese registro. El
      arreglo del XML solo sirve para instalaciones nuevas.

## Renombres en `stock.move`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

| v18 | v20 |
|---|---|
| `stock.move.product_uom` | **`uom_id`** |
| `stock.move.name` | **eliminado** |

La plantilla de la guia de despacho leia `line.product_uom.name` y reventaba al
imprimirse. Corregido en 1.35.0.

## Los campos Binary ya no aceptan `bytes`

| | |
|---|---|
| Estado | VERIFICADO |
| Como falla | Ejecucion |

```python
# v18
company.logo = base64.b64encode(datos)
# v20
from odoo.tools.binary import BinaryBytes
company.logo = BinaryBytes(datos)      # bytes CRUDOS, no base64
```

Con lo de antes: `TypeError: res.company.logo: use BinaryValue instead of bytes`.
Afecta a cualquier script que cargue imagenes (logos, firmas, sellos).

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

## `hr.contract` desaparecio: ahora es `hr.version`

El contrato dejo de ser un registro aparte. Odoo lo absorbio dentro de `hr`
como **`hr.version`** —"Employee Record"—, de modo que un empleado tiene
versiones sucesivas de sus condiciones en lugar de contratos enlazados.

| v18 | v20 |
|---|---|
| Modulo `hr_contract` | Fusionado en `hr` |
| Modelo `hr.contract` | `hr.version` |
| `employee_id.contract_id` | La version vigente del empleado |

Los campos del contrato sobreviven con su nombre —`wage`, `date_start`,
`date_end`, `structure_type_id`, `trial_date_end`, `resource_calendar_id`,
`hr_responsible_id`—, asi que lo que cambia es **como se llega a ellos**, no
como se llaman.

Cuidado al migrar: no es un renombrado mecanico. Un `Many2one('hr.contract')`
pasa a `Many2one('hr.version')`, pero la semantica cambia —una version no es un
contrato— y las busquedas por contrato vigente necesitan repensarse.

> OCA/payroll ya lo hizo en su rama 19.0: mantiene el campo `contract_id` y le
> cambia el comodelo a `hr.version`. Conservar el nombre del campo ahorra tocar
> todo el codigo que lo lee.

## `report_file` salio de `ir.actions.report`

Un informe que lo declare revienta al cargar el XML:

```
ParseError: while parsing report.xml, somewhere inside ...
```

Se borra el campo y ya. `report_name` sigue siendo el que manda.

## `ir.rule` e `ir.model.access` se unificaron en `ir.access`

v20 fundio los dos mecanismos de permisos en un solo modelo y un solo archivo.
Un `security.xml` con `<record model="ir.rule">` falla con `KeyError: 'ir.rule'`.

| v18 | v20 |
|---|---|
| `security/ir.model.access.csv` | `security/ir.access.csv` |
| Columnas `perm_read,perm_write,perm_create,perm_unlink` | Una columna `operation` con letras: `r`, `crud`… |
| `<record model="ir.rule">` en XML | Fila del mismo CSV, con su `domain` |

Las reglas de registro pasan a ser filas con dominio, y el grupo va en
`group_id/id`. Una regla sin grupo (multi-compañia) deja esa columna vacia.

## `res.bank` desaparecio

El modelo del banco como entidad existia en v18
(`odoo/addons/base/models/res_bank.py`) y en v20 **el archivo ya no esta**.
Odoo lo resolvio convirtiendolo en texto: `res.partner.bank.bank_name` es hoy
un `Char`.

Para una nomina venezolana eso es una perdida, no una simplificacion: los
archivos de pago al banco llevan el **codigo** de la entidad, no su nombre
escrito a mano. Quien migre algo que referencie bancos tiene que decidir con
que lo sustituye.

## `first_contract_date` se movio a `hr.employee`

Estaba accesible desde el contrato; ahora vive en el empleado. Un
`@api.depends('first_contract_date')` sobre `hr.version` **impide cargar el
modulo**:

```
ValueError: Wrong @depends on '_compute_...'. Dependency field
'first_contract_date' not found in ...
```

Se llega por el empleado: `@api.depends('employee_id.first_contract_date')`.

## Los grupos perdieron `category_id`

`res.groups.category_id` ya no existe. Se omite, o se usa `privilege_id`.

Ojo al buscarlo: `search_default_category_id` en el contexto de una accion
sobre `ir.module.module` **no es este campo** y no hay que tocarlo. Es un filtro
de busqueda.

## El contrato perdio su estado

En v18 `hr.contract` tenia `state` con `draft` / `open` / `close` / `cancel`, y un
cron —«HR Contract: update state»— que lo movia. **En v20 no queda nada de eso.** Un
contrato esta en vigor si hoy cae entre `contract_date_start` y `contract_date_end`,
y punto.

No avisa de forma clara: lo que se ve es el modulo negandose a cargar.

```
ValueError: Wrong @depends on '_compute_...' (compute method of field
hr.version.mnx_is_renewable). Dependency field 'state' not found in model hr.version.
```

El dominio que usa el propio core para «contratos vigentes»:

```python
[('contract_date_start', '<=', hoy),
 '|', ('contract_date_end', '=', False), ('contract_date_end', '>=', hoy)]
```

**Consecuencia practica:** un campo calculado a partir de ese dominio **no puede ser
`store=True`**. Un contrato vence porque pasa el tiempo, no porque alguien escriba en
el: con el valor almacenado, el boton de renovar no aparece el dia que toca.

## `date_start` y `date_end` de la version son calculados

`hr.version` tiene cuatro fechas y es facil escribir en las que no son:

| Campo | Que es | Se puede escribir |
|---|---|---|
| `date_version` | Desde cuando rige esta version | Si |
| `contract_date_start` | Inicio del contrato | Si |
| `contract_date_end` | Fin del contrato | Si |
| `date_start` / `date_end` | Vigencia de la version, calculada | **No** |

Escribir en `date_start` no da error al leer el codigo, pero el valor no llega a la
base. Las fechas del contrato son las de `contract_*`.

## El contrato ya no tiene formulario propio

v20 no trae ninguna vista `form` de `hr.version`: trae lista, grafico, pivote y
busqueda. **El formulario del contrato es el del empleado**, y sus campos viven en la
pestaña «Nomina» (`//page[@name='payroll_information']`).

Lo que eso obliga a cambiar en una vista heredada:

| v18 | v20 |
|---|---|
| `inherit_id` = `hr_contract.hr_contract_view_form` | `hr.view_employee_form` |
| `model` = `hr.contract` | **`hr.employee`** (el del padre) |
| `//group[@name='salary_info']` | `//group[@name='contract']` |
| `//div[@name='wage']` | igual, dentro de la pestaña Nomina |
| `//field[@name='date_end']` | `//div[@name='contract_dates']` |
| `hr_contract.hr_contract_view_tree` | `hr.hr_version_list_view` (modelo `hr.version`) |
| `hr_contract.hr_contract_view_search` | `hr.hr_version_search_view` |

> **La trampa:** `_inherits` delega **campos**, no **metodos**. Un boton
> `type="object"` puesto en el formulario del empleado busca el metodo en
> `hr.employee` y no lo encuentra en la version. Hace falta un envoltorio:
>
> ```python
> class HrEmployee(models.Model):
>     _inherit = 'hr.employee'
>
>     def mi_accion(self):
>         self.ensure_one()
>         return self.current_version_id.mi_accion()
> ```

## `hr.contract.history` desaparecio

Era el modelo detras de la pantalla «Historial de contratos» de v18. En v20 el
historial **es el listado de versiones del empleado** (`hr.hr_version_list_view`,
accion `hr.action_hr_version`), y la ficha ya trae un boton «History» que lo abre.

Un modulo que lo heredaba no arranca:

```
TypeError: Model 'hr.contract.history' does not exist in registry.
```

Si lo unico que hacia era añadir columnas, se añaden al listado de versiones y el
modulo se queda sin modelo propio.

## Campos del contrato que venian de Enterprise

Estos no los quito v20 «de Community»: **nunca estuvieron en Community**. Venian de
`hr_payroll` o `hr_contract_salary` de Enterprise, y el motor de OCA tampoco los trae.
Una localizacion que los use tiene que reponerlos.

| Campo | Modelo | Para que se usa en la localizacion |
|---|---|---|
| `wage_type` | `hr.version` | Mensual u horario; manda en el calculo |
| `hourly_wage` | `hr.version` | Jornal por hora, alimenta los dias trabajados |
| `contract_type_id` | `hr.version` | Sin sustituto; su sitio lo ocupa `employee_type_id` |
| `registration_number` | `hr.employee` | Retirado; la localizacion ya usa su propio codigo |
| `type_id` | `hr.payroll.structure` | Agrupa estructuras por tipo |
| `default_struct_id` | `hr.payroll.structure.type` | Estructura que se propone al generar recibos |
| `default_schedule_pay` | `hr.payroll.structure.type` | Periodicidad; decide como se promedia el salario |
| `wage_type` | `hr.payroll.structure.type` | Idem, a nivel de tipo |

**Consecuencia practica:** conviene reponerlos en el modulo mas bajo de la cadena que
los necesite, no en cada uno. Si dos modulos declaran el mismo campo con distinto tipo,
el que cargue segundo gana y el fallo aparece lejos de su causa.

## `hr.payroll.report` no existe fuera de Enterprise

El analisis de nomina —el pivote de recibos por empleado y mes— era de
`hr_payroll` (Enterprise). Ni Community ni OCA lo traen, y con el se van sus vistas
(`payroll_report_view_search`) y el menu del que colgaban los informes
(`menu_hr_payroll_report`).

Un modulo que lo extendia parcheando su SQL con `str.replace` no tiene nada que
parchear: hay que escribir la vista entera sobre las tablas de OCA
(`hr_payslip`, `hr_payslip_line`, `hr_payslip_worked_days`).

> Al escribirla, ojo con la granularidad: si se une con `employee_category_rel` —un
> empleado puede llevar varias etiquetas— cada etiqueta multiplica la fila y `wd.id`
> deja de ser clave unica. Se resuelve con `ROW_NUMBER() OVER (...)` como `id`.

## `hr.salary.rule` ya no apunta a su estructura

En Enterprise la regla pertenecia a **una** estructura (`struct_id`, Many2one). El
motor de OCA invierte la relacion: es la estructura la que lista sus reglas
(`rule_ids`, Many2many sobre `hr_structure_salary_rule_rel`). Una regla puede estar en
varias.

Los dominios que filtraban reglas por estructura fallan al validar la vista:

```
Unknown field "hr.salary.rule.struct_id" in domain of <field name="salary_rules_for_basic">
```

El lado inverso se puede declarar sin tocar OCA, pero **no como Many2many normal**:
`hr.payslip.line` hereda de `hr.salary.rule` por prototipo, asi que el campo se
clonaria con la misma tabla y las mismas columnas, y Odoo lo rechaza:

```
TypeError: Many2many fields hr.payslip.line.struct_ids and hr.salary.rule.struct_ids
use the same table and columns
```

Se declara calculado, con `search` propio: lo que los dominios necesitan es buscar, no
almacenar. Y la comparacion pasa de `=` a `in`, porque la cardinalidad cambio de verdad.

## Las vistas de busqueda ya no admiten `<group expand=... string=...>`

El `<group>` que agrupa los «Agrupar por» perdio los dos atributos. El error que sale
no nombra ninguno de los dos:

```
Invalid view <modelo>.view.search definition in False
```

El detalle solo aparece en el log, como aviso:

```
RELAXNG_ERR_INVALIDATTR: Invalid attribute expand for element group
RELAXNG_ERR_EXTRACONTENT: Element search has extra content: field
```

La forma valida es un `<group>` pelado, como hace el core.

## `res.groups.users` paso a llamarse `user_ids`

```
ValueError: Invalid field 'users' in 'res.groups'
```

Aparece al cargar un `<record model="res.groups">` con
`<field name="users" eval="[(4, ref('base.user_admin'))]"/>`.

## `mail.thread.cc` desaparecio

El mixin que guardaba los CC de los correos entrantes ya no existe:

```
TypeError: Model 'x' inherits from non-existing model 'mail.thread.cc'.
```

Si lo que se usaba de el era el hilo de mensajes, `mail.thread` a secas basta.

## `-i` sobre un modulo ya instalado no hace nada

No es una rotura de v20, pero cuesta la misma vuelta que una. Instalar con `-i` un
modulo que ya figura como `installed` **no carga sus datos**: Odoo lo salta sin decir
nada y el comando termina en verde.

Pasa, por ejemplo, al instalar un modulo con parte de su manifest comentado para
aislar un fallo: al restaurarlo, el `-i` de la segunda vuelta no vuelve a cargar nada.
Lo que se ve despues es un XML-id que «no existe» aunque el archivo lo declare.

**Consecuencia practica:** mirar el estado antes de elegir la bandera.

```bash
estado=$(psql -Atc "select state from ir_module_module where name='$m'")
[ "$estado" = installed ] && bandera=-u || bandera=-i
```

## Roturas por modulo

_(Se va llenando durante la migracion.)_

| Modulo | Sintoma | Version que lo introdujo | Solucion |
|---|---|---|---|
| | | | |
