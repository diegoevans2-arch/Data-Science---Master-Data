---
title: "Tomo 21 — Análisis de Supervivencia y Multi-Armed Bandits"
tags: [data-science, machine-learning, supervivencia, bandits, experimentacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 21
version: 6.2
updated: 2026-07-29
---

# ⏳ Tomo 21 — Análisis de Supervivencia y Multi-Armed Bandits

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[20-Sistemas-de-Recomendacion|20 · Sistemas de Recomendación]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Dos complementos que resuelven preguntas que el ML estándar responde mal: **(A)** *¿CUÁNDO ocurrirá el evento?* — el análisis de supervivencia modela el tiempo-hasta-el-evento respetando la **censura** (los casos donde el evento aún no ocurre), algo que un clasificador binario simplemente no sabe hacer; **(B)** *¿cómo decido MIENTRAS aprendo?* — los bandits reparten tráfico entre opciones aprendiendo sobre la marcha, el punto medio entre "decidir a ciegas" y "esperar meses el A/B test".

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** priorizar retención por *urgencia* (no solo por riesgo), planificar mantención por vida útil restante, optimizar campañas/precios/contenidos en tiempo real sin esperar el experimento completo.
> - **Costo de hacerlo mal:** modelos de churn que tiran a la basura la dimensión temporal, censura mal manejada que sesga TODO el análisis, y semanas de tráfico quemadas en variantes perdedoras.
> - **Pregunta ejecutiva que responde:** *¿cuánto tiempo me queda con este cliente/equipo — y cómo pruebo alternativas sin pagar la matrícula completa del experimento?*

---

# Parte A — Análisis de Supervivencia

Audiencia: 🔧 🧭 👔

## A.1 El problema y la censura

Audiencia: 🔧 👔

> [!tip] 💡 Analogía
> Preguntas "¿cuánto duran las ampolletas?" y pruebas 100 durante un año. Al cierre, 60 se quemaron (sabes su duración exacta) y 40 **siguen encendidas**. ¿Las botas del análisis? Perderías justo a las mejores. ¿Les pones "duración = 12 meses"? Mientes: durarán más. Lo único honesto es registrar "duró MÁS de 12 meses, no sé cuánto más" — eso es una observación **censurada**, y el análisis de supervivencia es la estadística construida para no desperdiciarla ni distorsionarla.

**🔧 Definición técnica:** se modela T = tiempo hasta el evento (fuga, falla, default, deserción). Dos funciones centrales, equivalentes (una determina la otra):

- **Supervivencia** `S(t) = P(T > t)`: la fracción que "sigue viva" pasado el tiempo t. Empieza en 1 y decrece.
- **Hazard** `h(t)`: el riesgo **instantáneo** de que el evento ocurra justo en t, dado que se sobrevivió hasta t. Es la forma más rica de ver el fenómeno: revela *cuándo* aprieta el peligro.

> [!tip] 💡 Supervivencia vs. hazard, en una imagen
> `S(t)` es "¿cuánta gente queda en la fiesta a cada hora?"; `h(t)` es "¿con qué ritmo se está yendo la gente en este instante?". La curva de supervivencia baja suave; el hazard puede tener forma de **bañera** (alto al inicio — fallas tempranas / clientes que nunca cuajan; bajo en el medio; alto al final — desgaste), muy común en equipos y en churn.

**Los tipos de censura y truncamiento** — nombrarlos evita el error de tratarlos todos igual:

| Tipo | Qué es | Ejemplo |
|---|---|---|
| **Censura por la derecha** (la común) | El estudio termina antes del evento: se sabe `T > t` | El cliente sigue activo al cierre del análisis |
| Censura por la izquierda | El evento ya había ocurrido antes de empezar a observar | Detectas la deserción pero no sabes cuándo empezó |
| Censura por intervalo | El evento ocurrió entre dos revisiones, sin fecha exacta | Falla detectada en la inspección trimestral |
| Truncamiento | Sujetos que ni siquiera entran en la muestra por una condición | Clientes que se fueron antes de que existiera el registro |

> [!danger] 🚨 El pecado del churn binario
> "¿Se fugará en 30 días: sí/no?" desecha la dimensión temporal (fugarse el día 2 ≠ el día 29), fuerza una ventana arbitraria y maneja mal a los clientes con menos de 30 días de observación. Peor aún: si **borras** los censurados o les imputas el tiempo observado como si fuera el final, sesgas sistemáticamente la supervivencia **hacia abajo** — subestimas la vida del cliente y tomas decisiones sobre una foto pesimista y falsa. Si la pregunta de negocio contiene un "cuándo", el marco correcto es supervivencia; el clasificador binario es la aproximación pobre.

**👔 En una frase para el negocio:** un cliente que sigue activo hoy no es un dato faltante, es información valiosa — botarlo o tratarlo como fuga temprana sesga cualquier número de retención que reportes.

## A.2 Kaplan-Meier y log-rank

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** el estimador de **Kaplan-Meier** (Kaplan & Meier, 1958) construye `S(t)` de forma **no paramétrica** (sin asumir ninguna forma): en cada tiempo de evento multiplica la supervivencia acumulada por la fracción que sobrevive entre los que aún están "en riesgo". Los censurados aportan mientras están observados y salen del denominador después, **sin sesgar**. Produce la clásica **curva escalonada**; la **mediana de supervivencia** es donde `S(t)` cruza 0.5.

```
 S(t) 1.0 ┤▔▔╲_
          │    ╲__          plan Premium
      0.5 ┤       ╲____ ← mediana ≈ 14 meses
          │   ╲_       ╲_____
          │     ╲__ plan Base
      0.0 └──┬────╲̲▁▁▁▁┬──────┬── meses
             6         12     24
```

**Complementos de lectura:**
- **Intervalo de confianza** de la curva (fórmula de Greenwood): la incertidumbre crece hacia la cola, donde quedan pocos en riesgo — no sobre-interpretes el extremo derecho de una curva KM.
- **Test de log-rank** ([[02-Fundamentos-Matematicos]]): compara dos o más curvas (plan A vs B, cohorte 2023 vs 2024) preguntando si la diferencia es real o azar. Es el "¿son distintas estas curvas?" estándar.

**🧭 Cuándo usarlo:** siempre como primer paso descriptivo — la KM por segmento es al análisis de supervivencia lo que el histograma al EDA ([[04-EDA]]). Muestra la forma, revela cruces de curvas (un plan mejor al inicio y peor después), y orienta el modelo posterior.

**👔 En una frase para el negocio:** la curva de "cuántos siguen vivos a cada mes" por segmento — la foto más honesta de la retención, construida sin botar a los clientes que siguen activos.

## A.3 Modelos: de Cox a los paramétricos

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El modelo de Cox asume que todos los clientes envejecen según la misma curva de riesgo base — cada factor (un reclamo, un plan caro) no le cambia la forma a esa curva, solo la multiplica por una constante: el que tuvo un reclamo no envejece distinto, envejece **más rápido**, con el mismo perfil de fondo. Los modelos AFT cambian la metáfora: en vez de multiplicar el riesgo, aceleran o frenan el reloj mismo — un factor no sube el peligro en cada instante, hace correr el tiempo hasta el evento al doble (o a la mitad) de velocidad.

**🔧 El modelo de Cox** (riesgos proporcionales) (Cox, 1972): `h(t│x) = h₀(t) · exp(β₁x₁ + … + βₚxₚ)`. Es **semi-paramétrico**: no asume la forma del riesgo base `h₀(t)`, solo cómo las features lo **multiplican**. Su gran ventaja es la interpretabilidad ejecutiva vía **hazard ratios**:

```
   exp(β) = 1.8  para "tuvo un reclamo"
   → 80% más riesgo instantáneo de fuga, a igualdad de todo lo demás
```

> [!warning] ⚠️ El supuesto que hay que verificar: proporcionalidad
> Cox asume que el efecto de una feature es **constante en el tiempo** (las curvas de hazard de dos grupos se mantienen proporcionales, no se cruzan). Si un plan es más riesgoso al inicio pero más seguro después, el supuesto se rompe y el hazard ratio único **miente**. Se verifica con los **residuos de Schoenfeld**; si falla, se usan efectos dependientes del tiempo o modelos estratificados.

**🔧 Modelos paramétricos y AFT:** cuando se asume una forma para el tiempo (Weibull, exponencial, log-normal), se obtienen modelos **paramétricos** que **extrapolan** más allá del período observado — clave para estimar vida útil o CLV a futuro, algo que la KM no hace. La formulación **AFT (Accelerated Failure Time)** es a menudo más intuitiva que la de hazards: modela cómo una feature **acelera o frena** el reloj del evento ("este factor hace que el desgaste corra al doble de velocidad").

| Enfoque | Asume forma del riesgo | Extrapola | Lectura |
|---|---|---|---|
| Kaplan-Meier | No | No | Descriptivo, por grupos |
| Cox (PH) | No (semi-paramétrico) | Limitada | Hazard ratios por feature |
| Paramétrico / AFT (Weibull…) | Sí | **Sí** | Tiempo esperado; proyección a futuro |

## A.4 ML de supervivencia y sus métricas

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El C-index es la pregunta de una carrera: si tomas dos corredores al azar, ¿el modelo acierta cuál llega primero a la meta (el evento), sin necesidad de saber el tiempo exacto de cada uno? Acertar el orden en todos los pares posibles da C-index = 1; acertar la mitad —lo mismo que tirar una moneda— da 0.5. Es el primo temporal del AUC: no evalúa si la probabilidad predicha es exacta, evalúa si el ranking de urgencia es correcto.

**🔧 Definición técnica:** cuando hay no-linealidades e interacciones, los modelos de árboles se adaptan al marco de supervivencia: **Random Survival Forests** (Ishwaran et al., 2008) y **gradient boosting de supervivencia** (scikit-survival, XGBoost con objetivo AFT) — capturan estructura compleja respetando la censura.

**Métricas específicas** (no sirven las de clasificación tal cual):

| Métrica | Qué mide | Nota |
|---|---|---|
| **C-index** (Harrell et al., 1982) | Probabilidad de ordenar bien **quién experimenta el evento primero** | El primo temporal del AUC ([[08-Metricas-de-Evaluacion]]); 0.5 = azar |
| Time-dependent AUC | Discriminación evaluada a horizontes concretos (a 6, 12 meses) | Cuando importa un horizonte de decisión específico |
| Brier score de supervivencia | Error de las probabilidades predichas en el tiempo | Mide **calibración**, no solo orden ([[11-Mejora-de-Modelos]]) |

**🧭 Cuándo usarlo:** churn/deserción con historia de distinta longitud, tiempo-a-falla en mantención, tiempo-a-default en crédito, tiempo-a-recompra. Aplica todo el rigor de siempre: validación temporal ([[10-Validacion-y-Leakage]]) y features disponibles al momento de predecir.

**👔 En una frase para el negocio:** pasa de "quién está en riesgo" a "quién está en riesgo **y con qué urgencia**" — la diferencia entre una lista y una agenda de intervención.

## A.5 Competing risks: cuando hay más de una salida

Audiencia: 🔧

**🔧 Definición técnica:** el marco básico asume un solo tipo de evento. Pero un cliente puede **fugarse O subir de plan**; un equipo puede **fallar por desgaste O ser retirado por obsolescencia**. Son **riesgos en competencia**: la ocurrencia de uno impide observar el otro. Tratar "upgrade" como censura simple **sesga** — porque el que sube de plan no es un caso "aún en riesgo de fuga", es una salida distinta.

Herramientas: la **función de incidencia acumulada (CIF)** en vez de la KM ingenua, y el modelo de **Fine-Gray** (Fine & Gray, 1999) para modelar el subhazard de cada evento por separado.

> [!tip] 💡 Analogía
> Preguntar "¿cuánto tarda un empleado en renunciar?" ignorando que también puede ser **promovido** o **jubilarse** es contar mal: los promovidos no son "futuros renunciantes que aún no renuncian", salieron por otra puerta. Competing risks es la contabilidad honesta de una sala con **varias puertas de salida**, cada una con su propio ritmo.

## A.6 Aplicaciones y guía rápida

Audiencia: 🧭

| Pregunta de negocio | Herramienta de entrada |
|---|---|
| ¿Cómo se ve la retención por segmento? | Kaplan-Meier + log-rank |
| ¿Qué factores aceleran la fuga, y cuánto? | Cox (hazard ratios) |
| ¿Cuánto vale/dura un cliente a futuro (CLV)? | Paramétrico / AFT (extrapola) |
| Muchas features, no-linealidades | Random Survival Forest / GBM de supervivencia |
| Varias salidas posibles (fuga vs upgrade) | Competing risks (Fine-Gray, CIF) |

---

# Parte B — Multi-Armed Bandits

Audiencia: 🔧 🧭 👔

## B.1 Exploración vs explotación

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Tu heladería favorita tiene 30 sabores. Pedir siempre chocolate (explotar) garantiza un buen helado y quizás te pierde el sabor de tu vida; probar uno nuevo cada vez (explorar) te asegura muchos helados mediocres. El dilema es universal — y "bandit" viene de las máquinas tragamonedas ("one-armed bandits"): ¿en cuál palanca sigo metiendo fichas, con información incompleta de todas?

**🔧 Definición técnica:** K brazos (variantes: titulares, ofertas, precios) con recompensas desconocidas; en cada ronda se elige un brazo y se observa su resultado. El objetivo es minimizar el **regret**: la diferencia acumulada entre lo que se ganó y lo que se habría ganado jugando siempre el mejor brazo (que nadie conoce de antemano).

```
   regret = Σ (recompensa del mejor brazo − recompensa del brazo jugado)
            t
```

El bandit optimiza **mientras** aprende; el A/B test aprende primero y optimiza después. Todo el arte está en explorar lo justo: **explorar de más** quema recompensa en brazos malos; **explotar de más** se queda pegado en un brazo subóptimo por no haberlo puesto a prueba lo suficiente.

## B.2 Los tres algoritmos canónicos

Audiencia: 🔧 🧭

| Algoritmo | Mecanismo | 💡 Analogía | Fortalezas / cuidados |
|---|---|---|---|
| ε-greedy | Con probabilidad 1−ε juega el mejor brazo conocido; con ε explora al azar | El goloso disciplinado: 9 de cada 10 veces chocolate, 1 de cada 10 sorpresa | Simple y decente; explora igual lo prometedor que lo claramente malo. Mejora: **ε decreciente** (explora mucho al inicio, poco al final) |
| UCB (Upper Confidence Bound) | Juega el brazo con mejor `media + bono de incertidumbre` (el bono crece si el brazo se probó poco) (Auer et al., 2002) | Optimismo ante la duda: al poco probado se le concede el beneficio de la duda | Garantías teóricas de regret; determinista; sensible a recompensas no estacionarias |
| Thompson Sampling | Mantiene una distribución (Beta para conversiones — [[02-Fundamentos-Matematicos]]) por brazo; en cada ronda **muestrea** de cada una y juega el mejor sorteo (Thompson, 1933) | Cada brazo compra rifas proporcionales a su credibilidad | Bayesiano, se auto-regula, excelente en la práctica; **el default moderno** |

> [!tip] 💡 Por qué Thompson Sampling se auto-regula tan bien
> La distribución Beta de cada brazo es **ancha** cuando hay pocos datos (así que su sorteo puede salir alto y el brazo se prueba) y **angosta** cuando ya se probó mucho (su sorteo se pega a la media real). La exploración no es una regla externa (como el ε): **emerge sola** de la incertidumbre. A medida que un brazo acumula evidencia, deja de sortear valores optimistas y el sistema converge sin que nadie ajuste un dial.

## B.3 Contextual bandits y el puente al reinforcement learning

Audiencia: 🔧 🧭

**🔧 Definición técnica:** en los bandits básicos el mejor brazo es fijo; en los **contextual bandits** depende del **contexto** (usuario, hora, dispositivo): la oferta óptima para un usuario mobile de noche no es la misma que para uno desktop de mañana. **LinUCB** y variantes (Li et al., 2010) aprenden esa dependencia — y son el **puente directo con la personalización y el cold start** de recomendadores ([[20-Sistemas-de-Recomendacion]]): un usuario nuevo es, literalmente, un bandit cuyos brazos son los ítems.

Un paso más allá —cuando las acciones **afectan el estado futuro** y las recompensas son **diferidas** (no ves el resultado hasta varias jugadas después)— vive el **reinforcement learning** completo ([[01-Introduccion-Ejecutiva]]). El bandit es el caso especial sin estado: cada ronda es independiente.

## B.4 Trampas de producción

Audiencia: 🔧 🧭

Lo que los ejemplos de manual omiten y la operación real cobra:

| Trampa | Qué pasa | Defensa |
|---|---|---|
| **No-estacionariedad** | El mejor brazo cambia (una creatividad se "quema", cambia la temporada) | Ventana deslizante o **descuento** de la evidencia vieja; el brazo debe poder "re-explorarse" |
| **Recompensas diferidas** | La conversión llega días después del clic; el bandit decide con señal incompleta | Modelar la demora; usar proxies tempranos con cuidado |
| **Winner-takes-all prematuro** | Un brazo con suerte inicial acapara el tráfico antes de tiempo | Priors adecuados; garantizar exploración mínima |
| **Feedback loop** | El brazo más servido acumula más datos y se auto-refuerza | El mismo problema de los recomendadores ([[20-Sistemas-de-Recomendacion]]): vigilar exposición |

## B.5 ¿Bandit o A/B test?

Audiencia: 🧭 👔

| Criterio | A/B test ([[02-Fundamentos-Matematicos]]) | Bandit |
|---|---|---|
| Objetivo | **Inferencia limpia**: medir el efecto con rigor | **Optimización**: maximizar resultado durante el aprendizaje |
| Tráfico a variantes perdedoras | Fijo hasta el final (costo de oportunidad) | Se reduce automáticamente |
| Nº de variantes | Pocas (2–4) | Muchas (decenas de titulares/creatividades) |
| Duración / estacionalidad | Ventana fija, controla día-de-semana por diseño | Continuo; cuidar no-estacionariedad (B.4) |
| Cuándo conviene | Decisiones estructurales de una vez (pricing de lista, rediseño) donde el TAMAÑO del efecto importa y alimentará causalidad ([[18-Causalidad-y-Uplift]]) | Optimización perpetua de piezas intercambiables (banners, asuntos de email, orden de ofertas) |

> [!warning] ⚠️ El bandit optimiza, pero no te da un tamaño de efecto limpio
> Como el bandit **cambia la asignación sobre la marcha**, no produce la estimación no sesgada del efecto que sí da un A/B con grupos fijos. Si necesitas responder *"¿cuánto exactamente mejora B sobre A?"* para alimentar una decisión causal ([[18-Causalidad-y-Uplift]]) o un caso de negocio, usa A/B. Si solo necesitas *"sírveme lo que mejor funcione ahora"*, usa bandit. Confundir los objetivos lleva a reportar "efectos" del bandit que no son válidos como inferencia.

**👔 En una frase para el negocio:** el A/B te compra una **verdad medible**; el bandit te compra **resultado mientras aprende** — usa el primero para decidir políticas y el segundo para operar el día a día.

> [!example] 📊 Caso de negocio — E-commerce: los banners que se optimizan solos
> **Problema:** marketing produce 15 creatividades por campaña y las rota "a ojo"; cuando el A/B tradicional declara un ganador, la campaña ya terminó.
>
> **Técnica aplicada:** Thompson Sampling sobre las 15 variantes con conversión como recompensa (prior Beta), segmentado en dos contextos gruesos (mobile/desktop); monitoreo de no-estacionariedad (una creatividad se agota — ventana deslizante); y una regla de gobernanza: las decisiones **estructurales** (¿descuento vs envío gratis?) se validan aparte con A/B formal + análisis causal ([[18-Causalidad-y-Uplift]]).
>
> **Resultado:** el tráfico migra solo hacia las 3 creatividades ganadoras en días (no semanas), el regret medido contra el mejor brazo ex-post es una fracción del esquema de rotación fija, y marketing deja de discutir por gustos: discute por curvas. La lección: **rotar al azar lo optimizable es regalar conversión; testear con rigor lo estructural es comprar certeza** — cada herramienta a lo suyo.

---

## 📖 Referencias de este tomo

- (Kaplan & Meier, 1958) — el estimador de supervivencia. · (Cox, 1972) — riesgos proporcionales.
- (Ishwaran et al., 2008) — Random Survival Forests. · (Harrell et al., 1982) — C-index.
- (Fine & Gray, 1999) — competing risks (subdistribution hazard).
- (Thompson, 1933) — el primer bandit bayesiano. · (Auer et al., 2002) — UCB. · (Li et al., 2010) — bandits contextuales.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[20-Sistemas-de-Recomendacion|20 · Sistemas de Recomendación]]

> 🏁 **Fin de la extensión aplicada (tomos 17–21) y de la Guía Maestra v6.** Vuelve al [[00-MOC-Guia-Maestra|índice maestro]] para navegar por perfil.
