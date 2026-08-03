# Estado de la migracion

Inventario de modulos del cliente y su avance hacia v20. Se llena cuando tengamos
acceso a los repos origen.

## Leyenda de estado

| Estado | Significado |
|---|---|
| `pendiente` | Sin tocar |
| `en curso` | Migracion en progreso |
| `instala` | Instala en v20 sin error, sin probar funcionalmente |
| `probado` | Probado contra el flujo real del cliente |
| `descartado` | No se migra (obsoleto o cubierto por el core de v20) |

## Inventario

| Modulo | Origen | Lineas | Depende de | Estado | Notas |
|---|---|---|---|---|---|
| _pendiente de clonar los repos del cliente_ | | | | | |

## Orden de trabajo sugerido

1. **Modulos sin dependencias entre si** primero, para calibrar cuanto duele el
   salto antes de comprometer estimaciones.
2. **Modulos base del cliente** (los que otros heredan) despues, porque su API
   condiciona al resto.
3. **Modulos de reportes y frontend al final**: son los mas afectados por OWL y
   por el cambio de motor PDF, y conviene atacarlos con el resto ya estable.

## Puntos de atencion conocidos v16 -> v20

Los saltos v16->v17->v18->v19->v20 acumulan cuatro releases. Lo que suele romper:

- **`attrs` y `states` en vistas**: eliminados desde v17. Se reemplazan por
  atributos directos (`invisible`, `readonly`, `required`) con expresion Python.
- **Bundles de assets**: se declaran en el manifest (`assets`), ya no en XML.
- **OWL**: los widgets viejos de JS ya no existen.
- **Vistas `tree` renombradas a `list`** en v18.
- **`name_get()` reemplazado por `_compute_display_name`** en v17.

Cada una se confirma y se anota en [BREAKING-CHANGES.md](BREAKING-CHANGES.md) a
medida que la encontremos en codigo real; la lista de arriba es lo esperado, no
lo verificado.
