---
title: "Tomo 05 — Escalado de Datos"
tags: [data-science, machine-learning, scaling, preprocessing, transformaciones]
audiencias: [tecnico, puente, ejecutivo]
tomo: 05
version: 6.2
updated: 2026-08-28
---

# ⚖️ Tomo 05 — Escalado de Datos

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[04-EDA|04 · EDA]] · Siguiente: [[06-Clustering|06 · Clustering ➡]]

---

> [!info] 📌 ¿Por qué importa el escalado?
> Si tienes una variable "ingresos" que va de 200.000 a 10.000.000 y otra "edad" que va de 18 a 80, el algoritmo va a pensar que ingresos es **miles de veces más importante** simplemente porque sus números son más grandes. Escalar pone a todas las variables en igualdad de condiciones. No es cosmético: es un requisito vital para algoritmos basados en **distancias** (KNN, SVM, K-Means), basados en **gradientes** (regresión logística, redes neuronales) y para la interpretabilidad de coeficientes en modelos lineales.

> [!abstract] 👔 Impacto ejecutivo
> Una decisión de dos líneas de código que puede invertir por completo los resultados de una segmentación o un scoring.
>
> - **Decisiones que habilita:** segmentaciones donde cada variable pesa por su señal y no por sus unidades, comparaciones justas entre indicadores medidos en escalas distintas.
> - **Costo de hacerlo mal:** clusters que son puro "quién gana más", modelos de distancia ciegos a todo menos la variable grande, y leakage silencioso si se escala antes de separar train/test.
> - **Pregunta ejecutiva que responde:** *¿los grupos y scores que me muestran reflejan comportamiento real, o solo la variable con los números más grandes?*

> [!tip] 💡 Analogía general: las pruebas de distintos colegios
> Alumnos de dos colegios rindieron pruebas diferentes: una se corrige de 0 a 100 y la otra de 0 a 1000. Si comparas puntajes crudos, el alumno con 900/1000 "se ve" mejor que el que sacó 95/100 — aunque ambos están al 90%. Escalar es convertir todos los puntajes a porcentaje para que la comparación sea justa. Con features pasa igual: sin escalar, comparas peras de 0–80 con manzanas de 0–10.000.000.

> [!warning] ⚠️ LA REGLA CLAVE (memorízala)
> **Si el modelo pregunta "¿es X mayor que Y?" no necesita escala. Si pregunta "¿qué tan lejos está X de Y?" sí la necesita.**
> Un árbol de decisión evalúa `edad > 30` — le da lo mismo si la edad está en años o en segundos. KNN calcula distancias, SVM busca márgenes, una red neuronal multiplica pesos: para todos ellos, la escala **es** la señal.

**El efecto en un dibujo — por qué la distancia miente sin escalar:**

```
 SIN ESCALAR                                  ESCALADO (z-score)
 edad: 18–80 · ingreso: 200K–10M              ambas ~ N(0,1)

 ingreso                                       ingreso_z
 10M ┤            ● B                          +2 ┤         ● B
     │                                            │
     │              distancia A–B                 │       ● A
     │              ≈ │Δingreso│               +0 ┤
     │              (la edad no pesa NADA)        │   distancia A–B ahora
 5M  ┤   ● A                                      │   mezcla edad E ingreso
     │                                         −2 ┤
     └────┬──────┬────── edad                     └────┬──────┬──── edad_z
         25     60                                    −1     +1
```

> [!example] 📊 Caso de negocio — Manufactura: los clusters que solo veían presión
> **Problema:** una planta industrial agrupa "modos de operación" de sus máquinas con K-Means sobre sensores: temperatura (20–90 °C), vibración (0–4 mm/s) y presión (100.000–600.000 Pa). Los clusters resultantes son inservibles: separan solo por presión — la variable con números gigantes — y el equipo de mantenimiento no ve reflejados los modos de falla que conoce.
>
> **Técnica aplicada:** StandardScaler dentro de un Pipeline (fit solo con datos de operación normal de entrenamiento), y RobustScaler como alternativa al detectar outliers de vibración durante fallas ([[04-EDA]]). Re-clustering con las tres señales ahora en igualdad de condiciones ([[06-Clustering]]).
>
> **Resultado:** los nuevos clusters separan modos de operación reconocibles (arranque en frío, régimen normal, pre-falla por vibración) y uno de ellos anticipa fallas con horas de ventaja. El cambio fue **una línea de preprocesamiento** — la lección: antes de comprar más sensores o modelos, verificar que los que hay compitan en igualdad de escala.

---

## 1. La tabla completa de técnicas

Audiencia: 🔧 🧭

| Técnica | Fórmula | Rango output | Fortaleza | Debilidad | Cuándo usar | 💡 Analogía |
|---|---|---|---|---|---|---|
| Min-Max Scaling | `(x − min) / (max − min)` | [0, 1] | Interpretable, acotado | Muy sensible a outliers (min/max se distorsionan) | Redes neuronales, imágenes, cuando se necesita rango [0,1] | Convertir toda nota a "porcentaje del curso" |
| StandardScaler (Z-score) | `(x − μ) / σ` | (−∞, +∞), media 0, std 1 | Robusto, el más utilizado | No acota el rango; μ y σ se afectan por outliers | **Default general**: SVM, PCA, regresión logística | Medir a todos en "desviaciones sobre el promedio" |
| RobustScaler | `(x − mediana) / IQR` | No acotado, centrado en mediana | Resistente a outliers (mediana e IQR) | No garantiza media 0 ni rango fijo | Datasets con outliers que no se quieren eliminar | La regla que ignora al multimillonario del barrio |
| MaxAbsScaler | `x / max(│x│)` | [−1, 1] | Preserva esparsidad (no centra) | Sensible al máximo absoluto | Matrices sparse, TF-IDF, datos ya centrados en 0 | Escalar sin despertar a los ceros dormidos |
| QuantileTransformer (uniform) | Percentiles → distribución uniforme | [0, 1] | Robusto a outliers extremos | Pierde la forma original de la distribución | Cuando se necesita distribución uniforme estricta | Convertir cada valor en su ranking percentil |
| QuantileTransformer (normal) | Percentiles → distribución normal | (−∞, +∞) | Fuerza normalidad real | Distorsiona distancias relativas | Algoritmos que asumen normalidad estricta | Rehacer la fila por estatura hasta formar campana |
| PowerTransformer (Box-Cox) | `(xᶺλ − 1)/λ` si λ≠0; `log(x)` si λ=0 | Varía | Estabiliza varianza, normaliza (Box & Cox, 1964) | **Solo datos estrictamente positivos** | Skew positivo: ingresos, precios, áreas | Planchar la cola larga de la distribución |
| PowerTransformer (Yeo-Johnson) | Extensión de Box-Cox | Varía | Acepta negativos y cero | Más compleja que Box-Cox | Como Box-Cox pero sin restricción de signo | El planchado que también acepta números rojos |
| Log transform (manual) | `log(x + 1)` = `log1p(x)` | Varía | Simple, muy efectiva contra skew | Solo no-negativos; cambia la interpretación | Ingresos, conteos, precios, tasas | Mirar los montos en "órdenes de magnitud" |
| Normalizer L2 | `x / ‖x‖₂` **por fila** | Norma 1 por muestra | Ideal para similitud coseno | Normaliza muestras, no features | TF-IDF, K-Means sobre texto, similitud de documentos | Comparar recetas por proporciones, no por tamaño de olla |
| Transformación cíclica | `sin(2π·x/max)`, `cos(2π·x/max)` | [−1, 1] ×2 columnas | Respeta la circularidad del tiempo | Duplica columnas; requiere conocer el período | Hora, día de semana, mes, ángulos | El reloj redondo: las 23:00 y la 01:00 son vecinas |

**🧭 Nota de lectura:** StandardScaler es el default sensato; el resto existe para cuando sus supuestos fallan (outliers → Robust; sparse → MaxAbs; colas → log/Power; normalidad exigida → Quantile normal; filas-documento → L2; tiempo circular → sin/cos).

---

## 2. Transformaciones no lineales en detalle

Audiencia: 🔧 🧭

### 2.1 ¿Cuándo usar transformación logarítmica?

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica — tres señales de que toca log:**

1. La distribución tiene fuerte **asimetría positiva** (cola larga a la derecha: ingresos, montos, conteos) ([[04-EDA]]).
2. Los residuos del modelo muestran **heterocedasticidad**: la varianza crece con el valor predicho ([[07-Modelos-Supervisados]]).
3. La relación feature–target es **multiplicativa**, no aditiva ("un 10% más" en lugar de "$10.000 más").

Usar `log1p` (log(x+1)) para tolerar ceros; recordar revertir con `expm1` al reportar ([[04-EDA]]).

**👔 En una frase para el negocio:** cuando los datos viven en porcentajes y múltiplos (como casi todo lo económico), el log los lleva al terreno donde los modelos lineales piensan bien.

### 2.2 Box-Cox: cómo se elige λ

Audiencia: 🔧

**🔧 Definición técnica:** `scipy.stats.boxcox()` estima automáticamente el λ que maximiza la log-likelihood de normalidad. Guía de lectura del λ resultante:

| λ óptimo | Transformación equivalente |
|---|---|
| λ = 1 | Sin transformación |
| λ = 0.5 | Raíz cuadrada |
| λ = 0 | Logaritmo |
| λ = −1 | Inversa (1/x) |

En sklearn: `PowerTransformer(method='box-cox')` (exige positivos estrictos) o `method='yeo-johnson'` (acepta ceros y negativos); ambos con `standardize=True` dejan además media 0 y varianza 1.

### 2.3 Transformación cíclica (sin/cos)

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> En una regla recta, las 23:00 y la 01:00 quedan en extremos opuestos (23 vs 1: ¡22 unidades de distancia!). En un **reloj redondo**, están pegadas — a dos horas una de otra. Codificar con sin/cos es dibujar el reloj: le devuelve al modelo la geometría circular que el número lineal le robó.

**🔧 Definición técnica:** para variables con periodicidad (hora del día, día de la semana, mes, dirección del viento): crear dos columnas `sin(2π·x/período)` y `cos(2π·x/período)`. Así la distancia entre el final y el inicio del ciclo es mínima, como corresponde. Se necesitan **ambas** columnas (solo el seno confunde la mañana con la tarde).

**🧭 Cuándo usarlo:** siempre que una variable temporal entre a un modelo de distancia o gradiente. Los árboles pueden sobrevivir sin ella (cortan rangos), pero también se benefician en períodos que "dan la vuelta".

**👔 En una frase para el negocio:** evita que el modelo crea que la medianoche está lejísimos de las 23:59 — un clásico silencioso en demanda horaria, turnos y estacionalidad.

---

## 3. Reglas críticas del escalado

Audiencia: 🔧 🧭 👔

### 3.1 Fit solo en train — sin excepciones

Audiencia: 🔧

> [!danger] 🚨 Regla absoluta: `fit()` solo sobre entrenamiento
> El `scaler.fit()` se ejecuta ÚNICAMENTE sobre datos de entrenamiento; luego `transform()` se aplica a train, validación y test por separado. Fitear con todo el dataset introduce **data leakage**: el modelo "conoce" la escala (min, max, μ, σ) de los datos de test antes de verlos ([[10-Validacion-y-Leakage]]).

```
 ❌ INCORRECTO — leakage de escala          ✅ CORRECTO — escala solo desde train

 1. fit(TODO el dataset)                    1. split → train / test
    (μ y σ ya "vieron" el test)             2. fit(train)      ← solo train define μ, σ
 2. split → train / test                    3. transform(train)
    (train y test comparten escala)         4. transform(test) ← con la escala de train
```

### 3.2 Algoritmos que NO requieren escalado

Audiencia: 🔧 🧭

> [!info] 📌 Los inmunes a la escala
> **Árboles de decisión, Random Forest, XGBoost, LightGBM, CatBoost, GBM clásico** — todo modelo basado en árboles es invariante a la escala (solo pregunta "¿mayor que el umbral?"). **Naive Bayes** tampoco lo requiere (trabaja con probabilidades por feature). En ellos, escalar no daña pero tampoco ayuda.

| Familia | ¿Requiere escalado? | Por qué |
|---|---|---|
| KNN, K-Means, DBSCAN/HDBSCAN, SVM/SVR | **Sí, obligatorio** | Operan con distancias o márgenes ([[06-Clustering]], [[07-Modelos-Supervisados]]) |
| Regresión lineal/logística regularizada, redes neuronales, PCA | **Sí** | Gradientes, penalizaciones y varianzas comparan magnitudes ([[11-Mejora-de-Modelos]], [[12-Deep-Learning]]) |
| Árboles, Random Forest, XGBoost/LightGBM/CatBoost, Naive Bayes | **No** | Umbrales por feature o probabilidades independientes de la escala |

### 3.3 Pipeline de sklearn: el cinturón de seguridad

Audiencia: 🔧

> [!warning] ⚠️ Regla crítica: el scaler vive dentro del Pipeline
> `Pipeline([('scaler', StandardScaler()), ('model', LogisticRegression())])` garantiza que el scaler se fittee **solo con el fold de train en cada iteración** del cross-validation. Es la forma correcta de integrar preprocesamiento y modelo — y la única que sobrevive honesta a un `GridSearchCV` ([[10-Validacion-y-Leakage]], [[13-MLOps-XAI-Etica]]).

```
 fold 1  [ train ][ train ][ test  ]  →  fit(scaler, train) → transform(train) → transform(test)
 fold 2  [ train ][ test  ][ train ]  →  fit(scaler, train) → transform(train) → transform(test)
 fold 3  [ test  ][ train ][ train ]  →  fit(scaler, train) → transform(train) → transform(test)

 el scaler se re-fittea en CADA fold, solo con la porción de train de ESE fold
```

### 3.4 Interacción escalado ↔ imputación — el orden del pipeline

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> No puedes medir tu estatura si te falta un pie: primero te ponen la prótesis (imputación), después te miden (escalado). Invertir el orden fuerza al metro a inventar una medida sin dato — y contamina la escala con valores ficticios.

**🔧 Definición técnica — el orden correcto dentro del Pipeline:**

```
 Pipeline([
   ('imputer', SimpleImputer(strategy='median')),  ← 1° llenar huecos
   ('scaler',  StandardScaler()),                  ← 2° escalar
   ('model',   LogisticRegression())               ← 3° entrenar
 ])
```

**🔧 Por qué este orden y no al revés:**

| Orden | Qué pasa | Problema |
|---|---|---|
| Imputar → Escalar ✅ | El scaler ve datos completos; sus estadísticos (μ, σ, min, max) son estables | Ninguno — es el correcto |
| Escalar → Imputar ❌ | El scaler calcula μ/σ ignorando NaNs (o falla); la imputación posterior inserta valores en escala original que ya no matchea | Los imputados quedan en escala distinta al resto; sesgo silencioso |

**🔧 Caveats adicionales:**

- **KNNImputer** opera con distancias → necesita features en la misma escala para funcionar. Pero si escalas antes, los NaNs no tienen escala. Solución: usar un IterativeImputer (que es model-based y tolera NaN) o una imputación por mediana primero, escalar, y luego refinar con KNN.
- **Imputación con constante (-999, 0)**: si se imputa con un valor fuera de rango y luego se aplica Min-Max, ese valor extremo colapsa el rango útil. Preferir mediana/media para imputar antes de scalers sensibles a extremos.
- **ColumnTransformer** cuando distintas columnas necesitan distinto tratamiento: numéricas → impute + scale; categóricas → impute + encode. Cada rama con su orden propio ([[03-Preparacion-de-Datos]]).

### 3.5 Escalado de features ordinales

Audiencia: 🔧

**🔧 Definición técnica:** variables categóricas con orden intrínseco (nivel_educativo: básica < media < superior < postgrado; satisfacción: 1 < 2 < 3 < 4 < 5) pueden codificarse como enteros y luego escalarse — pero con cuidado:

| Estrategia | Cuándo funciona | Cuándo falla |
|---|---|---|
| OrdinalEncoder → StandardScaler | Si los intervalos entre niveles son aproximadamente iguales (ratings 1-5) | Si los intervalos son desiguales (ingreso bajo/medio/alto donde "alto" es 10× más que "medio") |
| OrdinalEncoder → QuantileTransformer | Si la distribución de las categorías es muy desigual (80% en nivel 1, 10% en 2, etc.) | Pierde la noción de equidistancia; úsalo solo cuando la distribución manda más que el orden |
| Target Encoding (luego escalar) | Features de alta cardinalidad ordinal (ej: código postal que correlaciona con income) | Requiere regularización para evitar overfitting; debe hacerse con folds para evitar leakage ([[10-Validacion-y-Leakage]]) |

**🧭 Regla de decisión:** si la ordinal tiene ≤ 7 niveles con espaciado razonable, OrdinalEncoder + StandardScaler es suficiente. Si tiene muchos niveles o espaciado dudoso, considerar target encoding (con folds) o simplemente tratarla como numérica continua.

---

## 4. Guía rápida de selección

Audiencia: 🧭

| Situación | Técnica recomendada |
|---|---|
| Caso general, sin outliers graves | StandardScaler |
| Outliers presentes que no se quieren eliminar | RobustScaler |
| Redes neuronales / imágenes / se exige [0,1] | Min-Max |
| Matriz sparse (TF-IDF, one-hot masivo) | MaxAbsScaler |
| Skew positivo fuerte (montos, conteos) | log1p o PowerTransformer |
| Datos con negativos y skew | Yeo-Johnson |
| El algoritmo exige normalidad estricta | QuantileTransformer (normal) |
| Similitud entre documentos/filas | Normalizer L2 |
| Hora, día, mes, ángulo | Transformación cíclica sin/cos |
| Modelo de árboles/boosting | Ninguna (no la necesita) |

**Las tres reglas de oro del tomo:** (1) la escala se decide por el **modelo**, no por estética; (2) `fit()` solo en train, siempre dentro de un Pipeline; (3) ante duda con outliers, RobustScaler antes que borrar filas.

---

## 📖 Referencias de este tomo

- (Box & Cox, 1964) — la transformación de potencia original.
- (Géron, 2022) — pipelines de preprocesamiento y escalado en la práctica.
- (Kuhn & Johnson, 2019) — transformaciones de features y sus efectos en los modelos.
- Documentación oficial: [scikit-learn.org — Preprocessing](https://scikit-learn.org/stable/modules/preprocessing.html).

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[04-EDA|04 · EDA]] · Siguiente: [[06-Clustering|06 · Clustering ➡]]

> **Próximo tomo:** [[06-Clustering]] — aprendizaje no supervisado: K-Means y familia, DBSCAN/HDBSCAN/OPTICS, jerárquico con sus linkages, GMM, Spectral, Mean Shift, Affinity Propagation, y la tabla completa de métricas para evaluar clusters y elegir K.
