# 📊 Guía Maestra de Data Science

> Manual de referencia multi-tomo sobre **Data Science** y **Machine Learning** —del fundamento matemático a la puesta en producción— diseñado para leerse en tres capas: técnica, puente y ejecutiva.

[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-blue.svg)](LICENSE)
![Idioma](https://img.shields.io/badge/Idioma-Español-red)
![Formato](https://img.shields.io/badge/Formato-Obsidian%20%2F%20Markdown-7C3AED)
![Enfoque](https://img.shields.io/badge/Enfoque-Conceptual%20·%20sin%20código-2EA043)
![Tomos](https://img.shields.io/badge/Tomos-22-orange)

---

## 🎯 Objetivo

Este repositorio alberga la **Guía Maestra de Data Science**: una base de conocimiento estructurada, viva y autocontenida que recorre el **ciclo de vida completo** de un proyecto de datos —desde la pregunta de negocio hasta el monitoreo en producción—. Su propósito es doble:

- **Ser un manual de referencia** riguroso y a la vez accesible: que un mismo concepto pueda consultarlo un Data Scientist, un Product Owner y un gerente, cada uno en su nivel de profundidad.
- **Servir como fuente de verdad para las decisiones** de Data Science: qué técnica corresponde a cada problema, qué supuestos tiene y cuándo se rompen, cómo interpretar el resultado y cómo explicárselo al negocio.

No es un recetario de código. Es una guía de **criterio**: el *qué* y el *por qué*, no el *cómo se teclea*.

## 👥 Tres audiencias, un solo documento

El lector decide qué leer. Cada bloque está etiquetado visualmente para que sepas de inmediato qué te corresponde:

| Etiqueta | Perfil | Qué encuentra |
|---|---|---|
| 🔧 **Técnico** | Data Scientist · Data / ML Engineer | Definiciones formales, fórmulas, hiperparámetros, supuestos, limitaciones |
| 🧭 **Puente** | Product Owner · Líder técnico · Analista | Cuándo usar qué, trade-offs de decisión, traducción negocio↔técnica |
| 👔 **Ejecutivo** | Gerente · C-level · Stakeholder | Impacto en el negocio, riesgos, costo de hacerlo mal |
| 💡 **Analogía** | Todos | La intuición, con una analogía cotidiana. Sin prerrequisitos |

## 🗂️ Estructura

La guía se organiza en **22 tomos** dentro de [`Guia-Maestra-DS/`](Guia-Maestra-DS/), ordenados según el ciclo de vida de un proyecto de datos.

**Núcleo (00–16) — el recorrido principal:**

- **Fundamentos** — `01` Introducción Ejecutiva · `02` Fundamentos Matemáticos
- **Datos** — `03` Preparación de Datos · `04` EDA · `05` Escalado de Datos
- **Modelado** — `06` Clustering · `07` Modelos Supervisados · `08` Métricas de Evaluación · `09` Reglas de Asociación
- **Rigor y producción** — `10` Validación y Leakage · `11` Mejora de Modelos · `12` Deep Learning · `13` MLOps, XAI y Ética
- **Para el negocio** — `14` Interpretar Resultados (sin programar) · `15` Glosario Ejecutivo · `16` Bibliografía

**Extensión aplicada (17–21) — dominios especializados:**

`17` Series de Tiempo · `18` Causalidad y Uplift · `19` NLP y LLMs · `20` Sistemas de Recomendación · `21` Supervivencia y Bandits

> 🗺️ **Punto de entrada:** abre [`00-MOC-Guia-Maestra.md`](Guia-Maestra-DS/00-MOC-Guia-Maestra.md) — el índice maestro con el mapa de contenido, las rutas de lectura por perfil y el estado de cada tomo.

## 🧭 Cómo usarla

- **En Obsidian (recomendado):** abre la carpeta `Guia-Maestra-DS/` como *vault*. Los wikilinks, los callouts y el grafo de notas cobran vida.
- **En cualquier lector Markdown / GitHub:** cada tomo es un `.md` autocontenido y legible por sí solo; los diagramas son ASCII y las fórmulas, notación inline —sin dependencias externas—.
- **Por perfil:** el índice maestro propone rutas de lectura para ejecutivos (~2–3 h), perfiles puente y técnicos.

## 📐 Convenciones

- **Idioma:** español neutro profesional. Los **términos técnicos se mantienen en inglés** (`pipeline`, `overfitting`, `feature`, `data leakage`, `boosting`…): así el vocabulario coincide con la documentación y las herramientas reales.
- **Sin código:** el contenido vive en el plano conceptual y de decisión, por diseño. La sintaxis concreta de cada librería queda fuera del alcance.
- **Autocontenida:** cada tomo se lee sin los anteriores; los prerrequisitos se **enlazan** con wikilinks, no se repiten.
- **Verificable:** las afirmaciones se citan `(Autor, Año)` y se consolidan en el `16` · Bibliografía.

## 🤖 Doble función: manual humano + base de conocimiento para IA

Además de manual de lectura, la guía está pensada para actuar como **fuente de conocimiento primaria de asistentes de IA**: cuando se pide apoyo en un proyecto de datos, la guía funciona como base de verdad para las **decisiones** (qué hacer y por qué), no para la implementación. Las reglas editoriales y de trabajo que gobiernan este uso viven en [`Instrucciones-Guia-Maestra-DS.md`](Guia-Maestra-DS/Instrucciones-Guia-Maestra-DS.md).

## 🌱 Documento vivo

La guía se mantiene y evoluciona: cada tomo lleva versión y fecha de última revisión en su *frontmatter*, y el índice maestro conserva el tracker de estado. Es un proyecto en mejora continua, no una edición cerrada.

## 📄 Licencia

Distribuido bajo licencia **MIT**. Consulta el archivo [`LICENSE`](LICENSE) para los términos completos.

## ✍️ Autor

**Diego Evans Pérez Paz** — © 2026
