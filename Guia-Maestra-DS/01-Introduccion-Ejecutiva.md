---
title: "Tomo 01 — Introducción Ejecutiva"
tags: [data-science, machine-learning, introduccion, negocio, valor]
audiencias: [tecnico, puente, ejecutivo]
tomo: 01
version: 6.0
---

# 👔 Tomo 01 — Introducción Ejecutiva

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Siguiente: [[02-Fundamentos-Matematicos|02 · Fundamentos Matemáticos ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Este tomo es la puerta de entrada de toda la guía. Explica **qué es realmente el Data Science**, qué familias de problemas resuelve, cómo se ve el ciclo de vida de un proyecto y — lo más importante — **dónde se crea y dónde se destruye valor de negocio**. Si solo vas a leer un tomo completo, que sea este: los demás profundizan cada etapa que aquí se presenta como mapa.

> [!abstract] 👔 Impacto ejecutivo
> El Data Science no es un proyecto de tecnología: es una capacidad de decisión. Las organizaciones que lo dominan deciden con evidencia donde las demás deciden con intuición.
>
> - **Decisiones que habilita:** priorizar clientes a retener, aprobar créditos con riesgo cuantificado, anticipar demanda y quiebres de stock, detectar fraude antes de la pérdida, personalizar ofertas a escala.
> - **Costo de hacerlo mal:** modelos que "funcionan" en la presentación y fallan en producción, decisiones automatizadas que discriminan, inversiones en IA sin retorno medible, y pérdida de confianza interna en los datos.
> - **Pregunta ejecutiva que responde:** *¿qué decisiones de mi negocio podrían tomarse mejor, más rápido o a mayor escala si usara sistemáticamente los datos que ya tengo?*

---

## 1. ¿Qué es Data Science?

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Piensa en un médico clínico excepcional. Combina tres cosas: conocimiento científico (anatomía, farmacología), herramientas de diagnóstico (exámenes, imágenes) y experiencia con pacientes reales. Ninguna de las tres basta por sí sola: un enciclopedista sin pacientes no cura, una máquina de rayos X sin criterio no diagnostica. El Data Science es lo mismo aplicado a organizaciones: **estadística** (la ciencia), **computación** (las herramientas) y **conocimiento del negocio** (el paciente). El valor aparece solo cuando las tres se combinan en la misma "consulta".

**🔧 Definición técnica:** disciplina que extrae conocimiento accionable desde datos combinando estadística, computación y dominio del negocio. Su núcleo operativo es el **Machine Learning**: según la definición canónica de (Mitchell, 1997), un programa *aprende* si su desempeño en una tarea **T**, medido por una métrica **P**, mejora con la experiencia **E**. Ese triplete (tarea, métrica, experiencia) es la forma técnica de cualquier proyecto de ML: clasificar fraude (T), medido por recall (P), a partir del historial transaccional (E).

**🧭 Cuándo usarlo:** cuando la decisión se repite muchas veces (miles de créditos, millones de transacciones), hay datos históricos que contienen el patrón, y el costo de equivocarse es cuantificable. Si la decisión es única e irrepetible (¿compramos esta empresa?), el análisis clásico y el juicio experto siguen mandando: el ML aprende de repetición, no de excepciones.

**👔 En una frase para el negocio:** convierte el historial de tu operación en una máquina de predicciones que mejora las decisiones repetitivas y libera a tus expertos para las excepcionales.

### 1.1 AI, Machine Learning y Deep Learning: quién contiene a quién

Audiencia: 🧭 👔

```
┌────────────────────────────────────────────────────────────┐
│  INTELIGENCIA ARTIFICIAL (AI)                              │
│  Cualquier técnica que imite comportamiento inteligente    │
│  (incluye reglas escritas a mano, búsqueda, planificación) │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  MACHINE LEARNING (ML)                               │  │
│  │  Sistemas que aprenden patrones desde datos          │  │
│  │  sin ser programados regla por regla                 │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  DEEP LEARNING (DL)                            │  │  │
│  │  │  ML con redes neuronales profundas que         │  │  │
│  │  │  aprenden sus propias representaciones         │  │  │
│  │  │  (imágenes, texto, audio, secuencias)          │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

Regla de bolsillo 🧭: si alguien te vende "IA", pregunta si **aprende de datos** (ML) o si son **reglas fijas** (AI clásica). Y si aprende, pregunta si el problema realmente necesita deep learning ([[12-Deep-Learning]]) o si un modelo tabular clásico ([[07-Modelos-Supervisados]]) lo resuelve más barato y más explicable.

### 1.2 Data Science vs BI vs Estadística clásica

Audiencia: 🧭 👔

| Dimensión | Business Intelligence | Estadística clásica | Data Science / ML |
|---|---|---|---|
| Pregunta central | ¿Qué pasó? | ¿Es real este efecto? | ¿Qué va a pasar y qué hacemos? |
| Mirada temporal | Pasado (reportes, dashboards) | Pasado con inferencia rigurosa | Futuro (predicción) y acción (prescripción) |
| Producto típico | Dashboard, KPI, reporte | Test de hipótesis, intervalo de confianza | Modelo en producción que decide o recomienda |
| Volumen y variedad | Datos estructurados, agregados | Muestras diseñadas | Datos masivos, estructurados y no estructurados |
| Cuándo conviene | Monitorear la operación y comunicar resultados | Validar causalidad y efectos (¿la campaña funcionó?) | Automatizar predicciones repetitivas a escala |

No compiten: se apilan. El BI te dice que la fuga de clientes subió; la estadística confirma que no es ruido; el ML te dice **qué clientes específicos** se van a ir el próximo mes y cuánto vale retenerlos. La distinción entre modelar para *explicar* y modelar para *predecir* fue formalizada por (Breiman, 2001b) como "las dos culturas" del modelado estadístico.

---

## 2. Los cuatro tipos de aprendizaje

Audiencia: 🔧 🧭 👔

El ML se organiza según **qué información de supervisión** recibe el algoritmo durante el entrenamiento. Esta clasificación no es académica: determina qué datos necesitas tener antes de empezar y qué tomos de esta guía aplican a tu problema.

```
                        ¿Tienes ejemplos con la "respuesta correcta" (target)?
                                          │
              ┌───────────────────────────┼───────────────────────────────┐
              ▼                           ▼                               ▼
        SÍ, completos              SOLO ALGUNOS                    NO, ninguno
              │                           │                               │
     APRENDIZAJE SUPERVISADO     SEMI-SUPERVISADO              NO SUPERVISADO
     (T07, T08, T10, T11)        (combina ambos mundos)        (T06, T09)
              │                                                           │
   ¿El target es categoría o número?                        ¿Busco grupos o relaciones?
    ├─ Categoría → Clasificación                             ├─ Grupos    → Clustering (T06)
    └─ Número    → Regresión                                 └─ Relaciones→ Asociación (T09)

     Y aparte, aprendiendo por prueba y error con recompensas:
     APRENDIZAJE POR REFUERZO (agentes que interactúan con un entorno)
```

### Aprendizaje supervisado

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Es estudiar con un solucionario: te dan miles de ejercicios **con la respuesta correcta al lado** (cliente que sí pagó / cliente que no pagó). El estudiante (modelo) practica hasta que puede resolver ejercicios nuevos sin mirar la respuesta. La calidad del solucionario lo es todo: si las respuestas están mal anotadas, aprende mal.

**🔧 Definición técnica:** aprende una función f(X) → y desde pares (features, target) etiquetados. Si y es discreta → clasificación (fraude/no fraude); si es continua → regresión (monto de venta). Requiere datos históricos etiquetados, y su evaluación honesta exige separar train/test ([[10-Validacion-y-Leakage]]).

**🧭 Cuándo usarlo:** cuando el evento que quieres predecir ya ocurrió muchas veces en el pasado y quedó registrado. Es la familia con mayor retorno probado en negocio: scoring, churn, demanda, mantención predictiva. Detalle completo en [[07-Modelos-Supervisados]].

**👔 En una frase para el negocio:** "aprende del pasado etiquetado para predecir el futuro": si tu historial registra qué pasó con cada caso, puedes anticipar qué pasará con los casos nuevos.

### Aprendizaje no supervisado

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Llegas a una fiesta donde no conoces a nadie y nadie lleva etiqueta con su nombre o profesión. Aun así, tras observar un rato, distingues grupos: los que hablan de fútbol, los que bailan, los del rincón del café. Nadie te dijo cuántos grupos había ni cómo se llamaban: **la estructura emergió sola de observar similitudes**. Eso hace el aprendizaje no supervisado con tus datos.

**🔧 Definición técnica:** encuentra estructura en datos **sin target**: clusters (K-Means, HDBSCAN — [[06-Clustering]]), reglas de coocurrencia (Apriori, FP-Growth — [[09-Reglas-de-Asociacion]]) o representaciones comprimidas (PCA, UMAP — [[03-Preparacion-de-Datos]]). Su validación es más delicada precisamente porque no hay "respuesta correcta" contra la cual medir.

**🧭 Cuándo usarlo:** cuando etiquetar es caro o imposible, o cuando la pregunta es exploratoria: ¿cuántos tipos de clientes tengo?, ¿qué productos se compran juntos?, ¿qué transacciones se ven raras? Suele ser el paso previo que ordena el terreno para un proyecto supervisado posterior.

**👔 En una frase para el negocio:** descubre segmentos, patrones y anomalías que nadie sabía que existían — responde "¿qué hay en mis datos?" antes de "¿qué va a pasar?".

### Aprendizaje semi-supervisado

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un profesor corrige 50 ensayos de un total de 5.000 y con esos 50 entrena a un ayudante; el ayudante corrige el resto y el profesor solo revisa los casos donde el ayudante dudó. Con un puñado de ejemplos etiquetados y una montaña sin etiquetar, se logra casi el resultado de etiquetarlo todo, a una fracción del costo.

**🔧 Definición técnica:** combina pocos datos etiquetados con muchos sin etiquetar. Técnicas típicas: self-training (el modelo etiqueta lo que predice con alta confianza y se reentrena), label propagation sobre grafos de similitud, y pre-entrenamiento no supervisado + fine-tuning supervisado (el patrón dominante en deep learning moderno — [[12-Deep-Learning]]).

**🧭 Cuándo usarlo:** cuando etiquetar requiere expertos caros (diagnósticos médicos, revisión legal) pero los datos crudos abundan. El clustering ([[06-Clustering]]) también sirve aquí: agrupa primero y etiqueta un representante por grupo.

**👔 En una frase para el negocio:** reduce drásticamente el costo de etiquetado — pagas por etiquetar el 1% y el sistema aprovecha el 99% restante.

### Aprendizaje por refuerzo

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Así aprendiste a andar en bicicleta: nadie te dio un manual con las ecuaciones del equilibrio. Probaste, te caíste (castigo), avanzaste unos metros (recompensa), y tu cerebro fue ajustando la política de pedaleo hasta dominarla. El agente de refuerzo aprende igual: actuando, recibiendo recompensas o castigos, y ajustando su estrategia.

**🔧 Definición técnica:** un agente aprende una política de acciones interactuando con un entorno para maximizar la recompensa acumulada. Formalizado como procesos de decisión de Markov; algoritmos como Q-learning y policy gradients. Requiere un entorno donde equivocarse sea barato (simulador) o millones de interacciones.

**🧭 Cuándo usarlo:** decisiones **secuenciales** donde cada acción cambia el estado siguiente: pricing dinámico, gestión de inventario, bidding publicitario, robótica, recomendación interactiva. Es la familia más costosa de llevar a producción; agotar antes las alternativas supervisadas. Su versión mínima y de alto uso en negocio — los multi-armed bandits — está en [[21-Supervivencia-y-Bandits]].

**👔 En una frase para el negocio:** optimiza estrategias completas (no predicciones aisladas) en problemas donde las decisiones de hoy condicionan las opciones de mañana.

---

## 3. ¿Qué problemas de negocio resuelve?

Audiencia: 🧭 👔

Casi cualquier problema de datos cae en una de estas familias. La tabla traduce en ambas direcciones: si tienes la pregunta de negocio, te dice la técnica; si el equipo nombra la técnica, te dice qué pregunta está respondiendo.

| Familia de problema | Pregunta de negocio típica | Ejemplos por industria | Tomo |
|---|---|---|---|
| Clasificación | ¿Este caso es A o B? ¿Se irá este cliente? ¿Es fraude? | Banca: default de crédito · Salud: diagnóstico apoyado · Telco: churn | [[07-Modelos-Supervisados]] |
| Regresión | ¿Cuánto? ¿Qué valor tendrá? | Retail: demanda por SKU · Manufactura: vida útil de componente · Banca: monto de pérdida | [[07-Modelos-Supervisados]] |
| Forecasting (series de tiempo) | ¿Cuánto venderé las próximas 8 semanas? | Retail: planificación de inventario · Energía: carga de red | [[17-Series-de-Tiempo]] |
| Clustering / Segmentación | ¿Qué tipos de clientes/productos tengo? | Marketing: segmentos accionables · Telco: perfiles de uso | [[06-Clustering]] |
| Reglas de asociación | ¿Qué cosas ocurren juntas? | Retail: market basket · Salud: comorbilidades · Web: rutas de navegación | [[09-Reglas-de-Asociacion]] |
| Detección de anomalías | ¿Qué se sale del patrón normal? | Banca: fraude transaccional · Manufactura: fallas de sensor · TI: intrusiones | [[07-Modelos-Supervisados]] |
| Recomendación | ¿Qué le ofrezco a cada quién? | E-commerce: next best offer · Streaming: contenido | [[20-Sistemas-de-Recomendacion]] |
| NLP / Visión / Generativa | ¿Qué dice este texto? ¿Qué hay en esta imagen? | Salud: lectura de imágenes · Legal: clasificación de documentos · Servicio: asistentes | [[12-Deep-Learning]] |

> [!warning] ⚠️ Regla crítica: el problema se define antes que la técnica
> El error más caro de un proyecto de DS no es técnico: es resolver brillantemente la pregunta equivocada. Antes de hablar de algoritmos, exige por escrito: **qué decisión** se tomará con la predicción, **quién** la ejecutará, y **cuánto vale** acertar o costaría equivocarse. Si esas tres respuestas no existen, el proyecto aún no está listo para empezar.

---

## 4. El ciclo de vida de un proyecto de Data Science

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un proyecto de DS es como abrir un restaurante. Primero defines el concepto y el público (pregunta de negocio). Luego consigues y limpias los ingredientes — y descubres que esto toma más tiempo que todo lo demás (preparación de datos). Pruebas recetas (modelado), las haces catar a comensales que **no** participaron en la cocina (validación), y solo entonces abres al público (deployment). Y el trabajo no termina en la inauguración: los gustos cambian, los proveedores cambian, y el menú que no se actualiza vacía el local (drift y monitoreo).

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │ 1. PREGUNTA DE NEGOCIO   ¿Qué decisión mejora? ¿Cuánto vale? (T01)   │
  └──────────────┬───────────────────────────────────────────────────────┘
                 ▼
  ┌──────────────────────────┐   El ~80% del tiempo real del proyecto
  │ 2. DATOS: obtención,     │   se gasta aquí, no en los modelos.
  │    limpieza, calidad     │   (T03 · T04 · T05)
  │    EDA y escalado        │
  └──────────────┬───────────┘
                 ▼
  ┌──────────────────────────┐   Baseline simple primero; complejidad
  │ 3. MODELADO              │   solo si paga su costo. (T06–T09, T12)
  └──────────────┬───────────┘
                 ▼
  ┌──────────────────────────┐   Métricas alineadas al costo del error
  │ 4. EVALUACIÓN Y          │   y validación sin trampas. (T08 · T10)
  │    VALIDACIÓN HONESTA    │
  └──────────────┬───────────┘
                 ▼
  ┌──────────────────────────┐   Tuning, ensembles, calibración. (T11)
  │ 5. MEJORA ITERATIVA      │
  └──────────────┬───────────┘
                 ▼
  ┌──────────────────────────┐   Deployment, monitoreo de drift,
  │ 6. PRODUCCIÓN Y          │   reentrenamiento, ética. (T13 · T14)
  │    MONITOREO             │
  └──────────────┬───────────┘
                 │
                 └────────── iteración: el ciclo nunca "termina" ──────────┐
                                                                           ▼
                                                     vuelta a 1 con lo aprendido
```

**🧭 Las tres verdades incómodas del ciclo:**

1. **El 80% del esfuerzo es datos, no modelos.** Un dataset pequeño pero limpio y representativo supera a uno masivo pero sesgado ([[03-Preparacion-de-Datos]], [[04-EDA]], [[05-Escalado-de-Datos]]). El encuestón del *Literary Digest* en 1936 falló con 10 millones de respuestas por sesgo de selección: el tamaño no salva a la calidad.
2. **El modelo es una fracción del sistema.** El código de ML es una pieza pequeña rodeada de infraestructura de datos, serving y monitoreo; ignorar ese entorno genera "deuda técnica oculta" que se paga con intereses (Sculley et al., 2015). Por eso existe [[13-MLOps-XAI-Etica]].
3. **Un modelo sin monitoreo se degrada en silencio.** El mundo cambia (drift); el modelo entrenado con el mundo de ayer decide cada vez peor sobre el mundo de hoy ([[13-MLOps-XAI-Etica]]).

> [!danger] 🚨 El error más costoso: validar con trampa
> La causa número uno de modelos que brillan en la presentación y fallan en producción es el **data leakage**: información del futuro o del conjunto de prueba que se filtra al entrenamiento. Es silencioso, hace ver métricas espectaculares, y cuesta millones cuando se descubre tarde. Todo el [[10-Validacion-y-Leakage|Tomo 10]] está dedicado a detectarlo y prevenirlo. Señal de alerta ejecutiva: resultados "demasiado buenos para ser verdad" casi siempre lo son.

---

## 5. El valor de negocio (y el costo de hacerlo mal)

Audiencia: 👔 🧭

El valor no está en el modelo: está en la **decisión que el modelo mejora**. Un AUC de 0.92 no vale nada por sí mismo; vale porque permite llamar primero a los 500 clientes con mayor riesgo de fuga en lugar de llamar 5.000 al azar. La conversación ejecutiva correcta nunca es "¿qué tan preciso es el modelo?" sino "¿cuánto mejora la decisión respecto de cómo la tomamos hoy?" — y esa comparación contra el *baseline* actual debe ser explícita en cada proyecto.

**Dónde se destruye valor (los cuatro fracasos clásicos):**

- **Resolver la pregunta equivocada:** predicción técnicamente correcta que nadie usa porque no conecta con una decisión real.
- **Datos que no representan la realidad:** sesgos de selección, histórico contaminado, etiquetas mal puestas ([[03-Preparacion-de-Datos]]).
- **Métricas desalineadas del costo del error:** optimizar accuracy cuando lo caro son los falsos negativos ([[08-Metricas-de-Evaluacion]]).
- **Validación con trampa y producción sin monitoreo:** leakage ([[10-Validacion-y-Leakage]]) y drift ([[13-MLOps-XAI-Etica]]).

> [!example] 📊 Caso de negocio — Retención de clientes en banca
> **Problema:** un banco pierde ~2% de sus clientes de alto valor cada trimestre. La campaña de retención llama a 20.000 clientes elegidos por reglas simples (antigüedad, saldo), con 1 de cada 20 llamadas llegando a alguien que realmente pensaba irse. El costo por contacto es alto y el equipo comercial desconfía de las listas.
>
> **Técnica aplicada:** modelo de clasificación supervisada ([[07-Modelos-Supervisados]]) entrenado con el historial de fugas: transaccionalidad, uso de productos, reclamos, señales de inactividad. Validación temporal estricta para no "adivinar el pasado" ([[10-Validacion-y-Leakage]]), métrica priorizada: recall en el decil superior de riesgo ([[08-Metricas-de-Evaluacion]]), y explicabilidad por cliente para que el ejecutivo sepa *por qué* está en la lista ([[13-MLOps-XAI-Etica]]).
>
> **Resultado:** con el mismo presupuesto de llamadas, la campaña contacta al 20% de clientes que concentra ~70% de las fugas reales. La tasa de acierto por llamada se multiplica, el equipo comercial adopta la lista porque cada nombre viene con sus razones, y la conversación del comité pasa de "¿funciona el modelo?" a "¿cuánto más invertimos en retención?". El mismo patrón — priorizar con probabilidades en lugar de reglas fijas — se replica en cobranza, mantención predictiva y detección de fraude.

**Las cinco preguntas que un ejecutivo debe hacer ante cualquier proyecto de DS** (las respuestas detalladas están en [[14-Anexo-Interpretar-Resultados]]):

1. ¿Qué decisión concreta mejora este modelo y quién la ejecuta?
2. ¿Contra qué baseline se compara? ¿Cuánto mejor es que lo que hacemos hoy?
3. ¿La métrica elegida refleja el costo real de equivocarse en mi negocio?
4. ¿Cómo se validó? ¿Los datos de prueba eran realmente "futuro no visto"?
5. ¿Qué pasa después del deployment: quién monitorea, cada cuánto se reentrena, cuándo se apaga?

---

## 6. Los roles del equipo de datos

Audiencia: 🧭 👔

> [!tip] 💡 Analogía
> Una cocina profesional: el **Data Engineer** es quien consigue y almacena ingredientes frescos todos los días (sin él, no hay cocina); el **Data Scientist** es el chef que crea y prueba recetas; el **ML Engineer** industrializa la receta para servir 10.000 platos por noche sin que se caiga la calidad; el **Analista/BI** es el maître que observa el salón y reporta qué pidió la gente; y el **Product Owner** decide el menú según el público del restaurante. Pedirle a una sola persona los cinco roles funciona en un food truck, no en un restaurante a escala.

| Rol | Qué hace | Entregable típico | Tomos de esta guía que domina |
|---|---|---|---|
| Data Engineer | Pipelines de ingesta, calidad y disponibilidad de datos | Data warehouse/lake confiable | [[03-Preparacion-de-Datos]], [[13-MLOps-XAI-Etica]] |
| Data Scientist | Explora, modela, valida, comunica | Modelo validado + análisis | Todos, con foco en [[04-EDA]]–[[12-Deep-Learning]] |
| ML Engineer | Lleva modelos a producción y los mantiene sanos | API/servicio de predicción monitoreado | [[10-Validacion-y-Leakage]]–[[13-MLOps-XAI-Etica]] |
| Analista / BI | Mide el negocio, detecta señales, traduce hallazgos | Dashboards, análisis ad-hoc | [[01-Introduccion-Ejecutiva]], [[04-EDA]], [[08-Metricas-de-Evaluacion]], [[14-Anexo-Interpretar-Resultados]] |
| Product Owner / Sponsor | Define la decisión de negocio, prioriza, remueve bloqueos | Caso de negocio y criterios de éxito | [[01-Introduccion-Ejecutiva]], [[08-Metricas-de-Evaluacion]], [[10-Validacion-y-Leakage]], [[14-Anexo-Interpretar-Resultados]] |

---

## 7. Mapa de lectura por perfil

Audiencia: 🔧 🧭 👔

| Si eres… | Empieza por | Sigue con | Tu objetivo al terminar |
|---|---|---|---|
| 👔 Ejecutivo / Sponsor | Este tomo | [[14-Anexo-Interpretar-Resultados]] → [[15-Glosario-Ejecutivo]] → callouts 👔 de los tomos 08, 10 y 13 | Hacer las preguntas correctas y detectar humo en 5 minutos |
| 🧭 Product Owner / Líder / Analista | Este tomo | [[04-EDA]] → [[08-Metricas-de-Evaluacion]] → [[10-Validacion-y-Leakage]] → bloques 🧭 de [[03-Preparacion-de-Datos|03]], [[07-Modelos-Supervisados|07]] y [[11-Mejora-de-Modelos|11]] → [[13-MLOps-XAI-Etica]] | Traducir negocio↔técnica y desafiar decisiones con criterio |
| 🔧 Data Scientist / Engineer | [[02-Fundamentos-Matematicos]] | Lectura completa en orden 02 → 13 | Referencia de trabajo diaria: fórmulas, hiperparámetros, trade-offs |

Las rutas detalladas, con tiempos estimados, están en el índice maestro: [[00-MOC-Guia-Maestra#🧑‍🤝‍🧑 Rutas de lectura por perfil|MOC · Rutas de lectura]].

---

## 8. Mitos que cuestan dinero

Audiencia: 👔 🧭

| Mito | Realidad | Dónde profundizar |
|---|---|---|
| "Con más datos, el modelo mejora solo" | Más datos **sesgados** solo producen el error con más confianza. Calidad y representatividad primero. | [[03-Preparacion-de-Datos]] |
| "El modelo tiene 95% de accuracy, es excelente" | Con clases desbalanceadas, un modelo inútil alcanza 99% de accuracy prediciendo siempre "no pasa nada". | [[08-Metricas-de-Evaluacion]] |
| "El algoritmo es objetivo, no discrimina" | El modelo amplifica los sesgos del histórico con el que aprendió. La equidad se audita, no se supone. | [[13-MLOps-XAI-Etica]] |
| "Ya funciona, no hay que tocarlo más" | El mundo cambia (drift): todo modelo en producción se degrada sin monitoreo y reentrenamiento. | [[13-MLOps-XAI-Etica]] |
| "Necesitamos deep learning porque es lo más avanzado" | En datos tabulares, el gradient boosting suele ganar con una fracción del costo y mucha más explicabilidad. | [[07-Modelos-Supervisados]], [[12-Deep-Learning]] |
| "Los resultados del piloto son espectaculares" | Resultados demasiado buenos casi siempre esconden leakage. Exige validación temporal y auditoría de features. | [[10-Validacion-y-Leakage]] |

---

## 9. Checklist ejecutivo antes de aprobar un proyecto de DS

Audiencia: 👔

- [ ] La **decisión de negocio** que mejora está escrita en una frase, con dueño y frecuencia.
- [ ] El **valor de acertar** y el **costo de equivocarse** (en dinero) están estimados — eso definirá la métrica ([[08-Metricas-de-Evaluacion]]).
- [ ] Existe un **baseline**: cómo se toma hoy esa decisión y qué resultado da.
- [ ] Los **datos históricos existen**, son accesibles y contienen el desenlace que se quiere predecir.
- [ ] Hay un plan de **validación honesta** (datos de prueba que simulan el futuro) ([[10-Validacion-y-Leakage]]).
- [ ] Se definió qué pasa **después del piloto**: deployment, monitoreo, reentrenamiento, dueño operativo ([[13-MLOps-XAI-Etica]]).
- [ ] Se evaluaron **riesgos de sesgo y cumplimiento** si el modelo decide sobre personas ([[13-MLOps-XAI-Etica]]).

---

## 📖 Referencias de este tomo

- (Mitchell, 1997) — definición formal de aprendizaje T/P/E.
- (Breiman, 2001b) — las dos culturas del modelado: explicar vs predecir.
- (Davenport & Patil, 2012) — el rol del Data Scientist en la empresa.
- (Sculley et al., 2015) — deuda técnica oculta en sistemas de ML.
- (Géron, 2022) y (James et al., 2021) — panorámicas técnicas del ciclo completo.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Siguiente: [[02-Fundamentos-Matematicos|02 · Fundamentos Matemáticos ➡]]

> **Próximo tomo:** [[02-Fundamentos-Matematicos]] — el lenguaje en que están escritos los modelos: álgebra lineal, cálculo, probabilidad y estadística inferencial, con la misma estructura de analogías y capas por audiencia.

