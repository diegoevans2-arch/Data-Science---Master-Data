---
title: "Tomo 17 — Series de Tiempo y Forecasting"
tags: [data-science, machine-learning, time-series, forecasting]
audiencias: [tecnico, puente, ejecutivo]
tomo: 17
version: 6.1
updated: 2026-07-19
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

> [!danger] 🚨 Sobre-diferenciar es tan malo como no diferenciar
> Diferenciar de más **inyecta autocorrelación artificial** y agranda la varianza del pronóstico. Regla práctica: casi ninguna serie de negocio necesita `d > 2`. Si la serie tiene tendencia clara, empieza con `d=1` y verifica con los tests; no diferencies "por si acaso".

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

**🧭 Cuándo usarlo:** diagnóstico previo a ARIMA y **auditoría de residuos**. La prueba de que un modelo capturó toda la estructura: sus residuos NO deben mostrar autocorrelación (test de **Ljung-Box**, `p > 0.05` = residuos parecen ruido blanco = bien). Si los residuos aún tienen memoria, el modelo dejó señal sobre la mesa.

> [!note] En la práctica: `auto_arima`
> Leer ACF/PACF a ojo es un arte que conviene entender, pero en producción se usa búsqueda automática de órdenes (`auto_arima` de pmdarima, o el AutoARIMA de las librerías modernas) que optimiza AIC/BIC ([[02-Fundamentos-Matematicos]]). El diagnóstico manual sigue valiendo para **entender** lo que la búsqueda automática eligió y para auditar residuos.

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

> [!warning] ⚠️ Reglas de oro del forecasting
> (1) **Baseline naive primero**: si no le ganas al seasonal naive con holgura, no tienes modelo — tienes decoración. (2) **Jamás barajar**: split temporal siempre. (3) **Auditar features**: toda variable debe conocerse ANTES del momento del pronóstico (la promo "planificada" que en los datos históricos aparece corregida a posteriori es leakage clásico, [[10-Validacion-y-Leakage]]). (4) **Pronosticar el intervalo, no solo el punto**: la decisión de inventario vive en el P90, no en la media. (5) **Validar al horizonte real de la decisión**: si compras con 4 semanas de anticipación, valida a 4 semanas, no a 1.

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

Fichas completas en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[16-Bibliografia|16 · Bibliografía]] · Siguiente: [[18-Causalidad-y-Uplift|18 · Causalidad y Uplift ➡]]

> **Próximo tomo:** [[18-Causalidad-y-Uplift]] — de predecir a intervenir: cuándo una correlación autoriza una decisión, y a quién dirigir la acción porque la acción *cambia* su comportamiento.
