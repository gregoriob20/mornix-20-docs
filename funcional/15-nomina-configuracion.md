# Nómina: cómo se configura

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

## 1. El tipo de estructura

**Nómina → Configuración → Tipos de estructura.**

![Los tipos de estructura, con su tipo de sueldo y su estructura por defecto](img/nomina-tipos-estructura.png)

Agrupa estructuras que comparten la forma de pagar:

| Campo | Para qué sirve |
|---|---|
| **Tipo de sueldo** | Mensual u horario. Manda en el cálculo |
| **Moneda** | La moneda del cálculo. En Venezuela, bolívares |
| **Periodicidad de pago** | Quincenal, semanal, mensual… |
| **Estructura por defecto** | La que se propone al generar recibos de este tipo |

> **Los empleados entran a un lote por su tipo de estructura**, no por su
> departamento. Si un empleado no aparece al generar recibos, lo primero que hay
> que mirar es si su contrato tiene tipo de estructura y si es el del lote.

## 2. La estructura salarial

**Nómina → Configuración → Estructuras salariales.**

![La estructura, con su tipo, su periodicidad, sus departamentos y sus reglas](img/nomina-estructura-form.png)

La estructura dice **qué reglas se aplican** y en qué orden. Las reglas se
añaden en la pestaña de abajo; el orden de cálculo lo da la secuencia de cada
regla, no el orden de la lista.

Los **departamentos** acotan a quién alcanza la estructura.

## 3. Las reglas salariales

**Nómina → Configuración → Reglas salariales.**

![Las reglas, con su código, su categoría y su secuencia](img/nomina-reglas-listado.png)

Cada regla es un concepto del recibo: el sueldo, un bono, una deducción. Lo que
la define:

| Campo | Qué decide |
|---|---|
| **Categoría** | Si suma, si resta y en qué total entra |
| **Secuencia** | El orden de cálculo. Una regla solo puede usar lo ya calculado |
| **Condición** | Si la regla aplica a este recibo |
| **Cálculo** | Importe fijo, porcentaje sobre otra categoría, o código Python |

> **El código de la regla se propone solo**, a partir del código de la categoría
> y la secuencia (`DED10`, `ALW20`). Si se escribe uno a mano, **manda el
> escrito**: no se vuelve a recalcular aunque cambie la secuencia. Esto importa
> porque las fórmulas y los informes buscan las reglas **por su código**.

### Las categorías son el armazón

Una regla de porcentaje se calcula sobre **una categoría**, no sobre otra regla:
el aporte al IVSS es «4 % de la categoría sueldo básico». Así, añadir un
concepto de sueldo nuevo lo incluye en la base sin tocar la deducción.

Las categorías de esta implantación:

| Código | Qué agrupa |
|---|---|
| `BAS7` | Sueldo básico |
| `ALW` | Bonos y asignaciones |
| `GROSS` | Total de asignaciones |
| `DED` | Deducciones |
| `NET` | Neto a pagar |

> **Al añadir una categoría nueva hay que decirle al análisis de nómina que
> existe.** El informe suma por códigos de categoría, y una categoría que no
> esté en su lista **no se suma y no avisa**: el informe simplemente da de
> menos. Está en `l10n_ve_payroll_salary_rules_report`.

## 4. El contrato del empleado

**Empleados → ficha del empleado → pestaña Nómina.**

![La pestaña Nómina de la ficha: sueldo, tipo de sueldo, cestaticket y categoría de pago](img/nomina-empleado-contrato.png)

> **En Odoo 20 el contrato ya no es una pantalla aparte.** Es la pestaña
> «Nómina» de la ficha del empleado, y cada cambio de condiciones crea una
> **versión** nueva. El botón **Historial**, arriba, enseña todas.

Lo que la nómina necesita de aquí:

- **Fecha de inicio del contrato.** Sin ella el empleado no entra en ningún
  lote: para el sistema no tiene contrato en vigor.
- **Salario** y **tipo de sueldo**.
- **Cestaticket**, si la compañía lo paga.
- **Categoría de pago** — el tipo de estructura.

## Errores frecuentes

| Síntoma | Causa |
|---|---|
| El empleado no aparece al generar recibos | Su contrato no tiene fecha de inicio, ya venció, o su tipo de estructura no es el del lote |
| El recibo sale sin líneas | La estructura no tiene reglas, o ninguna cumple su condición |
| Una deducción no descuenta | Su categoría no es de deducciones, o la fórmula devuelve positivo |
| Un concepto no entra en el total | La regla se calcula después del total: revisar la secuencia |
| El informe de análisis suma de menos | Hay una categoría que el informe no conoce |
