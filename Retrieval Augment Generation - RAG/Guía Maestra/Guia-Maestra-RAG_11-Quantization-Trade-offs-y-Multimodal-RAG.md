---
title: "Tomo 11 — Quantization, trade-offs de cost/latency y multimodal RAG"
tags: [rag, quantization, int8, binary-quantization, matryoshka, cost, latency, caching, multi-tenancy, multimodal, vision-language-model, colpali]
audiencias: [tecnico, puente, ejecutivo]
tomo: 11
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 11 — Quantization, trade-offs de cost/latency y multimodal RAG

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · Producción: observability, evaluación y security]] · Siguiente → [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12 · ⭐ Complemento: técnicas avanzadas de query]]

---

> [!info] ¿Por qué importa esta sección?
> El [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]] instaló los instrumentos. Este tomo es el que **usa esas mediciones para tomar decisiones que cuestan dinero**: qué comprimir, qué modelo usar, qué guardar en memoria cara, qué cachear, y cuánta calidad estás dispuesto a ceder a cambio.
>
> El curso ordena el módulo así a propósito: *"Una vez que puedas evaluar tu sistema RAG y experimentar con configuraciones distintas, estarás listo para enfrentar algunos trade-offs familiares en muchos proyectos de software: **costo, velocidad y calidad**."* El orden no es casual — **optimizar sin observability es apostar**, y este tomo documenta un caso donde el propio curso cae en esa trampa.
>
> Y cierra con la frontera: qué pasa cuando lo que hay que buscar **no es texto**, sino una diapositiva, un gráfico o un PDF.

> [!abstract] 👔 Impacto ejecutivo
> Este es el tomo de la **factura mensual y del tiempo de espera**. Un sistema RAG que funciona pero cuesta el triple de lo previsto, o que tarda ocho segundos en responder, es un sistema que no se va a desplegar — aunque sus métricas de calidad sean excelentes.
> - **Decisiones que habilita:** poner un techo de costo por consulta y saber qué palanca moverlo; decidir entre endpoint por token o hardware dedicado; elegir cuánta calidad se cede por cuánto ahorro, **con números en vez de intuición**; incorporar al sistema documentación que hoy está en PDFs y presentaciones y por tanto es invisible.
> - **Costo o riesgo de hacerlo mal:** comprimir a ciegas degrada la calidad sin que nadie lo note (el fallo que este tomo documenta en el propio material del curso); no comprimir nada multiplica la factura de infraestructura por un factor de 4 a 32 sin necesidad.
> - **Pregunta ejecutiva que responde:** *"¿cuánto nos cuesta cada respuesta, cuánto tarda, y qué exactamente perdemos si lo bajamos a la mitad?"*

---

## 1. 🗜️ Quantization: comprimir modelos y vectores

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** *"En pocas palabras, quantization es **compresión** tanto para LLMs como para los vectores generados por embedding models. Quantization reemplaza los pesos del modelo dentro de un LLM, o los valores de un vector de embedding, con un tipo de dato comprimido de menor precisión. Esto hace que los modelos o los vectores, respectivamente, sean más pequeños, más baratos y más rápidos de ejecutar, **a menudo sin mucho sacrificio en relevancia de retrieval o calidad de respuesta**."*

> [!tip] 💡 Analogía — la del propio curso, y es muy buena
> Una imagen de alta calidad usa **24 bits** para el color de cada píxel. Se ve estupenda, pero ocupa mucho. Puedes comprimirla a 12 bits (la mitad del tamaño) o a 6 bits (un cuarto).
>
> *"Dicho esto, la imagen de 12 bits ya no se ve tan bien, y en la de 6 bits hay muchos artefactos de color visibles. Dependiendo de dónde estés usando la imagen, sin embargo, esta caída de calidad podría valer la pena por el ahorro sustancial de memoria."*
>
> La clave está en **"dependiendo de dónde"**: una miniatura de 6 bits en un listado es perfectamente aceptable; la misma compresión en la foto principal de un producto no lo es. Quantization aplica exactamente ese razonamiento a modelos y vectores.

### 1.1 Quantization de LLMs

**🔧 El punto de partida:** *"Los parámetros en un language model típico usan **16 bits** de memoria cada uno. Con los modelos modernos yendo de aproximadamente mil millones a un billón de parámetros, estos modelos son enormes, requiriendo mucha memoria para almacenarlos y GPUs potentes para ejecutarlos."*

**🔧 Qué hace la compresión:** *"Los modelos cuantizados comprimen esos parámetros de 16 bits a equivalentes de **8 o incluso 4 bits**. Esto reduce significativamente la memoria de GPU requerida para correr el modelo, a costa de un poco de rendimiento y calidad del modelo."*

```
   PESO ORIGINAL         CUANTIZADO 8-bit      CUANTIZADO 4-bit
   ┌──────────────┐      ┌──────┐              ┌───┐
   │ 16 bits      │  →   │8 bits│         →    │4 b│
   └──────────────┘      └──────┘              └───┘
   memoria: 100%         ~50%                  ~25%
   calidad:  100%        caída pequeña         caída mayor
```

**🧭 Cuándo usarlo:** el curso es directo — *"probablemente **deberías** experimentar con usar LLMs y embedding models cuantizados en enteros. La mayoría de los proveedores de LLM y de embedding models pondrán disponibles modelos cuantizados de 8 o 4 bits junto a sus modelos base. Los ahorros de espacio y costo que proporcionan pueden ser significativos, y las reducciones de calidad son bastante pequeñas."*

### 1.2 Quantization de vectores: el algoritmo, paso a paso

**🔧 El problema dimensionado:** *"Un vector bastante típico de 768 dimensiones usará 768 números de punto flotante de 32 bits. Eso son **3 kilobytes de datos por cada vector** en tu knowledge base."*

> [!important] Por qué 3 KB por vector es un problema y no un detalle
> Multiplica: un millón de chunks son **3 GB solo de vectores**. Diez millones, 30 GB. Y como explicó el [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]], el índice HNSW quiere vivir en **RAM** para que la búsqueda sea rápida — y la RAM es la memoria más cara del stack (§2.2). *"Los modelos de dimensionalidad más alta pueden requerir fácilmente varias veces esa cantidad."*

**🔧 El algoritmo de integer quantization**, que el curso desglosa y conviene reproducir porque es más simple de lo que parece:

```
   ① Para CADA DIMENSIÓN, encuentra el valor mínimo y el máximo
      que aparecen en todo tu conjunto de vectores.
      → eso define el rango de esa dimensión

   ② Divide ese rango en 256 secciones del mismo tamaño
      (256 = la cantidad de valores únicos que caben en 8 bits)

   ③ Numera las secciones: 0, 1, 2, 3 … 255

   ④ Cada float original se reemplaza por el número de la sección
      en la que cae

   ⑤ Guarda aparte el mínimo y el ancho de sección de cada dimensión
      → con eso puedes reconstruir una APROXIMACIÓN del float original
```

**🔧 El resultado:** *"esto significa que tus vectores son ahora inmediatamente **un cuarto de su tamaño original**, un ahorro masivo de espacio."*

> [!important] La sorpresa que hace que esto se use en todas partes
> *"A pesar de usar solo un cuarto de los datos y un algoritmo de compresión aparentemente ingenuo, la quantization de enteros de 8 bits rinde notablemente bien. Para benchmarks como **Recall at K**, podrías ver solo una caída de **unos pocos puntos porcentuales** con quantization de 8 bits."*
>
> Y el beneficio es doble: menos datos que almacenar **y** búsqueda más rápida, *"ya que los cálculos requeridos se han simplificado"*. Operar con enteros es más barato que operar con floats.

> [!note] Las cifras publicadas, y tres matices que el curso se salta — *fuente externa verificada*
> La afirmación *"unos pocos puntos porcentuales"* es cierta **para modelos buenos**, pero conviene tener los números reales delante. Shakir, Aarsen & Lee (2024) publican el rendimiento retenido sobre **MTEB Retrieval** tomando float32 como 100 %:
>
> | Modelo | Dim | int8 retenido | binary retenido |
> |---|---|---|---|
> | Cohere-embed-english-v3.0 | 1024 | **100 %** | 94,6 % |
> | mxbai-embed-large-v1 | 1024 | 97 % | 96,45 % |
> | e5-base-v2 | 768 | 94,68 % | **74,77 %** ⚠️ |
> | all-MiniLM-L6-v2 | 384 | 90,79 % | 93,79 % |
>
> **Matiz ① — la métrica no es la que dice el curso.** Estas cifras son **NDCG@10**, no Recall@K. Son métricas distintas; no conviene presentarlas como validación literal de la frase del curso.
>
> **Matiz ② — el rango real de int8 va de 0 % a ~9 % de caída**, no "unos pocos puntos" siempre. Depende fuertemente del modelo.
>
> **Matiz ③ — y este es el importante:** la compresión de 4× y de 32× es un **hecho aritmético**; el costo en calidad **no lo es**. Un embedding entrenado sin conciencia de cuantización puede degradarse mucho más de lo que sugiere el promedio. La única cifra que puedes dar por segura sin medir es la del tamaño.
>
> Velocidad medida en el mismo trabajo (búsqueda exacta en CPU): int8 ≈ **3,66×** más rápido; binary ≈ **24,76×** (hasta 45×).

### 1.3 Binary quantization: el extremo

**🔧 Definición:** *"comprime el tamaño de un vector por un factor de **32**, de 32 bits por dimensión a solo **1 bit**"*.

**🔧 Qué sobrevive a esa compresión:** *"A este nivel de compresión, cada valor en tu vector es o un 1 o un 0, y solo te dice **si ese valor en esa dimensión era un número positivo o negativo**."*

```
   VECTOR ORIGINAL (float32)      →   BINARIO (1 bit)
   [ 0.42, -0.13,  0.87, -0.55 ]  →   [ 1, 0, 1, 0 ]
   
   128 bytes (4 dims × 32 bits)   →   4 bits
```

> [!warning] ⚠️ Aquí la pérdida SÍ se nota — y depende del modelo mucho más de lo que parece
> El curso no lo disimula: *"Como podrás imaginar, a estos niveles extremos de compresión, el rendimiento puede caer **notoriamente** para retrieval basado en embedding models."*
>
> La contrapartida: *"la quantization de 1 bit resulta en retrieval basado en vectores significativamente más pequeño y más rápido."*
>
> **El dato externo que hay que tener presente:** en la tabla de §1.2, binary retiene un **96,45 %** con `mxbai-embed-large-v1` pero solo un **74,77 %** con `e5-base-v2` — veinticinco puntos de diferencia **con la misma técnica**. La compresión de 32× está garantizada; la calidad resultante no. **Binary quantization no es una decisión que se tome leyendo un blog: se toma midiendo sobre tu propio corpus y tu propio modelo.**

**🔧 El patrón que rescata la calidad — retrieval en dos fases:**

*"También puede emparejarse con otras técnicas, por ejemplo, haciendo retrieval rápido basado en un embedding model cuantizado a 1 bit, y luego **rescoring usando el vector original completo de 32 bits**."*

```
   FASE 1 — barrido barato          FASE 2 — precisión
   ┌──────────────────────┐         ┌──────────────────────┐
   │ vectores binarios    │         │ vectores float32     │
   │ 1 bit/dim, 32× más   │  ───►   │ originales           │
   │ pequeños             │  top-N  │ reordenan esos N     │
   │ búsqueda muy rápida  │         │ (over-fetch + rerank)│
   └──────────────────────┘         └──────────────────────┘
```

> [!note] Es el mismo patrón que ya conoces, con otro disfraz
> Barrer barato → refinar caro es exactamente la estructura del **re-ranking con cross-encoder** del [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]]: un modelo rápido y aproximado selecciona candidatos, uno lento y preciso los ordena. Y requiere el mismo cuidado: **over-fetch suficiente en la fase 1**, o la fase 2 solo reordena basura.

> [!tip] Cuánto recupera realmente el rescoring — *cifras externas verificadas*
> El curso menciona la técnica sin cuantificarla. Los números publicados (Shakir, Aarsen & Lee, 2024) muestran que **es la diferencia entre usable y no usable**:
> - **Binary con rescoring:** de 92,53 % a **96,45 %** de rendimiento retenido.
> - **int8 con `rescore_multiplier` de 4–5:** *"una notable retención de rendimiento del **99 %**"*.
>
> Ese `rescore_multiplier` es el **over-fetch** del Tomo 07 con otro nombre: traer 4–5× más candidatos en la fase barata para que la fase cara tenga con qué trabajar. Sin over-fetch, el rescoring no tiene nada que rescatar.
>
> **Contrapunto honesto:** Weaviate documenta recalls en torno a **0,74–0,76** con binary quantization sobre 100 000 vectores (DBPedia, ada-002 y Cohere v2), bastante por debajo del ~96 % anterior. La diferencia está en el rescoring y en el modelo — lo que refuerza el punto de §1.2: **la cifra que te sirve es la que midas tú.**

### 1.4 Matryoshka embeddings

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Son las **muñecas rusas** que les dan nombre, pero la intuición más útil es otra: es un vector escrito **en orden de importancia**, como un titular seguido de la noticia. Si solo lees el titular te enteras de lo esencial; si sigues leyendo afinas los detalles. En un embedding normal, en cambio, la información está repartida uniformemente — leer solo la mitad es como leer una noticia a la que le arrancaron media hoja al azar.

**🔧 Definición técnica:** *"Estos vectores están diseñados de modo que puedes elegir usar **solo un subconjunto de las dimensiones** de un vector al hacer cosas como comparar similaridad. Por ejemplo, si el vector completo tiene 1000 dimensiones, podrías elegir usar solo las primeras 500 o las primeras 100."*

**🔧 La propiedad que lo hace funcionar:** *"sus dimensiones están **ordenadas por cuán densas en información** son. Aquí, información significa cuánta varianza estadística esperarías ver en esa dimensión al embeber grandes cantidades de texto."*

> [!note] La fuente, y una precisión sobre esa explicación — *fuente externa verificada*
> El paper fundacional es **Kusupati et al. (2022), *Matryoshka Representation Learning*** (NeurIPS 2022). Describe el mecanismo como aprender representaciones **coarse-to-fine** anidadas, entrenadas con una pérdida multi-escala sobre prefijos del vector, de modo que *"un solo embedding se adapta a las restricciones computacionales de las tareas posteriores"*.
>
> **La precisión:** la formulación del curso — "dimensiones ordenadas por densidad de información / varianza" — es una **buena intuición divulgativa, pero no es cómo lo enuncia el paper**. El paper no habla de ordenar dimensiones por varianza, sino de entrenar prefijos anidados para que cada uno sea autosuficiente. El efecto observable es parecido; el mecanismo declarado, no. Conviene no atribuir esa frase al paper.
>
> Resultados del abstract que dimensionan la ganancia: hasta **14× menos tamaño de embedding** con la misma exactitud en ImageNet-1K, y hasta **14× de aceleración real** en recuperación a gran escala.

| | Embedding model típico | Matryoshka embedding model |
|---|---|---|
| **Distribución de varianza** | *"cada dimensión tendría aproximadamente la misma cantidad de varianza o información"* | *"las dimensiones tempranas tendrán más varianza, lo que significa más contenido informativo"* |
| **Truncar el vector** | Pierdes información arbitraria | *"las dimensiones posteriores… proporcionan relativamente menos información, así que pagas menos penalización por excluirlas"* |
| **Cuándo conviene** | Caso general | Cuando quieres poder **cambiar de fidelidad sobre la marcha** |

**🔧 Los dos modos de uso** que da el curso:

1. **Truncar y ya:** *"usar siempre solo las primeras 100 dimensiones, ahorrando espacio y llevando a cálculos más rápidos, mientras preservas la máxima cantidad posible de información."*
2. **Dos fases, como en binary:** *"hacer siempre el retrieval inicial usando las primeras 100 dimensiones, luego traer las 900 restantes — ahora usando las 1000 dimensiones completas — desde memoria más lenta y barata, para esencialmente rescorear el conjunto inicial de documentos que recuperaste."*

**🧭 Cuándo usarlo:** *"las propiedades flexibles de los modelos Matryoshka los hacen más adecuados para **entornos dinámicos** donde podrías querer cambiar rápidamente de representaciones vectoriales de baja a alta fidelidad."*

> [!note] Fíjate en la sinergia con la sección 2.2
> El modo 2 encaja exactamente con la jerarquía de memoria: **las primeras 100 dimensiones en RAM** (caras, rápidas, siempre presentes) y **las 900 restantes en disco** (baratas, más lentas, solo se leen para rescorear los pocos candidatos que sobrevivieron). Es la misma idea de "no pagues memoria cara por lo que rara vez usas", aplicada dentro de un solo vector.

### 1.5 Tabla comparativa

| Técnica | Compresión | Impacto en calidad | Cuándo conviene |
|---|---|---|---|
| **LLM int8 / int4** | ~2× / ~4× | *"caídas menores"* en benchmarks comunes | Casi siempre vale probarlo; los proveedores ya publican las variantes |
| **Vector int8** | **4×** | *"unos pocos puntos porcentuales"* en Recall@K | El punto de partida por defecto para vectores |
| **Vector binario (1-bit)** | **32×** | *"puede caer notoriamente"* | Escala muy grande, **emparejado con rescoring** en float32 |
| **Matryoshka** | Variable (eliges N dims) | Menor que truncar un vector normal, por diseño | Entornos dinámicos; retrieval en dos fases con jerarquía de memoria |

> [!danger] 🚨 El error que la analogía de la imagen advierte y casi nadie escucha
> La compresión **siempre** cuesta calidad. El curso repite que la caída "suele ser pequeña" — pero *pequeña* no es *cero*, y **solo lo sabes si lo mides**. Comprimir sin el instrumental del [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]] es cambiar calidad por dinero **a ciegas y sin recibo**.
>
> El orden correcto: instrumenta → establece la línea base de Recall@K y de calidad de respuesta → cuantiza → **vuelve a medir** → decide si el ahorro compensa. Ese último paso es el que se salta el assignment de este mismo módulo (§4.4).

---

## 2. 💰 Cost

Audiencia: 🔧 🧭 👔

**🔧 De dónde sale la factura:** *"Los dos costos más grandes en una aplicación RAG típica serán tu **vector database** y tus **large language models**."*

### 2.1 Reducir el costo de los LLMs

| Palanca | Qué hacer | Nota del curso |
|---|---|---|
| **Modelos más pequeños** | Probar modelos con menos parámetros o cuantizados, tanto para el LLM principal como para los routers | *"a menudo te sorprenderá gratamente lo bien que rinden los modelos pequeños, especialmente si tu LLM va a realizar un número limitado de tareas"* |
| **Fine-tuning de un modelo pequeño** | Especializarlo en la tarea concreta | *"puede llevar a buenos resultados a bajo costo"* |
| **Menos tokens de entrada** | Recuperar menos documentos — **bajar `top_k`** | *"los prompts de RAG pueden crecer rápidamente de tamaño, especialmente si recuperas muchos chunks largos"* |
| **Menos tokens de salida** | System prompts que pidan concisión, o un límite duro de tokens | *"muchos LLMs son verbosos, y recuerda: **pagas por cada token que generan**"* |
| **Hardware dedicado** | Pasar de endpoint por token a GPUs alquiladas por hora | Ver abajo |

**🔧 Sobre el hardware dedicado**, que es la decisión de mayor impacto a escala:

*"Los proveedores cloud de LLM como TogetherAI, AWS y Google ofrecen endpoints de inferencia convenientes, y a menudo tiene sentido usarlos cuando estás construyendo un prototipo. Sin embargo, si tu proyecto ha escalado a miles o millones de requests, podrías querer ahorrar dinero corriendo modelos en hardware dedicado alquilado a esas mismas compañías."*

```
   PROTOTIPO / VOLUMEN BAJO          PRODUCCIÓN A ESCALA
   ┌───────────────────────┐         ┌───────────────────────┐
   │ endpoint compartido   │         │ GPU dedicada          │
   │ pagas POR TOKEN       │   ──►   │ pagas POR HORA        │
   │ sin compromiso        │         │ + mejor fiabilidad:   │
   │ latencia variable     │         │   el hardware sirve   │
   │ (compites por él)     │         │   solo a tu tráfico   │
   └───────────────────────┘         └───────────────────────┘
        el cruce ocurre cuando el volumen justifica la hora
```

*"A escala, sin embargo, los ahorros de costo por pagar por hora en vez de por token pueden ser muy significativos."* Y el beneficio lateral: *"el beneficio adicional de los endpoints dedicados es **mejor fiabilidad**, ya que ese hardware está sirviendo solo a tu tráfico de usuarios y nada más."*

> [!note] El quiz lo confirma como respuesta correcta
> Entre los enfoques válidos para reducir costos de LLM, el quiz marca *"implementar un **router** que solo hace retrieval cuando es necesario"* y *"usar un **endpoint dedicado** una vez que el volumen lo justifica"*. Fíjate en el primero: **no toda consulta necesita retrieval**. Un saludo, una pregunta de seguimiento sobre lo ya dicho, o una consulta fuera de dominio no requieren tocar la vector database — y sin embargo muchos sistemas la consultan siempre.

### 2.2 Reducir el costo de la vector database

**🔧 La clave está en la jerarquía de memoria:** *"la mayoría de las databases te ofrecen múltiples tipos de memoria"*.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  RAM                    la más rápida, la MÁS CARA           │
   │  ↓  "varias veces más cara por GB que el disco"              │
   │  DISCO                  intermedia                           │
   │  ↓  "varias veces más cara que el cloud storage"             │
   │  CLOUD OBJECT STORAGE   la más lenta, la MÁS BARATA          │
   └──────────────────────────────────────────────────────────────┘
```

**🔧 La regla de asignación**, que el curso enuncia con precisión:

*"Si quieres ahorrar dinero, entonces quieres asegurarte de que **solo estás pagando por mantener información en almacenamiento rápido y caro si eso va a beneficiar de verdad el rendimiento de tu sistema**."*

| Qué | Dónde | Por qué |
|---|---|---|
| **El índice HNSW** | **RAM** | *"debería mantenerse en RAM para asegurar que la búsqueda vectorial corra lo más rápido posible"* — es lo que se recorre en cada consulta |
| **El contenido de los documentos** | Disco | *"probablemente no necesitan estar almacenados en RAM"* — solo se leen para los pocos chunks que ganaron |
| **Documentos de acceso frecuente** | Disco | Compromiso razonable |
| **Objetos de acceso raro** | Cloud object storage | Pagar lo mínimo por lo que casi nunca se toca |

> [!important] 🎯 La distinción que hace que esto funcione
> **El índice y el contenido no son la misma cosa.** El índice HNSW ([[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]]) se recorre entero en cada búsqueda: si está en disco, cada consulta paga esa lentitud. El **texto** de los chunks, en cambio, solo se lee para los `top_k` que sobrevivieron — cinco lecturas de disco por consulta, no un millón.
>
> Poner todo en RAM "por si acaso" es el error caro por defecto. Poner el índice en disco es el error caro en el otro sentido.

### 2.3 Multi-tenancy como palanca de costo

**🔧 Definición:** *"dividir todos los documentos en tu vector database **por el usuario u organización a la que pertenecen**."*

**🔧 El ejemplo del curso:** *"podrías tener un millón de documentos en tu vector database, propiedad de mil usuarios distintos. Cada usuario debería poder acceder solo a sus propios documentos, así que **cada usuario tendrá de hecho su propio índice HNSW** para los documentos asociados a él."*

**🔧 Por qué eso ahorra dinero:** *"Este sistema hace fácil cargar rápidamente los datos de un tenant en memoria rápida y cara **solo cuando es necesario**."*

Los dos ejemplos que da:
- *"podrías esperar hasta que un cliente realmente inicie sesión en tu sitio web para cargar sus vectores en RAM"*
- *"podrías por defecto mantener los datos de tus tenants europeos en almacenamiento más lento durante la noche en Europa"*

> [!tip] La misma decisión resuelve dos problemas distintos
> Multi-tenancy aparece **dos veces** en el Módulo 5, por razones que no tienen nada que ver entre sí:
> - Aquí, como palanca de **costo**: mover datos dentro y fuera de memoria cara con granularidad de cliente.
> - En el [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §6.2]], como mecanismo de **seguridad**: la forma robusta de garantizar que un usuario no acceda a documentos ajenos, frente al metadata filtering que es *"demasiado propenso a fallos"*.
>
> Que una sola decisión de arquitectura resuelva a la vez el costo y el control de acceso es poco habitual. **Si estás dudando si vale la pena, esta es la razón: se paga dos veces.**

### 2.4 El principio de fondo

*"La idea central de todas estas optimizaciones es que, como ingeniero, necesitas **entender el origen de tus costos y asegurarte de que están justificados por el rendimiento**. Para LLMs, modelos más pequeños y prompts más cortos suele ser el camino. Para vector databases, almacenar cantidades menores de datos en almacenamiento caro y moverse entre RAM, disco y object storage son las formas principales de ahorrar costos."*

> [!example] 📊 Caso de negocio — Turismo
> **Problema.** Una plataforma de reservas despliega un asistente que responde sobre destinos, políticas de cancelación, requisitos de entrada y recomendaciones de itinerario. Funciona bien y el uso crece. A los cuatro meses, finanzas escala el tema: el costo de inferencia se multiplicó por seis mientras las reservas subieron un 40 %. Nadie sabe explicar la diferencia, y la propuesta sobre la mesa es "usar un modelo más barato" sin más análisis.
>
> **Técnica aplicada.** Con los traces del [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]] se atribuye el costo por componente, y aparecen tres cosas que el promedio escondía. Primero, **una fracción alta de las consultas no necesitaba retrieval en absoluto** — saludos, agradecimientos y preguntas de seguimiento sobre lo ya respondido — y aun así todas pagaban la recuperación y el contexto asociado. Segundo, el `top_k` estaba fijo en un valor generoso heredado del prototipo, de modo que cada consulta arrastraba chunks largos que el modelo no usaba. Tercero, el pico estacional concentraba el tráfico en pocas semanas, pero la vector database mantenía **todos** los destinos en RAM todo el año, incluidos los de temporada opuesta.
>
> **Resultado.** Tres cambios, ninguno de ellos "usar un modelo peor": un router que decide si hace falta recuperar; `top_k` ajustado con la curva de calidad medida delante; y particionado por destino con carga a memoria rápida según estacionalidad. El costo por consulta bajó de forma sustancial **con la calidad medida antes y después** — que es lo que permitió defender el cambio sin discutir de intuiciones.
>
> La lección transferible: el ahorro no vino de degradar el modelo, sino de **dejar de pagar por trabajo que no se estaba usando.** Y solo se pudo ver porque el costo estaba atribuido por componente.

---

## 3. ⏱️ Latency

Audiencia: 🔧 🧭 👔

**🔧 El planteamiento:** *"Simplemente añadir un retriever a tu sistema añade latency. Y a medida que añades más componentes para aumentar la calidad de la respuesta, como re-ranking o construir sistemas agentic más complejos, la latency puede aumentar."*

### 3.1 Lo primero: cuánta latencia tolera TU caso

**🔧 El curso rechaza la pregunta genérica:** *"Cuán importante es la latency para tu sistema depende fuertemente del contexto en el que se usará."*

| Contexto | Optimizas para | Razonamiento del curso |
|---|---|---|
| **E-commerce** | **Latencia**, aun a costa de calidad | *"los clientes que navegan un sitio de e-commerce tienen notoriamente poca paciencia con tiempos de respuesta lentos, así que probablemente optimizarías tu servicio de recomendación de ítems para tener latency muy baja, posiblemente a costa de no recomendar el ítem perfecto del catálogo"* |
| **Diagnóstico médico** | **Calidad**, aun a costa de latencia | *"un sistema RAG diseñado para ayudar a médicos a diagnosticar enfermedades raras, por otro lado, probablemente se optimizaría para calidad de respuesta, incluso si eso significa que las respuestas tardan mucho más en producirse"* |

> [!warning] ⚠️ No hay una latencia "buena"
> Es la advertencia operativa de la sección: **el número objetivo lo fija el negocio, no la ingeniería.** Antes de optimizar, *"deberías entender cuánta latency tu sistema puede tolerar"*. Optimizar a 200 ms un sistema que toleraba 3 segundos es gastar calidad a cambio de nada.

### 3.2 La regla que ordena toda la optimización

**🔧 La heurística central del curso, y la que más tiempo ahorra:**

*"Al atacar la latency, una guía fácil de recordar es que **casi toda ella es resultado de ejecutar un transformer**. Como consecuencia, el mayor culpable serán tus llamadas al large language model. Aunque el retrieval sí añade un poco de latency —en particular algunas técnicas de re-ranking basadas en transformers—, las databases modernas, y en particular las vector databases, son muy rápidas y escalan bien."*

```
   ¿DÓNDE SE VA EL TIEMPO?

   ████████████████████████████████████  LLM principal
   ████████                              re-ranker (cross-encoder)
   ████                                  router LLM / query rewriter
   █                                     búsqueda vectorial
   
   → si vas a optimizar, EMPIEZA POR ARRIBA
```

> [!important] La consecuencia contraintuitiva
> Mucha gente empieza optimizando la base de datos, porque "la base de datos" suena a lo lento en cualquier otro sistema. **En RAG es al revés:** la vector database es de lo más rápido del pipeline, y el LLM es de lejos lo más lento. Es la misma lógica de diagnóstico del [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]] — mide antes de tocar — aplicada al tiempo en vez de a la calidad.

### 3.3 Las palancas, en el orden en que conviene aplicarlas

**🔧 ① El LLM principal.** *"Si quieres recortar latency, el mejor lugar para empezar es tu core language model."*
- **Modelo más pequeño o cuantizado:** *"los LLMs más pequeños, o los modelos cuantizados, **siempre correrán más rápido** en el mismo hardware, asumiendo que la misma memoria está disponible."* ← aquí se cobra la sección 1.
- **Router LLM:** *"usar un router LLM más pequeño, cuyo trabajo es mirar el prompt y decidir si un LLM más pequeño o más grande es la herramienta correcta para el trabajo. Si una query requiere razonamiento complejo, puede enrutarse a un modelo más grande y potente. Mientras tanto, las queries simples pueden enrutarse a modelos más pequeños y rápidos."*

> [!tip] Lo que hace elegante al router
> *"Esto ayuda a mantener baja la latency para prompts más simples **mientras permite que la latency aumente solo para los prompts más complejos que lo requieren**."*
>
> No es un compromiso a la baja: es **latencia proporcional a la dificultad**. El usuario que pregunta el horario de atención no paga el costo de la consulta compleja de otro. Conecta directo con los patrones agentic del [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 §5]].

**🔧 ② Caching semántico.** Para sistemas que reciben prompts muy parecidos:

*"mantienes una caché de prompts enviados frecuentemente y sus respuestas. Cuando se recibe un prompt nuevo, calculas rápidamente scores de similaridad entre el nuevo prompt y los de la caché. Si encuentras una coincidencia suficientemente cercana, puedes devolver inmediatamente la respuesta cacheada, **saltándote por completo el proceso de generación, relativamente lento**."*

Y la variante que preserva personalización: *"si quieres seguir usando caching pero con respuestas algo personalizadas, puedes recuperar la respuesta cacheada, pero luego pasar la respuesta cacheada y el prompt del usuario a un LLM más pequeño y rápido para hacer pequeños ajustes que la hagan más relevante al prompt."*

```
   prompt nuevo
        │
        ├─► similarity contra la caché ──► ¿match cercano?
        │                                      │
        │                              SÍ ─────┴───── NO
        │                               │              │
        │                    ┌──────────┴────┐         ▼
        │                    │               │    pipeline
        │              devolver tal      LLM pequeño   completo
        │              cual (instantáneo) personaliza
```

> [!warning] ⚠️ El umbral de similaridad es el parámetro peligroso
> El curso dice *"con un tuning cuidadoso, este enfoque puede mejorar mucho la latency del sistema para muchos prompts"*. El **"tuning cuidadoso"** es lo que hay que subrayar: un umbral demasiado laxo devuelve la respuesta de otra pregunta. *"¿Puedo cancelar sin costo?"* y *"¿puedo cancelar?"* son semánticamente cercanas y tienen respuestas distintas. Un falso positivo de caché no se ve como error — se ve como una respuesta segura y equivocada.

**🔧 ③ Los demás componentes transformer.** *"Una vez que has optimizado la latency de tu core LLM, el siguiente paso es abordar otros componentes basados en transformers de tu pipeline. Esto podría ser un query rewriter, un re-ranker, o un router LLM."*

Y el consejo, que es el más accionable del tomo:

> [!important] 🎯 Medir latencia **y** beneficio incremental, por componente
> *"Mi consejo aquí es **medir tanto la latency que cada componente añade a tu sistema como la calidad de respuesta incremental que proporcionan**. Podrías darte cuenta de que no estás obteniendo mucho beneficio de tu query rewriter, por ejemplo, y optar por eliminar ese componente."*
>
> Es la operacionalización de la observability del [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]]: cada componente del pipeline tiene un **precio en milisegundos** y un **beneficio en calidad**. Los componentes se acumulan por inercia — se añaden porque mejoran algo y nunca se revisan. Este es el momento de revisarlos.

**🔧 ④ El retriever, al final.** *"aunque la generación es típicamente la mayor fuente de latency, todavía hay formas de eliminar latency causada por tu retriever"*:
- **Binary quantization** de los embeddings: *"esto simplifica los cálculos de distancia vectorial subyacentes y ayuda a acelerar el retrieval"* ← otra vez la sección 1.
- **Sharding:** *"dividir databases más grandes en instancias separadas, especialmente una vez que se vuelven bastante grandes, también puede ayudar a reducir la latency de búsqueda"*.

> [!note] Pregunta invertida del quiz
> El quiz pregunta qué técnica **MENOS** ayuda a reducir la latencia, y la respuesta es *"eliminar documentos de acceso raro de tu vector database"*. Tiene sentido con el [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]] delante: la búsqueda en HNSW escala de forma **logarítmica** con el número de vectores, así que quitar documentos poco consultados apenas mueve la aguja en tiempo — aunque sí ahorre memoria (§2.2). **Es una palanca de costo, no de latencia.** Confundirlas es un error frecuente.

---

## 4. 💻 El assignment: optimizar costo sin perder exactitud

Audiencia: 🔧

El assignment graded del módulo retoma el ChatBot de "Fashion Forward Hub" del [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Módulo 4]] y pide **reducir su costo y su latencia sin perder exactitud**, con todo instrumentado en Phoenix.

### 4.1 El mecanismo: un flag `simplified` y una comparación A/B

**🔧 El diseño es limpio y vale la pena copiarlo:** cada función lleva un parámetro `simplified`. Con `False` corre el pipeline caro del M4; con `True`, la versión optimizada. Todo el notebook es una comparación lado a lado.

| # | Función | Qué se optimiza | Presupuesto |
|---|---|---|---|
| 1 | `check_if_faq_or_product` | Prompt del router FAQ/Product, más corto | ≤180 tokens, misma accuracy (5/5) |
| 2 | `query_on_faq` | Sustituir "meter todo el FAQ en el prompt" por **búsqueda semántica top-5** | <500 tokens |
| 3 | `decide_task_nature` | Prompt del router creative/technical, más corto | <150 tokens, ≥80 % accuracy |
| 4 | `get_relevant_products_from_query` | **Saltarse por completo la generación de metadata con LLM** | 20 product IDs, 0 tokens de LLM |

### 4.2 Los resultados reales

**🔧 Las cifras medidas del propio notebook**, que son el mejor argumento del módulo:

| Etapa | Antes | Después | Δ |
|---|---|---|---|
| Router FAQ/Product | 201–207 tokens | 123–129 | **−38 %** |
| FAQ end-to-end | 1.134 | 420 | **−63 %** |
| Metadata de productos | 1.631 | **0** | **−100 %** |
| Consulta de producto completa | 3.231 | 1.876 | **−42 %** |
| `answer_query` (t-shirts) | 3.301 | 1.737 | **−47 %** |

> [!important] El ejercicio 4 es el más instructivo de los cuatro
> El pipeline original gastaba **~1.500 tokens por consulta solo en generar los filtros de metadata** con un LLM, antes de buscar nada. El ejercicio pide eliminarlo y hacer búsqueda semántica directa: *"este enfoque es más rápido, usa menos tokens, y sigue siendo efectivo para la mayoría de las queries."*
>
> La lección general: **el paso más caro de un pipeline suele ser uno que se añadió para mejorar la precisión y nunca se volvió a cuestionar.** Es exactamente el "medir latencia y beneficio incremental por componente" de §3.3 ③, aplicado al costo.

**🔧 El patrón de instrumentación para atribuir costo**, que sí es reutilizable tal cual:

```python
with tracer.start_as_current_span("router_call", openinference_span_kind='llm') as router_span:
    router_span.set_input(kwargs)
    try:
        response = generate_with_single_input(**kwargs)
    except Exception as error:
        router_span.record_exception(error)
        router_span.set_status(Status(StatusCode.ERROR))
        raise                       # ← el assignment NO re-lanza aquí. Ver §4.3 ⑥
    else:
        # Convenciones OpenInference que permiten a Phoenix calcular COSTO
        router_span.set_attribute("llm.token_count.prompt",     response['usage']['prompt_tokens'])
        router_span.set_attribute("llm.token_count.completion", response['usage']['completion_tokens'])
        router_span.set_attribute("llm.token_count.total",      response['usage']['total_tokens'])
        router_span.set_attribute("llm.model_name", response['model'])
        router_span.set_attribute("llm.provider", 'together.ai')
        router_span.set_output(response)
        router_span.set_status(Status(StatusCode.OK))
```

> [!note] Por qué aquí la instrumentación es manual y en el lab era automática
> El notebook lo explica: a diferencia del Ungraded Lab 1, aquí **no** se usa `auto_instrument=True` porque hay llamadas al LLM que **no** deben trazarse — los ejemplos didácticos y las llamadas dentro de los unittests. Es un criterio válido y transferible: `auto_instrument` es cómodo, pero traza *todo*, incluido el ruido que contamina tus dashboards de costo.

### 4.3 🐛 Los bugs del assignment (verificados ejecutando)

> [!danger] 🚨 Hallazgos contrastados contra el código
> Este assignment es, con diferencia, el que más defectos acumula del curso. Los cuatro primeros están **verificados ejecutando el código**.

**① `get_params_for_task`: la rama `technical` es inalcanzable.** El bug más grave, y está en **código dado, no del alumno**:

```python
def get_params_for_task(task):
    PARAMETERS_DICT = {"creative":  {'top_p': 0.9, 'temperature': 1},
                       "technical": {'top_p': 0.7, 'temperature': 0.3}}
    if task == 'technical':
        param_dict = PARAMETERS_DICT['technical']   # se asigna…
    if task == 'creative':                          # …segundo if, independiente
        param_dict = PARAMETERS_DICT['creative']
    else:                                           # ← este else pertenece al SEGUNDO if
        param_dict = {'top_p': 0.5, 'temperature': 1}   # …y sobrescribe SIEMPRE
    return param_dict
```

**Salida real al ejecutarlo:**

```
task='technical'  -> {'top_p': 0.5, 'temperature': 1}
task='creative'   -> {'top_p': 0.9, 'temperature': 1}
task='otra'       -> {'top_p': 0.5, 'temperature': 1}
```

> [!warning] ⚠️ La consecuencia en cascada
> La docstring promete que *"las tareas technical se manejan con más foco y precisión"* con `temperature=0.3`. **Recibe `temperature=1`** — el máximo. Es decir, las consultas factuales ("¿cuál es la camiseta más barata?") se responden con la temperatura más creativa posible, exactamente lo contrario de lo diseñado.
>
> Y el efecto dominó: **el Ejercicio 3 completo es decorativo.** El alumno optimiza un router para distinguir creative de technical, y la mitad technical del árbol nunca usa sus parámetros. Peor aún: existe un `test_get_params_for_task` en `unittests.py` que (a) **el notebook nunca invoca** y (b) solo comprueba `isinstance(output, dict)`, así que jamás lo detectaría.
>
> La corrección es un `elif`.

**② El filtro de precio se descarta en el caso por defecto.** El prompt **ordena** al LLM: *"si no hay precio, pon min = 0 y max = inf"*. Y luego:

```python
if min_price <= 0 or max_price == 'inf':
    continue          # ← descarta AMBOS filtros, incluido el max real
```

**Verificado:** con la salida real del notebook para *"no quiero gastar más de 300 dólares en cada pieza"*, el LLM produce correctamente `{"min": 0, "max": 300}` → y **no se genera ningún filtro de precio**. La restricción de presupuesto del usuario se pierde entera, en silencio. Lo correcto es tratar los dos límites por separado.

**③ El prompt afirma lo contrario del orden real.** En el ejercicio 2:

```python
results.reverse()   # comentario: "para que el más relevante quede abajo"
...
f"...están ordenadas en relevancia DECRECIENTE, así que la primera es la más
   relevante y la última la menos relevante."
```

Tras el `.reverse()` el orden es **creciente**. El prompt le dice al modelo exactamente lo opuesto a lo que recibe — y anula la técnica de recency que el `.reverse()` pretendía explotar.

**④ El `set` de Python que se presenta como JSON.** El prompt de metadata interpola `values`, que es un dict de `set`:

```
Possible values for each feature is in the following json:
{'season': {'Spring', 'Fall', 'All seasons', 'Summer', 'Winter'}}
```

Eso **no es JSON válido** (JSON no tiene sets), y el prompt afirma que lo es. Peor para un assignment de medición: **el orden de iteración de un `set` de strings depende del hash de Python**, que varía entre procesos. El prompt de ~1.500 tokens **no es idéntico entre ejecuciones**, así que el baseline de tokens no es reproducible **ni con `temperature=0`**. Explica por qué el texto dice "alrededor de 1.500" mientras las celdas imprimen 1.491 y 1.631.

**⑤ El ChatBot destruye la contabilidad de tokens** — ya documentado en el [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §5.4]]: `total_tokens` se **sobrescribe** en vez de acumularse, ocultando ~50 % del costo real justo en la interfaz interactiva.

**⑥ `try/except` que registra el error y sigue.** Patrón repetido en tres funciones: si la llamada falla, se marca el span como ERROR pero **no se re-lanza**, y la siguiente línea usa `response` → `UnboundLocalError`, un error que no dice nada del fallo real. (En el bloque de §4.2 lo corregí añadiendo el `raise`.)

**⑦ `importance_order` con tres defectos simultáneos:** `'masterCategory'` aparece **duplicado** (una iteración de refiltrado que no relaja nada y gasta una query a Weaviate); `'price'` **no está en la lista**, así que los filtros de precio mueren en la primera iteración; y en la última iteración el slice queda vacío → `Filter.all_of([])`, que la librería no acepta → crash, que además se traga un `except Exception:` desnudo.

### 4.4 🎯 La crítica de fondo: mide tokens, nunca mide calidad

> [!danger] 🚨 La contradicción central del módulo
> El Módulo 5 dedica **tres lecciones completas** a evaluar calidad: la grilla scope × evaluator type, LLM-as-a-judge, recall/precision con dataset anotado, feedback humano. El quiz lo refuerza en 4 de 10 preguntas.
>
> **El assignment no implementa ni un solo eval de calidad.** Sus cuatro métricas graded son: número de tokens, número de veces que aparece la palabra "Question" en un prompt, coincidencia exacta de etiqueta, y coincidencia exacta de 20 IDs. **Nunca se compara la calidad de la respuesta antes y después de la simplificación.**

**Y hay un ejemplo donde la calidad empeoró de forma visible.** Para la consulta *"crea un look para un hombre que asista a una fiesta de boda de noche"*:

| | Resultado |
|---|---|
| `simplified=False` | **Camisa + corbata** (John Players Men Teal Shirt + Provogue Men Pink Tie) — un "look" plausible |
| `simplified=True` | **Zapatos + pack de corbata**, sin ninguna prenda superior. El propio modelo se disculpa en la respuesta: *"aunque el producto está categorizado en temporada Fall"* y *"el zapato negro es el único ítem específicamente etiquetado para uso nocturno"* |

El notebook imprime ambas respuestas, **no comenta la degradación**, y concluye: *"¡Y el total de tokens usados en una query fue mucho más bajo que antes!"*

> [!warning] ⚠️ Cómo leer esto sin descartar el assignment
> La técnica que enseña **es correcta y valiosa**: eliminar una llamada de 1.500 tokens que aportaba poco es exactamente el tipo de optimización que hay que buscar. Lo que falta es el **segundo medio** del trade-off.
>
> El assignment demuestra la mitad barata de la ecuación (el ahorro se mide con precisión) y omite la mitad cara (la calidad no se mide en absoluto). Y al omitirla, publica como éxito un caso donde el sistema devolvió un atuendo peor.
>
> **La corrección práctica**, si adaptas este patrón: cada vez que introduzcas un flag `simplified`, añade junto a la comparación de tokens una comparación de calidad — aunque sea un LLM-as-a-judge con rúbrica binaria (*"¿esta respuesta resuelve la consulta igual de bien que la otra? sí/no"*) sobre 30 consultas reales de tu propio tráfico ([[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §5]]). Sin eso, no estás optimizando: **estás recortando.**

**Otras tres contradicciones con la clase**, que conviene tener presentes:

| La clase enseña | El assignment hace |
|---|---|
| *"Experimenta recuperando menos documentos, es decir, reduciendo top-k"* | El grader compara contra un set **exacto de 20 IDs**: cualquier `limit ≠ 20` falla. **El único parámetro que la clase dice variar es el que el grader clava.** |
| Router que enruta a **modelos más pequeños o más grandes** según complejidad | Los dos routers eligen prompt y sampling, **nunca modelo**. La línea que lo haría está en el código pero **comentada**: `#params_dict['model'] = 'Qwen/Qwen3.5-9B'`. Toda respuesta final usa el modelo grande |
| Router que *"solo hace retrieval cuando es necesario"* | Ambas etiquetas (FAQ y Product) disparan retrieval siempre. No hay camino "responder sin recuperar" |

---

## 5. 🖼️ Multimodal RAG

Audiencia: 🔧 🧭 👔

**🔧 El problema:** *"A lo largo de este curso has visto sistemas RAG construidos sobre datos de texto, pero hoy en día la información se almacena en una enorme variedad de formatos. Presentaciones, PDFs o imágenes también incluyen información valiosa que idealmente querrías incluir en tu knowledge base."*

Es la tercera categoría de desafío de producción que listaba el [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §1]]: *"muchos datos ni siquiera están en formato de texto"*.

**🔧 Qué es un sistema multimodal típico:**

```
   ┌─────────────────────────────────────────────────────────┐
   │  ENTRADA:      texto  +  imágenes                       │
   │  KNOWLEDGE     archivos de texto  +  archivos de imagen │
   │  BASE:                                                   │
   │  SALIDA:       texto                                     │
   └─────────────────────────────────────────────────────────┘
```

Y hay que actualizar **dos componentes**: el retriever y el LLM.

### 5.1 El embedding model multimodal

> [!tip] 💡 Analogía
> Imagina un archivador donde las fichas no están ordenadas por idioma sino **por lo que significan**. La ficha de la palabra "perro", la de la palabra "puppy" y **la fotografía de un perro** acaban las tres en el mismo cajón — no porque se parezcan por fuera, sino porque hablan de lo mismo. Un embedding multimodal es ese archivador: le da igual si le entregas una palabra o una imagen, las coloca según lo que significan.

**🔧 Definición técnica:** *"Un embedding model multimodal es uno que puede embeber múltiples formatos de datos **en el mismo vector space**."*

**🔧 El comportamiento esperado**, según el curso: si embebes las palabras *dog* y *puppy*, sus vectores quedan cerca — como con un modelo de solo texto. *"Si le das a ese mismo modelo una imagen de un perro, sin embargo, el vector de esa imagen también terminaría en una parte cercana del vector space."* Y una imagen de un árbol y la palabra *tree* también quedarían juntas, *"pero en una parte distinta del vector space"*.

*"En otras palabras, los embedding models multimodales funcionan igual que los de texto, colocando ítems con significados similares más cerca. Gracias a su diseño, sin embargo, pueden realizar esa misma función con múltiples tipos o modalidades de datos."*

**🔧 Y entonces el retrieval no cambia en nada:**

```
   knowledge base (texto + imágenes) ──► mismo embedding multimodal ──► un solo espacio
   prompt (texto O imagen)           ──► mismo embedding multimodal ──► vector de query
                                                    │
                                     búsqueda vectorial NORMAL
                                                    │
                            texto e imágenes más cercanos al prompt
                                                    │
                                       augmented prompt como siempre
```

> [!important] Todo lo que aprendiste sigue valiendo
> Este es el mensaje tranquilizador de la sección: chunking, HNSW, hybrid search, re-ranking, `top_k` — **nada de eso cambia.** Lo único que cambia es qué modelo produce los vectores. La arquitectura del [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]] se mantiene intacta.

### 5.2 El language vision model

**🔧 Definición:** para que el modelo procese texto e imágenes, *"necesitarás usar un **language vision model**. Este tipo de modelo funciona de forma muy similar a un LLM de solo texto, pero tiene la capacidad de procesar imágenes que también han sido tokenizadas."*

**🔧 Cómo se tokeniza una imagen:** *"Un proceso típico para tokenizar una imagen es **partirla en parches separados**, cada uno representado como un token."*

**🔧 El orden de magnitud:** *"Dependiendo de su resolución, las imágenes podrían representarse en el orden de **100 tokens en el extremo bajo a cerca de 1.000 tokens** en el extremo alto."*

> [!warning] ⚠️ Conecta esto con la sección 2 antes de entusiasmarte
> Una imagen cuesta entre 100 y 1.000 tokens **de contexto**. Recuperar cinco imágenes puede sumar 5.000 tokens al prompt — más que todo el contexto de texto de un RAG típico.
>
> Todo el análisis de costo de §2.1 (*"pagas por cada token"*) y de latencia de §3.2 (*"casi toda la latencia es ejecutar un transformer"*) **se amplifica** en multimodal. No es un cambio de modelo gratuito: es una decisión de costo.

**🔧 Y de ahí en adelante, es un transformer normal:** *"Los language vision models funcionan de forma muy similar a los LLMs estándar, pasando esta secuencia de tokens multimodal a través de un transformer que puede desarrollar una comprensión matizada tanto del texto como de las imágenes en el prompt y sus relaciones. El modelo típicamente producirá tokens de texto como salida."* Es el mismo mecanismo del [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08]] con un vocabulario de entrada ampliado.

### 5.3 El caso que lo justifica todo: PDFs y presentaciones

**🔧 La ganancia práctica:** *"Lo bueno de actualizar un sistema RAG para manejar imágenes es que esto permite a tu sistema ingerir muchos formatos de archivo comunes que se convierten fácilmente en imágenes. Las diapositivas y los PDFs, por ejemplo, se tratan fácilmente como archivos de imagen."*

**🔧 Y el problema que aparece inmediatamente:** *"Un desafío con estos formatos, sin embargo, es **cuán densos en información pueden ser** las diapositivas y los PDFs. Una sola página o diapositiva puede contener texto, gráficos, leyendas e imágenes. **Un solo vector tendría dificultades para capturar todo el matiz de una página de un PDF.**"*

> [!important] 🎯 El chunking vuelve, en dos dimensiones
> *"En otras palabras, **necesitas chunkear tus imágenes igual que chunkearías tu texto**."*
>
> Es exactamente el problema del [[Guia-Maestra-RAG_06-Chunking|Tomo 06]]: un chunk demasiado grande diluye el significado y el vector se vuelve un promedio de todo y de nada. Una página de PDF completa **es** un chunk demasiado grande. La diferencia es que aquí el corte no es a lo largo de un texto lineal, sino sobre una superficie.

**🔧 El primer enfoque, y por qué no funcionó bien:** *"Inicialmente, esto se hacía con técnicas bastante sofisticadas para detectar distintas porciones de una página de PDF. Estos algoritmos intentan determinar qué pieza de la página es un gráfico, cuál es una imagen, cuál es texto, etcétera. **En la práctica, sin embargo, estas técnicas siguen siendo bastante propensas a errores y quisquillosas.**"*

**🔧 El enfoque que sí funciona — y es deliberadamente tonto:**

*"Un enfoque más nuevo, llamado **PDF RAG**, simplemente **divide cada página en una cuadrícula de cuadrados**, sin preocuparse de si esos bordes caen en lugares sensatos. Cada cuadrado se embebe entonces en un vector denso por un embedding model multimodal. Esto significa que tu página está representada por, digamos, **mil vectores en vez de uno**."*

> [!warning] ⚠️ Corrección de terminología: esto se llama **ColPali**, no "PDF RAG"
> **La técnica que el curso describe existe, está publicada y tiene otro nombre.** Es **ColPali** (Faysse et al., 2025, ICLR 2025), y la descripción del curso es una paráfrasis fiel — coincide mecanismo por mecanismo:
>
> | Lo que dice el curso | Lo que hace ColPali |
> |---|---|
> | "divide cada página en una cuadrícula, sin preocuparse de si los bordes caen en lugares sensatos" | Encoder SigLIP sobre imagen de 448×448 → rejilla fija de **32×32 = 1.024 parches**, ciega al layout |
> | "cada cuadrado se embebe en un vector denso por un modelo multimodal" | Capa de proyección que mapea cada token de salida del VLM a un espacio de **D=128** |
> | "mil vectores en vez de uno" | **1.024 parches** — la cifra es literalmente correcta (~257,5 KB por página) |
> | "funciona de forma muy similar a ColBERT… los scores se suman" | Late interaction: `LI(q,d) = Σᵢ maxⱼ ⟨E_q(i) , E_d(j)⟩` — **es MaxSim** |
>
> **"PDF RAG" no aparece como nombre de técnica en ninguna publicación revisada por pares.** Los términos establecidos son **Visual Document Retrieval (VDR)** — de ahí el nombre del benchmark que el propio paper introduce, **ViDoRe** —, *page-as-image retrieval*, o simplemente *ColPali-style late-interaction retrieval*.
>
> **Por qué importa esta corrección:** un lector que busque "PDF RAG" en la literatura **no encontrará nada**. Buscando ColPali o ViDoRe encuentra el paper, el benchmark, los modelos publicados y los sucesores. Es la diferencia entre poder seguir investigando y quedarse en el aire.
>
> **Sucesor a tener en el radar:** **ColQwen2**, el mismo framework con backbone Qwen2-VL en vez de PaliGemma, que admite resolución dinámica (hasta 768 parches). No tiene paper propio — se publica como artefacto de modelo y remite al BibTeX de ColPali. *Las cifras de mejora que circulan sobre él no están en documentación oficial; conviene contrastarlas en la leaderboard de ViDoRe antes de citarlas.*

```
   PÁGINA DE PDF                    →   CUADRÍCULA DE PARCHES
   ┌───────────────────────┐            ┌───┬───┬───┬───┬───┐
   │  Título               │            │ ▪ │ ▪ │ ▪ │ ▪ │ ▪ │
   │  ┌──────┐   texto     │            ├───┼───┼───┼───┼───┤
   │  │gráfico│  texto     │     →      │ ▪ │ ▪ │ ▪ │ ▪ │ ▪ │
   │  └──────┘   texto     │            ├───┼───┼───┼───┼───┤
   │  leyenda              │            │ ▪ │ ▪ │ ▪ │ ▪ │ ▪ │
   └───────────────────────┘            └───┴───┴───┴───┴───┘
   1 vector = todo mezclado             ~1000 vectores, cada uno
                                        embebido por el modelo multimodal
```

**🔧 Y el scoring es un viejo conocido:** *"La búsqueda vectorial funciona entonces de forma muy similar a **ColBERT**. Cada palabra en el prompt busca su cuadrado que mejor coincide en una página dada. Estos scores se suman entonces para puntuar la página completa del documento."*

> [!important] 🎯 Es ColBERT con parches en vez de tokens
> El [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]] explicó **late interaction**: en vez de comprimir el documento en un solo vector, se guarda un vector por token y cada token de la query busca su mejor pareja (MaxSim); los máximos se suman.
>
> Aquí es idéntico, cambiando *token de texto* por *parche de imagen*. **Si entendiste ColBERT, ya entendiste esto.** Y hereda su mismo trade-off: mucha más calidad de matching, a cambio de multiplicar por ~1.000 el número de vectores almacenados.

**🔧 Balance del enfoque**, con las palabras del curso:
- **A favor:** *"Este enfoque es muy flexible, ya que cualquier imagen puede dividirse en una cuadrícula de cuadrados. En la práctica, también rinde bien en tareas de retrieval."*
- **En contra:** *"Su principal desventaja es que **requiere que tu vector database almacene un número masivo de vectores**."*

> [!tip] 💡 Y aquí cierra el círculo del tomo
> ¿Cuál era la técnica para almacenar un número masivo de vectores sin arruinarse? **La sección 1.** Binary quantization comprime 32×; int8 comprime 4×; Matryoshka permite hacer el barrido con las primeras dimensiones. Multimodal a escala **no es viable sin quantization** — por eso el curso enseña la compresión antes que el multimodal, y por eso este tomo los reúne.

### 5.4 Estado de madurez

**🔧 El curso es honesto sobre dónde está esto:** *"El RAG multimodal sigue siendo una **tecnología de vanguardia** con desarrollos rápidos y activos. **La mayoría de los proveedores de LLM ofrecen un language vision model, mientras que los embedding models multimodales son ofertas relativamente más experimentales.**"*

> [!warning] ⚠️ La asimetría que condiciona lo que puedes construir hoy
> Esa frase merece leerse dos veces, porque es la restricción práctica real: **la mitad generativa está madura, la mitad de retrieval no.** Puedes pasarle imágenes a un modelo sin problema; lo difícil sigue siendo *encontrar* la imagen correcta entre un millón.
>
> Consecuencia de arquitectura: si necesitas multimodal hoy, evalúa primero si el retrieval puede seguir siendo de texto (por ejemplo, indexando descripciones o texto extraído) y reservar la parte visual para la generación. Es menos elegante que un pipeline multimodal puro, pero se apoya solo en la mitad madura del stack.
>
> *"Dicho esto, mientras buscas empujar la frontera de lo que tu sistema RAG puede hacer, espera ver progreso emocionante y continuo en el mundo del RAG multimodal."*

---

## 6. 🧭 Guía de decisión del tomo

Audiencia: 🧭 👔

```
   OPTIMIZAR UN RAG QUE YA FUNCIONA

   0. ¿Tienes observability (Tomo 10)?
        NO → PARA. Todo lo de abajo requiere medir antes y después.

   1. ¿Cuál es tu problema REAL: costo o latencia?
        Son distintos y las palancas no son las mismas.
        (Ej.: borrar documentos poco usados ahorra memoria
         pero NO reduce latencia — HNSW escala logarítmico.)

   2. COSTO
      ├─ ¿Cuantizaste? int8 en vectores es casi gratis en calidad → empieza ahí
      ├─ ¿Todo está en RAM? Solo el índice HNSW lo necesita
      ├─ ¿top_k heredado del prototipo? Bájalo midiendo la curva
      ├─ ¿Se hace retrieval SIEMPRE? Un router puede evitarlo
      └─ ¿Volumen alto y sostenido? Evalúa endpoint dedicado por hora

   3. LATENCIA
      ├─ Empieza por el LLM: casi toda la latencia es transformer
      ├─ Router a modelo pequeño para consultas simples
      ├─ ¿Prompts repetidos? Caching semántico (cuidado con el umbral)
      ├─ Mide CADA componente: latencia añadida vs. calidad añadida
      └─ Al final: binary quantization + sharding en el retriever

   4. Después de CUALQUIER cambio de arriba:
        ¿volviste a medir la CALIDAD?
        NO → no optimizaste, recortaste. (Es el error del assignment.)

   5. ¿Tu conocimiento vive en PDFs y presentaciones?
        → multimodal. Pero cuenta los tokens: 100–1.000 por imagen,
          y el retrieval multimodal aún es la parte inmadura del stack.
```

---

## 7. 📖 Glosario express de este tomo

| Término | Definición operativa |
|---|---|
| **Quantization** | Compresión de modelos o vectores reemplazando sus valores por un tipo de dato de menor precisión |
| **Integer quantization (int8)** | Mapear cada float32 a uno de 256 intervalos por dimensión. Compresión 4×, caída de pocos puntos en Recall@K |
| **Binary quantization** | 1 bit por dimensión: solo el signo. Compresión 32×, caída notoria — se compensa con rescoring en float32 |
| **Rescoring** | Recuperar rápido con vectores comprimidos y reordenar los candidatos con los vectores originales |
| **Matryoshka embedding** | Vector cuyas dimensiones están ordenadas por densidad de información, de modo que truncarlo cuesta poco |
| **Dedicated endpoint** | GPU alquilada por hora en vez de pagar por token. Más barato a volumen alto y más fiable |
| **Jerarquía de memoria** | RAM (rápida, cara) → disco → cloud object storage (lenta, barata). El índice HNSW va en RAM; el contenido, no |
| **Multi-tenancy** | Partición por usuario u organización, cada uno con su índice. Palanca de costo **y** de seguridad |
| **Router LLM** | Modelo pequeño que decide qué modelo (o qué camino) atiende cada consulta. Latencia proporcional a la dificultad |
| **Caching semántico** | Devolver una respuesta ya generada cuando el prompt nuevo se parece lo suficiente a uno anterior |
| **Sharding** | Dividir una database grande en instancias separadas para reducir la latencia de búsqueda |
| **Modelo multimodal** | Modelo que maneja varios tipos de dato. El emparejamiento más común es texto + imagen |
| **Embedding model multimodal** | Embebe texto e imágenes **en el mismo vector space**, por significado |
| **Language vision model** | LLM capaz de procesar imágenes tokenizadas además de texto. 100–1.000 tokens por imagen |
| **ColPali** | Dividir cada página en una cuadrícula de parches, embeberlos con un VLM y puntuar con late interaction estilo ColBERT. **El curso lo llama "PDF RAG"; ese nombre no existe en la literatura** |
| **Visual Document Retrieval (VDR)** | El término establecido para recuperar sobre documentos tratados como imagen. Da nombre al benchmark **ViDoRe** |

---

## 8. ✅ Checklist de comprensión

- [ ] Puedo explicar quantization con la analogía de la compresión de imagen, incluido el *"depende de dónde la uses"*.
- [ ] Sé describir el algoritmo de int8 quantization en sus cinco pasos.
- [ ] Sé que int8 comprime 4× con una caída de pocos puntos en Recall@K, y binaria 32× con caída notoria.
- [ ] Entiendo el patrón **binary + rescoring** y por qué es el mismo esquema que el re-ranking del Tomo 07.
- [ ] Sé qué hace especial a un Matryoshka embedding y cuándo conviene sobre truncar un vector normal.
- [ ] Sé cuáles son los dos mayores costos de un RAG y al menos tres palancas para cada uno.
- [ ] Entiendo por qué el índice HNSW va en RAM pero el contenido de los documentos no.
- [ ] Sé cuándo un endpoint dedicado sale más barato que pagar por token.
- [ ] Entiendo que multi-tenancy resuelve a la vez el costo y el control de acceso.
- [ ] Sé que **casi toda la latencia es ejecutar un transformer**, y por dónde empezar a optimizar.
- [ ] Sé por qué un router da *latencia proporcional a la dificultad* en vez de un compromiso a la baja.
- [ ] Entiendo el riesgo del umbral en el caching semántico (el falso positivo que parece una respuesta correcta).
- [ ] Sé medir **latencia añadida vs. calidad añadida** por componente, y eliminar los que no compensan.
- [ ] 🚨 Tengo claro que **después de comprimir hay que volver a medir la calidad** — y por qué el assignment del curso falla justo en eso.
- [ ] Sé que borrar documentos poco usados ahorra memoria pero **no** reduce latencia.
- [ ] Entiendo que un embedding multimodal pone texto e imágenes en el mismo espacio, y que el retrieval no cambia.
- [ ] Sé que una imagen cuesta 100–1.000 tokens de contexto y qué implica eso para costo y latencia.
- [ ] Entiendo por qué una página de PDF es un chunk demasiado grande y cómo lo resuelve la cuadrícula de parches.
- [ ] Sé que el retrieval multimodal es la parte **inmadura** del stack, y qué arquitectura alternativa permite eso.
- [ ] Sé que lo que el curso llama *"PDF RAG"* se llama **ColPali** en la literatura, y por qué buscar el nombre correcto importa.
- [ ] Entiendo que la compresión (4×, 32×) está garantizada pero **la calidad resultante depende del modelo** y hay que medirla.

---

## 🔗 Conexiones

- [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · Producción: observability, evaluación y security]] — **el prerrequisito estricto.** Sin las mediciones de ese tomo, todo lo de este se hace a ciegas. Además, su §6.4 documenta que la quantization es también una defensa parcial contra la inversión de embeddings.
- [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]] — el índice HNSW cuyo tamaño motiva toda la sección 1 y cuya ubicación en RAM define la §2.2. Su tabla de técnicas ya apuntaba aquí para quantization.
- [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing, arquitecturas y re-ranking]] — **ColBERT y late interaction**, el mecanismo que reaparece en §5.3 con parches de imagen en vez de tokens. Y el patrón barrer-barato-refinar-caro que se repite en §1.3.
- [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] — el problema del chunk demasiado grande, que en §5.3 reaparece en dos dimensiones sobre una página de PDF.
- [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación]] — el transformer que es *"casi toda la latencia"* (§3.2) y los parámetros de sampling que el assignment enruta mal (§4.3 ①).
- [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Hallucinations, evaluación y agentic RAG]] — los patrones agentic sobre los que se construye el router de §3.3, y las métricas de calidad que el assignment debió medir y no midió (§4.4).

---

## 📚 Referencias

**Fuente primaria — material del curso**
- DeepLearning.AI. *Retrieval Augmented Generation (RAG)*, Módulo 5: *RAG Systems in Production*. Lecciones sobre quantization, cost, latency y multimodal RAG; assignment graded `C1M5` ("Improving a RAG System"); quiz del módulo.

**Fuentes externas** — el curso alude a las tres sin citarlas; las fichas se verificaron para este tomo

- Faysse, M., Sibille, H., Wu, T., Omrani, B., Viaud, G., Hudelot, C., & Colombo, P. (2025). "ColPali: Efficient Document Retrieval with Vision Language Models". *Proceedings of ICLR 2025*. arXiv:2407.01449 — §5.3. ✅ *Verificada 2026-07-28 (arXiv y proceedings de ICLR consultados directamente; el PDF lleva el encabezado "Published as a conference paper at ICLR 2025"). **Es la técnica que el curso llama "PDF RAG"**, un nombre que no aparece en la literatura revisada por pares. El paper introduce además el benchmark **ViDoRe**.*
- Macé, Q., Loison, A., & Faysse, M. (2025). *ViDoRe Benchmark V2: Raising the Bar for Visual Retrieval*. arXiv:2505.17166 — §5.3. ✅ *Verificada 2026-07-28. Motivado por la saturación de V1 (modelos superando el 90 % de nDCG@5). Nota: es un release de benchmark, **no un paper de conferencia revisado por pares**.*
- Kusupati, A., Bhatt, G., Rege, A., Wallingford, M., Sinha, A., Ramanujan, V., Howard-Snyder, W., Chen, K., Kakade, S., Jain, P., & Farhadi, A. (2022). "Matryoshka Representation Learning". *Advances in Neural Information Processing Systems 35 (NeurIPS 2022)*. arXiv:2205.13147 — §1.4. ✅ *Verificada 2026-07-28 (proceedings de NeurIPS y arXiv consultados directamente).*
- Shakir, A., Aarsen, T., & Lee, S. (2024, 22 de marzo). *Binary and Scalar Embedding Quantization for Significantly Faster & Cheaper Retrieval*. Hugging Face Blog. [huggingface.co/blog/embedding-quantization](https://huggingface.co/blog/embedding-quantization) — §1.2–1.3. ✅ *Verificada 2026-07-28. Fuente canónica de las cifras de retención de rendimiento (Tom Aarsen mantiene `sentence-transformers`). **Es un blog técnico de referencia, no un paper revisado por pares** — se cita por sus mediciones reproducibles, documentadas también en los docs oficiales de sentence-transformers.*
- Weaviate (2024, 2 de abril). *32x Reduced Memory Usage With Binary Quantization*. [weaviate.io/blog/binary-quantization](https://weaviate.io/blog/binary-quantization) — §1.3. ✅ *Verificada 2026-07-28. Aporta el contrapunto de recalls ~0,74–0,76 sobre DBPedia, útil para no sobrevender la técnica.*

> [!warning] ⚠️ Dos afirmaciones que NO se pudieron verificar y por tanto no se citan como hechos
> - La cifra que circula ampliamente de que **ColQwen2 mejora en +5,1 nDCG@5 sobre ColPali v1.1** no aparece en documentación oficial (solo en blogs y redes). No se reproduce en el cuerpo del tomo.
> - La cifra de **"90–98 % de la calidad de búsqueda original"** que suele atribuirse a Cohere para embeddings cuantizados **no se pudo confirmar en una página de Cohere**; procede de un tercero citándolos. Los docs oficiales de Cohere documentan los tipos `int8`/`binary` pero **no publican cifras de retención de calidad**.

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · Producción: observability, evaluación y security]] · Siguiente → [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12 · ⭐ Complemento: técnicas avanzadas de query]]

> 🏁 **Fin del contenido del curso.** Con este tomo se cierran los cinco módulos del curso *Retrieval Augmented Generation* de DeepLearning.AI. Lo que sigue —Tomos 12 y 13— son **complementos de vanguardia** construidos con bibliografía externa: técnicas que el curso no cubre (query decomposition, multi-query, GraphRAG) y frameworks de orquestación (LangChain, LlamaIndex).
