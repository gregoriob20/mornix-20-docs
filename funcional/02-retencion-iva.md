# Retención de IVA

Cómo se genera el comprobante de retención de IVA, se confirma y se declara.

## Cuándo aplica

Cuando la compañía está marcada como agente de retención y el contacto también
lo está. El porcentaje sale del contacto: 75 % o 100 % del IVA facturado.

## El flujo, paso a paso

1. **Publicar la factura.** El comprobante no se puede generar sobre un
   borrador.
2. **Generar la retención** desde el botón de la factura. Se crea un
   comprobante con una línea por cada alícuota de la factura.
3. **Confirmar el comprobante.** Aquí es donde se asigna el **número de
   comprobante**, no antes. Mientras esté en borrador, el número está vacío.
4. El asiento contable se genera y **se concilia con la factura**: el saldo
   pendiente del proveedor baja por el importe retenido.

Desde la factura, el botón **Retención IVA** lleva al comprobante.

## El número de comprobante

Tiene un formato fijo que exige el SENIAT: año, mes y un correlativo de ocho
dígitos. Se toma de la secuencia del diario de retención y **no se puede
cambiar a mano** una vez asignado.

## Declarar: el archivo TXT

En **Contabilidad → Informes de Venezuela → Generar TXT IVA**:

1. Se elige el período.
2. Se generan las líneas, que salen de los comprobantes confirmados.
3. Se descarga el archivo para subirlo al portal del SENIAT.

> **Al generar el TXT, los comprobantes que entran quedan bloqueados.** No se
> pueden modificar ni devolver a borrador, porque ya se declararon. La factura
> muestra un aviso indicando en qué declaración quedó incluida.
>
> Si hay que corregir algo, primero se devuelve a borrador la declaración.

## Errores frecuentes

| Síntoma | Causa |
|---|---|
| «El diario no tiene cuenta de IVA por defecto» | Falta configurar la cuenta en el diario de retención |
| El comprobante no toma número | Está en borrador: el número se asigna al confirmar |
| El importe retenido no cuadra | Revisar el porcentaje del contacto y la clasificación de los impuestos |
