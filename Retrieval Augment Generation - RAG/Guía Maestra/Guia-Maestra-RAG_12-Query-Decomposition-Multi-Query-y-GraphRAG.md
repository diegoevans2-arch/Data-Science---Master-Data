---
title: "Tomo 12 — ⭐ Técnicas avanzadas de query: decomposition, multi-query y GraphRAG"
tags: [rag, complemento, vanguardia, query-decomposition, multi-query, graphrag, raptor, self-rag, ircot, step-back, hyde]
audiencias: [tecnico, puente, ejecutivo]
tomo: 12
version: 1.1
updated: 2026-08-28
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 12 — ⭐ Técnicas avanzadas de query: decomposition, multi-query y GraphRAG

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 · Quantization, trade-offs y multimodal RAG]] · Siguiente → [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13 · ⭐ Complemento: frameworks]]

---

> [!danger] 🚨 Léeme antes que nada: este tomo es distinto a los once anteriores
> **Los Tomos 01–11 tienen una fuente primaria** — el curso de DeepLearning.AI, con su material, sus notebooks y su código ejecutable. Cuando algo estaba mal, el material lo delataba.
>
> **Este tomo no la tiene.** Se construye enteramente con bibliografía externa. Eso cambia dos cosas:
> 1. **Cada afirmación va etiquetada** con su nivel de respaldo: consenso establecido, práctica común, o criterio.
> 2. **Todas las referencias se verificaron antes de escribir**, no después — con evidencia textual y URL. Las que no se pudieron verificar están marcadas como tales, y las cifras que circulan sin fuente localizable **se descartaron explícitamente** (§10.3).
>
> Y hay algo más incómodo, que es la razón de ser de este tomo:

> [!warning] ⚠️ La evidencia dice que buena parte de estas técnicas **no compensan**
> Este iba a ser un catálogo de "técnicas avanzadas que deberías conocer". Al buscar evaluaciones comparativas serias apareció otra cosa: **varias de las técnicas más populares empeoran el retrieval y multiplican la latencia por 3 a 6.**
>
> No las vamos a ocultar ni a vender. Este tomo es un **mapa de cuándo pagan y cuándo son humo caro** — que es más útil que un catálogo, y bastante menos común.

> [!info] ¿Por qué importa esta sección?
> Todo lo de este tomo cuesta **llamadas extra al LLM**: exactamente lo que el [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11]] te enseñó a controlar. Cada técnica de aquí es una apuesta: pagas latencia y tokens a cambio de una mejora que **puede no existir**.
>
> Saber cuáles pagan —y en qué condiciones exactas— es la diferencia entre un sistema optimizado y uno caro.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** rechazar con evidencia una propuesta de "vamos a añadir GraphRAG"; distinguir la técnica que resuelve tu problema real de la que solo suena avanzada; saber cuándo la respuesta correcta es **arreglar lo básico** en vez de añadir una capa.
> - **Costo o riesgo de hacerlo mal:** adoptar por moda una técnica que multiplica el costo por 350 y **pierde calidad** en tus consultas reales. Es un riesgo documentado, no hipotético.
> - **Pregunta que responde:** *"todo el mundo habla de esto — ¿lo necesitamos nosotros?"*

---

## 1. 🕰️ El problema tiene 55 años

Audiencia: 🔧 🧭

**Antes de hablar de LLMs**, conviene situar el problema, porque no es nuevo y eso cambia cómo se juzgan las soluciones.

> [!tip] 💡 Analogía
> Vas a una ferretería y pides "algo para que no se mueva la puerta". El dependiente no busca eso literalmente: **traduce** tu problema a *tope de puerta*, *bisagra*, o *cuña*, y te trae varias opciones. Tu formulación no era la mejor consulta posible — y él lo sabía.
>
> Todo este tomo trata de una sola idea: **la pregunta del usuario casi nunca es la mejor consulta para el buscador.** Lo que cambia es cuánto estás dispuesto a pagar por traducirla.

**🔧 El origen formal: Rocchio (1971).** El algoritmo de *relevance feedback* reformula el vector de consulta desplazándolo hacia el centroide de los documentos relevantes y alejándolo de los no relevantes.

**🧭 Por qué importa la fecha:** las técnicas de este tomo se presentan a menudo como invención de 2023. No lo son. Son la enésima iteración de un problema que la disciplina de information retrieval lleva **medio siglo** atacando. Esa perspectiva es el mejor antídoto contra el entusiasmo: **si algo llevara 55 años sin resolverse del todo, es improbable que un prompt lo cierre.**

> [!note] Estado: consenso absoluto (canon histórico)
> Rocchio es una referencia canónica del campo, recogida en *Introduction to Information Retrieval* de Manning, Raghavan & Schütze — el manual estándar. Ver la nota de verificación en §10.2: es un capítulo de libro de 1971 sin edición digital pública, verificado vía fuente secundaria autorizada.

---

## 2. ✍️ Query rewriting: reescribir la pregunta

Audiencia: 🔧 🧭

**🔧 Definición técnica:** un LLM reescribe la consulta del usuario para hacerla más "buscable" antes de enviarla al retriever. La referencia moderna es **Ma et al. (2023)**, el esquema *Rewrite-Retrieve-Read*.

> [!note] Esto ya lo viste
> El [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]] cubre query rewriting **como contenido del curso**. Aquí no se repite: lo que aporta este tomo es la **evidencia comparativa** de cuánto rinde, que el curso no da.

### 2.1 🚨 Y la evidencia comparativa es mala

**Wang et al. (EMNLP 2024)** midieron calidad *y* latencia sobre TREC DL19, contra un baseline denso simple:

| Método | mAP | nDCG@10 | Latencia |
|---|---|---|---|
| **Baseline** (LLM-Embedder, sin transformación) | **44,66** | **70,20** | **2,61 s** |
| + Query rewriting | 44,56 ↓ | 67,89 ↓ | 7,80 s (**3,0×**) |
| + Query decomposition | 41,93 ↓↓ | 66,10 ↓↓ | 14,98 s (**5,7×**) |
| + HyDE | 50,87 ↑ | 75,44 ↑ | 7,21 s |
| + Hybrid search | 47,14 ↑ | 72,50 ↑ | 3,20 s |

Conclusión textual de los autores: *"query rewriting and query decomposition did not enhance retrieval performance as effectively."*

**No es un resultado aislado.** Tres evaluaciones independientes coinciden:

- **RAGLAB** (EMNLP 2024, Demos): *"Naive RAG, RRR, Iter-RETGEN, and Active RAG demonstrate comparable performance across 10 datasets"* — es decir, query rewriting **empatado con RAG naive** en diez benchmarks.
- **Alushi et al.** (EACL SRW 2026): *"Performance of Query Rewriting is highly dataset-dependent and falls substantially below No RAG on the INSCIT, QReCC, and TopiOCQA datasets"* — peor que **no recuperar nada** en 3 de 8 datasets.
- **BERGEN** (Findings EMNLP 2024) apunta a dónde sí está el valor: *"Reranking has been often overlooked and should be used to have strong baselines for future research."*

Dos trabajos más lo confirman desde ángulos distintos:

- **Kotte (2026)**, *Not All Queries Need Rewriting*: la reescritura *"degrades nDCG@10 by 9,0 percent on FiQA, improves it by 5,1 percent on TREC-COVID"* y no tiene efecto en SciFact. Pero el remate es lo importante: la reescritura **selectiva** *"does not reliably outperform never rewriting, with **even oracle selection offering only modest gains**"*. Es decir: ni siquiera sabiendo de antemano cuándo reescribir se gana gran cosa.
- **Bigdeli et al. (2026)**, el estudio de reproducibilidad más riguroso que encontré (10 métodos de reformulación, 2 familias de LLM × 2 escalas × 3 paradigmas de retrieval × 9 datasets): *"improvements observed under lexical retrieval **do not consistently transfer to neural retrievers**"* y *"larger LLMs do not uniformly yield better downstream performance"*. Buena parte de la ganancia reportada en la literatura es **un artefacto del retriever contra el que se midió**.

> [!danger] 🚨 La lectura práctica
> Si tu sistema no tiene reranking y estás considerando query rewriting, **el orden está invertido**. Un cross-encoder de una sola pasada ([[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]]) tiene mejor evidencia y menor costo que reescribir la query con un LLM.
>
> Y hay un dato de reparto de tiempo que lo vuelve tangible. Ferrazzi et al. (ACL 2026, Industry Track) desglosan la latencia de un pipeline RAG "enhanced": la generación de la respuesta se lleva un 45–50 %… y **la reescritura de la query se lleva una proporción similar**, frente a un 0–5 % del retrieval y un 0–2 % del reranking.
>
> **Reescribir la query consume aproximadamente la mitad del reloj de tu sistema** — para una mejora que la evidencia no respalda.

---

## 3. 🪓 Query decomposition: partir la pregunta

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> *"¿Cuánto mide el edificio más alto de la ciudad donde nació el actor que ganó el Oscar en 2020?"* Ningún documento contiene esa respuesta. Pero **tres documentos sí la contienen entre los tres**: quién ganó el Oscar, dónde nació, y qué edificio es el más alto ahí.
>
> Descomponer es reconocer que algunas preguntas **no tienen un documento**, tienen una cadena.

**🔧 Definición técnica:** dividir una consulta compleja en sub-preguntas más simples, resolverlas en secuencia, y componer la respuesta final.

### 3.1 Cuál es la referencia canónica (y cuál no)

Hay una confusión frecuente que conviene deshacer:

| Paper | Venue | ¿Sirve como cita para RAG? |
|---|---|---|
| **Self-Ask** — Press et al. | Findings EMNLP 2023 | ✅ **Sí, es la canónica.** Integra explícitamente un buscador |
| **Least-to-Most** — Zhou et al. | ICLR 2023 | ⚠️ **No.** Es descomposición para *razonar*; **no contiene retrieval**. Sus benchmarks son manipulación simbólica y matemáticas |
| **DecomP** — Khot et al. | ICLR 2023 | ✅ Sí, complementaria: incorpora *"a symbolic information retrieval within our decomposition framework"* |

**Self-Ask** funciona así: *"the model explicitly asks itself (and answers) follow-up questions before answering the initial question"*, y esa estructura *"lets us easily plug in a search engine to answer the follow-up questions"*.

> [!important] 🎯 El hallazgo de Self-Ask que justifica toda la técnica
> El paper mide el ***compositionality gap***: el modelo sabe los hechos sueltos pero no los compone. Y descubre algo que no se arregla esperando:
>
> *"as model size increases we show that the single-hop question answering performance improves faster than the multi-hop performance does, therefore **the compositionality gap does not decrease**"*.
>
> Traducido: **escalar el modelo no cierra esta brecha.** Ese es el argumento fuerte —y honesto— a favor de descomponer: no es una muleta temporal a la espera de modelos mejores.

### 3.2 Cuándo paga, con números

La descomposición es el caso donde la evidencia se parte en dos, y la distinción es **la complejidad real de tus preguntas**:

| Escenario | Evidencia |
|---|---|
| **Consultas de un salto** (single-hop) | ❌ Wang et al.: 41,93 mAP vs 44,66 del baseline, a 5,7× la latencia. **Pierde** |
| **Consultas multi-hop genuinas** | ✅ Ammann et al. (ACL SRW 2025): MRR@10 de 0,464 → **0,635 (+36,7 %)**, y HotpotQA F1 31,3 → 35,0 |

Y el precio, del mismo trabajo: *"The overhead of question decomposition (qd) is **16,7 s/query**. This is primarily due to the additional LLM inference required to generate subqueries."*

> [!warning] ⚠️ Precisión sobre esa cifra
> Los 16,7 s son **latencia del pipeline de retrieval**, no end-to-end: el propio paper aclara que la tabla *"reports end-to-end retrieval latency (excluding generation)"*. El sobrecoste absoluto es real; **no** conviene convertirlo en un ratio dramático de sistema completo.

**🧭 Cuándo usarlo:** solo si has comprobado —midiendo, no suponiendo— que **una fracción relevante de tu tráfico real es multi-hop**. El [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §5]] explica cómo saberlo: agrupando las consultas reales por tipo.

**👔 En una frase para el negocio:** descomponer preguntas multiplica el costo por consulta, y solo lo recupera si tus usuarios hacen preguntas que encadenan varios hechos.

---

## 4. 🔀 Multi-query: la técnica que no tiene paper

Audiencia: 🔧 🧭

**🔧 Qué es:** generar varias reformulaciones de la misma pregunta, recuperar con todas, y fusionar los resultados en un ranking único.

> [!danger] 🚨 Hallazgo: no existe un paper fundacional en la literatura de LLMs
> Esta es una de las técnicas más citadas en tutoriales de RAG. **No tiene paper.** La búsqueda bibliográfica lo confirma, y hay una evidencia elegante de ello.
>
> El post donde se populariza es de **LangChain** (*Query Transformations*, 24 de octubre de 2023). Lista cinco técnicas, y su patrón de citación se delata solo:
>
> | Técnica del post | ¿Cita paper? |
> |---|---|
> | Rewrite-Retrieve-Read | ✅ Sí → arXiv 2305.14283 |
> | Step-Back Prompting | ✅ Sí → arXiv 2310.06117 |
> | Follow Up Questions | ❌ No |
> | **Multi Query Retrieval** | ❌ **No** |
> | **RAG-Fusion** | ❌ **No** (enlaza a un post de blog) |
>
> **Los propios autores citan paper cuando existe.** Para estas dos no citan nada — porque no hay nada que citar.

### 4.1 ¿Y el "paper de RAG-Fusion"?

Existe un artículo (Rackauckas, 2024) que suele ofrecerse como respaldo. **No califica como fundacional**, por tres razones:

1. **Es descriptivo, no constitutivo.** Su propio abstract dice que evalúa *"the newly popularized RAG-Fusion method"* — documenta algo que ya circulaba.
2. **Venue de bajo perfil** (IJNLC), no un venue reconocido de IR/NLP.
3. **Metodología débil:** un case study de una sola empresa con **evaluación manual**, sin benchmark estándar ni comparación controlada.

### 4.2 Pero el mecanismo sí tiene abolengo — de 1995

Y aquí está la parte interesante. Combinar múltiples representaciones de una consulta y fusionar sus rankings **está establecido en information retrieval desde hace tres décadas**:

- **Belkin, Kantor, Fox & Shaw (1995)**, *Combining the evidence of multiple query representations for information retrieval*. Encontraron que combinar consultas progresivamente daba rendimiento progresivamente mejor, significativamente superior a cualquier consulta individual — y que fusionar listas rankeadas superaba a cualquier lista suelta. **Eso es multi-query retrieval, publicado en 1995.**
- **Cormack, Clarke & Büttcher (2009)** aportan el **RRF** que RAG-Fusion usa como paso de fusión — el mismo que documenta el [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]].

> [!important] 🎯 Cómo citarlo honestamente
> No hace falta fingir una referencia. La formulación defendible —y más interesante— es:
>
> *"Multi-query retrieval no tiene paper fundacional en la literatura de LLMs; es una práctica popularizada por implementaciones de framework (LangChain, octubre de 2023). Su mecanismo subyacente —combinar múltiples representaciones de la consulta y fusionar los rankings— está establecido en IR desde Belkin et al. (1995), y la fusión por RRF desde Cormack et al. (2009)."*

**Estado: práctica común de industria, sin respaldo fundacional propio.** Señal reveladora de que el campo conoce su costo: DMQR-RAG (2024) propone *"an adaptive strategy selection method that **minimizes the number of rewrites** while optimizing overall performance"* — o sea, el número de reescrituras se trata como un costo a reducir.

### 4.3 🚨 Y cuando por fin se midió, el resultado fue negativo

No tener paper fundacional no significa no tener evidencia. La que hay es desfavorable:

- **Medrano et al. (2026)**, evaluando RAG-Fusion en producción: *"fusion variants **fail to outperform single-query baselines** on KB-level Top-k accuracy, with **Hit@10 decreasing from 0.51 to 0.48**"*. Y el diagnóstico de por qué, que es la parte instructiva: *"retrieval fusion does increase raw recall, but these gains are **largely neutralized after re-ranking and truncation**."*
- **Akarsu et al. (2026)** sobre documentos con tablas: *"Multi-query retrieval with RAG-Fusion provides **negligible improvement over BM25** (Recall@5: 0,640 vs. 0,644)"* — o sea, empatado con una técnica de 1994.
- **ARAGOG** (Eibich et al., 2024): *"Multi-query approaches **underperformed**"*.

> [!important] 🎯 Por qué la fusión no rescata nada
> El hallazgo de Medrano et al. explica el mecanismo del fracaso: multi-query **sí aumenta el recall bruto** —traes más documentos relevantes— pero esa ganancia **se neutraliza cuando el reranker reordena y el `top_k` corta**. Los documentos extra que encontraste no llegan al contexto.
>
> Dicho de otro modo: **si ya tienes reranking, multi-query te está haciendo pagar N llamadas al LLM para recuperar documentos que después vas a descartar.**

---

## 5. 🔭 Step-back prompting: subir un nivel de abstracción

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un cliente pregunta: *"¿por qué mi pedido 48812-B sigue en 'preparación' desde el martes?"* No hay documento sobre ese pedido. Pero si primero te preguntas *"¿cuáles son los estados del proceso de preparación y qué los bloquea?"*, **esa** pregunta sí tiene documentación — y contiene la respuesta.

**🔧 Definición técnica (Zheng et al., ICLR 2024):** derivar una pregunta más general —de principio— a partir de la específica, y usarla para recuperar el contexto que ancla la respuesta. El paper la define como *"a derived question from the original question at a higher level"*.

**🔧 ¿Aplica a RAG?** Sí, y el paper es explícito: *"The step-back question is used to retrieve relevant facts, which work as additional context"*.

> [!warning] ⚠️ Un matiz que la divulgación se salta
> Las ganancias más citadas del abstract (MMLU +7 %/+11 %) son precisamente las tareas donde **no se usa retrieval** — el propio paper dice: *"We do not use RAG for STEM tasks, because of the inherent reasoning nature"*.
>
> Las ganancias **con** RAG son otras: TimeQA +27 %, MuSiQue +7 %. Siguen siendo buenas, pero no son las que suelen citarse.

**Estado: práctica común.** Peer-reviewed en venue de primer nivel, pero la evidencia empírica proviene de un solo trabajo — el de sus propios autores. No encontré evaluación independiente de esta técnica (§10.3).

---

## 6. 🔁 Los patrones adaptativos: decidir sobre la marcha

Audiencia: 🔧

Las técnicas anteriores transforman la query **una vez, antes de buscar**. Esta familia hace algo distinto: **decide durante la ejecución** si hay que recuperar, si lo recuperado sirve, y si hace falta otra ronda.

### 6.1 IRCoT — la mejor sostenida de todo el tomo

**🔧 Definición (Trivedi et al., ACL 2023):** intercalar retrieval con los pasos del chain-of-thought. El razonamiento guía la siguiente búsqueda, y lo recuperado mejora el siguiente paso de razonamiento.

El diagnóstico del paper es la clave: *"what to retrieve depends on what has already been derived, which in turn may depend on what was previously retrieved"*. Un retrieval de un solo paso **no puede recuperar lo que solo se sabe que hace falta después de razonar un tramo**.

> [!note] Estado: consenso establecido — el más sólido de este tomo
> ACL long paper, usado sistemáticamente como baseline en la literatura posterior. Y en la evaluación independiente de **FlashRAG** es la técnica de bucle que **claramente se gana sus llamadas** en multi-hop: HotpotQA F1 41,5 y PopQA 45,6, por encima del RAG estándar (35,3 y 36,7).

### 6.2 Self-RAG — el prestigio más alto y la reproducción más desfavorable

**🔧 Definición (Asai et al., ICLR 2024 *Oral*):** un modelo que *"adaptively retrieves passages on-demand, and generates and reflects on retrieved passages and its own generations using special tokens, called reflection tokens"*.

**📄 Paper:** Asai, S., Wu, Z., Wang, Y., Sil, A. & Hajishirzi, H. (2023). *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection.* arXiv:2310.11511. Aceptado como **Oral** en ICLR 2024 — el nivel de prestigio más alto posible en ese venue.

#### 6.2.1 El mecanismo de reflection tokens — en detalle

La innovación central de Self-RAG no es *cuándo* recuperar, sino que **el propio modelo aprende a criticarse** mediante tokens especiales inyectados durante el fine-tuning. Hay cuatro tipos:

| Token | Pregunta que responde | Valores posibles | Cuándo se emite |
|---|---|---|---|
| **[Retrieve]** | *"¿Necesito buscar información para responder esto?"* | `yes` / `no` / `continue` | **Antes** de generar (o entre segmentos) |
| **[IsRel]** | *"¿Este pasaje recuperado es relevante para la query?"* | `relevant` / `irrelevant` | **Después** de recuperar, por cada documento |
| **[IsSup]** | *"¿Mi generación está respaldada por el pasaje?"* | `fully supported` / `partially supported` / `no support` | **Después** de generar un segmento |
| **[IsUse]** | *"¿Mi respuesta final es útil para el usuario?"* | Escala de 1 a 5 | Al **final** de la respuesta completa |

> [!important] 🎯 Esto no es un prompt — es vocabulario aprendido
> Los reflection tokens se añaden al vocabulario del modelo y se entrenan con supervisión (un modelo "critic" genera las etiquetas de entrenamiento). El generador aprende **cuándo emitirlos y con qué valor**. No hay un segundo modelo en inference — es un solo forward pass que intercala tokens de control con tokens de texto.

#### 6.2.2 El flujo completo

```
   QUERY del usuario
      │
      ▼
   El modelo genera el primer segmento y decide:
   ┌─────────────────────────────────────────────┐
   │  [Retrieve] = yes?                          │
   │     SÍ → invoca retriever → obtiene docs    │
   │     NO → genera sin contexto externo        │
   └─────────────────────────────────────────────┘
      │ (si recuperó)
      ▼
   Por cada documento recuperado:
   ┌─────────────────────────────────────────────┐
   │  [IsRel] = relevant?                        │
   │     SÍ → el doc entra al contexto           │
   │     NO → se descarta                        │
   └─────────────────────────────────────────────┘
      │
      ▼
   Genera respuesta (o segmento de respuesta) condicionada al contexto
      │
      ▼
   ┌─────────────────────────────────────────────┐
   │  [IsSup] = fully supported?                 │
   │     SÍ → acepta el segmento                 │
   │     NO → puede re-retriever o regenerar     │
   └─────────────────────────────────────────────┘
      │
      ▼
   [IsUse] evalúa la respuesta completa → score de utilidad
```

**El punto clave:** este ciclo puede ejecutarse **múltiples veces por respuesta**. El modelo puede decidir [Retrieve] = `yes` en mitad de una generación larga, buscar más contexto, evaluar si lo nuevo es relevante, y continuar. Eso lo diferencia de un RAG con un solo paso de retrieval.

#### 6.2.3 El costo de adopción — por qué casi nadie lo usa

> [!danger] 🚨 Dos cosas que hay que decir sobre Self-RAG
> **① No es una técnica de prompting: es un modelo entrenado.** Requiere fine-tuning del generador con los *reflection tokens*. Presentarlo junto a step-back o multi-query como si fueran intercambiables es engañoso — **el costo de adopción es de otro orden de magnitud.**
>
> **② En evaluación unificada, queda por debajo del RAG estándar.** FlashRAG reporta TriviaQA 38,2 vs 58,8 del RAG estándar, y HotpotQA F1 29,6 vs 35,3.
>
> **Matiz obligatorio, para no sobrevender el hallazgo:** FlashRAG marca Self-RAG con asterisco = *"the use of a trained generator"*, o sea que corre su propio modelo y no el de la comparación. **Parte de la brecha es desajuste experimental, no fallo del método.** Los propios autores advierten que *"this setting may differ from the original setting of the method"*. No es "Self-RAG está roto"; es "en un entorno unificado no reproduce su ventaja".

El problema de reproducibilidad tiene raíces estructurales:

1. **Dependencia del critic model.** La calidad de los reflection tokens depende de la calidad del modelo que genera las etiquetas de entrenamiento. Si entrenas con GPT-4 como critic, los tokens son mejores — pero el costo de generar los datos de entrenamiento es alto.
2. **El modelo base importa.** Self-RAG se demostró originalmente sobre Llama 2 7B/13B. Las ganancias reportadas son **relativas a ese baseline**. Modelos más capaces ya incorporan parte de la capacidad de autocrítica, reduciendo el delta.
3. **No hay un checkpoint universal.** Cada dominio necesita su propio fine-tuning, lo que multiplica el costo de adopción por cada caso de uso.

#### 6.2.4 Cuándo tiene sentido Self-RAG

| Condición | ¿Aplica? |
|---|---|
| La **calidad factual** es más importante que la latencia (e.g., médico, legal) | ✅ Candidato fuerte |
| Tienes presupuesto para **fine-tunear** el modelo generador con reflection tokens | ✅ Requisito obligatorio |
| Tu modelo base es pequeño (7B–13B) y necesitas que se "comporte mejor de lo que es" | ✅ Donde más delta se observa |
| Necesitas una solución **drop-in** sin reentrenar nada | ❌ Self-RAG no es eso |
| Tu modelo base ya es frontier (GPT-4, Claude 3.5) y rara vez alucina | ⚠️ El delta será mínimo |
| La latencia importa más que la precisión factual | ❌ Cada reflection token es cómputo extra |

**Estado: alto prestigio académico, adopción práctica limitada.** Esa tensión es el dato.

### 6.3 CRAG — evaluar lo recuperado y corregir

**📄 Paper:** Yan, S., Gu, J., Zhu, Y. & Ling, Z. (2024). *Corrective Retrieval Augmented Generation.* arXiv:2401.15884.

**🔧 Definición (Yan et al., 2024):** *"a lightweight retrieval evaluator is designed to assess the overall quality of retrieved documents for a query, returning a confidence degree based on which different knowledge retrieval actions can be triggered"*. Si el retrieval falla, se recurre a búsqueda web.

**🔧 El problema que ataca** es real y poco tratado: *"it relies heavily on the relevance of retrieved documents, raising concerns about how the model behaves if retrieval goes wrong."*

#### 6.3.1 El pipeline detallado — tres acciones posibles

El flujo de CRAG tiene una estructura de **triage** que decide qué hacer con los documentos recuperados:

```
   QUERY → Retriever → documentos candidatos
                              │
                              ▼
                   ┌─────────────────────┐
                   │  RETRIEVAL EVALUATOR │
                   │  (modelo ligero que  │
                   │   puntúa relevancia) │
                   └─────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         CORRECT         AMBIGUOUS       INCORRECT
     (confianza alta)  (confianza media) (confianza baja)
              │               │               │
              ▼               ▼               ▼
     Usar docs tal cual   Refinar docs    Web search
     como contexto        + re-retrieve   como fallback
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                   Knowledge Refinement
                   (descomponer en strips)
                              │
                              ▼
                        GENERACIÓN
```

**Las tres acciones en detalle:**

| Acción | Condición | Qué hace |
|---|---|---|
| **① Correct** | El evaluador da confianza **alta** — los docs recuperados son relevantes | Se pasa directamente a generación con los documentos como contexto. Opcionalmente se aplica *knowledge refinement* para filtrar ruido |
| **② Incorrect** | El evaluador da confianza **baja** — los docs son irrelevantes | Se **descarta** lo recuperado de la KB interna y se dispara una búsqueda web como fallback. Los resultados web pasan por *knowledge refinement* antes de alimentar al generador |
| **③ Ambiguous** | Confianza **intermedia** — mix de docs relevantes e irrelevantes | Se combinan ambas fuentes: se filtran los docs internos relevantes via *knowledge refinement*, se complementa con búsqueda web si hace falta, y se genera con el subset resultante |

#### 6.3.2 Knowledge Refinement — el paso que distingue a CRAG

Este es el mecanismo más distintivo del paper y el menos citado en tutoriales:

1. **Descomposición en knowledge strips.** Los documentos largos se segmentan en unidades mínimas de información (*strips*) — oraciones o proposiciones atómicas.
2. **Evaluación por strip.** Cada strip se evalúa independientemente: ¿es relevante para la query? Las irrelevantes se descartan.
3. **Recombinación.** Solo las strips que pasan el filtro se recomponen como contexto para el generador.

> [!tip] 💡 Por qué importa
> El retriever opera a nivel de **chunk** (típicamente 200–500 tokens). Un chunk puede contener 3 hechos relevantes y 7 irrelevantes. Sin refinamiento, esos 7 hechos irrelevantes **contaminan** el contexto y pueden provocar alucinaciones. CRAG ataca exactamente esa granularidad.
>
> Es la misma lógica que el *groundedness check* del [[Guia-Maestra-RAG_09-Guardrails-y-Seguridad|Tomo 09 §4.4]] — pero aplicada **antes** de generar, no después. CRAG evalúa la relevancia del contexto *preventivamente*; el T09 evalúa si la respuesta *ya generada* está anclada. Son complementarios: CRAG reduce la probabilidad de que el generador alucine; el groundedness check detecta las que se le escapan.

#### 6.3.3 El evaluador de retrieval — fortalezas y debilidades

El evaluador de CRAG es un modelo ligero (T5-large en la implementación original) entrenado para clasificar la relevancia doc-query. Ventajas y limitaciones:

**Ventajas:**
- Rápido — mucho más barato que usar el LLM generador para evaluar
- Se puede entrenar con datos de relevancia estándar (e.g., MS MARCO)
- Desacopla la decisión de "¿sirve esto?" de la generación

**Limitaciones (de la reproducción independiente):**
- El evaluador *"primarily relies on named entity alignment rather than semantic similarity"* — es decir, **es frágil** y se puede engañar con documentos que comparten entidades sin ser relevantes
- Documentos que mencionan las mismas entidades en un contexto diferente pasan como "relevantes" cuando no lo son

#### 6.3.4 Cuándo tiene sentido CRAG

| Condición | ¿Aplica? |
|---|---|
| Tu knowledge base **no cubre todo** — hay preguntas legítimas fuera de cobertura | ✅ Caso ideal: el fallback a web resuelve lagunas |
| Toleras **latencia extra** (evaluador + posible web search + refinement) | ✅ Requisito: CRAG añade 1–2 pasos al pipeline |
| Necesitas **alta fiabilidad factual** y prefieres buscar en web antes que alucinar | ✅ Mejor que generar con contexto irrelevante |
| Tu sistema es **solo intranet** sin acceso a web | ⚠️ Pierdes la acción "Incorrect" — solo queda refinar o abstener |
| La latencia es crítica (< 2s por respuesta) | ❌ Los pasos extra de evaluación y refinement cuestan tiempo |
| Tu KB es exhaustiva y rara vez falla el retriever | ⚠️ El evaluador añade costo sin beneficio si casi siempre puntúa "Correct" |

> [!note] Estado: preprint, con una reproducción independiente favorable pero débil
> CRAG **no ha pasado revisión por pares** — DBLP lo cataloga solo como CoRR. Existe una reproducción independiente (2026) que concluye que *"our open-source pipeline achieves comparable performance to the original system"* ✅, pero es un **preprint de autor único**: es señal, no prueba.
>
> Su aporte más útil es una crítica al propio método: el evaluador *"primarily relies on named entity alignment rather than semantic similarity"* — es decir, **es frágil**, y se puede engañar con documentos que comparten entidades sin ser relevantes.
>
> **Valor práctico:** la *arquitectura* de CRAG — evaluar antes de generar, con acciones diferenciadas — es más adoptable que la implementación específica. Puedes implementar la misma lógica de triage (correct/ambiguous/incorrect) con un reranker como evaluador y sin el web search, adaptándola a tu contexto.

---

## 7. 🕸️ GraphRAG y la recuperación estructurada

Audiencia: 🔧 🧭 👔

Esta es la sección donde la distancia entre la reputación de una técnica y su evidencia es mayor. La vamos a contar con el **costo por delante**, no escondido al final.

### 7.1 El problema que sí resuelve, y que es real

> [!tip] 💡 Analogía
> Pregúntale a un buscador *"¿cuál es el tema principal de esta biblioteca?"*. No hay ningún libro que responda eso — la respuesta **emerge de todos**, y ningún fragmento la contiene. Buscar por similitud es inútil: no hay nada parecido a la pregunta.
>
> Eso es el **global sensemaking**, y es el hueco genuino que RAG vectorial no cubre.

**🔧 El planteamiento (Edge et al., 2024):** *"RAG fails on global questions directed at an entire text corpus, such as 'What are the main themes in the dataset?', since this is inherently a query-focused summarization (QFS) task, rather than an explicit retrieval task."* Y por qué no basta con resumir todo: *"Prior QFS methods, meanwhile, do not scale to the quantities of text indexed by typical RAG systems."*

**🔧 El pipeline**, que es el mejor marco conceptual para entender el problema:

```
   DOCUMENTOS
      │  ① un LLM extrae ENTIDADES y RELACIONES de cada chunk
      ▼      (+ "gleanings": se le repregunta si olvidó alguna
   GRAFO       → cada gleaning es otra llamada al LLM)
      │  ② detección jerárquica de COMUNIDADES (algoritmo Leiden,
      ▼      recursivo hasta comunidades hoja)
   COMUNIDADES
      │  ③ se pregenera un RESUMEN por comunidad  ← aquí está el costo
      ▼
   RESÚMENES
      │  ④ query time: cada resumen produce una respuesta parcial…
      ▼  ⑤ …y todas se resumen en la respuesta final (map-reduce)
   RESPUESTA
```

### 7.2 🔴 El costo — y el silencio del paper

> [!danger] 🚨 El paper que propone la técnica NO reporta su costo
> Edge et al. documenta exhaustivamente los tokens de **query time**, pero **no da ni un token ni un dólar del indexado**. Lo único que ofrece es tiempo de reloj: *"Graph indexing with a 600 token window took **281 minutes** for the Podcast dataset"* (un corpus de ~1M tokens).
>
> **Ese silencio es en sí mismo un dato citable:** el talón de Aquiles de la técnica no está cuantificado en el trabajo que la propone.

Lo cuantificaron otros — incluida **Microsoft misma**:

| Fuente | Dato |
|---|---|
| **Microsoft**, al lanzar *LazyGraphRAG* (7 meses después) | El indexado de LazyGraphRAG cuesta *"**0,1 % of the costs of full GraphRAG**"* → implica **~1.000×**. Y describe los costos del original como *"**prohibitive** for some users and use cases"* |
| **GraphRAG-Bench**, evaluación independiente | MS-GraphRAG global: **~331.000 tokens/consulta** frente a **~879** de RAG vainilla → **~350×** |
| **Han et al.** | Construcción del índice: **5.560 s vs 135 s** de RAG → **41×** |
| **LightRAG** | Actualización incremental: **~14M tokens** para regenerar las comunidades. **No hay forma barata de añadir documentos** |

> [!warning] ⚠️ Y el "97 % de ahorro" que se cita no dice lo que parece
> El paper reporta que en resúmenes de comunidad raíz *"it required over 97% fewer tokens"*. Dos precisiones imprescindibles:
> 1. Es ahorro en **query time**, no en indexado.
> 2. Es contra la condición de **resumir todo el texto fuente**, **no contra RAG vectorial**.
>
> Y el propio paper acota cuándo compensa: *"**For situations requiring many global queries over the same dataset**, summaries of root-level communities […] provide a data index that is both superior to vector RAG and achieves competitive performance […] at a fraction of the token cost."* Es una afirmación **condicional** que la divulgación suele convertir en absoluta.

### 7.3 Qué dicen las evaluaciones independientes

- **GraphRAG-Bench:** *"recent studies report that **GraphRAG frequently underperforms vanilla RAG on many real-world tasks**"* y *"Basic RAG is comparable to or outperforms GraphRAG in simple fact retrieval tasks that does not require complex reasoning."*
- **Han et al.:** RAG gana en Natural Questions (64,78 vs 63,01 F1). Y un diagnóstico revelador del fallo: *"only about 65,8 % of answer entities appear in the constructed KG for HotpotQA"* — **el grafo pierde un tercio de las entidades de respuesta.**
- Donde **sí** gana, de forma reproducida por ambos equipos: razonamiento multi-hop, resumen contextual y sensemaking global.

Y en el propio paper, un dato que la narrativa omite: de las cuatro métricas evaluadas, **solo dos ganan claramente** (comprehensiveness y diversity); *empowerment* y *directness* dieron *"mixed results"*.

### 7.4 Las alternativas — y un patrón que dice mucho

| | Estructura | Venue | Tokens/consulta* |
|---|---|---|---|
| **MS GraphRAG** | Grafo + comunidades (Leiden) | ❌ **Preprint** | 38.700 – 331.000 |
| **RAPTOR** | **Árbol** (UMAP + GMM), sin grafo | ✅ ICLR 2024 | ~3.400 |
| **HippoRAG** | Grafo + Personalized PageRank | ✅ NeurIPS 2024 | ~7.200 |
| **HippoRAG 2** | Grafo + PPR mejorado | ✅ ICML 2025 | **~1.000** |
| **LightRAG** | Grafo + vectores, sin comunidades | ✅ Findings EMNLP 2025 | ~100.000 |
| *RAG vainilla* | — | — | *879 – 954* |

\* GraphRAG-Bench (Xiang et al.)

> [!important] 🎯 El patrón que ordena toda esta sección
> **El único trabajo que no pasó revisión por pares es el canónico.** GraphRAG lleva más de dos años como preprint con ~1.800 citas, mientras que **las cuatro alternativas están publicadas en ICLR, NeurIPS, ICML y EMNLP**.
>
> Y la frontera ya se movió: **HippoRAG 2 alcanza ~1.000 tokens por consulta —nivel RAG vainilla— ganando en multi-hop.** Si el problema que tienes es el que GraphRAG ataca, hoy hay opciones dos órdenes de magnitud más baratas.
>
> **RAPTOR** merece atención aparte porque resuelve el mismo problema jerárquico **sin construir un grafo**: reduce dimensionalidad (UMAP), agrupa (GMM), resume cada grupo, re-embebe y recursa hasta formar un árbol. Y *"scales linearly in terms of both build time and token expenditure"*.

### 7.5 Veredicto de madurez

> [!warning] ⚠️ **EMERGING**, no consenso — con bolsones de práctica común
> **La idea** de recuperación estructurada jerárquicamente (grafo o árbol) para *global sensemaking* y multi-hop es **práctica común emergente**, validada por equipos independientes en venues de primer nivel.
>
> **La implementación canónica de Microsoft** es **EMERGING y probablemente ya superada**. Las señales, todas verificadas:
> - Paper sin revisión por pares tras dos años.
> - El README oficial dice literalmente: *"the provided code serves as a demonstration and **is not an officially supported Microsoft offering**"*. Lo mantienen activamente, pero **se niegan a respaldarlo como producto**.
> - Sus propios autores publicaron un sucesor cuyo argumento de venta es que el costo del original es prohibitivo.
> - Benchmarks independientes lo ponen por debajo de RAG vainilla en tareas simples.
> - **No existe oferta gestionada en ningún hyperscaler.** Las técnicas que alcanzan consenso terminan siendo un botón en una consola cloud; esta no llegó.
>
> **Valor que sí conserva: pedagógico.** El pipeline entidades → Leiden → comunidades → map-reduce es el marco más claro que existe para *entender* el problema del global sensemaking. Merece conocerse. Recomendarlo como default de producción **no está respaldado por la evidencia**.

---

## 8. 📊 La tabla que resume el tomo

Audiencia: 🔧 🧭 👔

| Técnica | ¿Paga? | Evidencia | Costo |
|---|---|---|---|
| **Reranking** ([[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|T07]]) | ✅ **Sí, lo primero** | BERGEN: *"often overlooked […] should be used"* | 1 pasada de cross-encoder |
| **Hybrid search** ([[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|T04]]) | ✅ **Sí, barato** | +2,5 mAP por +0,6 s | Mínimo |
| **HyDE** ([[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|T07]]) | ⚠️ **Depende del dominio** | +6,2 mAP en TREC DL… pero **negativo** en documentos con tablas. Ver §8.1 | ~3× latencia. **1 pseudo-documento basta** |
| **Clasificar si hace falta recuperar** | ✅ **Sí — la mejor** | Mejora calidad **y baja** latencia (16,41 → 11,58 s) | Negativo: ahorra |
| **IRCoT** | ⚠️ Solo multi-hop | FlashRAG: HotpotQA 41,5 vs 35,3 | Varias rondas |
| **Query decomposition** | ⚠️ Solo multi-hop | −2,7 mAP single-hop / **+36,7 % MRR** multi-hop | +16,7 s/consulta |
| **Step-back** | ⚠️ Probablemente | TimeQA +27 % — un solo trabajo | 1 llamada extra |
| **Query rewriting** | ❌ **No, en general** | −0,1 mAP a 3× latencia; peor que *no recuperar* en 3/8 datasets | 3× latencia |
| **Multi-query / RAG-Fusion** | ❌ **No** | Sin paper fundacional **y** con evidencia negativa: Hit@10 de 0,51 → 0,48 (§4.3) | N llamadas + fusión |
| **Self-RAG** | ⚠️ Discutido | ICLR Oral, pero por debajo del RAG estándar en evaluación unificada | **Requiere entrenar el modelo** |
| **CRAG** | ⚠️ Prometedor | Preprint; reproducción favorable pero débil | Evaluador + web search |
| **GraphRAG (MS)** | ❌ **No como default** | Pierde contra RAG vainilla en tareas simples | **~350×** tokens |
| **RAPTOR / HippoRAG 2** | ✅ Si necesitas jerarquía | ICLR 2024 / ICML 2025 | ~3.400 / **~1.000** tokens |

> [!important] 🎯 El patrón, en una línea
> **Lo que se paga en una sola pasada (reranking, hybrid search) rinde. Lo que se paga en varias llamadas al LLM rinde solo si tus preguntas son genuinamente multi-hop.** Y la técnica más rentable de todas es la que **decide no hacer trabajo**.

### 8.1 ⚠️ El caso de HyDE: cuando la evidencia se contradice

HyDE aparecía como la transformación mejor sostenida. Al ampliar la búsqueda, dos trabajos independientes **se contradicen frontalmente**:

| Trabajo | Veredicto sobre HyDE |
|---|---|
| Wang et al. (EMNLP 2024), TREC DL | ✅ **+6,2 mAP** (44,66 → 50,87) |
| ARAGOG (Eibich et al., 2024) | ✅ *"HyDE and LLM reranking **significantly enhance** retrieval precision"* |
| **Akarsu et al. (2026)**, documentos con tablas | ❌ *"HyDE **underperforms even vanilla dense retrieval** across all metrics (Recall@5: 0,544 vs. 0,587)"* |

> [!important] 🎯 La contradicción **es** el hallazgo
> No hay que resolverla eligiendo bando: hay que leerla como lo que dice. **La utilidad de HyDE depende del dominio.**
>
> Tiene sentido mecánico: HyDE funciona inventando un documento plausible que responda la pregunta, y buscando con él. Eso ayuda cuando la respuesta es **prosa** que el modelo puede imitar. Y falla cuando la respuesta es una **cifra en una tabla** — el modelo no puede alucinar el número correcto, así que el pseudo-documento lo aleja del objetivo en vez de acercarlo.
>
> **Regla derivada:** si tu corpus es texto expositivo, HyDE es candidato serio. Si es numérico, tabular o de datos estructurados, **mídelo antes**: hay evidencia publicada de que empeora.
>
> Esto vale para todo el tomo: ninguna de estas técnicas es buena o mala en abstracto. **Lo son respecto a un corpus y un tipo de pregunta.**

> [!example] 📊 Caso de negocio — Farmacéutica
> **Problema.** Un equipo de I+D despliega un asistente sobre su corpus de literatura científica interna y publicaciones: mecanismos de acción, interacciones, resultados de ensayos. El sistema base funciona. El equipo técnico propone una modernización completa: query rewriting, multi-query con fusión, y GraphRAG sobre todo el corpus — "porque las preguntas son complejas y todo está relacionado".
>
> **Técnica aplicada.** Antes de construir nada, se hace lo que este tomo recomienda: **medir el tráfico real** ([[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §5]]) y clasificarlo. El resultado reordena las prioridades: la gran mayoría de las consultas son de **un solo salto** —"¿cuál es la dosis máxima de X?", "¿qué efectos adversos reporta el ensayo Y?"— y una minoría clara, pero de altísimo valor, es genuinamente multi-hop: *"¿qué compuestos de nuestro portafolio comparten mecanismo con los que fallaron en fase II por hepatotoxicidad?"*.
>
> Se implementa entonces **un clasificador de consulta** que separa ambos flujos: las de un salto van por el camino barato (hybrid search + reranking), y solo las multi-hop activan descomposición. GraphRAG se descarta como default y se acota a un experimento sobre un subcorpus, con el costo de indexación presupuestado por delante.
>
> **Resultado.** La calidad medida sube en las consultas multi-hop —que era donde de verdad dolía— sin que el costo por consulta se dispare en el tráfico mayoritario. La propuesta original habría aplicado el camino caro al 100 % del tráfico para beneficiar a una fracción.
>
> La lección transferible: **la pregunta correcta no es "¿qué técnica avanzada adoptamos?" sino "¿qué fracción de nuestro tráfico la necesita?"** — y esa respuesta sale de los logs, no de la literatura.

---

## 9. 🧭 Guía de decisión

Audiencia: 🧭 👔

```
   ¿AÑADIR UNA TÉCNICA AVANZADA DE QUERY?

   0. ¿Puedes MEDIR el efecto? (Tomo 10)
        NO → para. Todo lo de abajo es una apuesta a ciegas.

   1. ¿Ya tienes reranking y hybrid search?
        NO → hazlo primero. Mejor evidencia y menor costo
             que cualquier cosa de este tomo.

   2. ¿Qué fracción de tu tráfico REAL es multi-hop?
        (mídelo en los logs, no lo supongas)
        ├─ Poca → NO añadas decomposition ni IRCoT.
        │         La evidencia dice que perderás calidad y latencia.
        └─ Mucha → decomposition e IRCoT se justifican.
                   Considera un CLASIFICADOR que las active solo ahí.

   3. ¿Tus usuarios hacen preguntas GLOBALES sobre todo el corpus
      ("¿cuáles son los temas principales?")?
        NO → no necesitas GraphRAG. Ni RAPTOR.
        SÍ → evalúa RAPTOR o HippoRAG 2 ANTES que MS-GraphRAG:
             misma familia de problema, 2 órdenes de magnitud menos caro.

   4. ¿Vas a añadir documentos con frecuencia?
        SÍ → descarta GraphRAG: no tiene actualización incremental barata
             (~14M tokens por regeneración de comunidades).

   5. Antes de cualquiera de las anteriores, prueba lo más rentable:
      un CLASIFICADOR que decida si hace falta recuperar o transformar.
      Mejora la calidad Y baja la latencia. Es lo único de esta lista
      que sale gratis. (Ver el dato de producción abajo.)
```

> [!important] 🎯 El dato que justifica el paso 5, y viene de producción real
> **Hussain & Nielbo (2026)** analizaron 20.000 pares consulta-flujo del tráfico real de una enciclopedia nacional. El hallazgo:
>
> *"only **27,8 %** of real user queries need LLM augmentation"* — frente a **más del 90 %** que sugerían las consultas sintéticas con las que se suele evaluar.
>
> Y la consecuencia medida: una cascada que decide caso por caso *"improves quality by +0,140 Composite Overall points over **Always-HyDE**, reduces latency by **31,8 %**"*.
>
> **Ganar en calidad y en latencia a la vez** solo ocurre cuando dejas de hacer trabajo innecesario. Casi tres de cada cuatro consultas reales no necesitaban ninguna transformación — y los benchmarks sintéticos no lo mostraban.

> [!danger] 🚨 El anti-patrón que este tomo existe para evitar
> **Adoptar una técnica porque aparece en un tutorial, sin medir si tu tráfico la necesita.** Es la vía documentada por la que un sistema termina costando 3× más y respondiendo peor — con literatura peer-reviewed confirmándolo.
>
> Y hay una formulación mejor del mismo principio, de **Laitenberger, Manning & Liu (EMNLP 2025)**: proponen establecer un RAG simple y bien construido como baseline obligatorio *"to ensure that added pipeline complexity **is justified by clear performance gains**"*. Su resultado incómodo: ese baseline simple *"consistently matches or outperforms more intricate methods"* en varios benchmarks de contexto largo — **incluido RAPTOR**, con presupuestos de tokens igualados.
>
> Antes de añadir cualquier cosa de este tomo: **asegúrate de que tu baseline simple está bien hecho, y mídelo.**

---

## 10. 📖 Glosario, checklist y referencias

### 10.1 Glosario express

| Término | Definición operativa |
|---|---|
| **Query rewriting** | Reescribir la consulta con un LLM antes de buscar. Evidencia comparativa desfavorable fuera de casos específicos |
| **Query decomposition** | Partir una pregunta compleja en sub-preguntas encadenadas. Paga solo en multi-hop genuino |
| **Compositionality gap** | Que el modelo sepa los hechos sueltos pero no los componga. **No se cierra escalando el modelo** |
| **Multi-query retrieval** | Varias reformulaciones + fusión de rankings. Sin paper fundacional; ancestro real en IR de 1995 |
| **Step-back prompting** | Derivar una pregunta más general para recuperar el contexto de principio |
| **IRCoT** | Intercalar retrieval con los pasos del chain-of-thought |
| **Self-RAG** | Modelo **entrenado** con *reflection tokens* que decide cuándo recuperar y critica lo recuperado |
| **CRAG** | Evaluador ligero del retrieval que dispara acciones correctivas (incluida búsqueda web) |
| **Global sensemaking** | Preguntas sobre el corpus completo cuya respuesta **no está en ningún chunk** |
| **GraphRAG** | Entidades → grafo → comunidades (Leiden) → resúmenes → map-reduce |
| **Gleanings** | Repreguntar al LLM si olvidó entidades. Multiplica las llamadas del indexado |
| **RAPTOR** | Alternativa **sin grafo**: árbol jerárquico por UMAP + GMM + resumen recursivo. Escala lineal |
| **Relevance feedback** | Reformular la consulta con evidencia de qué resultó relevante (Rocchio, 1971) |

### 10.2 Checklist de comprensión

- [ ] Sé que este problema tiene 55 años y que las técnicas LLM son su última iteración, no su invención.
- [ ] Puedo citar la evidencia de que query rewriting y decomposition **empeoran** el retrieval en single-hop.
- [ ] Sé que la técnica más rentable es **decidir no recuperar** cuando no hace falta.
- [ ] Entiendo el *compositionality gap* y por qué **no se arregla con modelos más grandes**.
- [ ] Sé que **multi-query no tiene paper fundacional**, y sé citar su ascendencia real (Belkin 1995 + RRF).
- [ ] Distingo Self-Ask (canónico para RAG) de Least-to-Most (descomposición **sin** retrieval).
- [ ] Sé que **Self-RAG requiere entrenar el modelo**, no es prompting.
- [ ] Entiendo qué es el *global sensemaking* y por qué RAG vectorial no lo cubre.
- [ ] Sé que el paper de GraphRAG **no reporta su costo de indexado**, y que Microsoft lo cuantificó después en ~1.000×.
- [ ] Sé que el "97 % de ahorro" de GraphRAG es en query time y **no contra RAG vectorial**.
- [ ] Conozco RAPTOR y HippoRAG 2 como alternativas publicadas y dos órdenes de magnitud más baratas.
- [ ] 🚨 Antes de adoptar cualquier técnica de este tomo, **mido qué fracción de mi tráfico la necesita**.

### 10.3 Referencias

> [!note] Todas verificadas antes de escribir (2026-07-28)
> Con evidencia textual y URL. Las fichas completas se consolidan en el [[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 · Bibliografía]].

**Antecedente histórico**
- Rocchio, J. J. (1971). "Relevance feedback in information retrieval". En G. Salton (ed.), *The SMART Retrieval System — Experiments in Automatic Document Processing*, 313–323. Prentice Hall. ⚠️ *Verificada vía fuente secundaria autorizada (bibliografía de Manning, Raghavan & Schütze); el documento primario no tiene edición digital pública. Sin DOI.*
- Belkin, N. J., Kantor, P., Fox, E. A., & Shaw, J. A. (1995). "Combining the evidence of multiple query representations for information retrieval". *Information Processing & Management*, 31(3), 431–448. DOI 10.1016/0306-4573(94)00057-A. ✅
- Cormack, G. V., Clarke, C. L. A., & Büttcher, S. (2009). SIGIR '09, 758–759. ✅ *(ficha completa en el Tomo 16)*

**Transformación de queries**
- Ma, X., Gong, Y., He, P., Zhao, H., & Duan, N. (2023). "Query Rewriting for Retrieval-Augmented Large Language Models". *EMNLP 2023*. arXiv:2305.14283. ✅
- Press, O., Zhang, M., Min, S., Schmidt, L., Smith, N. A., & Lewis, M. (2023). "Measuring and Narrowing the Compositionality Gap in Language Models". *Findings of the ACL: EMNLP 2023*, 5687–5711. DOI 10.18653/v1/2023.findings-emnlp.378. ✅ **(Self-Ask — la canónica para RAG)**
- Zhou, D., Schärli, N., Hou, L., Wei, J., Scales, N., Wang, X., Schuurmans, D., Cui, C., Bousquet, O., Le, Q., & Chi, E. (2023). "Least-to-Most Prompting Enables Complex Reasoning in Large Language Models". *ICLR 2023*. arXiv:2205.10625. ✅ ⚠️ *No contiene retrieval — no citar como referencia de RAG. Venue tomado de los metadatos de arXiv: OpenReview bloqueó el acceso automatizado.*
- Khot, T., Trivedi, H., Finlayson, M., Fu, Y., Richardson, K., Clark, P., & Sabharwal, A. (2023). "Decomposed Prompting: A Modular Approach for Solving Complex Tasks". *ICLR 2023*. arXiv:2210.02406. ✅
- Zheng, H. S., Mishra, S., Chen, X., Cheng, H.-T., Chi, E. H., Le, Q. V., & Zhou, D. (2024). "Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models". *ICLR 2024*. arXiv:2310.06117. ✅
- Jagerman, R., Zhuang, H., Qin, Z., Wang, X., & Bendersky, M. (2023). *Query Expansion by Prompting Large Language Models*. arXiv:2305.03653. ✅ *Preprint sin venue.*

**Patrones adaptativos**
- Trivedi, H., Balasubramanian, N., Khot, T., & Sabharwal, A. (2023). "Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions". *ACL 2023*, 10014–10037. DOI 10.18653/v1/2023.acl-long.557. ✅ **(IRCoT)**
- Asai, A., Wu, Z., Wang, Y., Sil, A., & Hajishirzi, H. (2024). "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection". *ICLR 2024 (Oral)*. arXiv:2310.11511. ✅ *El estatus "Oral" está verificado; la afirmación "top 1 %" que circula, no.*
- Yan, S.-Q., Gu, J.-C., Zhu, Y., & Ling, Z.-H. (2024). *Corrective Retrieval Augmented Generation*. arXiv:2401.15884. ✅ ⚠️ **Preprint sin venue.**

**Recuperación estructurada**
- Edge, D., Trinh, H., Cheng, N., Bradley, J., Chao, A., Mody, A., Truitt, S., Metropolitansky, D., Ness, R. O., & Larson, J. (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization*. arXiv:2404.16130. ✅ ⚠️ **Preprint sin venue tras más de dos años.** *Nota: la v1 lista 8 autores y la v2 añade dos (10). Aquí se usa la v2, la vigente.*
- Sarthi, P., Abdullah, S., Tuli, A., Khanna, S., Goldie, A., & Manning, C. D. (2024). "RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval". *ICLR 2024*. arXiv:2401.18059. ✅
- Jiménez Gutiérrez, B., Shu, Y., Gu, Y., Yasunaga, M., & Su, Y. (2024). "HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models". *NeurIPS 2024*. arXiv:2405.14831. ✅
- Jiménez Gutiérrez, B., Shu, Y., Qi, W., Zhou, S., & Su, Y. (2025). "From RAG to Memory: Non-Parametric Continual Learning for Large Language Models". *ICML 2025*, PMLR 267, 21497–21515. ✅ **(HippoRAG 2)**
- Guo, Z., Xia, L., Yu, Y., Ao, T., & Huang, C. (2025). "LightRAG: Simple and Fast Retrieval-Augmented Generation". *Findings of the ACL: EMNLP 2025*, 10746–10761. DOI 10.18653/v1/2025.findings-emnlp.568. ✅ *Nota: fue enviado a ICLR 2025 y **retirado**; citarlo como "publicado en EMNLP" es impreciso — es **Findings**.*
- Traag, V. A., Waltman, L., & van Eck, N. J. (2019). "From Louvain to Leiden: guaranteeing well-connected communities". *Scientific Reports*, 9, 5233. ⚠️ *Tomada de la cita interna de Edge et al.; conviene confirmar el DOI en la fuente si se cita textualmente.*

**Evaluaciones comparativas** — la columna vertebral de este tomo
- Wang, X., Wang, Z., Gao, X., Zhang, F., Wu, Y., Xu, Z., Shi, T., Wang, Z., Li, S., Qian, Q., Yin, R., Lv, C., Zheng, X., & Huang, X. (2024). "Searching for Best Practices in Retrieval-Augmented Generation". *EMNLP 2024*, 17716–17736. arXiv:2407.01219. ✅ **La cita más importante del tomo.**
- Jin, J., Zhu, Y., Dong, G., Zhang, Y., Yang, X., Zhang, C., Zhao, T., Yang, Z., Dou, Z., & Wen, J.-R. (2025). "FlashRAG: A Modular Toolkit for Efficient Retrieval-Augmented Generation Research". *WWW 2025, Resource Track*. arXiv:2405.13576. ✅ ⚠️ *Las cifras citadas proceden de la v1; existe una v2 posterior — conviene contrastarlas antes de reproducirlas fuera de esta guía.*
- Zhang, X., Song, Y., Wang, Y., et al. (2024). "RAGLAB: A Modular and Research-Oriented Unified Framework for Retrieval-Augmented Generation". *EMNLP 2024, System Demonstrations*, 408–418. DOI 10.18653/v1/2024.emnlp-demo.43. ✅
- Rau, D., Déjean, H., Chirkova, N., Formal, T., Wang, S., Nikoulina, V., & Clinchant, S. (2024). "BERGEN: A Benchmarking Library for Retrieval-Augmented Generation". *Findings of EMNLP 2024*, 7640–7663. ✅
- Ammann, P. J. L., Golde, J., & Akbik, A. (2025). *Question Decomposition for Retrieval-Augmented Generation*. **ACL SRW 2025**. arXiv:2507.00355. ✅
- Xiang, Z., Wu, C., Zhang, Q., Chen, S., Hong, Z., Huang, X., & Su, J. (2025). *When to use Graphs in RAG: A Comprehensive Analysis for Graph Retrieval-Augmented Generation*. arXiv:2506.05690. ✅ ⚠️ *Los autores anuncian aceptación en ICLR'26 en su repositorio; **no se pudo confirmar de forma independiente** (OpenReview bloquea el acceso automatizado). Tratar como preprint.*
- Han, H., Ma, L., Wang, Y., et al. (2025). *RAG vs. GraphRAG: A Systematic Evaluation and Key Insights*. arXiv:2502.11371. ✅ ⚠️ *Preprint sin venue.*
- Laitenberger, A., Manning, C. D., & Liu, N. F. (2025). "Stronger Baselines for Retrieval-Augmented Generation with Long-Context Language Models". *EMNLP 2025*, 32559–32569. arXiv:2506.03989. ✅ **El mejor respaldo de la tesis del tomo:** un RAG simple bien construido iguala o supera a métodos más intrincados —incluido RAPTOR— con presupuestos de tokens igualados.
- Hussain, Z., & Nielbo, K. (2026). *The Coverage Illusion: From Pre-retrieval Routing Failure to Post-retrieval Cascades in a Production RAG System*. arXiv:2605.27220. ✅ ⚠️ *Preprint sin venue. La mejor evidencia de tráfico real contra las transformaciones "always-on": solo el 27,8 % de las consultas reales las necesitan.*
- Medrano, L., Verma, A., & Chhabra, M. (2026). *RAG-Fusion en producción*. arXiv:2603.02153. ✅ ⚠️ *Preprint sin venue. La evidencia negativa más directa contra multi-query/RAG-Fusion.*
- Akarsu, M., Karaman, R. K., & Mierbach, C. (2026). *From BM25 to Corrective RAG: Benchmarking Retrieval Strategies for Text-and-Table Documents*. arXiv:2604.01733. ✅ ⚠️ *Preprint sin venue. Fuente del hallazgo de que **HyDE empeora en corpus tabulares** (§8.1).*
- Kotte, V. (2026). *Not All Queries Need Rewriting*. arXiv:2603.13301. ✅ ⚠️ *Preprint de autor único.*
- Bigdeli, A., Hamidi Rad, R., Le, H. S., Incesu, M., Arabzadeh, N., Clarke, C. L. A., & Bagheri, E. (2026). *Reproducibility study of LLM query reformulation*. arXiv:2604.27421. ✅ ⚠️ *Preprint sin venue. El estudio de reproducibilidad más amplio sobre reescritura de queries.*
- Eibich, M., Nagpal, S., & Fred-Ojala, A. (2024). *ARAGOG: Advanced RAG Output Grading*. arXiv:2404.01037. ✅ ⚠️ *Preprint sin venue.*
- Ferrazzi, P., Cvjeticanin, M., Piraccini, A., & Giannuzzi, D. (2026). *Is Agentic RAG worth it?*. **ACL 2026 (Industry Track)**. arXiv:2601.07711. ✅ *Fuente del reparto de latencia: la reescritura de query consume ~la mitad del reloj.*
- Iturra-Bocaz, G., & Galuscakova, P. (2026). *A Reproducibility Study of Metacognitive Retrieval-Augmented Generation*. **SIGIR 2026**. arXiv:2604.19899. DOI 10.1145/3805712.3808551. ✅ ⚠️ *La aceptación en SIGIR consta en los metadatos de arXiv y el DOI está registrado, pero la página de ACM devuelve 403 — no se pudo confirmar en el registro del editor.*

**Fuentes de industria** (citadas como tales, no como literatura académica)
- LangChain (2023, 24 de octubre). *Query Transformations*. ✅ *Donde se popularizan multi-query y RAG-Fusion — sin citar paper, porque no existe.*
- Edge, D., Trinh, H., & Larson, J. (2024, 25 de noviembre). *LazyGraphRAG: Setting a new standard for quality and cost*. Microsoft Research Blog. ✅
- `microsoft/graphrag` (MIT). ✅ *README verificado: "not an officially supported Microsoft offering".*

> [!danger] 🚨 Cifras que circulan y este tomo NO usa
> Se descartaron por no ser trazables a una fuente primaria: el costo de **"$33.000 por indexar un dataset"** (blog de Medium), **"$7 por un libro de 32.000 palabras"** (blog), las estadísticas de adopción tipo **"67 % de las Fortune 500"** y **"ROI del 340 %"** (marketing sin metodología publicada), el **"1,7× más lento"** de RAG-Fusion, y las cifras de costo de un post de Microsoft Tech Community cuya extracción falló dos veces.
>
> Se listan aquí a propósito: **circulan mucho y alguien las va a encontrar.** Saber que no están respaldadas es parte del contenido.

---

## 🔗 Conexiones

- [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing y re-ranking]] — **el prerrequisito.** Ahí están query rewriting, NER, HyDE y el reranking, con el curso como fuente. Este tomo aporta la evidencia comparativa de cuánto rinden.
- [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search]] — hybrid search y **RRF**, el mecanismo de fusión que multi-query reutiliza.
- [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Evaluación y agentic RAG]] — los routers y flujos agentic sobre los que se construyen los patrones adaptativos de §6, y las métricas para saber si algo de esto funcionó.
- [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · Producción]] — **cómo medir qué fracción de tu tráfico es multi-hop.** Sin eso, todo este tomo es especulación.
- [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 · Quantization y trade-offs]] — el marco de costo/latencia contra el que se juzga cada técnica de aquí.

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 · Quantization, trade-offs y multimodal RAG]] · Siguiente → [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13 · ⭐ Complemento: frameworks]]
