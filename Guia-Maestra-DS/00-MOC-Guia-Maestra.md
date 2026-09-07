---
title: "Tomo 00 — MOC · Guía Maestra de Data Science"
tags: [data-science, machine-learning, moc, indice]
audiencias: [tecnico, puente, ejecutivo]
tomo: 00
version: 6.5
updated: 2026-09-06
type: vault-index
---

# 🗺️ Guía Maestra de Data Science v6 — Índice Maestro (MOC)

Audiencia: 🔧 🧭 👔

> [!info] 📌 ¿Qué es esta guía?
> Un recorrido completo por los fundamentos, pilares, técnicas, modelos, métricas y prácticas del **Data Science** y **Machine Learning**, diseñado para funcionar simultáneamente como **manual técnico de referencia** (fórmulas, hiperparámetros, supuestos, limitaciones) y como **material de divulgación para audiencias de negocio** (impacto en decisiones, costo de hacerlo mal, traducción negocio↔técnica).
>
> Cada concepto de la guía responde tres preguntas:
>
> 1. **¿Qué es?** — el cómo técnico 🔧
> 2. **¿Para qué sirve?** — el objetivo 🧭
> 3. **¿Por qué debería importarme?** — el impacto 👔

> [!abstract] 👔 Impacto ejecutivo
> Esta guía convierte el Data Science en un lenguaje compartido entre negocio y equipo técnico: todos leen el mismo documento, cada uno en su capa.
>
> - **Decisiones que habilita:** interpretar resultados de modelos sin intermediarios, priorizar inversiones en datos, auditar proyectos de ML antes de aprobarlos.
> - **Costo de no tenerla:** métricas mal interpretadas, proyectos aprobados sin criterios de validación y dependencia total de la traducción de terceros.
> - **Pregunta que responde:** *¿qué necesita entender cada rol de mi organización para que los proyectos de datos generen valor real?*

---

## 🧭 Cómo usar esta guía

**El lector decide qué leer.** Ningún perfil necesita leer un capítulo completo: cada bloque está etiquetado visualmente para que sepas de inmediato qué te corresponde.

### Sistema de etiquetado por audiencia

| Etiqueta           | Perfil                                       | Qué contiene                                                                              |
| ------------------ | -------------------------------------------- | ----------------------------------------------------------------------------------------- |
| 🔧 **[Técnico]**   | Data Scientist / Data Engineer / ML Engineer | Definiciones formales, fórmulas, hiperparámetros, supuestos, implementación, limitaciones |
| 🧭 **[Puente]**    | Product Owner / Líder técnico / Analista     | Traducción negocio↔técnica, cuándo usar qué, trade-offs de decisión                       |
| 👔 **[Ejecutivo]** | Gerente / C-level / Stakeholder              | Impacto en el negocio, decisiones que habilita, riesgos, costo de hacerlo mal             |
| 💡 **[Analogía]**  | Todos                                        | Explicación intuitiva con analogía cotidiana. Sin prerrequisitos                          |

Toda sección o concepto relevante abre con una línea del tipo `Audiencia: 🔧 🧭 👔` indicando a qué perfiles aplica.

### Código de callouts

| Callout | Emoji | Significado |
|---|---|---|
| `[!tip]` | 💡 | Analogía cotidiana para entender la intuición |
| `[!info]` | 📌 | Por qué importa esta sección en el ciclo de vida |
| `[!abstract]` | 👔 | Impacto ejecutivo: decisiones, riesgos, preguntas que responde |
| `[!example]` | 📊 | Caso de negocio genérico por industria (solo en secciones grandes) |
| `[!warning]` | ⚠️ | Regla crítica que no se puede violar |
| `[!danger]` | 🚨 | Error costoso (ej. data leakage, fit sobre test) |

---

## 🔄 El ciclo de vida de un proyecto de DS mapeado a tomos

```
   PREGUNTA DE NEGOCIO ─ Introducción Ejecutiva (T01)
            │
            ▼
   Fundamentos matemáticos y estadísticos (T02) ─── base transversal de todo
            │
            ▼
   Datos crudos ──► Preparación y calidad (T03) ──► EDA (T04) ──► Escalado (T05)
                                                                      │
            ┌─────────────────────────────────────────────────────────┘
            ▼
   ¿Hay etiquetas (target)?
    ├─ NO  ──► No supervisado: Clustering (T06) · Reglas de asociación (T09)
    └─ SÍ  ──► Supervisado: Modelos (T07) ──► Métricas de evaluación (T08)
            │
            ▼
   Validación honesta y anti-leakage (T10) ──► Mejora de modelos (T11) ──► Deep Learning (T12)
            │
            ▼
   Producción: MLOps · XAI · Ética (T13)
            │
            ▼
   Lectura de resultados para no técnicos (T14) · Glosario (T15) · Bibliografía (T16)
            │
            └──────────────────── iteración continua ◄────────────────────┘
```

---

## 📚 Los tomos — núcleo (00–16) y extensión aplicada (17–25)

La **extensión aplicada** (tomos 17–25) cubre los dominios especializados que el núcleo deja abiertos: forecasting, causalidad y uplift, NLP/LLMs, sistemas de recomendación, supervivencia + bandits, feature engineering avanzado, el debate tabular DL vs boosting, experimentación A/B, y privacidad + datos sintéticos.

| Tomo | Nota | Qué contiene | Audiencia principal |
|---|---|---|---|
| 00 | Este índice | Mapa de contenido, convenciones, rutas de lectura | 🔧 🧭 👔 |
| 01 | [[01-Introduccion-Ejecutiva]] | Qué es DS/ML, qué problemas resuelve, valor de negocio, ciclo de vida, mapa de lectura por perfil | 👔 🧭 (puerta de entrada de todos) |
| 02 | [[02-Fundamentos-Matematicos]] | Álgebra lineal, cálculo, probabilidad, distribuciones, inferencia estadística | 🔧 |
| 03 | [[03-Preparacion-de-Datos]] | Limpieza, nulos (MCAR/MAR/MNAR), outliers, encoding, desbalance, feature engineering/selection, reducción de dimensionalidad | 🔧 🧭 |
| 04 | [[04-EDA]] | Análisis exploratorio univariado, bivariado, del target, multivariado + herramientas | 🔧 🧭 |
| 05 | [[05-Escalado-de-Datos]] | Todas las técnicas de escalado y transformación + reglas críticas anti-leakage | 🔧 |
| 06 | [[06-Clustering]] | K-Means, DBSCAN/HDBSCAN, jerárquico, GMM y métricas de clustering | 🔧 🧭 |
| 07 | [[07-Modelos-Supervisados]] | Clasificación, regresión y detección de anomalías: mecanismo, hiperparámetros, cuándo usar cada uno | 🔧 🧭 |
| 08 | [[08-Metricas-de-Evaluacion]] | Matriz de confusión, métricas de clasificación y regresión, curvas ROC/PR, calibración, umbral óptimo | 🔧 🧭 👔 |
| 09 | [[09-Reglas-de-Asociacion]] | Support, Confidence, Lift; Apriori, FP-Growth, Eclat; market basket y más allá del retail | 🔧 🧭 |
| 10 | [[10-Validacion-y-Leakage]] | Esquemas de partición (K-Fold, Time Series Split…) y el catálogo completo de data leakage | 🔧 🧭 👔 |
| 11 | [[11-Mejora-de-Modelos]] | Bias-variance, tuning de hiperparámetros, regularización, ensembles, calibración, umbral de decisión | 🔧 🧭 |
| 12 | [[12-Deep-Learning]] | MLP, activaciones, optimizadores, CNN, RNN/LSTM, Transformers, transfer learning, frameworks | 🔧 🧭 |
| 13 | [[13-MLOps-XAI-Etica]] | Interpretabilidad (SHAP, LIME), reproducibilidad, deployment, monitoreo de drift, fairness | 🔧 🧭 👔 |
| 14 | [[14-Anexo-Interpretar-Resultados]] | Cómo leer matriz de confusión, ROC, feature importance y dashboards de drift **sin saber programar** | 👔 🧭 |
| 15 | [[15-Glosario-Ejecutivo]] | 65 términos técnicos traducidos a lenguaje de negocio | 👔 |
| 16 | [[16-Bibliografia]] | Referencias formales con datos de publicación completos | 🔧 🧭 👔 |
| 17 | [[17-Series-de-Tiempo]] | Forecasting: descomposición, estacionariedad, ACF/PACF, ETS/ARIMA/SARIMA/Prophet, ML global, walk-forward y MASE | 🔧 🧭 |
| 18 | [[18-Causalidad-y-Uplift]] | De predecir a intervenir: potential outcomes, DAGs, métodos observacionales (PSM, DiD, RDD, Double ML) y uplift modeling con Qini | 🔧 🧭 👔 |
| 19 | [[19-NLP-y-LLMs]] | Texto en la práctica: TF-IDF, embeddings, tareas y métricas, topic modeling, prompting, RAG y fine-tuning vs prompting | 🔧 🧭 👔 |
| 20 | [[20-Sistemas-de-Recomendacion]] | Qué mostrarle a cada quién: content-based, collaborative filtering, factorización, métricas de ranking, cold start y feedback loops | 🔧 🧭 |
| 21 | [[21-Supervivencia-y-Bandits]] | El *cuándo* (Kaplan-Meier, Cox, censura, C-index) y el *aprender decidiendo* (ε-greedy, UCB, Thompson Sampling) | 🔧 🧭 |
| 22 | [[22-Feature-Engineering-Avanzado]] | La cocina de las features: aggregation windows, ratios, interacciones, features temporales, patrones por dominio, reglas de disciplina | 🔧 🧭 👔 |
| 23 | [[23-Tabular-DL-vs-Boosting]] | El debate de 2026: cuándo XGBoost/LightGBM basta, cuándo TabPFN gana sin tuning, cuándo DL tabular se justifica — benchmarks, inductive biases y guía de decisión | 🔧 🧭 👔 |
| 24 | [[24-Experimentacion-AB]] | Diseño de experimentos controlados: sample size, MDE, sequential testing, CUPED, interleaving, trampas (SRM, spillover) y guía A/B vs quasi-experiment vs bandit | 🔧 🧭 👔 |
| 25 | [[25-Privacidad-y-Datos-Sinteticos]] | Differential privacy (ε, δ), federated learning, generación de datos sintéticos (CTGAN, TabDDPM), evaluación de calidad y guía de decisión regulatoria | 🔧 🧭 👔 |

> [!info] 📌 Tomos autocontenidos
> Cada tomo puede leerse sin haber leído los anteriores: los prerrequisitos se **enlazan** con wikilinks, no se repiten. Si un concepto te falta, el enlace te lleva directo a su definición.

---

## 🧑‍🤝‍🧑 Rutas de lectura por perfil

### 👔 Ruta Ejecutivo (≈ 2–3 horas de lectura)

Para tomar decisiones informadas sin entrar en fórmulas:

1. [[01-Introduccion-Ejecutiva]] — qué es esto y qué valor tiene
2. [[14-Anexo-Interpretar-Resultados]] — cómo leer lo que el equipo de datos te muestra
3. [[15-Glosario-Ejecutivo]] — el diccionario para las reuniones
4. Los callouts `[!abstract]` 👔 y `[!example]` 📊 de los tomos 08, 10 y 13 — métricas, validación y producción son donde se gana o se pierde el dinero

### 🧭 Ruta Puente (Product Owner / Líder / Analista)

Para traducir entre negocio y técnica y desafiar decisiones con criterio:

1. [[01-Introduccion-Ejecutiva]] → [[04-EDA]] → [[08-Metricas-de-Evaluacion]] → [[10-Validacion-y-Leakage]]
2. Luego los bloques 🧭 de [[03-Preparacion-de-Datos]], [[07-Modelos-Supervisados]] y [[11-Mejora-de-Modelos]]
3. Cierre con [[13-MLOps-XAI-Etica]] — qué exigir antes de aprobar un deployment

### 🔧 Ruta Técnico (lectura completa recomendada)

El orden natural es el del ciclo de vida: [[02-Fundamentos-Matematicos]] → [[03-Preparacion-de-Datos]] → [[04-EDA]] → [[05-Escalado-de-Datos]] → [[06-Clustering]] → [[07-Modelos-Supervisados]] → [[08-Metricas-de-Evaluacion]] → [[09-Reglas-de-Asociacion]] → [[10-Validacion-y-Leakage]] → [[11-Mejora-de-Modelos]] → [[12-Deep-Learning]] → [[13-MLOps-XAI-Etica]], y luego la extensión aplicada según tu dominio: [[17-Series-de-Tiempo]] · [[18-Causalidad-y-Uplift]] · [[19-NLP-y-LLMs]] · [[20-Sistemas-de-Recomendacion]] · [[21-Supervivencia-y-Bandits]] · [[22-Feature-Engineering-Avanzado]] · [[23-Tabular-DL-vs-Boosting]] · [[24-Experimentacion-AB]] · [[25-Privacidad-y-Datos-Sinteticos]]. Las analogías 💡 no son relleno: son anclas de memoria y material listo para explicar tu trabajo a stakeholders.

---

## 🔗 Documentos meta del proyecto

- [[Instrucciones-Guia-Maestra-DS|📖 Instrucciones de la Guía Maestra de DS]] — contrato de trabajo: rol, audiencias, reglas de idioma, política de fuentes, convención de "sin código", mandato de vanguardia, flujo y deuda técnica conocida. Claude lo lee al inicio de cada sesión.
- [[Prompts-NotebookLM-Guia-Maestra-DS|🎙️ Prompts para NotebookLM]] — los prompts para generar un podcast por tomo (23 episodios, uno por cada tomo que enseña conceptos: 01–14 y 17–25). Incluye el blindaje anti-invención que esta guía necesita por no tener fuente primaria, y el registro de analogías que evita que dos episodios suenen igual.

---

## 📐 Convenciones editoriales

- **Términos técnicos sin traducir**: dataframe, pipeline, feature, deployment, overfitting, boosting, encoding, drift, etc. se mantienen en inglés.
- **Fórmulas en notación inline legible**: `RMSE = √(Σ(y-ŷ)²/N)` — sin LaTeX complejo.
- **Diagramas en ASCII** dentro de bloques de código (sin Mermaid, sin imágenes): renderizan en cualquier lector Markdown.
- **Tablas comparativas** siempre con columnas técnicas **y** la columna "cuándo conviene".
- **Casos de negocio** solo en secciones grandes, con escenarios genéricos por industria (retail, banca, salud, telco, manufactura), sin nombres de empresas reales.
- **Citas** en formato `(Autor, Año)` dentro del texto, consolidadas en [[16-Bibliografia]].
- **Español neutro profesional**, tuteo permitido en analogías.

---

## ✅ Estado de la guía

| Tomo | Estado |
|---|---|
| 00 — MOC | ✅ Generado |
| 01 — Introducción Ejecutiva | ✅ Generado · 📚 bibliografía 6/6 verificada (2026-07-19) · 🔎 **vanguardia v6.1 (2026-09-06):** el «80 % del tiempo en datos» pasa a orden de magnitud con fuente (Anaconda 2020: ~45 %); *Literary Digest* corregido (10 M de papeletas, ~2,4 M de respuestas; no-respuesta, Squire 1988) |
| 02 — Fundamentos Matemáticos | ✅ Generado · 🔎 **vanguardia v6.1 (2026-09-06):** corrección — `lstsq` de SciPy resuelve por SVD (`gelsd`), no por QR (documentación de SciPy y LAPACK) |
| 03 — Preparación de Datos | ✅ Generado · 👥 +1 audiencia · ✏️ tabla Feature Selection sin sintaxis Python, columna "cuándo conviene" (v6.2, 2026-07-29) · 🔎 **vanguardia v6.4 (2026-09-06):** corregir el desbalance daña la calibración (van den Goorbergh 2022; Carriero 2025), la «ventaja global» de UMAP era la inicialización (Kobak & Linderman 2021; PaCMAP), permutation importance con correlación infla (Hooker 2021; Strobl 2008) + Boruta, IterativeImputer no es imputación múltiple por defecto (van Buuren 2011; Sperrin 2020), fila LOF corregida (`novelty=True`) |
| 04 — EDA | ✅ Generado · 👥 +7 audiencia (v6.1, 2026-07-29) · 🔧 **reparado (v6.3, 2026-09-05):** frontmatter colapsado en una sola línea y 19 callouts con título y cuerpo fusionados (daño introducido en el commit del 2026-08-28); 📚 Brink 2016 · 🔎 **vanguardia v6.4 (2026-09-06):** el profiler de referencia es ahora fg-data-profiling (ydata-profiling congelado en abril de 2026; Lux abandonado; missingno sin releases desde 2023), callout de data snooping (ESL §7.10.2; Cawley & Talbot 2010), tamaños de efecto y coeficientes de dependencia (Cramér's V corregido, η², Phi-K, ξ de Chatterjee, distance correlation), umbrales del VIF y |r| relativizados (O'Brien 2007; Dormann 2013), kurtosis en la escala de pandas, páginas de STL corregidas (3–73) |
| 05 — Escalado de Datos | ✅ Generado · 🚫 2 bloques `python` → diagramas ASCII (v6.1, 2026-07-29) · 👥 +5 audiencia · 🔎 **vanguardia v6.3 (2026-09-06):** Yeo-Johnson con su ficha (Yeo & Johnson 2000) y explicación; corrección de la fila «escalar → imputar» y del caveat de KNNImputer (los scalers de scikit-learn ignoran los NaN en fit y los dejan pasar en transform) |
| 06 — Clustering | ✅ Generado · 📚 bibliografía 4/4 verificada (2026-07-27) · ✏️ quitado `pip install` de la ficha HDBSCAN (v6.1, 2026-07-29) · 🔎 **vanguardia v6.2 (2026-09-06):** `n_init` de K-Means ya no es 10 por defecto (sklearn 1.4), HDBSCAN nativo no entrega outlier scores ni árbol condensado (McInnes, Healy & Astels 2017; GLOSH, Campello 2015), precauciones del patrón UMAP → HDBSCAN, el codo como criterio poco fiable (Schubert 2023), regla del gap corregida (Tibshirani 2001), silhouette (Rousseeuw 1987) y el sesgo convexo de los índices internos; DBSCAN Revisited (Schubert 2017), Gower 1971, k-prototypes (Huang 1998) |
| 07 — Modelos Supervisados | ✅ Generado · 📚 bibliografía 6/6 verificada (2026-07-27) · 🔎 **vanguardia v6.1 (2026-09-06):** los árboles, Random Forest y ExtraTrees de scikit-learn ya aceptan NaN (1.3/1.4/1.6 — la guía decía lo contrario), categóricas nativas en XGBoost (≥ 1.5) y LightGBM (Fisher 1958), HistGradientBoosting como cuarta implementación, puente a T23 (Grinsztajn 2022; McElfresh 2023), matiz de escala del SVM lineal |
| 08 — Métricas de Evaluación | ✅ Generado · 📚 bibliografía 5/5 verificada (2026-07-27) · 🆕 sección 6 "Criterios de información" (AIC/BIC/HQIC), puentes a MASE/T17 (v6.2, 2026-07-29) · 📚 4 fichas del sprint 08-28 verificadas; cita «(Robertson, 2009)» reemplazada por Manning et al. 2008 + Voorhees 1999 (v6.4, 2026-09-05) · 🔎 **vanguardia v6.5 (2026-09-06):** fila SMAPE corregida (no es simétrico: Goodwin & Lawton 1999), umbral de costos atribuido (Elkan 2001), Brier como proper scoring rule y su descomposición (Murphy 1973), reliability diagram y ECE con sus fallas (Niculescu-Mizil & Caruana 2005; Guo 2017; Nixon 2019), callout «las métricas para desbalance dependen de la prevalencia» (Brabec 2020; Saito & Rehmsmeier 2015; Chicco & Jurman 2020; Landis & Koch 1977), regla del 80 % con su fuente real (29 CFR §1607.4) |
| 09 — Reglas de Asociación | ✅ Generado · 📚 bibliografía 3/3 verificada (2026-07-27) · 👥 +3 audiencia (v6.1, 2026-07-29) · 📚 3 fichas del sprint 08-28 verificadas — Srikant & Agrawal 1996, Pei et al. 2001, Zaki 2001 (2026-09-05) · 🔎 **vanguardia v6.3 (2026-09-06):** corrección — la métrica de Zhang es direccional, no simétrica (código de mlxtend); métricas certainty/kulczynski/representativity; velocidad de FP-Growth ajustada a «un orden de magnitud» (Han 2000); referencia de Zaki 2001 añadida |
| 10 — Validación y Leakage | ✅ Generado · 📚 bibliografía 3/3 verificada (2026-07-27) · 👥 +3 audiencia · 🆕 walk-forward/rolling-origin, sliding vs. expanding window, leakage de exógenas futuras (v6.3, 2026-07-29) · 🔎 **vanguardia v6.5 (2026-09-06):** leakage y crisis de reproducibilidad (Kapoor & Narayanan 2023: dos tipos nuevos en la tabla, model info sheets), el IC «media ± 1,96·std/√K» del CV retirado (Bengio & Grandvalet 2004; Bates, Hastie & Tibshirani 2024), nested CV con respaldo (Cawley & Talbot 2010; Varma & Simon 2006), train-serving skew (Breck et al. 2017), fila Purged K-Fold → López de Prado 2018 |
| 11 — Mejora de Modelos | ✅ Generado · 📚 bibliografía 5/5 verificada (2026-07-27) · ✏️ columna "cuándo conviene" en Ensembles, wikilinks fuera de fence, 🆕 curvas de pérdida por época (v6.1, 2026-07-29) · 📚 atribuciones corporativas → papers (Erickson 2020, Wang et al. 2021 FLAML); fila TabPFN con evidencia independiente (TabArena) separada de las afirmaciones del fabricante (v6.3, 2026-09-05) · 🔎 **vanguardia v6.4 (2026-09-06):** presets de AutoGluon 1.x, Optuna con Hyperband y multi-fidelity, callout «¿cuánto paga el tuning?» (Probst 2019; Holzmüller 2024; TabArena), conformal prediction, `TunedThresholdClassifierCV` |
| 12 — Deep Learning | ✅ Generado · 📚 bibliografía 15/15 verificada · 🔎 vanguardia v6.1 (2026-07-27): MoE, Diffusion Models, nota de Mamba/SSM · 🆕 §6.2 serie→tensor, danger bidireccional+forecasting (v6.3, 2026-07-29) |
| 13 — MLOps, XAI y Ética | ✅ Generado · 📚 bibliografía 5/5 verificada · 🔎 vanguardia v6.2 (2026-07-27): EU AI Act/NIST/ISO 42001, model cards · 👥 +5 audiencia · ✅ dato del Digital Omnibus verificado y actualizado (v6.4, 2026-07-29) — ya es ley vigente (Reglamento UE 2026/1744) |
| 14 — Anexo Interpretación | ✅ Generado · 🔎 **vanguardia v6.2 (2026-09-06):** callout «el ranking no es único ni es causalidad» (importancia por impureza, permutation, SHAP y variables correlacionadas: Strobl 2008; Hooker 2021; Lundberg & Lee 2017), pregunta ejecutiva nueva y bloque de referencias (antes no tenía) |
| 15 — Glosario Ejecutivo | ✅ Generado · ✏️ orden alfabético corregido en D–F y P–S (v6.1, 2026-07-29) |
| 16 — Bibliografía | ✅ Generado · 👥 +1 audiencia · 📚 +12 fuentes nuevas (series de tiempo) (v6.6, 2026-07-29) · 📚 **+40 fichas verificadas** — secciones 19–22 nuevas (Tomos 22–25) y adiciones a 1, 6, 7 y 9; **173 fuentes** en total (v6.7, 2026-09-05) · 📚 **+140 fichas y +18 enlaces de documentación** del escaneo de vanguardia; **331 fuentes** en total (v6.8, 2026-09-06) |
| 17 — Series de Tiempo | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 7/7 verificada (2026-07-27) · 🆕 F_T/F_S, periodograma, regresión espuria/cointegración, Granger, diagnóstico de residuos (Box-Jenkins), estrategias multi-step, regresión armónica dinámica, random walk (v6.2, 2026-07-29) · 🔎 **vanguardia v6.3 (2026-09-06):** foundation models de series (TimesFM, Chronos, Moirai, Lag-Llama; GIFT-Eval), combinaciones y competencias M4–M6, Prophet en modo mantenimiento, conformal para series (ACI, EnbPI), CRPS |
| 18 — Causalidad y Uplift | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 10/10 verificada (2026-07-27) · ✏️ frase 👔 faltante en §3 (v6.2, 2026-07-29) · 🔎 **vanguardia v6.3 (2026-09-06):** DiD escalonado (Callaway & Sant'Anna; Goodman-Bacon; Sun & Abraham; de Chaisemartin & D'Haultfœuille), control sintético, pre-trends (Roth), E-value, R-learner y causal forest, RATE/AUTOC |
| 19 — NLP y LLMs | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · verificado vs. Guía RAG · 📚 bibliografía 15/15 verificada · 🔎 vanguardia v6.2 (2026-07-27): reasoning models, MCP/tool use, context rot, prompt caching · ✏️ analogía en Topic Modeling, frase 👔 en RAG (v6.3, 2026-07-29) |
| 20 — Sistemas de Recomendación | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 8/8 verificada · 🔎 vanguardia v6.2 (2026-07-27): generative recommenders (HSTU, OneRec) |
| 21 — Supervivencia y Bandits | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 8/8 verificada (2026-07-27) · 👥 +10 audiencia · ✏️ analogías en Cox/AFT y C-index, frase 👔 en A.1 (v6.2, 2026-07-29) · ✏️ navegación «Siguiente» hacia T22 (v6.3, 2026-09-05) · 🔎 **vanguardia v6.4 (2026-09-06):** métricas de supervivencia (C-index IPCW, AUC(t), IBS, D-calibration), boosting y redes de supervivencia (scikit-survival, XGBoost AFT, DeepSurv, DeepHit), off-policy evaluation, sesgo de los datos adaptativos (Nie 2018; Hadad 2021) |
| 22 — Feature Engineering Avanzado | ✅ Generado v1.0 (2026-08-28). Nuevo tomo de extensión: ciclo iterativo, familias de transformaciones (aggregation windows, ratios, temporales, texto/alta cardinalidad), patrones por dominio, reglas de disciplina. Bibliografía: Domingos 2012, Zheng & Casari 2018, Kuhn & Johnson 2019. · 🔧 v1.1 (2026-09-05): navegación, callout de reglas reparado, porcentajes etiquetados como criterio editorial; 📚 2/2 fichas verificadas · 🔎 **vanguardia v1.2 (2026-09-06):** point-in-time joins y feature stores, `TargetEncoder` con cross fitting y shrinkage (Micci-Barreca 2001), LLMs como generadores de features (CAAFE) y su límite de madurez, robustez de los árboles al ruido |
| 23 — Tabular DL vs Boosting | ✅ Generado v1.0 (2026-08-28). Estado del arte del debate tabular: benchmarks TabArena 2026, inductive biases de árboles vs NNs, familias DL (FT-Transformer, TabNet, TabPFN), guía de decisión por tamaño de datos. Bibliografía: Grinsztajn 2022, Hollmann 2023/2026, Gorishniy 2021, McElfresh 2023. · 🔧 v1.1 (2026-09-05): 📚 verificación bibliográfica con **dos correcciones de fondo** — «Hollmann et al. 2026» era Grinsztajn et al. 2025 (reporte técnico de Prior Labs) y el «Nature 2026 / TabPFN v2.6» no existe (el paper de *Nature* es Hollmann et al. 2025); tabla del §1 reescrita con evidencia independiente (TabArena, Erickson et al. 2025) y las cifras del fabricante marcadas como tales · 🔎 **vanguardia v1.2 (2026-09-06):** TabICL hasta 500K filas (el techo de 10K era de TabPFN v2), callout de límites fuera de i.i.d. (BeyondArena, preprint; Drift-Resilient TabPFN), MLP moderno (TabM, RealMLP) como línea DL, transfer learning tabular (CARTE vs. in-context) |
| 24 — Experimentación A/B | ✅ Generado v1.0 (2026-08-28). Diseño, sample size/MDE/power, randomización (CUPED/CUPAC), sequential testing, interleaving/switchback, múltiples métricas, trampas (SRM, spillover), guía de decisión. Bibliografía: Kohavi 2020, Johari 2017, Deng 2013. · 🔧 v1.1 (2026-09-05): 5 wikilinks a tomos inexistentes reparados, navegación estándar, 28 líneas de audiencia, callouts al código de la guía, caso de negocio dentro de `[!example]`; 📚 6/6 fichas verificadas (Johari et al. 2017: autor Koomen restituido) · 🔎 **vanguardia v1.2 (2026-09-06):** tabla de peeks con fuente (Armitage 1969), always-valid inference (Ramdas 2023; Lindon 2022), ajuste por regresión (Lin 2013; Guo 2021), interferencia (Johari & Li 2022; Bajari 2023), SRM (Fabijan 2019), §7.5 winner's curse |
| 25 — Privacidad y Datos Sintéticos | ✅ Generado v1.0 (2026-08-28). DP (ε,δ), mecanismos, composición, FL (FedAvg, non-IID), datos sintéticos (CTGAN, TabDDPM, copulas), evaluación tripartita, guía regulatoria. Bibliografía: Dwork 2006, McMahan 2017, Xu 2019, Kotelnikov 2023. · 🔧 v1.1 (2026-09-05): 6 wikilinks a tomos inexistentes reparados, navegación estándar, 31 líneas de audiencia; 📚 11/11 fichas verificadas (TabDDPM: autores corregidos; Carlini et al.: «Sehwag») · 🔎 **vanguardia v1.2 (2026-09-06):** EU AI Act + Reglamento 2026/1744, Ley 21.719 (Chile), Censo 2020 (ε = 19,61), §4.3 generadores LLM y de difusión latente (GReaT, TabuLa, REaLTabFormer, TabSyn), MIA y la refutación de DCR (Stadler 2022; Carlini 2022; Steinke 2023), TJUE C-413/23 P, EDPB 28/2024 |

**🔎 Extensión aplicada (17–21) nivelada al núcleo (v6.1, 2026-07-19):** los cinco tomos pasaron de 10–13 KB a 19–30 KB, con secciones nuevas de profundidad (forecasting jerárquico y probabilístico, supuestos de identificación causal y robustez, arquitectura de recomendadores en dos etapas, competing risks, trampas de bandits en producción). El Tomo 19 (NLP/LLMs) además se **verificó contra la Guía Maestra de RAG** para coherencia de fórmulas y terminología.

**👥 Etiquetado de audiencia normalizado (2026-07-29):** deuda #3 cerrada — 35 líneas `Audiencia:` agregadas vía auditoría por subagentes en los 8 tomos con huecos reales (arriba). Los otros 14 tomos ya cumplían la convención. Detalle en `Instrucciones-Guia-Maestra-DS.md` §3 y §12.

**🔬 Barrido de potenciación vs. curso externo de Series de Tiempo (2026-07-29):** un barrido de 10 agentes mapeó un módulo universitario completo de Series de Tiempo (ajeno al proyecto, usado solo para detectar vacíos — nunca como fuente citable) y lo contrastó contra la guía. Resultado: 13 adiciones de contenido en T17/T08/T10/T12/T11 (todas con referencia bibliográfica real verificada antes de escribir — 11 nuevas CONFIRMED + 1 con error de autor corregido) y 20 hallazgos de pulido editorial resueltos en 13 tomos. Detalle completo en `Instrucciones-Guia-Maestra-DS.md` §10.

**🏁 Guía completa: 26/26 notas generadas — núcleo 00–16 + extensión aplicada 17–25. Deuda técnica conocida: 0 ítems abiertos — la #11 (escaneo de vanguardia) se cerró el 2026-09-06 con los 19 tomos cubiertos** (ver `Instrucciones-Guia-Maestra-DS.md` §12).

**🔬 Sprint de profundización y nuevos tomos (2026-08-28):** 7 tomos profundizados (T04 v6.2 +8 KB, T05 v6.2 +3 KB, T08 v6.3 +8 KB, T09 v6.2 +3 KB, T10 v6.4 +4 KB, T11 v6.2 +9 KB, T14 v6.1 +4 KB) + 4 tomos nuevos (T22 Feature Engineering 15 KB, T23 Tabular DL vs Boosting 11 KB, T24 Experimentación A/B 28 KB, T25 Privacidad y Datos Sintéticos 37 KB). Total: **+130 KB** de contenido con bibliografía verificable.

**🔧 Protocolo de mantenimiento ejecutado (2026-09-05):** auditoría estructural de las 28 notas más verificación bibliográfica de todo lo escrito en el sprint del 2026-08-28, que **no había pasado por el protocolo §5**. Hallazgos y correcciones: (1) **T04 dañado en el commit del 28-ago** — frontmatter colapsado en una línea `## title:…` y 19 callouts con título y cuerpo fusionados (Obsidian los mostraba como títulos gigantes sin cuerpo); restaurado. (2) **T24 y T25 fuera de convención** — 11 wikilinks a tomos inexistentes (`18-Causalidad-Inferencia-Causal`, `21-Multi-Armed-Bandits`, `07-Estadistica-Inferencial`, `14-Metricas-Evaluacion`, `13-Fairness-Sesgo-Modelos`, `05-Feature-Engineering`, `08-Gradient-Boosting`, `17-MLOps-Produccion`, `19-Deep-Learning-Fundamentos`), cero líneas `Audiencia:`, navegación sin enlace al MOC y callouts fuera del código; normalizados. (3) **Bibliografía del sprint sin verificar ni consolidar** — 40 fichas verificadas contra fuente primaria (7 errores sustantivos corregidos) y consolidadas en las secciones 19–22 del Tomo 16. (4) **T23 corregido en su §1**: las cifras de TabPFN-2.5/2.6/3 eran afirmaciones del fabricante presentadas como evidencia y dos referencias no existían como estaban citadas. (5) Navegación «Siguiente» cerrada en T21→T22→T23→T24→T25. (6) **Podcasts**: episodios 22–25 escritos en `Prompts-NotebookLM-Guia-Maestra-DS.md` (serie 23/23, cuarto acto). Bitácora completa en `Instrucciones-Guia-Maestra-DS.md` §10 y deuda §12 (ítems 7–11).

**🔎 Escaneo de vanguardia sobre los tomos no cubiertos por el primero (2026-09-06):** un agente auditor por tomo — lee el tomo completo, busca con WebSearch vacíos genuinos de vigencia bajo las 3 reglas de blindaje del §10 de las Instrucciones y devuelve solo hallazgos con evidencia y URL —; la escritura la hizo Claude tomo por tomo, con el Tomo 16 en una fase aparte. Cubiertos **los 19 tomos, 67 hallazgos (36 de consenso, 22 de práctica común, 9 de criterio), 140 fichas y 18 enlaces de documentación nuevos**: T01 (v6.1), T02 (v6.1), T03 (v6.4), T04 (v6.4), T05 (v6.3), T06 (v6.2), T07 (v6.1), T08 (v6.5), T09 (v6.3), T10 (v6.5), T11 (v6.4), T14 (v6.2), T17 (v6.3), T18 (v6.3), T21 (v6.4), T22 (v1.2), T23 (v1.2), T24 (v1.2), T25 (v1.2). Diecinueve eran **afirmaciones hoy falsas, desactualizadas o sin respaldo**, no vacíos: los árboles de scikit-learn sí aceptan NaN y XGBoost/LightGBM sí manejan categóricas (T07); el intervalo «media ± 1,96·std/√K» del CV no es válido y la fila Purged K-Fold citaba una librería en vez de López de Prado (T10); el techo de 10K filas era de TabPFN v2, no de los foundation models tabulares (T23); LOF sí puntúa datos nuevos (`novelty=True`) y la preservación de estructura global de UMAP depende de la inicialización, no es intrínseca (T03); SMAPE no es simétrico y la «regla del 80 %» del Disparate Impact es un criterio administrativo de EE. UU., no un «umbral legal» que ISO 42001 o el AI Act referencien (T08); ydata-profiling fue renombrado y congelado, el umbral de kurtosis estaba en la escala equivocada y las páginas de STL eran 3–73 (T04); `n_init` de K-Means ya no es 10, el HDBSCAN nativo no entrega outlier scores ni árbol condensado y la regla del gap statistic no es «maximizar el gap» (T06); `lstsq` resuelve por SVD, no por QR (T02); la fila «escalar → imputar» y el caveat de KNNImputer describían un scikit-learn que ya no existe (T05); la métrica de Zhang no es simétrica (T09); el *Literary Digest* recibió ~2,4 millones de respuestas, no 10 (T01). Los cinco tomos de baja rotación (T01, T02, T05, T09, T14) se auditaron con un solo agente combinado. Bitácora completa en `Instrucciones-Guia-Maestra-DS.md` §10; la deuda #11 queda **cerrada**.

**📚 Bibliografía: 331 fuentes verificadas (2026-09-06)** — 308 en el cuerpo bibliográfico (Secciones 1–12, 14–22) + 23 enlaces de documentación oficial. Deuda #4 cerrada el 2026-07-27; el protocolo se re-aplicó el 2026-09-05 a las ~39 referencias que el sprint del 2026-08-28 había escrito sin verificar (40 fichas nuevas, 7 errores sustantivos corregidos), y el 2026-09-06 a las 140 fichas del escaneo de vanguardia (verificadas por el agente auditor contra registros primarios antes de escribirse; preprints etiquetados). Detalle en `Instrucciones-Guia-Maestra-DS.md` §5, §10 y §12.

---

**Navegación:** Siguiente tomo → [[01-Introduccion-Ejecutiva|01 · Introducción Ejecutiva]]
