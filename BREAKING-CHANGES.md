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
