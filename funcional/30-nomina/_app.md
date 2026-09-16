# Nómina

Cómo se configura la nómina venezolana y cómo se procesa una quincena, de punta
a punta: el sueldo y el cestaticket, los aportes y retenciones parafiscales
(IVSS, RPE, FAOV, INCES), la retención de ISLR, la garantía de prestaciones, y
el tablero que responde cuánto cuesta todo eso.

## Por qué esta nómina no es la de Odoo «a secas»

La nómina de Odoo Enterprise no está disponible en esta instalación, y la
venezolana tiene reglas que ninguna nómina genérica trae:

| Realidad de la nómina venezolana | Qué le añade la localización |
|---|---|
| Se paga en bolívares pero se piensa en divisa: el sueldo «son 450 $» | Una **tasa de nómina** por lote, congelada en cada recibo, y cada línea con su importe en divisa |
| Cada concepto tiene dos lados: lo que se le descuenta al trabajador y lo que aporta el patrono (IVSS 4 % / 9–11 %, FAOV 1 % / 2 %, RPE 0,5 % / 2 %) | Reglas por **categoría** —deducciones y aportes patronales separados— para que el neto y el costo salgan bien |
| El cestaticket y los bonos en divisa pesan tanto como el sueldo | Campos propios en el contrato y conceptos propios en el tablero |
| La garantía de prestaciones se acumula cada mes aunque no se pague | Un aporte patronal que no toca el neto pero sí el costo y el pasivo |
| Las mismas seis preguntas en cada cierre: costo total, cestaticket, prestaciones, parafiscales, bonos, ISLR | El **tablero de nómina**, con un indicador por pregunta |

La nómina se apoya en el motor `payroll` de la OCA, no en el de Odoo
Enterprise: los nombres de pantalla pueden diferir de la documentación oficial
de Odoo.

Estas guías explican cada pieza en tres tiempos —**para qué sirve**, **por qué
está así** y **cómo se hace**— con los números de la base de demostración: ocho
empleados, dos quincenas de septiembre de 2026, tasa de nómina **36,50 Bs por
dólar**.

## Las guías

| Guía | Qué contesta |
|---|---|
| [Cómo se configura](01-configuracion.md) | Tipo de estructura, estructura, reglas salariales y el contrato del empleado, con el recibo completo de un empleado como ejemplo |
| [Procesar una quincena](02-la-quincena.md) | El lote, la generación de recibos, cómo leer el cálculo línea a línea, confirmar |
| [Historial de contratos y análisis](03-historial-y-analisis.md) | Versiones del contrato, aviso de vencimiento, renovación, el pivote de análisis |
| [El tablero de nómina](04-el-tablero.md) | Costo total, cestaticket, prestaciones, parafiscales, bonos, ISLR: qué suma cada indicador y por qué |

> **El orden importa.** Sin el tipo de estructura, la estructura y las reglas
> configuradas, un lote se genera vacío o los recibos salen sin líneas. Empiece
> por la guía de configuración aunque solo venga a procesar un pago.
