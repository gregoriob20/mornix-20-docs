# Documentación · Migración Mornix a Odoo 20

Toda la documentación de la migración de Mornix a Odoo 20 — la localización
venezolana y la nómina. Vivía dentro del repositorio de código
(`mornix_localizacion_20`) y se separó aquí para que pueda consultarse sin
necesidad de acceso al código.

> **El código está en dos repositorios**, no en uno: la localización en
> `mornix_localizacion_20` y la nómina en `mornix_nomina_20`. La ficha
> [`modulos/nomina.md`](modulos/nomina.md) explica por qué y cómo se monta.

## Qué hay

| Carpeta | Para quién | Qué contiene |
|---|---|---|
| `funcional/` | Usuarios y consultores | Guías de uso: retenciones, libros fiscales, guías de despacho, doble moneda, portal del proveedor. Con capturas (`img/`) y vídeos (`media/`) |
| `modulos/` | Desarrollo | Qué se cambió en cada módulo y por qué, defecto por defecto. Una ficha por área: `l10n_ve_mornix` (localización fiscal) y `nomina` |
| `referencia/` | Desarrollo | Material de consulta de la migración |
| `qa/` | QA | Criterios y cobertura |
| `newsletter/` | Cliente | Resúmenes de novedades |

En la raíz, los documentos transversales:

- **`BREAKING-CHANGES.md`** — lo que Odoo 20 rompió respecto a v18, con la
  solución aplicada en cada caso. Es la primera parada ante un error raro.
- **`MIGRATION.md`** — el plan y su estado.
- **`CRITERIOS-ACEPTACION.md`** — qué se considera terminado.
- **`INVENTARIO.md`** — módulos de origen y su destino.
- **`PROPIEDAD-Y-LICENCIAS.md`** y **`censo-licencias.md`** — origen y licencia
  de cada módulo.

## Cómo se publica

Esta documentación no se lee solo aquí: alimenta el **portal de documentación**
de la instancia, servido por el módulo `mornix_docs`.

```
docs (este repo)  →  scripts/generar_docs_odoo.py  →  módulo mornix_docs  →  /documentacion
```

El generador vive en el repositorio de código y lee esta carpeta. Para que la
encuentre, clone ambos repositorios **hermanos**:

```
/opt/odoo-migration/          ← código
/opt/mornix-20-docs/          ← este repositorio
```

…o indique la ruta con la variable de entorno `MORNIX_DOCS_DIR`.

Tras editar cualquier `.md`:

```bash
cd /opt/odoo-migration
python3 scripts/generar_docs_odoo.py     # regenera las páginas y copia las capturas
docker compose run --rm odoo20 odoo -d odoo20 -u mornix_docs --stop-after-init
```

## Convenciones

- **Español**, y se escribe para quien va a leerlo: las guías de `funcional/`
  no presuponen conocimientos de Odoo.
- **Las capturas se generan, no se toman a mano.** `scripts/capturar_pantallas.py`
  las saca de la instancia real con Playwright, así que envejecen con el
  producto en lugar de quedarse obsoletas.
- **Se documenta el porqué, no solo el qué.** En `modulos/`, cada cambio explica
  qué estaba mal antes y qué pasaba si no se arreglaba.
