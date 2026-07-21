---
title: "Tomo 15 — Glosario Ejecutivo"
tags: [data-science, machine-learning, glosario, ejecutivo]
audiencias: [ejecutivo, puente]
tomo: 15
version: 6.0
---

# 📖 Tomo 15 — Glosario Ejecutivo

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[14-Anexo-Interpretar-Resultados|14 · Anexo: Interpretar Resultados]] · Siguiente: [[16-Bibliografia|16 · Bibliografía ➡]]

---

> [!info] 📌 ¿Por qué importa este glosario?
> El diccionario para las reuniones: cada término técnico traducido a **una frase de negocio**. No pretende rigor de manual (para eso están los tomos técnicos, enlazados en cada término): pretende que nunca más asientas con cara de entender.

Audiencia: 👔 🧭

## A–C

| Término | Traducción a lenguaje de negocio |
|---|---|
| Accuracy | El % de aciertos totales — el número que más engaña cuando lo que buscas es raro ([[08-Metricas-de-Evaluacion]]) |
| A/B testing | Probar dos versiones con clientes reales y dejar que los datos declaren el ganador — con reglas para no autoengañarse ([[02-Fundamentos-Matematicos]]) |
| ARIMA | El modelo clásico de pronóstico: proyecta el futuro usando los propios rezagos de la serie y sus errores recientes ([[17-Series-de-Tiempo]]) |
| AUC | Nota de 0.5 a 1.0 de qué tan bien el modelo ordena los casos de mayor a menor riesgo; 0.5 = moneda al aire ([[08-Metricas-de-Evaluacion]]) |
| Bandit (multi-armed) | El experimento que aprende sobre la marcha: mueve el tráfico hacia la opción ganadora mientras sigue probando ([[21-Supervivencia-y-Bandits]]) |
| Baseline | Lo que lograrías con el método más simple posible — la vara que todo modelo debe superar para justificar su existencia |
| Batch | Tanda de datos que se procesa de una vez (lo contrario de "en tiempo real") |
| Bias (sesgo del modelo) | El modelo es demasiado simple y se equivoca sistemáticamente: le queda grande el problema ([[11-Mejora-de-Modelos]]) |
| Boosting | Comité de modelos donde cada nuevo integrante se especializa en los errores del anterior — el campeón actual en datos de tablas ([[07-Modelos-Supervisados]]) |
| Calibración | Que el "90% de probabilidad" del modelo signifique de verdad 90 de cada 100 — requisito para usar los números en decisiones de plata ([[11-Mejora-de-Modelos]]) |
| Censura | El caso que sigue "vivo" al cierre del estudio: no sabes su duración final, solo que supera lo observado — botarlo o falsearlo sesga todo ([[21-Supervivencia-y-Bandits]]) |
| Churn | Fuga de clientes; "modelo de churn" = lista priorizada de quiénes están por irse |
| Clustering | Descubrir los grupos naturales que existen en tus datos sin decirle al sistema qué buscar ([[06-Clustering]]) |
| Cold start | El problema del recién llegado: sin historial no hay con qué personalizar ([[20-Sistemas-de-Recomendacion]]) |
| Confounder | El tercero escondido que mueve a las otras dos variables y fabrica la correlación — el enemigo nº1 de las conclusiones causales ([[18-Causalidad-y-Uplift]]) |
| Cross-validation | Tomar el examen varias veces con distintas preguntas para que la nota sea confiable y no suerte de un solo intento ([[10-Validacion-y-Leakage]]) |

## D–F

| Término | Traducción a lenguaje de negocio |
|---|---|
| Data drift | Los datos de hoy ya no se parecen a los del entrenamiento: el modelo opina sobre un mundo que dejó de existir ([[13-MLOps-XAI-Etica]]) |
| Data leakage | El modelo hizo trampa sin querer: vio información del futuro o de la respuesta durante el entrenamiento; brilla en el laboratorio y fracasa en la calle ([[10-Validacion-y-Leakage]]) |
| Dataset | El conjunto de datos: una tabla donde cada fila es un caso y cada columna una característica |
| Deep learning | Modelos de muchas capas que aprenden solos qué mirar — el motor de la visión artificial y los asistentes de lenguaje ([[12-Deep-Learning]]) |
| Deployment | Sacar el modelo del laboratorio y ponerlo a decidir en la operación real ([[13-MLOps-XAI-Etica]]) |
| EDA | La inspección de la casa antes de comprarla: revisar los datos a fondo antes de apostar un proyecto sobre ellos ([[04-EDA]]) |
| Embedding | Traducir cosas (palabras, clientes, productos) a coordenadas donde "parecido" significa "cerca" — así la máquina puede comparar lo incomparable ([[12-Deep-Learning]]) |
| Ensemble | Jurado de varios modelos votando juntos: casi siempre le gana al mejor juez solitario ([[11-Mejora-de-Modelos]]) |
| Estacionalidad | El patrón que se repite con calendario fijo: diciembre siempre se parece a diciembre ([[17-Series-de-Tiempo]]) |
| Feature | Cada variable que el modelo usa para decidir (edad, monto, días desde la última compra) |
| Forecasting | Pronosticar los valores futuros de una serie respetando el orden del tiempo — con intervalo, no solo el punto ([[17-Series-de-Tiempo]]) |
| Feature engineering | Cocinar variables nuevas con conocimiento del negocio — históricamente, la palanca de mejora más rentable ([[03-Preparacion-de-Datos]]) |
| Feature importance | El ranking de qué variables pesan más en las decisiones del modelo ([[14-Anexo-Interpretar-Resultados]]) |
| Fine-tuning | Tomar un modelo gigante ya educado y darle el curso de especialización en tu negocio ([[12-Deep-Learning]]) |
| Fairness | Que el modelo no trate sistemáticamente peor a un grupo protegido — se audita con métricas, no se asume ([[13-MLOps-XAI-Etica]]) |

## G–O

| Término | Traducción a lenguaje de negocio |
|---|---|
| Gradient descent | El método de aprendizaje: bajar la montaña del error a pasitos, corrigiendo el rumbo en cada paso ([[02-Fundamentos-Matematicos]]) |
| Hiperparámetro | Las perillas de configuración del modelo que se ajustan ANTES de entrenar — tunearlas es exprimir el motor ([[11-Mejora-de-Modelos]]) |
| Inferencia (serving) | El modelo respondiendo casos nuevos en producción; su costo por respuesta importa tanto como su calidad |
| Label / etiqueta | La respuesta correcta histórica con la que se entrena ("este cliente sí se fugó") |
| Learning rate | El tamaño del paso al aprender: muy grande se pasa de largo, muy chico no llega nunca ([[12-Deep-Learning]]) |
| LLM | Modelo de lenguaje gigante (tipo GPT): un Transformer entrenado con medio internet que ahora se adapta barato a cada negocio ([[12-Deep-Learning]]) |
| MLOps | La disciplina de operar modelos como se opera un sistema crítico: versionado, monitoreo, alarmas, reentrenamiento ([[13-MLOps-XAI-Etica]]) |
| Modelo | La fórmula aprendida de los datos que convierte características en predicciones |
| NDCG | La nota de un ranking: premia poner lo más relevante arriba, que es donde vive la atención ([[20-Sistemas-de-Recomendacion]]) |
| Outlier | El caso raro que se sale del patrón: error de tipeo, cliente excepcional o fraude — diagnosticarlo antes de borrarlo ([[03-Preparacion-de-Datos]]) |
| **Overfitting** | **El modelo aprendió tan bien el pasado que no sirve para el futuro** — memorizó el examen viejo en vez de aprender la materia ([[11-Mejora-de-Modelos]]) |

## P–S

| Término | Traducción a lenguaje de negocio |
|---|---|
| Pipeline | La línea de producción del dato: todos los pasos (limpiar, transformar, predecir) empaquetados para ejecutarse siempre igual ([[13-MLOps-XAI-Etica]]) |
| Precision | De todos los que el modelo acusó, ¿cuántos eran culpables? — la métrica de las falsas alarmas caras ([[08-Metricas-de-Evaluacion]]) |
| Propensity score | La probabilidad de haber recibido el tratamiento: permite comparar comparables cuando no hubo experimento ([[18-Causalidad-y-Uplift]]) |
| RAG | Darle al LLM tus documentos como apuntes abiertos antes de responder, para que cite en vez de inventar ([[19-NLP-y-LLMs]]) |
| Recall | De todos los culpables que había, ¿a cuántos atrapó? — la métrica de los casos que no se pueden escapar ([[08-Metricas-de-Evaluacion]]) |
| Regularización | El freno anti-memorización: sacrifica un poco de ajuste al pasado a cambio de mucha más confiabilidad en el futuro ([[11-Mejora-de-Modelos]]) |
| Reentrenamiento | Volver a educar el modelo con datos frescos porque el mundo cambió ([[13-MLOps-XAI-Etica]]) |
| ROC | El menú de todos los balances posibles entre detectar más y equivocarse más ([[14-Anexo-Interpretar-Resultados]]) |
| Scoring | Ponerle nota de riesgo/propensión a cada cliente o caso, para priorizar con datos y no con intuición |
| SHAP | El desglose de "por qué el modelo decidió esto" para cada caso individual — la boleta detallada de cada predicción ([[13-MLOps-XAI-Etica]]) |
| SMOTE | Fabricar ejemplos sintéticos del caso raro (fraude, falla) para que el modelo tenga con qué practicar ([[03-Preparacion-de-Datos]]) |

## T–Z

| Término | Traducción a lenguaje de negocio |
|---|---|
| Target | Lo que se quiere predecir: la columna respuesta |
| Test set | Los datos guardados bajo llave para el examen final: el modelo jamás los ve durante el entrenamiento ([[10-Validacion-y-Leakage]]) |
| Threshold (umbral) | El punto de corte donde el puntaje se convierte en acción ("sobre 0.7, llamar") — un dial de negocio, no una constante técnica ([[08-Metricas-de-Evaluacion]]) |
| Tokenización | Cortar el texto en las piezas mínimas que el modelo realmente lee ([[19-NLP-y-LLMs]]) |
| Training | El proceso de aprender de los datos históricos: la etapa cara en cómputo |
| Transfer learning | Contratar experiencia en vez de formar desde cero: partir de un modelo pre-entrenado y adaptarlo ([[12-Deep-Learning]]) |
| Transformer | La arquitectura estrella actual: lee todo el contexto a la vez y decide a qué prestar atención — el motor de los LLMs ([[12-Deep-Learning]]) |
| Underfitting | El modelo es tan simple que ni siquiera aprendió el pasado: le falta materia gris o mejores variables ([[11-Mejora-de-Modelos]]) |
| Uplift | No quién comprará, sino a quién la acción le CAMBIA la decisión: el efecto causal por persona ([[18-Causalidad-y-Uplift]]) |
| Validación | El control de calidad de la promesa: medir el modelo en condiciones que imitan el futuro real ([[10-Validacion-y-Leakage]]) |
| Varianza (del modelo) | El modelo cambia demasiado con cada muestra: inestable, resultados de lotería ([[11-Mejora-de-Modelos]]) |
| XAI | Explicabilidad: las técnicas para que el modelo muestre sus razones y deje de ser caja negra ([[13-MLOps-XAI-Etica]]) |

> [!tip] 💡 Cómo usar este glosario
> En la próxima reunión, cuando aparezca un término, no pidas que "lo expliquen simple" (nadie lo hace bien improvisando): búscalo aquí, y usa el wikilink si quieres el detalle técnico completo. 65 términos; si falta uno, es candidato a la próxima versión.

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[14-Anexo-Interpretar-Resultados|14 · Anexo: Interpretar Resultados]] · Siguiente: [[16-Bibliografia|16 · Bibliografía ➡]]
