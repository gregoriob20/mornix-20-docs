# 05 — Estado, pendientes y riesgos

> Localización de nómina ZOOM · Odoo 18 · corte **24/07/2026**
> Las cifras de esta sección fueron consultadas directamente en producción el 24/07/2026.

---

## 1. Fotografía de producción

| Indicador | Valor al 24/07/2026 |
|---|---|
| Empleados activos | **2.497** |
| Recibos con período dentro de julio 2026 | **14.587** |
| Reglas salariales activas en las tres estructuras Regulares | **579** |
| Compañías operativas | 3 (ZIS, CCZ, ZLS) |
| Versión desplegada del módulo núcleo | **18.0.1.18.47** |

### Lotes de julio 2026

Los lotes semanales del mes —vacaciones, Plus Económico, viáticos, destajistas, calidad de vida, anticipos de fondo de ahorro, gratificación nocturna— se encuentran **cerrados** en las tres compañías.

En estado *verificar* (pendientes de cierre a la fecha de corte):

| Lote | Compañía | Período |
|---|---|---|
| Nómina 30 Julio 26 | ZIS | 01/07 – 30/07 |
| BReconocimiento Talento Jul'26 | CCZ | 01/07 – 31/07 |
| BReconocimiento Talento Jul'26 | ZLS | 01/07 – 31/07 |

**Lectura:** la operación mensual está corriendo con normalidad. El día a día de nómina no depende de ningún desarrollo pendiente.

---

## 2. Qué quedó cerrado

### 2.1 Motor de cálculo

- **Días de descanso, domingos y feriados** derivados del calendario real del contrato, con zona horaria, incluida la variante donde un reposo no absorbe los descansos del período.
- **Turnos que cruzan la medianoche:** conteo por fecha local, horas por turno con descanso de cola, descarte de fragmentos degenerados en bordes de feriado.
- **Altas a mitad de mes:** se pagan los días pendientes correctamente, tanto en turno diurno como nocturno.
- **Precisión decimal:** eliminados los redondeos intermedios; `ROUND_HALF_UP` con `Decimal` en el paso final. Verificado contra cálculos manuales de liquidación.
- **Prorrateo de aumentos** a mitad de mes.

### 2.2 Prestaciones sociales

- Motor completo con historial mensual persistido (`social.benefits.history`) y traza de salario y bonos (`salary.bonus.history`).
- Promedios de 3, 6 y 12 meses calculados desde la traza, lo que hace **reproducible** el recálculo de meses pasados.
- Días adicionales del Art. 142 *b* como fotografía mensual, con reinicio en meses que no son de aniversario.
- Clasificación fijo/variable del bono nocturno resuelta por seguimiento del indicador en el contrato.
- Herencia de prestaciones entre contratos sucesivos del mismo trabajador y consolidación de registros huérfanos.
- Liquidación del saldo vivo en contratos cerrados.
- Asistente masivo resiliente: confirma cada 150 contratos, sobrevive a un tiempo de espera agotado, reporta desglose por compañía y tipo de trabajador.
- Tablero analítico OWL con filtros (egresados, temporales, tipo de fideicomiso) y exportación a Excel de 20 columnas.

### 2.3 Diferencial retroactivo

Regla de reconciliación por período que compara día a día lo pagado contra el historial de salario, liquida el faltante en el recibo siguiente y descuenta lo ya pagado. Desplegada en las tres compañías. Es el aporte de mayor impacto económico del proyecto: recuperó pagos que el sistema anterior no detectaba.

### 2.4 Generación de recibos y entradas de trabajo

- Generación de lotes **en segundo plano por cron**, por tandas, reanudable tras un reinicio del contenedor, con presupuesto de tiempo para no bloquear otras tareas programadas.
- Aislamiento del empleado que hace fallar la tanda; el resto del lote continúa.
- Avisos al analista con **causa real** por persona y diagnóstico de conflictos de entradas (quién, cuándo, qué hacer).
- Regeneración de entradas de trabajo **consciente del horario vigente en el período**, preservando ausencias validadas y protegiendo recibos ya validados.

### 2.5 Salidas bancarias

- TXT de nómina para Banco Provincial (Net Cash) y Banco Exterior, con RIF de la compañía del registro.
- TXT de fideicomiso con separación obligatoria **Apertura / Aporte**, tres redes de detección de historia previa, inclusión de personas sin cuenta en el banco del fideicomiso y formato alineado carácter por carácter al archivo que provee el banco.

### 2.6 Comunicación y reportes

- Envío automático de recibos por correo, con remitente corporativo preservado y sin duplicados.
- Reportes INCES, MINTRA, BANAVIH y de Costos y Gastos de nómina.
- Contabilización automatizada con modelo híbrido.

### 2.7 Calidad

**254 pruebas automatizadas** en los módulos de nómina: 183 en el núcleo, 38 en generación de recibos, 17 en regeneración de entradas y 16 en préstamos. Se ejecutan en cada build del ambiente de validación.

---

## 3. Pendientes abiertos

### 3.1 Incidente activo — cron de generación de PDF

**Prioridad: alta.**

El cron `Payroll: Generate pdfs` falla en producción con un error de plantilla QWeb sobre un conjunto de registros que se espera unitario (*singleton*). El fallo se origina en el reporte de recibo de pago de la **localización base**, no en los módulos de la capa ZOOM.

Detectado el 23/07/2026 al revisar los registros del servidor. **No bloquea el cálculo ni el cierre de nómina**, pero impide la generación automatizada de los PDF de recibos.

*Acción sugerida:* revisar la plantilla del reporte de recibo de pago en la localización base y asegurar que itere registro por registro.

### 3.2 Cierre de prestaciones — caso Directores

**Prioridad: alta.**

El historial de prestaciones en producción está calculado hasta **enero de 2026** (1.733 registros desde el 01/01/2026). El cierre y pago del período está a la espera de resolver el criterio de cálculo del personal de dirección, que tiene un régimen de días adicionales distinto.

*Acción sugerida:* definir con Recursos Humanos el criterio aplicable a directores y, una vez acordado, correr el asistente masivo por los meses pendientes. El material de referencia existe en la carpeta `Documentos/` (hojas de cálculo de días adicionales de directores).

### 3.3 Validación del formato de fideicomiso con el banco

**Prioridad: media.**

El archivo TXT de fideicomiso fue enviado al banco para validación de formato. Al 14/07/2026 no había respuesta formal.

*Acción sugerida:* dar seguimiento a la respuesta del banco antes de la primera corrida de apertura masiva. La salida ya fue verificada contra el archivo de ejemplo del propio banco, de modo que el riesgo es bajo, pero la confirmación escrita conviene tenerla.

### 3.4 Auditoría de la nómina Regular

**Prioridad: media.**

Queda pendiente una auditoría concepto por concepto de la estructura Regular, para dar por cerrado el día a día sin conciliación manual. La operación funciona; se trata de reducir la verificación manual residual.

---

## 4. Riesgos y deuda técnica

### 4.1 *Drift* entre estructuras espejo

**Riesgo: alto. Probabilidad: alta si no hay disciplina.**

Las estructuras 26 (ZIS), 184 (CCZ) y 217 (ZLS) deben mantenerse idénticas en lógica. Un arreglo aplicado a una sola hace que la misma nómina calcule distinto según la compañía del trabajador.

Ya ocurrió en la práctica con las reglas de SSO/RPE y con `BAS3`.

**Mitigación:** todo cambio a una regla de la Regular se aplica a las tres estructuras, y se verifica un caso testigo por compañía. Lo mismo para las tres Temporales (10, 182, 231).

### 4.2 `BAS3` en las estructuras Temporales

**Riesgo: medio.**

La regla de cantidad de días de descanso en las estructuras Temporales (10 y 182) utiliza un conteo fijo de sábados y domingos en lugar de derivarlo del calendario del contrato. En consecuencia **no respeta los calendarios de lunes a sábado**. La versión correcta —que deriva del calendario mediante el helper sin absorción de ausencias— está implementada en las estructuras Regulares.

**Mitigación sugerida:** portar la versión correcta a las tres Temporales, con la verificación caso a caso que exige un cambio de esta naturaleza.

### 4.3 Excepción documentada en `BAS144`

**Riesgo: bajo, si se conoce.**

El bloque de manejo de turnos que cruzan la medianoche en `BAS144` existe **solo en la estructura 26**. Fue un ajuste dirigido a un caso de ZIS.

**Mitigación:** no replicarlo a las otras estructuras "para homogeneizar" sin analizar antes si el caso aplica. Es la única excepción conocida a la regla de las tres estructuras espejo.

### 4.4 Dependencia de la localización base

**Riesgo: medio.**

La capa ZOOM depende de los módulos `l10n_ve_payroll_*`, que se integran como submódulo. Un cambio en esa base puede afectar el comportamiento sin que se vea en el repositorio de personalización. El incidente del cron de PDF (§3.1) es un ejemplo: el síntoma aparece en producción, la causa está en la capa inferior.

**Mitigación:** ante cualquier comportamiento inesperado, verificar también la revisión de la base, no solo la capa de personalización.

### 4.5 Despliegue: versión de manifiesto y builds concurrentes

**Riesgo: medio. Consecuencia: silenciosa.**

Dos errores de operación que se repiten y cuyo síntoma es engañoso:

1. **No subir la versión del manifiesto.** El build pasa en verde pero el cambio no se aplica en la base de datos. Parece un problema de código y es de despliegue.
2. **Subir código con un build en curso.** El build queda descartado y el despliegue nunca llega.

**Mitigación:** subir versión en cada cambio; esperar a que el servicio esté en línea antes de publicar.

### 4.6 Nombres de reglas traducibles

**Riesgo: bajo. Efecto: confusión.**

`hr.salary.rule.name` es traducible. Si se escribe desde consola en un solo idioma, la interfaz sigue mostrando el nombre anterior y parece que el cambio no se aplicó.

**Mitigación:** escribir siempre en `en_US` y `es_VE`.

### 4.7 Operaciones masivas

**Riesgo: medio.**

Con casi 2.500 trabajadores, cualquier operación sobre la población completa puede agotar memoria o tiempo de espera.

**Mitigación:** ya implementada en los procesos críticos — trabajo por tandas de 150 a 1.000 registros, confirmación e invalidación de caché entre tandas, y reanudación. Cualquier proceso nuevo debe seguir el mismo patrón.

---

## 5. Prioridades sugeridas para quien continúe

| Orden | Acción | Motivo |
|---|---|---|
| 1 | Resolver el cron de generación de PDF | Incidente activo en producción |
| 2 | Definir el criterio de directores y cerrar prestaciones | Bloquea el cierre y pago del período |
| 3 | Confirmar el formato de fideicomiso con el banco | Antecede a la primera apertura masiva |
| 4 | Portar la corrección de `BAS3` a las estructuras Temporales | Deuda técnica con impacto en cálculo |
| 5 | Completar la auditoría de la nómina Regular | Reduce verificación manual residual |
