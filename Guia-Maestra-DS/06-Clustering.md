---
title: "Tomo 06 — Clustering: Aprendizaje No Supervisado"
tags: [data-science, machine-learning, clustering, unsupervised, segmentacion]
audiencias: [tecnico, puente, ejecutivo]
tomo: 06
version: 6.1
updated: 2026-07-29
---

# 🧩 Tomo 06 — Clustering: Aprendizaje No Supervisado

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[05-Escalado-de-Datos|05 · Escalado de Datos]] · Siguiente: [[07-Modelos-Supervisados|07 · Modelos Supervisados ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Cuando no tienes etiquetas (y etiquetar miles de registros es caro o imposible), el clustering permite descubrir grupos naturales: segmentos de clientes, tipos de comportamiento, anomalías. Es la herramienta que te dice "hay 4 tipos de clientes en tu base" **antes de que tú lo sepas**. Resuelve tres problemas prácticos: segmentación, reducción de costos de etiquetado (semi-supervisado, [[01-Introduccion-Ejecutiva]]) y exploración de estructura. No existe un algoritmo universalmente superior: la elección depende de la **geometría** esperada de los clusters, el tamaño del dataset y la tolerancia al ruido.

> [!abstract] 👔 Impacto ejecutivo
> Segmentar bien multiplica la efectividad de todo lo que viene después; segmentar mal industrializa el error.
>
> - **Decisiones que habilita:** campañas y ofertas por segmento real de comportamiento, priorización de inversión por tipo de cliente, detección de patrones operativos desconocidos.
> - **Costo de hacerlo mal:** campañas dirigidas a grupos que solo existen en el PowerPoint, presupuestos repartidos por segmentos artificiales, y "clusters" que son puro efecto de una variable sin escalar ([[05-Escalado-de-Datos]]).
> - **Pregunta ejecutiva que responde:** *¿cuántos tipos de clientes/productos/comportamientos tengo realmente, y en qué se diferencian?*

> [!tip] 💡 Analogía general
> Llegas a organizar los grupos de una fiesta de 300 desconocidos. Nadie trae etiqueta. Puedes: plantar banderas y que cada quien vaya a la más cercana (**K-Means**), buscar los corrillos densos y dejar solos a los solitarios (**DBSCAN**), armar el árbol genealógico de quién se juntó con quién (**jerárquico**), o aceptar que algunos son 70% grupo-del-fútbol y 30% grupo-del-karaoke (**GMM**). Cada método "ve" grupos distintos — por eso elegir el algoritmo es parte del análisis, no un detalle.

> [!example] 📊 Caso de negocio — Retail: de una campaña única a segmentos que responden
> **Problema:** una cadena de retail envía la misma campaña a toda su base. Tasa de respuesta estancada, presupuesto de marketing cuestionado. "Segmentan" por edad y comuna — demografía, no comportamiento.
>
> **Técnica aplicada:** features RFM (recency, frequency, monetary) + comportamiento de categorías, escaladas con RobustScaler ([[05-Escalado-de-Datos]]). K-Means con K elegido por elbow + silhouette; HDBSCAN como contraste para detectar clientes "inclasificables" (ruido) que K-Means forzaba dentro de algún grupo. Perfilamiento de cada cluster con boxplots y parallel coordinates ([[04-EDA]]).
>
> **Resultado:** cinco segmentos accionables (ej.: "compradores intensivos en promoción", "dormidos de alto valor histórico") con mensajes y ofertas diferenciadas; el segmento "dormidos de alto valor" concentra la campaña de reactivación y responde varias veces mejor que el promedio histórico. El grupo "ruido" de HDBSCAN — clientes que no calzan con ningún patrón — se excluye de campañas masivas, ahorrando contactos inútiles.

**La geometría decide el algoritmo:**

```
  Blobs compactos y            Formas arbitrarias,           Densidades distintas,
  tamaños similares            ruido presente                jerarquía de grupos
      ●●●    ▲▲▲                 ●●●●●●                          ●●●●●
     ●●●●●  ▲▲▲▲                ●●    ●●        ▲                ●●●●●      ▲▲
      ●●●    ▲▲▲                 ●●●●●●   ▲                        ●        ▲▲▲▲▲
                                    (luna/anillo)                (denso)   (difuso)
   → K-Means, GMM             → DBSCAN, Spectral              → HDBSCAN, OPTICS
```

---

## 1. Algoritmos Particionales

Audiencia: 🔧 🧭

### K-Means

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Tienes 300 personas en un salón y quieres formar 5 grupos. Pones 5 banderas al azar. Cada persona camina a la bandera más cercana. Mueves cada bandera al centro de su grupo. Las personas se reagrupan. Repites hasta que nadie cambie de grupo. Las banderas son los **centroides**; las personas, tus datos.

**🔧 Definición técnica:** algoritmo de Lloyd: (1) inicializar K centroides; (2) asignar cada punto al centroide más cercano (distancia euclídea); (3) recalcular cada centroide como la media de sus puntos; (4) repetir hasta convergencia (asignaciones estables o cambio < tol). **K-Means++** (Arthur & Vassilvitskii, 2007) mejora la inicialización eligiendo centroides iniciales con probabilidad proporcional a la distancia² al centroide ya elegido más cercano — menos iteraciones y mejor calidad; es el default de sklearn. Hiperparámetros: `n_clusters` (K), `init` ('k-means++' o 'random'), `n_init` (número de inicializaciones, default 10 — se queda con la mejor), `max_iter`, `tol`. **Inertia**: suma de distancias² de cada punto a su centroide; decrece siempre al aumentar K → sirve para el elbow, no como métrica absoluta.

**Supuestos y limitaciones:** clusters esféricos y de tamaño similar; sensible a la escala (**escalar siempre**, [[05-Escalado-de-Datos]]); sensible a outliers (arrastran centroides, [[03-Preparacion-de-Datos]]); falla con formas no convexas (lunas, anillos).

**🧭 Cuándo usarlo:** el primer intento en datos tabulares escalados con clusters "tipo blob"; rápido, escalable, interpretable vía centroides. Si sospechas formas raras o mucho ruido, salta a densidad.

**👔 En una frase para el negocio:** el estándar para "divídeme la cartera en K grupos parecidos por comportamiento" — simple de explicar y de accionar.

### Mini-Batch K-Means

Audiencia: 🔧

**🔧 Definición técnica:** variante que usa mini-batches aleatorios en cada iteración en lugar del dataset completo. Drásticamente más rápido para N > 100K, con una pérdida de calidad leve. Mismo contrato que K-Means (`n_clusters`, escalado obligatorio).

**🧭 Cuándo usarlo:** datasets masivos o pipelines con restricción de tiempo; validar contra K-Means estándar en una muestra.

### K-Medoids (PAM / CLARA)

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> En vez de plantar una bandera en el "centro promedio" (que puede caer en medio de la nada), eliges como representante a **una persona real del grupo** — el delegado más central. Si un millonario excéntrico entra al grupo, el delegado sigue siendo alguien típico; el promedio, en cambio, se habría ido detrás del millonario.

**🔧 Definición técnica:** Partitioning Around Medoids: usa **medoids** (puntos reales que minimizan la distancia total al resto de su cluster) en lugar de centroides; intercambia iterativamente medoids con no-medoids aceptando el cambio si reduce el costo total. Ventajas sobre K-Means: robusto a outliers, funciona con **cualquier** métrica de distancia (no solo euclídea), y el "caso representativo" es un registro real e interpretable. Implementación: `KMedoids` en scikit-learn-extra; **CLARA** aplica PAM sobre muestras para escalar a datasets grandes.

**🧭 Cuándo usarlo:** cuando necesitas mostrar "el cliente típico de cada segmento" como caso real, cuando la distancia natural no es euclídea (Gower para mixtos, coseno para texto), o cuando los outliers contaminan los centroides.

**👔 En una frase para el negocio:** cada segmento queda representado por un caso real que el equipo comercial puede mirar y entender — no por un promedio abstracto que no existe.

---

## 2. Algoritmos Basados en Densidad

Audiencia: 🔧 🧭

### DBSCAN

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Estás en un parque mirando grupos de personas. Un "grupo" es una zona donde hay mucha gente junta: DBSCAN dice "si encuentro al menos 5 personas dentro de 10 metros, esto es un grupo". Las personas solas en un rincón son **ruido** (outliers). Y lo mejor: no necesitas decirle cuántos grupos hay.

**🔧 Definición técnica:** (Ester et al., 1996) define clusters como regiones de alta densidad: un **core point** tiene ≥ `min_samples` puntos dentro de su radio `eps`; los puntos alcanzables desde un core pertenecen al cluster; los no alcanzables quedan como ruido (etiqueta −1). Parámetros: `eps` (crítico y sensible) y `min_samples` (regla práctica: ≈ 2×D, con D = dimensionalidad). Estrategia para eps: **k-distance plot** — ordenar las distancias al k-ésimo vecino y buscar el "codo".

**Ventajas:** formas arbitrarias, outliers gratis, sin K. **Limitaciones:** muy sensible a eps; falla con densidades muy distintas entre clusters; sufre en alta dimensión (las distancias se degradan, [[03-Preparacion-de-Datos]]); O(N²) sin indexación espacial.

**🧭 Cuándo usarlo:** geometrías no convexas, presencia esperada de ruido, K desconocido — con densidad razonablemente uniforme. Escalado obligatorio ([[05-Escalado-de-Datos]]).

**👔 En una frase para el negocio:** encuentra los grupos con la forma que tengan **y además** te entrega la lista de casos que no pertenecen a ninguno — dos productos por el precio de uno.

### HDBSCAN

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Es DBSCAN con un dron: en vez de mirar el parque desde una sola altura (un solo eps), el dron sube lentamente y observa cómo los grupos se forman, fusionan y separan **a cada altitud**, quedándose con los grupos que se mantienen estables en un rango amplio. Ve la estructura completa, no una foto a una sola escala.

**🔧 Definición técnica:** (Campello et al., 2013) extiende DBSCAN construyendo la jerarquía de densidad completa: (1) distancias de alcanzabilidad mutua; (2) Minimum Spanning Tree; (3) dendrograma de clusters; (4) extracción de los clusters **estables** (los que persisten en un rango amplio de densidades) desde el árbol condensado. Parámetro principal: `min_cluster_size` — mucho más intuitivo que eps. Salidas extra: `probabilities_` (confianza de asignación por punto), `outlier_scores_` (grado de anomalía), `condensed_tree_` (visualización de la jerarquía). Disponible en scikit-learn-contrib y, desde sklearn 1.3, como `HDBSCAN` nativo.

**Ventajas sobre DBSCAN:** sin eps que tunear, maneja **densidad variable**, soft clustering vía probabilidades, selección automática del número de clusters, más estable en datos reales.

**🧭 Cuándo usarlo:** el default moderno para clustering exploratorio en datos reales, especialmente combinado con UMAP como reducción previa ([[03-Preparacion-de-Datos]]).

**👔 En una frase para el negocio:** el algoritmo que menos supuestos te obliga a inventar: encuentra los grupos que existen, con la forma y densidad que tengan, y te dice qué tan confiable es cada asignación.

> [!info] 📌 DBSCAN vs HDBSCAN en la práctica
> Para la mayoría de aplicaciones reales, **HDBSCAN es superior**: no requiere tunear eps, maneja mejor la densidad variable y sus resultados son más estables. DBSCAN solo es preferible si la densidad es claramente uniforme y eps es conocido a priori.

### OPTICS

Audiencia: 🔧

> [!tip] 💡 Analogía
> Un excursionista recorre todo el terreno anotando en cada paso "qué tan cuesta arriba fue llegar aquí" (distancia de alcanzabilidad). El gráfico resultante — el **reachability plot** — muestra valles (clusters) y cumbres (separaciones): un perfil de elevación de la densidad de tus datos.

**🔧 Definición técnica:** genera un ordering de los puntos según su distancia de alcanzabilidad, produciendo el reachability plot; los valles son clusters y pueden extraerse a distintos eps **simultáneamente** desde el mismo modelo. Más informativo que DBSCAN para explorar estructura; mismo espíritu jerárquico que HDBSCAN con salida distinta.

**🧭 Cuándo usarlo:** exploración de la estructura de densidad cuando quieres *ver* las escalas antes de comprometerte con parámetros.

---

## 3. Clustering Jerárquico

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El árbol genealógico de tus datos: al inicio cada punto es su propia "familia"; en cada generación se casan las dos familias más cercanas, hasta que todos son una. El **dendrograma** es ese árbol — y cortar el árbol a cierta altura define cuántas familias quedan. La gracia: no eliges K de antemano; eliges dónde cortar **después de ver** el árbol.

**🔧 Definición técnica — Agglomerative (bottom-up):** cada punto parte como cluster propio; en cada paso se fusionan los dos clusters más cercanos según el criterio de **linkage**; el resultado es un dendrograma cuyo eje Y es la distancia de fusión. Para obtener K clusters: cortar a la altura que produce K ramas (buscar el salto grande en Y).

| Linkage | Criterio de distancia entre clusters | Resultado típico | Sensibilidad a outliers | Cuándo conviene |
|---|---|---|---|---|
| Ward | Minimiza el incremento de varianza intra-cluster (≈ minimizar SSE) | Clusters compactos y de tamaño similar; el más parecido a K-Means | Moderada | Default general con distancia euclídea |
| Complete (máximo) | Distancia entre los dos puntos **más lejanos** | Clusters compactos, menos efecto cadena | Alta | Cuando se quieren grupos de diámetro controlado |
| Single (mínimo) | Distancia entre los dos puntos **más cercanos** | Cadenas (chaining effect); útil para formas elongadas | Muy alta: un outlier puede unir clusters | Estructuras filamentosas; usar con cuidado |
| Average (UPGMA) | Promedio de todas las distancias entre pares | Compromiso entre Ward y Complete | Moderada | Alternativa robusta general |
| Centroid | Distancia entre centroides | Puede producir inversiones en el dendrograma | Moderada | Poco usado; interpretación geométrica simple |

**Divisive (top-down):** parte de un único cluster y divide recursivamente (DIANA es el algoritmo clásico); computacionalmente más costoso que agglomerative y mucho menos usado.

**🧭 Cuándo usarlo:** N pequeño-mediano (el dendrograma de 100K puntos es ilegible y O(N²) en memoria), cuando la **jerarquía** en sí es valiosa (taxonomías de productos, especies, documentos) o cuando quieres decidir K mirando el árbol.

**👔 En una frase para el negocio:** entrega no solo los grupos sino el mapa de parentesco entre ellos — qué segmentos son primos hermanos y cuáles familias lejanas.

---

## 4. Modelos Probabilísticos y Otros Algoritmos

Audiencia: 🔧 🧭

### Gaussian Mixture Models (GMM)

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> En la fiesta hay gente que es 70% grupo-del-fútbol y 30% grupo-del-karaoke. K-Means los obligaría a elegir un solo grupo; GMM les permite tener **membresías parciales**. Cada grupo es una "campana" gaussiana, y cada persona recibe su porcentaje de pertenencia a cada campana.

**🔧 Definición técnica:** asume que los datos provienen de una mezcla de K gaussianas. Se ajusta con **EM (Expectation-Maximization)**: E-step calcula la probabilidad posterior de pertenencia de cada punto a cada componente; M-step actualiza parámetros (μ, Σ, peso) de cada componente; itera hasta converger. **Soft assignment**: probabilidades de pertenencia en lugar de asignación dura. Selección de K: **BIC** o **AIC** (elegir el K que minimiza BIC). Tipos de covarianza: `spherical` (esférica, ≈K-Means), `diag` (ejes alineados), `tied` (misma Σ para todos), `full` (Σ completa por componente, máxima flexibilidad y más parámetros).

**🧭 Cuándo usarlo:** clusters elípticos u orientados, fronteras difusas donde la probabilidad de pertenencia importa (scoring de segmento), generación de datos sintéticos. Requiere escalado y es sensible a inicialización (usa varios `n_init`).

**👔 En una frase para el negocio:** en vez de encasillar a cada cliente en un solo segmento, entrega el % de afinidad con cada uno — oro para personalización fina y para clientes "de frontera".

### Spectral Clustering

Audiencia: 🔧

> [!tip] 💡 Analogía
> No agrupes a la gente por dónde está parada, sino por su **red de amistades**: dos personas en extremos opuestos del salón pueden pertenecer al mismo grupo si están densamente conectadas a través de amigos. Spectral construye el grafo de "amistades" (similitudes) y corta el grafo donde las conexiones son débiles.

**🔧 Definición técnica:** construye un grafo de similitud (kernel RBF o k-vecinos), calcula el Laplaciano del grafo, toma los primeros K eigenvectors ([[02-Fundamentos-Matematicos]]) como nueva representación y aplica K-Means en ese espacio espectral. Brilla donde K-Means fracasa: dos lunas, anillos concéntricos, clusters conectados no convexos. Costo: O(N³) en el caso general → no escala a N grande sin aproximaciones.

**🧭 Cuándo usarlo:** N moderado con geometría no convexa clara, o cuando la similitud natural es un grafo (redes, documentos).

### Mean Shift

Audiencia: 🔧

> [!tip] 💡 Analogía
> Cada persona camina hacia donde ve más gente a su alrededor, una y otra vez. Todos los que terminan en la misma cima de "multitud" son un cluster. Nadie decidió cuántas cimas había: las cimas emergen del terreno.

**🔧 Definición técnica:** cada punto se desplaza iterativamente hacia la media de los puntos dentro de su ventana (`bandwidth`) hasta converger a un modo de densidad; los puntos que convergen al mismo modo forman un cluster. Sin K a priori y robusto a outliers, pero muy sensible al bandwidth (el eps de esta familia) y lento: O(N²) por iteración.

**🧭 Cuándo usarlo:** N pequeño-mediano con modos de densidad bien definidos (también popular en visión para tracking).

### Affinity Propagation

Audiencia: 🔧

> [!tip] 💡 Analogía
> Una elección de delegados por cartas: cada compañero envía mensajes a los demás sobre qué tan buen delegado sería cada uno ("responsibility") y qué tan dispuesto está a aceptarlo ("availability"). Tras varias rondas de correspondencia, emergen los delegados (**exemplars**) y sus grupos — sin que nadie fijara cuántos delegados habría.

**🔧 Definición técnica:** intercambio iterativo de mensajes entre pares de puntos para elegir exemplars (representantes reales, como los medoids). No requiere K, pero el hiperparámetro `preference` controla indirectamente cuántos clusters emergen (y es difícil de calibrar). Muy lento para N > 1.000 — O(N²) en memoria y tiempo por iteración.

**🧭 Cuándo usarlo:** N chico, cuando quieres exemplars reales y no tienes idea de K; en la práctica, K-Medoids suele ser la alternativa más controlable.

---

## 5. Métricas de Evaluación de Clustering

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Evaluar clusters sin etiquetas es como evaluar cuadrillas de trabajo sin conocer el organigrama oficial: miras si cada persona está más cerca de su cuadrilla que de la vecina (Silhouette), si las cuadrillas están apretadas y bien separadas (Davies-Bouldin, Calinski-Harabasz), o — si aparece el organigrama (etiquetas reales) — cuánto coincide tu agrupación con él (ARI, NMI).

| Métrica | Fórmula / Definición | Rango | Mejor valor | ¿Requiere etiquetas reales? | Notas |
|---|---|---|---|---|---|
| Inertia (WCSS) | `Σ dist²` de cada punto a su centroide | 0 a ∞ | Menor (relativo) | No | Decrece siempre con K → solo para elbow method |
| Silhouette Score | `(b − a) / max(a, b)`: a = distancia media intra-cluster, b = al cluster vecino más cercano; promedio global | −1 a +1 | +1 | No | El más informativo sin etiquetas; > 0.5 es bueno; caro para N grande |
| Davies-Bouldin | Promedio del peor ratio `(σᵢ + σⱼ) / dist(cᵢ, cⱼ)` por cluster | 0 a ∞ | Menor (0 = ideal) | No | Penaliza clusters dispersos con centroides cercanos |
| Calinski-Harabasz (VRC) | Varianza entre-clusters / intra-clusters, escalada por N y K | 0 a ∞ | Mayor | No | Rápida; favorece clusters compactos y separados |
| Rand Index | Proporción de pares de puntos con asignación concordante | 0 a 1 | 1 | Sí | Optimista con muchos clusters (acierta "por pares fáciles") |
| Adjusted Rand Index (ARI) | Rand corregido por el acuerdo esperado al azar | −1 a +1 | 1 | Sí | 0 = asignación aleatoria; el estándar con etiquetas |
| NMI (Normalized Mutual Info) | Información mutua normalizada entre particiones | 0 a 1 | 1 | Sí | Robusta a permutaciones de labels |
| V-measure | Media armónica de homogeneidad (cada cluster, una sola clase) y completitud (cada clase, un solo cluster) | 0 a 1 | 1 | Sí | Descompone el error en dos partes interpretables |
| Gap Statistic | Inertia observada vs esperada bajo referencia uniforme (sin estructura) | 0 a ∞ | Mayor gap | No | El K óptimo maximiza el gap; costosa de calcular |

> [!warning] ⚠️ Estrategia práctica para elegir K
> (1) **Elbow** con inertia: buscar el codo. (2) **Silhouette** sobre los K candidatos del codo. (3) **Davies-Bouldin** como confirmación. (4) Si existen etiquetas de referencia (aunque parciales), **ARI**. (5) Y siempre: inspección visual (UMAP 2D, [[03-Preparacion-de-Datos]]) + **sentido de negocio** — un K=7 estadísticamente óptimo que marketing no puede accionar vale menos que un K=4 útil.

```
 inertia │●                          silhouette │      ●
         │ ●                                    │    ●   ●
         │  ●                                   │  ●       ●
         │   ●___ codo (K≈4)                    │ ●          ●
         │       ●────●────●                    └─┬──┬──┬──┬──┬── K
         └────┬────┬────┬───── K                  2  3  4  5  6
              2    4    6                         máximo en K=4 → confirma
```

---

## 6. Síntesis: qué algoritmo según el problema

Audiencia: 🧭

| Situación | Algoritmo recomendado |
|---|---|
| Blobs compactos, K aproximado conocido, N grande | K-Means (o Mini-Batch si N > 100K) |
| Outliers que contaminan centroides / distancia no euclídea | K-Medoids |
| Formas arbitrarias + ruido, densidad uniforme | DBSCAN |
| Densidad variable, sin ganas de tunear eps (default moderno) | HDBSCAN |
| Explorar la estructura de densidad a varias escalas | OPTICS |
| La jerarquía importa (taxonomías) o quieres decidir K viendo el árbol | Agglomerative + dendrograma |
| Fronteras difusas, membresías parciales, clusters elípticos | GMM (K por BIC) |
| Formas no convexas con N moderado / datos de grafo | Spectral |
| Modos de densidad sin K, N chico | Mean Shift |
| Exemplars reales sin K, N muy chico | Affinity Propagation |

**Regla transversal:** todos los algoritmos de esta sección que usan distancias exigen **escalado previo** ([[05-Escalado-de-Datos]]) y se benefician de reducción de dimensionalidad en alta dimensión ([[03-Preparacion-de-Datos]]).

---

## 📖 Referencias de este tomo

- (MacQueen, 1967) — formulación original de K-Means.
- (Arthur & Vassilvitskii, 2007) — K-Means++.
- (Ester et al., 1996) — DBSCAN.
- (Campello et al., 2013) — HDBSCAN.
- (Hastie et al., 2009) y (Géron, 2022) — clustering en el contexto del aprendizaje estadístico y su práctica.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[05-Escalado-de-Datos|05 · Escalado de Datos]] · Siguiente: [[07-Modelos-Supervisados|07 · Modelos Supervisados ➡]]

> **Próximo tomo:** [[07-Modelos-Supervisados]] — el arsenal completo de clasificación, regresión y detección de anomalías, modelo por modelo.
