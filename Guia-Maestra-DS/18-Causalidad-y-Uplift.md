---
title: "Tomo 18 — Causalidad y Uplift Modeling"
tags: [data-science, machine-learning, causalidad, uplift, experimentacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 18
version: 6.3
updated: 2026-09-06
---

# 🎯 Tomo 18 — Causalidad y Uplift Modeling

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[17-Series-de-Tiempo|17 · Series de Tiempo]] · Siguiente: [[19-NLP-y-LLMs|19 · NLP y LLMs ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Todo lo anterior de esta guía **predice**; este tomo trata de **intervenir**. "¿Qué clientes se fugarán?" es predicción ([[07-Modelos-Supervisados]]); "¿la campaña REDUJO la fuga?" y "¿a quién debo contactar para que la llamada cambie su decisión?" son preguntas causales — y responderlas con correlaciones es la forma más elegante de quemar presupuesto. La advertencia de [[02-Fundamentos-Matematicos]] (correlación ≠ causalidad) deja aquí de ser advertencia y se convierte en método.

> [!abstract] 👔 Impacto ejecutivo
> La mayoría de las decisiones caras no preguntan "¿qué pasará?" sino "¿qué pasa SI hago esto?" — una pregunta de otra naturaleza.
>
> - **Decisiones que habilita:** medir el efecto real de campañas, precios y políticas (con o sin experimento), focalizar acciones en quienes la acción cambia, dejar de pagar por resultados que habrían ocurrido solos.
> - **Costo de hacerlo mal:** atribuir a la campaña lo que hizo la estacionalidad, contactar clientes que se habrían quedado gratis (o peor: despertar a los que se iban a quedar), y escalar "aprendizajes" que eran sesgo de selección.
> - **Pregunta ejecutiva que responde:** *¿esta acción CAUSÓ el resultado — y en quiénes vale la pena repetirla?*

> [!tip] 💡 Analogía general: el gallo y el amanecer
> El gallo canta y el sol sale: correlación perfecta, todos los días. Regálale el gallo al vecino y el sol saldrá igual. Media empresa moderna opera con lógica de gallo: "los que reciben el newsletter compran más" (¿o los que ya compraban más se suscribieron al newsletter?). La causalidad es el arte de distinguir el gallo del sol **antes** de invertir en gallos.

**La escalera de la causalidad** (Pearl & Mackenzie, 2018):

```
 Peldaño 3: IMAGINAR   "¿Qué habría pasado SI no hubiéramos lanzado?"  → contrafactuales
 Peldaño 2: HACER      "¿Qué pasa SI lanzamos la campaña?"             → intervención, P(y│do(x))
 Peldaño 1: VER        "¿Qué pasa cuando…?"                            → correlación, P(y│x)
            ────────────────────────────────────────────────────────
            Todo ML predictivo (tomos 03–12) vive en el peldaño 1.
            Las decisiones de negocio viven en los peldaños 2 y 3.
```

> [!warning] ⚠️ Por qué un modelo predictivo excelente no responde una pregunta causal
> Un modelo puede predecir el churn con AUC 0.9 y ser **inútil** para decidir a quién retener: aprendió que "quien llama a cancelar se va", un patrón cierto y sin ninguna palanca de acción. Predicción responde *qué va a pasar*; causalidad responde *qué cambia si intervengo*. Son preguntas de peldaños distintos, y **subir de peldaño requiere supuestos o experimentos, no más datos** ([[10-Validacion-y-Leakage]]).

---

## 1. Potential outcomes: el contrafactual

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Para saber si el paraguas te mantuvo seco necesitarías vivir la misma lluvia **dos veces**: una con paraguas y otra sin — y solo puedes vivir una. Ese "universo paralelo no observado" es el contrafactual, y el problema fundamental de la inferencia causal es que nunca lo ves. Todo el tomo es ingeniería para estimar el universo que no ocurrió.

**🔧 Definición técnica:** el modelo de Rubin (Rubin, 1974): cada unidad i tiene dos resultados potenciales — Y₁(i) con tratamiento e Y₀(i) sin él. El efecto individual `τᵢ = Y₁(i) − Y₀(i)` es **inobservable** (solo ves uno de los dos). Lo estimable:

| Estimando | Qué mide | Uso típico |
|---|---|---|
| **ATE** = E[Y₁ − Y₀] | Efecto promedio en toda la población | ¿Vale la pena la política en general? |
| **ATT** = E[Y₁ − Y₀ │ T=1] | Efecto promedio sobre los que fueron tratados | Evaluar una campaña ya ejecutada |
| **CATE** = E[Y₁ − Y₀ │ X=x] | Efecto por segmento/individuo | **La base del uplift** (sección 7) |

La aleatorización hace que tratados y no tratados sean intercambiables → la diferencia de medias estima el ATE sin sesgo.

### 1.1 Los cuatro supuestos que hacen posible la causalidad

Audiencia: 🔧

Estimar un contrafactual no sale gratis: descansa en supuestos que **hay que nombrar y defender**, porque cuando una afirmación causal falla, casi siempre es porque uno de estos se rompió en silencio.

| Supuesto | Qué exige | Cómo se rompe en la práctica |
|---|---|---|
| **SUTVA** | El tratamiento de una unidad no afecta el resultado de otra; una sola versión del tratamiento | Efectos de red/spillover (descuento a un usuario que se lo pasa a su amigo del grupo control) |
| **Ignorabilidad** (unconfoundedness) | Dado X, la asignación al tratamiento es "como aleatoria": no hay confounders no observados | Siempre hay una variable que no mediste (motivación, salud, urgencia) |
| **Positividad** (overlap) | Toda unidad tiene probabilidad > 0 de recibir cada tratamiento | Segmentos que SIEMPRE se tratan (o nunca): no hay con quién compararlos |
| **Consistencia** | El resultado potencial observado es el que corresponde al tratamiento recibido | Tratamiento mal definido o mal medido ("recibió la campaña" ¿la abrió? ¿la leyó?) |

> [!danger] 🚨 La ignorabilidad no se puede testear con los datos
> Este es el punto más incómodo de toda la inferencia observacional: **no existe un test estadístico que confirme que no hay confounders ocultos.** Los datos que tienes no pueden hablar de las variables que no mediste. Por eso los métodos de la sección 4 no "prueban" causalidad — la asumen bajo un supuesto que se defiende con conocimiento del dominio, no con un p-valor. La honestidad está en decir cuál es el supuesto, no en esconderlo.

**👔 En una frase para el negocio:** el impacto de una acción se define contra el mundo donde NO la hiciste — todo lo que no estime ese mundo comparándolo bien está midiendo otra cosa.

---

## 2. DAGs: confounders, colliders y mediadores

Audiencia: 🔧 🧭

**🔧 Definición técnica:** los DAGs (grafos dirigidos acíclicos) codifican supuestos causales y dictan **qué controlar y qué NO**:

```
 CONFOUNDER (controlar SÍ)      COLLIDER (controlar NO)        MEDIADOR (depende)
      Verano                      Talento    Suerte                Campaña
      ↙    ↘                          ↘      ↙                        ↓
 Helados   Ahogados                    Fama                       Visitas
                                 (condicionar en "fama"              ↓
 la correlación helados-         CREA correlación falsa           Ventas
 ahogados es fabricada           entre talento y suerte)      (controlar "visitas"
 por el verano                                                 borra el efecto que
                                                               quieres medir)
```

- **Confounder:** causa común de tratamiento y outcome → SIN controlarlo, la comparación está sesgada ([[02-Fundamentos-Matematicos]]).
- **Collider:** efecto común de dos causas → **controlarlo fabrica** una correlación que no existía (sesgo de selección: analizar "solo los clientes que llamaron a soporte" condiciona en un collider).
- **Mediador:** está en el camino causal → controlarlo borra parte del efecto que buscas medir.

**🔧 El backdoor criterion en una línea:** para estimar el efecto de T sobre Y, controla el conjunto de variables que **bloquea todos los caminos "por la puerta trasera"** (los que conectan T e Y sin pasar por la flecha causal), **sin** abrir caminos nuevos (sin condicionar colliders). Ese conjunto — no "todas las variables" — es lo que va en el ajuste.

> [!danger] 🚨 "Controlar por todo" es un error, no una virtud
> Meter todas las variables disponibles al ajuste no purifica el análisis: controlar colliders y mediadores **crea** sesgos nuevos. Qué controlar se decide con un diagrama causal (aunque sea dibujado a mano con el experto de negocio), no con un `SELECT *`. Corolario — la **Table 2 fallacy**: en una regresión multivariada, los coeficientes de las variables de control **no** son efectos causales interpretables; solo el de la variable de interés lo es (y solo si el ajuste fue el correcto). Leer toda la tabla como "efectos" es un error clásico de reporte.

**👔 En una frase para el negocio:** antes de "ajustar por las variables", hay que dibujar quién causa a quién — controlar la variable equivocada sesga en lugar de corregir.

---

## 3. El experimento (RCT / A/B test): el gold standard

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** la asignación aleatoria corta TODAS las flechas entrantes al tratamiento: no hay confounders, observados ni ocultos. Por eso el A/B test es el peldaño 2 comprado con diseño. Toda la maquinaria estadística ya está en [[02-Fundamentos-Matematicos]]: potencia y tamaño muestral ANTES, p-valor + IC + tamaño de efecto DESPUÉS, corrección por múltiples pruebas si hay varios tests.

> [!tip] 💡 Analogía
> La aleatorización es un cara-o-cruz que reparte a las personas en dos grupos **estadísticamente gemelos** en todo — lo que mediste y lo que no. Cualquier diferencia posterior en el resultado solo puede venir del tratamiento, porque es lo único que difiere entre los gemelos. Es la única máquina que fabrica contrafactuales creíbles sin supuestos sobre el dominio.

### 3.1 Lo que puede salir mal aunque aleatorices

Audiencia: 🔧 🧭

El RCT es el gold standard, pero no es a prueba de balas. Estas son las grietas que un análisis serio revisa:

| Problema | Qué es | Defensa |
|---|---|---|
| **Unidad de aleatorización equivocada** | Aleatorizar por usuario cuando el efecto contamina al grupo (redes sociales, marketplaces) | Aleatorizar por cluster (ciudad, tienda, cohorte) |
| **Spillover / contaminación** | El tratado influye en el control (viola SUTVA) | Diseño por cluster; medir el efecto de red aparte |
| **Non-compliance** | Asignaste tratamiento pero la persona no lo "tomó" (no abrió el mail) | **ITT** (intention-to-treat: analizas por asignación, no por cumplimiento) como estándar honesto; IV para el efecto en los que cumplen |
| **Sample Ratio Mismatch (SRM)** | La división real no es la planeada (50/50 que llega 48/52) | Chi-cuadrado sobre los tamaños; si falla, hay un bug en el pipeline del experimento |
| **Efecto novedad / primacía** | El cambio gusta (o molesta) solo por ser nuevo; se disipa | Correr el test suficiente tiempo; mirar la curva en el tiempo, no solo el promedio |
| **Peeking** | Mirar resultados y parar cuando "da significativo" | Fijar el tamaño de antemano, o usar tests secuenciales diseñados para eso |

> [!warning] ⚠️ ITT vs. per-protocol: analiza por como asignaste, no por como cumplieron
> Si analizas "solo los que efectivamente usaron el producto", estás condicionando en un **collider** (la decisión de usar depende de variables no observadas) y reintroduces el sesgo que la aleatorización había eliminado. El análisis honesto por defecto es **intention-to-treat**: cada quien cuenta en el grupo al que fue asignado, cumpla o no. Suena contraintuitivo, pero es lo que preserva la validez causal.

**🧭 Cuándo NO se puede experimentar:** ética (no le niegas un beneficio de salud a un grupo al azar), costo/riesgo (no subes precios al azar a la mitad de la cartera), historia (la política ya se aplicó a todos). Ahí entran los métodos observacionales.

**👔 En una frase para el negocio:** la aleatorización es la forma más barata de comprar una verdad causal — pero solo si nadie hizo peeking, revisaste el SRM y analizaste por asignación (ITT); un experimento corrido a la ligera dice mentiras con la autoridad de un p-valor.

---

## 4. Métodos observacionales: causalidad sin experimento

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Si no puedes hacer el experimento, lo **imitas**: buscas gemelos (matching), aprovechas que dos regiones venían caminando igual y solo una recibió la política (diff-in-diff), o usas una regla arbitraria de corte como si fuera un sorteo local (regresión discontinua). Cada método es una forma de fabricar el grupo de control que la realidad no te dio — y cada uno cobra su precio en supuestos.

| Método | Idea | Supuesto clave (el precio) | Cuándo conviene |
|---|---|---|---|
| Matching / Propensity Score | Emparejar tratados con no tratados de igual probabilidad de tratamiento `e(x) = P(T=1│X)` (Rosenbaum & Rubin, 1983) | **Ignorabilidad**: no hay confounders NO observados | Muchos controles disponibles y buenos covariables observados |
| Diff-in-Diff (DiD) | Comparar el cambio antes/después del grupo tratado contra el cambio del grupo no tratado | **Tendencias paralelas**: sin el tratamiento, ambos grupos habrían evolucionado igual | Políticas aplicadas a una región/tienda/segmento con comparables. Con adopción escalonada (varias fechas de inicio), usar un estimador robusto — ver callout |
| Regresión Discontinua (RDD) | Comparar justo a ambos lados de un umbral arbitrario (puntaje de corte para beca/crédito) | Alrededor del corte, caer a un lado u otro es "casi azar" | Existe una regla de asignación con umbral nítido |
| Variables Instrumentales (IV) | Usar una variable que mueve el tratamiento pero NO el outcome directamente | Exclusión: el instrumento solo actúa vía el tratamiento (difícil de defender) | Confounders no observados + un instrumento creíble |
| Synthetic Control (Abadie, Diamond & Hainmueller, 2010; guía práctica: Abadie, 2021) | Construir un "clon sintético" del tratado como combinación ponderada de no tratados | El clon replica bien la trayectoria pre-tratamiento | Pocas unidades grandes (una ciudad, una marca) |
| Double / Debiased ML | ML flexible para modelar tratamiento y outcome, con ortogonalización y cross-fitting para estimar el efecto (Chernozhukov et al., 2018) | Ignorabilidad + buenos modelos de nuisance | Muchos covariables, relaciones no lineales; el puente moderno ML↔causalidad |

> [!tip] 💡 La joya escondida: Diff-in-Diff en una imagen
> ```
>   resultado
>     │                          ● tratado (observado)
>     │                       ╱
>     │              ● ─ ─ ─ ○  ← contrafactual: dónde habría estado
>     │            ╱             el tratado SIN la política
>     │     ● ───╱── ● control
>     │   ╱     ╱
>     │ ●─────●
>     └──────┬────────┬──────► tiempo
>          antes   después
>   El EFECTO es la brecha entre el tratado real y su contrafactual (línea punteada),
>   que se estima asumiendo que ambos venían en PARALELO. Si no venían en paralelo
>   antes, el supuesto falla y el número miente.
> ```

> [!warning] ⚠️ DiD con adopción escalonada: el 2×2 no escala solo
> En la práctica la política rara vez llega a todos a la vez: se despliega por tiendas, regiones o cohortes en fechas distintas (*staggered adoption*). La extensión «natural» — una regresión con efectos fijos de unidad y de período (*two-way fixed effects*, TWFE) — **no** es un DiD promedio inocente: es un promedio ponderado de todos los 2×2 posibles, incluyendo comparaciones que usan a unidades ya tratadas como «control» de las recién tratadas (Goodman-Bacon, 2021). Si el efecto cambia con el tiempo o difiere entre cohortes, algunos pesos son negativos y el coeficiente puede tener el signo contrario al de todos los efectos reales (de Chaisemartin & D'Haultfœuille, 2020). El estándar actual es estimar el efecto por cohorte y período y agregarlo explícitamente (Callaway & Sant'Anna, 2021; Sun & Abraham, 2021); la síntesis para practicantes es Roth et al. (2023). Regla operativa: si tu tratamiento tiene más de una fecha de inicio, TWFE es la hipótesis a refutar, no el resultado.

> [!tip] 💡 Synthetic control: cuándo creerle y su puente con DiD
> El clon sintético solo es creíble si se cumplen condiciones que Abadie (2021) recomienda revisar antes de estimar: un *donor pool* de unidades comparables no afectadas por la política (sin spillover), sin anticipación del tratamiento, un período pre suficientemente largo para que el ajuste no sea casualidad, y una unidad tratada que quede «dentro del rango» de los donantes — los pesos son no negativos y suman uno, así que el método no extrapola. La inferencia no usa p-valores clásicos sino **placebos en el espacio**: se estima el «efecto» para cada donante como si hubiera sido el tratado y se mira si el real destaca entre ellos (Abadie, Diamond & Hainmueller, 2010). Cuando el ajuste pre no es perfecto, el **synthetic difference-in-differences** (Arkhangelsky et al., 2021) combina ambas ideas — pesos de unidad y de tiempo más efectos fijos — y resulta más robusto que DiD o synthetic control por separado.

**👔 En una frase para el negocio:** sin experimento igual se puede estimar impacto — pero cada método compra su conclusión con un supuesto que debe explicitarse y defenderse; pregunta siempre *cuál es el supuesto y qué pasa si falla*.

---

## 5. Cómo defender una afirmación causal: pruebas de robustez

Audiencia: 🔧 🧭 👔

Como la ignorabilidad no se puede testear (sección 1.1), un análisis observacional serio **no afirma y ya**: se somete a sí mismo a intentos de refutación. Estas son las pruebas que separan un hallazgo defendible de una corazonada con tabla.

| Prueba | Idea | Qué demuestra |
|---|---|---|
| **Placebo test** | Aplicar el método a un período/grupo donde el efecto DEBERÍA ser cero | Si "detecta" un efecto donde no puede haberlo, el método está sesgado |
| **Negative control** | Un outcome que el tratamiento no puede causar | Si aparece afectado, hay un confounder acechando |
| **Tendencias pre-tratamiento** | En DiD, verificar que los grupos venían paralelos ANTES | Un pre-trend visible derriba el supuesto; uno «no significativo» NO lo confirma — ver callout |
| **Sensibilidad a confounders ocultos** | ¿Cuán fuerte tendría que ser un confounder no medido para anular el efecto? (E-value; Rosenbaum bounds; robustness value de Cinelli & Hazlett, 2020) | Cuantifica cuán frágil es la conclusión |
| **Refutación por placebo de tratamiento** | Reasignar el tratamiento al azar y confirmar que el efecto desaparece | El efecto no era un artefacto del método (DoWhy lo automatiza) |

> [!warning] ⚠️ El pre-trend test no es un certificado de tendencias paralelas
> Mirar cómo venían los grupos antes sigue siendo obligatorio, pero con dos matices que la literatura reciente estableció. Primero, los tests de pre-tendencias suelen tener **poca potencia**: no rechazar no equivale a «paralelas confirmadas». Segundo, condicionar el análisis a haber «pasado» el test distorsiona la estimación y la cobertura de los intervalos de confianza (Roth, 2022). La respuesta moderna es cambiar el test binario por un **análisis de sensibilidad**: acotar cuánto podría desviarse la tendencia post-tratamiento respecto de la mayor desviación observada en el período pre (por ejemplo, «a lo más M veces esa diferencia») y reportar el rango de efectos compatible con cada cota (Rambachan & Roth, 2023). Un hallazgo es defendible si sobrevive a violaciones del tamaño de las que ya se vieron antes del tratamiento; si solo sobrevive con paralelismo exacto, es frágil y hay que decirlo.

> [!tip] 💡 Analogía
> Una afirmación causal seria se parece a un puente: no basta con que se vea firme, hay que **cargarlo a propósito** para ver si aguanta. Los placebo tests y los controles negativos son los camiones que subes al puente antes de abrirlo al público. Si el analista no intentó tumbar su propio hallazgo, alguien lo hará después — y más caro.

> [!danger] 🚨 La pregunta que desarma el 80% de los "hallazgos causales"
> *"¿Qué confounder no medido explicaría este resultado, y cuán grande tendría que ser?"* Si la respuesta es "uno pequeño y plausible bastaría" (por ejemplo: la motivación del cliente, que nunca mides), la conclusión es frágil por más sofisticado que haya sido el modelo. El **E-value** (VanderWeele & Ding, 2017) traduce esa fragilidad a un número: la asociación mínima (en escala de risk ratio) que un confounder tendría que tener, a la vez con el tratamiento y con el outcome, para anular el efecto. Úsalo con dos advertencias: es una cota bajo supuestos, no una medición, y es una función monótona del propio estimado, así que no aporta información nueva ni existe un umbral universal de «E-value suficiente» (Ioannidis, Tan & Blum, 2019 — una crítica breve y debatida, no un consenso). Para modelos de regresión, el complemento hoy habitual es el **robustness value** de Cinelli & Hazlett (2020): la fuerza mínima de asociación (en R² parcial con tratamiento y outcome) que necesitaría un confounder no observado para cambiar la conclusión, calculable con la salida estándar de la regresión y comparable contra covariables ya observadas («¿tendría que ser más fuerte que la edad?»). Esa comparación con variables conocidas es lo que convierte la sensibilidad en argumento de dominio y no en un número suelto.

**👔 En una frase para el negocio:** exige que quien te trae un número causal te muestre **cómo intentó refutarlo**; un efecto que nadie trató de tumbar no está validado, solo está sin auditar.

---

## 6. Falacias causales frecuentes

Audiencia: 🔧 🧭 👔

Los errores que más dinero cuestan no son de cálculo, son de razonamiento. Reconocerlos por nombre es media defensa.

| Falacia | Qué es | Ejemplo de negocio |
|---|---|---|
| **Paradoja de Simpson** | Una tendencia se invierte al desagregar por grupos (Simpson, 1951) | La campaña "mejora" la conversión global, pero la empeora en CADA segmento — el mix cambió |
| **Regresión a la media** | Un valor extremo tiende al promedio por azar, sin causa | "Intervenimos a los peores vendedores y mejoraron" — habrían mejorado solos |
| **Sesgo de supervivencia** | Analizar solo a los que "quedaron" | Estudiar la lealtad "de los clientes actuales" ignora a los que ya se fueron |
| **Sesgo de selección / collider** | Condicionar en un efecto común de dos causas | "Entre los contratados, talento y experiencia se correlacionan negativo" — lo creó el filtro de contratación |
| **Post hoc ergo propter hoc** | "Ocurrió después, luego lo causó" | Lanzamos el rebranding y subieron las ventas (¿o fue la temporada?) |
| **Efecto placebo / Hawthorne** | El solo hecho de medir o intervenir cambia el comportamiento | La productividad sube porque el equipo se sabe observado, no por la nueva política |

> [!warning] ⚠️ La paradoja de Simpson no es una curiosidad de manual
> Es el argumento más fuerte a favor de dibujar el DAG antes de agregar (sección 2): **si agregas cuando debías desagregar, o al revés, el número se invierte de signo**. Si un confounder de segmentación (el mix de clientes, el canal, la región) mueve tanto los grupos como el resultado, la cifra global puede decir lo contrario que la realidad de cada segmento. La única defensa es tener claro qué causa qué antes de sumar.

**👔 En una frase para el negocio:** la mayoría de los "aprendizajes" que resultan falsos no fallaron en la ejecución sino en el razonamiento causal — y esos errores se escalan con confianza hasta que cuestan caro.

---

## 7. Uplift Modeling: a quién dirigir la acción

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Cuatro tipos de clientes ante tu llamada de retención: el **persuadible** (se iba, tu llamada lo salva — el único que paga la campaña), el **seguro** (se quedaba igual — le regalaste el descuento), el **perdido** (se va igual — gastaste la llamada), y el **perro dormido** (se quedaba… hasta que tu llamada le recordó que podía irse). El modelo de churn clásico NO distingue entre ellos: rankea por riesgo. El uplift rankea por **cuánto cambia tu acción el resultado** — que es lo único que el presupuesto debería comprar.

```
                        ¿Responde SI lo tratas?
                          Sí            No
 ¿Responde si   Sí │  SEGURO 😐     PERRO DORMIDO 🚨   ← no contactar:
  NO lo tratas?    │  (gasto inútil) (¡la acción daña!)    ¡la acción empeora!
                No │  PERSUADIBLE ✅  PERDIDO 😞
                   │  (el objetivo)  (gasto inútil)
```

**🔧 Definición técnica:** el uplift estima el **CATE**: `τ(x) = P(y│x, tratado) − P(y│x, no tratado)`. Requiere datos con tratamiento y control (idealmente de un experimento previo).

**Metalearners** (Künzel et al., 2019):

| Learner | Cómo | Fortaleza / debilidad |
|---|---|---|
| **S-learner** | Un modelo con el tratamiento como una feature más | Simple; puede **diluir** el efecto si el tratamiento pesa poco entre muchas features |
| **T-learner** | Dos modelos separados (tratados y controles), se restan | Flexible; más **varianza**, sobre todo con grupos chicos |
| **X-learner** | Cruza imputaciones de efecto y pondera por propensity | Fuerte con **grupos desbalanceados** (control pequeño) |
| **Uplift trees** | Árboles que hacen split por diferencia de uplift, no de pureza | Interpretables; directos al objetivo causal |
| **R-learner** | Residualiza outcome y tratamiento contra X (como Double ML) y ajusta el CATE sobre los residuos (Nie & Wager, 2021) | Robusto a errores moderados en los modelos de nuisance; cualquier learner en ambas etapas |
| **Causal forest / GRF** | Random forest cuyos splits buscan heterogeneidad del efecto, con *honest splitting* (Wager & Athey, 2018; Athey, Tibshirani & Wager, 2019) | CATE con intervalos de confianza; estimador por defecto en grf y EconML |

> [!warning] ⚠️ El uplift no se evalúa como un clasificador
> No existe ground truth individual del efecto (nunca ves los dos mundos de una persona — sección 1). Por eso **no hay AUC de uplift al estilo clásico**. Se usa la **curva de uplift / coeficiente Qini** (Radcliffe, 2007): ordenar por uplift predicho y medir la ganancia incremental acumulada frente a targeting aleatorio — es el "AUC del mundo causal". Métricas complementarias: uplift@k (efecto en el top k% al que sí contactarías). **Librerías:** causalml (Uber), EconML (Microsoft), DoWhy (grafos, supuestos y refutación).
>
> **Dos matices actuales sobre la evaluación.** (1) La curva Qini clásica supone tratamiento aleatorizado; con datos observacionales, ordenar por uplift y comparar tasas crudas mezcla efecto con selección: hay que construirla con *doubly robust scores* (propensity más modelos de outcome). (2) Más allá de Qini, el **RATE** (*rank-weighted average treatment effect*; Yadlowsky et al., 2025) generaliza la curva: pondera el efecto acumulado por rango con pesos Qini o **AUTOC** y aporta un teorema del límite central para **testear** si la regla de priorización supera al azar. AUTOC tiene más potencia cuando pocos individuos concentran el efecto; Qini, cuando la heterogeneidad es difusa. Nota de ecosistema: EconML forma parte hoy de PyWhy.

**🧭 Cuándo usarlo:** retención, cross-sell, cobranza, pricing promocional — toda acción cara dirigida a personas donde parte del resultado ocurriría solo. Prerrequisito: haber corrido (o poder correr) un experimento con grupo de control para entrenar.

**👔 En una frase para el negocio:** deja de pagar por retener a quienes se quedaban gratis y de despertar perros dormidos — el uplift redirige el mismo presupuesto exclusivamente hacia los persuadibles.

> [!example] 📊 Caso de negocio — Telco: la campaña que funcionaba mejor haciendo menos
> **Problema:** la telco del [[03-Preparacion-de-Datos|Tomo 03]] ya predice churn con honestidad y llama al 20% de mayor riesgo. La campaña "funciona" (retiene), pero el costo por cliente salvado es alto y hay señales incómodas: algunos contactados se fugan MÁS que sus gemelos no contactados.
>
> **Técnica aplicada:** el experimento de la campaña anterior (con 10% de control aleatorio) se recicla como dataset de uplift. X-learner sobre las mismas features del modelo de churn; evaluación con curva Qini; segmentación de la cartera en los 4 cuadrantes.
>
> **Resultado:** un tercio de los contactados eran "seguros" (descuento regalado) y un grupo pequeño pero caro eran perros dormidos — clientes en piloto automático a quienes la llamada activó a cotizar competencia. Contactando solo al segmento persuadible, la campaña retiene casi lo mismo con la mitad de los contactos, y el costo por cliente salvado cae a la mitad. Moraleja ejecutiva: **riesgo alto no es sinónimo de "contactar"; la pregunta correcta es en quién la acción cambia el desenlace**.

---

## 8. Guía de decisión: qué herramienta causal, cuándo

Audiencia: 🔧 🧭

Síntesis operativa del tomo — de la pregunta a la herramienta:

| Tu situación | Herramienta | Nota |
|---|---|---|
| Puedes asignar al azar | **RCT / A/B test** | El gold standard; vigila las grietas de la sección 3.1 |
| Ya hay un experimento y quieres saber a QUIÉN dirigir | **Uplift (CATE)** | Recicla el control aleatorio como dataset |
| No puedes experimentar, tienes buenos covariables | **Matching / PSM** o **Double ML** | Descansa en ignorabilidad; somételo a robustez (sección 5) |
| Una política llegó a un grupo con comparables | **Diff-in-Diff** | Verifica tendencias paralelas pre-tratamiento |
| Hay una regla de corte nítida (umbral) | **RDD** | Solo estima el efecto CERCA del umbral |
| Confounders no observados + un instrumento creíble | **Variables instrumentales** | El supuesto de exclusión es el talón de Aquiles |
| Una sola unidad grande tratada (una ciudad) | **Synthetic control** | Necesita buen ajuste pre y un donor pool comparable; si el ajuste no es perfecto, Synthetic DiD (Arkhangelsky et al., 2021) |
| **En todos los casos** | **Dibuja el DAG y explicita los supuestos** | Y pregunta cómo se intentó refutar el hallazgo |

> [!tip] 🧭 El orden correcto de trabajo
> Pregunta causal clara → dibujar el DAG (qué controlar) → ¿se puede experimentar? → si sí, RCT; si no, el método observacional cuyo supuesto sea más defensible → **pruebas de robustez** (sección 5) → si necesitas focalizar, uplift. La sofisticación del modelo nunca sustituye a la claridad del supuesto: **un DiD simple con un supuesto creíble vale más que un Double ML sobre un DAG equivocado.**

---

## 📖 Referencias de este tomo

- (Rubin, 1974) — el modelo de resultados potenciales. · (Rosenbaum & Rubin, 1983) — propensity score.
- (Pearl & Mackenzie, 2018) — la escalera de la causalidad. · (Pearl, 2009) — el tratamiento formal con DAGs.
- (Hernán & Robins, 2020) — *Causal Inference: What If*, el manual moderno abierto.
- (Chernozhukov et al., 2018) — Double/Debiased ML. · (Künzel et al., 2019) — metalearners. · (Radcliffe, 2007) — Qini y evaluación de uplift.
- (Simpson, 1951) — la paradoja de Simpson. · (VanderWeele & Ding, 2017) — E-value y sensibilidad a confounders no medidos.
- (Callaway & Sant'Anna, 2021) · (Goodman-Bacon, 2021) · (Sun & Abraham, 2021) · (de Chaisemartin & D'Haultfœuille, 2020) — DiD con adopción escalonada y el problema de TWFE. · (Roth et al., 2023) — síntesis de la literatura reciente de DiD.
- (Roth, 2022) — pre-trend tests: poca potencia y distorsión por condicionar. · (Rambachan & Roth, 2023) — análisis de sensibilidad a violaciones de tendencias paralelas.
- (Cinelli & Hazlett, 2020) — robustness value y sensibilidad a confounders omitidos. · (Ioannidis, Tan & Blum, 2019) — límites del E-value.
- (Abadie, Diamond & Hainmueller, 2010) — synthetic control. · (Abadie, 2021) — cuándo usar synthetic controls. · (Arkhangelsky et al., 2021) — synthetic difference-in-differences.
- (Wager & Athey, 2018) — causal forests. · (Athey, Tibshirani & Wager, 2019) — generalized random forests. · (Nie & Wager, 2021) — R-learner. · (Yadlowsky et al., 2025) — RATE / AUTOC para evaluar reglas de priorización.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[17-Series-de-Tiempo|17 · Series de Tiempo]] · Siguiente: [[19-NLP-y-LLMs|19 · NLP y LLMs ➡]]

> **Próximo tomo:** [[19-NLP-y-LLMs]] — del TF-IDF al RAG: cómo se procesa texto en la práctica, cuándo basta un modelo clásico y cómo usar LLMs sin que inventen.
