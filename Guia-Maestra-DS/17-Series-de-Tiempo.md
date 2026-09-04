---
title: "Tomo 17 — Series de Tiempo y Forecasting"
tags: [data-science, machine-learning, time-series, forecasting]
audiencias: [tecnico, puente, ejecutivo]
tomo: 17
version: 6.2
updated: 2026-07-29
---

# 📈 Tomo 17 — Series de Tiempo y Forecasting

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[16-Bibliografia|16 · Bibliografía]] · Siguiente: [[18-Causalidad-y-Uplift|18 · Causalidad y Uplift ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> El forecasting es la pregunta de negocio más antigua del mundo — *¿cuánto venderé?* — y una disciplina con reglas propias: aquí el orden temporal ES la estructura, el vecino de una observación es su ayer, y romper esa cadena (barajando datos, filtrando el futuro) destruye todo. Este tomo complementa las piezas ya vistas — lags y rolling ([[03-Preparacion-de-Datos]]), Time Series Split ([[10-Validacion-y-Leakage]]), MAPE/SMAPE ([[08-Metricas-de-Evaluacion]]) — con el arsenal específico: descomposición, estacionariedad, la familia ARIMA, ETS, Prophet, el enfoque ML que domina las competencias modernas, y las prácticas que separan un pronóstico accionable de un número decorativo.

> [!abstract] 👔 Impacto ejecutivo
> Cada punto de error de pronóstico es inventario inmovilizado o venta perdida; cada mejora, capital liberado.
>
> - **Decisiones que habilita:** compras e inventario, dotación de personal, presupuestos y metas, planificación de capacidad, gestión de tesorería.
> - **Costo de hacerlo mal:** quiebres de stock y sobre-stock simultáneos, pronósticos "validados" con trampa temporal que fallan el primer mes, y metas construidas sobre tendencias mal extrapoladas.
> - **Pregunta ejecutiva que responde:** *¿cuánto y cuándo — y con qué margen de error — puedo esperar de esta variable en los próximos períodos?*

> [!tip] 💡 Analogía general: la marea, las olas y el clima
> Una serie de tiempo es el nivel del mar visto desde la playa: la **tendencia** es el cambio climático (sube lento y sostenido), la **estacionalidad** es la marea (sube y baja con horario conocido), el **ciclo** son las tormentas (recurrentes pero sin calendario fijo) y el **ruido** son las olas individuales (impredecibles una a una). Pronosticar es separar esas cuatro capas: la marea de mañana se predice casi perfecto, la ola individual jamás. Y el arte está en no confundirlas: quien lee una racha de olas grandes como "el mar está subiendo" toma una decisión sobre ruido.

> [!example] 📊 Caso de negocio — Retail: el pronóstico que liberó capital
> **Problema:** una cadena pronostica demanda por SKU con "el promedio de los últimos 3 meses". Resultado crónico: quiebres en productos estacionales y bodegas llenas de lo que dejó de rotar.
>
> **Técnica aplicada:** (1) baseline seasonal naive por SKU (la vara honesta); (2) jerarquía de modelos: Holt-Winters para series estables, LightGBM global con features de calendario, lags y rolling para el grueso del catálogo ([[03-Preparacion-de-Datos]], [[07-Modelos-Supervisados]]); (3) validación walk-forward con el horizonte real de compra (4 semanas) y métrica MASE para comparar SKUs de volúmenes distintos; (4) pronóstico de cuantiles (P50 y P90 — Quantile Loss, [[08-Metricas-de-Evaluacion]]) para decidir stock de seguridad por costo de quiebre.
>
> **Resultado:** el error agregado cae de forma sostenida frente al baseline, pero el valor real está en los cuantiles: comprar al P90 los SKUs de alto costo de quiebre y al P50 el resto reduce quiebres **y** capital inmovilizado a la vez. La lección: en forecasting, el intervalo vale más que el punto.

---

## 1. Qué hace único al forecasting

Audiencia: 🔧 🧭

Antes de las técnicas, la mentalidad. El forecasting **no es una regresión más** ([[07-Modelos-Supervisados]]): rompe tres supuestos que el resto del ML da por sentados, y cada ruptura tiene su consecuencia.

| Supuesto que el ML clásico asume | Qué pasa en series de tiempo | Consecuencia práctica |
|---|---|---|
| Las observaciones son **independientes** | Cada punto depende de su pasado (autocorrelación) | No puedes barajar; el orden ES la información |
| El train y el test vienen de la **misma distribución** | El futuro puede no parecerse al pasado (tendencia, drift) | Los modelos extrapolan a terreno no visto — riesgo estructural |
| Puedes **reordenar** los datos libremente | El tiempo tiene una sola dirección | Validar con K-Fold aleatorio es entrenar con el diario de mañana ([[10-Validacion-y-Leakage]]) |

> [!tip] 💡 Analogía
> Predecir con datos independientes es como adivinar el color de la próxima bola de una urna: cada extracción es un mundo nuevo. Predecir una serie de tiempo es como continuar una melodía: la próxima nota está atada a las que ya sonaron, y no puedes tocarlas en cualquier orden sin destruir la canción.

**🧭 Las tres preguntas que definen tu problema de forecasting** — respóndelas antes de elegir modelo:

1. **¿Horizonte?** ¿Predices el próximo paso (one-step) o los próximos *h* pasos (multi-step)? El horizonte de validación debe ser el horizonte **real de la decisión**, no uno cómodo.
2. **¿Cuántas series?** ¿Una serie larga, o miles de series relacionadas (un SKU por tienda)? Esto decide entre modelo local y modelo global (sección 6).
3. **¿Punto o distribución?** ¿Necesitas un número, o un intervalo de confianza? La decisión de inventario, capacidad o tesorería casi siempre vive en un cuantil, no en la media (sección 9).

**👔 En una frase para el negocio:** la primera pregunta de un proyecto de forecasting no es *"¿qué modelo?"* sino *"¿qué decisión alimenta este pronóstico, y con qué horizonte y qué tolerancia al error?"* — el modelo se elige después.

---

## 2. Anatomía de una serie: componentes y descomposición

Audiencia: 🔧 🧭

**🔧 Definición técnica:** toda serie se modela como combinación de **tendencia** (T), **estacionalidad** (S), **ciclo** y **residuo** (R). Dos composiciones posibles:

```
  ADITIVA        y = T + S + R      amplitud estacional CONSTANTE
  MULTIPLICATIVA y = T × S × R      la estacionalidad CRECE con el nivel
```

> [!tip] 💡 Cómo elegir entre aditiva y multiplicativa
> Mira los "dientes" estacionales de la serie: si los picos de diciembre miden **lo mismo** año a año → aditiva. Si cada diciembre es más alto porque el negocio creció → **multiplicativa** (típico en ventas). Truco práctico: la multiplicativa sobre `y` equivale a la aditiva sobre `log(y)` ([[05-Escalado-de-Datos]]) — por eso transformar con logaritmo "endereza" la estacionalidad creciente y suele ser el primer gesto del analista.

**Herramienta estándar: STL** (Seasonal-Trend decomposition using Loess) (Cleveland et al., 1990): robusta a outliers, permite estacionalidad que **evoluciona** en el tiempo, y está en statsmodels. Supera a la descomposición clásica (medias móviles), que asume estacionalidad fija y sufre en los extremos de la serie.

```
 Serie observada      =    Tendencia       +   Estacionalidad   +    Residuo
 ▂▄▆▄▂▄▆█▆▄▆█▇▅▇█          ▁▂▂▃▃▄▄▅▅▆▆▇▇        ▂▆▂▆▂▆▂▆▂▆▂▆        ▃▄▃▅▃▄▄▃▄▃
 (ventas mensuales)        (crecimiento)        (diciembre alto)     (lo no explicado)
```

**🧭 Cuándo usarlo:** SIEMPRE como primer paso del EDA temporal ([[04-EDA]]). La descomposición te dice qué modelo corresponde (¿hay estacionalidad que modelar?, ¿la tendencia es estable?), y el residuo revela outliers y **quiebres estructurales** (un cambio de nivel abrupto: una pandemia, un cambio de precio, una fusión). Un quiebre estructural no visto envenena cualquier modelo posterior.

**👔 En una frase para el negocio:** separa cuánto de tu crecimiento es real (tendencia) y cuánto es diciembre (estacionalidad) — la confusión entre ambos infla metas y bonos.

**🔧 Fuerza de tendencia y estacionalidad.** Con los tres componentes de la STL (T, S, R) se puede resumir, en una escala 0–1, cuánto de la variabilidad de la serie explica cada componente frente al residuo:

```
 F_T = max(0, 1 − Var(R) / Var(T + R))      fuerza de la tendencia
 F_S = max(0, 1 − Var(R) / Var(S + R))      fuerza de la estacionalidad
```

`F_T` cercano a 1 = la tendencia domina y el residuo aporta poco frente a ella; `F_S` cercano a 1 = el patrón estacional es lo que más explica la serie una vez descontada la tendencia. Estas medidas se propusieron originalmente para **clasificar y agrupar catálogos grandes de series por sus características** (Wang, Smith & Hyndman, 2006), y hoy son parte del instrumental estándar de análisis de series a escala (Hyndman & Athanasopoulos, 2021).

**🧭 Cuándo usarlo:** no tanto para leer una sola serie —para eso ya sirve la descomposición a ojo de arriba— sino para **triage de catálogos grandes** (miles de SKUs) antes de asignar modelo: series con `F_S` alto son candidatas naturales a Holt-Winters/SARIMA; series con `F_T` alto y `F_S` bajo casi no justifican un componente estacional; y el perfil agregado del catálogo ayuda a decidir entre segmentar por tipo de serie o entrenar un modelo global (sección 6 y sección 11).

> [!warning] ⚠️ El umbral "F > 0.6 = estacionalidad fuerte" es heurística, no un estándar publicado
> Es un criterio orientativo que circula en la práctica de clustering y triage de series, útil para ordenar un catálogo de mayor a menor estacionalidad — no un corte validado formalmente en la literatura. Trátalo como punto de partida, no como regla rígida de decisión.

---

## 3. Estacionariedad

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un río **estacionario** fluctúa siempre alrededor del mismo nivel: puedes aprender su comportamiento observándolo un mes. Un río en crecida constante nunca repite su nivel: lo que aprendiste ayer no describe mañana. Los modelos clásicos necesitan el río estable — y si no lo es, se modela el **cambio** del nivel (diferenciación), que sí suele ser estable.

**🔧 Definición técnica:** una serie es estacionaria si su media, varianza y autocorrelación **no dependen del tiempo**. Importa porque los modelos de la familia ARIMA aprenden una estructura de correlación fija; si esa estructura se mueve con el tiempo, lo aprendido en el tramo A no sirve en el tramo B.

**Los dos tests, y por qué se usan juntos:**

| Test | H₀ (hipótesis nula) | Lectura |
|---|---|---|
| **ADF** (Augmented Dickey-Fuller) | La serie NO es estacionaria | `p < 0.05` → rechazas H₀ → **sí** es estacionaria |
| **KPSS** | La serie SÍ es estacionaria | `p < 0.05` → rechazas H₀ → **no** es estacionaria |

> [!warning] ⚠️ Úsalos como pareja, no por separado
> Tienen hipótesis **invertidas**, y eso es una virtud: si ADF dice "estacionaria" **y** KPSS dice "estacionaria", tienes confianza real. Si se contradicen, la serie está en una zona gris (a menudo estacionaria en tendencia pero no en media) y necesitas más trabajo. Un solo test puede engañarte ([[02-Fundamentos-Matematicos]]).

**Remedios:** **diferenciación** (`y'ₜ = yₜ − yₜ₋₁`, el parámetro `d` de ARIMA; diferenciación estacional `yₜ − yₜ₋ₘ` para quitar el patrón anual), y transformación log/Box-Cox para estabilizar la varianza ([[05-Escalado-de-Datos]]).

**🔧 Retornos logarítmicos: la diferenciación estándar de precios.** Para series multiplicativas —precios de activos, tipos de cambio, cualquier variable que crece por tasas más que por montos fijos— la transformación de referencia no es la diferencia simple sino el **retorno logarítmico**: `r_t = ln(P_t / P_t₋₁)`. Es a la vez una diferenciación (de `ln(P)`) y una aproximación al retorno porcentual (para cambios pequeños, `r_t ≈ (P_t − P_t₋₁) / P_t₋₁`). Se modela `r_t` — normalmente mucho más cercano a estacionario que el nivel `P_t` — y el nivel se reconstruye acumulando: `P_t = P_0 · exp(Σ r_i)`. Este tomo es mayormente retail-céntrico (demanda, inventario); esta transformación es la puerta de entrada a series financieras, donde raramente se modela el precio en niveles (Hyndman & Athanasopoulos, 2021).

> [!danger] 🚨 Sobre-diferenciar es tan malo como no diferenciar
> Diferenciar de más **inyecta autocorrelación artificial** y agranda la varianza del pronóstico. Regla práctica: casi ninguna serie de negocio necesita `d > 2`. Si la serie tiene tendencia clara, empieza con `d=1` y verifica con los tests; no diferencies "por si acaso".

> [!danger] 🚨 Regresión espuria entre series con tendencia
> Dos series que solo comparten tendencia —sin que una cause la otra y sin que exista un confounder— producen una regresión con **r y R² altísimos** si se corren en niveles sin diferenciar (el ejemplo clásico de la literatura: consumo de manteca de maní contra tasas de divorcio). Es un mecanismo distinto de la correlación espuria por confounder ya vista en [[02-Fundamentos-Matematicos]]: aquí no hace falta una tercera variable, basta con que ninguna de las dos series sea estacionaria (Granger & Newbold, 1974). La señal de alerta es un R² alto acompañado de un estadístico **Durbin-Watson bajo** (residuos fuertemente autocorrelacionados) — si ves esa combinación, sospecha regresión espuria antes de creer el resultado. Remedio: diferenciar ambas series (o trabajar en retornos) antes de regresionar, o comprobar si existe **cointegración**.
>
> **Cointegración, en dos líneas:** dos series no estacionarias están cointegradas si existe una combinación lineal de ambas que sí es estacionaria — evidencia de una relación de equilibrio de largo plazo genuina, aunque cada serie por separado deambule sin límite. El procedimiento de referencia para probarlo es el de dos pasos de Engle y Granger: regresionar en niveles y testear la estacionariedad de los residuos de esa regresión (Engle & Granger, 1987).

**👔 En una frase para el negocio:** los patrones solo son aprendibles si el terreno de juego es estable — esta verificación evita proyectar como "patrón" lo que era una escalada sin freno.

---

## 4. ACF y PACF: la memoria de la serie

Audiencia: 🔧

> [!tip] 💡 Analogía
> El eco de un cañón en un valle: la **ACF** mide cuánto se sigue escuchando el disparo original a 1, 2, 3… segundos (correlación total con cada rezago, incluyendo ecos de ecos). La **PACF** aísla el eco **directo** de cada distancia, descontando los intermedios. Juntas dibujan el mapa de memoria de la serie.

**🔧 Definición técnica:** ACF = correlación de la serie consigo misma a rezago k; PACF = esa correlación **controlando** por los rezagos intermedios. La lectura clásica para elegir órdenes de ARIMA:

| Patrón observado | Sugiere | Orden |
|---|---|---|
| PACF **corta** en el rezago p, ACF decae gradual | Componente AutoRegresivo | AR(p) |
| ACF **corta** en el rezago q, PACF decae gradual | Componente de Media Móvil | MA(q) |
| Ambas decaen gradualmente | Mezcla | ARMA(p,q) |
| ACF decae **muy lento** (casi lineal) | Falta diferenciar | subir `d` |
| Picos en k = 12, 24, 36… (datos mensuales) | Estacionalidad | componente estacional (SARIMA) |

**🧭 Cuándo usarlo:** diagnóstico previo a ARIMA y **auditoría de residuos**. La prueba de que un modelo capturó toda la estructura: sus residuos NO deben mostrar autocorrelación (test de **Ljung-Box**, `p > 0.05` = residuos parecen ruido blanco = bien). Si los residuos aún tienen memoria, el modelo dejó señal sobre la mesa (la sección 5 amplía este diagnóstico).

**🔧 Cómo descubrir el período estacional cuando no se conoce.** No toda serie trae el período escrito de antemano (un log de sensores IoT, eventos de una app). Dos formas de encontrarlo:

- **ACF:** picos que se repiten a rezagos regulares (k, 2k, 3k…) delatan un período de largo k — la misma tabla de arriba, leída para ubicar el ciclo en vez de el orden del modelo.
- **Periodograma / densidad espectral:** descompone la serie en frecuencias y muestra cuánta varianza aporta cada una (Shumway & Stoffer, 2017). Un pico dominante en la frecuencia `f` implica un período `T = 1/f` (en número de observaciones); picos secundarios en múltiplos de `f` son **armónicos** — sub-ciclos del mismo patrón, no períodos independientes. Potencia concentrada en frecuencias **muy bajas** (cerca de cero) suele ser tendencia disfrazada de ciclo, no estacionalidad real: la firma típica de este caso es un "ciclo dominante" cuyo largo coincide con el largo total de la muestra, que no es un período sino la tendencia que el periodograma no logra separar.

Conecta con la sección 7: cada pico espectral genuino es, potencialmente, una estacionalidad adicional que un modelo de una sola estación (Holt-Winters, SARIMA) no captura por sí solo.

> [!note] En la práctica: `auto_arima`
> Leer ACF/PACF a ojo es un arte que conviene entender, pero en producción se usa búsqueda automática de órdenes (`auto_arima` de pmdarima, o el AutoARIMA de las librerías modernas) que optimiza AIC/BIC ([[08-Metricas-de-Evaluacion]]). El diagnóstico manual sigue valiendo para **entender** lo que la búsqueda automática eligió y para auditar residuos.

---

## 5. Modelos clásicos de forecasting

Audiencia: 🔧 🧭

| Modelo | Mecanismo | Cuándo conviene | Limitaciones | 💡 Analogía |
|---|---|---|---|---|
| Naive / Seasonal Naive | Mañana = hoy; diciembre = diciembre pasado | **SIEMPRE como baseline** — es la vara del MASE | No aprende nada más | El pronóstico del porfiado: "igual que la última vez" |
| Medias móviles / SES (suavizamiento exponencial simple) | Promedio ponderado que olvida el pasado exponencialmente (α) | Series sin tendencia ni estacionalidad | No extrapola tendencia | La memoria que se desvanece: lo reciente pesa más |
| Holt / Holt-Winters (ETS) | Agrega componente de tendencia (Holt) y de estacionalidad (Winters): nivel + tendencia + estación, aditivo o multiplicativo | Series con tendencia y estacionalidad estables; rápido y robusto | Una sola estacionalidad; sin variables externas | Tres diales que se auto-ajustan: nivel, pendiente y calendario |
| ARIMA(p,d,q) | AutoRegresivo (p rezagos de y) + Integrado (d diferenciaciones) + Media móvil (q rezagos del error) (Box & Jenkins, 1970) | Series estacionarizables con estructura de autocorrelación clara | Exige diagnóstico (ACF/PACF, Ljung-Box); univariado en su forma base | Predecir el próximo paso mirando los pasos y los tropiezos recientes |
| SARIMA (+SARIMAX) | ARIMA + componente estacional (P,D,Q)ₘ; la X agrega variables exógenas | Estacionalidad fuerte + necesidad de regresores externos (precio, feriados) | Muchos órdenes que calibrar (auto_arima ayuda) | ARIMA con calendario incorporado |
| Prophet | Modelo aditivo: tendencia por tramos (changepoints) + estacionalidades múltiples (Fourier) + feriados (Taylor & Letham, 2018) | Series de negocio diarias con feriados, cambios de tendencia y analistas no especialistas | Caja relativamente rígida; puede perder contra ETS/ARIMA bien tuneados | El plan de vuelo por capas: ruta base + temporadas + días especiales |

> [!note] ETS y ARIMA no son rivales: son dos idiomas para lo mismo
> **ETS** (Error-Trend-Seasonal) describe la serie por sus **componentes** (¿la tendencia es aditiva o amortiguada?, ¿la estacionalidad crece con el nivel?). **ARIMA** la describe por su **estructura de autocorrelación**. Muchas series se modelan bien con cualquiera de los dos; hay familias de ETS que tienen un ARIMA equivalente exacto. Regla práctica: **ETS si el pensamiento es "componentes", ARIMA si es "memoria y diferencias"**, y deja que la validación (sección 10) decida el ganador.

> [!note] El naive no es solo una vara de comparación: a veces es el modelo generador correcto
> Si la serie sigue un **random walk** (`yₜ = yₜ₋₁ + εₜ`, con `εₜ` ruido blanco), el naive no es un baseline modesto — es el pronóstico puntual **óptimo**: no hay información en el pasado que reduzca el error esperado del próximo paso, porque no queda estructura que explotar más allá del último valor observado. Si tus tests de estacionariedad y tu ACF/PACF (secciones 3 y 4) apuntan a que tu serie es, en esencia, un random walk, la meta realista deja de ser "vencerle al naive en el punto" —imposible por construcción— y pasa a ser modelar su **distribución y volatilidad** (sección 9). Nota de notación, para quien llegue desde material con otra convención: de aquí en más este tomo usa `φ` (phi) para los coeficientes autorregresivos y `θ` (theta) para los de media móvil.

**🔧 Diagnóstico de residuos: el ciclo completo de Box-Jenkins.** Ajustar un ARIMA (o cualquier modelo clásico) no termina en el fit. El método clásico (Box & Jenkins, 1970) es un **ciclo** — identificar → estimar → diagnosticar → iterar —, y un diagnóstico que falla te devuelve a identificar; no es un paso final que se marca y se olvida.

| Test | H₀ (hipótesis nula) | Qué revela si se rechaza | Consecuencia |
|---|---|---|---|
| **Ljung-Box** (autocorrelación) | Los residuos son ruido blanco | Queda señal sin capturar en la mesa | Agregar términos AR/MA o estacionales y volver a identificar (Ljung & Box, 1978) |
| **Jarque-Bera / Shapiro-Wilk** (normalidad) | Los residuos son normales | Colas más pesadas o asimetría de la asumida | NO invalida el punto pronosticado, pero sí los **intervalos paramétricos** que asumen normalidad → usar bootstrap o conformal prediction (sección 9) |
| **ARCH test** (heterocedasticidad condicional) | La varianza del residuo es constante en el tiempo | Volatilidad agrupada (tramos tranquilos y turbulentos que se alternan) | Los intervalos de ancho constante mienten; el modelo de referencia para esa varianza es GARCH (Engle, 1982) |

**Checklist visual de una línea** (los cuatro gráficos que acompañan la tabla): residuos vs. tiempo (¿hay patrones o rachas?), histograma de residuos (¿parece campana?), correlograma de residuos —la ACF de la sección 4 aplicada al error— (¿hay barras fuera de la banda de confianza?), y gráfico Q-Q (¿los puntos siguen la diagonal en las colas?).

> [!tip] 🧭 Árbol de decisión rápido: ¿por dónde empiezo?
> ```
>   ¿La serie tiene tendencia y/o estacionalidad claras?
>    ├─ NO  ──────────────► SES / media móvil (y quizás no necesitas más)
>    └─ SÍ
>        ¿Necesitas variables externas (precio, promo, clima)?
>         ├─ NO
>         │   ¿Una sola estacionalidad, serie estable?
>         │    ├─ SÍ ─────► Holt-Winters (ETS)  ó  SARIMA
>         │    └─ estacionalidad múltiple / feriados ─► Prophet ó MSTL
>         └─ SÍ ──────────► SARIMAX  ó  ML con features (sección 6)
>
>   ¿Muchas series relacionadas (miles de SKUs)? ──► modelo GLOBAL (sección 6)
>   En TODOS los casos: primero el baseline naive.
> ```

---

## 6. Machine Learning y modelos globales

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** transformar el problema temporal en tabular supervisado y aplicar gradient boosting ([[07-Modelos-Supervisados]]). El feature engineering ES el modelo aquí — más que la elección del algoritmo:

| Familia de feature | Ejemplos | Qué captura |
|---|---|---|
| Calendario | mes, día de semana, feriados, semana del año, cíclicas sin/cos ([[05-Escalado-de-Datos]]) | Estacionalidad y efectos de calendario |
| Lags | `y(t-1)`, `y(t-7)`, `y(t-365)` | Autocorrelación y memoria |
| Rolling statistics | media/mediana/std/max móvil de las últimas k ventanas ([[03-Preparacion-de-Datos]]) | Nivel y volatilidad recientes |
| Exógenas | precio, promoción, clima, eventos | Drivers externos de la demanda |
| Expanding / EWM | media acumulada, media exponencial | Tendencia de largo plazo |

> [!danger] 🚨 Los árboles NO extrapolan tendencia
> Un gradient boosting nunca predice un valor **fuera del rango que vio en train** ([[07-Modelos-Supervisados]]). En una serie con tendencia creciente sostenida, un modelo de árboles se "aplana" en el techo histórico y subestima el futuro sistemáticamente. Remedios: **destendenciar** (modelar el residuo de una descomposición), **diferenciar** (predecir el cambio `y(t)−y(t-1)` en vez del nivel), o combinar con un modelo lineal que sí extrapole. Ignorar esto es el error #1 del forecasting con ML.

**🔧 Estrategias para pronosticar múltiples pasos (multi-step).** Cuando el horizonte es `h > 1`, hay que decidir cómo se llega ahí — la elección no es cosmética, cambia el error y el costo de mantenimiento:

| Estrategia | Cómo funciona | Ventaja | Costo |
|---|---|---|---|
| **Recursiva** | Un solo modelo one-step; su propia predicción se reinyecta como input del paso siguiente | Un modelo, coherente paso a paso | El error se **acumula**: cada paso hereda el error del anterior |
| **Directa** | Un modelo distinto entrenado específicamente para cada horizonte (uno para t+1, otro para t+2…) | Sin acumulación de error entre pasos | `h` modelos que entrenar y mantener; sin garantía de coherencia entre pasos consecutivos |
| **Multi-output** | Un solo modelo que predice el vector completo `(t+1, …, t+h)` de una vez | Un modelo, sin acumulación de error | Exige un algoritmo que soporte salida vectorial; menos flexible paso a paso |

(Ben Taieb, Bontempi, Atiya & Sorjamaa, 2012) compara estas estrategias en competencias de forecasting multi-step y no encuentra una ganadora universal: la elección depende de cuánto ruido tiene el proceso generador y de cuán largo es el horizonte.

**El enfoque global — la idea que cambió el campo:** en lugar de entrenar un modelo por serie (**local**), se entrena **un solo modelo para todas las series a la vez**, con el ID de la serie como feature. Domina cuando hay muchas series relacionadas.

| | Modelo local (uno por serie) | Modelo global (uno para todas) |
|---|---|---|
| Nº de modelos con 10.000 SKUs | 10.000 | **1** |
| Series cortas o nuevas | Falla (poca historia) | **Aprende de las series hermanas** (cross-learning) |
| Mantenimiento | Pesadilla operativa | Un pipeline |
| Cuándo gana | Pocas series largas y muy distintas entre sí | Muchas series relacionadas |

La competencia **M5** (Walmart, ventas por SKU-tienda) la ganaron variantes de **LightGBM global** (Makridakis et al., 2022) — no deep learning, no ARIMA: boosting con buen feature engineering.

**Deep learning especializado** ([[12-Deep-Learning]]) paga cuando hay series masivas y patrones complejos compartidos:

| Modelo | Idea | Cuándo conviene |
|---|---|---|
| DeepAR | RNN autoregresivo probabilístico (Amazon) | Muchas series, se quiere distribución completa |
| N-BEATS / N-HiTS | Bloques de proyección backward/forward, sin componentes hechos a mano | Series abundantes, alto rendimiento sin feature engineering |
| Temporal Fusion Transformer (TFT) | Attention + variables estáticas/dinámicas/exógenas, con interpretabilidad | Problema rico en covariables, se valora explicabilidad |
| PatchTST / TimesFM y foundation models | Transformers sobre parches temporales; modelos preentrenados | Frontera actual; útiles con series largas o zero-shot |

**🧭 Cuándo usar ML/DL vs. clásico:** ML global para catálogos grandes, variables externas relevantes, no-linealidades. Clásico (ETS/ARIMA) para **pocas series largas y estables** — rinden igual con una fracción del esfuerzo y son más transparentes.

**👔 En una frase para el negocio:** con un catálogo grande, un solo modelo global bien alimentado de calendario y promociones suele vencer a mil modelitos artesanales — y es infinitamente más mantenible.

---

## 7. Estacionalidad múltiple, exógenas y forecasting jerárquico

Audiencia: 🔧 🧭

Tres complicaciones del mundo real que el modelo de manual no cubre.

**🔧 Estacionalidad múltiple.** Una serie horaria de demanda eléctrica tiene estacionalidad **diaria** (día/noche), **semanal** (finde vs. laboral) y **anual** (verano/invierno) — todas a la vez. Los modelos de una sola estación (Holt-Winters, SARIMA) no bastan. Opciones: **MSTL** (descomposición con múltiples estacionalidades), **Prophet** (suma términos de Fourier por estación), **TBATS**, o ML con features de calendario que codifican cada ciclo por separado.

**🔧 Variables exógenas.** No todo se explica con el pasado de la serie: precio, promociones, feriados, clima y eventos mueven la demanda. Se incorporan vía **SARIMAX** (la X), Prophet (regresores) o, con más flexibilidad, ML (una columna por variable).

> [!warning] ⚠️ La trampa de las exógenas: también hay que pronosticarlas
> Si tu modelo usa "temperatura" para predecir demanda, en producción necesitas la **temperatura futura** — que es a su vez un pronóstico con su propio error. Usar el valor real futuro de una exógena en validación es **leakage** ([[10-Validacion-y-Leakage]]): infla la métrica y no se podrá replicar. Distingue exógenas **conocidas de antemano** (feriados, precio planificado) de las que **hay que estimar** (clima, tráfico).

**🔧 Causalidad de Granger: screening de candidatas a exógena.** Antes de meter una variable candidata como exógena, una pregunta útil de filtro: ¿los rezagos de X mejoran la predicción de Y más allá de lo que ya aportan los propios rezagos de Y? Eso es lo que testea la causalidad de Granger (Granger, 1969) — no es un test binario de causa-efecto, es una pregunta de **poder predictivo incremental**. Tres advertencias antes de usarlo como filtro:

1. **"Granger-causa" no es causalidad real.** El nombre es engañoso: es precedencia predictiva, no el mecanismo causal que exploran las herramientas de [[18-Causalidad-y-Uplift]] — dos series pueden Granger-causarse mutuamente por un tercer factor común que mueve a ambas con rezagos distintos.
2. **Exige estacionariedad** (sección 3): aplicarlo sobre series con tendencia sin diferenciar hereda el mismo riesgo de regresión espuria ya visto arriba.
3. Que X Granger-cause a Y a través de sus **rezagos** no autoriza, por sí solo, a usar el valor **contemporáneo** de X como exógena — ese es un salto adicional. Esto último es un criterio de rigor razonable, no una regla codificada en la literatura del método: trátalo como tal, no como doctrina cerrada.

**🔧 Regresión armónica dinámica.** Otra forma de atacar estacionalidad múltiple o de período muy largo: en vez de forzar un componente estacional explícito (un SARIMA con `m=365` es, en la práctica, inviable de estimar), se ajusta un ARIMA sobre los residuos de **K pares de términos seno/coseno** por período, usados como variables exógenas — la misma idea que usa Prophet internamente para representar sus estacionalidades (Taylor & Letham, 2018). `K` controla la suavidad de la forma estacional (K bajo = curva suave, K alto = riesgo de sobreajuste) y se elige por AICc o por validación, igual que cualquier otro hiperparámetro ([[08-Metricas-de-Evaluacion]]). Ventaja doble: maneja períodos largos o múltiples donde SARIMA es inviable, y —a diferencia de la trampa de exógenas que acabamos de ver (el clima)— los regresores armónicos futuros se calculan **exactos**, porque el seno y el coseno de una fecha futura no tienen incertidumbre. El límite: la forma estacional queda fija una vez elegido K; si la estacionalidad cambia de forma en el tiempo, MSTL o Prophet (que sí la dejan evolucionar) pueden ajustar mejor (Hyndman & Athanasopoulos, 2021).

**🔧 Forecasting jerárquico.** En retail las series forman un árbol: SKU → categoría → tienda → región → total. El negocio necesita que los niveles **cuadren**: la suma de los pronósticos por SKU debe igualar el pronóstico de la categoría.

```
                    TOTAL PAÍS
                   /          \
             REGIÓN A       REGIÓN B
             /     \         /     \
         Tienda  Tienda  Tienda  Tienda      ← la suma de las hojas
           |       |        |       |           debe reconciliar con la raíz
         SKUs    SKUs     SKUs    SKUs
```

| Estrategia de reconciliación | Cómo | Trade-off |
|---|---|---|
| **Bottom-up** | Pronosticar las hojas, sumar hacia arriba | Preserva el detalle; el ruido de las hojas se acumula |
| **Top-down** | Pronosticar el total, repartir por proporciones históricas | Total estable; pierde señal específica de cada hoja |
| **Middle-out** | Pronosticar un nivel intermedio y propagar en ambos sentidos | Compromiso |
| **MinT (óptima)** | Reconciliación estadística que minimiza la varianza (Wickramasuriya et al., 2019) | Estado del arte; más complejo de implementar |

**👔 En una frase para el negocio:** si tus pronósticos por tienda no suman el pronóstico nacional, finanzas y operaciones están planificando con dos números distintos — la reconciliación es lo que hace que la organización hable de una sola cifra.

---

## 8. Casos especiales: demanda intermitente, outliers y cold start

Audiencia: 🔧 🧭

Situaciones donde el instrumental estándar falla y hay que cambiar de herramienta.

**🔧 Demanda intermitente** (muchos ceros: repuestos, artículos de baja rotación). El MAPE explota (divide por cero), ARIMA se pierde entre ceros, y la media es engañosa (predecir "0,3 unidades" cuando la venta es 0 ó 1). Herramienta correcta: **Croston** y sus variantes (SBA, TSB), que separan *cuánto se vende cuando se vende* de *cada cuánto se vende*. Métrica: no MAPE; usar MASE o errores absolutos agregados.

> [!tip] 💡 Analogía
> Pronosticar demanda intermitente con un modelo normal es como describir a alguien que va al gimnasio "en promedio 0,4 veces por día". El promedio es correcto y completamente inútil. Croston responde las dos preguntas que importan por separado: cuando va, ¿cuánto entrena?, y ¿cada cuántos días va?

**🔧 Outliers y quiebres estructurales.** Un pico único (un día de viralización, un error de carga) no es señal: si entra crudo, contamina lags y rolling. Detección con la descomposición (residuo anómalo, sección 2) o rangos robustos ([[03-Preparacion-de-Datos]]). Tratamiento: winsorizar, marcar con una feature dummy, o modelar el evento explícitamente. Distinto es un **quiebre estructural** (cambio de nivel permanente): ahí no se corrige el dato, se corta la serie o se agrega una variable de régimen.

**🔧 Cold start (serie nueva, sin historia).** Un SKU recién lanzado no tiene pasado que aprender. Soluciones: el **modelo global** (sección 6) presta lo aprendido de series hermanas; usar atributos del producto (categoría, precio) como features; o partir de un análogo hasta acumular historia propia.

**👔 En una frase para el negocio:** los productos de baja rotación y los recién lanzados son justo donde el pronóstico ingenuo más dinero pierde — y donde estas técnicas específicas más lo devuelven.

---

## 9. Pronóstico probabilístico: el intervalo importa más que el punto

Audiencia: 🔧 🧭 👔

> [!important] 📌 El cambio de mentalidad que más valor genera
> Casi ninguna decisión de negocio se toma con la media del pronóstico. El stock de seguridad, la capacidad de servidores, la dotación de un call center, la reserva de tesorería — todas viven en un **cuantil alto**, porque el costo de quedarse corto no es igual al de sobrar. Un pronóstico que entrega solo un número está escondiendo la información que la decisión realmente necesita.

**🔧 Definición técnica:** en vez de un valor, se estima la **distribución** del futuro (o un intervalo). Tres caminos:

| Enfoque | Cómo | Nota |
|---|---|---|
| Intervalos paramétricos | ARIMA/ETS entregan intervalos asumiendo residuos normales | Rápidos; el supuesto de normalidad suele quedar corto en las colas |
| Quantile regression | Entrenar el modelo con **Pinball/Quantile Loss** ([[08-Metricas-de-Evaluacion]]) para predecir el P10, P50, P90 directamente | Sin supuesto de distribución; un modelo por cuantil |
| Simulación / conformal | Bootstrap de residuos, o conformal prediction para intervalos con cobertura garantizada | Robustos; conformal es la frontera actual |

> [!tip] 💡 Analogía
> Un pronóstico de punto es el hombre del tiempo que dice "mañana, 22 grados". El probabilístico dice "entre 18 y 26, con 90% de confianza". Solo el segundo te deja decidir si llevas abrigo — y en negocios, el abrigo es el stock de seguridad, la reserva o el turno extra.

> [!warning] ⚠️ Un intervalo se evalúa por su cobertura, no por lo angosto
> Un intervalo del 90% es honesto si el valor real cae dentro **el 90% de las veces**. Un intervalo angosto que solo acierta el 60% es peor que uno ancho bien calibrado: da falsa seguridad. Al validar cuantiles, mide **cobertura empírica** además del Pinball Loss.

**👔 En una frase para el negocio:** pedir "el pronóstico" a secas es pedir media información; la pregunta completa es "el pronóstico **y su intervalo**", porque la decisión se toma en el margen, no en el centro.

---

## 10. Validación y métricas específicas

Audiencia: 🔧 🧭 👔

**🔧 Walk-forward (rolling origin):** la única validación honesta. Entrenar hasta t, predecir t+1…t+h (el horizonte REAL de la decisión), avanzar el origen y repetir ([[10-Validacion-y-Leakage]]).

```
 fold 1: [██████ train ██████][→ h ]
 fold 2: [████████ train ████████][→ h ]
 fold 3: [██████████ train ██████████][→ h ]
                    el origen avanza; el test SIEMPRE es futuro
```

Dos variantes: **ventana creciente** (expanding: el train acumula toda la historia) vs. **ventana deslizante** (sliding: el train mantiene un largo fijo y olvida lo viejo — mejor si hay drift o quiebres).

**🔧 Las métricas, y cuándo cada una:**

| Métrica | Fórmula / idea | Cuándo conviene | Cuidado |
|---|---|---|---|
| MAE | media de │y−ŷ│ | Interpretable en unidades del negocio | No comparable entre series de escalas distintas |
| RMSE | √(media de (y−ŷ)²) | Penaliza errores grandes | Sensible a outliers |
| MAPE | media de │y−ŷ│/│y│ × 100 | Comunicación ejecutiva ("nos equivocamos 8%") | **Explota con ceros**; asimétrico (castiga más sobre-estimar) |
| SMAPE | versión simétrica del MAPE | Corrige parte de la asimetría | Aún inestable cerca de cero |
| WAPE | Σ│y−ŷ│ / Σ│y│ | Robusto con ceros; agrega bien por SKU | — |
| **MASE** | MAE_modelo / MAE_naive_in-sample (Hyndman & Koehler, 2006) | **La métrica de referencia**: comparable entre series, definida con ceros | Requiere definir bien el naive estacional |
| Pinball / Quantile Loss | error asimétrico por cuantil | Pronóstico probabilístico (sección 9) | Se evalúa junto con la cobertura |

**Lectura del MASE:** `< 1` = le ganas al naive; `> 1` = tu modelo sofisticado **pierde** contra "igual que ayer". Es la vara que desinfla la mayoría de los modelos que "parecían buenos".

> [!danger] 🚨 La ilusión del gráfico desplazado
> Una predicción que en el gráfico "sigue de cerca" a la serie real casi siempre es, en realidad, la serie **desplazada un rezago**: el modelo no aprendió estructura, solo repite el valor de ayer con un paso de retraso — y visualmente eso se ve casi idéntico a un buen ajuste. La superposición visual **nunca** es evidencia suficiente: valida contra el naive con MASE (Hyndman & Koehler, 2006). Un gráfico que "se ve genial" junto a un MASE cercano a 1 es la señal de que estás mirando un espejismo, no un modelo.

> [!warning] ⚠️ Reglas de oro del forecasting
> (1) **Baseline naive primero**: si no le ganas al seasonal naive con holgura, no tienes modelo — tienes decoración. (2) **Jamás barajar**: split temporal siempre. (3) **Auditar features**: toda variable debe conocerse ANTES del momento del pronóstico (la promo "planificada" que en los datos históricos aparece corregida a posteriori es leakage clásico, [[10-Validacion-y-Leakage]]). (4) **Pronosticar el intervalo, no solo el punto**: la decisión de inventario vive en el P90, no en la media. (5) **Validar al horizonte real de la decisión**: si compras con 4 semanas de anticipación, valida a 4 semanas, no a 1. (6) **Mismo protocolo para todos los modelos comparados**: mismo horizonte, misma información disponible — un naive one-step contra un modelo evaluado a multi-step gana por construcción, no por mérito; mide además cómo se **degrada el error por paso del horizonte**, no solo el promedio agregado (Tashman, 2000).

**👔 En una frase para el negocio:** un modelo no vale por lo bien que su gráfico se superpone a la historia — vale por cuánto le gana al "igual que ayer" en el horizonte real de la decisión, medido con el mismo protocolo que a sus competidores.

---

## 11. Guía de decisión: qué modelo, cuándo

Audiencia: 🔧 🧭

Síntesis operativa del tomo — de la situación al punto de partida:

| Tu situación | Punto de partida | Por qué |
|---|---|---|
| Serie única, corta, estable | Holt-Winters (ETS) o SARIMA | Robustos, pocos datos, transparentes |
| Serie única con feriados y cambios de tendencia | Prophet | Maneja changepoints y calendario sin tuning fino |
| Miles de series relacionadas (retail) | LightGBM **global** con features | Cross-learning, un solo pipeline, ganó la M5 |
| Estacionalidad múltiple (horaria/eléctrica) | MSTL, Prophet o ML con features | Los modelos de una sola estación no bastan |
| Demanda intermitente (muchos ceros) | Croston / SBA / TSB | El instrumental normal falla con ceros |
| Serie nueva sin historia | Modelo global + atributos | Presta lo aprendido de series hermanas |
| Necesitas distribución, no punto | Quantile regression o DeepAR | La decisión vive en un cuantil |
| **Siempre, en todos los casos** | **Seasonal naive como baseline** | Es la vara que valida si vale la pena lo demás |

> [!tip] 🧭 El orden correcto de trabajo
> Descomponer → baseline naive → un modelo clásico simple → (solo si el volumen y las series lo justifican) ML global → probabilístico si la decisión lo pide. **Subir un escalón solo cuando la validación walk-forward demuestre que el anterior no basta.** La complejidad se gana, no se asume.

---

## 📖 Referencias de este tomo

- (Box & Jenkins, 1970) — la obra fundacional de ARIMA.
- (Cleveland et al., 1990) — STL.
- (Hyndman & Koehler, 2006) — MASE y métricas de forecast.
- (Taylor & Letham, 2018) — Prophet.
- (Makridakis et al., 2022) — resultados de la competencia M5 (dominio de los modelos globales).
- (Wickramasuriya et al., 2019) — reconciliación óptima (MinT) en forecasting jerárquico.
- (Hyndman & Athanasopoulos, 2021) — *Forecasting: Principles and Practice*, la referencia moderna abierta.
- (Wang, Smith & Hyndman, 2006) — medidas de fuerza de tendencia y estacionalidad (F_T, F_S) sobre componentes STL.
- (Shumway & Stoffer, 2017) — análisis espectral / periodograma para detectar el período estacional.
- (Granger & Newbold, 1974) — regresión espuria entre series con tendencia no estacionaria.
- (Engle & Granger, 1987) — cointegración y su prueba de dos pasos.
- (Ljung & Box, 1978) — test de Ljung-Box para autocorrelación de residuos.
- (Engle, 1982) — heterocedasticidad condicional (ARCH), base de GARCH.
- (Ben Taieb, Bontempi, Atiya & Sorjamaa, 2012) — comparación de estrategias de pronóstico multi-step.
- (Granger, 1969) — causalidad de Granger, como screening de variables exógenas.
- (Tashman, 2000) — protocolo comparable de evaluación out-of-sample en forecasting.

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[16-Bibliografia|16 · Bibliografía]] · Siguiente: [[18-Causalidad-y-Uplift|18 · Causalidad y Uplift ➡]]

> **Próximo tomo:** [[18-Causalidad-y-Uplift]] — de predecir a intervenir: cuándo una correlación autoriza una decisión, y a quién dirigir la acción porque la acción *cambia* su comportamiento.
