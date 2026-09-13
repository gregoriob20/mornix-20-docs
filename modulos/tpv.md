# Punto de venta — 12 módulos, el 40 % en JavaScript

> Estado: **los 12 instalan, el TPV abre y se puede armar una venta**. La
> pantalla de pago **no funciona todavía**: falta portar la lógica de IGTF y
> doble moneda a la interfaz de v20.
> Origen: `mornix-tech/nx_point_of_sale`, v18 — 12 módulos, 9.543 líneas.
> Destino: **repositorio propio**, `gregoriob20/mornix_pos_20`, montado en
> `addons/point_of_sale/`.

## 1. Por qué este es distinto

| | Líneas | |
|---|---:|---|
| Python | 3.221 | 34 % |
| XML | 2.520 | 26 % |
| **JavaScript** | **3.785** | **40 %** |

En la localización fiscal el JavaScript era el 0 %, y por eso aquella —siendo
mucho más grande— se migró sin sobresaltos de interfaz. El TPV es la aplicación
que más ha cambiado por dentro entre v18 y v20.

**Lo que no bloquea:** ninguna dependencia de Enterprise. Todo lo externo es
Community, y las dos dependencias internas ya estaban migradas
(`l10n_ve_nimetrix` → `l10n_ve_mornix`, `nimetrix_dual_currency` →
`mornix_dual_currency`).

> **Instalar no dice nada.** El JavaScript del TPV no se ejecuta al instalar:
> se compila cuando alguien abre la caja. Los 12 módulos instalaban sin un solo
> error del log **y el tablero no cargaba**. Todo lo de este documento salió de
> abrir el TPV en un navegador y mirar la consola.

## 2. Lo que v20 movió de sitio

`point_of_sale/static/src` se reorganizó entera. Once importaciones son
mudanzas —`app/store/pos_store` → `app/services/pos_store`, los popups bajo
`app/components/popups/`, los hooks bajo `app/hooks/`— y se reapuntan
comprobando cada destino contra el código de v20.

Dos no se mudaron, desaparecieron:

| v18 | v20 |
|---|---|
| `ReceiptScreen` | `FeedbackScreen` — otra cosa: el resumen del pago |
| `OrderWidget` | `OrderDisplay` |

## 3. Las ocho roturas que solo se ven con la caja abierta

Ninguna aparece en el log de instalación. Todas rompen el TPV en ejecución.

| Qué | Síntoma |
|---|---|
| `props.slots.default.__ctx` | El **tablero** no cargaba: «Ocurrió un error». Los menús de terminal alcanzaban el registro por un interno de OWL que v20 no expone |
| `register_payment_method` | El TPV no abría: v20 da de alta los terminales en el registro `pos_payment_providers` |
| Dos módulos, una misma clave | `DuplicatedKeyError`: en v18 el último en cargar ganaba en silencio; v20 exige `{force: true}` |
| `t-inherit="point_of_sale.ReceiptScreen"` | «Missing (extension) parent templates» |
| `Orderline.props.shape` | v20 no declara forma de props; el parche reventaba y se llevaba el TPV entero |
| `PosOrderline.getDisplayData()` | Sustituido por `lineScreenValues`, un getter del **componente** |
| `pos.config` suelto en una plantilla | v20 no deja `pos`, `ui` ni los getters sueltos en el contexto: se llega por `this.` |
| `this.pos.printReceipt()` | Ahora `pos.ticketPrinter.printOrderReceipt({order})` |

Y el terminal de pago cambió de mecanismo entero: el campo pasó de
`use_payment_terminal` a `payment_provider`, y el gancho de
`_get_payment_terminal_selection` a `_get_terminal_provider_selection`.

## 4. Hasta dónde llega hoy

El tablero carga y la caja abre:

![El tablero del TPV, con la caja y su botón](../funcional/img/tpv-tablero.png)

![El TPV abierto, con el control de apertura y los productos](../funcional/img/tpv-productos.png)

Y se arma una venta, **con la doble moneda funcionando**: cada línea lleva su
importe en divisa en rojo, y el pie suma impuestos y total en las dos monedas.

![Dos productos en el carrito, con el importe en divisa en cada línea](../funcional/img/tpv-venta.png)

> El precio en bolívares **no se escribe**: sale de `list_price_usd × tasa`. Es
> como trabaja esta localización, y quien monte una demo tiene que saberlo o los
> productos salen todos a 0,00.

## 5. Lo que falta

**La pantalla de pago no abre.** `nx_pos_dual_currency/models/pos_order.js` —326
líneas, la lógica de IGTF y doble moneda— llama a métodos que v20 renombró:

| v18 | v20 |
|---|---|
| `get_total_paid()` | `amountPaid` |
| `get_total_with_tax()` | `priceIncl` |
| `get_total_tax()` | `amountTaxes` |
| `get_due()` | `totalDue − amountPaid` |

No es un renombrado a ciegas: el módulo **sobrescribe** varios de esos métodos
para meter el IGTF, y esas sobrescrituras hoy no las llama nadie. Hay que
decidir concepto a concepto qué getter de v20 corresponde.

> **El IGTF es un impuesto.** Portarlo mal no rompe la pantalla, cambia lo que
> se le cobra al cliente. Esta parte pide revisión funcional con números reales,
> no solo que compile.

Después de eso: cobrar de verdad, imprimir fiscal, y las pruebas.

## 6. Cómo se reproduce

```bash
# La caja de pruebas
docker compose run --rm odoo20 odoo shell -d tpv_test < scripts/configurar_caja.py

# Abrirla en un navegador y recoger la consola
CAPTURAS_DB=tpv_test docker compose --profile capturas run --rm capturas \
    bash -c "pip install --quiet playwright==1.49.0 && python3 /scripts/abrir_tpv.py"
```

Lo que importa de ese segundo comando no son las capturas: son los errores de
consola. Es la única forma de saber si el TPV funciona.
