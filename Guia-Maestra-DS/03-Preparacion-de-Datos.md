---
title: "Tomo 03 — Preparación y Calidad de Datos"
tags: [data-science, machine-learning, data-quality, preprocessing, feature-engineering]
audiencias: [tecnico, puente, ejecutivo]
tomo: 03
version: 6.4
updated: 2026-09-06
---

# 🧹 Tomo 03 — Preparación y Calidad de Datos

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[02-Fundamentos-Matematicos|02 · Fundamentos Matemáticos]] · Siguiente: [[04-EDA|04 · EDA ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> El **80% del tiempo** en un proyecto real de ML se dedica a preparar datos. Si tus datos están sucios, sesgados o mal codificados, el mejor modelo del mundo producirá resultados basura: es como construir una casa sobre cimientos de arena — puede verse bien por fuera, pero se derrumba cuando la habitas. Un dataset pequeño pero limpio y representativo supera siempre a uno masivo pero sesgado: el *Literary Digest* predijo mal la elección de 1936 encuestando a 10 millones de personas con sesgo de selección — la advertencia histórica definitiva de que el volumen no compra calidad.

> [!abstract] 👔 Impacto ejecutivo
> La calidad de un modelo tiene un techo infranqueable: la calidad de sus datos. Todo peso invertido en modelado sobre datos sucios es un peso mal invertido.
>
> - **Decisiones que habilita:** confiar en los scores que alimentan campañas y créditos, presupuestar correctamente la fase de datos (la más larga), auditar por qué un modelo "de laboratorio" falla en producción.
> - **Costo de hacerlo mal:** métricas infladas por leakage que se desploman en producción, clientes duplicados que distorsionan todo, sesgos silenciosos que se vuelven decisiones automatizadas injustas.
> - **Pregunta ejecutiva que responde:** *¿por qué el equipo lleva semanas "limpiando datos" en lugar de entregar el modelo — y por qué eso es exactamente lo que debe estar haciendo?*

> [!example] 📊 Caso de negocio — Telco: el modelo perfecto que no funcionaba
> **Problema:** una telco desarrolla un modelo de churn con AUC 0.94 en laboratorio. Desplegado en producción, apenas supera al azar. La auditoría encuentra cuatro vicios de preparación: clientes duplicados (la misma persona con dos RUT formateados distinto, presente en train **y** test), nulos imputados con la media de **todo** el dataset, target encoding calculado sin folds, y una clase minoritaria del 3% ignorada.
>
> **Técnica aplicada:** reconstrucción disciplinada del pipeline: deduplicación semántica antes del split, imputación y encoding ajustados **solo con train** dentro de un Pipeline de sklearn ([[10-Validacion-y-Leakage]]), target encoding con K-fold, y desbalance tratado con class_weight + ajuste de umbral.
>
> **Resultado:** el AUC de laboratorio "cae" a 0.81 — pero ahora es **verdadero**: la métrica offline por fin predice el comportamiento en producción, la campaña de retención rinde, y el equipo aprende la lección ejecutiva del tomo: *vale más un 0.81 honesto que un 0.94 de utilería*.

**El pipeline de preparación — el orden importa:**

```
 Datos crudos
     │
     ▼
 1. INVENTARIO Y LIMPIEZA ── duplicados, tipos, inconsistencias, texto
     │
     ▼
 2. SPLIT TRAIN/TEST ◄────── ¡ANTES de imputar, escalar o balancear!
     │                        (regla anti-leakage nº1)
     ▼
 3. NULOS ─── imputers con fit() SOLO en train
     │
     ▼
 4. OUTLIERS ─ detectar → diagnosticar → tratar (o conservar)
     │
     ▼
 5. ENCODING de variables categóricas
     │
     ▼
 6. FEATURE ENGINEERING (crear) ──► 7. FEATURE SELECTION (podar)
     │
     ▼
 8. ESCALADO ([[05-Escalado-de-Datos]]) y/o REDUCCIÓN DE DIMENSIONALIDAD
     │
     ▼
 Dataset listo para modelar ([[06-Clustering]] · [[07-Modelos-Supervisados]])
```

Los pasos 3–8 deben vivir **dentro** de un Pipeline de sklearn para que el cross-validation los re-ajuste en cada fold sin contaminar ([[10-Validacion-y-Leakage]]).

---

## 1. Limpieza de Datos

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía general: la bodega del restaurante
> Antes de cocinar, el chef revisa la bodega: inventario de lo que hay, botar lo vencido, juntar los frascos repetidos, etiquetar lo que dice "¿azúcar o sal?". Cocinar sin ese orden es envenenar clientes con cara de eficiencia. La limpieza de datos es esa revisión de bodega — poco glamorosa, absolutamente decisiva.

### 1.1 Inventario inicial

Audiencia: 🔧

**🔧 Definición técnica:** antes de tocar nada: `shape`, `dtypes`, `head()`, `describe()`, `info()`, conteo de únicos y de nulos por columna. Entender qué hay, cuánto hay y de qué tipo es.

**🧭 Cuándo usarlo:** primer comando de todo proyecto. Un inventario de 10 minutos evita descubrir en la semana 6 que "monto" venía como string con separador de miles.

**👔 En una frase para el negocio:** es el "corte de inventario" del activo dato — nadie valora una bodega sin contarla primero.

### 1.2 Detección de duplicados

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El mismo cliente con dos fichas porque una vez lo escribieron "Pérez" y otra "Perez". Para ti es obvio que es uno; para el sistema son dos personas — y de pronto tu base "creció" 8% sin crecer, y ese cliente vale doble en cada análisis.

**🔧 Definición técnica:** `df.duplicated()` para duplicados exactos; para duplicados parciales (misma entidad con variaciones de escritura) se requiere normalización previa + reglas de matching (llaves compuestas, fuzzy matching). Decisión: eliminar o fusionar según contexto. Riesgo mayor: la misma entidad repartida entre train y test hace que el modelo "memorice" en lugar de generalizar ([[10-Validacion-y-Leakage]]).

**🧭 Cuándo usarlo:** siempre, y especialmente antes del split train/test. En datos de clientes, productos o pacientes, asumir que hay duplicados hasta demostrar lo contrario.

**👔 En una frase para el negocio:** los duplicados inflan métricas y presupuestos — deduplicar es la diferencia entre "tenemos 1.2M de clientes" y la verdad.

### 1.3 Corrección de tipos de datos

Audiencia: 🔧

> [!tip] 💡 Analogía
> Una fecha guardada como texto es un Ferrari en su caja: existe, pero no corre. Con "2024-01-05" como string no puedes calcular antigüedad, restar fechas ni ordenar cronológicamente de verdad.

**🔧 Definición técnica:** fechas como string → `datetime`; números leídos como `object` (por símbolos, comas o espacios) → `float/int`; categorías con codificación inconsistente ("M", "m", "Male", "male") → unificación; booleanos disfrazados ("Y"/"N", 0/1, "si"/"sí"). Cada tipo incorrecto bloquea las operaciones de su familia.

**🧭 Cuándo usarlo:** inmediatamente después del inventario. Regla práctica: ninguna columna queda como `object` sin una justificación explícita.

**👔 En una frase para el negocio:** datos con el tipo equivocado son activos congelados — están en la bodega pero no se pueden usar.

### 1.4 Inconsistencias semánticas

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Los sensores de humo del dataset: edad = −5, precio = −1000, fecha de término anterior a la de inicio. Cada uno es una alarma de que **el proceso que generó el dato falló** — y donde hay una alarma visible suele haber diez incendios silenciosos.

**🔧 Definición técnica:** valores fuera de rango lógico (edad negativa, porcentajes > 100), contradicciones entre columnas (fecha_fin < fecha_inicio, total ≠ suma de partes), y valores de relleno codificados como −99, 999 o "N/A" que se cuelan como números reales y destruyen medias y modelos. Detectarlos exige reglas de negocio explícitas, no solo estadística.

**🧭 Cuándo usarlo:** siempre que los datos crucen sistemas o involucren digitación humana. Los −99 disfrazados son especialmente peligrosos: sobreviven a `describe()` si no los buscas.

**👔 En una frase para el negocio:** cada inconsistencia es un defecto del proceso operativo aguas arriba — limpiarlas en el dataset es curar el síntoma; reportarlas al dueño del sistema es curar la enfermedad.

### 1.5 Normalización de texto

Audiencia: 🔧

> [!tip] 💡 Analogía
> "Stgo", "santiago", "SANTIAGO." y " Santiago" son la misma ciudad para cualquier persona — y **cuatro ciudades distintas** para la máquina. Sin normalizar, tu top 10 de ciudades tiene a Santiago compitiendo contra sí misma.

**🔧 Definición técnica:** strip de espacios, lowercase, manejo de tildes y caracteres especiales, estandarización de nombres (países, ciudades, categorías) contra tablas maestras. En texto libre: tokenización y limpieza según el uso posterior (features de texto, sección 6).

**🧭 Cuándo usarlo:** toda columna categórica de origen manual o multi-sistema, **antes** de deduplicar y de hacer encoding (si no, One-Hot creará una columna por cada variante de escritura).

**👔 En una frase para el negocio:** sin texto normalizado, los reportes por región/producto/canal mienten por fragmentación — la misma realidad repartida en filas que parecen distintas.

---

## 2. Tratamiento de Valores Nulos

Audiencia: 🔧 🧭 👔

> [!info] 📌 Por qué importa
> El primer paso no es contar cuántos datos faltan, sino entender **POR QUÉ** faltan. El mecanismo de missingness determina qué estrategia es válida y cuál introduce sesgo silencioso (Rubin, 1976; Little & Rubin, 2002).

> [!tip] 💡 Analogía: ¿qué pasó con las piezas del rompecabezas?
> Tienes un rompecabezas de 1.000 piezas y faltan algunas. **MCAR**: un niño botó piezas al azar — molesto, pero la imagen sigue siendo representativa. **MAR**: el gato se comió solo las piezas azules — el patrón depende de algo observable (el color), y puedes compensarlo. **MNAR**: alguien robó exactamente las piezas del tesoro — la ausencia depende del valor mismo, y reconstruir sin modelar el robo es engañarte. Cada escenario exige una estrategia distinta.

| Mecanismo | Descripción | 💡 En el rompecabezas | Implicación | Estrategia recomendada |
|---|---|---|---|---|
| MCAR (Missing Completely At Random) | La ausencia es aleatoria, sin relación con ninguna variable | El niño botó piezas al azar | Análisis con casos completos es válido, solo pierde potencia | Imputación simple o eliminación si < 5% |
| MAR (Missing At Random) | La ausencia depende de otras variables **observadas** (ej.: cierto segmento tiende a no reportar ingresos) | El gato se comió las piezas azules | Usar solo casos completos sesga; imputar condicionando en lo observado es válido | Imputación múltiple, KNN Imputer, IterativeImputer |
| MNAR (Missing Not At Random) | La ausencia depende del **valor faltante mismo** (ej.: ingresos altos ocultan su salario) | Robaron las piezas del tesoro | Sesgo inevitable sin modelar el mecanismo de ausencia | Modelar la missingness explícitamente + indicador binario |

### Técnicas de imputación

Audiencia: 🔧 🧭

| Técnica | Cómo funciona | 💡 Analogía | Cuándo conviene | Limitaciones |
|---|---|---|---|---|
| Media / mediana / moda | Reemplaza con el estadístico de la columna (media: simétricas; mediana: asimétricas; moda: categóricas) | Pintar el muro rayado del color promedio de la casa | Rápido, baseline, MCAR con pocos nulos | Subestima varianza, distorsiona correlaciones |
| Valor constante | Rellena con 0, −1 o categoría 'Unknown' | Etiqueta honesta: "aquí no sabemos" | Cuando el faltante tiene significado propio | El modelo puede leer el constante como valor real |
| Forward / backward fill | Propaga el último (o siguiente) valor conocido | Si el lunes no midieron la temperatura, vale la del domingo | Series de tiempo con continuidad física | Peligroso con gaps largos o series volátiles |
| KNN Imputer | Imputa con la media de los K vecinos más cercanos en el espacio de features | Preguntarle a los 5 vecinos más parecidos qué valor tendrían | Preserva relaciones entre variables; MAR | Costoso O(N²); exige escalado previo ([[05-Escalado-de-Datos]]) |
| IterativeImputer / imputación múltiple (MICE) | Modela cada feature con nulos en función de las demás, en rondas iterativas hasta converger | Detectives que refinan sus deducciones escuchándose en rondas | El estándar para MAR en datasets complejos | Más lento; requiere cuidado para no filtrar el target; por defecto entrega **una sola** imputación (ver abajo) |
| Imputación con modelo ML | Entrena un modelo predictivo por columna con nulos usando las demás como features | Contratar un adivino profesional por cada columna | Máxima precisión cuando la columna es crítica | Costoso; riesgo de sobre-confianza en valores inventados |
| Indicador de missingness | Columna binaria `era_nulo` junto a la imputación | Dejar una nota: "esta pieza faltaba" | Siempre que la ausencia pueda ser informativa (MNAR) | Duplica columnas; interacción con One-Hot |

**🧭 Cuándo usar qué:** primero diagnostica el mecanismo (compara perfiles de filas con y sin nulos; `missingno` ayuda, [[04-EDA]]). MCAR leve → simple; MAR → KNN/MICE; sospecha de MNAR → indicador + imputación + juicio de dominio. En árboles y boosting, recuerda que XGBoost/LightGBM manejan nulos nativamente ([[07-Modelos-Supervisados]]): a veces la mejor imputación es no imputar.

**🔧 Ojo con la etiqueta MICE.** La **imputación múltiple** (van Buuren & Groothuis-Oudshoorn, 2011) genera *m* datasets muestreando de la distribución predictiva y combina los resultados con las reglas de Rubin, para que la incertidumbre de lo imputado se refleje en los intervalos. `IterativeImputer`, en su modo por defecto, entrega **una sola** imputación de medias condicionales: mejor que la media global, pero con la misma subestimación de varianza; su documentación exige `sample_posterior` para usarlo como imputación múltiple. Para un modelo predictivo el criterio cambia (Sperrin et al., 2020): lo que importa no es la insesgadez de un coeficiente sino que el procedimiento de imputación sea **reproducible en producción** (sin el target, que allí no existe) y que el mecanismo de ausencia sea el mismo en desarrollo y despliegue. Bajo ese criterio, el indicador de missingness es legítimo con cualquier mecanismo, no solo MNAR, siempre que la ausencia vaya a seguir ocurriendo igual cuando el modelo prediga.

**👔 En una frase para el negocio:** los datos que faltan también cuentan una historia — quién no responde, qué sensor se apaga, qué campo nadie llena — y esa historia puede ser más predictiva que los datos presentes.

> [!danger] 🚨 Regla crítica: nunca imputar antes de separar train/test
> Los estadísticos de imputación (media, mediana, parámetros del KNN/MICE) deben calcularse **SOLO con datos de entrenamiento** y luego aplicarse a validación y test. Imputar con el dataset completo filtra información del test al train — data leakage puro ([[10-Validacion-y-Leakage]]). La forma segura: el imputer dentro del Pipeline de sklearn.

---

## 3. Detección y Tratamiento de Outliers

Audiencia: 🔧 🧭 👔

> [!info] 📌 Por qué importa
> Un outlier es una observación que se aleja significativamente del patrón general. Puede ser un **error de medición**, un **fraude**, o un **evento genuinamente inusual** — y cada diagnóstico exige un tratamiento opuesto. Borrarlos a ciegas puede eliminar justo la señal más valiosa del dataset.

> [!tip] 💡 Analogía general: la boleta de $10 millones en el kiosco
> En la caja de un kiosco aparece una boleta de $10.000.000. ¿Error de tipeo? ¿Compra mayorista real? ¿Lavado de dinero? El número es el mismo; el **diagnóstico** cambia todo: el error se corrige, el mayorista se conserva (¡es tu mejor cliente!), el fraude se investiga. Los outliers no se "eliminan": se diagnostican.

### 3.1 Métodos de detección

Audiencia: 🔧 🧭

| Método | Cómo funciona | 💡 Analogía | Cuándo conviene | Limitaciones |
|---|---|---|---|---|
| Z-score | `z = (x − μ) / σ`; sospechoso si `│z│ > 3` | Medir cuántas "desviaciones" te alejas del promedio del curso | Datos ~normales, univariado, rápido | Asume normalidad; los propios outliers contaminan μ y σ |
| IQR / Tukey | Outlier si `x < Q1 − 1.5×IQR` o `x > Q3 + 1.5×IQR` (3×IQR para extremos) | Quedar fuera de los "bigotes" del boxplot | Robusto (usa percentiles), default univariado | Umbral 1.5 es convención, no ley; univariado |
| Modified Z-score (MAD) | Usa mediana y desviación absoluta mediana en lugar de μ y σ | El Z-score con chaleco antibalas: los extremos no lo contaminan | Distribuciones asimétricas o contaminadas | Menos conocido; requiere umbral propio (~3.5) |
| DBSCAN / HDBSCAN | Puntos en regiones de baja densidad quedan etiquetados como ruido (−1) | Quien estaciona solo en el rincón vacío del estacionamiento | Outliers **multivariados**, formas arbitrarias, sin supuestos | Sensible a parámetros; costoso en alta dimensión ([[06-Clustering]]) |
| Isolation Forest | Árboles con splits aleatorios: los outliers se aíslan con pocos cortes (camino corto) | La persona fácil de identificar con dos preguntas: "¿vino en helicóptero?" | Alta dimensión, escalable, el workhorse moderno (Liu et al., 2008) | No captura bien outliers "locales"; contamination a estimar |
| Local Outlier Factor (LOF) | Compara la densidad local de un punto con la de sus vecinos; LOF ≫ 1 = raro **para su barrio** | Gastar $50.000/mes está bien en un barrio caro y es rarísimo en otro | Outliers contextuales, densidades heterogéneas (Breunig et al., 2000) | Lento con N grande; para datos nuevos requiere el modo `novelty=True` (scikit-learn ≥ 0.20), que no debe puntuar el propio train |
| Elliptic Envelope | Ajusta una gaussiana multivariada robusta (MCD) y marca lo que cae fuera del elipsoide | Dibujar el óvalo de "lo normal" y mirar quién quedó afuera | Datos ~gaussianos multivariados | Falla feo si los datos no son gaussianos |

Los métodos basados en modelos (Isolation Forest, LOF, One-Class SVM, Autoencoder) se retoman como **detección de anomalías** de pleno derecho en [[07-Modelos-Supervisados#3. Detección de Anomalías|Tomo 07]].

### 3.2 Estrategias de tratamiento

Audiencia: 🔧 🧭

| Estrategia | Cuándo corresponde | Riesgo si se abusa |
|---|---|---|
| Eliminación | Error de medición confirmado, o < 1% de los datos, siempre documentado | Borrar señal real (fraudes, clientes VIP) |
| Winsorización / capping | Reemplazar por el percentil límite (P1/P99): conserva el registro, limita su palanca | Aplana diferencias reales entre extremos |
| Transformación (log, Box-Cox, √) | Distribuciones sesgadas donde los extremos son legítimos ([[05-Escalado-de-Datos]]) | Complica la interpretación en unidades originales |
| Modelo robusto | RobustScaler + Huber regression, árboles, Random Forest: convivir con outliers sin tratarlos | Ocultar problemas de calidad que había que reportar |
| Mantener y modelar | Cuando el outlier **es** la señal: fraude, falla de equipo, cliente excepcional | Ninguno — es la decisión correcta más veces de lo que se cree |

**👔 En una frase para el negocio:** la pregunta nunca es "¿borramos los raros?" sino "¿este raro es un error, un riesgo o una oportunidad?" — cada respuesta vale dinero distinto.

> [!warning] ⚠️ Regla crítica
> Todo tratamiento de outliers se **documenta**: qué método de detección, qué umbral, cuántos registros afectó y por qué se decidió ese tratamiento. Un umbral movido sin registro es un resultado imposible de reproducir ([[13-MLOps-XAI-Etica]]).

---

## 4. Encoding de Variables Categóricas

Audiencia: 🔧 🧭

> [!info] 📌 Por qué importa
> Los algoritmos solo entienden números. La **forma** de traducir categorías a números cambia lo que el modelo puede aprender: una mala elección inventa órdenes que no existen, explota la dimensionalidad o filtra el target ([[10-Validacion-y-Leakage]]).

> [!tip] 💡 Analogía: ¿cómo le explicas colores a un algoritmo?
> Tienes 'Rojo', 'Azul', 'Verde'. Si pones Rojo=1, Azul=2, Verde=3, le estás diciendo al modelo que Verde es "más" que Rojo y que Azul está "entre medio" — un orden que inventaste. Cada método de encoding cambia cómo el dataset **se ve** para el algoritmo: más ancho, más angosto, más informativo o más traicionero.

| Método | Descripción técnica | 📐 Cómo se ve el dataset después | Ventajas | Cuándo usar / Limitaciones |
|---|---|---|---|---|
| Label Encoding | Asigna un entero único a cada categoría (0,1,2,…). `LabelEncoder` | La misma única columna, ahora de enteros: `color = [2, 0, 1, …]` | Compacto, sin expansión dimensional | Variables ordinales o modelos de árbol. **NUNCA** nominales + modelos lineales/distancia (inventa orden falso) |
| One-Hot Encoding | Una columna binaria por categoría. `OneHotEncoder`, `pd.get_dummies` | El dataset se **ensancha**: `color_rojo, color_azul, color_verde` con 0/1 | Sin orden artificial, interpretable | Nominales de cardinalidad baja (<20). Con alta cardinalidad explota la dimensionalidad |
| Ordinal Encoding | Enteros que preservan un orden **explícito** definido por ti. `OrdinalEncoder(categories=…)` | Una columna de enteros con semántica: bajo=0 < medio=1 < alto=2 | Preserva la semántica del orden real | Solo cuando el orden existe de verdad (tallas, niveles educativos) |
| Target Encoding | Reemplaza la categoría por la media del target en esa categoría | Una columna continua cargada de información predictiva: `ciudad → tasa media de churn de esa ciudad` | Maneja alta cardinalidad, muy informativo | ⚠️ Riesgo serio de leakage: calcular **siempre** con K-fold (ver danger abajo) |
| Frequency Encoding | Reemplaza por la frecuencia relativa de la categoría | Una columna continua: `ciudad → 0.23` (proporción de filas) | Sin explosión dimensional, simple | Alta cardinalidad sin relación directa con el target; colisiones si dos categorías comparten frecuencia |
| Binary Encoding | Convierte el índice de la categoría a binario; una columna por bit | log₂(K) columnas de 0/1: 100 categorías → 7 columnas | Mucho más compacto que One-Hot | Alta cardinalidad con modelos lineales; menos interpretable |
| Hashing Encoding | Función de hash asigna cada categoría a uno de K buckets fijos | K columnas fijas decididas de antemano, sin diccionario guardado | Memoria fija, tolera categorías nuevas en producción (streaming) | Colisiones inevitables (categorías distintas al mismo bucket); no invertible |
| Embeddings (DL) | Representación densa aprendida durante el entrenamiento. `nn.Embedding` | K columnas continuas **aprendidas**: cada categoría es un vector que captura semántica | Captura relaciones entre categorías; escala a millones | NLP, recomendación, cardinalidad extrema; requiere deep learning ([[12-Deep-Learning]]) |

**🧭 Guía rápida de decisión:** ¿el orden existe de verdad? → Ordinal. ¿Pocas categorías nominales? → One-Hot. ¿Muchas categorías? → Target (con K-fold), Frequency o Binary. ¿Millones de categorías o streaming? → Hashing o Embeddings. ¿Modelo de árboles? → Label/Ordinal suele bastar (los árboles no leen "orden" como magnitud, solo cortan). ¿CatBoost? → trae manejo nativo de categóricas ([[07-Modelos-Supervisados]]).

**👔 En una frase para el negocio:** la traducción de categorías a números decide cuánta información de "quién es el cliente" llega realmente al modelo — y es uno de los lugares favoritos del leakage para esconderse.

> [!danger] 🚨 Target Encoding sin K-fold = leakage garantizado
> Si la media del target por categoría se calcula con **todas** las filas, cada fila conoce su propio target a través de su categoría — el modelo "adivina" en validación lo que ya vio disfrazado. Forma correcta: out-of-fold target encoding (la media de cada fila se calcula sin usar esa fila ni su fold) + suavizado hacia la media global para categorías con pocas observaciones.

---

## 5. Manejo de Desbalance de Clases

Audiencia: 🔧 🧭 👔

> [!info] 📌 Por qué importa el desbalance
> Si el 99% de tus emails son normales y el 1% es fraude, un modelo que SIEMPRE diga "normal" tendrá 99% de accuracy y **cero utilidad**. En los problemas que más valen (fraude, diagnóstico, churn, fallas), la clase de interés es casi siempre la minoritaria: el desbalance engaña a las métricas simples y produce modelos decorativos ([[08-Metricas-de-Evaluacion]]).

> [!tip] 💡 Analogía general: las agujas en el pajar
> Buscas 30 agujas en un pajar de 3.000 pajas. Si premias al buscador por "objeto revisado correctamente", aprende la estrategia perfecta: decir "paja" siempre — 99% de acierto, ninguna aguja. Las técnicas de desbalance cambian el juego: o agregas agujas de práctica (oversampling), o sacas paja (undersampling), o pagas mucho más por aguja encontrada (cost-sensitive).

### 5.1 Oversampling — agregar minoría

Audiencia: 🔧

| Técnica | Cómo funciona | 💡 Analogía | Cuándo conviene | Limitaciones |
|---|---|---|---|---|
| RandomOverSampler | Replica aleatoriamente muestras minoritarias | Fotocopiar las mismas 30 agujas | Baseline rápido | Overfitting: el modelo memoriza copias idénticas |
| SMOTE | Genera muestras **sintéticas** interpolando entre un punto minoritario y sus K vecinos (Chawla et al., 2002) | Inventar alumnos "intermedios" entre dos compañeros reales | El estándar **histórico**; clases con estructura continua — ver la advertencia de §5.4 | Puede crear puntos en zonas de la mayoría; ruido si hay outliers |
| ADASYN | Como SMOTE pero genera más sintéticos donde el clasificador más se equivoca | Reforzar con más ejercicios justo los temas donde el alumno falla | Fronteras de decisión difíciles | Amplifica ruido si la frontera es ruidosa |
| SMOTENC | Variante de SMOTE para mezcla de features numéricas y categóricas | SMOTE que sabe que "ciudad" no se promedia | Datasets tabulares mixtos (el caso real típico) | Más lento; requiere declarar qué columnas son categóricas |
| Borderline-SMOTE | Solo sintetiza entre muestras minoritarias **en la frontera** de decisión | Practicar solo los casos límite, no los obvios | Cuando la confusión vive en la frontera | Ignora estructura interna de la clase minoritaria |

### 5.2 Undersampling — reducir mayoría

Audiencia: 🔧

| Técnica | Cómo funciona | 💡 Analogía | Cuándo conviene | Limitaciones |
|---|---|---|---|---|
| RandomUnderSampler | Elimina muestras mayoritarias al azar | Sacar paja del pajar a puñados | Datasets enormes donde sobra mayoría | Puede botar información valiosa |
| NearMiss (1/2/3) | Conserva las mayoritarias más cercanas a las minoritarias según tres criterios de proximidad | Quedarse solo con la paja que más se parece a una aguja | Afinar la frontera con control de qué se conserva | Sensible a ruido; tres variantes que elegir |
| TomekLinks | Elimina pares de clases opuestas que son mutuamente vecinos más cercanos | Separar a las parejas confundidas que están abrazadas en la frontera | Limpiar la frontera de decisión | Remueve pocos puntos; no balancea por sí solo |
| ENN (Edited Nearest Neighbours) | Elimina mayoritarias cuyo vecindario KNN vota distinto a su clase | Expulsar al hincha que quedó sentado en la barra del equipo contrario | Purgar ruido cerca de la frontera | Puede adelgazar demasiado zonas mixtas |
| Cluster Centroids | K-Means sobre la mayoría; se conservan los centroides como representantes | Reemplazar a la multitud por sus delegados electos | Compresión informada de la mayoría | Los centroides son puntos sintéticos, no reales |

### 5.3 Técnicas combinadas

Audiencia: 🔧

- **SMOTEENN** — SMOTE + ENN: genera sintéticos y luego **limpia** el ruido que quedó (el combo más agresivo).
- **SMOTETomek** — SMOTE + TomekLinks: genera sintéticos y despeja la frontera (más conservador que SMOTEENN).

**🧭 Cuándo conviene:** cuando SMOTE solo deja la frontera sucia; en la práctica, probar SMOTE → SMOTETomek → SMOTEENN en ese orden y comparar con validación honesta ([[10-Validacion-y-Leakage]]).

### 5.4 Estrategias a nivel de modelo (sin tocar los datos)

Audiencia: 🔧 🧭

| Estrategia | Cómo funciona | Cuándo conviene |
|---|---|---|
| `class_weight='balanced'` | Pondera la función de pérdida inversamente a la frecuencia de cada clase | Primera opción: sin datos sintéticos, disponible en casi todo sklearn |
| Ajuste del umbral de decisión | El default 0.5 no es ley: moverlo prioriza recall o precision según el costo del error | Siempre, como paso final calibrado al negocio ([[11-Mejora-de-Modelos]]) |
| Cost-sensitive learning | Costos asimétricos de FP y FN directamente en la pérdida | Cuando el negocio puede poner precio a cada tipo de error |
| `scale_pos_weight` (XGBoost) | `N_negativos / N_positivos` como peso de la clase positiva | El equivalente boosting de class_weight ([[07-Modelos-Supervisados]]) |

**🧭 Orden práctico de ataque:** (1) métricas correctas primero (AUC-PR, F1, MCC — [[08-Metricas-de-Evaluacion]]); (2) `class_weight` / `scale_pos_weight`; (3) ajuste de umbral; (4) recién entonces resampling (SMOTE y familia). Muchas veces (1)–(3) bastan y evitan inventar datos.

> [!warning] ⚠️ Corregir el desbalance distorsiona las probabilidades
> Oversampling, undersampling y SMOTE le enseñan al modelo una **prevalencia falsa**: el resultado son probabilidades de la clase minoritaria sistemáticamente sobreestimadas (miscalibration) **sin mejora de la discriminación** (AUC), demostrado por simulación con regresión logística (van den Goorbergh et al., 2022) y extendido a modelos de machine learning, donde los modelos sin corrección tuvieron calibración igual o mejor y la recalibración posterior no siempre reparó el daño (Carriero et al., 2025). Con clasificadores fuertes, el balanceo tampoco mejora el desempeño predictivo (Elor & Averbuch-Elor, 2022; preprint). Consecuencia práctica: si el output es un **score de riesgo** o alimenta decisiones por valor esperado (crédito, triage, priorización de campañas), no resamplees: entrena sobre la distribución real, evalúa con AUC-PR y calibración ([[08-Metricas-de-Evaluacion]]) y mueve el umbral ([[11-Mejora-de-Modelos]]). El resampling queda reservado para clasificadores débiles evaluados con métricas de etiqueta — y aun así exige recalibrar después.

**👔 En una frase para el negocio:** cuando el evento que importa es 1 de cada 100, el "99% de acierto" es la métrica del autoengaño — estas técnicas obligan al modelo a mirar las agujas y no el pajar.

> [!danger] 🚨 SMOTE solo en train, nunca en test
> Aplicar SMOTE (o cualquier resampling) al test contamina la evaluación: mides al modelo sobre una realidad inventada con distribución falsa. El resampling va **solo dentro del fold de entrenamiento** en cada iteración del cross-validation — el Pipeline de `imbalanced-learn` lo garantiza. El test se queda desbalanceado, porque así es el mundo donde el modelo va a vivir.

---

## 6. Feature Engineering

Audiencia: 🔧 🧭

> [!info] 📌 Por qué importa
> El modelo no ve la realidad: ve las features. Feature engineering es **traducir conocimiento del negocio a columnas** que el modelo pueda explotar — históricamente, la palanca de mejora más rentable en datos tabulares, por encima de cambiar de algoritmo.

> [!tip] 💡 Analogía general: nadie come harina
> El dato crudo es harina, huevos y mantequilla: nutritivo en potencia, incomible en la práctica. Feature engineering es cocinar: el ratio deuda/ingreso, la antigüedad en meses, el gasto promedio de los últimos 90 días — esos son los platos que el modelo sí puede digerir. La receta la dicta el conocimiento del negocio, no el algoritmo.

| Familia | Qué se construye | Ejemplos concretos |
|---|---|---|
| Variables derivadas | Ratios, diferencias, productos, conteos por entidad | `ingreso/gasto`, `precio_hoy − precio_ayer`, `área × precio_m2`, nº de compras por cliente |
| Extracción de fechas | Descomponer el timestamp en componentes con señal | Año, mes, día, hora, día_semana, semana_del_año, `es_fin_de_semana`, `dias_desde_ultima_compra` — la fecha cruda como número es casi inútil |
| Binning / discretización | Convertir continuas en tramos | `pd.cut` (igual ancho), `pd.qcut` (igual frecuencia), cortes de negocio (tramos etarios, deciles de ingreso). Captura no-linealidades con modelos lineales |
| Agregaciones temporales | Ventanas móviles y rezagos | `rolling_mean_7d`, `rolling_std_30d`, lags (`valor_t−1`, `valor_t−7`), diferencias de 1er y 2° orden — esencial en series de tiempo |
| Interacciones polinómicas | Productos y potencias de features | `PolynomialFeatures(degree=2)`: capta sinergias (edad × ingreso), con riesgo de explosión dimensional ([[07-Modelos-Supervisados]]) |
| Features de texto | Señales simples antes del NLP pesado | Largo del texto, nº de palabras, proporción de mayúsculas, presencia de keywords ("reclamo", "urgente") |
| Features geográficas | Del par lat/lon a señal útil | Distancia a puntos de referencia, codificación cíclica, clustering geográfico como categoría ([[06-Clustering]]) |

**🧭 Cuándo usarlo:** después de limpiar y entender los datos ([[04-EDA]]), antes de seleccionar. Regla de oro anti-leakage: toda feature debe poder calcularse **con la información disponible en el momento de la predicción** — un rolling que mira hacia adelante es leakage temporal ([[10-Validacion-y-Leakage]]).

**👔 En una frase para el negocio:** aquí es donde el conocimiento de tu gente se convierte en ventaja competitiva del modelo — el algoritmo es commodity, tus features no.

---

## 7. Feature Selection

Audiencia: 🔧 🧭 👔

> [!info] 📌 Por qué importa
> Más columnas **no** es mejor: features irrelevantes agregan ruido, alargan el entrenamiento, facilitan el overfitting y dificultan la interpretación. Seleccionar las features correctas es tan importante como elegir el modelo correcto.

> [!tip] 💡 Analogía: el chef que revisa la mesa
> Tu dataset es una mesa llena de ingredientes. El chef (feature selection) los revisa y dictamina: "esto no aporta sabor" (Variance Threshold), "esto combina con el plato principal" (correlación con el target), "voy a probar quitando ingredientes uno a uno para ver cuál hace falta de verdad" (RFE). La receta final usa menos ingredientes — y el plato sale igual de bueno o mejor.

| Método | Cómo funciona | Pros | Contras | Cuándo conviene |
|---|---|---|---|---|
| Variance Threshold | Elimina features con varianza < umbral. No mira el target | Muy rápido, sin modelo | Solo detecta constantes o casi constantes | Primer descarte barato en datasets anchos, antes de mirar el target |
| Correlación con target | Pearson/Spearman de cada feature vs target | Rápido e interpretable | Solo relaciones lineales/monotónicas ([[02-Fundamentos-Matematicos]]) | Screening univariado rápido cuando basta detectar relación lineal/monotónica simple |
| Chi-cuadrado (χ²) | Test de independencia feature categórica vs target categórico | Sin supuesto de linealidad | Solo features no negativas | Screening univariado rápido antes del modelo, con features categóricas |
| Mutual Information | Información compartida entre feature y target | Captura dependencias **no lineales** | Más lento que correlación | Screening univariado rápido antes del modelo, cuando la relación puede ser no lineal |
| RFE (Recursive Feature Elimination) | Entrena el modelo, elimina la feature menos importante, repite hasta quedar con K | Usa el modelo real para decidir | Costoso, puede ser inestable | Cuando el modelo final es lineal/árbol y n_features es moderado |
| RFECV | RFE + cross-validation para elegir K automáticamente | K óptimo sin adivinar | Muy costoso | Como RFE, pero cuando el presupuesto de cómputo permite que K se elija solo |
| Lasso (L1) | La regularización lleva coeficientes exactamente a 0 | Selección y regularización juntas ([[11-Mejora-de-Modelos]]) | Solo relaciones lineales; α controla la agresividad | Cuando quieres selección y modelo en un solo paso |
| Importancia RF / XGBoost | Reducción promedio de impureza (MDI) o ganancia por feature | Captura no linealidades, rápido | MDI sesgado hacia alta cardinalidad y continuas | Screening rápido con no linealidades, como filtro previo antes de afinar con permutation importance |
| Permutation Importance | Mide la caída de la métrica al permutar aleatoriamente cada feature en validación | Model-agnostic, menos sesgada | Lento; con features correlacionadas fuerza al modelo a extrapolar y puede **inflar** su importancia (Hooker et al., 2021; Strobl et al., 2008) | Cuando importa la relevancia REAL en validación, no in-sample |
| SHAP-based | Importancia global a partir de valores SHAP ([[13-MLOps-XAI-Etica]]) | Muy precisa, model-agnostic, con dirección del efecto | Costosa de calcular en modelos grandes | Cuando además de la relevancia necesitas explicar la dirección del efecto ante negocio/reguladores |
| Boruta | Wrapper sobre Random Forest: compara cada feature con copias permutadas de sí misma (*shadow features*) en rondas y conserva solo las que superan consistentemente a la mejor sombra (Kursa & Rudnicki, 2010) | Selección *all-relevant* con umbral estadístico, no top-K arbitrario | Costoso; hereda los sesgos de la importancia de RF | Tabular mediano donde no quieres perder ninguna feature relevante |

**🧭 Estrategia práctica:** filtra lo obvio primero (varianza ~0, duplicadas, >95% nulos), corre un método rápido (mutual information o importancia de un RF baseline), y refina con permutation importance sobre validación. RFECV solo si el presupuesto de cómputo lo permite. Y todo **dentro** del pipeline: seleccionar features mirando el dataset completo es otra puerta de leakage ([[10-Validacion-y-Leakage]]).

Con features correlacionadas — el caso real típico — la permutation importance no solo «reparte el crédito»: al permutar una variable manteniendo fija su compañera correlacionada, el modelo se evalúa sobre combinaciones que jamás ocurren en los datos, y esa **extrapolación** puede sobreestimar la importancia de las correlacionadas (Hooker et al., 2021); en Random Forest el sesgo hacia predictores correlacionados está documentado y su corrección es la *conditional permutation importance* (Strobl et al., 2008). Remedios prácticos: agrupar features correlacionadas (clustering jerárquico por Spearman y permutar el grupo completo, o conservar un representante por grupo), usar importancia condicional, o medir la caída de la métrica reentrenando sin la feature (*drop-column*). Boruta aporta una alternativa *all-relevant* con criterio estadístico explícito (Kursa & Rudnicki, 2010): lo que importa deja de ser un ranking a ojo y pasa a ser una decisión contrastada contra el azar.

**👔 En una frase para el negocio:** menos features bien elegidas = modelo más barato de mantener, más rápido, más explicable ante reguladores y menos propenso a romperse cuando un sistema fuente cambia.

---

## 8. Reducción de Dimensionalidad

Audiencia: 🔧 🧭

> [!info] 📌 Por qué importa: la maldición de la dimensionalidad
> Con muchas features, las distancias entre puntos se vuelven indistinguibles, los modelos exigen exponencialmente más datos para generalizar y el overfitting se dispara. Reducir dimensiones comprime la información esencial y descarta el ruido — la diferencia con feature selection: selection **elige** columnas originales; reducción **construye** columnas nuevas que resumen a las originales.

> [!tip] 💡 Analogía general: la maleta de viaje
> No llevas la casa al viaje: eliges lo esencial y lo doblas compacto. **Feature selection** elige qué prendas llevar (columnas originales); **reducción de dimensionalidad** manda todo a la máquina de sellado al vacío (combinaciones nuevas, más compactas, menos legibles). Ambas caben en la misma maleta chica; la diferencia es si al llegar reconoces tus prendas.

### PCA (Principal Component Analysis)

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un colegio con 30 notas por alumno las resume en 2–3 "promedios inteligentes": uno que captura lo académico general, otro lo deportivo-artístico. Nadie pierde su historia completa, pero ahora comparar alumnos es tratable. PCA fabrica esos promedios: combinaciones de las columnas originales ordenadas por cuánta variación explican.

**🔧 Definición técnica:** transformación lineal ortogonal que proyecta los datos sobre las direcciones de máxima varianza (componentes principales), obtenidas de la SVD/eigendecomposición de la matriz de covarianza ([[02-Fundamentos-Matematicos]]). Parámetro clave: `n_components` como entero o como varianza objetivo (`0.95` retiene las componentes que explican el 95%). No supervisado; las componentes son ortogonales y pierden el nombre de negocio de las columnas originales.

**🧭 Cuándo usarlo:** compresión antes de modelos sensibles a dimensionalidad (KNN, SVM), decorrelación de features, visualización inicial, reducción de ruido. **Siempre después de escalar** ([[05-Escalado-de-Datos]]): sin escalar, la feature de mayor rango secuestra las componentes.

**👔 En una frase para el negocio:** condensa cientos de métricas en unos pocos índices que retienen casi toda la variación — menos costo, menos ruido, a cambio de perder el nombre "humano" de cada eje.

> [!warning] ⚠️ PCA se ajusta después de escalar y solo con train
> `fit()` del scaler y del PCA con datos de entrenamiento; `transform()` sobre validación/test. PCA fiteado con el dataset completo es leakage ([[10-Validacion-y-Leakage]]).

### Kernel PCA · LDA · NMF

Audiencia: 🔧

- **Kernel PCA** — PCA + kernel trick (RBF, polinomial, sigmoide) para relaciones **no lineales**: proyecta implícitamente a un espacio de mayor dimensión donde lo enredado se estira, y ahí aplica PCA. 💡 *Desenredar el ovillo antes de doblarlo.* Cuándo: estructura no lineal clara que el PCA lineal no separa; más costoso y con hiperparámetros de kernel que tunear.
- **LDA (Linear Discriminant Analysis)** — reducción **supervisada**: busca las direcciones que maximizan la separación entre clases y minimizan la varianza intra-clase; produce a lo más C−1 componentes (C = nº de clases). Supone normalidad y homocedasticidad. 💡 *Acomodar a los invitados de la foto para que las familias queden claramente separadas.* Cuándo: preprocesamiento para clasificación con clases razonablemente gaussianas.
- **NMF (Non-negative Matrix Factorization)** — factoriza en dos matrices **no negativas**: produce "partes" aditivas interpretables. 💡 *Toda receta como suma de ingredientes — nunca "restar harina".* Cuándo: topic modeling de textos, imágenes, matrices de conteos donde la interpretabilidad por partes importa.

### t-SNE

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El fotógrafo que acomoda a un grupo enorme para la foto: se asegura de que los amigos queden juntos (estructura **local** impecable), pero la distancia entre grupos en la foto no es fiel al espacio real — dos familias pueden salir vecinas por conveniencia del encuadre. Preciosa para mirar; no midas distancias sobre ella.

**🔧 Definición técnica:** t-distributed Stochastic Neighbor Embedding (van der Maaten & Hinton, 2008): preserva vecindades locales llevando similitudes a probabilidades y minimizando la divergencia KL entre el espacio original y el embedding 2D/3D. No paramétrico y no determinista: sin `transform()` confiable para datos nuevos. Parámetro clave: `perplexity` (5–50, número efectivo de vecinos); distintas corridas dan distintas fotos.

**🧭 Cuándo usarlo:** **solo visualización** — explorar clusters, validar separabilidad de clases, comunicar estructura. Nunca como preprocessing de un modelo.

**👔 En una frase para el negocio:** es el mapa bonito para *ver* si existen grupos — no una base para medir ni para producción.

### UMAP

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El cartógrafo profesional: también dibuja mapas 2D del territorio de alta dimensión, pero cuida la geografía local **y** la global, dibuja más rápido, y deja el método documentado para mapear territorios nuevos con la misma proyección (`transform()`). Por eso su mapa sí sirve para navegar, no solo para enmarcar.

**🔧 Definición técnica:** Uniform Manifold Approximation and Projection (McInnes et al., 2018): basado en geometría Riemanniana y topología algebraica; construye un grafo de vecindades y optimiza un layout de baja dimensión que preserva estructura local y global. Parámetros: `n_neighbors` (tamaño de vecindad: chico = detalle local, grande = estructura global), `min_dist` (compacidad del embedding), `metric`. Tiene `transform()` reproducible → **sí sirve como preprocessing**.

**🧭 Cuándo usarlo:** visualización (mejor equilibrio local/global que t-SNE) y reducción previa a clustering o clasificación — el combo UMAP → HDBSCAN es un estándar moderno ([[06-Clustering]]).

**👔 En una frase para el negocio:** el mapa que además de bonito es **reutilizable**: los datos de mañana se proyectan al mismo mapa de hoy, lo que lo hace apto para sistemas reales.

### Autoencoders

Audiencia: 🔧

> [!tip] 💡 Analogía
> Aprender a resumir apuntes: comprimes el capítulo a una ficha (encoder) y luego intentas re-explicar todo el capítulo desde la ficha (decoder). Si logras reconstruirlo bien, la ficha contiene la esencia. La ficha — el cuello de botella — es tu representación reducida.

**🔧 Definición técnica:** red neuronal encoder–decoder entrenada para reconstruir su entrada; el cuello de botella (latent space) fuerza la compresión. Captura relaciones no lineales complejas; variante VAE para un espacio latente continuo y generativo ([[12-Deep-Learning]]). El error de reconstrucción sirve además como score de anomalía ([[07-Modelos-Supervisados]]).

**🧭 Cuándo usarlo:** alta dimensión no lineal (imágenes, señales, logs), cuando hay suficientes datos y presupuesto de entrenamiento; para tabular mediano, PCA/UMAP suelen bastar con una fracción del costo.

**👔 En una frase para el negocio:** compresión "a medida" aprendida de tus propios datos — máxima potencia, al precio de entrenar y mantener una red.

### Comparación directa: t-SNE vs UMAP

Audiencia: 🔧 🧭

| Aspecto | t-SNE | UMAP |
|---|---|---|
| Preserva | Local; global solo con inicialización informativa (PCA, el default de scikit-learn desde 1.2) | Local; global **depende de la inicialización** (spectral por defecto) — no es una ventaja intrínseca (Kobak & Linderman, 2021) |
| Velocidad | Lenta en N grande | Mucho más rápido |
| Determinismo | Estocástico; reproducible fijando la semilla | Estocástico; reproducible fijando la semilla |
| `transform()` para datos nuevos | No confiable | Sí — apto para producción |
| Uso legítimo | Visualización final | Visualización **y** preprocessing |
| Parámetros clave | `perplexity` | `n_neighbors`, `min_dist` |

> [!warning] ⚠️ Regla crítica: la «ventaja global» de UMAP era la inicialización
> La creencia de que UMAP preserva la estructura global y t-SNE no nace de comparar UMAP con inicialización spectral contra t-SNE con inicialización aleatoria: con inicialización informativa (PCA o spectral) t-SNE conserva la estructura global tan bien como UMAP, y con inicialización aleatoria UMAP la pierde igual (Kobak & Linderman, 2021). El protocolo moderno de t-SNE — PCA init, learning rate alto, exaggeration para N grande — está en Kobak & Berens (2019) y scikit-learn lo trae por defecto desde la versión 1.2. En rigor, ninguno de los dos garantiza local y global a la vez; PaCMAP fue diseñado para ese equilibrio (Wang, Huang, Rudin & Shaposhnik, 2021). Lo que sí sobrevive: en cualquiera de los tres mapas, **las distancias entre clusters y sus tamaños no se interpretan**. Para preprocessing antes de un modelo, UMAP conserva la ventaja práctica de `transform()` — un embedding t-SNE alimentando un modelo en producción sigue siendo un bug conceptual —, pero valida siempre que el embedding aporte frente al modelo entrenado sin reducción. *(Corregido el 2026-09-06: la tabla y esta regla daban por intrínseca la preservación global de UMAP y por no determinista solo a t-SNE.)*

---

## 9. Síntesis del tomo — las reglas de oro

Audiencia: 🔧 🧭 👔

1. **Split primero, todo lo demás después:** imputación, escalado, encoding con target, resampling y selección se ajustan **solo con train** — el Pipeline de sklearn es el cinturón de seguridad ([[10-Validacion-y-Leakage]]).
2. **Diagnóstico antes que tratamiento:** el mecanismo de los nulos (MCAR/MAR/MNAR) y la naturaleza del outlier (error/señal) deciden la técnica, no al revés.
3. **El desbalance se ataca primero con métricas y pesos**, después con datos sintéticos — y SMOTE jamás toca el test.
4. **Feature engineering con reloj en mano:** toda feature debe existir en el momento de la predicción; lo demás es adivinar el pasado.
5. **Menos es más:** selection y reducción de dimensionalidad no son opcionales en alta dimensión — son la vacuna contra la maldición de la dimensionalidad.

---

## 📖 Referencias de este tomo

- (Rubin, 1976) y (Little & Rubin, 2002) — mecanismos de missing data (MCAR/MAR/MNAR).
- (Chawla et al., 2002) — SMOTE.
- (Liu et al., 2008) — Isolation Forest.
- (Breunig et al., 2000) — Local Outlier Factor.
- (van der Maaten & Hinton, 2008) — t-SNE.
- (McInnes et al., 2018) — UMAP.
- (van den Goorbergh et al., 2022), (Carriero et al., 2025) — corregir el desbalance daña la calibración sin mejorar la discriminación. · (Elor & Averbuch-Elor, 2022) — **preprint**: el balanceo no mejora a los clasificadores fuertes.
- (Kobak & Linderman, 2021), (Kobak & Berens, 2019) — la inicialización decide la estructura global en t-SNE y UMAP. · (Wang, Huang, Rudin & Shaposhnik, 2021) — t-SNE, UMAP, TriMap y PaCMAP comparados.
- (Hooker et al., 2021), (Strobl et al., 2008) — permutation importance con features correlacionadas. · (Kursa & Rudnicki, 2010) — Boruta.
- (van Buuren & Groothuis-Oudshoorn, 2011) — MICE. · (Sperrin et al., 2020) — missing data para predicción vs. inferencia.
- Documentación oficial de scikit-learn (`IterativeImputer` con `sample_posterior`, `LocalOutlierFactor` con `novelty=True`, `TSNE` con `init='pca'` por defecto) → [[16-Bibliografia]] §13. *(Corregido el 2026-09-06: la fila LOF decía «sin predict() para datos nuevos».)*
- (Kuhn & Johnson, 2019) — feature engineering y selection aplicados.
- (Géron, 2022) — pipelines de preparación end-to-end en scikit-learn.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[02-Fundamentos-Matematicos|02 · Fundamentos Matemáticos]] · Siguiente: [[04-EDA|04 · EDA ➡]]

> **Próximo tomo:** [[04-EDA]] — el análisis exploratorio: univariado, bivariado, análisis del target, multivariado (VIF), y las herramientas que automatizan la primera mirada (ydata-profiling, sweetviz, missingno y compañía).



