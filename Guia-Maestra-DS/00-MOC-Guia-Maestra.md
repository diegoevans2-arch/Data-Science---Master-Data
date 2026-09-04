---
title: "Tomo 00 — MOC · Guía Maestra de Data Science"
tags: [data-science, machine-learning, moc, indice]
audiencias: [tecnico, puente, ejecutivo]
tomo: 00
version: 6.3
updated: 2026-08-28
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

El orden natural es el del ciclo de vida: [[02-Fundamentos-Matematicos]] → [[03-Preparacion-de-Datos]] → [[04-EDA]] → [[05-Escalado-de-Datos]] → [[06-Clustering]] → [[07-Modelos-Supervisados]] → [[08-Metricas-de-Evaluacion]] → [[09-Reglas-de-Asociacion]] → [[10-Validacion-y-Leakage]] → [[11-Mejora-de-Modelos]] → [[12-Deep-Learning]] → [[13-MLOps-XAI-Etica]], y luego la extensión aplicada según tu dominio: [[17-Series-de-Tiempo]] · [[18-Causalidad-y-Uplift]] · [[19-NLP-y-LLMs]] · [[20-Sistemas-de-Recomendacion]] · [[21-Supervivencia-y-Bandits]] · [[22-Feature-Engineering-Avanzado]]. Las analogías 💡 no son relleno: son anclas de memoria y material listo para explicar tu trabajo a stakeholders.

---

## 🔗 Documentos meta del proyecto

- [[Instrucciones-Guia-Maestra-DS|📖 Instrucciones de la Guía Maestra de DS]] — contrato de trabajo: rol, audiencias, reglas de idioma, política de fuentes, convención de "sin código", mandato de vanguardia, flujo y deuda técnica conocida. Claude lo lee al inicio de cada sesión.
- [[Prompts-NotebookLM-Guia-Maestra-DS|🎙️ Prompts para NotebookLM]] — los prompts para generar un podcast por tomo (19 episodios, uno por cada tomo que enseña conceptos). Incluye el blindaje anti-invención que esta guía necesita por no tener fuente primaria, y el registro de analogías que evita que dos episodios suenen igual.

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
| 01 — Introducción Ejecutiva | ✅ Generado · 📚 bibliografía 6/6 verificada (2026-07-19) |
| 02 — Fundamentos Matemáticos | ✅ Generado |
| 03 — Preparación de Datos | ✅ Generado · 👥 +1 audiencia · ✏️ tabla Feature Selection sin sintaxis Python, columna "cuándo conviene" (v6.2, 2026-07-29) |
| 04 — EDA | ✅ Generado · 👥 +7 audiencia (v6.1, 2026-07-29) |
| 05 — Escalado de Datos | ✅ Generado · 🚫 2 bloques `python` → diagramas ASCII (v6.1, 2026-07-29) · 👥 +5 audiencia |
| 06 — Clustering | ✅ Generado · 📚 bibliografía 4/4 verificada (2026-07-27) · ✏️ quitado `pip install` de la ficha HDBSCAN (v6.1, 2026-07-29) |
| 07 — Modelos Supervisados | ✅ Generado · 📚 bibliografía 6/6 verificada (2026-07-27) |
| 08 — Métricas de Evaluación | ✅ Generado · 📚 bibliografía 5/5 verificada (2026-07-27) · 🆕 sección 6 "Criterios de información" (AIC/BIC/HQIC), puentes a MASE/T17 (v6.2, 2026-07-29) |
| 09 — Reglas de Asociación | ✅ Generado · 📚 bibliografía 3/3 verificada (2026-07-27) · 👥 +3 audiencia (v6.1, 2026-07-29) |
| 10 — Validación y Leakage | ✅ Generado · 📚 bibliografía 3/3 verificada (2026-07-27) · 👥 +3 audiencia · 🆕 walk-forward/rolling-origin, sliding vs. expanding window, leakage de exógenas futuras (v6.3, 2026-07-29) |
| 11 — Mejora de Modelos | ✅ Generado · 📚 bibliografía 5/5 verificada (2026-07-27) · ✏️ columna "cuándo conviene" en Ensembles, wikilinks fuera de fence, 🆕 curvas de pérdida por época (v6.1, 2026-07-29) |
| 12 — Deep Learning | ✅ Generado · 📚 bibliografía 15/15 verificada · 🔎 vanguardia v6.1 (2026-07-27): MoE, Diffusion Models, nota de Mamba/SSM · 🆕 §6.2 serie→tensor, danger bidireccional+forecasting (v6.3, 2026-07-29) |
| 13 — MLOps, XAI y Ética | ✅ Generado · 📚 bibliografía 5/5 verificada · 🔎 vanguardia v6.2 (2026-07-27): EU AI Act/NIST/ISO 42001, model cards · 👥 +5 audiencia · ✅ dato del Digital Omnibus verificado y actualizado (v6.4, 2026-07-29) — ya es ley vigente (Reglamento UE 2026/1744) |
| 14 — Anexo Interpretación | ✅ Generado |
| 15 — Glosario Ejecutivo | ✅ Generado · ✏️ orden alfabético corregido en D–F y P–S (v6.1, 2026-07-29) |
| 16 — Bibliografía | ✅ Generado · 👥 +1 audiencia · 📚 +12 fuentes nuevas (series de tiempo) (v6.6, 2026-07-29) |
| 17 — Series de Tiempo | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 7/7 verificada (2026-07-27) · 🆕 F_T/F_S, periodograma, regresión espuria/cointegración, Granger, diagnóstico de residuos (Box-Jenkins), estrategias multi-step, regresión armónica dinámica, random walk (v6.2, 2026-07-29) |
| 18 — Causalidad y Uplift | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 10/10 verificada (2026-07-27) · ✏️ frase 👔 faltante en §3 (v6.2, 2026-07-29) |
| 19 — NLP y LLMs | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · verificado vs. Guía RAG · 📚 bibliografía 15/15 verificada · 🔎 vanguardia v6.2 (2026-07-27): reasoning models, MCP/tool use, context rot, prompt caching · ✏️ analogía en Topic Modeling, frase 👔 en RAG (v6.3, 2026-07-29) |
| 20 — Sistemas de Recomendación | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 8/8 verificada · 🔎 vanguardia v6.2 (2026-07-27): generative recommenders (HSTU, OneRec) |
| 21 — Supervivencia y Bandits | ✅ Generado · 🔎 revisado v6.1 (2026-07-19) · 📚 bibliografía 8/8 verificada (2026-07-27) · 👥 +10 audiencia · ✏️ analogías en Cox/AFT y C-index, frase 👔 en A.1 (v6.2, 2026-07-29) |
| 22 — Feature Engineering Avanzado | ✅ Generado v1.0 (2026-08-28). Nuevo tomo de extensión: ciclo iterativo, familias de transformaciones (aggregation windows, ratios, temporales, texto/alta cardinalidad), patrones por dominio, reglas de disciplina. Bibliografía: Domingos 2012, Zheng & Casari 2018, Kuhn & Johnson 2019. |
| 23 — Tabular DL vs Boosting | ✅ Generado v1.0 (2026-08-28). Estado del arte del debate tabular: benchmarks TabArena 2026, inductive biases de árboles vs NNs, familias DL (FT-Transformer, TabNet, TabPFN), guía de decisión por tamaño de datos. Bibliografía: Grinsztajn 2022, Hollmann 2023/2026, Gorishniy 2021, McElfresh 2023. |
| 24 — Experimentación A/B | ✅ Generado v1.0 (2026-08-28). Diseño, sample size/MDE/power, randomización (CUPED/CUPAC), sequential testing, interleaving/switchback, múltiples métricas, trampas (SRM, spillover), guía de decisión. Bibliografía: Kohavi 2020, Johari 2017, Deng 2013. |
| 25 — Privacidad y Datos Sintéticos | ✅ Generado v1.0 (2026-08-28). DP (ε,δ), mecanismos, composición, FL (FedAvg, non-IID), datos sintéticos (CTGAN, TabDDPM, copulas), evaluación tripartita, guía regulatoria. Bibliografía: Dwork 2006, McMahan 2017, Xu 2019, Kotelnikov 2023. |

**🔎 Extensión aplicada (17–21) nivelada al núcleo (v6.1, 2026-07-19):** los cinco tomos pasaron de 10–13 KB a 19–30 KB, con secciones nuevas de profundidad (forecasting jerárquico y probabilístico, supuestos de identificación causal y robustez, arquitectura de recomendadores en dos etapas, competing risks, trampas de bandits en producción). El Tomo 19 (NLP/LLMs) además se **verificó contra la Guía Maestra de RAG** para coherencia de fórmulas y terminología.

**👥 Etiquetado de audiencia normalizado (2026-07-29):** deuda #3 cerrada — 35 líneas `Audiencia:` agregadas vía auditoría por subagentes en los 8 tomos con huecos reales (arriba). Los otros 14 tomos ya cumplían la convención. Detalle en `Instrucciones-Guia-Maestra-DS.md` §3 y §12.

**🔬 Barrido de potenciación vs. curso externo de Series de Tiempo (2026-07-29):** un barrido de 10 agentes mapeó un módulo universitario completo de Series de Tiempo (ajeno al proyecto, usado solo para detectar vacíos — nunca como fuente citable) y lo contrastó contra la guía. Resultado: 13 adiciones de contenido en T17/T08/T10/T12/T11 (todas con referencia bibliográfica real verificada antes de escribir — 11 nuevas CONFIRMED + 1 con error de autor corregido) y 20 hallazgos de pulido editorial resueltos en 13 tomos. Detalle completo en `Instrucciones-Guia-Maestra-DS.md` §10.

**🏁 Guía completa: 26/26 notas generadas — núcleo 00–16 + extensión aplicada 17–25. Deuda técnica conocida: 0 ítems abiertos** (ver `Instrucciones-Guia-Maestra-DS.md` §12).

**🔬 Sprint de profundización y nuevos tomos (2026-08-28):** 7 tomos profundizados (T04 v6.2 +8 KB, T05 v6.2 +3 KB, T08 v6.3 +8 KB, T09 v6.2 +3 KB, T10 v6.4 +4 KB, T11 v6.2 +9 KB, T14 v6.1 +4 KB) + 4 tomos nuevos (T22 Feature Engineering 15 KB, T23 Tabular DL vs Boosting 11 KB, T24 Experimentación A/B 28 KB, T25 Privacidad y Datos Sintéticos 37 KB). Total: **+130 KB** de contenido con bibliografía verificable.

**📚 Bibliografía: 133 fuentes verificadas (2026-07-29)** — 128 en el cuerpo bibliográfico (Secciones 1–12, 14–18) + 5 enlaces de documentación oficial. Deuda #4 cerrada; detalle en `Instrucciones-Guia-Maestra-DS.md` §5. 10 de las 116 originales se agregaron el 2026-07-27 en el primer escaneo de vanguardia (Tomos 12, 13, 19 y 20); **12 más se agregaron el 2026-07-29** en el barrido de Series de Tiempo (Tomos 17, 08, 10 y 12).

---

**Navegación:** Siguiente tomo → [[01-Introduccion-Ejecutiva|01 · Introducción Ejecutiva]]
