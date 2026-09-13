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

Cada uno es un filtro, ya puesto:

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

## 3. Cómo se cruza

El tablero es un pivote: las agrupaciones ya preparadas son **concepto**,
**regla salarial**, **naturaleza**, **empleado**, **departamento**,
**estructura**, **lote** y **mes**. Cruzando concepto contra mes sale la
evolución; concepto contra departamento, en qué área se va el dinero.

> La «distribución de gastos por regla salarial» que suele pedir contabilidad
> es este mismo tablero agrupando por **Regla salarial** en vez de por
> concepto.

Todo se exporta a hoja de cálculo con el botón de descarga del pivote.

## 4. De dónde salen los conceptos

Cada **regla salarial** lleva su concepto. **Nómina → Configuración → Reglas
salariales**: la columna **Concepto del tablero** lo muestra, y se puede
cambiar.

![Las reglas, con su concepto](../img/nomina-reglas-listado.png)

El sistema lo propone a partir del código de la regla —`CESTA` es cestaticket,
`ISLR` es ISLR, `FAOVPAT` es FAOV— y **lo que se ponga a mano manda**: al
corregirlo queda marcado como fijado a mano y la propuesta automática no vuelve
a tocarlo, ni siquiera si luego cambia el código de la regla.

> **Si un total sale corto, mire aquí primero.** El filtro **Reglas sin
> clasificar** enseña las que el sistema no supo encuadrar: suman en «Otro», no
> desaparecen, pero mientras estén ahí el desglose miente por omisión.

## 5. Qué entra y qué no

- Entran los recibos **por verificar, hechos y pagados**. La pre-nómina cuenta
  a propósito: el tablero sirve sobre todo **antes** de cerrar la quincena, que
  es cuando todavía se puede corregir.
- Los recibos en **borrador** no entran.
- **El bruto y el neto del recibo no suman**: son la suma de lo demás. Están
  clasificados como «Total del recibo» y el tablero los oculta por defecto; si
  quiere cuadrar contra un recibo concreto, quite el filtro **Sin los totales
  del recibo** y aparecen.
