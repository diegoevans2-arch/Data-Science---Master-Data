---
title: "Tomo 23 — Datos Tabulares en 2026: Gradient Boosting vs Deep Learning"
tags: [data-science, machine-learning, tabular, boosting, deep-learning, tabpfn, xgboost]
audiencias: [tecnico, puente, ejecutivo]
tomo: 23
version: 1.0
updated: 2026-08-28
---

# 🏋️ Tomo 23 — Datos Tabulares en 2026: Gradient Boosting vs Deep Learning

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[22-Feature-Engineering-Avanzado|22 · Feature Engineering Avanzado]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> La pregunta "¿uso XGBoost o una red neuronal para mis datos tabulares?" es una de las más frecuentes en la práctica — y la respuesta cambió entre 2022 y 2026. Los árboles siguen dominando la mayoría de escenarios, pero los **tabular foundation models** (TabPFN, TabICL) y las **arquitecturas DL especializadas** (FT-Transformer, TabNet) están cerrando la brecha. Este tomo pone orden: cuándo cada familia gana, por qué, y cuál es el estado real del benchmark.

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

**🔧 El panorama empírico actual (fuentes: TabArena, Grinsztajn et al. 2022, Hollmann et al. 2026):**

| Franja de datos | Campeón 2022 | Campeón 2026 | Evidencia |
|---|---|---|---|
| **Pequeño (N ≤ 1K)** | RF / XGBoost | **TabPFN-3** (sin entrenar, inference <1s) | 100% win rate vs default XGBoost en clasificación ≤10K (Hollmann et al., 2026) |
| **Mediano (1K–10K)** | XGBoost tuneado | **TabPFN-2.5** ≈ AutoGluon (4h tuning) | TabArena benchmark; empate con el ensemble más caro (Hollmann et al., 2026) |
| **Grande (10K–100K)** | XGBoost / LightGBM tuneado | XGBoost / LightGBM tuneado; TabPFN competitivo (87% win rate vs default XGBoost) | Nature 2026: TabPFN v2.6 y árboles tuneados empatados en datos clínicos MIMIC-IV |
| **Muy grande (>100K)** | XGBoost / LightGBM / CatBoost | **XGBoost / LightGBM / CatBoost** | TabPFN no escala; DL especializado (FT-Transformer) empata pero no supera de forma consistente |

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
- **Transfer learning tabular**: pre-entrenar en un dataset grande de la misma distribución y fine-tunear en el target pequeño (raro en la práctica, pero es la promesa de TabPFN).

---

## 3. Las familias de modelos DL para tabular

Audiencia: 🔧 🧭

| Familia | Representantes | Mecanismo clave | Fortaleza | Limitación |
|---|---|---|---|---|
| **Attention-based** | FT-Transformer (Gorishniy et al., 2021), TabTransformer | Self-attention entre features (cada feature "mira" a las demás) | Captura interacciones de alto orden automáticamente; escala a datasets grandes | Más lento de entrenar; necesita tuning cuidadoso; ventaja inconsistente vs boosting |
| **Gating-based** | TabNet (Arık & Pfister, 2021) | Attention secuencial con masks que seleccionan features step-by-step | Feature selection intrínseca (interpretable); puede entrenar end-to-end | Inestable en entrenamiento; raramente supera a XGBoost tuneado en benchmarks independientes |
| **Foundation models** | TabPFN (Hollmann et al., 2023), TabPFN-3 (2026), TabICL | Transformer pre-entrenado en datos sintéticos; in-context learning (no entrena, solo infiere) | **Zero-shot**: inference en <1s sin entrenar; excelente en datasets pequeños | No escala >100K filas; caja negra total; sensible al tamaño del context |
| **Hybrid tree+DL** | NODE (Popov et al., 2020), GrowNet | Redes que emulan splits/ensembles diferenciables | Combina lo mejor de ambos mundos en teoría | Complejidad de implementación alta; no dominan benchmarks |

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
  │    │                              Si quieres explorar DL: FT-Transformer
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

2. **TabPFN para datasets chicos.** Si N ≤ 10K y tienes muchos datasets que evaluar (screening), TabPFN ahorra días de tuning.

3. **No confíes en un solo paper.** Los benchmarks de DL tabular suelen estar cherry-picked. Usa TabArena o tu propio benchmark con cross-validation honesta ([[10-Validacion-y-Leakage]]).

4. **El feature engineering sigue mandando.** Antes de cambiar de modelo, invierte en features ([[22-Feature-Engineering-Avanzado]]). Un XGBoost con buenas features supera a un FT-Transformer con features malas.

5. **Producción pesa.** Un XGBoost hace inference en microsegundos; un Transformer tabular necesita GPU. Si la latencia es restricción, el árbol gana por default.

6. **La frontera se mueve rápido.** TabPFN-3 (mayo 2026) ya supera a TabPFN-2.5 significativamente. Revisar cada 6 meses; pero la evidencia de 2022–2026 es consistente: en tabular, los árboles son la apuesta segura.

---

## 📖 Referencias de este tomo

- (Grinsztajn et al., 2022) — *Why do tree-based models still outperform deep learning on typical tabular data?* NeurIPS 2022. El benchmark de referencia (45 datasets, conclusión: árboles ganan en datasets medianos).
- (Hollmann et al., 2023) — *TabPFN: A Transformer That Solves Small Tabular Classification Problems in a Second*. ICLR 2023.
- (Hollmann et al., 2026) — *Advancing the State of the Art in Tabular Foundation Models* (TabPFN-2.5/3). arXiv 2511.08667.
- (Gorishniy et al., 2021) — *Revisiting Deep Learning Models for Tabular Data* (FT-Transformer). NeurIPS 2021.
- (Arık & Pfister, 2021) — *TabNet: Attentive Interpretable Tabular Learning*. AAAI.
- (McElfresh et al., 2023) — *When Do Neural Nets Outperform Boosted Trees on Tabular Data?* NeurIPS 2023. (Meta-análisis: NNs ganan en datasets grandes + numéricos + sin outliers extremos).
- (Erickson et al., 2020) — AutoGluon como referencia de ensemble multi-modelo ([[11-Mejora-de-Modelos]] §2.1).

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[22-Feature-Engineering-Avanzado|22 · Feature Engineering Avanzado]]

> **Conexiones clave:** [[07-Modelos-Supervisados]] (árboles y boosting) · [[11-Mejora-de-Modelos]] §2.1 (AutoML/TabPFN como benchmark) · [[12-Deep-Learning]] (arquitecturas de redes) · [[22-Feature-Engineering-Avanzado]] (la palanca que manda antes del modelo)
