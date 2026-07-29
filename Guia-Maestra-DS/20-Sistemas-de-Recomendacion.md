---
title: "Tomo 20 — Sistemas de Recomendación"
tags: [data-science, machine-learning, recomendadores, ranking, personalizacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 20
version: 6.2
updated: 2026-07-27
---

# 🎁 Tomo 20 — Sistemas de Recomendación

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[19-NLP-y-LLMs|19 · NLP y LLMs]] · Siguiente: [[21-Supervivencia-y-Bandits|21 · Supervivencia y Bandits ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Recomendar es decidir **qué mostrarle a cada quién** cuando el catálogo supera lo que cualquier persona puede revisar. Es un problema distinto a clasificar: el output es un **ranking**, el feedback es sesgado (solo ves clics sobre lo que TÚ decidiste mostrar), y el sistema modifica el mundo que luego mide (feedback loop). Este tomo completa las piezas ya sembradas: reglas de asociación ([[09-Reglas-de-Asociacion]]), SVD ([[02-Fundamentos-Matematicos]]) y embeddings ([[12-Deep-Learning]], [[19-NLP-y-LLMs]]).

> [!abstract] 👔 Impacto ejecutivo
> En catálogos grandes, el recomendador ES el vendedor: define qué ve el cliente y, por lo tanto, qué puede comprar.
>
> - **Decisiones que habilita:** personalización a escala (home, emails, next best offer), cross-selling sistemático, priorización de vitrina digital.
> - **Costo de hacerlo mal:** mostrar siempre lo mismo (burbuja de popularidad), medir "éxito" en un loop que se retroalimenta, y recomendar la parka a quien acaba de comprar una.
> - **Pregunta ejecutiva que responde:** *de todo mi catálogo, ¿qué N cosas le muestro a ESTA persona, en este momento, y cómo sé que funciona?*

> [!tip] 💡 Analogía general
> Tres vendedores de librería: el **novato** recomienda los best-sellers a todos (popularidad — y no es mal punto de partida); el **bibliotecario** te recomienda por el contenido de lo que ya leíste ("te gustó esta novela negra nórdica → aquí hay otra": content-based); y el **vendedor veterano** ni siquiera necesita leer los libros: sabe que "los clientes como tú" terminaron amando este otro (collaborative filtering). Los sistemas reales contratan a los tres y les dan el turno según cuánto saben del cliente.

---

## 1. Por qué recomendar no es clasificar

Audiencia: 🔧 🧭

Es tentador tratar la recomendación como una clasificación ("¿comprará el ítem sí/no?"), pero tres diferencias estructurales lo cambian todo — y cada una explica una decisión de diseño del resto del tomo.

| Diferencia | Clasificación ([[07-Modelos-Supervisados]]) | Recomendación |
|---|---|---|
| **El output** | Una etiqueta o probabilidad | Un **ranking**: importa el orden, no solo el acierto |
| **El feedback** | Etiquetas objetivas | **Sesgado**: solo ves reacción a lo que mostraste; lo no mostrado no es "negativo" |
| **El mundo** | Estático: medir no lo cambia | **Reflexivo**: lo que recomiendas hoy es el dato de entrenamiento de mañana (feedback loop) |

> [!tip] 💡 Analogía
> Un clasificador es un examen con pauta fija: la respuesta correcta existe y no cambia. Un recomendador es un **vitrinista**: lo que pone en la vitrina determina qué se vende, lo que se vende determina qué datos recibe, y esos datos determinan qué pondrá mañana. Nunca observa el mundo neutral — solo el mundo que él mismo fue moldeando.

**🧭 Las dos preguntas que definen el problema** — se resuelven con arquitecturas distintas (sección 4):

1. **Candidate generation:** de un catálogo de millones, ¿cuáles **cientos** son siquiera plausibles para esta persona? (rapidez sobre precisión)
2. **Ranking:** de esos cientos, ¿en qué **orden** exacto los muestro? (precisión sobre rapidez)

**👔 En una frase para el negocio:** recomendar no es adivinar si algo gusta, es **ordenar** un catálogo entero para una persona — y hacerlo sabiendo que la propia recomendación altera lo que luego se podrá medir.

---

## 2. Las familias de recomendadores

Audiencia: 🔧 🧭

| Familia | Mecanismo | Fortalezas | Limitaciones | Cuándo conviene |
|---|---|---|---|---|
| No personalizado (popularidad) | Top ventas/vistas, con recencia | Trivial, robusto, sin datos de usuario | Igual para todos; refuerza lo ya popular | **Baseline obligatorio** y fallback para usuarios nuevos |
| Content-based | Perfil del usuario a partir de los **atributos/contenido** de lo que consumió (TF-IDF o embeddings de ítems — [[19-NLP-y-LLMs]]); recomienda lo similar | No necesita otros usuarios; explica ("porque viste X"); sirve ítems nuevos | Encierra en lo ya conocido (poca serendipia) | Catálogos con buen contenido descriptivo; ítems nuevos frecuentes |
| Collaborative filtering (memoria) | "Usuarios parecidos a ti" (user-based) o "ítems que se consumen juntos" (item-based KNN) (Sarwar et al., 2001) | Capta gustos sin describir ítems; item-based es estable y explicable | Matriz rala; escala; nada que hacer con usuarios/ítems sin historial | El clásico "quienes compraron esto también…" |
| Matrix Factorization | Descompone la matriz usuario×ítem en factores latentes (sección 3) (Koren et al., 2009) | El estándar de precisión clásica; compacto; captura gustos latentes | Factores poco interpretables; reentrenar para usuarios nuevos | El caballo de batalla cuando hay historial abundante |
| Híbridos | Combinan content + collaborative (ponderación, switching o features conjuntas) | Cubre los puntos ciegos de cada familia | Más complejidad | La práctica real: casi todo sistema serio es híbrido |
| Deep / two-tower | Torre de usuario y torre de ítem producen embeddings; el producto punto rankea; candidatos + ranking en dos etapas (Covington et al., 2016) | Escala a millones; incorpora contexto (hora, dispositivo) | Infraestructura pesada; hambre de datos | Plataformas grandes con señales ricas ([[12-Deep-Learning]]) |
| Secuenciales / sesión | Modelan el **orden** de las interacciones (RNN/Transformer sobre la secuencia de consumo) | Capturan intención del momento ("está armando un viaje") | Datos de secuencia, más cómputo | Sesiones con intención clara: viajes, media, e-commerce |
| Generative recommenders (semantic IDs) | Tokeniza cada ítem como una secuencia de "semantic IDs" y entrena un único Transformer autoregresivo que **genera** la recomendación, fusionando candidate generation + ranking en un solo modelo (Zhai et al., 2024) | Escala con cómputo/datos como un LLM (el DLRM clásico se estanca); colapsa la arquitectura de dos etapas | Infraestructura de entrenamiento/inferencia tipo LLM; evidencia de producción aún concentrada en pocas plataformas | Escala extrema (miles de millones de usuarios) — **desarrollo reciente (2024–2025), verificar vigencia**; detalle en sección 4 |
| Reglas de asociación | "Con este ítem, va este otro" a nivel transacción | Simple, accionable, sin perfil de usuario | No personaliza por persona | Cross-sell de carrito ([[09-Reglas-de-Asociacion]]) |

> [!tip] 💡 Content-based vs. collaborative, en una frase
> **Content-based** mira el *ítem* ("esto se parece a lo que te gustó"). **Collaborative** mira a la *gente* ("personas como tú amaron esto"). El primero nunca te sorprende con algo distinto pero seguro sirve lo nuevo; el segundo te trae joyas inesperadas pero se queda mudo ante un ítem sin historial. Por eso casi nadie elige: se combinan.

---

## 3. Matrix factorization: los gustos como factores latentes

Audiencia: 🔧 🧭

**🔧 Definición técnica:** la matriz usuario×ítem (millones × millones, casi toda vacía) se aproxima como el producto de dos matrices delgadas: cada usuario `u` y cada ítem `i` quedan representados por un **vector de factores latentes** de baja dimensión (k ≈ 50–200), y la predicción es su producto punto:

```
   r̂(u,i) = pᵤ · qᵢ          (+ sesgos de usuario e ítem + media global)

   pᵤ = vector latente del usuario u   (¿cuánto le gusta cada "factor"?)
   qᵢ = vector latente del ítem i       (¿cuánto expresa el ítem cada "factor"?)
```

> [!tip] 💡 Analogía
> Es como descubrir, sin que nadie los nombre, los "ejes de gusto" de tu catálogo. En cine podrían emerger algo así como "acción↔drama", "comercial↔autor", "reciente↔clásico". Cada película tiene una posición en esos ejes y cada persona una preferencia. El modelo **inventa los ejes solo** para que el producto de ambos reconstruya los ratings observados — nadie le dijo qué es "acción", emergió de los datos. (Los ejes rara vez son tan legibles: son latentes, no interpretables.)

**🔧 Cómo se aprende:** no es la SVD algebraica exacta ([[02-Fundamentos-Matematicos]]) — esa exige la matriz completa. Se optimiza minimizando el error sobre las celdas **observadas** con regularización ([[11-Mejora-de-Modelos]]), por dos vías:

| Método | Idea | Cuándo |
|---|---|---|
| **SGD** | Descenso de gradiente celda a celda | Feedback explícito (ratings) |
| **ALS** (Hu et al., 2008) | Fijar una matriz y resolver la otra por mínimos cuadrados, alternando | Feedback implícito y datos masivos (paraleliza bien) |

Para **feedback implícito**, ALS modela dos cosas separadas: la **preferencia** (¿interactuó, sí/no?) y la **confianza** en esa preferencia (crece con la intensidad: 10 reproducciones pesan más que 1). Es la diferencia clave respecto a tratar los clics como ratings.

**👔 En una frase para el negocio:** la factorización comprime "todo lo que sabemos de gustos" en unos pocos números por persona y por producto — el motor que hace viable personalizar un catálogo gigante.

---

## 4. La arquitectura real: candidatos → ranking → re-ranking

Audiencia: 🔧 🧭 👔

Ningún sistema a escala rankea el catálogo entero por usuario en cada request: sería inviable en latencia. La arquitectura estándar tiene **tres etapas**, cada una con un objetivo distinto.

```
   CATÁLOGO (millones)
        │
        ▼
 ┌─────────────────────┐   Rapidísimo, recall alto, precisión baja.
 │ 1. CANDIDATE GEN    │   Varias fuentes en paralelo: ALS, embeddings
 │    → cientos        │   (two-tower + ANN), popularidad, reglas.
 └──────────┬──────────┘   "Traer todo lo plausible."
            ▼
 ┌─────────────────────┐   Modelo pesado (gradient boosting / red) con
 │ 2. RANKING          │   MUCHAS features: usuario, ítem, contexto
 │    → decenas        │   (hora, dispositivo, sesión). "Ordenar fino."
 └──────────┬──────────┘
            ▼
 ┌─────────────────────┐   Reglas de negocio: diversidad, no repetir lo
 │ 3. RE-RANKING       │   ya comprado, margen, stock, promociones,
 │    → los N finales  │   fairness. "Ajustar a la realidad del negocio."
 └──────────┬──────────┘
            ▼
     Lo que ve el usuario
```

> [!tip] 💡 Analogía
> Es el proceso de contratación de una empresa grande: primero un filtro de CV automático que descarta miles y deja cientos (candidate generation: rápido y tolerante), luego entrevistas a fondo que ordenan a los finalistas (ranking: caro y preciso), y al final el comité aplica criterios que no estaban en la entrevista — diversidad del equipo, presupuesto, encaje cultural (re-ranking). Cada etapa optimiza algo distinto, y confundirlas es caro.

> [!warning] ⚠️ El re-ranking es donde el negocio entra al modelo
> Muchas decisiones **no** son del modelo predictivo: no recomendar lo recién comprado, no agotar la diversidad, respetar stock y márgenes, cumplir cuotas de contenido. Meterlas como features del ranker las diluye; el patrón robusto es una **capa de re-ranking explícita** con esas reglas. Separa "qué le gusta a la persona" (aprendido) de "qué nos conviene y podemos mostrar" (reglas) — y hace el sistema auditable.

### Generative recommenders: cuando las tres etapas colapsan en una

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Las tres etapas de esta sección son como armar un mueble con instrucciones separadas para cortar, ensamblar y pintar — cada paso lo hace una estación distinta. Un **generative recommender** es el mueble impreso de una sola pieza: la misma máquina que "entiende" la forma final decide directamente qué producir, sin pasar la pieza de estación en estación.

**🔧 Definición técnica:** cada ítem del catálogo se codifica como una secuencia corta de tokens discretos ("semantic IDs", obtenidos cuantizando sus embeddings de contenido). Con esa tokenización, recomendar deja de ser "generar candidatos y luego ordenarlos" y pasa a ser un problema de **modelado generativo autoregresivo**: un único Transformer, entrenado sobre el historial de interacciones del usuario como si fuera una secuencia de lenguaje, genera directamente los próximos ítems a mostrar — fusionando candidate generation y ranking en un solo modelo. Meta reporta con su arquitectura HSTU (Hierarchical Sequential Transduction Units, hasta 1.5 billones de parámetros en inglés "trillion") **leyes de escala tipo LLM**: a diferencia del DLRM clásico de esta sección, que se estanca tras una época de entrenamiento, estos modelos siguen mejorando con más cómputo y datos, y ya está desplegado en múltiples superficies de producción con una mejora reportada de +12.4% en métricas online (Zhai et al., 2024). Kuaishou reporta un patrón análogo con OneRec, un modelo generativo end-to-end también en producción, con +1.6% en watch-time (Deng et al., 2025); reportes similares circulan en Meituan, Alibaba y ByteDance.

**🧭 Cuándo usarlo:** por ahora es una apuesta de plataformas con escala y presupuesto de investigación excepcionales. Para el resto de los catálogos, la arquitectura de tres etapas de esta sección sigue siendo el punto de partida correcto: exige menos infraestructura, es más auditable etapa por etapa, y su costo/beneficio está mejor probado. No es un reemplazo universal del pipeline clásico, sino una familia emergente para el extremo superior de escala — **desarrollo muy reciente (2024–2025): tratar como línea de investigación a monitorear, no como estándar de la industria todavía**.

**👔 En una frase para el negocio:** algunas de las plataformas más grandes del mundo ya no calculan "qué mostrar" en pasos separados — entrenan un solo modelo que **genera** la recomendación, con el mismo tipo de leyes de escala que impulsaron a los LLMs; vale la pena monitorearlo, pero replicarlo hoy fuera de una escala extrema es prematuro.

---

## 5. Feedback explícito vs implícito, y sus sesgos

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El feedback **explícito** es la encuesta de satisfacción: clara pero escasa (¿cuándo fue la última vez que calificaste con estrellas?). El **implícito** es el lenguaje corporal: clics, tiempo de lectura, compras, abandonos — abundante pero ambiguo (¿lo compró porque le encantó o porque era lo único que apareció?). Los sistemas reales viven del lenguaje corporal.

**🔧 Definición técnica:** explícito = ratings directos (escasos, sesgados hacia opiniones extremas). Implícito = interacciones (clic, dwell time, compra): abundante pero **sin negativos observados** — no ver un ítem no significa rechazarlo, quizás nunca se mostró.

**Los tres sesgos estructurales** — nombrarlos es media defensa:

| Sesgo | Qué es | Consecuencia |
|---|---|---|
| **Exposición** | Solo hay feedback de lo que se mostró | El modelo no puede aprender de lo que nunca sirvió |
| **Posición** | Lo primero recibe clics **por estar primero**, no por ser mejor | Confundir posición con calidad infla lo ya bien rankeado |
| **Popularidad** | Lo famoso acumula más evidencia | El sistema converge a los best-sellers (la burbuja) |

> [!tip] 🧭 La corrección: Inverse Propensity Scoring (IPS)
> La idea para desesgar: ponderar cada interacción observada por el **inverso de la probabilidad de que ese ítem se mostrara** en esa posición. Un clic en algo que casi nunca se muestra "vale" más que un clic en el primer resultado que ve todo el mundo. Es la misma lógica de reponderación que aparece en causalidad ([[18-Causalidad-y-Uplift]]): estás intentando estimar qué habría pasado en un mundo sin el sesgo de exposición. No es gratis (hay que estimar esas propensiones), pero es el puente entre "lo que el log registró" y "lo que a la gente realmente le gusta".

**👔 En una frase para el negocio:** el sistema aprende de lo que la gente HACE, no de lo que dice — pero lo que hace está condicionado por lo que el propio sistema le mostró: por eso la evaluación offline engaña (sección 7).

---

## 6. Cold start: el problema del recién llegado

Audiencia: 🔧 🧭

**🔧 Definición técnica:** sin historial no hay collaborative filtering. Hay tres variantes, cada una con su salida:

| Variante | Estrategia |
|---|---|
| **Usuario nuevo** | Popularidad **segmentada** (por origen, campaña, demografía disponible), preguntas de onboarding, o **bandits** que aprenden sus gustos en las primeras interacciones ([[21-Supervivencia-y-Bandits]]) |
| **Ítem nuevo** | Content-based con sus atributos/embeddings + un boost de **exploración** para juntar señal antes de juzgarlo |
| **Sistema nuevo** (arranque) | Empezar por content-based y reglas; el collaborative llega cuando se acumula historial |

La transición gradual **content → collaborative** a medida que llega historial es el patrón híbrido estándar.

> [!tip] 💡 Analogía
> Un mesero nuevo no tiene idea de tus gustos, así que hace lo sensato: te ofrece lo más pedido (popularidad) y te hace un par de preguntas (onboarding). Con cada visita aprende, y en algún momento deja de preguntar porque ya te conoce. Un sistema que **nunca** explora se queda de mesero-novato para siempre: sirve best-sellers y jamás descubre a quién tiene enfrente.

**👔 En una frase para el negocio:** las primeras interacciones de un cliente nuevo son una inversión en información — un sistema que solo explota lo conocido nunca aprende a quién tiene enfrente.

---

## 7. Métricas de ranking y evaluación

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> No basta con que el restaurante correcto esté "en la lista": importa si está en la **primera página o en la vigésima**. Las métricas de ranking premian poner lo relevante arriba — que es donde vive la atención (y el pulgar).

| Métrica | Qué mide | Nota de lectura |
|---|---|---|
| Precision@K | De los K mostrados, ¿cuántos fueron relevantes? | La métrica de la vitrina: K = tamaño del carrusel |
| Recall@K | De todo lo relevante que existía, ¿cuánto quedó en el top K? | Clave también para el retriever de un RAG ([[19-NLP-y-LLMs]]) |
| MAP | Promedio de precisión considerando el orden de los aciertos | Resume calidad de orden en listas largas |
| **NDCG@K** | Ganancia descontada por posición: `DCG = Σ relᵢ / log₂(i+1)`, normalizada por el ranking ideal | El estándar: acertar en la posición 1 vale más que en la 10 |
| MRR | Posición del primer acierto (1/rank) | Cuando importa "el primer resultado útil" |
| Hit rate@K | ¿Al menos un acierto en el top K? | Fácil de comunicar |
| Coverage / diversidad / novedad | % del catálogo que llega a recomendarse; variedad dentro de la lista; qué tan no-obvio es lo sugerido | Las métricas anti-burbuja: sin ellas, el sistema converge a los 20 best-sellers |

> [!note] Learning to rank: optimizar el orden, no el acierto individual
> Los rankers modernos no predicen "¿le gustará este ítem?" en aislamiento, sino que optimizan **el orden de la lista completa**. Tres enfoques: **pointwise** (predice un score por ítem y ordena — simple, ignora que es una lista), **pairwise** (aprende qué ítem va antes que cuál, par a par — BPR es el clásico para feedback implícito), **listwise** (optimiza directamente una métrica de lista como NDCG — el más alineado con el objetivo, el más complejo). La diferencia importa: un modelo con buen AUC individual puede producir un ranking mediocre.

**🔧 Validación:** siempre **temporal** — entrenar con el pasado, evaluar con interacciones futuras (leave-last-out por usuario); un split aleatorio deja que el sistema "recomiende" lo que la persona ya compró ([[10-Validacion-y-Leakage]]).

> [!danger] 🚨 La evaluación offline engaña — el feedback loop
> Los datos históricos registran clics **sobre lo que el sistema anterior decidió mostrar**: evaluar offline favorece a quien imita al sistema viejo (sesgo de exposición, sección 5) y castiga la novedad. El offline sirve para **descartar candidatos malos**; el veredicto lo da el **A/B test** con métricas de negocio (conversión, ingresos por sesión, retención — [[02-Fundamentos-Matematicos]]) y guardrails de diversidad. Y ojo con optimizar solo el clic: **se aprende a recomendar carnada** (clickbait).

---

## 8. Más allá del clic: diversidad, serendipia y objetivos de negocio

Audiencia: 🧭 👔

El error estratégico más común es optimizar una sola métrica (el CTR) y descubrir tarde que el sistema aprendió a maximizarla de formas indeseables.

| Objetivo | Por qué importa | Riesgo si se ignora |
|---|---|---|
| **Diversidad** | Una lista de 10 ítems casi idénticos aburre | Carruseles monótonos; el usuario ya vio "eso" |
| **Serendipia** | El valor está en lo relevante **inesperado** | Solo recomendar lo obvio no descubre nada nuevo |
| **Coverage / cola larga** | Rotar el catálogo, no solo el top 20 | Inventario muerto; dependencia de descuentos |
| **Objetivos de negocio** | Margen, stock, contratos, retención (no solo clic) | Optimizar clic puede vender lo barato y quemar lo rentable |
| **Fairness / no-daño** | Sesgos de exposición entre proveedores o grupos | Riesgo reputacional y regulatorio ([[13-MLOps-XAI-Etica]]) |

> [!warning] ⚠️ La trampa del filter bubble
> Un recomendador que solo explota lo que ya funcionó **encierra** al usuario en lo que ya conoce y **empobrece** el catálogo visible. El feedback loop lo agrava: menos diversidad → menos datos sobre la cola → aún menos diversidad. La defensa no es un modelo mejor, es un **objetivo mejor**: incluir diversidad/exploración como restricción explícita (en el re-ranking, sección 4) y medirla como métrica de primera clase, no como adorno.

> [!example] 📊 Caso de negocio — E-commerce: del carrusel único al híbrido con guardrails
> **Problema:** el home muestra el mismo carrusel de top-ventas a todos. CTR estancado, y el 70% del catálogo jamás se exhibe.
>
> **Técnica aplicada:** arquitectura de dos etapas (sección 4): candidatos desde ALS implícito (compras + carritos) + ítems similares por embeddings de contenido para lo nuevo; ranking final con boosting usando contexto (hora, categoría de la sesión); una capa de re-ranking con diversidad mínima y "no mostrar lo ya comprado"; popularidad segmentada como fallback de cold start. Evaluación offline con NDCG@10 y coverage, decisión final por A/B con ingresos por sesión.
>
> **Resultado:** mejora sostenida de conversión en el A/B, y — el efecto menos esperado — la cola larga del catálogo empieza a rotar: coverage se triplica, descomprimiendo inventario que antes solo se movía con descuentos. La lección: **el recomendador no solo sube el clic; redistribuye la demanda** — y eso también se gestiona.

---

## 9. Guía de decisión rápida

Audiencia: 🧭

| Tu situación | Punto de partida |
|---|---|
| Recién arrancas, sin datos | Popularidad segmentada (baseline y fallback obligatorio) |
| Catálogo con buen contenido, ítems nuevos frecuentes | Content-based (embeddings de ítem) |
| Historial abundante de interacciones | Matrix factorization (ALS si es implícito) |
| Quieres lo mejor de ambos | **Híbrido** (es lo que hace casi todo sistema serio) |
| Escala de millones + señales de contexto | Two-tower + arquitectura de dos etapas |
| La intención del momento importa (sesión) | Recomendador secuencial |
| Cross-sell de carrito | Reglas de asociación ([[09-Reglas-de-Asociacion]]) |
| Usuario/ítem nuevo | Cold start: popularidad + exploración/bandits ([[21-Supervivencia-y-Bandits]]) |
| **Siempre** | Baseline de popularidad, validación temporal, y **A/B test** como juez final |

> [!tip] 🧭 El orden correcto de trabajo
> Popularidad → content o collaborative simple → factorización → híbrido → deep, **subiendo de escalón solo cuando la medición (offline para descartar, A/B para decidir) lo justifique**. Y desde el día uno, tratar la **diversidad y el feedback loop** como parte del diseño, no como un problema a resolver después: cuando el sistema ya encerró a los usuarios en la burbuja, revertirlo es caro.

---

## 📖 Referencias de este tomo

- (Sarwar et al., 2001) — item-based collaborative filtering.
- (Koren et al., 2009) — matrix factorization (la síntesis del Netflix Prize).
- (Hu et al., 2008) — ALS para feedback implícito.
- (Rendle et al., 2009) — BPR: ranking pairwise para feedback implícito.
- (Covington et al., 2016) — arquitectura de dos etapas a escala (YouTube).
- (Ricci et al., 2022) — *Recommender Systems Handbook*, la referencia enciclopédica.
- (Zhai et al., 2024) — HSTU: recomendadores generativos con leyes de escala tipo LLM (Meta). Desarrollo reciente, verificar vigencia.
- (Deng et al., 2025) — OneRec: recomendador generativo end-to-end en producción (Kuaishou). Desarrollo reciente, verificar vigencia.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[19-NLP-y-LLMs|19 · NLP y LLMs]] · Siguiente: [[21-Supervivencia-y-Bandits|21 · Supervivencia y Bandits ➡]]

> **Próximo tomo:** [[21-Supervivencia-y-Bandits]] — los dos complementos finales: modelar el *cuándo* (análisis de supervivencia con censura) y aprender *mientras* se decide (multi-armed bandits).
