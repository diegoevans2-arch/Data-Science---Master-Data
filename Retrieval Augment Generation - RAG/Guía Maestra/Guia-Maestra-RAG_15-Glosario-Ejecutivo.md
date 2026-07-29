---
title: "Tomo 15 — Glosario ejecutivo"
tags: [rag, glosario, ejecutivo, vocabulario, transversal, referencia]
audiencias: [ejecutivo, puente, tecnico]
tomo: 15
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 15 — Glosario ejecutivo

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13 · ⭐ Complemento: frameworks]] · Siguiente → [[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 · Bibliografía]]
>
> *(Los Tomos 12 y 13 son complementos de vanguardia aún pendientes. Este tomo se lee perfectamente sin ellos.)*

---

> [!info] ¿Por qué importa este tomo?
> Existe por una razón concreta: **para que puedas sostener una conversación sobre un sistema RAG sin fingir que entiendes, y sin necesitar que alguien te la traduzca.**
>
> No es un resumen de la guía. Es un **traductor**. Cada término viene con dos cosas: qué es en una frase, y — lo que realmente importa — **qué decisión tuya depende de él**. Porque el problema del vocabulario técnico en una reunión no es no saber qué significa una palabra: es no saber si esa palabra implica que hay que aprobar más presupuesto, aceptar un riesgo, o hacer una pregunta incómoda.
>
> Los 11 tomos anteriores definen estos términos técnicamente. Aquí están traducidos.

> [!abstract] 👔 Impacto ejecutivo
> Este tomo cierra la **ruta ejecutiva** de la guía ([[Guia-Maestra-RAG_01-Introduccion-a-RAG|01]] → secciones 👔 de [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|09]] y [[Guia-Maestra-RAG_10-RAG-en-Produccion|10]] → este).
> - **Decisiones que habilita:** entender qué te están proponiendo antes de firmarlo; distinguir un problema real de un tecnicismo; saber qué preguntar cuando alguien dice "funciona bien".
> - **Costo o riesgo de no tenerlo:** aprobar arquitecturas por confianza en quien las presenta y no por comprensión de lo que hacen. Es la vía más común por la que un proyecto de IA se descubre inviable **después** de gastarse el presupuesto.
> - **Pregunta que responde:** *"¿qué me están diciendo exactamente, y qué debería preguntar yo?"*

---

## 1. 🧭 Cómo usar este tomo

Audiencia: 👔 🧭

Tres formas, según lo que necesites:

| Situación | Qué leer |
|---|---|
| **Tengo 10 minutos antes de una reunión** | La sección 2: los 12 términos que no puedes no saber |
| **Estoy en una fase concreta del proyecto** | La sección 3, que agrupa el vocabulario por el momento de la conversación en que aparece |
| **Alguien dijo una palabra y no sé qué es** | El índice alfabético de la sección 7 — están los **154 términos** de la guía, con el tomo donde se definen a fondo |

> [!warning] ⚠️ Antes de nada: la sección 4 es la más útil de este tomo
> Hay términos que **suenan casi iguales y significan cosas distintas** — y uno que significa **dos cosas incompatibles** según quién lo diga. Confundirlos lleva a conversaciones donde dos personas creen estar de acuerdo y no lo están. Si solo vas a leer una sección, lee esa.

---

## 2. 🎯 Los 12 términos que no puedes no saber

Audiencia: 👔

El mínimo absoluto. Con esto entiendes de qué va cualquier conversación sobre RAG.

| # | Término | Qué es | Por qué te importa a ti |
|---|---|---|---|
| 1 | **RAG** | Darle al modelo acceso de solo lectura a tu información justo antes de que responda | Es la alternativa barata y actualizable a reentrenar un modelo. La decisión de fondo del proyecto |
| 2 | **LLM** | El "cerebro" de lenguaje: escribe y razona, pero solo sabe lo que aprendió hasta su fecha de corte | Es el componente más caro y más lento. Casi toda la factura y casi toda la espera salen de aquí |
| 3 | **Knowledge base** | La colección de documentos confiables que alimenta al sistema | **Es tu activo.** Si está incompleta o desordenada, ninguna sofisticación técnica lo compensa |
| 4 | **Hallucination** | Cuando el modelo inventa una respuesta con tono seguro en vez de admitir que no sabe | El riesgo reputacional y legal número uno. No se ve como un error: se ve como una respuesta normal |
| 5 | **Grounding** | Que la respuesta se apoye en la evidencia entregada y no en la memoria del modelo | Es lo contrario de alucinar. Cuando alguien diga "el sistema está grounded", te está diciendo que puede citar de dónde sacó cada cosa |
| 6 | **Retrieval** | La búsqueda que trae los fragmentos relevantes para la pregunta | Si esto falla, todo lo demás da igual. **La mayoría de los fallos "del modelo" son en realidad fallos de aquí** |
| 7 | **Chunking** | Partir los documentos en fragmentos antes de indexarlos | Decide el nivel de detalle del sistema. Mal hecho, corta ideas por la mitad y el sistema responde a medias |
| 8 | **Embedding** | Convertir un texto en números para poder compararlo por significado | Lo que permite encontrar "cancelación de reserva" cuando el usuario escribió "quiero echarme atrás" |
| 9 | **Context window** | El máximo de texto que el modelo puede procesar de una sola vez | Un techo duro. Y **no basta con caber**: la calidad se degrada mucho antes de llegar al límite |
| 10 | **Token** | La pieza mínima de texto que el modelo maneja | **Es la unidad en la que te cobran.** Cuando alguien hable de reducir costos, habla de reducir esto |
| 11 | **Fine-tuning** | Reentrenar el modelo con tus datos | La alternativa cara a RAG. Sirve para cambiar *cómo se comporta*, **no** para meterle información |
| 12 | **Ground truth** | Las respuestas correctas marcadas a mano | Sin esto **no se puede medir nada**. Y cuesta trabajo humano. Si nadie lo menciona en el plan, el plan no incluye saber si funciona |

> [!important] 🎯 Si solo te llevas una idea de esta sección
> **Los números 6 y 11 son los que más dinero salvan.**
>
> El 6 porque cambia el diagnóstico: cuando el sistema responde mal, la pregunta correcta no es *"¿mejoramos el modelo?"* sino *"¿el documento correcto estaba entre los que encontró?"*. Si no estaba, cambiar de modelo no arregla nada.
>
> El 11 porque zanja un debate caro y recurrente: **RAG inyecta hechos, fine-tuning ajusta comportamiento.** Usar fine-tuning para meter información es el error costoso clásico — se desactualiza al día siguiente y el modelo igual inventa.

---

## 3. 💬 Vocabulario por momento de la conversación

Audiencia: 👔 🧭

### 3.1 Antes de aprobar: ¿qué es esto y por qué no es otra cosa?

| Término | Qué es | Por qué te importa |
|---|---|---|
| **Knowledge cutoff** | La fecha límite del conocimiento del modelo | Explica por qué un modelo excelente no sabe nada de tu empresa ni de lo que pasó el mes pasado |
| **Indexing** | El trabajo previo de preparar y organizar los documentos | Es coste de puesta en marcha, no de operación. Se paga una vez (y cada vez que cambian los documentos) |
| **Prompt aumentado** | La pregunta del usuario + la evidencia recuperada, entregadas juntas | Es literalmente lo que RAG hace. Todo lo demás es cómo conseguir esa evidencia |
| **Information retrieval** | La disciplina, previa a los LLMs, de encontrar información en grandes colecciones | Señal de madurez: media parte de RAG es tecnología de décadas, no una apuesta experimental |
| **Knowledge injection vs. domain adaptation** | Darle hechos (RAG) vs. enseñarle a comportarse (fine-tuning) | La formulación precisa del punto 11 de arriba. Útil para cortar la discusión |

### 3.2 La fase de búsqueda — cómo encuentra la información

| Término | Qué es | Por qué te importa |
|---|---|---|
| **Keyword search** | Buscar por las palabras exactas que escribió el usuario | Barato, rápido, y **mejor de lo que la gente cree** para códigos, referencias y nombres propios |
| **Semantic search** | Buscar por significado en vez de por palabras exactas | Lo que resuelve que cliente y documento usen palabras distintas para lo mismo |
| **Hybrid search** | Combinar ambas más filtros en un único ranking | La respuesta correcta casi siempre. No es "una u otra" |
| **Vocabulary mismatch** | Cuando el usuario y el documento dicen lo mismo con otras palabras | El problema concreto que justifica pagar por semantic search |
| **BM25** | El estándar de producción de la búsqueda por palabras | Si alguien propone "solo IA" para buscar, esto es lo que está descartando sin decirlo |
| **Metadata filtering** | Filtrar por etiquetas: área, fecha, permisos | 🚨 Útil para personalizar. **NO es un mecanismo de seguridad** (ver §4) |
| **Vector database** | Base de datos hecha para buscar por significado a escala | Suele ser el segundo mayor costo de infraestructura después del LLM |
| **`top_k` (retrieval)** | Cuántos fragmentos se recuperan y se le entregan al modelo | **La perilla que arbitra calidad contra costo.** Más fragmentos = mejores respuestas y factura más alta |
| **Re-ranking** | Reordenar los resultados con un modelo mejor antes de dárselos al LLM | Mejora barata y de alto impacto: los documentos correctos ya estaban, pero llegaban en mal orden |
| **HNSW** | El método estándar para buscar rápido entre millones de vectores | El que exige memoria RAM cara. Explica una parte grande de la factura de infraestructura |
| **ANN** | Buscar de forma astuta en vez de comparar contra todo | Casi perfecto y muchísimo más rápido. El "casi" es medible y se decide |
| **Over-fetch** | Traer más resultados de los necesarios para que el re-ranker tenga con qué trabajar | Si alguien añade re-ranking sin esto, el re-ranking no sirve de nada |
| **HyDE** | Inventar el documento ideal y buscar con él, en vez de con la pregunta | Truco contraintuitivo que mejora búsquedas mal formuladas |

### 3.3 La fase de respuesta — cómo redacta

| Término | Qué es | Por qué te importa |
|---|---|---|
| **System prompt** | Las instrucciones que gobiernan **todas** las respuestas | **La pieza de mayor apalancamiento del sistema.** Cambiarla cambia el comportamiento entero, gratis y al instante |
| **`temperature`** | El dial de creatividad | Bajo = predecible y factual. Alto = variado y arriesgado. Para respuestas factuales, bajo |
| **`top_k` (sampling)** | Cuántas palabras candidatas considera el modelo al escribir | ⚠️ **Nada que ver con el `top_k` de búsqueda.** Ver §4 |
| **Chain-of-thought** | Pedirle que razone paso a paso antes de responder | Mejora tareas de lógica. Cuesta más tokens, o sea más dinero |
| **Reasoning model** | Modelo que piensa antes de responder, de fábrica | Más preciso, **más lento y más caro**. Decisión de costo, no solo de calidad |
| **In-context learning** | Enseñarle con ejemplos dentro del prompt | Alternativa barata al fine-tuning para ajustar formato o tono |
| **Context pruning** | Recortar el historial de conversación para que quepa | Explica por qué un asistente "se olvida" de lo que se habló hace rato |

### 3.4 "¿Esto funciona?" — evaluación

| Término | Qué es | Por qué te importa |
|---|---|---|
| **Precision** | De lo que trajo, cuánto servía | Mide **confiabilidad**. Es la que no puede ser baja |
| **Recall** | De lo que existía, cuánto trajo | Mide **exhaustividad**. Puede ser baja legítimamente (ver la nota de abajo) |
| **Faithfulness** | Qué porcentaje de las afirmaciones está respaldado por la fuente | **La métrica central de un RAG.** Si te dan un solo número, pide este |
| **Response relevancy** | ¿La respuesta contesta lo que se preguntó? | Ojo: mide si responde, **no si es verdad**. Las dos cosas hacen falta |
| **RAGAS** | Librería estándar de métricas específicas para RAG | Que la mencionen es buena señal: significa que no van a inventarse las métricas |
| **LLM-as-a-judge** | Usar un LLM para evaluar las respuestas de otro | Barato y escalable. Requiere criterios claros y **no puede juzgar a un modelo de su propia familia** sin sesgo |
| **Agentic workflow** | Varios modelos, cada uno con una tarea, orquestados como un diagrama de flujo | Más capacidad y más puntos de fallo. Encarece el diagnóstico |
| **Router LLM** | Un modelo pequeño que decide por dónde sigue cada consulta | Palanca clave de costo: manda lo simple a lo barato y lo complejo a lo caro |
| **Self-consistency checking** | Preguntar varias veces y ver si el sistema se contradice | Detector de alucinaciones barato de implementar |

> [!important] 🎯 El malentendido más caro de esta sección
> **Un recall bajo puede ser el comportamiento correcto.** En un RAG el objetivo no es encontrar *todos* los documentos relevantes, sino **los mejores pocos**. Si en una categoría hay 600 documentos relevantes y el sistema entrega 5, el recall será ~1 % por pura aritmética — y estará funcionando bien.
>
> Lo que **no** puede ser bajo es la **precision**. Si de los 5 que entrega solo 1 sirve, ahí sí hay un problema.
>
> Si alguien te presenta un recall bajo como señal de fracaso, o te oculta la precision, esa es la conversación que hay que tener.

### 3.5 "¿Cuánto cuesta y qué puede salir mal?" — producción

| Término | Qué es | Por qué te importa |
|---|---|---|
| **Observability** | Saber qué hace el sistema por dentro a partir de lo que emite | **Sin esto, el primero en enterarse de un fallo es el cliente.** Es el prerrequisito de todo lo demás |
| **Trace** | El recorrido completo de una consulta por el sistema, paso a paso | Lo que convierte "no sabemos por qué falló" en "falló en el paso 3". Como el seguimiento de un paquete |
| **Custom dataset** | Guardar el tráfico real para poder re-correrlo | Permite probar un cambio contra preguntas **reales** en vez de inventadas. Los datos que no guardes no existirán después |
| **Data void** | Una consulta para la que tu knowledge base no tiene material serio | ⚠️ Modo de fallo propio de RAG: el sistema devolverá lo más parecido que encuentre, aunque sea una broma. **Ninguna métrica del modelo lo detecta** |
| **Quantization** | Comprimir modelos y vectores para que ocupen y cuesten menos | Ahorro grande (4× a 32×) por una pérdida de calidad **pequeña pero real, y que hay que medir** |
| **Caching semántico** | Devolver una respuesta ya generada si la pregunta se parece a una anterior | Gran ahorro de tiempo y dinero. Riesgo: si el umbral se afloja, responde la pregunta de otro |
| **Dedicated endpoint** | Alquilar hardware por hora en vez de pagar por consumo | A volumen alto sale mucho más barato y más fiable. Es una decisión de escala |
| **Multi-tenancy** | Separar los datos de cada cliente en su propio compartimento | Resuelve **dos** problemas a la vez: control de acceso y costo. Por eso suele valer la pena |
| **RBAC** | Permisos según el rol del usuario | El mecanismo correcto para que cada quien vea solo lo suyo |
| **On-premise** | Todo en hardware propio, sin que el dato salga nunca | La decisión cuando el dato no puede salir. **Se toma antes de construir**, no después |
| **Vector inversion** | Reconstruir el texto original a partir de sus números | 🚨 Desmonta la creencia de que "los vectores son anónimos porque son números". **No lo son** |
| **Negligent misrepresentation** | Responder legalmente por información incorrecta que publicaste | Lo que tu sistema afirma, **lo afirma tu organización**. Hay jurisprudencia |

---

## 4. 🚨 Términos que suenan igual y NO lo son

Audiencia: 👔 🧭 🔧

**La sección más útil de este tomo.** Estas confusiones producen reuniones donde dos personas creen estar de acuerdo y no lo están.

### 4.1 ⚠️ `top_k` significa DOS cosas incompatibles

Es la trampa más seria del vocabulario de RAG, y aparece constantemente porque ambos usos son legítimos y frecuentes.

| | `top_k` de **retrieval** | `top_k` de **sampling** |
|---|---|---|
| **Qué controla** | Cuántos documentos se recuperan y se le pasan al modelo | Entre cuántas palabras candidatas elige el modelo al escribir |
| **Dónde vive** | En el buscador | Dentro del generador |
| **Si lo subes** | Más contexto, mejores respuestas, **más caro** | Texto más variado y creativo, menos predecible |
| **Se discute en** | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|Tomo 02]] | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|Tomo 08]] |

> [!danger] 🚨 Cómo se manifiesta el malentendido
> Alguien dice *"bajemos el `top_k` para reducir costos"* y otro entiende *"hagamos las respuestas más deterministas"*. Ambos asienten. Nadie ha acordado nada.
>
> **La pregunta que lo desambigua en dos segundos:** *"¿`top_k` de documentos o de sampling?"*

### 4.2 ⚠️ `alpha` y `beta` son el MISMO parámetro

Ambos controlan **cuánto pesa el significado frente a las palabras exactas** en hybrid search. La teoría lo llama `beta`; las herramientas (Weaviate y el estándar de facto) lo llaman `alpha`.

> [!warning] Y hay un agravante
> **Algunas implementaciones invierten el sentido.** Antes de tocarlo, hay que verificar con un caso extremo (ponerlo a 0 y a 1) para saber en qué dirección va. Está documentado en el [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]].

### 4.3 ⚠️ "Recall" a secas vs. "recall del índice"

| | **Recall** (métrica de calidad) | **Recall del índice** (fidelidad técnica) |
|---|---|---|
| **Compara contra** | Los documentos relevantes que existen de verdad | Lo que habría encontrado una búsqueda exacta y lenta |
| **Qué te dice** | Si el sistema encuentra lo que hay | Si el atajo rápido pierde algo respecto al método perfecto |
| **Valor típico bueno** | **Puede ser muy bajo legítimamente** (§3.4) | Alto: 0,95+ |

Son números distintos que se llaman casi igual. Confundirlos lleva a alarmarse por el número equivocado.

### 4.4 ⚠️ "Data contamination" vs. "data void"

Nombres peligrosamente parecidos, conceptos **sin ninguna relación**:

- **Data contamination:** el examen con el que mides el modelo estaba en sus datos de entrenamiento. **Infla los resultados** y te hace creer que es mejor de lo que es.
- **Data void:** tu knowledge base no tiene material serio sobre una consulta. **El sistema devolverá lo más parecido que encuentre**, aunque sea irrelevante o satírico.

Uno es un problema de *medición*; el otro, de *cobertura de tu información*.

### 4.5 ⚠️ "Vocabulary" según quién lo diga

- En el contexto del **modelo**: el catálogo de piezas de texto que puede usar (decenas de miles).
- En el contexto de la **búsqueda por palabras**: el catálogo de palabras que aparecen en tus documentos.

### 4.6 ⚠️ "Index", "indexing" e "inverted index"

Tres cosas distintas que suenan a lo mismo:

| Término | Qué es |
|---|---|
| **Indexing** | La **fase** de preparar los documentos (ocurre antes, una vez) |
| **Index** | La **estructura** que hace rápida la búsqueda |
| **Inverted index** | Una **implementación concreta** de esa estructura, para búsqueda por palabras |

### 4.7 🚨 Y el malentendido más peligroso de todos: *metadata filtering* no es seguridad

Parece la solución natural — etiquetar cada documento con su área y filtrar por permisos. **No lo es.**

> [!danger] 🚨 Por qué
> El metadata filtering está pensado para **personalizar la relevancia**, no para poner una frontera de seguridad. Es demasiado frágil: un filtro mal construido, un campo mal poblado, o una consulta que se lo salta, y el documento restringido entra en la respuesta.
>
> **Lo robusto es que ese documento ni siquiera esté en el archivador que se consulta** — eso es multi-tenancy con RBAC.
>
> La diferencia es la de siempre en seguridad: **tapar algo** frente a **no tenerlo delante**.
>
> Si en una revisión de arquitectura te dicen que el control de acceso se hace con filtros de metadata, esa es la pregunta que hay que hacer.

---

## 5. 🗣️ Frases que vas a oír, traducidas

Audiencia: 👔

| Lo que dicen | Lo que significa | Qué preguntar |
|---|---|---|
| *"El modelo está alucinando"* | Está afirmando cosas falsas con seguridad | ¿Es fallo del buscador o del generador? ¿Estaba el documento correcto entre los recuperados? |
| *"Hay que mejorar el prompt"* | Quieren cambiar las instrucciones del modelo | ¿Ya descartaron que el problema sea de búsqueda? Es la causa más frecuente y no se arregla con el prompt |
| *"Funciona bien en las pruebas"* | Funciona con las preguntas que se les ocurrieron a ellos | ¿Incluyeron preguntas que el sistema **no** puede responder? Ahí es donde aparece la alucinación |
| *"Subimos el `top_k`"* | Recuperan más documentos por consulta | ¿Cuánto sube el costo por consulta? ¿Mejoró alguna métrica de calidad medida? |
| *"Le vamos a hacer fine-tuning"* | Quieren reentrenar el modelo | ¿Para meterle información o para cambiar cómo se comporta? **Si es lo primero, es el enfoque equivocado** |
| *"Es un tema de latencia"* | El sistema tarda | ¿Dónde exactamente? Casi toda la latencia está en el modelo, no en la base de datos |
| *"Lo tenemos monitoreado"* | Hay métricas de algún tipo | ¿Pueden reconstruir una respuesta concreta de ayer y decirme qué documentos usó? |
| *"La calidad es buena, un 85 %"* | Un promedio agregado | ¿Cómo se reparte por tipo de consulta? **El promedio esconde el segmento que falla** |
| *"Optimizamos el costo un 40 %"* | Gastan menos | ¿Midieron la calidad antes y después? Si no, no optimizaron: **recortaron** |
| *"Los embeddings son anónimos, son solo números"* | Creen que los vectores no exponen datos | **Falso.** Existe investigación publicada que reconstruye el texto original desde el vector |
| *"El control de acceso está resuelto con filtros"* | Usan metadata filtering como seguridad | 🚨 Ver §4.7. Esta es la pregunta que hay que hacer |

---

## 6. ❓ Las preguntas que puedes hacer

Audiencia: 👔

Ordenadas por fase del proyecto. Ninguna requiere saber programar.

```
   ANTES DE APROBAR
   ├─ ¿Por qué RAG y no fine-tuning? (respuesta correcta: porque
   │  necesitamos inyectar HECHOS que además cambian)
   ├─ ¿Qué documentos van a la knowledge base, y quién los mantiene?
   └─ ¿Qué preguntas NO va a poder responder este sistema?

   DURANTE LA CONSTRUCCIÓN
   ├─ ¿Cómo sabremos si el problema está en la búsqueda o en la redacción?
   ├─ ¿Quién construye el ground truth y cuánto trabajo humano es?
   └─ ¿Qué pasa cuando alguien pregunta algo que no está en los documentos?

   ANTES DE DESPLEGAR
   ├─ ¿Pueden mostrarme el recorrido completo de una consulta concreta?
   ├─ ¿Cuál es la faithfulness medida, y sobre qué conjunto de preguntas?
   ├─ ¿Cómo se separa lo que puede ver cada usuario? (si la respuesta
   │  menciona "filtros de metadata" → §4.7)
   └─ ¿Puede el dato salir de la organización dentro de un prompt?

   YA EN PRODUCCIÓN
   ├─ ¿Cuánto cuesta una consulta, y de qué componente sale ese costo?
   ├─ ¿Cómo se reparte la calidad por tipo de consulta, no en promedio?
   └─ En el último cambio de optimización, ¿midieron la calidad antes
      y después?
```

> [!tip] 💡 La pregunta única, si solo puedes hacer una
> **"¿Pueden reconstruir una respuesta concreta que dio el sistema la semana pasada y mostrarme qué documentos usó para producirla?"**
>
> Si la respuesta es sí, hay observability de verdad y el equipo puede diagnosticar problemas. Si es no, cualquier afirmación sobre calidad, costo o mejora es una opinión.

---

## 7. 🔤 Índice alfabético completo

Audiencia: 🔧 🧭 👔

Los **154 términos** de la guía, con el tomo donde se define a fondo. Los marcados con ⚠️ están en la sección 4 (confusiones frecuentes).

| Término | Tomo |
|---|---|
| `alpha` ⚠️ (≡ `beta`) | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| `beta` ⚠️ (≡ `alpha`) | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| `chunk_index` | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| `ef` / `ef_construction` | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| `max_connections` | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| `rerank_score` | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| `source_properties` | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| `temperature` | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| `top_k` (retrieval) ⚠️ | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| `top_k` (sampling) ⚠️ | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| `top_p` | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| @K | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Agentic workflow | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| ALCE | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Anchor / positive / negative | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| ANN | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Attention | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Attention head | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Autoregressive | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Auto-instrument | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Bag of words | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| BatchSpanProcessor | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Bi-encoder | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| Binary quantization | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| BM25 | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Caching semántico | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Chain-of-thought | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Chunk | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Chunk size | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Chunking | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Code-based eval | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Collection | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| ColBERT | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| ColPali | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Context pruning | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Context window | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Context-aware chunking | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| ContextCite | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Contrastive training | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Cosine similarity | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Cross-encoder | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| Custom dataset | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Data contamination ⚠️ | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Data void ⚠️ | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Dedicated endpoint | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Dot product | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Embedding / dense vector | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Embedding model | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Embedding model multimodal | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Encoder / decoder | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Euclidean distance | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Evaluator LLM | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Faithfulness | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Feedforward | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Filtered search | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Fine-tuning | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Fixed-size chunking | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Flat search cutoff | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Greedy decoding | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Ground truth | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Grounding | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Hallucination | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] · [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| HNSW | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Hybrid search | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] · [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| HyDE | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| IDF | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| In-context learning | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Index ⚠️ | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Indexing ⚠️ | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Information retrieval | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Integer quantization (int8) | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Inverted index ⚠️ | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Jerarquía de memoria | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Keyword search | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| kNN / exact search | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Knowledge base | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Knowledge cutoff | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Knowledge injection vs. domain adaptation | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Language vision model | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| LLM | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| LLM-as-a-judge | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] · [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| LLM-based chunking | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Logit bias | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Maldición de la dimensionalidad | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| MAP@K | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Matryoshka embedding | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| MaxSim | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| Messages format | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Metadata filtering 🚨 | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Modelo multimodal | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| MRR | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Multi-tenancy | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] · [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Negligent misrepresentation | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| NER | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| NSW | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Observability | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| On-premise | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| OpenInference | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| OpenTelemetry | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Over-fetch | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| Overlap | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| PCA | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Pre/post-filtering | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Precision | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Probability distribution | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Prompt / completion | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Prompt aumentado | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Proximity graph | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Quantization | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Query parsing | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| Query rewriting | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| RAG | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| RAGAS | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| RBAC | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Reasoning model | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Recall ⚠️ | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Recall del índice ⚠️ | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Recursive splitting | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Re-ranking | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|07]] |
| Repetition penalty | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Rescoring | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Response relevancy | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Retrieval | [[Guia-Maestra-RAG_01-Introduccion-a-RAG\|01]] |
| Router LLM | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] · [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| RRF | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Saturación (de benchmark) | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Score de relevancia | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Scope (de un eval) | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Self-consistency checking | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Semantic chunking | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| Semantic search | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| SFT / instruction tuning | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|09]] |
| Sharding | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Span | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Span kind | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Sparse vector | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Stopwords | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Stride | [[Guia-Maestra-RAG_06-Chunking\|06]] |
| System prompt | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Term frequency saturation | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| TF (term frequency) | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| TF-IDF | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Token | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Trace | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Transformer | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|08]] |
| Truncation | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Vector database | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Vector inversion | [[Guia-Maestra-RAG_10-RAG-en-Produccion\|10]] |
| Vector space | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|04]] |
| Vectorizer | [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN\|05]] |
| Visual Document Retrieval (VDR) | [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG\|11]] |
| Vocabulary (índice/corpus) ⚠️ | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |
| Vocabulary (LLM/tokenizer) ⚠️ | [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG\|02]] |
| Vocabulary mismatch | [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25\|03]] |

---

## 🔗 Conexiones

- [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01 · Introducción a RAG]] — el punto de partida de la ruta ejecutiva, y donde se define la mayoría del vocabulario de la sección 2.
- [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Evaluación]] — el origen del vocabulario de "¿esto funciona?" (§3.4) y de la nota sobre el recall bajo legítimo.
- [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · Producción]] — el origen del vocabulario de riesgo y operación (§3.5), incluida la regla de que **metadata filtering no es seguridad**.
- [[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 · Bibliografía]] — dónde verificar cualquier afirmación de esta guía contra su fuente.

> [!note] Este tomo se mantiene solo si los demás lo alimentan
> Cada tomo nuevo trae su propio "Glosario express". Al escribirlo, sus términos deben sumarse al índice de la sección 7 — y si alguno **colisiona** con un término existente (como pasó con `top_k`), va a la sección 4. Esa sección no se planificó: se descubrió al consolidar.

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13 · ⭐ Complemento: frameworks]] · Siguiente → [[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 · Bibliografía]]
