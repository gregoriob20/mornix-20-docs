# 03 — Prestaciones sociales y fideicomiso

> Localización de nómina ZOOM · Odoo 18 · corte 24/07/2026

---

## 1. Panorama

El cálculo de prestaciones sociales es el subsistema más grande y más sensible de la solución. Se apoya en tres modelos de trazabilidad, un asistente de cálculo masivo, un tablero analítico y una salida bancaria hacia el fideicomiso.

```
   Recibos de nómina (hr.payslip)
             │
             ▼
   salary.bonus.history       ← traza mensual de salario y bonos
             │
             ▼
   Motor de cálculo           ← models/hr_contract.py (4.080 líneas)
             │
             ▼
   social.benefits.history    ← un registro por contrato y mes
             │
      ┌──────┴──────┐
      ▼             ▼
   Tablero OWL   TXT fideicomiso (banco)
```

**Principio de diseño:** el cálculo **nunca** se rehace desde los recibos en caliente. Se apoya en una traza mensual persistida (`salary.bonus.history`), de modo que recalcular un mes pasado devuelve el mismo resultado que se obtuvo entonces, aunque el salario del trabajador haya cambiado desde entonces.

---

## 2. Modelos de trazabilidad

### 2.1 `social.benefits.history` — historial mensual de prestaciones

Un registro por **contrato y mes de cálculo**. Es la fuente de verdad de las prestaciones.

| Campo | Significado |
|---|---|
| `calculation_date` | Mes de cálculo |
| `employee_id`, `contract_id`, `department_id`, `company_id` | Identificación |
| `period_start`, `period_end` | Período cubierto |
| `base_salary_bs` | Salario base del mes |
| `daily_wage` | Salario diario |
| `comprehensive_salary` | **Salario integral** — base del cálculo |
| `aliquot_holidays`, `aliquot_profits` | Alícuotas de bono vacacional y de utilidades |
| `social_benefits_generated` | Prestaciones generadas en el mes (garantía, Art. 142 *a*) |
| `days_per_year_accumulated` | Días adicionales acumulados (Art. 142 *b*) |
| `employee_additional_days` | Días adicionales que corresponden al trabajador |
| `number_of_days_social_benefits` | Días de prestación del período |
| `benefit_interest` | Intereses del mes (Art. 143) |
| `accrued_social_benefits` | Acumulado histórico |
| `total_available` | Total disponible para el trabajador |
| `employee_type` | Clasificación (regular, vendedor, temporal) |
| `seller_avg_3m`, `seller_avg_6m` | Promedios de comisiones del personal de ventas |
| `total_vacations_days` | Días de vacaciones considerados |

### 2.2 `salary.bonus.history` — traza mensual de salario y bonos

Registra, mes a mes, el salario y los bonos vigentes de cada contrato. Es el insumo de los promedios de 3, 6 y 12 meses, y lo que permite recalcular un mes pasado con los valores de aquel momento.

Se alimenta automáticamente cuando cambia un campo rastreado del contrato (`_sync_salary_bonus_history`) y al cerrar cada recibo.

### 2.3 `average.wage.lines` — líneas de salario promedio

Detalle de los promedios calculados por tipo, asociados al recibo.

---

## 3. Cálculo del salario integral

El salario integral es la base de las prestaciones. Se compone de:

```
Salario integral = salario normal
                 + alícuota de bono vacacional
                 + alícuota de utilidades
                 + promedio de conceptos variables (según tipo de trabajador)
```

Existen **cuatro variantes** según el tipo de trabajador, todas en `models/hr_contract.py`:

| Variante | Método | Particularidad |
|---|---|---|
| General | `_calculate_comprehensive_salary()` | Caso base |
| Ventas | `_calculate_comprehensive_salary_seller()` | Incorpora el promedio de comisiones (3 y 6 meses) |
| Con bono nocturno | `_calculate_comprehensive_salary_regular_with_night_bonus()` | Distingue bono fijo de variable |
| Temporal / destajista | `_calculate_comprehensive_salary_temporary()` | Días adicionales propios; se excluye del fideicomiso |

### Precisión decimal

Las tres variantes eliminan los redondeos intermedios y aplican `ROUND_HALF_UP` con `Decimal` solo en el paso final. Python redondea por defecto "al par más cercano" (redondeo bancario), lo que producía diferencias de céntimos frente al sistema anterior. Este ajuste fue verificado contra cálculos manuales de liquidación.

### Bono nocturno: fijo o variable

Un bono nocturno **fijo** entra a la base de alícuotas del recibo regular. Uno **variable** entra por promedio.

La clasificación se decide leyendo el **seguimiento del indicador en el contrato** a lo largo del tiempo (`_zoom_night_bonus_fixed_timeline`), no la presencia del concepto en la traza de pagos. Motivo: la regla que paga el bono liquida por turnos trabajados sin consultar el indicador, de modo que la traza contiene pagos de ambos tipos. Clasificar por la traza dejaba a trabajadores variables fuera del promedio, con resultado cero.

Un bono fijo se aplica **solo desde su fecha de alta** en el seguimiento, no retroactivamente.

---

## 4. Reglas de negocio consolidadas

Cada una de estas reglas responde a un caso real detectado en la conciliación con el sistema anterior.

| Regla | Comportamiento |
|---|---|
| **Promedio de variables** | Incluye el mes de cálculo (Art. 142 *a*: el derecho se adquiere al iniciar el trimestre) |
| **Vacacional que cruza el mes** | Se cuenta **una sola vez**, en el mes en que se contó originalmente. Si ese mes ya tuvo salario regular, se admite contarla en el mes de cierre; si ese mes no tiene nóminas en el sistema, no se excluye |
| **Diferencia salarial** | Las líneas `DED188` identificadas como diferencia salarial se restan del **devengado**, nunca de las alícuotas |
| **Recibos de liquidación** | Quedan excluidos del devengado |
| **Aumentos a mitad de mes** | El salario se prorratea |
| **Alícuotas** | Usan el **último salario ajustado del mes** |
| **Contratos cerrados** | Se genera su último mes y se liquida el saldo vivo |
| **Contratos previos del mismo trabajador** | Las prestaciones se heredan del contrato anterior (`_inherit_social_benefits_from_previous_contract`) |
| **Registros huérfanos** | `migrate_orphan_benefits()` consolida historiales que quedaron sin contrato asociado |
| **Días adicionales (Art. 142 *b*)** | Se registran como fotografía mensual, no como acumulado; se reinician en meses que no son de aniversario |
| **Personal temporal y destajista** | No aporta al fideicomiso; se puede excluir del reporte analítico |

---

## 5. El asistente de cálculo masivo

`wizard/social_benefits_wizard.py`

### Parámetros

| Campo | Uso |
|---|---|
| `date_from` / `date_to` | Período a calcular |
| `department_ids`, `employee_ids`, `contract_ids` | Filtros opcionales de alcance |
| `employee_type` | Filtro por tipo de trabajador |
| `mode` | **Calcular** o **Reiniciar** |
| `force_recalculation` | Sobrescribe un mes ya calculado |

### Resultados que reporta

El asistente devuelve un resumen en pantalla: contratos totales, procesados, omitidos por no tener nóminas, omitidos por estar ya calculados, errores y bloques confirmados; más un desglose por compañía y por tipo de trabajador.

### Resiliencia

El cálculo de la población completa es una operación larga. Para que un tiempo de espera agotado no pierda todo el trabajo:

- se confirma la transacción cada **150 contratos** (`COMMIT_EVERY_CONTRACTS`), invalidando la caché entre bloques;
- los registros ya confirmados sobreviven a la interrupción;
- el registro por empleado se reduce a nivel de depuración: en consola queda únicamente el total final y los errores.

Además, un caché por lote (`_build_avg_recalc_cache`) elimina las consultas repetidas de promedios, que eran el cuello de botella principal.

### Modo Reiniciar

Dos variantes internas: reinicio **por fecha** (`_action_reset_by_date`), que reconstruye el acumulado desde la suma del historial restante, y reinicio **completo** (`_action_reset_full`). El primero es el que se usa habitualmente; el segundo borra el historial.

---

## 6. Tablero analítico de prestaciones

Componentes OWL en `static/src/components/`, montados sobre la vista `sb_dashboard_view.xml`.

| Componente | Contenido |
|---|---|
| `sb_kpi_card` | Indicadores principales, con minigráfico |
| `sb_chart_section` | Tendencia mensual |
| `sb_timeline` | Últimos cálculos; cada entrada abre el registro correspondiente |
| `sb_quick_stats` | Promedio por empleado |
| `sb_dept_breakdown` | Desglose por departamento |

**Filtros disponibles:** excluir egresados, excluir temporales, tipo de fideicomiso.

**Exportación a Excel:** hoja de resumen con indicadores y hoja de **Detalle** con encabezado completo de 20 columnas, alícuotas diarias y tipo de fideicomiso. Las filas marcadas *No genera* quedan fuera por defecto.

> **Criterio de la columna "Prestaciones Generadas":** desde la versión 18.0.1.18.47 incluye los días adicionales del aniversario (Art. 142 *b*) además de los días de garantía. Antes reportaba solo la garantía.

---

## 7. Fideicomiso — archivo TXT para el banco

Módulo `nimetrix_bbva_netcash_fideicomiso` (v18.0.1.0.10). Genera el archivo de aportes al fideicomiso **a partir de las prestaciones ya generadas** en el historial, no de un cálculo paralelo.

### 7.1 Forma de pago: Apertura, Aporte o Anticipo

El campo `fideicomiso_payment_form` divide la población de manera **obligatoria y excluyente**:

| Forma | Población | Nombre del archivo |
|---|---|---|
| **Apertura** | Personas que **nunca** generaron prestaciones antes del período — primer aporte, ocurre una sola vez | `Fideicomisoapertura{DDMMAA}.txt` |
| **Aporte** | Personas **con** historia previa de prestaciones generadas | `Fideicomisoaporte{DDMMAA}.txt` |
| **Anticipo** | Anticipos y préstamos contra el fideicomiso | — |

El criterio de corte es la existencia de historia con monto generado mayor que cero en el **mes anterior** al período. Un archivo **nunca** mezcla ambas formas.

**Tres redes de detección de historia previa**, en orden: historial propio en el sistema → acumulado migrado del sistema anterior → fecha de ingreso del trabajador. La segunda y la tercera existen porque hay personal cuya historia no está completa en Odoo.

### 7.2 Personas sin cuenta en el banco del fideicomiso

No se excluyen. La cuenta de abono se completa con el prefijo del banco más la cédula rellenada a 16 dígitos, replicando el patrón de los archivos reales de apertura. Solo se omite a quien no tiene ni cuenta ni cédula.

### 7.3 Formato de línea

El formato se alineó al archivo de ejemplo que provee el propio banco y a los archivos reales previamente procesados: **campos separados por espacio, coma entre nombre y apellido, apellido sin relleno**. No se fuerza mayúsculas. El diseño de registro documental de 2022 —campos concatenados— quedó obsoleto y produce rechazos.

La salida fue verificada carácter por carácter contra el archivo del banco.

### 7.4 Filtros

| Campo | Efecto |
|---|---|
| `fideicomiso_exclude_terminated` | Excluye personal egresado |
| `fideicomiso_exclude_temporary` | Excluye personal temporal y destajista, que no aporta al fideicomiso |
| `fideicomiso_source` | Origen de los datos |
| `fideicomiso_indicator` | Indicador de pago |
| `fideicomiso_concept` | Código de concepto (`000` para aportes y apertura) |

El monto trasladado al archivo **incluye los días adicionales** del Art. 142 *b*.

Existe un módulo equivalente para el otro banco: `nimetrix_banco_exterior_fideicomiso` (v18.0.1.0.2).

---

## 8. Reconciliación de diferencias retroactivas

Uno de los aportes con mayor impacto económico del proyecto.

**Problema.** Cuando un aumento de beneficio se cargaba después de calculada la nómina, se generaba una deuda con el trabajador que ningún sistema detectaba. El sistema anterior solo reconocía el último aumento sobre la quincena inmediata, sin retroactivos; el resto se pagaba a mano como asignaciones libres, y había casos que simplemente se perdían.

**Solución.** Una regla de reconciliación por período compara día por día lo efectivamente pagado contra el historial de salario, liquida el faltante en el recibo siguiente y descuenta lo ya pagado para no duplicar. Está desplegada en las tres compañías.

**Verificación.** Casos contrastados al céntimo contra cálculos manuales, incluyendo montos que el sistema anterior nunca había pagado. El barrido inicial identificó del orden de 3.600 casos con deuda cuantificada.

---

## 9. Pruebas automatizadas

El módulo núcleo tiene **183 pruebas** (6.734 líneas). Las directamente relacionadas con prestaciones:

| Archivo | Cubre |
|---|---|
| `test_hr_contract_social_benefits_calculations.py` | Cálculo de prestaciones por contrato |
| `test_social_benefits_integration.py` | Flujo integrado |
| `test_seller_comprehensive_salary.py` | Salario integral de personal de ventas |
| `test_hr_contract_vacation_bonus.py` | Bono vacacional |
| `test_average_wage_excel_cases.py` | Casos de promedio validados contra hoja de cálculo |
| `test_recalculate_averages_wizard.py` | Asistente de recálculo de promedios |
| `test_regenerate_history_wizard.py` | Asistente de reconstrucción de historial |

Se ejecutan en cada build del ambiente de validación.
