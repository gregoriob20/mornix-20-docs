# El tablero de nómina

Responde con números las preguntas que se repiten cada cierre: cuánto cuesta la
nómina, en qué se va, cuánto de cestaticket, cuánto se ha acumulado de
prestaciones, cuánto de parafiscales y cuánto de ISLR.

**Nómina → Informes → Tablero de nómina** (y **Costo total de nómina**, que es
el mismo dato entrando por el gráfico).

![El tablero, con el costo y el importe de cada concepto](../img/nomina-tablero-pivote.png)

## 1. Las dos columnas, y por qué no dan lo mismo

| Medida | Qué suma |
|---|---|
| **Costo para la empresa** | Asignaciones **más aportes patronales**: lo que de verdad sale de la empresa |
| **Importe** | Lo que vale cada línea, retenciones incluidas |

Una retención —ISLR, IVSS del trabajador— **no es costo**: ya está dentro del
sueldo del que se descuenta. Por eso en la fila del ISLR el costo es 0 y el
importe no: el importe dice cuánto se retuvo, que es otra pregunta.

> **Las retenciones se leen en positivo.** En el recibo valen negativo, porque
> así se calcula el neto; en el tablero se muestran como lo que son: «se
> retuvieron 6.521,25», no «-6.521,25».

## 2. Los indicadores

Cada uno es un filtro, ya puesto en el desplegable del buscador:

| Filtro | Qué contesta |
|---|---|
| **Cestaticket** | Cuánto se paga de bono de alimentación |
| **Prestaciones sociales** | Cuánto se ha acumulado de garantía de prestaciones |
| **Parafiscales (IVSS, RPE, FAOV, INCES)** | Cuánto se aporta y cuánto se retiene por cada uno |
| **ISLR retenido** | El acumulado de retención, por empleado o por periodo |
| **Bonos y beneficios adicionales** | Cuánto pesan sobre el costo |
| **Con importe en divisa** | Solo lo que tiene equivalente en divisa |

Y tres que separan quién paga qué: **Asignaciones**, **Aportes patronales** y
**Retenciones al trabajador**.

![El costo de la nómina por concepto](../img/nomina-tablero-costo.png)

## 3. Las consultas típicas, paso a paso

El tablero abre con dos filtros puestos —**el año en curso** y **Sin los
totales del recibo**— y agrupado por **Concepto**. Todo lo que sigue parte de
ahí.

### Cuánto costó la nómina este mes, por departamento

1. **Nómina → Informes → Costo total de nómina.**
2. En el buscador, pulse el filtro de fecha (**Hasta: 2026**) y cambie el
   periodo al **mes** que quiere.
3. Pulse la agrupación **Concepto > Mes** para quitarla, y en el desplegable
   elija **Agrupar por → Departamento**.
4. La medida ya es **Costo para la empresa**. Cada barra es un departamento.
5. Para verlo en tabla, pulse el icono de pivote (arriba a la derecha).

### Cuánto ISLR se le ha retenido a cada empleado en el año

1. **Nómina → Informes → Tablero de nómina.**
2. Desplegable del buscador → **ISLR retenido**.
3. Quite la agrupación **Concepto** y agrupe por **Empleado**.
4. Lea la columna **Importe**: es lo retenido. La columna **Costo para la
   empresa** sale a 0, y así debe ser.
5. Para el acumulado mes a mes: en las columnas del pivote pulse **Total → Hasta
   → Mes**.

### Aportes del patrono contra retenciones del trabajador en parafiscales

1. **Tablero de nómina** → filtro **Parafiscales (IVSS, RPE, FAOV, INCES)**.
2. Filas: **Concepto**. Columnas: pulse **Total → Naturaleza**.
3. Salen dos columnas por concepto: **Aporte patronal** y **Deducción al
   trabajador**. La primera cuesta a la empresa; la segunda, no.

### Cuánto se ha acumulado de prestaciones

1. **Tablero de nómina** → filtro **Prestaciones sociales**.
2. Agrupe por **Empleado** para el acumulado por persona, o por **Mes** para
   ver cómo crece.
3. **Costo para la empresa** es el acumulado del periodo elegido. Amplíe el
   filtro de fecha para tener el histórico.

### Distribución de gastos por regla salarial

1. **Tablero de nómina**, sin filtros de concepto.
2. Quite **Concepto** y agrupe por **Regla salarial**.
3. Cada fila es una regla, con su costo y su importe. Es lo que contabilidad
   suele pedir como «distribución por regla».

Todo se exporta a hoja de cálculo con el botón de descarga del pivote.

## 4. De dónde salen los conceptos

Cada **regla salarial** lleva su concepto. **Nómina → Configuración → Reglas
salariales**: la columna **Concepto del tablero** lo muestra, y se puede
cambiar.

![Las reglas, con su concepto](../img/nomina-reglas-listado.png)

El sistema lo propone a partir del código de la regla —`CESTA` es cestaticket,
`ISLR` es ISLR, `FAOVPAT` es FAOV— y **lo que se ponga a mano manda**.

### Paso a paso: corregir la clasificación de una regla

1. **Nómina → Configuración → Reglas salariales.**
2. Desplegable del buscador → **Agrupar por → Concepto del tablero**. Las
   reglas en **Otro** son las que hay que mirar.
3. Abra la regla y cambie **Concepto del tablero** al que corresponda.
4. Al guardar, la casilla **Concepto fijado a mano** queda marcada: la
   propuesta automática no vuelve a tocarla, ni siquiera si luego cambia el
   código de la regla.
5. Para volver a la propuesta automática: desmarque **Concepto fijado a mano**
   y guarde.
6. El tablero refleja el cambio en cuanto se recarga: no hay que recalcular
   ningún recibo.

> **Si un total sale corto, mire aquí primero.** El filtro **Reglas sin
> clasificar** del tablero enseña las líneas de esas reglas: suman en «Otro»,
> no desaparecen, pero mientras estén ahí el desglose miente por omisión.

## 5. Qué entra y qué no

- Entran los recibos **por verificar, hechos y pagados**. La pre-nómina cuenta
  a propósito: el tablero sirve sobre todo **antes** de cerrar la quincena, que
  es cuando todavía se puede corregir.
- Los recibos en **borrador** no entran.
- **El bruto y el neto del recibo no suman**: son la suma de lo demás. Están
  clasificados como «Total del recibo» y el tablero los oculta por defecto; si
  quiere cuadrar contra un recibo concreto, quite el filtro **Sin los totales
  del recibo** y aparecen.
