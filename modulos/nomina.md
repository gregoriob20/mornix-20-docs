# Nómina venezolana — 35 módulos sobre el motor de OCA

> Estado: **los 36 módulos libres instalan de una vez sobre una base vacía y las
> 119 pruebas pasan**, propias y del motor.
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
| Módulos del cliente | 56 en el árbol (54 migrados + el tablero y el portal del empleado, escritos aquí) |
| Motor de nómina | 1 (`payroll`, OCA adaptado) |
| Python | 346 archivos · 15.467 líneas |
| XML | 155 archivos · 9.495 líneas |
| Motor OCA | 32 archivos · 3.655 líneas de Python |
| Pruebas | **119**, en 13 módulos |

Reparto de las pruebas:

| Módulo | Pruebas |
|---|---:|
| `l10n_ve_payroll_hr_hcm` | 35 |
| `payroll` (motor OCA) | 27 |
| `mnx_l10n_ve_payroll_hr_contract_history` | 20 |
| `l10n_ve_payroll_hr_payroll` | 10 |
| `mornix_l10n_ve_payroll_dashboard` | 10 |
| `mornix_l10n_ve_payroll_portal` | 7 |
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

### Dos cosas más, vistas al escribir el paso a paso

- **El botón «Cerrar» del lote solo aparece en Borrador.** Una vez generados
  los recibos, el lote queda «Por verificar» y no hay botón que lo cierre: el
  cierre real es la confirmación de los recibos. Viene así del motor de OCA
  (`invisible="state != 'draft'"` en `close_payslip_run`). Está documentado en
  la guía tal como se comporta; decidir si el lote debe cerrarse solo cuando
  todos sus recibos estén en «Hecho» es una mejora pendiente.
- ~~**La pestaña Nómina de la ficha del empleado sale con rótulos encimados**~~
  **Resuelto el 23 de septiembre de 2026.** No era la maquetación de v20: era
  un `<group>` que `l10n_ve_payroll_bonus` metía **dentro** de
  `//group[@name='contract']`. Un grupo anidado convierte al padre en grupo de
  columnas, no de rejilla etiqueta/valor, y todo lo que había en él —los campos
  del core y los de la localización— se apilaba a lo ancho: los `o_row` encogían
  a un píxel («Bs» sobre «/ mes», el importe ilegible) y la casilla «Recibe
  comisión» perdía su rótulo. Se destapó midiendo el DOM con Playwright
  (`grid-template-columns` del grupo y anchos de cada fila), no a ojo. Cambios,
  cada uno en su módulo:

  | Módulo | Versión | Qué |
  |---|---|---|
  | `l10n_ve_payroll_bonus` | 1.0.3 | El anticipo quincenal va como etiqueta + fila, sin `<group>` anidado |
  | `l10n_ve_payroll_salary_fields` | 1.0.3 | El importe no repite la moneda (`no_symbol`: la enseña el selector); «/ mes» y «/ hora» en línea; la **fecha de fin** en su propia fila, porque «inicio → al → fin → Nuevo contrato» no cabe en la columna (los hijos de un `o_row` encogen por CSS del núcleo) |
  | `mnx_l10n_ve_payroll_hr_contract_history` | 1.0.2 | El botón **Historial** pasa de la fila de fechas a la cabecera «Resumen del contrato» |
  | `l10n_ve_payroll_estimated_profits` | 1.0.2 | Marcadores «Desde / Hasta» |
  | `l10n_ve_payroll_hr_payroll`, `l10n_ve_payroll_employee` | 1.0.3 | El término compartido «Tipo de sueldo» estaba traducido como «Tipo de salario por defecto» |
  | `l10n_ve_payroll_employee_seniority` | 1.0.1 | «Antigüedad» con acento |

  Lo que sigue en inglés en esa pestaña —«Load a Template», «40 hours/week»—
  es del propio `hr` de v20.

### El «Instructor» del empleado dejó de ser obligatorio (`l10n_ve_payroll_employee` 1.0.2)

v18 exigía el instructor (`coach_id`) en la ficha del empleado, y la migración
lo repuso igual en el formulario —v20 lo había quitado de ahí—. El usuario lo
pidió opcional el 23 de septiembre de 2026: no toda empresa asigna instructor,
y exigirlo frenaba el alta de empleados en `odoo20`. Sigue en el formulario,
junto al responsable, sin `required`; correo de trabajo, departamento, cédula y
país siguen siendo obligatorios. Actualizado en `odoo20`, `nomina_demo` y
`auromin`.

## 7. El tablero: lo que se pregunta cada cierre

`mornix_l10n_ve_payroll_dashboard`, escrito en esta migración, responde con
datos las preguntas que se repiten en toda implantación: costo total, reparto
por regla, cestaticket, prestaciones acumuladas, parafiscales, incidencia de
bonos —y cuánto de eso va en divisa— e ISLR retenido.

**El problema de fondo no es sumar: es saber qué suma dónde.** Los códigos de
las reglas los pone cada implantación, así que una lista fija de códigos dentro
del informe funciona en un cliente y miente en el siguiente. La solución es que
cada regla lleve su **concepto** como un campo más:

| | |
|---|---|
| Se propone | a partir del código de la regla (`CESTA` → cestaticket, `FAOVPAT` → FAOV) |
| Se corrige | a mano, y entonces queda marcado y la propuesta no vuelve a tocarlo |
| Lo que no encaja | cae en «Otro» y **sigue sumando**: no desaparece de los totales |

Sobre eso va una vista SQL, `mnx.payroll.dashboard`, con una fila por línea de
recibo y los importes en las dos monedas.

**Dos decisiones que sostienen los números:**

- **El bruto y el neto del recibo no son conceptos**, son sumas de lo demás. Se
  clasifican como «Total del recibo» y **no entran en el costo**: con ellos
  dentro, la demostración daba 593.705 donde el bruto real era 201.000 —el
  triple, y sin que ningún número pareciera raro—.
- **Una retención no es costo.** El ISLR ya está dentro del sueldo del que se
  descuenta; sumarlo otra vez sería contar dos veces el mismo bolívar. Por eso
  el costo son las **asignaciones más los aportes patronales**, y las
  retenciones se informan aparte, en positivo.

Tres roturas de v20 aparecieron construyéndolo:

| Qué | Síntoma |
|---|---|
| `<group string="…">` en una vista de búsqueda | v20 no admite el atributo: la vista queda inválida y **no instala** |
| `context_today().replace(...).strftime(...)` en el dominio de un filtro | El cliente no sabe evaluarlo: la pantalla se cae al abrirla. El filtro de periodo se declara con `date="campo"` |
| Un modelo `_auto = False` sin `_depends` | El ORM no vuelca lo pendiente antes de leer: un recibo recién calculado **no aparece**. En pantalla casi no se nota; en una prueba falla siempre |

Y un defecto propio, del mismo tipo que ya tenía el código de la regla: cambiar
la secuencia de una regla recalcula su código, y eso **borraba la clasificación
hecha a mano** sin decir nada. Se resolvió como lo resuelve
`l10n_ve_payroll_automatic_rule_code` con `nx_code_manual`: una marca de «esto
lo puso una persona».

10 pruebas propias. La guía de uso está en
[El tablero de nómina](../funcional/30-nomina/04-el-tablero.md).

## 8. Cómo se comprueba

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

## 9. Dónde vive el código

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

## 10. La guía de usuario

La rama funcional de la nómina son cinco documentos, con capturas y recorridos en
vídeo tomados de una quincena real:

| Guía | Qué cubre |
|---|---|
| [Cómo se configura](../funcional/30-nomina/01-configuracion.md) | Tipo de estructura, estructura, reglas y el contrato del empleado |
| [Procesar una quincena](../funcional/30-nomina/02-la-quincena.md) | Del lote vacío a los recibos confirmados |
| [Historial y análisis](../funcional/30-nomina/03-historial-y-analisis.md) | Historial de contratos, renovación y el informe |
| [El tablero de nómina](../funcional/30-nomina/04-el-tablero.md) | Costo total, cestaticket, prestaciones, parafiscales e ISLR |
| [El portal del empleado](../funcional/30-nomina/05-el-portal-del-empleado.md) | Acceso al portal y descarga de los recibos confirmados |

Las cinco llevan el **paso a paso** de cada proceso con los nombres de botones
y campos tal como se ven en pantalla, verificados abriendo cada formulario en
`nomina_demo`, no de memoria.

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

## 11. Traducciones

Se revisó **todo lo que sale en pantalla** de los módulos de la casa —campos,
selecciones, menús, vistas y mensajes de Python— exportando las traducciones
es_VE de cada base (`odoo i18n export`) y separando lo que de verdad se ve en
inglés de lo que simplemente no tiene `msgstr` porque ya está escrito en
español en el código. De 8.064 términos, **~330 salían en inglés**;
quedan **0**, y ninguno es una etiqueta: son fragmentos de código,
HTML y expresiones de plantilla que no se traducen.

Las traducciones viven en `scripts/i18n/*_es_VE.json` del repositorio de
migración y las aplica `scripts/completar_traducciones.py`, que reescribe el
`i18n/es_VE.po` de cada módulo a partir de la exportación —con sus referencias,
que es lo que el importador necesita— y se cargan con `odoo i18n import -w`.

> **Tres trampas de v20 que costaron una tarde**, anotadas en
> [Roturas entre versiones](../BREAKING-CHANGES.md): la exportación ya no es
> `--i18n-export` sino el subcomando `odoo i18n export`; un término compartido
> por varios módulos sale como `#. modules: a, b` y se pierde si solo se lee el
> singular; y las traducciones de código Python **exigen la marca
> `#. odoo-python`** en el `.po`, sin la cual el `_()` sigue en inglés aunque
> el `msgstr` esté lleno.

Lo que sigue en inglés no es nuestro: «Load a Template» en la ficha del
empleado y «Connect printers to your PoS» en la caja son huecos de la
traducción es_VE del propio Odoo.

> **Un «Talla» donde debía decir «Nombre».** El recibo de nómina en PDF —y la
> columna Nombre de reglas, categorías y lotes— salía rotulado «Talla». La
> causa estaba en `l10n_ve_payroll_contract_allocation/i18n/es_VE.po`: el
> término compartido `Name` (`#. modules: … payroll`) tenía como traducción la
> de la talla de dotación, y al cargarse pisaba el «Nombre» de todos los
> modelos del motor. Se corrigió a «Nombre» (también en `es.po`) y se recargó
> con `odoo i18n import -w` en `nomina_demo` y `auromin`. Lo destapó la
> vista previa del recibo en el portal del empleado.

## 12. El portal del empleado (`mornix_l10n_ve_payroll_portal` 1.0.0)

Cada empleado descarga sus recibos confirmados desde **Mi cuenta → Recibos de
nómina**, con un usuario de **portal**. Es el mismo patrón que las retenciones
en el portal del proveedor de la localización, y por las mismas razones: el
portal estándar, una tarjeta `portal.entry`, un listado paginado con búsqueda y
una vista previa con el informe incrustado y su descarga.

| Pieza | Qué hace |
|---|---|
| `hr.payslip` + `portal.mixin` | URL `/my/recibos-de-nomina/<id>`, token, `get_portal_url()`, nombre del PDF (`Recibo_de_nomina_SLIP_016_2026_09_16.pdf`) |
| `security/ir.access.csv` | **Una regla** para `base.group_portal`: lectura de `hr.payslip` solo en estado `done` y solo si el empleado apunta al usuario (`employee_id.user_id`) o a su contacto (`employee_id.work_contact_id`) |
| `controllers/portal.py` | `/my/recibos-de-nomina` (listado, ordenación por período o número, búsqueda por número o lote) y `/my/recibos-de-nomina/<id>` (vista previa, `?report_type=pdf&download=1`) |
| `hr.employee.action_mnx_acceso_portal` | Botón **Acceso al portal** en la cabecera de la ficha: abre el `portal.wizard` estándar con el contacto de trabajo ya elegido; exige correo de trabajo |
| Informe | `payroll.action_report_payslip`; si `l10n_ve_payroll_hr_payroll_receipt_payment` está instalado, ese informe ya es el recibo venezolano y sale sin más |

Dos decisiones que conviene conocer:

- **El listado filtra por persona también al usuario interno.** Las retenciones
  del proveedor no lo hacen —el contable quiere verlas todas—; aquí quien entra
  a «Mis recibos» quiere los suyos aunque sea el gerente de nómina. Los
  empleados de quien está conectado se buscan por `user_id` o por
  `work_contact_id`.
- **Los `nx_*_portal()` del recibo van con `sudo()`.** El usuario de portal lee
  el recibo, pero no la ficha del empleado ni la estructura; sin eso el listado
  daba «Error de acceso» al pintar un nombre.

Siete pruebas en `tests/test_portal.py`: el portal ve solo los recibos
confirmados del propio empleado; el de otro y el propio en borrador dan
`AccessError`; la URL, el neto y el nombre del PDF; el botón de acceso con y sin
correo; y, por HTTP, el listado (200, con su recibo y sin el ajeno), la vista
previa, la descarga (`application/pdf`, `attachment`), la redirección del
recibo ajeno y la tarjeta de «Mi cuenta».

> **Las pruebas HTTP piden `--db-filter`.** El `dbfilter=^%d$` del contenedor
> compara con el *host* de la petición, y el servidor de pruebas contesta en
> `localhost`: sin `--db-filter '^nomina_demo$'` las rutas del portal dan 404 en
> la prueba aunque funcionen en el navegador.

Para la demo, `scripts/configurar_portal_nomina.py` pone correo a los ocho
empleados y crea el usuario de portal de Argenis Rondón
(`argenis.rondon@demo.mornix`); es lo que usan las capturas
`nomina-portal-*.png`. **No se ejecuta en la base de un cliente**: allí el
acceso se da desde la ficha del empleado.
