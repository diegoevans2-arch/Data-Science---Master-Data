---
title: "Tomo 00 — MOC · Guía Maestra de Data Science"
tags: [data-science, machine-learning, moc, indice]
audiencias: [tecnico, puente, ejecutivo]
tomo: 00
version: 6.0
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

| Etiqueta | Perfil | Qué contiene |
|---|---|---|
| 🔧 **[Técnico]** | Data Scientist / Data Engineer / ML Engineer | Definiciones formales, fórmulas, hiperparámetros, supuestos, implementación, limitaciones |
| 🧭 **[Puente]** | Product Owner / Líder técnico / Analista | Traducción negocio↔técnica, cuándo usar qué, trade-offs de decisión |
| 👔 **[Ejecutivo]** | Gerente / C-level / Stakeholder | Impacto en el negocio, decisiones que habilita, riesgos, costo de hacerlo mal |
| 💡 **[Analogía]** | Todos | Explicación intuitiva con analogía cotidiana. Sin prerrequisitos |

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

## 📚 Los tomos — núcleo (00–16) y extensión aplicada (17–21)

La **extensión aplicada** (tomos 17–21) cubre los dominios especializados que el núcleo deja abiertos: forecasting, causalidad y uplift, NLP/LLMs, sistemas de recomendación, y supervivencia + bandits.

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
| 15 | [[15-Glosario-Ejecutivo]] | 40+ términos técnicos traducidos a lenguaje de negocio | 👔 |
| 16 | [[16-Bibliografia]] | Referencias formales con datos de publicación completos | 🔧 🧭 👔 |
| 17 | [[17-Series-de-Tiempo]] | Forecasting: descomposición, estacionariedad, ACF/PACF, ETS/ARIMA/SARIMA/Prophet, ML global, walk-forward y MASE | 🔧 🧭 |
| 18 | [[18-Causalidad-y-Uplift]] | De predecir a intervenir: potential outcomes, DAGs, métodos observacionales (PSM, DiD, RDD, Double ML) y uplift modeling con Qini | 🔧 🧭 👔 |
| 19 | [[19-NLP-y-LLMs]] | Texto en la práctica: TF-IDF, embeddings, tareas y métricas, topic modeling, prompting, RAG y fine-tuning vs prompting | 🔧 🧭 👔 |
| 20 | [[20-Sistemas-de-Recomendacion]] | Qué mostrarle a cada quién: content-based, collaborative filtering, factorización, métricas de ranking, cold start y feedback loops | 🔧 🧭 |
| 21 | [[21-Supervivencia-y-Bandits]] | El *cuándo* (Kaplan-Meier, Cox, censura, C-index) y el *aprender decidiendo* (ε-greedy, UCB, Thompson Sampling) | 🔧 🧭 |

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

El orden natural es el del ciclo de vida: [[02-Fundamentos-Matematicos]] → [[03-Preparacion-de-Datos]] → [[04-EDA]] → [[05-Escalado-de-Datos]] → [[06-Clustering]] → [[07-Modelos-Supervisados]] → [[08-Metricas-de-Evaluacion]] → [[09-Reglas-de-Asociacion]] → [[10-Validacion-y-Leakage]] → [[11-Mejora-de-Modelos]] → [[12-Deep-Learning]] → [[13-MLOps-XAI-Etica]], y luego la extensión aplicada según tu dominio: [[17-Series-de-Tiempo]] · [[18-Causalidad-y-Uplift]] · [[19-NLP-y-LLMs]] · [[20-Sistemas-de-Recomendacion]] · [[21-Supervivencia-y-Bandits]]. Las analogías 💡 no son relleno: son anclas de memoria y material listo para explicar tu trabajo a stakeholders.

---

## 🔗 Documentos meta del proyecto

- [[Instrucciones-Guia-Maestra-DS|📖 Instrucciones de la Guía Maestra de DS]] — contrato de trabajo: rol, audiencias, reglas de idioma, política de fuentes, convención de "sin código", mandato de vanguardia, flujo y deuda técnica conocida. Claude lo lee al inicio de cada sesión.

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
| 01 — Introducción Ejecutiva | ✅ Generado |
| 02 — Fundamentos Matemáticos | ✅ Generado |
| 03 — Preparación de Datos | ✅ Generado |
| 04 — EDA | ✅ Generado |
| 05 — Escalado de Datos | ✅ Generado |
| 06 — Clustering | ✅ Generado |
| 07 — Modelos Supervisados | ✅ Generado |
| 08 — Métricas de Evaluación | ✅ Generado |
| 09 — Reglas de Asociación | ✅ Generado |
| 10 — Validación y Leakage | ✅ Generado |
| 11 — Mejora de Modelos | ✅ Generado |
| 12 — Deep Learning | ✅ Generado |
| 13 — MLOps, XAI y Ética | ✅ Generado |
| 14 — Anexo Interpretación | ✅ Generado |
| 15 — Glosario Ejecutivo | ✅ Generado |
| 16 — Bibliografía | ✅ Generado |
| 17 — Series de Tiempo | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) |
| 18 — Causalidad y Uplift | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) |
| 19 — NLP y LLMs | ✅ Generado |
| 20 — Sistemas de Recomendación | ✅ Generado |
| 21 — Supervivencia y Bandits | ✅ Generado |

**🏁 Guía completa: 22/22 notas generadas — núcleo 00–16 + extensión aplicada 17–21 (v6.0).**

---

**Navegación:** Siguiente tomo → [[01-Introduccion-Ejecutiva|01 · Introducción Ejecutiva]]
