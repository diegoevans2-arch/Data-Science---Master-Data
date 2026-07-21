---
title: "Tomo 19 — NLP y LLMs Aplicados"
tags: [data-science, machine-learning, nlp, llm, rag]
audiencias: [tecnico, puente, ejecutivo]
tomo: 19
version: 6.0
---

# 💬 Tomo 19 — NLP y LLMs Aplicados

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[18-Causalidad-y-Uplift|18 · Causalidad y Uplift]] · Siguiente: [[20-Sistemas-de-Recomendacion|20 · Sistemas de Recomendación ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> El 80% de la información de una organización vive en texto: reclamos, contratos, correos, notas clínicas, tickets. El [[12-Deep-Learning|Tomo 12]] explicó el Transformer por dentro; este tomo cubre la **práctica**: cómo se convierte texto en features, cuándo basta un TF-IDF con regresión logística, cuándo conviene un modelo pre-entrenado, y cómo usar LLMs en producción sin que inventen. La regla de oro del tomo: **la solución más nueva no es la solución por defecto**.

> [!abstract] 👔 Impacto ejecutivo
> El texto es el activo de datos más grande y menos explotado de la mayoría de las empresas — y desde los LLMs, el más sobre-prometido.
>
> - **Decisiones que habilita:** clasificar y rutear reclamos/tickets automáticamente, buscar por significado en el conocimiento interno, resumir y extraer datos de documentos, asistentes con respuestas citables.
> - **Costo de hacerlo mal:** pagar API de LLM para lo que resolvía una regresión logística, asistentes que alucinan políticas inexistentes, y fuga de datos sensibles hacia servicios externos.
> - **Pregunta ejecutiva que responde:** *¿qué problema de texto tengo, cuál es la herramienta MÍNIMA que lo resuelve, y cómo controlo lo que el sistema dice?*

> [!tip] 💡 Analogía general
> Tres generaciones de lectores para tu bodega de documentos: el **contador de palabras** (TF-IDF: no entiende nada, pero cuenta tan bien que clasifica sorprendentemente bien), el **lector con diccionario mental** (embeddings: sabe que "boleta" y "factura" se parecen sin que nadie se lo diga), y el **licenciado elocuente** (LLM: redacta, resume y conversa — y cuando no sabe, improvisa con total seguridad). El arte está en contratar al más barato que haga el trabajo, y en nunca dejar al licenciado sin sus apuntes (RAG).

---

## 1. El pipeline clásico: de texto a números

Audiencia: 🔧 🧭

**🔧 Definición técnica — la cadena de montaje:**

1. **Normalización:** lowercase, manejo de tildes/símbolos, según la tarea ([[03-Preparacion-de-Datos]]).
2. **Tokenización:** cortar el texto en unidades. Por palabra (clásico) o por **subpalabras** (BPE, WordPiece): "inexplicable" → "in + explica + ble" — vocabulario acotado, sin palabras fuera de vocabulario; es lo que usan todos los Transformers ([[12-Deep-Learning]]).
3. **Stopwords / lematización:** quitar palabras vacías y reducir a raíz ("corriendo" → "correr"). Útil para BoW/TF-IDF; **innecesario y contraproducente** con Transformers (el contexto es la señal).
4. **Bag of Words / TF-IDF:** cada documento → vector de conteos. TF-IDF pondera: `tf-idf(t,d) = tf(t,d) × log(N/df(t))` — un término pesa más si es frecuente en ESTE documento y raro en el resto. N-gramas (bigramas/trigramas) capturan "no recomiendo" ≠ "recomiendo".
5. **Modelo clásico encima:** Naive Bayes, regresión logística o SVM lineal ([[07-Modelos-Supervisados]]) sobre la matriz sparse (MaxAbsScaler/Normalizer si aplica, [[05-Escalado-de-Datos]]).

**🧭 Cuándo usarlo:** clasificación de textos con miles de ejemplos etiquetados y vocabulario discriminante (spam, ruteo de tickets, categorización de reclamos). Es el **baseline obligatorio**: barato, rápido, interpretable (los coeficientes muestran qué palabras deciden) — y sorprendentemente difícil de vencer.

**👔 En una frase para el negocio:** antes de pagar por un LLM, exige ver el baseline TF-IDF: la mitad de las veces resuelve el 90% del problema al 1% del costo.

## 2. Embeddings: el significado como geometría

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un mapa donde las palabras viven según su significado: "boleta" y "factura" son vecinas, "reclamo" queda cerca de "queja" y lejos de "felicitación". Y con **sentence embeddings**, frases enteras tienen dirección en el mapa: buscar deja de ser "contiene la palabra X" y pasa a ser "queda cerca de lo que quise decir".

**🔧 Definición técnica:** vectores densos donde la similitud semántica es cercanía (coseno — [[05-Escalado-de-Datos]]). Evolución: **Word2Vec** (Mikolov et al., 2013) y **GloVe** (Pennington et al., 2014) — estáticos: una palabra, un vector, sin contexto ("banco" financiero = "banco" de plaza); **contextuales** — BERT (Devlin et al., 2019): el vector depende de la oración; **sentence embeddings** — SBERT (Reimers & Gurevych, 2019) y sucesores: un vector por frase/párrafo, la pieza que habilita búsqueda semántica, clustering de textos ([[06-Clustering]]) y el retrieval del RAG (sección 5). Se almacenan en índices vectoriales (FAISS, bases vectoriales) para búsqueda de vecinos a escala.

**🧭 Cuándo usarlo:** búsqueda semántica, deduplicación de textos, features de texto para modelos tabulares, agrupamiento de reclamos sin etiquetas, y como retriever de un RAG.

**👔 En una frase para el negocio:** convierte "buscar por palabra exacta" en "buscar por lo que quise decir" — la mejora de productividad más subestimada sobre el conocimiento interno.

## 3. Tareas y métricas del NLP

Audiencia: 🔧 🧭

| Tarea | Qué hace | Métrica típica | Nota |
|---|---|---|---|
| Clasificación de texto | Asignar categoría (spam, sentimiento, ruteo) | F1 (macro con desbalance) ([[08-Metricas-de-Evaluacion]]) | El caballo de batalla; baseline TF-IDF primero |
| NER (Named Entity Recognition) | Extraer entidades (nombres, montos, fechas, RUT) | F1 a nivel de entidad/span | Extracción estructurada desde texto libre |
| Resumen | Comprimir conservando lo esencial | ROUGE (solapamiento con resúmenes de referencia) (Lin, 2004) | ROUGE mide solapamiento, no veracidad |
| Traducción | Texto entre idiomas | BLEU (precisión de n-gramas vs referencia) (Papineni et al., 2002) | El clásico; complementar con evaluación humana |
| Generación (modelado de lenguaje) | Predecir/producir texto | Perplexity (qué tan "sorprendido" queda el modelo) | Baja perplexity ≠ texto útil o veraz |
| Retrieval / búsqueda | Traer los documentos relevantes | Recall@k, MRR, NDCG ([[20-Sistemas-de-Recomendacion]]) | La calidad del RAG se decide aquí |

## 4. Topic Modeling: los temas que nadie etiquetó

Audiencia: 🔧 🧭

**🔧 Definición técnica:** **LDA** (Blei et al., 2003): modelo probabilístico generativo — cada documento es una mezcla de temas y cada tema una distribución de palabras (priors Dirichlet, prima de la Beta — [[02-Fundamentos-Matematicos]]); requiere elegir K temas e interpretarlos. Moderno: **BERTopic** (Grootendorst, 2022): embeddings → reducción UMAP → clustering HDBSCAN → etiquetado por TF-IDF de clase ([[03-Preparacion-de-Datos]], [[06-Clustering]]) — temas más coherentes en textos cortos y sin fijar K a priori.

**🧭 Cuándo usarlo:** explorar de qué hablan miles de reclamos/encuestas/notas ANTES de decidir categorías; insumo para diseñar el clasificador supervisado posterior.

**👔 En una frase para el negocio:** el mapa de "de qué se queja la gente" sin leer 50.000 tickets ni predefinir las categorías.

## 5. LLMs en producción: prompting, RAG y fine-tuning

Audiencia: 🔧 🧭 👔

**🔧 Prompting:** zero-shot (solo la instrucción), **few-shot** (ejemplos en el prompt — el hallazgo de GPT-3: Brown et al., 2020), **chain-of-thought** (pedir razonamiento paso a paso mejora tareas complejas — Wei et al., 2022). Reglas prácticas: instrucciones explícitas, formato de salida especificado (JSON con schema), ejemplos representativos de los casos difíciles, y temperatura baja para tareas deterministas.

**🔧 RAG (Retrieval-Augmented Generation)** (Lewis et al., 2020): en lugar de esperar que el LLM "sepa" tu negocio, se le entregan los documentos relevantes en el momento de la pregunta:

```
 Pregunta del usuario
        │
        ▼
 1. RETRIEVER: embedding de la pregunta → búsqueda en el índice vectorial
    de TUS documentos (políticas, contratos, manuales)          [sección 2]
        │
        ▼
 2. CONTEXTO: los k pasajes más relevantes se insertan en el prompt
        │
        ▼
 3. LLM: responde USANDO ese contexto y CITANDO las fuentes
        │
        ▼
 Respuesta verificable (con referencia al documento y párrafo)
```

**🔧 ¿Prompting, RAG o fine-tuning? La decisión clave:**

| Necesidad | Solución | Por qué |
|---|---|---|
| Conocimiento actualizado / interno / citable | **RAG** | El conocimiento vive en el índice (actualizable al instante), no en los pesos; respuestas trazables |
| Formato, tono o tarea muy específica y repetitiva | **Fine-tuning** (LoRA — [[12-Deep-Learning]]) | Enseña el *cómo*; no es buena vía para inyectar el *qué* (hechos) |
| Tarea general con pocos casos | **Prompting few-shot** | Sin entrenamiento, iteración en minutos |
| Clasificación masiva con miles de etiquetas disponibles | Modelo chico fine-tuneado o TF-IDF | El LLM por API sale caro por predicción; destilar o volver a la sección 1 |

**🔧 Evaluación de LLMs:** un **golden set** (preguntas con respuestas correctas curadas por expertos) evaluado en cada cambio de prompt/modelo; métricas de fidelidad al contexto (¿la respuesta está sostenida por los pasajes?) y de retrieval (recall@k); *LLM-as-judge* (otro LLM evaluando respuestas) es útil para escalar pero se audita contra jueces humanos — no es oráculo. Y monitoreo en producción como cualquier modelo: el drift también existe aquí ([[13-MLOps-XAI-Etica]]).

> [!danger] 🚨 Los dos riesgos que pagan titulares
> **(1) Alucinación:** un LLM sin grounding redacta políticas, precios y jurisprudencia inexistentes con total fluidez. Mitigación: RAG con citas obligatorias, instrucción explícita de responder "no está en los documentos" cuando aplique, y golden set que incluya preguntas SIN respuesta en el corpus. **(2) Privacidad:** texto con datos personales viajando a APIs externas puede violar normativa. Mitigación: anonimización/enmascaramiento previo, acuerdos de procesamiento de datos, o modelos desplegados en infraestructura propia ([[13-MLOps-XAI-Etica]]).

> [!example] 📊 Caso de negocio — Banca: reclamos ruteados y normativa consultable
> **Problema:** 30.000 reclamos mensuales ruteados a mano (lento, inconsistente) y ejecutivos que responden consultas normativas "de memoria".
>
> **Técnica aplicada:** dos sistemas, cada uno con su herramienta mínima: (1) **ruteo de reclamos** con baseline TF-IDF + regresión logística — F1 macro 0.88 con datos históricos etiquetados; un fine-tuning de BERT chico agrega 4 puntos y se justifica por volumen; (2) **asistente normativo RAG** sobre el corpus interno de circulares y políticas, con citas obligatorias a documento y párrafo, y golden set de 300 preguntas mantenido por cumplimiento.
>
> **Resultado:** el ruteo pasa de horas a segundos con trazabilidad total; el asistente responde con la circular citada al pie (y dice "no está normado" cuando no lo está — el caso que más confianza generó en cumplimiento). El LLM quedó **solo** donde agregaba valor: redacción con fuentes; la clasificación masiva quedó en el modelo barato.

## 6. Guía de decisión rápida

Audiencia: 🧭

```
 ¿Clasificar/rutear texto con miles de ejemplos etiquetados?
   └─► TF-IDF + LogReg/NB (baseline) ─► fine-tune de encoder chico si el volumen lo paga
 ¿Buscar por significado en documentos internos?
   └─► Sentence embeddings + índice vectorial
 ¿Responder preguntas sobre TU conocimiento, con citas?
   └─► RAG (retriever + LLM + citas obligatorias + golden set)
 ¿Extraer campos de documentos (montos, fechas, cláusulas)?
   └─► NER fine-tuneado, o LLM con salida JSON validada
 ¿Pocos ejemplos y tarea general (resumir, redactar)?
   └─► LLM con few-shot prompting
 ¿Tono/formato propio, tarea repetitiva de alto volumen?
   └─► Fine-tuning ligero (LoRA) de un modelo abierto
```

---

## 📖 Referencias de este tomo

- (Mikolov et al., 2013), (Pennington et al., 2014) — word embeddings. · (Devlin et al., 2019) — BERT. · (Reimers & Gurevych, 2019) — Sentence-BERT.
- (Blei et al., 2003) — LDA. · (Grootendorst, 2022) — BERTopic.
- (Papineni et al., 2002) — BLEU. · (Lin, 2004) — ROUGE.
- (Brown et al., 2020) — few-shot. · (Wei et al., 2022) — chain-of-thought. · (Lewis et al., 2020) — RAG.
- (Jurafsky & Martin, 2024) — *Speech and Language Processing*, la referencia académica abierta.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[18-Causalidad-y-Uplift|18 · Causalidad y Uplift]] · Siguiente: [[20-Sistemas-de-Recomendacion|20 · Sistemas de Recomendación ➡]]

> **Próximo tomo:** [[20-Sistemas-de-Recomendacion]] — qué mostrarle a cada quién: collaborative filtering, factorización, métricas de ranking y las trampas del cold start y los feedback loops.
