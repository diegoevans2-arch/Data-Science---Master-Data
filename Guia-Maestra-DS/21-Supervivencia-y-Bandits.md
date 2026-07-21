---
title: "Tomo 21 — Análisis de Supervivencia y Multi-Armed Bandits"
tags: [data-science, machine-learning, supervivencia, bandits, experimentacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 21
version: 6.0
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

> [!tip] 💡 Analogía
> Preguntas "¿cuánto duran las ampolletas?" y pruebas 100 durante un año. Al cierre, 60 se quemaron (sabes su duración exacta) y 40 **siguen encendidas**. ¿Las botas del análisis? Perderías justo a las mejores. ¿Les pones "duración = 12 meses"? Mientes: durarán más. Lo único honesto es registrar "duró MÁS de 12 meses, no sé cuánto más" — eso es una observación **censurada**, y el análisis de supervivencia es la estadística construida para no desperdiciarla ni distorsionarla.

**🔧 Definición técnica:** se modela T = tiempo hasta el evento (fuga, falla, default, deserción). **Censura por la derecha:** el estudio termina (o el sujeto sale) antes del evento — se sabe T > t, no T. Ignorar censurados o imputarles el tiempo observado sesga sistemáticamente hacia abajo la supervivencia. Funciones centrales: **S(t) = P(T > t)** (supervivencia) y **hazard h(t)** (riesgo instantáneo de que el evento ocurra en t, dado que se sobrevivió hasta t).

> [!danger] 🚨 El pecado del churn binario
> "¿Se fugará en 30 días: sí/no?" desecha la dimensión temporal (fugarse el día 2 ≠ el día 29), fuerza una ventana arbitraria y maneja mal a los clientes con menos de 30 días de observación. Si la pregunta de negocio contiene un "cuándo", el marco correcto es supervivencia — el clasificador binario es la aproximación pobre.

## A.2 Kaplan-Meier y log-rank

**🔧 Definición técnica:** el estimador de **Kaplan-Meier** (Kaplan & Meier, 1958) construye S(t) de forma no paramétrica: en cada tiempo de evento multiplica por la fracción que sobrevive entre los aún en riesgo — los censurados aportan mientras están y salen del denominador después, sin sesgar. Produce la clásica **curva escalonada**; la mediana de supervivencia es donde S(t) cruza 0.5. Comparación de grupos (plan A vs plan B, cohorte 2023 vs 2024): **test de log-rank** ([[02-Fundamentos-Matematicos]]).

```
 S(t) 1.0 ┤▔▔╲_
          │    ╲__          plan Premium
      0.5 ┤       ╲____ ← mediana ≈ 14 meses
          │   ╲_       ╲_____
          │     ╲__ plan Base
      0.0 └──┬────╲̲▁▁▁▁┬──────┬── meses
             6         12     24
```

**👔 En una frase para el negocio:** la curva de "cuántos siguen vivos a cada mes" por segmento — la foto más honesta de la retención, construida sin botar a los clientes que siguen activos.

## A.3 Modelo de Cox y métodos ML

**🔧 Definición técnica:** el modelo de **riesgos proporcionales de Cox** (Cox, 1972): `h(t│x) = h₀(t) · exp(β₁x₁ + … + βₚxₚ)` — semi-paramétrico: no asume la forma del riesgo base h₀(t), solo cómo las features lo multiplican. Lectura ejecutiva de los **hazard ratios**: `exp(β) = 1.8` para "tuvo reclamo" significa 80% más riesgo instantáneo de fuga, a igualdad de lo demás. Supuesto clave: proporcionalidad (el efecto no cambia con el tiempo — verificar con residuos de Schoenfeld). **ML de supervivencia:** Random Survival Forests (Ishwaran et al., 2008) y gradient boosting de supervivencia (scikit-survival, XGBoost AFT) capturan no-linealidades; **métrica estándar: C-index** (Harrell et al., 1982) — la probabilidad de ordenar bien qué caso experimenta el evento primero (el primo temporal del AUC, [[08-Metricas-de-Evaluacion]]). Librerías: lifelines, scikit-survival.

**🧭 Cuándo usarlo:** churn/deserción con historia de distinta longitud, tiempo-a-falla en mantención, tiempo-a-default en crédito, tiempo-a-recompra. Aplica todo el rigor de siempre: validación temporal ([[10-Validacion-y-Leakage]]) y features disponibles al momento de predecir.

**👔 En una frase para el negocio:** pasa de "quién está en riesgo" a "quién está en riesgo **y con qué urgencia**" — la diferencia entre una lista y una agenda de intervención.

---

# Parte B — Multi-Armed Bandits

Audiencia: 🔧 🧭 👔

## B.1 Exploración vs explotación

> [!tip] 💡 Analogía
> Tu heladería favorita tiene 30 sabores. Pedir siempre chocolate (explotar) garantiza un buen helado y quizás te pierde el sabor de tu vida; probar uno nuevo cada vez (explorar) te asegura muchos helados mediocres. El dilema es universal — y "bandit" viene de las máquinas tragamonedas ("one-armed bandits"): ¿en cuál palanca sigo metiendo fichas, con información incompleta de todas?

**🔧 Definición técnica:** K brazos (variantes: titulares, ofertas, precios) con recompensas desconocidas; en cada ronda se elige un brazo y se observa su resultado. Objetivo: minimizar el **regret** — lo que se dejó de ganar por no haber jugado siempre el mejor brazo (que nadie conoce de antemano). El bandit optimiza **mientras** aprende; el A/B test aprende primero y optimiza después.

## B.2 Los tres algoritmos canónicos

| Algoritmo | Mecanismo | 💡 Analogía | Fortalezas / cuidados |
|---|---|---|---|
| ε-greedy | Con probabilidad 1−ε juega el mejor brazo conocido; con ε explora al azar | El goloso disciplinado: 9 de cada 10 veces chocolate, 1 de cada 10 sorpresa | Simple y decente; explora igual lo prometedor que lo claramente malo |
| UCB (Upper Confidence Bound) | Juega el brazo con mejor `media + bono de incertidumbre` (el bono crece si el brazo se ha probado poco) (Auer et al., 2002) | Optimismo ante la duda: al poco probado se le concede el beneficio de la duda | Garantías teóricas de regret; determinista; sensible a recompensas no estacionarias |
| Thompson Sampling | Mantiene una distribución (Beta para conversiones — [[02-Fundamentos-Matematicos]]) por brazo; en cada ronda **muestrea** de cada una y juega el mejor sorteo (Thompson, 1933) | Cada brazo compra rifas proporcionales a su credibilidad | Bayesiano, se auto-regula, excelente en la práctica; el default moderno |

**🔧 Contextual bandits:** el brazo óptimo depende del contexto (usuario, hora, dispositivo): LinUCB y variantes (Li et al., 2010) — el puente hacia la personalización y el cold start de recomendadores ([[20-Sistemas-de-Recomendacion]]). Y un paso más allá — estados, acciones que afectan el futuro, recompensas diferidas — vive el aprendizaje por refuerzo completo ([[01-Introduccion-Ejecutiva]]).

## B.3 ¿Bandit o A/B test?

| Criterio | A/B test ([[02-Fundamentos-Matematicos]]) | Bandit |
|---|---|---|
| Objetivo | **Inferencia limpia**: medir el efecto con rigor | **Optimización**: maximizar resultado durante el aprendizaje |
| Tráfico a variantes perdedoras | Fijo hasta el final (costo de oportunidad) | Se reduce automáticamente |
| Nº de variantes | Pocas (2–4) | Muchas (decenas de titulares/creatividades) |
| Duración / estacionalidad | Ventana fija, controla día-de-semana por diseño | Continuo; cuidar no-estacionariedad |
| Cuándo conviene | Decisiones estructurales de una vez (pricing de lista, rediseño) donde el TAMAÑO del efecto importa y alimentará causalidad ([[18-Causalidad-y-Uplift]]) | Optimización perpetua de piezas intercambiables (banners, asuntos de email, orden de ofertas) |

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
- (Thompson, 1933) — el primer bandit bayesiano. · (Auer et al., 2002) — UCB. · (Li et al., 2010) — bandits contextuales.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[20-Sistemas-de-Recomendacion|20 · Sistemas de Recomendación]]

> 🏁 **Fin de la extensión aplicada (tomos 17–21) y de la Guía Maestra v6.** Vuelve al [[00-MOC-Guia-Maestra|índice maestro]] para navegar por perfil.
