---
title: "Tomo 07 — Modelos Supervisados"
tags: [data-science, machine-learning, clasificacion, regresion, anomalias]
audiencias: [tecnico, puente, ejecutivo]
tomo: 07
version: 6.1
updated: 2026-09-06
---

# 🤖 Tomo 07 — Modelos Supervisados

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[06-Clustering|06 · Clustering]] · Siguiente: [[08-Metricas-de-Evaluacion|08 · Métricas de Evaluación ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> El modelo es la pieza que convierte datos en predicciones. **No existe "el mejor modelo"**: cada uno tiene supuestos, fortalezas y debilidades, y elegir el incorrecto es usar un martillo para apretar un tornillo. La elección depende del problema, el tamaño y estructura de los datos, los requisitos de interpretabilidad, la latencia de inferencia y el cómputo disponible. Principio rector — la Navaja de Ockham: **preferir el modelo más simple que resuelva el problema con rendimiento suficiente**.

> [!abstract] 👔 Impacto ejecutivo
> Cada familia de modelos compra algo distinto: velocidad, precisión, explicabilidad o tolerancia a datos imperfectos. Elegir es una decisión de negocio disfrazada de decisión técnica.
>
> - **Decisiones que habilita:** balancear precisión vs explicabilidad ante reguladores, dimensionar costos de entrenamiento e inferencia, exigir baselines antes de aprobar complejidad.
> - **Costo de hacerlo mal:** redes neuronales caras donde un boosting gana, modelos caja-negra en dominios regulados, o el clásico "modelo brillante que nadie puede mantener".
> - **Pregunta ejecutiva que responde:** *¿por qué el equipo eligió ESTE modelo y no uno más simple/barato/explicable?*

**El mapa del arsenal:**

```
                       ¿Qué predigo?
          ┌────────────────┼──────────────────────┐
          ▼                ▼                      ▼
     CATEGORÍA          NÚMERO               "LO RARO"
    (clasificación)    (regresión)        (anomalías, sin
          │                │                etiquetas o casi)
   LogReg · KNN      OLS · Ridge/Lasso/   Isolation Forest ·
   SVM · Árboles     ElasticNet · SVR ·   LOF · One-Class SVM ·
   RF · ExtraTrees   Polynomial · árboles Elliptic Envelope ·
   XGB/LGBM/CatB     y boosting de        Autoencoder
   Naive Bayes       regresión
          │                │
          └── métricas [[08-Metricas-de-Evaluacion]] · validación [[10-Validacion-y-Leakage]] ──┘
```

> [!example] 📊 Caso de negocio — Banca: el comité que exigía dos modelos
> **Problema:** un banco moderniza su scoring de crédito. Riesgo quiere el máximo poder predictivo; cumplimiento exige poder explicar cada rechazo al regulador.
>
> **Técnica aplicada:** estrategia de dos niveles: una **regresión logística** regularizada como modelo campeón regulatorio (coeficientes = puntos de score explicables) y un **gradient boosting** (LightGBM) como challenger de máximo rendimiento. Comparación honesta con stratified K-fold ([[10-Validacion-y-Leakage]]), métricas AUC-PR y KS por el desbalance ([[08-Metricas-de-Evaluacion]]), y SHAP sobre el boosting para auditar que no dependa de proxies indebidos ([[13-MLOps-XAI-Etica]]).
>
> **Resultado:** el boosting gana por margen relevante y SHAP demuestra factores razonables → se aprueba como motor de decisión con la logística como respaldo explicativo y sistema de contraste. La decisión de arquitectura no la tomó el data scientist solo: la tomó el trade-off precisión↔explicabilidad **puesto sobre la mesa del comité**.

---

## 1. Modelos de Clasificación

Audiencia: 🔧 🧭

### Regresión Logística

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un interruptor de seguridad que se activa cuando la evidencia acumulada supera un umbral. Cada feature suma o resta "puntos de sospecha"; la sigmoide convierte el puntaje total en una probabilidad suave entre 0 y 1, y el umbral decide cuándo saltar.

**🔧 Definición técnica:** `P(y=1|x) = σ(wᵀx + b) = 1/(1+e^−(wᵀx+b))`. Pérdida: Binary Cross-Entropy (Log Loss), convexa ([[02-Fundamentos-Matematicos]]) → óptimo global garantizado; se optimiza con LBFGS o descenso de gradiente. **Multiclase:** softmax (multinomial), One-vs-Rest (K clasificadores) u One-vs-One (K(K−1)/2). **Regularización:** parámetro `C` (inverso de λ: C pequeño = más regularización); `penalty` = 'l2' (default), 'l1' (selección de features), 'elasticnet'. **Supuestos:** linealidad en el espacio log-odds, independencia de observaciones, sin multicolinealidad severa ([[04-EDA]]).

**Ventajas:** probabilidades razonablemente calibradas, coeficientes interpretables (log-odds), rápido, excelente baseline. **Limitaciones:** no captura no-linealidades sin feature engineering ([[03-Preparacion-de-Datos]]), sensible a outliers. **Requisitos:** requiere escalado ([[05-Escalado-de-Datos]]); no maneja nulls; data tabular.

**🧭 Cuándo usarlo:** SIEMPRE como baseline; como campeón cuando la explicabilidad es requisito regulatorio; en producción de baja latencia. Caso de uso: spam, scoring, propensión.

**👔 En una frase para el negocio:** el modelo que cualquier auditor puede leer: cada factor tiene un peso visible y defendible.

### K-Nearest Neighbors (KNN)

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Para clasificar al nuevo vecino, preguntas a sus 5 vecinos más cercanos "¿de qué tipo es esta zona?" y le haces caso a la mayoría. No hay teoría: pura memoria del barrio.

**🔧 Definición técnica:** perezoso (lazy): memoriza el training set; para predecir calcula la distancia del punto nuevo a todos los de train, toma los K más cercanos y vota (clasificación) o promedia (regresión) (Cover & Hart, 1967). **Distancias:** euclídea (L2, default), Manhattan (L1, robusta), Minkowski, Hamming (categóricas), coseno (texto/embeddings). **Elección de K:** K=1 overfitting severo; K grande underfitting; regla inicial K=√N; afinar con CV; K impar evita empates binarios. **Aceleración:** KD-Tree y Ball-Tree bajan la búsqueda a O(D·log N) — efectivos con D < 20; en alta dimensión la búsqueda lineal vuelve a ganar.

**Ventajas:** cero entrenamiento, no paramétrico, fronteras arbitrarias. **Limitaciones:** inferencia O(N·D) por punto, memoria alta, maldición de la dimensionalidad ([[03-Preparacion-de-Datos]]). **Requisitos:** escalado **obligatorio**; no maneja nulls; pocas features idealmente.

**🧭 Cuándo usarlo:** datasets chicos-medianos con pocas dimensiones y fronteras irregulares; sistemas de similitud ("clientes parecidos a este"). Caso de uso: Iris, recomendación por similitud.

**👔 En una frase para el negocio:** decide por analogía con los casos históricos más parecidos — intuitivo de explicar, caro de servir a gran escala.

### Support Vector Machine (SVM / SVC)

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Trazar la **calle más ancha posible** entre dos barrios: la frontera no es cualquier línea que separe, sino la que deja el máximo margen a ambos lados. Y si los barrios están entrelazados, el kernel es un truco de perspectiva: los mira desde una dimensión superior donde sí se separan con una calle recta.

**🔧 Definición técnica:** maximiza el margen entre clases; los puntos que lo definen son los **vectores de soporte** (Cortes & Vapnik, 1995). Optimización convexa (QP). **Soft margin `C`:** C grande = margen estrecho, pocas violaciones (riesgo de overfitting); C pequeño = margen ancho, más tolerancia (más regularización). **Kernel trick:** proyección implícita a alta dimensión; kernels: lineal, RBF (default de facto), polinomial, sigmoide. **RBF gamma:** `K(x,x′) = exp(−γ‖x−x′‖²)`; γ grande = frontera muy local (overfit); γ pequeño = suave (underfit). Probabilidades vía Platt scaling (no nativas, [[11-Mejora-de-Modelos]]).

**Ventajas:** potente en alta dimensión (texto), fronteras complejas con pocos datos. **Limitaciones:** O(N²)–O(N³) con kernel: impracticable N > ~10–50K — el caso lineal (`LinearSVC`, SGD) escala casi linealmente a millones de filas, y el kernel RBF puede aproximarse (Nyström, random features) cuando N lo exige (documentación oficial de scikit-learn, consultada 2026-09-06); dos hiperparámetros sensibles (C, γ); poco interpretable. **Requisitos:** escalado **obligatorio**; no maneja nulls.

**🧭 Cuándo usarlo:** datasets chicos-medianos de alta dimensión (texto TF-IDF, bioinformática) donde el boosting no domina. Caso de uso: clasificación de documentos, detección de caras clásica.

**👔 En una frase para el negocio:** el especialista en trazar la frontera más prudente cuando hay pocos datos y muchas variables.

### Decision Tree

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El juego de las 20 preguntas: "¿gana más de $50K?", "¿tiene más de 40 años?" — cada pregunta parte los datos en dos, y al final de la cadena de preguntas hay un veredicto. Todo el modelo **es** un árbol de preguntas que cualquiera puede seguir con el dedo.

**🔧 Definición técnica:** divide recursivamente el espacio con reglas binarias `feature > umbral` que maximizan la pureza de los hijos. **Criterios:** Gini `1 − Σpᵢ²` (default, más rápido) vs Entropy `−Σpᵢ·log₂(pᵢ)` (Information Gain) — en la práctica casi idénticos. **Criterios de parada:** `max_depth`, `min_samples_split`, `min_samples_leaf`, `min_impurity_decrease`. **Poda:** cost-complexity pruning post-hoc con `ccp_alpha` (elimina subárboles que no pagan su complejidad).

**Ventajas:** interpretable al 100%, sin escalado, maneja no-linealidades e interacciones, rápido. **Limitaciones:** overfitting salvaje sin restricciones; **inestable** (datos levemente distintos → árbol muy distinto); sesgo hacia features con muchos valores únicos. **Requisitos:** NO requiere escalado. Valores faltantes: desde scikit-learn 1.3 (2023) los árboles aceptan NaN de forma nativa con `splitter='best'` — para cada umbral candidato el splitter evalúa enviar los faltantes al hijo izquierdo o al derecho y conserva la mejor opción; si una feature no tuvo faltantes en train, en predicción los NaN van al hijo con más muestras (documentación oficial de scikit-learn, consultada 2026-09-06). Imputar sigue siendo razonable cuando el mecanismo de ausencia importa o se exige trazabilidad ([[03-Preparacion-de-Datos]]). *(Corregido el 2026-09-06: la guía decía «sklearn no acepta nulls».)*

**🧭 Cuándo usarlo:** cuando la regla de decisión debe ser visible (riesgo crediticio ante reguladores, protocolos médicos); como pieza base de ensembles. Solo, rara vez es el mejor.

**👔 En una frase para el negocio:** el único modelo que se puede imprimir y colgar en la pared como diagrama de decisión.

### Random Forest

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> En lugar de un solo árbitro, consultas a **100 árbitros que vieron el partido desde ángulos distintos** (cada uno vio una muestra distinta del juego y se fijó en jugadas distintas). La decisión final es la mayoría — y la sabiduría de la multitud corrige los sesgos de cada árbitro individual.

**🔧 Definición técnica:** ensemble de árboles con **bagging + feature randomness** (Breiman, 2001a): (1) bootstrap: cada árbol entrena con una muestra con reemplazo (~63% de los datos); (2) en cada split solo se considera un subconjunto aleatorio de features (√D clasificación, D/3 regresión). **OOB error:** el ~37% no visto por cada árbol sirve de validación gratis (`oob_score=True`). **Feature importance:** MDI (reducción media de impureza) — sesgada hacia alta cardinalidad/continuas; preferir permutation importance para conclusiones ([[13-MLOps-XAI-Etica]]). **Hiperparámetros:** `n_estimators` (100–500, más mejora con rendimientos decrecientes), `max_depth`, `max_features`, `min_samples_leaf`.

**Ventajas:** robusto out-of-the-box, difícil de sobreajustar gravemente, paraleliza, poca sensibilidad a hiperparámetros. **Limitaciones:** inferencia más lenta con muchos árboles; **no extrapola** fuera del rango de train (regresión); menos interpretable que un árbol. **Requisitos:** NO requiere escalado; valores faltantes nativos desde scikit-learn 1.4 (2024) — y en ExtraTrees desde 1.6 — con la misma regla de enrutamiento que el árbol individual.

**🧭 Cuándo usarlo:** el todoterreno tabular: primer modelo serio tras el baseline, base sólida cuando no hay tiempo de tunear boosting. Caso de uso: churn, fraude tabular, scoring rápido.

**👔 En una frase para el negocio:** el caballo de batalla confiable: fuerte sin afinación fina, difícil de romper, razonablemente explicable.

### ExtraTrees (Extremely Randomized Trees)

Audiencia: 🔧

> [!tip] 💡 Analogía
> Los 100 árbitros de Random Forest, pero aún más despreocupados: en vez de buscar el mejor punto de corte en cada jugada, cada uno corta en un punto **al azar** dentro del rango. Individualmente más torpes; en conjunto, sorprendentemente buenos — y mucho más rápidos de entrenar.

**🔧 Definición técnica:** como Random Forest, pero (1) usa todo el dataset (sin bootstrap por defecto) y (2) los umbrales de split se eligen **aleatoriamente** en lugar de optimizarse. Resultado: más aleatorización → menos varianza, ligeramente más bias, entrenamiento notablemente más rápido.

**🧭 Cuándo usarlo:** alternativa a RF cuando el tiempo de entrenamiento importa o cuando RF muestra overfitting residual; vale la pena probar ambos — cuál gana depende del dataset.

### Gradient Boosting — XGBoost, LightGBM, CatBoost

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un equipo de aprendices donde cada nuevo miembro se especializa en corregir **exactamente lo que el equipo anterior hizo mal**: el primero da una respuesta burda, el segundo estudia sus errores y los corrige un poco, el tercero corrige los errores que quedaron… Cien correcciones humildes después, el equipo es formidable.

**🔧 Definición técnica:** ensemble secuencial: cada árbol ajusta los **residuos** (gradiente de la pérdida) del conjunto anterior; descenso de gradiente en el espacio de funciones. La tabla comparativa de las tres implementaciones dominantes:

| | XGBoost | LightGBM | CatBoost |
|---|---|---|---|
| Año | 2014 (Chen & Guestrin, 2016) | 2017 (Ke et al., 2017) | 2017 (Prokhorenkova et al., 2018) |
| Origen | DMLC (academia) | Microsoft | Yandex |
| Crecimiento del árbol | Level-wise por defecto (`grow_policy=depthwise`); leaf-wise opcional (`lossguide`, con `hist`/`approx`) | **Leaf-wise** (por hoja, más rápido y agresivo) | Symmetric trees (simétricos, más estables) |
| Velocidad de entrenamiento | Rápido | **Muy rápido** (leaf-wise + histogramas) | Moderado |
| Memoria | Moderada | Baja (formato columnar + histogramas) | Moderada-alta |
| Categóricas nativas | **Sí** desde 1.5 (`enable_categorical`); particiones óptimas desde 1.6 en `hist`/`approx` (`max_cat_to_onehot`) | **Sí**: split óptimo sobre las categorías codificadas como enteros (Fisher, 1958), que suele rendir mejor que one-hot; regularizar con `min_data_per_group`/`cat_smooth`; la alta cardinalidad es su punto débil | **Sí**, ordered target statistics (Prokhorenkova et al., 2018): el más robusto con muchas categorías |
| Datos faltantes | Manejo nativo | Manejo nativo | Manejo nativo |
| Regularización | L1 (alpha), L2 (lambda), gamma | L1, L2, min_child_samples | L2, bagging, random strength |
| Overfitting en datasets chicos | Puede overfit | **Más propenso** (leaf-wise) | Más robusto (symmetric) |
| GPU | Sí | Sí | Sí |
| Cuándo conviene | Referencia madura, ecosistema enorme | Datasets grandes, prioridad velocidad | Muchas categóricas, datasets chicos-medianos |

**Nota de vigencia (2026):** los tres boosters manejan categóricas y faltantes de forma nativa; la diferencia real está en el mecanismo — particiones por histograma en XGBoost/LightGBM frente a ordered target statistics en CatBoost — y en su robustez con alta cardinalidad (documentación oficial de XGBoost y LightGBM, consultada 2026-09-06). El encoding manual sigue siendo necesario para los modelos lineales y útil por trazabilidad ([[03-Preparacion-de-Datos]]). *(Corregido el 2026-09-06: la tabla decía que XGBoost «requiere encoding» y que el soporte de LightGBM era «parcial».)*

**La cuarta implementación: HistGradientBoosting (scikit-learn).** Desde la versión 0.21, scikit-learn incluye `HistGradientBoostingClassifier`/`Regressor`, inspirados en LightGBM: boosting por histogramas «órdenes de magnitud más rápido» que el `GradientBoosting` clásico a partir de decenas de miles de filas, con valores faltantes nativos, categóricas nativas (`categorical_features`, con detección automática desde el dtype del DataFrame desde 1.4) y early stopping activado por defecto sobre 10.000 muestras (documentación oficial de scikit-learn, consultada 2026-09-06). No pretende desplazar a las tres librerías especializadas en el extremo de rendimiento, pero es el boosting «de fábrica» dentro de los pipelines y la validación cruzada de scikit-learn, sin dependencias externas, y un challenger honesto antes de instalar una librería adicional.

**Hiperparámetros críticos comunes:** `n_estimators`/`num_boost_round`, `learning_rate` (eta: menor = mejor pero más árboles), `max_depth`/`num_leaves`, `subsample` (filas por árbol), `colsample_bytree` (columnas por árbol), `min_child_weight`/`min_data_in_leaf`. **Early stopping:** monitorear la métrica de validación y detener tras N rondas sin mejora — previene overfitting y fija `n_estimators` automáticamente ([[11-Mejora-de-Modelos]]).

**Ventajas:** estado del arte en tabular, nulls nativos, robustos. **Limitaciones:** más hiperparámetros que RF, secuencial (menos paralelizable que bagging), riesgo de overfit sin regularización. **Requisitos:** NO requieren escalado.

**🧭 Cuándo usarlo:** el candidato a campeón en casi cualquier problema tabular serio (competencias, scoring, demanda). Regla práctica: baseline logístico → RF → HistGradientBoosting → XGBoost/LightGBM/CatBoost tuneado.

**¿Y las redes neuronales?** La ventaja del boosting en tabular no es folclore de competencias: en benchmarks controlados, los modelos basados en árboles siguen siendo estado del arte en datos de tamaño medio (~10K filas) incluso sin contar su velocidad (Grinsztajn et al., 2022), y la comparación a gran escala de McElfresh et al. (2023) muestra que el margen depende de las propiedades del dataset más que de una superioridad absoluta. Este tomo se queda con la regla operativa — boosting como campeón por defecto en tabular —; el debate completo, incluidos los tabular foundation models y los MLP pre-afinados que hoy le compiten, está en [[23-Tabular-DL-vs-Boosting]].

**👔 En una frase para el negocio:** la tecnología que gana las competencias mundiales de datos tabulares — máxima precisión por peso invertido en cómputo, con la explicabilidad delegada a SHAP ([[13-MLOps-XAI-Etica]]).

### Naive Bayes

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un detective ingenuo pero veloz: asume que cada pista es independiente de las demás (que la huella no tiene nada que ver con el testigo), multiplica la fuerza de todas las pistas y da su veredicto. Su ingenuidad es falsa casi siempre — y aun así, con miles de pistas débiles (palabras de un email), acierta muchísimo.

**🔧 Definición técnica:** aplica Bayes ([[02-Fundamentos-Matematicos]]) con independencia condicional entre features dado el target: `P(y|x) ∝ P(y) · Πᵢ P(xᵢ|y)`. Variantes según la distribución asumida:

| Variante | Distribución asumida | Cuándo usar |
|---|---|---|
| GaussianNB | Normal por feature continua | Features continuas ~normales |
| MultinomialNB | Multinomial (conteos) | Conteos de palabras (TF), clasificación de texto |
| BernoulliNB | Bernoulli (binarias) | Presencia/ausencia de palabras |
| ComplementNB | Complemento de Multinomial | Texto con clases desbalanceadas |
| CategoricalNB | Categórica discreta | Features categóricas codificadas |

**Laplace smoothing (`alpha=1`):** si una feature nunca apareció en una clase durante train, P = 0 colapsaría todo el producto; sumar 1 a los conteos lo evita.

**Ventajas:** entrena en milisegundos, escala a millones de features, decente con poca data. **Limitaciones:** probabilidades mal calibradas (sobreconfiadas — calibrar si se usan, [[11-Mejora-de-Modelos]]); la independencia asumida limita el techo. **Requisitos:** NO requiere escalado.

**🧭 Cuándo usarlo:** baseline instantáneo de texto, filtros de spam, sistemas con restricción extrema de latencia/cómputo.

**👔 En una frase para el negocio:** el modelo más barato de operar del catálogo — imbatible en costo por predicción cuando el problema lo tolera.

---

## 2. Modelos de Regresión

Audiencia: 🔧 🧭

### Regresión Lineal (OLS)

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Estirar un hilo tenso entre una nube de puntos de modo que quede lo más cerca posible de todos a la vez: el hilo es la recta de mínimos cuadrados, y su inclinación te dice cuánto sube y con qué fuerza cada factor.

**🔧 Definición técnica:** minimiza la suma de residuos al cuadrado; solución cerrada `β = (XᵀX)⁻¹Xᵀy` (numéricamente vía QR, [[02-Fundamentos-Matematicos]]). **Supuestos de Gauss-Markov:** (1) linealidad `E[y] = Xβ`; (2) exogeneidad estricta `E[ε|X] = 0`; (3) homocedasticidad `Var[ε|X] = σ²I`; (4) sin multicolinealidad perfecta; (5) normalidad de errores — necesaria para inferencia (tests, IC), no para estimar. **Diagnóstico:** residuos vs fitted (homocedasticidad), Q-Q plot (normalidad), VIF ([[04-EDA]]), Durbin-Watson (autocorrelación).

**🧭 Cuándo usarlo:** relaciones aproximadamente lineales, necesidad de inferencia sobre coeficientes ("¿cuánto aporta cada factor?"), baselines de regresión. **Requisitos:** escalar para comparar coeficientes; sensible a outliers ([[03-Preparacion-de-Datos]]).

**👔 En una frase para el negocio:** el estándar para cuantificar "cuánto mueve la aguja cada variable" con respaldo estadístico formal.

### Ridge, Lasso y ElasticNet

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía del estudiante (la clave de la regularización)
> Un estudiante que memoriza las respuestas del examen pasado (overfitting) fracasa en el siguiente. La regularización le pone un peso en la mochila que lo obliga a simplificar: **L2 (Ridge)** le dice "puedes estudiar todos los temas, pero sin obsesionarte con ninguno" (encoge todos los coeficientes sin eliminarlos). **L1 (Lasso)** le dice "elige solo los 5 conceptos que de verdad importan y olvida el resto" (lleva coeficientes exactamente a cero — selección de features). **ElasticNet** combina ambos consejos: "estudia lo esencial con moderación y descarta lo claramente inútil".

| Modelo | Penalización | Efecto sobre coeficientes | Cuándo usar | Consideraciones |
|---|---|---|---|---|
| Ridge (L2) | `λ·Σβᵢ²` | Encoge hacia 0, nunca exactamente 0; reparte peso entre features correlacionadas | Multicolinealidad; cuando todas las features aportan algo | `β_ridge = (XᵀX + λI)⁻¹Xᵀy` — siempre soluble incluso con multicolinealidad. Escalar siempre |
| Lasso (L1) | `λ·Σ│βᵢ│` | Lleva coeficientes exactamente a 0 → **selección automática de features** | Se espera que pocas features importen ([[03-Preparacion-de-Datos]]) | Inestable con grupos de features muy correlacionadas (elige una arbitrariamente); sin forma cerrada (coordinate descent) |
| ElasticNet (L1+L2) | `λ₁·Σ│βᵢ│ + λ₂·Σβᵢ²` | Dispersión de Lasso + estabilidad de Ridge; agrupa features correlacionadas | Features correlacionadas + necesidad de selección; el default seguro | Dos hiperparámetros: `alpha` (magnitud total) y `l1_ratio` (mezcla: 0 = Ridge, 1 = Lasso) |

**Selección de λ:** cross-validation con `RidgeCV`, `LassoCV`, `ElasticNetCV` ([[10-Validacion-y-Leakage]]). La conexión geométrica con las normas L1/L2 está en [[02-Fundamentos-Matematicos]]; la vista general de regularización, en [[11-Mejora-de-Modelos]].

**👔 En una frase para el negocio:** el seguro anti-memorización de los modelos lineales — cambia un poco de ajuste al pasado por mucha más confiabilidad en el futuro.

### Support Vector Regression (SVR)

Audiencia: 🔧

> [!tip] 💡 Analogía
> Un tubo de goma de radio ε alrededor de la tendencia: los puntos **dentro** del tubo no molestan (error tolerado); solo los que se salen del tubo tiran de la función. El modelo se concentra en los casos que de verdad se desvían.

**🔧 Definición técnica:** busca la función que se desvía a lo más ε del target (epsilon-insensitive loss); solo los puntos fuera del tubo son vectores de soporte. Kernel trick para no-linealidad. Parámetros: `C` (regularización), `epsilon` (ancho del tubo), `kernel`. **Requisitos:** escalado obligatorio; mismo problema de escala O(N²) que el SVC con kernel (y las mismas salidas: `LinearSVR` o aproximación del kernel).

**🧭 Cuándo usarlo:** regresión no lineal en datasets chicos-medianos, cuando errores pequeños dan lo mismo y los grandes importan.

### Polynomial Regression

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Cambiar la regla rígida por una regla flexible que se curva. El peligro: con demasiada flexibilidad, la regla se retuerce para pasar por cada punto — incluido el ruido — y pierde toda capacidad de predecir el siguiente.

**🔧 Definición técnica:** `PolynomialFeatures(degree=k)` genera todas las potencias e interacciones hasta grado k, y se ajusta un modelo lineal encima. **Riesgo de explosión dimensional:** grado 2 con D features → D(D+1)/2 + D + 1 columnas; grado 3 con 10 features → 286 features. Overfitting rápido: **usar siempre con regularización** (Ridge/Lasso) y validación estricta.

**🧭 Cuándo usarlo:** curvaturas suaves y conocidas (grado 2–3, pocas features); más allá, árboles o boosting capturan no-linealidades sin la explosión.

### Árboles y boosting de regresión

Audiencia: 🔧

**🔧 Definición técnica:** los mismos Decision Tree, Random Forest, ExtraTrees y XGBoost/LightGBM/CatBoost de la sección 1 tienen versión de regresión: predicen el **promedio del target en la hoja** y optimizan MSE/MAE (o pérdidas robustas como Huber, [[08-Metricas-de-Evaluacion]]). Mismos trade-offs, misma inmunidad a la escala, mismo warning: los árboles **no extrapolan** fuera del rango visto en train (un RF nunca predecirá una demanda mayor a la máxima histórica).

---

## 3. Detección de Anomalías

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía general: el concierto
> Un concierto con 10.000 personas de pie aplaudiendo. Detectar anomalías es encontrar a la persona sentada mirando al techo. **Isolation Forest** la encuentra porque es fácil de *aislar* con pocos cortes ("¿está sentada? ¿mira al techo?" — dos preguntas y ya). **LOF** la encuentra porque su *densidad local* no calza con la de sus vecinos. **One-Class SVM** dibuja un círculo alrededor de "lo normal"; lo que queda fuera es raro. **Elliptic Envelope** asume que lo normal es un óvalo gaussiano y mira quién quedó fuera. **Autoencoder** aprendió a dibujar espectadores típicos — y a esa persona no logra dibujarla bien (alto error de reconstrucción).

| Algoritmo | Mecanismo | Parámetros clave | Cuándo usar | Limitaciones |
|---|---|---|---|---|
| Isolation Forest | Árboles con splits aleatorios; los outliers quedan aislados con caminos cortos → score de anomalía | `contamination` (proporción esperada), `n_estimators`, `max_samples` | Anomalías globales en alta dimensión; muy escalable; fraude, fallas (Liu et al., 2008) | Débil en outliers locales/contextuales; clusters densos de outliers lo confunden |
| Local Outlier Factor (LOF) | Compara densidad local del punto vs sus K vecinos; LOF ≫ 1 = mucho menos denso que su vecindario | `n_neighbors`, `metric`, `contamination` | Anomalías **locales**; datos con densidades heterogéneas (Breunig et al., 2000) | Sin `predict()` para datos nuevos (salvo modo novelty); lento con N grande |
| One-Class SVM | Aprende la frontera del territorio "normal" en un kernel space; lo exterior es anómalo | `nu` (cota de fracción de outliers), `kernel`, `gamma` | Alta dimensión, texto/imagen; solo ejemplos normales disponibles | Muy lento con N grande; sensible a escala; difícil de tunear |
| Elliptic Envelope | Ajusta gaussiana multivariada robusta (MCD) y marca las colas | `contamination`, `support_fraction` | Datos normales ~gaussianos multivariados | Solo anomalías globales en mundos gaussianos |
| Autoencoder | Red entrenada para reconstruir lo normal; error de reconstrucción alto = anomalía | Arquitectura, pérdida (MSE), umbral del error | Alta dimensión, imágenes, secuencias; mucha data normal disponible ([[12-Deep-Learning]]) | Requiere diseñar y entrenar la red; el umbral se define aparte |

**🧭 Cuándo usarlo:** cuando los casos malos son rarísimos o no etiquetados (fraude nuevo, fallas inéditas, intrusiones). Si tienes suficientes etiquetas de la clase rara, un clasificador supervisado con manejo de desbalance ([[03-Preparacion-de-Datos]]) suele ganar. Nota: estos mismos métodos aparecen como detectores de outliers en la preparación de datos ([[03-Preparacion-de-Datos]]) — allá limpian, acá **son el producto**.

**👔 En una frase para el negocio:** los vigilantes de lo inesperado: encuentran el fraude que nadie había visto antes, la falla que ningún manual describe — sin necesitar ejemplos previos de cada modalidad.

---

## 4. Guía de selección — resumen operativo

Audiencia: 🧭 👔

| Modelo | ¿Escalado? | ¿Nulls nativos? | ¿Categóricas nativas? | Interpretabilidad | Fuerza principal |
|---|---|---|---|---|---|
| Regresión Logística / OLS | Sí | No | No (encoding) | ⭐⭐⭐⭐⭐ | Baseline explicable y rápido |
| KNN | Sí (obligatorio) | No | No | ⭐⭐⭐ | Similitud directa, cero entrenamiento |
| SVM / SVR | Sí (obligatorio) | No | No | ⭐⭐ | Alta dimensión con pocos datos |
| Decision Tree | No | Sí (sklearn ≥ 1.3) | No | ⭐⭐⭐⭐⭐ | Reglas visibles |
| Random Forest / ExtraTrees | No | Sí (sklearn ≥ 1.4 / ≥ 1.6) | No | ⭐⭐⭐ | Robustez sin tuning |
| XGBoost / LightGBM / CatBoost | No | **Sí** | **Sí** (los tres; CatBoost el más robusto con alta cardinalidad) | ⭐⭐⭐ (con SHAP) | Estado del arte tabular |
| HistGradientBoosting (sklearn) | No | Sí | Sí (`categorical_features`) | ⭐⭐⭐ (con SHAP) | Boosting por histogramas sin dependencias externas |
| Naive Bayes | No | No | CategoricalNB | ⭐⭐⭐⭐ | Velocidad extrema, texto |
| Ridge / Lasso / ElasticNet | Sí | No | No | ⭐⭐⭐⭐⭐ | Linealidad regularizada + selección |

```
 ¿Tabular clásico? ──► baseline LogReg/OLS ──► RF ──► HistGradientBoosting ──► XGB/LGBM/CatBoost tuneado
 ¿Tabular y dudas si DL o foundation models compiten? ──► Tomo 23 (Tabular DL vs Boosting)
 ¿Texto disperso?  ──► Naive Bayes / LogReg + TF-IDF ──► (si no basta) DL [[12-Deep-Learning]]
 ¿Imagen/audio/secuencia? ──► directo a [[12-Deep-Learning]] (CNN/RNN/Transformers)
 ¿Sin etiquetas y buscando lo raro? ──► sección 3 (anomalías)
 ¿Regulador mirando? ──► campeón explicable + challenger potente con XAI [[13-MLOps-XAI-Etica]]
```

---

## 📖 Referencias de este tomo

- (Cover & Hart, 1967) — KNN. · (Cortes & Vapnik, 1995) — SVM. · (Breiman, 2001a) — Random Forests.
- (Chen & Guestrin, 2016) — XGBoost. · (Ke et al., 2017) — LightGBM. · (Prokhorenkova et al., 2018) — CatBoost.
- (Fisher, 1958) — el agrupamiento óptimo de categorías en que se apoya LightGBM. · (Grinsztajn et al., 2022) y (McElfresh et al., 2023) — árboles vs. redes en tabular ([[23-Tabular-DL-vs-Boosting]]).
- Documentación oficial consultada el 2026-09-06 (scikit-learn: valores faltantes en árboles y HistGradientBoosting; XGBoost: categóricas y `grow_policy`; LightGBM: categóricas y faltantes) → [[16-Bibliografia]] §13.
- (Liu et al., 2008) — Isolation Forest. · (Breunig et al., 2000) — LOF.
- (Hastie et al., 2009), (James et al., 2021), (Géron, 2022) — tratamiento integral de los modelos supervisados.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[06-Clustering|06 · Clustering]] · Siguiente: [[08-Metricas-de-Evaluacion|08 · Métricas de Evaluación ➡]]

> **Próximo tomo:** [[08-Metricas-de-Evaluacion]] — el termómetro del rendimiento: matriz de confusión, todas las métricas de clasificación y regresión, curvas ROC/PR, calibración y cómo elegir el umbral que maximiza el resultado de negocio.

