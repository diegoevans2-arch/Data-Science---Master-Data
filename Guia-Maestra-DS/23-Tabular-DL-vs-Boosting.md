---
title: "Tomo 23 — Datos Tabulares en 2026: Gradient Boosting vs Deep Learning"
tags: [data-science, machine-learning, tabular, boosting, deep-learning, tabpfn, xgboost]
audiencias: [tecnico, puente, ejecutivo]
tomo: 23
version: 1.2
updated: 2026-09-06
---

# 🏋️ Tomo 23 — Datos Tabulares en 2026: Gradient Boosting vs Deep Learning

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[22-Feature-Engineering-Avanzado|22 · Feature Engineering Avanzado]] · Siguiente: [[24-Experimentacion-AB|24 · Experimentación A/B ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> La pregunta "¿uso XGBoost o una red neuronal para mis datos tabulares?" es una de las más frecuentes en la práctica — y la respuesta cambió entre 2022 y 2026. Los árboles siguen dominando la mayoría de escenarios, pero los **tabular foundation models** (TabPFN, TabICL) y las **arquitecturas DL especializadas** (hoy, los MLP modernos como TabM y RealMLP; FT-Transformer y TabNet quedaron como referencia) están cerrando la brecha. Este tomo pone orden: cuándo cada familia gana, por qué, y cuál es el estado real del benchmark.

> [!abstract] 👔 Impacto ejecutivo
> Elegir mal entre boosting y DL tabular no solo cuesta rendimiento — cuesta tiempo de ingeniería, latencia de inferencia y complejidad de deploy.
>
> - **Decisiones que habilita:** justificar con evidencia si vale la pena experimentar con DL en tu dataset tabular, o si XGBoost/LightGBM es la apuesta correcta.
> - **Costo de hacerlo mal:** semanas de experimentación con redes neuronales que al final empatan con un XGBoost tuneado en 20 minutos, o descartar DL tabular sin saber que para tu caso específico (pocos datos, muchas features numéricas) TabPFN gana sin tuning.
> - **Pregunta ejecutiva que responde:** *¿hay un argumento técnico sólido para invertir en infraestructura DL para este problema tabular, o el boosting es suficiente?*

> [!tip] 💡 Analogía general
> Los árboles de decisión son artesanos: lentos para aprender cada oficio (entrenar), pero una vez que lo aprenden son rápidos, confiables y se explican solos. Las redes neuronales son polímatas: pueden aprender CUALQUIER cosa, pero necesitan mucha práctica (datos) y un taller caro (GPUs). Los tabular foundation models (TabPFN) son el polímata que ya hizo la práctica en millones de problemas sintéticos: llega y resuelve tu problema nuevo sin entrenar — pero no escala a talleres gigantes.

---

## 1. El benchmark definitivo — estado del arte a agosto 2026

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El ranking Elo del ajedrez pero para modelos tabulares: cada modelo "juega" contra los demás en cientos de datasets, y se le asigna un rating. Quien gana consistentemente sube; quien empata mantiene; quien pierde baja.

**🔧 El panorama empírico actual (fuentes: Grinsztajn et al., 2022; Hollmann et al., 2025; el benchmark vivo TabArena — Erickson et al., 2025; los reportes técnicos del fabricante se marcan como tales):**

| Franja de datos | Campeón 2022 | Campeón 2026 | Evidencia |
|---|---|---|---|
| **Pequeño (N ≤ 1K)** | RF / XGBoost | **TabPFN v2** (sin entrenar, inference en segundos) | *Nature* 2025: supera a los árboles tuneados en datasets pequeños (Hollmann et al., 2025); TabArena confirma que los foundation models destacan en datasets pequeños (Erickson et al., 2025) |
| **Mediano (1K–10K)** | XGBoost tuneado | **TabPFN v2 + tuning + ensembling** ≥ AutoGluon (4 h) | TabArena, subconjunto ≤ 10K filas: TabPFN v2 con tuning y post-hoc ensembling supera a AutoGluon; **sin** ese ensembling, el mejor con tuning convencional es CatBoost (Erickson et al., 2025) |
| **Grande (10K–100K)** | XGBoost / LightGBM tuneado | XGBoost / LightGBM / CatBoost tuneado; el DL alcanza con más presupuesto | TabArena: los árboles siguen siendo contendientes fuertes y el DL los alcanzó con presupuestos mayores y ensembling, sin superarlos de forma consistente (Erickson et al., 2025). El DL que empata es el **MLP moderno** (TabM, RealMLP): Gorishniy et al. (2025) muestran que supera a las arquitecturas de attention y retrieval. TabPFN v2 está limitado a ≤ 10K filas por diseño; TabICL cubre este rango sin entrenar (Qu et al., 2025) |
| **Muy grande (>100K)** | XGBoost / LightGBM / CatBoost | **XGBoost / LightGBM / CatBoost** | Los árboles siguen arriba, pero ya no porque los foundation models «no lleguen»: TabICL (Qu et al., 2025) maneja hasta 500K filas con recursos modestos y, en 53 datasets de más de 10K filas, supera a TabPFN v2 y a CatBoost. En tablas grandes y de alta dimensión la evidencia independiente más reciente sigue dando la ventaja a árboles y DL clásico (Purucker et al., 2026, preprint); el DL especializado empata pero no supera de forma consistente (Gorishniy et al., 2021; McElfresh et al., 2023). El techo de 10K filas es una decisión de diseño de TabPFN v2 (Hollmann et al., 2025), no un límite de la familia |

> [!warning] ⚠️ Lo que declara el fabricante no es lo que midió el árbitro
> Prior Labs publica versiones nuevas cada pocos meses: **TabPFN-2.5** (Grinsztajn et al., 2025 — reporte técnico en arXiv, sin revisión por pares), **TabPFN-2.6** (marzo de 2026, solo pesos publicados en Hugging Face, sin paper) y **TabPFN-3** (Grinsztajn et al., 2026 — reporte técnico que declara "hasta 20× más rápido que 2.5"). Sus cifras de *win rate* contra XGBoost por defecto y su liderazgo en TabArena son **afirmaciones del vendedor**: el paper independiente de TabArena (NeurIPS 2025) evaluó TabPFN **v2**, no estas versiones. Hasta que el leaderboard vivo o una evaluación independiente lo confirme, trátalas como práctica reportada por el fabricante, no como consenso. Y la comparación honesta es siempre contra un boosting **tuneado**, no contra el default.

**🔧 Resultado clave:** los árboles **no** han sido destronados en datasets grandes. La revolución está en datasets **pequeños y medianos**, donde los foundation models tabulares hacen inference sin entrenar y alcanzan o superan a los árboles tuneados.

---

## 2. ¿Por qué los árboles dominan datos tabulares? — los inductive biases

Audiencia: 🔧

> [!tip] 💡 Analogía
> Los árboles tienen "instinto" para datos tabulares como un perro pastor tiene instinto para ovejas: están naturalmente equipados para features heterogéneas, outliers, y relaciones no suaves. Las redes neuronales tienen instinto para datos con **estructura espacial o secuencial** (imágenes, texto) — en tabular necesitan que les enseñes artificialmente lo que un árbol ya sabe.

**🔧 Los 3 inductive biases que explican la ventaja (Grinsztajn et al., 2022):**

| Bias del árbol | Qué le da | Por qué las NNs luchan |
|---|---|---|
| **Invarianza a rotación de features** | Cada split usa UNA feature a la vez — no necesita que las features estén correlacionadas o en la misma escala | Las NNs operan con combinaciones lineales de todas las features: necesitan normalización y son sensibles a features irrelevantes |
| **Manejo nativo de irregularidad** | Features con outliers, distribuciones no suaves, mezcla de categóricas y numéricas → el árbol las maneja sin transformar | Las NNs asumen suavidad implícita; datos "ruidosos" o heterogéneos degradan el gradiente |
| **Aprendizaje orientado a features, no a muestras** | Cada split decide qué feature importa en qué región; features irrelevantes no se usan | Las NNs procesan TODAS las features en cada capa; necesitan regularización explícita (dropout, weight decay) para ignorar ruido |

**🔧 Cuándo las NNs SÍ ganan en tabular:**

- Datasets **muy grandes** (>100K–1M) con muchas features **numéricas, sin outliers extremos y con relaciones suaves** — el volumen compensa la falta de inductive biases.
- Datos con **sub-estructura aprendible**: series de tiempo embebidas como features, texto embeddeable, imágenes como features auxiliares → el DL integra modalidades que el árbol no puede.
- **Transfer learning tabular**: hay dos mecanismos que conviene no confundir. Los foundation models (TabPFN, TabICL) **no** se pre-entrenan en tablas reales de tu dominio ni se fine-tunean: se meta-entrenan en millones de tablas **sintéticas** y resuelven cada dataset nuevo por in-context learning, con la tabla completa como contexto (Hollmann et al., 2025; Qu et al., 2025). La transferencia entre tablas reales heterogéneas — columnas distintas, vocabularios distintos — es otro problema, y su respuesta más citada es CARTE (Kim et al., 2024): representa cada fila como un grafo y embebe nombres de columna y strings con vocabulario abierto, lo que permite pre-entrenar en una fuente y reutilizar en tablas que no comparten esquema.

---

## 3. Las familias de modelos DL para tabular

Audiencia: 🔧 🧭

| Familia | Representantes | Mecanismo clave | Fortaleza | Limitación |
|---|---|---|---|---|
| **Attention-based** | FT-Transformer (Gorishniy et al., 2021), TabTransformer | Self-attention entre features (cada feature "mira" a las demás) | Captura interacciones de alto orden automáticamente; escala a datasets grandes | Más lento de entrenar; necesita tuning cuidadoso; ventaja inconsistente vs boosting |
| **Gating-based** | TabNet (Arık & Pfister, 2021) | Attention secuencial con masks que seleccionan features step-by-step | Feature selection intrínseca (interpretable); puede entrenar end-to-end | Inestable en entrenamiento; raramente supera a XGBoost tuneado en benchmarks independientes |
| **MLP moderno** | RealMLP (Holzmüller et al., 2024), TabM (Gorishniy et al., 2025) | MLP con defaults meta-aprendidos y preprocesamiento robusto (RealMLP) o que imita un ensemble de MLPs compartiendo parámetros (TabM) | Es hoy la línea de DL más fuerte y práctica: más simple y rápida que la attention; es el representante DL de TabArena y del preset `extreme_quality` de AutoGluon ([[11-Mejora-de-Modelos]] §2.1) | Necesita tuning y ensembling para rendir; con defaults queda por debajo de los árboles (Erickson et al., 2025) |
| **Foundation models** | TabPFN v2 (Hollmann et al., 2025; v1: Hollmann et al., 2023), TabPFN-2.5/3 (reportes técnicos del fabricante, 2025–2026), TabICL (Qu et al., 2025) | Transformer pre-entrenado en datos sintéticos; in-context learning (no entrena, solo infiere) | **Zero-shot**: inference en <1s sin entrenar; excelente en datasets pequeños | TabPFN v2 no pasa de 10K filas por diseño; TabICL llega a 500K (Qu et al., 2025), pero por sobre eso y en alta dimensión los árboles siguen mandando; caja negra total; sensible al tamaño del context |
| **Hybrid tree+DL** | NODE (Popov et al., 2020), GrowNet | Redes que emulan splits/ensembles diferenciables | Combina lo mejor de ambos mundos en teoría | Complejidad de implementación alta; no dominan benchmarks |

> [!warning] ⚠️ Dónde se rompen los foundation models tabulares
> Casi todos los benchmarks que los coronan son **i.i.d.**: split aleatorio, tabla pequeña o mediana, columnas numéricas y categóricas limpias. La evaluación más amplia fuera de ese cuadro — BeyondArena, 142 datasets y 11 modelos con splits temporales, por grupos, tablas grandes, alta dimensión, texto libre y alta cardinalidad — encuentra que los foundation models destacan en lo i.i.d. pequeño y mediano, y que **árboles y DL tradicional dominan** en lo no i.i.d., lo grande y lo de alta dimensión (Purucker et al., 2026; **preprint** de junio de 2026, sin revisión por pares: léelo como señal, no como veredicto). El caso del **drift temporal** sí tiene respaldo revisado por pares: TabPFN asume i.i.d. y existe una variante entrenada para cambios de distribución en el tiempo, Drift-Resilient TabPFN (Helli et al., 2024). Regla: si tu validación es temporal o por grupos ([[10-Validacion-y-Leakage]]), no traslades el ranking de un benchmark aleatorio — mide en **tu** split.

---

## 4. Guía de decisión — cuándo usar qué

Audiencia: 🧭 👔

```
 ¿Tu dataset es tabular puro (sin imágenes/texto/series)?
  │
  ├── SÍ
  │    │
  │    ├── N ≤ 10K filas? ─────────► TabPFN (inference directa, 0 tuning)
  │    │                              Si no disponible: XGBoost + Optuna
  │    │
  │    ├── 10K < N ≤ 100K? ────────► XGBoost / LightGBM + Optuna
  │    │                              Benchmark contra AutoGluon (T11 §2.1)
  │    │                              Sin entrenar: TabICL (hasta 500K filas)
  │    │                              Si quieres explorar DL: TabM o RealMLP
  │    │
  │    └── N > 100K? ──────────────► LightGBM / CatBoost (velocidad)
  │                                   DL solo si tienes features muy numéricas
  │                                   y sin outliers extremos
  │
  └── NO (multimodal: texto + tabular, imagen + tabular)
       │
       └──────────────────────────► DL end-to-end (embeddings + tabular head)
                                     O: embeddear texto/imagen por separado
                                     y alimentar como features a boosting
```

---

## 5. Reglas prácticas para el equipo

Audiencia: 🧭

1. **Start with boosting.** XGBoost/LightGBM con early stopping y 100 trials de Optuna es tu baseline en 20 minutos. Cualquier DL tabular debe **superar esto** para justificarse.

2. **TabPFN para datasets chicos.** Si N ≤ 10K y tienes muchos datasets que evaluar (screening), TabPFN ahorra días de tuning — siempre que los datos sean i.i.d.; con drift temporal, grupos o texto libre, vuelve al árbol y mide en tu propio split (Purucker et al., 2026).

3. **No confíes en un solo paper.** Los benchmarks de DL tabular suelen estar cherry-picked. Usa TabArena o tu propio benchmark con cross-validation honesta ([[10-Validacion-y-Leakage]]).

4. **El feature engineering sigue mandando.** Antes de cambiar de modelo, invierte en features ([[22-Feature-Engineering-Avanzado]]). Un XGBoost con buenas features supera a un TabM con features malas.

5. **Producción pesa.** Un XGBoost hace inference en microsegundos; un Transformer tabular necesita GPU. Si la latencia es restricción, el árbol gana por default.

6. **La frontera se mueve rápido.** TabPFN-3 (reporte técnico, mayo de 2026) declara ser hasta 20× más rápido que TabPFN-2.5 — afirmación del fabricante, aún sin evaluación independiente. Revisar cada 6 meses; pero la evidencia de 2022–2026 es consistente: en tabular, los árboles son la apuesta segura.

---

## 📖 Referencias de este tomo

- (Grinsztajn et al., 2022) — *Why do tree-based models still outperform deep learning on typical tabular data?* NeurIPS 2022. El benchmark de referencia (45 datasets, conclusión: árboles ganan en datasets medianos).
- (Hollmann et al., 2023) — *TabPFN: A Transformer That Solves Small Tabular Classification Problems in a Second*. ICLR 2023.
- (Hollmann et al., 2025) — *Accurate predictions on small data with a tabular foundation model* (TabPFN v2). *Nature*, 637, 319–326.
- (Erickson et al., 2025) — *TabArena: A Living Benchmark for Machine Learning on Tabular Data*. NeurIPS 2025 (Datasets and Benchmarks). El benchmark vivo independiente; leaderboard en tabarena.ai.
- (Grinsztajn et al., 2025) — *TabPFN-2.5: Advancing the State of the Art in Tabular Foundation Models*. arXiv 2511.08667 — reporte técnico de Prior Labs, sin revisión por pares. *(Corregido el 2026-09-05: la guía lo atribuía a "Hollmann et al., 2026"; el primer autor es Grinsztajn y la v1 es de noviembre de 2025.)*
- (Grinsztajn et al., 2026) — *TabPFN-3: Technical Report*. arXiv 2605.13986 — reporte técnico de Prior Labs, sin revisión por pares.
- (Popov et al., 2020) — *Neural Oblivious Decision Ensembles for Deep Learning on Tabular Data* (NODE). ICLR 2020.
- (Gorishniy et al., 2021) — *Revisiting Deep Learning Models for Tabular Data* (FT-Transformer). NeurIPS 2021.
- (Gorishniy et al., 2025) — *TabM: Advancing Tabular Deep Learning with Parameter-Efficient Ensembling*. ICLR 2025. El MLP moderno que supera a attention y retrieval.
- (Holzmüller et al., 2024) — *Better by default: Strong pre-tuned MLPs and boosted trees on tabular data* (RealMLP). NeurIPS 2024 ([[11-Mejora-de-Modelos]]).
- (Qu et al., 2025) — *TabICL: A Tabular Foundation Model for In-Context Learning on Large Data*. ICML 2025. El foundation model que llega a 500K filas.
- (Helli et al., 2024) — *Drift-Resilient TabPFN: In-Context Learning Temporal Distribution Shifts on Tabular Data*. NeurIPS 2024.
- (Purucker et al., 2026) — *Beyond IID: How General Are Tabular Foundation Models, Really?* (BeyondArena). arXiv 2606.30410 — **preprint sin revisión por pares**, junio de 2026.
- (Kim et al., 2024) — *CARTE: Pretraining and Transfer for Tabular Learning*. ICML 2024. Transferencia entre tablas con esquemas distintos.
- (Arık & Pfister, 2021) — *TabNet: Attentive Interpretable Tabular Learning*. AAAI.
- (McElfresh et al., 2023) — *When Do Neural Nets Outperform Boosted Trees on Tabular Data?* NeurIPS 2023. (Meta-análisis: NNs ganan en datasets grandes + numéricos + sin outliers extremos).
- (Erickson et al., 2020) — AutoGluon como referencia de ensemble multi-modelo ([[11-Mejora-de-Modelos]] §2.1).

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[22-Feature-Engineering-Avanzado|22 · Feature Engineering Avanzado]] · Siguiente: [[24-Experimentacion-AB|24 · Experimentación A/B ➡]]

> **Conexiones clave:** [[07-Modelos-Supervisados]] (árboles y boosting) · [[11-Mejora-de-Modelos]] §2.1 (AutoML/TabPFN como benchmark) · [[12-Deep-Learning]] (arquitecturas de redes) · [[22-Feature-Engineering-Avanzado]] (la palanca que manda antes del modelo)
