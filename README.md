# 📚 Guías Maestras — Data Science & RAG

> Dos manuales de referencia multi-tomo —**Data Science / Machine Learning** y **Retrieval-Augmented Generation**— escritos para leerse en tres capas: técnica, puente y ejecutiva. Cada afirmación, verificada contra su fuente.

[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-blue.svg)](LICENSE)
![Idioma](https://img.shields.io/badge/Idioma-Español-red)
![Formato](https://img.shields.io/badge/Formato-Obsidian%20%2F%20Markdown-7C3AED)
![Guías](https://img.shields.io/badge/Guías-2-0EA5E9)
![Tomos](https://img.shields.io/badge/Tomos-41-orange)
![Fuentes verificadas](https://img.shields.io/badge/Fuentes%20verificadas-460-2EA043)
![Última revisión](https://img.shields.io/badge/Última%20revisión-2026--10--02-informational)

---

## 🎯 Objetivo

Este repositorio alberga **dos bases de conocimiento estructuradas, vivas y autocontenidas**:

- La **Guía Maestra de Data Science** recorre el ciclo de vida completo de un proyecto de datos —de la pregunta de negocio al monitoreo en producción—.
- La **Guía Maestra de RAG** recorre el pipeline completo de Retrieval-Augmented Generation —del keyword search a la puesta en producción, la evaluación y los frameworks de orquestación—.

El propósito de ambas es el mismo, y es doble:

- **Ser un manual de referencia** riguroso y a la vez accesible: que un mismo concepto pueda consultarlo un ingeniero, un Product Owner y un gerente, cada uno en su nivel de profundidad.
- **Servir como fuente de verdad para las decisiones**: qué técnica corresponde a cada problema, qué supuestos tiene y cuándo se rompen, cómo interpretar el resultado y cómo explicárselo al negocio.

## 📖 Las dos guías, y por qué son distintas

Son proyectos hermanos, pero **no se escriben igual** — y la diferencia es deliberada:

| | 📊 **Data Science** | 🔎 **RAG** |
|---|---|---|
| **Tomos** | 25 (+ índice maestro) | 16 (+ índice maestro) |
| **Fuente primaria** | Ninguna — se apoya solo en su bibliografía | Un curso completo de DeepLearning.AI: transcripciones, notebooks y láminas |
| **¿Lleva código?** | **No, por diseño.** Vive en el plano conceptual y de decisión | **Sí.** Código funcional, ejecutado y verificado (en el Tomo 14, las funciones puras; falta probarlo contra PostgreSQL y OpenAI reales) |
| **Cómo se verifica** | Cada afirmación se ancla a una referencia comprobable | Se contrasta contra el material del curso *y* se ejecuta el código |
| **Qué la hace fiable** | 333 fuentes verificadas una por una | 127 fuentes verificadas + código que se corrió de verdad |

> 💡 **Por qué la de DS no lleva código:** sin fuente primaria externa, la bibliografía es el **único** mecanismo de verificación de la guía. Por eso el esfuerzo se puso ahí y no en la sintaxis de cada librería, que envejece y que la documentación oficial ya cubre mejor.

## 👥 Tres audiencias, un solo documento

El lector decide qué leer. Cada bloque está etiquetado visualmente para que sepas de inmediato qué te corresponde:

| Etiqueta | Perfil | Qué encuentra |
|---|---|---|
| 🔧 **Técnico** | Data Scientist · Data / ML / AI Engineer | Definiciones formales, fórmulas, parámetros, supuestos, limitaciones |
| 🧭 **Puente** | Product Owner · Líder técnico · Analista | Cuándo usar qué, trade-offs de decisión, traducción negocio↔técnica |
| 👔 **Ejecutivo** | Gerente · C-level · Stakeholder | Impacto en el negocio, riesgos, costo de hacerlo mal |
| 💡 **Analogía** | Todos | La intuición, con una analogía cotidiana. Sin prerrequisitos |

## 🗂️ Guía Maestra de Data Science — 25 tomos

En [`Guia-Maestra-DS/`](Guia-Maestra-DS/), ordenados según el ciclo de vida de un proyecto de datos.

**Núcleo (01–16) — el recorrido principal:**

- **Fundamentos** — `01` Introducción Ejecutiva · `02` Fundamentos Matemáticos
- **Datos** — `03` Preparación de Datos · `04` EDA · `05` Escalado de Datos
- **Modelado** — `06` Clustering · `07` Modelos Supervisados · `08` Métricas de Evaluación · `09` Reglas de Asociación
- **Rigor y producción** — `10` Validación y Leakage · `11` Mejora de Modelos · `12` Deep Learning · `13` MLOps, XAI y Ética
- **Para el negocio** — `14` Interpretar Resultados (sin programar) · `15` Glosario Ejecutivo · `16` Bibliografía

**Extensión aplicada (17–25) — dominios especializados:**

- `17` Series de Tiempo · `18` Causalidad y Uplift · `19` NLP y LLMs · `20` Sistemas de Recomendación · `21` Supervivencia y Bandits
- `22` Feature Engineering Avanzado · `23` Tabular: DL vs Boosting · `24` Experimentación A/B · `25` Privacidad y Datos Sintéticos

> 🗺️ **Punto de entrada:** [`00-MOC-Guia-Maestra.md`](Guia-Maestra-DS/00-MOC-Guia-Maestra.md) — índice maestro con el mapa de contenido, las rutas de lectura por perfil y el estado de cada tomo.

## 🗂️ Guía Maestra de RAG — 16 tomos

En [`Retrieval Augment Generation - RAG/Guía Maestra/`](Retrieval%20Augment%20Generation%20-%20RAG/Gu%C3%ADa%20Maestra/), siguiendo el pipeline de RAG de principio a fin.

**Sobre el curso (01–11) — el recorrido principal:**

- **Fundamentos** — `01` Introducción a RAG · `02` LLMs y el pipeline RAG
- **Retrieval** — `03` Keyword search (TF-IDF, BM25) · `04` Semantic search, embeddings y hybrid search · `05` Vector databases y ANN (HNSW) · `06` Chunking · `07` Query parsing, scoring y re-ranking
- **Generación y evaluación** — `08` Transformers, sampling y prompt engineering · `09` Hallucinations, evaluación y agentic RAG
- **Producción** — `10` Observability, evaluación y security · `11` Quantization, trade-offs de cost/latency y multimodal RAG

**Complementos de vanguardia (12–14)** — lo que el curso no cubre, con bibliografía externa verificada **antes** de escribir:

`12` ⭐ Query decomposition, multi-query y GraphRAG · `13` ⭐ Frameworks de orquestación: LangChain, LlamaIndex, DSPy — y la opción de no usar ninguno · `14` ⭐ Structured data RAG: Text2SQL, Table QA y consultas sobre datos tabulares

**Transversales:** `15` Glosario Ejecutivo · `16` Bibliografía

> 🗺️ **Punto de entrada:** [`Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG.md`](Retrieval%20Augment%20Generation%20-%20RAG/Gu%C3%ADa%20Maestra/Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG.md)

## 🧭 Cómo usarlas

- **En Obsidian (recomendado):** abre la carpeta raíz como *vault*. Ambas guías comparten espacio de nombres, así que los wikilinks cruzan de una a otra: el tomo de NLP y LLMs de DS enlaza con los de RAG, y viceversa.
- **En cualquier lector Markdown / GitHub:** cada tomo es un `.md` autocontenido y legible por sí solo; los diagramas son ASCII y las fórmulas, notación inline —sin dependencias externas—.
- **Por perfil:** cada índice maestro propone rutas de lectura para ejecutivos, perfiles puente y técnicos.

![Vault de Obsidian con ambas guías](Vaul_Obsidian.png)

## 🤖 Doble función: manual humano + base de conocimiento para IA

Además de manuales de lectura, ambas guías están pensadas para actuar como **fuente de conocimiento primaria de asistentes de IA**. La división de trabajo es explícita: la guía de **DS** es base de verdad para las **decisiones** (qué hacer y por qué), no para la implementación; la de **RAG** sí incluye código funcional y decisiones de arquitectura justificadas, porque su material de origen permite verificarlo.

## 🆕 Qué trae esta versión (2026-10-02)

**57 correcciones repartidas en 28 tomos.** Una verificación cruzada —cada tomo contra los guiones de audio que se generan a partir de él— dejó 58 hallazgos **en los tomos mismos** (21 en la guía de DS y 37 en la de RAG); uno resultó ser un falso positivo y no se tocó. Cada corrección se verificó contra su fuente *antes* de editar: el texto del paper, el código fuente de la herramienta, su documentación oficial o la salida real de los notebooks del curso. Algunas:

- **DS:** en *Tabular: DL vs Boosting*, la invarianza a rotación se le atribuía al árbol y es de las redes —en datos tabulares, una desventaja— (Grinsztajn et al., 2022); un ejemplo de la paradoja de Simpson decía −1 pp y la cuenta da −1,8 pp; DP-SGD estaba rotulado como privacidad diferencial *local* y es *central*; dos umbrales («AUC sospechoso» e «importancia concentrada») tenían un valor distinto en cada tomo y quedaron unificados como reglas de bolsillo.
- **RAG:** `flat_search_cutoff` de Weaviate no depende del tamaño de la colección, solo aplica a búsquedas **con filtro** (se verificó en su código fuente); el hybrid search del assignment usa `relativeScoreFusion`, no RRF; los tomos de quantization y de evaluación traían cifras que no calzaban con su fuente (el blog de Hugging Face y las salidas del lab).
- **Las funciones puras del Tomo 14 (Text2SQL) se ejecutaron por primera vez.** Los validadores SQL dejaban pasar `SELECT 1;DROP TABLE x`, CTE que modifican datos y `SELECT … INTO`, y el bloque de pandas fallaba por un `import` faltante. Ahora los validadores pasan los 23 casos de prueba, y el tomo declara lo que sigue sin probarse: una base PostgreSQL real, el cliente de OpenAI y los ejemplos de LangChain y LlamaIndex.

**Bibliografía.** La guía de DS pasó de 331 a **333 fuentes verificadas** (310 obras + 23 enlaces de documentación oficial); la de RAG, de 119 a **127 fichas**. Las nuevas son las fuentes sobre las que descansan las correcciones —documentación y código fuente de Weaviate, PostgreSQL y Python, el benchmark RGB, la configuración de un modelo de embeddings, OWASP Top 10 para LLM—, y siguen etiquetadas como lo que son cuando no son papers.

## 🕓 Versión anterior (2026-09-06)

**Cuatro tomos nuevos en la guía de DS** (`22` Feature Engineering Avanzado, `23` Tabular: DL vs Boosting, `24` Experimentación A/B, `25` Privacidad y Datos Sintéticos) y **uno nuevo en la de RAG** (`14` Structured Data RAG).

**Escaneo de vanguardia sobre los 19 tomos de DS que faltaban.** Un auditor por tomo leyó el tomo completo y buscó vacíos de vigencia reales contra la literatura y la documentación oficial, con una regla dura: ninguna afirmación de actualidad entra sin verificación y cita. Resultado: **67 hallazgos** incorporados, **19 de ellos correcciones** de afirmaciones que habían quedado falsas o desactualizadas. Algunas:

- Los árboles de scikit-learn **sí** aceptan valores faltantes desde la 1.3, y XGBoost y LightGBM manejan categóricas de forma nativa.
- El intervalo `media ± 1,96·std/√K` sobre los folds de una validación cruzada **no** es un intervalo de confianza válido.
- El techo de 10.000 filas era una decisión de diseño de TabPFN v2, no un límite de los foundation models tabulares.
- La preservación de estructura global de UMAP frente a t-SNE era un efecto de la inicialización, no una ventaja intrínseca.
- `lstsq` de SciPy resuelve por SVD, no por QR; la métrica de Zhang no es simétrica; `n_init` de K-Means ya no vale 10 por defecto.

**Bibliografía.** La guía de DS pasó de 121 a **331 fuentes verificadas** (308 obras + 23 enlaces de documentación oficial); la de RAG, de 52 a **119 obras**. Cada ficha lleva la fecha de verificación y el registro consultado, y las fuentes que no son papers revisados por pares —reportes de fabricante, preprints, normativa, documentación— están etiquetadas como tales para no hacerlas pasar por lo que no son.

## 🌱 Documentos vivos

Ambas guías se mantienen y evolucionan: cada tomo lleva versión y fecha de última revisión en su *frontmatter*, y cada índice maestro conserva su tracker de estado y su bitácora de correcciones. El mantenimiento sigue un protocolo escrito —auditoría estructural, verificación bibliográfica contra fuente primaria, escaneo de vigencia y ejecución del código que se pueda ejecutar— cuyo detalle queda registrado en cada guía. Son proyectos en mejora continua, no ediciones cerradas.

## 📄 Licencia

El contenido propio de estas guías se distribuye bajo licencia **MIT** — consulta [`LICENSE`](LICENSE) para los términos completos. El material de curso incluido como referencia de trabajo pertenece a sus respectivos autores y no está cubierto por esta licencia.

## ✍️ Autor

**Diego Evans Pérez Paz** — © 2026
