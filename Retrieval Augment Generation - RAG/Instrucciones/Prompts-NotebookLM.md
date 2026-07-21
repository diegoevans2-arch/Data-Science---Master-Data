---
title: "Prompts para NotebookLM — Podcasts de la Guía Maestra de RAG"
tags: [meta, instrucciones, notebooklm, podcast, audio, rag, guia-maestra]
type: documentacion
project: guia-maestra-rag
version: 1.0
status: in-progress
author: El Egypcio
---

# 🎙️ Prompts para NotebookLM — un podcast por tomo

> [!important] Para qué sirve este archivo
> Contiene los prompts listos para pegar en **NotebookLM** y generar un *Audio Overview* (podcast) por cada tomo de la [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|Guía Maestra de RAG]], **preservando la intención comunicativa de la guía**: español con términos técnicos en inglés, triple audiencia, analogías primero y rigor técnico como piso.
>
> Este archivo crece junto con la guía: **cada tomo nuevo suma su bloque aquí.**

---

## 1. 🎯 El problema que resuelven estos prompts

Audiencia: 🧭

Si subes un tomo a NotebookLM y generas el audio sin instrucciones, pasan cuatro cosas — todas malas para esta guía en particular:

| Lo que hace por defecto | Por qué rompe la guía |
|---|---|
| **Traduce los términos técnicos** ("recuperador", "incrustación", "búsqueda híbrida") | Viola la regla crítica de la sección 4 de [[Instrucciones\|📖 Instrucciones]]. El vocabulario del ecosistema es en inglés |
| **Aplana la triple audiencia** en un registro único | Se pierde lo que distingue a la guía: que sirva al técnico *y* al ejecutivo |
| **Intenta leer código, tablas y diagramas ASCII** | No funciona en audio. Suena a ruido y quema minutos |
| **Busca cobertura completa** en vez de profundidad | Los tomos son densos; recitar todo produce un audio plano y olvidable |

Los prompts de abajo corrigen las cuatro.

---

## 2. ⚙️ Cómo usarlo

Audiencia: 🔧 🧭

```
   1.  Crear un notebook nuevo en NotebookLM  (uno por tomo)
              │
   2.  Subir el archivo .md del tomo como fuente
       (opcional: sumar el tomo anterior como contexto)
              │
   3.  Configurar el idioma de salida del audio → Español
              │
   4.  Audio Overview → Personalizar (Customize)
              │
   5.  Pegar el PROMPT BASE + el bloque específico del tomo
              │
   6.  Generar
```

> [!warning] Sobre el cuadro "Personalizar"
> El cuadro de personalización de NotebookLM **tiene un límite de caracteres** y la interfaz cambia con frecuencia. Por eso abajo hay **dos versiones de cada prompt**:
> - **Versión completa** — pégala si el cuadro te la acepta entera.
> - **Versión mínima** — el núcleo irrenunciable, por si tienes que recortar.
>
> Si el límite te aprieta, sacrifica primero las indicaciones de estructura y **conserva siempre la regla de los términos en inglés**: es la que más se nota si se pierde.

> [!tip] El idioma de salida se configura aparte
> Pedir "habla en español" dentro del prompt **no basta**: NotebookLM tiene un ajuste propio de idioma del audio. Configúralo antes de generar. El prompt refuerza el registro, no lo define.

---

## 3. 🧱 PROMPT BASE (común a todos los tomos)

Audiencia: 🔧 🧭

Este bloque va **siempre**. Después le agregas el bloque del tomo específico (sección 4).

### 3.1 Versión completa

```text
Español neutro profesional, tono de conversación entre colegas.

REGLA INNEGOCIABLE: NO traduzcas los términos técnicos. Pronúncialos en
inglés dentro de la frase en español: retriever, retrieval, embedding,
chunking, chunk, vector store, prompt, query, top_k, re-ranking, hybrid
search, knowledge base, hallucination, fine-tuning, context window,
grounding, keyword search, semantic search, token. Nunca digas
"recuperador", "incrustación" ni "búsqueda híbrida".

ROLES DE LOS DOS HOSTS:
- Host A pregunta como líder de producto: ¿para qué sirve?, ¿qué decisión
  habilita?, ¿qué cuesta hacerlo mal?
- Host B responde como AI engineer senior: preciso, sin diluir, pero
  traduciendo al lenguaje de negocio cuando A lo pide.

PARA CADA CONCEPTO, EN ESTE ORDEN:
1. La analogía cotidiana (sin prerrequisitos).
2. La definición técnica precisa.
3. Qué decisión de negocio habilita o qué riesgo evita.

NO leas código, tablas ni diagramas. Cuando el tomo muestre código, explica
la DECISIÓN que hay detrás, no la sintaxis.

Prioriza PROFUNDIDAD sobre cobertura: quédate en los momentos
contraintuitivos y en los errores costosos. Es preferible dejar temas fuera
antes que recitar todo el tomo.
```

### 3.2 Versión mínima

```text
Español neutro. NO traduzcas los términos técnicos: di retriever, embedding,
chunking, prompt, top_k, hybrid search, hallucination, context window en
inglés. Host A pregunta como líder de producto; Host B responde como AI
engineer senior. Cada concepto: analogía primero, después la precisión
técnica, y cierra con la decisión de negocio que habilita. No leas código ni
tablas. Prioriza lo contraintuitivo por encima de cubrirlo todo.
```

---

## 4. 📚 Bloques por tomo

Audiencia: 🔧 🧭

Cada bloque se **añade al prompt base**. Marcan el ángulo, el clímax y lo que conviene dejar fuera.

---

### 🎧 Tomo 01 — Introducción a RAG

```text
TEMA: qué es RAG y qué problema resuelve. Es el episodio de entrada: asume
cero conocimiento previo.

ARCO NARRATIVO:
1. Los tres límites de un LLM solo: knowledge cutoff, hallucinations, y que
   nunca vio tus datos privados. Usa la analogía del médico brillante que
   estudió hasta 2023 y desde entonces vive aislado en una isla sin internet.
2. Qué es RAG: darle el expediente correcto justo antes de que responda.
3. Las DOS FASES: indexing (offline, ordenar la biblioteca) vs. retrieval
   (online, consultarla). Insiste en la consecuencia: cambiar el embedding
   model obliga a re-indexar todo.

CLÍMAX DEL EPISODIO — dedícale tiempo real:
El experimento con y sin RAG sobre el PIB de Estados Unidos. Sin RAG el
modelo afirmó que "el crecimiento promedió 1.890 millones de dólares al mes"
(el PIB no se mide así) y habló de una supuesta "era Growth Oblivious" que no
existe. Con RAG dejó de inventar y declaró sus fuentes.
El giro que hay que subrayar: la respuesta SIN RAG era MÁS LARGA, MÁS
SEGURA Y MÁS AGRADABLE DE LEER. Si solo miras la superficie, gana la mala.
Cierra la idea así: una alucinación no se ve como un error, se ve como una
respuesta bien redactada.

CIERRE EJECUTIVO: cuando cambia un precio, actualizas la knowledge base.
No reentrenas nada.

DEJA FUERA: el detalle del código y la taxonomía Naive/Advanced/Modular.
```

---

### 🎧 Tomo 02 — Fundamentos: LLMs y el pipeline RAG

```text
TEMA: cómo funciona un LLM por dentro y por qué eso explica todo lo demás.

ARCO NARRATIVO:
1. Un LLM es "fancy autocomplete": solo predice el siguiente token. Explica
   que "the sun is exploding" no es un error de gramática, es improbable.
2. El loop de generación: distribución de probabilidad sobre el vocabulary y
   sorteo aleatorio. De ahí sale que el MISMO prompt dé respuestas distintas
   cada vez — y lo que eso rompe para quien viene del software determinista.
3. El retriever por dentro con la analogía del bibliotecario, y el dilema del
   top_k: traer de más ahoga en ruido y encarece; traer de menos pierde
   documentos relevantes. No hay número correcto universal: se mide.

CLÍMAX DEL EPISODIO:
La frase que lo explica todo: los LLMs están diseñados para generar texto
PROBABLE, no texto VERDADERO. La hallucination no es un bug ni una falla:
el modelo está haciendo exactamente aquello para lo que fue construido.
Usa la analogía del examen oral donde está prohibido decir "no sé": el
alumno no miente por malicia, es que su único trabajo es sonar bien.

CIERRE EJECUTIVO: entender esto cambia cómo se gobierna un proyecto de IA.
Es la diferencia entre esperar magia y diseñar controles.

DEJA FUERA: el detalle de la tokenización (BPE) y las cinco ventajas
enumeradas.
```

---

### 🎧 Tomo 03 — Keyword search: TF-IDF y BM25

```text
TEMA: la búsqueda por palabras exactas, la técnica más antigua del retriever
y todavía la más difícil de batir.

ARCO NARRATIVO:
1. Metadata filtering: la analogía es filtrar una planilla de cálculo. El
   punto contraintuitivo que hay que subrayar: los filtros NO salen de lo que
   el usuario escribió, salen de QUIÉN ES el usuario. Por eso nadie puede
   eludirlos escribiendo un prompt astuto — y por eso ahí vive el control de
   acceso del sistema.
2. Bag of words: meter las palabras en una bolsa y agitarla. "El perro mordió
   al cartero" y "el cartero mordió al perro" son idénticos para esta técnica.
3. La escalera del scoring en cuatro peldaños, cada uno arreglando el defecto
   del anterior: contar presencia, contar repeticiones, normalizar por largo,
   y finalmente pesar por rareza (IDF).
4. BM25: rendimientos decrecientes. El décimo vaso de agua no te quita la sed
   como el primero.

CLÍMAX DEL EPISODIO:
Un documento sobre pan de masa madre le ganó al documento sobre hornos de
pizza en una búsqueda sobre pizza. No fue un bug: la tokenización no quitaba
las stopwords, y resultó que "making" era más raro en ese corpus que "pizza",
así que pesaba más. Moraleja: en keyword search el preprocesamiento no es
limpieza opcional, es parte del algoritmo. Y es la demostración más nítida de
que esta técnica NO entiende de qué habla el texto.

CIERRE EJECUTIVO: si el metadata filtering está mal implementado, tu sistema
le muestra documentos confidenciales a quien no debe — y el LLM se los
redacta amablemente.

DEJA FUERA: las fórmulas, los nombres de las librerías y los valores
numéricos concretos.
```

---

### 🎧 Tomo 04 — Semantic search, embeddings y hybrid search

```text
TEMA: cómo una máquina entiende que "happy" y "glad" significan lo mismo, y
cómo se combinan las dos búsquedas.

ARCO NARRATIVO:
1. El problema: keyword search no conecta sinónimos, y en cambio sí confunde
   el Python lenguaje con el Python serpiente.
2. El vector space: cada texto es un punto, y los parecidos quedan cerca.
   Advertencia importante: los ejes NO significan nada. No hay "eje de
   comida". Solo importan las posiciones relativas.
3. Cómo se entrena: contrastive training. Usa la analogía de la fiesta con
   imanes — cada frase atrae a las parecidas y repele a las distintas, y sin
   que nadie dirija nada, la sala termina organizada en grupos temáticos.
4. Hybrid search y RRF: cómo se fusionan la lista de keyword y la de
   semantic en un ranking único.

DOS CLÍMAX — dales aire a ambos:

(a) EL CASO KYOTO. Ante "sugiere lugares para visitar en Asia", el modelo
    puso Santorini (Grecia) y Banff (Canadá) POR ENCIMA de Kyoto. La razón:
    un embedding model no sabe hechos, reproduce patrones de co-ocurrencia de
    su entrenamiento. Conclusión práctica: si necesitabas garantizar "solo
    Asia", eso era un filtro de metadata, no una esperanza depositada en el
    modelo.

(b) EL TRUNCATION SILENCIOSO. Embeber un texto completo y embeber solo sus
    primeros 3.000 caracteres devolvió EL MISMO VECTOR, idéntico. El modelo
    ignora todo lo que pase de su límite: sin error, sin warning, sin
    excepción. El contenido queda en tu knowledge base pero es
    irrecuperable. Por eso existe el chunking.

REMATE SOBRE RRF: en un caso real, el documento que era número 1 en semantic
search terminó CUARTO, y uno que no fue primero en ninguna lista terminó
PRIMERO. La razón: estar en ambas listas vale más que ser primero en una.
RRF no premia la excelencia en una técnica, premia el consenso entre ambas.

CIERRE EJECUTIVO: la pregunta correcta a un proveedor no es "¿usan IA para
buscar?", sino "¿cómo balancean búsqueda exacta y semántica, y cómo
garantizan los permisos?".

DEJA FUERA: las fórmulas de distancia y la tabla de dimensionalidad.
```

---

## 5. 🧩 Plantilla para tomos futuros

Audiencia: 🔧 🧭

Cuando se genere un tomo nuevo, su bloque se arma con esta estructura:

```text
TEMA: [una frase — qué cubre y en qué punto del recorrido va]

ARCO NARRATIVO:
1. [concepto de entrada + su analogía]
2. [desarrollo]
3. [el mecanismo central]

CLÍMAX DEL EPISODIO:
[El momento contraintuitivo del tomo. Idealmente algo que SORPRENDA: un
resultado que contradice la intuición, un error costoso, un caso donde la
técnica falla. Si el tomo tiene un dato verificado o un fallo real
documentado, ese es el clímax.]

CIERRE EJECUTIVO: [qué decisión habilita o qué riesgo evita, en una frase]

DEJA FUERA: [fórmulas, código, tablas numéricas, lo que no funciona en audio]
```

> [!tip] Cómo elegir el clímax
> La pregunta guía es: **"¿qué cuenta el oyente en el almuerzo del día siguiente?"** No es la definición del concepto — es el caso donde algo falló de forma sorprendente. La guía tiene muchos, y casi todos salieron de ejecutar código y mirar el resultado real: el pan de masa madre, Kyoto, el vector idéntico tras truncar, la respuesta sin RAG que sonaba mejor. **Un episodio con un buen clímax se recuerda; uno con cobertura completa se olvida.**

---

## 6. 📝 Notas de uso

Audiencia: 🔧 🧭

> [!note] Un notebook por tomo
> Es preferible a subir la guía completa: NotebookLM reparte la atención entre las fuentes, y con todos los tomos juntos el audio queda superficial en todo. Si un tomo depende del anterior (el 04 del 03, por ejemplo), sube el anterior **como fuente secundaria** para que tenga contexto, pero deja claro en el prompt cuál es el tomo protagonista.

> [!warning] Verifica el audio antes de compartirlo
> NotebookLM puede introducir imprecisiones o inventar transiciones que suenan bien pero no están en el tomo. **Escucha el episodio con el tomo delante** antes de darlo por bueno, sobre todo en las cifras y en los nombres de técnicas. Aplica el mismo criterio que la guía se aplica a sí misma: nada se da por correcto sin contrastar.

> [!tip] Si el audio traduce igual los términos técnicos
> Es el fallo más frecuente. Dos remedios: (1) repetir la lista de términos al final del prompt, no solo al principio; (2) añadir la frase *"si dudas entre el término en inglés y su traducción, usa SIEMPRE el inglés"*. Si aun así insiste, regenera — la salida varía entre ejecuciones.

> [!info] Estado de cobertura
> | Tomo | Bloque de prompt |
> |---|:---:|
> | 01 · Introducción a RAG | ✅ |
> | 02 · Fundamentos: LLMs y pipeline | ✅ |
> | 03 · Keyword search: TF-IDF y BM25 | ✅ |
> | 04 · Semantic search y embeddings | ✅ |
> | 05 en adelante | ⬜ pendiente del tomo |

---

## 🔗 Conexiones

- [[Instrucciones|📖 Instrucciones]] — el contrato de trabajo de la guía; la regla de idioma de la sección 4 es la que estos prompts protegen.
- [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ MOC de la Guía Maestra de RAG]] — índice y tracker de tomos.

> [!note] Este documento evoluciona
> Cada tomo nuevo agrega su bloque en la sección 4 y su fila en la tabla de cobertura. Si NotebookLM cambia su interfaz o sus límites, se ajusta la sección 2.
