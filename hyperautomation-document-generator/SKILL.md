---
name: hyperautomation-document-generator
description: Convierte notas o archivos de levantamiento/discovery en seis entregables consistentes para el equipo de hiperautomatización: SDD, estimación Planning Poker con Story Points, recomendaciones técnicas en Markdown y HTML, y diagramas As-Is/To-Be en draw.io. Usar cuando el usuario pegue o suba notas de reunión o discovery en texto, .txt, .docx, .doc, .pdf, .xlsx, imágenes o JSON de n8n.
---

# Generador de Documentación para Equipo de Hiperautomatización

Convierte notas/archivos de levantamiento en 6 entregables listos para el equipo:

1. `documento-maestro.md` — Documento Maestro de Automatización en formato SDD.
2. `matriz-estimacion.md` — Matriz de Estimación con Planning Poker y Story Points.
3. `recomendaciones-tecnicas.md` — Recomendaciones Técnicas.
4. `recomendaciones-tecnicas.html` — Versión visual de las recomendaciones.
5. `diagrama-as-is.drawio` — Diagrama del proceso actual.
6. `diagrama-to-be.drawio` — Diagrama del proceso propuesto.

## Stack prioritario del equipo

Priorizar SIEMPRE en este orden al recomendar herramientas:

| Herramienta | Casos de uso primarios |
|---|---|
| Automation Anywhere | RPA UI, automatización desktop, portales sin API, aplicaciones legacy |
| n8n | Orquestación de flujos, integraciones API-a-API, webhooks, scheduling |
| Google Apps Script | Automatizaciones sobre Google Workspace (Sheets, Drive, Gmail, Calendar, Forms) |
| Python | Procesamiento de datos, parsing de archivos, OCR, transformación, ML |
| Bonita BPM | Procesos con aprobaciones humanas, workflows con SLA, gestión de casos |

Solo sugerir alternativas externas si el stack no cubre el caso:
- OCR avanzado: Google Vision API desde Python.
- Notificaciones: Gmail vía Apps Script o nodos Email/Slack de n8n.
- Reportes: Google Sheets + Data Studio o Python + openpyxl.
- Credential Vault: AA Control Room Credential Vault.

## Paso 0 — Leer la entrada

- Texto pegado en chat: usar directamente como `NOTAS_CRUDAS`.
- `.txt`: leer el archivo de entrada.
- `.docx` / `.doc` (incluyendo exports de Confluence con MIME HTML encoded): si es `.doc`, convertirlo primero a `.docx`; extraer párrafos y tablas con `python-docx`. En exports de Confluence, filtrar líneas de HTML puro que comiencen con `<`, `Content-Type:` u otras cabeceras MIME, conservando el contenido útil.
- `.pdf`: extraer texto con `pdftotext`.
- `.xlsx`: extraer texto y contenido de hojas/tablas.
- Imagen (diagrama o screenshot): describir e inferir el proceso usando el contexto visual disponible; marcar inferencias como supuestos.
- JSON de n8n: inspeccionar `nodes` y extraer nombre y tipo de cada nodo.

No afirmar haber leído archivos o ejecutado comandos si no se tuvo acceso real a ellos.

## Paso 1 — Análisis de entidades

Antes de generar, extraer internamente y mostrar al usuario un resumen para validar antes de generar documentos:

| Entidad | Qué buscar |
|---|---|
| Nombre del proceso | Título o denominación del proceso a automatizar |
| Actores / Roles | Personas, sistemas y bots participantes |
| Proceso As-Is | Pasos manuales actuales, en orden |
| Proceso To-Be | Cómo quedaría automatizado |
| Herramienta del stack | Inferir según el stack prioritario si no se menciona |
| Inputs | Datos/archivos de entrada |
| Outputs | Datos/archivos producidos |
| Volumetría | Transacciones/día o mes, tiempos actuales y SLAs |
| Excepciones | Errores o casos fuera del flujo normal |
| Accesos necesarios | Sistemas, credenciales y bases de datos requeridos |

Marcar datos faltantes o inferidos como `⚠️ ASUNCIÓN:` o `⚠️ PENDIENTE VALIDAR:`. No inventar datos del negocio. Si el usuario pide explícitamente generar de inmediato, producir los documentos señalando las lagunas.

## Paso 2 — Documento Maestro de Automatización

Archivo: `documento-maestro.md`.

Usar esta estructura SDD de 11 secciones:

```markdown
# Documento Maestro de Automatización
## EQUIPO DE HIPERAUTOMATIZACIÓN
**Proceso:** [NOMBRE DEL PROCESO]
**Versión:** 1.0 | SOFTWARE DESIGN DOCUMENT (SDD)

### Control de Versiones
| Versión | Autor | Fecha | Descripción | Aprueba manager | Aprueba Negocio |
|---|---|---|---|---|---|
| 1.0 | [Nombre o pendiente] | [Fecha] | Creación del documento | [Manager o pendiente] | [Negocio o pendiente] |

## 1. Objetivo General
- Objetivo funcional y técnico del proceso.
- Motivación y beneficios esperados.
- KPIs: reducción de tiempo, eliminación de reprocesos y SLA objetivo. Si no hay valores, marcar pendiente; no inventarlos.

## 2. Alcance de la Automatización
- Incluye.
- Excluye.
- Supuestos: tabla `# | Supuesto | Detalle`.

## 3. Actores y Responsables
Tabla `Rol del Actor | Nombre`.

## 4. Volumetría y SLAs
Tabla `Item | Número`, incluyendo casos/mes, tiempo manual, SLA objetivo y porcentaje de automatización. Señalar datos faltantes.

## 5. AS-IS del proceso
Pasos numerados del proceso manual actual y referencia a `diagrama-as-is.drawio`.

## 6. TO-BE del proceso
### H1 — Captura/Trigger
Pasos del bot o flujo.
### H2 — Procesamiento principal
Pasos del bot o flujo.
### H3 — Registro y excepciones
Pasos del bot o flujo.
Incluir fragmento JSON ilustrativo del flujo cuando aplique, según herramienta (n8n, AA o Python), y referencia a `diagrama-to-be.drawio`.

## 7. Inputs y Outputs
Tablas `Nombre | Descripción | Formato` para Inputs y Outputs.

## 8. Diseño técnico de la solución
- Orquestador/Scheduler: AA Control Room / n8n / Apps Script trigger, según aplique.
- Motor: AA Bot / n8n workflow / Apps Script / Python.
- Conexión: API / UI RPA / DB / Google Workspace API.
- Logs: Google Sheets / Drive / SQL, según aplique.
- Notificaciones: Gmail Apps Script / n8n Email-Slack / Teams, según aplique.
- Reporting: Google Sheets + Data Studio / Python + openpyxl, según aplique.
- Credentials: AA Control Room Credential Vault cuando corresponda.

### 8.1. Tabla de Accesos Requeridos
| Aplicativo | Tipo | Rol/Perfil | Usuario Ejemplo | Entorno | Descripción |

## 9. Arquitectura Propuesta
Diagrama ASCII end-to-end y descripción narrativa.

## 10. Excepciones y Causales
| Código | Descripción | Acción Automática | Notificación |

## 11. Anexos de documentación
Lista de entregables pendientes (grabación, accesos, UAT, manual, etc.).
```

Mantener numeración y nombres de las 11 secciones. No dejar campos vacíos: usar `Pendiente de confirmar` con la marca de validación correspondiente.

## Paso 3 — Matriz de Estimación

Archivo: `matriz-estimacion.md`. Usar este formato Planning Poker:

```markdown
# Sesión de Estimación (Planning Poker)
## [Nombre de la Iniciativa]
> Propósito: estimación colaborativa del esfuerzo técnico.

## 1. Información General
| Campo | Descripción |
| Nombre de la Iniciativa | [Nombre] |
| Tipo de Iniciativa | Nueva automatización / Mejora / Soporte |
| Responsable DEV | [Nombre o pendiente] |
| Tech Lead | [Nombre o pendiente] |
| Manager | [Nombre o pendiente] |
| Fecha de Estimación | dd/mm/aaaa |
| Sprint / Iteración | Sprint X — Q# Año |

## 2. Contexto de la Iniciativa
Descripción funcional y objetivo.

## 3. Criterios para la Estimación
| Criterio | Consideración aplicada |
| Complejidad técnica | Nivel y justificación |
| Dependencias externas | Sistemas y accesos |
| Datos y volumen | Volumen de registros |
| Riesgos / incertidumbre | Riesgos identificados |
| Reutilización / código previo | Componentes del stack reutilizables |
| Testing y UAT | Tipo de validación |

## 4. Participantes y Estimaciones
> Escala Fibonacci: 1, 2, 3, 5, 8, 13, 20, 40, 100
| Rol / Persona | Estimación (SP) | Argumento breve |

## 5. Discusión y Consenso
Discrepancias y conclusión con SP acordado. Distinguir estimación preliminar de consenso real con el equipo.

## 6. Resultado Final
| Resultado | Valor |
| Puntaje final consensuado | X Story Points |
| Complejidad relativa | Nivel |
| Duración estimada | N días desarrollo + N días QA/UAT |
| Dependencias principales | Lista |
| Herramienta principal sugerida | Herramienta del stack |
```

Incluir esta tabla de referencia del equipo:

| SP | Complejidad | Alcance | Tiempo Total | Ejemplo en stack |
|---:|---|---|---|---|
| 1 | Muy baja | Ajuste mínimo | 0.5–1 día | Cambio de variable en bot AA / nodo n8n |
| 2 | Baja | Script menor | 1–2 días | Export SQL / Apps Script básico |
| 3 | Baja-Media | Una fuente, lógica lineal | 2–3 días | Flujo n8n simple con un input/output |
| 5 | Media | 2+ sistemas, validación condicional | 4–6 días | Carga archivo + procesamiento + logs |
| 8 | Media-Alta | Volumen alto o lógica compleja | 7–10 días | Bot AA UI compleja / n8n 5+ integraciones |
| 13 | Alta | Varias fuentes, dependencias externas | 10–15 días | AA + Python + Sheets + UAT |
| 20 | Muy Alta | End-to-end, infraestructura nueva | 15–20 días | Bonita BPM + n8n + AA con staging |
| 40 | Crítica/Épica | Proyecto completo, varios devs | 20–30 días | AA + Python + APIs + reportes completos |
| 100 | Épica extendida | Arquitectura nueva, múltiples sprints | 30–60 días | Plataforma hiperautomatización nueva |

La estimación debe indicarse como preliminar si no fue consensuada con participantes reales. Justificar SP y duración; no atribuir votaciones ficticias a personas.

## Paso 4 — Recomendaciones Técnicas (Markdown y HTML)

### `recomendaciones-tecnicas.md`

Incluir estas secciones:
1. Resumen Ejecutivo.
2. Herramienta Principal Recomendada (del stack) y justificación.
3. Stack Técnico para este Proceso: tabla `Componente | Herramienta | Rol`.
4. Flujo Técnico Propuesto: diagrama ASCII con herramientas reales del stack.
5. Integraciones Identificadas: tabla `Sistema | Tipo | Herramienta del stack | Complejidad`.
6. Alternativas: opcional, solo si el stack no cubre algo.
7. Consideraciones de Seguridad: AA Credential Vault, mínimo privilegio y logs sin datos sensibles, según aplique.
8. Riesgos y Mitigaciones.
9. Roadmap de Implementación: Fase 1 PoC → Fase 2 MVP → Fase 3 Estabilización.

### `recomendaciones-tecnicas.html`

Generar HTML completo, profesional, adaptable y con todos los datos disponibles del proceso; evitar placeholders vacíos. Incluir:
- Header con colores morado/verde del equipo y badges de herramientas.
- KPIs en tarjetas: casos/mes, tiempo actual, reducción estimada y SP. Si un dato no existe, mostrar claramente `Pendiente de validar`, no inventar un valor.
- Tabla de stack con color-coding: AA `#FED7D7`; n8n `#FEEBC8`; Apps Script `#C6F6D5`; Python `#BEE3F8`; Bonita BPM `#E9D8FD`.
- Flujo técnico en bloque oscuro con fondo `#1A202C` y texto verde `#68D391`.
- Tabla de riesgos con color por severidad.
- Sección de roadmap por fases.
- Footer del equipo.
- HTML válido con estilos incluidos, legible e imprimible.

## Paso 5 — Diagramas draw.io

Crear `diagrama-as-is.drawio` y `diagrama-to-be.drawio` como XML válido importable por diagrams.net.

Reglas generales:
- Swimlanes horizontales por actor/sistema y un carril para “Bot / Sistema Automatizado”.
- Flujo de izquierda a derecha.
- IDs incrementales desde `id="2"`; título en `id="2"`, swimlanes desde `id="10"`.
- Celdas de nodos dentro de su swimlane con `parent` correspondiente.
- Conexiones como celdas edge con `source` y `target` válidos.
- Incluir leyenda.
- As-Is: pasos problemáticos en rojo `fillColor=#f8cecc;strokeColor=#b85450`.
- To-Be: automatizaciones en verde `fillColor=#d5e8d4;strokeColor=#82b366`; pasos manuales restantes en amarillo.

Estilos por tipo:
- Inicio: `ellipse;fillColor=#d5e8d4;strokeColor=#82b366;fontStyle=1;`
- Fin: `ellipse;fillColor=#f8cecc;strokeColor=#b85450;fontStyle=1;`
- Actividad manual As-Is: `rounded=1;fillColor=#f8cecc;strokeColor=#b85450;`
- Actividad automática To-Be: `rounded=1;fillColor=#d5e8d4;strokeColor=#82b366;`
- Actividad manual restante: `rounded=1;fillColor=#fff2cc;strokeColor=#d6b656;`
- Decisión: `rhombus;fillColor=#fff2cc;strokeColor=#d6b656;`
- Sistema/API: `shape=mxgraph.flowchart.database;fillColor=#dae8fc;strokeColor=#6c8ebf;`
- Reporte/Documento: `shape=mxgraph.flowchart.document;fillColor=#fff2cc;strokeColor=#d6b656;`
- Flecha: `edgeStyle=orthogonalEdgeStyle;`

Usar este esqueleto base y reemplazar los campos entre corchetes:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mxfile host="app.diagrams.net" modified="[FECHA]" agent="hyperauto-skill" version="21.0.0">
  <diagram id="[as-is|to-be]-[proceso-slug]" name="[AS-IS|TO-BE] — [Nombre Proceso]">
    <mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1654" pageHeight="1169" math="0" shadow="0">
      <root>
        <mxCell id="0" /><mxCell id="1" parent="0" />
        <!-- TÍTULO id="2" -->
        <!-- SWIMLANES desde id="10" -->
        <!-- NODOS dentro de cada swimlane con parent="[id_swimlane]" -->
        <!-- FLECHAS con edge="1" source="X" target="Y" -->
        <!-- LEYENDA al final -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

Asegurar que los nombres de proceso, actores, pasos y herramientas sean consistentes entre SDD, matriz, recomendaciones y diagramas.

## Paso 6 — Entrega final

Cuando el entorno permita crear y adjuntar archivos, guardar los seis entregables en `/mnt/user-data/outputs/` y presentarlos en este orden:
1. `documento-maestro.md`
2. `matriz-estimacion.md`
3. `recomendaciones-tecnicas.md`
4. `recomendaciones-tecnicas.html`
5. `diagrama-as-is.drawio`
6. `diagrama-to-be.drawio`

Si no se dispone de herramientas para crear/adjuntar archivos, entregar los contenidos en bloques identificados por nombre de archivo y explicar la limitación; nunca afirmar que los archivos quedaron guardados o adjuntos si no es cierto.

Al finalizar, resumir:
- Nombre del proceso y herramienta principal recomendada del stack.
- SP estimados y duración referencial.
- Instrucción para draw.io: “Abre diagrams.net → File → Import from → Device”.
- Notas de calidad y pendientes de validación.

## Casos especiales y consistencia

- Si la entrada es un export de Confluence (`.doc` MIME), extraer texto de párrafos ignorando líneas de HTML puro que comiencen por `<`, `Content-Type:` u otras cabeceras.
- Marcar ambigüedades con `⚠️ ASUNCIÓN:` o `⚠️ PENDIENTE VALIDAR:`.
- Mantener consistencia en nombre del proceso, actores, volumetría, excepciones y herramientas entre los seis entregables.
- Si usa n8n, incluir un fragmento JSON de nodos en la sección 6 del SDD.
- Si usa Automation Anywhere, mencionar Control Room y Credential Vault.
- Si usa Apps Script, especificar el tipo de trigger (time-driven, `onFormSubmit`, `onEdit`, etc.).
- Si usa Bonita BPM, mencionar lanes de usuario humano y lanes de servicio automatizado.
- No recomendar herramientas externas cuando el stack prioritario cubra el caso.
