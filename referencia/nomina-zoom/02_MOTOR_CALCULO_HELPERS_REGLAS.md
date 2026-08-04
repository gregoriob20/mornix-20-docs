# 02 — Motor de cálculo, helpers y reglas salariales

> Localización de nómina ZOOM · Odoo 18 · corte 24/07/2026
> Datos de estructuras y reglas verificados contra producción el 24/07/2026.

---

## 1. Cómo se calcula un recibo: el modelo mental

En Odoo, una regla salarial no contiene lógica compleja: contiene una **expresión corta** que se evalúa en un entorno restringido (`safe_eval`). Toda la lógica pesada vive en **métodos Python del módulo** —los *helpers*— y la regla se limita a invocarlos.

```
hr.payslip  ──► compute_sheet()
                   │
                   ├─ para cada hr.salary.rule de la estructura:
                   │     condition_python      →  ¿aplica esta regla?
                   │     amount_python_compute →  ¿cuánto vale?
                   │            │
                   │            └─► payslip.get_rest_days()
                   │                payslip.get_holidays()
                   │                contract._calculate_comprehensive_salary()
                   │                …  ← AQUÍ está la lógica real
                   │
                   └─ resultado: hr.payslip.line
```

**Consecuencia práctica:** ante un número equivocado en un recibo, el problema casi nunca está en la regla. Está en el helper que la regla invoca, o en los datos de entrada (calendario del contrato, entradas de trabajo, historial de salario).

### Restricciones del entorno `safe_eval`

Dentro de `amount_python_compute` y `condition_python` **no se puede**:

- importar módulos (`import`);
- usar atributos especiales (`__class__`, `__dict__`, y similares);
- asignar atributos sobre objetos.

**Sí** están disponibles en el diccionario local: `payslip`, `contract`, `employee`, `categories`, `result`, `result_qty`, `date`, `datetime` y `relativedelta`. Cualquier cosa que no se pueda expresar con eso debe implementarse como helper en el módulo y llamarse desde la regla.

---

## 2. Compañías y estructuras

Producción opera con **tres compañías**. Cada una tiene su juego paralelo de estructuras.

| Sigla | Compañía | `company_id` | Regular | Temporal | Vacaciones |
|---|---|---:|---:|---:|---:|
| **ZIS** | ZOOM INTERNATIONAL SERVICES, C.A. | 3 | **26** | 10 | 31 |
| **CCZ** | CASA DE CAMBIO ZOOM, C.A. | 1 | **184** | 182 | 189 |
| **ZLS** | ZOOM LOGISTICS SERVICES, C.A. | 2 | **217** | 231 | 223 |

Estructuras adicionales:

| ID | Estructura | Compañía | Frecuencia |
|---:|---|---|---|
| 48 | Nómina Gratificación Nocturna | ZIS (3) | Semanal |
| 12 | Nómina Plus Económico | *global* (sin compañía) | Semanal |
| 178 | Nómina Viáticos por Transporte | CCZ (1) | Semanal |

> **CCZ es Casa de Cambio ZOOM.** Conviene fijarlo: la sigla se confunde con facilidad y un error aquí cambia de compañía un cálculo.

### 2.1 La regla de oro: las tres estructuras espejo

Las estructuras **26 / 184 / 217** son funcionalmente equivalentes: la misma nómina Regular en tres compañías distintas. A la fecha de corte suman **579 reglas salariales activas** (193 solo en la 26).

**Todo arreglo a una regla de la Regular debe aplicarse a las tres.** Aplicarlo en una sola produce *drift*: la misma nómina calcula distinto según la compañía del trabajador. Ya ocurrió en la práctica con las reglas de SSO/RPE y con `BAS3`.

Lo mismo aplica al par de estructuras Temporales (10 / 182 / 231).

**Excepción documentada:** el bloque de manejo de turnos que cruzan la medianoche en la regla `BAS144` existe **solo en la estructura 26**. No debe replicarse a ciegas: fue un ajuste dirigido a un caso de ZIS.

---

## 3. Familias de reglas

Los códigos siguen una convención por prefijo y el número corresponde a la secuencia de cálculo:

| Prefijo | Naturaleza | Ejemplos verificados en producción |
|---|---|---|
| `BAS*` | Cantidades base (días, no montos) | `BAS3` Cantidad Días De Descansos · `BAS144` Días Liquidados |
| `BASIC*` | Asignaciones | `BASIC145` Salario · `BASIC150` Día Feriado · `BASIC167` Trabajo Fuera de Oficina |
| `DED*` | Deducciones | `DED188` Deducción Libre |
| `COMP*` | Aportes y retenciones | `COMP196` Retención I.S.L.R. |

El orden de `sequence` importa: las reglas de cantidad (`BAS*`) se calculan antes que las de monto, porque estas últimas las consumen.

---

## 4. Catálogo de helpers

### 4.1 Helpers del recibo — `models/hr_payslip.py` (2.040 líneas)

Todos operan sobre un recibo (`self.ensure_one()`) y derivan del **calendario del contrato**, respetando su zona horaria. Esto es central: los cálculos de días no se hacen sobre el calendario gregoriano sino sobre el horario efectivamente asignado al trabajador.

**Días de descanso y domingos**

| Helper | Qué devuelve |
|---|---|
| `get_rest_days()` | Días de descanso del período según el calendario del contrato |
| `get_rest_days_no_leave_absorb()` | Variante para estructuras donde **un reposo o ausencia aprobada NO absorbe** los descansos del período. Es la versión correcta para las Regulares |
| `get_rest_days_vacation()` | Días de descanso dentro de un período vacacional |
| `get_sundays()` | Domingos del período según calendario y zona horaria |
| `get_sundays_vacation()` | Domingos comprendidos en vacaciones |

> La diferencia entre `get_rest_days()` y `get_rest_days_no_leave_absorb()` es una de las decisiones funcionales más delicadas del sistema: determina si a un trabajador de reposo se le pagan o no sus sábados y domingos. La variante sin absorción deriva el resultado del calendario real; una versión anterior usaba un conteo fijo que no respetaba calendarios de lunes a sábado.

**Vacaciones y feriados** — `models/hr_payslip_holidays.py`

| Helper | Qué devuelve |
|---|---|
| `get_holidays()` | Feriados del período |
| `get_holidays_vacation()` | Feriados comprendidos dentro de las vacaciones |
| `get_total_vacation_days()` | Días totales de vacaciones del recibo |
| `get_worked_days_vacation()` | Días efectivamente trabajados dentro del rango vacacional |
| `_get_vacation_date_range()` / `_get_vacation_legal_date_range()` | Rango real y rango legal del disfrute |

**Días y salario**

| Helper | Qué devuelve |
|---|---|
| `calculate_days_between_dates()` | Días pendientes según la frecuencia de la estructura: semanal 7, quincenal 15, mensual **30** |
| `get_employee_last_salary(date_from, date_to)` | Último salario vigente del trabajador en el rango |
| `_get_wage_at_date(ref_date)` | Salario vigente en una fecha puntual |
| `calculate_prorated_daily_rate()` | Tarifa diaria prorrateada |
| `_commercial_month_days(period_start, period_end, year, month)` | Días del mes comercial |
| `_count_working_days_in_range(date_from, date_to)` | Días laborables del rango |

**Brecha salarial (*gap*)** — mecanismo que reconcilia lo pagado contra lo que correspondía cuando un aumento se carga después del cálculo

| Helper | Qué devuelve |
|---|---|
| `compute_gap_calendar()` | Brecha por calendario |
| `compute_gap_breakdown()` | Desglose de la brecha, día por día |
| `calculate_gap_mixed_amount()` | Monto de la brecha en escenarios mixtos |

**Bono nocturno y promedios**

| Helper | Qué devuelve |
|---|---|
| `_get_bono_noct_rate()` | Porcentaje de bono nocturno aplicable |
| `_get_night_shift_start_date(search_until)` | Inicio del turno nocturno |
| `_calculate_average_for_type(average_type)` | Promedio salarial de un tipo |
| `_calculate_all_averages()` | Recalcula todos los promedios del recibo |
| `_meets_conditions_for_average_salary()` | Determina si el recibo entra en el cálculo de promedios |

**Otros**

| Helper | Qué hace |
|---|---|
| `action_payslip_done()` | Extiende el cierre del recibo: dispara historial y promedios |
| `_update_payslip_rules_to_contract()` | Sincroniza las reglas del recibo con las del contrato |
| `_resolve_salary_concept()` | Resuelve el concepto salarial agrupador |
| `_compute_leave_days_from_we(leave_code)` | Días de ausencia derivados de las entradas de trabajo |
| `_build_avg_cache_by_ref_date(payslips)` | Caché de promedios por fecha de referencia (evita consultas N+1) |

### 4.2 Helpers del contrato — `models/hr_contract.py` (4.080 líneas)

Aquí vive el motor de prestaciones y salarios integrales. Detalle funcional completo en el documento [03](03_PRESTACIONES_Y_FIDEICOMISO.md).

**Salario integral**

| Helper | Función |
|---|---|
| `_calculate_comprehensive_salary()` | Salario integral, caso general |
| `_calculate_comprehensive_salary_seller()` | Variante para personal de ventas (comisiones variables) |
| `_calculate_comprehensive_salary_regular_with_night_bonus()` | Variante con bono nocturno |
| `_calculate_comprehensive_salary_temporary()` | Variante para personal temporal y destajista |
| `_get_alicuots()` | Alícuotas de bono vacacional y utilidades |
| `_get_zoom_regular_aliquot_bases()` | Bases sobre las que se calculan las alícuotas |

> **Precisión decimal.** Estas funciones eliminan redondeos intermedios y aplican `ROUND_HALF_UP` con `Decimal` en el paso final, en lugar del redondeo bancario nativo de Python. Sin ese cambio aparecían diferencias de céntimos contra el sistema anterior.

**Promedios y trazabilidad**

| Helper | Función |
|---|---|
| `_get_zoom_avg_from_trace(...)` | Promedio de un concepto desde la traza mensual |
| `_get_zoom_benefits_avg(...)` | Promedio de prestaciones por tipo |
| `_get_zoom_wage_at_date(...)` / `_get_zoom_last_wage_in_month(...)` | Salario histórico en una fecha o al cierre del mes |
| `_get_12month_avg_integral_salary(...)` | Promedio de salario integral de 12 meses |
| `_get_seller_avg_months(...)` | Promedio de comisiones del personal de ventas |
| `_build_avg_recalc_cache(...)` | Caché por lote para el recálculo masivo |
| `_sync_salary_bonus_history(...)` | Sincroniza la traza cuando cambia un bono o el salario |

**Bono nocturno: fijo vs variable**

| Helper | Función |
|---|---|
| `_zoom_night_bonus_fixed_timeline(contracts)` | Línea de tiempo del indicador de bono fijo |
| `_zoom_nb_fixed_state_from_timeline(...)` | Estado fijo/variable en un mes dado |
| `_zoom_night_bonus_is_fixed_for_month(...)` | Decisión final para el mes |
| `_has_variable_zoom_night_bonus(...)` | Detecta bono nocturno variable |
| `_get_avg_zoom_night_bonus_from_history(...)` | Promedio del bono variable |

> La clasificación fijo/variable **se deriva del seguimiento del indicador en el contrato**, no de la presencia del concepto en la traza. La razón: la regla de pago del bono nocturno liquida por turnos trabajados sin mirar el indicador, de modo que la traza contiene pagos tanto de fijos como de variables. Usar la traza como criterio clasificaba erróneamente como fijos a trabajadores variables y los dejaba fuera del promedio.

**Diferencia salarial**

`_zoom_is_salary_diff_line(line)` identifica las líneas de *diferencia salarial* que deben restarse del devengado de prestaciones. Recursos Humanos las registra siempre con la regla `DED188` (*Deducción Libre*) pero con nombres muy variados —"Dif Salarial", "DIF SALARIO", "dif salario"…—, por lo que la detección combina **naturaleza de deducción + coincidencia flexible del nombre**, y no una comparación exacta de texto.

Estas diferencias se restan del **devengado**, nunca de las alícuotas: estas siguen calculándose sobre el salario completo.

---

## 5. Criterios de cálculo aplicados

| Criterio | Regla aplicada | Fundamento |
|---|---|---|
| Mes comercial | **30 días**; año de **360 días** | LOTTT, Art. 122 |
| Garantía de prestaciones | 15 días por trimestre, sobre el último salario devengado | LOTTT, Art. 142, literal *a* |
| Días adicionales | 2 días por año después del primer año, acumulativos hasta 30 | LOTTT, Art. 142, literal *b* |
| Cálculo retroactivo al terminar | 30 días por año de servicio o fracción superior a 6 meses | LOTTT, Art. 142, literal *c* |
| Monto a pagar | El **mayor** entre garantía acumulada (a + b) y cálculo retroactivo (c) | LOTTT, Art. 142, literal *d* |
| Intereses sobre prestaciones | Tasa publicada por el BCV | LOTTT, Art. 143 |
| Febrero | Se ajusta a 30 días comerciales para el cálculo | Mes comercial |

---

## 6. Cómo modificar una regla salarial en producción

Procedimiento obligatorio. Los cuatro pasos, en orden, sin saltarse ninguno.

**1 · Respaldo.** Antes de escribir nada, guardar el estado previo de la regla en un `ir.attachment` con el JSON de `amount_python_compute` y `condition_python`, para las tres estructuras. Confirmar la transacción.

**2 · Ensayo (*dry-run*).** Aplicar el cambio, recalcular **un** recibo de prueba, comparar el resultado y **revertir**. Nunca recalcular recibos en masa en producción para "ver si funciona": el ensayo real se hace en el ambiente de validación.

**3 · Aplicación.** Escribir en las tres estructuras espejo y confirmar.

**4 · Verificación.** Revisar un caso testigo por compañía y comparar contra el valor esperado.

**Advertencia sobre el nombre de la regla.** `hr.salary.rule.name` es un campo traducible. Al escribirlo desde consola hay que hacerlo en `en_US` **y** en `es_VE` usando `with_context(lang=...)`. Si solo se escribe uno, la interfaz sigue mostrando el nombre anterior.

---

## 7. Volumen en producción a la fecha de corte

| Indicador | Valor |
|---|---|
| Empleados activos | **2.497** |
| Reglas activas en las tres estructuras Regulares | **579** |
| Reglas activas en la estructura 26 (ZIS Regular) | **193** |
