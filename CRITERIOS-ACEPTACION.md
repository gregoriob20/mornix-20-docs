# Criterios de aceptacion

Un modulo pasa a desplegable solo si cumple **todos** los criterios de esta lista.
`odoo-qa` los verifica uno por uno con evidencia; `odoo-deploy` no despliega nada
que no tenga el veredicto `APROBADO`.

> Esta lista es la propuesta base del equipo tecnico. **Los criterios de negocio
> (seccion C) los tiene que confirmar el cliente**, porque son los unicos que
> definen si la migracion sirve para lo que la usan. Mientras no esten
> confirmados, un APROBADO solo cubre lo tecnico.

## A. Tecnicos — verificables por comando

| # | Criterio | Como se verifica |
|---|---|---|
| A1 | El modulo instala en una base limpia sin error | `odoo -d limpia -i <modulo> --stop-after-init` |
| A2 | El modulo actualiza sobre una base con datos sin error | `odoo -d con_datos -u <modulo> --stop-after-init` |
| A3 | Los tests del modulo pasan | `odoo -d prueba -u <modulo> --test-enable --stop-after-init` |
| A4 | El log de arranque no tiene ERROR ni CRITICAL nuevos | revisar log completo, comparar contra el arranque limpio |
| A5 | El log no tiene WARNING nuevos sin justificar | cada warning nuevo se explica o se corrige |
| A6 | Las vistas cargan sin error de validacion | abrir cada vista del modulo en la interfaz |
| A7 | Los reportes PDF se generan | generar cada reporte con datos reales |
| A8 | No quedan `attrs`, `states` ni `<tree>` en el XML | `grep -rn "attrs=\|states=\|<tree" addons/<modulo>/` |
| A9 | El manifest declara la version en **forma corta** (`x.y.z`), y se subio respecto al origen | `python3 scripts/verificar_estandar.py <modulo>` |
| A10 | Las dependencias del manifest existen en v20 | instalar en base limpia (lo cubre A1) |

## B. De datos — la parte que de verdad duele

| # | Criterio | Por que |
|---|---|---|
| B1 | Los datos existentes siguen accesibles despues de actualizar | Una migracion que instala limpio pero rompe los datos historicos del cliente es peor que no migrar. |
| B2 | Los campos que cambiaron de tipo o de nombre tienen script de migracion | Sin script, Odoo puede crear la columna nueva vacia y perder el dato viejo sin avisar. |
| B3 | Los asientos contables cuadran igual que antes de migrar | Comparar totales contra la instancia original. |
| B4 | Las secuencias fiscales no se reinician ni saltan | En Venezuela un salto de correlativo fiscal es un problema legal, no un bug cosmetico. |
| B5 | Las tasas de cambio y los montos en doble moneda dan igual | El cliente tiene 6 modulos de doble moneda: es su nucleo. |

## C. De negocio — los confirma el cliente

Estos no los inventa el equipo tecnico. Salen de una conversacion con quien usa
el sistema todos los dias.

| # | Criterio | Estado |
|---|---|---|
| C1 | Los flujos que el cliente usa a diario funcionan igual que antes | por confirmar |
| C2 | Los reportes legales (libros fiscales, retenciones, ARC) dan identico | por confirmar |
| C3 | La impresion fiscal funciona con el hardware real del cliente | por confirmar |
| C4 | El punto de venta opera completo, incluyendo pagos y vuelto | por confirmar |
| C5 | Los usuarios conservan sus permisos y accesos | por confirmar |

- [ ] Definir con el cliente que flujos entran en C1
- [ ] Conseguir juegos de datos reales (anonimizados) para probar C2 y B3
- [ ] Confirmar que impresoras fiscales hay que soportar (C3)

## D. De rendimiento — que no se degrade con volumen

El enemigo en Odoo casi nunca es el CPU: son las consultas que crecen con el
numero de registros (N+1). Codigo que funciona perfecto con los 5 registros de
un test y se arrastra con 50.000. Detalle y herramientas en
`.claude/skills/senior-qa/references/rendimiento.md`.

| # | Criterio | Como se verifica |
|---|---|---|
| D1 | Los computes y flujos criticos tienen presupuesto de consultas fijado en tests | `assertQueryCount` en la suite: el coste NO crece con el tamano del lote |
| D2 | Ningun compute hace `search`/`browse` dentro del bucle `for record in self` | revision de codigo (checklist en la referencia) |
| D3 | Los campos usados en dominios de busqueda frecuentes llevan `index=True` | revision de codigo + `EXPLAIN` sobre la consulta real |
| D4 | Los reportes por periodo (libros fiscales, TXT, XML) se midieron con datos masivos | `odoo populate` + cronometro; el umbral se anota en la doc del modulo |
| D5 | Los crons acotan su lote (limite o ventana), no barren la tabla entera | revision del dominio del cron |

D1 es el unico que previene regresiones en cada commit: los demas se auditan
al migrar cada modulo y cuando aparece lentitud real.

## Lo que un APROBADO no significa

- No significa que el modulo este probado con el volumen de datos real del
  cliente (D cubre que el coste no *crezca* con el volumen, no que se haya
  medido con el volumen exacto de produccion).
- No significa que los modulos de terceros que necesita existan ya en v20.
- No significa que la interfaz se vea igual: OWL cambio el frontend y algunas
  diferencias visuales son inevitables.

Estas tres cosas se validan aparte, con el cliente, antes de comprometer fecha
de salida a produccion.
