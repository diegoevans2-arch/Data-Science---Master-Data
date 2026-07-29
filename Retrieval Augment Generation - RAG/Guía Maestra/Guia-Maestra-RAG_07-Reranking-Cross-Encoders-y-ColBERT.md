---
title: "Tomo 07 — Query parsing, arquitecturas de scoring y re-ranking"
tags: [rag, query-parsing, query-rewriting, hyde, ner, bi-encoder, cross-encoder, colbert, reranking, maxsim]
audiencias: [tecnico, puente, ejecutivo]
tomo: 07
version: 1.1
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 07 — Query parsing, arquitecturas de scoring y re-ranking

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] · Siguiente → [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación]]

---

> [!info] ¿Por qué importa esta sección?
> Este tomo cierra el Módulo 3 y con él **todo el retriever**. Los tomos anteriores optimizaron el *centro* del pipeline: cómo buscar ([[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|05]]) y qué indexar ([[Guia-Maestra-RAG_06-Chunking|06]]). Aquí atacamos los dos **extremos**, que es donde quedan las mejoras más baratas y más rentables:
>
> - **Antes de buscar:** el prompt de un humano es una mala query de búsqueda. Arreglarlo es *query parsing*.
> - **Después de buscar:** el modelo rápido que usaste para buscar en millones de documentos no es el mejor para ordenar veinte. Cambiarlo es *re-ranking*.
>
> Y en el medio, las **arquitecturas de scoring** que explican por qué esas dos etapas existen. El re-ranking en particular es, según el curso, *"una de las primeras técnicas que deberías explorar"* — a menudo una línea de código para una mejora sustancial.

> [!abstract] 👔 Impacto ejecutivo
> Estas tres técnicas comparten un patrón: **mejoran la calidad sin cambiar el modelo generador ni re-indexar la base**. Son las palancas de mejor relación esfuerzo/resultado de todo el sistema.
> - **Decisiones que habilita:** subir la relevancia de un RAG ya desplegado sin rehacer la infraestructura; hacer que el sistema tolere cómo escriben los usuarios reales; justificar milisegundos extra de latencia con una mejora medible de precisión.
> - **Costo o riesgo de hacerlo mal:** pasar el prompt crudo del usuario al retriever desperdicia buena parte de la calidad del sistema; y usar el modelo de búsqueda como si fuera el mejor juez de relevancia deja los mejores documentos en el puesto 8 en vez del 1 — donde el LLM ya casi no los mira.
> - **Pregunta ejecutiva que responde:** *"el sistema encuentra los documentos correctos pero los ordena mal, o no entiende cómo pregunta la gente — ¿qué se puede hacer sin empezar de nuevo?"*

---

## 1. 🗺️ El mapa: tres momentos, tres optimizaciones

Audiencia: 🔧 🧭

Antes del detalle, dónde encaja cada pieza de este tomo:

```
   Prompt del usuario
   (conversacional, ambiguo, con paja)
            │
            ▼
   ┌───────────────────────────┐
   │  ① QUERY PARSING          │   PRE-retrieval
   │  reescribir · NER · HyDE  │   "arreglar la pregunta"
   └────────────┬──────────────┘
                ▼
   ┌───────────────────────────┐
   │  ② BÚSQUEDA               │   El retriever del Módulo 2 + 3:
   │  hybrid: BM25 + vectorial │   rápido, sobre millones de docs
   │  arquitectura BI-ENCODER  │   → OVER-FETCH: 20–100 docs
   └────────────┬──────────────┘
                ▼
   ┌───────────────────────────┐
   │  ③ RE-RANKING             │   POST-retrieval
   │  cross-encoder / LLM      │   caro pero solo sobre 20–100
   └────────────┬──────────────┘   → devuelve los 5–10 finales
                ▼
        augmented prompt  ──►  LLM
```

> [!tip] 💡 La idea que unifica el tomo
> Es el patrón del **embudo de dos velocidades**: un modelo **rápido y tosco** filtra millones hasta decenas; un modelo **lento y fino** ordena esas decenas. Ninguno de los dos podría hacer el trabajo completo — el rápido no discrimina lo suficiente, el fino no escala. Juntos dan calidad de modelo caro a precio de modelo barato.
>
> Es exactamente la arquitectura de dos etapas de los sistemas de recomendación, y la misma lógica que usa cualquier proceso de selección: filtro de CV automático primero, entrevista a fondo después.

---

## 2. 🧹 Query parsing: arreglar la pregunta

Audiencia: 🔧 🧭 👔

### 2.1 El problema: los humanos no escriben queries

**🔧 El diagnóstico del curso:** los sistemas RAG se despliegan en contextos donde el usuario espera **conversar** con un LLM, como si hablara con otra persona. Y como consecuencia directa:

> [!important] La frase clave
> **Los prompts escritos por humanos son malas queries de búsqueda.**
>
> En vez de pasar el prompt directamente a la vector database, el retriever puede **parsearlo** para identificar su intención y **editarlo, reescribirlo o transformarlo por completo** para optimizarlo para retrieval.

### 2.2 Query rewriting: la técnica que sí vas a usar

**🔧 Definición técnica:** la solución más simple y **de lejos la más usada**: un LLM reescribe la query antes de enviarla al retriever.

El curso da el prompt del reescritor casi literal, y vale como plantilla:

```
   El siguiente prompt fue enviado por un usuario para consultar una base de
   documentos médicos que vinculan síntomas con diagnósticos. Reescribe el
   prompt para optimizarlo para la búsqueda haciendo lo siguiente:

     · Clarificar frases ambiguas
     · Usar terminología médica donde corresponda
     · Agregar sinónimos que aumenten las probabilidades de encontrar
       documentos coincidentes
     · Eliminar información innecesaria o distractora

   [prompt del usuario]
```

**El antes y el después** del ejemplo del curso es la mejor demostración de por qué esto importa:

```
   ══ ANTES (lo que escribe una persona real) ══

   "I was out walking my dog, a beautiful black lab named Poppy, when she
    raced away from me and yanked on her leash hard while I was holding it.
    Three days later, my shoulder is still numb and my fingers are all pins
    and needles. What's going on?"

                              │  ← LLM reescritor
                              ▼

   ══ DESPUÉS (lo que un retriever puede usar) ══

   "Experienced a sudden forceful pull on the shoulder resulting in persistent
    shoulder numbness and finger numbness for three days. What are the
    potential causes or diagnoses such as neuropathy or nerve impingement?"
```

Desglosando qué hizo el reescritor:

| Operación | Qué desapareció / apareció |
|---|---|
| **Eliminar paja** | Fuera el labrador negro, fuera Poppy, fuera el paseo |
| **Clarificar** | *"yanked on her leash hard"* → *"sudden forceful pull on the shoulder"* |
| **Terminología del dominio** | *"pins and needles"* → *"numbness"*; se añaden *"neuropathy"*, *"nerve impingement"* |
| **Sinónimos** | Aumentan la probabilidad de match léxico ([[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]]) |

> [!tip] 💡 Analogía
> Es la diferencia entre lo que un paciente le cuenta al médico y **lo que el médico escribe en la ficha**. El paciente cuenta una historia con el perro, el clima y la hora; el médico anota *"parestesia distal post-tracción cervicobraquial, 72 h"*. Ambos describen lo mismo, pero **solo uno de los dos textos sirve para buscar en la literatura médica**. El query rewriter es el que traduce del primero al segundo.

> [!abstract] 👔 El veredicto del curso sobre costo-beneficio
> *"Aunque puedes y debes iterar sobre el prompt que usas para el query rewriting, en general **los beneficios que obtienes son sustanciales y justifican fácilmente el costo adicional** de la llamada al LLM necesaria para limpiar cada prompt."*
>
> Traducido: es una llamada extra al modelo por consulta, y **vale la pena**. Si vas a añadir una sola cosa a tu pipeline, empieza por aquí.

> [!warning] ⚠️ Lo que el curso no menciona y hay que vigilar
> El query rewriting agrega **una llamada de LLM en el camino crítico**, con tres consecuencias que conviene anticipar:
> - **Latencia:** el usuario espera esa llamada antes de que empiece la búsqueda.
> - **Un punto de fallo más:** si el reescritor cae o alucina, la query que llega al retriever puede ser peor que la original. Conviene un *fallback* a la query cruda.
> - **Pérdida de señal:** un reescritor agresivo puede **eliminar el término exacto** que iba a hacer el match (un SKU, un código, un nombre propio). Mitigación práctica: buscar con **ambas** queries —la original y la reescrita— y fusionar los resultados con RRF ([[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#6.3 Reciprocal Rank Fusion (RRF)|Tomo 04]]).

### 2.3 NER: extraer entidades de la query

**🔧 Definición técnica:** **Named Entity Recognition** reconoce **categorías de información** en la query: lugares, personas, fechas, personajes de ficción, etc. Esa información se usa después para informar la búsqueda vectorial **o el metadata filtering** que viene más adelante en el pipeline.

El curso usa **GLiNER**, un modelo de NER generalista: le pasas un texto y **una lista de tipos de entidad que quieres que identifique** (persona, libro, ubicación, fecha, actor, personaje), y devuelve la query etiquetada.

```
   Query:  "¿qué dijo Macron sobre Europa en abril de 2024?"
                       │
                       ▼  NER
   ┌──────────────────────────────────────────────────┐
   │  PERSONA:  Macron                                │
   │  LUGAR:    Europa                                │
   │  FECHA:    abril de 2024                         │
   └──────────────────────────────────────────────────┘
                       │
                       ├──► informa la búsqueda vectorial
                       └──► alimenta el METADATA FILTER:
                            Filter.by_property('pubDate')
                                  .greater_than('2024-04-01')
```

**Ventaja operativa que destaca el curso:** GLiNER es *"un modelo muy eficiente, y podemos correrlo cada vez que llega una query"*. Agrega algo de latencia, pero la calidad del retrieval puede mejorar significativamente.

> [!important] 🎯 Aquí se cierra un círculo del Tomo 03
> El [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25#2.2 La sutileza que casi todos pasan por alto|Tomo 03]] estableció que los filtros de metadata **no salen de lo que el usuario escribió, sino de quién es el usuario**. NER abre la **segunda vía legítima**: derivar filtros de la query, pero de forma **estructurada y explícita** — extrayendo una fecha o un lugar y convirtiéndolo en un filtro, no dejando que el embedding "adivine" la restricción.
>
> Y es la respuesta al **caso Kyoto** del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#5.3 🔬 Honestidad sobre qué NO capturan|Tomo 04]]: si necesitabas garantizar "solo Asia", NER + filtro es cómo se hace.

> [!warning] ⚠️ La limitación genuina de NER
> **Los modelos de NER solo extraen entidades de las categorías que se les configuró explícitamente para identificar.** Si tu dominio tiene entidades propias (un tipo de póliza, un código de repuesto, una figura regulatoria) y no están en la lista, NER no las va a ver. No es un extractor universal: es un extractor de lo que le pediste.

### 2.4 HyDE: buscar con un documento imaginario

Audiencia: 🔧 🧭

**🔧 Definición técnica:** **HyDE** (*Hypothetical Document Embeddings*) refina la query generando un **documento hipotético que sería el resultado ideal** de la búsqueda. Después se embebe **ese documento**, y **su vector** es el que se usa para buscar — no el de la query original.

```
   Query del usuario
   "¿por qué tengo el hombro dormido después de un tirón?"
            │
            ▼  LLM: "escribe el documento ideal que responda esto"
   ┌──────────────────────────────────────────────────────────┐
   │  DOCUMENTO HIPOTÉTICO (inventado, no se muestra a nadie) │
   │  "La tracción brusca del miembro superior puede producir │
   │   una neuropatía por compresión del plexo braquial. Los  │
   │   síntomas incluyen parestesia distal y..."              │
   └────────────────────────┬─────────────────────────────────┘
                            ▼  se embebe ESTE texto
                     vector de búsqueda
                            │
                            ▼
                    índice vectorial
```

> [!tip] 💡 La intuición: dejar de comparar peras con manzanas
> El argumento del curso es elegante. Normalmente el retriever tiene que emparejar **una pregunta con un documento** — dos tipos de texto **distintos**. Una pregunta es corta, interrogativa, coloquial; un documento es largo, afirmativo, técnico. Estás comparando *"apples to oranges"*.
>
> Con HyDE comparas **un documento hipotético perfecto contra documentos reales**: manzanas con manzanas. Le estás diciendo al retriever no solo *cuál es la intención de la pregunta*, sino **qué aspecto tendría un buen resultado**.

**El trade-off, según el curso:** HyDE *"efectivamente ofrece mejoras de rendimiento, al costo de latencia añadida en la búsqueda y de recursos computacionales para correr el LLM que genera los documentos hipotéticos"*.

> [!note] ⚠️ El detalle que la gente pasa por alto
> **El documento hipotético reemplaza a la query original en la búsqueda semántica.** No se concatena, no se promedia: se sustituye. Y como el documento lo *inventó* un LLM, puede contener afirmaciones falsas — lo cual **no importa**, porque nunca se le muestra al usuario ni al generador final. Solo se usa su vector. Es una de las pocas veces en RAG donde una alucinación es inofensiva y hasta útil.
>
> Riesgo real, en cambio: si el LLM inventa un documento sobre el **tema equivocado**, la búsqueda se desvía por completo. HyDE amplifica tanto los aciertos como los errores de interpretación.

### 2.5 Cuál usar: el consejo explícito del curso

> [!important] 🎯 El veredicto, casi literal
> *"En mi experiencia, tener **algún tipo** de query parsing es una pieza clave de tu sistema RAG. En casi todos los casos, el **query rewriting básico** —simplemente usar un LLM bien prompteado para hacer retoques básicos al prompt del usuario— es el enfoque correcto. Técnicas más avanzadas como NER, HyDE y otras pueden dar beneficios adicionales, pero son más complejas de operar y **no necesariamente darán mejores resultados**. Experimenta con ellas y deja que los resultados decidan cómo tiene que evolucionar tu proyecto."*

| Técnica | Costo | Cuándo |
|---|---|---|
| **Query rewriting** | 1 llamada LLM | **Empieza aquí. Casi siempre es suficiente** |
| **NER** | 1 modelo pequeño y eficiente | Cuando hay entidades que deben convertirse en **filtros** (fechas, lugares, categorías) |
| **HyDE** | 1 llamada LLM (genera texto largo) | Cuando la brecha pregunta↔documento es grande y lo demás ya está optimizado |

---

## 3. 🏗️ Las tres arquitecturas de scoring

Audiencia: 🔧 🧭

Todo el semantic search visto hasta ahora usa la arquitectura *vanilla*: un vector por documento, un vector por prompt, se comparan. Funciona muy bien — pero hay arquitecturas que puntúan mejor. Entenderlas es entender **por qué existe el re-ranking**.

### 3.1 Bi-encoder: el default

**🔧 Definición técnica:** es lo que hemos usado todo el curso. Cada documento recibe su vector de un embedding model; cuando llega el prompt, también se embebe; un algoritmo ANN encuentra rápidamente los documentos cercanos ([[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]]).

**El nombre lo dice todo:** *bi*-encoder = documentos y prompt se embeben **por separado**.

```
   OFFLINE                          ONLINE (al llegar la query)
   ───────                          ──────
   doc₁ ──► [encoder] ──► vec₁      prompt ──► [encoder] ──► vec_q
   doc₂ ──► [encoder] ──► vec₂                      │
   doc₃ ──► [encoder] ──► vec₃                      ▼
    ...                                    ANN sobre los vec₁..ₙ
   (¡todo precomputado!)                    (rapidísimo)
```

> [!important] Por qué esa separación es la clave de todo
> Como el documento y el prompt **nunca se ven** durante la codificación, **todos los documentos pueden embeberse por adelantado** y en query time solo hay que embeber el prompt. Eso es lo que hace la búsqueda tan rápida.
>
> Y es también su **límitación fundamental**: el modelo nunca considera el prompt y el documento *juntos*, así que no puede capturar interacciones finas entre ambos.

### 3.2 Cross-encoder: el gold standard que no escala

**🔧 Definición técnica:** para puntuar un documento, un cross-encoder **concatena el documento con el prompt** y pasa el texto combinado por lo que es esencialmente un embedding model especializado. Como ambos están en el input, el modelo puede entender **interacciones contextuales profundas** que un bi-encoder se pierde. La salida es directamente un **score de relevancia**, normalmente entre 0 y 1 — interpretable como la probabilidad de match.

```
   prompt: "great places to eat in New York"

   ┌────────────────────────────────────────────────────────┐
   │  [prompt + doc₁] ──► [CROSS-ENCODER] ──► 0.70          │
   │  [prompt + doc₂] ──► [CROSS-ENCODER] ──► 0.12          │
   │  [prompt + doc₃] ──► [CROSS-ENCODER] ──► 0.45          │
   └────────────────────────────────────────────────────────┘
                              ▲
        una pasada del modelo POR CADA PAR prompt-documento
```

**Un cross-encoder casi siempre da mejores resultados que un bi-encoder**, medido con las métricas habituales de relevancia. Y tiene un problema fatal:

> [!danger] 🚨 Por qué es inviable como técnica de búsqueda
> **Escala terriblemente.** Con millones o miles de millones de documentos, **cada prompt exigiría pasar miles de millones de pares por el modelo**. Y no hay forma de precomputar nada: el cross-encoder opera sobre el **par** prompt-documento, y el prompt no existe hasta que el usuario lo envía.
>
> Es la asimetría exacta que resuelve el re-ranking: **demasiado lento para buscar, perfecto para ordenar unos pocos.**

### 3.3 ColBERT: el intermedio

**🔧 Definición técnica:** **ColBERT** = *Contextualized Late Interaction over BERT*. La idea: generar los vectores de documento **por adelantado** como un bi-encoder, pero capturar **interacciones profundas** entre prompt y documento como un cross-encoder.

El truco está en el grano: en vez de **un vector por documento**, genera **un vector por cada token**.

```
   Documento de 1.000 tokens  →  1.000 vectores densos
   Prompt de 10 tokens        →  10 vectores densos
```

**El scoring se llama MaxSim:** cada token del prompt busca **su token más similar** en el documento.

```
                       ── tokens del DOCUMENTO ──►
                     New   York   City   cuisine   ...
        ┌──────────┬──────┬──────┬──────┬─────────┐
   p    │ great    │ 0.2  │ 0.2  │ 0.3  │  0.4    │  → max = 0.4
   r    │ places   │ 0.3  │ 0.3  │ 0.6  │  0.3    │  → max = 0.6
   o    │ eat      │ 0.1  │ 0.1  │ 0.2  │  0.8    │  → max = 0.8  ← "eat"↔"cuisine"
   m    │ New      │ 0.9  │ 0.7  │ 0.5  │  0.1    │  → max = 0.9
   p    │ York     │ 0.7  │ 0.9  │ 0.6  │  0.1    │  → max = 0.9
   t    └──────────┴──────┴──────┴──────┴─────────┘
                                                      SUMA = score del documento
```

Con un prompt de 10 tokens y un documento de 100, la grilla tiene 1.000 pares de scores de similitud. **Se toma el máximo de cada fila y se suman**: ese es el *MaxSim score*.

> [!tip] 💡 Analogía
> Un bi-encoder compara **dos resúmenes de una línea**: rápido, pero se pierde el detalle. Un cross-encoder **lee los dos textos juntos, palabra por palabra**: precisísimo y lentísimo. ColBERT es como tener **fichado cada concepto** de cada documento por separado: cuando llega la pregunta, cada palabra de la pregunta encuentra su mejor correspondencia entre las fichas. No lee todo junto, pero tampoco se conforma con el resumen.

| ✅ Gana | ⚠️ Cuesta |
|---|---|
| Escalabilidad cercana a un bi-encoder | **El almacenamiento crece con los tokens**: un documento de 2.000 tokens necesita 2.000 vectores (un bi-encoder necesitaría 1) |
| Gran parte de la riqueza de un cross-encoder | Scoring más costoso que un bi-encoder (aunque usable en tiempo casi real) |

**Cuándo se justifica:** el curso es concreto — cada vez más vector databases soportan ColBERT o enfoques similares, *"especialmente para proyectos que requieren precisión y comprensión contextual profunda. Por ejemplo, en los campos legal o médico, el trade-off de aumentar significativamente la huella de memoria de vectores a cambio de calidad de búsqueda puede valer la pena."*

### 3.4 Las tres, lado a lado

| | **Bi-encoder** | **Cross-encoder** | **ColBERT** |
|---|---|---|---|
| **Vectores por documento** | 1 | — (no precomputa) | **1 por token** |
| **¿Precomputable?** | ✅ Sí | ❌ No | ✅ Sí |
| **Calidad de ranking** | Buena | **La mejor** | Casi la de cross-encoder |
| **Velocidad** | **La mejor** | Inviable a escala | Cercana a bi-encoder |
| **Almacenamiento** | Mínimo | Mínimo | **Órdenes de magnitud más** |
| **Rol en el pipeline** | **La búsqueda** (default) | **El re-ranking** | Búsqueda de alta precisión |
| **Cuándo** | Siempre, por defecto | Sobre 20–100 candidatos | Legal, médico, alta precisión |

> [!important] 🎯 La conclusión operativa
> **El bi-encoder es el default por su combinación de calidad razonable, gran velocidad y almacenamiento mínimo.** El cross-encoder es el gold standard de calidad pero tan lento que no sirve para buscar — **y por eso se usa para re-rankear**. ColBERT ofrece casi la calidad del cross-encoder a velocidad casi de bi-encoder, pagando en memoria.

---

## 4. 🔄 Re-ranking

Audiencia: 🔧 🧭 👔

### 4.1 Qué es

**🔧 Definición técnica:** el **re-ranking** es un proceso **post-retrieval** en el que el conjunto inicial de documentos devuelto por la vector database se **vuelve a puntuar y reordenar** usando modelos de alto rendimiento pero costosos, **antes** de enviarlos al LLM.

La clave económica: **como solo hay que re-puntuar un puñado de documentos, es posible usar modelos caros que serían inviables para buscar en toda la knowledge base.**

**El ejemplo del curso** —admitidamente simplificado, pero clarísimo— con el prompt *"what is the capital of Canada?"*:

```
   La vector database devuelve documentos SEMÁNTICAMENTE relacionados
   que NO responden la pregunta:

     ✗ "Toronto is in Canada"
     ✗ "the capital of France is Paris"
     ✗ "Canada is the maple syrup capital of the world"
                                    ▲
              los tres comparten vocabulario y campo semántico
              con la query... y ninguno da la respuesta

                              │
                              ▼  RE-RANKER (cross-encoder)
                    re-puntúa mirando el PAR query-documento
                              │
                              ▼
              ✓ los verdaderamente relevantes suben al top
```

> [!tip] 💡 Por qué el bi-encoder falla justo aquí
> Los tres distractores están *cerca* de la query en el espacio vectorial porque comparten el tema (Canadá, capitales, geografía). Un bi-encoder compara **resumen contra resumen** y no puede notar que *"maple syrup capital"* usa "capital" en otro sentido, o que la frase sobre Francia responde **otra** pregunta.
>
> El cross-encoder lee la query y el documento **juntos**, y ahí sí la distinción es evidente. Es la mejor ilustración de qué compra exactamente esa arquitectura.

### 4.2 Over-fetch: el parámetro que hace que funcione

**🔧 Definición técnica:** en los sistemas con re-ranker se **recupera de más** (*over-fetch*) en la búsqueda inicial, para darle material al re-ranker.

```
   ① BÚSQUEDA (hybrid, bi-encoder)      → recupera 20–100 documentos
                    │
                    ▼
   ② RE-RANKER (cross-encoder)          → re-puntúa esos 20–100
                    │
                    ▼
   ③ SE DEVUELVEN                       → solo 5–10 al LLM

   Recomendación práctica del curso: over-fetch de 15–25 documentos
```

Gracias al re-ranker, esos 5–10 documentos finales son **mucho más relevantes** que los que habría devuelto una hybrid search simple.

> [!danger] 🚨 Sin over-fetch, el re-ranker no puede hacer su trabajo
> Esto es sutil y es **el error más frecuente al implementar re-ranking**: si recuperas 5 documentos y re-rankeas esos mismos 5, el re-ranker **solo puede reordenarlos** — nunca puede **rescatar** un documento relevante que quedó en el puesto 12 de la búsqueda inicial.
>
> ```
>   SIN over-fetch:   buscar 5 → rerankear 5 → devolver 5     ← solo reordena
>   CON over-fetch:   buscar 25 → rerankear 25 → devolver 5    ← rescata
> ```
>
> Y la mayor parte del valor del re-ranking está **justamente en el rescate**. (El assignment del curso comete este error — ver sección 5.3.)

**Sobre la latencia:** un cross-encoder agrega algo de latencia incluso re-rankeando solo 20–100 documentos. El veredicto del curso: *"este trade-off casi siempre vale la pena"*.

### 4.3 LLM-based re-ranking

**🔧 Definición técnica:** cada vez más se usa un **LLM** como re-ranker. La idea es similar al cross-encoder, pero en vez de pasar el par prompt-documento a un cross-encoder, se pasa **directamente a un LLM** entrenado para la tarea, que analiza el par, evalúa la relevancia y responde con un **score numérico**.

**Y comparte exactamente la misma limitación:** en ambos casos el scoring no puede empezar hasta que llega el prompt, y puntuar un documento individual sigue siendo costoso. Por eso el curso concluye que el LLM-based scoring *"podrá refinarse más, pero seguirá siendo una técnica de **re-ranking** que solo puede usarse después de que una búsqueda vectorial típica haya acotado la lista"*.

### 4.4 Por qué es la primera mejora que deberías probar

> [!important] 🎯 El argumento del curso
> *"Aunque un sistema RAG no requiere estrictamente re-ranking, a menudo es **bastante fácil de implementar** y produce **un rendimiento mucho mejor**. En muchas vector databases puede ser tan simple como **agregar una sola línea** a tu query indicando que quieres usar un re-ranker. Por eso, usar un re-ranker es una de las **primeras técnicas que deberías explorar** al intentar mejorar la relevancia de búsqueda."*
>
> Y el resumen operativo: **over-fetch de 15 a 25 documentos, re-rankear entre ellos** → gran mejora de relevancia al costo de un poco de latencia.

Y efectivamente, en Weaviate es una línea ([[Guia-Maestra-RAG_05-Vector-Databases-y-ANN#8. 🔄 Re-ranking: el adelanto|Tomo 05]]):

```python
from weaviate.classes.query import Rerank

response = collection.query.near_text(
    query="...",
    limit=5,
    rerank=Rerank(prop="chunk", query="...")     # ← toda la mejora, en una línea
)
```

---

> [!example] 📊 Caso de negocio — Legal: el buscador jurisprudencial que encontraba todo menos lo que servía
> **Problema:** un estudio jurídico monta un buscador sobre su archivo de jurisprudencia, doctrina y escritos propios. El retriever híbrido funciona: los abogados confirman que **los documentos relevantes aparecen entre los recuperados**. El problema es que aparecen en el puesto 7, el 9 y el 14 — y como al LLM se le pasan los primeros 5, **nunca llegan a la respuesta**. Además, los asociados junior consultan escribiendo el caso como se lo contó el cliente (*"al vecino le tiraron un muro y quiere que le paguen"*), lo que produce resultados muy distintos a cuando el socio busca con la terminología precisa.
>
> **Técnica aplicada:** las dos mejoras de los extremos, en el orden que recomienda el curso. **(1) Query rewriting** (sección 2.2): un LLM traduce la consulta coloquial a terminología jurídica y agrega sinónimos del dominio — el equivalente de la ficha médica del ejemplo de este tomo. Como el archivo está lleno de identificadores exactos (roles de causa, artículos, fechas), se busca con **la query original y la reescrita**, fusionando ambos rankings, para no perder el término literal. **(2) Re-ranking con over-fetch** (sección 4.2): en vez de recuperar 5 documentos, se recuperan **25** y un cross-encoder los re-puntúa leyendo cada par consulta-documento; solo entonces se recortan a los 5 finales. Ese over-fetch es lo que permite que el documento del puesto 14 **suba**, no solo que los 5 primeros se reordenen. Y sobre la mesa queda una tercera opción evaluada y postergada: **ColBERT**, que el propio curso señala como justificable en el ámbito legal precisamente porque el trade-off de memoria se paga con precisión.
>
> **Resultado:** los documentos que ya estaban siendo recuperados **empiezan a llegar a la respuesta**, sin cambiar el modelo generador, sin re-indexar el archivo y sin tocar el chunking. El costo es una llamada extra de LLM y unos milisegundos de re-ranking por consulta. La lección: **antes de asumir que tu retriever no encuentra los documentos, verifica si los encuentra y los ordena mal** — son dos problemas distintos, y el segundo es mucho más barato de arreglar.

---

## 5. 💻 El assignment: el retriever completo sobre Weaviate

Audiencia: 🔧

> [!note] Contexto del assignment graded C1M3
> **Dataset:** BBC News 2024, **75.256 chunks** (ojo: chunks, no artículos) ya pre-chunkeados.
> **Stack:** Weaviate embedded, embeddings `BAAI/bge-base-en-v1.5` (**768 dimensiones**), re-ranker `BAAI/bge-reranker-base`.
> **Cinco ejercicios**, uno por técnica de retrieval, y luego el pipeline RAG completo.

### 5.1 Los cinco retrievers

Todo el Módulo 2 y 3 condensado en cinco funciones:

```python
from weaviate.classes.query import Filter, Rerank

# ① Metadata filtering puro (Tomo 03)
collection.query.fetch_objects(
    filters=Filter.by_property(metadata_property).contains_any(values),
    limit=limit
)

# ② Semantic search (Tomo 04)
collection.query.near_text(query=query, limit=top_k)

# ③ BM25 / keyword (Tomo 03)
collection.query.bm25(query=query, limit=top_k)

# ④ Hybrid con RRF (Tomo 04)
collection.query.hybrid(query=query, alpha=alpha, limit=top_k)     # alpha=0.5

# ⑤ Semantic + re-ranking (este tomo)
collection.query.near_text(
    query=query,
    limit=top_k,
    rerank=Rerank(prop=rerank_property, query=rerank_query)
)
```

> [!note] `rerank_query` puede ser distinta de `query`
> El ejercicio 5 permite pasar una **segunda query solo para la etapa de re-ranking** (si se omite, usa la original). Es una capacidad real y poco conocida: puedes **buscar** con una formulación amplia y **re-rankear** con una más específica. No es query rewriting automático, pero es la misma idea aplicada a mano.

### 5.2 El resultado que vale todo el assignment

Para la query **`"Tell me about the last Taylor Swift show"`**, los tres retrievers devuelven cosas distintas:

| Retriever | Top-1 devuelto | ¿Correcto? |
|---|---|---|
| **Semantic** | *"'I've never had it this good' — Taylor Swift thanks fans after new Wembley record"* (Eras Tour) | ✅ **Exacto** |
| **BM25** | *"Killer Mike dismisses arrest at Grammys as 'speed bump'"* | ❌ **Irrelevante** |
| **Hybrid** (`alpha=0.5`) | Killer Mike, y **en segundo lugar** el de Taylor Swift | ⚠️ Mezcla |

> [!important] 🎯 Dos lecciones en una sola tabla
> **(1) El vocabulary mismatch, otra vez y al revés.** El [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN#6.6 Un hallazgo del lab que vale más que el código|Tomo 05]] mostró BM25 devolviendo *muy poco*; aquí lo muestra devolviendo *cualquier cosa*: un artículo sobre los Grammys, porque comparte palabras genéricas ("show", "last") sin tener nada que ver. **BM25 no entiende de qué habla el texto** — solo cuenta coincidencias ponderadas.
>
> **(2) Hybrid es una fusión, no una mejora automática.** Con `alpha=0.5` el sistema puso el resultado *malo* de BM25 en primer lugar y el *bueno* de semantic en segundo. Es exactamente la mecánica de RRF (fusiona posiciones, no calidad) del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#6.3 Reciprocal Rank Fusion (RRF)|Tomo 04]]: **si uno de los dos rankings es basura, la fusión lo arrastra al top.**
>
> Y aquí está el cierre elegante del tomo: **este es justamente el caso donde un re-ranker arregla el problema.** El documento correcto *está* entre los recuperados, solo está mal ordenado. Re-rankear el par query-documento con un cross-encoder pondría a Taylor Swift primero y a Killer Mike al fondo. El assignment tiene las dos piezas y **nunca las combina** (no hay `hybrid_retrieve` + `rerank`).
>
> El notebook no comenta ninguna de las dos cosas.

**Y otro dato de los tests del assignment**, con la query `"Conflicts in France"`: los tres retrievers devuelven **tres artículos completamente distintos** (uno sobre la lucha de poder post-electoral, otro sobre aviones del Día D, otro sobre un partido Francia–Israel). Es la evidencia más limpia de que las técnicas **no son intercambiables**.

### 5.3 Tres problemas del assignment

Audiencia: 🔧

> [!danger] 🚨 (1) El re-ranking se aplica SIN over-fetch — contradice la propia clase
> El ejercicio 5 aplica el re-ranker sobre `limit=top_k`:
>
> ```python
> collection.query.near_text(
>     query=query,
>     limit=top_k,                    # ← recupera exactamente lo que va a devolver
>     rerank=Rerank(...)
> )
> ```
>
> Con `top_k=2` se recuperan 2 documentos y se re-rankean esos 2. **El re-ranker solo puede reordenar; jamás puede rescatar** un documento relevante que quedó en el puesto 10 — que es de donde viene la mayor parte del valor del re-ranking (sección 4.2).
>
> La clase teórica del mismo módulo recomienda **over-fetch de 15–25 documentos**. El código del assignment no lo hace y no lo menciona. La versión correcta:
>
> ```python
> # Over-fetch, rerankear, y recortar al top_k final
> response = collection.query.near_text(
>     query=query,
>     limit=top_k * 5,                              # ← over-fetch (ej. 25 para top_k=5)
>     rerank=Rerank(prop=rerank_property, query=rerank_query),
>     return_metadata=MetadataQuery(score=True, rerank_score=True)   # ← para poder medirlo
> )
> return [o.properties for o in response.objects][:top_k]           # ← recortar después
> ```

> [!warning] ⚠️ (2) `alpha` nunca se varía, y no se piden scores
> `alpha` queda fijo en **0.5** en todo el notebook — nunca se prueba `alpha=0` ni `alpha=1` para mostrar el continuo BM25↔vectorial que la clase explicó. Y, igual que en los ungraded labs, **no se solicita metadata en ninguna query**: cero scores, cero distancias, cero `rerank_score`. Los "rankings" solo se infieren del orden de los objetos.
>
> Es la misma advertencia del [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN#6.5 El parámetro `alpha` de hybrid search|Tomo 05]]: **sin metadata estás tuneando a ciegas.**

> [!danger] 🚨 (3) El bug del prompt vuelve a aparecer — es un patrón
> El prompt final del assignment concatena f-strings sin separadores, produciendo literalmente:
>
> ```
>   ...add it to your overall knowledge.The news data is ordered by relevance.Query: Tell me...
>                                      ▲                                     ▲
>                                      └── sin espacio ni salto de línea ────┘
> ```
>
> **Es el mismo bug del assignment C1M1** que documentamos en el [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG#5. 💻 El pipeline completo en código|Tomo 02]] — y ahora en tres uniones en vez de una. Que se repita entre módulos lo convierte de descuido en **patrón a vigilar**: la delimitación entre instrucción y contenido es lo que el modelo usa para separar roles dentro del prompt.
>
> Y en el output final se ven las consecuencias: la query pedía *"provide links for the resources you use"* y **la respuesta no incluye ni una URL**, pese a que el prompt inyecta `URL:` en cada documento; la respuesta llega **truncada** por `max_tokens=500`; y su contenido deriva hacia Lula–Macron y los BRICS, es decir **material que no responde la pregunta** sobre EE.UU.–Brasil. Un prompt mal delimitado, sin instrucción de citar y con el presupuesto de tokens agotado produce exactamente eso.

> [!note] Detalle menor pero que rompe el código
> El *hint* del ejercicio 5 dice `reranker = Reranker(appropriate_parameters)`. **La clase no se llama `Reranker` sino `Rerank`** (`from weaviate.classes.query import Rerank`). Copiar el hint literalmente da `NameError`.

---

## 6. 🧭 Guía de decisión del tomo

Audiencia: 🔧 🧭

| Situación | Qué hacer |
|---|---|
| Vas a añadir **una sola** mejora al pipeline | **Re-ranking** (una línea) o **query rewriting** (una llamada) |
| Los usuarios escriben conversacionalmente | Query rewriting |
| Un reescritor agresivo borra términos exactos | Buscar con query original **y** reescrita, fusionar con RRF |
| Hay fechas, lugares o categorías en las queries | **NER** → convertirlas en filtros de metadata |
| Brecha grande pregunta↔documento, y el resto ya está optimizado | **HyDE** |
| El documento correcto se recupera pero mal ordenado | **Re-ranking con over-fetch** |
| Ya usas re-ranker y quieres más calidad | Subir el over-fetch; medir latencia |
| Dominio de alta precisión (legal, médico) y hay presupuesto de memoria | **ColBERT** |
| Quieres re-rankear con un LLM | Válido, pero misma limitación: solo post-búsqueda |
| **Siempre** | Over-fetch antes de re-rankear, y **pedir los scores** |

> [!tip] 🧭 El orden correcto de trabajo
> Retriever híbrido funcionando → **re-ranking con over-fetch** (la mejora más barata) → **query rewriting** → medir → y solo entonces evaluar NER, HyDE o ColBERT según lo que la medición muestre que falta. Las tres primeras cubren la mayor parte del terreno; las últimas tres son especialización.

---

## 7. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Query parsing** | Limpiar y arreglar la pregunta del usuario antes de buscar |
| **Query rewriting** | Que un LLM reescriba la pregunta para que sea buscable |
| **NER** | Detectar automáticamente nombres, fechas y lugares en la pregunta |
| **HyDE** | Inventar el documento ideal y buscar con él, en vez de con la pregunta |
| **Bi-encoder** | Comparar dos resúmenes. Rápido y el default para buscar |
| **Cross-encoder** | Leer pregunta y documento juntos. El mejor juez, demasiado lento para buscar |
| **ColBERT** | Un vector por palabra: casi la calidad del cross-encoder, casi la velocidad del bi-encoder |
| **MaxSim** | Cómo puntúa ColBERT: cada palabra de la pregunta busca su mejor pareja |
| **Re-ranking** | Reordenar los resultados con un modelo mejor antes de dárselos al LLM |
| **Over-fetch** | Traer más resultados de los que necesitas para que el re-ranker tenga con qué trabajar |
| **`rerank_score`** | La nota que el re-ranker le puso a cada documento. Sin pedirla, no ves nada |

---

## 8. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Puedo ubicar query parsing, búsqueda y re-ranking en el pipeline y decir qué hace cada uno.
- [ ] Entiendo por qué un prompt humano es una mala query de búsqueda.
- [ ] Sé qué cuatro operaciones hace un query rewriter y por qué el curso lo recomienda antes que todo lo demás.
- [ ] Conozco los tres riesgos del query rewriting (latencia, punto de fallo, pérdida de términos exactos).
- [ ] Sé para qué sirve NER en un retriever y cuál es su limitación genuina.
- [ ] Puedo explicar HyDE y por qué "comparar manzanas con manzanas" mejora el retrieval.
- [ ] Tengo claro que en HyDE el documento hipotético **reemplaza** a la query y que sus errores factuales no importan.
- [ ] Sé por qué la separación del bi-encoder es a la vez su virtud y su límite.
- [ ] Entiendo por qué un cross-encoder es mejor juez y por qué no puede usarse para buscar.
- [ ] Puedo explicar cómo ColBERT puntúa con MaxSim y qué paga por ello.
- [ ] Sé elegir entre las tres arquitecturas según calidad, velocidad y almacenamiento.
- [ ] Entiendo el ejemplo de la capital de Canadá y qué revela sobre los bi-encoders.
- [ ] **Sé que sin over-fetch el re-ranker solo reordena y nunca rescata.**
- [ ] Conozco los números del curso: over-fetch 15–25 (o 20–100), devolver 5–10.
- [ ] Entiendo por qué hybrid puede arrastrar un mal resultado al primer puesto, y cómo el re-ranking lo arregla.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] (qué se indexa)
- Siguiente tomo → [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación]] (arranca el Módulo 4: qué hace el LLM con lo recuperado)
- Dónde se ejecuta el re-ranking en una línea → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases]]
- Hybrid search, RRF y `alpha`/`beta` → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]]
- Metadata filtering, al que NER alimenta → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]]
- El bug del prompt que reaparece → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]]
- **Medir si todo esto sirve** → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Evaluación]]
- Query decomposition, multi-query y GraphRAG → [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12 · ⭐ Técnicas avanzadas de query]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 3: lecciones de **query parsing**, **bi-encoders / cross-encoders / ColBERT** y **re-ranking**; **assignment graded C1M3** (los cinco retrievers sobre Weaviate y el pipeline RAG completo), del que provienen el código y los resultados de la sección 5.

**Fuentes externas (complemento con bibliografía verificable):**
- Gao, L., Ma, X., Lin, J. & Callan, J. (2023). *Precise Zero-Shot Dense Retrieval without Relevance Labels*. ACL. — El paper de **HyDE**.
- Khattab, O. & Zaharia, M. (2020). *ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT*. SIGIR. — El paper de **ColBERT** y del scoring MaxSim.
- Santhanam, K., Khattab, O., Saad-Falcon, J., Potts, C. & Zaharia, M. (2022). *ColBERTv2: Effective and Efficient Retrieval via Lightweight Late Interaction*. NAACL. — La versión que reduce drásticamente el costo de almacenamiento, el principal problema de ColBERT.
- Nogueira, R. & Cho, K. (2019). *Passage Re-ranking with BERT*. arXiv:1901.04085. — El trabajo que estableció el cross-encoder como re-ranker.
- Reimers, N. & Gurevych, I. (2019). *Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks*. EMNLP. — Origen de la distinción bi-encoder / cross-encoder.
- Zaratiana, U., Tomeh, N., Holat, P. & Charnois, T. (2024). *GLiNER: Generalist Model for Named Entity Recognition using Bidirectional Transformer*. NAACL. — El modelo de NER que usa el curso.
- Documentación oficial de **Weaviate**: `Rerank`, `MetadataQuery`, `query.hybrid`, y el módulo `reranker-transformers`.

> [!note] Sobre el código y los valores de este tomo
> - **Del curso:** toda la teoría de las secciones 2 a 4 (query rewriting con su prompt y su ejemplo médico, NER/GLiNER, HyDE, las tres arquitecturas, MaxSim, re-ranking, over-fetch 15–25 / 20–100 → 5–10) proviene de las tres lecciones del Módulo 3. El código y los resultados de la sección 5 son literales del assignment graded C1M3.
> - **Hallazgos propios verificados contra el notebook:** que el ejercicio 5 aplica re-ranking **sin over-fetch**, contradiciendo la recomendación de la propia clase; que `alpha` nunca se varía y nunca se pide metadata; que el **bug de separadores en el prompt reaparece** desde C1M1, ahora en tres uniones; que la respuesta final ignora la instrucción de citar URLs, se trunca por `max_tokens` y deriva a material off-topic; y que el hint del ejercicio 5 nombra una clase inexistente (`Reranker` en vez de `Rerank`).
> - **Análisis propio:** la lectura de la tabla de 5.2 (que hybrid arrastró el mal resultado de BM25 al primer puesto **y que ese es exactamente el caso que un re-ranker arregla** — combinación que el assignment nunca hace), los tres riesgos del query rewriting, y las guías de decisión.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación: transformers, sampling y prompt engineering]]**, donde arranca el **Módulo 4** y el foco se corre por fin del retriever al generador: qué hace el LLM con todo lo que tanto trabajo costó recuperar, cómo se controla su salida, y cómo elegir el modelo.
