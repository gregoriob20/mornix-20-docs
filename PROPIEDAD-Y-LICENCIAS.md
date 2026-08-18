# Propiedad y licencias del código

Auditoría del 18/08/2026. Inventario técnico para revisión legal: los
hallazgos son verificables en el código; las decisiones con consecuencias
(contratos, relicenciar, distribuir) las toma un abogado con los contratos a
la vista.

## Estado del repositorio de migración

- Los cuatro módulos migrados declaran **LGPL-3**, coherente con Odoo
  Community; ninguno depende de Enterprise.
- El texto de la licencia está en `LICENSE` (raíz del repositorio).
- Las cabeceras de autor quedaron unificadas a **«Mornix C.A — antes
  Nimetrix C.A»** (mismo titular, razón social sucedida).
- Los commits llevan autor identificado y la co-autoría de Claude declarada.
  Los 65 commits históricos firmados como «root» son anteriores a fijar
  `git config user.name` (18/08/2026); no se reescriben porque ya están
  publicados.

## Hallazgos corregidos

### El asistente de empleados venía de un módulo GPL-2 (corregido en 1.34.1)

`wizards/nimetrix_employee_income_wh_islr.py` descendía, con cabecera
intacta, de `l10n_ve_full/wizard/employee_income_wh_islr.py` — autor
**Tecvemar, C.A. / Juan V. Márquez (2012)**, módulo declarado **GPL-2**.
GPL-2 sin cláusula «or later» es incompatible con relicenciar a LGPL-3, y la
autoría era de un tercero.

Se sustituyó por una **reimplementación independiente** (código y vista):
misma función — importar al XML del SENIAT las retenciones ISLR de empleados
desde el CSV de la nómina —, expresión nueva (validación por lotes, errores
acumulados con número de fila, todo-o-nada), cubierta por 4 pruebas en
`tests/test_importar_empleados_islr.py`. De paso: la versión heredada estaba
**rota en v20** (hacía `b64decode` de un campo Binary que ahora entrega los
bytes crudos), así que la pieza retirada ni siquiera funcionaba.

## Hallazgos que requieren decisión (no son de código)

1. **Titularidad Mornix ↔ cliente.** El repositorio se transfirió el
   18/08/2026 de la organización `mornix-tech` a la cuenta personal
   `gregoriob20`. Un repositorio de trabajo para cliente bajo una cuenta
   personal hace más urgente, no menos, dejar la titularidad por escrito: si
   la migración es obra por encargo o licencia de uso lo define el contrato
   de servicios, no dónde viva el git. Recomendación: devolverlo a una
   organización (de Mornix o del cliente, según diga el contrato) con el
   equipo como colaboradores.
2. **Zona gris con Oasis Consultora.** El `account_dual_currency` original
   (v16) es *Other proprietary* de Oasis; `nimetrix_dual_currency` (v18,
   LGPL-3, base de nuestro `mornix_dual_currency`) comparte con él conceptos
   y algunos nombres (`amount_ref`) aunque los cuerpos difieren
   sustancialmente. Hace falta el contrato Nimetrix–Oasis o constancia de
   desarrollo independiente.
3. **El mosaico del árbol original** condiciona la migración de los módulos
   restantes (censo completo abajo):
   - **OPL-1** (38): apps compradas (BrowseInfo, Emipro, Terabits…). Se
     pueden adaptar para uso del cliente, pero **no** copiarse a este
     repositorio ni relicenciarse; cada una exige licencia de compra vigente
     para la versión destino.
   - **AGPL-3** (15): derivar obliga a que el resultado sea AGPL-3.
   - **OEEL-1** (3): requieren suscripción Enterprise vigente.
   - **GPL-2** (3): mismas condiciones que motivaron la reescritura del
     asistente de empleados; nada de ellos puede entrar en módulos LGPL-3.
   - **Other proprietary** (5): Oasis Consultora; exige su contrato.

## Reglas operativas para el equipo

- Al migrar un módulo, **verificar primero su licencia y autor** en el censo
  de abajo. Si no es LGPL-3 de Nimetrix/Mornix, la migración no puede
  publicarse en este repositorio sin decisión previa.
- Código de terceros que se conserve mantiene su cabecera de autor original.
- No copiar código de módulos OPL-1/OEEL-1/propietarios a módulos propios,
  ni «adaptándolo»: la reescritura independiente es el único camino limpio.
- El censo se regenera, no se edita:

```bash
python3 scripts/censo_licencias.py addons /opt/odoo-client/v16 /opt/odoo-client/v18 \
    > docs/censo-licencias.md
```

El censo detallado vive en [`censo-licencias.md`](censo-licencias.md)
(generado; una fila por módulo con licencia, autor y nota de riesgo).
