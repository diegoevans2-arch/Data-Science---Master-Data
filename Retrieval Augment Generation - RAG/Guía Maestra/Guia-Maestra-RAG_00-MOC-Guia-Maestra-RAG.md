---
title: "Tomo 00 — MOC · Guía Maestra de RAG"
tags: [rag, moc, indice, guia-maestra]
audiencias: [tecnico, puente, ejecutivo]
tomo: 00
version: 2.2
status: in-progress
type: vault-index
project: guia-maestra-rag
author: El Egypcio
---

# 🗺️ MOC — Guía Maestra de RAG

> [!info] Qué es este documento
> Este es el **Map of Content (MOC)**: el índice maestro de la *Guía Maestra de RAG*. Desde aquí navegas a todos los tomos y consultas el estado de avance. Es el punto de entrada de la guía.

> [!important] Doble propósito de esta guía
> Esta guía es a la vez **manual didáctico para humanos** 📚 y **fuente de conocimiento para Claude** 🤖 cuando se pida apoyo en proyectos de RAG. Además es un **documento vivo**: se mantiene a la vanguardia de forma continua. Ver reglas completas en [[Instrucciones|📖 Instrucciones]].

---

## 📊 Estado del proyecto

**Curso base:** *Retrieval Augmented Generation (RAG)* — DeepLearning.AI (Coursera / USS)
**Estructura del curso:** 5 módulos (temario real indexado).
**Última actualización:** 2026-07-18 — Tomos 02, 03 y 04 generados. **Módulos 1 y 2 completos ✅** (con el retriever cubierto de punta a punta: metadata filtering, keyword search, semantic search y hybrid search). **Tomo 01 revisado a v2.0** para nivelarlo en profundidad con el resto (ver nota de revisión abajo). Pendiente de material: el Módulo 3 en adelante — el autor aún no ha subido ese contenido a `MATERIAL/`.

> [!note] Revisiones de tomos ya publicados
> - **Tomo 03 → v1.2 (2026-07-18).** Se indexaron las láminas *BM25 Scoring* y *BM25 Tunable Parameters*. **La fórmula de BM25 del tomo coincide exactamente con la del curso** — quedaba como el único punto donde la guía podía contradecir a la fuente primaria, y queda cerrado. Se afinaron los rangos de `k1` (1.2–2.0) y `b` (0–1) con la redacción de la lámina.
> - **Tomos 03 y 04 → v1.1 (2026-07-18).** Se indexó el **assignment graded C1M2** (`MATERIAL/EVALUACIÓN modulo 2/`), que no estaba disponible al escribirlos. Cambios: el Tomo 03 incorpora el stack real del curso (**`bm25s`**, no `rank_bm25`) con salida verificada contra la esperada del assignment, más una advertencia sobre `corpus.index()` y los 21 documentos con texto duplicado del dataset; el Tomo 04 reemplaza el ejemplo sintético de RRF por el **caso real del assignment** e incorpora que el curso implementa RRF **sin ponderación `beta`**. La fórmula de RRF y `K=60` quedaron confirmados contra el enunciado del curso.
> - **Tomo 01 → v2.0 (2026-07-18).** El tomo original se escribió antes de que se asentara el estándar de profundidad de la guía y quedó desnivelado (17 KB frente a 40+ KB del resto). La revisión agrega: la **fase de indexing** (que faltaba por completo frente a la de retrieval), el **caso de negocio** que exigía la plantilla para secciones grandes, los **outputs reales del experimento con/sin RAG** con las alucinaciones concretas identificadas, un marco de decisión frente a fine-tuning con señales de error, anti-patrones de inicio, y el mapa Naive/Advanced/Modular RAG. Sin cambios de alcance: el contenido nuevo no invade el territorio del Tomo 02.

Leyenda: ✅ Hecho · 🚧 En curso · ⬜ Pendiente

---

## 📚 Plan de tomos y tracker

Plan realineado al temario real de los 5 módulos del curso. Los tomos 01–11 siguen el curso como fuente primaria; los tomos 12–13 son complementos de vanguardia con bibliografía externa.

| #   | Tomo                                                                                                                                                     |   Módulo    | Estado |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------: | :----: |
| 00  | [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG\|MOC · Índice maestro]]                                                                                       |      —      |   🚧   |
| 01  | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|Introducción a RAG]]                                                                                           |     M1      |   ✅    |
| 02  | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|Fundamentos: LLMs y el pipeline RAG]]                                                             |     M1      |   ✅    |
| 03  | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|Information Retrieval: keyword search (TF-IDF, BM25)]]                                               |     M2      |   ✅    |
| 04  | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|Semantic search, embeddings y hybrid search]]                                                        |     M2      |   ✅    |
| 05  | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|Vector databases y ANN]]                                                                                   |     M3      |   ⬜    |
| 06  | [[Guia-Maestra-RAG_06-Chunking\|Chunking (básico y avanzado)]]                                                                                           |     M3      |   ⬜    |
| 07  | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|Reranking: cross-encoders y ColBERT]]                                                          |     M3      |   ⬜    |
| 08  | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|Generación: transformers, sampling, prompt engineering]]                               |     M4      |   ⬜    |
| 09  | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|Hallucinations, evaluación y agentic RAG]]                                                |     M4      |   ⬜    |
| 10  | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|RAG en producción: monitoring, cost, latency, security]]                                                        |     M5      |   ⬜    |
| 11  | [[Guia-Maestra-RAG_11-Multimodal-RAG-y-Quantization\|Multimodal RAG y quantization]]                                                                     |     M5      |   ⬜    |
| 12  | [[Guia-Maestra-RAG_12-Tecnicas-Avanzadas-Hybrid-HyDE-GraphRAG\|⭐ Complemento · Técnicas avanzadas: HyDE, query transformation, GraphRAG]]                |   Externo   |   ⬜    |
| 13  | [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex\|⭐ Complemento · Frameworks: LangChain / LlamaIndex]]                                              |   Externo   |   ⬜    |
| 15  | [[Guia-Maestra-RAG_15-Glosario-Ejecutivo\|Glosario ejecutivo]]                                                                                           | Transversal |   ⬜    |
| 16  | [[Guia-Maestra-RAG_16-Bibliografia\|Bibliografía]]                                                                                                       | Transversal |   ⬜    |

> [!note] El plano detallado por módulo
> - **Módulo 1 — RAG Overview** → Tomos 01–02 (intro a RAG, aplicaciones, arquitectura, LLMs, information retrieval, labs de llm calls + augmented prompts).
> - **Módulo 2 — Information Retrieval & Search Foundations** → Tomos 03–04 (retriever architecture, metadata filtering, TF-IDF, BM25, semantic search, embedding deepdive, hybrid search + RRF). ⚠️ **Excepción de ruteo:** la lección de *retriever evaluation* del M2 (precision@k, recall@k, MAP@K, MRR) y el Ungraded Lab 2 **no van al Tomo 04**: se consolidan en el **Tomo 09**, junto con la evaluación de la generación, para no partir el tratamiento de métricas en dos tomos distantes.
> - **Módulo 3 — Information Retrieval with Vector Databases** → Tomos 05–07 (ANN, vector databases, Weaviate, chunking + avanzado, query parsing, cross-encoders, ColBERT, reranking).
> - **Módulo 4 — LLMs & Text Generation** → Tomos 08–09 (transformers, sampling, choosing your LLM, prompt engineering, hallucinations, evaluación, agentic RAG, RAG vs. fine-tuning). El Tomo 09 recibe además las **métricas de retrieval del M2** (ver excepción de ruteo arriba).
> - **Módulo 5 — RAG Systems in Production** → Tomos 10–11 (production challenges, evaluation, logging/monitoring/observability, tracing, quantization, cost vs. quality, latency vs. quality, security, multimodal RAG).

> [!tip] ⭐ Tomos de complemento (vanguardia)
> Los tomos 12–13 cubren temas que el curso no aborda explícitamente pero que son práctica estándar en producción (HyDE, query rewriting/decomposition, GraphRAG, frameworks de orquestación). Se construyen con **bibliografía externa verificable** y se mantienen actualizados como parte del mandato de vanguardia. El número 14 queda reservado por si surgen tomos intermedios.
>
> **Corrección de alcance (2026-07-18):** el plan original asignaba *hybrid search con RRF* al Tomo 12 como complemento externo. Al indexar el material del Módulo 2 se constató que **el curso sí cubre hybrid search y RRF en detalle** (fórmula, hiperparámetros `k` y `beta`), de modo que pasó a ser contenido del **Tomo 04** con el curso como fuente primaria. El Tomo 12 queda acotado a lo que efectivamente es externo.

---

## 🧭 Rutas de lectura por perfil

> [!abstract] 👔 Ruta ejecutiva (el "para qué")
> [[Guia-Maestra-RAG_01-Introduccion-a-RAG|01 · Introducción]] → secciones 👔 de [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09 · Evaluación]] y [[Guia-Maestra-RAG_10-RAG-en-Produccion|10 · Producción]] → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|15 · Glosario]]. Entiende qué resuelve RAG, cómo se mide su valor, qué cuesta operarlo y el vocabulario mínimo.

> [!tip] 🧭 Ruta puente (el "cuándo y por qué")
> Toda la guía enfocándote en los bloques 🧭 y las tablas comparativas. Ideal para decidir arquitecturas y trade-offs (keyword vs. semantic vs. hybrid, cuándo rerankear, RAG vs. fine-tuning, cost vs. latency) sin implementar tú mismo.

> [!info] 🔧 Ruta técnica (el "cómo")
> Recorrido completo 01 → 13, con foco en los bloques 🔧 y el código. Base para construir sistemas RAG reales de punta a punta, incluyendo los complementos de vanguardia.

---

## 🔗 Documentos meta del proyecto

- [[Instrucciones|📖 Instrucciones]] — contrato de trabajo, reglas de idioma, bibliografía, flujo y mandato de vanguardia (Claude lo lee al inicio de cada sesión).
- [[Prompts-NotebookLM|🎙️ Prompts para NotebookLM]] — prompts listos para generar un podcast por tomo preservando la intención comunicativa de la guía (términos técnicos sin traducir, triple audiencia, analogía antes que definición).

---

> [!info] Cómo crece esta guía
> El flujo: el autor sube material del curso a la carpeta → Claude lo indexa y genera el tomo correspondiente → se actualiza este tracker. Además, Claude mantiene la guía a la vanguardia de forma proactiva, proponiendo actualizaciones cuando el estado del arte avanza. Ver detalle en [[Instrucciones|📖 Instrucciones]].
