---
title: "Tomo 11 — Técnicas de Mejora de Modelos"
tags: [data-science, machine-learning, hiperparametros, ensembles, regularizacion, calibracion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 11
version: 6.2
updated: 2026-08-27
---

# 🚀 Tomo 11 — Técnicas de Mejora de Modelos

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[10-Validacion-y-Leakage|10 · Validación y Leakage]] · Siguiente: [[12-Deep-Learning|12 · Deep Learning ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> La mejora de modelos **no empieza con hiperparámetros: empieza con el diagnóstico correcto**. Un modelo que no generaliza puede sufrir de bias (underfitting) o de varianza (overfitting), y el tratamiento es opuesto en cada caso. Tunear hiperparámetros sobre un modelo con underfitting severo es tiempo (y GPU) perdidos.

> [!abstract] 👔 Impacto ejecutivo
> Aquí se decide cuánto rendimiento extra se compra y a qué precio — y cuándo parar.
>
> - **Decisiones que habilita:** invertir el presupuesto de mejora donde el diagnóstico lo indica (más datos vs más complejidad vs mejores features), fijar umbrales de decisión con lógica de utilidad, exigir probabilidades calibradas antes de usarlas en decisiones.
> - **Costo de hacerlo mal:** semanas de tuning para rasguñar décimas que el negocio no nota, ensembles inmantenibles por 0.2 puntos de AUC, y decisiones tomadas sobre probabilidades que no eran probabilidades.
> - **Pregunta ejecutiva que responde:** *¿la próxima semana de trabajo del equipo debe ir a más datos, más features, más tuning — o a ninguna de las tres?*

---

## 1. Bias-Variance Tradeoff — el diagnóstico primero

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía de los arqueros
> Dos arqueros con problemas distintos: el primero dispara siempre **al mismo lugar, pero lejos del centro** (alto bias: es sistemáticamente malo — underfitting). El segundo **dispersa sus flechas por todo el blanco** (alta varianza: cada disparo depende del viento del día — overfitting). El arquero ideal agrupa sus flechas en el centro. Entrenar bien es diagnosticar cuál de los dos arqueros eres antes de comprar un arco nuevo.

**🔧 Definición técnica:**

- **Bias:** error sistemático — el modelo es demasiado simple para el patrón real. Señal: error alto **en train**. Tratamiento: modelo más complejo, más/mejores features ([[03-Preparacion-de-Datos]]), menos regularización.
- **Varianza:** sensibilidad al ruido del train — el modelo memoriza. Señal: error bajo en train, alto en test (**gap grande**). Tratamiento: más datos, regularización, modelo más simple, ensembles.
- **Descomposición:** `Error total = Bias² + Varianza + Ruido irreducible` (el ruido irreducible — Bayes error — es el piso teórico que ningún modelo puede perforar).
- **Learning curves:** error de train y validación vs **tamaño del dataset**. Underfitting: ambas convergen alto (más datos no ayuda). Overfitting: gap persistente (más datos sí ayuda). Sano: convergen bajo.
- **Validation curves:** error de train y val vs **un hiperparámetro** (ej. max_depth): visualizan dónde empieza el overfitting.
- **Curvas de pérdida por época (train vs validation):** una tercera lectura, distinta de las dos anteriores — aquí el eje X **no** es el tamaño del dataset (learning curves) ni un hiperparámetro (validation curves), sino la **época** de entrenamiento. Aplica a modelos que se ajustan de forma iterativa (redes neuronales, boosting con muchas rondas). Tres formas típicas y su lectura (Goodfellow et al., 2016; Géron, 2022):
    - Ambas curvas bajan y convergen → el modelo generaliza.
    - Train sigue bajando y validación empieza a subir (divergen) → overfitting; tratamiento en Early Stopping (sección 3) y/o más regularización.
    - Ambas altas y planas, sin bajar → underfitting: capacidad insuficiente o learning rate mal elegido.

```
 LEARNING CURVES
 underfitting            overfitting              sano
 error│ ══════ val       error│ ═══╗ val          error│ ═╗ val
      │ ══════ train          │    ╚═══           　    │  ╚══╗
      │ (juntas, ALTAS)       │ ────── train           │ ──╚═ train
      └────── N               └────── N   gap!         └────── N
 → más datos NO ayuda    → más datos SÍ ayuda     → estás bien
```

```
 LOSS POR ÉPOCA (train vs validation)
 sano                    overfitting              underfitting
 loss│ ╲                 loss│ ╲                  loss│ ══════ val
     │  ╲__ val               │  ╲___ train            │ ══════ train
     │  ╲__ train              │      ╲___╱ val↗         │ (juntas, ALTAS y planas)
     └────── época             └────── época             └────── época
 → converge, generaliza   → Early Stopping (§3)     → más capacidad o LR
```

Detalle de arquitecturas, optimizadores y scheduling de learning rate en [[12-Deep-Learning]].

**👔 En una frase para el negocio:** este diagnóstico de 10 minutos decide si el próximo millón se invierte en **más datos** o en **mejor modelo** — equivocar la receta duplica el gasto sin mover la métrica.

---

## 2. Optimización de Hiperparámetros

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Encontrar la combinación de un candado de 5 ruedas: **Grid Search** prueba TODAS las combinaciones en orden (exhaustivo, eterno). **Random Search** gira ruedas al azar — sorprendentemente eficaz, porque pocas ruedas importan de verdad (Bergstra & Bengio, 2012). **Bayesian/Optuna** es el cerrajero que escucha los clics: cada intento le enseña dónde probar el siguiente, y abandona a mitad de camino los intentos que ya suenan mal (pruning).

| Método | Mecanismo | Ventajas | Desventajas | Mejor para |
|---|---|---|---|---|
| Grid Search (GridSearchCV) | Evalúa TODAS las combinaciones del grid con CV interno | Exhaustivo, reproducible, simple | Explota exponencialmente: 5 params × 5 valores = 3.125 combos | Pocas combinaciones (≤ 100); ajuste fino cerca de un óptimo conocido |
| Random Search (RandomizedSearchCV) | Muestrea n_iter combinaciones aleatorias; acepta distribuciones (loguniform para learning_rate) | Mucho más eficiente al mismo costo; cubre mejor el espacio | No garantiza el óptimo; varía entre corridas | Primera exploración con muchos hiperparámetros |
| Bayesian Optimization | Modela métrica vs hiperparámetros con un surrogate (GP/TPE) y elige el próximo punto por mejora esperada | Converge rápido; cada evaluación informa la siguiente | Overhead del surrogate; secuencial por diseño | Cada evaluación cuesta horas (modelos/datasets grandes) |
| Optuna | Bayesian con TPE + **pruning** (mata trials malos temprano: MedianPruner, SuccessiveHalvingPruner); multi-objetivo Pareto (Akiba et al., 2019) | Espacio de búsqueda como código Python; visualizaciones de importancia de hiperparámetros; integra XGBoost/LightGBM/PyTorch/sklearn | Curva de aprendizaje inicial | **El estándar actual** para tuning serio |
| Hyperopt | TPE con espacio definido vía hp.choice/hp.uniform/hp.loguniform | Maduro; SparkTrials para paralelizar | API menos ergonómica; desarrollo menos activo | Alternativa sólida, entornos Spark |
| Ray Tune | Tuning distribuido; integra Lightning/TF/XGBoost; Population Based Training (PBT) | Escala horizontal a clusters; PBT adapta hiperparámetros DURANTE el entrenamiento | Requiere infraestructura; overhead para casos simples | Deep learning multi-GPU; búsquedas de días |
| Successive Halving (HalvingRandomSearchCV) | Muchos candidatos con pocos recursos; los peores se eliminan y los sobrevivientes reciben más | Muy eficiente en espacios grandes | Sesgo contra candidatos que arrancan lento | sklearn ≥ 0.24; buen trade-off velocidad/calidad |

> [!warning] ⚠️ Reglas del tuning honesto
> El tuning se hace con **validación separada del test final** ([[10-Validacion-y-Leakage]]): el test se toca una sola vez, al final. Y el orden importa: diagnóstico (sección 1) → features → modelo → recién entonces hiperparámetros. El tuning pule; no rescata.

**👔 En una frase para el negocio:** el tuning moderno (Optuna) exprime el mismo modelo con una fracción del cómputo del método fuerza-bruta — pero sigue siendo la **última** milla, no la primera.

---

### 2.1 AutoML — cuando la máquina tunea a la máquina

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Si Optuna es el cerrajero que busca la combinación, AutoML es el servicio que **elige la cerradura, la instala y la calibra** — tú le entregas el dataset y te devuelve un modelo listo. No elimina al Data Scientist; pero automatiza la parte donde el humano menos agrega valor: probar 20 modelos × 100 configuraciones cada uno.

**🔧 Definición técnica:** un sistema AutoML automatiza una o más etapas del pipeline: selección de modelo, feature engineering, tuning de hiperparámetros y ensemble. El valor está en que prueba combinaciones que un humano no exploraría por falta de tiempo.

| Framework | Mecanismo clave | Fortaleza | Limitación | Cuándo elegirlo |
|---|---|---|---|---|
| **AutoGluon** (AWS, 2020) | Multi-layer stacking de muchos modelos (KNN, LightGBM, XGBoost, CatBoost, RF, NN, TabPFN); selección y ensemble automáticos; presets (`best_quality`, `medium_quality`, `fast`) | El mejor out-of-the-box sin tunear nada; reproduce la estrategia ganadora de Kaggle (Erickson et al., 2020) | Caja negra pesada; requiere más RAM/CPU que un modelo solo; tiempo de entrenamiento alto en `best_quality` | Baseline fuerte rápida; comparar contra tu modelo manual; datasets tabulares medianos-grandes |
| **FLAML** (Microsoft, 2021) | Búsqueda económica (cost-frugal): elige modelos baratos primero, escala a caros solo si hay presupuesto; soporta presupuesto de tiempo | Extremadamente rápido para presupuestos bajos (1-5 min); ligero; buen trade-off velocidad/rendimiento | Menos potente que AutoGluon con presupuesto ilimitado | Prototipado rápido; recursos limitados; iteración veloz |
| **TabPFN** (Hollmann et al., 2023; v3 2026) | Transformer pre-entrenado sobre datos tabulares sintéticos; in-context learning — no entrena, solo infiere | Inference en <1 segundo; resultados competitivos con AutoGluon en datasets ≤ 10K filas (TabPFN-3 supera tuned ensembles en TabArena, 2026) | Limitado a datasets pequeños/medianos; no escala a millones de filas; caja negra total | Datasets pequeños donde entrenar es caro o hay muchos datasets distintos que evaluar |
| **H2O AutoML** | Búsqueda exhaustiva con stacking; interfaz web (Flow) y Python | Maduro, escalable, enterprise-ready | Más pesado de instalar; menos innovación reciente | Entornos enterprise con infraestructura H2O |

**🔧 Cuándo NO usar AutoML:**

- Cuando el cuello de botella es la **calidad de los datos**, no el modelo — AutoML no repara features malas ni corrige leakage ([[10-Validacion-y-Leakage]]).
- Cuando necesitas **interpretabilidad** total — un ensemble de 20 modelos stacked no se explica con SHAP fácilmente ([[13-MLOps-XAI-Etica]]).
- Cuando el **deployment** tiene restricciones de latencia/tamaño — el modelo de AutoGluon `best_quality` puede pesar GBs.

**🧭 El patrón recomendado:**

```
 AutoGluon (best_quality, 1h)
        │
        ▼
 Benchmark: ¿supera a mi modelo manual?
  ├── SÍ → estudiar qué modelos/features usó → informar mi trabajo
  └── NO → mi pipeline manual ya es competitivo; seguir con Optuna
```

AutoML no es el reemplazo del Data Scientist; es su **benchmark de cordura**: si AutoGluon con presets default supera tu modelo tuneado a mano, el problema está en tu pipeline, no en el tuning.

**👔 En una frase para el negocio:** AutoML democratiza el acceso a modelos competitivos en minutos — pero el Data Scientist sigue siendo quien diagnostica si el problema es de datos, de modelo, o de negocio.

---

## 3. Regularización

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Todas las técnicas de esta sección son variantes del mismo consejo al estudiante memorión ([[07-Modelos-Supervisados]]): "aprende el patrón, no la fotocopia". Unas le encogen los apuntes (L1/L2), otras le esconden aleatoriamente la mitad del cuaderno para que no dependa de ninguna página (Dropout), otras lo sacan del examen cuando empieza a inventar (Early Stopping).

| Técnica | Mecanismo | Dónde se usa | Nota clave |
|---|---|---|---|
| L1 (Lasso) | Suma `λ·Σ│βᵢ│` a la pérdida → coeficientes exactamente 0 | Modelos lineales; selección de features ([[03-Preparacion-de-Datos]]) | Geometría de rombo: soluciones sparse ([[02-Fundamentos-Matematicos]]) |
| L2 (Ridge / weight decay) | Suma `λ·Σβᵢ²` → encoge sin eliminar; reparte entre correlacionadas | Lineales y redes (weight decay) | Siempre estable; en Transformers vía AdamW ([[12-Deep-Learning]]) |
| ElasticNet | Mezcla L1+L2 (`l1_ratio`) | Lineales con features correlacionadas | El compromiso por defecto ([[07-Modelos-Supervisados]]) |
| Dropout | En cada paso de training desactiva una fracción `p` de neuronas; **las activas se escalan por `1/(1−p)`** para conservar la esperanza de la señal | Redes neuronales | Actúa como ensemble implícito de 2^N subredes (Srivastava et al., 2014); típico 0.2–0.5 densas, 0.1–0.2 conv |
| Early Stopping | Monitorea la pérdida de validación; detiene tras `patience` épocas sin mejora y restaura los mejores pesos | Redes y boosting (`early_stopping_rounds`) | Gratis y efectivo: fija además n_estimators/épocas óptimos |
| Batch Normalization | Normaliza activaciones por mini-batch (media≈0, std≈1) + parámetros aprendibles γ, β | Redes (CNN sobre todo) | Estabiliza, permite learning rates altos, regulariza de rebote (Ioffe & Szegedy, 2015) |
| Layer Normalization | Normaliza sobre las features, no sobre el batch | Transformers, RNN | No depende del batch size ([[12-Deep-Learning]]) |
| Data Augmentation | Transformaciones que preservan el label: flips/rotaciones/crops (imagen), sinónimos/back-translation (texto), mixup/cutmix | Deep learning; SMOTE es su pariente tabular ([[03-Preparacion-de-Datos]]) | Equivale a regalarle datos nuevos al modelo |

### 3.1 Early Stopping en profundidad

Audiencia: 🔧

Early Stopping merece tratamiento aparte porque cumple **tres funciones** a la vez: regulariza (evita overfitting), fija automáticamente el número óptimo de iteraciones/épocas, y ahorra cómputo.

**🔧 Mecanismo:** monitorea una métrica de validación (val_loss o la métrica objetivo) tras cada época/ronda. Si no mejora en `patience` evaluaciones consecutivas, detiene el entrenamiento y restaura los pesos del mejor checkpoint.

**🔧 Parámetros clave:**

| Parámetro | Qué controla | Valores típicos | Riesgo de mal ajuste |
|---|---|---|---|
| `patience` | Épocas sin mejora antes de parar | 5–20 (redes), 20–50 (boosting) | Muy bajo → para prematuramente; muy alto → pierde el beneficio |
| `min_delta` | Mejora mínima para contar como progreso | 1e-4 a 1e-3 | Si es 0, ruido aleatorio puede extender innecesariamente |
| `restore_best_weights` | Si restaura el checkpoint óptimo o deja los últimos | Siempre True | Si es False, el modelo final puede ser PEOR que el mejor intermedio |
| `monitor` | Métrica a vigilar | `val_loss` (genérico), `val_auc` (clasificación) | Monitorear train_loss es inútil (siempre baja) |

**🔧 Dónde aplica:**

- **Gradient Boosting:** `early_stopping_rounds` en XGBoost/LightGBM/CatBoost. Fija n_estimators óptimo automáticamente — nunca lo fijes a mano si puedes usar early stopping.
- **Redes neuronales:** `EarlyStopping` callback en Keras/PyTorch Lightning. Combinar con learning rate scheduling (ReduceLROnPlateau): primero reducir LR, luego parar si la reducción tampoco ayuda.
- **Relación con el diagrama de §1:** Early stopping es la respuesta directa al patrón "train sigue bajando, validación sube" en la curva de loss por época.

**🔧 Anti-patrón común:** fijar `n_estimators=1000` o `epochs=100` sin early stopping y luego preguntarse por qué el modelo sobreajusta. Early stopping es **gratis** en cómputo (lo ahorra) y en rendimiento (nunca perjudica si `restore_best_weights=True`).

**🧭 Cuándo usarlo:** **siempre** que el modelo se entrene iterativamente — no hay razón para no usarlo. La única excepción: entrenamiento de una sola pasada (modelos lineales con solver cerrado, KNN).

---

## 4. Métodos Ensemble

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un jurado le gana a un juez solitario cuando sus miembros son **competentes y diversos**: los errores individuales se cancelan. Bagging recluta jurados que vieron evidencias distintas; Boosting entrena cada jurado en los casos que el anterior falló; Stacking contrata a un juez presidente que aprendió **a quién creerle según el tipo de caso**.

| Técnica | Mecanismo detallado | Qué reduce | Ejemplos | Notas clave | Cuándo conviene |
|---|---|---|---|---|---|
| Bagging | B modelos independientes sobre B muestras bootstrap (con reemplazo); voto mayoritario o promedio | Varianza | Random Forest, BaggingClassifier ([[07-Modelos-Supervisados]]) | Los modelos base deben ser de ALTA varianza (árboles profundos); la independencia maximiza el beneficio | Alta varianza con un modelo base propenso a sobreajustar y se quiere robustez con poco esfuerzo de tuning |
| Pasting | Como bagging pero muestreo **sin** reemplazo | Varianza | BaggingClassifier(bootstrap=False) | Útil con datasets enormes donde el bootstrap no aporta | Dataset ya tan grande que el bootstrap no agrega diversidad real |
| Boosting | Modelos secuenciales; cada uno corrige los errores del conjunto anterior (residuos en Gradient Boosting; pesos de muestras en AdaBoost) | Bias y varianza | XGBoost, LightGBM, CatBoost, AdaBoost | Potente pero sobreajustable: regular con learning_rate + early stopping | Datos tabulares con señal fina por explotar, hay tolerancia para tunear, y las etiquetas son confiables (el ruido de etiquetas lo daña más que a bagging) |
| Stacking | Nivel 0: K modelos base; nivel 1: un meta-modelo aprende a combinar sus predicciones. **CRÍTICO: el meta-modelo se entrena con OOF (out-of-fold) predictions** para no filtrar el target (Wolpert, 1992) | Bias y varianza | RF + XGBoost + LightGBM combinados por una logística | El arma clásica de Kaggle; sin OOF, es leakage disfrazado ([[10-Validacion-y-Leakage]]) | Varios modelos fuertes y diversos ya entrenados, y hay presupuesto de cómputo/validación para entrenar el meta-modelo con OOF |
| Blending | Stacking simplificado: el meta-modelo se entrena sobre un holdout en lugar de OOF | Bias y varianza | Igual que stacking, más rápido | Usa menos datos para el meta-modelo; menos riesgo operativo | Mismo escenario que stacking pero se prefiere un holdout simple por tiempo o simplicidad operativa |
| Voting | Hard: mayoría de votos de clase. Soft: promedio de probabilidades (mejor si están calibradas) | Varianza | VotingClassifier/VotingRegressor | Soft > hard casi siempre — si las probabilidades son honestas (sección 5) | Pocos modelos que ya rinden bien por separado y no hay tiempo ni interés en tunear un meta-modelo |
| Snapshot Ensembles | Guarda los pesos de la red en varios mínimos del ciclo de learning rate (cosine annealing con restarts) y los ensembla | Varianza | Deep learning con entrenamiento caro | Ensemble "gratis": un solo entrenamiento, varios modelos | Redes neuronales donde entrenar varias veces desde cero es prohibitivo en tiempo o cómputo |

**👔 En una frase para el negocio:** los ensembles compran los últimos puntos de rendimiento al precio de más complejidad operativa — la pregunta correcta es si esos puntos pagan el mantenimiento extra ([[13-MLOps-XAI-Etica]]).

### 4.1 Feature importance como herramienta de mejora

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El coach que analiza el video post-partido: no entrena otra vez desde cero — mira qué jugadores rindieron, cuáles estorbaron y cuáles ni tocaron la pelota. Feature importance hace lo mismo con las variables: después de entrenar, revela cuáles aportaron y cuáles puedes eliminar para simplificar sin perder.

**🔧 Definición técnica:** medir la contribución de cada feature al rendimiento del modelo, para decidir si eliminarla (simplificar/regularizar) o si crearla mejor (feature engineering).

| Método | Mecanismo | Ventajas | Limitaciones |
|---|---|---|---|
| **Permutation Importance** | Permuta aleatoriamente una feature en el test set y mide cuánto cae la métrica; la caída = importancia (Breiman, 2001a) | Model-agnostic; mide impacto REAL en la métrica; detecta features que el modelo ignora | Lento con muchas features; subestima features correlacionadas (al permutar una, la otra compensa) |
| **SHAP values** | Descompone cada predicción en contribuciones aditivas por feature basándose en valores de Shapley (Lundberg & Lee, 2017) | Importancia global Y local; detecta interacciones; teóricamente fundado en teoría de juegos | Costoso computacionalmente para datasets grandes; requiere interpretación cuidadosa |
| **Gini / split importance** (tree-based) | Cuenta cuánta reducción de impureza (o ganancia) aporta cada feature en los splits del árbol | Gratis (viene con el modelo entrenado); rápido | Sesgada a favor de features con alta cardinalidad; no refleja impacto real en la métrica final |
| **Drop-column importance** | Reentrena sin la feature y compara; la pérdida = importancia | La más honesta conceptualmente | Carísima: N features × reentrenamiento completo; solo viable con pocas features o modelos rápidos |

**🔧 El ciclo de mejora basado en importancia:**

1. Entrenar modelo baseline.
2. Calcular permutation importance (o SHAP global).
3. **Eliminar features con importancia ≈ 0** → simplifica el modelo, puede mejorar generalización.
4. **Investigar las top-5 features** → ¿se pueden derivar más features de ellas? (interacciones, ratios, lags). Eso es feature engineering informado por el modelo.
5. Reentrenar y comparar.

**🔧 Conexiones:**
- Interpretabilidad completa (SHAP, LIME, PDP) → [[13-MLOps-XAI-Etica]]
- Feature selection y engineering → [[03-Preparacion-de-Datos]]
- Detección de leakage: una feature con importancia desproporcionada (>50% del total) es sospechosa → auditar disponibilidad temporal ([[10-Validacion-y-Leakage]])

**🧭 Cuándo usarlo:** después del primer modelo entrenado, **antes** de tunear hiperparámetros. Eliminar features inútiles y crear features derivadas de las importantes suele mover más la métrica que semanas de tuning.

---

## 5. Calibración de Probabilidades

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un pronosticador del tiempo que dice "90% de lluvia" y llueve solo 6 de cada 10 veces: útil para ordenar días de más a menos lluviosos, **inútil para decidir si suspender el evento**. Calibrar es reeducarlo para que su "90%" signifique 90 de cada 100. Muchos modelos son ese pronosticador: ordenan bien, pero sus números no son probabilidades de verdad.

**🔧 Definición técnica:** un modelo está calibrado si de los casos a los que asigna probabilidad p, la fracción positiva real es ≈ p. Diagnóstico: **reliability diagram** (deciles de probabilidad predicha vs frecuencia observada; calibrado = diagonal). Métricas: Brier Score, Log Loss ([[08-Metricas-de-Evaluacion]]).

| Método | Mecanismo | Cuándo conviene |
|---|---|---|
| Platt Scaling | Regresión logística sobre las salidas del modelo (en holdout) (Platt, 1999) | Pocas muestras de calibración; distorsión con forma sigmoide (SVM, Naive Bayes) |
| Isotonic Regression | Ajuste monotónico no paramétrico | Más flexible; requiere más datos (N > 1.000 o sobreajusta); ideal para árboles/ensembles |
| Temperature Scaling | Divide los logits por T antes del softmax; T se optimiza en validación | Redes neuronales modernas (suelen ser sobreconfiadas) (Guo et al., 2017) |
| CalibratedClassifierCV | Envuelve cualquier estimador sklearn con Platt ('sigmoid') o isotónica, con CV interno | La vía práctica estándar en sklearn |

**👔 En una frase para el negocio:** si las probabilidades del modelo alimentan precios, provisiones o priorización, calibrarlas no es opcional — decidir con probabilidades infladas es presupuestar con moneda falsa.

---

## 6. Ajuste del Umbral de Decisión

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> La sensibilidad de la alarma de tu casa: de fábrica viene "al medio" (0.5), pero si guardas lingotes ajustas la alarma sensible (toleras falsas alarmas), y si solo guardas recuerdos la pones tolerante. El modelo entrega el riesgo; **el umbral decide cuándo actuar — y ese es un dial de negocio**.

**🔧 Definición técnica:** el 0.5 por defecto rara vez es óptimo con clases desbalanceadas o costos asimétricos. Métodos: (1) **threshold optimization** — barrer umbrales de 0 a 1 sobre validación y maximizar la métrica objetivo (F1, F-beta, utilidad); (2) **precision_recall_curve** de sklearn — visualizar el trade-off y elegir el punto; (3) **expected profit framework** — con costos C_FP y C_FN, el umbral óptimo teórico es `P* = C_FP/(C_FP + C_FN)` (requiere probabilidades **calibradas**, sección 5; caso completo en [[08-Metricas-de-Evaluacion]]).

**👔 En una frase para el negocio:** mover el umbral es la mejora más barata del catálogo: cero reentrenamiento, impacto inmediato en la cuenta de resultados — siempre que las probabilidades sean honestas.

---

## 7. El orden correcto de la mejora (síntesis)

Audiencia: 🧭

```
 1. DIAGNÓSTICO  (learning/validation curves: ¿bias o varianza?)
        │
 2. DATOS Y FEATURES  (la palanca más rentable; usar feature importance §4.1)
        │
 3. MODELO ADECUADO  (baseline → RF → boosting: Modelos Supervisados)
        │
 4. REGULARIZACIÓN + EARLY STOPPING  (que no memorice)
        │
 5. TUNING (Optuna / AutoML §2.1 como benchmark)
        │
 6. ENSEMBLE (si los puntos extra pagan su costo)
        │
 7. CALIBRACIÓN ──► 8. UMBRAL POR UTILIDAD  (los pasos que tocan la caja)
```

Detalle de cada etapa: [[03-Preparacion-de-Datos]] · [[07-Modelos-Supervisados]]

---

## 📖 Referencias de este tomo

- (Bergstra & Bengio, 2012) — por qué random search vence a grid search.
- (Akiba et al., 2019) — Optuna. · (Wolpert, 1992) — stacked generalization.
- (Srivastava et al., 2014) — Dropout. · (Ioffe & Szegedy, 2015) — Batch Normalization.
- (Platt, 1999) — Platt scaling. · (Guo et al., 2017) — calibración de redes modernas.
- (Erickson et al., 2020) — *AutoGluon-Tabular: Robust and Accurate AutoML for Structured Data*. arXiv 2003.06505.
- (Hollmann et al., 2023) — *TabPFN: A Transformer That Solves Small Tabular Classification Problems in a Second*. ICLR.
- (Lundberg & Lee, 2017) — *A Unified Approach to Interpreting Model Predictions* (SHAP). NeurIPS.
- (Hastie et al., 2009), (Géron, 2022) — ensembles y regularización en contexto.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[10-Validacion-y-Leakage|10 · Validación y Leakage]] · Siguiente: [[12-Deep-Learning|12 · Deep Learning ➡]]

> **Próximo tomo:** [[12-Deep-Learning]] — de la neurona al Transformer: activaciones, optimizadores, scheduling, inicialización, CNN/RNN/LSTM, atención, arquitecturas generativas y transfer learning.
