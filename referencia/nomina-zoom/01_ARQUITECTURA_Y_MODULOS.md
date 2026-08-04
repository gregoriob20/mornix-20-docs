# 01 — Arquitectura y módulos

> Localización de nómina ZOOM · Odoo 18 · repositorio `nx-zoom` · corte 24/07/2026

---

## 1. Modelo en capas

La solución de nómina de ZOOM está organizada en tres capas superpuestas. Entender esta separación es lo primero, porque determina **dónde debe hacerse cada cambio**.

```
┌─────────────────────────────────────────────────────────────┐
│  CAPA C — Personalización ZOOM        (repositorio nx-zoom) │
│  nx_zoom_*, nx_*, nimetrix_*, mornix_*                      │
│  Reglas de negocio propias de ZOOM, TXT bancarios,          │
│  prestaciones, préstamos, entradas de trabajo, reportes.    │
├─────────────────────────────────────────────────────────────┤
│  CAPA B — Localización venezolana        (l10n_ve_payroll_*)│
│  Conceptos comunes a Venezuela: recibos, bonos, promedios,  │
│  fondo de ahorro, ARI/ISLR, BANAVIH, HCM.                   │
├─────────────────────────────────────────────────────────────┤
│  CAPA A — Odoo 18 estándar                                  │
│  hr, hr_contract, hr_payroll, hr_holidays,                  │
│  hr_work_entry_contract, hr_payroll_account.                │
└─────────────────────────────────────────────────────────────┘
```

**Regla de oro.** Un ajuste que sea propio de ZOOM va en la capa C. Si se toca la capa B se afecta a otras implantaciones que comparten esa base. Prácticamente toda la personalización de estos meses vive en la capa C, por diseño.

---

## 2. El módulo núcleo

### `nx_zoom_review_l10n_ve_payroll` — v18.0.1.18.47

Es el corazón de la solución: concentra el motor de cálculo, la trazabilidad histórica y el tablero de prestaciones.

| Métrica | Valor |
|---|---|
| Archivos Python | 46 |
| Líneas de Python | 17.625 |
| Vistas XML | 28 |
| Pruebas automatizadas | 183 (6.734 líneas de test) |

**Estructura**

```
nx_zoom_review_l10n_ve_payroll/
├── hooks.py                 → post_init_link_default_rules (enlaza reglas por defecto al instalar)
├── models/                  → 21 modelos (ver tabla siguiente)
├── wizard/                  → 3 asistentes
├── report/                  → recibo de nómina (payroll_receipt_payment.xml)
├── data/                    → conceptos salariales, tabla de vacaciones, crons
├── security/                → ir.model.access.csv
├── static/src/components/   → tablero OWL de prestaciones (6 componentes)
├── tests/                   → 18 archivos de prueba
└── i18n/                    → traducciones
```

**Modelos, por peso**

| Modelo (archivo) | Líneas | Responsabilidad |
|---|---:|---|
| `hr_contract.py` | 4.080 | Motor de prestaciones sociales, salarios integrales y promedios |
| `hr_payslip.py` | 2.040 | Helpers de días, feriados, brechas salariales y promedios del recibo |
| `social_benefits_history.py` | 1.052 | Historial mensual de prestaciones por contrato |
| `hr_payslip_holidays.py` | 301 | Feriados y rangos de vacaciones |
| `salary_bonus_history.py` | 141 | Traza mensual de bonos y salario, insumo de los promedios |
| `salary_concept.py` | 133 | Agrupador configurable de reglas salariales |
| `hr_employee_advances_loans_discounts_lines.py` | 73 | Cuotas de préstamos y anticipos |
| `average_wage_lines.py` | 63 | Líneas de salario promedio |
| `goal_history.py` | 44 | Historial de metas |
| Resto (12 modelos) | < 40 c/u | Familia, tabla de vacaciones, clase de cargo, traslados, configuración |

**Asistentes (`wizard/`)**

| Asistente | Función |
|---|---|
| `social_benefits_wizard` | Cálculo masivo de prestaciones sociales por período |
| `recalculate_averages_wizard` | Recálculo de salarios promedio |
| `regenerate_history_wizard` | Reconstrucción del historial de salarios y bonos |

**Dependencias declaradas**

```
base · hr_holidays
l10n_ve_payroll_advances_loans_discounts
l10n_ve_payroll_bonus
l10n_ve_payroll_employee_family_information
l10n_ve_payroll_average_wage
l10n_ve_payroll_hr_payroll_receipt_payment
l10n_ve_payroll_hr_plan_accumulation_vacation
l10n_ve_payroll_res_config_settings
l10n_ve_payroll_social_benefit_liabilities_fields
nx_zoom_hr_employee_custom
```

---

## 3. Catálogo de módulos

### 3.1 Núcleo de nómina

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nx_zoom_review_l10n_ve_payroll` | 18.0.1.18.47 | Motor de cálculo, prestaciones, promedios, tablero |
| `nx_zoom_plus` | 18.0.8.5.3 | Implementación ZOOM: exportación de pagos, cartas, filtros globales |
| `nx_zoom_extended` | 18.0.7.6.2 | Extensiones de empleado y HCM, datos familiares, sucursales |
| `nx_zoom_hr_employee_custom` | 18.0.0.2.6 | Campos propios de ZOOM en el empleado |
| `nimetrix_zoom_gross_wage` | 18.0.1.0.0 | Cálculo del salario bruto propio de ZIS |

### 3.2 Préstamos, anticipos y beneficios

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nx_zoom_payroll_plus` | 18.0.1.0.39 | Préstamos y cuotas, Plus Económico, permisos masivos por sucursal, recibos personalizados |
| `nx_zoom_payroll_advances` | 18.0.2.2.1 | Anticipos quincenales; botón de recálculo de promedios |
| `nx_zoom_savings_fund` | 18.0.1.0.0 | Fondo de ahorro: aportes, anticipos, retiros e intereses |
| `nx_special_bonus` | 18.0.1.0.0 | Bonos especiales y bonos por metas, en pestaña separada |

### 3.3 Entradas de trabajo y generación de recibos

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nx_work_entry_regen_schedule_aware` | 18.0.1.1.8 | Regeneración forzada de entradas de trabajo respetando el **horario vigente en el período** y sin dañar recibos validados |
| `nx_zoom_payslip_regen_work_entries` | 18.0.2.3.0 | Regeneración opcional desde el asistente de generación de recibos + **generación de lotes en segundo plano por cron**, reanudable |
| `nx_contract_schedule_history` | 18.0.1.0.0 | Historial de cambios de horario en el contrato (insumo del anterior) |
| `nx_zoom_contract_schedule_edit` | 18.0.1.1.0 | Edición controlada del horario del contrato |
| `nx_zoom_contract_calendar_copy` | 18.0.1.0.0 | Preserva el horario al duplicar un contrato |

### 3.4 Vacaciones, feriados y ausencias

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nx_zoom_vacation_sundays` | 18.0.1.0.0 | Campo *Cantidad de Domingos en Vacaciones*; se reinicia al cerrar |
| `nimetrix_vacations_employee_filter` | — | Filtro de vacaciones por empleado |

### 3.5 Exportación bancaria y fideicomiso

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nimetrix_bbva_netcash` | 18.0.2.0.1 | TXT de nómina Banco Provincial (Net Cash) |
| `nimetrix_bbva_netcash_fideicomiso` | 18.0.1.0.10 | TXT de aportes a fideicomiso a partir de las prestaciones generadas |
| `nimetrix_banco_exterior` | 18.0.1.0.3 | TXT de nómina y afiliación, Banco Exterior |
| `nimetrix_banco_exterior_fideicomiso` | 18.0.1.0.2 | Fideicomiso Banco Exterior (apertura y aporte) |
| `nimetrix_export_rif_empresa` | 18.0.1.0.0 | Usa el RIF de la compañía del registro, no la del entorno, al generar TXT |

### 3.6 Reportes legales y contabilización

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `mornix_zoom_payroll_accounting` | 18.0.3.0.0 | Distribución y contabilización automatizada de nóminas, modelo híbrido |
| `nx_payroll_cost_expense_report` | 18.0.1.0.0 | Totalización por concepto salarial separando Costos y Gastos según familia de cargos |
| `nimetrix_inces_report` | 18.0.1.0.1 | Reporte consolidado para la declaración INCES trimestral |
| `nimetrix_sucursal_reporte_mintra` | 18.0.1.0.0 | Filtro de sucursales para el reporte MINTRA |
| `nx_zoom_payslip_report_fix` | 18.0.1.0.1 | Corrige columnas de monto y cantidad en el recibo cuando la cantidad es 1 |
| `nimetrix_fix_report_layout` | 18.0.1.0.2 | RIF de la empresa en el encabezado del recibo |

### 3.7 Comunicación

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nx_zoom_payslip_email` | 18.0.1.3.1 | Envío de recibos por correo: remitente correcto, sin duplicados, credenciales robustas |

### 3.8 Datos maestros de personal

| Módulo | Versión | Responsabilidad |
|---|---|---|
| `nx_zoom_employee_intercompany_transfer` | 18.0.1.0.0 | Traslado de empleados entre compañías |
| `nx_zoom_hr_education_fields` | 18.0.1.0.0 | Campos de educación del empleado |
| `nimetrix_other_entries_contracts` | 18.0 | Campos adicionales en contratos |
| `nimetrix_employee_contract_seniority_task` | 0.1 | Antigüedad del contrato |
| `nimetrix_sucursales_categorias` | — | Sucursales y categorías (transversal) |

---

## 4. Grafo de dependencias de los módulos críticos

```
nx_zoom_review_l10n_ve_payroll  (núcleo)
   ├── l10n_ve_payroll_advances_loans_discounts
   ├── l10n_ve_payroll_bonus
   ├── l10n_ve_payroll_average_wage
   ├── l10n_ve_payroll_hr_payroll_receipt_payment
   ├── l10n_ve_payroll_hr_plan_accumulation_vacation
   ├── l10n_ve_payroll_social_benefit_liabilities_fields
   ├── l10n_ve_payroll_res_config_settings
   ├── l10n_ve_payroll_employee_family_information
   ├── nx_zoom_hr_employee_custom
   └── hr_holidays

   ↑ dependen del núcleo:
   ├── nx_zoom_payroll_advances
   ├── nx_zoom_vacation_sundays
   ├── nimetrix_zoom_gross_wage
   ├── nimetrix_bbva_netcash_fideicomiso
   └── nx_special_bonus  (+ nx_zoom_plus)

nx_zoom_payslip_regen_work_entries
   └── nx_work_entry_regen_schedule_aware
          └── nx_contract_schedule_history

nx_zoom_payroll_plus
   ├── nx_zoom_extended
   ├── bi_odoo_multi_branch_hr
   ├── hr_work_entry_contract_enterprise
   └── l10n_ve_payroll_advances_{loans_discounts, automated_amounts}
```

**Consecuencia práctica:** un cambio en el núcleo obliga a revisar los cinco módulos que dependen de él. Un cambio en `nx_contract_schedule_history` se propaga a toda la cadena de regeneración de entradas de trabajo.

---

## 5. Tareas programadas (crons)

| Cron | Módulo |
|---|---|
| NX Zoom: generar recibos de lote en segundo plano | `nx_zoom_payslip_regen_work_entries` |
| NX Zoom: auto-enviar recibos de nómina (lotes cerrados) | `nx_zoom_payslip_email` |
| Compute Vacation Bonus Days | `nx_zoom_review_l10n_ve_payroll` |
| Préstamos: Conciliar Cuotas con Pagos | `nx_zoom_payroll_plus` |
| Cierre Automático de Préstamos Pagados | `nx_zoom_payroll_plus` |
| Cierre Automático de Cabeceras Huérfanas de Préstamos | `nx_zoom_payroll_plus` |
| Plus Económico: Actualizar estado de envíos | `nx_zoom_payroll_plus` |
| Calculate Savings Fund Interests (Ext) | `nx_zoom_savings_fund` |

Los crons de préstamos y el de generación de recibos fueron endurecidos en julio de 2026 para **sobrevivir a un reinicio del contenedor** sin abortar el lote: distinguen una conexión muerta de un error transitorio recuperable y reanudan donde quedaron. Detalle en el documento 04.

---

## 6. Ramas y despliegue

El repositorio opera sobre Odoo.sh con el siguiente flujo:

| Rama | Uso |
|---|---|
| `Production` | Producción |
| `qa_nomina` | Validación funcional previa |
| `rel_DDMMYYYY` | Rama de entrega del día; se integra por *pull request* |
| `feat_*` | Desarrollo de una funcionalidad puntual |

### Dos reglas operativas que no son opcionales

**1. Subir la versión del manifiesto en CADA cambio.**
Odoo.sh solo ejecuta la actualización del módulo (`-u`) si detecta un cambio de versión en `__manifest__.py`. Si se modifica código o datos sin subir la versión, el build pasa en verde pero **el cambio no se aplica en la base de datos**. Es la causa número uno de "lo subí y no se ve".

**2. No hacer *push* mientras haya un build corriendo.**
El build en curso queda en estado `DROPPED` y el despliegue no llega. Se debe esperar a que Odoo esté en línea, y recién entonces confirmar y subir.

---

## 7. Convenciones del código

- **Idioma:** identificadores, nombres de método y comentarios en inglés; textos de interfaz en español.
- **Prefijo `_zoom_`:** los métodos privados propios de la personalización ZOOM llevan ese prefijo, para distinguirlos de las extensiones de la localización base.
- **Nombres de reglas salariales traducibles:** el campo `name` de `hr.salary.rule` es traducible. Si se escribe desde consola, hay que escribirlo en `en_US` **y** en `es_VE` usando `with_context(lang=...)`; de lo contrario la interfaz sigue mostrando el nombre viejo.
- **Pruebas:** los módulos críticos llevan suite propia. Total a la fecha: 254 pruebas — 183 en el núcleo, 38 en generación de recibos, 17 en regeneración de entradas, 16 en préstamos.
