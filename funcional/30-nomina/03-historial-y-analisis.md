# Historial de contratos y análisis

Dos pantallas que se usan después de pagar: el historial laboral de cada
persona, y los números de la nómina cruzados como haga falta.

## El historial de contratos

**Nómina → Historial de Contratos**, o el botón **Historial** de la ficha del
empleado.

![El historial, agrupado por empleado](../img/nomina-historial-contratos.png)

**Para qué**: reconstruir la vida laboral de una persona —cuánto ganaba en cada
momento, qué cargo tenía, cuándo cambió— sin buscar en papeles. Es lo que se
necesita para liquidar prestaciones, para una inspección del trabajo o para
responder «¿desde cuándo cobra esto?».

**Por qué no es una tabla aparte**: en Odoo 20 cada cambio de condiciones —un
aumento, un cambio de cargo, una renovación— crea una **versión** del contrato,
y el historial es esa lista. Por eso no hay nada que «archivar» a mano: el
pasado queda solo, y ninguna versión se pisa.

La lista se abre con los contratos **archivados incluidos**. Suelen ser justo
los antiguos, que es lo que se viene a buscar.

### Paso a paso: ver la historia laboral de una persona

1. **Nómina → Historial de Contratos.** La lista viene agrupada por empleado.
2. Pulse el nombre del empleado: se despliegan todas sus versiones de contrato,
   con fecha de inicio, fin, cargo y sueldo.
3. Para llegar desde la ficha: **Empleados → ficha → botón Historial** (arriba,
   junto a «Nóminas»). Es la misma lista, ya filtrada.

**Ejemplo.** Argenis Rondón, tras el aumento de octubre:

| Versión | Inicio | Fin | Cargo | Sueldo | Cestaticket |
|---|---|---|---|---:|---:|
| 1 | 01/01/2026 | 30/09/2026 | Técnico de mantenimiento | 16.500,00 | 4.000,00 |
| 2 | 01/10/2026 | — | Técnico de mantenimiento | 18.000,00 | 4.000,00 |

Las prestaciones de 2026 se liquidan con 5 días de salario por mes: nueve meses
a 16.500 (550,00 por día → 2.750,00 al mes) y tres a 18.000 (600,00 por día →
3.000,00 al mes). Sin las dos versiones, alguien tendría que recordar cuándo fue
el aumento.

## El aviso de vencimiento

Un contrato a término avisa antes de vencer, en una franja roja sobre la ficha
del empleado:

![La franja de aviso sobre la ficha del empleado](../img/nomina-aviso-vencimiento.png)

**Para qué**: que el vencimiento de un contrato a tiempo determinado no
sorprenda. Si el contrato vence y la persona sigue trabajando sin renovarlo, la
ley lo convierte en indefinido; si la empresa no quería eso, se enteró tarde.

Los días de antelación salen del **periodo de aviso de vencimiento** de la
compañía. El aviso cambia de texto según el caso: «vence en N días», «vence
mañana», «vence hoy», o «venció el … y sigue en curso».

### Paso a paso: cambiar con cuánta antelación avisa

1. **Ajustes → Empleados.**
2. Bloque de contratos: **Periodo de aviso de vencimiento de contrato**, en
   días.
3. **Guardar.** Es el mismo valor que usa la tarea automática que marca los
   contratos en «Advertencia»: se configura una vez y sirve para los dos.

**Ejemplo.** Periodo de aviso: **30 días**. Un contrato con fin el 30/09/2026
empieza a avisar el 31/08 («vence en 30 días»), el 29/09 dice «vence mañana», el
30 «vence hoy» y desde el 1 de octubre, si nadie lo renovó, «venció el
30/09/2026 y sigue en curso». Con 15 días de aviso en vez de 30, la franja
aparecería el 15/09; el valor bueno es el que le da tiempo a Recursos Humanos de
decidir y firmar.

## Renovar un contrato

Cuando el contrato ya venció, la cabecera de la ficha enseña **Renovar
contrato**.

**Por qué el botón está en el contrato y no en la ficha del empleado**: cuando
a alguien se le vence el contrato lo habitual es que el empleado esté
archivado, y entonces en la lista de Empleados no aparece. En el historial de
contratos sí.

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

**Ejemplo.** El contrato de Rafael Quintero venció el 30/09/2026 y se renueva
el 3 de octubre, cuando lo firman. El asistente propone **Fecha de inicio
01/10/2026** (no el 3: no hay hueco sin contrato), **Cargo** Vendedor, **Sueldo**
18.000,00; se cambia el sueldo a 20.000,00 y se pone **fin 30/09/2027**. El
recibo de la quincena del 1 al 15 de octubre lo toma ya con 20.000, y el
historial enseña las dos versiones seguidas.

> **El recorte del vencimiento no es cosmético**: Odoo no deja que dos
> contratos del mismo empleado se solapen en fechas. Sin recortar, la
> renovación fallaría con un error del núcleo difícil de interpretar.

## El análisis de nómina

**Nómina → Informes → Análisis de nómina.**

![El análisis, por empleado y mes](../img/nomina-analisis-pivote.png)

![Recorrido: el pivote y lo que se puede cruzar con él](../media/recorrido-nomina-analisis.webm)

**Para qué**: responder preguntas sobre el **recibo entero** —cuánto se pagó,
a quién, cuándo, en qué departamento— cruzando lo que haga falta, y bajarlo a
hoja de cálculo.

**Por qué un pivote y no un informe fijo**: cada mes la pregunta cambia:
contabilidad quiere el neto por departamento; gerencia, el bruto por mes;
el sindicato, los días trabajados. Un pivote contesta todas con las mismas
medidas cambiando filas y columnas.

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

**Ejemplo.** La quincena 2 de septiembre de 2026, por departamento:

| Departamento | Sueldo bruto | Deducciones | Sueldo neto |
|---|---:|---:|---:|
| Administración | 39.150,00 | 2.907,00 | 36.243,00 |
| Operaciones | 40.475,00 | 2.809,25 | 37.665,75 |
| Mantenimiento | 12.075,00 | 816,00 | 11.259,00 |
| Ventas | 10.575,00 | 688,50 | 9.886,50 |
| **Total** | **102.275,00** | **7.220,75** | **95.054,25** |

Administración tiene dos personas y Operaciones tres, y cuestan casi lo mismo:
el pivote lo enseña sin abrir un recibo. Añadiendo la medida **Nº de recibos**
se ve por qué.

> Para el desglose **por concepto** —cestaticket, prestaciones, parafiscales,
> ISLR— está el [tablero de nómina](04-el-tablero.md): el análisis mira el
> recibo entero, el tablero mira cada línea. El análisis no puede decir «cuánto
> de estos 95.054,25 fue cestaticket»; el tablero sí.
