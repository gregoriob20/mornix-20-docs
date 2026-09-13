# Historial de contratos y análisis

Dos pantallas que se usan después de pagar: el historial laboral de cada
persona, y los números de la nómina cruzados como haga falta.

## El historial de contratos

**Nómina → Historial de Contratos**, o el botón **Historial** de la ficha del
empleado.

![El historial, agrupado por empleado](../img/nomina-historial-contratos.png)

> **No es una tabla aparte.** En Odoo 20 cada cambio de condiciones —un aumento,
> un cambio de cargo, una renovación— crea una **versión** del contrato, y el
> historial es esa lista. Por eso no hay nada que «archivar» a mano: el pasado
> queda solo.

La lista se abre con los contratos **archivados incluidos**. Suelen ser justo
los antiguos, que es lo que se viene a buscar.

### Paso a paso: ver la historia laboral de una persona

1. **Nómina → Historial de Contratos.** La lista viene agrupada por empleado.
2. Pulse el nombre del empleado: se despliegan todas sus versiones de contrato,
   con fecha de inicio, fin, cargo y sueldo.
3. Para llegar desde la ficha: **Empleados → ficha → botón Historial** (arriba,
   junto a «Nóminas»). Es la misma lista, ya filtrada.

## El aviso de vencimiento

Un contrato a término avisa antes de vencer, en una franja roja sobre la ficha
del empleado:

![La franja de aviso sobre la ficha del empleado](../img/nomina-aviso-vencimiento.png)

Los días de antelación salen del **periodo de aviso de vencimiento** de la
compañía. El aviso cambia de texto según el caso: «vence en N días», «vence
mañana», «vence hoy», o «venció el … y sigue en curso».

### Paso a paso: cambiar con cuánta antelación avisa

1. **Ajustes → Empleados.**
2. Bloque de contratos: **Periodo de aviso de vencimiento de contrato**, en
   días.
3. **Guardar.** Es el mismo valor que usa la tarea automática que marca los
   contratos en «Advertencia»: se configura una vez y sirve para los dos.

## Renovar un contrato

Cuando el contrato ya venció, la cabecera de la ficha enseña **Renovar
contrato**. El botón **está en el contrato y no en la ficha del empleado** a
propósito: cuando a alguien se le vence el contrato lo habitual es que el
empleado esté archivado, y entonces en la lista de Empleados no aparece. En el
historial de contratos sí.

### Paso a paso: renovar

1. **Nómina → Historial de Contratos.** Busque el contrato vencido (o abra la
   ficha del empleado y pulse **Historial**).
2. Abra el contrato. Si ya venció y no se ha renovado, arriba está **Renovar
   contrato**.
3. Se abre el asistente «Renovar contrato», con todo propuesto:
   - **Fecha de inicio**: el día siguiente al vencimiento del anterior.
   - **Cargo**: el del contrato anterior. Si lo cambia, la referencia del
     contrato nuevo se reescribe con el cargo elegido.
   - **Referencia**, **fecha de fin** y **sueldo**: ajuste lo que cambie.
   - Calendario, tipo de estructura y campos de nómina se copian del anterior
     y no se preguntan.
4. **Renovar.**
5. Resultado: el contrato vencido queda cerrado el día previo al nuevo y en el
   historial; el nuevo queda en curso, enlazado por **Contrato anterior**. El
   botón **Renovar contrato** desaparece del viejo, para que no se encadenen
   renovaciones de la renovación sin darse cuenta.

> **El recorte del vencimiento no es cosmético**: Odoo no deja que dos
> contratos del mismo empleado se solapen en fechas. Sin recortar, la
> renovación fallaría con un error del núcleo difícil de interpretar.

## El análisis de nómina

**Nómina → Informes → Análisis de nómina.**

![El análisis, por empleado y mes](../img/nomina-analisis-pivote.png)

![Recorrido: el pivote y lo que se puede cruzar con él](../media/recorrido-nomina-analisis.webm)

Un pivote sobre los recibos, con una fila por recibo y jornada. Las medidas:

| Medida | Qué suma |
|---|---|
| **Sueldo básico** | La categoría de sueldo básico |
| **Sueldo bruto** | El total de asignaciones |
| **Sueldo neto** | Lo que se paga |
| **Asignaciones** / **Deducciones** | Los dos lados del recibo |
| **Días** / **Horas** | La jornada que alimentó el cálculo |
| **Nº de recibos** | Cuántos recibos hay detrás de cada cifra |

Entran los recibos **por verificar, hechos y pagados**: la pre-nómina se quiere
ver en el análisis, que es justo para lo que se revisa.

### Paso a paso: el neto pagado por departamento y mes

1. **Nómina → Informes → Análisis de nómina.**
2. En el pivote, pulse **Total** de las filas → elija **Departamento**.
3. Pulse **Total** de las columnas → **Hasta → Mes**.
4. **Medidas → Sueldo neto** (y quite las que no quiera ver).
5. Para exportarlo, el botón de descarga del pivote lo baja como hoja de
   cálculo.

> Para el desglose **por concepto** —cestaticket, prestaciones, parafiscales,
> ISLR— está el [tablero de nómina](04-el-tablero.md): el análisis mira el
> recibo entero, el tablero mira cada línea.
