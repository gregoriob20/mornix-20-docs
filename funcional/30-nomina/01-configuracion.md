# Cómo se configura

Lo que hay que dejar puesto **una vez** para que la nómina se pueda calcular:
el tipo de estructura, la estructura con sus reglas, y los datos de nómina de
cada empleado.

> La nómina se apoya en el motor `payroll` de la OCA, no en el de Odoo
> Enterprise. Los nombres de pantalla pueden diferir de lo que se ve en la
> documentación oficial de Odoo.

## El orden importa

Se configura de arriba abajo, porque cada pieza necesita la anterior:

1. **Tipo de estructura** — la periodicidad y la moneda.
2. **Estructura salarial** — qué reglas se aplican y a qué departamentos.
3. **Reglas salariales** — cómo se calcula cada concepto.
4. **El contrato del empleado** — el sueldo y a qué estructura pertenece.

Lo que sigue es cada paso tal como se hace en pantalla, con los nombres de
botones y campos tal como se ven, y con **un recibo real de la demo** como hilo:
Argenis Rondón, Mantenimiento, sueldo mensual **16.500 Bs**, cestaticket
**4.000 Bs**, nómina quincenal a **36,50 Bs por dólar**.

## El recibo que vamos a construir

Antes de configurar conviene ver a dónde se llega. Esto es lo que calcula el
sistema para la segunda quincena de septiembre de 2026:

| Concepto | Categoría | Cálculo | Bs | USD |
|---|---|---|---:|---:|
| Sueldo básico quincenal | Sueldo básico | 16.500 ÷ 2 | 8.250,00 | 226,03 |
| Cestaticket | Bonos y asignaciones | 4.000 ÷ 2 | 2.000,00 | 54,79 |
| Bono en divisa | Bonos y asignaciones | 50 $ fijos × 36,50 | 1.825,00 | 50,00 |
| **Total asignaciones** | Total asignaciones | suma | **12.075,00** | **330,82** |
| Seguro Social Obligatorio | Deducciones | −4 % del básico | −330,00 | −9,04 |
| Régimen Prestacional de Empleo | Deducciones | −0,5 % del básico | −41,25 | −1,13 |
| Ley de Vivienda y Hábitat (FAOV) | Deducciones | −1 % del básico | −82,50 | −2,26 |
| Retención de ISLR | Deducciones | −3 % del total asignaciones | −362,25 | −9,92 |
| **Neto a pagar** | Neto a pagar | asignaciones + deducciones | **11.259,00** | **308,47** |
| Garantía de prestaciones sociales | Aportes patronales | 16.500 ÷ 30 × 5 días | 2.750,00 | 75,34 |
| Aporte patronal IVSS | Aportes patronales | 9 % del básico | 742,50 | 20,34 |
| Aporte patronal FAOV | Aportes patronales | 2 % del básico | 165,00 | 4,52 |
| Aporte patronal INCES | Aportes patronales | 2 % del básico | 165,00 | 4,52 |
| **Costo para la empresa** | | asignaciones + aportes | **15.897,50** | **435,55** |

Tres cosas que se leen de ahí y que explican la configuración:

- Las **deducciones** son negativas y las **asignaciones** positivas, y el neto
  es la **suma** de todo lo anterior. Por eso una deducción se configura con
  porcentaje **negativo**.
- Los **aportes patronales** están **después del neto**: no se le descuentan a
  nadie, los paga la empresa. Si entraran antes, el bruto y el neto saldrían
  inflados.
- Todo se calcula sobre **categorías** (el 4 % es «del sueldo básico», no «de
  la regla SUELDO»): añadir un concepto nuevo a una categoría lo mete en la
  base de todo lo que dependa de ella.

## 1. El tipo de estructura

**Nómina → Configuración → Tipos de estructura.**

![Los tipos de estructura, con su tipo de sueldo y su estructura por defecto](../img/nomina-tipos-estructura.png)

**Para qué**: decir **cada cuánto** y **en qué moneda** se paga a un grupo de
empleados. Agrupa estructuras que comparten la forma de pagar.

**Por qué es lo primero**: **los empleados entran a un lote por su tipo de
estructura**, no por su departamento. Si un empleado no aparece al generar
recibos, lo primero que hay que mirar es si su contrato tiene tipo de
estructura y si es el del lote.

### Paso a paso: crear un tipo de estructura

1. **Nómina → Configuración → Tipos de estructura → Nuevo.**
2. **Tipo de estructura salarial**: el nombre. Por ejemplo, «Quincenal».
3. **Tipo de sueldo**: *Mensual* para sueldo fijo, *Por hora* para jornaleros.
   Manda en el cálculo.
4. **Moneda**: la del cálculo. En Venezuela, **VES**.
5. **País**: Venezuela.
6. **Horas laborables**: el calendario de trabajo que se aplica por defecto.
7. **Estructura por defecto**: déjelo vacío por ahora — la estructura todavía
   no existe. Se vuelve aquí en el paso 2.
8. **Guardar** (el icono de la nube, arriba a la izquierda).

![La ficha del tipo de estructura](../img/nomina-tipo-estructura-form.png)

**Ejemplo.** Una empresa paga a los empleados fijos por quincena y a los
obreros de planta por semana. Son **dos tipos de estructura** —«Quincenal» y
«Semanal»— porque el lote de la quincena no debe arrastrar a los obreros. Un
empleado que pase de la planta a la oficina cambia de tipo de estructura en su
contrato, y con eso cambia de lote.

## 2. La estructura salarial

**Nómina → Configuración → Estructuras salariales.**

![La estructura, con su tipo, su periodicidad, sus departamentos y sus reglas](../img/nomina-estructura-form.png)

**Para qué**: la estructura dice **qué reglas se aplican** y en qué orden a los
recibos que se generen con ella. Las reglas se añaden en la pestaña de abajo;
el orden de cálculo lo da la secuencia de cada regla, no el orden de la lista.
Los **departamentos** acotan a quién alcanza.

**Por qué separar estructura y reglas**: la misma regla —«Seguro Social 4 %»—
sirve en la nómina quincenal, en la de vacaciones y en la de utilidades. Se
define una vez y cada estructura decide si la incluye. Cambiar el porcentaje
cambia en todas a la vez.

### Paso a paso: crear la estructura

1. **Nómina → Configuración → Estructuras salariales → Nuevo.**
2. **Nombre**: «Nómina quincenal».
3. **Referencia**: un código corto y estable, por ejemplo `QUINC`. Los informes
   y las fórmulas lo usan.
4. **Tipo de estructura**: el creado en el paso 1. Al elegirlo, **Moneda** y
   **Tipo de salario por defecto** se rellenan solos.
5. **Pago programado predeterminado**: *Quincenal*, *Mensual*… Es lo que
   propone el lote al crearse.
6. **Departamentos**: opcional. Si se deja vacío, la estructura vale para todos.
7. Pestaña **Reglas salariales → Agregar una línea**: elija las reglas. Si
   todavía no existen, guarde la estructura vacía y vuelva después del paso 3.
8. **Guardar.**
9. Vuelva a **Tipos de estructura**, abra el suyo y ponga esta estructura en
   **Estructura por defecto**. Es la que el sistema propone al generar recibos.

**Ejemplo.** La estructura `QUINC` de la demo tiene las **13 reglas** del recibo
de arriba. Una estructura «Vacaciones» reutilizaría SUELDO, SSO, RPE y FAOV,
añadiría «Bono vacacional» y dejaría fuera el cestaticket, que no se paga en
vacaciones colectivas. Ninguna regla se duplica.

## 3. Las reglas salariales

**Nómina → Configuración → Reglas salariales.**

![Las reglas, con su código, su categoría, su concepto y su secuencia](../img/nomina-reglas-listado.png)

**Para qué**: cada regla es un concepto del recibo: el sueldo, un bono, una
deducción, un aporte. Lo que la define:

| Campo | Qué decide |
|---|---|
| **Categoría** | Si suma, si resta y en qué total entra |
| **Secuencia** | El orden de cálculo. Una regla solo puede usar lo ya calculado |
| **Condición** | Si la regla aplica a este recibo |
| **Cálculo** | Importe fijo, porcentaje sobre otra categoría, o código Python |
| **Concepto del tablero** | En qué indicador del tablero suma (ver [El tablero](04-el-tablero.md)) |

**Por qué la secuencia manda**: el sistema calcula las reglas **en orden de
secuencia** y cada una solo ve lo que ya está calculado. El neto (300) puede
sumar las deducciones (200–250) porque van antes; si el ISLR tuviera secuencia
310, el neto saldría sin descontarlo y nadie avisaría.

### Paso a paso: crear una regla

Con el ejemplo del aporte del trabajador al Seguro Social (4 % del sueldo
básico, que se descuenta):

1. **Nómina → Configuración → Reglas salariales → Nuevo.**
2. **Nombre**: «Seguro Social Obligatorio (4%)».
3. **Categoría**: *Deducciones*. Es lo que hace que reste.
4. **Código**: el sistema propone uno (`DED200`). Escriba `SSO` si prefiere un
   código que se lea: **lo que se escribe a mano manda** y no se vuelve a
   recalcular.
5. **Secuencia**: `200`. Tiene que ser **mayor** que la del sueldo básico,
   porque se calcula sobre él. Regla práctica: asignaciones del 10 al 90,
   total bruto en 100, deducciones del 200 al 290, neto en 300, aportes
   patronales del 400 en adelante.
6. **Concepto del tablero**: se propone *IVSS* por el código. Si no, elíjalo.
7. Pestaña **Configuración de reglas**:
   - Bloque **Condiciones**: *Siempre verdadero*, o *Rango*/*Expresión Python*
     si la regla aplica solo a algunos.
   - Bloque **Cálculo**: *Porcentaje (%)*.
   - **Porcentaje basado en**: `categories.BAS7`, la categoría del sueldo
     básico.
   - **Porcentaje**: `-4` — **negativo**, porque es una deducción.
     **Cantidad**: `1.0`.
8. Marque **Aparece en la nómina** para que salga impresa en el recibo.
9. **Guardar**, y añádala a la estructura (paso 2.7).

![La regla del Seguro Social: categoría, código, secuencia y su cálculo](../img/nomina-regla-form.png)

**Ejemplo, con el recibo de Argenis.** El sueldo básico quincenal es 8.250,00
(categoría `BAS7`). La regla SSO calcula `8.250,00 × −4 % = −330,00`. En el
recibo se lee «Importe 8.250,00 · Tasa −4,00 % · Total −330,00»: la base, el
porcentaje y el resultado, en ese orden.

### Las reglas de la demo, una a una

Las trece reglas del recibo de arriba, con lo que va en cada campo. Son un
punto de partida, no la ley: los porcentajes de la empresa real se ajustan a su
riesgo IVSS y a lo que diga el contrato colectivo.

| Código | Concepto | Categoría | Sec. | Cálculo | Valor |
|---|---|---|---:|---|---|
| `SUELDO` | Sueldo básico quincenal | Sueldo básico | 10 | Código Python | `result = contract.wage / 2` |
| `CESTA` | Cestaticket | Bonos y asignaciones | 20 | Código Python | `result = contract.cestaticket_salary / 2` |
| `BONODIV` | Bono en divisa | Bonos y asignaciones | 30 | Importe fijo | `1825` (50 $ a 36,50) |
| `BRUTO` | Total asignaciones | Total asignaciones | 100 | Código Python | `result = categories.BASIC` |
| `SSO` | Seguro Social Obligatorio | Deducciones | 200 | Porcentaje | `-4` sobre `categories.BAS7` |
| `RPE` | Régimen Prestacional de Empleo | Deducciones | 210 | Porcentaje | `-0.5` sobre `categories.BAS7` |
| `FAOV` | Ley de Vivienda y Hábitat | Deducciones | 220 | Porcentaje | `-1` sobre `categories.BAS7` |
| `ISLR` | Retención de ISLR | Deducciones | 250 | Porcentaje | `-3` sobre `categories.BASIC` |
| `NETO` | Neto a pagar | Neto a pagar | 300 | Código Python | `result = categories.BASIC + categories.DED` |
| `PREST` | Garantía de prestaciones sociales | Aportes patronales | 400 | Código Python | `result = (contract.wage / 30) * 5` |
| `IVSSPAT` | Aporte patronal IVSS | Aportes patronales | 410 | Porcentaje | `9` sobre `categories.BAS7` |
| `FAOVPAT` | Aporte patronal FAOV | Aportes patronales | 420 | Porcentaje | `2` sobre `categories.BAS7` |
| `INCES` | Aporte patronal INCES | Aportes patronales | 430 | Porcentaje | `2` sobre `categories.BAS7` |

Lo que conviene saber de cada bloque, y por qué está así:

- **Sueldo y cestaticket** leen el contrato (`contract.wage`,
  `contract.cestaticket_salary`) y dividen entre 2 porque la estructura es
  quincenal. En una estructura mensual la fórmula es `contract.wage`.
- **El bono en divisa** es un fijo en bolívares en la demo. En una empresa real
  lo habitual es `result = 50 * payslip.rate_amount`: 50 $ a la tasa del lote,
  para que siga valiendo 50 $ cuando la tasa cambie.
- **El ISLR** de la demo es un 3 % plano para que se vea en el tablero. La
  retención real usa el **porcentaje de la planilla ARI** del empleado, que se
  carga en su ficha (apartado 4): la regla sería
  `result = -categories.BASIC * contract.percentage_income_tax_islr / 100`, con
  la condición `contract.income_tax_islr` (la casilla **Retención de ISLR** de
  la ficha) para que solo descuente a quien la tenga marcada.
- **Las prestaciones** acumulan 5 días de salario por mes (los 15 días por
  trimestre del artículo 142 de la LOTTT). La regla de la demo lo hace **por
  recibo**; en una nómina quincenal real se divide entre 2 (`* 2.5`) para no
  acumular el doble.
- **INCES** se calcula aquí sobre el sueldo básico para simplificar; la ley lo
  fija en 2 % sobre el total de sueldos pagados. Si la empresa quiere el cálculo
  legal exacto, la base es `categories.BASIC`.

> **El código de la regla se propone solo**, a partir del código de la categoría
> y la secuencia (`DED10`, `ALW20`). Si se escribe uno a mano, **manda el
> escrito**: no se vuelve a recalcular aunque cambie la secuencia. Esto importa
> porque las fórmulas y los informes buscan las reglas **por su código**.

### Las categorías son el armazón

**Por qué**: una regla de porcentaje se calcula sobre **una categoría**, no
sobre otra regla: el aporte al IVSS es «4 % de la categoría sueldo básico».
Así, añadir un concepto de sueldo nuevo —una prima por antigüedad que sea
salario— lo incluye en la base sin tocar la deducción.

| Código | Qué agrupa | En el recibo de Argenis |
|---|---|---:|
| `BAS7` | Sueldo básico | 8.250,00 |
| `ALW` | Bonos y asignaciones | 3.825,00 |
| `GROSS` | Total de asignaciones | 12.075,00 |
| `DED` | Deducciones | −816,00 |
| `NET` | Neto a pagar | 11.259,00 |
| `COMP` | Aportes patronales | 3.822,50 |

**Ejemplo de por qué importa la base.** El ISLR se calcula sobre `BASIC` (todo
lo asignado: 12.075) y el IVSS sobre `BAS7` (solo el básico: 8.250). Si el
cestaticket estuviera en la categoría *Sueldo básico* en vez de *Bonos*, el IVSS
subiría a 410,00 y estaría mal: el cestaticket no es salario a efectos del
Seguro Social.

> **Al añadir una categoría nueva hay que decirle al análisis de nómina que
> existe.** El informe suma por códigos de categoría, y una categoría que no
> esté en su lista **no se suma y no avisa**: el informe simplemente da de
> menos. Está en `l10n_ve_payroll_salary_rules_report`.

## 4. El contrato del empleado

**Empleados → ficha del empleado → pestaña Nómina.**

![La pestaña Nómina de la ficha: el contrato, el sueldo, el cestaticket y los aportes](../img/nomina-empleado-contrato.png)

**Para qué**: el contrato es de donde las reglas leen los datos de **esta**
persona: su sueldo, su cestaticket, si le aplica cada aporte y qué porcentaje
de ISLR tiene.

> **En Odoo 20 el contrato ya no es una pantalla aparte.** Es la pestaña
> «Nómina» de la ficha del empleado, y cada cambio de condiciones crea una
> **versión** nueva. El botón **Historial**, arriba, enseña todas.

**Por qué las casillas de aportes están en la persona y no en la regla**: la
regla es igual para todos, pero no a todos les aplica. Un jubilado reingresado
no cotiza IVSS; un empleado que gana por debajo del mínimo tributable no tiene
retención de ISLR. La regla pregunta a la ficha —«¿a esta persona le aplica?»—
y así no hace falta una estructura distinta por cada excepción.

### Paso a paso: dejar a un empleado listo para la nómina

1. **Empleados → abrir la ficha → pestaña Nómina.**
2. Bloque **Resumen del contrato**:
   - **Contrato**: la fecha de inicio del contrato. **Sin ella el empleado no
     entra en ningún lote**: para el sistema no tiene contrato en vigor.
   - **Salario** y, debajo, el tipo (*Sueldo fijo*).
   - **Cestaticket**: el monto mensual del bono de alimentación, si la
     compañía lo paga.
   - El **tipo de estructura** (categoría de pago): el del paso 1.
3. Bloque **Aportes parafiscales**: marque lo que le aplica a esta persona —
   **Seguro Social Obligatorio**, **Régimen Prestacional de Empleo**, **FAOV
   (Ley de Política Habitacional)**, **INCES**, **Retención de ISLR** (y su
   porcentaje). Las reglas de nómina leen estas casillas para decidir si
   descuentan o no.
4. **Guardar.**
5. Si cambian las condiciones más adelante —un aumento, otro cargo—, no edite
   encima: **Nuevo contrato** crea una versión nueva y la anterior queda en el
   **Historial**.

**Ejemplo.** Argenis Rondón: **Contrato** 01/01/2026, **Salario** 16.500,00,
**Cestaticket** 4.000,00, tipo de estructura «Quincenal», con IVSS, RPE y FAOV
marcados. El 1 de octubre le suben el sueldo a 18.000: **Nuevo contrato** con
fecha 01/10/2026 y salario 18.000. La quincena del 16 al 30 de septiembre
sigue calculándose con 16.500 —su recibo ya está confirmado—, y la del 1 al 15
de octubre toma 18.000: sueldo quincenal 9.000, SSO 360, prestaciones 3.000.

## Errores frecuentes

| Síntoma | Causa |
|---|---|
| El empleado no aparece al generar recibos | Su contrato no tiene fecha de inicio, ya venció, o su tipo de estructura no es el del lote |
| El recibo sale sin líneas | La estructura no tiene reglas, o ninguna cumple su condición |
| Una deducción no descuenta | Su categoría no es de deducciones, o el porcentaje va en positivo |
| Un concepto no entra en el total | La regla se calcula después del total: revisar la secuencia |
| El neto sale más alto que el bruto | Un aporte patronal quedó con secuencia menor que 300 y el neto lo sumó |
| El IVSS sale sobre el cestaticket | El cestaticket está en la categoría *Sueldo básico*; va en *Bonos y asignaciones* |
| La regla cambió de código sola | Se creó sin código y cambió su secuencia: escriba el código a mano para fijarlo |
| El informe de análisis suma de menos | Hay una categoría que el informe no conoce |
