# Nómina venezolana — 35 módulos sobre el motor de OCA

> Estado: **los 35 módulos libres instalan de una vez sobre una base vacía y las
> 102 pruebas pasan**, propias y del motor.
> Origen: `mornix-tech/nx-nomina`, v18 — 55 módulos.
> Destino: **repositorio propio**, `gregoriob20/mornix_nomina_20`, montado en
> `addons/nomina/`.
> Quedan **20 módulos bloqueados**, ninguno por trabajo pendiente nuestro: todos
> esperan a un módulo de Enterprise, de terceros, o a que Odoo 20 decida algo.

La nómina no vive en el repositorio de código como el resto de áreas: tiene el
suyo, igual que esta documentación. Son 55 módulos y 25.000 líneas, con un ciclo
propio y un bloqueo —`res.bank`— que no depende de nosotros.

## 1. Los dos bloqueos de partida, y cómo se resolvieron

Al medir el repositorio de origen aparecieron dos problemas de naturaleza
distinta. Ninguno era «migrar código».

### `hr_payroll` es de Enterprise

**42 de los 55 módulos dependían de él**, directa o indirectamente. Sin esa base
no existen `hr.payslip`, `hr.salary.rule` ni `hr.payroll.structure` —391
menciones en el repositorio— y esos módulos ni siquiera se instalan.

**Resuelto con `OCA/payroll`**, rama 19.0, adaptado a v20 en `addons/payroll/`.
Aporta justo esos modelos, depende solo de Community (`hr_holidays` y `mail`) y
es LGPL-3. Se le hicieron **nueve cambios**, todos acotados y comentados, para
poder rehacerlos cuando OCA saque su propia rama: están uno a uno en
`addons/payroll/MIGRACION-V20.md`.

> **Consecuencia práctica:** OCA **no** es un reemplazo pieza por pieza de
> Enterprise. Trae los modelos centrales, no todo lo que había alrededor. La
> mayor parte del trabajo de esta migración fue esa diferencia, no v20.

### Odoo 20 eliminó `hr.contract`

Lo sustituyó por **`hr.version`** —"Employee Record"—, fusionado dentro de `hr`:
el contrato ya no es un registro aparte, sino una versión del empleado. Son
**304 menciones en 114 archivos** del repositorio de origen.

Lo que hace la migración llevadera es que **`hr.employee` delega en
`hr.version`** (`_inherits`): un campo añadido a la versión se ve desde el
empleado sin tocar nada más. Lo que **no** delega son los métodos — y ahí está la
trampa, explicada abajo.

## 2. Tamaño

| | |
|---|---|
| Módulos del cliente | 54 en el árbol (55 en el origen) |
| Motor de nómina | 1 (`payroll`, OCA adaptado) |
| Python | 346 archivos · 15.467 líneas |
| XML | 155 archivos · 9.495 líneas |
| Motor OCA | 32 archivos · 3.655 líneas de Python |
| Pruebas | **102**, en 11 módulos |

Reparto de las pruebas:

| Módulo | Pruebas |
|---|---:|
| `l10n_ve_payroll_hr_hcm` | 35 |
| `payroll` (motor OCA) | 27 |
| `mnx_l10n_ve_payroll_hr_contract_history` | 20 |
| `l10n_ve_payroll_hr_payroll` | 10 |
| `l10n_ve_payroll_res_config_settings` | 3 |
| resto (6 módulos) | 7 |

## 3. Lo que v20 rompió, y dónde duele

El detalle de cada rotura, con el error literal que produce, está en
[BREAKING-CHANGES.md](../BREAKING-CHANGES.md). Aquí, lo que cambió el diseño.

### El contrato no tiene formulario propio: es el del empleado

v20 no trae **ninguna vista `form` de `hr.version`** — trae lista, gráfico,
pivote y búsqueda. El formulario del contrato es la ficha del empleado, y sus
campos viven en la pestaña «Nómina».

Trece vistas cambian de padre **y de modelo**: una vista heredada declara el
modelo de su padre, así que pasan a ser de `hr.employee` aunque los campos sean
de la versión.

> **La trampa:** `_inherits` delega **campos**, no **métodos**. Un botón
> `type="object"` puesto en el formulario del empleado busca el método en
> `hr.employee` y no lo encuentra en la versión. Tres módulos necesitaron un
> envoltorio de dos líneas que llama a `self.current_version_id.<metodo>()`.

Y un efecto en cadena: una vista que hereda a otra vista nuestra tiene que
declarar el modelo de **la raíz**. Al cambiar el padre a `hr.employee`, tres
vistas nietas se quedaron diciendo `hr.version` y fallaban con un escueto
`Field 'version_id' does not exist`.

### El contrato tampoco tiene estado

`state` con `draft` / `open` / `close` desapareció, y con él el cron que lo
movía. **«En curso» es hoy un rango de fechas.** Afecta a seis módulos, entre
dominios de vista, `search()` y `filtered()`.

> **Consecuencia práctica:** un campo calculado sobre esas fechas **no puede ser
> `store=True`**. Un contrato vence porque pasa el tiempo, no porque alguien
> escriba en él: con el valor almacenado, el aviso de vencimiento no aparece el
> día que toca.

### Campos que había que reponer

Ninguno de estos lo «quitó» v20 de Community: **nunca estuvieron en Community**.
Venían de Enterprise, y OCA tampoco los trae.

| Campo | Modelo | Repuesto en |
|---|---|---|
| `wage_type`, `hourly_wage` | `hr.version` | `l10n_ve_payroll_salary_fields` |
| `default_schedule_pay` | `hr.payroll.structure.type` | `l10n_ve_payroll_res_config_settings` |
| `wage_type`, `default_struct_id` | `hr.payroll.structure.type` | `l10n_ve_payroll_hr_payroll` |
| `type_id` | `hr.payroll.structure` | `l10n_ve_payroll_hr_payroll` |
| `structure_id`, `department_id` | `hr.payslip.employees` | `l10n_ve_payroll_hr_payroll` |
| `currency_id` | `hr.payslip.line` | `l10n_ve_payroll_hr_payroll` |

> **Consecuencia práctica:** se reponen **en el módulo más bajo de la cadena que
> los necesita**, no en cada uno que los usa. Dos declaraciones del mismo campo
> con distinto tipo y el fallo aparece lejos de su causa.

### La regla salarial ya no apunta a su estructura

En Enterprise la regla pertenecía a **una** estructura (`struct_id`, Many2one).
En OCA es la estructura la que lista sus reglas (`rule_ids`, Many2many): una
regla puede estar en varias. Cinco dominios en cuatro módulos dejaron de validar.

El lado inverso se declara sin tocar OCA, pero **no como Many2many normal**:
`hr.payslip.line` hereda de `hr.salary.rule` por prototipo y el campo se clonaría
con la misma tabla y las mismas columnas. Se declara calculado, con `search`
propio — está comentado en
`l10n_ve_payroll_res_config_settings/models/hr_salary_rule.py`.

### El análisis de nómina, reescrito

`hr.payroll.report` era de Enterprise. El módulo que lo personalizaba lo hacía
**parcheando el SQL de Enterprise con `str.replace`**; sin el original no hay
nada que parchear.

Se escribió entero sobre las tablas de OCA, con sus vistas —pivote, gráfico,
lista, búsqueda—, su acción y un menú **«Informes»** del que cuelgan ahora los
informes de los demás módulos, que en v18 colgaban de
`hr_payroll.menu_hr_payroll_report`.

## 4. Dos defectos propios que el barrido destapó

No eran de v20. Estaban ahí desde antes y se vieron al pasar la suite entera.

**El código de la regla salarial se imponía, no se proponía.**
`l10n_ve_payroll_automatic_rule_code` lo calcula a partir de la categoría, y
hasta ahora machacaba cualquier código escrito a mano — en la creación y cada vez
que cambiaba la secuencia.

> Fallaba en silencio y caro: una regla creada con el código `TEST` pasaba a
> llamarse `DED10`, y toda expresión o informe que la buscara por su código
> dejaba de encontrarla. **El recibo salía sin esa línea, sin un solo aviso.**

Ahora el código se propone; si alguien lo escribe, manda él.

**Cancelar un recibo era imposible sin adelanto quincenal configurado.** Un aviso
de configuración —«No se ha configurado la estructura para reiniciar el adelanto
quincenal»— bloqueaba una operación que no tenía nada que ver con él. Ahora, si
no hay nada que reiniciar, no se reinicia nada y ya está.

## 5. Lo que está bloqueado, y por quién

20 módulos. **Ninguno espera trabajo nuestro.**

| Espera a | Módulos | Qué es |
|---|---:|---|
| `res.bank` | 9 | Odoo 20 eliminó el banco como entidad. Decisión aplazada a propósito |
| `hr_payroll_account` | 2 | Contabilización de la nómina, sin migrar aún |
| `hr_timesheet_attendance` | 2 | Enterprise |
| `hr_appraisal` | 2 | Enterprise |
| `planning`, `hr_work_entry_contract_enterprise` | 2 | Enterprise |
| `l10n_ve_payroll_export_payroll_payments` | 2 | No se trajo: depende de `res.bank` |
| `bi_odoo_multi_branch_hr` | 1 | Módulo de terceros |

La lista módulo a módulo está en `BLOQUEADOS.txt`, en el repositorio de la
nómina.

### `res.bank`: por qué se aplazó

`res.bank` existía en v18 y en v20 el archivo ya no está. Odoo lo resolvió
convirtiéndolo en texto: `res.partner.bank.bank_name` es hoy un `Char`.

Para nómina venezolana eso es una pérdida — los archivos de pago al banco
necesitan el **código** de la entidad, no su nombre escrito a mano.

**La decisión está aplazada a propósito** hasta ver qué hace Odoo 20 cuando
salga. Antes que inventar un diseño propio y tener que rehacerlo, se mira el
oficial. Qué observar cuando llegue, y las tres salidas que ya se ven, está en
`MIGRACION.md` del repositorio de la nómina.

## 6. Lo que queda a medias, y está marcado

Dos funcionalidades quedaron **guardadas tras una comprobación de existencia**,
no borradas. Cuando el módulo que falta llegue, se reactivan solas.

**Detección de conflictos de entradas de trabajo.** v20 dejó de materializar
`hr.work.entry` —`generate_work_entries` devuelve diccionarios, no registros— y
con el modelo se fueron `_check_if_error`, `_to_intervals` y el estado
`conflict`. La generación de recibos por lote funciona; lo que no hay es el aviso
de solapes.

**Contabilización.** Sin `hr_payroll_account` no existe `hr.payslip.move_id`. Los
tres sitios que lo miraban —cancelar un recibo, cancelar un lote, marcar como
pagado— comprueban antes si el campo está.

## 7. Cómo se comprueba

Instalar de una vez sobre una base **nueva**, no sobre una que ya los tenga.

```bash
cd /opt/odoo-migration/docker
docker compose run --rm odoo20 odoo -d nomina_test -i <lista de modulos> \
    --test-enable --test-tags <modulos con tests> --stop-after-init
```

> **Por qué en vacío.** Una instalación incremental esconde los choques de orden.
> Dos filas con el mismo id en un `ir.access.csv` pasan sin ruido sobre una base
> que ya los tenía —el segundo `INSERT` es un `UPDATE`— y en una base nueva
> revientan con `ON CONFLICT DO UPDATE command cannot affect row a second time`.

> **Cuidado con `-i`.** Sobre un módulo que ya figura como `installed` **no carga
> sus datos**: Odoo lo salta y el comando termina en verde. Mirar el estado antes
> de elegir entre `-i` y `-u`.

## 8. Dónde vive el código

El repositorio es `gregoriob20/mornix_nomina_20`, y Odoo lo lee desde
`addons/nomina/` del repositorio de código.

Es una **copia, no un symlink**, y a propósito: Docker monta `addons/` entera, y
un enlace que apunte fuera de esa carpeta no se resuelve dentro del contenedor.

```bash
git clone git@github.com:gregoriob20/mornix_nomina_20.git /opt/mornix-nomina-20
python3 scripts/sincronizar_nomina.py          # clon -> addons/nomina/
python3 scripts/sincronizar_nomina.py --subir  # addons/nomina/ -> clon
```

> Se edita **en `addons/nomina/`**, que es lo que ve Odoo, y se sube con
> `--subir` antes de commitear. Al revés, el contenedor sigue viendo lo viejo y
> se depura código que no se está ejecutando.

## 9. La guía de usuario

La rama funcional de la nómina son tres documentos, con capturas y recorridos en
vídeo tomados de una quincena real:

| Guía | Qué cubre |
|---|---|
| [Cómo se configura](../funcional/30-nomina/01-configuracion.md) | Tipo de estructura, estructura, reglas y el contrato del empleado |
| [Procesar una quincena](../funcional/30-nomina/02-la-quincena.md) | Del lote vacío a los recibos confirmados |
| [Historial y análisis](../funcional/30-nomina/03-historial-y-analisis.md) | Historial de contratos, renovación y el informe |

Las capturas se regeneran con un comando, contra una base de demostración con
ocho empleados y dos quincenas:

```bash
cd docker
CAPTURAS_AREA=nomina CAPTURAS_DB=nomina_demo \
    docker compose --profile capturas run --rm capturas
```

> **La base de la nómina es `nomina_demo`, no `odoo20`.** Cada área tiene la
> suya, y por eso las capturas se piden por área: una sola lista obligaría a que
> todas las pantallas existieran en todas las bases.

### Lo que la guía destapó

Escribirla no fue documentar lo que había: mirar las capturas una a una sacó
cinco defectos que ninguna prueba veía, cuatro de ellos **silenciosos**.

| Qué se veía | Qué estaba pasando |
|---|---|
| La columna «Total Ref» en 0,00 | El importe en divisa colgaba de un método del motor de Enterprise que OCA no llama nunca. La nómina en bolívares con referencia en divisa —el motivo de esta localización— llevaba sin funcionar |
| «Importe −21.000,00» en una deducción | Se invertía el signo de la **base** del porcentaje, no solo el del total |
| El análisis vacío con nóminas confirmadas | El informe colgaba de las líneas de jornada, y en v20 un recibo puede no tenerlas |
| El análisis sumando 32.000 donde había 191.705 | Una categoría que el informe no conocía: no suma y no avisa |
| «Expected singleton» al confirmar | Confirmar **más de un recibo a la vez** —lo normal— era imposible |

Los cinco están arreglados y con prueba propia. Es el argumento a favor de
documentar con capturas de verdad: un informe vacío y un recibo con un signo al
revés se ven de un vistazo, y no aparecen en ninguna traza.
