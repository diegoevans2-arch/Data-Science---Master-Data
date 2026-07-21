---
title: "Tomo 04 — Semantic search, embeddings y hybrid search"
tags: [rag, semantic-search, embeddings, vector-space, cosine-similarity, contrastive-training, hybrid-search, rrf, dense-retrieval]
audiencias: [tecnico, puente, ejecutivo]
tomo: 04
version: 1.1
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 04 — Semantic search, embeddings y hybrid search

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03 · Keyword search: TF-IDF y BM25]] · Siguiente → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]]

---

> [!info] ¿Por qué importa esta sección?
> El [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]] terminó con un hueco preciso: keyword search no puede conectar `"happy"` con `"glad"`, y en cambio sí confunde el `Python` lenguaje con el `Python` serpiente. Este tomo cierra ese hueco y **completa el retriever**. Aquí está la tecnología que la mayoría de la gente asocia con "IA en la búsqueda" —los embeddings— explicada hasta el fondo: qué son esos vectores, por qué funcionan, cómo se entrenan y, crucialmente, **qué NO capturan**.
>
> Y termina montando la pieza final: **hybrid search**, el mecanismo que combina las dos búsquedas en un único ranking. Al cerrar este tomo tienes el retriever completo de punta a punta.

> [!abstract] 👔 Impacto ejecutivo
> Semantic search es lo que permite que un usuario pregunte **con sus propias palabras** —sin conocer la jerga interna, sin adivinar el título exacto del documento— y aun así encuentre lo que necesita. Es la diferencia entre un buscador que exige que el usuario hable como el sistema y uno que se adapta al usuario.
> - **Decisiones que habilita:** desplegar asistentes conversacionales sobre documentación que el usuario no conoce; atender consultas de clientes que describen un problema sin usar el vocabulario técnico del manual; unificar búsqueda sobre corpus escritos por áreas distintas con vocabularios distintos.
> - **Costo o riesgo de hacerlo mal:** los embeddings son **opacos**. No puedes explicar por qué un documento quedó primero, y el sistema puede fallar silenciosamente con terminología muy específica o con siglas propias de tu negocio. Apostar todo a semantic search y eliminar keyword search es el error clásico — y produce un sistema que falla justo en las consultas más precisas.
> - **Pregunta ejecutiva que responde:** *"¿Cómo hago que la búsqueda entienda lo que el usuario quiso decir, sin obligarlo a saber cómo lo llamamos internamente?"*

---

## 1. 🎯 El problema a resolver

Audiencia: 🔧 🧭 👔

Recordemos el límite estructural de keyword search con dos ejemplos que lo dejan desnudo:

```
   ❌ FALSO NEGATIVO — mismo significado, palabras distintas

      Query:      "estoy happy con el resultado"
      Documento:  "el cliente quedó glad con el resultado"
      → sinónimos perfectos, cero palabras en común, NO lo encuentra


   ❌ FALSO POSITIVO — misma palabra, significado distinto

      Query:      "cómo instalar Python en Windows"
      Documento:  "el Python reticulado puede medir 6 metros"
      → coincidencia literal perfecta, documento completamente irrelevante
```

Semantic search ataca ambos casos porque **empareja por significado compartido**, no por caracteres compartidos.

---

## 2. 🧭 Semantic search: la idea

Audiencia: 🔧 🧭

**🔧 Definición técnica:** a alto nivel, semantic search funciona **exactamente igual** que keyword search: cada documento se mapea a un vector, el prompt también, se comparan los vectores para generar scores y se devuelven los documentos más cercanos.

La única diferencia —y lo cambia todo— es **cómo se construye el vector**:

| | Keyword search | Semantic search |
|---|---|---|
| **Cómo se genera el vector** | Contando cuántas veces aparece cada palabra del vocabulary | Pasando el texto por un **embedding model** |
| **Tipo de vector** | **Sparse** (disperso): decenas de miles de posiciones, casi todas en cero | **Dense** (denso): cientos o miles de posiciones, todas con valor |
| **Qué representa cada posición** | Una palabra concreta del vocabulary | Nada interpretable por separado |
| **Qué captura** | Presencia y frecuencia de palabras | Significado |

### 2.1 El espacio vectorial

Audiencia: 🔧 🧭 💡

**🔧 Definición técnica:** un **embedding model** mapea un texto a una **ubicación en un espacio**. Esa ubicación se expresa como un vector. Lo notable —lo que casi parece magia— es que el modelo ubica **textos de significado similar en posiciones cercanas**.

```
   Espacio vectorial en 2D (simplificación radical)

        y
        │           ● cuisine
        │        ● food
        │     ● pizza
        │
        │                          ● bear
        │                       ● lion
        │
        │                                    ● trombone
        │                                       ● cat  ← ni cerca del trombón
        └──────────────────────────────────────────────► x

   "pizza", "food" y "cuisine" caen juntos porque significan cosas parecidas.
   "trombone" y "cat" caen lejos entre sí: no tienen nada que ver.
```

> [!warning] Los ejes NO significan nada
> Es la confusión más común. **No hay un "eje de comida" y un "eje de animales".** Los ejes `x` e `y` no tienen ninguna interpretación simple. Lo único que importa es la **posición relativa**: qué tan cerca está un punto de otro. No busques significado en las coordenadas individuales; búscalo en las distancias.

### 2.2 Por qué tantas dimensiones

Audiencia: 🔧 🧭

En dos dimensiones no hay espacio suficiente para acomodar todas las relaciones que tiene el lenguaje. Con tres dimensiones ya hay más lugar para que se formen clusters de conceptos relacionados. Los modelos reales usan **cientos o miles de componentes**.

```
   2 dimensiones     →  un plano. Los clusters se pisan unos a otros.
   3 dimensiones     →  un volumen. Más espacio, más matices.
   768 dimensiones   →  imposible de graficar o incluso de imaginar...
                        ...pero matemáticamente valen los mismos principios.
```

> [!tip] 💡 Analogía
> Imagina que tienes que **ubicar a todos tus conocidos en una sala** de modo que quienes se parecen queden cerca. Con una sala rectangular (2D) te vas a frustrar rápido: tu primo es parecido a tu hermano *y* a tu compañero de trabajo, pero ellos dos no se parecen entre sí — no hay dónde ponerlo. Si en vez de una sala tuvieras un espacio de 768 dimensiones, siempre habría una dirección libre para acomodar cada matiz sin romper las demás cercanías. **Las dimensiones extra no son complejidad gratuita: son espacio para los matices.**

### 2.3 No solo palabras

Audiencia: 🔧 🧭

Existen embedding models para **palabras individuales, oraciones y hasta documentos completos**. Cambia el tipo de entrada, pero la salida es siempre la misma: **un único vector** que indica un punto en el espacio.

El ejemplo canónico del curso, con tres frases:

```
   A: "He spoke softly in class."
   B: "He whispered quietly during class."
   C: "Her daughter brightened the gloomy day."

              ● A
             ╱          A y B: casi las mismas palabras NO,
            ● B         pero el mismo significado SÍ → cerca


                                          ● C   ← lejos de ambas
```

Fíjate en el detalle: A y B comparten poquísimas palabras (`he`, `class`). Un keyword search las vería casi como no relacionadas. El embedding model las pone juntas porque **quieren decir lo mismo**.

---

## 3. 📐 Medir la cercanía

Audiencia: 🔧

Para cuantificar cuán similares son dos textos, se mide la distancia entre sus vectores. Hay tres métricas de uso habitual.

### 3.1 Euclidean distance

**🔧 Definición:** la distancia en línea recta entre dos vectores — la más corta posible. Es el teorema de Pitágoras escalado a más dimensiones.

```
   euclidean(q, d) = √( Σ (qⱼ - dⱼ)² )
                        j

   Menor distancia = más similares.
```

### 3.2 Cosine similarity

**🔧 Definición:** mide la similitud en la **dirección** de dos vectores, **sin importar** cuán lejos estén entre sí en el espacio.

```
   cosine_sim(q, d) = (q · d) / (‖q‖ · ‖d‖)

   Rango:  +1  → apuntan exactamente en la misma dirección
            0  → perpendiculares (sin relación)
           -1  → direcciones exactamente opuestas
```

> [!tip] 💡 Analogía
> La diferencia entre las dos métricas es la diferencia entre preguntar **"¿qué tan lejos estás de mí?"** (euclidean) y **"¿estamos mirando hacia el mismo lado?"** (cosine). Dos personas pueden estar a cuadras de distancia pero caminando en la misma dirección; para efectos de "¿vamos al mismo lugar?", la dirección importa más que la distancia.

Es la métrica **más usada** en semantic search, por una razón concreta que veremos en 3.4.

### 3.3 Dot product

**🔧 Definición:** mide el largo de la proyección de un vector sobre el otro. Si dos vectores son similares en **largo y dirección**, la proyección es grande. A 90° da cero. En direcciones opuestas da negativo.

```
   dot(q, d) = Σ qⱼ · dⱼ
               j

   Rango: de -∞ a +∞  (sin cota, a diferencia de cosine)
```

> [!note] La relación entre las tres
> `cosine_sim` es literalmente el `dot product` **normalizado** por las magnitudes. Si los vectores ya vienen normalizados a largo 1 —cosa que hacen muchos embedding models modernos, incluido el `bge` del lab—, entonces **dot product y cosine similarity dan exactamente el mismo ranking**, y el dot product es más barato de computar. Por eso muchas vector databases ofrecen ambos y recomiendan normalizar al indexar.

### 3.4 Cuándo discrepan (y por qué importa)

Audiencia: 🔧 🧭

Elegir métrica no es un detalle cosmético: **cambia el ranking**. Este es el ejemplo del Ungraded Lab 1, verificado:

```python
v1  = [1, 2]
arr = [[3, 2], [5, 6]]
```

| Vector | Cosine similarity | Euclidean distance |
|---|---|---|
| `[3, 2]` | 0.8682 | **2.0000** ← el más cercano |
| `[5, 6]` | **0.9734** ← el más similar | 5.6569 |

**Las dos métricas eligen ganadores distintos.** `[5,6]` está mucho más lejos en el espacio, pero apunta casi en la misma dirección que `v1`. Cosine premia la dirección; euclidean premia la proximidad.

El caso extremo lo deja clarísimo:

```
   cosine_sim([10,10], [100,100])  = 1.000000   ← idénticos en dirección
   euclidean ([10,10], [100,100])  = 127.2792   ← lejísimos en el espacio
```

### 3.5 🔬 Por qué cosine gana en alta dimensión

Audiencia: 🔧 🧭

El curso menciona que *"en espacios de muy alta dimensión, todo punto tiende a estar bastante lejos de todo otro punto"*. Vale la pena ver **cuán cierto** es, porque es la justificación real de por qué cosine similarity se impuso.

Midiendo la distancia euclidiana de un punto a 1.000 puntos aleatorios, a medida que sube la dimensión:

| Dimensiones | d_mín | d_máx | d_media | **Contraste** `(máx−mín)/media` |
|---:|---:|---:|---:|---:|
| 2 | 0.008 | 0.963 | 0.487 | **1.961** |
| 3 | 0.071 | 1.340 | 0.744 | **1.704** |
| 10 | 0.438 | 1.940 | 1.210 | **1.242** |
| 100 | 3.355 | 4.819 | 4.196 | **0.349** |
| 768 | 10.576 | 11.912 | 11.226 | **0.119** |
| 3072 | 22.020 | 23.354 | 22.653 | **0.059** |

> [!danger] La maldición de la dimensionalidad, en números
> En **768 dimensiones** —justamente las que usa el modelo del lab— el punto más cercano y el más lejano difieren en apenas un **12 % de la distancia media**. En 2 dimensiones esa diferencia era del **196 %**.
>
> Dicho de otro modo: **en alta dimensión, "el más cercano" deja de ser una distinción significativa** cuando la mides en línea recta. Todos los puntos se apelotonan a una distancia parecida. Este fenómeno está formalmente caracterizado en la literatura (Beyer et al., 1999) y es la razón práctica por la que la industria se estandarizó en **cosine similarity**: la *dirección* sigue discriminando bien donde la *distancia* ya no lo hace.

> [!abstract] 👔 En una frase para el negocio
> La métrica con que el sistema decide "qué se parece a qué" es una decisión de arquitectura que cambia los resultados que ve el usuario. No es un parámetro que se deje por defecto sin pensarlo.

---

## 4. 🎓 Cómo se entrena un embedding model

Audiencia: 🔧 🧭

El trabajo del modelo se enuncia fácil: **embeber texto similar a vectores cercanos, y texto distinto a vectores lejanos**. Cómo lo logra es lo interesante.

### 4.1 Positive pairs y negative pairs

**🔧 Definición técnica:** el entrenamiento se plantea en términos de pares:

```
   POSITIVE PAIR   "good morning"  ↔  "hello"
                   significados similares → deben quedar CERCA

   NEGATIVE PAIR   "good morning"  ↔  "that's a noisy trombone"
                   significados distintos → deben quedar LEJOS
```

El primer paso es compilar una colección enorme de estos pares, llamados **examples**. En sistemas reales hablamos de **millones de pares**. Un mismo texto aparece en muchos ejemplos, para capturar su relación con una gran variedad de conceptos.

### 4.2 Contrastive training

**🔧 Definición técnica:** al comenzar, el modelo embebe cada texto en un vector **aleatorio**. Son vectores sin sentido: un retriever construido con ese modelo devolvería basura. Entonces empieza el ciclo:

```
   ┌─────────────────────────────────────────────────────────┐
   │                                                         │
   │  1. Embeber todos los textos con los parámetros         │
   │     actuales del modelo                                 │
   │                        │                                │
   │                        ▼                                │
   │  2. Evaluar: ¿qué tan bien quedaron los positive        │
   │     pairs juntos y los negative pairs separados?        │
   │                        │                                │
   │                        ▼                                │
   │  3. Actualizar los parámetros internos para acercar     │
   │     los positive y alejar los negative                  │
   │                        │                                │
   └────────────────────────┼────────────────────────────────┘
                            │
                            └──► repetir muchísimas veces
```

Como el modelo se evalúa usando el **contraste** entre ejemplos positivos y negativos, la técnica se llama **contrastive training**.

### 4.3 Desde la perspectiva de un solo texto

Audiencia: 🔧 💡

Tomemos un texto como **anchor** (ancla) y veamos qué le pasa:

```
   ANCHOR:    "he could smell the roses"
   POSITIVE:  "a field of fragrant flowers"
   NEGATIVE:  "a lion roared majestically"


   ANTES del entrenamiento          DESPUÉS del entrenamiento
   (ubicaciones aleatorias)         (el ancla tiró y empujó)

      ● positive                        ● negative
                ● anchor
        ● negative                              ● anchor
                                                ● positive
```

Desde el punto de vista del ancla: **quiere tirar del positive hacia sí y empujar el negative lejos**.

> [!tip] 💡 Analogía
> Es una **fiesta con imanes**. Cada frase lleva un imán que la atrae hacia las frases que significan algo parecido y la repele de las que no. Al principio están todos parados al azar. Dejas actuar los imanes millones de veces y, sin que nadie dirija el proceso, la sala termina organizada en grupitos temáticos. Nadie decidió dónde iba el grupo de "flores": **se formó solo**.

Con tres frases el proceso es fácil de visualizar. Con **millones** de tripletas anchor/positive/negative se vuelve mucho más caótico: cada vector está siendo tironeado en muchas direcciones a la vez. Y ahí está la razón profunda de la alta dimensionalidad: **el espacio de muchas dimensiones le da al algoritmo opciones para acomodar todos esos tirones sin contradicciones**.

### 4.4 Las tres conclusiones que hay que llevarse

Audiencia: 🔧 🧭

> [!warning] 1. Las posiciones son abstractas y arbitrarias
> Antes de entrenar, una ubicación del espacio no significa nada. Después de entrenar, **sí** significa algo — pero **solo porque ahí se formó un cluster de conceptos similares**. Si repitieras el entrenamiento con otros valores iniciales aleatorios, **los mismos clusters se formarían en lugares distintos del espacio**. No hay nada intrínseco en las coordenadas.

> [!danger] 2. Solo se comparan vectores del MISMO embedding model
> Cada modelo se entrenó con datos distintos, con distinto número de dimensiones y con inicializaciones aleatorias distintas. **Comparar vectores de dos modelos distintos produce puro ruido.** En la práctica esto significa que si cambias de embedding model, **tienes que re-indexar toda tu knowledge base**. No es un detalle menor: es un costo de migración real que hay que presupuestar.

> [!note] 3. No necesitas entrenar uno
> Para construir un sistema RAG usarás **modelos off-the-shelf**, y hacen un trabajo notablemente bueno. Tampoco vas a implementar las métricas de distancia a mano. Entender el mecanismo sirve para **razonar mejor sobre sus resultados y sus fallas**, no para reimplementarlo.

---

## 5. 💻 Embeddings en la práctica

Audiencia: 🔧

El Ungraded Lab 1 del Módulo 2 usa **`BAAI/bge-base-en-v1.5`**, un modelo basado en transformer capaz de embeber oraciones completas, que genera vectores de **768 dimensiones**.

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("BAAI/bge-base-en-v1.5")

# Un string → un vector de 768 dimensiones
res = model.encode("RAG is awesome")
print(res.shape)                      # (768,)

# Una lista de strings → una matriz (n_textos × 768)
print(model.encode(["apple", "car"]).shape)    # (2, 768)
```

### 5.1 Qué captura realmente

Similitudes reales medidas en el lab, tomando `apple` como referencia:

| Comparación | Cosine similarity | Euclidean distance |
|---|---:|---:|
| apple ↔ apple | 1.0000 | 0.0000 |
| apple ↔ **fruit** | **0.7461** | **0.7126** |
| apple ↔ automobile | 0.6485 | 0.8384 |
| apple ↔ car | 0.5749 | 0.9221 |
| apple ↔ love | 0.5540 | 0.9445 |
| apple ↔ sentiment | 0.5020 | 0.9980 |

El modelo agrupa correctamente `apple` con `fruit` y aleja `sentiment`. Y nota la coherencia: **el orden de ambas columnas es idéntico** (una asciende mientras la otra desciende).

> [!warning] ⚠️ Los valores absolutos de cosine engañan
> El rango teórico de cosine similarity es de **−1 a +1**, pero mira los números reales: **todos caen entre 0.50 y 1.00**. Con modelos de sentence embedding modernos, incluso pares sin ninguna relación (`apple` ↔ `sentiment`) dan **0.50**, no 0.
>
> **Consecuencia práctica:** un umbral absoluto del tipo *"acepta solo documentos con similarity > 0.8"* es una heurística frágil y no transferible entre modelos. Lo que importa es el **ranking relativo** dentro de una misma consulta, no el valor absoluto. Si vas a usar un umbral, **calíbralo empíricamente contra tu corpus y tu modelo**, y vuelve a calibrarlo si cambias cualquiera de los dos.

### 5.2 Un retriever semántico completo

```python
def retrieve_relevant(query, documents, metric='cosine_similarity'):
    """
    Rankea documentos por similitud con la query.
    Ojo con el sentido del orden: cosine ordena DESCENDENTE (más alto = mejor),
    euclidean ordena ASCENDENTE (más bajo = mejor).
    """
    query_emb     = model.encode(query)
    documents_emb = model.encode(documents)

    if metric == 'cosine_similarity':
        distances = cosine_similarity(query_emb, documents_emb)
        vals = [(doc, d) for doc, d in zip(documents, distances)]
        vals.sort(reverse=True, key=lambda x: x[1])   # descendente
    elif metric == 'euclidean':
        distances = euclidean_distance(query_emb, documents_emb)
        vals = [(doc, d) for doc, d in zip(documents, distances)]
        vals.sort(key=lambda x: x[1])                 # ascendente
    return vals
```

> [!tip] 🧭 El error de signo más común en RAG
> Ordenar al revés. Con **cosine similarity, más alto es mejor**; con **euclidean distance, más bajo es mejor**. Invertir el criterio produce un retriever que devuelve sistemáticamente **lo menos relevante** — y como igual devuelve documentos con aspecto normal, el bug puede pasar inadvertido mucho tiempo. Es la clase de error que solo se detecta midiendo.

### 5.3 🔬 Honestidad sobre qué NO capturan

Audiencia: 🔧 🧭 👔

El propio lab expone un resultado incómodo y vale oro. Con la query *"Suggest to me great places to visit in Asia"* sobre una lista de destinos turísticos, el ranking sale:

```
   1. 0.6081  The Great Wall of China ...          ✓ Asia
   2. 0.5821  Mt. Fuji ...                         ✓ Asia
   3. 0.5605  The Maldives ...                     ✓ Asia
   4. 0.5512  Santorini ...                        ✗ Grecia
   5. 0.5220  Banff National Park ...              ✗ Canadá
   6. 0.5150  Kyoto's cherry blossoms ...          ✓ Asia  ← ¿en sexto lugar?
```

**Kyoto —indiscutiblemente Asia— queda por debajo de Santorini y Banff.** La explicación del propio curso: los embedding models capturan similitud según **los contextos en que las palabras aparecen juntas** en su training data. Aunque Kyoto sea factualmente más relevante, el modelo aprendió asociaciones más fuertes entre frases genéricas de viaje (*"places to visit"*) y destinos como Santorini o Banff, por su co-ocurrencia frecuente en textos turísticos.

> [!danger] La lección que hay que internalizar
> **Un embedding model no razona sobre hechos: reproduce patrones estadísticos de co-ocurrencia de su entrenamiento.** "Kyoto está en Asia" es conocimiento factual; el modelo no lo consulta, solo tiene una noción difusa de qué textos suelen aparecer cerca de cuáles.
>
> Por eso semantic search **no reemplaza** a keyword search ni al metadata filtering. Si necesitabas garantizar "solo destinos de Asia", eso era un **filtro de metadata** (`continente = "Asia"`), no una esperanza depositada en el embedding. Cada técnica cubre lo que la otra no puede — y este ejemplo es la demostración más limpia de por qué.

### 5.4 ⚠️ La trampa del truncation

Audiencia: 🔧 🧭 👔

Los embedding models tienen un **límite de entrada**, igual que los LLMs. Lo que excede ese límite **se descarta en silencio**. El lab lo demuestra de forma brutal:

```python
big_text = open("large_text.txt").read()
len(big_text)                                        # 3955 caracteres

emb_completo = model.encode(big_text)
emb_recortado = model.encode(big_text[:3000])        # 955 caracteres MENOS

np.array_equal(emb_completo, emb_recortado)          # True
```

**El mismo vector. Ni un solo elemento distinto.** El context window de `bge-base-en-v1.5` es de **512 tokens**; todo lo que viene después es ignorado por completo.

> [!danger] Por qué esto es peligroso
> No hay error. No hay warning. No hay excepción. El modelo devuelve un vector **perfectamente válido** que simplemente **no representa** buena parte del documento. Si indexas documentos largos sin partirlos:
> - Todo el contenido más allá del token 512 es **invisible para el retriever**. Existe en tu knowledge base pero es irrecuperable.
> - Dos documentos que solo difieren después del corte reciben **vectores idénticos**.
>
> Esta es exactamente la razón por la que existe el **chunking**, y por qué es un tomo entero: [[Guia-Maestra-RAG_06-Chunking|Tomo 06]]. **Verifica siempre el límite de tokens de tu embedding model antes de indexar nada.**

### 5.5 Visualizar el espacio: PCA

Audiencia: 🔧

768 dimensiones no se pueden graficar. El lab usa **PCA (Principal Component Analysis)** para proyectarlas a 2, lo suficiente para ver los clusters:

```python
from sklearn.decomposition import PCA

embeddings = model.encode(sentences)
embeddings_2d = PCA(n_components=2).fit_transform(embeddings)
```

> [!note] Qué es y qué no es esta visualización
> PCA **descarta información** para poder dibujar: comprime 768 dimensiones en 2. Sirve para **intuir** que se forman clusters, no para juzgar distancias reales. Dos puntos que en el gráfico se ven pegados pueden estar lejos en el espacio original. Úsalo como herramienta didáctica y de exploración; **nunca para depurar un ranking**.

---

## 6. 🔀 Hybrid search: armando el retriever completo

Audiencia: 🔧 🧭 👔

Ya tenemos las tres técnicas. Esta sección las une — y con esto el retriever queda completo.

### 6.1 Repaso de las tres piezas

| Técnica | Fortaleza | Límite | Velocidad |
|---|---|---|---|
| **Metadata filtering** | Único que da un **sí/no rígido** | No busca; ignora el contenido | Muy rápida |
| **Keyword search** | Coincidencia **exacta**; excelente con términos técnicos y nombres de producto | No encuentra significados equivalentes con otras palabras | Rápida |
| **Semantic search** | Flexibilidad por **significado**; ninguna otra la ofrece | Más lenta y más costosa computacionalmente; opaca | Lenta |

### 6.2 El pipeline

```
   Prompt
     │
     ├──────────────────────┐
     ▼                      ▼
  KEYWORD               SEMANTIC          ← ambas búsquedas, en paralelo
  SEARCH                SEARCH
     │                      │
   50 docs               50 docs          ← cada una devuelve su ranking
     │                      │
     ▼                      ▼
  METADATA              METADATA          ← se filtran por criterios rígidos
  FILTER                FILTER
     │                      │
   35 docs               30 docs          ← quedan menos, y en distinto orden
     │                      │
     └──────────┬───────────┘
                ▼
     RECIPROCAL RANK FUSION               ← se combinan en UN ranking
                │
                ▼
            top_k docs  ──►  augmented prompt
```

Muchos documentos aparecerán en **ambas** listas, pero rankeados distinto — porque el criterio de scoring era distinto. El problema a resolver: **cómo combinar dos rankings en uno solo.**

### 6.3 Reciprocal Rank Fusion (RRF)

Audiencia: 🔧 🧭

**🔧 Definición técnica:** el algoritmo estándar para fusionar rankings. Premia a los documentos por quedar **bien rankeados en cualquiera de las listas**.

```
                        1
   RRF(d) =   Σ     ─────────
            listas   k + rank(d)

   donde:
     rank(d) = posición del documento d en esa lista (1 = primero)
     k       = hiperparámetro que controla cuánto domina el primer puesto
```

**Ejemplo del curso** (con `k = 0`): un documento que quedó **2º** en una lista y **10º** en la otra:

```
   1/(0+2)  +  1/(0+10)  =  0.5 + 0.1  =  0.6
```

> [!warning] Lo que RRF ignora deliberadamente
> **RRF solo mira la POSICIÓN, no el score que la produjo.** Si el documento nº1 sacó un score altísimo y el nº2 uno mediocre, esa diferencia **se descarta**: solo cuenta que uno fue primero y el otro segundo.
>
> Suena a pérdida de información, pero es exactamente el punto: los scores de BM25 y los de cosine similarity **viven en escalas incomparables** (BM25 no tiene cota superior; cosine va de −1 a 1). Normalizarlos entre sí es frágil y depende del corpus. **Comparar posiciones, en cambio, siempre funciona.** Esa robustez es la razón de su popularidad.

### 6.4 El hiperparámetro `k`

**🔧 Qué controla:** cuánto pesa estar en el primer puesto. Verificado numéricamente:

| `k` | Puesto 1 | Puesto 10 | Ventaja del 1º |
|---:|---:|---:|---:|
| 0 | 1.0000 | 0.1000 | **10.00×** |
| 10 | 0.0909 | 0.0500 | 1.82× |
| 50 | 0.0196 | 0.0167 | 1.18× |
| 60 | 0.0164 | 0.0143 | **1.15×** |

Con `k = 0`, el documento mejor rankeado en **cualquiera** de las listas se dispara al tope del ranking global, aunque solo una lista lo haya destacado. Subir `k` **aplana** esa ventaja: sigue conviniendo ser primero, pero ya no domina.

> [!note] El valor por defecto de la industria es `k = 60`
> Proviene del paper original de RRF (Cormack et al., 2009) y es el default en la mayoría de las implementaciones. El curso ilustra la mecánica con `k = 0` y `k = 50` por claridad pedagógica; para un sistema real, **`k = 60` es el punto de partida sensato**.
>
> Nota de precisión al hacer cuentas a mano: la fórmula es `1/(k + rank)`, así que con `k = 50` el primer puesto aporta `1/51 ≈ 0.0196` (no `1/50`) y el décimo aporta `1/60 ≈ 0.0167`.

### 6.5 RRF sobre datos reales: el caso del assignment

Audiencia: 🔧 🧭 👔

El assignment graded C1M2 implementa las tres piezas sobre el dataset de 870 noticias y deja ver el comportamiento de RRF mejor que cualquier ejemplo sintético. Para la query **`"What are the recent news about GDP?"`**, cada técnica devuelve su propio `top_5` (índices de documento):

```
   SEMANTIC SEARCH :  [743, 673, 626, 752, 326]
   BM25            :  [752, 673, 289, 626,  43]
                       ▲         ▲    ▲
                       │         │    └── solo en una lista
                       │         └─────── en ambas
                       └───────────────── #1 en BM25, #4 en semantic
```

**Solo 3 de 5 documentos coinciden.** Las dos técnicas, sobre la misma query y el mismo corpus, encuentran cosas distintas — que es exactamente el argumento para usar ambas.

Al fusionarlas con RRF (`K=60`, sin pesos), el resultado —ejecutado y **coincidente con la salida esperada del assignment**— es:

| Doc | Puesto semantic | Puesto BM25 | Score RRF | Posición final |
|---:|:---:|:---:|---:|:---:|
| **673** | 2º | 2º | 0.032258 | **1º** |
| **752** | 4º | **1º** | 0.032018 | **2º** |
| **626** | 3º | 4º | 0.031498 | **3º** |
| 743 | **1º** | — | 0.016393 | 4º |
| 289 | — | 3º | 0.015873 | 5º |

> [!important] Lee esa tabla dos veces: es toda la tesis de RRF
> - **`doc 743` era el número 1 de semantic search** — y termina **cuarto**, porque BM25 no lo encontró.
> - **`doc 673` no fue primero en ninguna lista** (2º y 2º) — y termina **primero**.
> - Los tres documentos que aparecen en **ambas** listas ocupan **las tres primeras posiciones**, por encima de cualquier documento que solo una técnica encontró.
>
> **Estar en ambas listas vale más que ser primero en una.** RRF no premia la excelencia en una técnica: premia el **consenso entre técnicas**. Y esa es justamente la propiedad que quieres, porque un documento que tanto la búsqueda literal como la semántica consideran relevante es una apuesta mucho más segura que uno que solo brilla en una.
>
> Fíjate también en lo ajustado del top 3: 0.032258 vs 0.032018 vs 0.031498. Con `K=60` los scores quedan muy comprimidos — exactamente el efecto de aplanamiento que describe la sección 6.4.

> [!note] El assignment implementa RRF **sin** `beta`
> La firma del ejercicio es `reciprocal_rank_fusion(list1, list2, top_k=5, K=60)`: **ambas listas pesan igual**. El `beta` que discute la clase teórica (siguiente sección) no se implementa en el código del curso. Vale saberlo para no buscarlo en el notebook — y para entender que la versión ponderada es un paso adicional que tendrás que agregar tú en un sistema real.
>
> Nota de validación: `K=60` no es un valor que yo trajera de fuera — **es el default del propio assignment**, y coincide con el del paper original de RRF (Cormack et al., 2009).

### 6.6 El hiperparámetro `beta`: el balance del sistema

Audiencia: 🔧 🧭 👔

**🔧 Definición:** `beta` pondera cuánto pesa cada ranking en la fusión. Es **la perilla más importante de todo el retriever**.

```
   score_final(d) = beta · RRF_semantic(d)  +  (1 - beta) · RRF_keyword(d)

   beta = 0.8  →  80 % semantic, 20 % keyword
   beta = 0.7  →  punto de partida recomendado por el curso
   beta = 0.0  →  keyword search puro
   beta = 1.0  →  semantic search puro
```

Verificado sobre dos rankings de ejemplo — fíjate cómo **el ganador cambia según `beta`**:

```
   keyword_ranking  = [docA, docB, docC, docD, docE]
   semantic_ranking = [docC, docE, docA, docF, docB]

   beta=0.0  →  docA > docB > docC > docD > docE > docF     (keyword puro)
   beta=0.3  →  docA > docC > docB > docE > docD > docF
   beta=0.5  →  docC > docA > docE > docB > docF > docD     ← cambia el ganador
   beta=0.7  →  docC > docA > docE > docB > docF > docD
   beta=1.0  →  docC > docE > docA > docF > docB > docD     (semantic puro)
```

`docA` era **1º en keyword y 3º en semantic**; `docC` era **3º en keyword y 1º en semantic**. Con `beta` bajo gana `docA`; al cruzar `beta = 0.5`, gana `docC`. **La misma query, el mismo corpus, distinta respuesta** — según una sola perilla.

Nota también el comportamiento de los extremos: `docF` (solo en la lista semántica) y `docD` (solo en la keyword) **quedan al fondo en las configuraciones balanceadas**. RRF premia estructuralmente a los documentos que **ambas** técnicas consideran relevantes.

> [!tip] 🧭 Cómo elegir `beta`
> | Tu caso | `beta` sugerido | Por qué |
> |---|---|---|
> | Documentación técnica, catálogos con SKU, normativa, código | **0.3 – 0.5** (más keyword) | Los términos exactos son lo que el usuario busca y no admiten sinónimos |
> | Base de conocimiento general, FAQ, soporte conversacional | **0.7** (default del curso) | El usuario describe su problema con sus palabras |
> | Contenido narrativo, búsqueda exploratoria | **0.8 – 0.9** (más semantic) | El vocabulario exacto importa poco |
>
> **Y después mídelo.** `beta` es el parámetro que más se beneficia de ser tuneado empíricamente contra queries reales de tus usuarios.

### 6.7 RRF en código

```python
def rrf_score(rank, k=60):
    """Aporte de un documento que quedó en posición `rank` (1-indexado)."""
    return 1.0 / (k + rank)


def fusionar(rankings_con_peso, k=60):
    """
    Fusiona múltiples rankings en uno solo.
    rankings_con_peso: lista de tuplas (ranking, peso), ej. [(semantico, 0.7), (keyword, 0.3)]
    """
    total = {}
    for ranking, peso in rankings_con_peso:
        for pos, doc in enumerate(ranking, start=1):
            total[doc] = total.get(doc, 0.0) + peso * rrf_score(pos, k)
    return sorted(total.items(), key=lambda x: x[1], reverse=True)


# Uso: 70 % semantic, 30 % keyword
resultado = fusionar([(semantic_ranking, 0.7), (keyword_ranking, 0.3)])
top_k = [doc for doc, _ in resultado[:5]]
```

> [!info] Las perillas del retriever completo
> Cerrando el Módulo 2, estas son **todas** las palancas de ajuste que tienes:
> - **`k1` y `b`** de BM25 → saturación y penalización por largo ([[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]])
> - **Qué metadata** filtrar y **cuándo** aplicarla (pre vs. post-filtering)
> - **Qué embedding model** usar y su límite de tokens
> - **Qué métrica de distancia** (cosine / euclidean / dot)
> - **`k`** de RRF → cuánto domina el primer puesto
> - **`beta`** → el balance keyword ↔ semantic
> - **`top_k`** → cuántos documentos van al LLM
>
> Ninguna se deduce: todas **se miden**. Y para medirlas necesitas métricas de retrieval, que es lo que viene en [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]].

---

## 7. ⚖️ Fortalezas y debilidades de semantic search

Audiencia: 🔧 🧭 👔

> [!tip] ✅ Fortalezas
> - **Empareja por significado.** Resuelve sinónimos, paráfrasis y vocabulario distinto — el hueco exacto de keyword search.
> - **Desambigua por contexto.** `Python` lenguaje y `Python` serpiente caen en zonas distintas del espacio.
> - **Tolera cómo escribe la gente de verdad.** El usuario no necesita conocer la terminología interna del corpus.
> - **Flexibilidad que ninguna otra técnica ofrece.**

> [!warning] ⚠️ Debilidades
> - **Más lenta y más costosa** computacionalmente que keyword search: hay que ejecutar un modelo neuronal por cada texto.
> - **Opaca.** No puedes explicar por qué un documento quedó primero. Un problema real en contextos que exigen auditabilidad.
> - **No garantiza coincidencia exacta.** Con un SKU, una sigla interna o un código normativo puede fallar donde BM25 acierta trivialmente.
> - **Reproduce patrones, no hechos.** El caso Kyoto (5.3): asociación estadística ≠ conocimiento factual.
> - **Truncation silencioso.** Todo lo que exceda el límite de tokens desaparece sin aviso (5.4).
> - **Cambiar de modelo obliga a re-indexar** toda la knowledge base (4.4).

---

## 8. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Semantic search** | Buscar por significado en vez de por palabras exactas |
| **Embedding model** | El modelo que convierte texto en una lista de números que representa su significado |
| **Embedding / dense vector** | Esa lista de números. "Denso" porque todas las posiciones tienen valor |
| **Vector space** | El "mapa" donde cada texto ocupa un punto y los parecidos quedan cerca |
| **Cosine similarity** | Medir si dos textos "apuntan hacia el mismo lado". De −1 a 1; más alto = más parecido |
| **Euclidean distance** | Distancia en línea recta. Más baja = más parecido |
| **Dot product** | Cosine sin normalizar. Sin cota superior |
| **Maldición de la dimensionalidad** | En muchas dimensiones todo queda a distancia parecida, y "el más cercano" pierde sentido |
| **Contrastive training** | Entrenar acercando lo que se parece y alejando lo que no |
| **Anchor / positive / negative** | El texto de referencia, uno parecido y uno distinto: la tripleta de entrenamiento |
| **Truncation** | Cuando el modelo ignora en silencio el texto que excede su límite |
| **PCA** | Técnica para aplastar muchas dimensiones a 2 y poder dibujarlas |
| **Hybrid search** | Combinar búsqueda exacta + semántica + filtros en un ranking único |
| **RRF** | El algoritmo que fusiona dos rankings mirando posiciones, no scores |
| **`beta`** | La perilla que decide cuánto pesa el significado vs. las palabras exactas |

---

## 9. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Sé explicar los dos fallos de keyword search (falso negativo por sinónimos, falso positivo por homónimos).
- [ ] Entiendo que semantic search usa el mismo esqueleto que keyword search y que lo único que cambia es cómo se genera el vector.
- [ ] Distingo un sparse vector de un dense vector.
- [ ] Sé que los ejes del vector space no tienen interpretación y que solo importan las posiciones relativas.
- [ ] Puedo explicar la diferencia entre cosine similarity, euclidean distance y dot product, y cuándo dan rankings distintos.
- [ ] Entiendo por qué la alta dimensionalidad favorece a cosine similarity.
- [ ] Sé qué son los positive/negative pairs y en qué consiste el contrastive training.
- [ ] Tengo claro que **no se pueden comparar vectores de modelos distintos**, y que cambiar de modelo obliga a re-indexar.
- [ ] Sé por qué un umbral absoluto de cosine similarity es una heurística frágil.
- [ ] Entiendo el caso Kyoto: los embeddings capturan co-ocurrencia estadística, no hechos.
- [ ] Sé que el truncation es silencioso y por qué obliga a hacer chunking.
- [ ] Puedo calcular a mano un score de RRF y explicar qué controlan `k` y `beta`.
- [ ] Entiendo por qué RRF compara posiciones en vez de scores.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03 · Keyword search: TF-IDF y BM25]] (la otra mitad del hybrid search)
- Siguiente tomo → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]] (cómo se busca entre millones de vectores sin compararlos todos)
- **Consecuencia directa del truncation** → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] (cómo partir documentos que no caben)
- Mejorar el ranking antes del corte `top_k` → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Reranking]]
- **Medir el retriever** (precision@k, recall@k, MAP@K, MRR) → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Evaluación]]
- Técnicas avanzadas sobre esta base (HyDE, query transformation, GraphRAG) → [[Guia-Maestra-RAG_12-Tecnicas-Avanzadas-Hybrid-HyDE-GraphRAG|Tomo 12]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 2: Information Retrieval & Search Foundations (Coursera). Lecciones sobre semantic search, embedding models, hybrid search y RRF; Ungraded Lab 1 (vector embeddings) y **assignment graded C1M2** (semantic search con `bge-base-en-v1.5` + BM25 + RRF sobre 870 noticias).

**Fuentes externas (complemento con bibliografía verificable):**
- Cormack, G. V., Clarke, C. L. A. & Buettcher, S. (2009). *Reciprocal Rank Fusion Outperforms Condorcet and Individual Rank Learning Methods*. SIGIR '09. — Paper original de RRF; origen del valor por defecto `k = 60`.
- Beyer, K., Goldstein, J., Ramakrishnan, R. & Shaft, U. (1999). *When Is "Nearest Neighbor" Meaningful?*. ICDT. — Caracterización formal de la concentración de distancias en alta dimensión (sección 3.5).
- Reimers, N. & Gurevych, I. (2019). *Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks*. EMNLP. — Base de los sentence embedding models y de la librería `sentence-transformers` usada en el lab.
- Karpukhin, V. et al. (2020). *Dense Passage Retrieval for Open-Domain Question Answering*. EMNLP. — Dense retrieval aplicado a QA; contrastive training con negativos.
- Schroff, F., Kalenichenko, D. & Philbin, J. (2015). *FaceNet: A Unified Embedding for Face Recognition and Clustering*. CVPR. — Origen del esquema anchor / positive / negative (triplet loss) descrito en la sección 4.3.
- Mikolov, T. et al. (2013). *Efficient Estimation of Word Representations in Vector Space*. ICLR Workshop. — word2vec: el trabajo que popularizó la idea de significado como posición en un espacio vectorial.
- Documentación oficial: model card de `BAAI/bge-base-en-v1.5` (Hugging Face) — dimensiones y límite de tokens citados en este tomo.

> [!note] Sobre el código y los números de este tomo
> - **Del curso (fuente primaria):** los ejemplos con `sentence-transformers` y los valores de similitud de las secciones 5.1 y 5.3 provienen del **Ungraded Lab 1**; los rankings y la fusión de la sección 6.5 provienen del **assignment graded C1M2**, y fueron **re-ejecutados por mí para confirmar que reproducen la salida esperada del assignment** (`[673, 752, 626, 743, 289]`). La fórmula de RRF y el valor `K=60` están tomados del enunciado del propio assignment.
> - **Complemento propio, ejecutado y verificado** con `numpy 2.5.1`: las secciones 3.4 y 3.5 (discrepancia cosine/euclidean, concentración de distancias en alta dimensión), 6.4 (efecto de `K`) y 6.6 (efecto de `beta`). Ninguna salida de este tomo es ilustrativa.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]]**, donde arranca el Módulo 3 con el problema que este tomo dejó abierto: comparar la query contra **cada uno** de los vectores de la knowledge base es inviable con millones de documentos. La respuesta son los algoritmos de **Approximate Nearest Neighbor** y las **vector databases** construidas sobre ellos.
