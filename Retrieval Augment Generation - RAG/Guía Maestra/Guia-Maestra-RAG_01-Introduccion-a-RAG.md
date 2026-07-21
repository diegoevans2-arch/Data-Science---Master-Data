---
title: "Tomo 01 — Introducción a RAG (Retrieval-Augmented Generation)"
tags: [rag, llm, retrieval, generative-ai, data-science, knowledge-cutoff, hallucination, indexing]
audiencias: [tecnico, puente, ejecutivo]
tomo: 01
version: 2.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 01 — Introducción a RAG

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Siguiente → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02 · Fundamentos: LLMs y el pipeline RAG]]

---

> [!info] ¿Por qué importa esta sección?
> Este es el tomo cero conceptual de todo el manual. Antes de tocar embeddings, vector stores o re-ranking, hay que entender **qué problema resuelve RAG y por qué se volvió el patrón dominante** para conectar modelos de lenguaje con conocimiento propio y actualizado. Todo lo que viene después —chunking, retrieval, evaluación— son piezas al servicio de la idea que se explica aquí. Si un stakeholder solo va a leer un tomo, es este.

> [!abstract] 👔 Impacto ejecutivo
> RAG es hoy la vía más rápida, barata y segura para que un LLM responda usando **los datos de tu organización** (documentos internos, catálogos, políticas, noticias recientes) sin tener que reentrenar el modelo.
> - **Decisiones que habilita:** desplegar asistentes que citan fuentes internas, chatbots de soporte sobre tu propia documentación, búsqueda semántica sobre bases de conocimiento, análisis de información posterior al entrenamiento del modelo.
> - **Costo o riesgo de hacerlo mal:** sin RAG, el modelo "alucina" (inventa datos con tono seguro), no conoce nada posterior a su fecha de corte, y no puede citar de dónde sacó una afirmación → riesgo reputacional y legal.
> - **Pregunta ejecutiva que responde:** *"¿Cómo hago que la IA responda con MI información, sin filtrarla a terceros ni pagar un reentrenamiento millonario?"*

---

## 1. 🩺 El problema que RAG viene a resolver

Audiencia: 🔧 🧭 👔

Un LLM (Large Language Model) como GPT, Llama o Claude aprende todo lo que "sabe" durante su fase de entrenamiento, que termina en una fecha concreta. Esa fecha se llama **knowledge cutoff** (fecha de corte de conocimiento). El modelo del ejercicio que usamos en el curso, `llama-3-1-8b-instruct-turbo`, tiene su corte en **diciembre de 2023**.

Esto genera tres limitaciones estructurales:

> [!danger] Los tres pecados del LLM "a secas"
> 1. **Knowledge cutoff** — no sabe nada que haya ocurrido después de su fecha de corte. Pregúntale por un evento de 2024 y no tiene forma de conocerlo.
> 2. **Alucinaciones (hallucinations)** — cuando no sabe algo, no dice "no sé": **inventa** una respuesta con total seguridad gramatical. Es el fallo más peligroso porque es indistinguible de una respuesta correcta.
> 3. **Sin acceso a datos privados** — nunca vio tus documentos internos, tu catálogo de productos ni tus políticas. Vive encerrado en lo que aprendió de internet público.

> [!tip] 💡 Analogía
> Un LLM sin RAG es como un **médico brillante que estudió toda la carrera hasta 2023 y desde entonces vive aislado en una isla sin internet**. Sabe muchísimo de medicina general, pero si le preguntas por un fármaco aprobado en 2024, o por el historial clínico específico de *tu* paciente, no tiene cómo saberlo. RAG es **darle acceso al expediente del paciente y a las últimas publicaciones médicas justo antes de que te responda**. Sigue siendo el mismo médico con el mismo cerebro; solo que ahora responde con la información correcta enfrente.

### 1.1 El cutoff no se arregla solo

Audiencia: 🧭 👔

Una objeción frecuente en reuniones ejecutivas: *"¿no basta con esperar al próximo modelo?"*. No, por tres razones:

```
   El cutoff se mueve...            ...pero el problema no desaparece

   modelo v1  ──┐
                ├── cutoff dic-2023      ① Siempre existe una brecha entre
   modelo v2  ──┘                            el cutoff y HOY. Nunca es cero.
                ├── cutoff jun-2025
                                         ② Tus datos privados NUNCA estuvieron
   hoy ─────────┤                            en el entrenamiento de ningún
                                             modelo, con cualquier cutoff.

                                         ③ Reentrenar cuesta caro y lento.
                                            Tu catálogo cambia hoy, no en la
                                            próxima versión del modelo.
```

**👔 En una frase para el negocio:** el knowledge cutoff no es un bug que el proveedor vaya a arreglar; es una propiedad estructural de cómo se construyen estos modelos. RAG no es un parche temporal: es la arquitectura correcta.

---

## 2. 🎯 Qué es RAG, en concreto

Audiencia: 🔧 🧭 👔

**RAG (Retrieval-Augmented Generation)** — Generación Aumentada por Recuperación — es una técnica que, **antes** de que el modelo responda, **recupera** (retrieval) fragmentos de información relevante desde una fuente externa (tus documentos) y los **inyecta en el prompt** junto a la pregunta del usuario. El modelo entonces genera (generation) su respuesta basándose en ese contexto recién entregado.

La ecuación mental es simple:

```
  Respuesta = LLM( pregunta + información recuperada relevante )
```

En lugar de confiar en la memoria del modelo, le entregas la "chuleta" correcta en el momento justo. No modificas el modelo: modificas **lo que ve** cuando le preguntas.

> [!abstract] 👔 En una frase para el negocio
> RAG le da al modelo acceso de solo lectura a tu conocimiento en el instante de responder, sin reentrenarlo y sin que tus datos salgan de tu control.

---

## 3. ⏱️ Las dos fases: indexing y retrieval

Audiencia: 🔧 🧭 👔

Esta es la distinción que más confusión evita, y conviene fijarla antes de mirar el pipeline. **Un sistema RAG no corre en un solo momento: corre en dos, y son radicalmente distintos.**

```
  ═══════════════════════════════════════════════════════════════════
   FASE 1 · INDEXING       offline — se corre UNA VEZ y se repite solo
                           cuando cambian los documentos
  ═══════════════════════════════════════════════════════════════════

     Documentos          Partir en          Convertir a        Guardar en
     crudos        ──►   fragmentos   ──►   vectores     ──►   el índice
     PDF · CSV           (chunking)         (embedding)
     HTML · DB                                                     │
                                                                   ▼
                                                          ╔═════════════════╗
                                                          ║ KNOWLEDGE BASE  ║
                                                          ║    indexada     ║
                                                          ╚═════════════════╝
                                                                   │
  ═════════════════════════════════════════════════════════════════│═════
   FASE 2 · RETRIEVAL      online — se corre en CADA consulta      │
  ═════════════════════════════════════════════════════════════════│═════
                                                                   │
     Pregunta ──►  1. RETRIEVAL  ◄──────────────────────────────────┘
     del usuario         │
                         ▼
                   2. FORMATTING ──► 3. AUGMENTATION ──► 4. GENERATION
                                                              │
                                                              ▼
                                                          Respuesta
```

> [!tip] 💡 Analogía
> Es la diferencia entre **ordenar una biblioteca** y **consultarla**. Ordenar los libros por tema, ponerles etiquetas y armar el fichero es un trabajo grande que haces **una vez** (y repites cuando llegan libros nuevos). Buscar un libro concreto, en cambio, lo haces **cada vez que alguien pregunta**, y toma segundos porque el trabajo pesado ya está hecho. Si tuvieras que reordenar la biblioteca entera con cada consulta, el sistema sería inservible.

**🧭 Por qué esta distinción cambia decisiones de diseño:**

| | **Indexing** (fase 1) | **Retrieval** (fase 2) |
|---|---|---|
| **Cuándo corre** | Offline, antes de cualquier consulta | Online, en cada consulta |
| **Frecuencia** | Una vez + actualizaciones | Miles de veces al día |
| **Tolerancia a latencia** | Alta: puede tardar horas | **Nula**: el usuario está esperando |
| **Dónde duele el costo** | Costo puntual de procesamiento | **Costo recurrente por consulta** |
| **Qué se decide aquí** | Cómo partir los documentos, qué embedding model usar, qué metadata guardar | `top_k`, balance de técnicas de búsqueda, formato del prompt |

> [!warning] La consecuencia que más sorprende
> **Cambiar una decisión de la fase 1 obliga a reprocesar todo.** Si cambias de embedding model o de estrategia de chunking, no basta con desplegar código nuevo: hay que **re-indexar la knowledge base completa**. Por eso las decisiones de indexing se toman con más cuidado que las de retrieval — estas últimas se pueden ajustar en caliente.

Este tomo y el pipeline que verás abajo se concentran en la **fase 2**, que es donde ocurre la "magia" visible. La fase 1 se desarrolla en [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] y [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases]].

---

## 4. 🏗️ La arquitectura de consulta, paso a paso

Audiencia: 🔧 🧭 👔

Ya con la knowledge base indexada, este es el flujo que se dispara con cada pregunta. Es el "esqueleto" que iremos desarrollando pieza por pieza en los tomos siguientes:

```
                          ┌─────────────────────────┐
   Pregunta del usuario ─►│  1. RETRIEVAL           │
   ("¿Qué pasó con la     │  Busca los documentos   │
    economía en 2024?")   │  más relevantes         │◄── Knowledge base
                          └───────────┬─────────────┘    (indexada en la fase 1)
                                      │
                                      ▼
                          ┌─────────────────────────┐
                          │  2. FORMATTING          │
                          │  Ordena los documentos  │
                          │  recuperados en texto   │
                          └───────────┬─────────────┘
                                      ▼
                          ┌─────────────────────────┐
                          │  3. AUGMENTATION        │
                          │  pregunta + documentos  │
                          │  = prompt aumentado     │
                          └───────────┬─────────────┘
                                      ▼
                          ┌─────────────────────────┐
                          │  4. GENERATION          │
                          │  El LLM responde usando │──► Respuesta fundamentada
                          │  el contexto entregado  │     (y con fuentes citables)
                          └─────────────────────────┘
```

Desglosando cada etapa:

### 🔧 Etapa 1 — Retrieval (recuperación)
Dada la pregunta del usuario, el sistema busca en la knowledge base los `top_k` fragmentos más **relevantes**. Esa relevancia se calcula combinando búsqueda por palabras exactas y por significado —lo veremos en detalle en los Tomos [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|03]] y [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|04]]—. Devuelve una lista de referencias a los documentos más pertinentes.

### 🔧 Etapa 2 — Formatting (formateo)
Los documentos recuperados se convierten en un bloque de texto estructurado y legible para el modelo (título, descripción, fecha, fuente). Un buen formateo mejora la calidad de la respuesta y permite que el modelo **cite** de dónde sacó cada dato.

### 🔧 Etapa 3 — Augmentation (aumento del prompt)
Se combina la pregunta original con el contexto recuperado en un único **prompt aumentado**. Aquí es donde ocurre la "magia": el modelo recibe la pregunta *y* la evidencia al mismo tiempo. Una plantilla típica:

```
Responde la pregunta del usuario usando la información provista abajo.
La información es de 2024 y no debes basarte solo en tu conocimiento previo.

Pregunta: {query}

Noticias 2024: {documents}
```

### 🔧 Etapa 4 — Generation (generación)
El LLM produce la respuesta final, ahora **fundamentada** en el contexto entregado en lugar de en su memoria. El resultado es más preciso, actualizado y —crucialmente— **trazable a una fuente**.

> [!tip] 💡 Analogía del bibliotecario
> Imagina que le haces una pregunta a un **bibliotecario experto**. No te responde de memoria: primero **camina hasta la estantería y trae los 3 libros más relevantes** (retrieval), los **abre en las páginas correctas y te las ordena sobre la mesa** (formatting), **lee tu pregunta junto a esas páginas** (augmentation), y recién entonces **te da una respuesta apoyándose en lo que tiene enfrente** (generation). RAG convierte al LLM en ese bibliotecario en vez de en alguien que improvisa de memoria.

---

## 5. 💻 El pipeline mínimo en código

Audiencia: 🔧

Aquí está el corazón técnico de un RAG básico, tal como lo construimos en el assignment C1M1 del curso. La profundidad de cada componente (cómo funciona `retrieve` por dentro, qué embeddings usa) se desarrolla en tomos posteriores.

> [!note] Contexto del ejercicio
> **Dataset:** *News Headlines 2024* (Kaggle) — titulares de BBC News, The Guardian y WSJ.
> **Modelo:** `llama-3-1-8b-instruct-turbo` vía Together AI, con cutoff en **diciembre de 2023**.
> **Objetivo:** que el modelo pueda hablar de eventos de 2024 que nunca vio en su entrenamiento.

**Paso 0 — La forma real de los datos.** Cada documento de la knowledge base es un diccionario. Vale la pena mirarlo porque el formateo posterior depende de estos campos:

```python
{
  "guid":         "5dae28f191cfd1047f67c409e616fc3f",
  "title":        "Paris's Moulin Rouge loses windmill sails overnight",
  "description":  "The cause of the sails' collapse from the roof of the world "
                  "famous cabaret club is not yet clear.",
  "venue":        "BBC",
  "url":          "https://www.bbc.co.uk/news/world-europe-68895836",
  "published_at": "2024-04-25",
  "updated_at":   "2024-04-26"
}
```

Los campos que realmente alimentan al LLM son **`title`, `description`, `published_at` y `url`**: contenido, actualidad y fuente citable.

**Paso 1 — Las dos funciones que el curso entrega hechas.** Conviene conocer sus contratos porque todo lo demás se apoya en ellos:

```python
retrieve(query: str, top_k: int) -> list[int]
# Recibe una query y devuelve los ÍNDICES de los top_k documentos más similares.
# Su implementación (embeddings + similarity) se abre en los Tomos 03 y 04.

query_news(indices: list[int]) -> list[dict]
# Recibe índices y devuelve los documentos completos correspondientes.
```

**Paso 2 — Recuperar los datos relevantes.** Consolida ambas funciones en una sola:

```python
def get_relevant_data(query: str, top_k: int = 5) -> list[dict]:
    """
    Recupera los top_k documentos más relevantes para una query.
    1. retrieve()   -> índices de los documentos más similares
    2. query_news() -> documentos completos a partir de los índices
    """
    # Índices de los top_k documentos relevantes según la query
    relevant_indices = retrieve(query, top_k)

    # Documentos completos correspondientes a esos índices
    relevant_data = query_news(relevant_indices)

    return relevant_data
```

Comportamiento real, con `top_k=1`:

```python
>>> get_relevant_data("Greatest storms in the US", top_k=1)
[{'title': 'Large tornado seen touching down in Nebraska',
  'description': 'Severe and powerful storms have moved across several US states, '
                 'leaving many experiencing power shortages.',
  'venue': 'BBC',
  'url': 'https://www.bbc.co.uk/news/world-us-canada-68860070',
  'published_at': '2024-04-26', ...}]
```

Fíjate en lo que acaba de pasar: la query decía `"storms"`, el documento dice `"tornado"`. **No hubo coincidencia literal de palabras** y aun así lo encontró. Ese es el retriever semántico trabajando — el mecanismo se abre en el [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]].

**Paso 3 — Formatear los documentos recuperados.** Convierte la lista de documentos en un string estructurado que el modelo pueda leer con claridad:

```python
def format_relevant_data(relevant_data):
    """
    Formatea una lista de documentos en un string estructurado para el prompt.
    """
    formatted_documents = []

    for document in relevant_data:
        # Un documento por línea, con salto de línea antes de la URL
        formatted_document = (
            f"Title: {document['title']}, "
            f"Description: {document['description']}, "
            f"Published at: {document['published_at']}\n"
            f"URL: {document['url']}"
        )
        formatted_documents.append(formatted_document)

    return "\n".join(formatted_documents)
```

Estas dos funciones alimentan a `generate_final_prompt` (que combina query + documentos) y a `llm_call` (que orquesta todo y llama al modelo). **Ambas se detallan en el [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG#5. 💻 El pipeline completo en código|Tomo 02]]**, junto con un bug de formato del prompt original que conviene conocer.

> [!warning] Regla de oro del formateo
> El string que entregas al modelo es *literalmente* lo que verá como evidencia. Si el formato es confuso o le faltan campos (como la fuente o la fecha), el modelo no podrá citar correctamente ni ponderar la actualidad de la información. **El formateo no es cosmético: es parte del contrato de calidad del sistema.**

---

## 6. 🔬 Con RAG vs sin RAG: el experimento, con resultados reales

Audiencia: 🔧 🧭 👔

El valor de RAG se ve mejor por contraste. El assignment permite correr **la misma pregunta** con y sin RAG mediante el flag `use_rag`. Ante *"Tell me about the US GDP in the past 3 years"*, estas son las respuestas **reales** del modelo.

### ❌ Sin RAG (`use_rag=False`)

El modelo responde de memoria. Suena impecable. Contiene esto:

> *"2023 [...] This was widely hailed as the **first confirmed 'soft landing' in U.S. history in nearly 50 years** [...] **Growth averaged around $1.89 billion per month.**"*
>
> *"Entering 2024, the economy remains unusually strong compared to historical norms, **often referred to as a 'Growth Oblivious' era**."*

Desarmemos las tres afirmaciones:

| Afirmación | Qué pasa con ella |
|---|---|
| **"Growth averaged around $1.89 billion per month"** | El crecimiento del PIB **no se mide en dólares por mes**. Y aunque se midiera, 1.890 millones mensuales sobre una economía de ~27 **billones** de dólares es una cifra sin sentido. Es un número con forma de dato |
| **"a 'Growth Oblivious' era"** | **Término inventado.** No existe en la literatura económica. Suena a jerga técnica precisamente porque el modelo generó lo que *parecía* jerga técnica |
| **"first confirmed soft landing in nearly 50 years"** | Internamente contradictorio (*"primera"* y *"en 50 años"* a la vez) y presentado como hecho establecido |

Y todo esto sobre **2024**, un año que el modelo **no vio jamás** — su cutoff es diciembre de 2023.

### ✅ Con RAG (`use_rag=True`)

El modelo recibe noticias reales de 2024 recuperadas del dataset. Abre así:

> *"**Based on general economic knowledge up to 2023 and the specific 2024 news articles provided**, here is an overview of the U.S. Gross Domestic Product (GDP) performance over the past three years..."*

Y entrega cifras por año (2021: ~5,7 % · 2022: ~1,8 % · 2023: ~2,3 %) sin inventar unidades ni acuñar términos.

> [!note] 🔬 Lo que este experimento realmente demuestra (y lo que no)
> Conviene ser preciso, porque es fácil sobrevender RAG. La versión con RAG **no es más larga ni más exhaustiva sobre 2024** — de hecho se concentra en 2021-2023. Lo que cambió es otra cosa, y es más valiosa:
>
> 1. **Declara sus fuentes.** Distingue explícitamente lo que sabe de entrenamiento y lo que viene del contexto entregado.
> 2. **Dejó de inventar.** Desaparecen las unidades absurdas y los términos acuñados.
> 3. **Se volvió calibrada.** Sin RAG el modelo era *más confiado* justamente donde *menos sabía*.
>
> **RAG no hizo al modelo más inteligente: lo hizo más honesto.** Y nota la asimetría peligrosa: la respuesta **sin** RAG era más larga, más asertiva y más agradable de leer. Si solo miras la superficie, la respuesta mala gana.

> [!abstract] 👔 La lección ejecutiva
> El mismo modelo, con la misma pregunta, produce una respuesta **confiable o peligrosa** según tenga o no acceso a la información correcta en el momento de responder. En un contexto empresarial esa diferencia separa un asistente que puedes desplegar de uno que es un riesgo legal.
>
> Y el corolario incómodo para quien aprueba presupuestos: **una alucinación no se ve como un error.** Se ve como una respuesta bien redactada. No la va a detectar el usuario final — solo la detecta un sistema diseñado para citar fuentes.

---

## 7. ⚖️ RAG vs. las alternativas: el marco de decisión

Audiencia: 🔧 🧭 👔

Existen otras formas de "meterle conocimiento" a un LLM. Esta tabla ubica RAG frente a ellas:

| Enfoque | Cómo funciona | Costo | Datos actualizados | Cuándo conviene |
|---|---|---|---|---|
| **RAG** | Recupera contexto y lo inyecta en el prompt | 💲 Bajo | ✅ Sí, en tiempo real | Conocimiento que cambia, necesitas citar fuentes, datos privados |
| **Fine-tuning** | Reentrena parcialmente el modelo con tus datos | 💲💲💲 Alto | ❌ Congelado al reentrenar | Cambiar el *estilo* o *formato* de respuesta, tareas muy específicas |
| **Prompt sin RAG** | Todo depende de la memoria del modelo | 💲 Nulo | ❌ Solo hasta el cutoff | Conocimiento general estable (definiciones, historia) |
| **Contexto completo (long context)** | Metes TODO el documento en el prompt | 💲💲 Medio-alto | ✅ Sí | Documentos cortos; se vuelve caro/inviable con corpus grandes |

> [!tip] 💡 Analogía del estudiante (RAG vs. Fine-tuning)
> **Fine-tuning** es mandar al estudiante a un curso intensivo para que *interiorice* una materia nueva: caro, lento, y si el temario cambia hay que repetir el curso. **RAG** es dejar que el estudiante rinda el examen **con el libro abierto**: no cambia lo que el estudiante es, pero puede consultar la página exacta justo cuando la necesita, y si actualizas el libro, la respuesta se actualiza sola.

### 7.1 Señales de que elegiste mal

Audiencia: 🧭 👔

Más útil que la tabla es saber **reconocer el error una vez cometido**:

| Síntoma que observas | Probablemente elegiste… | Deberías haber usado… |
|---|---|---|
| El modelo responde con datos correctos pero en un tono/formato que no es el tuyo | **RAG** para un problema de estilo | Fine-tuning |
| El modelo tiene el estilo perfecto pero inventa datos de producto | **Fine-tuning** para un problema de conocimiento | RAG |
| Los costos por consulta se dispararon y la latencia es alta | **Long context** metiendo todo el corpus | RAG con `top_k` acotado |
| Cada actualización de contenido exige un proyecto de semanas | **Fine-tuning** sobre datos que cambian | RAG |
| El sistema no puede decir de dónde sacó una afirmación | **Fine-tuning** o prompt a secas | RAG (es el único que da trazabilidad) |

> [!important] No son excluyentes
> La pregunta *"¿RAG o fine-tuning?"* está mal planteada con frecuencia. **Resuelven problemas distintos y se combinan bien:** fine-tuning para que el modelo adopte el tono, el formato y el vocabulario de tu dominio; RAG para que tenga los hechos correctos delante. Un sistema maduro suele usar los dos. La comparación profunda está en [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]].

---

## 8. 🧭 Cuándo usar RAG (y cuándo no)

Audiencia: 🧭 👔

> [!tip] 🧭 Úsalo cuando…
> - Tu conocimiento **cambia con frecuencia** (noticias, precios, inventario, regulaciones).
> - Necesitas que las respuestas **citen la fuente** (compliance, soporte, legal, salud).
> - Trabajas con **datos privados** que el modelo nunca vio y que no quieres exponer a un reentrenamiento.
> - El corpus es **grande** y no cabe entero en el prompt.

> [!warning] 🧭 Piénsalo dos veces cuando…
> - La tarea es puro **estilo o formato** (ahí conviene fine-tuning, no RAG).
> - El conocimiento es **general y estable** (definiciones, historia): el modelo ya lo sabe.
> - Tu corpus es **minúsculo** y cabe completo en el contexto (quizás no necesitas la maquinaria de retrieval todavía).

> [!example] 📊 Caso de negocio — Asistente de catálogo en retail
> **Problema:** una cadena de retail con 40.000 SKUs quiere un asistente que responda preguntas de clientes y de vendedores en tienda: disponibilidad, especificaciones técnicas, compatibilidad entre productos, políticas de garantía y devolución. El catálogo cambia **a diario** (precios, stock, altas y bajas de producto) y las políticas cambian por temporada. Un LLM genérico inventa especificaciones que suenan plausibles y cita políticas de devolución que no son las de la empresa.
>
> **Por qué las alternativas no sirven:**
> - *Fine-tuning*: quedaría desactualizado al día siguiente del entrenamiento, y reentrenar a diario es inviable en costo y tiempo.
> - *Long context*: 40.000 fichas de producto no caben en ninguna context window, y aunque cupieran, el costo por consulta sería prohibitivo.
>
> **Técnica aplicada:** RAG sobre la base de productos y el manual de políticas. El retriever entrega las fichas relevantes a la consulta; el LLM redacta la respuesta citando SKU y fecha de vigencia de la política.
>
> **Resultado:** cuando cambia un precio o entra un producto nuevo, se **actualiza la knowledge base** — el mismo movimiento que ya se hace en cualquier base de datos. **No se reentrena nada.** El asistente refleja el cambio apenas se re-indexa. Y como cada respuesta cita el SKU y la política de origen, un supervisor puede auditar cualquier afirmación que llegó a un cliente.

---

## 9. 🚫 Anti-patrones al empezar

Audiencia: 🔧 🧭

Errores frecuentes en los primeros sistemas RAG. Ninguno es evidente antes de cometerlo:

> [!danger] Los cinco tropiezos clásicos
> 1. **Tratar el retrieval como resuelto.** "Ya conecté un vector store, listo." La calidad del sistema **está dominada por la calidad del retrieval**: si el retriever no trae el documento correcto, ningún prompt ni ningún modelo lo van a salvar. Es donde hay que invertir el esfuerzo.
> 2. **No medir.** Sin métricas de retrieval no sabes si un cambio mejoró o empeoró el sistema — y como las respuestas siempre *suenan* bien, la intuición engaña. ([[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]])
> 3. **Subir el `top_k` como reflejo.** "Que le lleguen más documentos, por si acaso." Más contexto **no es mejor contexto**: encarece cada consulta, agrega ruido que distrae al modelo y eventualmente agota la context window.
> 4. **Ignorar los permisos hasta el final.** Si el control de acceso no está en el diseño desde el día uno, terminas con un sistema que le muestra a cualquiera cualquier documento. ([[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]])
> 5. **Indexar documentos completos sin partirlos.** Los embedding models **truncan en silencio** todo lo que exceda su límite: el contenido queda en la knowledge base pero es irrecuperable, y sin error alguno. ([[Guia-Maestra-RAG_06-Chunking|Tomo 06]])

---

## 10. 🔬 El mapa del campo: Naive, Advanced y Modular RAG

Audiencia: 🔧 🧭

> [!note] Complemento de vanguardia — no es material del curso
> Lo que construimos en este tomo tiene nombre propio en la literatura, y ubicarlo ayuda a entender hacia dónde va el resto de la guía. La taxonomía más citada divide el campo en tres paradigmas (Gao et al., 2023):

```
   NAIVE RAG          indexar → recuperar → generar
   ───────────        El pipeline lineal de este tomo. Funciona,
                      y es el punto de partida correcto.
                            │
                            ▼
   ADVANCED RAG       Agrega optimizaciones ANTES y DESPUÉS del retrieval:
   ──────────────     · pre-retrieval:  reescribir la query, hybrid search
                      · post-retrieval: re-ranking, compresión del contexto
                      → Tomos 03, 04, 07 y 12
                            │
                            ▼
   MODULAR RAG        El pipeline deja de ser lineal. Componentes
   ───────────        intercambiables, rutas condicionales, bucles,
                      y un agente que decide qué recuperar y cuándo.
                      → Tomo 09 (agentic RAG)
```

**🧭 Cómo usar este mapa:** no es una escalera que haya que subir entera. **Naive RAG bien hecho supera a Advanced RAG mal hecho**, y la mayoría de los sistemas en producción viven en algún punto del segundo escalón. La progresión correcta es: construye el pipeline lineal, **mídelo**, y agrega complejidad solo donde la medición demuestre que hace falta.

---

## 11. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **RAG** | Darle al modelo acceso de solo lectura a tu información justo antes de que responda |
| **LLM** | El "cerebro" de lenguaje: escribe y razona, pero solo sabe lo que aprendió hasta su fecha de corte |
| **Knowledge cutoff** | La fecha límite del conocimiento del modelo; no sabe nada posterior |
| **Hallucination (alucinación)** | Cuando el modelo inventa una respuesta con tono seguro en vez de admitir que no sabe |
| **Knowledge base** | La colección de documentos confiables que alimenta al sistema |
| **Indexing** | El trabajo previo de preparar y organizar los documentos para poder buscarlos rápido |
| **Retrieval** | La búsqueda que trae los fragmentos de información más relevantes para la pregunta |
| **Embedding** | La forma en que un texto se convierte en números para poder compararlo por significado (Tomo 04) |
| **Chunking** | Partir documentos largos en fragmentos manejables antes de indexarlos (Tomo 06) |
| **Prompt aumentado** | La pregunta del usuario + la evidencia recuperada, entregadas juntas al modelo |
| **`top_k`** | Cuántos fragmentos relevantes se recuperan (los "k" mejores) |
| **Grounding** | Que la respuesta se apoye en evidencia entregada y no en la memoria del modelo |
| **Fine-tuning** | Reentrenar el modelo con tus datos; caro y se congela en el tiempo (alternativa a RAG) |

---

## 12. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Entiendo los tres problemas del LLM sin RAG: cutoff, alucinaciones, sin datos privados.
- [ ] Sé argumentar por qué esperar al próximo modelo no resuelve el problema.
- [ ] Sé explicar RAG como "recuperar contexto e inyectarlo en el prompt antes de generar".
- [ ] **Distingo la fase de indexing (offline) de la de retrieval (online)** y sé qué se decide en cada una.
- [ ] Entiendo por qué cambiar el embedding model o el chunking obliga a re-indexar todo.
- [ ] Reconozco las 4 etapas de consulta: retrieval → formatting → augmentation → generation.
- [ ] Entiendo por qué el formateo de los documentos afecta la calidad y la trazabilidad.
- [ ] Puedo señalar alucinaciones concretas en la respuesta sin RAG del experimento.
- [ ] Tengo claro que **la respuesta sin RAG era más larga y más confiada** — y por qué eso es peligroso.
- [ ] Distingo RAG de fine-tuning, sé cuándo conviene cada uno y **por qué no son excluyentes**.
- [ ] Puedo nombrar al menos tres anti-patrones de los primeros sistemas RAG.
- [ ] Puedo argumentar el valor de negocio de RAG ante un stakeholder no técnico.

---

## 🔗 Conexiones

- Siguiente tomo → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02 · Fundamentos: LLMs y el pipeline RAG]] (por qué el LLM alucina, qué hace el retriever por dentro, y el pipeline completo)
- La búsqueda por palabras exactas → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03 · Keyword search]] (y el control de acceso vía metadata)
- La búsqueda por significado → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search y embeddings]]
- El motor de la fase 1 → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]]
- Cómo partir los documentos → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]]
- Medir si tu RAG funciona, y RAG vs. fine-tuning en detalle → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Hallucinations y evaluación]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 1: RAG Overview (Coursera). Assignment C1M1; todas las salidas del experimento de la sección 6 provienen de ese notebook.

**Fuentes externas (complemento con bibliografía verificable):**
- Lewis, P. et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS. — Paper fundacional de RAG.
- Gao, Y. et al. (2023). *Retrieval-Augmented Generation for Large Language Models: A Survey*. arXiv:2312.10997. — Origen de la taxonomía Naive / Advanced / Modular RAG de la sección 10.
- Meta AI (2024). *Llama 3.1* — documentación del modelo `llama-3-1-8b-instruct-turbo` usado en el assignment.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02 · Fundamentos: LLMs y el pipeline RAG]]**, donde abrimos las dos cajas que aquí quedaron cerradas: **por qué** un LLM alucina (y por qué no es un bug), **cómo** decide el retriever qué es relevante, y el pipeline completo en código incluyendo las etapas de augmentation y generation.
