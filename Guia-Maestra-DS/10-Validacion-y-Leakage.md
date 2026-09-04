---
title: "Tomo 10 — Validación y Data Leakage"
tags: [data-science, machine-learning, validacion, cross-validation, leakage]
audiencias: [tecnico, puente, ejecutivo]
tomo: 10
version: 6.4
updated: 2026-08-28
---

# 🛡️ Tomo 10 — Validación y Data Leakage

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[09-Reglas-de-Asociacion|09 · Reglas de Asociación]] · Siguiente: [[11-Mejora-de-Modelos|11 · Mejora de Modelos ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Un modelo mal validado puede reportar métricas increíbles y fallar en producción: es el estudiante que practica **con las respuestas del examen** — saca 100 en la casa y reprueba cuando las preguntas cambian. La validación correcta es la diferencia entre un modelo que *parece* funcionar y uno que *funciona*. Los errores de validación son silenciosos y caros, y el **data leakage** es su causa más común.

> [!abstract] 👔 Impacto ejecutivo
> Este tomo protege la inversión completa: de nada sirven datos perfectos y modelos brillantes si la evaluación es una ilusión.
>
> - **Decisiones que habilita:** aprobar modelos con evidencia honesta, comparar candidatos con rigor estadístico, detectar trampas antes del deployment.
> - **Costo de hacerlo mal:** el patrón más caro de la industria — métricas offline espectaculares, producción decepcionante, credibilidad del equipo de datos erosionada.
> - **Pregunta ejecutiva que responde:** *¿el rendimiento que me prometen sobrevivirá al contacto con el mundo real?*

> [!tip] 💡 Analogía general
> Validar es tomarle la prueba al alumno con ejercicios que **jamás vio**, en las mismas condiciones del examen real. Todo lo demás de este tomo son variaciones de esa idea: cómo repartir los ejercicios (esquemas de partición), cómo saber si la nota de dos alumnos difiere de verdad (tests estadísticos), y cómo descubrir si alguien vio la pauta antes (leakage).

---

## 1. Esquemas de Partición

Audiencia: 🔧 🧭

| Estrategia | Descripción | Ventajas | Limitaciones | Mejor tipo de data / volumen |
|---|---|---|---|---|
| Holdout simple (train/test) | Partición única aleatoria (80/20 o 70/30) | Rapidísimo; suficiente con N enorme | Alta varianza: el resultado depende de QUÉ cayó en test | Baselines rápidos; N > 1M donde CV es caro |
| Train / Val / Test | Tres particiones: entrenar / tunear hiperparámetros / estimación final intocable | La estimación final no se contamina con decisiones de tuning | Reduce el train efectivo | Proyectos formales con tuning ([[11-Mejora-de-Modelos]]); el test NO se toca hasta el final |
| K-Fold CV | K partes; K rondas donde cada fold es test una vez; métrica = promedio | Robusto; usa todos los datos para entrenar y evaluar; varianza reducida (Kohavi, 1995) | Entrena K modelos; no preserva proporciones de clase | El estándar general; K = 5 o 10 |
| Stratified K-Fold | K-Fold preservando la proporción de clases en cada fold | Estimación correcta con desbalance | Solo clasificación | **Siempre** en clasificación, más aún con desbalance ([[03-Preparacion-de-Datos]]) |
| Repeated K-Fold | K-Fold repetido M veces con splits distintos | Aún menos varianza de la estimación | M×K entrenamientos | Comparaciones finas con presupuesto de cómputo |
| LOOCV (Leave-One-Out) | K = N: cada muestra es test una vez | Casi sin sesgo; máximo uso de datos | Carísimo (N modelos); alta varianza con ruido | Datasets muy chicos (N < 50), modelos baratos |
| Leave-P-Out | Todas las combinaciones de P muestras como test | Exhaustivo | Combinatorialmente inviable salvo P pequeño y N chico | Solo casos minúsculos |
| Time Series Split (walk-forward / rolling-origin) | El train siempre **precede** temporalmente al test; variante **expanding** o **sliding** (ver nota) | Respeta la flecha del tiempo; sin leakage temporal | Sin aleatorización; los primeros folds entrenan con poca historia | Cualquier dato con dependencia temporal: ventas, sensores, finanzas |
| Purged K-Fold | Time series split con **gap** (embargo) entre fin de train e inicio de test | Previene fuga por autocorrelación entre vecinos temporales | Sacrifica datos en el gap | Trading algorítmico, sensores de alta frecuencia (mlfinlab) |

**Expanding vs. sliding window:** el Time Series Split (también llamado *walk-forward* o *rolling-origin*) tiene dos variantes. **Expanding window** — el train acumula toda la historia sin descartar nada — conviene con series **estables** y **poca historia** disponible: cada dato pasado sigue aportando señal. **Sliding window** — el train mantiene un largo fijo y "olvida" lo más antiguo — conviene cuando hay **drift** o cambios de régimen, porque historia muy vieja deja de representar el proceso actual y solo agrega ruido (Hyndman & Athanasopoulos, 2021; Bergmeir & Benítez, 2012). Igual de determinante: **validar al horizonte real de la decisión**, no al horizonte que resulte más cómodo de calcular (Tashman, 2000) — desarrollo completo de horizonte y métricas de forecasting en [[17-Series-de-Tiempo|Tomo 17 §10]].

```
 K-FOLD (K=5)                             TIME SERIES SPLIT
 ronda 1: [T][E][E][E][E]                 fold 1: [train──][test]
 ronda 2: [E][T][E][E][E]                 fold 2: [train──────][test]
 ronda 3: [E][E][T][E][E]                 fold 3: [train──────────][test]
 ...  T=test  E=entrenamiento             el test SIEMPRE es futuro del train
 métrica final = promedio de las rondas   (nunca se baraja el tiempo)
```

> [!warning] ⚠️ Regla temporal inquebrantable
> Si los datos tienen tiempo, el split lo respeta: **ordenar cronológicamente y separar por fecha**, jamás barajar. Validar un forecast con K-Fold aleatorio es entrenar con el diario de mañana — la métrica será hermosa y falsa.

### 1.1 Nested Cross-Validation — validar Y tunear sin trampas

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un examen con dos niveles: el profesor interno (inner CV) elige qué libro de texto es mejor para estudiar (model selection + tuning); el profesor externo (outer CV) evalúa al alumno con un examen que ninguno de los dos vio. Si el profesor interno también pusiera la nota final, estaría evaluando lo que él mismo optimizó — sesgo garantizado.

**🔧 Definición técnica:** dos loops de CV anidados:

- **Outer loop (K folds):** estima el rendimiento **generalizado** del mejor modelo — cada fold es un test set final.
- **Inner loop (dentro de cada fold del outer):** selecciona el modelo / hiperparámetros óptimos usando solo el train del fold externo.

```
 Outer fold 1: [=====train_outer=====][test_outer]
                     │
                     └─ Inner K-Fold: selecciona mejores hiperparams
                     └─ Entrena modelo final con TODOS los train_outer
                     └─ Evalúa en test_outer → score_1

 Outer fold 2: [test_outer][=====train_outer=====]
                                  │
                                  └─ Inner K-Fold ...
                                  └─ score_2
 ...
 Resultado final = promedio(score_1, score_2, ..., score_K)
```

**🔧 Cuándo es obligatorio:**

- Datasets pequeños (< 5K filas) donde un holdout de test pierde demasiados datos.
- Cuando se van a comparar múltiples familias de modelos Y tunear cada una — sin nested CV, el test set se "gasta" en las decisiones de selección.
- Papers y publicaciones donde se requiere una estimación insesgada del error.

**🧭 Cuándo puedes saltarlo:** con datasets grandes (> 100K), un train/val/test simple con holdout generoso es suficiente y mucho más barato computacionalmente.

### 1.2 GroupKFold — splits conscientes de entidades

Audiencia: 🔧 🧭

**🔧 Definición técnica:** variante de K-Fold donde todas las observaciones de un mismo **grupo** (paciente, tienda, cliente, sesión) caen en el mismo fold. Evita que el modelo memorice características idiosincráticas de una entidad en train y las reconozca en test.

**🔧 Cuándo es obligatorio:**

- **Medicina:** múltiples registros por paciente (visitas, mediciones). Sin GroupKFold, el modelo aprende "cómo habla el paciente 42" en vez del patrón general.
- **Retail multi-tienda:** predecir ventas por tienda — si la tienda A está en train Y test, el modelo memoriza la locación en vez de la dinámica.
- **Imágenes médicas / satelitales:** múltiples crops del mismo scan/imagen — split por imagen original, no por crop.
- **NLP con múltiples textos del mismo autor:** el modelo aprende estilo, no contenido.

**🔧 Variantes en sklearn:** `GroupKFold`, `StratifiedGroupKFold` (preserva balance de clases dentro de la restricción de grupo), `LeaveOneGroupOut` (el equivalente de LOOCV pero por grupo).

### 1.3 Adversarial Validation — ¿mi train y test son del mismo planeta?

Audiencia: 🔧 🧭

**🔧 Definición técnica:** entrenar un clasificador binario para distinguir "¿esta fila viene de train o de test?". Si el AUC es ~0.5, ambos conjuntos son indistinguibles (bien). Si es >> 0.5, hay un **dataset shift** entre train y test — el modelo va a fallar por distributional mismatch, no por falta de capacidad.

**🔧 Qué hacer si el AUC es alto (> 0.7):**

- Revisar las features con mayor importancia en el clasificador adversarial — suelen ser timestamps, IDs, o variables que cambiaron de definición entre periodos.
- Considerar remover esas features del modelo principal, o resamplear el train para que se parezca más al test.
- Si el shift es temporal (train = 2023, test = 2025), puede ser **concept drift** legítimo — y el modelo necesita datos más recientes, no más features.

**🧭 Cuándo usarlo:** siempre que el modelo rinda bien en CV offline pero mal en producción. Es el diagnóstico de primera línea para el síntoma "mis métricas se degradan al deployar".

---

## 2. Evaluación Estadística de Resultados

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Dos corredores terminan en 10.81s y 10.79s. ¿El segundo es "más rápido", o es viento y azar de ese día? Repetir la carrera (folds) y aplicar el test correcto es la única forma de saberlo. Con modelos: 0.82 vs 0.80 de AUC en un split único **no** declara un ganador.

**🔧 Definición técnica:**

- **Intervalo de confianza del CV:** reportar media ± std de los folds; IC 95% ≈ `media ± 1.96·std/√K` ([[02-Fundamentos-Matematicos]]). Dos modelos con IC ampliamente traslapados no son distinguibles con esos datos.
- **Paired t-test sobre folds:** compara los K scores de ambos modelos **sobre los mismos folds** (pareado). Cuidado: los folds comparten datos de train → no son independientes → el t-test estándar es demasiado optimista.
- **Corrección de Nadeau-Bengio:** ajusta la varianza del t-test por la correlación entre folds (Nadeau & Bengio, 2003) — el test correcto para comparar modelos vía CV repetido.
- **Test de McNemar:** para clasificadores evaluados en el MISMO test set: examina la matriz de desacuerdos (casos donde uno acierta y el otro no); apropiado cuando reentrenar K veces es inviable.

**👔 En una frase para el negocio:** antes de premiar al "modelo ganador por dos décimas", exige el test que confirme que esas décimas no son ruido — cambiar de modelo también tiene costo.

---

## 3. Data Leakage — el error silencioso

Audiencia: 🔧 🧭 👔

> [!danger] 🚨 El error más costoso del ML aplicado
> El leakage produce modelos que parecen excelentes en desarrollo y fallan en producción. Es la causa Nº1 de la brecha entre métricas offline y realidad (Kaufman et al., 2012). Es silencioso: nada "falla" — los números simplemente mienten.

> [!tip] 💡 Analogía general
> Es el alumno que encontró la pauta del examen en la fotocopiadora. Sus notas de práctica son perfectas — y no aprendió nada. El día del examen real (producción), donde no hay pauta que copiar, se derrumba. Peor: durante meses, todos celebraron sus notas.

### 3.1 Tipos de leakage

Audiencia: 🔧 🧭

| Tipo | Qué es | Ejemplo típico |
|---|---|---|
| Target leakage | Una feature contiene información del target o solo disponible DESPUÉS del evento | Predecir si el paciente tomará un medicamento usando "tomó medicamento 30 días después"; la feature `dias_hasta_proximo_control` del caso del [[04-EDA|Tomo 04]] |
| Train-test contamination | Preprocesamiento ajustado ANTES del split: imputar/escalar/seleccionar features con TODO el dataset | `scaler.fit(X)` antes de separar ([[05-Escalado-de-Datos]]); SMOTE aplicado al test ([[03-Preparacion-de-Datos]]); target encoding sin folds |
| Leakage temporal | Información del futuro para predecir el pasado | Usar el promedio anual para predecir enero (el promedio incluye feb–dic); features rolling mal alineadas; usar los valores REALES futuros de variables exógenas (clima, precio del competidor) al validar — en producción no los tendrás, hay que pronosticarlos o usar solo regresores conocidos de antemano ([[17-Series-de-Tiempo|Tomo 17 §7]]) |
| Leakage por duplicados | La misma entidad (o casi) presente en train y test | Cliente duplicado con formatos distintos; imágenes aumentadas repartidas entre train y test |
| Leakage por ID / timestamp | IDs o timestamps que codifican el target indirectamente | IDs correlativos donde los fraudes se cargaron al final; timestamp que delata el proceso de etiquetado |

### 3.2 Cómo detectarlo

Audiencia: 🔧 🧭

- **Métrica sospechosamente alta:** AUC > 0.98 en un problema real de negocio es casi siempre leakage — la perfección en datos reales es una bandera roja, no un triunfo.
- **Importancia concentrada:** una sola feature explica casi todo ([[13-MLOps-XAI-Etica]]) → probable proxy del target.
- **Correlación feature-target > 0.95** en el EDA ([[04-EDA]]).
- **Gap CV vs producción:** la señal definitiva (y la más cara): brillante offline, mediocre online.

### 3.3 Cómo prevenirlo

Audiencia: 🔧 🧭 👔

- **Pipeline de sklearn para TODO el preprocesamiento:** cada fold re-ajusta imputers, scalers, encoders y selección solo con su train ([[05-Escalado-de-Datos]], [[03-Preparacion-de-Datos]]).
- **Separación temporal por diseño:** en datos con tiempo, split cronológico + gap si hay autocorrelación.
- **Auditoría de features, una por una:** ¿estaba disponible este dato en el momento exacto de la predicción? ¿podría codificar el target por la puerta trasera? La auditoría de 30 minutos más rentable del proyecto.

**👔 En una frase para el negocio:** cuando un resultado parece demasiado bueno para ser verdad, la respuesta correcta no es celebrar — es auditar; el leakage se paga con intereses después del deployment.

> [!example] 📊 Caso de negocio — Telco: el churn model de AUC 0.97 que no retuvo a nadie
> **Problema:** un operador de telecomunicaciones entrena un modelo para predecir churn (fuga de clientes) y alcanza un **AUC de 0.97** en validación. El equipo lo celebra y lanza una campaña de retención dirigida por el modelo. Tres meses después, la campaña no mueve la aguja: los clientes marcados como "seguros de irse" o bien ya se habían ido, o no tenían intención de hacerlo. El modelo era un oráculo en desarrollo y un inútil en producción.
>
> **Técnica aplicada:** auditoría de features una por una (sección 3.3). Aparecen **dos fugas**. Una de **target leakage**: la feature `dias_desde_gestion_retencion` estaba poblada porque el equipo de retención *ya* contactaba a los clientes en riesgo — la feature no predecía el churn, lo **reflejaba**. Otra de disponibilidad temporal: `estado_linea` incluía el valor `baja_solicitada`, que solo existe **después** de que el cliente decidió irse. Se reconstruye el dataset con criterio *point-in-time*: únicamente features disponibles en el instante de la predicción, y se cambia el K-Fold aleatorio por **Time Series Split** (sección 1) para respetar la flecha del tiempo.
>
> **Resultado:** el AUC honesto **cae a 0.78** — un número mucho menos glamoroso, pero real. Con ese modelo, la campaña por fin identifica clientes accionables *antes* de que decidan irse, y el uplift se vuelve medible ([[18-Causalidad-y-Uplift]]). La lección: **un AUC de 0.97 en un problema de negocio real casi nunca es un triunfo; es una invitación a auditar.** El modelo "peor" fue el que finalmente generó valor.

---

## 📖 Referencias de este tomo

- (Kohavi, 1995) — estudio clásico de cross-validation y holdout.
- (Nadeau & Bengio, 2003) — la corrección de varianza para comparar modelos con CV.
- (Kaufman et al., 2012) — taxonomía del leakage en minería de datos.
- (Hastie et al., 2009), (Géron, 2022) — validación en el flujo completo.
- (Hyndman & Athanasopoulos, 2021) — *Forecasting: Principles and Practice*, respaldo de la validación walk-forward y sus variantes expanding/sliding.
- (Bergmeir & Benítez, 2012) — uso correcto de cross-validation en predictores de series de tiempo.
- (Tashman, 2000) — revisión de pruebas out-of-sample y del efecto del horizonte en la precisión del pronóstico.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[09-Reglas-de-Asociacion|09 · Reglas de Asociación]] · Siguiente: [[11-Mejora-de-Modelos|11 · Mejora de Modelos ➡]]

> **Próximo tomo:** [[11-Mejora-de-Modelos]] — del diagnóstico bias-variance al tuning de hiperparámetros, la regularización, los ensembles, la calibración de probabilidades y el ajuste fino del umbral.
