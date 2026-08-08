# Capturas de la documentación

## Regenerarlas automáticamente

Las capturas del sistema NO se toman a mano: las genera Playwright recorriendo
las pantallas con el usuario `capturas.docs` (interfaz en español, permisos de
contable, no de administrador):

```bash
cd docker
docker compose --profile capturas run --rm capturas
```

Las pantallas que recorre están en `scripts/capturar_pantallas.py`. Para añadir
una, se añade una línea a `PANTALLAS` (backend) o a `PORTAL` (portal) con el
nombre del archivo, la ruta y el selector que debe existir antes de disparar.

Cuando la interfaz cambie, se regeneran todas con ese comando y se vuelve a
publicar. A mano solo van las que necesiten estados difíciles de montar.

Las imágenes que se ven en `/documentacion` se guardan **aquí**, junto al
Markdown que las usa, y se publican desde el módulo al generar.

## Cómo añadir una

1. Guarda el PNG en esta carpeta con un nombre que diga qué es:
   `retencion-iva-boton-generar.png`, no `captura1.png`.
2. Refiérela desde el Markdown con ruta relativa:

   ```markdown
   ![El botón «Generar retención» en la factura de proveedor](img/retencion-iva-boton-generar.png)
   ```

   El texto entre corchetes sale como pie de la imagen, y es lo que lee quien
   navega sin ver las imágenes. Escríbelo como una frase, no como un nombre de
   archivo.
3. Regenera y publica:

   ```bash
   python3 scripts/generar_docs_odoo.py
   docker compose run --rm odoo20 odoo -d <base> -u mornix_docs --stop-after-init
   ```

## Qué capturar

- El **ancho justo**: la ventana entera de un portátil se lee mal en la
  documentación. Recorta al bloque del que estás hablando.
- **Sin datos reales del cliente**: RIF, nombres y montos de verdad no deben
  salir. Usa los contactos de demostración.
- **Interfaz en español**, que es como la usa el cliente.

## Formatos

`.png` para capturas de interfaz, `.svg` para diagramas. `.jpg` solo para
fotos, que aquí no debería haber.
