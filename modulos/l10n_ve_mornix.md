# l10n_ve_mornix — Localización venezolana

> Módulo piloto de la migración a v20. Estado: **instala, actualiza y pasa sus
> 213 pruebas sin errores ni advertencias**.
> Origen: `nx-desarrollo/nx_localizacion`, rama `main`, versión `18.0.0.11.0`.
> Destino: `addons/localizacion/l10n_ve_mornix`, versión `1.34.0` (Odoo la
> prefija con la serie vigente → `19.5.1.34.0`).
>
> Los nombres **de módulo** pasaron de `nimetrix` a `mornix`. Los nombres
> **técnicos de los modelos** (`nimetrix.fiscal.book`, `nimetrix.wh.iva`…) se
> conservan a propósito: renombrarlos obligaría a migrar datos en las bases de
> los clientes sin ganar nada funcional.

## 1. Qué hace

Localización fiscal venezolana. Cubre:

- Retenciones de IVA (cliente y proveedor), comprobante y TXT para el SENIAT
- Retenciones de ISLR (cliente y proveedor), comprobante y XML
- Retenciones municipales
- Libros fiscales de compras y ventas
- ARC (comprobante anual de retenciones a empleados)
- Unidad tributaria y su histórico
- Numeración de control de facturas
- Listas de precios con IVA y en divisa

Es la **capa B** del modelo en capas (ver skill `modulo-mornix`): lo que
comparten todas las implantaciones venezolanas. De este módulo cuelga casi todo
lo demás del cliente.

## 2. Tamaño

| | |
|---|---|
| Python | 50 archivos · 9.388 líneas |
| XML | 48 archivos · 5.358 líneas |
| JavaScript | 0 — **no toca el frontend** |
| Modelos propios | 24 |
| Modelos que extiende | 12 |
| Reglas de acceso | 27 |
| Pruebas | 173 (17 archivos) — 30 heredadas, 143 escritas en la migración |

Que no tenga JavaScript es la razón por la que este módulo, siendo el más
grande, no fue el más difícil de migrar: OWL es lo que más rompe entre v16 y
v20, y aquí no aplica.

## 2.1 ~~Dependencia no declarada~~ — RESUELTA en 1.24.0

**El ciclo desapareció.** Durante casi toda la migración, el libro fiscal
dependía de `mornix_dual_currency` sin declararlo, y no se podía declarar:

```
l10n_ve_mornix  →  mornix_dual_currency  →  mornix_currency_rate
                                                   →  l10n_ve_mornix
```

Odoo se habría negado a instalar. El vínculo era **una sola línea**: el SQL del
libro leía `account_move.amount_exempt_bs`, campo de ese módulo, para la columna
de ventas exentas.

Desde que el desglose de IVA vive en la propia factura (§6.4), el libro usa su
campo `nx_base_exenta` y no necesita nada de fuera.

**Verificado instalando `l10n_ve_mornix` solo en una base limpia**: los otros dos
módulos quedan `uninstalled`, `amount_exempt_bs` no existe, y el libro fiscal
funciona. Y el libro produce **exactamente las mismas cifras** que antes del
cambio, comparadas fila a fila sobre 53 facturas.

Las demás referencias de este módulo a campos de `dual_currency` (`nx_rate`,
`nx_currency_ref_rate_fixed`) ya estaban protegidas con `'campo' in _fields`, así
que funcionan con o sin el módulo instalado.

- [x] ~~Decidir cómo romper el ciclo~~ — hecho: bajar el dato a este módulo, que
      era la salida recomendada.

### Un riesgo que apareció al hacerlo

El desglose solo cuenta impuestos marcados como IVA (`nx_type_tax`) **y** con
tipo de alícuota (`nx_appl_type`). Un impuesto sin clasificar **aporta cero al
libro, sin avisar**: la factura sale listada con las columnas vacías.

Detectado sobre datos reales: una factura con el impuesto «22 %» del plan
genérico, sin clasificar, salía en cero.

`action_confirm_check()` lo comprueba antes de confirmar el libro y lista las
facturas afectadas. `nx_facturas_sin_clasificar()` las devuelve.

- [ ] **Al migrar la base del cliente**, clasificar todos los impuestos de IVA
      antes de generar el primer libro:
      `SELECT id, name FROM account_tax WHERE nx_appl_type IS NULL;`

## 3. Dependencias

```
base_vat · base_address_extended · l10n_ve · account_debit_note
sale · purchase · product
```

Las siete existen en v20. `l10n_ve` aporta el plan de cuentas venezolano, que
**hay que cargar explícitamente** en cada base: sin él no hay cuentas ni diarios
y nada contable funciona.

```bash
docker compose run --rm -T odoo20 odoo shell -d <base> --no-http <<'EOF'
env['account.chart.template'].try_loading('ve', env.ref('base.main_company'))
env.cr.commit()
EOF
```

## 4. Modelos

### 4.1 Retención de IVA

| Modelo | Archivo | Líneas | Responsabilidad |
|---|---|---:|---|
| `nimetrix.wh.iva` | `models/nimetrix_wh_iva.py` | 580 | Comprobante de retención de IVA: numeración, estados, totales |
| `nimetrix.wh.iva.line` | `models/nimetrix_wh_iva_line.py` | 115 | Línea por factura retenida. Restricción única sobre `nx_invoice_id` |
| `nimetrix.wh.iva.line.tax` | `models/nimetrix_wh_iva_line_tax.py` | 33 | Desglose de impuestos de la línea |
| `nimetrix.wh.iva.txt` | `models/nimetrix_wh_iva_txt.py` | 490 | Generación del TXT para el SENIAT |
| `nimetrix.wh.iva.libro.resumen` | `models/nimetrix_wh_iva_libro_resumen.py` | 699 | Resumen del libro de IVA y su PDF |

### 4.2 Retención de ISLR

| Modelo | Archivo | Líneas | Responsabilidad |
|---|---|---:|---|
| `nimetrix.wh.islr.doc` | `models/nimetrix_wh_islr_doc.py` | 415 | Comprobante de retención de ISLR |
| `nimetrix.wh.islr.doc.invoices` | `models/nimetrix_wh_islr_doc_invoices.py` | 728 | Cálculo por factura: concepto, tarifa, sustraendo |
| `nimetrix.wh.islr.doc.line` | `models/nimetrix_wh_islr_doc_line.py` | 64 | Línea de concepto retenido |
| `nimetrix.wh.islr.concept` | `models/nimetrix_wh_islr_concept.py` | 30 | Catálogo de conceptos ISLR |
| `nimetrix.wh.islr.rates` | `models/nimetrix_wh_islr_rates.py` | 59 | Tarifas por concepto y tipo de persona |
| `nimetrix.wh.islr.xml` | `models/nimetrix_wh_islr_xml.py` | 347 | XML de retenciones para el SENIAT |

**El cálculo de ISLR depende del tipo de persona** (natural/jurídica,
residente/no residente). Ver la sección 6: ahí está el punto más delicado de
esta migración.

### 4.3 Libros fiscales y unidad tributaria

| Modelo | Archivo | Líneas | Responsabilidad |
|---|---|---:|---|
| `nimetrix.fiscal.book` | `models/nimetrix_fiscal_book.py` | 666 | Libro de compras y de ventas, exportación a Excel |
| `nimetrix.account.ut` | `models/nimetrix_account_ut.py` | 71 | Unidad tributaria vigente e histórico |
| `nimetrix.move.line.resumen` | `models/nimetrix_move_line_resumen.py` | 42 | Resumen de asientos para el libro |

### 4.4 Modelos del core que extiende

| Modelo | Archivo | Líneas | Qué agrega |
|---|---|---:|---|
| `account.move` | `models/account_move.py` | 1.318 | Número de control, retenciones asociadas, montos exentos, IGTF |
| `res.partner` | `models/res_partner.py` | 325 | RIF (`vat` y sus dos mitades), tipo de persona, validaciones |
| `product.pricelist.item` | `models/product_pricelist_item.py` | 248 | Precio con IVA y en divisa |
| `account.move.reversal` | `models/account_move_reserval.py` | 116 | Nota de crédito con número de control |
| `res.company` | `models/res_company.py` | 122 | RIF (`vat`), firma del representante, validaciones configurables |
| `product.template` / `product.product` | | 109 / 86 | Precios con IVA y en divisa |
| `account.payment` | `models/account_payment.py` | 99 | Validaciones de RIF en el pago |
| `account.tax` | `models/account_tax.py` | 90 | Marcado de impuestos de retención |
| `sale.order` / `purchase.order` | | 78 / 70 | RIF del contacto en el pedido |

## 5. Asistentes y reportes

| Asistente | Líneas | Para qué |
|---|---:|---|
| `nimetrix.wh.islr.list` | 344 | Listado de retenciones ISLR en Excel |
| `nimetrix.employee.income.wh_islr` | 171 | Carga masiva de ingresos de empleados (ARC) desde CSV |
| `nimetrix.wh.iva.list` | 125 | Listado de retenciones IVA |
| `nimetrix.change.invoice.sin_cred` | 34 | Marcar factura sin derecho a crédito fiscal |
| `nimetrix.wiz.nroctrl` | 27 | Asignación de número de control |
| `nimetrix.fiscal.book.confirm` | 22 | Confirmación del libro fiscal |

Reportes en `report/`: comprobante de retención de IVA, comprobante de ISLR,
listados de ambos, y el libro fiscal.

## 6. Lo que cambió al migrar a v20

Todas las roturas están documentadas en detalle en
[../BREAKING-CHANGES.md](../BREAKING-CHANGES.md). Aquí lo específico de este
módulo, ordenado por riesgo para el negocio.

### 6.1 `company_type` desapareció — riesgo alto

v20 eliminó `res.partner.company_type` y convirtió `is_company` en un campo
**calculado y almacenado**:

```python
is_company = (commercial_partner_id == self) and not _is_vat_void(vat)
```

Esa heurística clasifica como empresa a cualquier contacto con RIF. En Venezuela
una persona natural **también** tiene RIF, así que quedaría mal clasificada — y
de esa clasificación depende la tarifa de retención de ISLR.

**Qué se hizo:** en `models/res_partner.py` se repone `company_type` como campo
de interfaz con la misma definición que tenía en v18, y se devuelve a
`is_company` su carácter escribible (`readonly=False`). Así el valor que el
usuario fijó se respeta y la heurística de Odoo queda solo como valor inicial.

- [ ] **Verificar con datos reales**: al restaurar una base del cliente, revisar
      que ningún contacto cambie de tipo. Un proveedor que pase de natural a
      jurídica cambia su retención.

### 6.2 Normalización del RIF — riesgo controlado, pero frágil

v20 normaliza el campo `vat` con `stdnum` y le quita los separadores:
`J-99800001-6` → `J998000016`. En v18 no pasaba, porque `_fix_vat_number`
comparaba el prefijo del vat contra el código de país, no coincidía para
Venezuela y devolvía el valor intacto.

**En el estado final, esto NO afecta al módulo.** Verificado en la base odoo20:
el `vat` conserva sus guiones (`'J-98765432-1'` se guarda tal cual). La razón es que el override de
`_check_vat` exime a Venezuela del chequeo estándar, y es precisamente
`_check_vat` quien escribe de vuelta el valor normalizado.

Sí golpeó **durante** la migración: mientras el override estaba roto (llamaba a
`super().check_vat()`, que ya no existe), Odoo normalizó los RIF y las
validaciones del módulo —que exigían los guiones— rechazaban el valor que el
propio Odoo acababa de escribir. Por eso `nx_validate_rif` y
`nx_validate_rif_er` ahora normalizan antes de comparar: toleran ambos formatos.

El equilibrio es frágil y depende de un override. `tests/test_seniat_rif_formato.py`
lo fija: si alguien quita la exención, o si Odoo mueve la normalización a otro
punto del ciclo, esos tests fallan antes de que salga un TXT mal formado.

- [ ] Si en la base del cliente quedó algún RIF normalizado de un intento
      previo, hay que detectarlo: `vat NOT LIKE '%-%'`. No bloquea nada —las
      validaciones toleran ambos formatos— pero conviene saber cuántos son.

### 6.2.2 El identificador se consolidó en `vat` (versión 1.2.0)

v18 guardaba el mismo dato en tres sitios:

| Campo | Qué guardaba |
|---|---|
| `vat` | el estándar de Odoo, a menudo vacío |
| `nx_rif` | copia con formato legible, sincronizada a mano en los dos sentidos |
| `nx_identification_id` | la cédula, usada como identificador de reserva |

Los reportes al SENIAT elegían entre ellos repitiendo en cada sitio la forma
`nx_rif or (nx_nationality + nx_identification_id)`.

**Desde 1.2.0 hay un solo campo: `vat`.** El tipo de documento
(`nx_nationality`) y el número (`nx_document_number`) son sus **dos mitades
calculadas**: se escriben por separado en el formulario y el modelo las
concatena; escribir directamente en `vat` las rellena a la inversa.

```
vat = 'J-98765432-1'   <->   tipo 'J' + número '98765432-1'
```

Por qué importa, más allá de la limpieza: `same_vat_partner_id`, la detección de
duplicados, el portal y cualquier módulo de terceros consultan `vat`. Con el
identificador en un campo propio, ninguno de ellos lo veía.

Efectos colaterales del cambio, todos verificados:

- `same_vat_partner_id` ya **no se redefine**: el cálculo del core sirve tal cual
  y además cubre casos que la copia local no contemplaba (registros archivados,
  jerarquía de contactos, multi-compañía).
- La unicidad del RIF pasó a ser una sola constraint sobre `vat`. Sigue
  comparando cadenas exactas, así que **el mismo RIF cargado con y sin guiones
  no se detecta como duplicado** — igual que en v18.
- `nx_documento_seniat()` centraliza el formato sin separadores que piden los
  TXT y XML. Antes cada reporte lo resolvía por su cuenta.
- El formato aceptado se amplió para admitir el identificador derivado de la
  cédula (`V` + 7 u 8 dígitos), que es lo que la migración compone para los
  contactos que solo tenían cédula. Sin eso quedarían imposibles de guardar.

### 6.2.3 Campo eliminado: "Declaración legal de IVA" (versión 1.3.0)

`nx_vat_subjected` se declaraba y se pintaba en la pestaña de Retenciones, pero
**ningún cálculo lo leía** — ni este módulo, ni sus reportes, ni los repos v16 y
v18 del cliente. Su ayuda prometía que se usaría para la declaración del IVA;
nunca llegó a usarse. Eliminado.

Se auditaron a la vez los otros campos de esa pestaña. Los tres restantes **sí
se usan**, y uno de ellos es la razón por la que conviene auditar antes de
borrar:

| Campo | Consumidor |
|---|---|
| `nx_islr_withholding_agent` | `account_move.py` decide si aplica la retención; filtra el desplegable del comprobante |
| `nx_islr_exempt` | `islr_doc_invoices.py` → `apply_income = not vendor.nx_islr_exempt` |
| `nx_contribuyente_seniat` | **Ningún consumidor en este módulo.** Lo usa `nimetrix_iva_resumen_report` (v18, sin migrar): con `'especial'` el libro resumen calcula el período **quincenal** en vez de mensual |

`nx_contribuyente_seniat` parecía muerto mirando solo este módulo. Es el mismo
patrón de las 42 dependencias no declaradas: **el módulo no es la unidad de
análisis correcta.** Antes de eliminar un campo hay que buscarlo en todos los
repos del cliente, incluidos los que aún no se han migrado.

**Se eliminó igualmente en 1.5.0, por decisión del cliente**, sabiendo lo
anterior. Ver 6.2.5.

### 6.2.4 Se retiraron las dos excepciones de ISLR (versión 1.4.0)

**Cambio de comportamiento fiscal, decidido por el cliente. No es una limpieza.**

Se eliminaron `nx_islr_withholding_agent` y `nx_islr_exempt`, que eran las dos
únicas formas de **no** retener ISLR:

| Campo | Qué permitía | Qué pasa ahora |
|---|---|---|
| `nx_islr_withholding_agent` | Que una compañía no actuara como agente de retención | La compañía **siempre** retiene; queda solo la condición del tipo de documento |
| `nx_islr_exempt` | Que un proveedor concreto quedara exento | **Ningún proveedor está exento**; `apply_income` arranca en `True` |

Un proveedor que hoy figure como exento pasará a que se le retenga.

`migrations/1.4.0/pre-quitar-flags-islr.py` **lista por nombre** los contactos
exentos y cuenta las compañías que no eran agentes, antes de que Odoo elimine
las columnas. Si esa cuenta no es cero al migrar la base del cliente, hay que
revisarla antes de emitir retenciones. En la base de pruebas dio cero en ambas.

- [ ] **Al migrar la base real, leer ese log.** Es la única oportunidad: después
      del update las columnas ya no existen.

Efecto colateral: `_nx_get_partners` devolvía una tupla de tres, y el tercer
elemento era ese flag. **Ninguno de los tres sitios que la desempaquetaban lo
usaba** — ya era código muerto antes del cambio. Ahora devuelve dos.

### 6.2.5 Se eliminó el tipo de contribuyente SENIAT (versión 1.5.0)

Decisión del cliente, tomada sabiendo que el campo **sí tiene un consumidor**.

`nx_contribuyente_seniat` (ordinario / especial / formal / gubernamental) no lo
leía ningún cálculo de este módulo, pero sí lo hace
**`nimetrix_iva_resumen_report`** (repo `nx_tools`, v18, todavía sin migrar), en
`prior_period_dates`: con el valor `'especial'` el libro resumen de IVA calcula
el período **quincenal** en vez de mensual.

> **Al migrar `nimetrix_iva_resumen_report` hay que decidir de dónde sale ese
> dato.** Si no, su libro resumen calculará siempre el período mensual, que para
> un contribuyente especial es el equivocado.

`migrations/1.5.0/pre-quitar-contribuyente.py` vuelca el reparto de valores al
log y lista por nombre los contribuyentes especiales antes de que la columna
desaparezca. En la base de pruebas: 2 contactos, ambos `ordinario`.

- [ ] **Al migrar la base real, leer ese log** y llevar la lista de especiales a
      quien vaya a migrar `nimetrix_iva_resumen_report`.

Con esto la pestaña Retenciones queda solo con los dos campos de IVA
(`nx_wh_iva_agent` y `nx_wh_iva_rate`), que sí alimentan el cálculo de la
retención y el comprobante.

- [ ] **Un cambio de comportamiento a confirmar**: en el XML de ISLR,
      `nx_onchange_partner_id` devolvía `nx_rif[2:]`, que sobre un RIF con
      guiones dejaba `'12345678-9'`. Sobre el documento normalizado equivale a
      quitar solo la letra, y ahora devuelve `'123456789'`. Hay que verificar
      cuál espera el SENIAT.

### 6.2.1 El dígito verificador del RIF no se valida

Consecuencia de la misma exención: `base_vat` valida el dígito verificador, pero
Venezuela queda fuera. El módulo solo comprueba el **formato** con su propia
regex. Un RIF con el dígito mal entra sin protestar — verificado.

No es una rotura de la migración, es cómo estaba diseñado. Queda documentado
porque no es obvio.

- [ ] Confirmar con el cliente si se espera esa permisividad.

### 6.3 La validación del vat cambió de sitio

Ya no es la constraint `check_vat` de `base_vat`, sino el inverse del campo:
`vat` → `_inverse_vat` → `_check_vat` → `_run_vat_checks`. El override del
módulo llamaba a `super().check_vat()` y lanzaba `AttributeError`.

**Qué se hizo:** el override pasa a `_check_vat`. De paso se corrigió que el
`return` dentro del bucle hacía que el primer registro decidiera por todo el
conjunto; ahora filtra registro a registro, que es lo que el código pretendía.

### 6.4 Seguridad reescrita

`ir.model.access` e `ir.rule` se fundieron en `ir.access`. Los 24 accesos y las
4 reglas multi-compañía del módulo pasaron a un único `security/ir.access.csv`,
generado con `scripts/migrar_seguridad.py`.

### 6.5 Funcionalidad perdida

- **Contexto `default_admin_note` en los recibos.** v20 eliminó las acciones
  `account.action_move_out_receipt_type` y `action_move_in_receipt_type`. Los
  tipos `out_receipt`/`in_receipt` siguen existiendo, pero sin punto de entrada.
  - [ ] Confirmar con el cliente si usa recibos. Si los usa, hay que crear una
        acción propia en la capa C que reponga ese contexto.

### 6.6 Código muerto encontrado

Cosas que ya estaban rotas en v18 y que la migración sacó a la luz:

- **4 métodos `name_get()`** en `nimetrix_wh_islr_doc`, `nimetrix_wh_iva`,
  `nimetrix_wh_islr_xml` y `nimetrix_wh_iva_txt`. Odoo dejó de llamarlos en
  v17: esos modelos llevan desde entonces mostrando mal su nombre.
  Además, el de `nimetrix_wh_islr_doc` itera `for item in self.browse()`, que
  sobre un recordset vacío no ejecuta nada.
  - [ ] Reemplazar por `_compute_display_name` — requiere decidir con el cliente
        qué debe mostrar cada uno, así que no se toca en la migración.
- **Imports muertos de `decimal_precision`** en 4 archivos. Ese módulo no existe
  ni en v18: el import fallaba también allí. Se comprobó contra la imagen
  oficial `odoo:18`.
- **`from odoo import sys`**, que funcionaba por accidente porque el
  `__init__.py` de Odoo importaba `sys`.

Que esos imports rompan en v18 significa que **el código de la rama `main` de
`nx_localizacion` no es el que corre en producción**.

- [ ] Aclarar con el equipo qué rama refleja producción.

### 6.3 El Libro Resumen de IVA nunca habia funcionado (versión 1.8.0 y 1.9.0)

**No es una rotura de la migración.** Se comprobó el mismo defecto en v16, v18 y
v20: es el mismo archivo arrastrado entre versiones con el prefijo del módulo
cambiado. El botón se detenía en el primero de **cinco** problemas, y ninguno
daba error al instalar:

| # | Qué pasaba | Dónde |
|---|---|---|
| 1 | Las consultas pedían los campos del resumen **sin el prefijo `nx_`** (`fecha_fact`, `type`, `state`…). 81 referencias | v16, v18, v20 |
| 2 | Filtraba por `state == 'confirmed'`, estado que `account.move` **nunca ha tenido**: devolvía vacío y el libro salía en cero | v16, v18, v20 |
| 3 | Llamaba a `self.line.formato_fecha2(…)`, y `line` era un campo **comentado** | v16, v18, v20 |
| 4 | Componía el RIF con `res.partner.doc_type`, que no existe en este módulo | v16, v18, v20 |
| 5 | `base64.encodestring` (eliminado en Python 3.9) y escribía `bytes` en un campo Binary, que v20 rechaza | solo v20 |

Los cuatro primeros venían de antes; solo el quinto lo introdujo v20.

Ahora genera: **13,8 KB, 60 filas, firma OLE2 válida**. `tests/test_libro_resumen_iva.py`
lo ejerce de punta a punta —pulsa el botón y abre el archivo— porque los cinco
fallos solo se veían al usarlo.

#### Lo que se eliminó

`get_invoice()` producía el **listado detallado factura por factura**. Tampoco
había funcionado nunca y **no lo llamaba nadie**: escribía 28 claves en
`nimetrix.wh.iva.libro.pdf.resu`, un modelo que no existe —el declarado era
`…pdf.resumen`— y que además solo tenía un campo, `name`. De las 28 claves,
**27 no existían en ningún sitio**.

Se eliminó, junto con lo que solo existía para él:

| Eliminado | Por qué |
|---|---|
| `get_invoice()` | Sin llamadas. Escribía en un modelo inexistente |
| Modelo `nimetrix.wh.iva.libro.pdf.resumen` | Solo lo usaba `get_invoice`. Un campo, **0 filas** en la base |
| `doc_cedula`, `doc_cedula2` | Solo los usaba `get_invoice`. Leían `res.partner.doc_type`, que no existe |
| `float_format`, `float_format2` | Sin llamadas |
| Su línea en `ir.access.csv` | El modelo ya no existe |
| 9 imports | Quedaron sin uso: `api`, `tools`, `UserError`, `io`, `xlsxwriter`, `shutil`, `csv`, `logging`, `DEFAULT_SERVER_DATE_FORMAT` |

`migrations/1.9.0/post-borrar-tabla-pdf-resumen.py` suelta la tabla huérfana que
Odoo deja al desaparecer un modelo. **Cuenta las filas antes de borrar**: si en
alguna base del cliente alguien llegó a meter datos, conserva la tabla y avisa
en el log.

El archivo pasó de 699 a 536 líneas. El Excel sale idéntico —13,8 KB, 60 filas—,
verificado antes y después de borrar.

- [ ] **Si el cliente necesita ese listado detallado**, hay que especificarlo de
      cero: qué columnas lleva y de dónde sale cada una. El código eliminado
      está en el historial de git y en `legacy/`, pero no sirve de referencia
      funcional: nunca produjo una fila.
- [ ] Confirmar que el filtro correcto es `state = 'posted'`. Es lo único que
      tiene sentido para un libro fiscal, pero determina qué facturas entran.

#### `nx_llenar` es peligroso

Es el método que puebla el resumen. No lo llama nada, y empieza con
`search([])` → `unlink()`: **borra el resumen de todas las facturas de todas las
compañías** y lo reconstruye recorriendo cada asiento publicado. Ejecutarlo en
producción reescribe el histórico completo del libro fiscal.

- [ ] Acotarlo por compañía y por rango de fechas antes de exponerlo en la
      interfaz.


### 6.4 El desglose de IVA pasó a la factura (versión 1.11.0)

`nimetrix.move.line.resumen` guardaba el desglose por alícuota en un modelo
aparte. Ahora son **campos calculados de `account.move`**:

```
nx_base_exenta
nx_base_general      nx_iva_general
nx_base_reducida     nx_iva_reducida
nx_base_adicional    nx_iva_adicional
nx_base_imponible    nx_iva_total
nx_total_documento
```

**Siempre en bolívares**, facture la factura en la moneda que facture. Los libros
y las retenciones se presentan al SENIAT en la moneda de la compañía: un desglose
en divisa no sirve para declarar, y obligaba a convertir en cada consumidor —con
el riesgo de convertir dos veces.

Por eso se calcula sobre `balance` y no sobre `price_subtotal` ni
`amount_currency`, que están en la moneda de la factura. Comprobado sobre una
factura de 100 USD a 750 Bs: `amount_currency = -100`, `balance = -75.000`.

Consecuencia directa: **`conv_div_nac` desapareció del Libro Resumen.** Convertía
los importes del resumen a bolívares; ahora ya vienen así, y aplicarla sería
convertir dos veces.

`nx_total_documento` es el total a efectos fiscales: bases gravadas + exento +
IVA. **No es `amount_total`**: deja fuera lo que no es IVA, como el IGTF.

**Los cinco motivos, todos verificados sobre la base:**

| # | Problema de la tabla |
|---|---|
| 1 | Era **1:1 disfrazada de 1:N**: el código siempre escribía `alicuota_line_ids[0]` |
| 2 | Solo se llenaba al pulsar **"Crear Retención"**, no al publicar. Una venta sin retención **no entraba nunca** en el Libro Resumen |
| 3 | Duplicaba estado, tipo y fechas de la factura; copias que se desincronizaban |
| 4 | Emparejaba por `tax_group_id`: dos alícuotas con el mismo grupo **mezclaban sus montos** |
| 5 | Era una foto: editar la factura después no la actualizaba |

De dónde sale cada cifra, comprobado sobre una factura de 4 líneas de 1.000:

- **base** → `price_subtotal` de las líneas de producto, por el `nx_appl_type` de
  su impuesto de IVA
- **monto** → `amount_currency` de las líneas de impuesto, por el `nx_appl_type`
  de su `tax_line_id`

El exento no genera línea de impuesto (0 %), por eso solo aporta base.

#### Se verificó que los números no cambian

Antes de sustituir nada se compararon los nueve valores sobre la misma factura,
con la tabla vieja poblada:

```
base exenta 1000=1000 · base gral 1000=1000 · IVA gral 160=160
base red 1000=1000 · IVA red 80=80 · base adic 1000=1000 · IVA adic 310=310
base imponible 3000=3000 · IVA total 550=550
```

`migrations/1.11.0/pre-comparar-y-soltar-resumen.py` repite esa comparación
**fila a fila sobre la base del cliente** antes de soltar la tabla, y registra en
el log cualquier factura donde no coincidan. Si aparecen diferencias, la causa
habitual es el defecto 4 —alícuotas compartiendo grupo— y **el valor bueno es el
nuevo**.

#### Un cambio de comportamiento que sí altera cifras

El Libro Resumen pasa a incluir **todas las facturas publicadas** del período, no
solo las que tuvieran retención. Es lo correcto para un libro de IVA —hasta ahora
declaraba de menos— pero **los débitos y créditos fiscales van a subir**.

Se añadió además el filtro por compañía, que la versión anterior no tenía.

- [ ] Al migrar la base real, comparar el Libro Resumen del último período contra
      el presentado, y explicar la diferencia antes de declarar.


### 6.5 `nx_sin_cred` decía una cosa y hacía otra (versión 1.12.0)

El campo se mostraba como **"Excluir este documento del libro fiscal"**. Es
falso: el libro fiscal no lo mira.

| Dónde | Qué decía |
|---|---|
| El formulario de la factura | "Excluir este documento del libro fiscal" |
| El asistente | "Exento de impuestos" |
| Su propia ayuda | "si la factura está exenta de IVA" |

Tres nombres distintos, y el visible era el equivocado. **Lo que hace de verdad**
es que `nx_check_wh_apply()` devuelva `False`, o sea: **que a esa factura no se
le genere la retención de IVA**.

El filtro que habría justificado la etiqueta está **comentado desde v16** en
`nimetrix_fiscal_book.py`, y además usaba el nombre viejo del campo (`sin_cred`,
sin prefijo), así que tampoco habría funcionado. La exclusión del libro se
configura con `nx_excluded_fiscal_position_ids`.

Se unificó la etiqueta en los tres sitios: **"Excluir de la retención de IVA"**,
con una ayuda que aclara que no afecta a los libros. **No cambia ningún
comportamiento**: solo deja de engañar a quien lo marca.

El nombre técnico se conserva (`nx_sin_cred`, de *sin crédito fiscal*) para no
migrar el dato.

- [ ] Revisar con el cliente qué facturas tienen hoy la casilla marcada. Si
      alguien la marcó creyendo que excluía del libro fiscal, esa factura lleva
      tiempo **sin retención de IVA** sin que nadie lo pretendiera.


### 6.6 El número de comprobante de ISLR se trunca y sale igual en todos

**Encontrado al validar el flujo completo con `scripts/generar_flujo_demo.py`.**

`nimetrix.wh.islr.doc.name` —"Número de Comprobante"— está declarado con
`size=14`. La secuencia que lo alimenta produce 19 caracteres:

```
secuencia  RE/ISLR/%(year)s/ + padding 6  ->  'RE/ISLR/2026/000125'   (19)
guardado                                  ->  'RE/ISLR/2026/0'        (14)
```

El correlativo queda cortado y **todos los comprobantes se llaman igual**.
Medido sobre la base: **16 comprobantes, 1 solo nombre distinto**.

Viene de v16: el prefijo y el tamaño del campo nunca fueron compatibles.

El correlativo fiscal de verdad es otro campo, `nx_number`, y ese **sí funciona**:
sale `00000100`, `00000101`… correlativo y único. Solo se asigna al cerrar el
comprobante (`nx_action_done`), no al confirmarlo.

Para comparar, el comprobante de IVA usa el formato del SENIAT —`AAAAMM` + 8
dígitos = 14 caracteres— y encaja exacto: `20260600000005`.

**Resuelto (1.15.0)**: el cliente confirmó que lleva el mismo formato del
SENIAT que el IVA. La secuencia pasó a `%(year)s%(month)s` con padding 8 —14
caracteres exactos— y el mes ya se sustituye con cero a la izquierda. Verificado
sobre el flujo completo: `20260800000130`, `…131`, `…132`, `…133`, únicos.

`migrations/1.15.0` actualiza la secuencia en las bases existentes (el registro
vive en un `noupdate="1"`) y **cuenta los comprobantes que hoy comparten
nombre**, sin renumerarlos: reescribir el número de un documento fiscal ya
presentado no es cosa de una migración.

- [ ] **Decisión del cliente**: qué se hace con los comprobantes de ISLR ya
      emitidos que comparten nombre. La migración los lista en el log.

#### Otra cosa que el flujo dejó a la vista

`nx_retencion_seq_get()` hace cirugía posicional sobre la cadena para sustituir
el mes:

```python
if not account_month == local_number[4:6]:
    local_number = local_number[:4] + account_month + local_number[6:]
```

Eso asume que los caracteres 4 y 5 son el mes, cierto para `AAAAMM…` pero no
para `RE/ISLR/…`, donde son `SL`. Con el prefijo actual corrompe la cadena en
vez de corregirla. Se resuelve solo si se adopta el formato del SENIAT.


### 6.7 El XML de ISLR limpia unos campos y otros no

Verificado generando el archivo completo (11.623 bytes, 36 detalles):

| Campo | Qué sale | Cómo lo trata el código |
|---|---|---|
| `RifRetenido` | `V400000038` | `.replace("-", "")` |
| `NumeroControl` | `CTRL000007` | `.replace("-", "")` y recorta a 10 |
| `NumeroFactura` | `F-000007` | **`[-10:]` a secas, conserva el guion** |

Los tres son identificadores del mismo documento y solo dos se normalizan. No es
una rotura de la migración —viene así de v16— pero el archivo se presenta al
SENIAT.

- [ ] **Confirmar con el cliente** si el SENIAT acepta el número de factura con
      separadores. Si no, falta el `.replace("-", "")`, igual que en los otros
      dos campos.

### 6.8 Números de control ausentes en datos migrados

Al generar el XML sobre los datos de prueba aparecieron **48 líneas sin número
de control**, procedentes de 12 facturas de proveedor cargadas sin él. El XML
ahora las lista en vez de reventar (ver 6.6).

Importa para la migración real: **el número de control de una factura de
proveedor no se puede inventar** —viene impreso en el documento del proveedor—.
Si la base de origen lo tiene vacío en facturas ya retenidas, hay que
recuperarlo del sistema anterior antes de poder declarar ese período.

- [ ] Medir en la base del cliente cuántas facturas de proveedor con retención
      de ISLR no tienen número de control:
      `SELECT COUNT(*) FROM account_move WHERE move_type IN ('in_invoice','in_refund')
       AND state='posted' AND COALESCE(nx_nro_ctrl,'')='';`


### 6.9 Bloqueo de lo ya declarado al SENIAT (versión 1.20.0)

Un documento que entró en un archivo o libro presentado **no se toca**: ni se
anula, ni se devuelve a borrador, ni se le cambian importes o fechas.

| Documento presentado | Bloquea |
|---|---|
| TXT de retención de IVA | los comprobantes de retención de IVA que lista |
| XML de retención de ISLR | los comprobantes de ISLR que lista |
| Libro fiscal de compras | las facturas de proveedor que incluyó |
| Libro fiscal de ventas | las facturas de cliente que incluyó |

La lógica común vive en el mixin `nimetrix.bloqueo.fiscal`. Cada modelo aporta
solo un método: dónde buscar al declarante.

Además del bloqueo hay un **aviso visible** en el formulario, para que el
usuario lo sepa al abrir y no al recibir un error:

> 🔒 **Bloqueado por declaración.** Este documento ya está incluido en
> «Libro compras junio» y no puede modificarse.

#### Por inclusión, no por rango de fechas

La primera versión buscaba al declarante por fechas: «¿hay un libro confirmado
que cubra la fecha de esta factura?». **Las pruebas la tumbaron**, y con razón:
un comprobante creado después de presentar el archivo de su período nacía
bloqueado, sin poder ni confirmarse.

Ahora se pregunta si el documento **estuvo realmente** en la declaración:

- TXT y XML ya guardaban sus líneas; se consulta ese vínculo.
- El libro fiscal no guardaba nada —es un reporte SQL—, así que se le añadió
  `nx_declared_move_ids`, que se llena al confirmarlo y se vacía al devolverlo
  a borrador o cancelarlo.

Una factura cargada más tarde en un período ya declarado **no queda bloqueada**:
no estuvo en el libro, y decir lo contrario sería mentir sobre lo presentado.

#### Lo que deliberadamente NO bloquea

`account.move` limita el bloqueo a los campos que salen impresos en el libro.
**`line_ids` queda fuera a propósito**: registrar un pago escribe ahí, y cobrar
una factura ya declarada tiene que seguir siendo posible — no altera lo
declarado. Tampoco se bloquea el chatter.

Un bloqueo que impida cobrar sería peor que no tenerlo: la gente pediría
desactivarlo.


### 6.10 Saldos por cobrar y por pagar en divisa (versión 1.26.0)

Dos botones en el contacto, junto a Ventas y Facturado, con el saldo en la
moneda referencial. Al pulsarlos abren los documentos que lo componen.

#### De dónde sale la cifra

De los **apuntes contables**, no de las facturas: `amount_residual` vive ahí, y
así entran también los asientos manuales sobre las cuentas por cobrar y por
pagar, que una suma de facturas dejaría fuera.

Hay dos campos y según el caso aplica uno:

| Caso | Campo |
|---|---|
| El apunte ya está en la moneda referencial | `amount_residual_currency` — se suma **tal cual** |
| Está en otra moneda | `amount_residual` (moneda de la compañía), convertido a la tasa de la fecha del apunte |

Lo que ya está en divisa no se convierte: la cifra es exacta y convertirla solo
metería error de redondeo. Verificado sobre datos reales — `INV/2026/00017` está
en USD y aporta sus **122,00** directos, aunque su residual en bolívares sea
91.755,50.

#### Dos cosas que costaron

**No usa `nx_amount_residual_usd`** de `mornix_dual_currency`, que habría sido lo
inmediato: eso volvería a cerrar el ciclo roto en 1.24.0 (§2.1).

**`nx_currency_ref_id` no es un `related` a `company_id`.** Un contacto
normalmente **no tiene compañía** —se comparte entre todas—, así que el related
devolvía vacío y los saldos salían sin convertir. Se calcula cayendo a la
compañía activa. Hay una prueba que lo vigila.

Los botones parten de los **mismos apuntes** que calculan la cifra, así que la
lista y el número no pueden discrepar.


### 6.11 Libro de inventario integrado (versión 1.27.0)

Viene de `nx_localizacion/nimetrix_stock_account_report` (v18), ahora dentro de
este módulo. Asistente con filtro por producto o categoría, y PDF con existencia
inicial, entradas, salidas y existencia final valoradas.

**La rotura:** todo el reporte se apoyaba en `stock.valuation.layer`, modelo que
v20 eliminó. La valoración vive ahora en el propio `stock.move`.

Cuidado con `product.value`: por el nombre parece el sustituto y **no lo es** —
guarda las revaluaciones manuales, no cada movimiento. Migrar por ahí daría un
reporte casi vacío, sin error.

#### Tres defectos que venían de v18

| Qué pasaba | Consecuencia |
|---|---|
| La existencia final se buscaba con `create_date >= to_date` | Eran los movimientos **posteriores** al período. Y el resultado ni se usaba |
| Se filtraba por `create_date`, no por la fecha del movimiento | Un asiento cargado con retraso caía en el período equivocado |
| Ocho `print()` en el flujo del reporte | Ruido en el log de producción |

El módulo añade la dependencia `stock_account`, que antes no hacía falta.

- [ ] El asistente solo lista productos con movimientos en el período. Confirmar
      con el cliente si el libro debe incluir también los que no se movieron
      pero tienen existencia.


### 6.12 Resumen de Ventas y Compras, versión completa (1.28.0)

Viene de `nx_tools/nimetrix_iva_resumen_report`, un módulo aparte que **extendía
este mismo modelo** con un reporte más completo. Integrado aquí.

No eran dos reportes: eran **dos generaciones del mismo**. El que ya vivía en
`l10n_ve_mornix` era el borrador —su título decía literalmente
`Ventas Internas Gravadas por Alicuota General ---2`—. El del módulo suelto es
el terminado, y añade:

- Ventas internas por alícuota reducida (8 %)
- Ajustes a los débitos fiscales de períodos anteriores
- Certificados de débitos fiscales exonerados
- **Sección completa de créditos fiscales**, con compras de importación
- Crédito deducible por prorrateo
- Cuota tributaria del período y excedente para el mes siguiente

Pasa de 60 filas casi vacías a **42 con contenido real**.

#### Nunca había funcionado

`generate_xls_report` llamaba a `update_resume()`, `prior_period_dates()` y
`float_format2()`, todas definidas en `account.wizard.libro.resumen` —un modelo
**sin relación de herencia** con el que las invocaba—. Reventaba con
`AttributeError` en la primera línea. Es el mismo patrón del Libro Resumen (§6.6).

#### Cuatro adaptaciones

| Qué | Por qué |
|---|---|
| `prior_period_dates` se deduce del **rango pedido** | Antes lo sacaba de `nx_contribuyente_seniat`, campo eliminado. Un rango de ≤16 días es quincena; el resto, mes |
| `update_resume` queda vacío | Repoblaba `nimetrix.move.line.resumen`, tabla eliminada en 1.11.0 — y hacía un `cr.commit()` a media operación. El desglose ahora se calcula solo |
| `self.line.formato_fecha2()` → `self.formato_fecha2()` | `line` era un campo comentado desde v16 |
| `base64.b64encode(...)` → `.decode()`, y `nx_rif` → `vat` | Roturas de v20 y de la consolidación del identificador |

- [ ] **Antes de presentar**: comparar el resumen del último período contra el
      que se venga entregando. Las secciones nuevas hacen que aparezcan cifras
      que hasta ahora no salían.


### 6.13 Retención municipal integrada (versión 1.29.0)

Viene de `nx_localizacion/nimetrix_retencion_municipal` (v18): la retención del
impuesto sobre actividades económicas que practican (o nos practican) los
municipios. Nueve modelos, doce vistas, un reporte y un asistente, ahora dentro
de este módulo.

**Sin dependencia de `mornix_dual_currency`.** El módulo original la traía por
tres usos, todos sustituidos: `nx_currency_ref_id` ya es un campo de este
módulo, `amount_total_bs` pasa a `nx_total_documento`, y la conversión de la
base imponible usa el conversor estándar (`currency._convert`) en vez de leer
`nx_rate` a mano.

#### Qué hace el flujo

1. Se marca el concepto municipal en la línea de la factura
   (`account.move.line.concept_municipal_id`), con su alícuota y su alcaldía.
2. Al publicar la factura, si el contacto está sujeto a retención municipal,
   nace el comprobante (`retention.municipal`) con una línea por concepto.
3. El comprobante toma número de la secuencia del diario, genera su asiento y
   **se concilia contra la cuenta por cobrar o por pagar de la factura**.
4. Devolver la factura a borrador deshace comprobante y asiento.
5. El asistente saca el PDF del período, separando clientes de proveedores.

#### Ocho defectos que venían de v18

Ninguno es de v20: son fallos que el módulo arrastraba y que el flujo completo
saca a la luz.

| Qué pasaba | Consecuencia |
|---|---|
| `action_post` no escribía `state` en la rama del comprobante nuevo | La retención generaba su asiento y **se quedaba en «Borrador» para siempre**. Es la rama que corre siempre |
| El asiento no se conciliaba con la factura | El residual del cliente no bajaba: el importe quedaba abierto por los dos lados |
| Las líneas del asiento llevaban `move_id` **de la factura** | Apuntes colgados del documento equivocado |
| La base imponible era `abs(amount_untaxed)`, repetido en cada línea | Con dos conceptos municipales, **se retenía dos veces sobre el total de la factura** |
| Esa misma base no se convertía a moneda de compañía | Una factura de 100 USD declaraba 100 «bolívares» de base |
| `button_draft` borraba el asiento con `DELETE FROM account_move` en SQL crudo | Se saltaba el ORM, dejaba apuntes conciliados apuntando a un asiento inexistente, y sin retención componía `WHERE id = False` y reventaba |
| Nada iteraba: `action_post`, `post_retention_municipal`, `action_retention_municipal` | «Expected singleton» al validar varias facturas desde la lista, que es el uso normal |
| `action_cancel` marcaba la retención y dejaba el asiento **publicado** | El importe seguía contabilizado después de anular |

Y cuatro más de menor calado: el asistente del reporte era un `models.Model`
(cada apertura dejaba una fila permanente en la base), dos defaults se evaluaban
al importar el módulo (`datetime.now()` — todas las retenciones nacían con la
fecha de arranque del servidor), `_compute_all_amount` acumulaba fuera del bucle
y `_compute_rt_amount` hacía `+=` sobre su propio campo calculado.

#### La fecha de abono del reporte nunca salió

Buscaba los pagos con `account.payment.move_id == factura.id`. Pero `move_id`
es el asiento **del pago**, no la factura: la condición no se cumple nunca y la
columna salía siempre en blanco. Odoo relaciona ambos por conciliación, en
`account.move.matched_payment_ids`. Es el cuarto caso en este módulo de una
función que no funcionó en ninguna versión.

#### Dos defectos propios, encontrados al probar

El comprobante de retención lleva `municipal_id` como traza inversa, igual que
la factura. El hook de `button_draft` no los distinguía, así que poner el
comprobante en borrador —algo que hace la propia cancelación— **destruía la
retención de la que colgaba**. Se filtran los asientos de tipo `entry`.
Lo encontraron las pruebas automáticas.

El segundo solo salió al correr el flujo **sobre datos reales**: la base se
calculaba convirtiendo el subtotal con la tasa del día, mientras el asiento de
la factura estaba hecho a la tasa de la factura. Cuando no coinciden, el
asiento de retención **no concilia**: en la prueba, 100 conciliados de 1500 y
un residual de −1400 que no se corresponde con nada. Ahora la base sale del
`balance` de las líneas, que ya viene en moneda de compañía a la tasa correcta
—el mismo criterio del desglose de IVA (§6.4)—.

> Las pruebas automáticas no lo veían porque montan la tasa que van a usar. Un
> generador que trabaje sobre la base real vale para esto: encuentra lo que un
> fixture, por construcción, deja fuera.

#### Cómo verlo funcionando

```bash
cd docker
docker compose run --rm -T odoo20 odoo shell -d <base> --no-http \
    < scripts/generar_flujo_municipal.py
```

Monta alcaldía, conceptos, diario y contactos; genera cuatro facturas (un
concepto, dos conceptos, compra y divisa) y comprueba 24 cosas sobre los datos
ya puestos en la base, incluida la vuelta a borrador. Deja todo confirmado para
poder abrirlo en la interfaz.

#### Cambios de interfaz

- La pestaña «Retención municipal» de la factura, que solo mostraba un campo de
  solo lectura, pasa a **botón inteligente** junto a los de IVA e ISLR.
- La cabecera del comprobante comparaba el nombre con `'/'`, valor que este
  modelo nunca usa: el rótulo «Borrador» no aparecía jamás.
- Tres campos usaban `readonly="state == 'done'"`, y `done` no es un estado de
  este modelo: quedaban editables con la retención ya publicada.

#### Corregido de paso

El menú del **libro de inventario** (§6.11) cuelga de
`nimetrix_menu_venezuela_reporting`, pero su XML se cargaba antes que
`views/menu_items.xml`. La **instalación fresca** moría con «External ID not
found»; en una actualización no se notaba porque el menú ya existía. Reordenado
en el manifiesto.

- [ ] Los conceptos municipales y las alcaldías son datos maestros del cliente:
      hay que cargarlos antes de usar el módulo. No vienen en `data/`.
- [ ] Confirmar con el cliente si la base municipal debe calcularse sobre el
      subtotal de las líneas con concepto (lo que hace ahora) o sobre otra
      magnitud según la ordenanza de cada alcaldía.


### 6.14 Restricciones varias (versión 1.30.0)

Viene de `nx_localizacion/nimetrix_restrictions` (v18). Tres barreras: no borrar
asientos publicados, no cancelar facturas de otro período, y no instalar Studio
ni tocar vistas del sistema desde la interfaz.

| Qué pasaba en v18 | Consecuencia |
|---|---|
| `views/account_move_view.xml` **no estaba en el manifiesto** | El botón «Cancelar» de documentos con hash inalterable no se cargó nunca |
| Copiaba dentro veinte líneas del decorador `assert_log_admin_access` del núcleo | Arrastraba la dependencia del paquete `decorator` para volver a aplicar lo que el propio método ya aplica |
| `action_cancel_hash` leía `self.move_type` sin `ensure_one()` | «Expected singleton» con más de un registro |
| Ambos métodos leían `rec.date.month` sin comprobar que hubiera fecha | |

> El bloqueo de `ir.ui.view` es deliberadamente duro: **solo el superusuario**
> crea, modifica o borra vistas. La instalación de módulos corre como
> superusuario, así que no la estorba, pero un administrador normal no podrá
> editar vistas desde la interfaz. Es lo que el cliente quiere para que la
> localización no se altere sin pasar por el repositorio.

### 6.15 Reporte ARC (versión 1.30.0)

Viene de `nx_localizacion/nimetrix_report_arc` (v18): el comprobante anual de
retenciones de ISLR que se entrega al proveedor.

**La rotura de v20:** `company.mobile`. El campo `mobile` salió de
`res.partner`, que es de donde `res.company` lo tomaba.

La columna «DÍA» se quitaba con un archivo aparte, `fix_report_arc.xml`, hecho
de xpath posicionales sobre la plantilla
(`//div[2]/div/table[2]/thead/tr[2]/th`). Se aplicó en la plantilla y el parche
desapareció — y con él **un `KeyError`**: la clave `dia` solo se escribía para
el primer y el último mes del rango, así que un rango de tres meses o más
reventaba al pintar el mes intermedio. El parche lo tapaba sin arreglarlo.

También: `cod_concepto` se leía fuera del bucle que lo define, así que un
concepto sin tasas configuradas heredaba el código del concepto anterior.

### 6.16 Guías de despacho y facturación desde inventario (versión 1.31.0)

Viene de `nx_localizacion/l10n_ve_stock_account` (v18), 3.233 líneas: guías de
despacho numeradas, motivos de traslado (venta, donación, consignación,
exportación, autoconsumo), creación de facturas y notas de crédito desde el
albarán, notificaciones de cierre fiscal y una tarea programada de facturación.

Es de `binaural-dev` bajo LGPL-3, con aportes del equipo del cliente.

#### Roturas de v20

| Qué | Sustituto |
|---|---|
| `odoo.osv` | El paquete ya no existe. Los dos imports no se usaban |
| **`stock.return.picking` eliminado** | La devolución se hace copiando el albarán y el core deja la traza en `return_id`: `is_return` se calcula de ahí y el archivo entero sobra |
| `ir.model.access` | `ir.access`, con los cuatro booleanos en una cadena `crud` |
| `move_ids_without_package` | `move_ids` |
| `stock.move.name` | Eliminado |
| `stock.move.product_uom` | `uom_id` |
| `sale.order.line.tax_id` | `tax_ids` (y v18 leía `.amount` del many2many: con dos impuestos tomaba solo el primero) |
| `res.users.groups_id` | `group_ids` |
| `product.type = 'product'` | `consu` + `is_storable` |
| `ir.actions.report.report_file` | Eliminado |
| `@constrains('picking_type_code')` | No es escribible: se depende de `picking_type_id` |

#### Dos defectos que las pruebas destaparon

`_check_is_donation` comparaba `order.is_donation` con
`order._origin.is_donation` dentro de un `@api.constrains`. En un registro ya
guardado **`_origin` es el propio registro**: los dos valores son siempre
iguales y la restricción no saltó nunca. `_origin` solo guarda el valor
anterior en los registros virtuales de los onchange. Pasa a `write`, comparando
contra el valor en base.

`print_dispatch_guide` devolvía `self.env.ref(...).read()[0]` — el diccionario
de campos del `ir.actions.report`, con `create_uid` y `write_date` dentro—, no
una acción ejecutable: **el informe no se imprimía**.

#### Lo que no se trajo

El `post_init_hook` de v18 traducía a mano el nombre y la descripción del
módulo en `ir_module_module` con SQL crudo, porque las traducciones del
manifiesto no se cargan solas para módulos de terceros. Al fusionarse dentro de
`l10n_ve_mornix`, que ya tiene su propio nombre, el parche sobra.

#### La guía se numera al confirmar el pedido (1.31.0)

v18 la numeraba solo al **validar el albarán** (`_action_done`). Entre
confirmar el pedido y despachar la mercancía, la guía no existía —que es justo
cuando el cliente la necesita—. Ahora `sale.order.action_confirm` numera los
albaranes del pedido; `_action_done` sigue numerando los que no vienen de un
pedido, y `_set_guide_number` es idempotente, así que nadie consume dos
números.

El campo `guide_number` se ataba a `dispatch_guide_controls`, que exige el
albarán validado: desde que se numera al confirmar, eso lo dejaba oculto justo
cuando hace falta. Ahora se ve en cuanto existe.

Dos defectos más que salieron por el camino:

- `_set_guide_number` no filtraba por paso de entrega. En un almacén de dos o
  tres pasos, PICK y PACK llevan el mismo pedido y el mismo documento, así que
  **cada uno se llevaba su propio número** y la secuencia avanzaba de tres en
  tres. Corregido: solo el albarán que sale del almacén.
- `_compute_show_print_button_when_is_dispatch_guide` calculaba un campo que
  nunca se declaró. Odoo no lo llamaba y leerlo daba `AttributeError`.

> El botón de imprimir la guía **sigue apareciendo solo con el albarán
> validado** (`dispatch_guide_controls`). Si la guía debe poder imprimirse
> desde que se confirma el pedido, para que acompañe la mercancía, hay que
> cambiar esa condición — no se tocó porque no se pidió.

- [ ] **Almacén de dos o tres pasos: el pedido no se puede confirmar.** PICK y
      PACK son albaranes de tipo `internal`, y `_check_transfer_reason_required`
      es un `@api.constrains` que corre en el `create`, cuando esos albaranes
      todavía no tienen su enlace al pedido. Resultado: `ValidationError`
      («The 'Transfer Reason' field is mandatory») al confirmar **cualquier**
      pedido de venta. Con entrega en un solo paso, que es como está hoy la
      instalación, no se nota. La salida es mover la comprobación a la
      validación del albarán, pero eso cambia cuándo se le exige el motivo al
      usuario: hay que confirmarlo con el cliente antes.

- [ ] La tarea programada de facturación automática (`ir_cron`) queda instalada
      y **activa**. Confirmar con el cliente el día y la hora antes de pasar a
      producción.
- [ ] `auto_install: True` del módulo original desaparece al fusionarse. Nada
      que hacer, pero conviene saberlo si alguien busca por qué ya no aparece
      solo en la lista de módulos.


### 6.17 Las retenciones en el portal del cliente (versión 1.32.0)

Un proveedor al que se le retiene necesita su comprobante, y hasta ahora tenía
que pedirlo por correo. Ahora lo ve y se lo descarga desde el mismo portal donde
consulta sus pedidos y facturas.

Tres apartados en «Mi cuenta», uno por tipo, cada uno con su contador, su
listado paginado y su **vista previa con el informe incrustado**, igual que una
cotización, con el mismo enlace con `access_token`:

```
/my/retenciones-iva/<id>?access_token=…
/my/retenciones-islr/<id>?access_token=…
/my/retenciones-municipales/<id>?access_token=…
```

Los tres modelos heredan `portal.mixin` a través de un mixin propio,
`nx.portal.retencion.mixin`, que centraliza lo común. Cada tipo solo declara
dónde tiene el contacto, cuál es su estado publicable y qué informe usa: los
nombres de campo no son uniformes porque los modelos vienen de módulos
distintos, y renombrarlos obligaría a migrar columnas.

#### Qué se ve y qué no

| Regla | Por qué |
|---|---|
| Solo los comprobantes del contacto y sus hijos | Una sucursal ve los de su casa matriz, igual que con las facturas |
| Solo los confirmados | Un borrador todavía puede cambiar de importe |
| Solo los que **emite la compañía** | El comprobante de una retención que nos practica un cliente lo emite él: no existe PDF nuestro, y el informe aborta con «se genera solo para los Proveedores» |
| Solo lectura | El portal no escribe |

Las reglas están tanto en el dominio del controlador como en `ir.access`, para
que no dependa de por dónde se entre.

#### El botón «Vista previa»

En el formulario del comprobante, junto a «Imprimir comprobante», con el mismo
sentido que el de una cotización: abre la vista previa del portal con el enlace
que lleva el `access_token`, o sea exactamente lo que verá el proveedor.

**Solo en las retenciones a proveedores.** Las que nos practica un cliente las
documenta él: no existe comprobante nuestro —el informe aborta con «se genera
solo para los Proveedores»— y no tienen nada que hacer en su portal. Por eso el
botón no está en los formularios de cliente, y el método lo comprueba además por
su cuenta: ocultar un botón no impide llamar al método.

| Formulario | Botón |
|---|---|
| Retención de IVA, proveedor | Sí, con el comprobante publicado |
| Retención de IVA, cliente | No |
| Retención de ISLR, proveedores | Sí, con el comprobante publicado |
| Retención de ISLR, clientes | No |
| Retención municipal | Solo si es `in_tax`, la que practicamos nosotros |

#### Dos cosas que costaron encontrar

**Las tarjetas de «Mi cuenta» ya no se declaran en la plantilla.** v20 las
convirtió en registros de `portal.entry`; la herencia de
`portal.portal_my_home` se procesa sin error y la tarjeta simplemente no
aparece. Detalle en `docs/BREAKING-CHANGES.md`.

**Un `t-set` dentro de un `t-call` no llega a la cabecera de la plantilla
llamada**: se evalúa al desplegar el cuerpo, que ocurre después. Por eso el
título del listado salía vacío. Va en el cuerpo, o se pasa desde el
controlador. Es el mismo motivo por el que el `<title>` de la documentación
salía sin texto.

- [ ] El portal muestra el comprobante en PDF. Confirmar con el cliente si
      además debe poder descargarse el XML o el TXT, que hoy solo existen
      agregados por período, no por comprobante.


### 6.18 Los catorce reportes, generados de punta a punta (1.32.x)

Se generó un ejemplar real de cada reporte de la localización —los nueve PDF,
los tres Excel, el TXT del SENIAT y el XML de ISLR— con los datos de
demostración. El ejercicio destapó tres fallos que la carga del módulo no ve,
porque solo aparecen al **renderizar**:

| Reporte | Fallo | Causa |
|---|---|---|
| Guía de despacho (PDF) | No se generaba | La plantilla leía `partner.mobile` (eliminado en v20) y `move_ids_without_package` (ídem) |
| ARC (PDF) | No se generaba | La plantilla leía `company.nx_rif` y `supplier.nx_rif`, eliminados en la consolidación del identificador (1.2.0) |
| ARC (acción de informe) | Cosmético | Declaraba `model: account.wh.islr.list`, un modelo que no existe. Funcionaba porque todo viaja por `data` |

La lección quedó en la regla de trabajo: **que el PDF salga no significa que
lleve datos, y que el módulo cargue no significa que los reportes rendericen.**
Después de migrar un módulo con reportes, hay que generar cada uno.

Mirar el PDF generado destapó dos más en el ARC, invisibles sin ojos encima:

- **Las filas salían corridas una columna.** Al retirar la columna «DÍA» quedó
  una celda vacía huérfana en la rama de los meses sin datos: 10 celdas para 9
  columnas, y los acumulados desbordaban fuera de la tabla.
- **«Total cantidad pagada» traía el impuesto**, no el monto de la operación
  — desde v16. Con una base de 372 millones y una retención de 18,6, la
  columna decía que se pagaron 18,6. Ahora lleva la base, y el impuesto queda
  solo en su columna.


También se corrigió el fixture de `mornix_dual_currency`, que usaba el RIF
`J-12345678-9`: tan genérico que un contacto real de la base lo tenía puesto y
la constraint de unicidad tumbaba todos los `setUpClass`. Ahora usa el rango
reservado a pruebas (`J-998xxxxx`).

### 6.19 El cliente era editable con la factura publicada (1.32.x)

Observación de Leonardo, verificada sobre el sistema: el campo de cliente se
dejaba editar en una factura publicada. El ORM rechazaba el cambio al guardar
(«You cannot modify the following readonly fields on a posted move»), así que
no había riesgo de datos — pero el usuario editaba, intentaba guardar, y se
llevaba el error al final.

**La causa es una trampa general de las vistas heredadas**: nuestro
`position="replace"` sobre `partner_id` descarta **todos** los atributos del
nodo original, y con ellos se fue el `readonly="state != 'draft'"` del
estándar (también el `placeholder` y el foco inicial, repuestos de paso).

> Regla al hacer un `replace`: copiar los atributos del nodo original y
> quitar conscientemente los que se quieran quitar. Un atributo olvidado no
> falla en la carga ni en el auditor — degrada la vista en silencio.

Queda `tests/test_vista_factura.py` fijando que el `readonly` del contacto no
vuelva a perderse, y que el ORM sigue siendo la barrera definitiva.

### 6.20 Diferencial cambiario en la moneda secundaria (mornix_dual_currency)

Observación de Leonardo, reproducida con números: una factura de 5.000 Bs
emitida a una tasa y cobrada a otra queda saldada en Bs, pero la cuenta por
cobrar quedaba **abierta en USD** (los apuntes llevan su valor en divisa a la
tasa de su fecha). El balance comparado en la moneda secundaria no cuadraba.

Dos defectos encadenados y su arreglo, en `mornix_dual_currency`:

1. **Los cobros llevaban el ref en bolívares.** El cálculo dividía entre
   `nx_rate or 1`, y los asientos de pago tienen `nx_rate = 1`: el «valor en
   divisa» del cobro era el monto en Bs. Ahora, sin tasa propia, se usa la
   misma fuente que las facturas (la tasa BCV del día exacto) y el conversor
   estándar como último recurso. **Misma fuente a ambos lados**: si cada lado
   convirtiera con una distinta, un cobro del mismo día ya no cuadraría.
2. **No existía el diferencial en divisa.** Al completarse una conciliación,
   si queda residuo en la moneda secundaria se genera un asiento que vale
   **cero en Bs** (no toca la contabilidad legal) y lleva el ajuste en divisa
   contra las cuentas de ganancia/pérdida cambiaria. Es idempotente por
   conciliación (`nx_full_reconcile_ref_id`) y el diferencial en Bs de Odoo
   pasa a valer cero en divisa, por la razón espejo.

Verificado sobre la instancia: los dos escenarios (factura en USD con
diferencial Bs, y factura en Bs cobrada a otra tasa) cierran en ambas monedas.
Cinco pruebas nuevas en `mornix_dual_currency/tests/test_diferencial_ref.py`.

### 6.21 Migración retroactiva del diferencial en divisa (mornix_dual_currency 1.8.0)

El arreglo de 6.20 actúa solo en las conciliaciones nuevas. La migración
`1.8.0/post-diferencial-ref-retroactivo.py` arregla el histórico, y correrla
dos veces no duplica nada:

1. **Recalcula** el valor en divisa de los asientos sin tasa propia (los pagos,
   que llevaban el ref en bolívares) — con compute directo: `add_to_compute`
   considera vigente el valor almacenado y no reevalúa.
2. **Genera el ajuste** de cada conciliación completa con residuo en divisa,
   con la misma rutina idempotente del flujo normal.
3. **Verifica** que ningún asiento publicado quede descuadrado en divisa, y
   hace una segunda pasada dirigida sobre los que el filtro por tasa no atrapó.

La verificación destapó dos reglas nuevas del cálculo, que van más allá de la
migración:

- **La tasa implícita del asiento manda.** Si un asiento tiene una línea
  denominada en la divisa (la CxP de un comprobante de retención, a la tasa de
  la factura), esa línea implica la tasa de todo el asiento. Convertir las
  demás piernas con otra fuente lo descuadraba por céntimos — y la validación
  del cuadre en divisa habría **bloqueado el siguiente write** sobre él.
- **`debit_ref`/`credit_ref` derivan del balance.** Ese compute tenía su propia
  lógica duplicada que volvía a dividir por `nx_rate`; con tasa implícita ahora
  deriva del `balance_ref`, la única fuente.

Resultado sobre la instancia: **cero asientos publicados descuadrados en
divisa** en toda la base.

### 6.22 Los seis formatos de retención: corrección (1.33.0) y rediseño (1.34.0)

Dos entregas seguidas sobre los mismos seis archivos: los **tres comprobantes**
(IVA, ISLR y municipal) y los **tres reportes detallados**. Conviene leerlas
juntas porque la segunda se apoya en la primera.

#### 1.33.0 — lo que estaba roto

Corrigió defectos de salida, no de estética: tres convenciones numéricas
conviviendo en la misma fila (`5220.0`, `5,220.00` y `4.500,00`), RIF partidos
en tres líneas, cabeceras de columna rotas a media palabra, una página en
blanco al final de cada comprobante, formatos de papel declarados
`default="True"` —que los convertía en el formato por omisión de **toda** la
base— y Bootstrap cargado desde un CDN que wkhtmltopdf nunca alcanza porque
corre sin salida a internet. Introdujo `models/nx_report_format.py` y la hoja
de estilos compartida `report/nx_report_styles.xml`.

#### 1.34.0 — cómo se ven

Rediseño estético sobre esa base. **9 archivos modificados, ninguno nuevo,
ningún cambio de esquema, ninguna consulta nueva.** Los proveedores de datos
quedaron correctos en 1.33.0 y no se tocaron; el único `.py` que cambia es
`models/nx_report_format.py`, que gana el ayudante de paleta.

| Antes | Ahora |
|---|---|
| Rejilla completa: cada celda en su recuadro | Banda oscura de cabecera, filete horizontal y alternancia de filas. La división vertical queda en un tono casi invisible |
| Bloques de identificación como tabla con bordes | Tarjetas con esquina curva y franja de título — **oscura** el agente de retención, **clara** el sujeto retenido |
| Tira de celdas donde el nº de comprobante pesaba lo mismo que su etiqueta | Cinta de fondo sólido con el número en 11,5 pt |
| El comprobante terminaba a media página | Fila de fichas de totales, con el impuesto retenido destacado en oscuro |
| Línea de firma suelta, con la imagen encima por margen negativo | Recuadro punteado curvo con la firma **dentro**, centrada |
| Color de banda escrito a mano en la hoja de estilos | Sale de **Ajustes → Diseño del documento**, con el contraste del texto calculado |

Las fichas no son relleno: el impuesto retenido es la cifra por la que se lee
el comprobante y estaba enterrada en la decimocuarta columna de la tabla.

#### El color sale de Ajustes, no del código

La banda —cabeceras de tabla, cinta y ficha destacada— toma
`res.company.primary_color`, y la barra de acento bajo el encabezado toma
`secondary_color`. Son los dos campos de **Ajustes → Diseño del documento**,
que es donde el cliente configura su identidad y donde espera cambiarla. Sin
ajuste se usa el azul pizarra de partida.

El esqueleto (grises de filete, cebra y texto de tabla) **se queda neutro a
propósito**: si la marca tiñe también los separadores, un color saturado
convierte el comprobante en un cartel.

**El texto sobre la banda se calcula, no se fija.** `nx.report.format.get_theme`
mide la luminancia del color elegido según WCAG 2.1 y conmuta a tinta oscura en
cuanto pasa de medio tono. Con un amarillo corporativo —caso nada raro— el
texto blanco de siempre habría desaparecido. Los tintes claros de las tarjetas
y el color de marca para texto sobre fondo claro se derivan del mismo color
mezclando con blanco y oscureciendo hasta que la luminancia baja del umbral.

**Consecuencia práctica:** el color se interpola en el CSS al renderizar, no con
variables CSS. El Qt WebKit de wkhtmltopdf no soporta `var()` — descarta la
declaración entera y el color se pierde sin avisar. Por eso la hoja de estilos
necesita que quien la invoca deje puesta la compañía del documento:

```xml
<t t-set="nx_company" t-value="d.company_id"/>
<t t-call="l10n_ve_mornix.nx_report_styles"/>
```

Como el `t-call` va en el `<head>`, fuera del `t-foreach`, la compañía se toma
del primer registro del lote (`data[:1].company_id`). Imprimir de golpe
comprobantes de dos compañías distintas saldría con el color de la primera; el
caso no se da porque los asistentes filtran por compañía, y la alternativa
—repetir la hoja de estilos entera por documento— sale mucho más cara.

#### Lo que obliga el motor de PDF

Curvar las esquinas de una tabla tiene una consecuencia que no es obvia:
**`border-collapse: collapse` ignora `border-radius`**. No es cosa de
wkhtmltopdf, pasa en todos los navegadores — al colapsar los bordes el motor
descarta el radio. Por eso las tablas de detalle pasaron a
`border-collapse: separate` con `border-spacing: 0`, y el radio se declara en
las celdas de las esquinas.

**Consecuencia práctica agradable:** al cortar página la cabecera se repite ya
curvada, y el pie de totales cierra la caja donde de verdad termina.

Lo demás se verificó renderizando una página de prueba contra el **mismo
binario** que usa Odoo, no contra un navegador:

| | |
|---|---|
| `border-radius` | Sí, con y sin prefijo |
| `linear-gradient` | **No se pinta.** El fondo queda transparente y un bloque con texto blanco encima desaparece entero |
| `box-shadow` | Sí, pero descartado: en láser una sombra gris ensucia |
| Flexbox | No fiable. Los centrados verticales van con `vertical-align: middle` sobre celda de tabla |
| Borde en `inline-block` dentro de celda | **No se pinta**; el fondo y el radio sí. Por eso las píldoras van con relleno |
| Filetes de `.5pt` | Se pierden al rasterizar a 110 dpi. `.6pt` es el mínimo que sobrevive; los marcos van a `.7pt` |

Las dos últimas costaron una vuelta de render cada una y quedan anotadas en la
propia hoja de estilos.

#### Lo que quedó abierto

- [ ] **El ARC no se rediseñó a propósito.** Reproduce un formulario prescrito
      por el SENIAT —campos numerados, rejilla de tipo de agente, casillas
      SI/NO— y esa maqueta no es libre de cambiar. Sí tiene un defecto real:
      su formato de papel declara `margin_top: 30` con `header_spacing: 50` y
      **el logo sale cortado por arriba**. Arreglo de dos líneas, pero sobre un
      formato legal: se decide con el equipo.
- [ ] **Sigue sin haber campo de sello.** Los comprobantes estampan solo
      `nx_firma_representante`. Añadirlo es un cambio de esquema en capa B. El
      recuadro de firma ya está dimensionado para alojar firma y sello juntos:
      sería un `fields.Binary` más un `<img class="nx-firma"/>` por
      comprobante, sin tocar la hoja de estilos.
- [ ] **El diseño está validado contra wkhtmltopdf.** Si v20 pasa a
      `base_report_paper_muncher` (decisión abierta en
      `docs/BREAKING-CHANGES.md`) hay que volver a mirar los seis: las tres
      soluciones de arriba son rodeos a limitaciones de Qt WebKit que Paper
      Muncher no tiene, así que no deberían romper nada — pero no está
      comprobado.
- [ ] **Los PDF de muestra se generaron sobre Odoo 18**, en el banco de pruebas
      con datos reales (71 retenciones de IVA, 47 de ISLR, 31 municipales).
      Falta regenerarlos sobre v20 para confirmar que el render es idéntico.

### 6.23 Los huecos del análisis de cobertura, cerrados

El analizador de la skill `senior-qa` encontró seis modelos con lógica propia
y cero pruebas. Los seis quedaron cubiertos con pruebas reales (no esqueletos):

| Modelo | Qué se fijó |
|---|---|
| `nimetrix.wh.islr.xml` | Los totales del XML del SENIAT suman sus líneas, siguen a las altas/bajas, y el archivo generado lleva el RIF y la estructura |
| `sale.order.line` | Precio negativo se rechaza; precio cero pasa en borrador pero no confirma; con el candado apagado se permite |
| `product.template` | Precio USD negativo se rechaza; el onchange convierte a Bs con la tasa inversa |
| `ir.ui.view` | El candado de vistas: un administrador no crea/modifica/borra, el superusuario sí |
| `purchase.order` | Un proveedor sin RIF no puede comprar |
| `nimetrix.wh.iva.line` | `_amount_all` suma las líneas de impuesto (ojo: `nx_move_id` apunta a la factura, no al apunte — el nombre engaña) |

Con esto, **ningún modelo del módulo con lógica propia queda sin prueba**
(la advertencia del analizador sigue vigente: es búsqueda de texto, no
cobertura real — sirve para encontrar huecos, no para declararse cubierto).

### 6.24 QA de cobertura en mornix_dual_currency y mornix_docs

Mismo análisis que 6.23, sobre los otros dos módulos:

- **mornix_docs**: limpio de entrada — sus dos modelos con lógica ya estaban
  cubiertos por las 15 pruebas del módulo.
- **mornix_dual_currency**: un hueco real en `product.pricelist.item` — pero
  con matiz: `_compute_fixed_price` ya lo cubrían las pruebas de
  `l10n_ve_mornix`, y el analizador solo mira las pruebas del propio módulo.
  Lo genuinamente sin probar eran la etiqueta de precio en sus ramas de
  porcentaje y fórmula, y el tooltip de la regla — cubierto con 4 pruebas
  nuevas, sin duplicar lo que ya existía.
- `res.company` y `res.config.settings` de dual_currency siguen marcados
  «sin mención», y se quedan así a propósito: cero métodos, solo campos —
  una prueba ahí sería mencionar por mencionar.

### 6.25 QA de cobertura en guías de despacho y portal

Mismo análisis, sobre las dos áreas que faltaban de este módulo. Doce pruebas
nuevas (`test_nx_stock_wizards.py` y `TestMixinPortal` en
`test_portal_retenciones.py`) que cubren el asistente de facturación masiva,
el asistente de autoconsumo, el paso de entrega del tipo de operación, el
generador del cron de facturación y los diez métodos del mixin del portal.
`transfer.reason` y `nimetrix.fiscal.notification` quedan sin mención a
propósito: cero métodos.

Las pruebas destaparon **dos defectos reales de la migración**, ya corregidos:

- **Facturar una guía nacida de un pedido de venta estaba roto**:
  `_get_invoice_lines_for_invoice` leía `sale_line_id.tax_id`, que v20
  renombró a `tax_ids`. El mismo rename ya se había corregido en
  `nx_stock_stock_move.py`, pero esta aparición se escapó del barrido — y las
  pruebas viejas no la pisaban porque facturaban traslados sin pedido.
- **El cron de facturación moría entero**: su dominio de búsqueda incluía
  `("invoice_count", "=", 0)`, y `invoice_count` es un compute sin almacenar
  — v20 lo rechaza con `ValueError` («no SQL representation»). Ahora se
  filtra en Python tras el search.

Dos matices que dejaron las pruebas y conviene saber:

- En v20 un producto puede **no tener categoría** (`categ_id` ya no trae la
  categoría «All» por defecto). La cadena de respaldo de la cuenta de
  ingresos (producto → categoría) puede acabar en `False` y la factura
  revienta contra `check_accountable_required_fields`: el producto de una
  guía debe tener cuenta de ingresos propia o categoría con cuenta.
- `_compute_invoice_count` no declara `depends` (cuenta con un `search`):
  dentro de una misma transacción el valor queda cacheado. En la interfaz no
  se nota porque cada request trae un env fresco; en pruebas se invalida a
  mano con `env.invalidate_all()`.

Suite completa de los tres módulos tras el cierre: **347 pruebas, 0 fallos**.

### 6.26 Rendimiento: los computes de la lista de guías, en lote (1.33.2)

El QA incorporó criterios de rendimiento (sección D de
`docs/CRITERIOS-ACEPTACION.md`): el coste en consultas no debe crecer con el
número de registros. El estreno fue sobre los dos N+1 conocidos de este
módulo, en `stock.picking`:

- `_compute_invoice_count` hacía **un search por albarán** — una lista de 80
  guías eran 80 consultas, y el cron de facturación lo disparaba sobre todo
  el histórico despachado. Además asignaba sobre `self` entero en cada
  vuelta del bucle, así que una lista con varios albaranes prefetcheados
  mostraba en **todos** el conteo del último — bug funcional, no solo de
  velocidad, que venía de v18.
- `_compute_invoice_state` tenía el mismo search por registro.

Ambos quedan en **una consulta agrupada por lote** (`_read_group`). Tres
pruebas nuevas en `tests/test_rendimiento.py` fijan el presupuesto con
`assertQueryCount` (12 albaranes ≤ 2 consultas; con el N+1 eran 12+) y
verifican que cada albarán muestra su propio conteo. La guía de cómo escribir
estas pruebas está en la skill de QA
(`references/rendimiento.md`), con los puntos calientes que siguen
pendientes de medir: libros fiscales, listados de retención y los computes
de doble moneda sobre `account.move.line`.

**Consecuencia práctica**: la columna «Facturas» de la lista de guías era
mentirosa cuando había varias a la vista; si algún usuario reportó conteos
raros ahí en v18, era esto.

### 6.27 El asistente de retenciones de empleados, reescrito (1.34.1)

La auditoría de propiedad del código (ver
[`PROPIEDAD-Y-LICENCIAS.md`](../PROPIEDAD-Y-LICENCIAS.md)) encontró que el
asistente de importación de retenciones ISLR de empleados descendía de
`l10n_ve_full` (Tecvemar C.A., 2012, **GPL-2**) — licencia incompatible con
la LGPL-3 del módulo y autoría de un tercero. Se sustituyó por una
reimplementación independiente: misma función (el CSV de la nómina entra a la
declaración XML del SENIAT), código nuevo.

Lo que cambia para el usuario, a mejor:

- Los errores del archivo se reportan **todos juntos y con número de fila**;
  antes reventaba en el primero y había que iterar a ciegas.
- Con cualquier error **no se importa nada** (antes también, pero ahora es
  explícito en el mensaje).
- Reimportar reemplaza las líneas de empleados sin duplicar y sin tocar las
  líneas de facturas.

**Consecuencia práctica**: la versión heredada estaba rota en v20 de todos
modos — hacía `b64decode` de un campo Binary que ahora entrega los bytes
crudos, así que cualquier importación fallaba con «no es texto UTF-8». Si la
nómina reporta ese error en otras pantallas con carga de archivos, es el
mismo patrón. Cubierto por 4 pruebas (`test_importar_empleados_islr.py`).

## 7. Cómo levantarlo

```bash
cd docker
docker compose run --rm odoo20 odoo -d <base> -i l10n_ve_mornix \
    --without-demo --stop-after-init
```

Pruebas:

```bash
docker compose run --rm odoo20 odoo -d <base> -u l10n_ve_mornix \
    --test-enable --stop-after-init
```

Instancia de prueba viva: **https://odoo20.migracion.mornix.tech**
