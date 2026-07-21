---
title: "Tomo 03 — Information Retrieval: keyword search (TF-IDF, BM25)"
tags: [rag, information-retrieval, keyword-search, tf-idf, bm25, metadata-filtering, sparse-vectors, inverted-index, hybrid-search]
audiencias: [tecnico, puente, ejecutivo]
tomo: 03
version: 1.2
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 03 — Information Retrieval: keyword search (TF-IDF, BM25)

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02 · Fundamentos: LLMs y el pipeline RAG]] · Siguiente → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search y embeddings]]

---

> [!info] ¿Por qué importa esta sección?
> Aquí empieza el **Módulo 2** y entramos al primer componente que realmente se construye. El [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]] dejó al retriever descrito como "asigna un score de relevancia y rankea". Este tomo responde **cómo se calcula ese score**, empezando por la familia de técnicas más antigua y —esto sorprende a mucha gente— **todavía la más difícil de superar**: keyword search. Además establece el **mapa completo del retriever**, que es el plano que ordena todo el Módulo 2.
>
> Hay una razón práctica para no saltarse este tomo e ir directo a los embeddings: en benchmarks reales BM25 sigue siendo un baseline competitivo que muchos sistemas "modernos" no logran batir de forma consistente. Un ingeniero que solo sabe hacer semantic search tiene medio retriever.

> [!abstract] 👔 Impacto ejecutivo
> Keyword search es la parte del sistema que garantiza que, cuando un usuario escribe el **nombre exacto de tu producto, un SKU o un término regulatorio**, el sistema lo encuentre literalmente. Es barata, madura, auditable y explicable.
> - **Decisiones que habilita:** desplegar búsqueda sobre documentación técnica y catálogos sin depender de infraestructura de IA; cumplir requisitos de **control de acceso** por perfil de usuario vía metadata filtering; auditar por qué el sistema devolvió un documento (algo que un embedding no permite explicar con la misma claridad).
> - **Costo o riesgo de hacerlo mal:** sin metadata filtering, un sistema RAG puede **filtrar documentos confidenciales a usuarios sin permisos** — el riesgo más grave y más frecuente de esta capa. Sin keyword search, el sistema falla justo en las consultas más específicas (códigos, nombres propios, jerga técnica), que suelen ser las de mayor valor.
> - **Pregunta ejecutiva que responde:** *"¿Cómo garantizo que cada usuario vea solo lo que le corresponde, y que una búsqueda por el código exacto de un producto siempre lo encuentre?"*

---

## 1. 🗺️ El mapa del retriever

Audiencia: 🔧 🧭 👔

Antes de bajar al detalle, el plano completo. Un retriever moderno no usa **una** técnica: usa **tres**, y las combina.

```
   Prompt del usuario
          │
          ├──────────────────────┐
          ▼                      ▼
   ┌─────────────┐        ┌─────────────┐
   │  KEYWORD    │        │  SEMANTIC   │
   │  SEARCH     │        │  SEARCH     │
   │             │        │             │
   │ palabras    │        │ significado │
   │ exactas     │        │ similar     │
   │ (Tomo 03)   │        │ (Tomo 04)   │
   └──────┬──────┘        └──────┬──────┘
          │ ~20-50 docs          │ ~20-50 docs
          ▼                      ▼
   ┌────────────────────────────────────┐
   │        METADATA FILTERING          │◄─── Atributos del USUARIO
   │  mismo criterio sobre ambas listas │     (área, permisos,
   └──────┬──────────────────────┬──────┘      región, plan)
          │ filtrada             │ filtrada
          └───────────┬──────────┘
                      ▼
             ┌─────────────────┐
             │  FUSIÓN Y       │   Las dos listas se combinan
             │  RANKING FINAL  │   en un único ranking
             └────────┬────────┘
                      ▼
                 top_k docs  ──►  augmented prompt  ──►  LLM
```

A este esquema se le llama **hybrid search**, porque el ranking final descansa en múltiples técnicas. Cada una aporta algo que las otras no pueden:

| Técnica | Qué aporta | Qué NO puede hacer |
|---|---|---|
| **Keyword search** | Sensibilidad a las **palabras exactas** del prompt. Si el usuario escribió `ISO-27001`, garantiza que aparezcan documentos con ese literal | Encontrar un documento que dice lo mismo con otras palabras |
| **Semantic search** | Flexibilidad para hallar documentos de **significado similar** aunque no compartan vocabulario | Garantizar coincidencia literal de un término técnico |
| **Metadata filtering** | Excluir documentos por **criterios rígidos** (permisos, fecha, región) — lo único que da garantías duras | Buscar. Ignora por completo el contenido del documento |

> [!tip] 🧭 La decisión de diseño de fondo
> Diseñar un retriever de alto rendimiento **no es elegir la mejor técnica**: es entender las fortalezas relativas de cada una y **calibrar el balance entre ellas** según los datos y las consultas de tu proyecto. Un corpus de jerga técnica pide más peso a keyword; un corpus conversacional pide más peso a semantic.

Los dos primeros bloques —metadata filtering y keyword search— son el contenido de este tomo. Semantic search y la mecánica de fusión son el [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]].

---

## 2. 🔒 Metadata filtering

Audiencia: 🔧 🧭 👔

Empezamos por la más simple de las tres — y la más subestimada.

### 2.1 Qué es

**🔧 Definición técnica:** metadata filtering usa **criterios rígidos** para acotar los documentos que devuelve un retriever, basándose en la **metadata** del documento: título, autor, fecha de creación, privilegios de acceso, sección, región, etc. No mira el contenido; solo los atributos.

> [!tip] 💡 Analogía
> **Si alguna vez filtraste una tabla en una planilla de cálculo, ya hiciste metadata filtering.** Es exactamente eso: aplicar un criterio estricto para decidir qué filas de una colección grande quieres conservar. Nada más sofisticado que eso.

El ejemplo canónico es un diario. Cada artículo del archivo histórico está etiquetado con título, fecha de publicación, autor y sección. Consultar ese índice se parece muchísimo a escribir SQL:

```
  Artículos de la sección "Opinión",
  escritos entre junio y julio de 2024,
  por un autor determinado
  → solo pasan los que cumplen TODAS las condiciones
```

### 2.2 La sutileza que casi todos pasan por alto

Audiencia: 🔧 🧭 👔

Dos observaciones cambian por completo cómo se implementa esta capa:

> [!warning] Las dos reglas del metadata filtering
> **1. No se usa para hacer retrieval, sino para acotar el resultado de las otras técnicas.** Es un refinador, no un buscador.
>
> **2. Los filtros normalmente NO se derivan de lo que el usuario escribió en el prompt, sino de *quién es* el usuario que consulta.** Esta es la clave, y es contraintuitiva: el filtro no sale de la query, sale del contexto de la sesión.

Ejemplos concretos de la segunda regla, siguiendo con el diario:

- **Paywall.** Cada artículo lleva metadata `acceso: libre | suscriptor`. El sistema detecta si quien busca tiene suscripción activa; si no, aplica un filtro que **excluye los artículos de pago** de los resultados.
- **Región.** Cada artículo lleva la región donde se publicó. El sistema detecta desde dónde consulta el lector y devuelve solo artículos de su región.
- **Departamento.** Documentos relevantes para ingeniería vs. documentos de RR.HH. El sistema sabe a qué equipo pertenece el usuario y filtra en consecuencia.

**👔 En una frase para el negocio:** metadata filtering es la capa donde vive el **control de acceso** de tu sistema RAG. Si está mal implementada, un empleado puede recibir en su respuesta un fragmento de un documento que no debería poder leer — y el LLM se lo va a redactar amablemente.

### 2.3 Ventajas y límites

Audiencia: 🔧 🧭

| ✅ Ventajas | ⚠️ Limitaciones |
|---|---|
| **Conceptualmente simple** → fácil de entender y de depurar | **No es una técnica de búsqueda**, es un refinador de las otras |
| **Rápida, madura y muy optimizada** (décadas de bases de datos detrás) | **Excesivamente rígida**: cumple o no cumple, sin matices |
| **Es la única que permite criterios rígidos** sobre qué se recupera y qué no | **Ignora por completo el contenido** del documento |
| Auditable: siempre puedes explicar por qué un documento entró o salió | **No rankea**: una vez pasado el filtro, todos los documentos son iguales |

> [!danger] Un retriever que solo use metadata filtering es inútil
> Podría devolverte "todos los artículos de opinión de julio", pero **no tiene forma de saber cuál responde tu pregunta**. Es simple y efectivo, pero **necesita ir acompañado** de una técnica que evalúe si el contenido es realmente relevante. Casi todos los sistemas RAG que construyas incluirán filtros de metadata de algún tipo; ninguno debería depender solo de ellos.

### 2.4 En código

Audiencia: 🔧

```python
# Knowledge base minimalista: cada documento lleva su metadata
KB = [
    {"id": 1, "titulo": "Política de vacaciones", "area": "hr",  "acceso": "publico"},
    {"id": 2, "titulo": "Runbook de despliegue",  "area": "eng", "acceso": "interno"},
    {"id": 3, "titulo": "Escala salarial 2026",   "area": "hr",  "acceso": "confidencial"},
    {"id": 4, "titulo": "Guía de onboarding",     "area": "eng", "acceso": "publico"},
]

def aplicar_filtros(documentos, **criterios):
    """
    Filtra documentos que cumplan TODOS los criterios (AND lógico).
    Los criterios vienen del contexto del usuario, no de su prompt.
    """
    return [d for d in documentos
            if all(d.get(clave) == valor for clave, valor in criterios.items())]

# El filtro se arma con atributos de la SESIÓN, no de la query
usuario = {"area": "hr", "clearance": "publico"}

visibles = aplicar_filtros(KB, area=usuario["area"], acceso=usuario["clearance"])
# → ['Política de vacaciones']
# La escala salarial queda fuera aunque sea del área correcta: su acceso es confidencial
```

> [!note] 🔬 Complemento de vanguardia — pre-filtering vs. post-filtering
> El curso describe el filtro aplicándose **después** de que cada técnica de búsqueda devolvió sus 20–50 documentos. Eso se llama **post-filtering**, y tiene un modo de falla que conviene conocer antes de llevarlo a producción: **puedes terminar con menos resultados que tu `top_k`, o incluso con ninguno.**
>
> ```
>   ranking por score:  [doc3, doc1, doc2, doc4]      top_k = 2
>
>   POST-filtering  → tomas top 2 = [doc3, doc1] → filtras → [doc1]      1 documento ❌
>   PRE-filtering   → filtras primero → [doc1, doc4] → tomas top 2       2 documentos ✅
> ```
>
> Si `doc3` era confidencial, el post-filtering lo descarta **después** de haber gastado un cupo del `top_k`, y el LLM recibe menos evidencia de la que pediste. Las vector databases modernas (Weaviate, Qdrant, Pinecone, Milvus) soportan **filtered search**: aplicar el filtro *durante* la búsqueda, no después. Técnicamente no es trivial —filtrar un índice ANN sin destruir su eficiencia es un problema de investigación activo (Gollapudi et al., 2023)—, pero es la opción correcta cuando el filtro es restrictivo.
>
> **Regla práctica:** si tu filtro descarta una fracción pequeña del corpus, post-filtering está bien. Si descarta la mayor parte (ej. un usuario que solo ve el 2% de los documentos), necesitas pre-filtering o el retriever te va a devolver listas vacías.

> [!example] 📊 Caso de negocio — Asistente interno multi-área en banca
> **Problema:** un banco despliega un asistente RAG sobre su intranet: políticas, manuales de producto, procedimientos de riesgo y documentación de RR.HH. El corpus es único, pero **no todo el mundo puede ver todo**: las escalas salariales son de RR.HH., los modelos de scoring crediticio son de Riesgos, y los procedimientos de auditoría son confidenciales.
> **Técnica aplicada:** cada documento se indexa con metadata `area` y `nivel_acceso`. En cada consulta, el sistema resuelve la identidad del empleado contra el directorio corporativo y construye el filtro **desde su perfil**, nunca desde el texto de su pregunta. El filtro se aplica como pre-filtering en la vector database.
> **Resultado:** un mismo asistente sirve a toda la organización sin duplicar infraestructura, y la trazabilidad de acceso queda auditable documento por documento. El punto crítico: **un usuario no puede eludir el filtro escribiendo un prompt astuto**, porque el filtro nunca dependió de su prompt.

---

## 3. 🔤 Keyword search: la mecánica

Audiencia: 🔧 🧭

Esta técnica lleva **décadas** moviendo bases de datos y buscadores. Su simplicidad y efectividad la mantienen como componente central de los retrievers modernos.

**🔧 Idea central:** keyword search recupera documentos según **cuántas palabras comparten con el prompt**. La apuesta es directa: *un documento que contiene muchas palabras de la pregunta probablemente sea relevante*.

### 3.1 Bag of words

Audiencia: 🔧 🧭 💡

**🔧 Definición técnica:** tanto el prompt como cada documento se tratan como una **bag of words** (bolsa de palabras): **el orden se ignora por completo**; solo importa qué palabras están y con qué frecuencia.

```
   "making pizza without a pizza oven"

   ┌──────────────────────────────────┐
   │   making  ×1     pizza   ×2      │   ← el orden se perdió;
   │   without ×1     a       ×1      │     solo quedan las cuentas
   │   oven    ×1                     │
   └──────────────────────────────────┘
```

> [!tip] 💡 Analogía
> Es como meter todas las palabras de un texto en una **bolsa y agitarla**. Pierdes la gramática, pierdes el orden, pierdes los matices — pero conservas de qué se habla y cuánto se insiste en cada cosa. Sorprendentemente, para encontrar documentos eso alcanza bastante más de lo que uno esperaría.

> [!warning] Lo que se pierde en la bolsa
> `"el perro mordió al cartero"` y `"el cartero mordió al perro"` producen **exactamente la misma bag of words**. Keyword search no puede distinguirlas. Guarda este límite: es una de las razones por las que existe el semantic search.

### 3.2 Sparse vectors y el inverted index

Audiencia: 🔧

Esas cuentas se guardan en un **vector**. El vector tiene una posición por cada palabra del **vocabulary** del sistema, que fácilmente puede tener decenas de miles de posiciones. Como casi todas quedan en cero, se les llama **sparse vectors** (vectores dispersos).

Generando un sparse vector por documento y apilándolos, se obtiene una grilla: la **term-document matrix**, también llamada **inverted index**.

```
                  doc1   doc2   doc3   doc4   doc5
                 ┌──────────────────────────────────┐
      making     │   1      0      1      0      0  │
      pizza      │   2      1      0      5      0  │  ← fila = una palabra
      oven       │   1      1      0      0      1  │
      bread      │   0      0      1      0      1  │
      ...        │   ...                            │
                 └──────────────────────────────────┘
                     ▲
                     └── columna = un documento
```

> [!note] ¿Por qué "invertido"?
> Lo natural es partir de un **documento** y preguntarse qué palabras contiene. Aquí es al revés: partes de una **palabra** y encuentras de inmediato todos los documentos que la contienen. Esa inversión es lo que hace la búsqueda rápida — y se construye **una sola vez**, antes de procesar cualquier consulta. En tiempo de query solo hay que vectorizar el prompt y leer filas.

**En código** (salida real sobre el corpus de ejemplo de este tomo):

```python
from sklearn.feature_extraction.text import CountVectorizer

DOCS = [
    "Making pizza at home without a pizza oven",
    "The best pizza oven for your backyard",
    "A complete guide to making sourdough bread at home",
    "Pizza history: how pizza came to New York pizza pizza pizza",
    "Choosing an oven for baking bread and cakes",
]

# Construye el vocabulary y la term-document matrix de una vez
count_vec = CountVectorizer()
X = count_vec.fit_transform(DOCS)      # matriz sparse (scipy csr_matrix)

print(X.shape)                          # (5, 26)  -> 5 documentos, 26 términos
print(type(X).__name__)                 # csr_matrix -> almacenamiento sparse

# La fila de un término = en qué documentos aparece y cuántas veces
vocab = list(count_vec.get_feature_names_out())
print(X.toarray()[:, vocab.index("pizza")])   # [2 1 0 5 0]
print(X.toarray()[:, vocab.index("oven")])    # [1 1 0 0 1]
```

Con apenas 5 documentos la matriz ya está **72,3 % vacía**. En un corpus real la dispersión supera el 99 %, y por eso jamás se almacena densa.

### 3.3 La escalera del scoring

Audiencia: 🔧 🧭

Aquí está la parte elegante. El scoring de keyword search se construye en **cuatro peldaños**, y cada uno corrige un defecto del anterior. Entender la escalera es entender TF-IDF.

```
  PELDAÑO 1 — Conteo binario
  ───────────────────────────────────────────────────────────────
  Cada keyword del prompt que aparezca en el documento suma 1 punto.
  Prompt con 5 keywords → score máximo 5.

     ❌ Problema: no distingue un documento que menciona "pizza"
        una vez de otro que la menciona veinte veces.

  PELDAÑO 2 — Term frequency (TF)
  ───────────────────────────────────────────────────────────────
  Suma un punto por CADA aparición, no solo por la primera.

     ❌ Problema: los documentos largos ganan siempre, simplemente
        por ser largos. Un libro entero contendrá "pizza" más veces
        que un párrafo perfecto sobre pizza.

  PELDAÑO 3 — Normalización por largo
  ───────────────────────────────────────────────────────────────
  Divide el score por el número de palabras del documento.

     ✅ Ahora premia documentos donde los keywords son una PROPORCIÓN
        alta del texto.
     ❌ Problema: sigue tratando igual a "the" que a "pizza". Encontrar
        "the" no dice nada; encontrar "pizza" lo dice todo.

  PELDAÑO 4 — Inverse Document Frequency (IDF)
  ───────────────────────────────────────────────────────────────
  Pondera cada palabra por lo RARA que es en el corpus.
```

**🔧 Cómo se calcula el IDF.** Para cada palabra del vocabulary, cuenta en cuántos documentos aparece y divide por el total de documentos — eso es la *document frequency*:

```
  Corpus de 100 documentos:

    "pizza" aparece en   5 docs  →  df = 5/100  = 0.05
    "the"   aparece en 100 docs  →  df = 100/100 = 1.0

  Como queremos premiar lo raro, INVERTIMOS la fracción:

    IDF("pizza") = 100/5   = 20
    IDF("the")   = 100/100 = 1

  Pero 20× de ventaja es demasiado agresivo. Se aplica logaritmo
  para amortiguar:

    IDF(t) = log( N / df(t) )
```

> [!note] Detalle técnico: la base del logaritmo da igual
> Usar `log`, `log2` o `log10` cambia todos los valores por un factor constante, y como el ranking solo depende del **orden relativo**, el resultado final es idéntico. Las implementaciones usan logaritmo natural por convención.

**El paso final:** se multiplican las filas del inverted index por el IDF de cada palabra. La matriz resultante es la **TF-IDF matrix**. Para scorear, se hace lo mismo que antes: por cada keyword del prompt, recorrer su fila y sumarle a cada documento el valor que tenga ahí.

```
  tf_idf(t, d) = tf(t, d) · log( N / df(t) )
                  ───────    ────────────────
                  ¿cuánto     ¿qué tan rara es
                  aparece?    en el corpus?
```

> [!abstract] 👔 En una frase para el negocio
> TF-IDF le enseña al buscador que **no todas las palabras valen lo mismo**: encontrar el código de un producto en un documento importa muchísimo más que encontrar la palabra "el". Es la diferencia entre un buscador que entiende qué es distintivo y uno que cuenta palabras a ciegas.

### 3.4 TF-IDF en código

Audiencia: 🔧

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

QUERY = "making pizza without a pizza oven"

# 1. Construir la matriz TF-IDF del corpus
tfidf = TfidfVectorizer()
D = tfidf.fit_transform(DOCS)       # (5 docs × 26 términos)

# 2. Vectorizar el prompt con el MISMO vocabulary (transform, no fit_transform)
q = tfidf.transform([QUERY])

# 3. Scorear y rankear
scores = cosine_similarity(q, D).ravel()
for idx in scores.argsort()[::-1]:
    print(f"{scores[idx]:.4f}  {DOCS[idx]}")
```

Salida real:

```
0.8657  Making pizza at home without a pizza oven
0.5535  Pizza history: how pizza came to New York pizza pizza pizza
0.2895  The best pizza oven for your backyard
0.1319  A complete guide to making sourdough bread at home
0.0875  Choosing an oven for baking bread and cakes
```

> [!warning] ⚠️ El IDF de scikit-learn NO es el de la fórmula del curso
> Si calculas el IDF a mano con `log(N/df)` y lo comparas con `tfidf.idf_`, los números **no van a coincidir** — y no es un error tuyo. `TfidfVectorizer` usa por defecto `smooth_idf=True`, que aplica una variante suavizada:
>
> ```
>   fórmula del curso :  IDF(t) = log( N / df(t) )
>   scikit-learn      :  IDF(t) = log( (1+N) / (1+df(t)) ) + 1
> ```
>
> Sobre el corpus de arriba, para `pizza` (N=5, df=3): la fórmula del curso da **0.5108** y scikit-learn da **1.4055**. El `+1` final garantiza que ningún término tenga IDF cero (un término presente en todos los documentos igual aporta algo), y el suavizado evita divisiones por cero con vocabulary no visto. **El ranking apenas cambia**, pero si estás depurando valores concretos, tienes que saber cuál fórmula estás mirando.

---

## 4. 🏆 BM25: el estándar de producción

Audiencia: 🔧 🧭 👔

TF-IDF es el baseline clásico. Pero el algoritmo que usan **la mayoría de los retrievers reales** es **BM25** (*Best Matching 25*) — se llama así porque fue la **25ª variante** de una serie de funciones de scoring propuestas por sus creadores.

Funciona de forma muy parecida a TF-IDF, con dos mejoras que importan mucho en la práctica.

### 4.1 Mejora 1 — Term frequency saturation

Audiencia: 🔧 🧭 💡

**🔧 Definición técnica:** en BM25, las apariciones adicionales de un keyword tienen **rendimientos decrecientes**. Un documento que menciona "pizza" 20 veces **no es el doble de relevante** que uno que la menciona 10 veces. Ese descuento progresivo se llama **term frequency saturation**.

> [!tip] 💡 Analogía
> Es la diferencia entre el **primer vaso de agua cuando tienes sed** y el décimo. El primero te cambia la vida; el décimo apenas aporta. TF-IDF le da el mismo valor a todos los vasos; BM25 entiende que el hambre de evidencia se sacia.

```
   score
     │                                          ╱
     │                                        ╱      TF-IDF
     │                                      ╱        (lineal, sin techo)
     │                                    ╱
     │                                  ╱
     │           ╭──────────────────────────────────  BM25
     │      ╭────╯                    ╱               (satura)
     │   ╭──╯                       ╱
     │ ╭─╯                        ╱
     │╭╯                        ╱
     ╰─────────────────────────────────────────────►
        nº de apariciones del keyword en el documento
```

Esto es también una **defensa anti-spam**: sin saturación, rellenar un documento con la misma palabra cien veces lo catapultaría al primer lugar.

### 4.2 Mejora 2 — Document length normalization

Audiencia: 🔧 🧭

**🔧 Definición técnica:** BM25 sigue penalizando los documentos largos, pero **la penalización también es decreciente**. TF-IDF puede castigarlos con demasiada dureza, descontando de más textos que son largos *y* buenos. Con BM25, un documento largo **sigue puntuando alto mientras mantenga una densidad razonable de keywords**.

### 4.3 La fórmula y sus dos perillas

Audiencia: 🔧

```
                      tf(t,d) · (k1 + 1)
  score(q,d) =  Σ  IDF(t) · ──────────────────────────────────────────
               t∈q          tf(t,d) + k1 · ( 1 - b + b · |d| / avgdl )

  donde:
    tf(t,d)  = veces que el término t aparece en el documento d
    |d|      = largo del documento d (en palabras)
    avgdl    = largo promedio de los documentos del corpus
    k1       = controla la TERM FREQUENCY SATURATION
    b        = controla la LENGTH NORMALIZATION
```

> [!note] Fórmula verificada contra la lámina del curso
> Esta expresión es **idéntica a la que presenta la clase** (lámina *BM25 Scoring*), reescrita en notación inline. La lámina muestra el score de un solo término; el sumatorio sobre los keywords de la query lo agrega la narración: *"esta fórmula genera un score de relevancia para un keyword en un documento; sumándolos sobre todos los keywords se obtiene el score total del documento"*, que es lo que se usa para rankear.

A diferencia de TF-IDF, BM25 trae **dos hiperparámetros ajustables** — y eso es justamente lo que la hace flexible. Estos son los rangos tal como los define el curso:

| Perilla | Qué controla | Rango | Efecto de subirla | Efecto de bajarla |
|---|---|---|---|---|
| **`k1`** | Cuánto influye la term frequency en el score | **1.2 – 2.0** típicamente | Aumenta el impacto de la frecuencia: las repeticiones siguen sumando más tiempo (más parecido a TF-IDF) | Lo reduce: satura antes, la 2ª aparición ya aporta poco |
| **`b`** | El grado de normalización por largo del documento | **0 a 1** | `b=1` → normalización **completa** | `b=0` → **sin** normalización: el largo deja de importar |

> [!tip] 🧭 Cómo leer el rango de `b`
> El curso lo plantea como un balance: `b` **arbitra entre favorecer documentos cortos y favorecer documentos largos**. Con `b=0` el largo se ignora por completo y los documentos extensos dejan de ser penalizados; con `b=1` la penalización es total. El valor de facto de la industria es **`b=0.75`** (el default de Lucene/Elasticsearch y de la mayoría de las librerías), que es un punto intermedio con sesgo hacia normalizar.

Comprobado sobre el corpus de ejemplo (score del documento que repite `pizza` cinco veces):

```
   k1 = 0.5  → 0.5317      b = 0.00 → 0.7642
   k1 = 1.2  → 0.6775      b = 0.50 → 0.7403
   k1 = 1.5  → 0.7289      b = 0.75 → 0.7289
   k1 = 3.0  → 0.9211      b = 1.00 → 0.7179

   Subir k1 premia más la repetición.
   Subir b castiga más al documento largo.
```

**🧭 Cuándo tocar cada una:** en un retriever de producción se **tunean contra tu propio corpus**. Corpus de documentos homogéneos en largo → `b` importa poco. Corpus con documentos muy dispares (tweets y PDFs de 80 páginas en el mismo índice) → `b` es crítico. Los valores por defecto `k1=1.5, b=0.75` son un punto de partida razonable, no una respuesta.

### 4.4 BM25 en código (versión didáctica)

Audiencia: 🔧

Esta versión usa `rank_bm25`, que expone `k1` y `b` de forma explícita — útil para experimentar con las perillas. El stack que usa el curso es distinto y está en la sección 4.5.

```python
from rank_bm25 import BM25Okapi

# BM25 trabaja sobre listas de tokens, no sobre strings
corpus_tokens = [doc.lower().split() for doc in DOCS]

bm25 = BM25Okapi(corpus_tokens, k1=1.5, b=0.75)

# Scores de todos los documentos frente a la query
scores = bm25.get_scores(QUERY.lower().split())

# O directamente los top_k documentos
top_3 = bm25.get_top_n(QUERY.lower().split(), DOCS, n=3)
```

> [!danger] ⚠️ Resultado real: la trampa de las stopwords
> Ejecutando ese código tal cual sobre el corpus de ejemplo, el ranking sale así:
>
> ```
>   1. 2.6148  Making pizza at home without a pizza oven
>   2. 0.7289  Pizza history: how pizza came to New York pizza pizza pizza
>   3. 0.6591  A complete guide to making sourdough bread at home   ← ¿?
>   4. 0.6505  The best pizza oven for your backyard
> ```
>
> **Un documento sobre pan de masa madre le gana al documento sobre hornos de pizza.** No es un bug de la librería: es que `.split()` no elimina **stopwords**, así que el artículo `"a"` y el verbo `"making"` están puntuando. En este corpus `"making"` aparece en 2 de 5 documentos y por lo tanto tiene **más IDF (0.3365) que `"pizza"` (0.1987)**, que aparece en 3. El documento de pan comparte `making` + `a` y eso le alcanza para colarse.
>
> Quitando stopwords antes de tokenizar, el ranking se corrige solo:
>
> ```
>   1. 1.1651  Making pizza at home without a pizza oven
>   2. 0.6980  Pizza history: how pizza came to New York pizza pizza pizza
>   3. 0.6933  The best pizza oven for your backyard          ← ahora sí
>   4. 0.3313  A complete guide to making sourdough bread at home
> ```
>
> **La lección:** en keyword search el **preprocesamiento no es un paso opcional de limpieza, es parte del algoritmo.** Tokenización, minúsculas, stopwords y stemming/lemmatización cambian el ranking de forma material. Es también la demostración más nítida de que keyword search **no entiende de qué habla el texto**: solo cuenta coincidencias ponderadas.

> [!note] Detalle de implementación: el IDF puede ser negativo
> En la formulación clásica de BM25, un término presente en **más de la mitad** de los documentos produce un IDF negativo — es decir, contenerlo te *restaría* puntos. Las implementaciones lo corrigen de formas distintas: **Lucene/Elasticsearch** usan `log(1 + (N - df + 0.5)/(df + 0.5))`, que es siempre positivo; **`rank_bm25`** deja la fórmula clásica pero pisa los IDF negativos con un piso de `epsilon · average_idf` (con `epsilon=0.25` por defecto). Si comparas scores entre dos motores BM25 y no cuadran, esta suele ser la razón.

### 4.5 El stack real del curso: `bm25s`

Audiencia: 🔧

El assignment C1M2 no usa `rank_bm25` sino **[`bm25s`](https://bm25s.github.io/)**, una implementación optimizada que se encarga ella misma de la tokenización (incluidas las stopwords, el problema de la sección anterior).

El corpus se construye **concatenando título y descripción** de cada noticia:

```python
import bm25s

# 870 noticias de BBC / The Guardian / WSJ (news_data_dedup.csv)
corpus = [x['title'] + " " + x['description'] for x in NEWS_DATA]

retriever = bm25s.BM25(corpus=corpus)
retriever.index(bm25s.tokenize(corpus))       # se indexa UNA vez

# En cada consulta: tokenizar la query y recuperar
results, scores = retriever.retrieve(bm25s.tokenize(query), k=top_k)
```

Resultado real sobre la query `"What are the recent news about GDP?"` (ejecutado y verificado contra la salida esperada del assignment):

```
   índice  score   documento
   ─────────────────────────────────────────────────────────────────────────
     752   5.0626  GDP and the Dow Are Up. But What About American Well-Being?
     673   4.8727  What the GDP Report Says About Inflation: A Hot First Quarter
     289   3.8447  A GDP Warning as Signs of Stagflation Appear
     626   3.3193  ...
      43   2.9149  ...
```

> [!danger] ⚠️ Cuidado con recuperar los índices vía `corpus.index(doc)`
> Cuando instancias `bm25s.BM25(corpus=corpus)`, el método `.retrieve()` devuelve **los textos** de los documentos, no sus índices. El camino natural para recuperar los índices —y el que sugieren las pistas del assignment— es:
>
> ```python
> top_k_indices = [corpus.index(doc) for doc in results[0]]   # ⚠️ frágil
> ```
>
> Tiene dos problemas reales:
> 1. **`list.index()` devuelve la PRIMERA coincidencia.** Verificado sobre este mismo corpus: **21 de los 870 documentos tienen un `title + description` exactamente idéntico a otro.** Para esos, el índice recuperado puede no ser el del documento que BM25 realmente rankeó. (El archivo se llama `news_data_dedup.csv`, pero la deduplicación no eliminó los textos repetidos.)
> 2. Es un escaneo lineal por cada resultado — irrelevante con 870 documentos, costoso con millones.
>
> **La alternativa limpia:** si **no** le pasas el argumento `corpus`, `bm25s` devuelve directamente los índices:
>
> ```python
> retriever = bm25s.BM25()                       # sin corpus=
> retriever.index(bm25s.tokenize(corpus))
>
> results, scores = retriever.retrieve(bm25s.tokenize(query), k=top_k)
> top_k_indices = [int(i) for i in results[0]]   # ✅ índices directos
> ```
>
> Ambas variantes devuelven `[752, 673, 289, 626, 43]` en esta query concreta; la segunda es ~17× más rápida en la recuperación de índices y es inmune al problema de los duplicados.

### 4.6 TF-IDF vs. BM25

Audiencia: 🔧 🧭

| Criterio | TF-IDF | BM25 | Cuándo conviene cada uno |
|---|---|---|---|
| **Term frequency** | Lineal, sin techo | Saturación con rendimientos decrecientes | BM25 salvo que necesites replicar un baseline clásico |
| **Largo del documento** | Penalización directa, puede ser excesiva | Penalización decreciente y ajustable | BM25 si tu corpus tiene documentos de largo dispar |
| **Hiperparámetros** | Ninguno | `k1` y `b` ajustables | TF-IDF si no tienes datos para tunear; BM25 si sí |
| **Costo computacional** | Bajo | Prácticamente el mismo | Empate: BM25 no cuesta más caro |
| **Calidad de recuperación** | Baseline razonable | **Significativamente mejor** en la práctica | BM25 |
| **Rol típico** | Didáctico, features para ML clásico | **El estándar de producción** | — |

> [!tip] 🧭 La recomendación práctica
> **Usa BM25.** Cuesta esencialmente lo mismo que TF-IDF, rinde bastante mejor y sus dos perillas te dan margen para adaptarlo a tus datos. TF-IDF vale la pena entenderlo porque explica *por qué* BM25 hace lo que hace — pero rara vez es la elección correcta para un sistema nuevo.

---

## 5. ⚖️ Fortalezas y debilidades de keyword search

Audiencia: 🔧 🧭 👔

**Recapitulando la técnica completa:** documentos y prompts se convierten en sparse vectors que cuentan apariciones del vocabulary; TF-IDF o BM25 procesan esos vectores para scorear y rankear, tomando en cuenta la rareza del keyword, su frecuencia en el documento y el largo del documento.

> [!tip] ✅ Fortalezas
> - **Simplicidad.** Un enfoque directo que funciona bien en la práctica, sin infraestructura de IA detrás.
> - **Un baseline sorprendentemente duro.** Frecuentemente rinde muy bien por sí solo y **fija una marca que técnicas más avanzadas a veces no logran superar**. Si tu sistema semántico no le gana a BM25, tienes un problema que resolver antes de seguir agregando complejidad.
> - **Garantía de coincidencia literal.** Los documentos recuperados **contienen** las palabras del usuario. Cuando esperas terminología técnica, nombres exactos de producto, SKUs, códigos normativos o identificadores, este matching exacto es **irremplazable**.
> - **Explicable y auditable.** Puedes señalar exactamente qué término hizo subir un documento.
> - **Barato.** Sin GPU, sin llamadas a modelos, sin costo por token.

> [!warning] ⚠️ La debilidad estructural
> Keyword search depende de que la query **contenga literalmente las mismas palabras** que el documento. Si el usuario formula algo con el mismo significado pero distinto vocabulario, **no encuentra nada**. Es el problema del *vocabulary mismatch*:
>
> ```
>   Query:      "¿cómo hago que mi auto arranque en invierno?"
>   Documento:  "Procedimiento de encendido del vehículo a baja temperatura"
>
>   Palabras en común: prácticamente ninguna.
>   Keyword search:    no lo encuentra. ❌
> ```
>
> Ese hueco exacto es el que viene a llenar el **semantic search** del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]].

---

## 6. 📊 Las tres técnicas, lado a lado

Audiencia: 🔧 🧭 👔

| | **Metadata filtering** | **Keyword search** | **Semantic search** (Tomo 04) |
|---|---|---|---|
| **Qué mira** | Atributos del documento | Palabras literales | Significado |
| **Qué devuelve** | Subconjunto (sin orden) | Ranking por score | Ranking por similarity |
| **De dónde sale el criterio** | Del **usuario**/sesión | Del prompt | Del prompt |
| **Costo computacional** | Muy bajo | Bajo | Medio-alto (embeddings) |
| **Explicabilidad** | Total | Alta | Baja |
| **Falla cuando…** | Se usa sola (no busca) | Cambia el vocabulario | Se necesita coincidencia exacta |
| **Cuándo conviene** | **Siempre**, como capa de permisos y acotación | Jerga técnica, códigos, nombres propios, corpus especializado | Lenguaje natural, preguntas conversacionales, sinónimos |

> [!abstract] 👔 La conclusión ejecutiva del tomo
> No existe "la mejor técnica de búsqueda". Existe una **combinación calibrada** para tus datos y tus usuarios. La pregunta correcta ante un proveedor o un equipo interno no es *"¿usan IA para buscar?"*, sino *"¿cómo balancean búsqueda exacta y búsqueda semántica, y cómo garantizan los permisos?"*.

---

## 7. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Metadata filtering** | Filtrar documentos por sus etiquetas (área, permisos, fecha). La capa de control de acceso |
| **Pre/post-filtering** | Si el filtro de permisos se aplica antes o después de buscar. Aplicarlo después puede dejarte sin resultados |
| **Keyword search** | Buscar por las palabras exactas que escribió el usuario |
| **Bag of words** | Tratar un texto como un montón de palabras sueltas, ignorando el orden |
| **Sparse vector** | La lista de conteos de palabras de un texto; casi toda ceros |
| **Vocabulary** | El catálogo de todas las palabras que el sistema conoce |
| **Inverted index** | El índice que va de una palabra a todos los documentos que la contienen. Lo que hace rápida la búsqueda |
| **TF (term frequency)** | Cuántas veces aparece una palabra en un documento |
| **IDF (inverse document frequency)** | Qué tan rara es una palabra en todo el corpus. Las raras valen más |
| **TF-IDF** | El baseline clásico: frecuencia × rareza |
| **BM25** | El estándar de producción. TF-IDF con rendimientos decrecientes y dos perillas ajustables |
| **Term frequency saturation** | Que repetir una palabra deje de sumar puntos indefinidamente |
| **Stopwords** | Palabras vacías ("el", "de", "a") que conviene descartar antes de buscar |
| **Vocabulary mismatch** | Cuando el usuario y el documento dicen lo mismo con palabras distintas, y la búsqueda literal falla |
| **Hybrid search** | Combinar búsqueda exacta + semántica + filtros en un único ranking |

---

## 8. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Puedo dibujar el mapa del retriever: dos búsquedas en paralelo, filtro de metadata, fusión, `top_k`.
- [ ] Entiendo que los filtros de metadata salen del **perfil del usuario**, no de su prompt — y por qué eso importa para seguridad.
- [ ] Sé por qué un retriever basado solo en metadata filtering sería inútil.
- [ ] Distingo pre-filtering de post-filtering y sé cuándo el segundo rompe el `top_k`.
- [ ] Sé explicar qué es una bag of words y qué información se pierde al usarla.
- [ ] Entiendo qué es un sparse vector y por qué el inverted index acelera la búsqueda.
- [ ] Puedo recorrer la escalera del scoring: binario → TF → normalizado por largo → TF-IDF.
- [ ] Sé calcular un IDF a mano y explicar por qué se le aplica logaritmo.
- [ ] Conozco las dos mejoras de BM25 sobre TF-IDF y para qué sirven `k1` y `b`.
- [ ] Entiendo por qué el preprocesamiento (stopwords) cambia materialmente el ranking.
- [ ] Puedo explicar el vocabulary mismatch y por qué obliga a incorporar semantic search.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02 · Fundamentos: LLMs y el pipeline RAG]] (de dónde viene el score de relevancia)
- Siguiente tomo → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search y embeddings]] (la otra mitad del retriever, y la fusión de ambas listas)
- El motor a escala → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]] (dónde vive el índice en producción, y el filtered search)
- Cómo se parten los documentos antes de indexarlos → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]]
- Mejorar el ranking antes del corte `top_k` → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Reranking]]
- Medir si el retriever funciona → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Evaluación]]
- Hybrid search en profundidad (RRF, pesos) → [[Guia-Maestra-RAG_12-Tecnicas-Avanzadas-Hybrid-HyDE-GraphRAG|Tomo 12 · Técnicas avanzadas]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 2: Information Retrieval & Search Foundations (Coursera). Lecciones sobre arquitectura del retriever, metadata filtering, keyword search / TF-IDF y BM25; **assignment graded C1M2** (implementación de BM25 con `bm25s` sobre el dataset de noticias).

**Fuentes externas (complemento con bibliografía verificable):**
- Spärck Jones, K. (1972). *A Statistical Interpretation of Term Specificity and Its Application in Retrieval*. Journal of Documentation, 28(1). — Paper que introduce el IDF.
- Robertson, S. E., Walker, S., Jones, S., Hancock-Beaulieu, M. M. & Gatford, M. (1995). *Okapi at TREC-3*. Proceedings of the Third Text REtrieval Conference (TREC-3), NIST. — Origen de BM25.
- Robertson, S. & Zaragoza, H. (2009). *The Probabilistic Relevance Framework: BM25 and Beyond*. Foundations and Trends in Information Retrieval, 3(4). — Tratamiento formal de BM25 y sus hiperparámetros.
- Manning, C. D., Raghavan, P. & Schütze, H. (2008). *Introduction to Information Retrieval*. Cambridge University Press. — Texto canónico: bag of words, inverted index, TF-IDF, normalización.
- Gollapudi, S. et al. (2023). *Filtered-DiskANN: Graph Algorithms for Approximate Nearest Neighbor Search with Filters*. Proceedings of the ACM Web Conference (WWW '23). — Sustento del problema de combinar filtros con búsqueda vectorial eficiente (pre-filtering).
- Documentación oficial: `scikit-learn` (`TfidfVectorizer`, `CountVectorizer`), `rank_bm25` (`BM25Okapi`) y **`bm25s`** (https://bm25s.github.io/, la librería que usa el assignment del curso) — variantes de fórmula y parámetros por defecto citados en este tomo.

> [!note] Sobre el código de este tomo — qué es del curso y qué es complemento
> - **Del curso (fuente primaria):** la sección 4.5 reproduce el stack real del **assignment graded C1M2**, que implementa BM25 con la librería `bm25s` sobre el dataset de 870 noticias. La salida mostrada fue **ejecutada y coincide exactamente con la salida esperada del assignment** (`[752, 673, 289, 626, 43]`).
> - **Complemento propio:** el resto del código y los valores numéricos —construcción del inverted index, TF-IDF con `scikit-learn`, los experimentos con `k1` y `b`, la trampa de las stopwords y el ejemplo de metadata filtering— son escritos por mí sobre `scikit-learn 1.9.0`, `numpy 2.5.1`, `pandas` y `rank_bm25`, y también **ejecutados y verificados**. Ninguna salida de este tomo es ilustrativa.
>
> La teoría proviene íntegramente del curso; la advertencia sobre `corpus.index()` y los 21 documentos duplicados es un hallazgo propio verificado sobre el dataset del assignment.
>
> **Fórmulas:** la de BM25 (sección 4.3) y sus rangos de hiperparámetros están **contrastados contra las láminas del curso** (*BM25 Scoring* y *BM25 Tunable Parameters*) y coinciden. La de TF-IDF y la de IDF siguen la formulación que desarrolla la clase.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search y embeddings]]**, donde atacamos el vocabulary mismatch: cómo un embedding model convierte texto en vectores densos que capturan significado, cómo se mide la cercanía entre ellos (cosine similarity vs. distancia euclidiana), cómo se fusionan las dos listas en hybrid search, y cómo se evalúa si el retriever realmente está funcionando (precision@k y recall@k).
