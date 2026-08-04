# Referencia de cambios v16 → v20, por módulo de Odoo

Documento para equipos que van a migrar sus propios módulos. Está organizado por
**módulo del núcleo de Odoo**: busque el que su código extiende y lea qué
cambió debajo.

## Cómo leer este documento

| Marca | Significado |
|---|---|
| **VERIFICADO** | Comprobado contra el código de `master` y reproducido en una migración real |
| **ESPERADO** | Documentado por Odoo, todavía no confirmado contra código propio |

Y sobre cómo falla cada cosa, que es lo que decide cuánto cuesta encontrarla:

| Momento | Qué significa |
|---|---|
| **Instalación** | El módulo no instala. Se ve enseguida. Es el caso barato. |
| **Ejecución** | El módulo instala bien y revienta cuando alguien usa esa pantalla o genera ese reporte. |
| **Silencioso** | No falla nunca. Simplemente el resultado es otro. **Es el caso caro.** |

---

## Antes de empezar: dos cosas que no dependen del módulo

### La versión del manifest decide si su módulo existe

v20 valida el campo `version`. Si no coincide con la serie que corre, marca el
módulo `installable=False`, deja un **WARNING** (no un error), **termina con
código de salida 0** y no instala nada.

```python
'version': '18.0.1.2.3'   # rechazada
'version': '20.0.1.2.3'   # rechazada también: la serie de master es 19.5
'version': '1.2.3'        # correcta: Odoo le antepone la serie vigente
```

Use siempre la **forma corta**. Es la única que sobrevive al release de v20 sin
tener que volver a tocar todos los manifests. **VERIFICADO · Instalación**

### El manifest de su módulo probablemente miente

En un análisis sobre 86 módulos encontramos **42 campos usados sin declarar la
dependencia**, en 9 módulos. El síntoma típico: el módulo instala limpio y falla
al generar un reporte, porque el SQL referencia un campo de otro módulo.

Antes de migrar, conviene medir de qué depende su código **de verdad**, no lo que
dice el manifest.

---

## `base` — el núcleo

### `res.partner`

| Cambio | Estado | Cómo falla |
|---|---|---|
| **`company_type` eliminado.** Era el Selection `person`/`company` | VERIFICADO | Instalación |
| **`is_company` pasa a ser calculado y almacenado**, derivado de si el contacto es su propia entidad comercial y tiene identificación fiscal válida | VERIFICADO | **Silencioso** |
| **`company_registry` eliminado.** Lo sustituye el mecanismo de identificadores adicionales | VERIFICADO | Instalación |

El segundo es el peligroso. Si su localización distingue persona natural de
jurídica —y de eso dependen retenciones o impuestos— la heurística de Odoo puede
clasificar mal: en países donde una persona natural también tiene identificación
fiscal, **queda clasificada como empresa**.

Odoo prevé que cada localización refine esa definición. La salida más
conservadora es devolverle a `is_company` su carácter escribible y reponer
`company_type` como campo de interfaz:

```python
is_company = fields.Boolean(compute='_compute_is_company', store=True, readonly=False)

company_type = fields.Selection(
    selection=[('person', 'Individual'), ('company', 'Company')],
    compute='_compute_company_type', inverse='_write_company_type')
```

Así el valor que el usuario fijó se respeta y la heurística queda solo como valor
inicial.

### `res.groups`

| Cambio | Estado | Cómo falla |
|---|---|---|
| **`category_id` eliminado.** Lo sustituye `privilege_id`, que apunta al modelo nuevo `res.groups.privilege` | VERIFICADO | Instalación |

Los grupos técnicos u ocultos en v20 simplemente **no declaran** `privilege_id`.
Si su grupo usaba `base.module_category_hidden`, el equivalente es omitir el campo:

```xml
<!-- v18 -->
<field name="category_id" ref="base.module_category_hidden"/>
<!-- v20: se quita la línea -->
```

### Seguridad: `ir.model.access` e `ir.rule` se fundieron

| Cambio | Estado | Cómo falla |
|---|---|---|
| Ambos modelos pasan a ser un único **`ir.access`** | VERIFICADO | Instalación |

En disco, `security/ir.model.access.csv` más los XML de reglas se convierten en
un solo `security/ir.access.csv`:

```
v18   id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
v20   id,name,model_id,group_id/id,operation,domain
```

Tres detalles que cuestan tiempo si no se saben:

- `model_id` deja de ser un identificador externo (`model_account_move`) y pasa a
  ser el **nombre del modelo** (`account.move`).
- Los cuatro permisos se funden en una cadena CRUD: `r`, `cru`, `crud`.
- Las reglas globales quedan con el grupo **vacío** y `operation=crud`.

Cuidado al deducir el nombre del modelo desde el identificador externo: falla con
los modelos que llevan guión bajo. `model_x_y_wh_islr` puede ser `x.y.wh.islr` o
`x.y.wh_islr`, y solo el código lo sabe.

### `ir.config_parameter`

| Cambio | Estado | Cómo falla |
|---|---|---|
| `get_param` / `set_param` reemplazados por accesores tipados | VERIFICADO | Ejecución |

```python
# v18
self.env['ir.config_parameter'].sudo().get_param('clave', default='x')
# v20
self.env['ir.config_parameter'].sudo().get_str('clave', 'x')
# también get_bool, get_int, get_float y sus set_*
```

### `ir.actions.report`

| Cambio | Estado | Cómo falla |
|---|---|---|
| `report_file` eliminado | VERIFICADO | Instalación |

---

## El ORM

Es el bloque con más cambios estructurales, y varios fallan solo al ejecutar.

| Cambio | Estado | Cómo falla |
|---|---|---|
| `odoo/models.py` y `odoo/fields.py` pasan a ser **paquetes**; aparece `odoo/orm/` | VERIFICADO | Instalación |
| **`odoo/osv/` eliminado**, incluido `expression`. Ahora es `odoo.orm.domains.Domain` | VERIFICADO | Instalación |
| `_sql_constraints` sin soporte | VERIFICADO | Instalación |
| **`_uid` ya no existe** en los recordsets | VERIFICADO | Ejecución |
| `check_access_rights` y `check_access_rule` → **`check_access`** | VERIFICADO | Ejecución |
| `_where_calc` y `_apply_ir_rules` → **`_search`** | VERIFICADO | Ejecución |
| Los objetos **`SQL` son opacos**: sin `.code` ni `.params` | VERIFICADO | Ejecución |
| Los campos `Selection` ya no aceptan `''` como vacío; hay que usar `False` | VERIFICADO | Ejecución |
| `name_get()` → `_compute_display_name` | ESPERADO (v17) | **Silencioso** |

### Restricciones SQL

```python
# v18
_sql_constraints = [('mi_uniq', 'unique (campo)', 'Mensaje')]
# v20
_mi_uniq = models.Constraint('unique (campo)', 'Mensaje')
```

El nombre del atributo sustituye al identificador de la tupla.

### Construir SQL a partir de un dominio

Es el cambio que más código mueve. Afecta a cualquier reporte o libro que arme
su propia consulta:

```python
# v18
am.check_access_rights('read')
query = am._where_calc(dominio)
am._apply_ir_rules(query)
cr.execute(f"SELECT ... FROM {query.from_clause.code} WHERE {query.where_clause.code}",
           query.where_clause.params)

# v20
from odoo.tools import SQL
am.check_access('read')
query = am._search(dominio)          # ya trae las reglas de registro aplicadas
cr.execute(SQL("SELECT %s FROM %s WHERE %s",
               SQL(select_clause, *select_params),
               query.from_clause,
               query.where_clause))
```

En `SQL(code, *args)`, los argumentos que son objetos `SQL` se insertan como SQL
y el resto viaja como parámetro. Eso es lo que evita la inyección.

### `name_get` merece un párrafo aparte

No da error: **deja de llamarse**. Un modelo que todavía lo define muestra mal su
nombre desde v17, sin que nada lo avise. Si su base viene de v16, es probable que
tenga varios y que nadie lo haya notado.

---

## `account` — contabilidad

| Cambio | Estado | Cómo falla |
|---|---|---|
| **`account.account.deprecated` eliminado.** Lo sustituye `active`, invertido | VERIFICADO | Ejecución |
| Las **acciones de recibo desaparecieron** (`action_move_out_receipt_type`, `action_move_in_receipt_type`). Los tipos `out_receipt`/`in_receipt` siguen existiendo, pero sin punto de entrada | VERIFICADO | Instalación |
| **`amount_residual` salió de la lista de apuntes.** El campo sigue en el modelo; en la vista lo sustituye `residual_at_date` | VERIFICADO | Instalación |

```python
# v18
[('deprecated', '=', False)]
# v20
[('active', '=', True)]
```

---

## `base_vat` — identificación fiscal

Dos cambios que van juntos y que importan mucho en cualquier localización.

| Cambio | Estado | Cómo falla |
|---|---|---|
| La validación pasó de la restricción `check_vat` al **inverse del campo**: `vat` → `_inverse_vat` → `_check_vat` → `_run_vat_checks` | VERIFICADO | Ejecución |
| `_check_vat` **normaliza y reescribe** el identificador, quitándole los separadores | VERIFICADO | **Silencioso** |

Si su módulo sobreescribía `check_vat`, la llamada a `super()` lanza
`AttributeError` porque ese método ya no existe. El override debe pasar a
`_check_vat(self, validation="error")`.

Y la normalización: `J-12345678-9` queda almacenado como `J123456789`. En v18 no
pasaba para muchos países, porque la función comparaba el prefijo del
identificador contra el código de país, no coincidía y devolvía el valor intacto.

**Consecuencia:** los reportes y archivos que impriman el identificador tomándolo
de `vat` lo verán sin separadores. Si presenta declaraciones a una administración
tributaria, verifíquelo antes.

Un módulo que exima a su país del chequeo estándar conserva el formato — porque
la reescritura vive dentro del método que se está salteando.

### Si su módulo guarda el identificador en un campo propio

Es el patrón más común en las localizaciones venezolanas: el identificador vive
en un campo del módulo (`x_rif`, `cedula`, `nit`…) y `vat` queda vacío o
desactualizado, sincronizado a mano con onchanges y constraints.

**Conviene aprovechar la migración para consolidarlo en `vat`.** No es cosmético:

- `same_vat_partner_id`, la detección de contactos duplicados, el portal, los
  documentos electrónicos y cualquier módulo de terceros consultan `vat`. Si el
  identificador no está ahí, ninguno de ellos lo ve.
- Con dos campos, uno de los dos siempre termina desactualizado, y cuál de ellos
  se imprime depende de qué reporte se ejecute.

Si necesita conservar el tipo de documento y el número como campos separados en
el formulario, decláre­los **calculados a partir de `vat` con `inverse`**, no como
campos almacenados aparte: así se escriben por separado pero solo se guarda el
dato una vez.

Dos trampas verificadas al hacerlo:

1. **No le ponga `default` al campo calculado con inverse.** Un valor por defecto
   entra en el `create` como si lo hubiera escrito el usuario, dispara el inverse
   antes de que nadie mire el `vat` recibido y, como todavía no hay número, lo
   deja vacío. El valor inicial póngalo en el propio `compute`.

2. **Al quitar un campo, Odoo elimina su columna** al terminar la actualización
   (`ir.model.fields.unlink` la suelta). Comprobado: tras el update, las columnas
   viejas ya no están en `information_schema`. El script de migración que traslada
   los datos es la única oportunidad de rescatarlos, y el respaldo previo la única
   red. No hay vuelta atrás después.

Y una tercera al actualizar el módulo:

```
Field "x_rif" does not exist in model "res.partner"
```

La vista nueva es correcta; la que la invalida es una **vista hija que todavía
tiene en la base la versión anterior**, porque Odoo valida contra el arch
combinado y aún no le tocó el turno de reescribirse. Se resuelve con un script
`pre-` que borre esos registros de `ir_ui_view`: el propio update los recrea
desde el XML unas líneas más adelante y recuperan su xmlid.

---

## `product` — productos y precios

| Cambio | Estado | Cómo falla |
|---|---|---|
| El div `list_price_uom` se renombró a **`list_price_per_uom_id`** | VERIFICADO | Instalación |
| **`product.pricelist.item.product_uom`** se renombró a `product_uom_name` | VERIFICADO | Instalación |
| **`product_variant_easy_edit_view` eliminada** sin equivalente | VERIFICADO | Instalación |
| Los kanban de producto se reestructuraron: el precio dejó de estar en el primer `<span>` | VERIFICADO | Instalación |

Sobre los kanban: conviene **no anclar a la maquetación**. En vez de
`//main[hasclass('pe-2')]/span[1]`, ancle al envoltorio del propio campo:

```xml
<xpath expr="//field[@name='list_price']/.." position="replace">
```

Sobrevive a los cambios de diseño, que en Odoo son frecuentes.

---

## `purchase` — compras

| Cambio | Estado | Cómo falla |
|---|---|---|
| El botón **`action_purchase_history` salió del formulario**. El action sigue existiendo, el botón no | VERIFICADO | Instalación |
| **`taxes_id` se renombró a `tax_ids`** en la línea de pedido | VERIFICADO | Instalación |

Si insertaba campos después de ese botón, en v20 el equivalente es anclar
después de `price_unit`: era su vecino inmediato.

---

## `stock_account` — valoración de inventario

El cambio más profundo de este bloque.

| Cambio | Estado | Cómo falla |
|---|---|---|
| **El modelo `stock.valuation.layer` fue eliminado.** Lo sustituye `product.value` | VERIFICADO | Ejecución |
| En `account.move`, `stock_valuation_layer_ids` pasa a ser **`stock_move_ids`** | VERIFICADO | Ejecución |

```python
# v18
if move.stock_valuation_layer_ids: ...
# v20
if move.stock_move_ids: ...
```

Si su módulo no declara `stock_account` como dependencia —cosa frecuente—
conviene proteger el acceso, para que funcione con y sin ese módulo instalado:

```python
capas = getattr(move, 'stock_move_ids', False)
```

---

## Vistas — aplica a todos los módulos

| Cambio | Estado | Cómo falla |
|---|---|---|
| `attrs` y `states` eliminados | ESPERADO (v17) | Instalación |
| `<tree>` renombrado a `<list>` | ESPERADO (v18) | Instalación |
| **`<group>` ya no se admite dentro de `<search>`** | VERIFICADO | Instalación |
| **Un `<label for="x">` huérfano invalida la vista** | VERIFICADO | Instalación |
| Muchos anclajes de `xpath` se movieron o renombraron | VERIFICADO | Instalación |

El del `label` es fácil de pasar por alto: si su módulo reemplaza un campo por
otro, tiene que reemplazar **también su etiqueta**. En v18 la etiqueta quedaba
huérfana en silencio; en v20 la vista no pasa la validación.

```xml
<xpath expr="//label[@for='vat']" position="replace">
    <label for="mi_campo" string="Mi etiqueta"/>
</xpath>
<xpath expr="//field[@name='vat']" position="replace">
    <field name="mi_campo"/>
</xpath>
```

---

## JavaScript — OWL 2 pasa a OWL 3

**El cambio de mayor alcance para módulos con interfaz propia.**

| Cambio | Estado | Cómo falla |
|---|---|---|
| **`useRef` ya no existe** | VERIFICADO | Ejecución |
| `useEffect` → `useLayoutEffect` | VERIFICADO | Ejecución |
| `t-portal` → `t-custom-portal` | VERIFICADO | Ejecución |
| `t-model` → `t-custom-model` | VERIFICADO | Ejecución |
| `defaultProps` ya no se respeta; los valores por defecto van en el esquema de props | VERIFICADO | Ejecución |
| Servicios de enterprise (`home_menu`) no existen en community | VERIFICADO | Ejecución |

Odoo incluye una capa de compatibilidad (`web/static/src/owl2/owl3_compatibility_layer.js`),
cargada por defecto, que amortigua parte del salto. No lo elimina: los cambios
de la tabla siguen siendo necesarios.

Hooks que expone OWL 3: `useApp`, `useById`, `useConfig`, `useEffect`,
`useListener`, `useOnChange`, `usePlugin`, `useProps`, `useScope`.

### Por qué esto merece atención aparte

Un componente que falle **en la barra de navegación deja toda la interfaz en
blanco**, no solo su widget. Y ninguna prueba de Python lo detecta: hay que
abrir el navegador.

Por eso, al estimar el costo de migrar un módulo, **el volumen de JavaScript es
mejor predictor que el de Python**.

## Pruebas

| Cambio | Estado | Cómo falla |
|---|---|---|
| **`SingleTransactionCase` eliminada** | VERIFICADO | Ejecución |
| `--without-demo=all` ya no acepta valor; use `--without-demo` | VERIFICADO | Ejecución |

El reemplazo de `SingleTransactionCase` es `TransactionCase`, pero **el
aislamiento cambia**: la primera compartía una transacción entre todos los tests
de la clase; la segunda da una por test, con rollback. Los tests que dependían de
datos creados por un test anterior van a fallar — y ese fallo es correcto.

---

## Entorno

| Requisito | v16 | v20 |
|---|---|---|
| Python | 3.7+ | **3.12+** |
| PostgreSQL | 10+ | **16+** |

| Cambio | Estado | Cómo falla |
|---|---|---|
| **`http_interface` pasa a `127.0.0.1`** por defecto (en v16 y v18 era «todas las interfaces») | VERIFICADO | **Silencioso** |
| `addons_path` **acepta comodines** (`/ruta/*`) | VERIFICADO | — |
| El plan de cuentas **hay que cargarlo explícitamente** | VERIFICADO | Ejecución |

El de `http_interface` merece atención: un Odoo en contenedor arranca **sin un
solo error en el log** y queda inalcanzable. Hay que abrirlo dentro del
contenedor y limitar la exposición en el proxy.

---

## Reportes PDF

Si su entorno usa `wkhtmltopdf` de los repositorios de Ubuntu 24.04, va **sin el
Qt parcheado**, con dos consecuencias, ambas silenciosas:

1. **Exige un display.** Sin él falla con `QPainter::begin(): Returned false`,
   **devuelve código de salida 0** y no escribe el archivo. Se resuelve
   envolviéndolo con `xvfb`.
2. **No admite encabezados ni pies de página.** Con cualquier opción
   `--header-*` o `--footer-*` termina sin generar nada, también con código 0.

Afecta a **todos** los reportes PDF de Odoo, no solo a los propios.

Además, v20 incorpora `base_report_paper_muncher`, el motor de render propio de
Odoo, que no existía en v18. Conviene decidir con qué motor se generan los
reportes antes de invertir tiempo en ajustarlos.

---

## Lo que más nos costó encontrar

Si tuviéramos que señalar los cinco que más tiempo consumieron, y por qué:

1. **La versión del manifest.** El modo de fallo parece un éxito: código de
   salida 0 y nada instalado.
2. **La API de consultas SQL.** El módulo instala perfecto y revienta al generar
   el reporte. Nada en la instalación toca ese camino.
3. **`is_company` calculado.** No falla nunca. Solo cambia una clasificación de
   la que dependen cálculos fiscales.
4. **Los anclajes de vistas movidos.** Individualmente triviales, pero son
   muchos y hay que resolverlos de a uno.
5. **Las dependencias no declaradas.** El módulo funciona en un entorno donde
   «casualmente» está todo instalado, y falla en otro.

Los tres primeros no los encuentra una instalación limpia. Los encuentra una
prueba que recorra ese camino, o un usuario en producción.
