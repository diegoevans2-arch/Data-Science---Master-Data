---
title: "Tomo 02 — Fundamentos: LLMs y el pipeline RAG"
tags: [rag, llm, tokens, autoregressive, hallucination, context-window, retriever, grounding, information-retrieval]
audiencias: [tecnico, puente, ejecutivo]
tomo: 02
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 02 — Fundamentos: LLMs y el pipeline RAG

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01 · Introducción a RAG]] · Siguiente → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03 · Keyword search: TF-IDF y BM25]]

---

> [!info] ¿Por qué importa esta sección?
> El [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01]] respondió *qué es* RAG y presentó el pipeline como una caja negra de cuatro etapas. Este tomo **abre las dos cajas centrales**: qué hace un LLM por dentro cuando genera texto, y qué hace un retriever por dentro cuando busca. No es teoría decorativa: **casi todas las decisiones de diseño de un sistema RAG son consecuencia directa de estas dos mecánicas.** Por qué el modelo alucina, por qué el `top_k` es un dilema y no un número obvio, por qué un prompt largo cuesta caro, por qué el chunking existe — todo se explica aquí. Si el Tomo 01 es el *qué*, este es el *por qué*.

> [!abstract] 👔 Impacto ejecutivo
> Entender que un LLM es una máquina de predecir texto probable —no una base de datos de hechos— cambia por completo cómo se gobierna un proyecto de IA generativa. Es la diferencia entre esperar magia y diseñar controles.
> - **Decisiones que habilita:** definir qué tareas se le pueden delegar a un LLM y cuáles no; exigir citación de fuentes como requisito de arquitectura, no como *nice to have*; presupuestar el costo real de operación (que escala con el largo del prompt, no con el número de usuarios).
> - **Costo o riesgo de hacerlo mal:** tratar al modelo como un oráculo infalible lleva a desplegar sistemas que fallan en silencio. Una alucinación no se ve como un error: se ve como una respuesta perfectamente redactada. En banca, salud o legal, ese es un riesgo material.
> - **Pregunta ejecutiva que responde:** *"¿Por qué la IA inventa cosas, y qué palanca concreta tengo para evitarlo?"*

---

## 1. 🌍 Dónde se usa RAG hoy

Audiencia: 🧭 👔

Antes de la mecánica interna, conviene ver el mapa de aplicación. La regla que las une todas es simple:

> [!tip] La regla del oro para detectar oportunidades de RAG
> **Cada vez que tienes acceso a información que probablemente no estuvo en el entrenamiento de un LLM, hay una aplicación potencial de RAG.** Información privada, reciente, o muy específica de un nicho: los tres perfiles clásicos.

| Aplicación | Qué es la knowledge base | Por qué RAG es la respuesta |
|---|---|---|
| **Code generation** | Tu propio repositorio: clases, funciones, definiciones, estilo de código | El modelo vio "todo GitHub público", pero no *tu* proyecto. Sin las definiciones reales, genera código plausible que no compila |
| **Chatbots corporativos** | Documentos internos: productos, políticas, inventario, troubleshooting | Cada empresa tiene productos, políticas y tono propios. Sin ellos, el modelo responde genérico o engañoso |
| **Salud y legal** | Historiales clínicos, papers recientes, documentos de un caso | Dominios donde la precisión es imperativa y la información es vasta, privada y de nicho. Aquí RAG suele ser **la única forma viable** de desplegar un LLM |
| **AI-assisted web search** | Internet entera | Los buscadores siempre funcionaron como retrievers; el resumen con IA es la capa de generación sobre ese retrieval |
| **Asistentes personales** | Tus emails, mensajes, calendario, carpeta de documentos | Knowledge base pequeña pero **densa en contexto**. Poca información, altísima relevancia para lo que estás haciendo ahora |

> [!example] 📊 Caso de negocio — Soporte técnico en telco
> **Problema:** una empresa de telecomunicaciones recibe miles de consultas diarias sobre planes, cobertura y troubleshooting. El equipo de soporte tarda en promedio 6 minutos por ticket buscando en una base documental de 4.000 páginas que cambia cada mes. Un LLM genérico responde con información de otras operadoras o directamente inventa pasos de configuración.
> **Técnica aplicada:** RAG sobre la base documental interna (manuales técnicos, catálogo de planes vigente, historial de tickets resueltos). El retriever entrega los 5 fragmentos más relevantes; el LLM redacta la respuesta citando el documento fuente.
> **Resultado:** el agente humano recibe un borrador fundamentado y verificable en segundos en vez de buscar a mano. Cuando el catálogo cambia, se actualiza la knowledge base — **no se reentrena nada**. La trazabilidad a documento fuente permite auditar cualquier respuesta entregada al cliente.

---

## 2. 🧠 El LLM por dentro

Audiencia: 🔧 🧭

Aquí está el corazón conceptual de este tomo. Todo lo que sigue explica comportamientos que después vas a tener que gestionar en producción.

### 2.1 Fancy autocomplete: prompt y completion

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> A veces se llama a los LLMs, medio en broma, **"autocompletado con esteroides"**. Y la broma es una descripción justa. Es el mismo mecanismo del teclado de tu celular cuando sugiere la siguiente palabra — solo que en vez de mirar las últimas dos palabras, mira miles, y en vez de tres sugerencias tiene cien mil.

**🔧 Definición técnica:** un LLM hace **una sola cosa**: predecir qué token debería aparecer a continuación en un texto. El texto de entrada se llama **prompt**; el texto que el modelo produce se llama **completion**.

Considera el prompt incompleto `"What a beautiful day, the sun is..."`. Las continuaciones posibles no son correctas o incorrectas en términos gramaticales, sino **probables o improbables**:

| Completion | Gramatical | Probable | Comentario |
|---|:---:|:---:|---|
| `shining` | ✅ | ⭐⭐⭐ | La más esperable |
| `rising` | ✅ | ⭐⭐ | Válida, menos frecuente en ese contexto |
| `out` | ✅ | ⭐⭐ | Válida |
| `exploding` | ✅ | ⭐ | Gramaticalmente impecable, semánticamente absurdo |

> [!warning] La distinción que lo explica todo
> `"the sun is exploding"` no es un error de gramática. Es inglés perfecto. El problema es que es **improbable**. Un LLM no distingue *verdadero* de *falso*: distingue **probable** de **improbable**. Guarda esta frase — es la raíz de las alucinaciones (sección 2.6).

**🧭 Cuándo importa:** siempre que alguien en tu equipo diga "el modelo *sabe* que…". No sabe: estima. Esa reformulación cambia cómo diseñas validaciones.

---

### 2.2 Tokens, no palabras

Audiencia: 🔧 🧭

**🔧 Definición técnica:** técnicamente, un LLM no genera palabras sino **tokens**: piezas de palabras. La mayoría de los modelos maneja un **vocabulary** de entre 10.000 y más de 100.000 tokens.

```
  "London"            →  [London]                    1 token (palabra frecuente)
  "door"              →  [door]                      1 token
  "unhappy"           →  [un][happy]                 2 tokens (compuesta)
  "programmatically"  →  [program][matically]        2+ tokens
  "?" "," "."         →  [?] [,] [.]                 la puntuación también tokeniza
```

> [!tip] 💡 Analogía
> Los tokens son las **piezas de LEGO del lenguaje**. Si el modelo tuviera que asignar una pieza única a cada palabra existente, necesitaría un catálogo infinito (y jamás podría escribir una palabra nueva o un nombre propio raro). Con piezas más pequeñas y combinables, construye *cualquier* palabra —incluso una que nunca vio— ensamblando trozos.

**🧭 Por qué te importa en la práctica:** los tokens son la **unidad de facturación y la unidad de límite**. Los proveedores cobran por token, y el `context window` se mide en tokens. Cuando estimes costos de un sistema RAG, la cuenta se hace en tokens, no en palabras. Regla gruesa útil: en inglés ≈ 0,75 palabras por token; en español el ratio es peor (más tokens por palabra), lo que encarece el mismo texto traducido.

> [!note] Complemento de vanguardia
> El algoritmo dominante para construir estos vocabularios es **BPE (Byte-Pair Encoding)**, introducido para traducción automática por Sennrich et al. (2016) y hoy estándar en la mayoría de los LLMs. No es material del curso, pero es el nombre que verás en la documentación de cualquier tokenizer.

---

### 2.3 El loop de generación

Audiencia: 🔧

**🔧 Definición técnica:** para *cada* token nuevo que agrega, el modelo repite un ciclo de tres pasos:

```
   ┌────────────────────────────────────────────────────────────────┐
   │                                                                │
   │   1. PROCESAR EL ESTADO ACTUAL                                 │
   │      Lee todo el texto hasta ahora (prompt + lo ya generado)   │
   │      y construye una representación profunda de las            │
   │      relaciones entre cada token y el significado global.      │
   │                          │                                     │
   │                          ▼                                     │
   │   2. CALCULAR PROBABILIDADES                                   │
   │      Recorre TODO su vocabulary (decenas o cientos de miles    │
   │      de tokens) y asigna a cada uno la probabilidad de ser     │
   │      el siguiente.                                             │
   │                                                                │
   │        shining   ████████████████████████  80%                 │
   │        rising    ████                       9%                 │
   │        out       ██                         5%                 │
   │        warming   █                          3%                 │
   │        ...                                                     │
   │        exploding ▏                       0.001%   ← nunca es 0 │
   │                          │                                     │
   │                          ▼                                     │
   │   3. ELEGIR AL AZAR desde esa distribución                     │
   │      No toma siempre el más probable: hace un sorteo           │
   │      ponderado. 80 de cada 100 veces saldrá "shining"...       │
   │      pero puede salir otro.                                    │
   │                          │                                     │
   └──────────────────────────┼─────────────────────────────────────┘
                              │
                     token elegido se AGREGA al texto
                              │
                              └──► vuelve al paso 1 (con el texto ya más largo)
```

El punto crítico del paso 3: el modelo **no elige el token más probable**, sino que hace un **muestreo aleatorio ponderado** por la distribución. Ningún token tiene probabilidad exactamente cero — por eso, ocasionalmente, sale algo raro.

**🧭 Cuándo usarlo:** entender este loop es lo que te permite después ajustar los parámetros de sampling (`temperature`, `top_p`) para controlar cuán conservador o creativo es el modelo. Se desarrolla en [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08]].

---

### 2.4 Autoregresividad: el modelo se influye a sí mismo

Audiencia: 🔧 🧭

**🔧 Definición técnica:** cada token generado pasa a formar parte del input del siguiente ciclo. A esta propiedad —el output se vuelve input— se le llama comportamiento **autoregressive** (auto-influyente).

La consecuencia es que una elección temprana condiciona todo lo que viene después:

```
  Prompt:  "What a beautiful day, the sun is ___"

           ┌─ elige "shining" ──► "...in the sky"      ✓ coherente
           │
           └─ elige "warming"  ──► "...our faces"      ✓ coherente
                                    ▲
                                    └── ¡pero es un camino COMPLETAMENTE distinto!
```

Ambas ramas son perfectamente coherentes. Simplemente son universos paralelos que se bifurcaron en un sorteo.

> [!tip] 💡 Analogía
> Es un **narrador de cuentos improvisando en voz alta**. Cada frase que dice lo compromete con la siguiente: si dijo "el héroe entró al castillo", ya no puede seguir con "y siguió nadando". Lo bueno es que el relato queda coherente. Lo incómodo es que si la primera frase salió torcida, **el resto del cuento se acomoda a la frase torcida** en vez de corregirla.

> [!warning] Consecuencia operativa: el mismo prompt no da la misma respuesta
> Por la combinación de aleatoriedad + autoregresividad, **ejecutar el mismo prompt varias veces contra el mismo modelo produce completions distintas.** Esto rompe la intuición de quien viene de software determinista, y tiene tres implicancias fuertes:
> - Tus **tests no pueden comparar strings exactos**; hay que evaluar por propiedades o con métricas (ver [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]]).
> - Un bug reportado por un usuario **puede no ser reproducible**.
> - Una demo que salió perfecta ante el directorio **puede salir distinta en la siguiente ejecución**. (Sí, esto le ha pasado a mucha gente.)

---

### 2.5 Cómo aprendió: el entrenamiento en una página

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** bajo el capó hay una **red neuronal** con miles de millones de **parámetros** (pesos numéricos). Antes de entrenar, esa red produce puro ruido. El entrenamiento es un ciclo masivo:

```
  1. Se le muestra un fragmento incompleto del training data
  2. El modelo predice qué token viene después
  3. Se compara con el token real
  4. Se ajustan los parámetros según el error
  5. Repetir... billones de veces
```

Los modelos actuales se entrenan sobre **billones de tokens** de texto, mayormente de internet abierto. Al hacerlo, el modelo absorbe dos cosas a la vez: la **información factual** contenida en esos textos y los **estilos lingüísticos** con que fueron escritos. Por eso puede escribir sobre casi cualquier tema y en casi cualquier registro: porque ejemplos de ese tema y ese registro estaban en los datos.

> [!abstract] 👔 En una frase para el negocio
> El "conocimiento" del modelo es un subproducto estadístico de haber leído internet, no una base de datos consultada. No hay un registro que se pueda auditar, corregir ni actualizar: por eso el conocimiento propio de la empresa **se inyecta en el prompt (RAG), no se instala en el modelo.**

---

### 2.6 Por qué alucina (y por qué no es un bug)

Audiencia: 🔧 🧭 👔

Ahora se puede cerrar el círculo. Si preguntas por datos internos de tu empresa o por las noticias de hoy, el modelo casi con certeza no fue entrenado con esa información. ¿Qué hace entonces? **Lo único que sabe hacer: generar la secuencia de palabras más probable.** El resultado suena bien y es falso.

> [!danger] La frase que hay que memorizar
> **Los LLMs están diseñados para generar texto *probable*, no texto *verdadero*.**
> Para un LLM, la "verdad" es simplemente que una secuencia de palabras sea estadísticamente plausible según su training data. No hay un módulo de verificación. No hay una intención de engañar. No está "fallando": está haciendo exactamente aquello para lo que fue construido.

Esto reencuadra la palabra *hallucination*: el modelo **no está teniendo un episodio psicológico ni funcionando mal**. Es un nombre desafortunado para un comportamiento esperable.

> [!tip] 💡 Analogía
> Imagina a alguien que rinde un examen oral con la regla estricta de **no poder decir "no sé" jamás**. Es elocuente, tiene cultura general enorme y suena convincente siempre. Ante una pregunta cuya respuesta ignora, no se queda callado: **construye la respuesta que más se parece a lo que debería ser una respuesta correcta.** No miente por malicia — es que su único trabajo es sonar bien, y nadie le dio permiso para callarse.

**🧭 La palanca concreta:** cuando el training data de buena calidad cubre el tema, "lo probable" y "lo verdadero" tienden a coincidir. Las alucinaciones aparecen donde esa coincidencia se rompe. Por lo tanto el objetivo de diseño es: **asegurar que el modelo tenga acceso a la información relevante en el momento de responder.** Eso es exactamente RAG.

---

### 2.7 Grounding: la solución que RAG explota

Audiencia: 🔧 🧭

**🔧 Definición técnica:** RAG funciona porque los LLMs son **excelentes entendiendo el contexto que se les entrega en el prompt**. Si agregas información relevante al prompt, el modelo la comprende y la incorpora a su respuesta **aunque nunca haya estado en su training data**. A ese efecto se le llama **grounding**: la información recuperada *aterriza* la respuesta en evidencia concreta.

Esta es la observación clave de todo el enfoque:

```
   ┌──────────────────────────────────────────────────────────┐
   │  Un LLM no necesita haber sido entrenado con un dato     │
   │  para poder usarlo — le basta con tenerlo a la vista.    │
   └──────────────────────────────────────────────────────────┘
```

**👔 En una frase para el negocio:** grounding es lo que convierte una respuesta opinable en una respuesta **verificable**, y es la base técnica de cualquier requisito de citación de fuentes.

---

### 2.8 Context window: por qué no puedes meter todo

Audiencia: 🔧 🧭 👔

Si agregar información al prompt mejora las respuestas, la tentación obvia es **agregar toda la que tengas**. No se puede, por dos razones distintas:

> [!warning] Los dos techos del prompt largo
> **1. Costo computacional.** Antes de generar *cada* token nuevo, el modelo hace un barrido computacionalmente costoso sobre **todos** los tokens que ya existen —incluido el prompt entero—. Prompt más largo = cada token nuevo cuesta más = más caro y más lento. Y no crece de forma lineal: el mecanismo de attention escala **de forma cuadrática** con el largo de la secuencia (Vaswani et al., 2017).
>
> **2. Límite duro: el context window.** Es la secuencia máxima que un modelo puede procesar de una vez. Modelos antiguos: unos pocos miles de tokens. Modelos recientes: hasta millones. Pero siempre hay un techo, y al superarlo el sistema simplemente falla o trunca.

```
   Información agregada al prompt  ──────────────────────►

   │◄──── zona útil ────►│◄─ zona cara ─►│◄─ pared ─►│
   │                     │               │           │
   │  Mejora la          │  Sigue        │  Excede   │
   │  respuesta          │  funcionando  │  el       │
   │                     │  pero cada    │  context  │
   │                     │  query cuesta │  window   │
   │                     │  más y tarda  │  → falla  │
   │                     │  más          │           │
```

> [!tip] 💡 Analogía
> El context window es **el escritorio del modelo**. Puedes apilarle documentos para que trabaje mejor, pero el escritorio tiene un tamaño físico. Y aunque quepan, mientras más papeles apilas, más tarda en revisarlos todos antes de escribir cada frase. Un escritorio ordenado con los 5 documentos correctos rinde más que uno sepultado bajo 500.

> [!note] 🔬 Complemento de vanguardia — "¿los context windows gigantes matan a RAG?"
> Es la pregunta que aparece cada vez que sale un modelo con más contexto. La respuesta corta es **no**, por tres razones con respaldo:
> 1. **Degradación posicional.** Los modelos no usan uniformemente todo su contexto: la información ubicada al medio de un prompt largo se aprovecha significativamente peor que la del principio o el final — el fenómeno *lost in the middle* (Liu et al., 2023). Más contexto no equivale a más comprensión.
> 2. **Economía.** Pagas por token de entrada en cada query. Un retriever que entrega 5 chunks relevantes en lugar de 500 documentos completos reduce el costo unitario en órdenes de magnitud, y la diferencia se multiplica por el volumen de tráfico.
> 3. **Escala real.** Ningún context window contiene un corpus empresarial completo (millones de documentos que además cambian a diario).
>
> El context window grande **no reemplaza al retriever: lo vuelve más tolerante.** Permite entregar más chunks o chunks más grandes sin romper nada, pero la necesidad de seleccionar *cuáles* sigue intacta.

---

## 3. 🔍 El retriever por dentro

Audiencia: 🔧 🧭

El segundo componente. Su trabajo se enuncia fácil —*encontrar en la knowledge base los documentos que ayuden al LLM a responder*— y es engañosamente difícil, porque:

- Los usuarios **no escriben queries SQL bien estructuradas**: conversan con el sistema como le hablarían a una persona.
- Los documentos son emails, memos internos, papers médicos: **estructurados para que los lea un humano, no para que los busque una máquina**.
- Y todo eso hay que resolverlo **en fracciones de segundo**.

### 3.1 La analogía de la biblioteca

Audiencia: 🔧 🧭 👔 💡

> [!tip] 💡 Analogía
> Entras a una biblioteca con la pregunta *"¿cómo hago pizza estilo Nueva York en casa?"*. La biblioteca tiene miles de libros **organizados en secciones y estanterías** por tema, género y autor (eso es el **index**). Le haces la pregunta al **bibliotecario**, que hace tres cosas: entiende el *significado* de tu pregunta, deduce que debe buscar en cocina / cocina italiana / quizás Nueva York, y **vuelve con los libros más pertinentes** — no con todos los libros de la biblioteca.

El mapeo es directo:

| En la biblioteca | En el retriever |
|---|---|
| Colección de libros | **Knowledge base** de documentos |
| Organización en secciones y estanterías | **Index** de los documentos |
| El bibliotecario entiende tu pregunta | El retriever procesa el prompt para captar su significado |
| Sabe a qué estantes ir | Busca en el index |
| Vuelve con los libros pertinentes | Devuelve los documentos más relevantes |

### 3.2 Scoring y ranking

Audiencia: 🔧

**🔧 Definición técnica:** al buscar, el retriever **rankea** los documentos de la knowledge base por relevancia respecto al prompt. Cada documento recibe un **score numérico** que cuantifica esa relevancia — habitualmente alguna medida de **similarity** entre el texto del prompt y el texto del documento. Se devuelven los de score más alto.

```
   Query: "¿Cómo hago pizza estilo Nueva York en casa?"
                          │
                          ▼
   ┌──────────────────────────────────────────────────────────┐
   │  SCORING sobre la knowledge base                         │
   ├──────────────────────────────────────────────────────────┤
   │  doc_412  "Masa de pizza napolitana paso a paso"   0.91  │ ◄─┐
   │  doc_077  "Guía de hornos caseros para pizza"      0.88  │ ◄─┤ top_k = 3
   │  doc_205  "Historia de la pizza en Nueva York"     0.84  │ ◄─┘
   │  ─────────────────── corte top_k ──────────────────────  │
   │  doc_119  "Recetas de pan de masa madre"           0.71  │ ← ¿relevante y descartado?
   │  doc_988  "Mantenimiento de refrigeradores"        0.12  │
   │  ...                                                     │
   └──────────────────────────────────────────────────────────┘
```

Existen varias familias de técnicas para calcular ese score. Son el contenido completo del Módulo 2 del curso: **keyword search** ([[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03]]) y **semantic search** ([[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]]).

### 3.3 El dilema del `top_k`

Audiencia: 🔧 🧭 👔

Un buen retriever debe hacer **dos cosas opuestas a la vez**: devolver lo relevante *y* **retener lo irrelevante**. Ahí está la tensión de diseño:

| Estrategia | Qué pasa | Consecuencia |
|---|---|---|
| **Devolver demasiado** (`top_k` alto) | Técnicamente tienes todos los documentos relevantes… enterrados en una montaña de ruido | Prompts caros, latencia alta, y eventualmente se agota el context window. El LLM además se distrae con evidencia irrelevante |
| **Devolver muy poco** (`top_k` bajo) | Solo entra lo mejor rankeado | Te pierdes información valiosa que quedó en el puesto 2, 3 o 4. Respuestas incompletas |

> [!warning] El mundo perfecto no existe
> En un mundo ideal el retriever rankearía perfectamente y elegiría el número exacto de documentos a devolver. **En la práctica, siempre rankeará algunos documentos relevantes demasiado abajo y algunos irrelevantes demasiado arriba.** Por eso no existe un `top_k` universalmente correcto: es un parámetro que **se monitorea y se experimenta**, no se deduce.

**🧭 Cuándo usarlo:** `top_k` es probablemente la primera perilla que vas a tocar al tunear un sistema RAG. Empieza en 3–5, mide, y muévete. Las herramientas para medirlo con rigor están en [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]]; las técnicas para mejorar el ranking antes de cortar (**re-ranking**) están en [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]].

> [!abstract] 👔 En una frase para el negocio
> El `top_k` es la perilla que arbitra directamente entre **calidad de respuesta** y **costo por consulta**. No es un detalle técnico: es una decisión de producto con impacto en la factura mensual.

### 3.4 De dónde viene todo esto

Audiencia: 🔧 🧭

**🔧 Contexto:** los retrievers no se inventaron con los LLMs. Mucho software que usas a diario hace exactamente lo mismo:

- Un **buscador web** recupera páginas relevantes para una búsqueda.
- Una **base de datos relacional** recupera filas que satisfacen una query SQL.

El campo de **information retrieval** ya era una disciplina madura cuando aparecieron los primeros LLMs (Manning et al., 2008), y sus ideas son la base directa del diseño de los retrievers en sistemas RAG. Esto es una buena noticia práctica: no estás en territorio experimental — estás aplicando décadas de teoría consolidada.

### 3.5 Vector databases: el adelanto

Audiencia: 🔧 🧭

En teoría hay muchas formas de implementar un retriever, y como la mayoría de las empresas ya tiene sus datos en bases relacionales, sería cómodo dejarlos ahí. **No son estrictamente necesarias**, pero *a escala* la mayoría de los retrievers se construye sobre una **vector database**: un tipo de base de datos especializada en encontrar rápidamente los documentos que más se parecen a un prompt.

Se desarrolla completo en [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05]].

---

## 4. 🏗️ La arquitectura completa y sus cinco ventajas

Audiencia: 🔧 🧭 👔

Con ambos componentes entendidos, se puede ver el sistema completo. Lo notable: **la experiencia de usuario es idéntica a la de un LLM normal.** Escribes un prompt, recibes una respuesta. Todo el trabajo extra es invisible.

```
   ══════════ USO NORMAL DE UN LLM ══════════

   Usuario ──► prompt ──► [ LLM ] ──► respuesta


   ══════════ SISTEMA RAG ══════════

   Usuario ──► prompt ──┬──────────────────────────────────────┐
                        │                                      │
                        ▼                                      │
                 ┌─────────────┐                               │
                 │  RETRIEVER  │◄──── knowledge base           │
                 │             │      (una base de datos       │
                 │  busca +    │       de documentos útiles)   │
                 │  rankea     │                               │
                 └──────┬──────┘                               │
                        │ documentos relevantes                │
                        ▼                                      │
                 ┌─────────────────────────────┐               │
                 │  AUGMENTED PROMPT           │◄──────────────┘
                 │  prompt original            │
                 │  + documentos recuperados   │
                 └──────────────┬──────────────┘
                                ▼
                          ┌──────────┐
                          │   LLM    │  ← desde aquí opera como
                          └────┬─────┘    cualquier LLM normal
                               ▼
   Usuario ◄────────────── respuesta
            (misma UX, un poco más de latency,
             mucha más probabilidad de ser correcta)
```

La única diferencia estructural entre usar un LLM directo y un sistema RAG es **la incorporación del retriever**. Ese agregado aparentemente simple habilita cinco ventajas:

| # | Ventaja | Qué significa | 👔 Por qué importa al negocio |
|---|---|---|---|
| 1 | **Acceso a información nueva** | Políticas de empresa, datos personales, titulares de esta mañana. Para cierta información, RAG es **la única vía** de ponerla frente a un LLM | Habilita casos de uso que sin esto son directamente imposibles |
| 2 | **Menos hallucinations** | Las alucinaciones suelen originarse en temas ausentes o poco frecuentes en el training data. La información en el prompt hace *grounding* y reduce respuestas genéricas o engañosas | Reduce riesgo reputacional y legal |
| 3 | **Actualización barata** | Reentrenar un modelo es caro y lento. En RAG **actualizas la knowledge base como cualquier base de datos**; apenas se indexa, el LLM ya responde con lo nuevo | Time-to-market de horas en vez de meses; el costo de mantener el sistema al día se desploma |
| 4 | **Citación de fuentes** | El sistema puede incluir la referencia en el augmented prompt, y el LLM propagarla a la respuesta | Permite que un humano **verifique**. Requisito de hecho en compliance, salud y legal |
| 5 | **Separación de responsabilidades** | El retriever filtra el mundo y encuentra lo importante; el LLM se dedica a redactar bien. Cada componente hace aquello en lo que es fuerte | Arquitectura más depurable: cuando algo falla, puedes aislar **si falló el retrieval o la generación** |

> [!tip] 🧭 La ventaja 5 es la más subestimada
> Separar retrieval de generation no es solo elegancia arquitectónica: es lo que hace **diagnosticable** el sistema. Cuando una respuesta sale mal, la primera pregunta siempre es *"¿el retriever trajo los documentos correctos?"*. Si los trajo y la respuesta igual es mala, el problema es de generación (prompt, modelo). Si no los trajo, ninguna mejora al prompt te va a salvar. Sin esa separación estarías depurando una caja negra.

---

## 5. 💻 El pipeline completo en código

Audiencia: 🔧

El [[Guia-Maestra-RAG_01-Introduccion-a-RAG#4. El pipeline mínimo en código|Tomo 01]] mostró las dos primeras piezas (`get_relevant_data` y `format_relevant_data`). Aquí se cierra el circuito con las dos que faltaban, del assignment C1M1.

> [!note] Contexto del ejercicio
> Dataset: *News Headlines 2024* (Kaggle) — titulares de BBC News, The Guardian y WSJ. Modelo: `llama-3-1-8b-instruct-turbo` vía **Together AI** (el curso usa modelos open-source justamente para poder mirar bajo el capó). Campos relevantes de cada documento: `title`, `description`, `published_at`, `url`.

**Paso 3 — Construir el augmented prompt.** Esta función es el punto donde ocurre la *augmentation*. El flag `use_rag` es el recurso pedagógico central del ejercicio: permite correr la **misma** query con y sin RAG.

```python
def generate_final_prompt(query, top_k=5, use_rag=True, prompt=None):
    """
    Construye el prompt final. Si use_rag=False, devuelve la query cruda:
    ese es el grupo de control del experimento.
    """
    # Sin RAG: la query viaja sola, el modelo responde solo de memoria
    if not use_rag:
        return query

    # 1. RETRIEVAL — los top_k documentos más relevantes para la query
    relevant_data = get_relevant_data(query, top_k=top_k)

    # 2. FORMATTING — de lista de dicts a string estructurado
    retrieve_data_formatted = format_relevant_data(relevant_data)

    # 3. AUGMENTATION — query + evidencia en un solo prompt
    if prompt is None:
        prompt = (
            f"Answer the user query below. There will be provided additional "
            f"information for you to compose your answer. The relevant information "
            f"provided is from 2024 and it should be added as your overall knowledge "
            f"to answer the query, you should not rely only on this information to "
            f"answer the query, but add it to your overall knowledge.\n"
            f"Query: {query}\n"
            f"2024 News: {retrieve_data_formatted}"
        )
    else:
        # Plantilla propia con placeholders {query} y {documents}
        prompt = prompt.format(query=query, documents=retrieve_data_formatted)

    return prompt
```

**Paso 4 — Generar.** El orquestador final: arma el prompt, llama al modelo, extrae el contenido.

```python
def llm_call(query, top_k=5, use_rag=True, prompt=None):
    """
    Orquesta el pipeline completo: retrieval → formatting → augmentation → generation.
    """
    # Etapas 1 a 3, encapsuladas
    prompt = generate_final_prompt(query, top_k, use_rag, prompt)

    # Etapa 4: GENERATION
    generated_response = generate_with_single_input(prompt)

    # La respuesta viene como dict; nos interesa el contenido
    generated_message = generated_response['content']

    return generated_message
```

Uso:

```python
query = "Tell me about the US GDP in the past 3 years."

respuesta_con_rag = llm_call(query, use_rag=True)    # responde con noticias de 2024
respuesta_sin_rag = llm_call(query, use_rag=False)   # responde solo de memoria (cutoff dic-2023)
```

> [!danger] ⚠️ Bug real en el prompt del assignment (corregido arriba)
> La versión original del notebook concatena los f-strings **sin salto de línea entre la última instrucción y la query**, produciendo literalmente:
> ```
> ...but add it to your overall knowledge.Query: Tell me about the US GDP...
> ```
> Fíjate en `knowledge.Query:` — pegado. No rompe la ejecución y el modelo suele tolerarlo, pero **degrada la delimitación entre instrucción y contenido**, que es justamente lo que un LLM usa para separar el rol de cada bloque del prompt. En el código de arriba se corrigió agregando `\n` al final de la instrucción. Es un recordatorio de la regla de oro del Tomo 01: **el string que entregas al modelo es literalmente lo que ve como evidencia.** Los detalles de formato del prompt no son cosméticos.

> [!note] Segunda observación sobre el diseño del experimento
> Con `use_rag=False` la función devuelve **la query pelada**, sin ninguna instrucción de sistema, mientras que con `use_rag=True` el modelo recibe instrucciones detalladas. Estrictamente, la comparación mezcla dos variables: *tener contexto* y *tener instrucciones*. Para el propósito didáctico funciona perfecto (el efecto del contexto domina ampliamente), pero si vas a **medir** el aporte de RAG en un sistema propio, mantén el prompt de instrucciones idéntico en ambas ramas y varía **solo** la presencia de los documentos.

---

## 6. 🧭 Tabla de decisiones: qué palanca mover

Audiencia: 🔧 🧭

Consolidado de este tomo — cuando un sistema RAG falla, esta tabla ubica el problema:

| Síntoma | Causa probable | Palanca | Dónde se profundiza |
|---|---|---|---|
| El modelo inventa datos que no están en los documentos | El retriever no trajo evidencia útil; el LLM rellena con lo probable | Subir `top_k`, revisar el retrieval, mejorar el prompt para forzar "no sé" | [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG\|Tomo 09]] |
| Las respuestas son correctas pero incompletas | `top_k` demasiado bajo: se cortaron documentos relevantes | Subir `top_k` o aplicar re-ranking antes del corte | [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT\|Tomo 07]] |
| Costo por query demasiado alto | Prompts largos: demasiado contexto o documentos muy grandes | Bajar `top_k`, chunks más pequeños, re-ranking para enviar menos y mejor | [[Guia-Maestra-RAG_06-Chunking\|Tomo 06]] |
| Error de context window excedido | El augmented prompt superó el límite del modelo | Bajar `top_k`, chunking, o modelo con ventana mayor | [[Guia-Maestra-RAG_06-Chunking\|Tomo 06]] |
| El retriever trae documentos fuera de tema | La técnica de scoring no captura el significado de la query | Cambiar keyword → semantic, o combinar ambas (hybrid) | [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings\|Tomo 04]] |
| La misma pregunta da respuestas distintas | Comportamiento esperado: sampling aleatorio + autoregresividad | Ajustar parámetros de sampling; no esperar determinismo | [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting\|Tomo 08]] |

---

## 7. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Token** | La pieza mínima de texto que el modelo maneja (un trozo de palabra). Es la unidad en la que te cobran |
| **Prompt / completion** | Lo que le entregas al modelo / lo que el modelo escribe de vuelta |
| **Vocabulary** | El catálogo de piezas de texto que el modelo puede usar (10.000 a 100.000+) |
| **Autoregressive** | Cada palabra que el modelo escribe condiciona la siguiente; por eso no se puede "corregir a mitad de camino" |
| **Probability distribution** | El ranking de probabilidades con que el modelo elige la siguiente pieza de texto |
| **Grounding** | Aterrizar la respuesta en evidencia entregada, en vez de en la memoria del modelo |
| **Context window** | El máximo de texto que el modelo puede procesar de una sola vez. Un techo duro |
| **Knowledge base** | La base de datos de documentos confiables que alimenta al sistema |
| **Index** | La organización interna que hace que buscar en la knowledge base sea rápido |
| **Score de relevancia** | La nota numérica que el retriever le pone a cada documento frente a la pregunta |
| **`top_k`** | Cuántos documentos se entregan al modelo. La perilla que arbitra calidad vs. costo |
| **Information retrieval** | La disciplina (previa a los LLMs) de encontrar información relevante en grandes colecciones |

---

## 8. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Sé explicar que un LLM solo predice el siguiente token, y por qué eso lo hace "fancy autocomplete".
- [ ] Entiendo la diferencia entre *probable* y *verdadero*, y cómo de ahí nacen las hallucinations.
- [ ] Puedo describir el loop de generación: procesar → distribución de probabilidad → muestreo aleatorio.
- [ ] Entiendo qué es el comportamiento autoregressive y por qué el mismo prompt da respuestas distintas.
- [ ] Sé qué es el grounding y por qué es el mecanismo que hace funcionar a RAG.
- [ ] Puedo explicar los dos techos del prompt largo: costo computacional y context window.
- [ ] Sé mapear la analogía de la biblioteca a los componentes del retriever (knowledge base, index, scoring).
- [ ] Entiendo el dilema del `top_k` en ambas direcciones y por qué no hay un valor universalmente correcto.
- [ ] Puedo enumerar las cinco ventajas que aporta agregar un retriever a un LLM.
- [ ] Sé por qué separar retrieval de generation hace el sistema diagnosticable.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01 · Introducción a RAG]] (el qué y el pipeline de 4 etapas)
- Siguiente tomo → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03 · Keyword search: TF-IDF y BM25]] (la primera familia de técnicas de scoring)
- El otro camino del scoring → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04 · Semantic search y embeddings]]
- El motor a escala → [[Guia-Maestra-RAG_05-Vector-Databases-y-ANN|Tomo 05 · Vector databases y ANN]]
- Cómo se parten los documentos → [[Guia-Maestra-RAG_06-Chunking|Tomo 06 · Chunking]]
- Mejorar el ranking antes del corte `top_k` → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Reranking]]
- La generación en profundidad (sampling, temperature) → [[Guia-Maestra-RAG_08-Generacion-Transformers-Sampling-Prompting|Tomo 08 · Generación]]
- Medir hallucinations y calidad → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Hallucinations y evaluación]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 1: RAG Overview (Coursera). Lecciones sobre aplicaciones de RAG, arquitectura, el LLM y el retriever; assignment C1M1.

**Fuentes externas (complemento con bibliografía verificable):**
- Vaswani, A. et al. (2017). *Attention Is All You Need*. NeurIPS. — Arquitectura transformer y el mecanismo de attention cuyo costo cuadrático explica por qué los prompts largos escalan mal.
- Sennrich, R., Haddow, B. & Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units*. ACL. — Byte-Pair Encoding, base de la tokenización de la mayoría de los LLMs actuales.
- Liu, N. F. et al. (2023). *Lost in the Middle: How Language Models Use Long Contexts*. TACL. — Evidencia de que los modelos aprovechan peor la información ubicada en el medio de contextos largos; sustento del argumento de que un context window grande no reemplaza al retriever.
- Ji, Z. et al. (2023). *Survey of Hallucination in Natural Language Generation*. ACM Computing Surveys, 55(12). — Taxonomía y causas de las hallucinations.
- Holtzman, A. et al. (2020). *The Curious Case of Neural Text Degeneration*. ICLR. — Estrategias de sampling (nucleus / top_p) sobre la distribución de probabilidad descrita en la sección 2.3.
- Manning, C. D., Raghavan, P. & Schütze, H. (2008). *Introduction to Information Retrieval*. Cambridge University Press. — Texto canónico del campo de information retrieval que precede y fundamenta a los retrievers modernos.
- Lewis, P. et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS. — Paper fundacional de RAG.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_03-Keyword-Search-TF-IDF-y-BM25|Tomo 03 · Information Retrieval: keyword search (TF-IDF, BM25)]]**, donde arranca el Módulo 2 y entramos al detalle de la primera familia de técnicas de scoring: la búsqueda por palabras clave que lleva décadas moviendo buscadores y bases de datos, y que sigue siendo una pieza clave de los retrievers modernos.
