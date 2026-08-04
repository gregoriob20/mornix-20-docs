# Plan de pruebas — l10n_ve_mornix

> Módulo piloto de la migración a v20.
> Estado técnico: instala, actualiza y pasa sus **120 pruebas** sin errores ni
> advertencias. **Eso no significa que esté aprobado**: ver la sección 5.

## 1. Qué está cubierto hoy

**120 pruebas automatizadas en 12 archivos, todas en verde.**

| Archivo | Casos | Qué cubre | Origen |
|---|---:|---|---|
| `test_invoice_wh_iva_islr.py` | 4 | Flujo completo de factura de 40 líneas con retención de IVA e ISLR, para los cuatro tipos de persona: **PJDO, PJND, PNRE, PNNR** | heredado |
| `test_product_pricelist.py` | 17 | Precios en divisa, precio fijo, fórmulas, etiquetas y trampas de valores falsy | heredado |
| `test_res_company.py` | 8 | Validación del RIF de la compañía y moneda de referencia | heredado |
| `test_account_move_invoice_num.py` | 1 | El número de control sobrevive a borrador → publicar → borrador → publicar | heredado |
| `test_account_ut.py` | 8 | Unidad tributaria: valor vigente por fecha, conversiones y el caso sin UT | **nuevo** |
| `test_seniat_rif_formato.py` | 8 | Formato del RIF almacenado y el que llega al TXT y al XML | **nuevo** |
| `test_wh_iva_txt.py` | 7 | Formato del número de documento en el TXT del SENIAT | **nuevo** |
| `test_fiscal_book.py` | 11 | Qué documentos entran en cada libro fiscal | **nuevo** |
| `test_numeracion_control.py` | 8 | Correlativo fiscal y selección de secuencia por tipo de documento | **nuevo** |
| `test_fiscal_book_xlsx.py` | 6 | Exportación a Excel: nombre de hoja dentro de los límites y archivo válido | **nuevo** |
| `test_libro_resumen_iva.py` | 5 | Libro Resumen de IVA: consultas válidas y el Excel se genera | **nuevo** |
| `test_wh_iva_numero.py` | 11 | Número del comprobante de retención: formato de 14, sustitución del mes contable, correlativo | **nuevo** |

Que los cuatro tipos de persona estén cubiertos es lo más valioso de la suite
heredada: es exactamente el eje que v20 puso en riesgo al eliminar
`company_type`.

De los tests nuevos, los de `test_seniat_rif_formato.py` son los que más
protegen: fijan que el RIF conserve sus guiones, cosa que depende de un override
frágil. Si alguien lo quita, esos tests fallan antes de que salga un TXT mal
formado.

### Dos comportamientos que los tests nuevos dejaron a la vista

Ninguno es una rotura de la migración; ambos venían de antes y no eran obvios:

1. **El dígito verificador del RIF no se valida.** El módulo exime a Venezuela
   del chequeo de `base_vat` y solo comprueba el formato con su regex. Un RIF
   con el dígito mal entra sin protestar.
2. **`nx_get_number` ignora su propio parámetro `inv_type`.** El código quiere
   quedarse solo con dígitos cuando el tipo es `vou_number`, pero usa
   `elif i.isalnum()`: para una letra la primera condición es falsa y cae al
   `elif`, que la acepta igual. Ambos tipos dan el mismo resultado.

Los tests fijan el comportamiento **real**, no el aparente. Cambiarlos alteraría
lo que sale en el TXT, y eso lo decide el cliente.

- [ ] Confirmar ambos puntos con el cliente.

## 2. Qué NO está cubierto

**33 de los 46 modelos no aparecen en ninguna prueba.** Medido con
`.claude/skills/senior-qa/scripts/coverage_analyzer.py`. Los que importan:

| Modelo sin cubrir | Por qué importa |
|---|---|
| `nimetrix.wh.iva` | El comprobante de retención de IVA. Solo se ejercita de refilón desde el test de factura |
| `nimetrix.wh.iva.txt` | **TXT del SENIAT.** Un error aquí es un problema con la administración tributaria |
| `nimetrix.wh.islr.xml` | **XML del SENIAT.** Igual |
| `nimetrix.fiscal.book` | Libros de compras y ventas |
| `nimetrix.account.ut` | Unidad tributaria: entra en el cálculo del sustraendo de ISLR |
| `nimetrix.wh.islr.rates` | Tarifas de retención. Un error cambia todos los montos |
| `account.move.reversal` | Notas de crédito con número de control |
| `nimetrix.employee.income.wh_islr` | Carga masiva del ARC |

## 3. Plan por flujo

Cada caso indica **cómo se verifica** y **contra qué se compara**. El criterio
no es "funciona", es "da lo mismo que en v18".

### 3.1 Retención de IVA

| # | Caso | Verificación | Estado |
|---|---|---|---|
| IVA-1 | Retención sobre factura de proveedor, tasa 75% | Monto retenido = base × 75%, comparado con la instancia v18 | cubierto parcial |
| IVA-2 | Retención tasa 100% (contribuyente especial) | Idem | **pendiente** |
| IVA-3 | Comprobante: numeración correlativa sin saltos | Correlativo verificado, más la sustitución del mes contable en las posiciones 4:6 | cubierto (`test_wh_iva_numero.py`) |
| IVA-4 | Cancelar y rehacer una retención | El correlativo no se repite ni se salta | **pendiente** |
| IVA-5 | Factura sin derecho a crédito fiscal | El asistente marca y el libro la excluye | **pendiente** |
| IVA-6 | TXT del SENIAT: estructura y longitud de campos | Comparar byte a byte con un TXT generado en v18 con los mismos datos | **pendiente** |
| IVA-7 | TXT: formato del RIF | El TXT normaliza vía `nx_documento_seniat()`; se verificó que `vat` conserva los guiones al guardarse | cubierto (`test_seniat_rif_formato.py`) |

### 3.2 Retención de ISLR

| # | Caso | Verificación | Estado |
|---|---|---|---|
| ISLR-1 | PJDO — jurídica domiciliada | Tarifa y sustraendo correctos | cubierto |
| ISLR-2 | PJND — jurídica no domiciliada | Idem, y exención de la validación de vat | cubierto |
| ISLR-3 | PNRE — natural residente | Idem | cubierto |
| ISLR-4 | PNNR — natural no residente | Idem | cubierto |
| ISLR-5 | Sustraendo con la unidad tributaria vigente | Cambiar la UT y verificar que el cálculo la sigue | cubierto (`test_account_ut.py`) |
| ISLR-6 | ~~Proveedor exento~~ | El campo se eliminó en 1.4.0 por decisión del cliente: ya no hay proveedores exentos | **no aplica** |
| ISLR-7 | Varios conceptos en una misma factura | Cada concepto con su tarifa | **pendiente** |
| ISLR-8 | XML del SENIAT | Comparar con el XML generado en v18 | **pendiente** |
| ISLR-9 | ARC: carga masiva desde CSV | Importar el CSV de plantilla y verificar totales | **pendiente** |

### 3.3 Libros fiscales

| # | Caso | Verificación | Estado |
|---|---|---|---|
| LIB-1 | Libro de ventas de un período | Totales cuadran con la suma de facturas | **pendiente** |
| LIB-2 | Libro de compras de un período | Idem | **pendiente** |
| LIB-3 | Exclusión por posición fiscal | El dominio añade `fiscal_position_id not in` | cubierto (`test_fiscal_book.py`) |
| LIB-4 | Exportación a Excel | Se verifica que el .xlsx sea un zip válido con workbook. **Omitido** sin `mornix_dual_currency` | parcial (`test_fiscal_book_xlsx.py`) |
| LIB-5 | Nombre de la hoja de Excel | Se verifica longitud ≤31 y ausencia de caracteres prohibidos, que v20 ya no sanea | cubierto (`test_fiscal_book_xlsx.py`) |

### 3.4 Numeración de control

| # | Caso | Verificación | Estado |
|---|---|---|---|
| NUM-1 | El número persiste tras republicar | | cubierto |
| NUM-2 | Nota de crédito usa su propia secuencia | Y cae en la de factura si no está definida | cubierto (`test_numeracion_control.py`) |
| NUM-3 | Correlativo por diario, sin saltos | 3 facturas seguidas dan 3 números consecutivos, en factura y en control | cubierto (`test_numeracion_control.py`) |
| NUM-4 | Asignación manual con el asistente | | **pendiente** |

### 3.5 Precios y divisa

Los 17 casos existentes cubren bien esta área. Faltan:

| # | Caso | Verificación | Estado |
|---|---|---|---|
| PRE-1 | Redondeo con tasa de cambio no redonda | Usar tasa 36,58 y verificar que el total cuadra con la suma de líneas | **pendiente** |
| PRE-2 | Precio con IVA en la lista de precios | | **pendiente** |

## 4. Pruebas de datos — la parte que falta entera

Nada de lo anterior se ha probado sobre datos reales del cliente. Sobre base
limpia, estos módulos casi siempre pasan.

| # | Caso | Por qué |
|---|---|---|
| DAT-1 | Restaurar un volcado real del cliente y actualizar el módulo | Es la única prueba que vale para el criterio B |
| DAT-2 | Ningún contacto cambia de tipo de persona tras migrar | v20 recalcula `is_company`. Un cambio aquí altera retenciones |
| DAT-3 | Los RIF históricos siguen con su formato | Si algún intento previo los normalizó, quedaron sin guiones: `nx_rif NOT LIKE '%-%'` |
| DAT-4 | Las secuencias fiscales continúan, no se reinician | Un salto de correlativo es un problema legal |
| DAT-5 | Los totales de los libros fiscales coinciden con los de v18 | Comparar período cerrado contra período cerrado |

- [ ] Conseguir un volcado real anonimizado. **Sin esto, DAT-1 a DAT-5 no se
      pueden ejecutar y el módulo no puede aprobarse.**

## 5. Por qué esto todavía no es un APROBADO

Los 88 tests en verde cubren el criterio **A3**. Faltan:

- **A2** — actualizar sobre una base con datos: solo se probó sobre base limpia.
- **A6** — vistas: no se han abierto una por una en la interfaz.
- **A7** — reportes PDF: no se ha generado ninguno con datos reales.
- **B1 a B5** — todo el bloque de datos.
- **C1 a C5** — los criterios de negocio, que confirma el cliente.

Según `docs/CRITERIOS-ACEPTACION.md`, un criterio no verificado es un criterio
no cumplido. El veredicto actual es **RECHAZADO por no verificable**, y lo que
falta para cambiarlo es un volcado real del cliente y una sesión de validación
funcional.

Que instale limpio y pase sus pruebas es una buena noticia sobre el código.
No es una afirmación sobre la contabilidad del cliente.

## 6. Cómo ejecutar

```bash
cd docker

# suite completa
docker compose run --rm odoo20 odoo -d odoo20 -u l10n_ve_mornix \
    --test-enable --stop-after-init

# una sola clase
docker compose run --rm odoo20 odoo -d odoo20 -u l10n_ve_mornix \
    --test-enable --test-tags /l10n_ve_mornix:TestInvoiceWithholdingIvaIslr \
    --stop-after-init

# qué modelos no tienen prueba
python3 .claude/skills/senior-qa/scripts/coverage_analyzer.py \
    addons/localizacion/l10n_ve_mornix
```

Instancia viva para validación funcional: **https://odoo20.migracion.mornix.tech**

## 7. Decisiones pendientes del cliente

Los tests fijan el comportamiento **actual** en cada uno de estos puntos. No se
cambió ninguno, porque cambiarlos altera lo que ve o presenta el cliente y esa
no es una decisión técnica. Cuando haya respuesta, se ajusta el test y el
código a la vez.

| # | Qué hace hoy | Por qué importa |
|---|---|---|
| 1 | El número de documento del TXT del SENIAT **lleva las letras** (`FAC00123`, no `00123`) | El código parece querer solo dígitos, pero un `elif` mal puesto acepta las letras. Si el SENIAT espera solo dígitos, el archivo sale mal |
| 2 | Un RIF con **dígito verificador inválido se acepta** | El módulo exime a Venezuela del chequeo de `base_vat` y solo valida el formato |
| 3 | Un número de retención de **más de 14 caracteres se trunca en silencio** | `size=14` corta antes de que la validación llegue a mirarlo. Solo se rechazan los cortos |
| 4 | El mensaje de error del número de retención dice *"no puede tener más de 14"* pero rechaza cualquier largo distinto de 14 | Enganoso con un número corto |
| 5 | El contexto `default_admin_note` de los recibos **se perdió** | v20 eliminó las acciones de recibo. Solo importa si el cliente usa recibos |
| 6 | Los 4 `name_get()` muertos **no se tocaron** | Reemplazarlos por `_compute_display_name` exige decidir qué debe mostrar cada modelo |

## 8. Qué bloquea cerrar el resto

| Bloqueo | Qué destraba |
|---|---|
| `mornix_dual_currency` sin migrar | LIB-1, LIB-2, LIB-4 y los 4 tests omitidos. El libro fiscal usa `amount_exempt_bs`, que define ese módulo |
| Sin volcado real del cliente | Todo el bloque B (DAT-1 a DAT-5), que es el que de verdad prueba una migración |
| Sin respuesta a la sección 7 | IVA-6 e ISLR-8: no se puede comparar un TXT o un XML contra lo esperado sin saber qué se espera |
