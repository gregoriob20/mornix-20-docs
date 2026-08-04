# Documentación de la Localización de Nómina — ZOOM (Odoo 18)

**Entrega de transferencia de conocimiento**
Fecha de corte: **24 de julio de 2026**

---

## 1. Qué es este paquete

Este conjunto de documentos describe la **localización de nómina venezolana implantada para ZOOM sobre Odoo 18**, tal como se encuentra desplegada en el repositorio `nx-zoom` a la fecha de corte.

Su propósito es que cualquier persona con perfil técnico o funcional pueda:

- entender **cómo está construida** la solución y qué hace cada módulo;
- ubicar **dónde se calcula cada concepto** de nómina y con qué criterio legal;
- **operar el día a día** (lotes, recibos, TXT bancarios, reportes) sin depender de conocimiento tácito;
- conocer el **estado real** de lo entregado y lo que continúa abierto.

Todo lo aquí descrito está anclado a evidencia verificable en el repositorio: rutas de archivo, versiones de manifiesto y commits. No hay afirmaciones de memoria sin respaldo.

---

## 2. Alcance

El paquete cubre la **localización de nómina de ZOOM que vive en el repositorio `nx-zoom`** (`git@github.com:nimetrix/nx-zoom.git`): el módulo núcleo `nx_zoom_review_l10n_ve_payroll` y los módulos satélite de nómina, préstamos, entradas de trabajo, fideicomiso, exportación bancaria, reportes legales y contabilización.

Los módulos de la localización base (`l10n_ve_payroll_*`) se mencionan **únicamente como dependencias** donde el grafo de módulos lo exige; su interior no forma parte de esta entrega.

---

## 3. Cómo leer el paquete

| # | Documento | Para quién | Cuándo consultarlo |
|---|---|---|---|
| **01** | [Arquitectura y módulos](01_ARQUITECTURA_Y_MODULOS.md) | Técnico | Antes de tocar cualquier código: qué módulo hace qué y de quién depende |
| **02** | [Motor de cálculo, helpers y reglas](02_MOTOR_CALCULO_HELPERS_REGLAS.md) | Técnico / Funcional avanzado | Cuando haya que ajustar una regla salarial o entender de dónde sale un número |
| **03** | [Prestaciones sociales y fideicomiso](03_PRESTACIONES_Y_FIDEICOMISO.md) | Técnico / Funcional | Para el cálculo mensual de prestaciones, el tablero analítico y los TXT del banco |
| **04** | [Runbook operativo](04_RUNBOOK_OPERATIVO.md) | Analista de nómina / Soporte | En el día a día: generar lotes, regenerar entradas, emitir TXT, enviar recibos |
| **05** | [Estado, pendientes y riesgos](05_ESTADO_PENDIENTES_Y_RIESGOS.md) | Líder técnico / Gerencia | Para saber qué está cerrado, qué queda abierto y dónde están los riesgos |
| **06** | [Cronología del trabajo](06_CRONOLOGIA_TRABAJO.md) | Todos | Para reconstruir el porqué de una decisión y ubicar el material ya entregado |

**Ruta sugerida de lectura**

- *Perfil técnico que recibe el sistema:* 01 → 02 → 03 → 05
- *Analista de nómina:* 04 → 03 → 05
- *Responsable del proyecto:* 05 → 06 → 01

---

## 4. Cifras de referencia a la fecha de corte

| Indicador | Valor |
|---|---|
| Módulos de nómina y RRHH en `nx-zoom` | **35** |
| Versión del módulo núcleo `nx_zoom_review_l10n_ve_payroll` | **18.0.1.18.47** |
| Tamaño del módulo núcleo | 46 archivos `.py` · 17.625 líneas · 28 vistas XML |
| Pruebas automatizadas en los módulos de nómina | **254** |
| Compañías en producción | 3 — ZIS, CCZ, ZLS |

---

## 5. Material complementario ya entregado

Además de este paquete, ZOOM y el equipo de Recursos Humanos ya cuentan con material de uso e inducción entregado previamente, listado en el documento [06 — Cronología del trabajo](06_CRONOLOGIA_TRABAJO.md), sección *Material entregado*: guías de uso para el analista de nómina, plan de inducción, presentaciones del día a día e informes de estatus del proyecto.

---

## 6. Envío

Este paquete se hace llegar **por correo electrónico a las personas pertinentes**. Los archivos están en formato Markdown, legibles en cualquier editor de texto y convertibles a PDF o documento de oficina sin pérdida de contenido.

---

## 7. Contenido de la carpeta

```
Cierre_Mornix_Jul2026/
├── 00_INDICE_ENTREGA.md                 ← este documento
├── 01_ARQUITECTURA_Y_MODULOS.md
├── 02_MOTOR_CALCULO_HELPERS_REGLAS.md
├── 03_PRESTACIONES_Y_FIDEICOMISO.md
├── 04_RUNBOOK_OPERATIVO.md
├── 05_ESTADO_PENDIENTES_Y_RIESGOS.md
└── 06_CRONOLOGIA_TRABAJO.md
```
