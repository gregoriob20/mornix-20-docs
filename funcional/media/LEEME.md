# Videos de la documentación

Los videos que se ven en `/documentacion` se guardan aquí y se publican con el
mismo generador que las capturas.

## Recorridos automáticos (Playwright)

```bash
cd docker
docker compose --profile capturas run --rm capturas \
    bash -c "pip install --quiet playwright==1.49.0 && python3 /scripts/grabar_recorridos.py"
```

Los recorridos están en `scripts/grabar_recorridos.py`: cada uno es una lista
de pasos (goto / click / pausa / scroll). Añadir un video = añadir una entrada
a `RECORRIDOS`. Cuando la interfaz cambie, se regeneran con ese comando, igual
que las capturas.

## Referenciarlos desde el Markdown

La misma sintaxis que las imágenes — se decide por la extensión:

```markdown
![Pie del video](media/recorrido-x.webm)
```

## Video-guías con voz (Remotion)

Estos recorridos son la materia prima. La composición con títulos y narración
sigue `GUIA_VIDEO_TUTORIALES_REMOTION` (Morna Tech) y requiere la skill
`video-tutorial-creator`, que trae el template y los presets — no improvisar
el setup de Remotion: la guía lo prohíbe y sus errores conocidos lo justifican.
Voz sin API key: `edge-tts` con `es-VE-PaolaNeural`.

## Reglas

- **Formato**: `.webm` (lo que graba Playwright) o `.mp4` (lo que rinde Remotion).
- **Peso**: los videos entran en git de momento; si el conjunto crece, se
  moverán a un almacén externo — por eso viven en `media/`, separados.
- **Sin datos reales del cliente en pantalla**, igual que las capturas.
