---
title: "Tomo 08 — Métricas de Evaluación"
tags: [data-science, machine-learning, metricas, evaluacion, roc]
audiencias: [tecnico, puente, ejecutivo]
tomo: 08
version: 6.3
updated: 2026-08-27
---

# 🌡️ Tomo 08 — Métricas de Evaluación

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[07-Modelos-Supervisados|07 · Modelos Supervisados]] · Siguiente: [[09-Reglas-de-Asociacion|09 · Reglas de Asociación ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> La métrica que eliges determina **qué optimiza tu modelo**. Si eliges mal, optimiza lo incorrecto: es como evaluar a un portero solo por los goles que mete — va a pasar el partido en el área contraria y dejar el arco vacío. La elección de la métrica no es técnica: **es una decisión de negocio**. En detección de cáncer, un falso negativo (enfermo clasificado sano) cuesta radicalmente distinto que un falso positivo. La métrica debe codificar esa asimetría.

> [!abstract] 👔 Impacto ejecutivo
> El "95% de accuracy" es la frase más peligrosa de la sala de reuniones: puede describir un modelo excelente o uno perfectamente inútil.
>
> - **Decisiones que habilita:** alinear lo que el modelo optimiza con el costo real del error, comparar modelos con reglas de juego justas, fijar el umbral de decisión que maximiza utilidad.
> - **Costo de hacerlo mal:** modelos que optimizan la métrica equivocada (y por lo tanto la decisión equivocada), falsos éxitos de laboratorio, presupuesto de revisión humana mal dimensionado.
> - **Pregunta ejecutiva que responde:** *¿esta métrica mide lo que a MI negocio le cuesta o le renta, o solo lo que era fácil de calcular?*

> [!example] 📊 Caso de negocio — Pagos: el umbral que dejó de regalar dinero
> **Problema:** una fintech bloquea transacciones con score de fraude > 0.5 "porque 0.5 es lo estándar". Resultado: demasiados bloqueos falsos (clientes furiosos, ventas perdidas) y aun así se escapan fraudes caros.
>
> **Técnica aplicada:** cuantificar el costo real: FP (bloquear venta legítima) ≈ $8 de margen + fricción; FN (dejar pasar fraude) ≈ $95 promedio. Con la curva Precision-Recall y el framework de **expected profit**, el umbral óptimo teórico queda en `P* = C_FP/(C_FP+C_FN) = 8/103 ≈ 0.08` — muy lejos del 0.5 folclórico. Se calibra el modelo (reliability diagram, [[11-Mejora-de-Modelos]]) para que sus scores sean probabilidades de verdad, y se ajusta el umbral por segmento de monto.
>
> **Resultado:** el mismo modelo, sin reentrenar nada, captura una fracción sustancialmente mayor del fraude tolerando bloqueos baratos, y el trade-off queda documentado en pesos: cada punto de recall adicional tiene su precio visible. La lección: **el umbral es una palanca de negocio, no una constante de la naturaleza**.

---

## 1. La Matriz de Confusión — la fuente primaria

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El test de embarazo de los modelos: hay cuatro resultados posibles — acertar que sí (TP), acertar que no (TN), la falsa alarma (FP) y el "no lo vio venir" (FN). **Toda** métrica de clasificación es una receta cocinada con estos cuatro ingredientes; antes de cualquier métrica, mira la matriz completa.

|  | **Predicho Positivo** | **Predicho Negativo** |
|---|---|---|
| **Real Positivo** | TP (Verdadero Positivo): correcto, detectado | FN (Falso Negativo): error tipo II, el "miss" |
| **Real Negativo** | FP (Falso Positivo): error tipo I, la falsa alarma | TN (Verdadero Negativo): correcto, rechazado |

La conexión con los errores tipo I/II de la inferencia está en [[02-Fundamentos-Matematicos]]; la lectura paso a paso para no técnicos, en [[14-Anexo-Interpretar-Resultados]].

---

## 2. Métricas de Clasificación

Audiencia: 🔧 🧭

| Métrica | Fórmula | Rango | 💡 Analogía | Aplica a | Cuándo priorizarla |
|---|---|---|---|---|---|
| Accuracy | `(TP+TN)/(TP+TN+FP+FN)` | [0,1] | Nota global del examen: % de respuestas correctas | Cualquier clasificador | Clases balanceadas y errores de igual costo (MNIST, flores) |
| Precision | `TP/(TP+FP)` | [0,1] | "Si digo que llueve, ¿realmente llueve?" | Cualquier clasificador | FP costoso: spam (no botar emails legítimos), alertas que gatillan trabajo humano caro |
| Recall (Sensitivity, TPR) | `TP/(TP+FN)` | [0,1] | "Si llueve, ¿siempre lo detecto?" | Cualquier clasificador | FN costoso: cáncer, fraude, seguridad — no dejar pasar casos |
| Specificity (TNR) | `TN/(TN+FP)` | [0,1] | "Si NO llueve, ¿sé quedarme callado?" | Cualquier clasificador | FP costoso en screening masivo (ansiedad/costos de confirmación) |
| F1-Score | `2·(Prec·Rec)/(Prec+Rec)` | [0,1] | El promedio exigente: castiga al que descuida uno de los dos | Cualquier clasificador | Ambos errores importan y hay desbalance moderado (texto, NLP) |
| F-beta | `(1+β²)·(Prec·Rec)/(β²·Prec+Rec)` | [0,1] | El F1 con la balanza inclinada a elección | Cualquier clasificador | β=2: recall manda (fraude); β=0.5: precision manda (spam) |
| AUC-ROC | Área bajo TPR vs FPR al variar el umbral | [0,1]; 0.5 = azar | ¿Qué tan bien ordena el modelo: positivos arriba, negativos abajo? | Modelos con score/probabilidad | Comparar modelos sin fijar umbral; ranking general |
| AUC-PR | Área bajo Precision vs Recall | [0,1] | AUC-ROC versión "clase rara": mide el orden donde duele | Modelos con score | Desbalance severo (0.1% positivos) donde ROC es optimista (Davis & Goadrich, 2006) |
| Log Loss (Cross-Entropy) | `−(1/N)·Σ[y·log(p)+(1−y)·log(1−p)]` | [0,∞) | Multa por sobreconfianza: decir 99% y errar sale carísimo | Modelos probabilísticos | Las probabilidades alimentan decisiones/ranking aguas abajo |
| Brier Score | `(1/N)·Σ(pᵢ−yᵢ)²` | [0,1] | El MSE de las probabilidades (Brier, 1950) | Modelos probabilísticos | Evaluar calibración ([[11-Mejora-de-Modelos]]) |
| Cohen's Kappa | `(pₒ−pₑ)/(1−pₑ)` | [−1,1] | Tu nota descontando lo que habrías acertado tirando la moneda (Cohen, 1960) | Cualquier clasificador | Desbalance; comparar contra el azar: κ>0.8 excelente, κ<0.4 pobre |
| MCC | `(TP·TN−FP·FN)/√[(TP+FP)(TP+FN)(TN+FP)(TN+FN)]` | [−1,1] | La nota más justa: exige acertar en las cuatro casillas a la vez (Matthews, 1975) | Cualquier clasificador binario | El mejor indicador único con desbalance severo |

> [!warning] ⚠️ La trampa del accuracy
> Con 1% de fraude, el modelo "todo es legítimo" tiene 99% de accuracy y 0% de utilidad. Con desbalance, el trío honesto es **AUC-PR + MCC + matriz de confusión completa** — y accuracy queda de adorno ([[03-Preparacion-de-Datos]]).

---

## 3. Curvas ROC y PR en detalle

Audiencia: 🔧 🧭

**🔧 Curva ROC:** traza TPR (recall) vs FPR (1−specificity) al mover el umbral de 1 a 0. Perfecto: AUC = 1; azar: la diagonal (AUC = 0.5). La esquina superior izquierda es el ideal (detectarlo todo sin falsas alarmas). Interpretación probabilística del AUC: probabilidad de que un positivo aleatorio reciba mayor score que un negativo aleatorio.

```
 TPR 1 ┤        ╭────────●  ← modelo bueno (AUC ~0.9)
       │      ╭─╯
       │    ╭─╯   ╱ diagonal = azar (AUC 0.5)
       │  ╭─╯   ╱
       │ ╭╯   ╱
     0 └─┴──╱────────────┬ FPR
       0                 1
```

**🔧 Curva PR:** Precision vs Recall al variar el umbral. Con clases muy desbalanceadas es **más informativa que ROC**: el FPR usa los TN (abundantes), lo que infla el ROC; la PR solo mira cómo le va al modelo con la clase rara. Baseline de PR = prevalencia de la clase positiva (no 0.5).

**🔧 Elección del umbral óptimo — nunca 0.5 por decreto:**

- **Youden's J:** maximiza `TPR − FPR` (Youden, 1950) — el punto del ROC más lejos de la diagonal.
- **Punto más cercano a la esquina** superior izquierda del ROC.
- **Máximo F-beta:** con el β que codifica tu asimetría de costos.
- **Expected profit:** si FP cuesta C_FP y FN cuesta C_FN, el umbral óptimo teórico es `P* = C_FP/(C_FP+C_FN)` — el costo de negocio directamente en la decisión (ver caso de negocio y [[11-Mejora-de-Modelos]]).

**🔧 Calibration plot (reliability diagram):** divide las predicciones en deciles de probabilidad y compara probabilidad media predicha vs frecuencia observada real. Modelo calibrado = puntos sobre la diagonal. Correcciones (Platt, Isotonic): [[11-Mejora-de-Modelos]].

**👔 En una frase para el negocio:** la curva es el menú de trade-offs disponibles; el umbral es el plato que eliges — y esa elección vale dinero, no es un tecnicismo.

---

## 4. Métricas Multiclase — los promedios

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Tres formas de promediar las notas de un colegio: dándole el mismo peso a cada **curso** aunque tengan distinto tamaño (macro), ponderando por el número de alumnos por curso (weighted), o juntando a todos los alumnos en una sola lista (micro). El mismo colegio puede lucir muy distinto según el promedio elegido.

| Averaging | Cómo calcula | Cuándo usar |
|---|---|---|
| macro | Promedio simple de la métrica por clase — todas las clases pesan igual | Las clases minoritarias importan tanto como las grandes (fallas raras, enfermedades poco comunes) |
| weighted | Promedio ponderado por el soporte (frecuencia) de cada clase | La importancia es proporcional al volumen; default informativo en sklearn |
| micro | Agrega TP/FP/FN globales y calcula una sola vez | Equivale a accuracy en multiclase single-label; domina la clase grande |
| samples | Promedia la métrica calculada por muestra | Solo multi-label (cada muestra puede tener varias etiquetas) |

### 4.1 La confusion matrix multiclase — cómo leerla

Audiencia: 🔧 🧭 👔

**🔧 Definición técnica:** con K clases, la matriz es K×K. La diagonal principal contiene los aciertos; cada celda fuera de la diagonal indica cuántas veces se confundió la clase-fila con la clase-columna. Las **filas** suman al total real de cada clase; las **columnas** suman al total predicho.

```
 Predicho →     gato   perro   pájaro
 Real ↓
 gato         [ 45 ]     3       2      ← 50 gatos reales
 perro           4    [ 38 ]     8      ← 50 perros reales
 pájaro          1       2    [ 47 ]    ← 50 pájaros reales
                                        
               50       43      57      ← totales predichos
```

**🔧 Lecturas clave:**

- **Por fila:** recall de esa clase = diagonal / suma de la fila. ¿Cuántos de los reales detecté?
- **Por columna:** precision de esa clase = diagonal / suma de la columna. ¿De los que predije, cuántos eran de verdad?
- **Celdas off-diagonal ruidosas:** señalan confusiones sistemáticas — informan feature engineering (¿qué distingue perro de pájaro? quizás el tamaño → agregar feature de peso).
- **Normalizar por fila** (porcentajes) cuando las clases tienen distinto tamaño: así las clases grandes no dominan visualmente.

**👔 En una frase para el negocio:** la matriz multiclase te dice no solo cuánto se equivoca tu modelo, sino **con qué lo confunde** — y esa confusión específica puede tener costos muy distintos (confundir un tumor benigno con maligno vs al revés).

---

## 5. Métricas de Ranking

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Las métricas de ranking no preguntan "¿acertaste o no?", sino "¿pusiste lo bueno arriba en la lista?". Es la diferencia entre evaluar si el bibliotecario encontró tu libro (clasificación) vs si lo puso primero en la pila de recomendaciones (ranking).

**🔧 Definición técnica:** se usan cuando el output del modelo es una **lista ordenada** de ítems, no una etiqueta. Aplican a sistemas de recomendación ([[20-Sistemas-de-Recomendacion]]), motores de búsqueda, y cualquier problema donde el orden importa.

| Métrica | Fórmula / Mecanismo | Rango | 💡 Analogía | Cuándo usarla |
|---|---|---|---|---|
| Precision@K | De los top-K recomendados, ¿qué fracción es relevante? | [0,1] | "De las 10 películas que me recomendaste, ¿cuántas me gustaron?" | K fijo por diseño de UI (top-5, top-10) |
| Recall@K | De todos los relevantes, ¿qué fracción aparece en el top-K? | [0,1] | "De las 20 películas que me gustarían, ¿cuántas están en tu top-10?" | Cobertura del catálogo relevante |
| MAP@K (Mean Average Precision) | Promedio del AP por query; AP = promedio de precision@k solo en las posiciones donde hay un acierto | [0,1] | La nota promedio del buscador: premia poner los aciertos arriba, no dispersos | Comparar modelos de retrieval/recomendación (Robertson, 2009) |
| MRR (Mean Reciprocal Rank) | Promedio de 1/rank del primer resultado relevante | [0,1] | "¿Cuánto tuve que scrollear hasta encontrar algo útil?" | Cuando solo importa el PRIMER resultado correcto (Q&A, navigational search) |
| NDCG@K (Normalized Discounted Cumulative Gain) | Suma de relevancia descontada por log₂(rank), normalizada por el DCG ideal | [0,1] | Premia poner lo MÁS relevante primero, con descuento logarítmico por posición | Relevancia gradual (ratings 1-5, no binaria); el estándar en search y recomendadores (Järvelin & Kekäläinen, 2002) |
| Hit Rate@K | ¿Al menos un ítem relevante está en el top-K? (1 o 0 por query) | {0,1} | "¿Sí o no encontré algo bueno en la primera página?" | Evaluación rápida de sistemas de recomendación |

**🧭 Cuándo usarlo:** siempre que el modelo devuelva una lista ordenada — incluye RAG retrieval (precision@k y recall@k del retriever), sistemas de recomendación, y resultados de search. Las métricas de ranking del retriever RAG están detalladas en la Guía Maestra de RAG (Tomo 09).

**👔 En una frase para el negocio:** no basta con que el modelo "sepa" lo relevante — tiene que ponerlo donde el usuario lo ve primero. Estas métricas miden exactamente eso.

---

## 6. Métricas de Fairness

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un modelo puede tener 95% de accuracy global y ser profundamente injusto: el portero que ataja todo del lado izquierdo y deja pasar todo del lado derecho tiene 50% de saves globales, pero la mitad del arco está desnuda. Las métricas de fairness miden si el rendimiento es **parejo entre grupos protegidos** — género, etnia, edad, localidad.

**🔧 Definición técnica:** miden la equidad del modelo con respecto a un **atributo sensible** (protected attribute). No reemplazan a las métricas de rendimiento — las complementan. Un modelo justo Y preciso es el objetivo; uno justo pero inútil no sirve, y uno preciso pero discriminatorio es un riesgo legal y reputacional.

| Métrica | Definición | 💡 Analogía | Cuándo priorizarla |
|---|---|---|---|
| **Demographic Parity** (Statistical Parity) | P(ŷ=1 \| grupo A) = P(ŷ=1 \| grupo B). La tasa de predicción positiva es igual entre grupos. | "Apruebo al mismo % de candidatos de cada grupo" | Asignación de recursos (créditos, becas) donde la igualdad de acceso es el objetivo explícito |
| **Equalized Odds** | TPR y FPR iguales entre grupos. El modelo se equivoca por igual en ambos. | "Mi detector de fraude falla con la misma frecuencia sin importar quién seas" | El estándar más robusto para corrección de bias — no sacrifica precisión global innecesariamente (Hardt et al., 2016) |
| **Equal Opportunity** | Solo el TPR es igual entre grupos (caso relajado de Equalized Odds). | "Si estás enfermo, te detecto con la misma probabilidad seas del grupo que seas" | Cuando el FN es el error costoso y diferencial (screening médico, admisiones) |
| **Predictive Parity** | Precision igual entre grupos: P(y=1 \| ŷ=1, A) = P(y=1 \| ŷ=1, B). | "Si te señalo, la probabilidad de que realmente seas positivo es la misma" | Alertas donde actuar sobre un FP tiene costo diferencial por grupo (intervenciones policiales) |
| **Disparate Impact Ratio** | Tasa positiva del grupo desfavorecido / tasa del grupo favorecido. Umbral legal (US): ≥ 0.8 ("regla del 80%"). | "¿Apruebo al grupo B al menos al 80% de la tasa del grupo A?" | Auditoría regulatoria; ISO 42001 y EU AI Act lo referencian |
| **Calibration by Group** | El modelo está calibrado dentro de cada grupo: P(y=1 \| score=s, grupo) = s. | "Cuando digo 70% para cualquier grupo, realmente el 70% es positivo" | Modelos probabísticos usados en decisiones de alto impacto (sentencias, créditos) |

> [!warning] ⚠️ El teorema de la imposibilidad de Chouldechova (2017) / Kleinberg et al. (2016)
> Cuando las tasas base difieren entre grupos (prevalencia distinta del target), **no se puede satisfacer simultáneamente** Demographic Parity, Equalized Odds y Calibration by Group. La elección de cuál priorizar es **ética y de negocio**, no técnica — y debe documentarse explícitamente.

**🧭 Cuándo usarlo:** siempre que el modelo afecte personas y exista un atributo sensible en los datos (o inferible de proxies). Especialmente en: scoring crediticio, contratación, admisiones, pricing, atención médica, justicia. Ver contexto regulatorio en [[13-MLOps-XAI-Etica]].

**👔 En una frase para el negocio:** un modelo que discrimina no solo es éticamente inaceptable — es un riesgo legal cuantificable. Estas métricas son el termómetro que lo detecta antes del regulador.

---

## 7. Métricas de Regresión

Audiencia: 🔧 🧭

| Métrica | Fórmula | Unidad | Sensible a outliers | 💡 Analogía | Cuándo usar |
|---|---|---|---|---|---|
| MAE | `(1/N)·Σ│y−ŷ│` | La del target | No (robusta) | "En promedio, ¿por cuántos pesos me equivoco?" | Errores de igual importancia; comunicación directa al negocio |
| MSE | `(1/N)·Σ(y−ŷ)²` | Target² (distorsionada) | Muy alta | El profesor que castiga al cuadrado los errores grandes | Función de pérdida de entrenamiento; errores grandes inaceptables |
| RMSE | `√MSE` | La del target | Alta | Como MAE pero con lupa en los errores grandes | El estándar de reporte en regresión |
| MAPE | `(1/N)·Σ│y−ŷ│/│y│·100%` | % | No | "¿En qué % me equivoco?" | Errores relativos; ⚠️ indefinida con y=0 y asimétrica: para el mismo error absoluto, el APE crece cuanto más chico es y; el sobre-pronóstico queda sin techo mientras el sub-pronóstico satura en 100% (Hyndman & Koehler, 2006) |
| SMAPE | `(1/N)·Σ 2│y−ŷ│/(│y│+│ŷ│)·100%` | % | No | El MAPE simétrico que no explota cerca de cero | Alternativa estable a MAPE en demanda con ceros |
| R² | `1 − SS_res/SS_tot` | Sin unidad | Moderada | "¿Qué % de la variabilidad explica mi modelo?" | Comparar contra el baseline "predecir la media" |
| Adjusted R² | `1 − (1−R²)(N−1)/(N−p−1)` | Sin unidad | Moderada | R² con castigo por acumular variables inútiles | Comparar modelos con distinto nº de features |
| Huber Loss | Cuadrática si │error│≤δ; lineal si >δ | Varía | Controlada (δ) | El híbrido: exigente en lo normal, tolerante con lo extremo | Robustez a outliers sin perder sensibilidad ([[07-Modelos-Supervisados]]) |
| Quantile / Pinball Loss | Asimétrica según el cuantil τ | Varía | No | Pedir el percentil 90 en vez del promedio: "dime el escenario pesimista" | Forecast con costos asimétricos (quiebre de stock vs sobre-stock, [[17-Series-de-Tiempo]]) |

> [!warning] ⚠️ R² puede ser negativo
> R² < 0 significa que el modelo es **peor que predecir siempre la media** del target. Ocurre al evaluar en un dominio distinto al de entrenamiento o con overfitting severo que colapsa en test. Si lo ves, el problema no es la métrica: es el modelo o la validación ([[10-Validacion-y-Leakage]]).

Las métricas específicas de forecasting (MASE, WAPE, cobertura de intervalos) están en [[17-Series-de-Tiempo]] §10.

---

## 8. Criterios de información: comparar modelos sin tocar el test

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Es la nota de un profesor que no solo revisa si la tarea salió bien, sino que descuenta puntos por cada fórmula extra que el alumno necesitó para llegar al resultado. Dos alumnos con la misma respuesta correcta no sacan la misma nota si uno llegó en dos pasos y el otro en ocho: probablemente el segundo memorizó el problema en vez de entenderlo. Los criterios de información hacen lo mismo con los modelos — premian el ajuste, pero cobran una multa por cada parámetro que costó lograrlo.

**🔧 Definición técnica:** combinan el ajuste del modelo (la log-verosimilitud `ln L`: qué tan bien explica los datos que ya vio) con una penalización por su complejidad (`k`, el número de parámetros), para comparar candidatos ajustados sobre el **mismo** conjunto de datos sin necesitar un test set separado. AIC (Akaike, 1974) y BIC (Schwarz, 1978) nacieron de derivaciones estadísticas distintas — no son la misma fórmula con otro nombre —, pero los tres criterios de esta tabla se leen igual: **menor valor = mejor modelo**.

| Criterio | Fórmula | Qué penaliza | Cuándo conviene |
|---|---|---|---|
| **AIC** (Akaike Information Criterion) | `AIC = 2k − 2·ln(L)` | Un costo fijo de 2 por parámetro, sin importar cuántos datos hay | Foco en capacidad predictiva antes que en "el modelo verdadero"; N pequeño/moderado; motor por defecto de búsquedas automáticas de orden como `auto_arima` ([[17-Series-de-Tiempo]]) |
| **BIC** (Bayesian/Schwarz Information Criterion) | `BIC = k·ln(n) − 2·ln(L)` | Más fuerte que AIC a medida que crece n: el término `ln(n)` encarece cada parámetro adicional con muestras grandes | Buscar el modelo más parsimonioso; n grande, donde AIC tiende a preferir modelos más complejos de lo necesario (Hyndman & Athanasopoulos, 2021) |
| HQIC (Hannan-Quinn) | `HQIC = 2k·ln(ln(n)) − 2·ln(L)` | Penalización intermedia entre AIC y BIC | Solo mención: aparece como opción adicional en algunas librerías de series de tiempo; uso bastante menos extendido que AIC/BIC |

**🧭 Cuándo usarlo:** cuando hay que elegir entre varios candidatos ajustados sobre los mismos datos — el orden (p,d,q) de un ARIMA, el K de un Gaussian Mixture Model ([[06-Clustering]]), cuántas variables entran a una regresión — y correr una validación cruzada completa por cada uno sale caro. El criterio de información da un ranking en un solo ajuste por candidato, como filtro previo antes de gastar cómputo de validación en los finalistas.

> [!warning] ⚠️ El error costoso: comparar peras con manzanas, y confundirlo con validación real
> 1. **Solo son comparables entre modelos ajustados sobre los MISMOS datos y la MISMA transformación.** Comparar el AIC de una serie en nivel contra el AIC de la misma serie diferenciada es inválido: la verosimilitud se calcula sobre una escala de datos distinta en cada caso, y el número deja de ser comparable entre sí (Hyndman & Athanasopoulos, 2021).
> 2. **NO sustituyen la validación out-of-sample.** Miden ajuste penalizado **in-sample** — qué tan bien explica el modelo los datos que ya vio, con descuento por complejidad —, no qué tan bien va a predecir datos nuevos. Elegir el modelo final solo por AIC/BIC, sin pasar por [[10-Validacion-y-Leakage]], repite con otro disfraz el error de evaluar sobre el propio set de entrenamiento.

**👔 En una frase para el negocio:** son un filtro rápido y barato para descartar candidatos antes de invertir tiempo de validación real — nunca el criterio final para darle luz verde a un modelo.

---

## 9. Guía rápida de selección de métrica

Audiencia: 🧭 👔

| Escenario | Métricas recomendadas |
|---|---|
| Clasificación balanceada | Accuracy + F1-macro |
| Desbalance moderado | AUC-ROC + F1-weighted |
| Desbalance severo | **AUC-PR + MCC** |
| FN muy costoso (no perder casos) | Recall, F2 |
| FP muy costoso (falsas alarmas caras) | Precision, F0.5 |
| Se usan las probabilidades aguas abajo | Log Loss + Brier (y calibrar, [[11-Mejora-de-Modelos]]) |
| Regresión estándar | RMSE + R² |
| Regresión robusta a outliers | MAE + Huber |
| Errores relativos / demanda | SMAPE, o MASE para comparar entre series ([[17-Series-de-Tiempo]]) |
| Escenarios asimétricos (stock, capacidad) | Quantile Loss |
| Ranking / recomendación | **NDCG@K + MAP@K** ([[20-Sistemas-de-Recomendacion]]) |
| Fairness / equidad entre grupos | Equalized Odds + Disparate Impact Ratio ([[13-MLOps-XAI-Etica]]) |

**Regla final 👔:** la métrica se elige **antes** de entrenar, escribiendo el costo de cada tipo de error en pesos. Si nadie puede poner precio al FP y al FN, la discusión de métricas es prematura.

---

## 📖 Referencias de este tomo

- (Brier, 1950), (Youden, 1950), (Cohen, 1960), (Matthews, 1975) — las métricas originales.
- (Davis & Goadrich, 2006) — la relación entre curvas ROC y PR.
- (Hastie et al., 2009), (James et al., 2021), (Géron, 2022) — evaluación de modelos en contexto.
- (Akaike, 1974), (Schwarz, 1978) — AIC y BIC, los criterios de información de la sección 6.
- (Hardt et al., 2016) — *Equality of Opportunity in Supervised Learning*. NeurIPS. El paper fundacional de Equalized Odds.
- (Chouldechova, 2017) — *Fair prediction with disparate impact*. Big Data 5(2). El teorema de imposibilidad.
- (Järvelin & Kekäläinen, 2002) — *Cumulated Gain-based Evaluation of IR Techniques*. ACM TOIS 20(4). NDCG original.
- (Hyndman & Koehler, 2006) — la asimetría del MAPE. (Hyndman & Athanasopoulos, 2021) — por qué los criterios de información no son comparables entre transformaciones distintas de los datos.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[07-Modelos-Supervisados|07 · Modelos Supervisados]] · Siguiente: [[09-Reglas-de-Asociacion|09 · Reglas de Asociación ➡]]

> **Próximo tomo:** [[09-Reglas-de-Asociacion]] — support, confidence, lift y compañía: descubrir qué cosas ocurren juntas en tus transacciones, y cuándo esas reglas engañan.
