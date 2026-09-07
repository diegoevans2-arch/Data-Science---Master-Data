---
title: "Tomo 06 — Chunking (básico y avanzado)"
tags: [rag, chunking, fixed-size, overlap, recursive-splitting, semantic-chunking, contextual-retrieval, indexing]
audiencias: [tecnico, puente, ejecutivo]
tomo: 06
version: 1.2
updated: 2026-09-05
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 06 — Chunking (básico y avanzado)

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]] · Siguiente → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing, arquitecturas y re-ranking]]

---

> [!info] ¿Por qué importa esta sección?
> El [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]] resolvió **cómo** buscar entre millones de vectores. Este tomo retrocede un paso y responde algo anterior: **¿qué es, exactamente, cada uno de esos vectores?** Porque un vector por libro y un vector por párrafo producen sistemas radicalmente distintos.
>
> El chunking es, junto con el retriever híbrido, **la decisión que más mueve la calidad de un RAG** — y la más fácil de subestimar, porque parece un detalle de preprocesamiento. No lo es: define el grano de todo lo que el sistema podrá encontrar y de todo lo que el LLM llegará a leer.

> [!abstract] 👔 Impacto ejecutivo
> Chunking es dónde se decide si tu asistente responde con **el párrafo exacto** que resuelve la duda o con **veinte páginas** donde la respuesta está enterrada. Es una decisión de calidad de producto disfrazada de detalle técnico.
> - **Decisiones que habilita:** controlar el costo por consulta (menos contexto = menos tokens = menos factura); mejorar la precisión de las respuestas sin cambiar de modelo; hacer que las citas apunten a un pasaje útil y no a un documento entero.
> - **Costo o riesgo de hacerlo mal:** chunks demasiado grandes diluyen el significado y llenan el context window; demasiado pequeños pierden el contexto y devuelven fragmentos ininteligibles. Y hay un fallo peor: **indexar documentos sin partirlos hace que todo lo que exceda el límite del embedding model sea invisible para el retriever** — presente en la base, irrecuperable en la práctica.
> - **Pregunta ejecutiva que responde:** *"¿por qué el asistente encuentra el documento correcto pero responde con la parte equivocada?"*

---

## 1. 🎯 Las tres razones para chunkear

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** **chunking** es la práctica de partir los documentos largos de la knowledge base en fragmentos de texto más pequeños (**chunks**). El curso da tres razones, y conviene tenerlas separadas porque apuntan a problemas distintos:

```
   ┌─────────────────────────────────────────────────────────────────┐
   │  1. TOKEN LIMITS                                                │
   │     Muchos embedding models tienen un límite de texto que       │
   │     pueden embeber. Lo que sobra se descarta EN SILENCIO.       │
   │                                                                 │
   │  2. IMPROVED RELEVANCY                                          │
   │     Chunks bien dimensionados mejoran las métricas de           │
   │     búsqueda del retriever.                                     │
   │                                                                 │
   │  3. SOLO EL CONTEXTO RELEVANTE AL LLM                           │
   │     Se le envía el pasaje que importa, no el documento entero.  │
   └─────────────────────────────────────────────────────────────────┘
```

> [!important] La razón nº1 ya la vimos, y era grave
> El [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#5.4 ⚠️ La trampa del truncation|Tomo 04]] demostró que `model.encode(texto)` y `model.encode(texto[:3000])` devuelven **el mismo vector, byte a byte** — el modelo ignora todo lo que pase de sus 512 tokens sin error ni warning. **El chunking es la solución a ese problema.** No es una optimización: es lo que evita que la mitad de tu corpus sea invisible.

### 1.1 El experimento mental de los mil libros

Audiencia: 🔧 🧭 💡

El curso lo plantea así y es la mejor forma de entenderlo:

```
   SIN CHUNKING                          CON CHUNKING
   ─────────────                         ────────────
   1.000 libros                          1.000 libros
        │                                     │
        ▼                                     ▼
   1.000 vectores                        1.000.000 de párrafos
   (uno por libro)                       (mil vectores por libro)
        │                                     │
   ✗ el significado de un libro          ✓ cada vector representa
     ENTERO comprimido en un               UN tema concreto
     solo vector → promedia todos
     los temas de todos los capítulos    ✓ recuperas el párrafo,
                                           no el libro
   ✗ recuperar = traer un libro
     completo → llena el context         ✓ las vector databases
     window del LLM                        escalan sin problema a
                                           millones de vectores
```

> [!tip] 💡 Analogía
> Es la diferencia entre un catálogo de biblioteca que solo tiene **una ficha por libro** y uno que tiene **una ficha por párrafo**. Con el primero puedes preguntar "¿tienen algo de cocina italiana?"; con el segundo puedes preguntar "¿cuánto tiempo se levanta la masa de pizza napolitana?" y que te lleven a la línea exacta. El libro entero comprimido en una ficha es un promedio de todo lo que dice — y **el promedio de muchos temas no se parece a ninguno**.

**👔 En una frase para el negocio:** el tamaño del chunk decide el nivel de detalle con que tu sistema es capaz de responder; ningún modelo, por bueno que sea, puede recuperar una precisión que el índice no tiene.

---

## 2. 📏 El tamaño del chunk: el dilema central

Audiencia: 🔧 🧭

**🔧 La primera consideración es el tamaño**, y falla por los dos extremos:

```
  DEMASIADO GRANDE                                    DEMASIADO PEQUEÑO
  (nivel capítulo o libro)                            (nivel palabra)
  ────────────────────────                            ─────────────────
  ✗ un solo vector no captura                         ✗ el vector pierde TODO el
    el significado matizado                             contexto de la frase y el
                                                        párrafo que lo rodean
  ✗ llena rápido el context
    window del LLM                                    ✗ la relevancia de búsqueda
                                                        cae igual
  ✗ el vector "promedia" muchos
    temas distintos                                   ✗ incluso a nivel FRASE puede
                                                        ser demasiado fino

              ╲                                              ╱
               ╲                                            ╱
                ╲──────────  EL EQUILIBRIO  ──────────────╱
                     ni demasiado ni demasiado poco
                       contexto en un solo vector
```

> [!warning] ⚠️ No existe un tamaño universal
> El curso es explícito: *"no hay un enfoque de talla única para el tamaño del chunk"*. Lo que existe es un **equilibrio a encontrar con tus datos** — y una recomendación de arranque (sección 3.4) para no quedarse paralizado.

**El comentario del propio lab** resume el dilema con precisión: los chunks pequeños son muy detallados pero *"pueden no tener suficiente información para ser útiles en la búsqueda"*; y a medida que los chunks crecen, *"sus vector embeddings asociados se vuelven más generales"* hasta dejar de servir.

---

## 3. 🔨 Estrategias básicas

Audiencia: 🔧

### 3.1 Fixed-size chunking

**🔧 Definición:** la forma más simple. Se decide de entrada que **todos los chunks tendrán el mismo tamaño**. Con 250 caracteres:

```
   Documento: ───────────────────────────────────────────────────►

   Chunk 1: [ chars    1 – 250 ]
   Chunk 2:                [ chars  251 – 500 ]
   Chunk 3:                              [ chars  501 – 750 ]
   ...                                                 y así hasta el final
```

**El problema es obvio:** no hay ninguna garantía de que los cortes caigan en lugares sensatos. El corte suele caer **en medio de una palabra** o separar dos partes de una idea que iba junta.

Y esto no es teórico — es exactamente lo que produce el lab del curso. Con `chunk_size = 100` palabras sobre un capítulo de **1.403 palabras** (→ **15 chunks**), el corte entre el chunk 1 y el 2 queda así:

```
   Chunk 1: "...Git stores and thinks about information in"
   Chunk 2: "a very different way, ..."
                    ▲
                    └── la frase se partió por la mitad
```

### 3.2 Overlap: la corrección estándar

**🔧 Definición:** se permite que los chunks **se solapen**. Los chunks miden 250 caracteres pero comparten 25 con el anterior y el siguiente:

```
   Chunk 1 │ chars    1 – 250 │
   Chunk 2         │ chars  226 – 475 │
   Chunk 3                  │ chars  451 – 700 │
                     ▲
                     └── 25 caracteres compartidos = 10 % de overlap
```

El overlap se expresa como **porcentaje del chunk** (aquí, 10 %).

**Qué gana:** minimiza los casos donde una palabra queda cortada de su contexto. Las palabras del **medio** de un chunk tienen contexto a ambos lados; las de los **bordes** aparecen en **dos chunks**, lo que aumenta las probabilidades de que en alguno estén junto al contexto relevante.

**Qué cuesta:** más overlap suele mejorar la relevancia de búsqueda, pero **agrega más vectores con información redundante** a la base.

> [!danger] ⚠️ El lab implementa el overlap de forma NO estándar — y el efecto es medible
> Esto vale documentarlo porque cambia los números y es fácil de copiar sin notarlo. La implementación del lab **prepende** el overlap y avanza con un paso (*stride*) igual al `chunk_size` completo:
>
> ```python
> for i in range(0, len(text_words), chunk_size):          # el paso es chunk_size
>     chunk_words = text_words[max(i - overlap_int, 0): i + chunk_size]
> ```
>
> Consecuencias verificadas en los outputs del propio lab:
>
> | | Implementación estándar | La del lab |
> |---|---|---|
> | Tamaño real del chunk | `chunk_size` | **`chunk_size + overlap`** |
> | Nº de chunks vs. sin overlap | **Aumenta** | **Idéntico** |
> | Primer chunk | lleva overlap | **no lleva** |
>
> Con `chunk_size=100, overlap=0.2` el lab produce **15 chunks de 120 palabras** (no de 100), los mismos 15 que sin overlap. Con `chunk_size=25` produce 57 chunks de **30** palabras. Por eso, cuando el notebook rotula las estrategias como *"chunks de 25 palabras"* y *"de 100 palabras"*, **los chunks reales miden 30 y 120**. El `chunk_size` del lab es el **paso**, no el tamaño.
>
> La implementación habitual (la de LangChain y similares) usa `stride = chunk_size − overlap`, lo que mantiene el tamaño en `chunk_size` y **aumenta** el número de chunks. Ninguna de las dos está "mal", pero **hay que saber cuál estás usando** para interpretar los tamaños y el costo de almacenamiento.

### 3.3 Recursive character text splitting

**🔧 Definición:** una estrategia más dinámica. En vez de cortar cada N caracteres, se elige **un carácter por el cual partir** — por ejemplo el salto de línea, que suele separar párrafos.

```
   Texto (AsciiDoc / Markdown / prosa)          Chunks resultantes
   ──────────────────────────────────           ──────────────────
   "Taylor Swift is performing three
    sold-out shows in Vancouver this      ──►   Chunk 1  (párrafo 1)
    weekend as part of her Eras Tour."

   "Thousands of fans are traveling to
    the city, causing hotel demand to     ──►   Chunk 2  (párrafo 2)
    spike."

   "Local hotels are fully booked, and
    prices have more than doubled since   ──►   Chunk 3  (párrafo 3)
    last month."
```

| ✅ Ventaja | ⚠️ Costo |
|---|---|
| **Respeta la estructura del documento** → más probable que los conceptos relacionados queden juntos en un mismo chunk | **Tamaño variable** → riesgo de chunks muy grandes o muy pequeños según dónde caigan los saltos de línea |

**🔧 Y se adapta al tipo de documento.** Si la knowledge base es heterogénea, se parte cada tipo por donde tiene sentido:

| Tipo de documento | Separador natural |
|---|---|
| HTML | tags de párrafo o de header |
| Código Python | definiciones de función |
| Texto plano | saltos de línea |

> [!note] ⚠️ El nombre "recursive" promete algo que el lab no hace
> La sección del lab se titula *Recursive Character Splitting* y muestra una imagen con ese nombre, pero **el código solo hace un `split()` por un único separador** (`"\n\n"` o `"\n=="`): no hay cascada de separadores ni fallback por tamaño.
>
> El splitter *recursivo* de verdad —el `RecursiveCharacterTextSplitter` de LangChain, que es a lo que alude el nombre— funciona distinto y merece conocerse: intenta partir por una **lista ordenada** de separadores (`["\n\n", "\n", " ", ""]`), y **solo si el chunk resultante sigue siendo demasiado grande** baja al siguiente separador. Así respeta la estructura *y* acota el tamaño, que es justo la debilidad de un split simple. Es el default sensato en la práctica y no aparece en el lab.

### 3.4 El punto de partida recomendado

> [!tip] 🧭 Si no sabes por dónde empezar, el curso te da un número
> **Chunks de tamaño fijo de ~500 caracteres, con un overlap de 50 a 100 caracteres.** Es un default razonable para prototipar, y evita quedarse paralizado optimizando antes de tener un baseline.
>
> Ojo con las unidades: el curso recomienda **caracteres**, pero muchas librerías cuentan **tokens** y el código del lab cuenta **palabras**. Como referencia gruesa en inglés: 500 caracteres ≈ 100 palabras ≈ 125 tokens. Fija la unidad explícitamente en tu configuración o los números no significan nada.

### 3.5 Los chunks heredan la metadata

**🔧 Regla práctica:** si tus documentos tienen metadata, **los chunks deben heredarla** — quizás con información adicional sobre su ubicación en el documento original. Es lo que permite después filtrar por permisos, fecha o categoría ([[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]]) y citar la fuente con precisión.

Así lo hace el lab:

```python
def build_chunk_objs(book_text_obj, chunks):
    chunk_objs = []
    for i, c in enumerate(chunks):
        chunk_objs.append({
            "chapter_title": book_text_obj["chapter_title"],   # heredado
            "filename":      book_text_obj["filename"],        # heredado
            "chunk":         c,                                 # el texto
            "chunk_index":   i,                                 # ← posición en el doc
        })
    return chunk_objs
```

Ese `chunk_index` es la pieza que permite reconstruir el orden, traer chunks vecinos si hace falta, y citar *"párrafo 4 del documento X"* en vez de solo *"documento X"*.

---

## 4. 🧠 Estrategias avanzadas

Audiencia: 🔧 🧭

El chunking tiene beneficios claros, pero al partir los documentos **también corre el riesgo de romper el texto de una forma que pierde el contexto relevante**. El curso lo ilustra con un ejemplo memorable:

> [!warning] ⚠️ El ejemplo del sueño olímpico
> `"That night she dreamed, as she did often, that she was finally an Olympic champion."`
>
> Según dónde caiga el corte, el chunk puede hacer parecer que **nuestra soñadora ya es medallista de oro**, en vez de estar soñando con una gloria futura. Fixed-size y recursive character splitting **no ofrecen ninguna protección** contra este tipo de error.

Las técnicas avanzadas intentan construir chunks **según el significado** del texto.

### 4.1 Semantic chunking

**🔧 Definición técnica:** agrupa frases en un mismo chunk **si tienen significado similar**. El algoritmo recorre el documento frase por frase:

```
   Para cada frase:
     1. Vectorizar el CONTENIDO ACTUAL del chunk
     2. Vectorizar la SIGUIENTE frase
     3. ¿La distancia entre ambos vectores está bajo un UMBRAL?
          SÍ  → significados similares → la frase se AGREGA al chunk
          NO  → se CIERRA el chunk y se empieza uno nuevo desde esa frase
```

Visualmente, es lo que hace el gráfico del curso:

```
   disimilitud
        │                              ╱╲          ← cruza el umbral:
   ─────┼─────────────────────────────╱──╲───────    se corta el chunk
 umbral │        ╱╲      ╱╲          ╱    ╲
        │   ╱╲  ╱  ╲    ╱  ╲   ╱╲   ╱      ╲
        │  ╱  ╲╱    ╲  ╱    ╲ ╱  ╲ ╱        ╲
        └──────────────────────────────────────────► frases
         │◄────────── chunk 1 ──────────►│◄ chunk 2
```

**Qué gana:** chunks de tamaño variable que **siguen el hilo de pensamiento del autor**. Si el autor se va por una tangente conceptual dentro de un párrafo, o desarrolla la misma idea a lo largo de dos párrafos seguidos, semantic chunking pone los cortes en los lugares correctos.

**Qué cuesta:** es **computacionalmente caro** — hay que calcular vectores repetidamente para cada frase de la knowledge base. En cambio, suele dar retrieval de alta calidad medido con las métricas habituales de precision y recall.

### 4.2 LLM-based chunking

**🔧 Definición técnica:** se le entrega el documento a un LLM junto con instrucciones sobre el tipo de chunks que se quieren. Por ejemplo: *separa los chunks según el significado, mantén los conceptos similares juntos, y separa el texto cuando se discutan temas nuevos.* El modelo genera los chunks como generaría cualquier otro texto.

Es inherentemente un enfoque de **caja negra**, pero **de muy alto rendimiento**. Y el curso añade una observación económica relevante: *a medida que bajan los costos de los LLMs, el LLM-based chunking se vuelve cada vez más viable*.

### 4.3 Context-aware chunking: la mejora que aplica sobre cualquier estrategia

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** usar un LLM para **agregar contexto a cada chunk**. No es una estrategia de partición alternativa: es una capa que se **suma a cualquiera** de las anteriores. Se le pide al modelo que, además de crear el chunk, le añada un texto de resumen que explique **su contexto en el documento completo**.

> [!tip] 💡 El ejemplo del curso: la lista de agradecimientos
> Un autor cierra un post de blog agradeciendo a colaboradores. Eso produce un chunk que es **solo una lista de nombres** — imposible de interpretar por sí solo, e imposible de recuperar con sentido. El LLM le añade una línea explicando qué es y de qué post viene.
>
> Y aquí está lo elegante: ese texto añadido está disponible **dos veces**:
> 1. **Al vectorizar** el chunk → mejora la relevancia de búsqueda.
> 2. **Al recuperarlo** → ayuda al LLM a entender el significado global del fragmento.

**El trade-off es inusualmente bueno:** requiere preprocesamiento costoso (un LLM debe recorrer toda la knowledge base, documento por documento y chunk por chunk), pero a cambio da búsquedas más relevantes y **esencialmente ningún impacto en la velocidad de búsqueda** — porque todo el costo se pagó offline.

> [!note] 🔬 Complemento de vanguardia — esto tiene nombre propio en la industria
> La técnica que el curso describe como *context-aware chunking* se popularizó como **Contextual Retrieval** (Anthropic, 2024), y el paper de referencia reporta reducciones sustanciales en la tasa de fallo de retrieval al combinar chunks contextualizados con BM25 y re-ranking. Es una de las mejoras con mejor relación esfuerzo/beneficio del ecosistema actual, y encaja exactamente con el consejo del curso de la sección 4.4.

### 4.4 Cuándo usar cuál

Audiencia: 🧭 👔

El curso cierra con un criterio que vale más que la lista de técnicas:

| Estrategia | Costo | Cuándo conviene |
|---|---|---|
| **Fixed-size / recursive** | Muy bajo | **Prototipado y default razonable.** Buen punto de partida siempre |
| **Semantic chunking** | Alto (vectorizar cada frase) | Cuando la medición demuestre que el chunking simple limita la calidad |
| **LLM-based** | Alto (y caja negra) | Alto rendimiento; cada vez más viable a medida que bajan los costos |
| **Context-aware** | Alto offline, **cero en query time** | **La primera mejora a explorar** más allá de lo básico |

> [!important] 🎯 La frase del curso que hay que llevarse
> *"Como diseñador de sistemas RAG, el objetivo no es implementar la técnica de chunking más vanguardista del mercado. Es entender qué opciones existen, cuán adecuadas son para tus datos, y si los costos y beneficios justifican implementarla."*
>
> Y el corolario operativo: **experimenta con un subconjunto pequeño de tus datos** y comprueba si las técnicas avanzadas realmente mejoran la relevancia antes de comprometerte con el costo.

### 4.5 Late chunking y contextual chunking — avances post-curso

Audiencia: 🔧 🧭

**Late chunking (Jina AI, 2024):**
- El approach estándar: chunking → embedding (cada chunk se embeddea por separado, pierde contexto del documento)
- Late chunking invierte: se pasa el DOCUMENTO COMPLETO por el modelo de embedding (hasta el context window del encoder), y DESPUÉS se segmenta el pool de token embeddings en chunks. Así cada chunk "sabe" en qué documento estaba.
- Ventaja: los chunks heredan el contexto semántico del documento sin necesidad de overlap ni prepending de metadata
- Limitación: requiere modelos de embedding que soporten documentos largos (jina-embeddings-v3 soporta 8192 tokens); no escala a documentos de 100 páginas sin truncar
- Cuándo usarlo: documentos medianos (< 8K tokens) donde el contexto inter-chunk es crítico

**Contextual chunking (Anthropic, 2024 — "Contextual Retrieval"):**
- El problema: un chunk como "La empresa facturó $2M en Q3" pierde sentido sin saber QUÉ empresa
- La solución: antes de embeddear, un LLM (generalmente barato/rápido) genera una frase de contexto corta para cada chunk: "Este chunk proviene del informe financiero anual 2024 de Acme Corp, sección de resultados trimestrales."
- Esa frase se prepend al chunk antes de embeddear Y antes de pasar al LLM generador
- Resultado reportado por Anthropic: reducción de ~49% en retrieval failures al combinarlo con hybrid search (BM25 + semantic)
- Trade-off: costo de la llamada al LLM para generar contexto × N chunks. Mitigación: usar un modelo barato (Haiku, GPT-4o-mini) y cachear
- Cuándo usarlo: knowledge bases donde los chunks pierden sentido sin el contexto del documento padre (informes financieros, contratos, manuales técnicos largos)

**Tabla comparativa:**
| Approach | Mecanismo | Costo extra | Mejora típica en retrieval | Mejor para |
|---|---|---|---|---|
| Overlap estándar (§3.2) | Compartir tokens entre chunks vecinos | ~0 (solo storage) | Marginal | Baseline, textos narrativos |
| Metadata prepending (§3.5) | Agregar título/sección al inicio del chunk | ~0 | Moderada | Documentos con estructura clara |
| Late chunking | Embedding full-doc → segment pool | ~0 (solo inference time) | Significativa en docs medianos | Docs < 8K tokens con alta interdependencia |
| Contextual chunking | LLM genera contexto por chunk | Alto (1 LLM call / chunk) | ~49% menos retrieval failures | KB heterogénea, docs largos, alto impacto de errores |

> [!quote] 📚 Bibliografía
> - Günther, M., Mohr, I., Williams, D. J., Wang, B., & Xiao, H. (2024). *"Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models"*. arXiv:2409.04701 (preprint, Jina AI). *(Título corregido el 2026-09-05: "Embeddings", no "Representations".)*
> - Anthropic (2024, 19 de septiembre). *"Introducing Contextual Retrieval"*. Blog post.

---

## 5. 💻 El experimento del lab: cuatro estrategias sobre el mismo corpus

Audiencia: 🔧 🧭

> [!note] Contexto del lab
> **Corpus:** el libro **Pro Git** (formato AsciiDoc), secciones de los capítulos 1 y 2 → **14 documentos**.
> **Capítulo de la demo:** `what-is-git.asc`, **1.403 palabras** (≈1.824 tokens).
> **Embeddings:** `BAAI/bge-base-en-v1.5`. **Vector DB:** Weaviate embedded.
> **Las cuatro estrategias se indexan a la vez** en la misma collection, etiquetadas con un campo `chunking_strategy`, lo que permite filtrar por estrategia y comparar.

### 5.1 Las cuatro estrategias y sus números reales

```python
for strategy_name, chunks in [
    ["fixed_size_25",      get_chunks_fixed_size_with_overlap(text, 25, 0.2)],
    ["fixed_size_100",     get_chunks_fixed_size_with_overlap(text, 100, 0.2)],
    ["para_chunks",        get_chunks_by_paragraph(text)],        # split("\n\n")
    ["para_chunks_min_25", mixed_chunking(text)]                 # split("\n==") + fusión
]:
```

Conteos reales en la vector database, para los mismos 14 documentos:

| Estrategia | Chunks | Media por documento | Tamaño real del chunk |
|---|---:|---:|---|
| `fixed_size_25` | **672** | 48 | 25 → **30** palabras (con overlap) |
| `fixed_size_100` | **173** | 12,4 | 100 → **120** palabras |
| `para_chunks` | **549** | 39,2 | variable (¡desde 4 palabras!) |
| `para_chunks_min_25` | **93** | 6,6 | variable (hasta **1.296** palabras) |
| **Total** | **1.487** | | |

> [!abstract] 👔 Lo que esa tabla significa en dinero
> La misma knowledge base produce entre **93 y 672** vectores según la estrategia — un factor de **7×** en costo de almacenamiento, de indexado y de memoria del índice. Y no es que "más sea mejor": son sistemas con comportamientos distintos. **La estrategia de chunking es una decisión de infraestructura, no solo de calidad.**

### 5.2 El resultado del experimento: no hay ganador único

El lab lanza dos queries de naturaleza distinta y ahí está la lección del tomo:

**Query A — amplia: `"history of git"`**

| Estrategia | Qué recuperó |
|---|---|
| `fixed_size_25` | 25 palabras: el encabezado *"A Short History of Git"* y **corta justo antes del contenido útil** |
| `fixed_size_100` | 100 palabras: ya incluye la cronología 1991–2002, BitKeeper, la ruptura de 2005 |
| `para_chunks` | Un párrafo (42 palabras) y **una sola frase de 18 palabras** |
| `para_chunks_min_25` | La **sección completa** (218 palabras) con cronología y objetivos de diseño |

→ **Ganan los chunks largos.** El propio lab lo dice: *"los resultados muestran que los chunks más largos tienden a rendir mejor"*; los de 25 palabras casan semánticamente pero *"carecen de contexto suficiente"*.

**Query B — específica: `"how to add the url of a remote repository"`**

| Estrategia | Qué recuperó |
|---|---|
| `fixed_size_25` | 30 palabras que contienen **exactamente la respuesta**: `` git remote add <shortname> <url> `` |
| `para_chunks` | Como resultado **nº2**: el encabezado suelto `"==== Adding Remote Repositories"` — **4 palabras, inútil** |
| `para_chunks_min_25` | Secciones de 226 y 264 palabras: la respuesta está ahí, enterrada |

→ **Ganan los chunks cortos.** Weaviate localiza el pasaje exacto, y el lab añade una observación que suele olvidarse: los resultados largos *"pueden requerir más esfuerzo cognitivo del usuario"*.

> [!important] 🎯 La conclusión del experimento
> **No existe una estrategia ganadora: depende del tipo de query que reciba tu sistema.**
>
> ```
>    Queries AMPLIAS / conceptuales   →  chunks LARGOS
>    ("háblame de la historia de X")     (contexto completo)
>
>    Queries ESPECÍFICAS / puntuales  →  chunks CORTOS
>    ("cómo se ejecuta el comando Y")    (el pasaje exacto)
> ```
>
> Y de ahí sale la consecuencia práctica: **para elegir tu chunking necesitas saber qué te van a preguntar.** Si tu sistema recibe ambos tipos de query —lo habitual—, la respuesta suele ser una estrategia intermedia, o chunks pequeños con contexto añadido (4.3), o recuperar chunks vecinos al recuperar uno.

### 5.3 El gotcha de los encabezados

Audiencia: 🔧

Partir por un separador produce basura, y el lab lo demuestra con datos:

| Separador | Chunks | Problema observado |
|---|---:|---|
| `"\n\n"` (párrafo) | 31 | El chunk nº3 es **solo un encabezado**: `"==== Snapshots, Not Differences"` |
| `"\n=="` (sección AsciiDoc) | 7 | El chunk nº1 es `"[[what_is_git_section]]"` — **un ancla huérfana de un token** |

Y no es un problema cosmético: en la Query B, `para_chunks` devolvió **un encabezado de 4 palabras como resultado nº2**, ocupando un lugar del `top_k` con un objeto inútil.

> [!warning] ⚠️ Reglas de higiene que el lab no aplica y tú sí deberías
> El lab indexa todo lo que el `split()` produce, incluidos strings vacíos, encabezados sueltos y anclas. Antes de indexar:
> - **Descarta chunks triviales** (bajo un mínimo de palabras/tokens).
> - **Fusiona los encabezados con el chunk siguiente** — el propio lab lo sugiere en el texto, aunque su implementación lo hace mal (ver abajo).
> - **Normaliza el whitespace** y elimina artefactos del formato (anclas, marcadores).
>
> Un chunk basura no solo es inútil: **desplaza a un chunk útil** del `top_k`.

### 5.4 Tres bugs del lab que valen como lección

Audiencia: 🔧

> [!danger] 🚨 (1) `mixed_chunking` promete partir chunks grandes y nunca lo hace
> Su docstring dice que *"los chunks más grandes pueden partirse por la mitad o por marcadores internos"*. **El código solo fusiona chunks pequeños; nunca divide los grandes.** Resultado medible: la estrategia `para_chunks_min_25` recuperó un chunk de **1.296 palabras / 8.975 caracteres** — el problema exacto que el chunking venía a resolver.
>
> Y además concatena sin separador (`chunk_buffer + chunk`), produciendo salidas como `"[[what_is_git_section]]= What is Git?"`, y como `split("\n==")` consume el separador, los encabezados **pierden dos `=`** (`=== What is Git?` → `= What is Git?`). Si vas a fusionar chunks, **reinserta el separador**.

> [!danger] 🚨 (2) El bug que invalida parcialmente la comparación
> En el schema de la collection, la línea que limita qué se vectoriza **está comentada**:
>
> ```python
> vectorizer_config=[Configure.NamedVectors.text2vec_transformers(
>     name="vector",
>     #source_properties=['chunk'],     # ← COMENTADA
>     ...
> )]
> ```
>
> Sin `source_properties`, Weaviate vectoriza **todas** las propiedades de texto: `chunk` + `chapter_title` + `filename` + **`chunking_strategy`**. Es decir, cada embedding lleva incrustado el literal `"fixed_size_25"` o `"para_chunks"`.
>
> **Consecuencia:** dos chunks con texto idéntico tienen vectores distintos solo por su etiqueta de estrategia — precisamente la variable que el experimento quiere comparar. Los resultados siguen siendo ilustrativos, pero **no son una comparación limpia**. Es la demostración perfecta de por qué el Tomo 05 insiste en declarar `source_properties` de forma explícita.

> [!warning] ⚠️ (3) Cero métricas cuantitativas
> El "experimento" son **dos queries con `limit=2`, juzgadas a ojo**. No se imprime ni una distancia ni un score, no hay ground truth, no hay precision/recall/NDCG. Cualquier afirmación de "qué estrategia gana" en el notebook es **anecdótica**.
>
> Para decidir de verdad tu estrategia de chunking necesitas lo del [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]]: un set de queries representativas con documentos relevantes marcados, y **recall@k** por estrategia. Es trabajo, y es la única forma de no elegir por intuición.

### 5.5 El experimento RAG: el hallazgo que el lab no comenta

Audiencia: 🔧 🧭 👔

El lab cierra alimentando un LLM con lo recuperado por cada estrategia, **compensando el tamaño** para igualar el contexto:

```python
n_chunks_by_strat = {
    'fixed_size_25':      8,   # muchos chunks cortos
    'para_chunks':        8,
    'fixed_size_100':     2,   # pocos chunks largos
    'para_chunks_min_25': 2,
}
```

Comparando las cuatro respuestas generadas para *"history of git"*:

| Estrategia | ¿Menciona BitKeeper 2002? | ¿Linus Torvalds? | ¿Objetivos de diseño? | Ruido |
|---|:---:|:---:|:---:|---|
| `fixed_size_25` | ❌ | ❌ | ❌ | **Alto** — mete el commit inicial de 2008 como si fuera "historia de Git" |
| `fixed_size_100` | ✅ | ❌ | Parcial | Bajo |
| `para_chunks` | ❌ | ❌ | ❌ | **Alto** — mezcla rendimiento local y comparativas CLI/GUI |
| `para_chunks_min_25` | ✅ | ✅ | ✅ completo | Bajo |

> [!important] 🎯 El chunking no solo afecta la búsqueda: afecta lo que el LLM responde
> Con **la misma cantidad de contexto** y el mismo modelo, la estrategia de chunks largos produjo la respuesta **más completa y fiel**, y las de chunks cortos **arrastraron ruido del contexto adyacente** — inventando conexiones entre fragmentos que no tenían relación.
>
> Es el eslabón que cierra el tomo: los chunks mal dimensionados no producen "resultados un poco peores", producen **respuestas con hechos equivocados**. Y ese fallo se ve como una respuesta bien redactada — el patrón del [[Guia-Maestra-RAG_01-Introduccion-a-RAG#6. 🔬 Con RAG vs sin RAG: el experimento, con resultados reales|Tomo 01]].
>
> **El notebook imprime las cuatro respuestas y termina con "Congratulations!" sin analizarlas.** Es el hallazgo más valioso del lab y queda sin comentar.

---

> [!example] 📊 Caso de negocio — Salud: el protocolo cortado por la mitad
> **Problema:** una red de clínicas indexa sus protocolos clínicos y guías de práctica para que el personal los consulte desde el móvil. Los documentos son PDFs largos, de 40 a 120 páginas. El primer despliegue usa chunks de tamaño fijo sin más criterio, y el equipo médico reporta dos fallos que **no son "resultados un poco peores"**: consultas que devuelven un fragmento con una **dosis sin el paciente al que corresponde** (el corte cayó entre la condición y la posología), y consultas sobre criterios de derivación que devuelven un chunk que es **solo el encabezado de la sección**, sin el contenido.
>
> **Técnica aplicada:** tres cambios, en orden de impacto. **(1) Respetar la estructura del documento** en vez de cortar cada N caracteres: recursive splitting real —cascada de separadores con límite de tamaño— para que un protocolo se parta por sección y no a mitad de una tabla de dosificación. **(2) Higiene de chunks** (sección 5.3): descartar los que no alcanzan un mínimo de contenido y **fusionar cada encabezado con el bloque que le sigue**, para que ningún título viaje solo. **(3) Context-aware chunking** (sección 4.3): un LLM añade a cada chunk una línea que indica de qué protocolo y de qué sección proviene — texto que sirve dos veces, al vectorizar y al recuperar. Los chunks heredan la metadata del documento (protocolo, versión, fecha de vigencia) más su `chunk_index`, de modo que cada respuesta cita **protocolo, versión y sección**, no un PDF de 90 páginas.
>
> **Resultado:** las respuestas pasan de "el documento correcto" a "el párrafo correcto, citado con su versión vigente" — que es la única forma en que un profesional clínico puede verificarlas antes de actuar. Y el efecto secundario que decidió la adopción: al citar la versión, el sistema **dejó al descubierto protocolos desactualizados** que seguían circulando en la intranet. La lección: **en dominios donde el error tiene consecuencias, el chunking deja de ser una decisión de relevancia y pasa a ser una de seguridad** — un fragmento sin su contexto no es información incompleta, es información equivocada.

---

## 6. 🧭 Guía de decisión del tomo

Audiencia: 🔧 🧭

| Situación | Qué hacer |
|---|---|
| Empezando, sin baseline | Fixed-size ~500 caracteres, overlap 50–100. Medir |
| Documentos con estructura clara (Markdown, HTML, código) | Recursive splitting **real** (lista de separadores + límite de tamaño) |
| Queries mayoritariamente conceptuales | Chunks más largos |
| Queries mayoritariamente puntuales | Chunks más cortos |
| Ambos tipos de query (lo normal) | Tamaño intermedio + **context-aware** (4.3) |
| Chunks que se recuperan sin sentido por sí solos | **Context-aware chunking** — la mejor relación esfuerzo/beneficio |
| El chunking simple limita la calidad (medido) | Semantic o LLM-based, probando en un subconjunto |
| Encabezados o fragmentos triviales en el `top_k` | Higiene: mínimo de tamaño, fusión de encabezados, limpieza de artefactos |
| **Siempre** | Heredar metadata + `chunk_index`, y declarar `source_properties` explícitamente |

> [!tip] 🧭 El orden correcto de trabajo
> Fixed-size con overlap → **medir con un set de queries reales** → probar context-aware → y solo si la medición lo justifica, semantic o LLM-based. La sofisticación se gana con evidencia. Y fija la **unidad** (caracteres, palabras o tokens) desde el primer día: la mitad de las confusiones de esta capa vienen de comparar números en unidades distintas.

---

## 7. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Chunk** | Un pedazo de documento: la unidad mínima que el sistema puede encontrar y citar |
| **Chunking** | Decidir en qué pedazos se corta cada documento antes de indexarlo |
| **Chunk size** | Cuán grande es cada pedazo. Decide el nivel de detalle del sistema |
| **Overlap** | Cuánto comparten dos pedazos vecinos, para no cortar ideas por la mitad |
| **Stride** | Cuánto se avanza entre un chunk y el siguiente. Con overlap, no es lo mismo que el tamaño |
| **Fixed-size chunking** | Cortar cada N caracteres/palabras. Simple y ciego a la estructura |
| **Recursive splitting** | Cortar por la estructura del documento (párrafos, secciones), con límite de tamaño |
| **Semantic chunking** | Cortar donde cambia el tema, midiendo significado. Caro y preciso |
| **LLM-based chunking** | Pedirle a un modelo que decida los cortes |
| **Context-aware chunking** | Añadirle a cada pedazo una nota que explique de dónde viene |
| **`chunk_index`** | La posición del pedazo en su documento. Permite citar y reconstruir el orden |
| **`source_properties`** | Qué campos se convierten en vector. Si no lo declaras, se vectoriza de más |

---

## 8. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Sé enumerar las tres razones para chunkear y cuál resuelve el truncation silencioso.
- [ ] Puedo explicar por qué un vector por libro entero da mala relevancia.
- [ ] Entiendo cómo falla el chunk demasiado grande **y** el demasiado pequeño.
- [ ] Sé qué es el overlap, cómo se expresa y qué cuesta.
- [ ] Distingo `chunk_size` de `stride`, y sé que el lab del curso los confunde.
- [ ] Entiendo la diferencia entre un split por un separador y un splitter **recursivo** real.
- [ ] Conozco el default de arranque del curso (~500 caracteres, overlap 50–100) y su ambigüedad de unidades.
- [ ] Sé por qué los chunks deben heredar la metadata y para qué sirve `chunk_index`.
- [ ] Puedo explicar el ejemplo del sueño olímpico y qué riesgo ilustra.
- [ ] Entiendo cómo funciona semantic chunking y por qué es caro.
- [ ] Sé por qué context-aware chunking tiene el mejor trade-off (costo offline, cero en query time).
- [ ] Tengo claro que **no hay estrategia ganadora**: depende del tipo de query.
- [ ] Sé que el chunking afecta no solo la búsqueda sino **la veracidad de la respuesta generada**.
- [ ] Entiendo por qué olvidar `source_properties` contamina los embeddings.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]] (dónde se indexan los chunks)
- Siguiente tomo → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing, arquitecturas y re-ranking]]
- **El problema que este tomo resuelve** → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#5.4 ⚠️ La trampa del truncation|Tomo 04 · el truncation silencioso]]
- La fase de indexing en el pipeline → [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01]]
- Metadata que los chunks heredan, y filtros → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]]
- **Cómo medir qué estrategia gana de verdad** → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Evaluación]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 3: lecciones de **chunking** (básico) y **advanced chunking**; **Ungraded Lab 2** (chunking sobre el libro Pro Git), del que provienen todos los conteos, tamaños y comparaciones de la sección 5.

**Fuentes externas (complemento con bibliografía verificable):**
- Anthropic (2024, 19 de septiembre). *Introducing Contextual Retrieval*. Anthropic News. — La técnica que el curso llama *context-aware chunking*, con resultados medidos al combinarla con BM25 y re-ranking.
- Günther, M., Mohr, I., Williams, D. J., Wang, B., & Xiao, H. (2024). *Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models*. arXiv:2409.04701 (preprint). — Late chunking (sección 4), la alternativa de Jina AI al contextual retrieval.
- Documentación oficial de **LangChain**: `RecursiveCharacterTextSplitter` (la cascada de separadores que el lab no implementa) y `SemanticChunker`.
- Documentación oficial de **Weaviate**: `Configure.NamedVectors`, `source_properties`, `Tokenization.FIELD`.
- Corpus del lab: **Pro Git** (Chacon, S. & Straub, B.), repositorio `progit/progit2`, formato AsciiDoc.

> [!note] Sobre el código y los valores de este tomo
> - **Del curso:** toda la teoría de las secciones 1 a 4 (las tres razones, el dilema del tamaño, fixed-size, overlap, recursive, semantic, LLM-based y context-aware) proviene de las dos lecciones de chunking. Los conteos, tamaños y outputs de la sección 5 son literales del Ungraded Lab 2.
> - **Hallazgos propios verificados contra el notebook y sus outputs:** que el overlap está implementado con *stride = chunk_size* (y por tanto los chunks miden `size + overlap` y su número no cambia); que la sección "recursive" no implementa recursión; que `mixed_chunking` no parte los chunks grandes y produjo uno de 1.296 palabras; que **`source_properties` está comentado** y contamina los embeddings con la etiqueta de estrategia; que el experimento no tiene métricas cuantitativas; y el **análisis del experimento RAG de 5.5**, que el notebook imprime pero nunca comenta.
> - **Complemento propio marcado en el texto:** las reglas de higiene de 5.3, la nota sobre el splitter recursivo real, la equivalencia de unidades del default, y las guías de decisión.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing, arquitecturas de scoring y re-ranking]]**, donde se cierra el Módulo 3 con las tres optimizaciones que faltan: limpiar la **query** antes de buscar (query rewriting, NER, HyDE), las arquitecturas que puntúan mejor que un bi-encoder (**cross-encoders** y **ColBERT**), y el **re-ranking** que las hace viables en producción.
