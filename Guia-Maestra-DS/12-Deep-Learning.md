---
title: "Tomo 12 — Deep Learning: de los Fundamentos a las Arquitecturas Modernas"
tags: [data-science, machine-learning, deep-learning, redes-neuronales, transformers]
audiencias: [tecnico, puente, ejecutivo]
tomo: 12
version: 6.0
---

# 🧠 Tomo 12 — Deep Learning: de los Fundamentos a las Arquitecturas Modernas

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[11-Mejora-de-Modelos|11 · Mejora de Modelos]] · Siguiente: [[13-MLOps-XAI-Etica|13 · MLOps, XAI y Ética ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> El Deep Learning no es "ML con más capas": es un paradigma donde el modelo **aprende automáticamente representaciones jerárquicas** de los datos, eliminando el feature engineering manual. Una red profunda aprende bordes en la primera capa, formas en la segunda, objetos en la tercera — la representación emerge del entrenamiento (Goodfellow et al., 2016). Es el estado del arte indiscutido en imágenes, texto, audio y secuencias; en tabular, el boosting sigue peleando ([[07-Modelos-Supervisados]]).

> [!abstract] 👔 Impacto ejecutivo
> La tecnología detrás de todo lo que hoy se llama "IA": visión, lenguaje, generación. Cara de entrenar, transformadora cuando el problema la amerita.
>
> - **Decisiones que habilita:** automatizar tareas de percepción (leer documentos, inspeccionar productos, entender clientes), aprovechar modelos pre-entrenados a una fracción del costo histórico.
> - **Costo de hacerlo mal:** GPU quemada en problemas que un boosting resolvía, modelos gigantes sin datos que los justifiquen, y proyectos "IA" sin caso de negocio.
> - **Pregunta ejecutiva que responde:** *¿este problema necesita deep learning, y si sí, entrenamos desde cero o adaptamos un modelo pre-entrenado?*

> [!tip] 💡 El ejemplo introductorio: el niño que aprende a reconocer gatos
> ¿Cómo le enseñas a un niño qué es un gato? No le dictas reglas ("orejas puntiagudas, bigotes…"): le muestras **miles de fotos** de gatos y no-gatos, y aprende solo. Primero detecta bordes, luego formas (orejas, ojos), luego combinaciones (cara de gato), y finalmente el concepto "gato". Cada capa de una red profunda es exactamente eso: un nivel de abstracción construido sobre el anterior. Eso es el **aprendizaje jerárquico de representaciones**.

> [!example] 📊 Caso de negocio — Manufactura: inspección visual sin escribir reglas
> **Problema:** una planta inspecciona soldaduras con reglas de visión artesanales que fallan con cada cambio de iluminación o proveedor; los defectos nuevos pasan de largo.
>
> **Técnica aplicada:** CNN pre-entrenada en ImageNet adaptada por **transfer learning** (feature extraction primero, fine-tuning parcial después) con solo unos miles de fotos etiquetadas por los propios inspectores; data augmentation (rotaciones, brillo) para multiplicar el dataset ([[11-Mejora-de-Modelos]]); umbral de decisión calibrado para minimizar falsos negativos (defecto que escapa) tolerando revisión humana de dudosos ([[08-Metricas-de-Evaluacion]]).
>
> **Resultado:** la tasa de defectos que llegan a cliente cae de forma sostenida; los inspectores pasan de mirar todo a revisar solo el 8% dudoso; y el mismo esqueleto (backbone congelado + cabeza nueva) se reutiliza para tres líneas de producto más. La lección ejecutiva: **casi nadie entrena desde cero — se adapta lo pre-entrenado**.

---

## 1. Arquitectura Base: el Perceptrón Multicapa (MLP)

Audiencia: 🔧 🧭

**🔧 Definición técnica — las seis piezas del entrenamiento:**

1. **Neurona (unidad):** recibe entradas x₁…xₙ con pesos w₁…wₙ y sesgo b; calcula la preactivación `z = Σwᵢxᵢ + b` y aplica la activación `a = f(z)`.
2. **Capa densa (fully connected):** M neuronas conectadas a toda la capa anterior: `salida = f(Wx + b)` con W de M×N — un producto matricial ([[02-Fundamentos-Matematicos]]).
3. **Forward pass:** propagar la entrada capa a capa hasta la salida = la predicción.
4. **Función de pérdida:** cuantifica el error: cross-entropy (clasificación binaria/multiclase), MSE/MAE (regresión) ([[08-Metricas-de-Evaluacion]]).
5. **Backpropagation:** la regla de la cadena aplicada hacia atrás calcula `∂L/∂w` para **cada** peso ([[02-Fundamentos-Matematicos]]); automatizado por autodiferenciación (PyTorch Autograd, TF GradientTape).
6. **Actualización de pesos:** el optimizer aplica `w ← w − α·∂L/∂w` con learning rate α (sección 3).

```
  x₁ ─w₁─┐
  x₂ ─w₂─┤  z = Σwᵢxᵢ + b   a = f(z)
  x₃ ─w₃─┼──────► [Σ│f] ──────► salida hacia la próxima capa
   ...   │
  xₙ ─wₙ─┘        forward ──►
                  ◄── backward (la culpa del error, repartida por la regla de la cadena)
```

**👔 En una frase para el negocio:** el MLP es el "hola mundo" de las redes — en tabular rara vez vence al boosting, pero es el bloque con que se arman todas las arquitecturas que sí cambian el juego.

---

## 2. Funciones de Activación

Audiencia: 🔧 🧭

> [!info] 📌 Descriptor
> La activación decide si una neurona "se enciende" y cuánto. **Sin ella, apilar capas daría una sola regresión lineal gigante**: la no-linealidad es lo que permite capturar patrones complejos.

| Función | Fórmula | Rango | Ventajas | Problemas | Uso típico | 💡 Analogía |
|---|---|---|---|---|---|---|
| Sigmoid | `1/(1+e⁻ˣ)` | (0,1) | Salida interpretable como probabilidad | Vanishing gradient si │x│>3; no centrada en 0 | SOLO capa de salida binaria | Un dimmer que se satura en los extremos |
| Tanh | `(eˣ−e⁻ˣ)/(eˣ+e⁻ˣ)` | (−1,1) | Centrada en 0; gradiente más fuerte que sigmoid | Vanishing si │x│>2 | RNN legacy; redes pequeñas | El dimmer simétrico: apaga hacia ambos lados |
| ReLU | `max(0, x)` | [0,∞) | Baratísima; sin vanishing para x>0; activaciones sparse | Dying ReLU (neuronas muertas en 0); no acotada | **Default en capas ocultas** (CNN, MLP) | El portero: deja pasar lo positivo, bloquea lo negativo |
| Leaky ReLU | `max(αx, x)`, α=0.01 | (−∞,∞) | Evita dying ReLU (gradiente pequeño en x<0) | α es un hiperparámetro más | Cuando hay neuronas muertas | El portero que deja pasar un hilito de lo negativo |
| PReLU | `max(αx, x)`, α aprendible | (−∞,∞) | α se optimiza durante el training | Un parámetro extra por neurona; riesgo de overfit | CNN donde vale la pena afinar | El portero que aprende cuánto hilito dejar pasar |
| ELU | `x` si x>0; `α(eˣ−1)` si x≤0 | (−α,∞) | Suave en 0; medias de activación cercanas a 0 | Exponencial = más costo | Redes profundas buscando media ≈ 0 | La rampa suave en vez del escalón |
| SELU | `λ·ELU` | (−λα,∞) | **Auto-normalización**: capas que se mantienen normalizadas solas | Exige init lecun_normal y sin otras normalizaciones | Feedforward profundas sin BatchNorm | La activación con termostato incorporado |
| GELU | `x·Φ(x)` (CDF Normal) | (−0.17,∞) | Empíricamente superior en Transformers | Más costosa que ReLU | BERT, GPT, ViT — el default Transformer | La ReLU probabilística: deja pasar según cuán "seguro" es x |
| Swish | `x·sigmoid(βx)` | (−∞,∞) | Suave, no monótona, auto-gate | β fijo (SiLU) o aprendible | EfficientNet, visión moderna | La compuerta que se modera a sí misma |
| Softmax | `eˣᵢ/Σeˣⱼ` | (0,1), suma 1 | Distribuye probabilidad entre K clases | Saturación con clase dominante; usar log-sum-exp | ÚNICA salida multiclase | La votación: cada clase recibe su porción del 100% |
| Mish | `x·tanh(softplus(x))` | (−0.31,∞) | Muy suave; buenos resultados en detección | Costosa | YOLOv4 y derivados | La Swish pulida |

---

## 3. Optimizadores

Audiencia: 🔧 🧭

> [!info] 📌 Descriptor
> El optimizador decide **cómo** usar los gradientes del backprop para actualizar los pesos: la estrategia para bajar la montaña de la pérdida ([[02-Fundamentos-Matematicos]]).

| Optimizador | Mecanismo matemático | Ventajas | Limitaciones | Cuándo usar | 💡 Analogía |
|---|---|---|---|---|---|
| SGD + Momentum | `v ← βv − α∇L; w ← w + v` — acumula velocidad de pasos previos | Simple; con buen schedule, la mejor generalización en visión | Sensible al learning rate; requiere schedule manual | Fine-tuning de visión con cosine annealing | El esquiador con inercia: las colinas chicas no lo frenan |
| RMSProp | `v ← ρv + (1−ρ)∇L²; w ← w − α∇L/√(v+ε)` — divide por la magnitud reciente del gradiente | Bueno en gradientes no estacionarios (RNN) | Sin corrección de sesgo | RNN/LSTM, series de tiempo | El corredor que acorta el paso en terreno pedregoso |
| Adam | Momentum (m) + RMSProp (v) + corrección de sesgo: `w ← w − α·m̂/√(v̂+ε)` (Kingma & Ba, 2015) | Robusto; defaults que funcionan (α=0.001, β₁=0.9, β₂=0.999) | Puede generalizar peor que SGD en visión | **El default general**: NLP, tabular, prototipos | El mejor de ambos mundos: inercia + paso adaptativo |
| AdamW | Adam con weight decay **desacoplado**: `w ← w(1−λ) − α·m̂/√(v̂+ε)` | Regularización L2 que sí funciona con adaptativos | — | **Transformers y fine-tuning de LLMs** | Adam con el freno de mano bien conectado |
| Adagrad | Acumula TODOS los gradientes²: el lr efectivo decae monótonamente | Bueno para features sparse | El lr muere con el tiempo — hoy casi no se usa | Histórico: NLP sparse pre-Adam | El caminante que se cansa y nunca se recupera |
| Adadelta | Adagrad con ventana móvil (no acumula todo) | No requiere fijar α | Convergencia más lenta que Adam | Cuando tunear lr es un problema | El caminante que sí recupera el aliento |
| LARS / LAMB | Learning rate adaptado **por capa** según ‖w‖/‖∇L‖ | Permite batch sizes gigantes (16K–32K) distribuidos | Complejo; solo paga con batches enormes | Pre-training de LLMs multi-GPU | Cada vagón del tren con su propio acelerador |

---

## 4. Learning Rate Scheduling

Audiencia: 🔧 🧭

> [!info] 📌 Descriptor: ¿qué es el learning rate?
> El learning rate (α) controla el **tamaño de cada paso** del optimizador. Muy grande: saltas sobre el valle y nunca converges. Muy chico: el entrenamiento es eterno. El **scheduling** lo varía durante el entrenamiento: pasos grandes al inicio para avanzar, pequeños al final para afinar — como estacionar: llegas rápido y maniobras lento.

| Schedule | Mecanismo | Cuándo conviene |
|---|---|---|
| Step Decay (StepLR) | lr × γ cada `step_size` épocas (ej. ×0.1 cada 30) | Simple y probado; visión clásica |
| Exponential Decay | `lr × γ^época` — decaimiento suave continuo | Alternativa continua al step |
| Cosine Annealing | lr sigue un coseno de lr_max a lr_min; con **warm restarts** el ciclo se reinicia y permite escapar de mínimos locales | Visión moderna; combina con Snapshot Ensembles ([[11-Mejora-de-Modelos]]) |
| OneCycle | Sube de lr_max/25 a lr_max y decae a lr_max/25000; momentum inverso | Convergencia rápida con pocas épocas (FastAI) |
| Warmup + Cosine | Lineal de 0 a lr_max los primeros pasos, luego coseno | **Estándar en Transformers**: el warmup estabiliza el arranque |
| ReduceLROnPlateau | Reduce lr cuando la métrica de validación se estanca (`patience`) | Adaptativo y prudente; el default sin opiniones |
| Cyclic LR (CLR) | Cicla entre lr_min y lr_max (triangular, exp_range) | Explorar; escapar de mínimos locales |

---

## 5. Inicialización de Pesos

Audiencia: 🔧

> [!info] 📌 Descriptor y objetivo
> Antes de entrenar, los pesos necesitan valores iniciales. **Todos en cero es fatal**: todas las neuronas de una capa reciben el mismo gradiente, aprenden lo mismo y la red colapsa en una sola neurona repetida (simetría no rota). Muy grandes o muy chicos: las señales explotan o se desvanecen capa a capa. El objetivo de una buena inicialización: **mantener la varianza de activaciones y gradientes estable a través de las capas** desde el primer paso.

| Método | Descripción | Cuándo usar |
|---|---|---|
| Xavier / Glorot | `W ~ U(±√(6/(n_in+n_out)))` — varianza constante entre capas | Sigmoid, Tanh; default de Keras en densas |
| He / Kaiming | `W ~ N(0, √(2/n_in))` — Xavier ajustado para la mitad muerta de ReLU | ReLU y familia; default de PyTorch en conv |
| LeCun | `W ~ N(0, √(1/n_in))` | SELU (requisito de la auto-normalización) |
| Ortogonal | Matriz ortogonal aleatoria | RNN/LSTM: preserva la norma de la señal en el tiempo |
| Ceros / constante | Todos iguales | **NUNCA para pesos** (simetría); solo para sesgos (0 o 0.01) |

---

## 6. Arquitecturas Principales

Audiencia: 🔧 🧭 👔

> [!tip] 💡 ¿Qué procesa cada arquitectura?
> **MLP:** tablas numéricas → predicciones (en tabular, XGBoost suele ganarle, [[07-Modelos-Supervisados]]). **CNN:** imágenes (píxeles H×W×C) → clasificación, detección, segmentación. **RNN/LSTM:** secuencias paso a paso → series de tiempo, texto secuencial (superadas por Transformers en NLP desde ~2018). **Transformer:** cualquier secuencia en paralelo → GPT (texto), ViT (imágenes), AlphaFold (proteínas) — el estado del arte en casi todo.

### 6.1 CNN — Redes Convolucionales

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Una linterna pequeña que recorre la foto: en cada posición ilumina un parche y pregunta "¿hay un borde aquí? ¿una esquina?". La **misma** linterna (mismos pesos) recorre toda la imagen — por eso detecta el gato esté donde esté. Capas sucesivas usan linternas que buscan patrones cada vez más complejos: bordes → texturas → orejas → gato.

**🔧 Definición técnica:** la **capa convolucional** aplica K filtros F×F sobre el mapa de entrada (parámetros: `out_channels`, `kernel_size`, `stride`, `padding`), produciendo K mapas de características. Tres propiedades fundamentales: **locality** (cada neurona ve solo su receptive field), **weight sharing** (el mismo filtro para toda la imagen → órdenes de magnitud menos parámetros que una densa) y **translation invariance**. **Pooling:** Max (conserva lo prominente), Average (suaviza), **GAP** (Global Average Pooling: H×W×C → vector C; reemplaza densas finales en arquitecturas modernas). **BatchNorm en CNN:** normaliza por canal sobre (N,H,W), típicamente después de la conv y antes de la activación ([[11-Mejora-de-Modelos]]). **Historia esencial:** LeNet (1998, LeCun et al.) → AlexNet (2012, arranca la era DL) → VGG (2014, profundidad con kernels 3×3) → **ResNet** (2015, He et al.) → EfficientNet (2019, scaling balanceado) → ViT (2020, atención en visión, Dosovitskiy et al., 2021). **Skip connections (ResNet):** `h(x) = F(x) + x` — si F tiende a 0 la capa aprende la identidad; resuelve el vanishing gradient y habilita redes de 100+ capas.

**🧭 Cuándo usarla:** imágenes y señales con estructura espacial; en la práctica, casi siempre vía transfer learning (sección 7). **👔 En una frase:** los ojos artificiales del negocio — inspección, conteo, lectura de documentos.

### 6.2 RNN, LSTM y GRU

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Leer una frase palabra por palabra con una **libreta de apuntes**: en cada palabra decides qué anotar y qué borrar de la libreta (el estado). La RNN simple tiene mala memoria (los apuntes viejos se borronean — vanishing); la **LSTM** agrega una libreta de largo plazo con tres porteros: uno decide qué **olvidar**, otro qué **anotar**, otro qué **mostrar**. La GRU es la versión con dos porteros: más simple, casi igual de buena.

**🔧 Definición técnica:** RNN: `hₜ = f(W_hh·hₜ₋₁ + W_xh·xₜ + b)` — la misma red aplicada en cada paso temporal. **Vanishing/exploding gradient:** el gradiente se multiplica por W_hh en cada paso: con ‖W‖<1 desaparece, con ‖W‖>1 explota (gradient clipping como parche). **LSTM** (Hochreiter & Schmidhuber, 1997): cell state Cₜ de largo plazo gobernado por **forget gate** (qué olvidar), **input gate + candidato** (qué agregar) y **output gate** (qué exponer como hₜ); los gates usan sigmoid (0–1 = apertura). Maneja dependencias de cientos a ~1000 pasos. **GRU:** 2 gates (reset, update), estado único, menos parámetros, rendimiento comparable, más rápida. **Bidireccionales:** procesan la secuencia en ambos sentidos y concatenan contexto pasado+futuro.

**🧭 Cuándo usarlas:** series de tiempo de tamaño moderado, sensores, secuencias donde un Transformer es sobredimensionado. En NLP, los Transformers las desplazaron. **👔 En una frase:** la memoria artificial para datos que llegan en orden — útil aún donde lo gigante no se justifica.

### 6.3 Transformers y el Mecanismo de Atención

Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogías del Transformer
> **Self-attention:** cada palabra de la oración les pregunta a todas las demás "¿cuánto me importas para entender mi significado?". Así, "ella" en *"el gato pisó la alfombra porque ella era suave"* aprende a mirar a "alfombra", no a "gato". **Encoder-only (BERT):** un lector que ve TODA la oración de una vez para entenderla (clasificar, buscar). **Decoder-only (GPT):** un escritor que genera palabra por palabra sin espiar el futuro. **Encoder-decoder (T5):** un traductor que primero entiende todo y luego reescribe.

**🔧 Definición técnica:** (Vaswani et al., 2017). Motivación: las RNN son secuenciales (no paralelizables) y pierden contexto largo; el Transformer procesa toda la secuencia **en paralelo** con atención. **Scaled dot-product attention:** con Q=XW_Q, K=XW_K, V=XW_V: `Attention(Q,K,V) = softmax(QKᵀ/√d_k)·V` — el factor 1/√d_k evita que el producto punto sature el softmax. **Multi-head:** h atenciones paralelas con proyecciones distintas (cada head captura relaciones diferentes: sintaxis, correferencia…), concatenadas y proyectadas. **Positional encoding:** sin recurrencia no hay noción de orden → se suman codificaciones de posición (senos/cosenos a distintas frecuencias, o embeddings aprendidos). **Bloque completo:** Multi-Head Attention → Add&Norm (residual + LayerNorm) → Feed-Forward (2 densas con GELU) → Add&Norm; los residuales permiten apilar decenas de bloques. **Variantes:** encoder-only (BERT: representaciones bidireccionales para clasificación/NER/QA), decoder-only (GPT: generación autoregresiva), encoder-decoder (T5/BART: traducción, resumen).

```
 BLOQUE TRANSFORMER
 entrada + positional encoding
        │
        ▼
 ┌─ Multi-Head Self-Attention ─┐
 │  softmax(QKᵀ/√d_k)·V ×h     │──► Add & Norm (residual)
 └──────────────────────────────┘        │
                                         ▼
                       ┌─ Feed-Forward (GELU) ─┐──► Add & Norm ──► siguiente bloque
                       └───────────────────────┘
```

**🧭 Cuándo usarlo:** todo NLP moderno, visión (ViT), audio, multimodal, biología — vía modelos pre-entrenados de Hugging Face; entrenar uno desde cero es territorio de laboratorios. **👔 En una frase:** la arquitectura que convirtió el lenguaje en un problema resuelto a nivel comercial — y el motor de los LLMs que hoy redefinen procesos completos.

### 6.4 Autoencoders, VAE, GAN y GNN

Audiencia: 🔧 🧭

- **Autoencoder:** encoder comprime → cuello de botella (latent space) → decoder reconstruye; aprende la esencia de los datos. Usos: reducción de dimensionalidad no lineal ([[03-Preparacion-de-Datos]]), denoising, detección de anomalías por error de reconstrucción ([[07-Modelos-Supervisados]]). 💡 *Resumir el capítulo en una ficha y re-explicarlo desde la ficha.*
- **VAE (Variational Autoencoder):** (Kingma & Welling, 2014) el espacio latente se vuelve una **distribución** continua (μ, σ + reparameterization trick) → se puede muestrear y generar datos nuevos coherentes. 💡 *En vez de una ficha exacta, guardas "la receta con tolerancias" — y puedes cocinar variaciones.*
- **GAN (Generative Adversarial Network):** (Goodfellow et al., 2014) dos redes en duelo: el **generador** falsifica, el **discriminador** detecta falsificaciones; compitiendo, ambos mejoran hasta que las falsificaciones son indistinguibles. Usos: imágenes sintéticas, super-resolución, data augmentation. Entrenamiento notoriamente inestable (mode collapse). 💡 *El falsificador y el perito: la carrera armamentista que perfecciona al falsificador.*
- **GNN (Graph Neural Network):** opera sobre grafos vía **message passing**: cada nodo agrega información de sus vecinos en rondas sucesivas. Usos: fraude en redes de transacciones, recomendación, moléculas/proteínas, logística. 💡 *Cada persona actualiza su opinión escuchando a sus amigos, ronda tras ronda — al final, la red entera "sabe".*

---

## 7. Transfer Learning y Fine-tuning

Audiencia: 🔧 🧭 👔

> [!info] 📌 Descriptor: qué es y dónde ocurre en el flujo
> **Transfer learning** = reutilizar un modelo pre-entrenado con millones de datos como punto de partida para tu problema con pocos datos. **Fine-tuning** = ajustar sus pesos a tu dominio. Ocurre DESPUÉS de elegir arquitectura y ANTES del deployment: eliges un backbone pre-entrenado, decides cuánto congelar, entrenas lo demás con learning rate pequeño.

> [!tip] 💡 Analogía del chef
> Un chef francés que aprende cocina japonesa no parte de cero: ya sabe cortar, emulsionar, controlar el fuego (pre-training). Solo necesita aprender técnicas y sabores japoneses específicos (fine-tuning). Infinitamente más rápido que formar desde cero a alguien que jamás pisó una cocina.

| Estrategia | Descripción | Cuándo usar | Trade-off |
|---|---|---|---|
| Feature extraction | Congela TODO el modelo base; entrena solo el clasificador superior | Dataset chico (<1K); dominio similar al pre-training | Rápido y estable; techo limitado si el dominio difiere |
| Fine-tuning parcial | Descongela las últimas K capas; lr pequeño (1e-5 a 1e-4) | Dataset mediano; dominio algo distinto | El equilibrio estándar |
| Fine-tuning completo | Descongela todo; lr mínimo (1e-6 a 1e-5) para no borrar lo aprendido (catastrophic forgetting) | Dataset grande; dominio muy distinto | Máxima adaptación; máximo costo y riesgo |
| Layer-wise LR decay | lr más chico en capas profundas, más grande arriba | Fine-tuning de Transformers grandes | Mejor retención del conocimiento base |
| LoRA (Low-Rank Adaptation) | Congela W originales; entrena solo matrices de bajo rango ΔW = A·B con r ≪ d (Hu et al., 2021) | LLMs de miles de millones de parámetros; ~0.1% de parámetros entrenables | Levemente bajo el fine-tuning completo en algunos tasks; revolucionario en costo |
| Prompt / prefix tuning | Congela TODO; solo entrena tokens/prefijos aprendibles en la entrada o capas | LLMs enormes donde hasta LoRA pesa | El más barato; el menos expresivo |

**👔 En una frase para el negocio:** el pre-entrenado + adaptación barata (LoRA) cambió la economía del deep learning — lo que costaba un datacenter hoy cuesta una GPU y un fin de semana.

---

## 8. Frameworks y Ecosistema

Audiencia: 🔧 🧭

| Framework | Paradigma | Fuerte en | Diferenciador técnico | Cuándo elegir |
|---|---|---|---|---|
| Scikit-Learn | ML clásico, API fit/predict | Todo lo NO profundo + pipelines | El estándar de tabular: Pipeline, ColumnTransformer, métricas ([[13-MLOps-XAI-Etica]]) | Baselines, tabular, preprocesamiento — antes de pensar en DL |
| PyTorch | Eager (define-by-run) | Investigación, flexibilidad | Autograd dinámico (debug con print/pdb); ~80% de los papers; torch.compile() para producción | Modelos custom, investigación, control total |
| TensorFlow 2 / Keras | Eager + Graph (tf.function) | Producción industrial | TFServing, TFLite (mobile/edge), TF.js (browser), XLA, TPU nativo; Keras como API de alto nivel | Deployment a escala, mobile/edge, ecosistema Google |
| JAX | Funcional compilado (XLA) | Investigación de alto rendimiento | jit, vmap, grad, pmap; puro y sin side effects | Vanguardia, TPUs, eficiencia extrema |
| Hugging Face Transformers | API unificada sobre PyTorch/TF/JAX | NLP y multimodal pre-entrenado | +100K modelos en el Hub; AutoModel/AutoTokenizer; Trainer; PEFT (LoRA) | Transfer learning de lenguaje/visión: el punto de partida por defecto |
| FastAI | Alto nivel sobre PyTorch | Prototipado veloz, educación | OneCycle, discriminative LRs, defaults excelentes | Resultados fuertes en días, no semanas |
| Lightning | PyTorch sin boilerplate | Proyectos serios en PyTorch | Elimina el loop manual; multi-GPU/TPU sin cambiar código; logging integrado | Reproducibilidad y escala con PyTorch |
| ONNX | Formato de intercambio | Deployment cross-framework | Exporta de PyTorch/TF y ejecuta en ONNX Runtime/TensorRT/OpenVINO/CoreML | Inferencia máxima velocidad, portabilidad ([[13-MLOps-XAI-Etica]]) |

---

## 📖 Referencias de este tomo

- (Goodfellow et al., 2016) — el texto de referencia del campo.
- (Vaswani et al., 2017) — "Attention Is All You Need": el Transformer.
- (Hochreiter & Schmidhuber, 1997) — LSTM. · (LeCun et al., 1998) — LeNet. · (He et al., 2016) — ResNet. · (Dosovitskiy et al., 2021) — ViT.
- (Kingma & Ba, 2015) — Adam. · (Ioffe & Szegedy, 2015) — BatchNorm. · (Srivastava et al., 2014) — Dropout.
- (Goodfellow et al., 2014) — GAN. · (Kingma & Welling, 2014) — VAE. · (Hu et al., 2021) — LoRA.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[11-Mejora-de-Modelos|11 · Mejora de Modelos]] · Siguiente: [[13-MLOps-XAI-Etica|13 · MLOps, XAI y Ética ➡]]

> **Próximo tomo:** [[13-MLOps-XAI-Etica]] — lo que separa el experimento del sistema: interpretabilidad, reproducibilidad, deployment, monitoreo de drift, fairness y el decálogo de mejores prácticas.
