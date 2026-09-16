# Procesar una quincena

El recorrido completo de un pago, desde el lote vacío hasta los recibos
confirmados, con la **segunda quincena de septiembre de 2026** de la demo como
ejemplo: ocho empleados, tasa de nómina 36,50.

## El flujo

1. **Crear el lote** con el período y la estructura.
2. **Generar los recibos**: el sistema busca a quién le toca y crea uno por
   persona.
3. **Revisar** el cálculo, recibo a recibo si hace falta.
4. **Confirmar.** A partir de aquí los importes quedan fijos.

![Recorrido: del listado de lotes al recibo de una persona](../media/recorrido-nomina-quincena.webm)

**Por qué en cuatro pasos y no en uno**: entre generar y confirmar hay una
**pre-nómina** que se puede revisar y corregir sin consecuencias. El tablero y
el análisis ya la leen —a propósito—, así que un total que no cuadra se detecta
antes de pagar, no después.

## 1. El lote

**Nómina → Procesamientos de nóminas.**

![Los lotes del mes, con su período y su estado](../img/nomina-lotes-listado.png)

**Para qué**: el lote es el sobre en el que van todos los recibos de un mismo
pago: una quincena, un mes, las utilidades. Define **el período**, **la
estructura** y **la tasa** que comparten.

**Por qué un lote por pago y no un recibo suelto por persona**: porque el
período y la tasa tienen que ser los mismos para todos —a nadie se le paga la
quincena a una tasa distinta—, y porque los informes y la contabilidad trabajan
por lote: «la nómina del 16 al 30 costó 137.256,67».

### Paso a paso: crear el lote

1. **Nómina → Procesamientos de nóminas → Nuevo.**
2. **Nombre**: algo que se reconozca en la lista dentro de un año — «Quincena
   2 — septiembre 2026».
3. **Período**: desde y hasta. **Define qué contratos entran**: los que estaban
   en vigor en esas fechas.
4. **Estructura**: la de la nómina que se paga (*Nómina quincenal*).
5. **Quincenas**: *Primera* o *Segunda*. Aparece al elegir una estructura
   quincenal y es obligatorio: sin él el campo sale en rojo y no deja seguir.
6. **Tasa**: la tasa de cambio del período, si la estructura la pide. Ver
   abajo por qué importa.
7. **Guardar.** El lote queda en **Borrador**, sin recibos.

**Ejemplo.** «Quincena 2 — septiembre 2026», del **16/09/2026** al
**30/09/2026**, estructura *Nómina quincenal*, **Quincenas: Segunda**, **Tasa
36,50**. Un empleado contratado el 20 de septiembre entra en este lote (su
contrato estaba en vigor dentro del período) aunque después haya que ajustarle
los días trabajados; uno que renunció el 10 no entra.

### La tasa se congela en el recibo

Cuando la estructura lo exige, el lote lleva una **tasa de nómina**: la que se
usa para leer los importes en divisa.

**Por qué es una tasa distinta de la del BCV del día**: la nómina se acuerda y
se paga a una tasa —la del día de pago, o la que fije la empresa—, y esa cifra
debe **seguir siendo la misma** cuando se abra el recibo dentro de tres meses.
Si el recibo leyera la tasa del día, un bono de 50 $ pagado en septiembre
aparecería como 45 $ en diciembre sin que nadie hubiera tocado nada. Por eso la
tasa se elige al crear el lote y **queda congelada en cada recibo**.

**Ejemplo.** El bono en divisa de Argenis: 1.825,00 Bs a la tasa del lote,
36,50 = **50,00 $**. Si en diciembre la tasa del BCV es 40,00, el recibo de
septiembre sigue diciendo 50,00 $; el de diciembre, con su propia tasa, dirá
2.000,00 Bs = 50,00 $. Los dos cuadran con lo acordado.

## 2. Generar los recibos

![El lote con sus ocho recibos generados](../img/nomina-lote-form.png)

**Para qué**: que el sistema busque a quién le toca cobrar en este lote y
calcule un recibo por persona, con las reglas de la estructura y los datos del
contrato de cada uno.

### Paso a paso: generar

1. Con el lote en **Borrador**, pulse **Generar recibos de nómina**.
2. Se abre una ventana con la lista de **Empleados** que entran. Revísela:
   - Quite a quien no corresponda (la x de su línea).
   - Si falta alguien, **no lo añada aquí**: cierre y revise su contrato (ver
     abajo). Añadirlo a mano genera un recibo que la estructura no sabe calcular.
3. **Generar.**
4. Resultado: el lote pasa a **Por verificar** y aparece un recibo por
   empleado, cada uno con sus líneas ya calculadas y en estado **En espera**.

Entra un empleado si, durante el período del lote:

- su contrato estaba en vigor —empezado y no vencido—, **y**
- su tipo de estructura es el de la estructura del lote.

**Por qué no se añade a mano al que falta**: si el asistente no lo propone es
porque su contrato no cumple una de las dos condiciones, y el recibo que se
genere a mano saldrá vacío o mal calculado (sin sueldo, sin tipo de
estructura). Arreglar el contrato arregla este lote y todos los que vengan.

> En Odoo 20 **el contrato ya no tiene estado**. «En vigor» es cuestión de
> fechas: no hay nada que «abrir» ni que «cerrar».

**Ejemplo.** El asistente propone **8 empleados**: tres de Operaciones, dos de
Administración, dos de Ventas y uno de Mantenimiento. Los recibos van del
**SLIP/009** al **SLIP/016**. Si faltara Argenis, lo primero es abrir su ficha →
pestaña Nómina: ¿tiene fecha de **Contrato**? ¿su tipo de estructura es
«Quincenal»?

Los recibos de todos los lotes se ven juntos en **Nómina → Nóminas del
empleado**:

![El listado de recibos, con su empleado, su período y su estado](../img/nomina-recibos-listado.png)

## 3. Revisar el cálculo

![El recibo por dentro: cada concepto con su base, su tasa y su total](../img/nomina-recibo-calculo.png)

![Recorrido: el recibo de una persona, de la cabecera al cálculo](../media/recorrido-nomina-recibo.webm)

**Para qué**: comprobar que cada concepto salió de la base correcta con el
porcentaje correcto, **antes** de que el recibo quede en firme.

### Paso a paso: revisar un recibo

1. Desde el lote, pulse sobre el recibo (o **Nómina → Nóminas del empleado**).
2. Pestaña **Cálculo de la nómina**: la cuenta completa, línea a línea.
3. Pestaña **Días trabajados y entradas**: los días y horas que alimentaron el
   cálculo. Si un día no cuadra, el problema está aquí, no en la regla.
4. Si algo cambió después de generar —un dato del contrato, una regla—, pulse
   **Calcular hoja**: recalcula el recibo con los datos de ahora.
5. Si hay que corregir algo del propio recibo: **Cambiar a borrador**, corregir,
   **Calcular hoja** otra vez.
6. **Refrescar datos externos** vuelve a leer el contrato del empleado si se
   modificó después de crear el recibo.

Cómo se lee cada línea:

| Columna | Qué dice |
|---|---|
| **Importe** | La base del cálculo. En un porcentaje, **sobre cuánto** se aplica |
| **Tasa (%)** | El porcentaje. Negativo en las deducciones |
| **Total** | Lo que el concepto aporta al recibo |
| **Total Ref** | El mismo importe en la moneda de referencia, a la tasa del recibo |

### Ejemplo: el recibo SLIP/016, línea a línea

Argenis Rondón, sueldo 16.500, cestaticket 4.000, tasa 36,50:

| Concepto | Importe (base) | Tasa (%) | Total | Total Ref (USD) |
|---|---:|---:|---:|---:|
| Sueldo básico quincenal | 8.250,00 | 100 | 8.250,00 | 226,03 |
| Cestaticket | 2.000,00 | 100 | 2.000,00 | 54,79 |
| Bono en divisa | 1.825,00 | 100 | 1.825,00 | 50,00 |
| **Total asignaciones** | 12.075,00 | 100 | **12.075,00** | 330,82 |
| Seguro Social Obligatorio | 8.250,00 | −4,00 | 330,00 | 9,04 |
| Régimen Prestacional de Empleo | 8.250,00 | −0,50 | 41,25 | 1,13 |
| Ley de Vivienda y Hábitat | 8.250,00 | −1,00 | 82,50 | 2,26 |
| Retención de ISLR | 12.075,00 | −3,00 | 362,25 | 9,92 |
| **Neto a pagar** | 11.259,00 | 100 | **11.259,00** | 308,47 |
| Garantía de prestaciones sociales | 2.750,00 | 100 | 2.750,00 | 75,34 |
| Aporte patronal IVSS | 8.250,00 | 9,00 | 742,50 | 20,34 |
| Aporte patronal FAOV | 8.250,00 | 2,00 | 165,00 | 4,52 |
| Aporte patronal INCES | 8.250,00 | 2,00 | 165,00 | 4,52 |

Cómo se revisa con esto delante:

- **¿El SSO está bien?** Base 8.250 (solo el básico, no el cestaticket) × 4 % =
  330,00. ✔
- **¿El ISLR está bien?** Base 12.075 (todo lo asignado) × 3 % = 362,25. ✔ Si la
  base fuera 8.250, la regla estaría apuntando a `BAS7` en vez de a `BASIC`.
- **¿El neto cuadra?** 12.075,00 − (330,00 + 41,25 + 82,50 + 362,25) =
  11.259,00. ✔
- **¿Los aportes tocaron el neto?** No: el neto va antes que ellos. Si el neto
  diera 15.081,50, un aporte tiene secuencia menor que 300.

> **Las deducciones se imprimen en positivo**, como en cualquier recibo de
> nómina, aunque por dentro se calculen en negativo —el neto se obtiene
> sumando—. La base, en cambio, no cambia de signo: el aporte del 4 % se lee
> «8.250,00 al −4 %», no «−8.250,00».
>
> Una deducción que aparezca **en negativo** no es un error de presentación: es
> una deducción que **suma** al neto, como la devolución de un descuento.

## 4. Confirmar

**Para qué**: dejar los recibos en firme: el estado pasa de **En espera** a
**Hecho** y los importes ya no se recalculan.

**Por qué no se paga sin confirmar**: un recibo en espera se recalcula cada vez
que alguien pulsa **Calcular hoja**, y un cambio en una regla lo alteraría
después de haberlo pagado. Confirmar es decir «esto es lo que se pagó»; a
partir de ahí, corregir exige cancelar y dejar rastro.

### Paso a paso: confirmar uno a uno

1. Abra el recibo.
2. **Confirmar.** El estado pasa a **Hecho**.

### Paso a paso: confirmar todo el lote de una vez

1. **Nómina → Nóminas del empleado.** Filtre por el lote (busque su nombre).
2. Marque la casilla de la cabecera para seleccionar todos los recibos.
3. Menú **Acción → Cambiar estado**.
4. Elija **Hecho** y pulse **Ejecutar**.

Si hace falta deshacer un recibo ya confirmado: **Cancelar nómina** lo deja en
**Rechazada**, y desde ahí **Cambiar a borrador** permite corregir y volver a
calcular. Un recibo rechazado no entra en ningún informe.

**Ejemplo.** Al revisar la quincena se descubre que a Marielys Chirinos le
falta el bono en divisa. Su recibo está **En espera**: **Cambiar a borrador**,
se corrige la condición de la regla o el dato del contrato, **Calcular hoja**,
y **Confirmar**. Los otros siete recibos no se tocan. Si ya estuviera en
**Hecho**, el camino es **Cancelar nómina → Cambiar a borrador → Calcular hoja →
Confirmar**, y el chatter guarda quién lo hizo y cuándo.

> **El botón «Cerrar» del lote solo aparece con el lote en Borrador**, no en
> «Por verificar». El cierre real de la quincena es la confirmación de los
> recibos: con todos en **Hecho**, la quincena está cerrada aunque el lote siga
> diciendo «Por verificar». Está anotado como pendiente de revisar.

Solo a partir de la confirmación los recibos entran en el **análisis de
nómina** y en el **tablero** —que también cuentan, si se les pide, los que
están por verificar—.

### Lo que costó la quincena

Con los ocho recibos confirmados, esto es lo que la quincena 2 de septiembre
deja en los informes (los mismos números que enseña el tablero):

| | Bs | USD |
|---|---:|---:|
| Total asignaciones (bruto) | 102.275,00 | 2.802,05 |
| Deducciones a los trabajadores | 7.220,75 | 197,83 |
| **Neto pagado** | **95.054,25** | **2.604,23** |
| Aportes patronales (prestaciones + IVSS + FAOV + INCES) | 34.981,67 | 958,40 |
| **Costo para la empresa** (bruto + aportes) | **137.256,67** | **3.760,46** |

El costo para la empresa es un **44 % más** que lo que ven los empleados en su
cuenta: esa diferencia —aportes que no pasan por el neto— es lo que el tablero
está hecho para enseñar.

## Errores frecuentes

| Síntoma | Causa |
|---|---|
| El asistente no propone a nadie | Ningún contrato en vigor en el período con ese tipo de estructura |
| «Quincenas» en rojo | La estructura es quincenal y falta decir cuál de las dos |
| La columna **Total Ref** sale en 0,00 | El lote no lleva tasa de nómina |
| Un empleado sale dos veces | Ya tenía un recibo en el lote: el asistente no duplica, revisar si hay dos lotes |
| No deja confirmar | El recibo no está **En espera**: recalcule primero con **Calcular hoja** |
| El recibo tiene líneas pero el total no cuadra | Revise la secuencia de las reglas: una que se calcula después del total no entra en él |
| El bono en divisa cambió de valor en un recibo viejo | La regla lee la tasa del día en vez de `payslip.rate_amount`, la del lote |
