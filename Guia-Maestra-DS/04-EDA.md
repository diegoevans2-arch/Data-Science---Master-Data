---
title: "Tomo 04 — EDA: Análisis Exploratorio de Datos"
tags: [data-science, machine-learning, eda, visualizacion, data-quality]
audiencias: [tecnico, puente, ejecutivo]
tomo: 04
version: 6.1
updated: 2026-07-29
---

# 🔍 Tomo 04 — EDA: Análisis Exploratorio de Datos

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[03-Preparacion-de-Datos|03 · Preparación de Datos]] · Siguiente: [[05-Escalado-de-Datos|05 · Escalado de Datos ➡]]

---

> [!info] 📌 ¿Por qué importa el EDA?
> El EDA es el paso que te dice "oye, tus datos están sesgados", "esta variable no sirve para nada" o "hay una relación oculta que ningún modelo va a encontrar sin tu ayuda". **Entrenar sin EDA es conducir con los ojos cerrados.** Es la auditoría previa que decide dónde invertir el esfuerzo de preparación ([[03-Preparacion-de-Datos]]) y la primera línea de defensa contra el data leakage ([[10-Validacion-y-Leakage]]). Y no es un paso único: se hace **antes** del preprocesamiento para guiar las decisiones y **después** para validar que las transformaciones hicieron lo esperado.

> [!abstract] 👔 Impacto ejecutivo
> El EDA es la inspección técnica antes de comprometer recursos en entrenamiento: barato de hacer, carísimo de saltar.
>
> - **Decisiones que habilita:** aprobar (o frenar) el paso a modelado con evidencia, dimensionar el esfuerzo real de limpieza, detectar a tiempo sesgos y trampas que invalidarían todo lo posterior.
> - **Costo de hacerlo mal:** semanas de modelado sobre datos con errores de unidades, leakage descubierto post-deployment, y "hallazgos" que eran artefactos de calidad de datos.
> - **Pregunta ejecutiva que responde:** *¿estos datos son suficientemente sanos y honestos como para apostar un proyecto sobre ellos?*

> [!tip] 💡 Analogía general: la inspección de la casa antes de comprarla
> Nadie compra una casa solo por las fotos del anuncio. Contratas un inspector que revisa cimientos (calidad de datos), instalaciones (relaciones entre variables), humedades escondidas (nulos y outliers) y si la ampliación tiene permisos (leakage). El EDA es esa inspección: dos días de trabajo que te salvan de comprar una ruina remodelada con maquillaje.

> [!example] 📊 Caso de negocio — Salud: la auditoría que evitó el deployment de una trampa
> **Problema:** un grupo hospitalario construye un modelo de reingreso a 30 días. El equipo, presionado por el calendario, salta directo al modelado y celebra un AUC de 0.97. Antes del deployment, se exige un EDA formal.
>
> **Técnica aplicada:** el EDA disciplinado encuentra cuatro problemas en dos días: (1) el histograma de glucosa es **bimodal** — dos hospitales del grupo registran en unidades distintas (mg/dL vs mmol/L); (2) el target tiene 8% de positivos y nadie había planificado el manejo del desbalance ([[03-Preparacion-de-Datos]]); (3) los nulos de presión arterial se concentran en pacientes de urgencias — missingness MNAR, no aleatoria; (4) la feature `dias_hasta_proximo_control` tiene mutual information altísima con el target… porque **solo se registra para pacientes que ya reingresaron**: leakage de manual ([[10-Validacion-y-Leakage]]).
>
> **Resultado:** corregidas las unidades, eliminada la feature filtrada y tratado el desbalance, el AUC honesto es 0.79 — y ese sí se sostiene en producción. El hallazgo de unidades, de paso, corrige los reportes clínicos del grupo. Dos días de EDA evitaron un deployment tramposo y un escándalo clínico.

**El EDA es de doble pasada:**

```
              ┌────────────── EDA · 1ª pasada ───────────────┐
              │ ¿qué hay? ¿qué falta? ¿qué está raro?        │
 Datos crudos ┤ ¿el target está sano? ¿hay señales de        ├──► decisiones de limpieza,
              │ leakage?                                     │    imputación y encoding
              └──────────────────────────────────────────────┘    ([[03-Preparacion-de-Datos]])
                                   │
                                   ▼
                    Preparación y transformaciones
                                   │
                                   ▼
              ┌────────────── EDA · 2ª pasada ───────────────┐
              │ ¿las transformaciones hicieron lo esperado?  │
              │ ¿la imputación deformó distribuciones?       ├──► dataset validado
              │ ¿apareció algo nuevo?                        │    → modelar
              └──────────────────────────────────────────────┘
```

> [!warning] ⚠️ Los estadísticos mienten sin gráficos
> El cuarteto de (Anscombe, 1973): cuatro datasets con la **misma** media, varianza, correlación y recta de regresión — y formas completamente distintas (una lineal, una curva, una con un outlier que fabrica la relación…). Moraleja permanente: `describe()` nunca reemplaza al gráfico.

---

## 1. Análisis Univariado

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Antes de la reunión grupal, el buen jefe hace un 1:1 con cada persona del equipo. El univariado es eso: entrevistar a **cada columna por separado** — quién es, cómo se distribuye, qué esconde — antes de estudiar cómo se relacionan entre sí.

### 1.1 Variables numéricas continuas

Audiencia: 🔧

**🔧 Definición técnica — el kit completo:**

| Herramienta | Qué revela | 💡 Analogía | Detalles clave |
|---|---|---|---|
| Estadísticos descriptivos | Centro, dispersión y extremos: mean, median, std, min, max, Q1, Q3, P5, P95, P99 (`describe()` + percentiles custom) | La ficha médica básica de la columna | Si media ≫ mediana → asimetría positiva; revisar P99 vs max delata outliers |
| Histograma | La forma real de la distribución | El censo por tramos: cuánta gente hay en cada rango de edad | El nº de bins cambia la historia: Sturges `k = 1 + log₂(N)`, regla de la raíz `k = √N`, o Freedman-Diaconis (ancho `2·IQR/N^(1/3)`, robusta a outliers). Probar más de uno |
| KDE plot | Versión suavizada y continua del histograma | La silueta de la montaña dibujada a mano alzada | No presuponer normalidad: mirar bimodalidades (¡mezcla de poblaciones o de unidades!) |
| Boxplot | Resumen de 5 números + outliers | La radiografía compacta: mediana, caja (Q1–Q3), bigotes (1.5×IQR) y puntos fuera | Ideal para comparar la misma variable entre grupos |
| Skewness | Asimetría de la distribución ([[02-Fundamentos-Matematicos]]) | ¿La cola arrastra la media, como el multimillonario del barrio? | Positiva fuerte → candidata a log/Box-Cox ([[05-Escalado-de-Datos]]) |
| Kurtosis | Peso de las colas | ¿Cada cuánto llega la ola gigante? | > 3 = colas pesadas: más outliers que la Normal; revisar [[03-Preparacion-de-Datos]] |

**🧭 Cuándo usarlo:** para cada numérica, sin excepción. Qué buscar: bimodalidad (mezcla de poblaciones o de unidades — ver caso de negocio), picos artificiales (valores de relleno tipo −99 o 999), truncamientos (¿todo corta justo en un límite de sistema?), y colas que exigirán transformación o tratamiento de outliers.

**👔 En una frase para el negocio:** es el chequeo columna por columna que descubre errores de origen (unidades, rellenos, topes de sistema) que ningún promedio agregado va a mostrar.

### 1.2 Variables categóricas

Audiencia: 🔧

**🔧 Definición técnica — el kit completo:**

| Herramienta | Qué revela | Detalles clave |
|---|---|---|
| Frecuencias absolutas y relativas | Peso de cada categoría: `value_counts()`, `value_counts(normalize=True)` | Categorías con frecuencia casi nula → candidatas a agruparse en 'Other' antes del encoding |
| Cardinalidad | Nº de valores únicos: `nunique()` | Alta cardinalidad (> 50 categorías) exige estrategia especial de encoding ([[03-Preparacion-de-Datos]]): Target/Frequency/Hashing en lugar de One-Hot |
| Barplot / countplot | Distribución visual de categorías | Ordenar por frecuencia para lectura inmediata; el orden alfabético esconde el patrón |

> [!tip] 💡 Analogía
> La cardinalidad es el menú del restaurante: 12 platos se gestionan; 900 platos son una señal de que algo anda mal — o de que muchos "platos" son el mismo escrito distinto ("Stgo" / "Santiago"), un problema de normalización de texto ([[03-Preparacion-de-Datos]]), no de cocina.

**🧭 Cuándo usarlo:** toda categórica antes del encoding. Qué buscar: variantes de escritura de la misma categoría, cardinalidad inesperada (¿por qué "región" tiene 87 valores si el país tiene 16?), categorías dominantes (> 90% en una sola → poca señal) y categorías nuevas que podrían aparecer en producción.

**👔 En una frase para el negocio:** revela si tus catálogos (productos, regiones, canales) están sanos — la fragmentación de categorías corrompe silenciosamente todos los reportes por segmento.

---

## 2. Análisis Bivariado

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Después de los 1:1, toca ver cómo **bailan en pareja**: hay dúos que se coordinan perfecto (correlación fuerte), dúos que se pisan (relación inversa) y dúos que bailan canciones distintas (independencia). El bivariado examina cada pareja de variables — y la elección de gráfico y test depende del tipo de cada bailarín.

**🔧 La matriz de decisión completa** (gráfico + test según los tipos involucrados):

| Combinación | Gráficos | Test estadístico ([[02-Fundamentos-Matematicos]]) | Qué buscar |
|---|---|---|---|
| Numérica vs Numérica | Scatter plot; hexbin o KDE 2D cuando hay demasiados puntos | Correlación de Pearson (lineal) y Spearman (monotónica) | Forma de la relación (¿lineal, curva, umbral?), clusters, outliers bivariados, relaciones que Pearson no ve (r≈0 con patrón claro) |
| Numérica vs Categórica | Boxplot por grupo; violin plot (boxplot + KDE); barplot de medias con intervalos de confianza | t-test (2 grupos), ANOVA (3+), Mann-Whitney / Kruskal-Wallis si no hay normalidad | ¿Las distribuciones difieren entre grupos de verdad o solo en el gráfico? Validar con el test antes de declarar hallazgo |
| Categórica vs Categórica | Crosstab con proporciones; heatmap de la tabla de contingencia | Chi-cuadrado de independencia (frecuencias esperadas ≥ 5) | Dependencias entre catálogos (¿el plan contratado depende de la región?), celdas vacías o dominantes |

### Heatmap de correlación

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El mapa de amistades del curso: de un vistazo ves qué variables "andan siempre juntas". Dos features abrazadas (│r│ > 0.8) son sospechosas de contar la misma historia — y un modelo lineal no sabrá a cuál darle el crédito (multicolinealidad, [[02-Fundamentos-Matematicos]]).

**🔧 Definición técnica:** matriz de correlación de Pearson entre todas las numéricas, visualizada con máscara triangular (la mitad es espejo). Buscar: pares con │r│ > 0.8 (candidatos a fusión o eliminación), features con r ≈ 0 contra todo (candidatas a salir… tras verificar no-linealidad con mutual information, [[03-Preparacion-de-Datos]]), y bloques de features correlacionadas (familias redundantes).

**🧭 Cuándo usarlo:** siempre, temprano y barato. Limitación: solo ve relaciones **lineales** — complementar con Spearman para monotónicas y mutual information para el resto.

**👔 En una frase para el negocio:** detecta indicadores duplicados disfrazados de distintos — pagar por almacenar y mantener dos veces la misma señal es más común de lo que parece.

### Pairplot

Audiencia: 🔧

> [!tip] 💡 Analogía
> El speed-dating de las variables: todas las parejas posibles en una sola grilla — scatter de cada par y la distribución de cada una en la diagonal. Con más de ~15 features la fiesta se vuelve ilegible: selecciona antes a los candidatos.

**🔧 Definición técnica:** `seaborn.pairplot(df, hue='target')` — grilla de scatters por par + histograma/KDE en la diagonal, coloreable por clase. Útil para ver separabilidad de clases y relaciones a granel. Costoso en cómputo y en píxeles: usar con subconjuntos de features (< 15) o con una muestra de filas.

**🧭 Cuándo usarlo:** tras preseleccionar las features más prometedoras; excelente para presentar estructura general y detectar clusters visuales que motiven un [[06-Clustering]].

**👔 En una frase para el negocio:** la foto panorámica del dataset — una imagen que resume cien tablas para decidir en qué relaciones profundizar.

---

## 3. Análisis del Target

Audiencia: 🔧 🧭 👔

> [!info] 📌 Por qué importa
> El target es la variable que el modelo va a aprender: si está desbalanceado, sesgado o contaminado, **todo lo demás hereda el problema**. Es el análisis con mayor retorno por minuto invertido de todo el EDA.

> [!tip] 💡 Analogía
> Es estudiar al rival antes del partido: puedes entrenar mil jugadas (features), pero si no sabes cómo juega el equipo contrario (target) — si casi nunca ataca (clase rara), si cambia de táctica por temporada (drift), si te filtraron su plan (leakage) — vas a preparar el partido equivocado.

### 3.1 Target de clasificación

Audiencia: 🔧

**🔧 Definición técnica:** `value_counts(normalize=True)` para el balanceo. Si la clase minoritaria < 10%, diseñar la estrategia de desbalance **desde el inicio** ([[03-Preparacion-de-Datos]]) y elegir métricas acordes — accuracy queda descartada de plano ([[08-Metricas-de-Evaluacion]]). Verificar también: ¿las clases significan lo mismo en todo el histórico? (cambios de definición de "churn" a mitad de período son más comunes de lo que se admite).

### 3.2 Target de regresión

Audiencia: 🔧

**🔧 Definición técnica:** histograma + estadísticos del target: rango, media vs mediana, outliers extremos. Si es muy asimétrico, considerar transformar con `log1p` (y **revertir con `expm1`** al reportar predicciones — las métricas en escala log no se comunican al negocio). Outliers del target merecen diagnóstico propio: ¿errores o los casos más valiosos? ([[03-Preparacion-de-Datos]]).

### 3.3 Relación features–target

Audiencia: 🔧

**🔧 Definición técnica:** para cada feature: scatter vs target (numérica) o boxplot por clase (categórica); mutual information con el target para capturar relaciones no lineales ([[03-Preparacion-de-Datos]]). Detectar features con **cero** relación (candidatas a salir) y features con relación **demasiado perfecta** (candidatas a leakage — ver abajo).

### 3.4 Detección temprana de leakage

Audiencia: 🔧 👔

**🔧 Definición técnica:** tres banderas rojas en el EDA: (1) feature con correlación r ≈ 1 (o mutual information desproporcionada) con el target; (2) nombres sospechosos — columnas que contienen "resultado", "final", "aprobado", "post_", o IDs/timestamps generados después del evento; (3) features con disponibilidad temporal dudosa: ¿este dato existía **en el momento de la predicción**? El catálogo completo de tipos de leakage y su prevención está en [[10-Validacion-y-Leakage]].

**👔 En una frase para el negocio:** cinco minutos preguntando "¿y este dato existía cuando había que decidir?" valen más que cualquier métrica de laboratorio.

> [!danger] 🚨 La feature demasiado buena para ser verdad
> Si una sola feature "explica" casi todo el target, no celebres: audita. En 9 de cada 10 casos es un proxy del target que se registró **después** del evento (el `dias_hasta_proximo_control` del caso de negocio). El modelo no descubrió nada: le soplaron la respuesta.

---

## 4. Análisis Multivariado

Audiencia: 🔧 🧭

### 4.1 Multicolinealidad y VIF

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Tres comentaristas deportivos que repiten exactamente lo mismo con distintas palabras: en el panel "hay tres voces", pero la información es una. El VIF mide cuánto de cada feature es repetición de las demás.

**🔧 Definición técnica:** `VIF_j = 1 / (1 − R²_j)`, donde R²_j resulta de regresar la feature j contra todas las demás. Lectura: VIF ≈ 1 independiente; VIF > 5 multicolinealidad moderada; VIF > 10 severa. Impacta la interpretabilidad de modelos lineales (coeficientes inestables, [[02-Fundamentos-Matematicos]]). Tratamiento: elegir una representante por cluster de features correlacionadas, combinar (ratios, PCA — [[03-Preparacion-de-Datos]]) o regularizar con Ridge ([[11-Mejora-de-Modelos]]).

**🧭 Cuándo usarlo:** obligatorio antes de interpretar coeficientes; recomendable siempre que haya familias de features derivadas de la misma fuente (monto_total, monto_promedio, monto_máximo…).

**👔 En una frase para el negocio:** evita pagar el costo de mantener diez indicadores que aportan la información de tres — y protege las conclusiones de "qué factor pesa más".

### 4.2 Parallel coordinates

Audiencia: 🔧

> [!tip] 💡 Analogía
> El electrocardiograma de cada fila: cada observación es una línea que atraviesa todos los ejes (features). Cuando las líneas de una clase siguen un ritmo distinto a las de otra, estás **viendo** separabilidad multidimensional a ojo desnudo.

**🔧 Definición técnica:** cada eje vertical es una feature (escalada, [[05-Escalado-de-Datos]]); cada observación, una polilínea coloreada por clase o cluster. Útil para detectar patrones entre clases en alta dimensión y para perfilar clusters ya construidos ([[06-Clustering]]). Con muchas filas: muestrear o usar transparencia.

### 4.3 Análisis de grupos

Audiencia: 🔧 🧭

**🔧 Definición técnica:** si existe una hipótesis de segmentación (por región, por plan, por canal), comparar estadísticos y distribuciones entre los grupos potenciales — puede revelar que "un solo modelo para todos" es la decisión equivocada, o motivar un clustering formal previo ([[06-Clustering]]).

**🧭 Cuándo usarlo:** cuando el negocio ya habla en segmentos; el EDA confirma o refuta que esos segmentos existen en los datos.

---

## 5. Herramientas de EDA

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Cocinar a mano vs el robot de cocina: `pandas + seaborn` es el cuchillo del chef (control absoluto, más lento); los profilers automáticos son el robot que pica todo en 5 minutos (perfecto para empezar, insuficiente para el plato final). Los profesionales usan ambos: robot para la primera pasada, cuchillo para lo fino.

| Herramienta | Descripción | Fortaleza principal | Limitación | 🧭 Momento ideal |
|---|---|---|---|---|
| pandas + matplotlib/seaborn | EDA manual con control total sobre cada análisis y visualización | Máxima flexibilidad y personalización | Requiere código para cada análisis | Análisis dirigido y reproducible; la 2ª pasada del EDA |
| ydata-profiling (ex pandas-profiling) | Reporte HTML automático completo: estadísticos, distribuciones, correlaciones, alertas de calidad | Visión completa en una línea de código | Lento en datasets grandes (> 500K filas → muestrear) | Primera mirada del proyecto, en minutos |
| sweetviz | Reporte HTML visual con **comparación automática train vs test** | Detección de drift/diferencias entre conjuntos | Menos detalle estadístico que ydata-profiling | Justo después del split: ¿train y test se parecen? |
| dtale | Dashboard interactivo en el navegador desde Jupyter: filtros, gráficos, correlaciones en vivo | Exploración ad-hoc sin escribir código | Requiere servidor activo | Sesiones exploratorias con stakeholders al lado |
| missingno | Visualización especializada de nulos: matriz, heatmap, dendrograma | Revela si los nulos están correlacionados (pista MCAR/MAR/MNAR, [[03-Preparacion-de-Datos]]) | Solo análisis de missingness | Al diagnosticar el mecanismo de los nulos |
| plotly / plotly express | Gráficos interactivos: zoom, hover, filtros; integración nativa en Jupyter | Interactividad superior para explorar y presentar | Más lento de renderizar que seaborn | Presentaciones y dashboards exploratorios |
| Lux | Recomendación automática de visualizaciones según el dataframe activo | Sugiere qué explorar a continuación | En estado beta, menos estable | Complemento experimental del flujo en Jupyter |

**👔 En una frase para el negocio:** la primera radiografía completa del dataset cuesta una línea de código y cinco minutos — no aprobarla antes de un proyecto es negligencia barata de evitar.

---

## 6. Checklist de un EDA completo

Audiencia: 🔧 🧭

- [ ] **Inventario:** shape, dtypes, nulos y únicos por columna ([[03-Preparacion-de-Datos]])
- [ ] **Univariado numérico:** distribución, skew/kurtosis, outliers, valores de relleno sospechosos, bimodalidades
- [ ] **Univariado categórico:** frecuencias, cardinalidad, variantes de escritura
- [ ] **Target:** balanceo (clasificación) o forma (regresión); definición estable en el tiempo
- [ ] **Features vs target:** relación de cada feature; mutual information; ninguna "demasiado perfecta"
- [ ] **Bivariado:** matriz de correlación + tests donde importe declarar diferencias
- [ ] **Multivariado:** VIF sobre las familias de features correlacionadas
- [ ] **Nulos:** patrón de missingness con missingno; hipótesis MCAR/MAR/MNAR
- [ ] **Leakage:** auditoría de disponibilidad temporal de cada feature ([[10-Validacion-y-Leakage]])
- [ ] **Documentación:** hallazgos, decisiones y pendientes escritos — el EDA que no se documenta se repite

---

## 📖 Referencias de este tomo

- (Tukey, 1977) — el texto fundacional del análisis exploratorio de datos.
- (Anscombe, 1973) — el cuarteto: por qué los estadísticos no reemplazan a los gráficos.
- (James et al., 2021) y (Géron, 2022) — el EDA como etapa del flujo aplicado de ML.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[03-Preparacion-de-Datos|03 · Preparación de Datos]] · Siguiente: [[05-Escalado-de-Datos|05 · Escalado de Datos ➡]]

> **Próximo tomo:** [[05-Escalado-de-Datos]] — todas las técnicas de escalado y transformación (Min-Max, Standard, Robust, Quantile, Power, log, L2, cíclica) y las reglas críticas que evitan el leakage de escala.

