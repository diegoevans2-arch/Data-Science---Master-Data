---
title: "Tomo 05 — Vector databases y ANN (HNSW)"
tags: [rag, vector-database, ann, hnsw, knn, weaviate, proximity-graph, hybrid-search, alpha]
audiencias: [tecnico, puente, ejecutivo]
tomo: 05
version: 1.1
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 05 — Vector databases y ANN (HNSW)

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search, embeddings y hybrid search]] · Siguiente → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]]

---

> [!info] ¿Por qué importa esta sección?
> Aquí empieza el **Módulo 3** y el proyecto cambia de naturaleza: pasamos de *entender* el retrieval a **ponerlo en producción**. El [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]] dejó el retriever conceptualmente completo, pero con un supuesto oculto e insostenible: que comparar la query contra **cada uno** de los vectores del corpus es viable. Con mil documentos lo es. Con mil millones, no.
>
> Este tomo resuelve ese problema y explica por qué las **vector databases** se volvieron casi sinónimo de RAG. Es también el tomo que más se parece a un manual de operación: aquí aparecen por primera vez los parámetros que vas a tunear de verdad.

> [!abstract] 👔 Impacto ejecutivo
> La diferencia entre un prototipo que funciona con 500 documentos y un sistema que responde en 200 ms sobre 50 millones **no es el modelo: es el índice**. Esta es la capa que decide si tu RAG es un demo o un producto.
> - **Decisiones que habilita:** dimensionar infraestructura de retrieval con criterio; elegir vector database (o justificar no usar una); negociar el trade-off entre latencia, costo de memoria y calidad de resultados; poner números a un SLA de búsqueda.
> - **Costo o riesgo de hacerlo mal:** implementar búsqueda vectorial de forma ingenua produce un sistema que funciona en la demo y **se cae en producción** — no por un bug, sino porque el costo crece linealmente con el catálogo. Y al revés: montar infraestructura de vector database para 2.000 documentos es sobre-ingeniería pura.
> - **Pregunta ejecutiva que responde:** *"¿este sistema va a seguir respondiendo rápido cuando tengamos 100 veces más documentos, y cuánto nos va a costar?"*

---

## 1. 🐌 El problema: kNN no escala

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** la forma más simple de retrieval vectorial es **k-Nearest Neighbors (kNN)** — la que hemos usado implícitamente en todo el curso hasta ahora:

```
   1. Embeber cada documento de la knowledge base
   2. Embeber el prompt
   3. Calcular la distancia del prompt a CADA vector de documento
   4. Ordenar por distancia
   5. Devolver los k más cercanos
```

Es fácil de entender, fácil de implementar y **da el resultado exactamente correcto**. También se llama **exact search** o *brute force*, y tiene un solo defecto: escala terriblemente.

```
   El nº de cálculos crece LINEALMENTE con el corpus

   1.000 documentos          →          1.000 distancias por búsqueda
   1.000.000 documentos      →      1.000.000 distancias por búsqueda
   1.000.000.000 documentos  →  1.000.000.000 distancias por búsqueda
                                        ▲
                                        └── un millón de veces más lento
                                            que la primera búsqueda
```

> [!tip] 💡 Analogía
> Buscar con kNN es entrar a una biblioteca y **preguntarle a cada libro, uno por uno**, "¿te pareces a lo que busco?". Con una estantería funciona. Con la Biblioteca Nacional te jubilas antes de terminar. Lo notable es que el método es *perfecto* — vas a encontrar el mejor libro con certeza absoluta — solo que la certeza cuesta un tiempo que nadie tiene.

**👔 En una frase para el negocio:** la búsqueda exacta es la que garantiza el mejor resultado y la que hace inviable el producto; toda la ingeniería de esta capa consiste en cambiar una pizca de exactitud por varios órdenes de magnitud de velocidad.

---

## 2. 🎯 ANN: cambiar exactitud por velocidad

Audiencia: 🔧 🧭

**🔧 Definición técnica:** los retrievers reales usan una familia de algoritmos llamada **Approximate Nearest Neighbors (ANN)**. Usan estructuras de datos inteligentes para acelerar la búsqueda de forma radical, y para lograrlo hacen **un pequeño sacrificio en la calidad de los resultados**: no garantizan encontrar los documentos *absolutamente* más cercanos, pero encuentran documentos muy cercanos.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  kNN  →  resultado exacto,        crecimiento LINEAL         │
   │  ANN  →  resultado aproximado,    crecimiento ~LOGARÍTMICO   │
   └──────────────────────────────────────────────────────────────┘
```

> [!important] La palabra "aproximado" asusta más de lo que debería
> Un retriever **ya es aproximado por naturaleza** mucho antes de llegar al índice: el embedding model comprime significado con pérdida, el chunking parte los documentos por donde puede, y la noción misma de "relevante" es difusa. Sumar una aproximación más en el índice —que en la práctica recupera casi siempre los mismos documentos que la búsqueda exacta— es un costo marginal frente al beneficio.
>
> Lo que **sí** hay que hacer es medirlo. La métrica se llama **recall del índice**: qué fracción de los verdaderos k vecinos más cercanos devolvió el ANN. Un índice bien configurado alcanza recall de 0,95–0,99 con una fracción del costo de kNN. Si nadie lo mide, nadie sabe qué se está perdiendo.

---

## 3. 🕸️ NSW: el proximity graph

Audiencia: 🔧

El curso desarrolla un algoritmo ANN concreto: **Navigable Small World (NSW)**. Entenderlo es entender la familia entera.

### 3.1 Construcción del grafo (offline)

**🔧 Definición técnica:** antes de procesar cualquier búsqueda, el algoritmo construye una estructura llamada **proximity graph**:

```
   1. Calcular la distancia entre cada vector y todos los demás
   2. Agregar un NODO por documento
   3. Conectar cada documento con unos POCOS de sus más cercanos (aristas)

   Resultado: una estructura tipo telaraña

            ●────────●
           ╱ ╲      ╱ ╲
          ●───●────●───●
           ╲ ╱ ╲  ╱ ╲ ╱
            ●───●────●
             ╲      ╱
              ●────●

   Recorrerlo = saltar de un documento a sus vecinos por las aristas
```

> [!warning] ⚠️ Construir el índice es caro — y por eso se precomputa
> Ese paso 1 es **computacionalmente intensivo** (comparar todo contra todo). La buena noticia: se hace **una sola vez**, offline, antes de recibir cualquier prompt. Es la misma lógica de las dos fases del [[Guia-Maestra-RAG_01-Introduccion-a-RAG#3. ⏱️ Las dos fases: indexing y retrieval|Tomo 01]]: el trabajo pesado va en indexing, no en query time.
>
> Consecuencia operativa que sorprende a mucha gente: **insertar documentos en una vector database no es un `INSERT` barato.** Cada objeto nuevo dispara (1) la vectorización del texto y (2) la actualización del índice HNSW. Cargar un corpus grande toma tiempo real.

### 3.2 Búsqueda (online)

```
   Prompt ──► vector de query
                    │
                    ▼
   1. Elegir un punto de entrada AL AZAR (candidate vector)
      ─ sin ninguna suposición de que esté cerca del prompt ─
                    │
                    ▼
   2. Mirar los vecinos del candidato actual
      ¿cuál está más cerca del vector de query?
                    │
                    ▼
   3. Ese vecino se convierte en el nuevo candidato  ──┐
                    │                                  │
                    ▼                                  │
   4. ¿Algún vecino está más cerca?  ── SÍ ────────────┘
                    │
                   NO
                    ▼
        Devolver el candidato actual
```

Como en cada paso solo hay que evaluar **unos pocos vecinos**, cada salto es rapidísimo.

> [!tip] 💡 Analogía
> Es orientarse en una ciudad desconocida **preguntando en cada esquina**: "¿por dónde me acerco a la plaza?". No conoces el mapa completo, pero cada respuesta te deja un poco más cerca, y llegas en unos pocos saltos en vez de recorrer todas las calles. El detalle importante: como decides con información **local** en cada esquina, puedes terminar en una plaza muy buena pero no necesariamente en *la* mejor de la ciudad.

**🔧 Por qué es aproximado, exactamente:** el algoritmo no puede elegir la mejor ruta **global** por el grafo — solo la mejor en cada momento. Puede existir un vector más cercano al prompt que simplemente no es alcanzable por el camino que tomó. En la práctica encuentra vectores muy cercanos, y mucho más rápido que kNN.

---

## 4. 🏔️ HNSW: el estándar de la industria

Audiencia: 🔧 🧭

**🔧 Definición técnica:** **HNSW (Hierarchical Navigable Small World)** mejora NSW acelerando drásticamente la parte inicial de la búsqueda. La idea: en vez de un solo grafo, construir un **proximity graph jerárquico de varias capas**.

Con una knowledge base de 1.000 documentos, el ejemplo del curso:

```
   CAPA 3   ●        ●        ●          ← 10 vectores  (se descartan al azar
              ╲    ╱   ╲    ╱              todos menos 10)
               ●  ●     ●  ●              grafo propio de esos 10
   ─────────────────┼───────────────────
                    │ se baja al mejor candidato encontrado
                    ▼
   CAPA 2   ● ● ● ● ● ● ● ● ● ●          ← 100 vectores (se descartan
             ╲│╱ ╲│╱ ╲│╱ ╲│╱               todos menos 100)
              ●   ●   ●   ●               grafo propio de esos 100
   ─────────────────┼───────────────────
                    │
                    ▼
   CAPA 1   ●●●●●●●●●●●●●●●●●●●●        ← LOS 1.000 vectores
            (el grafo completo)             el mejor candidato aquí
                    │                        es el que se devuelve
                    ▼
              resultado final
```

**El recorrido:** la búsqueda **empieza en la capa superior** (la más rala), encuentra el mejor candidato ahí, **baja** a la capa siguiente usando ese candidato como punto de partida, y repite hasta la capa 1.

> [!tip] 💡 Analogía
> Es planificar un viaje con **tres mapas de distinto zoom**. Primero el mapa de países: decides el continente con dos o tres miradas. Luego el mapa de regiones: te ubicas en la provincia. Recién al final el callejero de la ciudad. Nadie busca una dirección leyendo el callejero de todo el planeta — y eso es exactamente lo que hace kNN.

**🔧 Por qué es tan eficiente:** en las capas altas el algoritmo da **saltos grandes** para llegar al vecindario aproximado del prompt. Cuando por fin considera vectores en la capa 1, el candidato ya está muy cerca. Como al subir de capa hay exponencialmente menos vectores, el runtime de HNSW es **aproximadamente logarítmico**, mientras kNN es lineal.

```
   kNN   →  O(n)       n = nº de documentos
   HNSW  →  O(log n)   aproximadamente
```

> [!abstract] 👔 El número que importa
> Esto es lo que permite que la búsqueda vectorial escale a **miles de millones de vectores** y aun así responda en **unos pocos cientos de milisegundos**. Sin HNSW (o un algoritmo equivalente), los sistemas RAG a escala corporativa simplemente no existirían.

### 4.1 Las tres cosas que hay que recordar de ANN

Audiencia: 🔧 🧭

El curso las resume así, y vale copiarlas tal cual porque son el criterio de decisión completo:

| # | Propiedad | Consecuencia práctica |
|---|---|---|
| 1 | **Son mucho más rápidos que kNN** | Es lo que hace posible la búsqueda vectorial a escala |
| 2 | **No garantizan encontrar los mejores matches** | Hay que medir el recall del índice, no asumirlo |
| 3 | **Todo depende de construir un buen proximity graph** | Proceso costoso, pero **precomputable** antes de recibir prompts |

> [!note] 🔬 Complemento de vanguardia — HNSW no es el único ANN
> El curso enseña HNSW porque es el más usado, pero conviene conocer el mapa por si el proyecto lo pide:
>
> | Familia | Idea | Cuándo conviene |
> |---|---|---|
> | **HNSW** (grafo jerárquico) | Lo de este tomo (Malkov & Yashunin, 2020) | El default: mejor calidad/latencia. Cuesta RAM |
> | **IVF** (índice invertido + clustering) | Partir el espacio en celdas; buscar solo en las cercanas | Corpus enormes con RAM limitada |
> | **PQ / quantization** | Comprimir los vectores a menos bits | Cuando la memoria es el cuello de botella ([[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11]]) |
> | **ScaNN / DiskANN** | Optimizados para disco o para hardware específico | Escalas extremas |
>
> Se combinan (HNSW+PQ es común). El benchmark de referencia independiente es **ANN-Benchmarks** (Aumüller et al., 2020), que compara recall vs. queries/segundo — la curva que de verdad hay que mirar al elegir.

---

## 5. 🗄️ Qué es una vector database

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** una **vector database** es una base de datos diseñada **desde cero** para almacenar datos vectoriales de alta dimensión e implementar algoritmos orientados a vectores como el ANN que acabamos de ver. Ganaron popularidad a **principios de los 2020**, como respuesta a la disponibilidad masiva de LLMs y a la explosión de técnicas basadas en embeddings.

**Por qué no basta con una base relacional:** las bases relacionales estándar **rinden mal** en semantic search — su desempeño se parece mucho al del kNN ineficiente. Las vector databases están optimizadas para las tareas que importan aquí: construir el proximity graph que alimenta HNSW y calcular distancias vectoriales.

> [!tip] 🧭 ¿Necesitas una vector database?
> El propio curso lo dijo en el [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]]: **no son estrictamente necesarias.** El criterio honesto:
>
> | Tu situación | Recomendación |
> |---|---|
> | Prototipo, < ~10k chunks | **FAISS en memoria** o incluso numpy. No montes infraestructura |
> | Ya usas PostgreSQL y el volumen es moderado | **pgvector** — evita un sistema nuevo que operar |
> | Producción, corpus grande o creciente, filtros y metadata | **Vector database dedicada** (Weaviate, Qdrant, Milvus, Pinecone…) |
> | Escala masiva, equipo de infra | Evaluar por benchmark propio, no por marketing |
>
> Un dato que refuerza esto y que aparece en la configuración real del índice (sección 7): Weaviate trae un `flat_search_cutoff` de **40.000 objetos** — por debajo de ese umbral **usa búsqueda exacta (flat) en vez de HNSW**, porque a esa escala el grafo no paga. La propia herramienta admite que HNSW es para cuando hay volumen.

**El curso usa Weaviate**, una vector database open-source que corre local o en la nube. La advertencia del propio instructor vale oro: *si eliges otra vector database, casi con certeza ofrecerá funcionalidad muy similar* — lo que importa es el **workflow**, no el vendor.

### 5.1 El workflow completo

```
   ═══ PREPARACIÓN (offline, una vez) ═══

   1. Configurar la base de datos
   2. Cargar los documentos
   3. Crear los SPARSE vectors     → alimentan keyword search ([[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]])
   4. Crear los DENSE embeddings   → alimentan semantic search ([[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]])
   5. Crear el índice ANN (HNSW)
                    │
   ═══ OPERACIÓN (online, cada query) ═══
                    ▼
   6. Ejecutar búsquedas: semantic · keyword · hybrid · con filtros
```

Los pasos 3 a 5 los hace la base **automáticamente** en la mayoría de los casos — no se implementan a mano.

---

> [!example] 📊 Caso de negocio — Medios: el archivo de 40 años que nadie podía consultar
> **Problema:** un grupo de comunicación tiene cuatro décadas de archivo — notas, transcripciones de radio, fichas de video — que en total superan los **12 millones de piezas**. El buscador interno es una base relacional con `LIKE`: los periodistas tardan minutos y terminan preguntándole a los colegas con más años en la casa. El primer prototipo de búsqueda semántica funciona **precioso** con las 3.000 notas de prueba y, al apuntarlo al archivo completo, cada consulta pasa a tardar **más de un minuto**. Nada se rompió: el prototipo comparaba contra 3.000 vectores y ahora compara contra 12 millones.
>
> **Técnica aplicada:** el diagnóstico es el de la sección 1 — el prototipo hacía kNN, cuyo costo crece **linealmente**. Se migra a una vector database con índice **HNSW**, cuyo costo crece de forma aproximadamente logarítmica. Sobre eso, tres decisiones que el tomo justifica: **(1)** `ef` se calibra en caliente hasta encontrar el punto donde el recall del índice deja de mejorar (los primeros valores perdían notas relevantes); **(2)** los filtros de sección, fecha y derechos de uso se aplican como **filtered search** —dentro de la búsqueda, no después— para que el `top_k` no se vacíe con las piezas cuyos derechos ya expiraron; **(3)** se mide el **recall del índice** contra un kNN exacto sobre una muestra, para saber qué está perdiendo la aproximación en vez de suponerlo.
>
> **Resultado:** las consultas vuelven a resolverse en **cientos de milisegundos** sobre el archivo completo, con un recall de índice medido y aceptado explícitamente como trade-off. Y una consecuencia de negocio que no estaba en el plan: el archivo dejó de ser un depósito y **pasó a ser material reutilizable** — las notas de contexto histórico que antes solo conocían los veteranos ahora las encuentra cualquiera. La lección: **el prototipo que funciona con mil documentos no prueba nada sobre el sistema que tendrá un millón; la escala no es un problema de tamaño, es un problema de algoritmo.**

---

## 6. 💻 Weaviate en la práctica

Audiencia: 🔧

Todo lo que sigue proviene del **Ungraded Lab 1 del Módulo 3**, con el cliente Python v4.

> [!note] Contexto del lab
> **Modo:** *Embedded Weaviate* — la base corre en el propio proceso, sin servidor aparte. Ideal para aprender.
> **Dataset:** 20 destinos turísticos de EE.UU. con 8 campos (`place`, `state`, `description`, `best_season_to_visit`, `attractions`, `budget`, `user_ratings`, `last_updated`).
> **Modelos reales:** embeddings con **`BAAI/bge-base-en-v1.5`** (el mismo del Módulo 2 — ver la advertencia de 6.1) y reranking con **`BAAI/bge-reranker-base`**, servidos por un Flask local en `127.0.0.1:5000`.

### 6.1 Conexión

```python
import weaviate

client = weaviate.connect_to_embedded(
    persistence_data_path="./.collections",       # los datos PERSISTEN aquí
    environment_variables={
        "ENABLE_API_BASED_MODULES": "true",
        "ENABLE_MODULES": "text2vec-transformers, reranker-transformers",
        "TRANSFORMERS_INFERENCE_API": "http://127.0.0.1:5000/",   # dónde vectorizar
        "RERANKER_INFERENCE_API":     "http://127.0.0.1:5000/",   # dónde rerankear
    }
)

# ... trabajo ...

client.close()      # el lab insiste: no olvidar cerrar el cliente
```

> [!warning] ⚠️ El lab nunca dice qué modelo de embeddings usa
> El notebook habla de `text2vec-transformers`, que es el **módulo** de Weaviate — no el modelo. Los nombres reales (`BAAI/bge-base-en-v1.5` y `BAAI/bge-reranker-base`) solo aparecen en los archivos de soporte `utils.py` y `flask_app.py`.
>
> Esto importa por la regla del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#4.4 Las tres conclusiones que hay que llevarse|Tomo 04]]: **solo se comparan vectores del mismo embedding model**, y cambiarlo obliga a re-indexar todo. Si no sabes qué modelo está vectorizando tu base, no sabes cuándo tienes que re-indexar. **Deja siempre el nombre y la versión del modelo explícitos en la configuración de tu proyecto.**

### 6.2 Definir el vectorizer y crear la collection

Una **collection** es el equivalente a una tabla. Lo interesante es que el schema declara **qué propiedades se vectorizan**:

```python
from weaviate.classes.config import Configure, Property, DataType

vectorizer_config = [Configure.NamedVectors.text2vec_transformers(
    name="vector",
    source_properties=[            # ← SOLO estas propiedades se concatenan y vectorizan
        'place', 'state', 'description',
        'best_season_to_visit', 'attractions', 'budget'
    ],
    vectorize_collection_name=False,   # no anteponer el nombre de la collection al texto
    inference_url="http://127.0.0.1:5000",
)]

collection = client.collections.create(
    name='example_collection',
    vectorizer_config=vectorizer_config,
    reranker_config=Configure.Reranker.transformers(),      # habilita re-ranking
    properties=[
        Property(name="place",                vectorize_property_name=True, data_type=DataType.TEXT),
        Property(name="state",                vectorize_property_name=True, data_type=DataType.TEXT),
        Property(name="description",          vectorize_property_name=True, data_type=DataType.TEXT),
        Property(name="best_season_to_visit", vectorize_property_name=True, data_type=DataType.TEXT),
        Property(name="attractions",          vectorize_property_name=True, data_type=DataType.TEXT),
        Property(name="budget",               vectorize_property_name=True, data_type=DataType.TEXT),
        Property(name="user_ratings",  data_type=DataType.NUMBER),   # no vectorizada
        Property(name="last_updated", data_type=DataType.DATE),      # no vectorizada
    ]
)
```

> [!tip] 💡 Dos decisiones de diseño escondidas en ese schema
> **(1) No todo se vectoriza.** `user_ratings` y `last_updated` quedan fuera de `source_properties`: son **numérica y fecha**, sirven para *filtrar*, no para buscar por significado. Vectorizar un rating no tiene sentido semántico.
>
> **(2) `vectorize_property_name=True` mete el nombre del campo en el texto a vectorizar.** El lab explica por qué con un ejemplo perfecto: el valor `"Moderate"` del campo `budget`, **solo**, no dice nada — podría ser un clima moderado o un esfuerzo moderado. Vectorizar `"budget Moderate"` sí ubica el concepto. Es una forma barata y elegante de inyectar contexto al embedding.

**Nota sobre idempotencia:** intentar crear una collection que ya existe **lanza excepción** (`422: class name Example_collection already exists`). El patrón habitual es verificar con `client.collections.exists(...)` antes de crear.

> [!note] Detalle que confunde: Weaviate capitaliza el nombre
> Se crea con `name='example_collection'` (minúscula), pero internamente la clase pasa a ser **`Example_collection`**. El error lo dice así, y `client.collections.list_all()` devuelve `dict_keys(['Example_collection'])`. Los helpers aceptan la minúscula, así que rara vez molesta — hasta que estás depurando por qué un nombre "no coincide".

### 6.3 Insertar datos por batch

```python
from weaviate.util import generate_uuid5

with collection.batch.fixed_size(batch_size=1, concurrent_requests=1) as batch:
    for document in data:
        uuid = generate_uuid5(document)      # UUID derivado del CONTENIDO
        batch.add_object(properties=document, uuid=uuid)
```

Dos ideas valiosas aquí:

- **`generate_uuid5(document)` deriva el ID del contenido**, así que reinsertar el mismo documento sobrescribe en vez de duplicar. Es **deduplicación gratis por diseño** — un patrón que conviene copiar.
- **El batch** decide cuántos objetos van por lote, **acumula errores** para poder corregirlos después, y reduce el número de llamadas de red.

> [!warning] ⚠️ Lo que pasa realmente en cada `add_object`
> El lab lo advierte y es fácil de subestimar: **(1)** el texto se vectoriza (llamada al embedding model) y **(2)** el índice HNSW se actualiza. Por eso "cargar datos" en una vector database no se parece a un `INSERT` — con corpus grandes es un job, no una operación.
>
> El lab usa `batch_size=1, concurrent_requests=1` (los valores más conservadores posibles, didácticos): 20 documentos tardan ~7 segundos. **En producción se suben ambos parámetros** de forma sustancial.

### 6.4 Los cuatro modos de búsqueda

Aquí se materializa todo el Módulo 2. Nota que **los cuatro aceptan los mismos filtros y el mismo `limit`**:

```python
from weaviate.classes.query import Filter

QUERY = 'I want suggestions to travel during Winter. I want cheap places.'

# ── 1. FETCH: sin búsqueda, solo filtro (metadata filtering puro, Tomo 03)
collection.query.fetch_objects(
    limit=2,
    filters=Filter.by_property('user_ratings').greater_or_equal(3.5)
)

# ── 2. SEMANTIC (near_text): dense vectors + HNSW  (Tomo 04)
collection.query.near_text(query=QUERY, limit=4)

# ── 3. KEYWORD (bm25): sparse vectors + inverted index  (Tomo 03)
collection.query.bm25(query=QUERY, limit=4)

# ── 4. HYBRID: ambas en paralelo, fusionadas por alpha  (Tomo 04)
collection.query.hybrid(query=QUERY, alpha=0.3, limit=4)
```

Y los filtros se componen sobre cualquiera de ellos:

```python
collection.query.near_text(
    query=QUERY,
    filters=Filter.by_property('budget').equal('Low'),                    # igualdad
    limit=4
)

collection.query.near_text(
    query=QUERY,
    filters=Filter.by_property('budget').contains_any(['Low','Moderate']), # pertenencia
    limit=4
)
```

> [!important] 🎯 Esto es el pre-filtering del que hablaba el Tomo 03
> Fíjate en lo que acabas de ver: el filtro va **dentro** de la llamada de búsqueda, no después. Weaviate lo aplica **durante** la búsqueda (*filtered search*), que es exactamente el **pre-filtering** que el [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25#2.4 En código|Tomo 03]] recomendaba frente al post-filtering del diagrama de clase. La vector database resuelve por ti el problema de quedarte con menos resultados que tu `top_k`.
>
> (En el schema real esto aparece como `filter_strategy: "sweeping"` — la estrategia con que Weaviate combina filtro e índice.)

### 6.5 El parámetro `alpha` de hybrid search

Audiencia: 🔧 🧭 👔

**🔧 Definición:** en `hybrid()`, **`alpha` pondera cuánto pesa el lado semántico** frente al léxico:

```
   alpha = 1.0   →  semantic (vector) puro
   alpha = 0.5   →  mitad y mitad
   alpha = 0.0   →  keyword (BM25) puro

   El lab usa alpha = 0.3   →  30 % semantic / 70 % keyword
```

> [!danger] ⚠️ Tres trampas con `alpha` — dos son del propio material
> **(1) `alpha` es el mismo parámetro que el curso llamó `beta` en el Módulo 2.** La clase teórica lo llamó `beta`; el código lo llama `alpha`. Misma perilla, mismo significado (peso del lado semántico). Ya está anotado en el [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#6.6 El hiperparámetro `beta`: el balance del sistema|Tomo 04]].
>
> **(2) El notebook lo explica al revés.** Dice que `alpha` controla *"cuánto BM25 quieres mezclar"*, lo que sugiere que subir `alpha` sube el peso de BM25. Es lo contrario: **subir `alpha` sube el peso semántico y BAJA el de BM25**. La clase teórica del Módulo 3 sí lo dice bien (*"con alpha en 0.25, vector search recibe 25%"*). Si sigues el notebook al pie de la letra, tunearás la perilla en la dirección equivocada.
>
> **(3) Verifica el sentido con un caso extremo.** No todas las implementaciones usan la misma convención. Antes de confiar en un valor, corre `alpha=0` y `alpha=1` y mira **cuál lado se apaga**. Treinta segundos que evitan semanas de tuning al revés.

> [!warning] ⚠️ Y una advertencia honesta sobre el lab: no demuestra nada
> El lab corre `hybrid(alpha=0.3)` y devuelve **exactamente los mismos 4 documentos, en el mismo orden**, que `near_text` con el mismo filtro. Además **nunca solicita metadata**, así que no hay ni un score ni una distancia en todo el notebook (`distance`, `certainty`, `score` y `rerank_score` salen `None` en cada resultado).
>
> Traducido: el notebook **no aporta evidencia** de que la fusión esté haciendo algo. Para ver el efecto hay que pedir la metadata explícitamente:
>
> ```python
> from weaviate.classes.query import MetadataQuery
>
> result = collection.query.hybrid(
>     query=QUERY, alpha=0.3, limit=4,
>     return_metadata=MetadataQuery(score=True, explain_score=True)   # ← imprescindible
> )
> for obj in result.objects:
>     print(obj.metadata.score, obj.properties['place'])
> ```
>
> **Regla general:** si no pides la metadata, estás tuneando a ciegas. Y `explain_score` en Weaviate te muestra el desglose de la fusión — la herramienta de depuración de hybrid search.

### 6.6 Un hallazgo del lab que vale más que el código

Audiencia: 🔧 🧭

Con `limit=4`, la búsqueda BM25 devuelve **un solo documento**:

```
   collection.query.bm25(query='...Winter... cheap places.', limit=4)
   →  1 resultado:  Times Square
```

No es un bug: **`limit` es un techo, no una cuota**. BM25 solo devuelve documentos con solape léxico real, y en este corpus casi ninguno comparte palabras con la query. Los destinos hablan de *"snowy"*, *"skiing"*, *"affordable"* — no de *"Winter"* ni *"cheap"*.

> [!important] Es la mejor demostración de por qué existe hybrid search
> El [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25#5. ⚖️ Fortalezas y debilidades de keyword search|Tomo 03]] cerró con el *vocabulary mismatch* como debilidad estructural de keyword search. Aquí está, medido: **pediste 4 documentos y el léxico solo pudo darte 1.** Semantic search devolvió 4 (aunque con sus propios problemas, ver abajo). Ninguna de las dos técnicas sola es suficiente — y eso es literalmente el argumento de `hybrid()`.
>
> El notebook no comenta esta discrepancia, que es el punto pedagógico más fuerte de toda la sección.

Y en el otro sentido, el resultado semántico también falla de forma instructiva: ante *"viajar en **invierno**, lugares **baratos**"*, los puestos 2 a 4 son destinos de `Summer` y `Spring/Fall` con `budget: Moderate`. Es el **caso Kyoto** del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings#5.3 🔬 Honestidad sobre qué NO capturan|Tomo 04]] otra vez: el embedding captura "destino turístico agradable", no la restricción lógica "invierno Y barato". **Las restricciones duras se resuelven con filtros de metadata, no esperando que el embedding las entienda.**

---

## 7. ⚙️ Los parámetros reales del índice

Audiencia: 🔧

Esta sección es la joya operativa del tomo. Al imprimir el schema, el lab expone la configuración real con que Weaviate montó el índice — los parámetros que en producción **sí** vas a tocar.

### 7.1 HNSW

```
   distance_metric        = "cosine"          ← coherente con el Tomo 04
   ef_construction        = 128
   max_connections        = 32
   ef                     = -1                ← -1 = dinámico
   dynamic_ef_min         = 100
   dynamic_ef_max         = 500
   dynamic_ef_factor      = 8
   flat_search_cutoff     = 40000
   cleanup_interval_seconds = 300
   quantizer              = null              ← sin compresión (Tomo 11)
```

| Parámetro | Qué controla | Trade-off |
|---|---|---|
| **`ef_construction`** | Cuántos candidatos se evalúan **al construir** el grafo | ↑ mejor grafo (mejor recall) · indexado más lento |
| **`max_connections`** | Aristas por nodo (la "M" del paper) | ↑ mejor recall · **más RAM** y grafo más lento de construir |
| **`ef`** | Cuántos candidatos se evalúan **al buscar** | ↑ mejor recall · **más latencia**. La perilla de query time |
| **`dynamic_ef_*`** | Weaviate ajusta `ef` solo, según el `limit` pedido | Por eso `ef = -1`: delega en el rango 100–500 |
| **`flat_search_cutoff`** | Bajo este nº de objetos usa búsqueda **exacta** | Con 20 documentos, el lab **ni siquiera usó HNSW** |
| **`distance_metric`** | La métrica de similitud | `cosine`, alineado con el Tomo 04 |

> [!tip] 🧭 Las dos perillas que importan, y cuándo tocarlas
> **`ef_construction` + `max_connections`** son de **build time**: mejoran la calidad del grafo a costa de RAM y tiempo de indexado. Se deciden una vez y cambiarlas implica **reconstruir el índice**.
>
> **`ef`** es de **query time**: es el dial recall↔latencia que puedes mover en caliente. **Si tu búsqueda va rápido pero se pierde documentos, esta es la primera perilla.** Y como está en `-1` (dinámico), lo primero es saber si tu base la está ajustando sola.

> [!warning] ⚠️ El detalle que reencuadra todo el lab
> Con `flat_search_cutoff = 40000` y **20 documentos**, el lab corrió **búsqueda exacta**, no HNSW. Todo lo aprendido sobre el grafo jerárquico es cierto y necesario — pero no se ejercitó ahí. Es un recordatorio útil: **a escala de prototipo, el índice sofisticado no aporta**; la propia herramienta lo desactiva.

### 7.2 BM25 — y una confirmación bonita

```
   b               = 0.75
   k1              = 1.2
   stopwords.preset = "en"
```

> [!important] ✅ Coherencia confirmada con el Tomo 03
> Los defaults de BM25 en Weaviate son **exactamente** los que el [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25#4.3 La fórmula y sus dos perillas|Tomo 03]] documentó como estándar de la industria: `k1 = 1.2` (dentro del rango 1.2–2.0 de la lámina del curso) y `b = 0.75`. Y nota el `stopwords.preset = "en"`: **la base elimina stopwords por ti** — precisamente el paso cuya ausencia hizo que un documento sobre pan de masa madre le ganara al de hornos de pizza en el Tomo 03. En una vector database, el preprocesamiento viene resuelto.

---

## 8. 🔄 Re-ranking: el adelanto

Audiencia: 🔧 🧭

El lab cierra habilitando un **reranker**, que se activa con un parámetro más:

```python
from weaviate.classes.query import Rerank

response = collection.query.near_text(
    query="I want suggestions to travel during Winter. I want cheap and fun places.",
    limit=5,
    rerank=Rerank(
        prop="attractions",        # sobre qué propiedad rerankear
        query="Fun places"         # opcional: si se omite, usa la query original
    )
)
```

Esa es la promesa que el curso hace del re-ranking: en muchas vector databases es **literalmente una línea más** en la query. El mecanismo (cross-encoders), el por qué funciona y cuántos documentos over-fetchear se desarrollan en [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]].

> [!danger] 🚨 Bug real en el lab: la celda de re-ranking no demuestra re-ranking
> Vale documentarlo porque es el tipo de error que se copia sin notarlo. La celda de búsqueda asigna a **`response`**, pero la celda que imprime itera sobre **`result`** — una variable vieja que todavía contiene el resultado de `hybrid()` de la sección anterior:
>
> ```python
> response = collection.query.near_text(..., rerank=Rerank(...))   # asigna a `response`
> for obj in result.objects:                                        # ⚠️ imprime `result`
>     print_object_properties(obj.properties)
> ```
>
> Tres pruebas de que el output mostrado **no** viene del reranker:
> 1. El rerank pide `limit=5`, pero se imprimen **4** objetos.
> 2. La salida es **idéntica** a la de la celda de hybrid search.
> 3. La query de rerank **no** lleva filtro de `budget`, pero los 4 objetos mostrados son justo los filtrados por `contains_any(['Low','Moderate'])`.
>
> Y aunque se corrigiera la variable, seguiría sin verse el efecto: **no se pide `rerank_score`**, así que saldría `None`. La versión que sí demuestra algo:
>
> ```python
> from weaviate.classes.query import MetadataQuery, Rerank
>
> response = collection.query.near_text(
>     query="...", limit=5,
>     rerank=Rerank(prop="attractions", query="Fun places"),
>     return_metadata=MetadataQuery(score=True, rerank_score=True)   # ← ver el efecto
> )
> for obj in response.objects:                                       # ← la variable correcta
>     print(obj.metadata.rerank_score, obj.properties['place'])
> ```

---

## 9. 🧭 Guía de decisión del tomo

Audiencia: 🔧 🧭

| Situación | Qué hacer |
|---|---|
| Prototipo con pocos miles de chunks | FAISS/numpy. El índice ANN no aporta todavía |
| Ya tienes Postgres, volumen moderado | `pgvector` antes de sumar un sistema nuevo |
| Producción con filtros, metadata y crecimiento | Vector database dedicada |
| Búsqueda rápida pero se pierde documentos | Subir **`ef`** (query time, en caliente) |
| Recall insuficiente incluso con `ef` alto | Subir `ef_construction` / `max_connections` → **re-indexar** |
| Memoria como cuello de botella | Quantization ([[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11]]) |
| Necesitas coincidencia exacta *y* semántica | `hybrid()` y tunear `alpha` — midiendo, con metadata |
| Restricciones duras (permisos, fecha, categoría) | **Filtros**, no esperar que el embedding las entienda |
| No sabes qué está perdiendo tu índice | Medir **recall del índice** contra un kNN exacto sobre una muestra |

> [!tip] 🧭 El orden correcto de trabajo
> Empezar exacto y simple → medir → introducir ANN cuando el volumen lo exija → tunear `ef` en caliente → reconstruir el índice solo si hace falta. Y en **todos** los casos: **pedir la metadata** (`score`, `distance`, `explain_score`). Un retriever que no reporta sus scores es un retriever que no se puede depurar.

---

## 10. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **kNN / exact search** | Comparar contra todo. Perfecto y lentísimo |
| **ANN** | Comparar de forma astuta. Casi perfecto y rapidísimo |
| **Recall del índice** | Qué fracción de los mejores resultados reales encontró el atajo |
| **Proximity graph** | El mapa de "quién se parece a quién" que se arma antes de buscar |
| **NSW** | Navegar ese mapa saltando de vecino en vecino |
| **HNSW** | Lo mismo, pero con mapas de varios niveles de zoom. El estándar |
| **`ef` / `ef_construction`** | Cuánto esfuerzo pones al buscar / al construir el mapa |
| **`max_connections`** | Cuántos vecinos conoce cada documento. Más = mejor y más RAM |
| **Vector database** | Base de datos hecha para buscar por significado a escala |
| **Collection** | El equivalente a una tabla |
| **Vectorizer** | El componente que convierte tus textos en vectores al insertarlos |
| **`alpha`** | La perilla que decide cuánto pesa el significado vs. las palabras exactas |
| **Filtered search** | Aplicar los filtros *durante* la búsqueda, no después |
| **Flat search cutoff** | El umbral bajo el cual conviene buscar a la antigua |

---

## 11. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Sé explicar por qué kNN es exacto y a la vez inviable a escala.
- [ ] Entiendo qué sacrifica ANN y por qué el sacrificio suele valer la pena.
- [ ] Sé que la "aproximación" del índice debe **medirse** (recall del índice), no asumirse.
- [ ] Puedo describir cómo se construye y cómo se recorre un proximity graph.
- [ ] Entiendo por qué HNSW empieza por la capa más rala y qué gana con eso.
- [ ] Sé por qué el runtime de HNSW es logarítmico y el de kNN lineal.
- [ ] Puedo argumentar cuándo **no** hace falta una vector database.
- [ ] Entiendo que insertar en una vector database implica vectorizar + actualizar el índice.
- [ ] Sé qué propiedades conviene vectorizar y cuáles solo filtrar.
- [ ] Distingo los cuatro modos de query y sé que todos aceptan filtros.
- [ ] Tengo claro que **`alpha` ≡ `beta`**, que sube el peso *semántico*, y que hay que verificar el sentido.
- [ ] Sé que sin pedir metadata (`score`, `distance`) estoy tuneando a ciegas.
- [ ] Entiendo la diferencia entre las perillas de build time (`ef_construction`, `max_connections`) y de query time (`ef`).
- [ ] Puedo explicar por qué BM25 devolvió 1 documento pidiendo 4, y qué demuestra eso.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search, embeddings y hybrid search]] (el supuesto que este tomo vino a arreglar)
- Siguiente tomo → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] (qué se indexa, antes de indexarlo)
- Query parsing, cross-encoders y re-ranking → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]]
- Las dos fases (indexing vs retrieval) → [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01]]
- Keyword search, `k1`/`b` y pre/post-filtering → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]]
- Comprimir vectores para ahorrar memoria → [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11]]
- Medir el retriever de punta a punta → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 3: Information Retrieval with Vector Databases (Coursera). Lecciones de introducción al módulo, ANN/HNSW y vector databases; **Ungraded Lab 1** (Weaviate API), del que provienen todo el código y los parámetros de las secciones 6 y 7.

**Fuentes externas (complemento con bibliografía verificable):**
- Malkov, Y. A. & Yashunin, D. A. (2020). *Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs*. IEEE Transactions on Pattern Analysis and Machine Intelligence, 42(4), 824–836. — El paper de HNSW.
- Malkov, Y., Ponomarenko, A., Logvinov, A. & Krylov, V. (2014). *Approximate Nearest Neighbor Algorithm Based on Navigable Small World Graphs*. Information Systems, 45, 61–68. — NSW, el antecesor.
- Johnson, J., Douze, M. & Jégou, H. (2019). *Billion-Scale Similarity Search with GPUs*. IEEE Transactions on Big Data. — FAISS.
- Aumüller, M., Bernhardsson, E. & Faithfull, A. (2020). *ANN-Benchmarks: A Benchmarking Tool for Approximate Nearest Neighbor Algorithms*. Information Systems, 87. — El benchmark independiente de recall vs. throughput.
- Documentación oficial de **Weaviate** (cliente Python v4): `connect_to_embedded`, `Configure.NamedVectors`, `query.near_text/bm25/hybrid`, `Filter`, `Rerank`, `MetadataQuery`, y los defaults de HNSW e `inverted_index_config`.

> [!note] Sobre el código y los valores de este tomo
> - **Del curso:** toda la teoría de kNN/ANN/NSW/HNSW y el workflow de vector database provienen de las lecciones. El código de las secciones 6 y 8 y **todos** los parámetros de la sección 7 son literales del Ungraded Lab 1 y de su dump de schema.
> - **Complemento propio, marcado como tal en el texto:** los snippets con `MetadataQuery` (6.5 y 8), la tabla de familias de ANN (4.1), las guías de decisión (5 y 9), y las tres trampas de `alpha` (6.5). Los **tres bugs y las dos omisiones del lab** que se documentan (variable `result` vs `response` en el rerank, `alpha` explicado al revés, metadata nunca solicitada, BM25 devolviendo 1 de 4, resultados semánticos que contradicen la query) son hallazgos propios verificados contra el notebook.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]]**, donde retrocedemos un paso en el pipeline: antes de indexar hay que decidir **en qué pedazos** se parte cada documento. Es una de las decisiones que más mueve la calidad de un RAG — y la razón por la que el truncation silencioso del Tomo 04 no es un problema fatal.
