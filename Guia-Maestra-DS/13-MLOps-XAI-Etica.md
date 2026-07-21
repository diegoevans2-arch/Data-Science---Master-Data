---
title: "Tomo 13 — MLOps, XAI y Ética"
tags: [data-science, machine-learning, mlops, xai, fairness, etica]
audiencias: [tecnico, puente, ejecutivo]
tomo: 13
version: 6.1
updated: 2026-07-19
---

# 🏭 Tomo 13 — MLOps, XAI y Ética

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[12-Deep-Learning|12 · Deep Learning]] · Siguiente: [[14-Anexo-Interpretar-Resultados|14 · Anexo: Interpretar Resultados ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Un modelo que no se puede explicar, reproducir, monitorear y actualizar **no tiene valor real**: es un experimento con buena prensa. Aquí viven las prácticas transversales que convierten el experimento en sistema — y las que evitan que ese sistema discrimine o se degrade en silencio. El código de ML es una pieza pequeña rodeada de infraestructura (Sculley et al., 2015): este tomo trata de todo lo que rodea al modelo.

> [!abstract] 👔 Impacto ejecutivo
> Aquí se juega la diferencia entre "hicimos un piloto de IA" y "operamos IA con control".
>
> - **Decisiones que habilita:** aprobar modelos ante reguladores con explicaciones defendibles, detectar degradación antes de que cueste dinero, auditar equidad antes del escándalo.
> - **Costo de hacerlo mal:** modelos irreproducibles ("funcionaba la semana pasada"), drift silencioso decidiendo mal durante meses, sesgos automatizados con costo legal y reputacional.
> - **Pregunta ejecutiva que responde:** *¿podemos explicar, repetir, vigilar y defender cada decisión que este sistema toma?*

---

## 1. Interpretabilidad y Explicabilidad (XAI)

Audiencia: 🔧 🧭 👔

> [!info] 📌 Por qué importa
> La interpretabilidad no es un lujo: en finanzas, salud y sectores regulados es **requisito legal** (GDPR y equivalentes), y en todos lados es la herramienta para detectar sesgos, leakage ([[10-Validacion-y-Leakage]]) y errores sistemáticos. Existe un espectro: modelos intrínsecamente interpretables (lineales, árboles) ↔ cajas negras con explicabilidad post-hoc (Molnar, 2022).

> [!tip] 💡 Analogías prácticas de XAI (con su métrica y rangos)
> **Feature Importance:** el ranking de ingredientes por cuánto sabor aportan — genera importancias 0–1 que suman 1; > 10% en una sola feature merece atención (¿y si es leakage?). **Permutation Importance:** barajar una columna y ver cuánto cae el AUC — caída > 0.05 = feature importante. **PDP:** "si subo los m² de 50 a 200, ¿qué pasa con el precio predicho?" — revela la **forma** de la relación (lineal, meseta, umbral). **SHAP:** descompone CADA predicción en la contribución exacta de cada feature — │SHAP│ > 0.1 (en la escala del output) = contribución relevante. **LIME:** un mini-modelo lineal que explica UNA predicción en su vecindario.

### Métodos globales (explican el modelo completo)

| Método | Mecanismo | Qué métrica genera / rango válido | Fortalezas | Cuidados |
|---|---|---|---|---|
| Feature Importance (MDI) | Reducción media de impureza por feature en los árboles | Importancias normalizadas 0–1 (suman 1) | Gratis en RF/boosting | Sesgada hacia alta cardinalidad y continuas ([[07-Modelos-Supervisados]]) |
| Permutation Importance | Caída de la métrica al permutar cada feature en validación | Δmétrica por feature (ej. ΔAUC) | Model-agnostic, más honesta que MDI | Features correlacionadas se reparten el crédito; costosa |
| PDP (Partial Dependence Plot) | Efecto marginal promedio de 1–2 features sobre la predicción | Curva de respuesta | Revela forma y dirección del efecto | Promedia: oculta heterogeneidad; asume independencia |
| ICE (Individual Conditional Expectation) | El PDP trazado para **cada** instancia | Familia de curvas individuales | Destapa subgrupos con efectos opuestos | Ilegible sin muestreo |
| SHAP | Valores de Shapley: reparto justo de `f(x) − E[f(x)]` entre features (Lundberg & Lee, 2017) | Contribución aditiva por feature y predicción | Propiedades formales: efficiency, symmetry, dummy, additivity; **TreeExplainer** lo hace tratable en árboles (O(TLD²)); summary plot (global) y dependence plot (interacciones) | Costoso fuera de árboles; explicar ≠ causalidad |

### Métodos locales (explican UNA predicción)

- **LIME:** (Ribeiro et al., 2016) perturba la instancia, obtiene predicciones del modelo negro y ajusta un modelo lineal ponderado por cercanía — la explicación es ese modelo local. Rápido e intuitivo; inestable si el vecindario está mal definido.
- **SHAP local / force plot:** los valores SHAP de una instancia muestran qué features empujaron su predicción sobre o bajo el valor base — el estándar para "¿por qué rechazaron MI crédito?".
- **Counterfactuals (DiCE):** "¿qué debería cambiar esta instancia para cambiar la predicción?" — accionable por diseño: "con 6 meses más de antigüedad y deuda bajo X, el crédito se aprueba".

**👔 En una frase para el negocio:** la explicabilidad convierte al modelo de oráculo en colega: sus razones se pueden auditar, discutir y defender ante el cliente y el regulador.

---

## 2. Reproducibilidad

Audiencia: 🔧 🧭

> [!info] 📌 Descriptor
> Reproducibilidad = obtener **exactamente** los mismos resultados con los mismos datos, código y configuración. En ML es difícil: semillas aleatorias, versiones de librerías, orden de procesamiento y hardware conspiran en contra.

> [!tip] 💡 Analogía de la receta
> "Agregué un poco de sal y cociné un rato" no es una receta: nadie puede replicar tu plato. Reproducibilidad es escribir "5 g de sal, 12 minutos a 180 °C en horno de convección" — y que tu colega, siguiéndola, obtenga el mismo plato. Si tu mejor modelo no es una receta exacta, no es un activo: es una anécdota.

| Práctica | Cómo se implementa |
|---|---|
| Semillas aleatorias | Fijar TODAS: `random.seed(42)`, `np.random.seed(42)`, `torch.manual_seed(42)` + `torch.cuda.manual_seed_all(42)`, `tf.random.set_seed(42)`, `random_state=42` en sklearn. Documentarlas |
| Entorno reproducible | `requirements.txt` con versiones exactas (pip freeze), `environment.yml` (conda), imagen Docker versionada, hash del commit de Git |
| DVC (Data Version Control) | Git para datos: versiona datasets en S3/GCS/Azure; `dvc.yaml` define el pipeline; `dvc repro` lo reproduce completo; `dvc diff` compara versiones |
| MLflow Tracking | `log_param()`, `log_metric()`, `log_artifact()` por experimento; UI para comparar corridas; Model Registry para versionar modelos |
| Weights & Biases | Alternativa con UI superior: Sweeps (búsqueda distribuida de hiperparámetros), Reports compartibles, Artifacts versionados; integración nativa con PyTorch/Keras/HF |

---

## 3. MLOps — de experimento a sistema

Audiencia: 🔧 🧭 👔

### 3.1 Pipelines de ML

- **sklearn Pipeline:** `Pipeline([('scaler', StandardScaler()), ('model', RFC())])` — encadena preprocesamiento + modelo; `fit()` solo toca train; exportable como objeto único; la vacuna anti-leakage ([[05-Escalado-de-Datos]], [[10-Validacion-y-Leakage]]).
- **ColumnTransformer:** transformaciones distintas por subconjunto de columnas (numéricas → StandardScaler; categóricas → OneHotEncoder) integradas al Pipeline.
- **FunctionTransformer:** convierte cualquier función Python en un paso de Pipeline.

### 3.2 Serialización y serving

| Herramienta | Qué hace | Nota clave |
|---|---|---|
| pickle | Serialización nativa de Python | ⚠️ Inseguro con fuentes no confiables; frágil entre versiones |
| joblib | Pickle optimizado para arrays numpy | El estándar para modelos sklearn: `joblib.dump/load` |
| ONNX | Formato de intercambio entre frameworks | `torch.onnx.export()` → ONNX Runtime / TensorRT / OpenVINO; inferencia optimizada ([[12-Deep-Learning]]) |
| FastAPI + Docker | El patrón artesanal estándar: cargar el modelo al startup, endpoint POST /predict, contenedor portable | Control total; tú operas el servicio |
| BentoML | Framework de serving especializado: runners autoescalables, batching para alto throughput | Menos código de infraestructura |
| MLflow serve | `mlflow models serve` expone el modelo como API REST | Rápido de montar desde el registry |

### 3.3 Monitoreo de drift en producción

> [!tip] 💡 Analogía
> El modelo es un mapa; el mundo, el territorio. **Data drift:** el territorio cambió de aspecto (llegan clientes distintos a los del mapa). **Concept drift:** cambiaron las reglas del territorio (los mismos clientes ahora se comportan distinto — pre y post pandemia). **Performance drift:** el GPS empieza a equivocarse y lo notas… si tienes contra qué comparar. Un mapa sin actualizaciones es una promesa de perderse.

| Tipo de drift | Qué cambia | Cómo se detecta |
|---|---|---|
| Data drift (feature drift) | La distribución de las features de entrada | Comparar distribuciones train vs producción: KL divergence, Wasserstein, Jensen-Shannon |
| Concept drift | La relación features → target (el mundo cambió) | Degradación con features estables; análisis por segmento |
| Performance drift | La métrica del modelo en producción | Requiere ground truth (a veces retrasado): monitorear cuando llegue |

- **Evidently AI:** open-source; reports y dashboards de data/target drift y calidad de datos.
- **WhyLabs:** SaaS de monitoreo casi en tiempo real con alertas automáticas.
- **Estrategias de reentrenamiento:** programado (cada N semanas), por trigger (métrica bajo umbral / drift detectado), continuo (online learning). La elección depende de la velocidad del drift y del costo de reentrenar.

**👔 En una frase para el negocio:** un modelo sin monitoreo es deuda técnica acumulándose en silencio — el drift no avisa, solo cobra.

> [!example] 📊 Caso de negocio — Manufactura: el modelo de mantenimiento que se apagó sin avisar
> **Problema:** una planta manufacturera despliega un modelo de **mantenimiento predictivo** que anticipa fallas de maquinaria con semanas de antelación a partir de sensores de vibración y temperatura. Durante seis meses funciona: caen las paradas no planificadas, el ahorro es medible. Luego las fallas sorpresivas regresan — y nadie entiende por qué. El modelo **sigue "funcionando"**: recibe datos, devuelve predicciones, no arroja ningún error. Simplemente se equivoca, en silencio, durante semanas.
>
> **Técnica aplicada:** monitoreo de drift retroactivo (sección 3.3). Se descubre **data drift**: se recalibró un lote de sensores y se incorporó una línea nueva con firmware distinto, de modo que la distribución de las features de entrada se **desplazó** respecto a la de entrenamiento. El modelo, sin reentrenar, extrapolaba sobre datos que ya no se parecían a los que vio. Se instrumenta detección de drift (Jensen-Shannon y Wasserstein sobre las features clave con **Evidently AI**), alertas por umbral, y un **trigger de reentrenamiento** atado a esa señal. Como la degradación había sido invisible por falta de trazabilidad, se versionan además datos (**DVC**) y modelo (**MLflow**) para poder reconstruir qué versión decidía qué en cada momento (sección 2).
>
> **Resultado:** el drift ahora se detecta en **días, no en trimestres**; el reentrenamiento se dispara solo cuando la distribución se desvía; y cada predicción es trazable a una versión reproducible. El costo evitado no es el del modelo — es el de las paradas de línea que volvieron a ocurrir mientras nadie miraba. La lección: **un modelo sin monitoreo no falla ruidosamente; se degrada en silencio y cobra la cuenta después.** El deployment no es la meta, es el kilómetro cero.

---

## 4. Ética en IA y Fairness

Audiencia: 🔧 🧭 👔

> [!info] 📌 Descriptor: ¿qué es fairness?
> Fairness (equidad) es el principio de que un modelo debe tratar a las personas de manera justa, sin discriminar por características protegidas (género, etnia, edad, nacionalidad). Un modelo puede ser preciso y a la vez éticamente inaceptable: precisión y justicia son ejes distintos, y la segunda **se audita, no se supone**.

> [!tip] 💡 Analogía del juez robot
> Un juez robot aprende de miles de sentencias históricas. Si los jueces humanos fueron sistemáticamente más duros con un grupo, el robot replica y **amplifica** esa dureza — sin odiar a nadie: los datos ya venían torcidos. Las métricas de fairness son las auditorías que verifican si el robot trata a todos los grupos con la misma vara. Y las tres varas principales: **Demographic Parity** — "las becas se reparten en igual proporción entre escuelas"; **Equal Opportunity** — "si eres buen estudiante, la beca te llega sin importar tu escuela"; **Equalized Odds** — "ni te dan beca sin merecerla, ni te la niegan mereciéndola".

| Concepto de fairness | Definición | 💡 Analogía | Cuándo usar |
|---|---|---|---|
| Demographic Parity | Igual proporción de predicciones positivas entre grupos: `P(Ŷ=1│A=0) = P(Ŷ=1│A=1)` | Becas repartidas por igual entre escuelas | El outcome debe distribuirse parejo, independiente del mérito relativo |
| Equal Opportunity | Igual TPR (recall) entre grupos: `P(Ŷ=1│Y=1, A=a)` constante | Todo buen estudiante recibe beca, venga de donde venga | No discriminar en la detección de positivos reales (créditos a solventes) |
| Equalized Odds | TPR **y** FPR iguales entre grupos | Ni beca inmerecida ni beca negada al que la merece | La vara más estricta; difícil de lograr con alta accuracy simultánea |
| Predictive Parity | Igual precision (PPV) entre grupos | Si el robot dice "becado", acierta igual en toda escuela | El costo del falso positivo debe repartirse parejo |
| Individual Fairness | Individuos similares → predicciones similares | Dos gemelos académicos reciben el mismo veredicto | Cuando la justicia se define caso a caso, no por grupo |

- **Fairlearn** (Microsoft): `MetricFrame` para métricas por grupo; mitigación por reductions (optimización con restricciones) y post-processing (umbrales por grupo).
- **AIF360** (IBM): 70+ métricas de fairness y 11 algoritmos de mitigación (pre/in/post-processing).
- **Privacidad diferencial:** publicar estadísticos sin exponer individuos, agregando ruido calibrado para garantizar (ε,δ)-privacy (Dwork et al., 2006); usada por Google y Apple en telemetría; OpenDP/SmartNoise para ML.

> [!danger] 🚨 Error costoso: "el algoritmo es neutro"
> Ningún modelo entrenado con historia humana es neutro por defecto. Si decide sobre personas (crédito, contratación, salud, justicia), la auditoría de fairness por grupo protegido es parte del checklist de salida a producción — no un anexo voluntario (Barocas et al., 2019).

---

## 5. Decálogo de Mejores Prácticas

Audiencia: 🔧 🧭 👔

1. **Baseline antes que optimización:** un modelo simple (logística, árbol) define el piso; la complejidad debe ganarse su lugar.
2. **Semillas fijadas desde el día uno:** la reproducibilidad es respeto por el tiempo del equipo.
3. **Pipeline antes que el primer modelo:** nunca procesar datos fuera del pipeline ([[10-Validacion-y-Leakage]]).
4. **El EDA no es opcional:** entrenar sin explorar es conducir a ciegas ([[04-EDA]]); una correlación espuria detectada a tiempo ahorra semanas.
5. **Nunca fitear transformaciones con test:** ni el scaler, ni el imputer, ni el encoder. Nunca ([[05-Escalado-de-Datos]]).
6. **La métrica se alinea al costo real del error**, no a lo fácil de reportar ([[08-Metricas-de-Evaluacion]]).
7. **Cross-validation estratificada como estándar:** un split único es estadísticamente frágil ([[10-Validacion-y-Leakage]]).
8. **Diagnóstico antes que tuning:** learning curves primero; tunear un modelo con underfitting es tiempo perdido ([[11-Mejora-de-Modelos]]).
9. **Versionar datos (DVC), código (Git) y modelos (MLflow/W&B):** "el modelo que funcionó la semana pasada" debe poder reconstruirse.
10. **Monitorear drift desde el día 1 del deployment:** el mundo cambia; los modelos estáticos se degradan; el monitoreo es parte del producto, no un extra.

---

## 6. Ecosistema de Herramientas — referencia rápida

Audiencia: 🔧 🧭

| Categoría | Herramientas | Uso principal |
|---|---|---|
| ML clásico | scikit-learn, statsmodels, scipy | Modelos, preprocesamiento, estadística |
| Gradient Boosting | XGBoost, LightGBM, CatBoost | Estado del arte tabular ([[07-Modelos-Supervisados]]) |
| Deep Learning | PyTorch, TensorFlow/Keras, JAX, FastAI | Redes neuronales ([[12-Deep-Learning]]) |
| NLP / LLMs | Hugging Face Transformers, NLTK, spaCy, LangChain | Modelos de lenguaje y pipelines de LLM |
| Datos y EDA | pandas, polars, numpy, ydata-profiling, sweetviz, dtale, missingno | Manipulación y exploración ([[04-EDA]]) |
| Visualización | matplotlib, seaborn, plotly, altair, bokeh | Gráficos estáticos e interactivos |
| Feature engineering | feature-engine, featuretools, tsfresh | Construcción automática de features ([[03-Preparacion-de-Datos]]) |
| Desbalance | imbalanced-learn (SMOTE, ADASYN…) | Manejo de clases desbalanceadas |
| Hiperparámetros | Optuna, Hyperopt, Ray Tune, Keras Tuner | Optimización bayesiana y distribuida ([[11-Mejora-de-Modelos]]) |
| Interpretabilidad | SHAP, LIME, ELI5, Alibi, InterpretML | Explicabilidad global y local |
| MLOps / Tracking | MLflow, Weights & Biases, DVC, Neptune | Experimentos, versionado, registro |
| Serving / Deploy | FastAPI, BentoML, Seldon, ONNX Runtime, TorchServe | Modelos en producción |
| Monitoreo | Evidently AI, WhyLabs, Arize, Fiddler | Drift y degradación |
| Fairness | Fairlearn, AIF360, What-If Tool | Auditoría y mitigación de sesgos |
| Cloud ML | AWS SageMaker, Google Vertex AI, Azure ML | Pipelines end-to-end gestionados |
| Anomaly detection | PyOD, alibi-detect | Outliers y anomalías ([[07-Modelos-Supervisados]]) |
| Association rules | mlxtend, PyFIM | Market basket ([[09-Reglas-de-Asociacion]]) |

---

## 📖 Referencias de este tomo

- (Lundberg & Lee, 2017) — SHAP. · (Ribeiro et al., 2016) — LIME.
- (Molnar, 2022) — *Interpretable Machine Learning* (libro abierto de referencia).
- (Sculley et al., 2015) — deuda técnica oculta en sistemas de ML.
- (Dwork et al., 2006) — privacidad diferencial. · (Barocas et al., 2019) — fairness y ML.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[12-Deep-Learning|12 · Deep Learning]] · Siguiente: [[14-Anexo-Interpretar-Resultados|14 · Anexo: Interpretar Resultados ➡]]

> **Próximo tomo:** [[14-Anexo-Interpretar-Resultados]] — el anexo para no técnicos: cómo leer matriz de confusión, ROC, feature importance y dashboards de drift, con las preguntas que todo ejecutivo debería hacer.
