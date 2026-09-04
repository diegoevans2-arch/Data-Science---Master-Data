---
title: "Tomo 09 — Hallucinations, evaluación y agentic RAG"
tags: [rag, evaluacion, metricas, precision, recall, map, mrr, ragas, faithfulness, hallucinations, agentic-rag, fine-tuning]
audiencias: [tecnico, puente, ejecutivo]
tomo: 09
version: 1.1
updated: 2026-08-28
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 09 — Hallucinations, evaluación y agentic RAG

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación: transformers, sampling y prompt engineering]] · Siguiente → [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · RAG en producción]]

---

> [!info] ¿Por qué importa esta sección?
> Los ocho tomos anteriores llenaron el sistema de **perillas**: `top_k`, `alpha`, tamaño de chunk, overlap, `ef`, over-fetch, `temperature`, `top_p`, el system prompt, el modelo. Y hasta ahora **no tenemos forma de saber si moverlas mejora o empeora las cosas.**
>
> Este es el tomo que cierra ese hueco, y por eso es el más largo de la guía. Trae las métricas del **retriever** (que quedaron pendientes de ruteo desde el Módulo 2) y las del **generador**, más la detección de **hallucinations**, los flujos **agentic** y la comparación honesta entre **RAG y fine-tuning**.
>
> Si un tomo de esta guía cambia cómo trabajas, probablemente sea este: **sin medición, todo lo anterior es fe.**

> [!abstract] 👔 Impacto ejecutivo
> Este es el tomo que convierte un piloto de IA en un sistema **auditable**. La diferencia entre "el asistente funciona bien" (una opinión) y "el asistente recupera el documento correcto el 94 % de las veces y el 97 % de sus afirmaciones están respaldadas por la fuente" (un hecho).
> - **Decisiones que habilita:** aprobar o rechazar un cambio con evidencia en vez de intuición; poner un SLA de calidad sobre un sistema generativo; decidir si un modelo nuevo mejora o solo cambia; exigir trazabilidad antes de un despliegue en un área regulada.
> - **Costo o riesgo de hacerlo mal:** sin métricas, **cada "mejora" es una apuesta** y las regresiones pasan inadvertidas hasta que las reporta un usuario. Y sin grounding verificado, el sistema puede afirmar con total fluidez algo que no existe — el fallo que no se ve como error.
> - **Pregunta ejecutiva que responde:** *"¿cómo sé que esto funciona, y cómo sabré que sigue funcionando después del próximo cambio?"*

---

## 1. 🎯 Lo primero: separar responsabilidades

Audiencia: 🔧 🧭 👔

**🔧 El punto de partida más importante del tomo**, y el que evita semanas perdidas:

```
   ┌───────────────────────────────────────────────────────────────┐
   │  TRABAJO DEL RETRIEVER                                        │
   │  encontrar la información relevante en la knowledge base      │
   ├───────────────────────────────────────────────────────────────┤
   │  TRABAJO DEL LLM                                              │
   │  usar esa información para construir una respuesta de calidad │
   └───────────────────────────────────────────────────────────────┘
```

> [!important] 🎯 La consecuencia práctica
> *"Si el problema en última instancia está en tu retriever, **no querrás perder tiempo reescribiendo tu system prompt**."*
>
> Antes de tocar nada, la pregunta de diagnóstico es siempre la misma: **¿el documento correcto estaba entre los recuperados?** Si no estaba, ninguna mejora al prompt te va a salvar. Si estaba y la respuesta igual salió mal, el problema es de generación. Es la propiedad de **diagnosticabilidad** que el [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]] listó como la quinta ventaja de RAG — y aquí es donde se cobra.

**Y qué se le pide exactamente al LLM**, asumiendo que el retriever hizo su parte (traer información mayormente relevante, con algún documento irrelevante colado):

1. Responder al prompt del usuario.
2. **Incorporar** la información relevante en su respuesta.
3. **Citarla** apropiadamente.
4. **Resistirse a distraerse** con la información irrelevante que se recuperó.

> [!warning] ⚠️ Por qué las métricas del LLM son distintas a todas las anteriores
> Fíjate en que esas cuatro conductas son **subjetivas**. *"¿Cómo puedes decir cuantitativamente que una respuesta hace un buen trabajo respondiendo la pregunta original, o ignorando la información irrelevante?"*
>
> De ahí sale el hecho central de la sección 4: **la mayoría de las métricas específicas del LLM se apoyan en usar otros LLMs** para evaluar la calidad. No es pereza metodológica — es que incorporar LLMs al proceso de evaluación *"permite cierto grado de flexibilidad o subjetividad de forma escalable"*.

---

## 2. 📏 Medir el retriever

Audiencia: 🔧 🧭

> [!note] Excepción de ruteo declarada en el MOC
> Esta sección proviene del **Módulo 2** del curso (lección de *retriever evaluation* y su Ungraded Lab 2), no del Módulo 4. Se consolidó aquí a propósito: **partir el tratamiento de métricas entre dos tomos distantes haría que ninguno de los dos sirviera** como referencia. Aquí están todas juntas, del retriever y del generador.

**🔧 Los tres ingredientes** comunes a casi toda métrica de calidad de retrieval:

```
   ① EL PROMPT
      (los retrievers rinden bien en unos y mal en otros)
   ② LA LISTA RANKEADA que el retriever devolvió para ese prompt
   ③ EL GROUND TRUTH: todos los documentos relevantes que existen
      en la knowledge base y que el retriever DEBERÍA haber devuelto
```

> [!important] La frase que resume el costo de todo esto
> *"En otras palabras: **si quieres calificar a tu retriever, necesitas conocer las respuestas correctas.**"* Ese tercer ingrediente es el que cuesta — volvemos a él en 2.6.

### 2.1 Precision y recall

**🔧 Definiciones:**

```
                 documentos relevantes recuperados
   PRECISION = ──────────────────────────────────────
                 TOTAL de documentos recuperados

                 documentos relevantes recuperados
   RECALL    = ──────────────────────────────────────────────────
                 TOTAL de documentos relevantes en la knowledge base
```

**El ejemplo trabajado del curso.** Sabes que la knowledge base tiene **10 documentos relevantes** para un prompt, porque los marcaste a mano:

```
   ══ Corrida 1 ══
   Recuperas 12 documentos, de los cuales 8 son relevantes

   precision = 8/12 = 66 %
   recall    = 8/10 = 80 %

   ══ Corrida 2 ══ (ajustas la configuración del retriever)
   Recuperas 15 documentos, de los cuales 9 son relevantes

   precision = 9/15 = 60 %   ↓ bajó
   recall    = 9/10 = 90 %   ↑ subió
```

*"Tu retriever devolvió tres documentos más que la vez anterior, pero **solo un documento relevante más**."* Cambiaste un poco de precision por un poco más de recall.

> [!tip] 💡 Qué mide cada una, en una línea
> - **Precision** castiga al retriever por devolver documentos **irrelevantes** → puede pensarse como **cuán confiables** son los resultados.
> - **Recall** castiga al retriever por **dejar fuera** documentos relevantes → mide **cuán exhaustivo** es.
>
> Y la única forma de tener ambas perfectas: *"rankear los documentos relevantes más alto **y** devolver solo esos documentos"*. En la práctica, siempre hay un intercambio.

### 2.2 @K — por qué las métricas llevan un número

**🔧 El problema:** las métricas de retrieval **dependen de cuántos documentos devuelve** el retriever. Para estandarizar, se hablan en términos de los **top K** documentos.

Con esta lista rankeada (✓ = relevante), y sabiendo que hay **8 documentos relevantes** en la knowledge base:

```
   posición:  1   2   3   4   5   6   7   8   9  10
              ✓   ✗   ✗   ✓   ✗   ✗   ✓   ✓   ✓   ✓

   precision@5  = 2/5  = 40 %      ← solo 2 de los top 5 son relevantes
   precision@10 = 6/10 = 60 %      ← 6 de los top 10
   recall@10    = 6/8  = 75 %      ← de los 8 que existían, encontró 6
```

**Qué K elegir:** *"top 5, top 2 o top 1 pueden usarse cuando importan estándares más estrictos. A menudo, sin embargo, se usa un rango algo más generoso entre **top 5 y top 15**."*

### 2.3 MAP@K — premiar el buen orden

**🔧 Definición:** *Mean Average Precision at K* evalúa la **precisión promedio de los documentos relevantes** dentro de los primeros K recuperados. Da una visión más holística porque **le importa dónde caen los aciertos**, no solo cuántos hay.

El cálculo del curso, paso a paso. **Average precision at 6** sobre esta lista:

```
   posición   relevante?   precision@posición
   ─────────────────────────────────────────────
      1           ✓         1/1 = 1.0     ← se suma
      2           ✗         —
      3           ✗         —
      4           ✓         2/4 = 0.5     ← se suma
      5           ✓         3/5 = 0.6     ← se suma
      6           ✗         —

   Se suman SOLO las filas con documento relevante:
      1.0 + 0.5 + 0.6 = 2.1

   Se divide por el nº de relevantes recuperados en el top K (3):
      2.1 / 3 = 0.7          ← average precision
```

Y **MAP** es el promedio de esa *average precision* a través de muchos prompts — *"te dice cuál sería la precisión promedio para un prompt típico que tu retriever recibe"*.

> [!important] 🎯 Por qué MAP es la métrica que castiga el mal orden
> *"MAP premia rankear los documentos relevantes bien arriba. Si un documento irrelevante se cuela en un puesto alto del ranking, **disminuirá la precisión en el rank de cada documento relevante que esté debajo**, bajando el promedio general."*
>
> Es exactamente el problema que el [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]] resolvió con re-ranking: recuperar el documento correcto y ponerlo en el puesto 14 **no es lo mismo** que ponerlo primero. Precision y recall no ven esa diferencia; MAP sí. **Si quieres justificar la inversión en un re-ranker, MAP es la métrica que la muestra.**

### 2.4 MRR — el primer acierto

**🔧 Definición:** *Reciprocal Rank* mide la posición del **primer** objeto relevante en la lista devuelta.

```
   primer relevante en posición 2  →  RR = 1/2 = 0.5
   primer relevante en posición 4  →  RR = 1/4 = 0.25

   Cuanto más abajo aparece el primer relevante, peor el reciprocal rank.
```

Promediado sobre múltiples prompts da el **Mean Reciprocal Rank**. El ejemplo del curso: cuatro búsquedas donde el primer documento relevante apareció en las posiciones **1, 3, 6 y 2**:

```
   1/1  +  1/3  +  1/6  +  1/2   =   1 + 0.333 + 0.167 + 0.5  =  2.0

   MRR = 2.0 / 4 = 0.5
```

**Qué refleja:** *"cuán pronto, en promedio, puedes encontrar un ítem relevante en el ranking del retriever"*. Y enfatiza la importancia de **incluir al menos un documento relevante lo más arriba posible**.

> [!tip] 🧭 Cuándo MRR es la métrica correcta
> Cuando al usuario le basta **una** buena respuesta. Un buscador de documentación, un asistente de soporte, un "¿cuál es la política de X?" — si el primer resultado sirve, el resto es irrelevante. En cambio, si el usuario necesita **todo** lo que existe sobre un tema (revisión legal, auditoría), MRR no es la métrica: es recall.

### 2.5 Cómo usarlas juntas

Audiencia: 🔧 🧭

El curso da una jerarquía explícita y muy útil:

| Métrica | Rol | Cuándo mirarla |
|---|---|---|
| **Recall / Recall@K** | *"La más fundamental y la más citada"* — captura el objetivo más básico del retriever: **encontrar los documentos relevantes** | Siempre. Es la base |
| **Precision** | ¿Está incluyendo muchos documentos irrelevantes? | Cuando el ruido llena el context window o distrae al LLM |
| **MAP** | ¿Los está **rankeando** eficazmente? | Al evaluar re-ranking o cambios de scoring |
| **MRR** | Más especializada: ¿cómo rinde en el **extremo superior** del ranking? | Cuando basta un buen resultado |

```
   Empieza por RECALL@K
        │
        ├─ ¿bajo? → el retriever no encuentra. Revisa chunking (T06),
        │            hybrid search (T04), el índice (T05)
        │
        └─ ¿alto pero las respuestas son malas?
                 │
                 ├─ mira PRECISION → ¿demasiado ruido?
                 ├─ mira MAP       → ¿está mal ordenado? → re-ranking (T07)
                 └─ ambos bien     → el problema es del LLM → sección 4
```

### 2.6 El costo real: el ground truth

> [!warning] ⚠️ La única desventaja de estas métricas, según el curso
> *"Si hay una desventaja en estas métricas, es simplemente que **todas dependen de tener documentos relevantes de ground truth** para una colección de prompts de muestra. Esto puede ser un proceso manual y que consume mucho tiempo de compilar."*
>
> Y el contrapeso: *"El resultado final, sin embargo, es un sistema que **puedes monitorear tanto durante el desarrollo como una vez en producción**."*

**🧭 Cómo abordarlo sin morir en el intento:** no necesitas etiquetar todo el corpus. Necesitas un **set de queries representativas** (decenas, no miles) con sus documentos relevantes marcados. Prioriza: las queries más frecuentes, las que más importan al negocio, y las que **ya fallaron** — esas últimas son las más valiosas, porque son tu suite de regresión.

### 2.7 💻 El lab: el trade-off precision-recall, medido

Audiencia: 🔧

> [!note] Contexto del lab (Ungraded Lab 2 del Módulo 2)
> **Dataset:** *20 Newsgroups*, **11.314 documentos** en 20 categorías. **Embeddings:** `BAAI/bge-base-en-v1.5`, precomputados. **Ground truth:** la categoría del documento — se considera relevante si su categoría coincide con la esperada para la query. **10 queries** de prueba.

El lab corre las mismas 10 queries con `K = 5, 20, 50`. Los resultados muestran el trade-off en números:

| K | Precision (típica) | Recall (típico) |
|---:|---|---|
| **5** | **1.00** en 8 de 10 queries | ~0.01 |
| **20** | baja en varias (0.65–1.00) | ~0.03 |
| **50** | sigue bajando (0.60–1.00) | 0.05–0.08 |

> [!important] 🎯 Tres lecturas de esa tabla, y la tercera es la que importa
> **(1) El trade-off es real y monótono.** A medida que K crece, recuperas más de lo relevante que existe (recall ↑) al costo de incluir irrelevantes (precision ↓).
>
> **(2) El recall se ve "terrible" y no lo es.** Con `K=5` el recall es ~1 %. Suena catastrófico hasta que haces la cuenta: **cada categoría tiene entre 500 y 600 documentos**, así que recuperar 5 nunca puede dar más de ~1 % de recall. El número está limitado por la aritmética, no por el retriever.
>
> **(3) Y de ahí sale la conclusión que cambia cómo eliges K en un RAG:**
>
> > *"Para sistemas RAG, K = 5 a K = 20 suele ser óptimo. Estos valores dan alta precision manteniendo el tamaño del contexto manejable para el LLM. **Aunque el recall sea bajo, el objetivo es encontrar los documentos MÁS relevantes, no TODOS los relevantes.**"*
>
> Esa distinción es el corazón del asunto. En un buscador tradicional de tipo auditoría quieres **todos**. En un RAG quieres **los mejores pocos**, porque el LLM tiene un context window y porque cada token cuesta. **Un recall bajo en un RAG puede ser exactamente el comportamiento correcto** — lo que no puede ser bajo es la precision.

Y un dato con textura: la query *"historical influence of politics on society"* fue **consistentemente la peor** (precision 0.40–0.52 en todos los K). El lab lo interpreta bien: la query es **semánticamente ambigua**, o la categoría `talk.politics.misc` es difícil de distinguir de sus vecinas. Es un buen recordatorio de que **el promedio esconde las queries problemáticas** — y esas son las que hay que mirar.

---

## 3. 🚨 Hallucinations

Audiencia: 🔧 🧭 👔

**🔧 El planteamiento del curso:** *"Las hallucinations son una preocupación constante al trabajar con LLMs, y **incluso un sistema RAG bien diseñado puede alucinar**. Detectarlas, reducirlas y asegurar que el LLM cite fuentes con precisión son, por tanto, las partes más importantes de construir tu pipeline RAG."*

### 3.1 El caso del descuento estudiantil

> [!danger] 🚨 El ejemplo del curso, que vale contar completo
> Tienes tu primer RAG funcionando: un chatbot de servicio al cliente de una tienda online. Un usuario escribe y pregunta si ofrecen **descuento para estudiantes**.
>
> ```
>    El retriever encuentra información sobre descuentos para
>    JUBILADOS y para CLIENTES NUEVOS — ambos del 10 %.
>
>    El system prompt le pide al LLM ser ÚTIL con los clientes.
>                            │
>                            ▼
>    "Absolutely, you can get 10 % off with a valid student ID —
>     the same great discount we offer our seniors and new customers."
>                            │
>                            ▼
>    El usuario, encantado, sigue comprando esperando su descuento.
>
>    El único problema: ESE DESCUENTO NO EXISTE. El LLM lo inventó.
> ```
>
> Fíjate en la mecánica del fallo: **cada pieza del sistema hizo algo razonable.** El retriever trajo documentos genuinamente relacionados. El system prompt pedía ser útil. El LLM combinó ambas cosas de la forma más plausible posible. **Nada falló** — y el resultado es falso.

### 3.2 Por qué alucinan, y por qué es tan grave

**🔧 El recordatorio:** un modelo de lenguaje está diseñado para producir **secuencias de palabras probables**, con algo de aleatorización para dar variedad. Las secuencias probables **suelen** ser factualmente correctas, pero no siempre. *"Los modelos de lenguaje no están diseñados para diferenciar entre verdadero y falso, solo entre probable e improbable."* (Es la tesis del [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]], y aquí se cobra.)

**Las tres razones por las que son problemáticas:**

```
   ① Lo obvio: no quieres que tu modelo dé información inexacta.

   ② Casi por definición, las hallucinations SUENAN PLAUSIBLES,
      así que son más difíciles de detectar que un sinsentido total.

   ③ Con el tiempo, alucinaciones ocasionales hacen que tus usuarios
      PIERDAN LA CONFIANZA en el sistema — incluso si la mayoría del
      contenido generado es correcto.
```

> [!abstract] 👔 La tercera es la que mata el proyecto
> Las razones 1 y 2 son problemas técnicos. La 3 es un problema de **adopción**: un asistente en el que nadie confía deja de usarse, y entonces no importa cuán bueno sea. **La confianza se pierde de a una alucinación y se recupera de a muchas respuestas verificables** — de ahí que las citas (3.5) no sean un adorno.

### 3.3 Las hallucinations vienen en grados

**🔧 Y esto es importante para diseñar la evaluación.** Volviendo al descuento:

| Tipo | Ejemplo con el descuento |
|---|---|
| **Error de detalle** | Describe correctamente el descuento real para jubilados y su forma de reclamarlo, pero **dice 5 % en vez de 10 %** |
| **Negación de un hecho** | Afirma que **no existe** descuento para jubilados, cuando sí existe |
| **Invención completa** | Inventa un descuento que la empresa **no ofrece** |

> [!important] 🎯 La consecuencia metodológica
> *"Esto significa que necesitarás **evaluar el texto que tu LLM genera en muchos niveles**"* si quieres tener confianza en su exactitud. Una métrica binaria de "¿alucinó o no?" no captura la diferencia entre equivocarse en un porcentaje y fabricar un producto entero — y esas dos cosas tienen consecuencias de negocio muy distintas.

### 3.4 La verdad incómoda

> [!warning] ⚠️ El curso es honesto y conviene citarlo tal cual
> *"Aquí viene la verdad cruda y dura. **No hay una solución perfecta para las hallucinations**, o al menos no actualmente. Por suerte, sin embargo, **RAG es uno de los mejores enfoques disponibles**, y hay formas de refinar los sistemas RAG para disminuir aún más su frecuencia."*
>
> Es una frase que conviene tener a mano en reuniones: **no se puede prometer cero alucinaciones.** Lo que sí se puede prometer es un sistema que las minimiza, las hace detectables y las hace verificables.

### 3.5 Las herramientas, en orden de utilidad

**Sin knowledge base — `self-consistency checking`:**

**🔧 Definición:** generar repetidamente completions para el **mismo prompt** y verificar si la información factual que contienen es **consistente entre sí**. La idea: *"si el modelo está alucinando, lo hará de forma inconsistente, y las diferencias factuales entre completions serán detectables"*.

> [!note] Y el veredicto del propio curso
> *"En la práctica, sin embargo, este método puede ser **costoso y poco confiable**."* Cuesta N veces más (N llamadas por respuesta) y una alucinación consistente pasa el filtro. Está aquí por completitud: si tienes knowledge base, hay algo mejor.

**Con knowledge base — grounding y citas:**

```
   ① GROUNDING VÍA SYSTEM PROMPT
      Modificar el system prompt para decir que el LLM
      SOLO puede hacer afirmaciones factuales basadas en
      la información recuperada.
              │
              ▼
   ② EXIGIR CITAS
      Promptear al modelo para que cite sus fuentes al final
      de cada oración o párrafo. Doble beneficio:
        · aumenta la probabilidad de que fundamente la respuesta
        · permite que un lector humano VERIFIQUE las afirmaciones
              │
              ▼
   ⚠️ EL RIESGO: que el LLM ALUCINE LAS CITAS.
      Algunos modelos fine-tuneados para citar generan citas
      válidas de forma más confiable, pero si quieres más
      confianza necesitas un sistema EXTERNO. → 3.6
```

### 3.6 ContextCite: verificar el grounding por fuera

**🔧 Definición técnica:** **ContextCite** es un sistema que **puntúa cuán bien fundamentada está una respuesta** en un conjunto de materiales fuente. Su mecánica:

```
   Respuesta del LLM, procesada ORACIÓN POR ORACIÓN
              │
              ▼
   Cada oración se ATRIBUYE a uno de los documentos de contexto
   que fueron recuperados y entregados al LLM
              │
              ▼
   Se genera un TAG por oración indicando su documento fuente
              │
              ├──► si la afirmación NO tiene material que la respalde
              │    → se etiqueta como "NO SOURCE"   ← 🎯 la señal clave
              │
              └──► algunas implementaciones dan además un score de
                   similitud entre la oración y el documento identificado
```

**Y tiene dos usos distintos**, que conviene no confundir:

| Uso | Para qué |
|---|---|
| **En producción** | Generar las citas de la respuesta final del LLM |
| **En evaluación** | Medir **con qué frecuencia** el LLM fundamenta sus respuestas en los documentos recuperados |

> [!tip] 💡 Por qué el tag "no source" es lo más valioso de todo el tomo
> Una alucinación es, por definición, una afirmación **sin respaldo en el contexto**. ContextCite convierte eso en una **señal mecánica**: oración por oración, o hay fuente o no la hay. Deja de ser un juicio subjetivo y pasa a ser algo que puedes **contar**.

### 3.7 ALCE: el benchmark de citación

**🔧 Definición:** un esfuerzo reciente para medir **cuán bien un sistema referencia y cita fuentes** al generar respuestas. Provee knowledge bases pre-armadas y preguntas de muestra; corres tu sistema RAG sobre esos prompts y le pides a ALCE que evalúe la respuesta generada. Da scores en **tres métricas**:

| Métrica | Qué mide |
|---|---|
| **Fluency** | ¿Qué tan claro es el texto final? |
| **Correctness** | ¿Qué tan factualmente correcto? |
| **Citation quality** | ¿Qué tan bien las citas provistas se alinean con las fuentes correctas a citar? |

> [!note] Qué esperar de un benchmark así, y qué no
> *"Estos benchmarks **no controlan las hallucinations en tu sistema de producción**, pero sí te dan cierta idea de cuán bien tu sistema las está evitando y citando fuentes."* Es una medición de laboratorio: útil para comparar configuraciones, insuficiente como control operativo.

### 3.8 La receta completa contra hallucinations

> [!important] 🎯 El orden que recomienda el curso
> 1. **Construir un sistema RAG** ya es *"el paso individual más efectivo para minimizar las hallucinations"*. Si estás leyendo esta guía, ya lo hiciste.
> 2. **Enfocar la energía en asegurar el grounding** refinando el **system prompt**.
> 3. **Testear con benchmarks enfocados en hallucinations** para asegurar respuestas fundamentadas y bien citadas.
>
> Y un cuarto punto que el curso menciona en 3.5 y merece destacarse: **incluir en tu golden set preguntas SIN respuesta en el corpus.** Es la prueba que casi nadie hace y la que más revela: un sistema honesto responde *"no está en los documentos"*; uno que alucina se inventa algo. Si tu set de evaluación solo tiene preguntas respondibles, **nunca vas a medir esa conducta**.

---

## 4. 📊 Medir el generador: RAGAS

Audiencia: 🔧 🧭

**🔧 El contexto:** como las conductas del LLM son subjetivas (sección 1), las métricas específicas del LLM usan **otros LLMs** para evaluar. *"Una buena fuente de estas métricas específicas de RAG es la librería open-source **Ragas**."*

### 4.1 Response relevancy

**🔧 Definición:** mide si una respuesta es **realmente relevante** al prompt del usuario — *"independientemente de si es factualmente correcta"*. Su mecánica es ingeniosa: **funciona al revés**.

```
   ① La respuesta generada por tu sistema RAG se le da a un LLM NUEVO,
      que genera VARIOS PROMPTS DE MUESTRA que cree que podrían haber
      llevado a esa respuesta.
              │
              ▼
   ② Tanto el prompt ORIGINAL del usuario como esos prompts de muestra
      se embeben a vectores semánticos.
              │
              ▼
   ③ Se calcula la COSINE SIMILARITY entre el prompt real y cada
      prompt de muestra.
              │
              ▼
   ④ Se PROMEDIAN esos scores → response relevancy
```

> [!tip] 💡 Analogía
> Es como verificar una respuesta de examen **tapando la pregunta**: le muestras solo la respuesta a alguien y le pides que adivine qué se preguntó. Si adivina algo muy parecido a la pregunta real, la respuesta estaba bien enfocada. Si adivina algo distinto, el alumno respondió otra cosa.

**Su límite, explícito en el curso:** *"esta métrica no asegura necesariamente que la respuesta esté proveyendo información factual, pero sí verifica que puedas razonablemente trabajar hacia atrás desde la respuesta que el LLM dio hasta el prompt que originalmente recibió"*. Mide **pertinencia**, no **verdad**.

### 4.2 Faithfulness — la métrica central de un RAG

**🔧 Definición:** mide si el LLM **realmente está usando** la información recuperada.

```
   ① Un LLM identifica TODAS LAS AFIRMACIONES FACTUALES
      hechas dentro de la respuesta.
              │
              ▼
   ② Más llamadas al LLM determinan CUÁNTAS de esas afirmaciones
      están respaldadas por alguna de las piezas de información
      recuperadas de la knowledge base.
              │
              ▼
   ③  faithfulness = afirmaciones respaldadas / afirmaciones totales
```

> [!important] 🎯 Es la métrica que mide directamente lo que te importa
> Si el sistema debe responder **solo** con base en tus documentos, `faithfulness` **es** ese requisito convertido en número. Una faithfulness de 0.8 significa que **una de cada cinco afirmaciones que tu asistente hace no está respaldada por la fuente** — y esa es exactamente la definición operativa de una alucinación en un RAG.
>
> Nota la simetría con ContextCite (3.6): ambos descomponen la respuesta en afirmaciones y las atribuyen al contexto. **Son dos implementaciones de la misma idea**, y ambas convierten el grounding en algo contable.

**Y otras métricas de RAGAS** siguen enfoques similares para evaluar *"sensibilidad a información irrelevante recuperada de la knowledge base"* y *"capacidad de citar fuentes con precisión"*.

> [!warning] ⚠️ El patrón común, y su costo
> *"Un patrón a través de todas estas métricas es la **dependencia de llamadas a LLMs** en algún punto del proceso de evaluación, y posiblemente también de ejemplos de ground truth de respuestas correctas."*
>
> Traducido: **evaluar tu RAG cuesta llamadas a LLMs.** Una suite de evaluación sobre 100 queries con tres métricas puede costar cientos de llamadas. Es un costo real que hay que presupuestar — y la razón por la que se corre en CI o por lotes, no en cada request.
>
> Y recuerda el sesgo del [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08]]: **no uses como juez un modelo de la misma familia que el evaluado.**

### 4.3 Las métricas de sistema

**🔧 Y la alternativa más barata**, que el curso presenta como complemento:

```
   Si tus usuarios pueden marcar las respuestas con 👍 / 👎
              │
              ▼
   Haces A/B TEST de cambios en tu system prompt
              │
              ▼
   Y observas el impacto en la satisfacción general
```

*"La idea es que **mides el rendimiento de todo el sistema pero aíslas los cambios a la configuración del LLM**, permitiéndote atribuir los cambios de rendimiento general a los cambios en tu LLM."*

> [!tip] 🧭 La combinación que recomienda el curso
> *"Como la calidad de las respuestas de un LLM es algo subjetiva, deberías planear usar **o evals basados en LLM-as-a-judge, o feedback humano**, para evaluar la calidad del LLM. Una combinación de estas técnicas te permitirá evaluar con confianza."*
>
> En la práctica: **LLM-as-a-judge para iterar rápido** (barato, escalable, corre en CI) y **feedback humano para validar** que el juez está bien calibrado. Si las dos señales divergen, confía en la humana y recalibra el juez.

### 4.4 LLM-as-judge en profundidad — evaluación escalable del generador

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Evaluar manualmente cada respuesta de tu RAG es como contratar un inspector para revisar cada plato que sale de la cocina — no escala. LLM-as-judge es poner una cámara inteligente que inspecciona automáticamente y solo llama al inspector humano cuando detecta algo raro.

**🔧 Definición técnica:** usar un LLM (generalmente uno más potente que el del pipeline, o el mismo con un prompt de evaluación distinto) para **puntuar las respuestas del sistema** según criterios definidos en una rúbrica. Es el estándar de facto en 2026 para evaluación continua en producción porque combina escalabilidad con correlación razonable con juicio humano (~80-85% de acuerdo inter-rater, comparable a acuerdo humano-humano).

**🔧 Los tres paradigmas de judging:**

| Paradigma | Mecanismo | Ventaja | Limitación | Cuándo usarlo |
|---|---|---|---|---|
| **Pointwise** | El judge evalúa UNA respuesta y le asigna un score (1-5) según una rúbrica | Simple; independiente; paralelizable | Calibración difícil: ¿qué es un "3"? Depende del judge | Monitoreo continuo; dashboards de calidad |
| **Pairwise** | El judge compara DOS respuestas (A vs B) y declara cuál es mejor | Más estable que pointwise; humanos también prefieren comparar | No da score absoluto; escala mal a N candidatos | A/B testing de prompts; comparar modelos |
| **Listwise** | El judge rankea N respuestas de mejor a peor | Eficiente para evaluar muchos candidatos a la vez | Position bias severo; inconsistente con listas largas | Selección de modelo entre muchos candidatos |

**🔧 Los frameworks de referencia:**

- **G-Eval (Liu et al., 2023):** el judge genera primero un chain-of-thought con los pasos de evaluación según la rúbrica, luego emite un score. La rúbrica es personalizable por caso de uso. Criterios típicos: coherence, relevance, fluency, groundedness, completeness. Resultado: alta correlación con humanos en NLG tasks.

- **MT-Bench / Chatbot Arena (Zheng et al., 2023):** pairwise comparison a gran escala. MT-Bench usa preguntas curadas multi-turn; Arena usa crowdsourcing con Elo rating. El insight: pairwise con GPT-4 como judge tiene >80% agreement con humanos — comparable al acuerdo entre evaluadores humanos.

**🔧 Comparativa de métodos de evaluación:**

| Método | Velocidad | Costo por 1000 evals | Correlación con humanos | Mejor para |
|---|---|---|---|---|
| Human evaluation | Horas/días | $$$$ (anotadores) | Referencia (100%) | Ground truth; calibración; edge cases |
| LLM-as-judge (pointwise) | Segundos | $ (API calls) | ~80-85% | Monitoreo continuo; CI/CD pipeline |
| RAGAS (faithfulness + relevancy) | Segundos | $ (API calls) | ~75-80% (métrica-specific) | Evaluación del pipeline RAG end-to-end |
| Métricas automáticas (BLEU, ROUGE) | Milisegundos | Gratis | ~50-60% | Solo como pre-filtro; NO como métrica final en generación abierta |

**🧭 Cuándo usarlo:** para **toda evaluación recurrente** del generador. La evaluación humana queda reservada para (1) calibrar al judge, (2) edge cases que el judge marca con baja confianza, (3) auditorías periódicas de calidad.

### 4.5 Groundedness verification automatizada

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un periodista puede escribir un artículo brillante — pero si el editor no puede verificar las fuentes de cada afirmación, no se publica. Groundedness verification es ese editor: extrae cada claim de la respuesta y verifica si hay evidencia en los chunks recuperados.

**🔧 El patrón en tres pasos:**

```
 Respuesta del LLM
        │
        ▼
 ┌─── CLAIM EXTRACTION ───┐
 │ "La política permite     │
 │  devolución en 30 días"  │
 │ "El envío es gratis      │
 │  sobre $50.000"          │
 └───────────┬──────────────┘
             │
             ▼
 ┌─── EVIDENCE MATCHING ──────────────────┐
 │ Para cada claim: ¿hay un chunk que lo  │
 │ soporte? (NLI / similarity / LLM-judge)│
 └───────────┬────────────────────────────┘
             │
             ▼
 ┌─── SCORING ────────────────────────────┐
 │ % de claims con soporte = groundedness │
 │ score. Claims sin soporte = potencial  │
 │ hallucination → flag o bloquear        │
 └────────────────────────────────────────┘
```

**🔧 Conexión con §4.2 (Faithfulness de RAGAS):** RAGAS faithfulness ya implementa este patrón — descompone la respuesta en statements y verifica cada uno contra el contexto. La §4.5 lo generaliza como **guardrail de producción**: correr el check ANTES de entregar la respuesta al usuario, no solo como evaluación offline.

**🔧 Herramientas que lo implementan:**

| Herramienta | Approach | Integración |
|---|---|---|
| **RAGAS** (faithfulness metric) | Decompose → NLI per statement | Python; evaluación batch o real-time |
| **DeepEval** | LLM-as-judge con rúbrica de groundedness | Python; integra con pytest |
| **TruLens** | Modular: groundedness, relevance, harmfulness | Dashboard + Python; tracing integrado |
| **Custom prompts** | Un prompt que pregunta "¿esta afirmación está soportada por el contexto?" | Cualquier LLM; máximo control, mínima abstracción |

**🔧 Como guardrail de producción (real-time):**

- Si el groundedness score < umbral (ej: < 0.7), **no entregar la respuesta** — en su lugar, responder con un fallback: *"No tengo suficiente información en mis fuentes para responder esto con confianza."*
- Trade-off: agregar el check añade latencia (1-3s) y costo (una llamada extra al LLM). Solución: correrlo async y hacer streaming con retractación si falla, o aplicarlo solo a respuestas de alto riesgo (dominios regulados).
- Detalle de guardrails completos en [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §5]].

### 4.6 Sesgos y limitaciones de LLM-as-judge

Audiencia: 🔧

> [!warning] ⚠️ No es un evaluador perfecto — conocer sus sesgos es obligatorio antes de confiarle decisiones

| Sesgo | Descripción | Mitigación |
|---|---|---|
| **Verbosity bias** | Prefiere respuestas largas sobre cortas, independientemente de la calidad | Incluir en la rúbrica: "la brevedad es una virtud si responde la pregunta" |
| **Position bias** | En pairwise, tiende a preferir la respuesta que aparece primero (o segunda, según modelo) | Evaluar en ambos órdenes y promediar; descartar si son inconsistentes |
| **Self-preference** | Si el judge es el mismo modelo que generó la respuesta, se puntúa más alto a sí mismo | Usar un modelo diferente como judge; o al menos una familia distinta |
| **Sycophancy** | El judge tiende a estar de acuerdo con la respuesta si se le presenta como "correcta" | Presentar sin marco; pedir que busque errores activamente |
| **Formato bias** | Markdown, bullet points y estructuras bien formateadas reciben scores más altos | Evaluar contenido separado de formato; o normalizar formato antes |

**🔧 El patrón recomendado para producción:**

1. **LLM-as-judge para el 95% del tráfico** — monitoreo continuo con alertas.
2. **Evaluación humana mensual sobre una muestra** (~100-200 respuestas) — para calibrar el judge.
3. Si la correlación judge-humano cae por debajo de 0.75: **recalibrar** la rúbrica o cambiar de modelo judge.
4. **Ensemble de judges** para decisiones críticas: dos modelos distintos evalúan; si discrepan, escala a humano.

**📚 Referencias de las secciones 4.4–4.6:**

- (Liu et al., 2023) — *G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment*. arXiv 2303.16634.
- (Zheng et al., 2023) — *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*. NeurIPS 2023.

---

## 5. 🤖 Agentic RAG

Audiencia: 🔧 🧭 👔

**🔧 Definición:** un **agentic workflow** significa usar **varios LLMs** a lo largo de tu sistema RAG, cada uno responsable de **un solo paso** del proceso general.

> [!note] Ya lo has visto sin que se llamara así
> *"Ya has visto esta idea con LLMs usados para tareas como query expansion, prompt rewriting o generación de citas."* El query rewriting del [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]] **ya era** un componente agentic. Esta sección le pone nombre al patrón y lo generaliza.

**Los dos cambios respecto al uso normal de un LLM:**

```
   USO NORMAL:  prompt ──► LLM ──► respuesta

   AGENTIC:
     ① Las tareas se tratan como una SERIE DE PASOS Y DECISIONES,
        cada uno completable por una llamada a un LLM DISTINTO.

     ② Los LLMs reciben acceso a un abanico más amplio de HERRAMIENTAS:
        un intérprete de código, un navegador web, o —en el caso de
        RAG— una vector database.
```

### 5.1 El flujo del curso

```
   Usuario envía prompt
        │
        ▼
   ┌──────────────────────┐
   │  ROUTER LLM (chico)  │  ¿este prompt requiere consultar la
   │  salida: sí / no     │  vector database? Está tuneado solo
   └──────┬───────────┬───┘  para esta tarea.
          │NO         │SÍ
          │           ▼
          │    ┌─────────────────┐
          │    │ VECTOR DATABASE │
          │    └────────┬────────┘
          │             ▼
          │    ┌──────────────────────┐
          │    │  EVALUATOR LLM       │  ¿los documentos recuperados
          │    │  ¿alcanza?           │  son SUFICIENTES para responder?
          │    └──┬───────────────┬───┘
          │       │NO             │SÍ
          │       └──► más        │
          │            retrievals │
          ▼                       ▼
   ┌────────────────────────────────────┐
   │  LLM GENERADOR                     │
   │  construye la respuesta            │
   └───────────────┬────────────────────┘
                   ▼
   ┌────────────────────────────────────┐
   │  LLM DE CITAS                      │
   │  recorre la respuesta y añade      │
   │  las citas                         │
   └────────────────────────────────────┘
```

**Los dos puntos clave que el curso extrae**, válidos para cualquier sistema agentic:

> [!important] 🎯 (1) Diseñar un sistema agentic es esencialmente dibujar un diagrama de flujo
> *"Cada LLM en el diagrama sigue siendo solo texto de entrada y texto de salida, pero el sistema está armado de modo que **cada LLM completa una tarea** en el viaje del prompt por el sistema RAG."* No hay magia: hay orquestación.

> [!important] 🎯 (2) No necesitas usar el mismo LLM en cada paso
> *"El router y el evaluator podrían ser modelos livianos, rápidos y baratos de correr, ya que tienen una tarea única y relativamente simple. Podrías usar un modelo más grande para generar el borrador de la respuesta, y elegir un modelo especializado en generación de citas para ese paso."*
>
> Esto tiene una implicación económica fuerte: **un sistema agentic bien diseñado puede ser más barato que uno monolítico**, porque reserva el modelo caro para el único paso que lo necesita.

### 5.2 Los cuatro patrones

| Patrón | Cómo funciona | Ejemplo del curso |
|---|---|---|
| **Sequential** | La salida se mueve linealmente por una serie de LLMs | Todo prompt pasa por query parser → query rewriter → generador → citador. Cada LLM se especializa en su paso |
| **Conditional** | Un LLM decide **cuál de varios caminos** sigue el prompt | El router que decide si hace falta retrieval. O uno que decide **cuál de varios LLMs** con distintas fortalezas debe generar la respuesta |
| **Iterative** | Como el conditional, pero **rutea a un punto anterior** del sistema, formando un **bucle** | Generar código que se integra a un code base: un evaluator LLM juzga cada borrador —quizás con ayuda de un intérprete— y da feedback hasta considerarlo apto |
| **Parallel** | Un **orquestador** parte el prompt en tareas distintas y las asigna a LLMs separados; un **sintetizador** recombina el trabajo | Comparar los hallazgos clave de dos papers: dos LLMs resumen y evalúan uno cada uno, y el orquestador combina |

```
   SEQUENTIAL     A ──► B ──► C

   CONDITIONAL    A ──┬──► B
                      └──► C

   ITERATIVE      A ──► B ──► C
                       ▲      │
                       └──────┘

   PARALLEL         ┌──► B ──┐
                  A ┼──► C ──┼──► síntesis
                    └──► D ──┘
```

**Sobre la implementación:** *"Para sistemas agentic simples, puedes implementar la lógica del flujo tú mismo. A medida que las cosas se complican, hay una amplia variedad de herramientas, librerías y plataformas diseñadas para ayudarte."*

### 5.3 El cambio de mentalidad

> [!important] 🎯 La frase más importante de la lección
> *"Hay un **cambio de mentalidad** en juego aquí. Los LLMs empiezan a parecer un poco menos como soluciones autónomas y más como **piezas modulares que encajan dentro de un flujo de trabajo mayor**. De repente, estás más que contento de usar modelos más pequeños o modelos que solo destacan en unas pocas tareas, porque sus capacidades están bien alineadas con las porciones del flujo de las que son responsables."*
>
> Es el mismo reencuadre que hizo el re-ranking en el [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]]: dejar de buscar **un** modelo que haga todo bien, y empezar a componer modelos que hagan **una cosa** muy bien.

### 5.4 💻 El assignment: un router real, con sus grietas

Audiencia: 🔧

> [!note] Contexto del assignment graded C1M4
> Un chatbot RAG para una tienda de ropa ficticia, sobre una collection de **44.423 productos** en Weaviate. Es un **router de dos niveles** con hasta **cuatro llamadas secuenciales** al LLM.

```
   Query del usuario
        │
        ▼
   ① ROUTER nivel 1  →  FAQ | Product | None      (temp=0.3, max_tokens=1)
        │
        ├─ FAQ ──────► generador con la FAQ completa en el prompt
        │
        └─ Product
             │
             ▼
        ② ROUTER nivel 2 → creative | technical    (temp=0, max_tokens=1)
             │                    │
             │                    └──► define temperature y top_p (T08 §4.4)
             ▼
        ③ EXTRACTOR de metadata → JSON de filtros  (temp=0, max_tokens=1500)
             │
             ▼
        ④ Búsqueda semántica + filtros de metadata → generador
```

> [!tip] 🎯 Lo elegante del paso ③
> El extractor de metadata convierte *"un look para hombre en un día soleado, no más de 300 dólares por pieza"* en filtros estructurados:
>
> ```json
> {"gender": ["Men"], "articleType": ["Tshirts","Shorts","Casual Shoes"],
>  "price": {"min": 0, "max": 300}, "season": ["Summer"], "usage": ["Casual","Sports"]}
> ```
>
> Es **NER aplicado** ([[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]] §2.3): extraer entidades de la query y convertirlas en filtros duros, en vez de esperar que el embedding entienda "menos de 300 dólares". Y es la respuesta definitiva al caso Kyoto del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]].

> [!danger] 🚨 Cinco defectos del assignment, y el tercero arruina el ejemplo
> **(1) El filtro de precio se descarta cuando `min = 0`.** El código hace `if min_price <= 0 or max_price == 'inf': continue`. Con la salida real del extractor (`{"min": 0, "max": 300}`) → `0 <= 0` es verdadero → **no se aplica ningún filtro de precio**. El *"no quiero gastar más de 300 dólares"* se ignora en silencio, después de haber gastado una llamada de LLM en extraerlo.
>
> **(2) Los comparadores son estrictos.** Usa `greater_than` / `less_than`, así que con `{"min": 50, "max": 200}` un producto de exactamente 50 o 200 queda fuera — contradiciendo el *"at least 50 dollars"* de su propia query de ejemplo.
>
> **(3) La lógica de relajación de filtros está invertida, y se ve en el output.** La lista de prioridades tiene `'masterCategory'` **dos veces** y **le falta `'articleType'`**; y el bucle elimina **cuatro filtros de golpe en la primera iteración** en vez de relajar de a poco. Como retorna en cuanto encuentra ≥5 resultados, en la práctica **casi siempre tira `gender`, `season`, `usage` y `masterCategory` de entrada**.
>
> El resultado es visible: para *"un look para un hombre en una fiesta de boda de noche"*, el sistema devolvió **unos zapatos y una corbata**. No un traje, no una camisa. Un "look" de dos accesorios — porque los filtros que garantizaban coherencia se descartaron en el primer intento.
>
> **(4) El prompt del extractor interpola `set` de Python como si fueran JSON.** Dice *"los valores posibles se dan en el siguiente JSON: {values}"*, pero `values` contiene `set`, cuyo `repr` usa comillas simples y **cuyo orden de iteración no es estable entre procesos**. El prompt cambia en cada reinicio del kernel — **irreproducible**, en un ejercicio que corre con `temperature=0` precisamente para ser determinista.
>
> **(5) El ChatBot final corre con un modelo distinto al de todos los ejercicios.** `ChatBot.chat` fuerza `Llama-3.3-70B-Instruct-Turbo`, descartando el modelo que el estudiante configuró. El notebook no lo menciona en ninguna parte.

> [!warning] ⚠️ Y el bug de separadores, por tercera vez
> Dos f-strings concatenados sin separador, uno **visible en el output real**: `...Product Type: Tshirts. Product Category: Topwear Product Color: ...` — "Topwear" y "Product Color" pegados. Y en un prompt de fallback: `"...rephrase it.Answer it based on..."`.
>
> Van **tres módulos consecutivos** (C1M1, C1M3, C1M4) con el mismo defecto. Ya no es un descuido: es un **patrón del material**, y la razón por la que el [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08]] §4.1 explica que los delimitadores son lo que el modelo usa para separar instrucción de contenido.

---

## 6. ⚖️ RAG vs fine-tuning

Audiencia: 🔧 🧭 👔

**🔧 Definición:** mientras RAG **aumenta el prompt**, el **fine-tuning reentrena** un LLM off-the-shelf para mejorar su rendimiento en un contexto específico, actualizando sus **parámetros internos**.

```
   SUPERVISED FINE-TUNING (SFT)
   Se reentrena con un dataset ETIQUETADO del dominio al que se adapta.

   INSTRUCTION FINE-TUNING (un caso de SFT)
   El dataset incluye tanto INSTRUCCIONES (un prompt o pregunta)
   como la MEJOR RESPUESTA esperada (ground truth).
              │
              ▼
   Se le dan las instrucciones, se mide cuán cerca queda su salida
   de las respuestas correctas, y se ajustan sus parámetros internos
   para alinearlo mejor.
```

*"Este proceso es muy similar a la forma en que los modelos se entrenan inicialmente, pero el dataset viene de un dominio específico en el que el modelo se está especializando."*

### 6.1 El ejemplo de salud

```
   Modelo GENERAL, preguntado por: "dolor articular, erupción
   cutánea, sensibilidad al sol"
        └──► respuesta genérica, en tono genérico

   Mismo modelo con INSTRUCTION TUNING sobre muchas instrucciones
   y respuestas del dominio médico
        └──► responde con más exactitud, más detalle, y en un
             ESTILO apropiado para el dominio médico
```

### 6.2 El costo que casi nadie menciona

> [!danger] 🚨 El fine-tuning puede **empeorar** el rendimiento en otros dominios
> *"Aunque el rendimiento del modelo mejorará en ese dominio, el fine-tuning puede en realidad **disminuir el rendimiento en otros dominios**. El proceso de fine-tuning solo está optimizando el rendimiento en el dominio objetivo, lo que significa que a veces los ajustes hechos a los parámetros internos llevarán a un rendimiento más bajo con otro tipo de pedidos."*
>
> Y el matiz que lo hace aceptable: *"siempre y cuando el modelo solo vaya a usarse dentro del dominio para el que se especializa, este trade-off normalmente vale la pena"*.

**🧭 Y de ahí sale el caso de uso ideal**, que conecta directo con la sección 5:

> [!tip] 🎯 Fine-tuning + agentic: la combinación que el curso destaca
> *"Un lugar donde esto es particularmente cierto es para los **modelos pequeños usados dentro de sistemas agentic**. Si sabes de antemano que el único trabajo de un modelo será determinar si un prompt requiere retrieval de una vector database, estás más que contento de usar un modelo pequeño y liviano, y **fine-tunearlo agresivamente para que rinda bien solo en esa única tarea**."*
>
> Es el argumento más fuerte a favor del fine-tuning en un RAG, y **no es sobre el modelo principal**: es sobre los routers y evaluators del flujo agentic. Un router fine-tuneado es barato, rápido y muy preciso en su única tarea — y a nadie le importa que haya perdido capacidad general.

### 6.3 La regla que hay que memorizar

> [!important] 🎯 El consenso actual, en una línea
> ```
>    RAG          →  KNOWLEDGE INJECTION   (el QUÉ: los hechos)
>    FINE-TUNING  →  DOMAIN ADAPTATION     (el CÓMO: estilo, formato, tarea)
> ```
>
> Y la advertencia que explica por qué tanta gente lo hace mal: *"vale notar que el fine-tuning **usualmente no es una buena forma de enseñarle información nueva** a un LLM. La forma en que un modelo se adapta con fine-tuning tiende a tener mayor impacto en **cómo** responde a los prompts —las palabras que usa, el estilo, la estructura— y **menos impacto en QUÉ información conoce**."*

| Necesitas… | Usa | Por qué |
|---|---|---|
| Que el LLM tenga **información nueva** | **RAG** | La inyectas en el prompt y un modelo off-the-shelf la incorpora |
| Que se **especialice** en una tarea o dominio | **Fine-tuning** | Sobre todo si maneja **una tarea discreta**, como rutear prompts |

### 6.4 Y la respuesta correcta suele ser "ambos"

> [!important] La conclusión del curso
> *"Al decidir si usar fine-tuning o RAG, **la mejor opción podría ser ambos**. Cada enfoque mejora el rendimiento del modelo de formas distintas, y hay beneficios en usarlos juntos."*
>
> El caso concreto que menciona: *"podrías fine-tunear un modelo específicamente para **incorporar información recuperada** en sus respuestas finales. En otras palabras, estás ayudando al modelo a especializarse en su rol dentro del sistema RAG."*
>
> *"RAG y fine-tuning a veces se describen como alternativas en competencia, pero se ven más adecuadamente como **herramientas complementarias**."*

> [!note] Lo que el curso NO cubre, y lo dice
> *"Si quieres incorporar fine-tuning a tu propio sistema RAG, te recomiendo que tomes un curso separado sobre fine-tuning. Es un tema complejo y sería imposible cubrirlo adecuadamente dentro de este curso."*
>
> Y una salida práctica muy sensata: *"a menudo puedes encontrar modelos que **ya han sido fine-tuneados** y adaptados a una tarea o dominio particular"*. Antes de entrenar, busca en los repositorios públicos.
>
> ⚠️ **Nota sobre el material:** en la carpeta del assignment hay un archivo `clothing_ft_format.csv` (1.000 filas, columna única `text`, formato de fine-tuning causal) que **no se usa en ninguna parte del curso**. El assignment **no hace fine-tuning**. Es un residuo, probablemente de otra versión del material — no lo busques en el notebook.

---

> [!example] 📊 Caso de negocio — Sector público: el asistente que tuvo que probar que no inventaba
> **Problema:** un organismo público despliega un asistente para consultas ciudadanas sobre trámites, plazos y requisitos de beneficios sociales. Funciona bien en las pruebas internas y el equipo quiere lanzarlo. Legal lo bloquea con una pregunta que nadie puede responder: *"¿cómo demuestran que no le va a decir a un ciudadano que tiene derecho a un beneficio que no le corresponde?"* — el escenario del descuento estudiantil (sección 3.1), pero con consecuencias legales y un ciudadano que puede haber tomado decisiones basándose en la respuesta.
>
> **Técnica aplicada:** se construye la capa de medición completa, en el orden de este tomo. **(1) Separar el diagnóstico** (sección 1): un set de ~80 consultas reales del centro de atención, con los documentos relevantes marcados a mano, priorizando las más frecuentes y **las que ya habían fallado**. Se mide `recall@k` para saber si el retriever encuentra, y `MAP@K` para saber si además ordena bien. **(2) Grounding verificable** (3.5–3.6): el system prompt restringe las afirmaciones a los documentos recuperados y exige citar norma y artículo; un sistema externo de atribución verifica oración por oración, y **las afirmaciones sin respaldo se marcan como `no source`** en vez de pasar. **(3) Faithfulness como métrica de salida** (4.2): el porcentaje de afirmaciones respaldadas se convierte en el indicador que Legal acepta como criterio de aprobación. **(4) El set incluye deliberadamente consultas sin respuesta en el corpus** (3.8): trámites que el organismo no gestiona, para medir si el asistente dice *"esto no está en la normativa que administro"* o improvisa. **(5) Un router liviano** (sección 5) separa consultas informativas de las que requieren derivación a un humano — y ese router, por ser una tarea única y discreta, es el candidato natural a fine-tuning (6.2), no el modelo principal.
>
> **Resultado:** el asistente sale a producción con un criterio de aprobación **numérico** en vez de una opinión, y con cada respuesta trazable a norma y artículo. Pero el hallazgo que cambió el proyecto fue el del punto (4): en la primera medición, **el sistema inventaba una respuesta plausible ante consultas fuera de su ámbito** en una fracción nada despreciable de los casos — un fallo que ninguna de las pruebas internas había detectado, porque a nadie se le había ocurrido preguntar algo que el corpus no cubría. La lección: **el set de evaluación que solo contiene preguntas respondibles mide la mitad del sistema** — y la mitad que no mide es justamente la que produce el titular.

---

## 7. 🧭 Guía de decisión del tomo

Audiencia: 🔧 🧭

| Síntoma / situación | Qué hacer |
|---|---|
| "El sistema no funciona bien" | **Primero**: ¿el documento correcto estaba entre los recuperados? Eso decide si es del retriever o del LLM |
| Necesitas una sola métrica para empezar | **Recall@K** — la más fundamental |
| El retriever encuentra pero las respuestas son malas | Mira **precision** (¿ruido?) y **MAP** (¿mal orden?) |
| Quieres justificar un re-ranker | **MAP@K** es la métrica que muestra la mejora de orden |
| Basta un buen resultado por consulta | **MRR** |
| El recall te parece bajísimo | Verifica cuántos relevantes existen. En un RAG **un recall bajo puede ser correcto** |
| ¿El LLM responde lo que se le preguntó? | **Response relevancy** |
| ¿El LLM se inventa cosas? | **Faithfulness** — y atribución oración por oración |
| Quieres detectar alucinaciones sin knowledge base | Self-consistency checking (caro y poco confiable) |
| Quieres que las respuestas sean verificables | System prompt restrictivo + **citas obligatorias** + verificación externa |
| Quieres medir si tu sistema sabe decir "no sé" | **Incluye preguntas sin respuesta en el corpus** en tu golden set |
| Evaluación barata y continua | 👍/👎 de usuarios + A/B test aislando un cambio a la vez |
| Un paso del flujo es simple y repetitivo | Sepáralo en un **LLM chico** (patrón agentic) — y considera fine-tunearlo |
| Necesitas **hechos** nuevos | **RAG** |
| Necesitas **estilo, formato o tarea** específica | **Fine-tuning** |
| **Siempre** | No uses como juez un modelo de la familia del evaluado |

> [!tip] 🧭 El orden correcto de trabajo
> Ground truth mínimo (decenas de queries, no miles) → **recall@k** para separar retriever de generador → **faithfulness** para el grounding → citas verificables → y solo entonces agentic y fine-tuning. Y una regla que vale más que todas las métricas: **mide antes de cambiar y después de cambiar.** Un cambio sin medición previa no es una mejora: es una apuesta.

---

## 8. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Ground truth** | Las respuestas correctas marcadas a mano. Sin esto no se puede medir nada |
| **Precision** | De lo que trajo, cuánto servía. Mide **confiabilidad** |
| **Recall** | De lo que existía, cuánto trajo. Mide **exhaustividad** |
| **@K** | Medido sobre los K primeros resultados |
| **MAP@K** | Como precision, pero **premiando poner lo bueno arriba** |
| **MRR** | Qué tan pronto aparece el primer resultado útil |
| **Hallucination** | Cuando el modelo afirma algo falso con total fluidez |
| **Grounding** | Que la respuesta se apoye en los documentos entregados |
| **Self-consistency checking** | Preguntar varias veces y ver si se contradice |
| **ContextCite** | Sistema que atribuye cada oración a su documento fuente, o la marca **sin fuente** |
| **ALCE** | Benchmark de citación: fluency, correctness y calidad de las citas |
| **RAGAS** | Librería open-source de métricas específicas para RAG |
| **Response relevancy** | ¿La respuesta contesta lo que se preguntó? (no si es verdad) |
| **Faithfulness** | Qué % de las afirmaciones está respaldado por la fuente. **La métrica central de un RAG** |
| **LLM-as-a-judge** | Usar un LLM para evaluar a otro. Barato, escalable, hay que calibrarlo |
| **Agentic workflow** | Varios LLMs, cada uno con una tarea, orquestados como un diagrama de flujo |
| **Router LLM** | Un modelo chico que solo decide por dónde sigue el prompt |
| **Evaluator LLM** | Un modelo que juzga si lo recuperado alcanza para responder |
| **SFT / instruction tuning** | Reentrenar el modelo con ejemplos etiquetados del dominio |
| **Knowledge injection vs domain adaptation** | Darle hechos (RAG) vs enseñarle a comportarse (fine-tuning) |

---

## 9. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Sé separar la responsabilidad del retriever de la del LLM, y por qué eso es lo primero al diagnosticar.
- [ ] Conozco los tres ingredientes de una métrica de retrieval y por qué el ground truth es el caro.
- [ ] Puedo calcular precision y recall, y explicar el trade-off entre ambas.
- [ ] Entiendo por qué las métricas llevan `@K` y qué rango recomienda el curso.
- [ ] Sé calcular una average precision y explicar **por qué MAP castiga el mal orden**.
- [ ] Sé calcular MRR y cuándo es la métrica correcta.
- [ ] Conozco la jerarquía: recall como base, precision y MAP sobre ella, MRR especializada.
- [ ] **Entiendo por qué un recall bajo en un RAG puede ser el comportamiento correcto.**
- [ ] Puedo explicar el caso del descuento estudiantil y por qué cada componente actuó razonablemente.
- [ ] Conozco las tres razones por las que las hallucinations son problemáticas, y por qué la tercera mata proyectos.
- [ ] Sé que las hallucinations vienen en **grados** y por qué eso obliga a evaluar en varios niveles.
- [ ] Tengo claro que **no existe solución perfecta** y que RAG es el mejor enfoque disponible.
- [ ] Entiendo self-consistency checking y por qué el curso lo considera caro y poco confiable.
- [ ] Sé cómo funciona la atribución oración por oración y por qué el tag "no source" es la señal clave.
- [ ] Conozco las tres métricas de ALCE.
- [ ] Puedo explicar cómo funciona response relevancy (el truco de trabajar hacia atrás) y qué **no** mide.
- [ ] Sé cómo se calcula faithfulness y por qué es la métrica central de un RAG.
- [ ] Entiendo que evaluar cuesta llamadas a LLMs, y que no debo usar un juez de la misma familia.
- [ ] Sé por qué debo incluir **preguntas sin respuesta** en mi set de evaluación.
- [ ] Puedo describir los cuatro patrones agentic y dar un ejemplo de cada uno.
- [ ] Entiendo que un sistema agentic **puede ser más barato** por reservar el modelo caro a un solo paso.
- [ ] Sé la regla RAG = knowledge injection / fine-tuning = domain adaptation.
- [ ] Entiendo que el fine-tuning **degrada otros dominios** y por qué eso está bien en un router.
- [ ] Tengo claro que la respuesta correcta suele ser **usar ambos**.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación]] (las perillas que este tomo enseña a medir)
- Siguiente tomo → [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · RAG en producción]] (monitoring, costo, latencia y seguridad)
- **Por qué el modelo alucina, a nivel de mecánica** → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]]
- Las decisiones del retriever que estas métricas evalúan → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]] · [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]] · [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]]
- El chunking que MAP y recall permiten comparar de verdad → [[Guia-Maestra-RAG_06-Chunking|Tomo 06]]
- Query parsing y re-ranking, los primeros componentes agentic → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — **Módulo 4**: lecciones de hallucinations, evaluación del LLM, agentic RAG y RAG vs. fine-tuning; **assignment graded C1M4** (el chatbot con router de dos niveles de la sección 5.4).
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — **Módulo 2**: lección de *retriever evaluation* y **Ungraded Lab 2** (métricas sobre 20 Newsgroups), de donde proviene toda la sección 2 por la excepción de ruteo declarada en el MOC.

**Fuentes externas (complemento con bibliografía verificable):**
- Es, S., James, J., Espinosa-Anke, L. & Schockaert, S. (2023). *RAGAS: Automated Evaluation of Retrieval Augmented Generation*. — La librería y las métricas de la sección 4.
- Cohen-Wang, B., Shah, H., Georgiev, K. & Madry, A. (2024). *ContextCite: Attributing Model Generation to Context*. NeurIPS. — El sistema de atribución oración por oración de la sección 3.6.
- Gao, T., Yen, H., Yu, J. & Chen, D. (2023). *Enabling Large Language Models to Generate Text with Citations*. EMNLP. — El benchmark **ALCE** y sus tres métricas.
- Manning, C. D., Raghavan, P. & Schütze, H. (2008). *Introduction to Information Retrieval*. Cambridge University Press. — Precision, recall, MAP y MRR en su formulación canónica.
- Ji, Z. et al. (2023). *Survey of Hallucination in Natural Language Generation*. ACM Computing Surveys, 55(12). — Taxonomía de hallucinations.
- Ouyang, L. et al. (2022). *Training Language Models to Follow Instructions with Human Feedback*. NeurIPS. — Instruction fine-tuning.
- Hu, E. J. et al. (2021). *LoRA: Low-Rank Adaptation of Large Language Models*. arXiv:2106.09685 (ICLR 2022). — El fine-tuning eficiente que hace viable la sección 6.
- Anthropic (2024). *Building Effective Agents*. — Los patrones de workflow (sequential, conditional/routing, iterative, parallel) de la sección 5.2.
- Lewis, P. et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS. — El paper fundacional.

> [!note] Sobre el código y los valores de este tomo
> - **Del curso:** toda la teoría de las secciones 1 a 6 proviene de las lecciones (M4 para hallucinations, evaluación, agentic y fine-tuning; M2 para las métricas de retrieval). Los ejemplos numéricos de precision, recall, MAP y MRR son los del curso, **recalculados y verificados** (MAP = 0.7; MRR = 0.5). Los resultados de 2.7 son del Ungraded Lab 2 del M2, y el flujo y los defectos de 5.4 son del assignment graded C1M4.
> - **Hallazgos propios verificados contra el assignment:** que el **filtro de precio se descarta cuando `min = 0`** (anulando en silencio la restricción que el extractor acababa de inferir); que los comparadores son estrictos y excluyen los extremos; que la lista de prioridades de relajación tiene un campo **duplicado** y **le falta `articleType`**, con la lógica **invertida** (elimina cuatro filtros en la primera iteración) — y que eso explica el "look de boda" compuesto por unos zapatos y una corbata; que el prompt del extractor **interpola `set` de Python** presentándolos como JSON, con orden no reproducible entre procesos; que el **ChatBot final usa un modelo distinto** al de todos los ejercicios sin documentarlo; y el **bug de separadores por tercer módulo consecutivo**, uno de ellos visible en el output.
> - **Análisis propio marcado en el texto:** la simetría entre ContextCite y faithfulness como dos implementaciones de la misma idea, el argumento de que un sistema agentic bien diseñado puede ser **más barato** que uno monolítico, la lectura de que el extractor de metadata del assignment es **NER aplicado**, la estrategia para construir ground truth sin morir en el intento (2.6) y las guías de decisión.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · RAG en producción: monitoring, cost, latency, security]]**, donde arranca el **Módulo 5** y el sistema sale del laboratorio: qué se rompe con tráfico real, cómo se observa lo que pasa dentro, cuánto cuesta operarlo, y qué superficies de ataque abre un sistema que lee documentos y escribe texto.
