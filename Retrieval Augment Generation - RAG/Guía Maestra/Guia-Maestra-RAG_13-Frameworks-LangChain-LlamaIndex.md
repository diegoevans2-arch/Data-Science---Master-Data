---
title: "Tomo 13 — ⭐ Frameworks de orquestación: LangChain, LlamaIndex, DSPy y la opción de no usar ninguno"
tags: [rag, complemento, vanguardia, frameworks, langchain, langgraph, llamaindex, dspy, haystack, ragflow, abstraccion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 13
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 13 — ⭐ Frameworks de orquestación: LangChain, LlamaIndex, DSPy y la opción de no usar ninguno

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12 · ⭐ Técnicas avanzadas de query]] · Siguiente → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]

---

> [!danger] 🚨 Dos avisos antes de empezar
> **① Este tomo no tiene fuente primaria.** Como el [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12]], se construye con bibliografía externa verificada antes de escribir. Lo que no se pudo confirmar está marcado como tal (§10.4).
>
> **② Es el tomo que más rápido envejece de la guía**, y está diseñado para eso. Se separa en dos capas:
> - **Lo durable** (§1–§4, §9–§10): qué abstrae un framework, qué cuesta, cómo decidir. Esto sigue valiendo dentro de tres años.
> - **Lo perecedero** (§5–§8): versiones, APIs y código, **en cajas fechadas**. Verifica contra la documentación oficial antes de copiar nada.
>
> Ya tenemos precedente de por qué importa: el endpoint `llama-3-1-8b-instruct-turbo` que usa el curso **ya no aparece** en el catálogo de Together AI ([[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 §13.1]]).

> [!info] ¿Por qué importa este tomo?
> Porque la decisión de usar o no un framework se toma **al principio del proyecto**, cuando menos información tienes, y **condiciona todo lo demás**: cómo depuras, cómo instrumentas, cuánto control tienes cuando algo se rompe, y cuánto trabajo cuesta cambiar de idea.
>
> Es una decisión de arquitectura disfrazada de decisión de librería.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** elegir con criterio entre velocidad inicial y control posterior; entender por qué un equipo pide "reescribir sin el framework" a los doce meses; presupuestar el costo real de una dependencia que rota su API.
> - **Costo o riesgo de hacerlo mal:** adoptar una capa que acelera el prototipo y **estorba en producción** — un patrón documentado por quienes lo vivieron. O lo contrario: rechazar toda abstracción y reimplementar mal lo que ya estaba resuelto.
> - **Pregunta que responde:** *"¿necesitamos un framework, y qué nos cuesta si nos equivocamos?"*

---

## 1. 🎯 La pregunta correcta no es cuál, sino si

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un framework es una **cocina industrial montada**: llegas y ya está todo — fogones, extractor, cámaras, el orden de las estaciones. Empiezas a cocinar en media hora en vez de en tres meses.
>
> El precio aparece el día que quieres hacer algo que la cocina no previó. Mover un fogón implica tocar la instalación de gas. Y si el fabricante rediseña el modelo, tu cocina deja de tener repuestos.
>
> **La pregunta no es qué cocina comprar. Es si tu restaurante necesita una cocina llave en mano o una diseñada para lo que tú cocinas.**

### 1.1 El dato con el que hay que abrir

Los once tomos anteriores de esta guía documentan un curso profesional de RAG de punta a punta: retrieval, chunking, hybrid search, re-ranking, generación, evaluación, producción y multimodal. Se verificó qué frameworks usa su material:

```
   Imports de langchain / llama_index / haystack / dspy   →   0
   Menciones en notebooks, utils.py y transcripciones     →   0

   Lo que SÍ usa:  weaviate · openai · together · sentence_transformers
```

> [!important] 🎯 Cero. Ni un import, ni una mención
> Un curso completo de RAG **construyó todo el pipeline con clientes directos** y nunca necesitó nombrar un framework de orquestación. No es que los descarte: **no aparecen en su horizonte**.
>
> Eso encuadra el tomo. La pregunta por defecto no es *"¿cuál elijo?"* sino ***"¿necesito uno?"*** — y para aprender y construir el pipeline, la respuesta demostrada es que no.
>
> Lo cual **no significa que nunca sirvan.** Significa que la carga de la prueba está del lado de adoptarlos, no del lado de evitarlos.

### 1.2 La recomendación de mayor autoridad disponible dice lo mismo

**Anthropic**, en *Building effective agents* (diciembre de 2024) — la misma fuente que el [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]] usa para los patrones agentic:

> *"These frameworks make it easy to get started by simplifying standard low-level tasks like calling LLMs, defining and parsing tools, and chaining calls together. However, **they often create extra layers of abstraction that can obscure the underlying prompts and responses, making them harder to debug.**"*

Y la recomendación explícita:

> *"**We suggest that developers start by using LLM APIs directly**: many patterns can be implemented in a few lines of code. If you do use a framework, ensure you understand the underlying code."*

> [!note] Matiz honesto: no es una postura anti-framework
> El mismo artículo lista frameworks recomendados. La tesis real es **"empieza sin, añade si duele"** — y el corolario, que es lo accionable: *asegúrate de entender el código que hay debajo*.

### 1.3 ⚖️ La asimetría probatoria

Al buscar evidencia en ambas direcciones aparece un desequilibrio que conviene nombrar:

| | Quién lo sostiene |
|---|---|
| **En contra** de adoptar por defecto | Un **laboratorio frontera** (Anthropic) · quien **midió** interceptando llamadas API (Husain, 2024) · un equipo que los usó **más de 12 meses en producción y los retiró** (Octomind, 2024) |
| **A favor** | Mayoritariamente **quien vende** el framework, o servicios sobre él. El argumento más sólido no-vendor es de *time-to-market*, no de calidad |

> [!warning] ⚠️ Cómo leer esto sin caer en el otro extremo
> La asimetría **no convierte a los frameworks en malos**. Significa que las críticas vienen de gente con incentivo a callarse y experiencia directa, mientras los elogios vienen mayoritariamente de gente con incentivo a hablar. Eso cambia el peso probatorio, no el veredicto.
>
> Y hay una razón estructural para el desequilibrio: **nadie escribe un post titulado "seguimos usando LangChain y nos va bien"**. El sesgo de publicación juega en contra del framework.

---

## 2. 🔍 Qué abstrae realmente un framework

Audiencia: 🔧 🧭

Antes de decidir, conviene saber qué se compra exactamente. Un framework de orquestación cubre cuatro cosas — y **solo la primera es genuinamente tediosa de hacer a mano**:

| Capa | Qué te da | ¿Cuánto ahorra de verdad? |
|---|---|---|
| **① Integraciones** | Conectores a decenas de vector DBs, proveedores de LLM, loaders de formatos | **Mucho, si usas muchas.** Poco si usas dos |
| **② Utilidades** | Splitters de texto, parsers de salida, gestión de memoria | Medio. Suelen ser 30–80 líneas cada una |
| **③ Orquestación** | Encadenar pasos, ramificar, reintentar, mantener estado | **Poco.** En Python eso son funciones y `if` |
| **④ Patrones prefabricados** | "Chains" y agentes listos para usar | **El más atractivo al empezar y el más problemático después** |

> [!important] 🎯 Dónde gotea siempre la abstracción
> Las capas ① y ② envejecen bien: son plomería que rara vez quieres reescribir. **Las capas ③ y ④ son las que producen los arrepentimientos**, porque son justo donde tu caso se vuelve particular.
>
> El patrón que se repite en todos los relatos de retirada es el mismo: *el prefabricado te lleva al 80 % en un día, y el 20 % restante requiere bajar de nivel — pero la abstracción está diseñada precisamente para que no bajes.*
>
> Lo dijo el CEO de LangChain sobre su propio producto (§4.2): *"the same high level interfaces… **were now getting in the way** when people tried to customize them to go to production."*

---

## 3. 💸 El costo de la abstracción, medido

Audiencia: 🔧 🧭 👔

La crítica de que "los frameworks añaden overhead" suele quedarse en impresión. Existe **una medición sólida**, y viene del lugar menos sospechoso: **el propio vendor midiéndose a sí mismo.**

> [!danger] 🚨 LangChain mide que su propio harness gastaba 5.395 tokens por turno
> Del changelog oficial de LangChain, entrada de `deepagents` v0.7.0b2 (24 de julio de 2026):
>
> *"A leaner, more configurable harness by default. On a default-agent turn, **input tokens drop 65 % (5.395 → 1.895)** vs. v0.6.12, validated against our revamped evaluation suite **with no quality regression**."*
>
> *"Isolated to the default agent's tool schemas, **total description tokens drop 43 % (4.005 → 2.302)**."*
>
> Léelo despacio: el framework consumía **5.395 tokens de entrada por turno** en sus propios prompts y esquemas — y **~3.500 de ellos eran eliminables sin perder calidad**.
>
> Esos tokens los pagabas tú, en cada llamada, sin verlos. Es exactamente lo que el [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 §2]] enseña a vigilar: **cada token cuesta dinero y latencia**.

**🔧 El argumento metodológico complementario.** Hamel Husain (2024) hizo algo simple y demoledor: **interceptar las llamadas a la API** para ver los prompts que los frameworks envían realmente. Su tesis: *"the prompts sent by these tools to the LLM… is the fastest way to understand how they work"*.

Hallazgos concretos: llamadas redundantes (una cadena "inteligente" que necesitaba cuatro), prompts verbosos, y **DSPy generando cientos de llamadas y más de 30 minutos** durante la optimización — que en ese caso no es un bug, es el costo del paradigma (§7).

> [!note] Su recomendación no es "no uses frameworks"
> Es **"exige ver el prompt antes de confiar"**. Y es una regla operativa excelente: si no puedes inspeccionar fácilmente lo que tu framework envía al modelo, no puedes depurar ni el costo ni la calidad.

> [!warning] ⚠️ Lo que NO se puede afirmar
> **No existe ningún benchmark independiente publicado** que mida latencia o tokens de LangChain (o cualquiera) contra un equivalente escrito a mano. Las únicas cifras de overhead que existen son **las que el propio vendor publicó sobre sí mismo**.
>
> Si alguien te dice *"LangChain añade un X % de latencia"*, ese número **no está publicado por nadie**.

---

## 4. 🔄 La rotación de APIs: el costo que no se presupuesta

Audiencia: 🔧 🧭

Esta es la objeción más citada contra LangChain y merece tratarse con datos, no con anécdotas.

### 4.1 El historial, con fechas

| Hito | Fecha | Qué rompió |
|---|---|---|
| **0.1.0** | ene 2024 | Nacen `langchain-core` y `langchain-community`. Nueva política: los *breaking* suben la minor |
| **0.2.0** | may 2024 | Desacople `langchain` ↔ `langchain-community` |
| *(oct 2024)* | — | *"Most chains and agents were **marked as deprecated**"* en favor de LangGraph |
| **0.3.0** | sep 2024 | **Pydantic 1 → 2 forzado**, sin puentes. Se cae Python 3.8 |
| **1.0.0** | oct 2025 | Todo lo legacy se mueve a `langchain-classic`; nace `create_agent`; se cae Python 3.9 |

**🔧 Y hay dos señales adicionales del mismo período:**

- **`langchain-community` fue descontinuado** (anuncio 22 de mayo de 2026; repositorio archivado el 19 de junio de 2026). Razones oficiales: creció demasiado con integraciones de calidad dispar y el ciclo de release compartido impedía iterar. **No se declaró ventana de mantenimiento ni de seguridad.**
- La versión **1.3.5 fue retirada (*yanked*)** por *"compatibility regression with first-party Deep Agents summarization integration"* — es decir, **una regresión entre dos paquetes del propio proyecto**.

### 4.2 El vendor lo reconoce por escrito

Es lo que da credibilidad a la crítica. Harrison Chase, CEO de LangChain, en *Reflections on Three Years of Building LangChain* (octubre de 2025):

> *"While `langchain` was the fastest place to get started, **we traded power for ease of use**. The same high level interfaces in `langchain` that made it easy to get started **were now getting in the way** when people tried to customize them to go to production."*

Y la lista de problemas que asume como propios:

> *"Some problems we could fix: like **preventing breaking changes, making hidden prompts explicit, package bloat, dependency conflicts, outdated documentation**."*

Antes, respondiendo en público a la crítica de Octomind (junio de 2024) — a la que calificó de *"level-headed and precise"*:

> *"The initial version of LangChain was pretty high level and **absolutely abstracted away too much**. We're moving more and more to low level abstractions."*

### 4.3 Sí hay política de versionado — con tres huecos

Existe una política declarada: *breaking changes* solo en versiones mayores, y lo deprecado sigue funcionando durante toda la serie 1.x. Es una mejora real. Pero conviene leer la letra chica:

> [!warning] ⚠️ Los tres huecos de la promesa de estabilidad
> **① Cubre solo el núcleo.** La política aplica a `langchain` y `langchain-core` más los partner packages propios. Textual: *"Other partner packages **may follow different stability and versioning policies**."*
>
> **② La capa que te recomiendan usar primero está fuera de semver.** La documentación oficial dice empezar por **Deep Agents** — y su propia página de versionado lo describe como *"a pre-1.0 package under active development"* donde *"the API **may change between minor versions**"*. Y se cumplió: `deepagents` v0.7.0b2 trajo **tres breaking changes en un salto de minor**.
>
> **③ `langchain-community` nunca estuvo cubierto** — y ahora está archivado.

**🧭 La lectura práctica:** la promesa de estabilidad es real **para el núcleo**, y no cubre la capa donde vive la mayor parte del código de aplicación de un usuario nuevo. No es letra muerta, pero tampoco es lo que uno entiende al leer "tenemos política de versionado".

---

## 5. 📦 El panorama hoy

Audiencia: 🔧 🧭

> [!warning] ⚠️ SECCIÓN PERECEDERA — datos verificados el 28–29 de julio de 2026
> Todo lo que sigue en §5–§8 caduca. Las versiones y APIs **cambian en semanas**. Verifica contra la documentación oficial antes de usar cualquier dato de aquí. Lo durable del tomo está en §1–§4 y §9–§10.

| Proyecto | Versión | Licencia | ★ | Posicionamiento declarado |
|---|---|---|---|---|
| **LangChain** | 1.3.14 (16 jul 2026) | MIT | 142.830 | *"The agent engineering platform"* |
| **LangGraph** | 1.2.10 (28 jul 2026) | MIT | ~38,4k | Runtime de orquestación de bajo nivel |
| **RAGFlow** | — | Apache-2.0 | **86.282** | Motor RAG desplegable (no librería) |
| **LlamaIndex** | 0.14.23 (24 jun 2026) | MIT | 51.178 | *"Document agent and **OCR platform**"* |
| **DSPy** | 3.2.1 (5 may 2026) | **MIT** | 36.444 | *"Program, don't prompt"* |
| **Haystack** | **3.0.0** (20 jul 2026) | Apache-2.0 | 26.048 | Orquestación *production-ready* |
| **Semantic Kernel** | — | MIT | 28.380 | Microsoft; fuerte en .NET/enterprise |

> [!note] Dos observaciones que las comparativas suelen omitir
> **① RAGFlow es el elefante ausente.** Con 86.282 estrellas supera a LlamaIndex, Haystack, DSPy y LangGraph **juntos**, y falta en casi todas las comparativas anglosajonas. La razón es categorial: **es un motor desplegable, no una librería que importas** — no compite en el mismo eje, pero si tu pregunta es "quiero RAG funcionando", es una respuesta legítima que nadie te menciona.
>
> **② Haystack es el más viejo (2019) y el de disciplina de release más visible:** el único con versionado mayor serio (2.0 en 2024 → 3.0 en 2026) y **114 issues abiertas** frente a 588 de LlamaIndex y 613 de DSPy. Poca estrella, mucha ingeniería.

---

## 6. 🦜 LangChain hoy

Audiencia: 🔧

**🔧 La estructura del ecosistema** (julio de 2026):

```
   deepagents          harness "batteries-included"  ← te recomiendan empezar aquí
        │                                              (y está FUERA de semver, §4.3)
   langchain           create_agent, init_chat_model, tools
        │
   langgraph           runtime: estado, durabilidad, human-in-the-loop
        │
   langchain-core      mensajes, modelos, prompts, Runnable, retrievers
        │
   langsmith (SDK)     ← DEPENDENCIA OBLIGATORIA de langchain-core
```

> [!warning] ⚠️ Un detalle de acoplamiento que conviene conocer
> **`langsmith` es dependencia obligatoria de `langchain-core`.** No puedes instalar LangChain sin el cliente de su plataforma comercial de observabilidad. No se activa solo —hace falta configurar las variables de entorno— pero el paquete viaja contigo.

### 6.1 🔎 Qué pasó con LCEL

LCEL —el *LangChain Expression Language*, encadenar componentes con el operador `|`— fue durante dos años **la** forma de escribir LangChain. Hoy:

| Evidencia | Resultado |
|---|---|
| Sitemap oficial (548 URLs) | **0 URLs contienen "lcel"** |
| Página conceptual `/docs/concepts/lcel` | **308 Permanent Redirect** al overview |
| Guía de migración a v1 (32.402 caracteres) | **0 menciones de "LCEL"** |
| Clases `Runnable*` en `langchain-core` 1.x | **Presentes y sin marca de deprecación**; el operador `\|` sigue documentado |

> [!important] 🎯 La lectura correcta: no fue deprecado, fue **retirado como modelo mental**
> El `Runnable` sigue vivo y es el sustrato de todo. Lo que desapareció es el *pitch*: "compón tu app encadenando con `|`" ya no es lo que LangChain enseña. **Los Runnables pasaron de ser la interfaz de autor a ser plomería interna.**
>
> ⚠️ *No encontré una declaración oficial explícita sobre el estatus de LCEL — la evidencia es de **ausencia documental**, no una declaración. Los artículos que afirman "LCEL sigue vigente para cadenas simples" son content-farms sin fuente.*

### 6.2 💻 El código oficial de RAG hoy — y lo que revela

Este es el ejemplo actual de RAG en dos pasos de la documentación oficial (consultado el 29 de julio de 2026). Está reproducido casi literal, con los comentarios originales:

```python
from langchain_openai import ChatOpenAI
from langsmith import traceable

llm = ChatOpenAI(model="...", temperature=1)   # ← el ID de modelo, el tuyo

@traceable()                                    # ← opcional: solo tracing
def rag_bot(question: str) -> dict:
    # LangChain retriever will be automatically traced
    docs = retriever.invoke(question)
    docs_string = "".join(doc.page_content for doc in docs)

    instructions = f"""You are a helpful assistant who is good at analyzing source information and answering questions.
       Use the following source documents to answer the user's questions.
       If you don't know the answer, just say that you don't know.
       Use three sentences maximum and keep the answer concise.

<context>
{docs_string}
</context>"""

    # langchain ChatModel will be automatically traced
    ai_msg = llm.invoke([
        {"role": "system", "content": instructions},
        {"role": "user", "content": question},
    ])
    return {"answer": ai_msg.content, "documents": docs}
```

> [!important] 🎯 Fíjate en lo que **no** hay
> **Ni `PromptTemplate`, ni el operador `|`, ni `RetrievalQA`, ni LCEL.** El prompt es un **f-string** y la orquestación es **Python plano**.
>
> Este es el patrón oficial de 2026, escrito por LangChain. Y dice algo importante: **en su propio ejemplo de RAG, LangChain aporta el retriever y el wrapper del modelo — no la orquestación.** Que es, exactamente, la capa ③ de la tabla de §2: la que menos ahorra.
>
> Si quitas el decorador `@traceable`, este código es indistinguible de uno escrito sin framework, salvo por dos imports.

> [!note] 🐛 Un detalle sobre el estado de la documentación
> Las docs de LangChain 1.x **ya no publican una chain mínima de RAG**. La página oficial de retrieval tiene una sección "2-step RAG" con **solo un diagrama y dos enlaces, cero código**; el único ejemplo completo vive en un tutorial de evaluación. Y una tarjeta de enlace afirma que otro tutorial *"incluye un workflow mínimo de retrieve-then-generate"* — **no lo incluye**.
>
> Es coherente con lo que el propio CEO listó como problema pendiente: *"outdated documentation"*.

---

## 7. 🧭 LlamaIndex, DSPy y los demás

Audiencia: 🔧 🧭

### 7.1 LlamaIndex: ya no es lo que su fama dice

Se conoce como *"el data framework"*, especializado en ingesta e indexación frente a la orquestación de LangChain. **Ese posicionamiento está desactualizado.** Hoy su propio repositorio se describe como *"the leading **document agent and OCR platform**"*, LlamaCloud se renombró a **LlamaParse**, y su documentación **ya no enseña RAG puro como caso base**: enseña un agente con RAG como herramienta.

**🧭 La lectura de negocio:** la capa open-source es hoy la puerta de entrada a un producto comercial de **parsing y OCR documental**. La diferencia real con LangChain no es técnica sino de modelo: **LlamaIndex monetiza el parsing de documentos; LangChain monetiza orquestación y observabilidad.**

> [!warning] ⚠️ Y sigue pre-1.0 tras casi cuatro años
> Versión 0.14.23, con la cadencia de releases desacelerando. Un `0.x` de cuatro años no es necesariamente inestable, pero **no ofrece ninguna garantía formal de compatibilidad**.

**🔧 Nota estructural:** `llama-index` es un **meta-paquete**, no el framework. El framework real es `llama-index-core` más **300+ paquetes de integración** publicados por separado.

### 7.2 DSPy: el único que propone otra cosa

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Los demás frameworks te dan **mejores herramientas para escribir el prompt a mano**. DSPy te dice: *deja de escribirlo*. Tú declaras **qué entra y qué sale**, das ejemplos y una métrica, y un **compilador** busca el prompt que maximiza esa métrica.
>
> Es la diferencia entre un buen editor de texto y un compilador.

**🔧 Definición técnica**, del abstract del paper:

> *"existing LM pipelines are typically implemented using hard-coded 'prompt templates', i.e. **lengthy strings discovered via trial and error**"*

> *"we introduce DSPy, a programming model that abstracts LM pipelines as text transformation graphs… DSPy modules are **parameterized**, meaning they can learn… **We design a compiler that will optimize any DSPy pipeline to maximize a given metric.**"*

> [!important] 🎯 Es el único de la familia con validación académica de primer nivel
> - **DSPy** — Khattab et al., **ICLR 2024 (Spotlight)**
> - **MIPRO**, su optimizador — **EMNLP 2024**
> - **GEPA**, la evolución — **ICLR 2026 (Oral)**
> - Y un predecesor con linaje: **DSP** (2022), el mismo grupo
>
> Frente a LangChain, LlamaIndex y Haystack, que son **productos de ingeniería sin paper**. Esa diferencia importa: lo que está validado en DSPy es el **método de optimización de prompts**, no el framework como producto.

**🔧 La evidencia de que funciona, y su límite:** el paper reporta que unas pocas líneas de DSPy permiten a modelos superar el few-shot estándar *"generally by over 25 %"* y las cadenas escritas por expertos *"by up to 5-46 %"*.

> [!danger] 🚨 Dos advertencias imprescindibles sobre esas cifras
> **① Son de 2023.** Se midieron con GPT-3.5 y llama2-13b — modelos que no razonaban paso a paso de fábrica. **Nadie ha replicado esas ganancias con modelos frontera actuales**, que ya hacen chain-of-thought nativo.
>
> **② Y hay un hallazgo incómodo desde dentro.** Un trabajo de 2026 que integra DSPy en un benchmark grande reporta que las mejoras vienen *"mostly from introducing **chain-of-thought**, and **little additional benefit from more advanced optimizers**"*.
>
> Lo notable: **dos de sus coautores son autores del paper original de DSPy.** Eso hace el hallazgo **más** fuerte, no menos — es una admisión, no un ataque.
>
> Traducido: buena parte del beneficio atribuido a la optimización automática podría ser, simplemente, que te obliga a usar chain-of-thought.

**🧭 Cuándo tiene sentido:** cuando tienes una **métrica automatizable** y un conjunto de ejemplos, y el problema es de *calidad de prompt* más que de plomería. **Cuándo no:** cuando no puedes definir la métrica — sin ella el compilador no tiene qué optimizar. Y recuerda el costo medido: **cientos de llamadas y más de 30 minutos** por compilación.

### 7.3 Los demás, en breve

- **Haystack** — el más veterano y el de mejor disciplina de ingeniería. Su versión 3.0 (julio de 2026) mueve el centro a los agentes, y su ángulo declarado es la **soberanía**: *"a loop you can read, context you can keep, and a model you can replace"*. Buena opción si te pesan el control y la trazabilidad.
- **RAGFlow** — otra categoría: un **motor desplegable**, no una librería. Si la pregunta es "quiero un RAG funcionando sin construirlo", es una respuesta seria.
- **Semantic Kernel** — la opción natural en ecosistemas .NET y Microsoft.

---

## 8. 🔭 ¿Un framework ayuda o estorba a la observabilidad?

Audiencia: 🔧 🧭

Es la pregunta que conecta este tomo con el [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]], y la respuesta tiene dos mitades que conviene no mezclar.

**✅ Capturar las trazas es objetivamente más fácil con framework.** Los frameworks tienen callbacks internos, así que la instrumentación es automática:

| Destino | Trabajo necesario |
|---|---|
| **LangSmith** (propio, cerrado) | **2 variables de entorno, cero código**. Los comentarios oficiales lo dicen: *"LangChain retriever will be automatically traced"* |
| **Arize Phoenix / OpenInference** | **Una línea**: `LangChainInstrumentor().instrument()` |
| **Código a mano** | Un decorador o un span **por función** ([[Guia-Maestra-RAG_10-RAG-en-Produccion\|Tomo 10 §4]]) |

**⚠️ Interpretarlas es plausiblemente más difícil — pero no está medido.** La instrumentación de LangChain captura *"la jerarquía de runnables que los envuelve"*: obtienes más spans, pero muchos son de la maquinaria del framework, no de tu lógica.

> [!warning] ⚠️ Aquí hay que ser honesto sobre lo que no se sabe
> **No encontré ninguna medición publicada** de profundidad de traza o relación señal/ruido comparando una app con framework contra una escrita a mano. El argumento de "las trazas de LangChain son ruidosas" es **plausible y coherente con su arquitectura, pero no está medido**. No lo presentes como hecho.

> [!note] 📌 Dos correcciones de precisión sobre herramientas del Tomo 10
> - **Las convenciones semánticas GenAI de OpenTelemetry (`gen_ai.*`) NO son estables.** Su documentación las marca literalmente como **`Status: Development`** a julio de 2026. Afirmar que existe "un estándar estable de tracing para GenAI" es falso hoy.
> - **Arize Phoenix se distribuye bajo licencia Elastic-2.0**, que es *source-available* y **no está aprobada por la OSI**. El curso —y el [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]] siguiéndolo— la describe como "open-source". El instrumentor `openinference-instrumentation-langchain` sí es Apache-2.0. La distinción importa si tu organización tiene política de licencias.

---

## 9. 🧭 Guía de decisión

Audiencia: 🧭 👔

> [!danger] 🚨 Y el hallazgo que enmarca toda esta sección
> **No existe ninguna evaluación comparativa revisada por pares de LangChain, LlamaIndex, DSPy y Haystack.** Buscado en arXiv, OpenAlex, Semantic Scholar y MDPI.
>
> La elección de framework se toma hoy sobre **evidencia anecdótica, comparativas de blog no reproducibles y afinidad arquitectónica**. Cualquiera que te presente un ranking objetivo está extrapolando.
>
> Eso no paraliza la decisión — la reorienta: como no hay un ganador demostrable, **el criterio correcto no es "cuál es mejor" sino "cuál encaja con mi caso y cuánto me cuesta salir de él"**.

```
   ¿USAR UN FRAMEWORK?

   1. ¿Estás aprendiendo o prototipando?
        → Empieza SIN. El curso entero de esta guía se hizo así.
          Entenderás qué hace cada pieza antes de que algo la esconda.

   2. ¿Cuántas integraciones distintas necesitas de verdad?
        Muchas (varios vector DBs, muchos formatos) → el framework paga (capa ①)
        Dos o tres → escríbelas. Son menos código del que crees.

   3. ¿Tu pipeline es estándar o particular?
        Estándar → los prefabricados te ahorran semanas
        Particular → 🚨 es justo donde la abstracción estorba.
                     Es el modo de fallo documentado.

   4. ¿Puedes VER el prompt que se envía al modelo?
        NO → resuélvelo antes de seguir. Sin eso no depuras
             ni el costo ni la calidad.

   5. ¿Tu problema es de plomería o de calidad de prompt?
        Plomería → LangChain / LlamaIndex / Haystack
        Calidad de prompt, y TIENES una métrica automatizable → DSPy
        (sin métrica, DSPy no tiene qué optimizar)

   6. ¿Quieres RAG funcionando, no construirlo?
        → mira un motor desplegable (RAGFlow), no una librería.

   7. Antes de casarte: ¿cuánto cuesta SALIR?
        Cuenta cuántos archivos importarían el framework.
        Si es tu capa de orquestación entera, el costo de salida
        es una reescritura.
```

> [!important] 🎯 La regla que resume el tomo
> **Usa un framework por sus integraciones, no por su orquestación.**
>
> Las capas ① y ② (conectores y utilidades) son plomería que no quieres reescribir y envejece bien. Las capas ③ y ④ (orquestación y prefabricados) son las que producen arrepentimientos — y son, precisamente, las que en Python cuestan menos hacer a mano.
>
> Es lo que muestra el propio código oficial de RAG de LangChain en §6.2: **aporta el retriever y el wrapper del modelo; la orquestación es Python plano.**

> [!example] 📊 Caso de negocio — Energía
> **Problema.** Una distribuidora eléctrica construye un asistente interno sobre normativa técnica, procedimientos de mantenimiento y protocolos de seguridad. El equipo —tres personas, ninguna especialista en IA— arranca con un framework porque *"así no reinventamos la rueda"*. En seis semanas tienen un prototipo que funciona y todo el mundo está contento.
>
> Los problemas llegan al pasar a producción. Primero, **el costo por consulta es el doble de lo estimado** y nadie sabe explicar por qué: el prompt real que se envía al modelo no es visible en el código, lo construye el framework. Segundo, seguridad exige que ciertos procedimientos **solo sean accesibles a personal certificado** — y encajar ese control en la cadena prefabricada obliga a pelearse con la abstracción, porque el punto donde hay que interceptar está enterrado. Tercero, una actualización menor de una dependencia rompe el pipeline durante dos días.
>
> **Técnica aplicada.** En vez de la reescritura completa que pide el equipo, se hace la separación de §2: **se conserva el framework para las capas ① y ②** —los conectores al vector store y los splitters, que funcionan bien y nadie quiere mantener— y **se reemplaza la capa ③ por Python plano**: cuatro funciones (`retrieve`, `format_context`, `build_prompt`, `generate`) encadenadas en un `def` de quince líneas. El prompt pasa a ser un f-string visible en el repositorio. El control de acceso se resuelve con tenants separados ([[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 §6.2]]), que es donde correspondía.
>
> **Resultado.** Al hacerse visible el prompt aparece el origen del sobrecosto —contexto que el framework añadía y nadie había pedido— y el costo por consulta baja. El control de acceso deja de ser un parche. Y el equipo conserva lo que el framework hacía bien.
>
> La lección transferible: **la decisión no era "framework sí o no", era "en qué capa".** Plantearla como todo o nada llevaba a una reescritura innecesaria o a seguir peleando con la abstracción equivocada.

---

## 10. 📖 Glosario, checklist y referencias

### 10.1 Glosario express

| Término | Definición operativa |
|---|---|
| **Framework de orquestación** | Librería que encadena las piezas de un pipeline LLM. Cubre integraciones, utilidades, orquestación y patrones prefabricados |
| **LCEL** | *LangChain Expression Language*: componer con el operador `\|`. No deprecado, pero **retirado como modelo mental** |
| **Runnable** | La abstracción base de LangChain. Hoy es plomería interna, no la interfaz de autor |
| **LangGraph** | Runtime de orquestación de bajo nivel de LangChain: estado, durabilidad, human-in-the-loop |
| **`create_agent`** | La API central de LangChain 1.x, sobre el runtime de LangGraph |
| **Partner package** | Paquete de integración por proveedor (`langchain-openai`, etc.). **Puede seguir otra política de versionado** |
| **Compilador de prompts** | El paradigma de DSPy: optimizar automáticamente instrucciones y ejemplos contra una métrica |
| **Teleprompter / optimizador** | En DSPy, el algoritmo que hace esa optimización (MIPROv2, GEPA, BootstrapFewShot…) |
| **Motor RAG desplegable** | Producto que se instala y opera (RAGFlow), frente a una librería que se importa |
| **Source-available** | Código visible pero con licencia **no aprobada por la OSI** (ej. Elastic-2.0 de Phoenix). No es lo mismo que open source |
| **Fuga de abstracción** | Cuando hay que bajar al nivel que la abstracción oculta — y está diseñada para que no bajes |

### 10.2 Checklist de comprensión

- [ ] Sé que el curso completo de esta guía se construyó **sin ningún framework** — cero imports, cero menciones.
- [ ] Puedo distinguir las cuatro capas de un framework y **cuál de ellas produce los arrepentimientos**.
- [ ] Conozco la única medición sólida de overhead que existe: **5.395 → 1.895 tokens por turno**, publicada por el propio vendor.
- [ ] Sé que **no existe benchmark independiente** de overhead de framework vs. código a mano.
- [ ] Puedo relatar el historial de rupturas de API de LangChain con fechas.
- [ ] Sé que la política de estabilidad **no cubre la capa que te recomiendan usar primero**.
- [ ] Entiendo qué pasó con LCEL: no deprecado, **retirado como modelo mental**.
- [ ] Sé que el ejemplo oficial de RAG de LangChain **no usa LCEL ni `PromptTemplate`** — es un f-string y Python plano.
- [ ] Sé que LlamaIndex **pivotó** a plataforma de OCR y sigue pre-1.0 tras casi cuatro años.
- [ ] Entiendo por qué DSPy es **categorialmente distinto**, y por qué sus cifras de 2023 no son extrapolables.
- [ ] Conozco el hallazgo de que las ganancias de DSPy vienen sobre todo del **chain-of-thought**, no de los optimizadores.
- [ ] Sé que **no existe evaluación comparativa peer-reviewed** de los cuatro frameworks.
- [ ] 🚨 Antes de adoptar: **puedo ver el prompt que se envía al modelo**, y **sé cuánto costaría salir**.

### 10.3 Referencias

> [!note] Verificadas el 28–29 de julio de 2026, antes de escribir
> Fichas completas en el [[Guia-Maestra-RAG_16-Bibliografia|Tomo 16 · Bibliografía]].

**Académicas** — solo DSPy tiene linaje publicado
- Khattab, O., Singhvi, A., Maheshwari, P., Zhang, Z., Santhanam, K., Vardhamanan, S., Haq, S., Sharma, A., Joshi, T. T., Moazam, H., Miller, H., Zaharia, M., & Potts, C. (2024). "DSPy: Compiling Declarative Language Model Calls into State-of-the-Art Pipelines". *ICLR 2024 (Spotlight)*. arXiv:2310.03714. ✅ ⚠️ *Divergencia real de título: el arXiv (nunca actualizado desde la v1) dice **"Self-Improving Pipelines"**; el camera-ready de ICLR dice **"State-of-the-Art Pipelines"**. El BibTeX oficial del proyecto mezcla ambos.*
- Khattab, O., Santhanam, K., Li, X. L., Hall, D., Liang, P., Potts, C., & Zaharia, M. (2022). *Demonstrate-Search-Predict*. arXiv:2212.14024. ✅ *Preprint. El predecesor.*
- Opsahl-Ong, K., Ryan, M. J., Purtell, J., Broman, D., Potts, C., Zaharia, M., & Khattab, O. (2024). "Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs" (MIPRO). *EMNLP 2024*. arXiv:2406.11695. ✅
- Agrawal, L. et al. (2026). "GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning". *ICLR 2026 (Oral)*. arXiv:2507.19457. ✅
- Sarmah, B., Dutta, S., Grigoryan, A., Tiwari, M., Pasquali, S., & Mehta, D. (2024). *A Comparative Study of DSPy Teleprompter Algorithms*. arXiv:2412.15298. ✅ *Preprint, muestra pequeña, pero **independiente**: ninguno de sus autores figura en el paper de DSPy.*
- Aali, A. et al. (2026). *Structured Prompts Improve Evaluation of Language Models*. arXiv:2511.20836. ✅ ⚠️ ***NO es evaluación independiente**: dos coautores (Singhvi, Potts) son autores del paper original de DSPy. Fuente del hallazgo de que las ganancias vienen sobre todo del chain-of-thought.*

**Posturas con autoridad**
- Schluntz, E., & Zhang, B. (2024, 19 de diciembre). *Building effective agents*. Anthropic. ✅ *La fuente de mayor autoridad del tomo: recomienda empezar con las APIs directas.*
- Chase, H. (2025, 20 de octubre). *Reflections on Three Years of Building LangChain*. ✅ *Donde el vendor reconoce por escrito los problemas de sobre-abstracción.*
- Husain, H. (2024, 14 de febrero). *"Fuck You, Show Me The Prompt."* ✅ *El argumento metodológico: intercepta las llamadas y enseña los prompts reales.*
- Octomind (2024, ~20 de junio). *Why we no longer use LangChain for building our AI agents*. ⚠️ **`octomind.dev` no resolvió DNS el 29 de julio de 2026.** *El contenido y la discusión se conservan en el hilo de Hacker News (id 40739982, 480 puntos), donde además respondió el CEO de LangChain. **Citar el hilo, no la URL original.***

**Documentación oficial** (perecedera — §5–§8)
- LangChain: changelog (medición de `deepagents` v0.7.0b2), guía de migración a v1, política de versionado y release, tutorial de evaluación de LangSmith, issue de descontinuación de `langchain-community`. ✅
- LlamaIndex, DSPy, Haystack, RAGFlow: repositorios y PyPI. ✅
- OpenTelemetry: convenciones semánticas GenAI — **`Status: Development`**. ✅

### 10.4 🔴 Lo que NO se pudo verificar, y por tanto el tomo no afirma

Por transparencia, y porque son justo las cosas que suelen afirmarse sin base:

1. **Ningún benchmark independiente** de overhead de latencia o tokens de framework vs. código a mano. Las únicas cifras son del propio vendor.
2. **Ninguna medición** de profundidad de traza o relación señal/ruido. La crítica de "trazas ruidosas" es plausible, **no medida**.
3. **Ninguna evaluación comparativa peer-reviewed** de los cuatro frameworks (§9).
4. Precios, tiers y condiciones de self-hosting de **LangSmith**.
5. Si se puede **exportar trazas desde LangSmith** hacia un collector externo.
6. El **post original de Octomind** (DNS caído).
7. Una **declaración oficial explícita** sobre el estatus de LCEL — la evidencia es de ausencia documental.
8. Las cifras de adopción empresarial de DSPy (*self-reported*, sin verificación externa) y el claim de *"2× más rápido"* de un caso de cliente — blog de vendor coescrito por un contribuidor del proyecto, **sin metodología**.

> [!danger] 🚨 Fuentes descartadas a propósito
> Se descartó un volumen considerable de contenido tipo *"LangChain vs LangGraph 2026"* de granjas de contenido y agregadores con pinta de generados por IA: afirmaciones confiadas, **cero fuentes**, y en algunos casos comillas presentadas como citas oficiales sin URL. También un *"hasta un 60 % más rápido"* sin metodología rastreable.
>
> Se listan aquí porque **dominan los resultados de búsqueda** sobre este tema. Saber que no están respaldadas es parte del contenido del tomo.

---

## 🔗 Conexiones

- [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12 · ⭐ Técnicas avanzadas de query]] — el otro tomo sin fuente primaria, con el mismo blindaje metodológico y una conclusión hermana: **la evidencia respalda menos de lo que la divulgación promete**.
- [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10 · Producción]] — la observabilidad de §8, y las dos correcciones de precisión sobre OpenTelemetry y la licencia de Phoenix.
- [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11 · Trade-offs de cost/latency]] — el marco de costo contra el que se juzgan los 5.395 tokens por turno de §3.
- [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Agentic RAG]] — los patrones de workflow de Anthropic, misma fuente que la recomendación de §1.2.
- [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]] — donde LangChain aparece por única vez en el resto de la guía, y como complemento externo: sus splitters son el ejemplo canónico de la **capa ②**, la que sí conviene no reescribir.

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_12-Query-Decomposition-Multi-Query-y-GraphRAG|Tomo 12 · ⭐ Técnicas avanzadas de query]] · Siguiente → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]

> 🏁 **Fin de los complementos de vanguardia.** Con este tomo se cierra el cuerpo de la guía: once tomos sobre el curso (01–11) y dos complementos externos (12–13). Lo que sigue son los transversales: el [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|glosario ejecutivo]] y la [[Guia-Maestra-RAG_16-Bibliografia|bibliografía]].
