# Cambios que rompen: v16 / v18 -> v20 (master)

Bitacora de roturas encontradas durante la migracion. Cada entrada anota en que
version se introdujo el cambio y como se resolvio, para no re-investigar lo mismo
en el siguiente modulo.

## Estado de la version destino

`master` declara internamente:

```python
version_info = (19, 5, 0, ALPHA, 1, '')   # odoo/release.py
```

Es decir, **la serie que reporta el runtime es `19.5`, no `20.0`**. Trabajamos
sobre master tratandolo como v20 por decision del proyecto, porque la rama `20.0`
todavia no existe en `odoo/odoo` (solo hay `16.0`, `17.0`, `18.0`, `19.0` y
`master`).

Consecuencias a re-verificar cuando Odoo publique la rama `20.0`:

- [ ] Codigo del cliente que compare contra `odoo.release.version_info` vera `19.5`.
- [ ] El campo `version` de los manifests (`20.0.x.y.z`) no coincide con la serie.
- [ ] APIs de master pueden cambiar antes del release: revisar todo lo marcado
      como `RIESGO-MASTER` en este archivo.

## Requisitos de plataforma

| | v16 | v18 | v20 (master) |
|---|---|---|---|
| Python minimo | 3.7 | 3.10 | **3.12** |
| PostgreSQL minimo | 10 | 12 | **16** (`MIN_PG_VERSION`) |
| Addons en el core | 487 | 626 | 632 |

## Dependencias Python

Cambios de `requirements.txt` entre 18.0 y master:

- Se eliminaron todos los pines condicionados a `python_version < '3.12'`; el
  archivo quedo mucho mas corto porque master solo soporta Python >= 3.12.
- **Entran**: `python-magic`, `h11==0.16.0`, `tzdata` (solo win32).
- **Sale `python-ldap`**: ya no esta en `requirements.txt`, quedo unicamente en
  `Recommends` de `debian/control`. El `Dockerfile.v20` lo instala aparte.
  - [ ] Confirmar si el cliente usa `auth_ldap`.

## `http_interface` ahora escucha solo en localhost

**Confirmado en codigo, no supuesto.** Default de `--http-interface`:

| version | default |
|---|---|
| 16.0 | `''` (todas las interfaces) |
| 18.0 | `''` (todas las interfaces) |
| master | `127.0.0.1` |

`odoo/tools/config.py` en master fuerza `127.0.0.1` cuando el valor viene vacio.
Cualquier despliegue en contenedor o detras de un proxy reverso deja de responder
al actualizar, sin ningun error en el log: Odoo arranca normal y reporta
`HTTP service running on 127.0.0.1:8069`.

Solucion aplicada en `docker/odoo.conf.tmpl`: `http_interface = 0.0.0.0`. La
exposicion real la limita compose, que publica los puertos solo en el localhost
del host.

- [ ] Revisar la configuracion de produccion del cliente antes de subir a v20.

## La version del manifest ahora se valida (y decide si el modulo existe)

**El hallazgo mas importante hasta ahora. Afecta a los 226 modulos.**

v20 valida el campo `version` del manifest contra la serie que corre. Si no
coincide, marca el modulo `installable=False`:

```
WARNING: The module l10n_ve_nimetrix has an incompatible version,
         setting installable=False
```

`odoo/modules/module.py:453`. **No existe en 18.0.**

### Por que duele

El modo de fallo es silencioso. Odoo:

- registra un **WARNING**, no un error;
- **termina con codigo de salida 0**;
- simplemente no instala el modulo.

Un `-i mi_modulo` sobre un modulo con la version vieja "funciona": el proceso
termina bien y no instala nada. Si uno mira solo el codigo de salida, parece
exito. Nos paso en el primer intento del piloto.

### Que version poner

`check_version()` exige que la version empiece con la serie. Y **la serie de
master es `19.5`, no `20.0`**:

| Version en el manifest | Resultado |
|---|---|
| `18.0.0.11.0` | rechazada, no instala |
| `20.0.1.0.0` | **rechazada tambien** — la serie es 19.5, no 20.0 |
| `19.5.1.0.0` | aceptada, pero queda obsoleta cuando salga la rama 20.0 |
| `1.0.0` | **aceptada**: con 2 o 3 partes, `adapt_version()` le antepone la serie vigente |

**Convencion del proyecto: usar la forma corta `x.y.z`.** Es la unica que
sobrevive al release de v20 sin tener que tocar 226 manifests otra vez.

Detalle en `adapt_version()`: solo antepone la serie si la version tiene 3
partes o menos. Con 4 o 5 partes la deja tal cual, y entonces tiene que empezar
por la serie a mano.

- [ ] Al migrar cada modulo, cambiar `version` a la forma corta.

## `--without-demo=all` ya no acepta valores

Menor, pero rompe los comandos de siempre:

```
WARNING: option --without-demo: since 19.0, invalid boolean value: 'all',
         assume True
```

Desde 19.0 es un booleano. `--without-demo=all` sigue funcionando por
interpretacion benevola, pero lo correcto es `--without-demo` a secas.

## La validacion del `vat` cambio de sitio y ahora normaliza

**Doble cambio en `base_vat`.**

### Donde se valida

| version | mecanismo |
|---|---|
| 18.0 | `@api.constrains('vat', 'country_id') def check_vat` en `base_vat` |
| master | inverse del campo: `vat` -> `_inverse_vat` -> `_check_vat` -> `_run_vat_checks` |

Un modulo que sobreescriba `check_vat` y llame a `super().check_vat()` lanza
`AttributeError: 'super' object has no attribute 'check_vat'`. El override debe
pasar a `_check_vat(self, validation="error")`.

### Que se guarda

`_check_vat` escribe de vuelta el vat **normalizado**: `_format_vat_number`
aplica `stdnum` y elimina los separadores, asi que `J-99800001-6` queda
`J998000016`.

En 18.0 no pasaba para Venezuela: `_fix_vat_number` comparaba el prefijo del vat
contra el codigo de pais (`j-` contra `ve`), no coincidia, y devolvia el valor
sin tocar.

**Consecuencia para las localizaciones que guardan el identificador con
formato**: los reportes, TXT y XML que tomen el numero de `vat` lo veran sin
separadores. Un modulo que exima a su pais del chequeo estandar (como hace
`l10n_ve_nimetrix`) conserva el formato, porque la reescritura vive dentro del
metodo que se esta salteando.

- [ ] Al migrar cada modulo, revisar si compara `vat` contra un patron con
      separadores.

## La API de consultas SQL se rehizo entera

Afecta a cualquier modulo que construya SQL a mano a partir de un dominio —
tipico en reportes y libros. Todo esto **desaparecio** en v20:

| v18 | v20 |
|---|---|
| `check_access_rights('read')` | `check_access('read')` — sobre un recordset vacio comprueba el permiso a nivel de modelo, que es lo que hacia el metodo viejo |
| `check_access_rule(...)` | absorbido por `check_access` |
| `_where_calc(domain)` | `_search(domain)`, que ya devuelve un `Query` |
| `_apply_ir_rules(query)` | lo aplica `_search` por dentro |
| `sql.code` / `sql.params` | **los objetos `SQL` son opacos**: no exponen el texto ni los parametros |

Antes se concatenaban cadenas; ahora se componen objetos `SQL` y se pasa el
resultado entero al cursor:

```python
# v18
query = am._where_calc(domain)
am._apply_ir_rules(query)
cr.execute(f"SELECT ... FROM {query.from_clause.code} WHERE {query.where_clause.code}",
           query.where_clause.params)

# v20
from odoo.tools import SQL
query = am._search(domain)
cr.execute(SQL("SELECT %s FROM %s WHERE %s",
               SQL(select_clause, *select_params),
               query.from_clause,
               query.where_clause))
```

En `SQL(code, *args)`, los argumentos que son objetos `SQL` se insertan como
SQL; el resto viaja como parametro. Es lo que evita la inyeccion.

**Modo de fallo:** el modulo instala perfecto y revienta al generar el reporte.
Nada en la instalacion toca ese camino.

## `ir.config_parameter.get_param` reemplazado por accesores tipados

| v18 | v20 |
|---|---|
| `get_param(key, default)` | `get_str`, `get_bool`, `get_int`, `get_float` |
| `set_param(key, value)` | `set_str`, `set_bool`, `set_int`, `set_float` |

Mismo modo de fallo silencioso: solo aparece cuando se ejecuta el codigo que lo
usa.

## `_uid` ya no existe en los recordsets

En 18.0, `odoo/models.py` definia `_uid = property(lambda self: self.env.uid)`.
En master no existe. Rompe sobre todo en valores por defecto:

```python
# v18
default=lambda s: s._uid
# v20
default=lambda s: s.env.uid
```

Falla en el momento de calcular el default, no al importar, asi que aparece
tarde: el modulo instala bien y revienta al abrir el formulario o al crear un
registro en un test.

- 6 archivos del codigo del cliente lo usan.

## `SingleTransactionCase` eliminada

**Confirmado en codigo.** `odoo/tests/common.py`:

| version | tiene `SingleTransactionCase` |
|---|---|
| 16.0 | si |
| 18.0 | si |
| master | **no** |

En v20 quedan `BaseCase`, `TransactionCase` y `HttpCase`.

El reemplazo es `TransactionCase`, pero el aislamiento cambia:
`SingleTransactionCase` compartia una unica transaccion entre todos los tests de
la clase, `TransactionCase` da una por test con rollback. Los tests que dependian
de datos creados por un test anterior van a fallar, y ese fallo es correcto.

El codigo actual del cliente no la usa (verificado con grep sobre
`/opt/odoo-client/`), asi que hoy no bloquea nada. Queda anotado por si aparece
en codigo que todavia no hemos revisado.

## Motor de reportes PDF

master incorpora el addon **`base_report_paper_muncher`** ("Report Engine: Paper
Muncher"), el motor de render propio de Odoo (https://odoo.github.io/paper-muncher/).
No existe en 18.0. `wkhtmltopdf` sigue referenciado en varios addons del core.

- [ ] Definir con que motor se renderizan los reportes del cliente en v20.
- Nota del entorno: Ubuntu 24.04 empaqueta `wkhtmltopdf 0.12.6-2build2` **sin el
  Qt parcheado** (upstream no publica build para Noble). Los encabezados y pies
  de pagina pueden renderizar distinto que en produccion. Si un reporte se ve
  mal, sospechar de esto antes que del codigo migrado.

## Roturas por modulo

_(Se va llenando durante la migracion.)_

| Modulo | Sintoma | Version que lo introdujo | Solucion |
|---|---|---|---|
| | | | |
