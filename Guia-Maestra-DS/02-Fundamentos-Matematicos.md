---
title: "Tomo 02 — Fundamentos Matemáticos y Estadísticos"
tags: [data-science, machine-learning, matematicas, estadistica, fundamentos]
audiencias: [tecnico, puente, ejecutivo]
tomo: 02
version: 6.0
---

# 📐 Tomo 02 — Fundamentos Matemáticos y Estadísticos

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[01-Introduccion-Ejecutiva|01 · Introducción Ejecutiva]] · Siguiente: [[03-Preparacion-de-Datos|03 · Preparación de Datos ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Sin estos fundamentos, un Data Scientist opera a ciegas: no sabe por qué un modelo converge o diverge, no puede diagnosticar errores de optimización, y no puede validar si sus resultados son estadísticamente reales o simples coincidencias. Es la diferencia entre **usar** una herramienta y **entender cómo funciona**. Las matemáticas no son un accesorio teórico: son la infraestructura que permite a los modelos aprender, generalizar y decidir bajo incertidumbre. La definición de (Mitchell, 1997) — un programa aprende si su desempeño en la tarea **T**, medido por **P**, mejora con la experiencia **E** — exige formalizar matemáticamente esos tres elementos, y este tomo entrega el lenguaje para hacerlo.

> [!abstract] 👔 Impacto ejecutivo
> Este tomo es el "control de calidad" invisible de todo lo demás: aquí se decide si un hallazgo es señal o ruido, y si un entrenamiento converge o quema presupuesto de cómputo sin resultado.
>
> - **Decisiones que habilita:** distinguir resultados reales de coincidencias antes de escalar una iniciativa, auditar A/B tests y pilotos, dimensionar por qué un modelo cuesta lo que cuesta entrenar.
> - **Costo de hacerlo mal:** tomar decisiones de inversión sobre "mejoras" que eran azar estadístico, features redundantes que vuelven los modelos inestables, y proyectos de optimización que nunca convergen.
> - **Pregunta ejecutiva que responde:** *¿este número que me están mostrando es una verdad del negocio o una casualidad de esta muestra?*

**El mapa del tomo — cuatro pilares y para qué sirve cada uno:**

```
  ÁLGEBRA LINEAL         CÁLCULO                 PROBABILIDAD            INFERENCIA
  cómo se representan    cómo aprenden los       cómo se razona bajo     cómo se valida que un
  y transforman los      modelos (optimizar      incertidumbre           hallazgo sea real y
  datos                  el error)                                       no azar
        │                      │                       │                       │
        ▼                      ▼                       ▼                       ▼
  dataset = matriz       gradiente, backprop,    scores = probabilidades  IC, p-valor, potencia,
  SVD/PCA, embeddings,   descenso de gradiente,  Bayes, distribuciones,   t-test/ANOVA/chi²,
  normas L1/L2           convexidad              esperanza y varianza     Bonferroni/FDR
        │                      │                       │                       │
        └─── [[03-Preparacion-de-Datos]] ── [[07-Modelos-Supervisados]] ── [[10-Validacion-y-Leakage]] ───┘
                          (dónde estos fundamentos se vuelven práctica)
```

---

## 1. Álgebra Lineal

Audiencia: 🔧 🧭

> [!info] 📌 Rol en el ciclo de vida
> El álgebra lineal es el **lenguaje nativo de los datos**: cada dataset es una matriz, cada muestra un vector, cada imagen un tensor. Dominarla permite manipular millones de registros con operaciones vectorizadas eficientes, entender qué hace PCA por dentro, y diagnosticar por qué un modelo lineal se vuelve inestable (Goodfellow et al., 2016).

> [!tip] 💡 Analogía general: el edificio de departamentos
> Imagina que un dataset es un edificio. Cada fila (muestra) es un departamento, cada columna (feature) es una característica del departamento (metros cuadrados, piso, número de baños). La matriz completa es el edificio entero. El álgebra lineal te da las herramientas para recorrer todos los departamentos **simultáneamente** — compararlos, agruparlos, transformarlos — sin ir puerta por puerta.

### 1.1 Escalares, vectores, matrices y tensores

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Es la jerarquía de un archivo Excel: un **escalar** es una celda; un **vector** es una fila o columna completa; una **matriz** es la hoja entera; un **tensor** es el libro con muchas hojas. Y un video es un tensor de orden 4: una pila de imágenes (alto × ancho × color) que avanza en el tiempo.

**🔧 Definición técnica:** las cuatro estructuras fundamentales de datos numéricos. Un escalar es un número (orden 0); un vector, una secuencia ordenada (orden 1); una matriz, una tabla N×D (orden 2); un tensor de orden n generaliza a n dimensiones. Frameworks como NumPy, PyTorch y TensorFlow operan sobre tensores de forma nativa, con operaciones vectorizadas que corren en C/GPU en lugar de loops de Python.

**🧭 Cuándo usarlo:** siempre — es la representación de todo: un dataframe tabular es una matriz N×D; un batch de 32 imágenes RGB de 224×224 es un tensor (32, 224, 224, 3); un embedding de cliente es un vector de 128 dimensiones. Pensar en "formas" (shapes) de tensores es la habilidad diaria número uno al depurar pipelines y redes.

**👔 En una frase para el negocio:** todos tus datos — tablas, imágenes, texto, audio — terminan convertidos en estas estructuras; por eso el mismo equipo y las mismas herramientas pueden atacar problemas tan distintos.

### 1.2 Operaciones matriciales

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El producto matricial es como pagar la nómina con **una transferencia masiva** en lugar de mil transferencias individuales: la misma operación aplicada a todos a la vez. Y la transpuesta es girar la hoja de cálculo: lo que era fila pasa a ser columna.

**🔧 Definición técnica:** suma, producto, transpuesta e inversa. El producto AB exige compatibilidad de dimensiones: columnas de A = filas de B; (N×D)·(D×K) → (N×K). La inversa A⁻¹ solo existe si `det(A) ≠ 0`. Una capa densa de red neuronal es literalmente `salida = f(WX + b)`: un producto matricial más una activación ([[12-Deep-Learning]]).

**🧭 Cuándo usarlo:** cada vez que el código evita un loop: la predicción de un modelo lineal sobre un millón de filas es un solo producto `Xβ`. Si tu pipeline itera fila por fila, casi siempre hay una formulación matricial órdenes de magnitud más rápida.

**👔 En una frase para el negocio:** es la razón de que "scorear" toda tu cartera de clientes tome segundos y no días: las operaciones se aplican en bloque, no caso a caso.

### 1.3 Sistemas de ecuaciones lineales: Ax = b

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Conoces el total de tres boletas del mismo food truck (b) y los precios del menú (A); quieres deducir cuántos completos y cuántas bebidas llevó cada boleta (x). Resolver Ax = b es exactamente ese juego de deducción — y a veces no hay solución exacta (boletas con propina incluida) y hay que buscar la **mejor aproximación**: eso es mínimos cuadrados.

**🔧 Definición técnica:** encontrar x tal que Ax = b. Métodos directos: eliminación gaussiana, factorización LU; iterativos: Jacobi, Gauss-Seidel (útiles con matrices enormes y sparse). Cuando no existe solución exacta (más ecuaciones que incógnitas), se minimiza `‖Ax − b‖²`: es el fundamento de la regresión lineal OLS, cuya solución cerrada es `β = (XᵀX)⁻¹Xᵀy` ([[07-Modelos-Supervisados]]).

**🧭 Cuándo usarlo:** detrás de cada regresión lineal, calibración y ajuste de curvas. En la práctica no invocas Gauss a mano — `np.linalg.solve` o `lstsq` — pero entenderlo explica por qué la multicolinealidad (ver 1.10) rompe la solución.

**👔 En una frase para el negocio:** es la maquinaria que responde "¿cuánto aporta cada factor al resultado?" cuando ajustamos un modelo explicativo de ventas, costos o riesgo.

### 1.4 SVD — Descomposición en Valores Singulares

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Toda canción grabada puede separarse en pistas — voz, batería, bajo, coros — **ordenadas por volumen**. La SVD hace eso con cualquier matriz de datos: la separa en "pistas" (componentes) ordenadas por cuánta información aportan. Si te quedas solo con las tres pistas principales, la canción sigue siendo perfectamente reconocible: eso es comprimir sin perder la esencia.

**🔧 Definición técnica:** toda matriz A se factoriza como `A = UΣVᵀ`, donde U y V son ortogonales y Σ es diagonal con los valores singulares ordenados de mayor a menor. Truncar a los k mayores valores da la mejor aproximación de rango k (teorema de Eckart-Young). Es el fundamento matemático de PCA ([[03-Preparacion-de-Datos]]), de los sistemas de recomendación por factorización y de la compresión de datos; numéricamente más estable que trabajar con la matriz de covarianza.

**🧭 Cuándo usarlo:** reducción de dimensionalidad, compresión, deduplicación de señal en matrices grandes y ralas (texto TF-IDF, ratings usuario×producto). Si tienes 500 features y sospechas que "la información real" cabe en 30 direcciones, la SVD lo confirma mirando cuánta varianza concentran los primeros valores singulares.

**👔 En una frase para el negocio:** permite quedarse con el 95% de la señal usando una fracción de los datos — menos almacenamiento, entrenamientos más rápidos y modelos más estables, sin sacrificar precisión relevante.

### 1.5 Eigendecomposición (valores y vectores propios)

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Estira una masa de pizza con ambas manos: hay direcciones en las que la masa **solo se alarga, sin torcerse**. Esas direcciones privilegiadas son los eigenvectors de la transformación, y cuánto se alarga en cada una es su eigenvalue. Toda transformación lineal, por complicada que parezca, tiene sus "direcciones naturales de estiramiento".

**🔧 Definición técnica:** v es eigenvector de A con eigenvalue λ si `Av = λv`: la transformación no cambia la dirección de v, solo la escala por λ. Para matrices simétricas (como la de covarianza), los eigenvectors son ortogonales y los eigenvalues reales: PCA busca precisamente los eigenvectors de la covarianza — las direcciones de máxima varianza — y usa los eigenvalues para medir cuánta varianza explica cada una.

**🧭 Cuándo usarlo:** PCA y sus parientes, análisis de estabilidad, PageRank (el ranking original de Google es un eigenvector), clustering espectral ([[06-Clustering]]). También para diagnosticar covarianzas: eigenvalues casi cero delatan features redundantes.

**👔 En una frase para el negocio:** encuentra los "ejes maestros" que explican la mayor parte de la variación entre tus clientes o productos — la base de segmentaciones y visualizaciones que sí reflejan la estructura real.

### 1.6 Factorización QR

Audiencia: 🔧

> [!tip] 💡 Analogía
> El *mise en place* de un chef: antes de cocinar, separa los ingredientes en bandejas independientes que no se mezclan (Q, columnas ortogonales) y deja la receta escrita en pasos escalonados (R, triangular) que se ejecutan de atrás hacia adelante. Cocinar así es más limpio y a prueba de errores que improvisar con todo revuelto.

**🔧 Definición técnica:** `A = QR` con Q ortogonal (QᵀQ = I) y R triangular superior. Se construye con Gram-Schmidt o reflexiones de Householder. Es el método numéricamente estable para resolver mínimos cuadrados: en lugar de invertir XᵀX (que amplifica errores de redondeo cuando hay features correlacionadas), se resuelve `Rβ = Qᵀy` por sustitución hacia atrás. Es lo que `lstsq` usa por debajo.

**🧭 Cuándo usarlo:** no lo llamas directamente, pero explica por qué las librerías serias no calculan `(XᵀX)⁻¹` jamás: estabilidad numérica. Si implementas regresión "a mano" con la fórmula del libro y los coeficientes salen absurdos, la QR es la respuesta correcta.

**👔 En una frase para el negocio:** es la diferencia entre cálculos financieramente confiables y coeficientes corruptos por errores de redondeo — ingeniería invisible que protege la calidad de los números que llegan al comité.

### 1.7 Normas vectoriales: L1, L2 y L∞

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Tres formas de medir el mismo viaje: **L2** es la distancia en línea recta del dron; **L1** es lo que marca el taxímetro recorriendo las calles en cuadrícula de Manhattan; **L∞** solo mira el tramo más largo del trayecto — el peor cuello de botella.

**🔧 Definición técnica:**

| Norma | Fórmula | Geometría | Uso principal en ML | Cuándo conviene |
|---|---|---|---|---|
| L1 | `‖x‖₁ = Σ│xᵢ│` | Rombo | Regularización Lasso: empuja coeficientes exactamente a 0 (feature selection) | Cuando esperas que pocas features importen ([[11-Mejora-de-Modelos]]) |
| L2 | `‖x‖₂ = √(Σxᵢ²)` | Círculo | Regularización Ridge, distancia euclídea de KNN/K-Means, MSE | Default general; encoge sin eliminar; distancias geométricas |
| L∞ | `‖x‖∞ = max│xᵢ│` | Cuadrado | Cotas del peor caso, robustez adversarial | Cuando el error máximo importa más que el promedio |

**🧭 Cuándo usarlo:** cada vez que un algoritmo mide "qué tan lejos" o "qué tan grande": la elección de norma cambia el resultado. KNN con L1 vs L2 produce vecinos distintos; Lasso vs Ridge produce modelos distintos ([[07-Modelos-Supervisados]], [[11-Mejora-de-Modelos]]). Las distancias exigen features en la misma escala ([[05-Escalado-de-Datos]]).

**👔 En una frase para el negocio:** define qué significa "parecido" y "error grande" para el modelo — decidir la norma es decidir si castigas muchos errores chicos o un error catastrófico.

### 1.8 Espacios vectoriales y bases

Audiencia: 🔧

> [!tip] 💡 Analogía
> Para dar una dirección dices "3 cuadras al norte y 2 al este": norte y este son tu **base** — referencias independientes con las que describes cualquier punto. Si alguien propone como referencias "norte" y "norte con leve inclinación", la segunda es casi redundante: con referencias así, cualquier dirección se vuelve ambigua y frágil. Elegir buenas referencias es elegir una buena base.

**🔧 Definición técnica:** un espacio vectorial es el conjunto de todas las combinaciones lineales posibles de sus vectores. Una base es un conjunto de vectores linealmente independientes que genera todo el espacio; sus coordenadas son únicas. Las bases ortonormales (vectores perpendiculares de largo 1, construibles con Gram-Schmidt) son las más cómodas: proyectar sobre ellas es un simple producto punto. Las features de un dataset forman el sistema de coordenadas del espacio donde viven las muestras.

**🧭 Cuándo usarlo:** PCA es literalmente un **cambio de base**: rota el sistema de coordenadas hacia las direcciones de máxima varianza. Los embeddings ([[12-Deep-Learning]]) son bases aprendidas donde "parecido semántico" se vuelve "cercano geométricamente".

**👔 En una frase para el negocio:** los mismos datos pueden describirse con ejes torpes o con ejes reveladores; gran parte del valor de un buen equipo de datos está en encontrar el sistema de coordenadas donde el patrón se vuelve obvio.

### 1.9 Proyecciones

Audiencia: 🔧

> [!tip] 💡 Analogía
> La sombra de tu mano en la pared: un objeto 3D convertido en figura 2D. La sombra pierde una dimensión pero, si eliges bien el ángulo de la lámpara, conserva la silueta más reconocible posible. Proyectar datos es elegir ese ángulo: perder dimensiones perdiendo la **menor** información posible.

**🔧 Definición técnica:** la proyección ortogonal de un vector sobre un subespacio es el punto del subespacio más cercano al vector; el residuo es perpendicular al subespacio. En regresión, ŷ es la proyección de y sobre el espacio generado por las columnas de X (por eso los residuos de OLS son ortogonales a las features). PCA proyecta sobre los ejes de máxima varianza; los filtros de Kalman proyectan estados estimados con nueva evidencia.

**🧭 Cuándo usarlo:** siempre que reduces dimensiones o ajustas un modelo lineal, hay una proyección debajo. Entenderla aclara qué información se pierde: todo lo perpendicular al subespacio elegido.

**👔 En una frase para el negocio:** es el mecanismo con que se comprimen cientos de variables en 2–3 ejes visualizables sin destruir el patrón — lo que hace posible "ver" una cartera completa de clientes en un gráfico.

### 1.10 Determinante y rango — la puerta a la multicolinealidad

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El **determinante** es el factor de zoom de una fotocopiadora: det = 2 duplica las áreas, det = 0.5 las achica… y det = 0 aplasta la hoja hasta dejarla convertida en una línea — una operación **irreversible**: de la línea no puedes reconstruir el dibujo. El **rango** es como un panel de 5 "expertos" donde dos se limitan a repetir lo que dice el jefe: en la práctica solo tienes 3 opiniones independientes.

**🔧 Definición técnica:** el determinante mide el factor de escala (con signo) de la transformación; `det(A) = 0` ⇔ A no es invertible ⇔ columnas linealmente dependientes. El rango es el número de columnas linealmente independientes: rango < D indica redundancia entre features. En regresión, si X tiene columnas casi dependientes, XᵀX es casi singular: los coeficientes explotan, cambian de signo entre muestras y pierden interpretabilidad. Eso es la **multicolinealidad**; se diagnostica con VIF en el EDA ([[04-EDA]]) y se trata eliminando/combinando features o con regularización Ridge ([[11-Mejora-de-Modelos]]).

**🧭 Cuándo usarlo:** antes de interpretar coeficientes de cualquier modelo lineal, verifica que no haya multicolinealidad severa. Señales: coeficientes gigantes con signos contraintuitivos, resultados que cambian drásticamente al agregar una feature más.

**👔 En una frase para el negocio:** cuando dos indicadores cuentan la misma historia, el modelo no sabe a cuál darle el crédito — y las conclusiones tipo "el precio pesa más que la ubicación" dejan de ser confiables; detectar redundancia protege la calidad de las decisiones basadas en "qué factor importa más".

> [!warning] ⚠️ Regla crítica
> Nunca interpretes la importancia de variables de un modelo lineal sin revisar multicolinealidad primero. Dos features correlacionadas pueden repartirse el efecto de forma arbitraria — incluso con signos opuestos — y llevar a decisiones de negocio exactamente al revés.

---

## 2. Cálculo Diferencial e Integral

Audiencia: 🔧 🧭

> [!info] 📌 Rol en el ciclo de vida
> El cálculo es el **motor de la optimización**: entrenar un modelo es minimizar una función de pérdida, y el cálculo aporta las herramientas para hacerlo — saber hacia dónde moverse (gradiente), cómo repartir el ajuste entre millones de parámetros (regla de la cadena) y cuándo confiar en que el mínimo encontrado es el bueno (convexidad).

> [!tip] 💡 Analogía general: la montaña con niebla
> Estás en la cima de una montaña cubierta de niebla y necesitas bajar al valle. No ves el camino completo, pero sientes con los pies hacia dónde baja el terreno. El cálculo te da esa "sensación del terreno": la **derivada** es la pendiente bajo tus pies, el **gradiente** la dirección de mayor pendiente, y el **descenso de gradiente** la estrategia de dar pasos cuesta abajo hasta llegar al valle (el error mínimo).

### 2.1 Derivadas y derivadas parciales

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Una mesa de sonido con cien perillas: la derivada parcial es lo que pasa con el volumen general cuando mueves **una sola perilla** dejando las demás quietas. Entrenar un modelo es ajustar miles de perillas, y necesitas saber el efecto individual de cada una.

**🔧 Definición técnica:** la derivada mide la tasa de cambio instantánea de una función. En ML, `∂L/∂w` es la derivada de la pérdida L respecto del peso w: cuánto sube o baja el error si muevo ese peso una pizca. Signo y magnitud indican dirección y tamaño del ajuste necesario.

**🧭 Cuándo usarlo:** todo modelo entrenado por optimización (regresión, redes, boosting) vive de derivadas. También al diseñar funciones de pérdida: deben ser derivables (o casi, como ReLU) para que el entrenamiento fluya.

**👔 En una frase para el negocio:** es el mecanismo que le dice al modelo "cuánto y hacia dónde corregir" en cada uno de sus miles de ajustes internos — la física detrás de la palabra "aprender".

### 2.2 Gradiente

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Una brújula que en lugar de apuntar al norte apunta **cuesta arriba**. Para bajar de la montaña caminas exactamente al revés de lo que marca. Su largo también informa: aguja larga = pendiente fuerte (pasos con efecto), aguja corta = terreno casi plano (estás cerca de un valle... o de una meseta).

**🔧 Definición técnica:** vector de todas las derivadas parciales: `∇L = (∂L/∂w₁, …, ∂L/∂wₙ)`. Apunta en la dirección de máximo crecimiento de L; su negativo, hacia el máximo descenso. `‖∇L‖ ≈ 0` caracteriza puntos críticos (mínimos, máximos o sillas). El descenso de gradiente actualiza `w ← w − α·∇L`.

**🧭 Cuándo usarlo:** monitorear la norma del gradiente diagnostica entrenamientos: gradientes que explotan o se desvanecen son la patología central de las redes profundas ([[12-Deep-Learning]]) y motivan técnicas como gradient clipping, BatchNorm y skip connections.

**👔 En una frase para el negocio:** es la brújula del Machine Learning: si el equipo dice que "el gradiente se desvanece", el modelo dejó de recibir instrucciones útiles y el entrenamiento está quemando cómputo sin mejorar.

### 2.3 Regla de la cadena — el corazón del backpropagation

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> El pastel salió malo y hay que repartir responsabilidades hacia atrás: ¿cuánta culpa tuvo el horno?, ¿cuánta la masa?, ¿cuánta el que compró la harina? Cada eslabón recibe su parte proporcional de la culpa del resultado final. **Backpropagation es exactamente eso**: repartir la culpa del error, capa por capa hacia atrás, para que cada peso sepa cuánto corregirse.

**🔧 Definición técnica:** para funciones compuestas, `d(f∘g)/dx = f'(g(x)) · g'(x)`. Una red neuronal es una composición de decenas de funciones; la regla de la cadena permite calcular el gradiente de la pérdida respecto de **cada** peso propagando derivadas desde la salida hacia la entrada. Los frameworks lo automatizan con autodiferenciación (PyTorch Autograd, TF GradientTape) ([[12-Deep-Learning]]).

**🧭 Cuándo usarlo:** es el algoritmo que hace entrenable cualquier arquitectura diferenciable. Entenderlo explica fenómenos concretos: multiplicar muchas derivadas < 1 hace desaparecer el gradiente en capas tempranas (vanishing gradient) — la razón histórica de que las redes profundas no funcionaran antes de ReLU y las skip connections.

**👔 En una frase para el negocio:** es el invento que hace posible entrenar sistemas con millones de parámetros en horas y no en siglos — la tecnología habilitante detrás de todo el deep learning moderno.

### 2.4 Descenso de gradiente: Batch, SGD y Mini-batch

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Tres formas de decidir cada paso al bajar la montaña: **Batch** encuesta a todo el pueblo antes de moverse (paso certero, pero carísimo); **SGD** le pregunta a un solo vecino (rápido y errático — aunque a veces ese zigzag te saca de un hoyo donde ibas a quedar atrapado); **Mini-batch** consulta a un comité de 32: el equilibrio que usa casi todo el mundo.

**🔧 Definición técnica:** actualización iterativa `w ← w − α·∇L`, donde α es el learning rate. Variantes según cuántos datos calculan el gradiente en cada paso:

| Variante | Datos por paso | Ventajas | Limitaciones | Cuándo conviene |
|---|---|---|---|---|
| Batch GD | Todo el dataset | Gradiente exacto, convergencia estable | Costo por paso enorme; no escala; puede estancarse en mínimos locales | Datasets chicos, problemas convexos |
| SGD | 1 muestra | Barato por paso; el ruido ayuda a escapar de mínimos locales | Trayectoria muy ruidosa; requiere schedule de α | Streaming, online learning |
| Mini-batch GD | 32–512 muestras | Equilibrio ruido/eficiencia; aprovecha paralelismo de GPU | Un hiperparámetro más (batch size) | **El default en deep learning** |

El learning rate α es el hiperparámetro crítico: muy grande → saltas por encima del valle y diverges; muy chico → convergencia eterna. Los optimizadores modernos (Adam, AdamW) y los schedules de α se detallan en [[12-Deep-Learning]].

**🧭 Cuándo usarlo:** siempre que no exista solución cerrada (redes, regresión logística a gran escala, boosting en su espacio funcional). Si el loss no baja: revisa α, escalado de features ([[05-Escalado-de-Datos]]) y calidad de datos, en ese orden.

**👔 En una frase para el negocio:** es el proceso iterativo de "prueba y corrección" que consume las horas de GPU de la factura de cloud — entender sus variantes es entender dónde se puede acelerar o abaratar el entrenamiento.

### 2.5 La Hessiana

Audiencia: 🔧

> [!tip] 💡 Analogía
> La pendiente (gradiente) te dice hacia dónde baja el terreno; la **curvatura** te dice si estás parado en un valle, en una cima o en una silla de montar — que baja hacia adelante pero sube hacia los lados. La Hessiana es el mapa de curvatura del terreno completo.

**🔧 Definición técnica:** matriz de segundas derivadas parciales `H[i,j] = ∂²L/∂wᵢ∂wⱼ`. En un punto crítico: H definida positiva → mínimo local; definida negativa → máximo; eigenvalues de signos mezclados → punto de silla (el caso dominante en alta dimensión). El método de Newton usa H para converger más rápido (`w ← w − H⁻¹∇L`), pero calcular H es O(n²) en memoria: prohibitivo en redes grandes, de ahí los métodos cuasi-Newton (L-BFGS) y los optimizadores de primer orden.

**🧭 Cuándo usarlo:** optimización clásica de modelos medianos (la regresión logística de sklearn usa LBFGS por defecto), análisis de la geometría de la pérdida, y diagnósticos de condicionamiento (razón entre eigenvalues extremos: mal condicionamiento = convergencia lenta).

**👔 En una frase para el negocio:** explica por qué algunos entrenamientos convergen en minutos y otros se arrastran: la "forma del terreno" del problema importa tanto como la potencia de cómputo.

### 2.6 Convexidad

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Un **bol de cocina** vs una **cordillera**: suelta una canica en el bol y siempre termina en el mismo fondo único, sin importar desde dónde la sueltes. En la cordillera puede quedar atrapada en cualquier hoyo local, y ninguno garantiza ser el punto más bajo. Optimizar funciones convexas es jugar en el bol; entrenar redes neuronales es explorar la cordillera.

**🔧 Definición técnica:** f es convexa si el segmento entre dos puntos cualquiera de su gráfica queda por encima de la función: `f(tx + (1−t)y) ≤ t·f(x) + (1−t)·f(y)`. En problemas convexos, todo mínimo local es global: la regresión lineal (MSE) y la regresión logística (log loss) son convexas — solución única y garantizada. Las redes neuronales no lo son: dependen de inicialización, y el "ruido bueno" de SGD ayuda a encontrar buenos mínimos.

```
      CONVEXO (bol)                    NO CONVEXO (cordillera)
   \                 /              \    /\      /\        /
    \               /                \  /  \    /  \  /\  /
     \             /                  \/    \  /    \/  \/
      \           /                    ·     \/      ·
       \_________/                  mínimo   ·    mínimos
            ·                       local  global   locales
      mínimo global único
```

**🧭 Cuándo usarlo:** al elegir modelo según garantías: si necesitas reproducibilidad exacta y auditoría (regulación financiera), los modelos convexos dan la misma respuesta siempre. Si el equipo reporta "el modelo dio distinto en dos corridas", la no-convexidad (+ semillas aleatorias, [[13-MLOps-XAI-Etica]]) es la explicación.

**👔 En una frase para el negocio:** separa los modelos con respuesta única y garantizada de los que exigen arte y varios intentos — un criterio real al decidir entre un modelo simple auditable y una red profunda.

### 2.7 Integrales

Audiencia: 🔧

> [!tip] 💡 Analogía
> Sumar rebanadas: para saber cuánta masa tiene un pan con forma irregular, lo cortas en rebanadas finísimas y las sumas. La integral es esa suma llevada al límite de rebanadas infinitamente delgadas — el "área bajo la curva".

**🔧 Definición técnica:** acumulación continua. En ML aparece en tres lugares clave: (1) toda densidad de probabilidad integra 1 (`∫f(x)dx = 1`), y probabilidades de intervalos son áreas bajo la PDF; (2) la esperanza es una integral: `E[X] = ∫x·f(x)dx`; (3) el AUC-ROC es literalmente el área bajo una curva ([[08-Metricas-de-Evaluacion]]). En la práctica se calculan numéricamente (regla del trapecio) o por muestreo Monte Carlo.

**🧭 Cuándo usarlo:** interpretar probabilidades continuas, métricas de área (AUC-ROC, AUC-PR), inferencia bayesiana (normalizar el posterior exige integrar). Saber que "probabilidad = área" evita el clásico error de pedir P(X = valor exacto) en variables continuas (es 0).

**👔 En una frase para el negocio:** cada vez que el equipo reporta "AUC de 0.91", está reportando un área bajo una curva — este es el concepto que la sostiene.

---

## 3. Probabilidad y Teoría de la Incertidumbre

Audiencia: 🔧 🧭

> [!info] 📌 Rol en el ciclo de vida
> Los modelos de ML no entregan certezas: asignan **probabilidades**. La teoría de probabilidad es el lenguaje para interpretarlas correctamente, combinarlas con información previa (Bayes) y no confundir coincidencia con relación real (Bishop, 2006).

> [!tip] 💡 Analogía general: el detector de mentiras de tus datos
> La probabilidad es un detector de mentiras para patrones: te ayuda a discernir si lo que encontraste es una verdad del proceso (ocurre consistentemente) o una coincidencia que apareció en esta muestra particular. Sin ella, no distingues señal de ruido.

### 3.1 Espacio muestral y probabilidad condicional

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> P(lloverá hoy) es una cosa; P(lloverá hoy │ el cielo está negro) es otra muy distinta. Condicionar es **encoger el universo**: ya no consideras todos los días posibles, solo los días de cielo negro, y recalculas las chances dentro de ese subconjunto. Todo pronóstico serio es condicional: la pregunta nunca es "¿qué pasará?" sino "¿qué pasará *dado lo que ya sé*?".

**🔧 Definición técnica:** el espacio muestral Ω es el conjunto de todos los resultados posibles; los eventos son subconjuntos de Ω. La probabilidad condicional es `P(A│B) = P(A∩B) / P(B)`: la probabilidad de A restringida al universo donde B ocurrió. Es la base de los clasificadores bayesianos y de la lectura correcta de cualquier score: un modelo de churn no estima P(fuga), estima P(fuga │ features del cliente).

**🧭 Cuándo usarlo:** siempre que leas un score o una tasa: pregunta *condicional a qué*. Una tasa de fraude del 0.1% global puede ser 15% condicional a "transacción nocturna internacional". Segmentar es condicionar.

**👔 En una frase para el negocio:** el contexto cambia las probabilidades — el valor de los modelos está justamente en calcular las chances *para cada caso específico*, no el promedio de todos.

### 3.2 Teorema de Bayes

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un médico ante un examen positivo de una enfermedad **rara**. Antes del examen, su sospecha era bajísima (prior). Llega el resultado positivo (evidencia) y no concluye "enfermo seguro": actualiza. Si la enfermedad afecta a 1 de cada 1.000 personas y el test acierta el 99%, entre 1.000 pacientes habrá ≈1 enfermo detectado y ≈10 sanos con falso positivo: ¡la mayoría de los positivos están sanos! La intuición falla; Bayes no.

**🔧 Definición técnica:** `P(A│B) = P(B│A)·P(A) / P(B)`; en lenguaje de inferencia: `posterior ∝ likelihood × prior`. Permite invertir condicionales: de P(síntoma│enfermedad), que conoce la ciencia, a P(enfermedad│síntoma), que necesita la decisión. Es la base de Naive Bayes ([[07-Modelos-Supervisados]]), la inferencia bayesiana, el A/B testing bayesiano y la inferencia variacional.

**🧭 Cuándo usarlo:** cuando combinas una tasa base con evidencia nueva: scoring de fraude, diagnóstico, filtros de spam. Regla de oro: nunca leas la "precisión del test" sin la prevalencia — con eventos raros, hasta un test excelente produce mayoría de falsas alarmas (por eso existen métricas específicas para desbalance, [[08-Metricas-de-Evaluacion]]).

**👔 En una frase para el negocio:** enseña que un detector "99% preciso" de un evento raro genera montañas de falsas alarmas — el argumento matemático para dimensionar equipos de revisión antes de comprar cualquier sistema de detección.

### 3.3 Independencia vs correlación

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Las ventas de helado y los ahogamientos suben y bajan juntos todos los años. ¿El helado ahoga? No: el **verano** mueve a ambos. Correlación es bailar la misma canción; causalidad es que uno le pague la orquesta al otro. Y dos dados son independientes: por muchos seises que saque el primero, no le sopla nada al segundo.

**🔧 Definición técnica:** A y B son independientes si `P(A∩B) = P(A)·P(B)` (equivalente: P(A│B) = P(A)). La correlación de Pearson mide **dependencia lineal** en [−1, 1]. Trampas clásicas: correlación 0 no implica independencia (relaciones no lineales, como y = x², dan r ≈ 0); correlación alta no implica causalidad (confounders, casualidad, causalidad inversa). Naive Bayes *asume* independencia condicional entre features — falsa casi siempre, útil de todos modos ([[07-Modelos-Supervisados]]).

**🧭 Cuándo usarlo:** en EDA, para leer heatmaps de correlación con escepticismo ([[04-EDA]]); en feature selection, para no eliminar features "no correlacionadas" que tienen relación no lineal con el target (usa mutual information, [[03-Preparacion-de-Datos]]).

**👔 En una frase para el negocio:** que dos métricas se muevan juntas no significa que empujar una mueva la otra — antes de invertir sobre una correlación, exige el experimento que pruebe causalidad ([[10-Validacion-y-Leakage]]).

> [!danger] 🚨 Error costoso: actuar sobre correlaciones espurias
> "Los clientes que usan la app churnean menos" puede significar que la app retiene… o que los clientes ya fieles son los que instalan la app. Invertir millones en instalar la app a todos, sin un experimento controlado, es apostar a que la flecha causal apunta en la dirección conveniente.

### 3.4 Variables aleatorias

Audiencia: 🔧

> [!tip] 💡 Analogía
> Una tómbola que convierte azar en números: giras y sale un valor. Si la tómbola entrega valores que se **cuentan** (0, 1, 2 reclamos), es discreta; si entrega valores que se **miden** (3.7182 minutos de espera), es continua. Toda métrica de tu negocio es una tómbola con su propio patrón interno.

**🔧 Definición técnica:** una variable aleatoria X es una función que asigna un número a cada resultado del espacio muestral. Discretas: valores contables (Bernoulli, Binomial, Poisson). Continuas: valores en intervalos (Normal, Exponencial, Beta, Gamma). La distinción define qué herramientas aplican: PMF vs PDF, sumas vs integrales, y qué modelos/tests son válidos.

**🧭 Cuándo usarlo:** al modelar cualquier cantidad incierta: el primer paso es clasificarla (¿conteo, proporción, tiempo, medida?) para elegir distribución y método correctos (ver sección 4).

**👔 En una frase para el negocio:** formaliza la idea de que ventas, demanda o fallas no son números fijos sino rangos con probabilidades — el punto de partida para planificar con escenarios en lugar de promesas.

### 3.5 PDF, PMF y CDF

Audiencia: 🔧

> [!tip] 💡 Analogía
> **PMF**: la lista de precios de un kiosco — cada producto exacto tiene su probabilidad (P(sale un 3) = 1/6). **PDF**: la crema sobre una torta — en un punto exacto no hay "cantidad de crema", solo en porciones; la probabilidad vive en los intervalos (áreas). **CDF**: el ranking percentil de una prueba — "¿qué fracción quedó igual o por debajo de este puntaje?"; siempre sube, de 0 a 1.

**🔧 Definición técnica:** PMF (discreta): `p(x) = P(X = x)`, suma 1. PDF (continua): f(x) tal que `P(a ≤ X ≤ b) = ∫ₐᵇ f(x)dx`; f(x) puede superar 1, las probabilidades son áreas. CDF: `F(x) = P(X ≤ x)`, monótona creciente de 0 a 1, une ambos mundos; su inversa da los cuantiles (la base del QuantileTransformer, [[05-Escalado-de-Datos]], y de la Quantile Loss, [[08-Metricas-de-Evaluacion]]).

**🧭 Cuándo usarlo:** histogramas y KDE estiman la PDF empírica ([[04-EDA]]); percentiles y boxplots leen la CDF; simulaciones Monte Carlo muestrean de estas funciones para valorar riesgo.

**👔 En una frase para el negocio:** son los tres formatos en que se responde "¿qué tan probable es cada escenario?" — el detalle exacto, la forma general y el acumulado tipo percentil.

### 3.6 Esperanza, varianza y covarianza

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> La **esperanza** es el promedio del casino: en una noche cualquiera puede ganar cualquiera, pero a millones de jugadas la casa converge exactamente a su margen. La **varianza** es qué tan brava es la montaña rusa alrededor de ese promedio. La **covarianza** es si dos acciones bailan la misma canción (suben juntas), canciones opuestas, o ni se escuchan.

**🔧 Definición técnica:** `E[X] = Σx·p(x)` (o `∫x·f(x)dx`): el centro de gravedad de la distribución. `Var[X] = E[(X−μ)²]`: dispersión cuadrática media; su raíz es σ. `Cov(X,Y) = E[(X−μₓ)(Y−μᵧ)]`: co-movimiento lineal; normalizada da la correlación de Pearson. Momentos superiores (3° y 4°) describen asimetría y colas (ver 3.7). La matriz de covarianza es el insumo de PCA y del Elliptic Envelope ([[07-Modelos-Supervisados]]).

**🧭 Cuándo usarlo:** todo reporte serio acompaña el promedio con su dispersión: una media de 100 ± 5 y una de 100 ± 80 exigen decisiones opuestas. En validación, la varianza entre folds mide la confiabilidad de la métrica ([[10-Validacion-y-Leakage]]).

**👔 En una frase para el negocio:** el promedio dice cuánto esperar; la varianza dice cuánto puede doler la diferencia — presupuestar con la primera e ignorar la segunda es planificar sin margen de riesgo.

### 3.7 Skewness y kurtosis

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> **Skewness**: en un barrio donde vive un multimillonario, el ingreso "promedio" es una fantasía — la cola derecha arrastra la media muy por encima de la mediana. **Kurtosis**: dos playas con la misma ola promedio; en una, cada tanto llega una ola gigante (colas pesadas). Los mercados financieros son la segunda playa: ignorar la kurtosis es que el "evento de una vez por siglo" te sorprenda cada década.

**🔧 Definición técnica:** skewness (3er momento estandarizado) mide asimetría: positiva → cola derecha, media > mediana > moda (ingresos, precios, montos); negativa → cola izquierda. Kurtosis (4° momento) mide el peso de las colas: > 3 (leptocúrtica) = colas pesadas, más outliers que la Normal; < 3 (platicúrtica) = colas livianas. Diagnóstico estándar en el EDA univariado ([[04-EDA]]); skew fuerte motiva transformaciones log/Box-Cox ([[05-Escalado-de-Datos]]).

**🧭 Cuándo usarlo:** antes de usar media y desviación estándar como resúmenes (con skew fuerte, mediana e IQR representan mejor), antes de aplicar tests que asumen normalidad, y al decidir transformar el target en regresión ([[03-Preparacion-de-Datos]]).

**👔 En una frase para el negocio:** avisa cuándo el promedio miente: con datos asimétricos, "el cliente promedio" no existe — y las decisiones deben mirar medianas, percentiles y riesgo de extremos.

---

## 4. Distribuciones de Probabilidad

Audiencia: 🔧 🧭

> [!info] 📌 Rol en el ciclo de vida
> Cada distribución es un **patrón de azar con nombre propio**. Reconocer cuál siguen tus datos define qué modelos y tests son válidos: la regresión lineal asume residuos normales, la logística nace de una Bernoulli, y los conteos de eventos raros piden Poisson. Usar el patrón equivocado invalida silenciosamente las conclusiones.

> [!tip] 💡 Analogía general: moldes de repostería
> Las distribuciones son moldes: la masa (tus datos) toma la forma del molde que le corresponde por naturaleza. El trabajo no es forzar la masa en tu molde favorito, sino **reconocer qué molde la produjo** — porque cada molde trae su set de herramientas (fórmulas, tests, modelos) que solo funcionan con él.

| Distribución | Parámetros | Forma / Uso en ML | 💡 Analogía |
|---|---|---|---|
| Normal (Gaussiana) | μ (media), σ (desv. estándar) | Campana simétrica. Supuesto de residuos en regresión lineal, inicialización de pesos, límite del TCL | Las estaturas de un país: casi todos cerca del promedio, gigantes y muy bajos son rarísimos |
| Bernoulli | p (prob. de éxito) | Experimento binario 0/1. Fundamento de la regresión logística y de toda clasificación binaria | Un único lanzamiento de una moneda (posiblemente cargada): sale cara o no |
| Binomial | n (ensayos), p | Nº de éxitos en n ensayos independientes. Conteos acotados: conversiones sobre visitas | Contar cuántas caras salen en 10 lanzamientos de la misma moneda |
| Poisson | λ (tasa media) | Nº de eventos en una ventana de tiempo/espacio. Conteos raros: reclamos/hora, fallas/día, clics | Clientes que entran a una tienda por hora: sabes el ritmo promedio, no el minuto exacto |
| Uniforme | a, b (límites) | Todos los valores equiprobables. Generación aleatoria, inicialización, priors "sin opinión" | Una rifa justa: todos los números tienen exactamente la misma chance |
| Beta | α, β | Distribución **de probabilidades** en [0,1]. Prior bayesiano de proporciones, A/B testing bayesiano | La reputación de un vendedor online: con 2 reseñas es difusa, cada nueva reseña la afina |
| Gamma | α (forma), β (escala) | Tiempos de espera acumulados, montos positivos asimétricos. Generaliza la Exponencial | Cuánto esperas en la fila hasta que atiendan a las k personas delante tuyo |
| Exponencial | λ (tasa) | Tiempo entre eventos Poisson. **Sin memoria**. Supervivencia, confiabilidad, colas | El tiempo hasta el próximo bus "sin horario": llevar 10 min esperando no lo acerca nada |
| t de Student | ν (grados de libertad) | Como la Normal pero de colas pesadas. Inferencia con σ desconocida y N pequeño; base del t-test | La Normal con seguro contra sorpresas: con pocos datos, deja margen para extremos |
| Chi-cuadrado (χ²) | ν (grados de libertad) | Suma de normales estándar al cuadrado. Tests de bondad de ajuste e independencia | El medidor de "cuán lejos quedó lo observado de lo esperado", acumulando desvíos al cuadrado |

**La campana Normal y su regla 68–95–99.7** (la referencia mental más usada de la estadística):

```
                          ▄▄█████▄▄
                       ▄██│       │██▄
                     ▄█   │       │   █▄
                   ▄█     │       │     █▄
                 ▄█       │       │       █▄
               ▄█  ◄──────┼ 68% ──┼──────►  █▄
             ▄█   ◄───────┼─ 95% ─┼───────►   █▄
          ▄▄█    ◄────────┼ 99.7% ┼────────►    █▄▄
    ──────┴─────┬─────┬───┴───┬───┴───┬─────┬─────┴──────
              μ−3σ  μ−2σ    μ−σ  μ  μ+σ   μ+2σ  μ+3σ
```

Si algo "normal" cae a más de 3σ de la media, es un evento de ~0.3% de probabilidad: por eso `│z│ > 3` es el umbral clásico de outlier ([[03-Preparacion-de-Datos]]).

**🧭 Guía rápida — ¿qué distribución sospechar según el dato?**

| Tipo de dato | Sospecha primero | Ejemplo |
|---|---|---|
| Medida continua simétrica | Normal | Errores de medición, promedios de muestras grandes |
| Resultado sí/no | Bernoulli (uno) / Binomial (varios) | Conversión de una campaña |
| Conteo de eventos por período | Poisson | Reclamos por semana, fallas por turno |
| Tiempo hasta el próximo evento | Exponencial / Gamma | Tiempo entre caídas del sistema |
| Proporción o tasa en [0,1] | Beta | Tasa de conversión estimada con pocos datos |
| Media con muestra chica | t de Student | Piloto con 15 tiendas |

**👔 En una frase para el negocio:** reconocer el patrón de azar correcto es lo que separa un pronóstico honesto de uno decorativo — el molde equivocado subestima justo los riesgos que más duelen.

---

## 5. Estadística Inferencial

Audiencia: 🔧 🧭 👔

> [!info] 📌 Rol en el ciclo de vida
> La inferencia permite concluir sobre la **población** a partir de una **muestra**, y validar si un resultado — la mejora de un modelo, el lift de una campaña — es estadísticamente real o producto del azar (Wasserman, 2004). Es la caja de herramientas detrás de todo A/B test y de la comparación seria de modelos ([[10-Validacion-y-Leakage]]).

> [!tip] 💡 Analogía general: catar la sopa
> Para saber si a la olla gigante le falta sal, no te la tomas entera: pruebas **una cucharada**. La inferencia estadística es la ciencia de la cucharada: qué tan grande debe ser, qué tan revuelta la olla, y cuánta confianza da el veredicto. Todo lo demás — p-valores, intervalos, potencia — son refinamientos de esa idea.

> [!example] 📊 Caso de negocio — E-commerce: la fábrica de falsos ganadores
> **Problema:** un e-commerce corre ~25 experimentos A/B por trimestre (botones, precios, copys). Con umbral p < 0.05 y sin corrección, la aritmética es cruel: solo por azar se esperan 1–2 "ganadores" falsos por trimestre. El equipo celebra lifts que luego no aparecen en la cuenta de resultados, y la credibilidad del programa de experimentación se erosiona.
>
> **Técnica aplicada:** (1) cálculo de potencia **antes** de cada test para fijar tamaño muestral y duración mínima (no parar el test "cuando se ve bonito"); (2) corrección por comparaciones múltiples — FDR de Benjamini-Hochberg sobre el lote de experimentos del trimestre; (3) reporte estándar con intervalo de confianza del lift, no solo el p-valor.
>
> **Resultado:** se declaran menos ganadores, pero los que se implementan sostienen su lift en producción. La conversación ejecutiva cambia de "¿por qué la mejora desapareció?" a "¿cuánto invertimos en escalar este cambio?", y el programa de experimentación recupera autoridad para frenar lanzamientos sin evidencia.

### 5.1 Estimación puntual e intervalos de confianza

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un dardo vs un aro: la estimación puntual lanza **un dardo** ("la conversión es 3.2%"); el intervalo de confianza lanza **un aro** ("entre 2.8% y 3.6%"). Y la garantía es sutil: si repitieras el estudio 100 veces, ~95 de los aros capturarían el valor verdadero. La honestidad del aro está en su tamaño: aro gigante = sabes poco.

**🔧 Definición técnica:** un estimador puntual resume el parámetro en un valor (media muestral x̄ estima μ). El IC del 95% para una media: `x̄ ± 1.96·σ/√N` (con σ conocida o N grande; con N chico se usa t de Student). Interpretación frecuentista correcta: el 95% se refiere al **procedimiento** (de 100 intervalos construidos así, ~95 contienen el parámetro), no a "95% de probabilidad de que el parámetro esté aquí". El ancho decrece con √N: para achicar el intervalo a la mitad, necesitas 4× más datos.

**🧭 Cuándo usarlo:** siempre que reportes una métrica estimada: conversión, NPS, AUC de un modelo (media ± IC sobre los folds del cross-validation, [[10-Validacion-y-Leakage]]). Dos modelos cuyos IC se traslapan ampliamente no son distinguibles con esos datos.

**👔 En una frase para el negocio:** exige siempre el rango, no solo el número — "3.2%" y "3.2% ± 2%" autorizan decisiones completamente distintas.

### 5.2 Pruebas de hipótesis y p-valor

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Un juicio penal. H₀ es la presunción de inocencia ("no hay efecto real"); los datos son la evidencia; el **p-valor** responde: *si el acusado fuera inocente, ¿qué tan raro sería encontrar esta evidencia?* Si es rarísima (p pequeño), condenas — rechazas la inocencia. Ojo: un juicio nunca "prueba inocencia"; solo declara si la evidencia alcanzó o no para condenar.

**🔧 Definición técnica:** se plantea H₀ (hipótesis nula: no hay efecto) contra H₁ (alternativa). El p-valor es `P(observar datos al menos tan extremos como estos │ H₀ es verdadera)`. Convención: p < 0.05 → se rechaza H₀ (umbral arbitrario pero estándar; dominios críticos usan 0.01 o menos). El p-valor **no** es: la probabilidad de que H₀ sea cierta, ni la probabilidad de equivocarse, ni el tamaño del efecto. Un p minúsculo con efecto irrelevante es común con N gigante: significancia estadística ≠ relevancia práctica.

**🧭 Cuándo usarlo:** validar campañas, comparar modelos ([[10-Validacion-y-Leakage]]), verificar si una diferencia entre segmentos es real ([[04-EDA]]). Reporta siempre p-valor + tamaño del efecto + IC: los tres juntos cuentan la historia completa.

**👔 En una frase para el negocio:** el p-valor mide qué tan improbable sería tu resultado si todo fuera puro azar — es el filtro mínimo, no el veredicto final: pregunta también *cuánto* es el efecto y *cuánto vale*.

> [!warning] ⚠️ Regla crítica: el p-valor no se "cosecha"
> Mirar el experimento todos los días y detenerlo justo cuando p < 0.05 (*p-hacking* / *optional stopping*) infla los falsos positivos muy por encima del 5% prometido. El tamaño muestral y la duración se fijan **antes** de empezar, con un cálculo de potencia.

### 5.3 Errores tipo I y II, y potencia estadística

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Una alarma de incendios. **Error tipo I (α):** la alarma suena sin fuego — falsa alarma, evacuas el edificio por nada. **Error tipo II (β):** hay fuego y la alarma calla — el error que puede costar el edificio. **Potencia (1−β):** qué tan sensible es el detector cuando el fuego es real. Ajustar la alarma es un trade-off: más sensible = más falsas alarmas; más tolerante = más incendios perdidos. No existe alarma perfecta; existe la alarma calibrada al costo de cada error.

**🔧 Definición técnica:**

| | H₀ verdadera (no hay efecto) | H₀ falsa (el efecto existe) |
|---|---|---|
| **Rechazo H₀** | ❌ Error tipo I (prob. = α) — falso positivo | ✅ Detección correcta (potencia = 1−β) |
| **No rechazo H₀** | ✅ Conclusión correcta | ❌ Error tipo II (prob. = β) — falso negativo |

La potencia depende de: tamaño muestral N, tamaño del efecto real, variabilidad de los datos y α. Estándar de diseño: potencia ≥ 0.80. El cálculo de potencia previo responde "¿cuántos datos necesito para detectar un efecto de tamaño X con 80% de probabilidad?". Este mismo trade-off α/β reaparece como precision/recall en clasificación ([[08-Metricas-de-Evaluacion]]): son las dos caras del mismo dilema.

**🧭 Cuándo usarlo:** diseñar cualquier experimento (antes, no después); interpretar resultados "no significativos": con potencia baja, no encontrar efecto no prueba que no exista — solo que tu lupa era demasiado débil para verlo.

**👔 En una frase para el negocio:** un piloto sin cálculo de potencia puede estar condenado desde el día uno a "no encontrar nada" — pregunta siempre *¿teníamos potencia suficiente para detectar el efecto que buscábamos?* antes de archivar una iniciativa.

### 5.4 El catálogo de tests estadísticos

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Los tests son destornilladores: cada tipo de tornillo (pregunta + tipo de dato) tiene el suyo. Forzar el destornillador equivocado — un t-test sobre datos brutalmente asimétricos con N = 12 — no da un error en pantalla: da una conclusión inválida con cara de válida. La tabla es el estuche completo.

**🔧 Definición técnica y guía de selección:**

| Prueba | Pregunta que responde (ejemplo) | Cuándo usar | Supuestos clave |
|---|---|---|---|
| t-test (1 muestra) | ¿La media difiere del valor objetivo? ("¿el tiempo medio de atención es 5 min?") | Comparar media muestral vs valor hipotético | Normalidad o N grande (TCL) |
| t-test (2 muestras independientes) | ¿La campaña A vendió más que la B? | Comparar medias de dos grupos distintos | Normalidad; homogeneidad de varianzas (verificar con Levene; si falla, usar variante de Welch) |
| t-test apareado | ¿Mejoró el indicador tras la intervención, **en los mismos** sujetos? | Antes/después sobre las mismas unidades | Diferencias distribuidas normalmente |
| ANOVA (1 vía) | ¿Alguna de las 4 sucursales rinde distinto? | Comparar medias de 3+ grupos (evita multiplicar t-tests) | Normalidad; homogeneidad de varianzas |
| Chi-cuadrado (χ²) | ¿El plan contratado depende de la región? | Independencia entre dos variables categóricas | Frecuencias esperadas ≥ 5 por celda |
| Mann-Whitney U | ¿Un grupo tiende a valores mayores? (datos sesgados) | Alternativa no paramétrica al t-test de 2 muestras | Sin supuesto de normalidad; ordinal o continua |
| Kruskal-Wallis | ¿Difieren 3+ grupos? (sin normalidad) | ANOVA no paramétrico | Sin supuesto de normalidad |
| Correlación de Pearson | ¿Asociación **lineal** entre dos continuas? | Relación lineal, datos "limpios" | Linealidad; normalidad bivariada; sensible a outliers |
| Correlación de Spearman | ¿Asociación **monotónica** (crece, aunque no en línea recta)? | Ordinales, outliers, relaciones no lineales monótonas | Solo monotonicidad; robusta a outliers |

Flujo de decisión rápido 🧭: ¿comparas **medias** o **categorías**? → medias: 2 grupos (t-test) o 3+ (ANOVA); categorías: χ². ¿Datos claramente no normales y N chico? → cambia a la versión no paramétrica (Mann-Whitney / Kruskal-Wallis). ¿Mides asociación? → lineal: Pearson; monotónica u ordinal: Spearman. Estos mismos tests reaparecen en el EDA bivariado ([[04-EDA]]) y en la comparación de modelos ([[10-Validacion-y-Leakage]]).

**👔 En una frase para el negocio:** existe un test correcto para cada pregunta con datos — pedir "qué test usaron y por qué" es una forma legítima y rápida de auditar la seriedad de un análisis.

### 5.5 Ley de los Grandes Números (LGN)

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> El casino no sabe si **tú** ganarás esta noche — y no le importa. Con millones de jugadas, el promedio de la casa converge matemáticamente a su margen de diseño. La suerte individual es real en lo chico e irrelevante en lo grande: a escala, el azar se vuelve contabilidad.

**🔧 Definición técnica:** con muestras i.i.d., la media muestral converge a la esperanza poblacional cuando N → ∞: `x̄ₙ → μ`. Justifica que las métricas calculadas sobre conjuntos de validación grandes sean estimaciones confiables del rendimiento real, y que las simulaciones Monte Carlo converjan a la respuesta correcta.

**🧭 Cuándo usarlo:** para dimensionar cuánta data necesita una evaluación confiable: con test sets chicos, la métrica baila entre corridas (por eso existe cross-validation, [[10-Validacion-y-Leakage]]); con millones de casos, el promedio es sólido.

**👔 En una frase para el negocio:** explica por qué los modelos aciertan "en el agregado" aunque fallen casos individuales — y por qué las conclusiones sobre muestras chicas merecen desconfianza sistemática.

### 5.6 Teorema Central del Límite (TCL)

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> La magia de los promedios: no importa qué forma rara tenga lo que midas — dados, ingresos torcidos, tiempos de espera — si tomas muchas muestras y promedias cada una, el histograma de esos **promedios** dibuja siempre la misma campana. Es el gran unificador: procesos distintos, promedios igualmente predecibles.

**🔧 Definición técnica:** la distribución de la media muestral se aproxima a una Normal `N(μ, σ²/N)` cuando N crece, **independientemente de la distribución original** (con varianza finita). Regla práctica: N ≥ 30 suele bastar (más si la asimetría es fuerte). Es la razón de que t-tests, ANOVA e intervalos de confianza funcionen incluso con datos no normales, siempre que se trabaje sobre medias de muestras razonables.

**🧭 Cuándo usarlo:** fundamenta casi toda la inferencia práctica: IC de métricas, comparación de medias, control estadístico de procesos. Cuando N es chico y los datos muy asimétricos, el TCL aún no "activó": usa tests no paramétricos o bootstrap.

**👔 En una frase para el negocio:** es la garantía matemática de que los promedios de muestras razonables se comportan de forma predecible — el fundamento de por qué se puede confiar en encuestas, pilotos y muestras sin medir a toda la población.

### 5.7 Comparaciones múltiples: Bonferroni y FDR

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Si compras un boleto de lotería, ganar sería asombroso; si compras 10.000, que uno gane no dice nada. Cada test estadístico al 5% es un boleto: corre 20 tests y *esperas* un "descubrimiento" falso por puro azar. Las correcciones suben la vara de asombro cuando hiciste muchas preguntas a la vez.

**🔧 Definición técnica:** con k tests al nivel α, la probabilidad de ≥1 falso positivo es `1 − (1−α)ᵏ` (con k = 20 y α = 0.05: ≈64%). **Bonferroni** controla el error familiar usando `α/k` por test: simple y conservador (con k grande, mata la potencia). **FDR (Benjamini-Hochberg)** controla la *proporción esperada de falsos descubrimientos* entre los rechazos: ordena los p-valores y acepta hasta donde `p₍ᵢ₎ ≤ (i/k)·q`; mucho más potente cuando k es grande. Bonferroni cuando un solo falso positivo es inaceptable; FDR cuando exploras muchas hipótesis y toleras una fracción controlada de falsos hallazgos.

**🧭 Cuándo usarlo:** screening de features contra el target ([[03-Preparacion-de-Datos]]), lotes de A/B tests, comparación de muchos modelos/configuraciones ([[10-Validacion-y-Leakage]], [[11-Mejora-de-Modelos]]), análisis por múltiples segmentos ("funciona en mujeres de 25–34 en el sur…" — cuantos más cortes, más boletos de lotería).

**👔 En una frase para el negocio:** cuando el equipo probó veinte cosas y una "funcionó", la pregunta correcta es *¿corrigieron por comparaciones múltiples?* — de lo contrario, es probable que estén celebrando un boleto premiado por azar.

---

## 6. Síntesis: dónde se usa cada fundamento

Audiencia: 🔧 🧭

| Fundamento | Técnicas de ML que lo usan directamente | Tomo donde se aplica |
|---|---|---|
| SVD / eigendecomposición / proyecciones | PCA, LSA, sistemas de recomendación, clustering espectral | [[03-Preparacion-de-Datos]], [[06-Clustering]] |
| Normas L1/L2 | Lasso, Ridge, ElasticNet, distancias de KNN/K-Means | [[07-Modelos-Supervisados]], [[11-Mejora-de-Modelos]] |
| Rango / determinante | Diagnóstico de multicolinealidad (VIF) | [[04-EDA]] |
| Gradiente + regla de la cadena | Backpropagation, descenso de gradiente, boosting | [[12-Deep-Learning]], [[07-Modelos-Supervisados]] |
| Convexidad | Garantías de regresión lineal/logística y SVM | [[07-Modelos-Supervisados]] |
| Bayes | Naive Bayes, calibración, A/B testing bayesiano | [[07-Modelos-Supervisados]], [[11-Mejora-de-Modelos]] |
| PDF/CDF y cuantiles | KDE del EDA, QuantileTransformer, Quantile Loss | [[04-EDA]], [[05-Escalado-de-Datos]], [[08-Metricas-de-Evaluacion]] |
| Esperanza/varianza/covarianza | Bias-variance tradeoff, PCA, evaluación de riesgo | [[11-Mejora-de-Modelos]] |
| Distribuciones | Supuestos de modelos, detección de outliers, GMM | [[03-Preparacion-de-Datos]], [[06-Clustering]] |
| Tests, IC, potencia, FDR | Comparación de modelos, A/B testing, EDA bivariado | [[10-Validacion-y-Leakage]], [[04-EDA]] |

---

## 📖 Referencias de este tomo

- (Mitchell, 1997) — definición formal de aprendizaje.
- (Goodfellow et al., 2016) — capítulos de álgebra lineal, probabilidad y optimización aplicadas a ML.
- (Bishop, 2006) — tratamiento probabilístico riguroso de los fundamentos.
- (Hastie et al., 2009) y (James et al., 2021) — puente entre estadística clásica y aprendizaje estadístico.
- (Wasserman, 2004) — inferencia estadística concisa y completa.
- (Géron, 2022) — perspectiva práctica de estos fundamentos en código.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[01-Introduccion-Ejecutiva|01 · Introducción Ejecutiva]] · Siguiente: [[03-Preparacion-de-Datos|03 · Preparación de Datos ➡]]

> **Próximo tomo:** [[03-Preparacion-de-Datos]] — donde se gasta el 80% del tiempo real de un proyecto: limpieza, nulos (MCAR/MAR/MNAR), outliers, encoding, desbalance, feature engineering/selection y reducción de dimensionalidad.


