# l10n_ve_nimetrix — Localización venezolana

> Módulo piloto de la migración a v20. Estado: **instala, actualiza y pasa sus
> 30 pruebas sin errores ni advertencias.**
> Origen: `nx-desarrollo/nx_localizacion`, rama `main`, versión `18.0.0.11.0`.
> Destino: `addons/localizacion/l10n_ve_nimetrix`, versión `1.0.0` (Odoo la
> prefija con la serie vigente → `19.5.1.0.0`).

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
| Pruebas | 30 (4 archivos) |

Que no tenga JavaScript es la razón por la que este módulo, siendo el más
grande, no fue el más difícil de migrar: OWL es lo que más rompe entre v16 y
v20, y aquí no aplica.

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
| `res.partner` | `models/res_partner.py` | 362 | RIF, tipo de persona, agente de retención, validaciones |
| `product.pricelist.item` | `models/product_pricelist_item.py` | 248 | Precio con IVA y en divisa |
| `account.move.reversal` | `models/account_move_reserval.py` | 116 | Nota de crédito con número de control |
| `res.company` | `models/res_company.py` | 114 | RIF, firma del representante, validaciones configurables |
| `product.template` / `product.product` | | 109 / 86 | Precios con IVA y en divisa |
| `account.payment` | `models/account_payment.py` | 99 | Validaciones de RIF en el pago |
| `account.tax` | `models/account_tax.py` | 90 | Marcado de impuestos de retención |
| `sale.order` / `purchase.order` | | 78 / 70 | RIF y tipo de documento en el pedido |

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

**En el estado final, esto NO afecta al módulo.** Verificado en la base piloto:
el `vat` y el `nx_rif` conservan sus guiones. La razón es que el override de
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
      previo, hay que detectarlo: `nx_rif NOT LIKE '%-%'`.

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

## 7. Cómo levantarlo

```bash
cd docker
docker compose run --rm odoo20 odoo -d <base> -i l10n_ve_nimetrix \
    --without-demo --stop-after-init
```

Pruebas:

```bash
docker compose run --rm odoo20 odoo -d <base> -u l10n_ve_nimetrix \
    --test-enable --stop-after-init
```

Instancia de prueba viva: **https://piloto.migracion.mornix.tech**
