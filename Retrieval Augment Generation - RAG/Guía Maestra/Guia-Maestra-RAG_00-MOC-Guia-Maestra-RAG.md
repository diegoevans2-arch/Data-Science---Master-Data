---
title: "Tomo 00 — MOC · Guía Maestra de RAG"
tags: [rag, moc, indice, guia-maestra]
audiencias: [tecnico, puente, ejecutivo]
tomo: 00
version: 2.3
updated: 2026-09-05
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
**Última actualización:** 2026-09-05 — 🔧 **PROTOCOLO DE MANTENIMIENTO (§9 de Instrucciones) ejecutado sobre toda la guía.** Hallazgos y correcciones: (1) el **[[Guia-Maestra-RAG_14-Structured-Data-RAG|Tomo 14 · Structured Data RAG]]**, generado el 2026-08-28, no estaba en este tracker, **ningún tomo lo enlazaba** (nota huérfana) y su bibliografía no había pasado por verificación — hoy queda integrado: fila en el plan, navegación T13 → T14 → T15, rama propia en el diagrama (GATE 0), y bibliografía en el Tomo 16 §17 con **4 errores corregidos** (autor "Baber" → Bahdanau; título/autores de DAIL-SQL; año/venue de BIRD y una cifra que no salía del paper; URL de LangChain muerta) más un import inexistente en su código. (2) Las referencias de los **Tomos 12 y 13** —verificadas al escribirlos— **nunca se habían consolidado en el Tomo 16**: hoy sí (§15–§16, 48 fichas). (3) La revisión del 2026-08-28 (T04 v1.4; T06, T09, T10, T12, T13 v1.1) tampoco había quedado registrada aquí; sus 11 fuentes nuevas se verificaron: **una referencia inexistente en el T04** ("Yamada et al., 2024") se reemplazó por sus dos fuentes reales y se corrigió un título en el T06 (Tomo 16 §18). (4) Reparados: un wikilink roto en el T12 (apuntaba a un tomo inexistente), un ancla rota en el T02 y la navegación del T15. (5) Episodio 14 del podcast agregado en `Prompts-NotebookLM.md`. La guía tiene hoy **16 tomos + el diagrama de flujo**: 11 del curso, 3 complementos (12–14) y 2 transversales.

**Anterior:** 2026-07-29 — 🏁🏁 **GUÍA COMPLETA: 15/15 TOMOS.** Con el Tomo 13 se cierra el cuerpo entero: **11 tomos sobre el curso** (01–11, Módulos 1–5), **2 complementos de vanguardia** (12–13, con bibliografía externa verificada antes de escribir) y **2 transversales** (15 glosario, 16 bibliografía). El Tomo 13 detectó además **dos imprecisiones en el Tomo 10**, ya corregidas: la licencia de Arize Phoenix es **Elastic-2.0** (*source-available*, no OSI) y no "open-source" como dice el curso; y las convenciones GenAI de OpenTelemetry siguen en **`Status: Development`** — no hay estándar estable de tracing para GenAI a julio de 2026.

**Anterior:** 2026-07-28 — **MÓDULO 5 + TRANSVERSALES COMPLETOS. 13 de 15 tomos hechos.** Tras cerrar el M5 se generaron los dos tomos transversales: el **[[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]** (154 términos consolidados, con una sección nueva de **colisiones de vocabulario** que solo se detectan al consolidar — la más grave: `top_k` significa dos cosas incompatibles según el tomo) y el **[[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 · Bibliografía]]** (**52/52 obras verificadas contra fuente primaria**, con 4 errores corregidos y 6 fichas precisadas). Solo quedan los complementos de vanguardia 12 y 13.

**Anterior:** 2026-07-28 — **MÓDULO 5 COMPLETO. EL CURSO QUEDA CUBIERTO DE PUNTA A PUNTA.** Se indexó todo el material del M5 (10 transcripciones, quiz, `C1M5_Ungraded_Lab_1` y assignment graded `C1M5`) y se generaron los **Tomos 10 y 11**. Con esto, los **11 tomos que siguen el curso como fuente primaria están terminados**: retriever (M1–M3), generador (M4), evaluación (M4+M2) y producción (M5). Lo que queda son los **complementos de vanguardia** (Tomos 12–13) y los **transversales** (15 glosario, 16 bibliografía), ninguno bloqueado por material. El reparto del M5 entre los Tomos 10 y 11 cambió respecto al plan original — ver la corrección de plan más abajo.

> [!tip] ✅ Excepción de ruteo del Módulo 2: CERRADA
> Las métricas de retrieval del M2 (precision@k, recall@k, MAP@K, MRR) y su Ungraded Lab 2 (20 Newsgroups) se documentaron en el **Tomo 09 §2**, junto a las métricas del generador. El objetivo de la excepción —no partir el tratamiento de métricas entre dos tomos distantes— se cumplió: el Tomo 09 es ahora el punto único de consulta para evaluación, tanto del retriever como del LLM.

> [!note] 🔗 Enlaces obsoletos corregidos al abrir el Tomo 12 (2026-07-28)
> La corrección de plan nº1 (hybrid search + RRF del Tomo 12 → **Tomo 04**) y la nº2 (HyDE → **Tomo 07**) se aplicaron en su momento al plan, pero **dos wikilinks quedaron apuntando al contenido antiguo** durante meses. Al abrir el Tomo 12 se detectaron y corrigieron:
> - **Tomo 03** prometía *"Hybrid search en profundidad (RRF, pesos) → Tomo 12"*. Ahora apunta al **Tomo 04 §6**, donde efectivamente está.
> - **Tomo 04** prometía *"técnicas avanzadas (HyDE, query transformation, GraphRAG) → Tomo 12"*. HyDE ahora apunta al **Tomo 07**; el resto sigue en el 12.
>
> **Nombre de archivo del Tomo 12 renombrado** a `...12-Query-Decomposition-Multi-Query-y-GraphRAG`. El anterior (`...12-Tecnicas-Avanzadas-Hybrid-HyDE-GraphRAG`) anunciaba *Hybrid* y *HyDE*, que ya **no** son su contenido. Seguro de hacer porque el archivo aún no existía; los 5 wikilinks entrantes se actualizaron de forma atómica.
>
> **Lección:** mover contenido entre tomos no basta con registrarlo en el plan — hay que **buscar los wikilinks que prometían ese contenido**. Un `grep` del nombre de archivo del tomo afectado, cada vez que se mueve algo, lo habría atrapado el mismo día.

> [!warning] ⚠️ Corrección de plan derivada del material del Módulo 5 (2026-07-28)
> **Quantization se movió del Tomo 11 al bloque de trade-offs, y el reparto del M5 cambió.** El plan original agrupaba *"multimodal RAG y quantization"* en el Tomo 11, probablemente porque ambos suenan a temas de vanguardia. Al indexar el material se constató que **el curso enseña quantization como la base de los trade-offs de cost y latency**, no como un tema afín a multimodal: la lección la abre diciendo *"explorarás estos trade-offs en los próximos videos, pero primero introduzcamos un concepto importante, quantization"*, y las dos lecciones siguientes (cost y latency) la referencian de forma continua (modelos cuantizados, binary quantization para acelerar el retrieval). Separarla de cost/latency habría partido el arco pedagógico del curso.
>
> **Reparto final de las 9 lecciones de contenido del M5:**
> - **Tomo 10** ← production challenges · componentes de observability · plataformas (Phoenix) · custom datasets · **security**. Ancla el *Ungraded Lab 1* (tracing con OpenTelemetry y Phoenix).
> - **Tomo 11** ← **quantization** · cost · latency · multimodal RAG. Ancla el *assignment graded C1M5* (que es de optimización de costo/tokens).
>
> **Nombre de archivo del Tomo 11: SÍ cambió**, a diferencia del caso del Tomo 07. La razón es que aquí era seguro hacerlo: el archivo **todavía no existía** y los 4 wikilinks entrantes (3 en el Tomo 05, 1 en el Tomo 08) se actualizaron de forma atómica en la misma sesión, verificados por búsqueda. En el caso del T07 el archivo ya estaba publicado y apuntado desde cinco tomos, y por eso allí se conservó el nombre.

> [!warning] ⚠️ Correcciones de plan derivadas del material del Módulo 3
> Al indexar el M3 se constató que dos cosas planificadas como "complemento externo" **están en el curso**, igual que pasó antes con hybrid search y RRF:
> 1. **HyDE es contenido del curso** (lección de query parsing del M3), no vanguardia externa. Se documenta en el **Tomo 07** con el curso como fuente primaria. El **Tomo 12** queda acotado a lo que sí es externo: query decomposition, multi-query, GraphRAG.
> 2. **El Tomo 07 amplió su alcance** para recibir las tres lecciones de *query time* del M3 (query parsing + arquitecturas + re-ranking). Su título visible cambió a *"Query parsing, arquitecturas de scoring y re-ranking"*; **el nombre de archivo se mantiene** (`...07-Reranking-Cross-Encoders-y-ColBERT`) para no romper los wikilinks que ya lo apuntan desde los Tomos 02–06.

> [!note] Revisiones de tomos ya publicados
> - **Tomos 04, 05, 06 y 07 → nivelación de casos de negocio (2026-07-25).** Una auditoría contra el checklist §12 de [[Instrucciones|📖 Instrucciones]] detectó que estos cuatro tomos **no tenían el callout `[!example]` 📊 de caso de negocio**, mientras los Tomos 01–03 sí. La plantilla lo define como discrecional (*"solo en secciones grandes donde valga la pena"*), pero cuatro de siete sin él era una deriva — y justo en el bloque que le habla al perfil ejecutivo 👔. Se agregó uno por tomo, con industria fresca (ya estaban usadas retail, telco y banca) y mostrando la técnica propia de cada tomo: **T04 seguros** (hybrid search y el vocabulary mismatch de las pólizas), **T05 medios** (el archivo de 12 M de piezas donde el prototipo kNN colapsó), **T06 salud** (el protocolo clínico cortado a mitad de dosis), **T07 legal** (los documentos que se recuperaban pero llegaban en el puesto 14). **Los 7 tomos tienen ahora su caso de negocio.**
> - **Tomo 04 → v1.2 (2026-07-25).** Se agregó la reconciliación **`alpha` ≡ `beta`**: la clase teórica del M2 llama `beta` al peso del lado semántico en hybrid search, pero el parámetro real en Weaviate (y el estándar de facto) se llama **`alpha`**. Incluye la advertencia de que algunas implementaciones **invierten el sentido** y de verificar con un caso extremo (`alpha=0` / `alpha=1`) antes de tunear.
> - **Tomo 03 → v1.2 (2026-07-18).** Se indexaron las láminas *BM25 Scoring* y *BM25 Tunable Parameters*. **La fórmula de BM25 del tomo coincide exactamente con la del curso** — quedaba como el único punto donde la guía podía contradecir a la fuente primaria, y queda cerrado. Se afinaron los rangos de `k1` (1.2–2.0) y `b` (0–1) con la redacción de la lámina.
> - **Tomos 03 y 04 → v1.1 (2026-07-18).** Se indexó el **assignment graded C1M2** (`MATERIAL/EVALUACIÓN modulo 2/`), que no estaba disponible al escribirlos. Cambios: el Tomo 03 incorpora el stack real del curso (**`bm25s`**, no `rank_bm25`) con salida verificada contra la esperada del assignment, más una advertencia sobre `corpus.index()` y los 21 documentos con texto duplicado del dataset; el Tomo 04 reemplaza el ejemplo sintético de RRF por el **caso real del assignment** e incorpora que el curso implementa RRF **sin ponderación `beta`**. La fórmula de RRF y `K=60` quedaron confirmados contra el enunciado del curso.
> - **Tomo 01 → v2.0 (2026-07-18).** El tomo original se escribió antes de que se asentara el estándar de profundidad de la guía y quedó desnivelado (17 KB frente a 40+ KB del resto). La revisión agrega: la **fase de indexing** (que faltaba por completo frente a la de retrieval), el **caso de negocio** que exigía la plantilla para secciones grandes, los **outputs reales del experimento con/sin RAG** con las alucinaciones concretas identificadas, un marco de decisión frente a fine-tuning con señales de error, anti-patrones de inicio, y el mapa Naive/Advanced/Modular RAG. Sin cambios de alcance: el contenido nuevo no invade el territorio del Tomo 02.

Leyenda: ✅ Hecho · 🚧 En curso · ⬜ Pendiente

---

## 📚 Plan de tomos y tracker

Plan realineado al temario real de los 5 módulos del curso. Los tomos 01–11 siguen el curso como fuente primaria; los tomos 12–14 son complementos de vanguardia con bibliografía externa.

| #   | Tomo                                                                                                                                                     |   Módulo    | Estado |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------: | :----: |
| 00  | [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG\|MOC · Índice maestro]]                                                                                       |      —      |   🚧   |
| 01  | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|Introducción a RAG]]                                                                                           |     M1      |   ✅    |
| 02  | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|Fundamentos: LLMs y el pipeline RAG]]                                                             |     M1      |   ✅    |
| 03  | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|Information Retrieval: keyword search (TF-IDF, BM25)]]                                               |     M2      |   ✅    |
| 04  | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|Semantic search, embeddings y hybrid search]]                                                        |     M2      |   ✅    |
| 05  | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|Vector databases y ANN (HNSW)]]                                                                            |     M3      |   ✅    |
| 06  | [[Guia-Maestra-RAG_06-Chunking\|Chunking (básico y avanzado)]]                                                                                           |     M3      |   ✅    |
| 07  | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|Query parsing, arquitecturas de scoring y re-ranking]]                                         |     M3      |   ✅    |
| 08  | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|Generación: transformers, sampling, prompt engineering]]                               |     M4      |   ✅    |
| 09  | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|Hallucinations, evaluación y agentic RAG]]                                                |   M4 + M2   |   ✅    |
| 10  | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|Producción: observability, evaluación y security]]                                                              |     M5      |   ✅    |
| 11  | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|Quantization, trade-offs de cost/latency y multimodal RAG]]                              |     M5      |   ✅    |
| 12  | [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG\|⭐ Complemento · Técnicas avanzadas de query: decomposition, multi-query, GraphRAG]]     |   Externo   |   ✅    |
| 13  | [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex\|⭐ Complemento · Frameworks: LangChain, LlamaIndex, DSPy — y no usar ninguno]]                     |   Externo   |   ✅    |
| 14  | [[Guia-Maestra-RAG_14-Structured-Data-RAG\|⭐ Complemento · Structured Data RAG: Text2SQL, Table QA y consultas sobre datos tabulares]]                  |   Externo   |   ✅    |
| 15  | [[Guia-Maestra-RAG_15-Glosario-Ejecutivo\|Glosario ejecutivo]]                                                                                           | Transversal |   ✅    |
| 16  | [[Guia-Maestra-RAG_16-Bibliografia\|Bibliografía]]                                                                                                       | Transversal |   ✅    |

> [!tip] 🗺️ El plano visual de la guía completa
> [[Guia-Maestra-RAG_Diagrama-Flujo-ASCII|Diagrama ASCII · Flujo completo de procesos RAG]] — los 15 tomos puestos en **un solo flujo**, de arriba abajo: las 6 fases (0 decidir · 1 indexar · 2 responder · 3 evaluar · 4 producir · 5 optimizar), los 7 gates de decisión en el orden correcto (Zoom D), y una etiqueta `T##` en cada paso para saltar al tomo que lo desarrolla. Incluye zooms del retriever completo, de agentic RAG y del ciclo de producción, más un mapa de cobertura tomo por tomo.
>
> Sirve para lo que la tabla de arriba no puede: **ubicar dónde cae un problema concreto** antes de entrar al tomo que lo trata. Trae su propia guía de lectura por perfil (👔 solo las fases y los gates · 🧭 fases + los `[!]` de trade-off · 🔧 el diagrama completo con los zooms).

> [!note] El plano detallado por módulo
> - **Módulo 1 — RAG Overview** → Tomos 01–02 (intro a RAG, aplicaciones, arquitectura, LLMs, information retrieval, labs de llm calls + augmented prompts).
> - **Módulo 2 — Information Retrieval & Search Foundations** → Tomos 03–04 (retriever architecture, metadata filtering, TF-IDF, BM25, semantic search, embedding deepdive, hybrid search + RRF). ⚠️ **Excepción de ruteo:** la lección de *retriever evaluation* del M2 (precision@k, recall@k, MAP@K, MRR) y el Ungraded Lab 2 **no van al Tomo 04**: se consolidan en el **Tomo 09**, junto con la evaluación de la generación, para no partir el tratamiento de métricas en dos tomos distantes.
> - **Módulo 3 — Information Retrieval with Vector Databases** → Tomos 05–07. **Ruteo final:** T05 = kNN → ANN → NSW → HNSW + vector databases + Weaviate (Ungraded Lab 1); T06 = chunking básico y avanzado (Ungraded Lab 2); T07 = query parsing (rewriting, NER/GLiNER, **HyDE**) + arquitecturas de scoring (bi-encoder, cross-encoder, ColBERT) + re-ranking (assignment graded C1M3). El T07 agrupa las tres lecciones de *query time* (antes y después del retrieval), mientras el T06 se queda con lo de *index time*.
> - **Módulo 4 — LLMs & Text Generation** → Tomos 08–09. **Ruteo final (9 lecciones):** T08 = transformer, sampling, elección de LLM, prompt engineering básico y avanzado (**Ungraded Labs 1 y 2**); T09 = hallucinations, evaluación con RAGAS, agentic RAG y RAG vs. fine-tuning (**assignment graded C1M4**). El Tomo 09 recibe además las **métricas de retrieval del M2** (ver excepción de ruteo arriba).
>   - ⚠️ **Corrección de ruteo (2026-07-26):** el **Ungraded Lab 2 del M4 es de prompt engineering**, no de evaluación como sugería el orden de las lecciones. Su contenido (clasificador con few-shot, parámetros condicionados a la tarea, salida estructurada con Pydantic) se documentó en el **Tomo 08 §4.4**.
>   - 📄 **Archivo huérfano detectado:** `clothing_ft_format.csv` (1.000 filas, columna única `text`, formato de fine-tuning causal) está en la carpeta del assignment pero **no se referencia en ningún archivo del curso**. El assignment **no hace fine-tuning**: es un chatbot RAG con router de LLM. Se documenta como residuo, probablemente de otra versión del material.
> - **Módulo 5 — RAG Systems in Production** → Tomos 10–11. **Ruteo final (9 lecciones de contenido):** T10 = production challenges + observability (componentes, plataformas/Phoenix, custom datasets) + security, con el **Ungraded Lab 1** (tracing con OpenTelemetry y Phoenix); T11 = quantization + cost + latency + multimodal RAG, con el **assignment graded C1M5**. Ver la corrección de plan arriba: quantization se movió al bloque de trade-offs porque el curso la enseña como su fundamento.
>   - 🐛 **Hallazgo en el Ungraded Lab 1:** el índice del notebook enlaza una sección *"5 - Evaluating a RAG system"* que **no existe**, y `requirements.txt` fija `arize-phoenix-evals` que **nunca se importa**. El lab se titula "tracing and evaluation" y solo hace tracing — mientras el quiz del módulo sí evalúa LLM-as-a-judge y métricas de calidad. Documentado en el [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §4.3]].
>   - 🐛 **El assignment C1M5 es el más defectuoso del curso.** Siete bugs documentados en el [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 §4.3]], cuatro de ellos **verificados por ejecución**. El más grave: `get_params_for_task` usa `if`/`if`/`else`, lo que hace la rama `technical` **inalcanzable** — devuelve `temperature=1` cuando declara `0.3`, dejando el Ejercicio 3 completo sin efecto. Es código dado, no del alumno, y su unittest ni se invoca ni lo detectaría.
>   - ⚠️ **Contradicción de fondo del módulo:** el M5 dedica tres lecciones a evaluar **calidad**, y su assignment optimiza **solo tokens** — publicando como éxito un caso donde la respuesta empeoró visiblemente (un "look de boda" que pasó de camisa+corbata a zapatos+corbata, sin prenda superior) sin comentarlo. Analizado en el [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 §4.4]].
>   - 📕 **Corrección de terminología con impacto práctico:** lo que el curso llama *"PDF RAG"* es **ColPali** (Faysse et al., ICLR 2025) — coincide mecanismo por mecanismo. *"PDF RAG"* **no existe como término en la literatura**, así que quien lo busque no encontrará nada. Documentado con la ficha verificada en el [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 §5.3]].

> [!warning] ⚠️ El Tomo 12 salió distinto de lo planificado — y conviene saber por qué (2026-07-28)
> El plan lo describía como *"técnicas avanzadas que son práctica estándar en producción"*. Al ser el primer tomo **sin fuente primaria**, se verificó toda la bibliografía **antes** de escribir — y la búsqueda de evaluaciones comparativas devolvió lo contrario de lo esperado:
> - **Query rewriting y query decomposition empeoran el retrieval** en consultas de un salto, a 3× y 5,7× la latencia (Wang et al., EMNLP 2024). Cuatro evaluaciones independientes coinciden.
> - **Multi-query / RAG-Fusion no tiene paper fundacional** (es folclore de framework, popularizado por un post de LangChain de oct-2023) **y** la evidencia que existe es negativa: Hit@10 de 0,51 → 0,48.
> - **GraphRAG cuesta ~350× más tokens** que RAG vainilla y pierde contra él en tareas simples. Su paper sigue siendo preprint tras dos años, mientras **todas sus alternativas están publicadas** (RAPTOR/ICLR, HippoRAG/NeurIPS, HippoRAG 2/ICML, LightRAG/Findings-EMNLP).
> - Lo que **sí** paga: reranking, hybrid search, y —lo más rentable de todo— **un clasificador que decida no recuperar**: solo el 27,8 % de las consultas reales necesitan aumentación (Hussain & Nielbo, 2026, sobre 20.000 consultas de producción).
>
> **El tomo se escribió como mapa de "qué paga y qué no", no como catálogo.** Es más útil así, y es lo que la evidencia sostiene.

> [!tip] ⭐ Tomos de complemento (vanguardia)
> Los tomos 12–14 cubren temas que el curso no aborda explícitamente pero que son práctica estándar en producción (query rewriting/decomposition, GraphRAG, frameworks de orquestación, Text2SQL y Table QA sobre datos estructurados). Se construyen con **bibliografía externa verificable** y se mantienen actualizados como parte del mandato de vanguardia. El número 14 —antes reservado— lo ocupa desde el 2026-08-28 el complemento de Structured Data RAG.
>
> **Corrección de alcance (2026-07-18):** el plan original asignaba *hybrid search con RRF* al Tomo 12 como complemento externo. Al indexar el material del Módulo 2 se constató que **el curso sí cubre hybrid search y RRF en detalle** (fórmula, hiperparámetros `k` y `beta`), de modo que pasó a ser contenido del **Tomo 04** con el curso como fuente primaria. El Tomo 12 queda acotado a lo que efectivamente es externo.

---

## 🧭 Rutas de lectura por perfil

> [!abstract] 👔 Ruta ejecutiva (el "para qué")
> [[Guia-Maestra-RAG_01-Introduccion-a-RAG|01 · Introducción]] → secciones 👔 de [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09 · Evaluación]] y [[Guia-Maestra-RAG_10-RAG-en-Produccion|10 · Producción]] → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|15 · Glosario]]. Entiende qué resuelve RAG, cómo se mide su valor, qué cuesta operarlo y el vocabulario mínimo.

> [!tip] 🧭 Ruta puente (el "cuándo y por qué")
> Toda la guía enfocándote en los bloques 🧭 y las tablas comparativas. Ideal para decidir arquitecturas y trade-offs (keyword vs. semantic vs. hybrid, cuándo rerankear, RAG vs. fine-tuning, cost vs. latency) sin implementar tú mismo.

> [!info] 🔧 Ruta técnica (el "cómo")
> Recorrido completo 01 → 14, con foco en los bloques 🔧 y el código. Base para construir sistemas RAG reales de punta a punta, incluyendo los complementos de vanguardia.

---

## 🔗 Documentos meta del proyecto

- [[Instrucciones|📖 Instrucciones]] — contrato de trabajo, reglas de idioma, bibliografía, flujo y mandato de vanguardia (Claude lo lee al inicio de cada sesión).
- [[Prompts-NotebookLM|🎙️ Prompts para NotebookLM]] — prompts listos para generar un podcast por tomo preservando la intención comunicativa de la guía (términos técnicos sin traducir, triple audiencia, analogía antes que definición).

---

> [!info] Cómo crece esta guía
> El flujo: el autor sube material del curso a la carpeta → Claude lo indexa y genera el tomo correspondiente → se actualiza este tracker. Además, Claude mantiene la guía a la vanguardia de forma proactiva, proponiendo actualizaciones cuando el estado del arte avanza. Ver detalle en [[Instrucciones|📖 Instrucciones]].
