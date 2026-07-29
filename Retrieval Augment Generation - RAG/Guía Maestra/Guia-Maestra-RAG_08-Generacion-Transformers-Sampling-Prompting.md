---
title: "Tomo 08 — Generación: transformers, sampling y prompt engineering"
tags: [rag, transformer, attention, sampling, temperature, top-p, top-k, prompt-engineering, reasoning-models, benchmarks]
audiencias: [tecnico, puente, ejecutivo]
tomo: 08
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 08 — Generación: transformers, sampling y prompt engineering

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing, arquitecturas y re-ranking]] · Siguiente → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Hallucinations, evaluación y agentic RAG]]

---

> [!info] ¿Por qué importa esta sección?
> Aquí arranca el **Módulo 4** y el foco se corre por fin del retriever al **generador**. Los cinco tomos anteriores se dedicaron a encontrar los documentos correctos; este trata de lo que pasa con ellos después — y como dice el curso, *"el retriever es una parte crítica, pero el LLM es el verdadero cerebro de la operación"*.
>
> El [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02]] explicó el LLM a nivel de comportamiento: predice el siguiente token, por eso alucina. Este tomo abre la caja **a nivel de arquitectura** — y no por curiosidad: casi todas las perillas que vas a tocar en producción (temperature, top_p, el system prompt, el modelo que eliges) solo tienen sentido si entiendes qué están modificando.

> [!abstract] 👔 Impacto ejecutivo
> Este es el tomo de las **perillas del generador**: las que cambian el tono, la confiabilidad, la latencia y el costo de cada respuesta sin tocar una línea del retriever.
> - **Decisiones que habilita:** elegir el modelo con criterio (y saber que la elección es **temporal**); configurar el sistema para que sea factual o creativo según el caso de uso; escribir el system prompt que gobierna todo lo que el asistente dice; decidir si un reasoning model justifica su costo.
> - **Costo o riesgo de hacerlo mal:** los defaults de sampling producen respuestas más creativas de lo que un caso factual tolera — y esa "creatividad" en un asistente corporativo se llama *invención*. En el otro extremo, un modelo sobredimensionado multiplica la factura sin mejorar nada que el usuario note.
> - **Pregunta ejecutiva que responde:** *"¿por qué el asistente responde distinto cada vez que le preguntan lo mismo, y qué control tengo sobre eso?"*

---

## 1. 🧠 El transformer por dentro

Audiencia: 🔧 🧭

**🔧 Contexto histórico:** el transformer se propuso en un paper seminal de 2017 titulado *Attention Is All You Need*, centrado en el problema de la **traducción automática**. Tenía dos componentes:

```
   ┌──────────────┐         ┌──────────────┐
   │   ENCODER    │────────►│   DECODER    │
   │              │         │              │
   │ procesa el   │         │ usa esa      │
   │ texto origen │         │ comprensión  │
   │ (alemán) y   │         │ para generar │
   │ construye su │         │ el texto en  │
   │ significado  │         │ inglés       │
   └──────────────┘         └──────────────┘
```

> [!important] Por qué esto importa para RAG
> **La mayoría de los LLMs solo incluye el segundo componente, el decoder**, porque lo único que les interesa es generar texto. Los **transformers completos** se usan típicamente **dentro de los embedding models**, cuyo objetivo es construir representaciones semánticas ricas.
>
> Es decir: el embedding model del [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]] y el LLM que genera la respuesta son **dos usos distintos de la misma familia de arquitectura**. Una comprime significado; la otra produce texto.

### 1.1 El viaje de un prompt

**🔧 Definición técnica:** el recorrido de un prompt por el decoder de un transformer, paso a paso:

```
   PROMPT: "the brown dog sat next to the red fox"
        │
        ▼
   ① TOKENIZACIÓN
      El texto se parte en tokens.
        │
        ▼
   ② EMBEDDING INICIAL
      Cada token recibe un vector denso: la "primera conjetura" de su
      significado. Es ESTÁTICO — el mismo token siempre recibe el mismo
      vector de arranque.
        │
        ▼
   ③ VECTOR POSICIONAL
      Cada token recibe además un vector que codifica DÓNDE está en el
      prompt. (Sin esto, "el perro mordió al cartero" y "el cartero
      mordió al perro" serían idénticos — el problema de la bag of
      words del Tomo 03.)
        │
        ▼
   ④ ATTENTION  ◄─────────────────────────┐
      Cada token mira a TODOS los demás:   │
      ve su significado y su posición, y   │
      decide a cuáles prestar atención.    │  se repite
        │                                  │  8 – 64 veces
        ▼                                  │
   ⑤ FEEDFORWARD                           │
      La parte más grande del modelo (la   │
      que tiene MÁS parámetros). Asigna    │
      vectores actualizados: la "segunda   │
      conjetura", ahora informada por el   │
      contexto. ─────────────────────────►─┘
        │
        ▼
   ⑥ DISTRIBUCIÓN DE PROBABILIDAD
      Sobre TODO el vocabulary. Un puñado de tokens tiene probabilidad
      alta; los ~100.000 restantes reciben valores ínfimos pero no cero.
        │
        ▼
   ⑦ SORTEO PONDERADO
      Se elige un token de esa distribución. → sección 2
        │
        ▼
   El token se AÑADE al prompt y se repite TODO el proceso
   hasta alcanzar el límite de tokens o generar el token de fin.
```

### 1.2 Attention: qué es y por qué hay muchas cabezas

Audiencia: 🔧 🧭 💡

**🔧 Definición técnica:** *attention* es una forma elegante de decir **"¿qué otros tokens deberían tener el mayor impacto en mi significado?"**. En la frase *"the brown dog sat next to the red fox"*, el token `dog` probablemente presta la mayor atención a `brown` y `sat`:

```
   dog  ──70%──►  brown      (qué tipo de perro es)
        ──20%──►  sat        (qué está haciendo)
        ──10%──►  el resto
```

El mecanismo que asigna esa atención se llama **attention head**, y los modelos incluyen **muchas**:

```
   ATTENTION HEAD 1 (relaciones objeto ↔ descripción)
      fox ──────► brown          "¿de qué color es?"

   ATTENTION HEAD 2 (relaciones espaciales)
      fox ──────► sat, next      "¿dónde está?"

   Modelos pequeños:  8 – 16 heads
   Modelos grandes:   más de 100
```

> [!warning] ⚠️ Las cabezas no se especializan en categorías que un humano asignó
> El ejemplo de arriba es didáctico. En realidad, las relaciones que captura cada attention head **no son un conjunto ordenado de categorías definidas por personas**, sino un conjunto **complejo y abstracto de relaciones aprendidas durante el entrenamiento**. Nadie le dijo a la cabeza 2 que se dedicara al espacio.

> [!tip] 💡 Analogía
> Imagina un comité leyendo la misma frase, donde cada miembro tiene **una obsesión distinta**: uno solo se fija en quién hace qué, otro en los adjetivos, otro en el orden temporal, otro en cosas que ni sabríamos nombrar. Cada uno subraya la frase con su propio criterio, y al final se juntan todos los subrayados. **Cien lecturas simultáneas de la misma frase, cada una con un punto de vista distinto** — de ahí sale una representación tan detallada de las relaciones entre las palabras.

### 1.3 Las tres consecuencias para RAG

Audiencia: 🔧 🧭 👔

El curso cierra la lección conectando la arquitectura con el diseño del sistema, y las tres conclusiones valen copiarse tal cual:

| # | Consecuencia | Por qué |
|---|---|---|
| 1 | **Explica por qué RAG funciona en primer lugar** | Los LLMs pueden entender profundamente el significado y la relevancia de la información añadida al prompt — gracias al procesamiento del attention y al conocimiento del mundo contenido en las capas feedforward |
| 2 | **Los LLMs siguen siendo inherentemente aleatorios** | Aunque inyectes información significativa en el prompt, el modelo **puede elegir al azar no generar texto basado en ella**. Controlar esa aleatoriedad y confirmar el grounding sigue siendo necesario |
| 3 | **Un LLM es computacionalmente carísimo** | Generar **un solo token** exige muchísimo procesamiento, y el costo **crece con el largo** del prompt y de la completion: cada token debe mirar a todos los demás |

> [!abstract] 👔 De dónde viene la factura
> La consecuencia nº3 tiene una traducción directa: *"la mayor parte de los costos de operar un sistema RAG viene de correr estos modelos transformer, potentes pero caros"*. No viene de la vector database, ni del índice, ni del almacenamiento. **Viene de los tokens.** Y por eso todo lo que reduzca el largo del prompt —mejor chunking, mejor re-ranking, context pruning (sección 5.4)— es también una decisión de presupuesto.

---

## 2. 🎛️ Sampling: controlar la aleatoriedad

Audiencia: 🔧 🧭

**🔧 El punto de partida:** cada token que un LLM añade a la completion es una **elección aleatoria ponderada**. Con modelos open source puedes ver la distribución que se usó para elegir. Para el prompt `"the sky is"`:

```
   blue     ████████████████████████████  50 %
   bright   ██████████████                25 %
   ...      resto ≤ 10 %, cayendo rápido a menos de 1 %
```

Y la **forma** de esa curva es interpretable:

```
   DISTRIBUCIÓN CON PICO (confianza)      DISTRIBUCIÓN PLANA (incertidumbre)

   █                                       █ █ █ █ █
   █                                       █ █ █ █ █
   █ ▄                                     █ █ █ █ █
   █ █ ▄ ▁ ▁                               █ █ █ █ █
   ─────────────                           ───────────
   un claro ganador                        sin ganador claro:
                                           el modelo tiene muchas
                                           direcciones posibles
```

> [!important] Decodificar y controlar esa curva ES el tuning del LLM
> Todas las técnicas de esta sección hacen lo mismo desde ángulos distintos: **cambiar la forma de la distribución** o **recortar de qué parte de ella se puede elegir**.

### 2.1 Greedy decoding

**🔧 Definición:** instruir al modelo para que **no haga una elección aleatoria** y siempre tome el token de mayor probabilidad.

| ✅ Ventaja | ⚠️ Costo |
|---|---|
| **El LLM se vuelve determinista**: el mismo prompt da siempre la misma respuesta | El texto sale **predecible**, a veces genérico o forzado |
| | El modelo puede **quedarse atascado** repitiendo la misma secuencia. No le importa si la completion tiene sentido global: sigue eligiendo el más probable, y **una vez que entra en un bucle no tiene mecanismo para salir** |

**🧭 Cuándo conviene:** cuando la salida determinista y predecible es deseable — **completado de código**, o como **ajuste temporal para depurar** el sistema.

### 2.2 Temperature

**🔧 Definición:** el parámetro más usado para controlar la aleatoriedad. Piénsalo como un **dial que cambia la forma** de la distribución.

```
   temperature = 0     █                    greedy decoding
                       ▁ ▁ ▁ ▁              (el más probable = 100 %)

   temperature = 0.5   █                    distribución MÁS PICUDA
                       █ ▄                  solo los muy probables tienen chance
                       █ █ ▁ ▁

   temperature = 1     █                    la distribución ORIGINAL
                       █ ▄                  (el default)
                       █ █ ▄ ▁

   temperature = 1.2   █ ▄                  distribución MÁS PLANA
                       █ █ ▄ ▄              → más variedad, más "creativa"
                       █ █ █ █

   temperature = 5     █ █ █ █ █            MUY plana → todos los tokens
                       █ █ █ █ █            casi igual de probables,
                       █ █ █ █ █            aunque no tengan sentido
```

> [!note] Dos precisiones sobre temperature
> - **El orden de los tokens no cambia**, solo sus probabilidades. Temperature no reordena: reescala.
> - **`temperature = 1` NO es determinista.** El lab lo marca como punto importante y conviene subrayarlo: la distribución original sigue teniendo cola, y el modelo puede elegir de ahí. El único valor determinista es **0**.
> - **No tiene tope teórico** (cualquier positivo), aunque los proveedores suelen imponer un límite. `top_p`, en cambio, está acotado a [0, 1].

### 2.3 Top-k y top-p: recortar la cola

**🔧 El problema que resuelven:** cualquiera sea la temperature, la distribución **sigue teniendo una cola larga hacia la derecha**, llena de tokens sin sentido que el LLM tiene una pequeña probabilidad de elegir.

**Top-k sampling** — el más simple: limita al modelo a elegir entre los **k tokens más probables**.

```
   top_k = 5
                    │
   jumped  ████████ │ 32 %   ✓
   fence   ██████   │ 25 %   ✓
   gate    █████    │ 18 %   ✓   ← los 5 admitidos
   wall    ███      │ 12 %   ✓
   river   ██       │  9 %   ✓
   ─────────────────┤
   house   ██         7 %    ✗
   pizza   ▏          0.1 %  ✗   ← descartados por CONTEO
   banana  ▏          0.009 %✗
```

**Top-p sampling** (*nucleus sampling*) — limita a los tokens cuya **probabilidad acumulada** cae bajo un umbral.

```
   top_p = 85 %
                                    acumulado
   jumped  ████████  32 %  ✓         32 %
   fence   ██████    25 %  ✓         57 %
   gate    █████     18 %  ✓         75 %
   wall    ███       12 %  ✓         87 %  ← supera el 85 %: se corta AQUÍ
   ──────────────────────────
   river   ██         9 %  ✗
   house   ██         7 %  ✗         ← descartados por MASA de probabilidad
```

> [!important] 🎯 Por qué top-p suele ser mejor que top-k
> **Top-p es más dinámico y responsivo.** Con `top_k`, el modelo siempre elige del **mismo tamaño de pool**, sin importar la forma de la distribución. Con `top_p`:
>
> - Si el modelo está **seguro** (unos pocos tokens con probabilidad muy alta) → limita la elección a esos pocos.
> - Si el modelo está **inseguro** (distribución plana, sin ganador claro) → **permite elegir de un pool mucho más grande**.
>
> Es decir: `top_p` se adapta a la confianza del modelo; `top_k` no. Por eso la recomendación general del curso es **fijar temperature y top_p**, y recurrir al resto solo ante problemas concretos.

### 2.4 Técnicas sobre tokens individuales

A diferencia de las anteriores (que reforman la curva completa), estas apuntan a **probabilidades de palabras concretas**:

| Técnica | Qué hace | Para qué |
|---|---|---|
| **Repetition penalty** | Reduce la probabilidad de palabras **que ya aparecieron** en la completion | Los LLMs tienden a repetir la misma palabra o frase, lo que suena poco natural |
| **Logit biasing** | Sube o baja **permanentemente** la probabilidad de tokens específicos | Bajar palabrotas para que el sistema no las genere; **subir** las categorías si tu RAG es un clasificador, para garantizar que siempre elija entre ellas |

> [!tip] 🧭 El uso de logit bias que más se subestima
> El caso del clasificador es elegante: si tu sistema debe responder **una de N categorías**, en vez de rezar para que el modelo obedezca la instrucción, **le subes la probabilidad a esos N tokens**. Deja de ser una petición y pasa a ser una restricción mecánica.

### 2.5 La configuración recomendada por el curso

**🔧 El combo que la lección propone como punto de partida sensato y de propósito general:**

```python
temperature        = 0.8
top_p              = 0.9
repetition_penalty = 1.2
```

*"Este LLM será un poco más conservador en su elección de tokens, evita elegir de la cola lejana de la distribución, y penaliza levemente los tokens repetidos."*

Y el criterio para ajustarlo:

| Tu caso | temperature | top_p |
|---|---|---|
| **Generar código o responder preguntas factuales** | **más baja** | **más bajo** |
| **Dominio creativo** | más alta | más alto |

Después de eso: *"considera introducir repetition penalties, logit biases u otras técnicas de sampling en respuesta a problemas específicos que identifiques"* — es decir, **reactivamente, no preventivamente**.

### 2.6 💻 El lab: qué pasa de verdad al mover las perillas

Audiencia: 🔧

> [!note] Contexto del lab
> **Modelo:** `Qwen/Qwen3.5-9B` (hardcodeado en todo el módulo). **Prompt de prueba:** *"In one sentence, explain to me what is RAG"*. Todas las salidas que siguen son **reales** del Ungraded Lab 1.

**El hallazgo más valioso del lab — el colapso por temperature:**

```
   temperature = 0.3   ✅ "RAG is a technique that enhances large language
                          models by dynamically fetching relevant information
                          from external knowledge sources to ground their
                          responses, thereby improving accuracy and reducing
                          hallucinations."
                          → correcta, concisa, estable

   temperature = 1.5   ⚠️  Arranca bien y COLAPSA a los ~15 tokens:
                          "...enabling them to unlock static problem-specifics
                          through—including mini bull only عشرات tavola data—AJ
                          of external siden-years。 hierbei Sigmaauptsuche/에는..."
                          → ensalada multilingüe que consume los 500 tokens

   temperature = 3     ❌ Incoherente DESDE EL PRIMER TOKEN:
                          "Boston Asia辈子小品 bakery抽象它可以DC打折主页..."
                          → ni una frase válida
```

> [!danger] 🚨 El efecto de segundo orden que casi nadie anticipa
> Fíjate en que los dos casos degradados **agotaron el `max_tokens`**. La explicación del lab es precisa y muy útil: con temperature alta, **el token de fin de completion pierde probabilidad relativa igual que todos los demás**. El modelo no para porque no "quiere" parar — parar dejó de ser probable.
>
> Consecuencia práctica: **subir la temperature no solo degrada la calidad, sube la latencia y el costo.** Una respuesta que debía tomar 40 tokens se come 500. Es una perilla que toca las tres cosas a la vez.

**El sweet spot creativo — `temperature` combinada con `top_p`:**

Con el prompt *"Write a small poem about a flying rabbit"*:

| Configuración | Resultado |
|---|---|
| `temp=0.3, top_p=0.8` | Poema pulcro, métrica y rima consistentes |
| **`temp=1.5, top_p=0.5`** | **Más largo y más creativo, y SIGUE coherente** — el mejor resultado del lab |
| `temp=3, top_p=0.05` | Degrada, pero con **palabras reales** en vez de tokens de vocabularios arbitrarios |

> [!important] 🎯 Lo que demuestra esa tabla
> **`top_p` rescata parcialmente a una temperature alta.** Con `temp=1.5` sola el modelo colapsó; con `temp=1.5` + `top_p=0.5` produjo el texto más interesante del lab sin perder coherencia. Es la evidencia empírica de por qué el curso recomienda **configurar las dos juntas**: temperature reforma la distribución, `top_p` impide que el sorteo caiga en la basura.
>
> Y el límite: con `temp=3` ni `top_p=0.05` alcanza. Hay temperaturas que no se pueden rescatar.

### 2.7 ⚠️ Cuatro problemas del lab que conviene conocer

Audiencia: 🔧

> [!danger] 🚨 (1) El lab afirma determinismo y su propio output lo refuta
> Tras correr tres veces el mismo prompt con `top_p = 0`, el texto del notebook afirma que las salidas son *"exactamente iguales"*. **No lo son** — las tres difieren visiblemente (una abre `"RAG is a technique..."`, las otras `"RAG (Retrieval Augmented Generation) is a technique..."`, con verbos y cierres distintos). Lo mismo ocurre con `top_k = 0`.
>
> La causa probable: **el endpoint ignora o acota `top_p=0` / `top_k=0`** en vez de forzar greedy decoding. El punto pedagógico central de la sección queda demostrado con evidencia que lo contradice.
>
> **La lección real, que es más útil:** no confíes en `top_p=0` ni `top_k=0` para obtener determinismo contra un endpoint que no controlas. **Verifícalo empíricamente** corriendo el mismo prompt tres veces. Si necesitas determinismo de verdad, `temperature = 0` es el mecanismo, y aun así conviene comprobarlo.

> [!warning] ⚠️ (2) `top_k = 0` no significa lo que el lab dice
> El notebook lo presenta como una vía al determinismo. En la convención estándar (HuggingFace, Together), **`top_k = 0` DESACTIVA el filtro** — es decir, considera *todo* el vocabulary. Es lo **opuesto** al determinismo, y el propio output del lab (tres respuestas distintas) es consistente con esa lectura, no con la del texto.

> [!warning] ⚠️ (3) Las etiquetas de `repetition_penalty` no coinciden con los valores enviados
> ```python
> results = [... for r in [None, 1.2, 2]]                    # valores ENVIADOS
> for i,(result, rp) in enumerate(zip(results, [0.3,1.5,3])): # valores IMPRESOS
> ```
> La lista `[0.3, 1.5, 3]` parece copiada de la celda de temperature. El output anuncia *"Repetition Penalty = 0.3 / 1.5 / 3"* cuando en realidad se usó `None / 1.2 / 2`. Además `repetition_penalty = 0.3` sería semánticamente absurdo: por convención, **valores < 1 premian la repetición** en vez de castigarla.
>
> Dato honesto que sale de mirar los outputs reales: con `1.2` y `2` **el texto no colapsó** — sigue siendo perfectamente legible, con solo tres artefactos léxicos (una palabra en chino, una en portugués, un bullet mal formado). El notebook anticipa un desastre que sus propios datos no muestran.

> [!note] (4) El truco del cache-buster, que revela algo importante
> Varias celdas usan `max_tokens = 500 + random.randint(1,200)` con el comentario de que sirve *"para saltarse el sistema de caché"*. La implicación que el lab no explicita: **sin variar algún parámetro, dos llamadas idénticas devuelven la respuesta cacheada.**
>
> Es un detalle crítico para cualquiera que mida variabilidad: si corres el mismo prompt dos veces y obtienes lo mismo, **puede ser el caché y no el determinismo del modelo**. Es exactamente la clase de confusión que produjo el problema (1).

---

## 3. 🎯 Elegir tu LLM

Audiencia: 🔧 🧭 👔

**🔧 El planteamiento:** hay una enorme variedad de LLMs con distintos niveles de rendimiento, capacidades y perfiles de costo. Elegir bien impacta **velocidad, calidad y presupuesto**.

### 3.1 Las métricas fáciles de cuantificar

| Métrica | Qué dice | Matiz importante |
|---|---|---|
| **Model size** | Miles de millones de parámetros. Pequeños: 1–10 B · Grandes: 100–500 B o más | Los grandes son **típicamente, pero no siempre**, más capaces — y **siempre** más caros de correr |
| **Costo** | Precio por millón de tokens, a veces distinto para input y output | Los más nuevos, grandes y capaces cuestan más |
| **Context window** | Máximo de tokens que puede procesar, **repartido entre prompt y completion** | Un límite grande da flexibilidad, pero **igual pagas por token** |
| **Time to first token** y **tokens/segundo** | Latencia y velocidad | Si tu RAG es interactivo, puede valer sacrificar otras áreas por un modelo rápido |
| **Training / knowledge cutoff** | Último punto temporal representado en el entrenamiento | **Incluso en un RAG**, un cutoff más reciente se considera preferible |

> [!note] Por qué el cutoff sigue importando aunque tengas RAG
> Es contraintuitivo: si le inyectas los documentos, ¿para qué importa lo que el modelo sabía? Porque el modelo necesita **conocimiento del mundo** para interpretar lo que le entregas. Un modelo con cutoff antiguo puede no entender los términos, productos o marcos regulatorios que aparecen en tus documentos recientes.

### 3.2 Los tres tipos de benchmark

**🔧 El problema:** lo que más importa es la **calidad**, y es lo más difícil de cuantificar — desde razonar problemas matemáticos hasta simplemente producir texto agradable de leer. Hay tres familias:

| Tipo | Cómo funciona | Ejemplo | Fortaleza / debilidad |
|---|---|---|---|
| **Automated** | Tareas evaluables con código: test de opción múltiple, desafíos de matemática o programación | **MMLU** (*Massive Multitask Language Understanding*): 57 materias, de STEM a humanidades y derecho | Escalable y reproducible · no captura calidad subjetiva |
| **Human scoring** | Dos LLMs anónimos responden el mismo prompt; evaluadores humanos eligen cuál prefieren. Los resultados alimentan el **algoritmo Elo** — el mismo que rankea ajedrecistas | **LLM Arena**, uno de los benchmarks más citados | Captura matices que lo automático no puede · caro y lento |
| **LLM-as-a-judge** | Un LLM puntúa las respuestas de otro contra un set de respuestas de referencia, dando un *win rate* | — | **Barato y flexible** · el juez debe calibrarse (ver abajo) |

> [!danger] 🚨 El sesgo de familia en LLM-as-a-judge
> *"Los modelos GPT de OpenAI preferirán otros modelos GPT. Los modelos Gemini de Google preferirán otros modelos Gemini."*
>
> Es un sesgo documentado y concreto: **el juez tiende a preferir respuestas de su propia familia de modelos**. Se puede reducir recalibrando el modelo evaluador, pero si vas a usar LLM-as-a-judge —y en el [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]] lo vas a usar mucho— **no elijas como juez un modelo de la misma familia que el evaluado**.

### 3.3 Qué hace bueno a un benchmark

Audiencia: 🔧 🧭

Cuatro criterios del curso:

1. **Relevante para tu proyecto.** Si tu aplicación nunca va a generar código, compararlos en un benchmark de código no sirve de nada.
2. **Difícil.** Si todos los modelos puntúan bien, el benchmark no diferencia nada.
3. **Reproducible.** Los scores no deben cambiar drásticamente entre corridas, y los resultados que citan los proveedores deben ser verificables. El quiz del módulo da una precisión útil: **toda la variabilidad debería venir de la aleatoriedad del modelo, no del método de cálculo.** Si repetir la evaluación cambia el número por cómo se computa, el benchmark no mide nada.
4. **Alineado con el rendimiento real.** Un LLM que puntúa bien en programación debería **escribir buen código en la práctica**. Aquí el curso da un consejo poco habitual y muy sensato: *"puede que necesites leer foros de desarrolladores para asegurarte de que los scores son un buen indicador del rendimiento real"*.

> [!warning] ⚠️ Data contamination
> Los LLMs se entrenan con billones de tokens raspados de internet. **Es posible que el dataset del benchmark esté incluido en esos datos de entrenamiento.** En ese caso el modelo sobre-rinde porque **ya vio las preguntas y las respuestas exactas** durante su entrenamiento. Es la razón principal por la que un score alto no siempre se traduce en calidad real.

### 3.4 La saturación: por qué tu elección es temporal

```
   score del
   benchmark
      100 │                    ▁▂▄▆████████  ← SATURADO: casi todos los
          │                 ▄▆                 modelos puntúan al máximo,
          │            ▂▄▆                     el benchmark ya no diferencia
          │       ▁▂▄▆                         → hay que crear uno nuevo
          │   ▂▄▆                                (que también se saturará)
        0 │▂▄▆
          └──────────────────────────────────► tiempo (pocos años)
```

> [!important] 🎯 La conclusión ejecutiva de esta sección
> *"Los modelos que se lanzan hoy son usualmente significativamente mejores que los modelos de hace un par de años, y **cualquier modelo que elijas hoy probablemente necesitará ser reemplazado** a medida que se introduzcan modelos más capaces."*
>
> Traducción para arquitectura: **elegir el LLM es una decisión importante pero temporal.** Diseña el sistema para que **cambiar de modelo sea barato** — abstrae la llamada, no acoples prompts a un modelo específico, y mantén un set de evaluación que te permita comparar el candidato nuevo contra el actual ([[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09]]). Planifica el reemplazo desde el día uno.

---

## 4. ✍️ Prompt engineering

Audiencia: 🔧 🧭

**🔧 Definición:** *prompt engineering* es un término paraguas para una variedad de técnicas de prompting que tienden a producir resultados de mayor calidad.

### 4.1 El formato messages

**🔧 Definición técnica:** el formato más común es el **messages format de OpenAI**, que estructura los prompts como una serie de mensajes en JSON. Cada mensaje tiene `content` (el texto) y `role`:

| Role | Qué contiene |
|---|---|
| **`system`** | Instrucciones de alto nivel que influyen el comportamiento general del modelo |
| **`user`** | Prompts que el usuario del sistema ya envió |
| **`assistant`** | Respuestas que el LLM generó previamente |

```python
messages = [
    {"role": "system",    "content": "You are a very ironic, but helpful assistant."},
    {"role": "user",      "content": "Explain what a Cyclotomic Polynomial is. Max 5 sentences."},
]
```

> [!important] 🎯 El LLM no recuerda nada
> Esto desarma un malentendido muy extendido. Cuando tienes una conversación de ida y vuelta con un LLM (**multi-turn conversation**), *"en realidad no está recordando lo que dijiste antes"*. Detrás de escena, **la conversación completa se convierte a este formato messages**, con tu mensaje nuevo al final, y **toda la conversación se envía al LLM con cada prompt nuevo**.
>
> Consecuencias directas, y las dos importan:
> - **El costo de una conversación crece con cada turno**, porque reenvías todo el historial cada vez.
> - **El context window se consume** con la conversación, no solo con los documentos recuperados (sección 5.4).

**🔧 Un nivel más abajo:** ese objeto JSON se convierte en **un único string de texto** que el modelo procesa. Ese *chat template* usa etiquetas especiales (flechas, barras verticales) para marcar el inicio y fin de cada mensaje. **Los LLMs se entrenan para reconocer esas etiquetas** y distinguir entre mensajes de sistema, usuario y asistente.

> [!tip] 💡 Por qué esto explica el bug que perseguimos hace tres tomos
> Si el modelo distingue los roles por **delimitadores**, entonces la delimitación **dentro** del contenido también importa. Es exactamente por eso que el bug de los f-strings concatenados sin separador (`knowledge.The news data...Query:`) que apareció en los assignments C1M1 y C1M3 no es cosmético: **estás borrando la frontera que el modelo usa para saber dónde termina tu instrucción y dónde empieza el contenido.**

### 4.2 El system prompt

**🔧 Definición:** provee al LLM instrucciones de alto nivel sobre cómo debe comportarse. Si quieres que siempre hable en un tono particular o siga ciertos procedimientos, **ahí va esa información**.

El curso hace algo muy útil: muestra el system prompt real de un chatbot popular. Lo que salta a la vista:

```
   ① Es ENORME. Múltiples páginas.
      No siempre necesitas tanto, pero saber que tienes esa
      flexibilidad es un buen recordatorio.

   ② Al inicio: el knowledge cutoff del modelo y LA FECHA ACTUAL.
      Le permite al LLM determinar cuán desactualizada está su
      información y si está en posición de responder ciertas cosas.

   ③ Después: el PROCESO y el TONO.
      · razonar paso a paso
      · no ayudar con pedidos potencialmente dañinos
      · responder en markdown

   ④ Y personalidad: "es intelectualmente curioso y disfruta
      escuchar lo que los humanos piensan".
```

> [!tip] 🧭 Qué poner en el system prompt de un RAG
> El curso es concreto. Puedes instruir al modelo a:
> - **Usar solo los documentos recuperados** para responder los prompts
> - **Juzgar si un documento es relevante**
> - **Citar las fuentes** en su respuesta
> - Responder con gran detalle, o de forma sucinta
>
> Y la razón para invertir tiempo aquí: *"los system prompts usualmente se añaden a cada prompt que tu LLM procesará, así que refinarlos es una gran forma de mejorar el estilo y la calidad de los resultados"*. Es la pieza de mayor apalancamiento del sistema: **una línea bien escrita afecta todas las respuestas**.

### 4.3 La plantilla de prompt

**🔧 Definición:** el prompt aumentado incluye potencialmente muchas piezas de información, así que conviene construir una **plantilla bien pensada** que fije la estructura y decida dónde se inyecta cada contenido.

```
   ┌─────────────────────────────────────────────────────┐
   │  ① SYSTEM PROMPT                                    │
   │     guía de alto nivel sobre cómo comportarse       │
   ├─────────────────────────────────────────────────────┤
   │  ② HISTORIAL (si soportas multi-turn)               │
   │     mensajes previos entre usuario y LLM            │
   ├─────────────────────────────────────────────────────┤
   │  ③ CONTEXTO RECUPERADO                              │
   │     los top 5–10 chunks del retriever               │
   │     + instrucciones de cómo procesarlos             │
   ├─────────────────────────────────────────────────────┤
   │  ④ EL PROMPT MÁS RECIENTE del usuario               │
   └─────────────────────────────────────────────────────┘
```

**Y la razón práctica de usar plantilla:** *"hace fácil experimentar con distintas estructuras de prompt. Puedes modificar componentes individuales del prompt general y ver cómo impacta la respuesta final"*. Sin plantilla, cada experimento es una reescritura.

---

### 4.4 💻 El LLM como clasificador y la salida estructurada

Audiencia: 🔧

Dos patrones del **Ungraded Lab 2** que aparecen en casi todo sistema RAG de producción, y que son prompt engineering aplicado.

**Patrón 1 — El LLM como clasificador (router).** El escenario del lab: una tienda que vende ropa *y* suplementos, y hay que decidir a qué base consultar. Los tres consejos del propio lab:

```
   1. Sé PRECISO. Explica exactamente qué quieres que haga y qué devuelva.
   2. Añade EJEMPLOS, con su resultado esperado.
   3. Añade ejemplos DIFÍCILES — los que sabes que le costará decidir.
```

Y el tercero es el que marca la diferencia. Los ejemplos "difíciles" del lab enseñan una regla implícita —**manda el sustantivo principal, no el modificador**— sin enunciarla:

```
   "Best weight loss products that are stylish"   → Nutritional  (no Outfit)
   "Athletic wear that boosts performance"        → Outfit       (no Nutritional)
```

El lab obtiene **9 de 9 aciertos** en su set de prueba con `max_tokens = 2`.

> [!warning] ⚠️ Tres cosas que el lab hace mal y conviene no copiar
> **(1) No normaliza la salida.** Compara `result == expected_label` sin `.strip()` ni `.lower()`, con `max_tokens=2`. Si el modelo emite un espacio o un salto de línea antes de la etiqueta, la comparación falla y **el motivo no es visible**. Lo irónico: el propio notebook aconseja *"implementar checks en tu código para evitar recibir un valor inesperado"* — y no los implementa.
>
> **(2) El clasificador corre sin fijar `temperature`.** Para una tarea cuya tesis es *"minimizar la probabilidad de outputs inesperados"*, dejar la temperatura al default del proveedor es incoherente. **Los 9/9 no son reproducibles.** Un router debe correr con `temperature = 0`.
>
> **(3) Instrucciones ambiguas contra un test rígido.** El prompt pide las etiquetas en minúscula (`"nutritional"` / `"outfit"`) pero los ejemplos y la comparación usan capitalizadas. Si el modelo obedece la instrucción literal, falla el test.
>
> **La versión robusta:** `temperature=0`, normalizar con `.strip()`, y **validar contra el conjunto permitido** con un fallback explícito — como sí hace el assignment (`if label not in ['FAQ','Product']: label = None`).

**Patrón 2 — Parámetros condicionados a la tarea.** Un router decide la naturaleza de la query y de ahí salen los parámetros de sampling. Es la sección 2 de este tomo aplicada:

| Naturaleza | `temperature` | `top_p` |
|---|---|---|
| **Technical** (datos, precios, disponibilidad) | 0.3 | 0.7 |
| **Creative** (proponer looks, redactar) | 1.0 | 0.9 |
| **Fallback** (etiqueta inválida) | 0.3 | 0.7 |

*(valores del assignment graded C1M4)*

> [!tip] 🧭 La regla numérica que da el curso
> El assignment es explícito: *"una temperature demasiado alta llevará al modelo a resultados sin sentido, así que **mantenla por debajo de 1.3**, y si está cerca de 1.3, asegúrate de **bajar el `top_p`**"*. Es exactamente el mecanismo de rescate que el Lab 1 demostró empíricamente (sección 2.6): `top_p` acota el daño de una temperature alta.
>
> Cuesta **dos llamadas al LLM por respuesta** (router + generador). Es el precio de que un mismo sistema pueda ser factual y creativo según lo que le pregunten — el caso de negocio del final de este tomo.

**Patrón 3 — Salida estructurada de verdad.** Para que el output alimente otro sistema hay dos caminos, y el lab muestra ambos:

```python
# ── Camino A: pedirlo en el prompt (frágil)
#    Un prompt largo con ejemplos few-shot del JSON deseado.

# ── Camino B: imponerlo por schema (robusto)
from pydantic import BaseModel, Field

class VoiceNote(BaseModel):
    title:       str       = Field(description="A title for the voice note")
    summary:     str       = Field(description="A short one sentence summary.")
    actionItems: list[str] = Field(description="A list of action items")

response_format = {
    "type":   "json_schema",
    "schema": VoiceNote.model_json_schema(),      # ← el schema sale del modelo Pydantic
}

result = generate_with_multiple_input(messages, response_format=response_format)
```

> [!danger] 🚨 El caso de estudio más valioso del lab: un few-shot mal escrito se propaga al output
> El prompt de domótica del lab (convertir lenguaje natural a JSON) contiene errores en sus **propios ejemplos**, y el modelo los **replicó fielmente**:
>
> | Error en el few-shot | Consecuencia |
> |---|---|
> | La spec declara `volume (integer)` y `temperature (integer)`, pero los ejemplos usan **strings** (`'volume': '100'`) | El modelo devolvió `"temperature": "24"` y `"volume": "100"` — **strings, no integers** |
> | Un ejemplo usa **comillas simples** de Python dentro del JSON (`{'color': 'yellow'}`) | JSON inválido. En esta corrida el modelo acertó, pero se le está enseñando a emitir JSON no parseable |
> | Un ejemplo usa `"action": "turn on"` sobre el speaker | Acción que **no está** en la lista permitida para ese dispositivo (`play`, `pause`, `stop`, `set volume`) |
> | Dos ejemplos numerados `2.`, sin `3.` | El modelo ve una lista mal formada |
>
> **La lección, que vale para cualquier few-shot:** el modelo aprende de tus ejemplos **incluido lo que hiciste mal**. Un ejemplo con el tipo equivocado enseña el tipo equivocado. Antes de confiar en un prompt few-shot, **valida tus propios ejemplos contra el schema que dices querer**.

> [!warning] ⚠️ Y una omisión que desperdicia media herramienta
> El lab dice que usa Pydantic *"para ayudarlo a validar la estructura de datos"*, pero el código **solo genera el schema** (`model_json_schema()`) y **nunca valida la respuesta** (`VoiceNote.model_validate(...)` no aparece). Tampoco envuelve el `json.loads` en un `try/except`.
>
> El valor de Pydantic aquí es doble: **describe** el contrato *y* **verifica** que la respuesta lo cumpla. Usar solo la primera mitad deja el sistema sin red justo donde más se necesita:
>
> ```python
> import json
> try:
>     nota = VoiceNote.model_validate(json.loads(result['content']))   # ← valida de verdad
> except Exception:
>     ...   # reintentar, o degradar con gracia
> ```
>
> Nota de portabilidad: la forma `{"type": "json_schema", "schema": ...}` es la de **Together**; OpenAI usa `{"type":"json_schema","json_schema":{"name":..., "strict":true, "schema":...}}`. El código del lab no es portable tal cual.

---

## 5. 🚀 Prompting avanzado

Audiencia: 🔧 🧭

### 5.1 In-context learning

**🔧 Definición técnica:** ayudar al LLM a aprender qué tipo de output quieres **añadiendo ejemplos al prompt**. Si construyes un chatbot de servicio al cliente, el prompt puede incluir ejemplos de pedidos previos de clientes **junto con respuestas de alta calidad** a esos pedidos. Los ejemplos le enseñan al modelo la estructura y el tono.

```
   1 ejemplo   → one-shot learning
   varios      → few-shot learning
```

> [!important] 🎯 Y aquí ocurre algo elegante: RAG para recuperar los ejemplos
> Hay dos formas de implementarlo:
> 1. **Hardcodear** uno o más ejemplos en el prompt. Si quieres comportamientos estables, esto solo puede mejorar la calidad.
> 2. **Usar RAG para recuperar los ejemplos.** Indexas conversaciones exitosas con clientes en tu vector database; cuando llega una consulta nueva sobre un tema, **recuperas conversaciones previas sobre ese mismo tema** y las inyectas en el prompt.
>
> El curso lo dice bien: *"en muchos sentidos, esto es solo RAG normal. Pero el hecho de que estés recuperando específicamente **ejemplos de respuesta** puede mejorar aún más la calidad."* Es el mismo mecanismo del [[Guia-Maestra-RAG_01-Introduccion-a-RAG|Tomo 01]] aplicado a un objetivo distinto: en vez de recuperar **hechos**, recuperas **formas de responder**.

### 5.2 Razonamiento paso a paso

**🔧 Dos técnicas emparentadas:**

| Técnica | Cómo |
|---|---|
| **Scratchpad** | Decirle al LLM que piense en voz alta antes de responder. Una forma común: indicarle que **los tokens entre etiquetas `<scratchpad>`** son espacio para pensar y brainstorming, y **no son parte de su respuesta final** |
| **Chain-of-thought** | Instruirlo a abordar la pregunta **paso a paso** en vez de responder de inmediato: primero generar los pasos necesarios, después seguirlos |

**Dos beneficios**, y el segundo se menciona menos:
1. Aumenta la probabilidad de que las respuestas finales sean **más precisas**.
2. Como el LLM **muestra su trabajo**, es **más fácil rastrear los problemas** cuando su razonamiento se cae. Es debuggability, no solo calidad.

### 5.3 Reasoning models — y la advertencia que sorprende

**🔧 Definición:** las estrategias de razonamiento fueron tan exitosas que **muchos LLMs ahora se diseñan como reasoning models desde fábrica**. Destacan en tareas complejas: código, matemática, planificación, puzzles, y flujos que requieren múltiples pasos.

**Por dentro:** primero generan **reasoning tokens** (donde planifican y consideran opciones, como el scratchpad), y después **response tokens** con la respuesta final. Algunos proveedores solo dan acceso a los segundos; otros permiten ver ambos.

> [!warning] ⚠️ Los reasoning tokens son tokens: los pagas
> *"Esos reasoning tokens son parte de lo que hace a estos modelos más precisos. Pero siguen siendo tokens normales con todos los costos asociados a generarlos."* Los reasoning models son **más lentos y más caros de correr**. La pregunta no es si son mejores: es si en tu caso el aumento de precisión justifica el aumento de costo por llamada.

> [!danger] 🚨 Muchas técnicas de prompt engineering NO funcionan bien con reasoning models
> Este es el punto contraintuitivo de la lección, y hay que tenerlo presente porque invalida buena parte de lo aprendido en las secciones anteriores:
>
> | Técnica | Con un reasoning model |
> |---|---|
> | *"Piensa paso a paso"* | **Innecesario** — ya está entrenado para hacerlo |
> | **In-context learning** | **Puede funcionar mal**: el modelo intenta incorporar los ejemplos provistos a la pregunta que está respondiendo |
> | Objetivos específicos y formato exacto | **Sí funcionan mejor** |
> | Principios guía de alto nivel y enfoques a evitar | Sí, se pueden dar |
> | Volcado completo del contexto recuperado | **Sí** — con estos modelos puedes simplemente darles todos los documentos |
>
> Y para qué son especialmente buenos en un RAG: **evaluar la relevancia** de los documentos recuperados y **decidir cómo incorporar** esa información en una respuesta que requiere pasos complejos.
>
> Regla operativa: **consulta la documentación del proveedor sobre cómo promptear cada modelo.** Lo que funciona en uno puede degradar al otro.

### 5.4 Gestión del context window y context pruning

Audiencia: 🔧 🧭 👔

**🔧 El problema:** el prompt inicial **y** los tokens que el LLM genera consumen porciones del context window. Y todas las técnicas avanzadas lo agravan:

```
   inyectar documentos del retriever      ─┐
   añadir ejemplos de in-context learning  ├──► alargan el prompt,
   un reasoning model planificando         │    la respuesta, o ambos
   el historial de una conversación       ─┘

   → "es fácil llenar rápidamente tu context window si no estás
      prestando atención"
```

**Las soluciones, según el tipo de conversación:**

**Single-turn:** *"el mejor arreglo es simplemente validar que estás obteniendo valor de tu técnica de prompt engineering. Si chain-of-thought o in-context learning no te está dando mejor rendimiento, es mejor quitar esos componentes del sistema."* Sencillo y honesto: si no lo mediste, no sabes si esa técnica te está costando context window a cambio de nada.

**Multi-turn** — aquí entra el **context pruning**:

| Técnica | Cómo |
|---|---|
| **Ventana fija** | Mantener solo los últimos N mensajes (por ejemplo, los 5 últimos del usuario y del LLM) |
| **Resumen con LLM** | Un LLM aparte **resume los mensajes viejos**, reduciendo su tamaño pero preservando los puntos clave |
| **Descartar reasoning tokens** | Si usas un reasoning model en multi-turn, *"casi con certeza querrás descartar los reasoning tokens del historial y quedarte solo con los response tokens"* |
| **Solo los chunks de la última pregunta** | En un RAG, *"usualmente solo quieres incluir los chunks recuperados para responder la pregunta más reciente, no de cada pregunta anterior"* |

> [!tip] 🧭 Y la opción que siempre está sobre la mesa
> *"Por supuesto, si tu aplicación necesita conversaciones multi-turn con contexto profundo y rico, siempre puedes cambiar a un modelo con un context window más largo. Dicho eso, **igual querrás ser cuidadoso con cómo diseñas los prompts**, porque incluso en modelos con context windows largos, **los prompts largos son lentos y caros de correr**."*
>
> Es el mismo argumento del [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG#2.8 Context window: por qué no puedes meter todo|Tomo 02]]: un context window grande no elimina el problema, lo hace más tolerante.

### 5.5 El consejo de cierre del curso

> [!important] 🎯 No necesitas todo esto
> *"Las técnicas de prompt engineering pueden mejorar el rendimiento de tu LLM, pero tu sistema RAG **no necesariamente necesita emplearlas**. Una plantilla de prompt simple y un system prompt bien escrito **podrían ser todo lo que tu proyecto necesita**. Cuando se trata de técnicas más avanzadas, te aconsejo que las añadas a tu proyecto **solo después de que sea claro que las necesitas**."*
>
> Y la frase que resume el tomo: *"promptear en general puede ser más un arte que una ciencia"* — así que experimenta y encuentra lo que funciona en **tu** sistema.

---

> [!example] 📊 Caso de negocio — Educación: un solo producto, dos perfiles de sampling
> **Problema:** una plataforma de aprendizaje despliega un asistente con dos funciones sobre el mismo corpus de material curricular: **(a)** explicar conceptos y responder dudas de contenido, y **(b)** generar ejercicios de práctica variados. Usando la misma configuración para ambas, ninguna funciona bien. Con los defaults del proveedor, las explicaciones **varían entre sesiones** —dos alumnos preguntan lo mismo y reciben respuestas distintas, y a veces una desliza una fórmula que no está en el material— mientras que los ejercicios salen **repetitivos**: variaciones mínimas del mismo enunciado, que los alumnos detectan de inmediato.
>
> **Técnica aplicada:** se separan **dos perfiles de sampling** para las dos funciones, que es el punto de este tomo. Para las **explicaciones**: `temperature` baja y `top_p` bajo — el criterio explícito del curso para preguntas factuales — más un system prompt que instruye a responder **solo** con el material recuperado y a decir que no está cubierto cuando no lo está. Para los **ejercicios**: `temperature` más alta acompañada de `top_p` acotado, la combinación que el Ungraded Lab 1 demostró como el punto dulce creativo (sección 2.6) — variedad real sin la ensalada incoherente que produce subir la temperature sola. Se añade `repetition_penalty` solo en la rama de ejercicios, y de forma reactiva, tras observar enunciados repetidos. Para los ejercicios de matemática con varios pasos se evalúa un **reasoning model**, aceptando su mayor costo por llamada solo en esa rama.
>
> **Resultado:** el mismo modelo y el mismo corpus sirven dos comportamientos deliberadamente distintos. Las explicaciones se vuelven **estables y verificables**; los ejercicios, genuinamente variados. Y un efecto secundario que cambió la operación: al fijar la temperature baja en la rama factual, las respuestas se volvieron **reproducibles**, lo que permitió por primera vez que un profesor revisara y aprobara una respuesta sabiendo que el siguiente alumno recibiría la misma. La lección: **la configuración de sampling no es un ajuste global del sistema, es una decisión por caso de uso** — y en un mismo producto pueden convivir varias.

---

## 6. 🧭 Guía de decisión del tomo

Audiencia: 🔧 🧭

| Situación | Qué hacer |
|---|---|
| Punto de partida general | `temperature=0.8`, `top_p=0.9`, `repetition_penalty=1.2` |
| Preguntas factuales, RAG con citas, código | `temperature` y `top_p` **bajos** |
| Dominio creativo | Ambos **más altos** — pero acompaña la temperature con `top_p` |
| Necesitas determinismo | `temperature = 0`. **Y verifícalo**: corre el mismo prompt tres veces |
| El sistema debe elegir entre N categorías | **Logit bias** sobre esos N tokens |
| Texto repetitivo | `repetition_penalty` — de forma **reactiva**, no preventiva |
| Respuestas que se alargan sin sentido | Baja la temperature: el token de fin recuperó probabilidad |
| El formato de salida es difícil de describir | **In-context learning** con ejemplos de los casos difíciles |
| Tarea con varios pasos lógicos | Chain-of-thought, o directamente un **reasoning model** |
| Usas un reasoning model | **Quita** el "piensa paso a paso" y el in-context learning; da objetivos y formato |
| Conversación multi-turn que crece | **Context pruning**: ventana fija, resumen, o descartar reasoning tokens |
| Eligiendo modelo | Cuantificables para descartar, benchmarks **relevantes a tu caso** para decidir |
| **Siempre** | Abstrae la llamada al LLM: **vas a cambiar de modelo** |

> [!tip] 🧭 El orden correcto de trabajo
> System prompt bien escrito + plantilla simple → medir → fijar `temperature` y `top_p` según el caso de uso → y **solo entonces** evaluar técnicas avanzadas, cada una contra la pregunta *"¿me está dando algo a cambio del context window que consume?"*. La sofisticación se gana; en prompting, más que en ninguna otra capa, se gana con experimentación.

---

## 7. 📖 Glosario express de este tomo

Audiencia: 👔 💡

| Término técnico | Traducción a lenguaje de negocio |
|---|---|
| **Transformer** | La arquitectura sobre la que se construyen los LLMs y los embedding models |
| **Encoder / decoder** | Comprender texto / generar texto. Los LLMs son casi solo decoder |
| **Attention** | Qué otras palabras influyen en el significado de cada palabra |
| **Attention head** | Una "lectura" de la frase con un criterio propio. Los modelos tienen decenas o cientos |
| **Feedforward** | La parte más grande del modelo: donde vive la mayoría de los parámetros |
| **Greedy decoding** | Elegir siempre la palabra más probable. Determinista y monótono |
| **`temperature`** | El dial de creatividad. Bajo = predecible y factual; alto = variado y riesgoso |
| **`top_k`** | Elegir solo entre las k palabras más probables. Pool de tamaño fijo |
| **`top_p`** | Elegir según probabilidad acumulada. Se **adapta** a la confianza del modelo |
| **Repetition penalty** | Castigar palabras ya usadas para que el texto no suene repetitivo |
| **Logit bias** | Subir o bajar a mano la probabilidad de palabras concretas |
| **Messages format** | La estructura `system` / `user` / `assistant` con que se arma un prompt |
| **System prompt** | Las instrucciones que gobiernan **todas** las respuestas. La pieza de mayor apalancamiento |
| **In-context learning** | Enseñarle con ejemplos dentro del prompt (one-shot / few-shot) |
| **Chain-of-thought** | Pedirle que razone paso a paso antes de responder |
| **Reasoning model** | Modelo que piensa antes de responder de fábrica. Más preciso, más lento, más caro |
| **Context pruning** | Recortar el historial de conversación para que quepa en el context window |
| **Data contamination** | Que el benchmark estuviera en los datos de entrenamiento. Infla los scores |
| **Saturación** | Cuando todos los modelos puntúan al máximo y el benchmark deja de servir |

---

## 8. ✅ Checklist de comprensión

Audiencia: 🔧 🧭 👔

- [ ] Sé por qué los LLMs son solo el decoder y los embedding models usan el transformer completo.
- [ ] Puedo describir el viaje de un prompt: tokenización → embedding → posición → attention → feedforward → repetir → distribución → sorteo.
- [ ] Entiendo qué es un attention head y por qué hay decenas o cientos.
- [ ] Sé enunciar las tres consecuencias de la arquitectura para RAG, y que **el costo viene de los tokens**.
- [ ] Distingo greedy decoding, `temperature`, `top_k` y `top_p`, y sé qué modifica cada uno.
- [ ] Sé por qué `top_p` se adapta a la confianza del modelo y `top_k` no.
- [ ] Entiendo que `temperature = 1` **no** es determinista.
- [ ] Sé que subir la temperature también **alarga la respuesta y sube la latencia**, y por qué.
- [ ] Conozco la configuración de arranque del curso y cómo ajustarla según el caso de uso.
- [ ] Sé que no hay que confiar en `top_p=0` / `top_k=0` para determinismo, y por qué el lab lo demuestra al revés.
- [ ] Entiendo que el LLM **no recuerda**: la conversación completa se reenvía en cada turno.
- [ ] Sé qué poner en un system prompt de un RAG y por qué es la pieza de mayor apalancamiento.
- [ ] Conozco los tres tipos de benchmark y el sesgo de familia del LLM-as-a-judge.
- [ ] Entiendo data contamination y saturación, y por qué la elección de modelo es temporal.
- [ ] Sé que in-context learning puede recuperarse **con RAG**.
- [ ] Tengo claro que varias técnicas de prompting **no funcionan bien con reasoning models**.
- [ ] Conozco las cuatro técnicas de context pruning.

---

## 🔗 Conexiones

- Tomo anterior → [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07 · Query parsing, arquitecturas y re-ranking]] (cierre del retriever)
- Siguiente tomo → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Hallucinations, evaluación y agentic RAG]] (cómo saber si todo esto funciona)
- **El comportamiento que este tomo explica a nivel de arquitectura** → [[Guia-Maestra-RAG_02-Fundamentos-LLMs-y-Pipeline-RAG|Tomo 02 · Fundamentos]]
- El otro uso del transformer (embedding models) → [[Guia-Maestra-RAG_04-Semantic-Search-y-Embeddings|Tomo 04]]
- Qué se le entrega al LLM y en qué formato → [[Guia-Maestra-RAG_06-Chunking|Tomo 06]] y [[Guia-Maestra-RAG_07-Reranking-Cross-Encoders-y-ColBERT|Tomo 07]]
- Costo, latencia y quantization en producción → [[Guia-Maestra-RAG_10-RAG-en-Produccion|Tomo 10]] · [[Guia-Maestra-RAG_11-Quantization-Trade-offs-y-Multimodal-RAG|Tomo 11]]
- Índice general → [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]]

---

## 📚 Referencias

**Fuente primaria (curso):**
- DeepLearning.AI. *Retrieval-Augmented Generation (RAG)* — Módulo 4: LLMs & Text Generation (Coursera). Lecciones de arquitectura transformer, sampling, elección de LLM, prompt engineering y prompt engineering avanzado; **Ungraded Lab 1** (sampling), del que provienen los outputs y valores de la sección 2.6; **Ungraded Lab 2** (prompt engineering), del que provienen los patrones de la sección 4.4; y el **assignment graded C1M4**, del que provienen la tabla de parámetros por tarea y la regla de mantener `temperature < 1.3`.

**Fuentes externas (complemento con bibliografía verificable):**
- Vaswani, A. et al. (2017). *Attention Is All You Need*. NeurIPS. — El paper del transformer que la lección cita por título.
- Holtzman, A., Buys, J., Du, L., Forbes, M. & Choi, Y. (2020). *The Curious Case of Neural Text Degeneration*. ICLR. — El paper de **nucleus sampling (top_p)**.
- Fan, A., Lewis, M. & Dauphin, Y. (2018). *Hierarchical Neural Story Generation*. ACL. — Origen del **top-k sampling**.
- Brown, T. et al. (2020). *Language Models are Few-Shot Learners*. NeurIPS. — In-context learning / few-shot.
- Wei, J. et al. (2022). *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*. NeurIPS.
- Kojima, T. et al. (2022). *Large Language Models are Zero-Shot Reasoners*. NeurIPS. — El *"let's think step by step"*.
- Hendrycks, D. et al. (2021). *Measuring Massive Multitask Language Understanding* (MMLU). ICLR.
- Chiang, W.-L. et al. (2024). *Chatbot Arena: An Open Platform for Evaluating LLMs by Human Preference*. ICML. — El sistema Elo de LLM Arena y el sesgo del juez.
- Liu, N. F. et al. (2023). *Lost in the Middle: How Language Models Use Long Contexts*. TACL. — Por qué un context window grande no resuelve la gestión de contexto.

> [!note] Sobre el código y los valores de este tomo
> - **Del curso:** toda la teoría de las secciones 1 a 5 proviene de las cinco lecciones del Módulo 4. Los valores de sampling recomendados (`0.8 / 0.9 / 1.2`), los tres tipos de benchmark, las técnicas de context pruning y la advertencia sobre reasoning models son literales de las lecciones. Los outputs de la sección 2.6 son reales del Ungraded Lab 1.
> - **Hallazgos propios verificados contra los notebooks:** que el Lab 1 **afirma determinismo con `top_p=0` y sus tres salidas son distintas**; que describe `top_k=0` como mecanismo de determinismo cuando en la convención estándar **desactiva el filtro**; que las etiquetas impresas de `repetition_penalty` (`0.3/1.5/3`) **no corresponden a los valores enviados** (`None/1.2/2`); que con `1.2` y `2` el texto **no colapsó** como el notebook anticipa; y el papel del `max_tokens` aleatorio como *cache-buster*, con su implicación de que dos llamadas idénticas devuelven respuesta cacheada. Del **Lab 2**: que el clasificador corre **sin fijar `temperature`** (sus 9/9 no son reproducibles) y sin normalizar la salida, pese a que el propio notebook aconseja lo contrario; que Pydantic se usa para **generar el schema pero nunca para validar**; y el caso completo del **few-shot con JSON inválido y tipos mal declarados que el modelo replicó fielmente** en su output.
> - **Análisis propio marcado en el texto:** la conexión del chat template con el bug de separadores de C1M1/C1M3, el argumento de por qué el knowledge cutoff sigue importando con RAG, la lectura de la tabla de 2.6 (que `top_p` rescata parcialmente una temperature alta) y las guías de decisión.

> [!info] Continuará
> **Próximo tomo → [[Guia-Maestra-RAG_09-Hallucinations-Evaluacion-y-Agentic-RAG|Tomo 09 · Hallucinations, evaluación y agentic RAG]]**, donde se responde la pregunta que este tomo dejó abierta: acabamos de llenar el sistema de perillas —temperature, top_p, system prompt, modelo— y **no tenemos forma de saber si moverlas mejora o empeora las cosas**. Ese tomo trae las métricas, tanto del retriever como del generador, más la detección de hallucinations, los flujos agentic y la comparación honesta entre RAG y fine-tuning.
