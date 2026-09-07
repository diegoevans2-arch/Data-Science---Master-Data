---
title: "Tomo 22 — Feature Engineering Avanzado"
tags: [data-science, machine-learning, feature-engineering, features, tabular]
audiencias: [tecnico, puente, ejecutivo]
tomo: 22
version: 1.2
updated: 2026-09-06
---

# 🔩 Tomo 22 — Feature Engineering Avanzado

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[21-Supervivencia-y-Bandits|21 · Supervivencia y Bandits]] · Siguiente: [[23-Tabular-DL-vs-Boosting|23 · Tabular DL vs Boosting ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> En datos tabulares, feature engineering sigue siendo **la palanca más rentable de mejora** — por encima de cambiar de algoritmo o tunear hiperparámetros (Domingos, 2012). El T03 introduce el concepto y las técnicas básicas; este tomo profundiza: familias de transformaciones, patrones por dominio, aggregation windows, interacciones, y el flujo iterativo que conecta EDA → features → model → importancia → más features.

> [!abstract] 👔 Impacto ejecutivo
> La diferencia entre un modelo "decente" y uno que mueve KPIs no suele ser el algoritmo — es la calidad de lo que le das de comer.
>
> - **Decisiones que habilita:** invertir el esfuerzo del equipo donde más rinde (features, no GPU), dimensionar el conocimiento de dominio requerido, priorizar qué datos capturar.
> - **Costo de hacerlo mal:** modelos ciegos a patrones que un analista de negocio ve a simple vista, meses de tuning que una sola feature hubiera resuelto, y features que no existirán en producción (leakage temporal).
> - **Pregunta ejecutiva que responde:** *¿el equipo está extrayendo toda la señal posible de los datos que ya tenemos, o estamos pagando por más datos que no vamos a saber usar?*

> [!tip] 💡 Analogía general
> Los datos crudos son harina, huevos y mantequilla. El modelo es un comensal que solo acepta platos preparados. Feature engineering es la cocina: ratios, promedios móviles, conteos, diferencias — las recetas que convierten ingredientes en algo que el comensal puede digerir. Un buen chef (domain expert + data scientist) saca 10 platos de los mismos 3 ingredientes; un mal chef quema la harina.

> [!example] 📊 Caso de negocio — Educación: la feature que ningún modelo encontraba solo
> **Problema:** una universidad predice deserción estudiantil con features demográficas y de rendimiento. El modelo (LightGBM tuneado con Optuna) llega a AUC 0.74 y se estanca — más tuning no ayuda, más datos tampoco.
>
> **Técnica aplicada:** en sesión de feature engineering con el equipo académico, se crean: (1) `ratio_aprobacion_ultimos_2_semestres` (captura tendencia, no nivel); (2) `dias_desde_ultima_actividad_plataforma` (engagement reciente); (3) `n_asignaturas_repetidas_acumulado` (patrón de "arrastre"). Ninguna existía en los datos crudos; todas se derivan de datos que ya tenían.
>
> **Resultado:** AUC sube a 0.83 sin cambiar modelo ni hiperparámetros. La feature #2 (actividad en plataforma) se convierte en trigger de intervención temprana: el sistema alerta cuando un estudiante lleva más de 14 días sin actividad. Feature engineering convirtió un modelo "decente" en una herramienta de gestión.

---

## 1. El ciclo iterativo de feature engineering

Audiencia: 🔧 🧭

```
 ┌─────────────────────────────────────────────────────────┐
 │                                                         │
 │  EDA (T04)                                              │
 │  ¿qué patrones hay? ¿qué relaciones con el target?      │
 │                                                         │
 └──────────────────────────┬──────────────────────────────┘
                            │
                            ▼
 ┌──────────────────────────────────────────────────────────┐
 │  CREAR FEATURES                                          │
 │  Conocimiento de dominio + patrones del EDA              │
 │  → aggregations, ratios, lags, interacciones, flags      │
 └──────────────────────────┬───────────────────────────────┘
                            │
                            ▼
 ┌──────────────────────────────────────────────────────────┐
 │  ENTRENAR + EVALUAR (T07, T08, T10)                      │
 │  ¿Mejoró la métrica? ¿Empeoró? ¿Sin cambio?              │
 └──────────────────────────┬───────────────────────────────┘
                            │
                            ▼
 ┌──────────────────────────────────────────────────────────┐
 │  FEATURE IMPORTANCE (T11 §4.1)                           │
 │  ¿Cuáles features nuevas aportaron? ¿Cuáles sobran?      │
 │  ¿Las top-5 sugieren nuevas derivaciones?                │
 └──────────────────────────┬───────────────────────────────┘
                            │
                            └───────────── volver arriba ──►
```

**🧭 La regla de oro:** feature engineering no es un paso que se hace una vez — es un **loop** que se repite hasta que la importancia de las features nuevas se aplana (rendimientos decrecientes). Cada iteración es: hipótesis → crear feature → evaluar impacto → decidir.

---

## 2. Familias de transformaciones

Audiencia: 🔧 🧭

### 2.1 Aggregation windows (ventanas de agregación)

> [!tip] 💡 Analogía
> No miras un solo partido para evaluar a un jugador — miras los últimos 5, los últimos 10, la temporada entera. Las ventanas de agregación hacen lo mismo con tus datos: resumen el comportamiento reciente, medio y largo plazo de cada entidad.

**🔧 Definición técnica:** calcular estadísticos (mean, sum, count, min, max, std) sobre una ventana temporal o de registros previos, **agrupados por entidad**.

| Tipo de ventana | Ejemplo | Qué captura |
|---|---|---|
| Últimos N días/meses | `monto_promedio_ultimos_90_dias` | Comportamiento reciente — la señal más fresca |
| Rolling (fija) | `std_transacciones_rolling_7d` | Volatilidad / estabilidad reciente |
| Expanding (acumulada) | `total_compras_historico` | Nivel absoluto / tenure del cliente |
| Diferencias entre ventanas | `gasto_30d - gasto_90d/3` | Tendencia: ¿está acelerando o frenando? |
| Ratio ventana/total | `compras_7d / compras_historico` | Concentración reciente (¿se activó?) |

> [!danger] 🚨 La trampa temporal de las ventanas
> Cada ventana debe calcularse **con datos disponibles en el momento de la predicción**. Si predices churn al 1 de marzo, la ventana "últimos 30 días" solo puede usar feb 1–28. Usar datos posteriores al punto de predicción es leakage temporal ([[10-Validacion-y-Leakage]]).

**🔧 Cómo se resuelve en producción — point-in-time correctness.** La forma estandarizada de garantizar que cada ventana use solo información disponible al momento de la predicción es el **point-in-time join**: se parte de una tabla de entidades con su timestamp de predicción y, para cada fila, se recupera el valor de cada feature vigente **a esa fecha**, mirando hacia atrás hasta un máximo (TTL) y, opcionalmente, filtrando por el timestamp de creación del registro para que correcciones o backfills posteriores no contaminen el histórico. Los **feature stores** (Feast es el proyecto open source de referencia) implementan ese join y reutilizan la misma definición de la feature para entrenar (offline) y para servir (online), lo que evita el *training-serving skew* (documentación oficial de Feast, consultada 2026-09-06). Si tu pipeline calcula ventanas «a mano» en un notebook, el test mínimo es reproducir el valor de una feature para una fecha pasada y comparar. Arquitectura y herramientas en [[13-MLOps-XAI-Etica]].

### 2.2 Ratios e interacciones

Audiencia: 🔧

**🔧 Definición técnica:** combinar dos o más features en una nueva que captura una relación que ninguna de las originales expresa sola.

| Tipo | Ejemplo | Por qué funciona |
|---|---|---|
| Ratio | `deuda / ingreso` | El monto de deuda absoluto no dice nada sin saber cuánto gana |
| Diferencia | `precio_actual - precio_hace_30d` | Captura tendencia que las dos features por separado no expresan |
| Producto (interacción) | `edad × antigüedad_laboral` | Para modelos lineales que no capturan interacciones automáticamente |
| Flag binario | `1 si (saldo < 0 AND dias_sin_pago > 30)` | Codifica una regla de negocio conocida como feature |
| Frecuencia relativa | `compras_cat_A / total_compras` | Share de una categoría sobre el total — normalización por comportamiento |

**🧭 Cuándo crear interacciones:** (1) cuando el EDA muestra que el target depende de la **combinación** de dos variables, no de cada una por separado; (2) para modelos lineales y logísticos que no capturan interacciones por diseño; (3) cuando un experto de dominio dice "lo que importa es X relativo a Y".

### 2.3 Features temporales

Audiencia: 🔧

| Feature | Derivada de | Qué captura |
|---|---|---|
| Hora / día de semana / mes | Timestamp | Patrones cíclicos (→ encoding sin/cos, [[05-Escalado-de-Datos]]) |
| Días desde último evento | Timestamp + evento | Recencia — la feature más poderosa en muchos dominios (RFM) |
| Días hasta próximo evento conocido | Fecha futura conocida (vencimiento, renovación) | Urgencia / ventana de oportunidad |
| Velocidad de cambio | `(valor_t - valor_t-1) / Δt` | Aceleración vs estancamiento |
| Lag features | `valor en t-1, t-2, ..., t-n` | Autocorrelación directa — esencial para series ([[17-Series-de-Tiempo]]) |
| Conteo de eventos en ventana | Count en los últimos N días | Frecuencia de comportamiento |

### 2.4 Features de texto y categorías de alta cardinalidad

Audiencia: 🔧

| Técnica | Input | Output | Cuándo usarla |
|---|---|---|---|
| Count/TF-IDF de keywords | Texto libre | Features numéricas sparse | Modelos tabulares que no aceptan texto crudo |
| Embeddings (sentence-transformers) | Texto libre | Vector denso (384–768 dims) | Modelos downstream que aceptan embeddings; complementar con PCA para reducir dims |
| Target encoding | Categórica alta cardinalidad | Numérica continua | Códigos postales, SKUs, IDs de productos — con folds para evitar leakage [[10-Validacion-y-Leakage]] |
| Frequency encoding | Categórica | Numérica (% del total) | Cuando la popularidad de la categoría es señal (ciudades, marcas) |
| Entity embeddings (DL) | Categórica | Vector denso aprendido | Tabular DL (TabM, RealMLP, FT-Transformer; [[23-Tabular-DL-vs-Boosting]]); transferibles entre tareas del mismo dominio |

**🔧 Nota sobre target encoding (2023+).** Los «folds para evitar leakage» ya no requieren implementación artesanal: el `TargetEncoder` de scikit-learn (desde la versión 1.3) aplica **cross fitting** interno al ajustar-y-transformar — cada fold se codifica con las estadísticas de los otros k−1 — y su documentación advierte que ajustar y transformar por separado sobre el mismo set de entrenamiento **sí** filtra el target y está desaconsejado. Además aplica **shrinkage** hacia la media global, ponderado por el tamaño de cada categoría (Micci-Barreca, 2001), con suavizado automático de tipo Bayes empírico; esto protege a las categorías raras (SKUs o códigos postales con pocas filas). La alternativa integrada al modelo son las *ordered target statistics* de CatBoost (Prokhorenkova et al., 2018). Regla práctica: si tu encoder no hace cross fitting ni suaviza, la validación de [[10-Validacion-y-Leakage]] sobreestimará el desempeño en las categorías poco frecuentes.

---

## 3. Patrones por dominio

Audiencia: 🧭 👔

| Dominio | Features clásicas de alto impacto | Fuente de conocimiento |
|---|---|---|
| **Banca / Fintech** | Ratio deuda/ingreso, velocidad de gasto, conteo de productos contratados, días desde último pago, % de utilización de línea | Oficial de crédito + reglas de riesgo |
| **Retail / E-commerce** | RFM (Recencia, Frecuencia, Monto), share de categoría, ticket promedio, tendencia de gasto, hora de compra | Equipo comercial + analistas CRM |
| **Telecomunicaciones** | Minutos/datos consumidos vs plan, ratio quejas/meses, cambios de plan recientes, tenure, indicadores de red | Equipo de retención + ingeniería de red |
| **Salud** | Comorbilidades contadas, tendencia de marcadores en últimos 3 exámenes, días entre controles, adherencia a medicación | Equipo clínico + epidemiólogos |
| **Educación** | Tasa de aprobación por periodo, créditos acumulados vs esperados, actividad en plataforma (recencia + frecuencia), asignaturas repetidas | Equipo académico + registrar |
| **Industria / IoT** | Estadísticos de sensores en ventanas (mean, std, max), cruces de umbrales, tiempo desde última alerta, features frecuenciales (FFT) | Ingenieros de proceso + mantenimiento |

**👔 En una frase para el negocio:** el conocimiento de dominio se convierte en ventaja competitiva cuando se traduce a features — el AutoML clásico ([[11-Mejora-de-Modelos]] §2.1) sigue sin inventar lo que solo el experto del negocio sabe.

**🔧 Matiz 2023+ — los LLMs como generadores de hipótesis.** Un LLM ya produce features semánticamente significativas a partir de la descripción del dataset y los nombres de columna: CAAFE mejoró 11 de 14 datasets de benchmark y subió el ROC AUC medio de 0.798 a 0.822 (Hollmann, Müller & Hutter, 2023). Lo que aporta es conocimiento **genérico** del dominio (un índice de masa corporal, un ratio deuda/ingreso), no las reglas propias del negocio ni la disponibilidad temporal de cada columna, que el modelo no conoce. Úsalo como generador de hipótesis para el loop de §1, con revisión humana de cada feature (regla 1 de §4). Como herramienta madura todavía no existe: una revisión de 53 métodos de feature engineering automático los encontró difíciles de usar, sin documentación y sin comunidades activas (Schäfer et al., 2025; preprint).

---

## 4. Reglas de disciplina

Audiencia: 🔧 🧭

> [!warning] ⚠️ Las 5 reglas que evitan que feature engineering se convierta en feature disaster
>
> 1. **Disponibilidad temporal:** toda feature debe existir **en el momento de la predicción**. Si predices hoy, solo puedes usar datos de ayer o antes. Verificar con el test de: "¿tendré esta columna llena cuando el modelo corra en producción a las 6am?" ([[10-Validacion-y-Leakage]])
>
> 2. **Documentar cada feature:** nombre, fórmula, fecha de creación, hipótesis detrás, resultado de la evaluación. Un notebook con 200 features sin documentación es deuda técnica explosiva.
>
> 3. **Una feature a la vez (idealmente):** para entender causalidad entre la feature nueva y la mejora de la métrica, agregarlas de a una o en lotes temáticos pequeños. Si agregas 50 de golpe y sube 2 puntos, no sabes cuáles aportaron.
>
> 4. **Feature selection post-creación:** crear muchas, evaluar importancia, **podar** las que no aportan. Más features ≠ mejor modelo — puede empeorar por ruido, multicolinealidad y costo de cómputo ([[03-Preparacion-de-Datos]] §7). **Matiz:** el costo en precisión de las features ruidosas depende del modelo. En el benchmark de referencia, los modelos de árboles resultaron robustos a las features no informativas, mientras que las redes tipo MLP se degradan con ellas — una de las tres razones por las que los árboles siguen ganando en tabular (Grinsztajn et al., 2022; [[23-Tabular-DL-vs-Boosting]]). Consecuencia: con gradient boosting puedes probar lotes de features candidatas sin temer un derrumbe de la métrica, y la poda se justifica sobre todo por costo, mantenimiento, estabilidad e interpretabilidad; con modelos lineales o redes, la poda sí protege la precisión y debe ser más estricta.
>
> 5. **Reproducibilidad en el pipeline:** cada feature se calcula dentro del Pipeline de producción, no en un notebook aparte. Si la feature requiere un query de 5 tablas, ese query debe estar versionado y automatizado ([[13-MLOps-XAI-Etica]]). Si la feature vive en un feature store, la definición es una sola para offline y online (§2.1).

---

## 5. El orden de prioridades — dónde invertir primero

Audiencia: 🧭 👔

```
 Impacto típico (datos tabulares, 2026)
 ─────────────────────────────────────────────────
 1. Features de dominio bien pensadas       ████████████ (~60% de la mejora)
 2. Aggregation windows + lags              ██████       (~20%)
 3. Interacciones y ratios                  ███          (~10%)
 4. Encoding avanzado (target, embeddings)  ██           (~7%)
 5. Transformaciones de distribución        █            (~3%)
 ─────────────────────────────────────────────────
 Nota: porcentajes orientativos — criterio editorial de la guía, no una
 medición publicada — y varían por proyecto.
 El mensaje: features de dominio primero, siempre.
```

---

## 📖 Referencias de este tomo

- (Domingos, 2012) — *A Few Useful Things to Know about Machine Learning*. CACM 55(10), 78–87. "The most important factor is the features used."
- (Zheng & Casari, 2018) — *Feature Engineering for Machine Learning: Principles and Techniques for Data Scientists*. O'Reilly Media. El manual práctico de referencia.
- (Kuhn & Johnson, 2019) — *Feature Engineering and Selection: A Practical Approach for Predictive Models*. Chapman & Hall/CRC.
- (Breiman, 2001a) — permutation importance como guía para feature engineering iterativo.
- (Micci-Barreca, 2001) — *A preprocessing scheme for high-cardinality categorical attributes in classification and prediction problems*. ACM SIGKDD Explorations 3(1). El shrinkage del target encoding.
- (Prokhorenkova et al., 2018) — CatBoost: *ordered target statistics* como alternativa integrada al modelo ([[07-Modelos-Supervisados]]).
- (Hollmann, Müller & Hutter, 2023) — *Large Language Models for Automated Data Science: Introducing CAAFE for Context-Aware Automated Feature Engineering*. NeurIPS 2023.
- (Schäfer et al., 2025) — *How Usable is Automated Feature Engineering for Tabular Data?* arXiv 2508.13932 — **preprint** (short paper de un track no archivado de AutoML 2025), sin revisión por pares formal.
- (Grinsztajn et al., 2022) — robustez de los árboles a las features no informativas ([[23-Tabular-DL-vs-Boosting]]).
- Documentación oficial: `TargetEncoder` de scikit-learn y *point-in-time joins* de Feast → [[16-Bibliografia]] §13.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[21-Supervivencia-y-Bandits|21 · Supervivencia y Bandits]] · Siguiente: [[23-Tabular-DL-vs-Boosting|23 · Tabular DL vs Boosting ➡]]

> **Conexiones clave:** [[03-Preparacion-de-Datos]] (encoding, feature selection) · [[04-EDA]] (hallazgos que disparan features) · [[05-Escalado-de-Datos]] (transformación cíclica sin/cos) · [[10-Validacion-y-Leakage]] (disponibilidad temporal) · [[11-Mejora-de-Modelos]] §4.1 (importance como guía) · [[17-Series-de-Tiempo]] (lags y ventanas)
