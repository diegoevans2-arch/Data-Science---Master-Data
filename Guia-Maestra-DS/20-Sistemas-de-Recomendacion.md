---
title: "Tomo 20 — Sistemas de Recomendación"
tags: [data-science, machine-learning, recomendadores, ranking, personalizacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 20
version: 6.0
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

## 1. Las familias de recomendadores

Audiencia: 🔧 🧭

| Familia | Mecanismo | Fortalezas | Limitaciones | Cuándo conviene |
|---|---|---|---|---|
| No personalizado (popularidad) | Top ventas/vistas, con recencia | Trivial, robusto, sin datos de usuario | Igual para todos; refuerza lo ya popular | **Baseline obligatorio** y fallback para usuarios nuevos |
| Content-based | Perfil del usuario a partir de los **atributos/contenido** de lo que consumió (TF-IDF o embeddings de ítems — [[19-NLP-y-LLMs]]); recomienda lo similar | No necesita otros usuarios; explica ("porque viste X"); sirve ítems nuevos | Encierra en lo ya conocido (poca serendipia) | Catálogos con buen contenido descriptivo; ítems nuevos frecuentes |
| Collaborative filtering (memoria) | "Usuarios parecidos a ti" (user-based) o "ítems que se consumen juntos" (item-based KNN) (Sarwar et al., 2001) | Capta gustos sin describir ítems; item-based es estable y explicable | Matriz rala; escala; nada que hacer con usuarios/ítems sin historial | El clásico "quienes compraron esto también…" |
| Matrix Factorization | Descompone la matriz usuario×ítem en factores latentes: `r̂(u,i) = pᵤ · qᵢ` — SVD del mundo real, aprendida por optimización (Koren et al., 2009); **ALS** para feedback implícito (Hu et al., 2008) | El estándar de precisión clásica; compacto; captura gustos latentes | Factores poco interpretables; reentrenar para usuarios nuevos | El caballo de batalla cuando hay historial abundante |
| Híbridos | Combinan content + collaborative (ponderación, switching o features conjuntas) | Cubre los puntos ciegos de cada familia | Más complejidad | La práctica real: casi todo sistema serio es híbrido |
| Deep / two-tower | Torre de usuario y torre de ítem producen embeddings; el producto punto rankea; candidatos + ranking en dos etapas (Covington et al., 2016) | Escala a millones; incorpora contexto (hora, dispositivo) | Infraestructura pesada; hambre de datos | Plataformas grandes con señales ricas ([[12-Deep-Learning]]) |
| Reglas de asociación | "Con este ítem, va este otro" a nivel transacción | Simple, accionable, sin perfil de usuario | No personaliza por persona | Cross-sell de carrito ([[09-Reglas-de-Asociacion]]) |

## 2. Feedback explícito vs implícito

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El feedback **explícito** es la encuesta de satisfacción: clara pero escasa (¿cuándo fue la última vez que calificaste con estrellas?). El **implícito** es el lenguaje corporal: clics, tiempo de lectura, compras, abandonos — abundante pero ambiguo (¿lo compró porque le encantó o porque era lo único que apareció?). Los sistemas reales viven del lenguaje corporal.

**🔧 Definición técnica:** explícito = ratings directos (escasos, sesgados hacia opiniones extremas). Implícito = interacciones (clic, dwell time, compra): abundante pero **sin negativos observados** — no ver un ítem no significa rechazarlo, quizás nunca se mostró. ALS para implicit feedback (Hu et al., 2008) modela preferencia + **confianza** creciente con la intensidad de interacción. Sesgos estructurales: **exposición** (solo hay feedback de lo mostrado), **posición** (lo primero recibe clics por estar primero), **popularidad** (lo famoso acumula evidencia).

**👔 En una frase para el negocio:** el sistema aprende de lo que la gente HACE, no de lo que dice — pero lo que hace está condicionado por lo que el propio sistema le mostró: por eso la evaluación offline engaña (sección 4).

## 3. Cold start: el problema del recién llegado

Audiencia: 🔧 🧭

**🔧 Definición técnica:** sin historial no hay collaborative filtering. **Usuario nuevo:** arrancar con popularidad segmentada (por origen, campaña, demografía disponible), preguntas de onboarding, o **bandits** que aprenden sus gustos en las primeras interacciones ([[21-Supervivencia-y-Bandits]]). **Ítem nuevo:** content-based con sus atributos/embeddings + boost de exploración para juntar señal. La transición gradual de content → collaborative a medida que llega historial es el patrón híbrido estándar.

**👔 En una frase para el negocio:** las primeras interacciones de un cliente nuevo son una inversión en información — un sistema que solo explota lo conocido nunca aprende a quién tiene enfrente.

## 4. Métricas de ranking y evaluación

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> No basta con que el restaurante correcto esté "en la lista": importa si está en la **primera página o en la vigésima**. Las métricas de ranking premian poner lo relevante arriba — que es donde vive la atención (y el pulgar).

| Métrica | Qué mide | Nota de lectura |
|---|---|---|
| Precision@K | De los K mostrados, ¿cuántos fueron relevantes? | La métrica de la vitrina: K = tamaño del carrusel |
| Recall@K | De todo lo relevante que existía, ¿cuánto quedó en el top K? | Clave también para el retriever de un RAG ([[19-NLP-y-LLMs]]) |
| MAP | Promedio de precisión considerando el orden de los aciertos | Resume calidad de orden en listas largas |
| **NDCG@K** | Ganancia descontada por posición: `DCG = Σ relᵢ / log₂(i+1)`, normalizada por el ranking ideal | El estándar: acertar en la posición 1 vale más que en la 10 |
| Hit rate@K | ¿Al menos un acierto en el top K? | Fácil de comunicar |
| Coverage / diversidad / novedad | % del catálogo que llega a recomendarse; variedad dentro de la lista; qué tan no-obvio es lo sugerido | Las métricas anti-burbuja: sin ellas, el sistema converge a los 20 best-sellers |

**🔧 Validación:** siempre **temporal** — entrenar con el pasado, evaluar con interacciones futuras (leave-last-out por usuario); un split aleatorio deja que el sistema "recomiende" lo que la persona ya compró ([[10-Validacion-y-Leakage]]).

> [!warning] ⚠️ La evaluación offline engaña — el feedback loop
> Los datos históricos registran clics **sobre lo que el sistema anterior decidió mostrar**: evaluar offline favorece a quien imita al sistema viejo (sesgo de exposición) y castiga la novedad. El offline sirve para descartar candidatos malos; el veredicto lo da el **A/B test** con métricas de negocio (conversión, ingresos por sesión, retención — [[02-Fundamentos-Matematicos]]) y guardrails de diversidad. Y ojo con optimizar solo el clic: se aprende a recomendar carnada.

> [!example] 📊 Caso de negocio — E-commerce: del carrusel único al híbrido con guardrails
> **Problema:** el home muestra el mismo carrusel de top-ventas a todos. CTR estancado, y el 70% del catálogo jamás se exhibe.
>
> **Técnica aplicada:** híbrido por etapas: candidatos desde ALS implícito (compras + carritos) + ítems similares por embeddings de contenido para lo nuevo; ranking final con boosting usando contexto (hora, categoría de la sesión); popularidad segmentada como fallback de cold start; evaluación offline con NDCG@10 y coverage, decisión final por A/B con ingresos por sesión y un guardrail de diversidad mínima.
>
> **Resultado:** mejora sostenida de conversión en el A/B, y — el efecto menos esperado — la cola larga del catálogo empieza a rotar: coverage se triplica, descomprimendo inventario que antes solo se movía con descuentos. La lección: **el recomendador no solo sube el clic; redistribuye la demanda** — y eso también se gestiona.

---

## 📖 Referencias de este tomo

- (Sarwar et al., 2001) — item-based collaborative filtering.
- (Koren et al., 2009) — matrix factorization (la síntesis del Netflix Prize).
- (Hu et al., 2008) — ALS para feedback implícito.
- (Covington et al., 2016) — arquitectura de dos etapas a escala (YouTube).
- (Ricci et al., 2022) — *Recommender Systems Handbook*, la referencia enciclopédica.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[19-NLP-y-LLMs|19 · NLP y LLMs]] · Siguiente: [[21-Supervivencia-y-Bandits|21 · Supervivencia y Bandits ➡]]

> **Próximo tomo:** [[21-Supervivencia-y-Bandits]] — los dos complementos finales: modelar el *cuándo* (análisis de supervivencia con censura) y aprender *mientras* se decide (multi-armed bandits).
