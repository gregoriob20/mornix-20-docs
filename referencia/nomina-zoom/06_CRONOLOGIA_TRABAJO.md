# 06 — Cronología del trabajo y material entregado

> Localización de nómina ZOOM · Odoo 18 · repositorio `nx-zoom` · corte 24/07/2026

---

## 1. Para qué sirve este documento

Un cambio de código dice **qué** se hizo. Rara vez dice **por qué**. Esta cronología reconstruye el razonamiento detrás de las decisiones importantes, para que quien continúe no deshaga por desconocimiento algo que se hizo por una razón.

Base: **389 cambios** registrados entre el **20 de noviembre de 2025** y el **23 de julio de 2026**.

| Período | Cambios | Foco |
|---|---:|---|
| Nov – Dic 2025 | 18 | Arranque: comisiones, aprobaciones, primeros TXT bancarios |
| Enero 2026 | 74 | Salida a producción: archivos bancarios, préstamos, fideicomiso |
| Febrero 2026 | 32 | Préstamos y cuotas, BANAVIH, reportes |
| Marzo 2026 | 24 | Días de descanso, conciliación de préstamos, MINTRA |
| Abril 2026 | 46 | Prestaciones sociales: primera arquitectura sólida |
| Mayo 2026 | 45 | Precisión decimal, promedios, auditoría contra el sistema anterior |
| Junio 2026 | 57 | Nocturnos, bono nocturno fijo/variable, fideicomiso BBVA |
| Julio 2026 | 93 | Diferencial retroactivo, generación resiliente, apertura de fideicomiso |

---

## 2. Hitos, en orden

### Noviembre – Diciembre 2025 · Arranque

Primeros módulos de apoyo: comisiones y descuentos, aprobaciones multinivel, reporte de días pendientes. Primera generación de archivos TXT para Banco Provincial con filtrado inteligente de población.

### Enero 2026 · Salida a producción

El mes de mayor volumen de cambios. La operación entró en vivo y aparecieron los casos reales.

- **Archivos bancarios:** formato Provincial y carta, separación entre archivo de mismo banco y de otros bancos, filtro global, corrección para que los montos en cero no sobrescriban valores existentes. Se sumó Banesco y Banco Exterior con autorrelleno de cédula.
- **Fideicomiso:** primer módulo de fideicomiso sobre Banco Exterior.
- **Préstamos y anticipos:** ajuste de la fecha de fin de anticipos, reestructuración de pestañas, integración con fondo de ahorro.
- **Bonos especiales:** módulo propio con empresa pagadora.
- **ARI e ISLR:** actualización del método de cálculo.

> **Nota histórica.** El 27 de enero se revirtieron siete cambios para volver a una versión estable. Es la única reversión de esa magnitud del proyecto y conviene tenerla presente al leer el historial de ese mes.

### Febrero 2026 · Préstamos y reportes legales

- **Préstamos:** recálculo, modelo para eliminar cuotas, lógica de doble pago de cuota, regeneración de cuotas, verificación del cron de cálculo, paso a estado *pagado*.
- **BANAVIH:** manejo de separadores decimales en FAOV y determinación de la fecha de egreso por estado de contrato o vencimiento.
- **Febrero como mes comercial:** ajuste para que el cálculo use 30 días.
- **Fondo de ahorro:** corrección de cálculo.
- **Historial de horarios de contrato:** primera versión.

### Marzo 2026 · Días de descanso y conciliación

- **Días de descanso:** primera corrección de fondo de la cantidad de días, con registro de depuración para poder diagnosticar en producción.
- **Préstamos:** conciliación de cuotas contra pagos, botón de pago automático, corrección del salto de cuota.
- **MINTRA:** reporte con filtro de sucursales.
- **Plus Económico:** tablero personalizado, generación automática de Excel, reversión de pago manual, reporte analítico.
- **Recibo de vacaciones:** corrección del rango de fechas.

### Abril 2026 · Prestaciones sociales

El mes en que las prestaciones sociales tomaron su forma actual.

- Condición de aniversario para la generación.
- Prestaciones asociadas al **contrato activo**.
- Configuración de bonos que entran al salario promedio.
- Métodos de feriados y vacaciones, días pendientes, saldos negativos de préstamos.
- Migración del campo de personal directivo.
- Ciclo intensivo de corrección de los helpers de nocturnos.
- Reporte de costos por nómina.
- Corrección del asistente de generación de lotes.

### Mayo 2026 · Precisión y auditoría

El mes de la conciliación contra el sistema anterior. Cada corrección responde a una diferencia concreta detectada en un caso real.

- **Precisión decimal:** eliminación de cuatro redondeos intermedios en el cálculo del salario integral y cambio a `ROUND_HALF_UP` con `Decimal`. El redondeo bancario nativo de Python producía diferencias de céntimos. Validado contra hojas de cálculo de liquidación.
- **Días adicionales (Art. 142 *b*):** llevados al campo correcto.
- **Garantía trimestral:** pasa a registrarse como fotografía mensual, no como acumulado.
- **Promedio de 12 meses:** cambia su fuente al historial de prestaciones e inyecta el salario integral del mes en curso.
- **Reinicio de días adicionales** en meses que no son de aniversario.
- **Helpers de vacaciones y feriados:** descansos en los bordes del período, manejo de zona horaria, feriado dentro de vacaciones.
- **Rendimiento:** el asistente pasa a confirmar cada 500 registros y a reportar desglose por estructura, concepto y compañía. Se corrige el uso de agrupaciones con campos relacionados, no soportadas directamente por la versión 18 del ORM: se resuelve en dos consultas.
- **Restricción de unicidad** sobre el historial, para bloquear duplicados de raíz.
- **BANAVIH:** inclusión de nuevos ingresos cuyo recibo cubre días del mes anterior.

### Junio 2026 · Nocturnos y fideicomiso bancario

- **Turnos que cruzan la medianoche:** horas por día calculadas por turno con descanso de cola; conteo por fecha local; descarte de fragmentos degenerados en bordes de feriado.
- **Altas a mitad de mes** en turno diurno: se pagan los días pendientes.
- **Regeneración de entradas de trabajo consciente del horario:** módulo nuevo. Usa el horario vigente **en el período**, no el actual del contrato; preserva ausencias validadas; depura duplicados por solapamiento.
- **Bono nocturno:** campo de porcentaje en el contrato, bono fijo por porcentaje, divisor por meses laborados; los regulares con bono variable usan la vía regular.
- **Promedio de variables:** incluye el mes de cálculo (Art. 142 *a*).
- **Conceptos de vacación que cruzan el mes:** atribuidos al mes mayoritario.
- **Salario histórico:** se recupera desde la traza mensual, lo que hace reproducible el recálculo de meses pasados.
- **Contratos cerrados:** generación del último mes y liquidación del saldo vivo, con protección contra registros gemelos.
- **Consolidación de registros huérfanos** de prestaciones.
- **Fideicomiso BBVA Net Cash:** módulo nuevo, con visibilidad de banco, cuenta y estructura según el modo.
- **Rendimiento:** caché por lote que elimina las consultas repetidas de promedios; confirmación cada 200 registros.
- **Opción Sobrescribir** para recalcular un mes ya cerrado.

### Julio 2026 · Diferencial retroactivo y operación resiliente

El mes de mayor volumen del proyecto.

**Diferencial retroactivo (semana del 6).** El aporte de mayor impacto económico. Los aumentos de beneficio cargados después del cálculo generaban una deuda que ningún sistema detectaba. Se implementó una regla de reconciliación por período que compara día a día lo pagado contra el historial, liquida el faltante en el recibo siguiente y descuenta lo ya pagado. Desplegada en las tres compañías. La identificación de la línea de diferencia salarial es flexible por diseño: Recursos Humanos la registra siempre con la misma regla pero con nombres muy variados.

**Fideicomiso (semanas del 9 y del 15).**

- Formato de línea alineado al archivo que provee el propio banco —campos separados por espacio, coma entre nombre y apellido— en lugar del diseño documental de 2022, que estaba obsoleto. Verificado carácter por carácter.
- División obligatoria **Apertura / Aporte** según la historia del mes anterior; un archivo nunca mezcla ambas.
- Las personas sin cuenta en el banco del fideicomiso dejan de excluirse: la cuenta se completa con el prefijo del banco más la cédula a 16 dígitos.
- Tres redes de detección de historia previa, incluida la del acumulado migrado del sistema anterior y la fecha de ingreso.
- Filtros para excluir egresados y personal temporal.
- El monto del archivo incluye los días adicionales del Art. 142 *b*.

**Generación de recibos resiliente (semanas del 20).**

- Generación por cron en segundo plano, por tandas con confirmación, reanudable tras un reinicio del contenedor.
- Presupuesto de tiempo de 5 minutos por pasada, para no bloquear las demás tareas programadas; el cron se despierta por disparo explícito.
- Aislamiento del empleado que rompe la tanda.
- Avisos con la **causa real** por persona, agrupados, colapsables y descartables sin volver a ejecutar.
- Diagnóstico de conflictos de entradas de trabajo: quién las creó, cuándo y qué hacer.
- Crons de préstamos endurecidos: distinguen una conexión de base de datos muerta de un error transitorio recuperable.

**Prestaciones, refinamiento final.**

- Los recibos de liquidación quedan fuera del devengado.
- Cada vacacional que cruza el mes se cuenta una sola vez.
- La clasificación fijo/variable del bono nocturno pasa a derivarse del seguimiento del indicador en el contrato, no de la traza de pagos.
- Tablero: indicador de prestaciones acumuladas en lugar de intereses, línea de tiempo navegable, filtros de egresados y temporales, Excel con encabezado completo de 20 columnas.
- La columna *Prestaciones Generadas* incorpora los días adicionales del aniversario.
- Vacaciones pendientes que no absorben descansos ni feriados.

**Otros.**

- Recibo de Plus Económico rediseñado, con trazabilidad de corridas y líneas persistentes.
- RIF de la empresa en el encabezado del recibo.
- Historial de finalización de contratos, con asistente de renovación traducido.
- Correo de recibos: eliminado el servidor SMTP gestionado por el módulo, que había causado una interrupción del servicio.

---

## 3. Decisiones cuyo motivo conviene no olvidar

| Decisión | Por qué se tomó así |
|---|---|
| Los promedios se leen de una traza mensual, no de los recibos | Permite recalcular un mes pasado y obtener el mismo resultado de entonces |
| El bono nocturno se clasifica por el indicador del contrato, no por la traza | La regla de pago liquida por turnos sin mirar el indicador; la traza mezcla fijos y variables |
| La diferencia salarial se detecta por naturaleza + nombre flexible | Recursos Humanos usa una sola regla con nombres muy variados |
| Se eliminaron los redondeos intermedios | El redondeo bancario de Python producía diferencias de céntimos contra el cálculo manual |
| La regeneración usa el horario vigente en el período | Usar el horario actual falsea todo período anterior a un cambio de turno |
| Los lotes se generan en segundo plano por cron | Con 2.500 trabajadores, el primer plano agota el tiempo de espera |
| El asistente confirma cada 150 contratos | Una interrupción no puede costar la corrida completa |
| El módulo de correo no gestiona su propio servidor SMTP | Hacerlo causó una interrupción real del servicio de correo |
| Apertura y Aporte de fideicomiso nunca se mezclan | Es un requisito del banco, no una preferencia de diseño |
| El formato del TXT sigue el archivo del banco, no el diseño de 2022 | El diseño documental está obsoleto y produce rechazos |

---

## 4. Material ya entregado a ZOOM

Estos documentos fueron elaborados y entregados a lo largo del proyecto. Se encuentran en la carpeta `Documentos/` y complementan este paquete técnico.

### 4.1 Guías de uso e inducción

| Documento | Destinatario |
|---|---|
| `Guia_Uso_Nomina_Analista_ZOOM_18Jul2026.html` | Analista de nómina |
| `Guia_Uso_Nomina_RRHH_ZOOM_17Jul2026.html` | Equipo de Recursos Humanos |
| `Guia_Facilitador_Induccion_Analistas_ZOOM_18Jul2026.html` | Quien dicte la inducción |
| `Plan_Induccion_Analistas_Nomina_ZOOM_18Jul2026.html` y su versión PDF | Coordinación |
| `Informe_Plan_Induccion_RRHH_Odoo_ZOOM_16Jul2026.html` | Coordinación |
| `Presentacion_Induccion_RRHH_Odoo_ZOOM_16Jul2026.html` | Sesión de inducción |
| `Presentacion_DiaADia_Analista_Nomina_ZOOM_18Jul2026.html` | Sesión del día a día |
| `Presentacion_Nomina_DiaADia_RRHH_ZOOM_17Jul2026.html` | Sesión del día a día |
| `Kit_Escenarios_QA_CAPACITACION_INTERNO.md` | Práctica guiada |

### 4.2 Informes de estatus

Serie de informes y presentaciones ejecutivas, cada uno con su par informe + presentación:

`Informe_Estatus_Nomina_Prestaciones_ZOOM_18Jun2026` · `Informe_Estatus_Nomina_ZOOM_01Jul2026` · `06Jul2026` · `07Jul2026` · `14Jul2026`

### 4.3 Informes de avance y minutas

| Documento |
|---|
| `Informe Semanal Nómina — 13 al 24 de Abril 2026.pdf` |
| `Informe_Semanal_Nomina_28Abr_05May2026.html` |
| `Informe_Semanal_Nomina_11_18May2026.html` |
| `Informe Semanal de Estabilización — 11 al 18 May 2026.pdf` |
| `Presentacion_Avances_Semana_18-25_Mayo2026.html` |
| `Informe_General_Minuta_ZOOM_07May2026.html` y `MINUTA-VISITA ZOOM 07 Mayo 2026.docx` |
| `Informe_General_Migracion_ZOOM_Mayo2026.html` |
| `Informe_Requerimiento_Maestro_Formulas_ZOOM_Jun2026.html` y su presentación |
| `Plan de Migración — Nómina Enterprise · ZOOM Internacional.pdf` |

### 4.4 Material de validación y conciliación

Hojas de cálculo de conciliación contra el sistema anterior, casos de prueba de prestaciones, análisis de diferencias, plantillas de documentación y reportes de validación por período. Todo en `Documentos/`, nombrado por fecha y concepto.

---

## 5. Cierre

La solución está en producción, operando con normalidad en tres compañías y cerca de 2.500 trabajadores. Los pendientes conocidos están inventariados en el documento [05](05_ESTADO_PENDIENTES_Y_RIESGOS.md), con su prioridad y una acción sugerida para cada uno.

El equipo de Recursos Humanos ya recibió formación sobre el uso del sistema y opera el ciclo mensual de forma autónoma.
