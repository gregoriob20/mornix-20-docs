# 04 — Runbook operativo

> Localización de nómina ZOOM · Odoo 18 · corte 24/07/2026
> Documento de consulta para el día a día del analista de nómina y del soporte técnico.

---

## 1. Ciclo mensual de nómina

```
1. Verificar contratos y horarios      →  ¿altas, egresos, cambios de horario?
2. Regenerar entradas de trabajo       →  solo si hubo cambios de horario o ausencias
3. Crear el lote y generar los recibos →  en segundo plano
4. Revisar avisos del lote             →  omitidos, conflictos, bloqueados
5. Calcular los recibos                →  compute_sheet
6. Validar y confirmar el lote
7. Enviar recibos por correo           →  automático al cerrar el lote
8. Generar el TXT bancario             →  pago de nómina
9. Calcular prestaciones del mes       →  asistente masivo
10. Generar el TXT de fideicomiso      →  apertura y aporte, por separado
11. Contabilizar la nómina
```

---

## 2. Generación de recibos en segundo plano

Módulo `nx_zoom_payslip_regen_work_entries` (v18.0.2.3.0).

Con una población de casi 2.500 trabajadores, generar un lote en primer plano agota el tiempo de espera del navegador. Por eso la generación se encola y la ejecuta un cron.

### Cómo se usa

1. Crear el lote (`hr.payslip.run`) con su período.
2. Abrir el asistente de generación de recibos y seleccionar la población.
3. Marcar o desmarcar **Regenerar entradas de trabajo** (`ve_regenerate_entries`). Si el período ya tiene las entradas correctas, desmarcarlo ahorra tiempo considerable.
4. Confirmar. El lote pasa a estado de generación y el cron **NX Zoom: generar recibos de lote en segundo plano** toma el trabajo.

### Qué muestra el lote mientras trabaja

| Campo | Contenido |
|---|---|
| `ve_gen_state` | Estado de la generación |
| `ve_gen_progress` | Avance legible (procesados sobre total) |
| `ve_gen_total` / `ve_gen_done` | Contadores |
| `ve_gen_heartbeat` | Última señal de vida del proceso |
| `ve_gen_skipped_report` | Personas sin recibo, **agrupadas por causa real** |
| `ve_gen_conflict_entry_ids` | Entradas de trabajo en conflicto |
| `ve_gen_blocked_employee_ids` | Bloqueados por ausencia |
| `ve_gen_error` | Último error |

### Comportamiento ante interrupciones

- **Trabajo por tandas con confirmación.** Cada tanda se confirma; si el contenedor se reinicia, lo generado se conserva y al reintentar continúa donde quedó, sin duplicar.
- **Señal de vida.** El proceso refresca su marca de tiempo durante la regeneración de entradas. Si la marca queda vieja, el sistema entiende que el proceso murió y permite reintentar.
- **Presupuesto de tiempo.** El cron cede el turno tras un lapso acotado (5 minutos) para no bloquear las demás tareas programadas, y se vuelve a despertar por disparo explícito, no reescribiendo su próxima ejecución.
- **Aislamiento del caso que rompe.** Si un empleado hace fallar la tanda, se lo procesa aislado y el resto del lote continúa (`_ve_generate_isolated`).

### Avisos al analista

El aviso de "sin recibo" indica la **causa real** por persona, agrupada, no un texto genérico. Ante un conflicto de entradas de trabajo, el sistema informa **quién** las creó, **cuándo** y **qué hacer**. El aviso es colapsable y descartable, y descartarlo **no** vuelve a ejecutar la generación.

### Botones disponibles

| Acción | Método |
|---|---|
| Reintentar generación | `action_ve_retry_generation` |
| Cancelar generación | `action_ve_cancel_generation` |
| Ver omitidos | `action_ve_open_skipped` |
| Ver conflictos | `action_ve_open_conflicts` |
| Ver diferidos | `action_ve_open_deferred` |
| Descartar aviso | `action_ve_dismiss_skipped` |

---

## 3. Regeneración de entradas de trabajo

Módulo `nx_work_entry_regen_schedule_aware` (v18.0.1.1.8).

**El punto crítico:** regenerar entradas de trabajo de un período pasado usando el horario **actual** del contrato produce resultados incorrectos si el trabajador cambió de horario. Este módulo resuelve exactamente eso.

### Cómo elige el horario

Toma el **cambio de horario más reciente con fecha anterior al inicio del período** como calendario base, apoyándose en `nx_contract_schedule_history`. El calendario actual del contrato se usa **solo** cuando el contrato no tiene ningún historial previo.

### Salvaguardas

- **Ausencias validadas se preservan.** La regeneración no borra permisos ni reposos ya aprobados.
- **Recibos validados se protegen.** Si el período tiene recibos ya validados, se emite una advertencia estructurada en lugar de sobrescribir.
- **Sin duplicados.** Se depuran las entradas duplicadas por solapamiento.
- **Recálculo posterior.** Tras regenerar, se recalculan los recibos del período; ese recálculo está aislado para que un fallo no aborte la regeneración completa.

### Cuándo regenerar

| Situación | ¿Regenerar? |
|---|---|
| Cambio de horario del contrato en el período | **Sí** |
| Ausencia cargada después de generar el recibo | **Sí** |
| Alta a mitad de mes | **Sí** |
| Nada cambió desde la última generación | No — desmarcar la casilla |

---

## 4. Turnos que cruzan la medianoche

Los turnos nocturnos que empiezan un día y terminan al siguiente requieren tratamiento especial:

- el conteo de días se hace por **fecha local**, no por fecha UTC;
- las horas por día se calculan **por turno**, incluyendo el descanso de cola;
- se descartan los fragmentos degenerados en los bordes de feriado.

Sin este tratamiento, un trabajador nocturno aparecía con un día de más o de menos en el mes.

---

## 5. Prestaciones sociales del mes

Ver detalle funcional en el documento [03](03_PRESTACIONES_Y_FIDEICOMISO.md).

**Procedimiento**

1. Confirmar que **todos** los lotes del mes están cerrados. El cálculo lee los recibos.
2. Abrir el asistente de prestaciones sociales.
3. Fijar `date_from` y `date_to` del mes.
4. Dejar `mode` en **Calcular**.
5. Marcar `force_recalculation` **solo** si se está rehaciendo un mes ya calculado.
6. Ejecutar y revisar el resumen: procesados, omitidos por falta de nóminas, omitidos por ya calculados, errores.

**Qué esperar.** La corrida global de la población completa es larga. Confirma cada 150 contratos, de modo que una interrupción conserva lo ya procesado. En consola solo aparece el total final y los errores.

**Si algo sale mal.** El modo **Reiniciar** por fecha reconstruye el acumulado desde la suma del historial restante. Es preferible al reinicio completo, que borra el historial.

---

## 6. Archivos TXT para el banco

### 6.1 Pago de nómina

| Banco | Módulo |
|---|---|
| Banco Provincial (Net Cash) | `nimetrix_bbva_netcash` (18.0.2.0.1) |
| Banco Exterior | `nimetrix_banco_exterior` (18.0.1.0.3) |

Consideraciones:

- El RIF que se imprime es el de la **compañía del registro**, no el de la compañía del entorno del usuario (`nimetrix_export_rif_empresa`). Con tres compañías, esto importa.
- Existe separación entre archivo de mismo banco y archivo de otros bancos.
- Hay un filtro global para acotar la población del archivo.
- Los montos en cero no sobrescriben valores existentes.

### 6.2 Fideicomiso

| Banco | Módulo |
|---|---|
| Provincial | `nimetrix_bbva_netcash_fideicomiso` (18.0.1.0.10) |
| Banco Exterior | `nimetrix_banco_exterior_fideicomiso` (18.0.1.0.2) |

**Procedimiento**

1. Calcular las prestaciones del mes **antes** de generar el archivo. El TXT lee el historial de prestaciones generadas.
2. Generar **dos archivos separados**:
   - **Apertura** — quienes nunca generaron prestaciones antes del período;
   - **Aporte** — quienes ya tienen historia previa.
   Un archivo nunca mezcla ambas formas.
3. Aplicar los filtros según corresponda: excluir egresados, excluir temporales y destajistas.
4. Verificar el nombre generado: `Fideicomisoapertura{DDMMAA}.txt` o `Fideicomisoaporte{DDMMAA}.txt`.

**Formato.** Campos separados por espacio, coma entre nombre y apellido, apellido sin relleno. Este es el formato que el banco procesa; el diseño de registro documental de 2022 está obsoleto.

**Personas sin cuenta en el banco del fideicomiso.** Entran igual: la cuenta se completa con el prefijo del banco más la cédula a 16 dígitos. Solo se omite a quien no tiene ni cuenta ni cédula.

---

## 7. Envío de recibos por correo

Módulo `nx_zoom_payslip_email` (v18.0.1.3.1). Cron: **NX Zoom: auto-enviar recibos de nómina (lotes cerrados)**.

Puntos aprendidos, ya resueltos en el código:

- El módulo **no** gestiona su propio servidor SMTP. Una versión anterior lo hacía y fue la causa de una interrupción del servicio de correo. Hoy usa el servidor configurado en el sistema, fijado por filtro de dominio para preservar el remitente corporativo.
- `partner_to` se limpia para evitar correos duplicados.
- Los errores de credenciales se registran como advertencia, no como error, para no ensuciar el registro con ruido recurrente.

**Si los recibos no salen:** revisar primero el servidor de correo saliente configurado en el sistema, no el módulo.

---

## 8. Reportes legales y contabilización

| Reporte | Módulo | Frecuencia |
|---|---|---|
| INCES | `nimetrix_inces_report` | Trimestral |
| MINTRA | `nimetrix_sucursal_reporte_mintra` | Según requerimiento |
| BANAVIH / FAOV | localización base | Mensual |
| Costos y Gastos de nómina | `nx_payroll_cost_expense_report` | Mensual |

**Contabilización.** `mornix_zoom_payroll_accounting` (v18.0.3.0.0) distribuye y contabiliza con modelo híbrido. El reporte de Costos y Gastos separa ambos conceptos según la familia del cargo.

**Nota sobre BANAVIH:** los nuevos ingresos cuyo recibo cubre días del mes anterior deben quedar incluidos. Es un caso que se corrigió expresamente y conviene verificar tras cada corrida.

---

## 9. Préstamos, anticipos y fondo de ahorro

### Préstamos — `nx_zoom_payroll_plus` (v18.0.1.0.39)

Crons asociados:

| Cron | Función |
|---|---|
| Préstamos: Conciliar Cuotas con Pagos | Concilia cuotas contra pagos efectivos |
| Cierre Automático de Préstamos Pagados | Cierra los préstamos saldados |
| Cierre Automático de Cabeceras Huérfanas de Préstamos | Limpia cabeceras sin líneas |

Asistentes disponibles: pago de cuota individual, pago masivo, omisión de cuota (*skip*), reporte de préstamos, reporte de recibos Plus.

Los crons de préstamos toleran un reinicio del contenedor: distinguen una conexión de base de datos muerta de un error transitorio recuperable, y solo abortan en el primer caso.

### Anticipos — `nx_zoom_payroll_advances` (v18.0.2.2.1)

Gestión de anticipos quincenales, con botón de recálculo de promedios salariales.

### Fondo de ahorro — `nx_zoom_savings_fund` (v18.0.1.0.0)

Aportes, anticipos, préstamos, retiros e intereses. Cron mensual de intereses: **Calculate Savings Fund Interests (Ext)**.

---

## 10. Plus Económico

Estructura 12, frecuencia semanal, sin compañía asignada (global). Gestionado desde `nx_zoom_payroll_plus`.

Funcionalidad disponible: corridas con líneas persistentes y vista de lista para trazabilidad, reporte analítico en Excel, envío por correo con cron de actualización de estado de envíos, reversión de pago manual y encabezado de recibo propio.

---

## 11. Protocolo de cambios en producción

**Consultas de lectura:** usar el canal de solo lectura. Es más liviano y no puede romper nada.

**Cambios:** los cuatro pasos, siempre en orden.

| Paso | Qué hacer |
|---|---|
| **1 · Respaldo** | Guardar el estado previo en un `ir.attachment` con el JSON de los campos que se van a tocar, para las tres estructuras. Confirmar |
| **2 · Ensayo** | Aplicar, calcular **un** caso de prueba, comparar, **revertir** |
| **3 · Aplicación** | Escribir en las tres estructuras espejo. Confirmar |
| **4 · Verificación** | Revisar un caso testigo por compañía |

**Prohibido:**

- Actualizar tablas por SQL directo en producción. Los cambios se hacen sobre las reglas salariales, nunca sobre los recibos.
- Recalcular recibos en masa en producción "para probar". La prueba se hace en el ambiente de validación.

**Al desplegar:**

- Subir la versión del manifiesto en **cada** cambio, o Odoo.sh no ejecutará la actualización.
- No subir código con un build en curso: queda descartado y no se despliega.

---

## 12. Diagnóstico rápido

| Síntoma | Dónde mirar primero |
|---|---|
| "Subí el cambio y no se ve" | ¿Se subió la versión del manifiesto? ¿El build terminó en verde? |
| Días de descanso mal contados | Calendario del contrato y su zona horaria; ¿la estructura usa la variante sin absorción? |
| Trabajador sin recibo en el lote | Aviso de omitidos del lote: da la causa real por persona |
| Entradas de trabajo en conflicto | Aviso de conflictos: indica quién las creó y cuándo |
| Promedio en cero para un trabajador con bono nocturno | Seguimiento del indicador de bono fijo en el contrato |
| Diferencia de céntimos contra un cálculo manual | Redondeo: el sistema usa `ROUND_HALF_UP` sin redondeos intermedios |
| El archivo del banco es rechazado | Formato de línea y forma de pago; apertura y aporte no se mezclan |
| Recibos no llegan por correo | Servidor de correo saliente del sistema, no el módulo |
| Un cálculo cambia según la compañía | *Drift* entre estructuras espejo: comparar 26 / 184 / 217 |
