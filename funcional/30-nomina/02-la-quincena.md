# Procesar una quincena

El recorrido completo de un pago, desde el lote vacío hasta los recibos
confirmados.

## El flujo, paso a paso

1. **Crear el lote** con el período y la estructura.
2. **Generar los recibos**: el sistema busca a quién le toca y crea uno por
   persona.
3. **Revisar** el cálculo, recibo a recibo si hace falta.
4. **Confirmar.** A partir de aquí los importes quedan fijos.

![Recorrido: del listado de lotes al recibo de una persona](../media/recorrido-nomina-quincena.webm)

## 1. El lote

**Nómina → Procesamientos de nóminas → Nuevo.**

![Los lotes del mes, con su período y su estado](../img/nomina-lotes-listado.png)

Un lote es una quincena, un mes o cualquier período que se pague junto. Lo que
se rellena:

| Campo | Qué es |
|---|---|
| **Período** | Desde y hasta. Define qué contratos entran |
| **Estructura** | Con qué reglas se calcula |
| **Quincenas** | Primera o segunda. Obligatorio si la estructura es quincenal |
| **Tasa** | La tasa de cambio del período, si la estructura la pide |

### La tasa se congela en el recibo

Cuando la estructura lo exige, el lote lleva una **tasa de nómina**: la que se
usa para leer los importes en divisa.

> **Es una tasa distinta de la del BCV del día.** Se elige al crear el lote y
> **queda congelada en cada recibo**: abrir un recibo de hace tres meses enseña
> los importes a la tasa con la que se pagó, no a la de hoy. Es lo que permite
> que un recibo antiguo siga cuadrando.

## 2. Generar los recibos

Con el lote en borrador, el botón **Generar recibos** abre un asistente con los
empleados que entran. Se pueden quitar los que no correspondan.

![El lote con sus ocho recibos generados](../img/nomina-lote-form.png)

Entra un empleado si, durante el período del lote:

- su contrato estaba en vigor —empezado y no vencido—, **y**
- su tipo de estructura es el de la estructura del lote.

> En Odoo 20 **el contrato ya no tiene estado**. «En vigor» es cuestión de
> fechas: no hay nada que «abrir» ni que «cerrar».

Al terminar, el lote queda **Por verificar** y cada recibo trae sus líneas
calculadas.

Los recibos de todos los lotes se ven juntos en **Nómina → Nóminas del
empleado**:

![El listado de recibos, con su empleado, su período y su estado](../img/nomina-recibos-listado.png)

## 3. Revisar el cálculo

Entrando a un recibo, la pestaña **Cálculo de la nómina** enseña la cuenta
completa, línea a línea.

![El recibo por dentro: cada concepto con su base, su tasa y su total](../img/nomina-recibo-calculo.png)

![Recorrido: el recibo de una persona, de la cabecera al cálculo](../media/recorrido-nomina-recibo.webm)

Cómo se lee cada línea:

| Columna | Qué dice |
|---|---|
| **Importe** | La base del cálculo. En un porcentaje, **sobre cuánto** se aplica |
| **Tasa (%)** | El porcentaje. Negativo en las deducciones |
| **Total** | Lo que el concepto aporta al recibo |
| **Total Ref** | El mismo importe en la moneda de referencia, a la tasa del recibo |

> **Las deducciones se imprimen en positivo**, como en cualquier recibo de
> nómina, aunque por dentro se calculen en negativo —el neto se obtiene
> sumando—. La base, en cambio, no cambia de signo: el aporte del 4 % se lee
> «21.000,00 al −4 %», no «−21.000,00».
>
> Una deducción que aparezca **en negativo** no es un error de presentación: es
> una deducción que **suma** al neto, como la devolución de un descuento.

Si algo no cuadra, **Cambiar a borrador**, corregir, y **Calcular hoja** otra
vez.

## 4. Confirmar

**Confirmar** deja los recibos en firme. El lote se cierra.

Solo a partir de aquí los recibos entran en el **análisis de nómina**: el
informe cuenta los confirmados y, si se le pide, también los que están por
verificar.

## Errores frecuentes

| Síntoma | Causa |
|---|---|
| El asistente no propone a nadie | Ningún contrato en vigor en el período con ese tipo de estructura |
| «Quincenas» en rojo | La estructura es quincenal y falta decir cuál de las dos |
| La columna **Total Ref** sale en 0,00 | El lote no lleva tasa de nómina |
| Un empleado sale dos veces | Ya tenía un recibo en el lote: el asistente no duplica, revisar si hay dos lotes |
| No deja confirmar | Algún recibo del lote está en un estado que no lo permite |
