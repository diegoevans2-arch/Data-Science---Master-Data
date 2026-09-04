---
title: "Tomo 25 — Privacidad y Datos Sintéticos"
tags:
  - privacidad
  - differential-privacy
  - federated-learning
  - datos-sinteticos
  - CTGAN
  - TabDDPM
  - GDPR
  - synthetic-data
audiencias:
  - tecnico
  - puente
  - ejecutivo
tomo: 25
version: 1.0
updated: 2026-08-28
---

# Tomo 25 — Privacidad y Datos Sintéticos

> Navegación: ← [[24-Experimentacion-AB]]

---

## 1. ¿Por qué la privacidad importa para Data Science?

👔 [Ejecutivo] Los datos son el combustible de los modelos, pero cada registro puede representar a una persona real. Regulaciones globales imponen obligaciones legales y multas millonarias; más allá del compliance, proteger la privacidad es un requisito ético y un habilitador de confianza.

💡 [Analogía] Imagina que tus datos son radiografías médicas: tienen enorme valor diagnóstico, pero mostrarlas sin consentimiento viola la intimidad del paciente. La privacidad en DS busca extraer el "diagnóstico" (el patrón estadístico) sin exponer la "radiografía" (el dato individual).

### 1.1 Panorama regulatorio

| Regulación | Jurisdicción | Aspecto clave para DS |
|---|---|---|
| **GDPR** (2018) | UE / EEA | Minimización de datos, derecho al olvido, bases legales para profiling |
| **EU AI Act** (2024) | UE | Clasificación por riesgo; sistemas de alto riesgo requieren datos representativos y auditables |
| **CCPA / CPRA** | California, EE.UU. | Opt-out de venta de datos, derecho a conocer inferencias |
| **LGPD** | Brasil | Consentimiento explícito, datos sensibles con protección reforzada |
| **Ley Marco de IA** (en discusión) | Chile, Colombia, México | Transparencia algorítmica, evaluación de impacto |

🧭 [Puente] La conexión con fairness ([[13-Fairness-Sesgo-Modelos]]) es directa: datos desbalanceados o censurados por privacidad pueden amplificar sesgos si no se gestionan de forma conjunta.

> [!warning] Compliance ≠ Ética
> Cumplir el GDPR no garantiza que tu modelo sea justo. Un modelo puede ser legal pero discriminatorio si los datos anonimizados preservan correlaciones proxy con atributos protegidos.

### 1.2 Amenazas concretas en pipelines de ML

```
┌─────────────────────────────────────────────────────────┐
│              AMENAZAS DE PRIVACIDAD EN ML               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Entrenamiento         Modelo           Inferencia      │
│  ─────────────         ──────           ──────────      │
│  • Datos en claro      • Membership     • Model         │
│    en storage            inference         inversion    │
│  • Logging excesivo      attack         • Attribute     │
│  • Feature stores      • Memorización     inference     │
│    sin ACL               de outliers    • Extraction    │
│                                           attack        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Differential Privacy (DP)

🔧 [Técnico] Differential privacy es el estándar formal para cuantificar la pérdida de privacidad. Proporciona una garantía matemática de que la presencia o ausencia de un individuo en el dataset no cambia significativamente la distribución de salidas de un mecanismo.

### 2.1 Definición formal

Un mecanismo aleatorizado **M** satisface **(ε, δ)-differential privacy** si para todo par de datasets adyacentes D y D' (que difieren en un solo registro) y para todo subconjunto de salidas S:

```
P[M(D) ∈ S] ≤ e^ε · P[M(D') ∈ S] + δ
```

- **ε (epsilon)**: presupuesto de privacidad. Menor ε → mayor privacidad, menor utilidad.
- **δ (delta)**: probabilidad de fallo catastrófico. Idealmente δ < 1/n² donde n es el tamaño del dataset.

💡 [Analogía] Epsilon es como el volumen de un micrófono espía: con ε = 0, el micrófono está apagado (privacidad perfecta, pero no escuchas nada útil). Con ε → ∞, el micrófono amplifica todo (cero privacidad, máxima utilidad).

### 2.2 Mecanismos fundamentales

| Mecanismo | Tipo de query | Ruido añadido | Distribución |
|---|---|---|---|
| **Laplace** | Numéricas (conteo, suma) | Proporcional a sensibilidad/ε | Laplace(0, Δf/ε) |
| **Gaussian** | Numéricas (requiere (ε,δ)-DP) | σ calibrado a sensibilidad | N(0, σ²) con σ ∝ Δf·√(2ln(1.25/δ))/ε |
| **Exponential** | Categóricas / selección | Score function + muestreo | P[salida r] ∝ exp(ε·u(D,r) / 2Δu) |

🔧 [Técnico] La **sensibilidad** (Δf) mide cuánto puede cambiar la respuesta de una query al añadir/remover un registro. Para una query de conteo, Δf = 1. Para una suma sobre valores en [0, B], Δf = B.

### 2.3 Composición

```
┌────────────────────────────────────────────────────────────┐
│           TEOREMAS DE COMPOSICIÓN                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Composición básica:     k mecanismos ε-DP                 │
│                          → (k·ε)-DP total                  │
│                                                            │
│  Composición avanzada:   k mecanismos ε-DP                 │
│                          → (ε√(2k·ln(1/δ)) + kε(e^ε-1),    │
│                             δ)-DP                          │
│                                                            │
│  Rényi DP (RDP):         Composición lineal en α           │
│                          → conversión final a (ε,δ)-DP     │
│                          (más tight para SGD iterativo)    │
│                                                            │
│  Moments Accountant:     Usado por DP-SGD (Abadi et al.)   │
│                          → tracking preciso por epoch      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 2.4 Trade-off privacidad–utilidad

🧭 [Puente] No existe almuerzo gratis: proteger más implica degradar señal estadística.

| ε típico | Interpretación | Caso de uso |
|---|---|---|
| 0.1 – 1.0 | Privacidad fuerte | Censos gubernamentales (US Census 2020 usó ε ≈ 19.6 para las tablas más detalladas) |
| 1.0 – 10.0 | Privacidad moderada | Modelos internos de ML con datos sensibles |
| > 10.0 | Privacidad débil | Analítica exploratoria, desarrollo |

> [!tip] Regla práctica para comunicar a negocio
> "Con ε = 1, un atacante que vea la salida del modelo no puede distinguir si tú estuviste en los datos de entrenamiento con más del 63% de confianza vs. 50% de base (ratio de odds ≈ e¹ ≈ 2.7x)."

---

## 3. Federated Learning

🔧 [Técnico] Federated learning (FL) permite entrenar modelos de ML sobre datos distribuidos en múltiples dispositivos o instituciones sin centralizar los datos crudos. Solo se intercambian actualizaciones de modelo (gradientes o pesos).

### 3.1 Arquitectura general

```
┌──────────────────────────────────────────────────────────────┐
│                  FEDERATED LEARNING                          │
│                                                              │
│    ┌──────────┐     ┌──────────┐     ┌──────────┐            │
│    │ Cliente 1│     │ Cliente 2│     │ Cliente k│            │
│    │  Datos   │     │  Datos   │     │  Datos   │            │
│    │  locales │     │  locales │     │  locales │            │
│    └────┬─────┘     └────┬─────┘     └────┬─────┘            │
│         │                │                │                  │
│         │  Δw₁           │  Δw₂           │  Δwₖ              │
│         ▼                ▼                ▼                  │
│    ┌─────────────────────────────────────────────┐           │
│    │           SERVIDOR AGREGADOR                │           │
│    │                                             │           │
│    │   w(t+1) = w(t) + (1/k) · Σ Δwᵢ             │           │
│    │   (FedAvg: promedio ponderado por n_i)      │           │
│    └─────────────────────────────────────────────┘           │
│         │                                                    │
│         │  w(t+1) broadcast                                  │
│         ▼                                                    │
│    Todos los clientes reciben modelo actualizado             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Algoritmos principales

| Algoritmo | Idea central | Ventaja | Limitación |
|---|---|---|---|
| **FedAvg** (McMahan et al., 2017) | Cada cliente hace E epochs locales, servidor promedia pesos | Reduce comunicación vs. SGD distribuido | Diverge con datos muy non-IID |
| **FedSGD** | Un solo paso de gradiente local por ronda | Convergencia estable | Alto costo de comunicación |
| **FedProx** (Li et al., 2020) | Añade término de proximidad al objetivo local | Tolera heterogeneidad | Hiperparámetro μ adicional |
| **SCAFFOLD** (Karimireddy et al., 2020) | Corrección de varianza con control variates | Mejor convergencia teórica | Mayor complejidad de implementación |

### 3.3 Heterogeneidad de datos (non-IID)

🧭 [Puente] En la práctica, los datos de cada cliente no son una muestra i.i.d. del mismo proceso generador. Un hospital rural tiene distribución de patologías diferente a un hospital universitario.

**Tipos de heterogeneidad:**

- **Label skew**: cliente A solo tiene clase 0 y 1; cliente B tiene clases 2, 3, 4.
- **Feature skew**: mismas etiquetas pero features con distribuciones distintas (ej. imágenes de distinta calidad).
- **Quantity skew**: cliente A tiene 100K muestras, cliente B tiene 500.

> [!note] Impacto en la práctica
> Con non-IID severo, FedAvg puede converger a un modelo que es peor que entrenar solo con los datos del cliente más grande. Estrategias de mitigación: data sharing parcial, personalización local (fine-tuning post-federación), o clustering de clientes.

### 3.4 Differential Privacy + Federated Learning

```
┌───────────────────────────────────────────────────────┐
│         DP-FedAvg (combinación DP + FL)               │
├───────────────────────────────────────────────────────┤
│                                                       │
│  1. Cliente i computa gradiente local: g_i            │
│  2. Clip gradiente: ĝ_i = g_i · min(1, C/‖g_i‖)       │
│  3. Agrega ruido: ĝ_i + N(0, σ²C²I)                   │
│  4. Servidor agrega: (1/k)·Σ(ĝ_i + ruido)             │
│  5. Moments accountant trackea ε acumulado            │
│                                                       │
│  Resultado: modelo entrenado con garantía (ε,δ)-DP    │
│  respecto a cualquier registro individual             │
│                                                       │
└───────────────────────────────────────────────────────┘
```

🔧 [Técnico] El clipping (paso 2) es esencial: acotar la norma del gradiente acotar la sensibilidad, lo que permite calibrar el ruido gaussiano. Sin clipping, un solo outlier puede tener influencia ilimitada.

---

## 4. Datos Sintéticos — Generación

👔 [Ejecutivo] Los datos sintéticos son registros artificiales generados por un modelo que aprendió las propiedades estadísticas de datos reales. Permiten compartir, explorar y entrenar sin exponer información personal real.

### 4.1 CTGAN (Conditional Tabular GAN)

🔧 [Técnico] Propuesto por Xu et al. (2019) en el contexto de Synthetic Data Vault (SDV).

**Innovaciones clave:**

- **Mode-specific normalization**: para columnas continuas, ajusta una mezcla de gaussianas (VGM) y muestrea condicionando en el modo, evitando colapso en distribuciones multimodales.
- **Conditional generator**: el generador recibe una condición sobre una columna discreta, forzando balance en la generación (training-by-sampling).
- **PacGAN**: el discriminador ve paquetes de muestras para detectar mode collapse.

```
┌─────────────────────────────────────────────────┐
│              CTGAN ARCHITECTURE                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  Datos reales                                   │
│      │                                          │
│      ▼                                          │
│  Mode-specific normalization                    │
│      │                                          │
│      ▼                                          │
│  ┌──────────────┐      ┌──────────────────┐     │
│  │  Generator   │─────▶│  Discriminator   │     │
│  │  (MLP +      │      │  (PacGAN, ve     │     │
│  │   cond mask) │◀─────│   pacs de rows)  │     │
│  └──────────────┘ loss └──────────────────┘     │
│       │                                         │
│       ▼                                         │
│  Datos sintéticos                               │
│  (desnormalizados)                              │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 4.2 TabDDPM (Diffusion para datos tabulares)

🔧 [Técnico] Propuesto por Kotelnikov et al. (2023). Aplica modelos de difusión (denoising diffusion probabilistic models) a datos tabulares mixtos (continuos + categóricos).

**Diferencias respecto a GANs:**

| Aspecto | CTGAN (GAN) | TabDDPM (Diffusion) |
|---|---|---|
| Entrenamiento | Adversarial (inestable) | Denoising (estable, un solo objetivo) |
| Mode collapse | Riesgo alto | Riesgo bajo (cobertura completa) |
| Columnas categóricas | Embedding + softmax | Difusión multinomial dedicada |
| Calidad reportada | Buena en datasets pequeños | Superior en benchmarks (≥15 datasets) |
| Costo computacional | Moderado | Alto (muchos pasos de difusión) |

**Proceso de difusión para tabulares:**
```
┌──────────────────────────────────────────────────────┐
│              TabDDPM: FORWARD + REVERSE              │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Forward (destruir):                                 │
│  x₀ ──▶ x₁ ──▶ x₂ ──▶ ... ──▶ xT (ruido puro)        │
│     +ε₁    +ε₂    +ε₃                                │
│                                                      │
│  Reverse (reconstruir):                              │
│  xT ──▶ x̂(T-1) ──▶ ... ──▶ x̂₀ (dato sintético)       │
│     MLP predice ε en cada paso                       │
│                                                      │
│  Para categóricas: difusión sobre simplex            │
│  (transiciones de probabilidad entre categorías)     │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 4.3 Métodos basados en Copulas (SDV / Synthetic Data Vault)

🧭 [Puente] Enfoque más simple y rápido que deep learning. Modela distribuciones marginales por separado y luego captura dependencias entre columnas usando copulas gaussianas.

**Pipeline:**
1. Estimar distribución marginal de cada columna (parametric fit o KDE).
2. Transformar cada marginal a uniforme [0,1] via CDF.
3. Ajustar una copula gaussiana (correlación entre uniformes transformadas).
4. Para generar: muestrear de la copula → invertir CDF de cada marginal.

**Ventajas**: interpretable, rápido, funciona bien con datasets pequeños.
**Limitaciones**: no captura dependencias no lineales complejas ni interacciones de orden alto.

### 4.4 Evaluación de calidad de datos sintéticos

> [!important] Tres dimensiones de evaluación
> No basta con que "se vean bien". Los datos sintéticos deben evaluarse en tres ejes ortogonales: fidelidad, utilidad y privacidad.

| Dimensión | Métrica | Qué mide |
|---|---|---|
| **Fidelidad** | Column-wise stat similarity (KS test, TV distance) | ¿Las distribuciones marginales coinciden? |
| **Fidelidad** | Pairwise correlation difference | ¿Las dependencias entre columnas se preservan? |
| **Utilidad** | Train on Synthetic, Test on Real (TSTR) | ¿Un modelo entrenado en sintéticos generaliza a datos reales? |
| **Utilidad** | Feature importance ranking similarity | ¿Las señales predictivas se mantienen? |
| **Privacidad** | Distance to Closest Record (DCR) | ¿Los sintéticos están lejos de cualquier real? |
| **Privacidad** | Membership Inference Attack (MIA) | ¿Se puede determinar si un registro real fue usado para entrenar el generador? |
| **Privacidad** | Attribute inference | ¿Se pueden inferir atributos sensibles de registros reales usando sintéticos? |

```
┌──────────────────────────────────────────────────────────┐
│       EVALUACIÓN: TRIÁNGULO DE TRADE-OFFS                │
│                                                          │
│                    FIDELIDAD                             │
│                       ▲                                  │
│                      / \                                 │
│                     /   \                                │
│                    /     \                               │
│                   / SWEET  \                             │
│                  /  SPOT    \                            │
│                 /            \                           │
│                ▼              ▼                          │
│          UTILIDAD ◀────────▶ PRIVACIDAD                  │
│                                                          │
│  Máxima fidelidad = copia exacta = cero privacidad       │
│  Máxima privacidad = ruido puro = cero utilidad          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Cuándo usar datos sintéticos

👔 [Ejecutivo] Los datos sintéticos no reemplazan a los reales; los complementan en escenarios donde acceder a datos reales es costoso, lento, riesgoso o imposible.

### 5.1 Casos de uso principales

| Escenario | Problema | Cómo ayudan los sintéticos |
|---|---|---|
| **Data augmentation para clases raras** | Fraude: 0.1% de transacciones. Modelo sufre recall bajo | Generar sintéticos de la clase minoritaria con CTGAN condicionado. Complementa SMOTE con relaciones más complejas |
| **Sharing entre organizaciones** | Hospital A quiere colaborar con Hospital B sin exponer PHI | Cada hospital genera sintéticos locales; se comparten sin riesgo de re-identificación |
| **Testing de pipelines** | Equipo de ingeniería necesita datos realistas para CI/CD | Sintéticos con misma estructura y distribución; no requieren acceso a producción |
| **Pre-entrenamiento / transfer** | Dominio con pocos datos etiquetados | Pre-entrenar en sintéticos abundantes, fine-tune en reales escasos |
| **Democratización interna** | Solo 3 personas tienen acceso al dataset sensible | Generar versión sintética accesible para analistas exploratorios |

### 5.2 Flujo de decisión

```
┌────────────────────────────────────────────────────────────┐
│  ¿Necesitas datos que no puedes usar directamente?         │
│                         │                                  │
│                         ▼                                  │
│              ┌─── ¿Por qué no? ───┐                        │
│              │                     │                       │
│         Regulación/         Escasez/                       │
│         Privacidad          Desbalance                     │
│              │                     │                       │
│              ▼                     ▼                       │
│    Eval: ¿DP formal         Eval: ¿El generativo           │
│    es suficiente?           captura la señal               │
│         │     │             predictiva?                    │
│        Sí    No                   │                        │
│         │     │              TSTR test                     │
│         ▼     ▼                   │                        │
│      DP-SGD  Sintéticos     ┌─────┴─────┐                  │
│      noise   + eval DCR     │           │                  │
│              + MIA          TSTR≈TRTR   TSTR<<TRTR         │
│                              │           │                 │
│                          ✓ Usar      ✗ Mejorar             │
│                          sintéticos   generador            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 6. Limitaciones y riesgos

> [!danger] Los datos sintéticos no son una bala de plata
> Generarlos mal puede ser peor que no usarlos: pueden dar falsa confianza de privacidad mientras filtran información, o degradar modelos downstream por artefactos del generador.

### 6.1 Riesgos técnicos

| Riesgo | Descripción | Señales de alerta |
|---|---|---|
| **Mode collapse** | El generador solo produce un subconjunto de los modos reales | Baja diversidad; coverage metric < 0.5 |
| **Memorización** | Registros sintéticos son copias (o near-copies) de reales | DCR ≈ 0 para algunos registros; MIA AUC > 0.6 |
| **Sesgo amplificado** | El generador sobre-representa patrones mayoritarios | Disparate impact en sintéticos > que en reales |
| **Distribución shift** | Sintéticos capturan snapshot temporal; datos reales driftan | TSTR degrada con el tiempo; necesita re-generación periódica |
| **False sense of privacy** | Publicar sintéticos sin evaluar privacidad formalmente | No se reporta DCR ni se corre MIA; "son sintéticos, no pasa nada" |

### 6.2 Riesgos organizacionales

🧭 [Puente]

- **Governance gap**: ¿Quién es responsable de los sintéticos? ¿Se versionan? ¿Se documentan con datasheets?
- **Regulatory ambiguity**: el GDPR no menciona explícitamente datos sintéticos; la interpretación varía entre DPAs (Data Protection Authorities).
- **Over-reliance**: equipos que solo trabajan con sintéticos pierden intuición sobre artefactos de datos reales.

> [!warning] Memorización en modelos generativos
> Carlini et al. (2023) demostraron que diffusion models pueden memorizar y regurgitar datos de entrenamiento. Esto aplica también a generadores tabulares: si el dataset de entrenamiento es pequeño y el modelo es expresivo, la memorización es probable.

---

## 7. Guía de decisión: ¿Anonimizar, agregar DP, federar o sintetizar?

👔 [Ejecutivo] Cada técnica de privacidad tiene su lugar. La elección depende del contexto operativo, el nivel de riesgo y los recursos disponibles.

```
┌────────────────────────────────────────────────────────────────┐
│          ÁRBOL DE DECISIÓN DE PRIVACIDAD                       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ¿Los datos salen de la organización?                          │
│         │                    │                                 │
│        NO                   SÍ                                 │
│         │                    │                                 │
│         ▼                    ▼                                 │
│  ¿Necesitas garantía    ¿Las partes pueden                     │
│  formal de privacidad?   colaborar en infra?                   │
│     │          │              │          │                     │
│    NO         SÍ            SÍ         NO                      │
│     │          │              │          │                     │
│     ▼          ▼              ▼          ▼                     │
│  Pseudoni-   DP local      Federated   Datos                   │
│  mización    (DP-SGD,      Learning    Sintéticos              │
│  + controls  ruido en      (+ DP       (evaluar con            │
│  de acceso   queries)      opcional)   DCR + MIA)              │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 7.1 Comparación de enfoques

| Criterio | Anonimización clásica | Differential Privacy | Federated Learning | Datos Sintéticos |
|---|---|---|---|---|
| **Garantía formal** | No (vulnerable a linkage attacks) | Sí (ε, δ cuantificables) | Parcial (sin DP adicional, gradientes filtran info) | No formal (requiere evaluación empírica) |
| **Impacto en utilidad** | Variable (k-anonymity destruye granularidad) | Controlado por ε | Bajo si datos son IID | Depende de calidad del generador |
| **Complejidad de implementación** | Baja | Media-Alta | Alta (infra distribuida) | Media |
| **Escalabilidad** | Alta | Alta | Media (comunicación) | Alta (genera offline) |
| **Mejor para** | Datasets pequeños, bajo riesgo | Queries agregadas, ML con DP-SGD | Múltiples instituciones, datos in-situ | Compartir datasets, augmentation, testing |

> [!tip] Combinaciones habituales
> En la práctica, las técnicas se combinan:
> - **DP + FL**: entrenar federado con ruido DP en gradientes (gold standard para salud).
> - **Sintéticos + DP**: entrenar el generador con DP-SGD para tener garantía formal sobre los sintéticos.
> - **Anonimización + Sintéticos**: pseudonimizar y luego generar sintéticos como capa adicional.

---

## 8. Caso de negocio: Colaboración hospitalaria para predicción de readmisión

👔 [Ejecutivo]

### 8.1 Contexto

Tres hospitales (A, B, C) quieren desarrollar un modelo de predicción de readmisión a 30 días. Ninguno tiene suficientes datos individualmente para un modelo robusto. Compartir datos crudos viola regulaciones locales de protección de datos de salud.

### 8.2 Solución implementada

```
┌────────────────────────────────────────────────────────────┐
│            PIPELINE: PREDICCIÓN DE READMISIÓN              │
│            CON FL + DP + DATOS SINTÉTICOS                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Fase 1: Entrenamiento federado con DP                     │
│  ─────────────────────────────────────                     │
│  • FedAvg con 50 rondas, E=5 epochs locales                │
│  • DP-SGD con clipping C=1.0, σ=0.8                        │
│  • Budget total: ε=8.0, δ=1e-5                             │
│  • Resultado: modelo global con AUC 0.78                   │
│                                                            │
│  Fase 2: Generación de sintéticos para research            │
│  ─────────────────────────────────────────────             │
│  • Hospital A genera 50K registros con CTGAN               │
│  • Evaluación: TSTR AUC = 0.74 (vs. TRTR = 0.78)           │
│  • DCR mínimo = 3.2 (>umbral de 1.0)                       │
│  • MIA AUC = 0.52 (≈ aleatorio → no hay filtración)        │
│  • Dataset sintético compartido para investigación         │
│                                                            │
│  Fase 3: Monitoreo continuo                                │
│  ──────────────────────────                                │
│  • Re-entrenamiento federado trimestral                    │
│  • Re-generación de sintéticos con drift detection         │
│  • Auditoría de privacidad anual                           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 8.3 Resultados

| Métrica | Solo Hospital A | Modelo Federado (A+B+C) | Modelo en Sintéticos |
|---|---|---|---|
| AUC | 0.71 | 0.78 | 0.74 |
| Recall@10% | 0.35 | 0.48 | 0.42 |
| Privacidad | Sin garantía | ε = 8.0 formal | DCR > 3.0, MIA ≈ random |

🧭 [Puente] El modelo federado con DP logra +7 puntos de AUC vs. entrenamiento aislado, con garantías formales. Los sintéticos permiten que equipos de research externos analicen patrones sin acceso a datos reales, con pérdida moderada de utilidad (-4 AUC).

> [!success] Impacto de negocio
> - Reducción de readmisiones estimada: 12% → ahorro de ~$2.4M/año entre los tres hospitales.
> - Tiempo de aprobación regulatoria para compartir datos: de 8 meses (datos reales) a 3 semanas (sintéticos post-evaluación).
> - Nuevas líneas de investigación habilitadas por dataset sintético público.

---

## 9. Checklist de implementación

🔧 [Técnico]

### Para Differential Privacy:
- [ ] Definir threat model: ¿quién es el adversario? ¿Qué sabe?
- [ ] Establecer presupuesto ε con stakeholders (no solo técnicos)
- [ ] Elegir mecanismo según tipo de query/task
- [ ] Implementar composition accounting (RDP o moments accountant)
- [ ] Validar utilidad con ε elegido antes de desplegar
- [ ] Documentar: qué ε, qué δ, qué sensibilidad se asumió

### Para Federated Learning:
- [ ] Evaluar heterogeneidad de datos entre participantes
- [ ] Definir protocolo de comunicación y frecuencia de rondas
- [ ] Implementar secure aggregation si servidor no es trusted
- [ ] Decidir si agregar DP a los gradientes
- [ ] Planificar manejo de clientes intermitentes (stragglers)
- [ ] Establecer governance: quién posee el modelo global

### Para Datos Sintéticos:
- [ ] Elegir generador según tamaño y complejidad del dataset
- [ ] Evaluar las tres dimensiones: fidelidad, utilidad, privacidad
- [ ] Establecer umbrales de aceptación para DCR y MIA
- [ ] Documentar con datasheet: qué datos se usaron, qué generador, qué evaluación
- [ ] Si se busca garantía formal: entrenar generador con DP-SGD
- [ ] Re-evaluar periódicamente si distribución real cambia (drift)

---

## 10. Conexiones con otros tomos

| Tomo | Conexión |
|---|---|
| [[05-Feature-Engineering]] | Features derivadas pueden filtrar PII; aplicar DP antes de feature store |
| [[08-Gradient-Boosting]] | DP-GBT (gradient boosting con DP) como alternativa a DP-SGD para tabulares |
| [[13-Fairness-Sesgo-Modelos]] | Privacidad y fairness tienen tensiones: proteger atributos puede impedir auditar sesgo |
| [[17-MLOps-Produccion]] | Pipelines de datos sintéticos deben integrarse en CI/CD con validación automática |
| [[19-Deep-Learning-Fundamentos]] | DP-SGD es la técnica estándar para entrenar redes neuronales con DP |
| [[24-Experimentacion-AB]] | A/B tests con DP: agregar ruido a métricas de experimentos para proteger usuarios individuales |

---

## Referencias

1. **Dwork, C.** (2006). "Differential Privacy." *Proceedings of the 33rd International Colloquium on Automata, Languages and Programming (ICALP)*, Part II, pp. 1–12. Springer, LNCS 4052.

2. **Dwork, C., & Roth, A.** (2014). "The Algorithmic Foundations of Differential Privacy." *Foundations and Trends in Theoretical Computer Science*, 9(3–4), pp. 211–407.

3. **McMahan, B., Moore, E., Ramage, D., Hampson, S., & Arcas, B. A.** (2017). "Communication-Efficient Learning of Deep Networks from Decentralized Data." *Proceedings of the 20th International Conference on Artificial Intelligence and Statistics (AISTATS)*, pp. 1273–1282.

4. **Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., & Zhang, L.** (2016). "Deep Learning with Differential Privacy." *Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security (CCS)*, pp. 308–318.

5. **Xu, L., Skoularidou, M., Cuesta-Infante, A., & Veeramachaneni, K.** (2019). "Modeling Tabular Data using Conditional GAN." *Advances in Neural Information Processing Systems (NeurIPS)*, 32.

6. **Kotelnikov, A., Barber, D., & Nazaret, B.** (2023). "TabDDPM: Modelling Tabular Data with Diffusion Models." *Proceedings of the 40th International Conference on Machine Learning (ICML)*, PMLR 202, pp. 17564–17579.

7. **Patki, N., Wedge, R., & Veeramachaneni, K.** (2016). "The Synthetic Data Vault." *IEEE International Conference on Data Science and Advanced Analytics (DSAA)*, pp. 399–410.

8. **Carlini, N., Hayes, J., Nasr, M., Jagielski, M., Sehber, V., Tramèr, F., Balle, B., Ippolito, D., & Wallace, E.** (2023). "Extracting Training Data from Diffusion Models." *Proceedings of the 32nd USENIX Security Symposium*, pp. 5253–5270.

9. **Kairouz, P., McMahan, H. B., et al.** (2021). "Advances and Open Problems in Federated Learning." *Foundations and Trends in Machine Learning*, 14(1–2), pp. 1–210.

10. **Li, T., Sahu, A. K., Zaheer, M., Sanjabi, M., Talwalkar, A., & Smith, V.** (2020). "Federated Optimization in Heterogeneous Networks." *Proceedings of Machine Learning and Systems (MLSys)*, 2, pp. 429–450.

---

> Navegación: ← [[24-Experimentacion-AB]]
