---
title: "Tomo 24 — Experimentación A/B y Diseño de Experimentos"
tags:
  - experimentacion
  - ab-testing
  - causalidad
  - estadistica
  - diseño-experimental
audiencias:
  - tecnico
  - puente
  - ejecutivo
tomo: 24
version: 1.0
updated: 2026-08-28
---

# Tomo 24 — Experimentación A/B y Diseño de Experimentos

> [!abstract] Resumen ejecutivo
> Un A/B test es el gold standard para establecer causalidad en productos digitales: aleatorizamos usuarios entre variantes y medimos el efecto. Este tomo cubre desde el pre-requisito fundamental (¿puedo aleatorizar?) hasta diseños avanzados (sequential testing, interleaving, switchback), pasando por el dimensionamiento correcto del experimento y las trampas que invalidan conclusiones.

**Navegación:** ← [[23-Tabular-DL-vs-Boosting]] · Siguiente tomo pendiente

---

## 1. ¿Cuándo se puede aleatorizar?

👔 [Ejecutivo] El A/B test solo funciona cuando podemos asignar aleatoriamente a los usuarios entre tratamiento y control **sin restricciones éticas, legales o técnicas** que lo impidan.

🔧 [Técnico] El supuesto clave es la **exchangeability**: dado que la asignación es aleatoria, los grupos son comparables en expectativa sobre todas las variables — observadas y no observadas. Si no puedes aleatorizar, necesitas métodos quasi-experimentales (ver [[18-Causalidad-Inferencia-Causal]]).

> [!question] Pregunta de filtro antes de diseñar
> 1. ¿Puedo asignar aleatoriamente la intervención?
> 2. ¿Hay riesgo ético o legal en negar el tratamiento al control?
> 3. ¿Hay interferencia inevitable entre unidades (e.g., precios en un marketplace)?
>
> Si la respuesta a (1) es "no", o (2)/(3) son "sí" sin mitigación posible → quasi-experiment o diseño observacional.

💡 [Analogía] Un A/B test es como un ensayo clínico con placebo: si no puedes dar placebo (por ética) o si los pacientes comparten pastillas entre sí (interferencia), el diseño se rompe.

🧭 [Puente] La aleatorización es lo que distingue un A/B test de un análisis observacional. Sin ella, correlación ≠ causalidad. Con ella, la diferencia entre grupos **es** el efecto causal (en expectativa).

---

## 2. Diseño del experimento: dimensionamiento

### 2.1 Los cuatro parámetros fundamentales

🔧 [Técnico]

| Parámetro | Símbolo | Significado | Valor típico |
|-----------|---------|-------------|--------------|
| Significance level | α | P(rechazar H₀ \| H₀ cierta) — false positive rate | 0.05 |
| Power | 1 − β | P(rechazar H₀ \| H₁ cierta) — detectar efecto real | 0.80 |
| Minimum Detectable Effect | MDE (δ) | Efecto mínimo que vale la pena detectar | Definido por negocio |
| Varianza de la métrica | σ² | Dispersión natural de la métrica primaria | Estimada de datos históricos |

La fórmula clásica para dos proporciones (two-sample z-test, split 50/50):

```
n por grupo ≈ (Z_{α/2} + Z_β)² · 2σ² / δ²

Donde:
  Z_{α/2} ≈ 1.96  (para α = 0.05, two-sided)
  Z_β     ≈ 0.84  (para power = 0.80)

Ejemplo numérico:
  σ = 0.10 (desviación estándar de conversion rate)
  δ = 0.005 (MDE = +0.5 pp en conversión)

  n ≈ (1.96 + 0.84)² · 2 · (0.10)² / (0.005)²
    ≈ 7.84 · 0.02 / 0.000025
    ≈ 6,272 por grupo → ~12,544 total
```

### 2.2 Duración del experimento

🧭 [Puente] La duración no es solo "hasta juntar la n". Hay que cubrir:

- **Ciclos completos**: al menos 1–2 semanas para capturar patrones día-de-semana.
- **Novelty/primacy effects**: la primera exposición no es representativa del estado estable.
- **Ramp-up gradual**: empezar con 1–5% de tráfico para detectar bugs antes de escalar.

> [!warning] Error frecuente
> Calcular sample size, dividir por tráfico diario, y correr exactamente esos días. Esto ignora la dependencia temporal y puede coincidir con eventos atípicos (holidays, campañas). Siempre redondear a semanas completas.

### 2.3 El MDE como decisión de negocio

👔 [Ejecutivo] El MDE responde a: "¿Cuál es el efecto mínimo que justifica el costo de implementar el cambio?" Si el cambio es barato de implementar, puedes buscar efectos pequeños (pero necesitarás más muestra). Si es costoso, un MDE grande reduce la duración del test.

```
Trade-off del MDE:

  MDE grande ──→ menos muestra ──→ test corto ──→ pero no detectas efectos pequeños
       │
       ▼
  MDE pequeño ──→ más muestra ──→ test largo ──→ pero capturas mejoras sutiles
```

---

## 3. Randomización

### 3.1 Unidad de randomización

🔧 [Técnico]

| Unidad | Cuándo usarla | Riesgo |
|--------|--------------|--------|
| Usuario (user_id) | Default en la mayoría de productos | Usuarios con múltiples dispositivos pueden tener experiencia inconsistente |
| Sesión | Cuando la intervención es efímera y no hay carryover | Mismo usuario puede ver ambas variantes → dilución |
| Cookie/device_id | Cuando no hay login | Reset de cookies contamina el experimento |
| Cluster (geo, tienda, equipo) | Cuando hay interferencia intra-cluster | Menos unidades efectivas → menor power |

> [!tip] Regla de oro
> La unidad de randomización debe ser la unidad más pequeña en la que la intervención es **estable** y donde la **interferencia** entre unidades es despreciable.

### 3.2 Stratified randomization

🔧 [Técnico] Si hay covariables de alto impacto (plataforma, país, segmento de usuario), estratificar la asignación garantiza balance exacto en esas dimensiones:

```
Población total
    │
    ├── Estrato: iOS
    │     ├── 50% → Tratamiento
    │     └── 50% → Control
    │
    ├── Estrato: Android
    │     ├── 50% → Tratamiento
    │     └── 50% → Control
    │
    └── Estrato: Web
          ├── 50% → Tratamiento
          └── 50% → Control
```

Reduce varianza residual sin sesgo. Especialmente útil cuando los estratos tienen métricas base muy diferentes.

### 3.3 CUPED / CUPAC — Variance reduction

🔧 [Técnico] **CUPED** (Controlled-experiment Using Pre-Experiment Data) usa datos pre-experimentales como covariable para reducir la varianza del estimador:

```
Métrica ajustada:
  Ŷ_adjusted = Ŷ − θ · (X_pre − X̄_pre)

Donde:
  X_pre  = valor de la métrica en el período pre-experimento
  θ      = Cov(Y, X_pre) / Var(X_pre)

Reducción de varianza:
  Var(Ŷ_adjusted) = Var(Ŷ) · (1 − ρ²)

  Si ρ(Y, X_pre) = 0.7 → reducción del 51% en varianza
                         → equivale a duplicar la muestra
```

🧭 [Puente] CUPED no cambia el diseño del experimento — es un ajuste en el análisis. Permite detectar efectos más pequeños con la misma muestra, o correr experimentos más cortos con el mismo power.

**CUPAC** (Controlled-experiment Using Predictions As Covariates) generaliza CUPED: en lugar de usar solo la métrica pre-periodo, usa un modelo predictivo (e.g., expected revenue por usuario basado en features históricas) como covariable. Mayor reducción de varianza cuando la métrica tiene baja autocorrelación temporal.

> [!cite] Referencia
> Deng, A., Xu, Y., Kohavi, R., & Walker, T. (2013). "Improving the Sensitivity of Online Controlled Experiments by Utilizing Pre-Experiment Data." *Proceedings of WSDM 2013*, pp. 123–132.

---

## 4. Sequential testing vs fixed-horizon

### 4.1 El problema del peeking

🧭 [Puente] En un test fixed-horizon, calculas sample size, corres el experimento, y analizas **una sola vez** al final. Pero en la práctica, los equipos miran resultados diariamente. Cada "peek" es un test de hipótesis implícito que infla el false positive rate:

```
Número de peeks    α efectivo (si α nominal = 0.05)
      1                    0.05
      5                    ~0.14
     10                    ~0.19
     50                    ~0.30
    100                    ~0.37

Fuente: simulaciones con z-tests repetidos sobre H₀ verdadera
```

👔 [Ejecutivo] Si tu equipo mira resultados antes de tiempo y para el test cuando "se ve significativo", está tomando decisiones con una tasa de falsos positivos mucho mayor que el 5% acordado.

### 4.2 Sequential testing — siempre válido

🔧 [Técnico] Los métodos de sequential testing permiten monitorear resultados continuamente mientras controlan el α global:

**Spending functions (Lan-DeMets):**
- Definen cómo "gastar" el α a lo largo del tiempo.
- α-spending de O'Brien-Fleming: conservador al inicio, gasta más al final.
- α-spending de Pocock: gasto uniforme.

**Always-valid p-values / confidence sequences:**
- Construyen intervalos de confianza que son válidos en **cualquier** stopping time.
- Basados en mixture martingales o e-values.
- Permiten parar en cuanto el intervalo excluye cero (o en cuanto decides que el efecto es demasiado pequeño → futility stop).

```
Fixed-horizon:
  ────────────────────────────────────X  (analizar solo aquí)
  t=0                              t=T

Sequential (spending function):
  ──────●──────●──────●──────●──────X
  t=0   t1     t2     t3     t4    t=T
        │      │      │      │      │
     α₁=0.001 α₂=0.005 α₃=0.014 α₄=0.025 α₅ (residual)
     (suma ≤ 0.05)

Always-valid:
  ════════════════════════════════════►
  Monitorear continuamente; parar cuando
  confidence sequence excluye 0 (o por futility)
```

### 4.3 ¿Cuándo usar cada approach?

| Escenario | Recomendación |
|-----------|--------------|
| Feature launch con fecha fija | Fixed-horizon + CUPED |
| Optimización continua, quieres parar pronto si hay winner claro | Sequential testing |
| Riesgo alto de degradación (e.g., checkout flow) | Sequential con futility boundary |
| Muchos tests en paralelo, sin urgencia | Fixed-horizon (más simple) |

> [!cite] Referencia
> Johari, R., Pekelis, L., & Walsh, D. (2017). "Peeking at A/B Tests: Why It Matters, and What to Do About It." *Proceedings of KDD 2017*, pp. 1517–1525.

---

## 5. Interleaving y switchback designs

### 5.1 Interleaving (ranking & recommendations)

🔧 [Técnico] En sistemas de ranking (search, recommendations), el interleaving mezcla resultados de dos algoritmos en una sola lista y mide preferencia del usuario por clicks/engagement:

```
Algoritmo A produce:  [a1, a2, a3, a4, a5]
Algoritmo B produce:  [b1, b2, b3, b4, b5]

Interleaved list (Team Draft):
  Posición 1: a1 (de A)
  Posición 2: b1 (de B)
  Posición 3: b2 (de B)
  Posición 4: a2 (de A)
  ...

Métrica: proporción de clicks atribuidos a A vs B
```

🧭 [Puente] La ventaja del interleaving es **sensibilidad**: como cada usuario ve ambos algoritmos simultáneamente, actúa como su propio control. Detecta diferencias con 10–100x menos muestra que un A/B test tradicional entre usuarios. La desventaja: solo mide preferencia relativa en engagement, no efectos en métricas downstream (revenue, retention).

### 5.2 Switchback design (marketplaces, ride-sharing)

🔧 [Técnico] Cuando la unidad de interés (e.g., un mercado geográfico) tiene pocos clusters y hay **interferencia** fuerte entre usuarios (e.g., el precio de un ride afecta a todos los riders y drivers en la zona), se usa switchback:

```
Tiempo →   T1    T2    T3    T4    T5    T6    T7    T8
         ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
Ciudad A │  A  │  B  │  A  │  B  │  A  │  B  │  A  │  B  │
         ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
Ciudad B │  B  │  A  │  B  │  A  │  B  │  A  │  B  │  A  │
         └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Cada período = horas (e.g., 3–6 horas)
Alternancia evita confounding temporal
Análisis: difference-in-means con clustering por (ciudad × período)
```

> [!warning] Carryover effects
> Si el tratamiento tiene efectos residuales que persisten al período siguiente (e.g., usuarios que descargaron la app por una promo siguen activos después), el switchback está sesgado. Solución: descartar datos de los primeros minutos/hora de cada período (burn-in) o modelar el carryover explícitamente.

---

## 6. Múltiples métricas y corrección

### 6.1 Decision metrics vs guardrail metrics

👔 [Ejecutivo]

```
┌─────────────────────────────────────────────────────────┐
│              Jerarquía de métricas                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────────────────┐                              │
│  │  OEC (Overall         │  ← Métrica primaria de       │
│  │  Evaluation Criterion)│    decisión. Una sola.       │
│  └───────────────────────┘                              │
│                                                         │
│  ┌───────────────────────┐                              │
│  │  Decision metrics     │  ← Métricas secundarias      │
│  │  (2-4 métricas)       │    que informan la decisión  │
│  └───────────────────────┘                              │
│                                                         │
│  ┌───────────────────────┐                              │
│  │  Guardrail metrics    │  ← No deben degradarse.      │
│  │  (latency, crashes,   │    Veto power: si se rompen, │
│  │   revenue, etc.)      │    no se lanza.              │
│  └───────────────────────┘                              │
│                                                         │
│  ┌───────────────────────┐                              │
│  │  Debug / diagnostic   │  ← Para entender mecanismos  │
│  │  metrics              │    No para decidir           │
│  └───────────────────────┘                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

🧭 [Puente] El dimensionamiento del test (sample size) se hace sobre la **OEC**. Las guardrail metrics se evalúan con un criterio asimétrico: solo importa la degradación (one-sided test). Las debug metrics no requieren corrección por múltiples comparaciones.

### 6.2 Corrección por múltiples comparaciones

🔧 [Técnico]

| Método | Controla | Cuándo usarlo |
|--------|----------|---------------|
| Bonferroni | FWER (Family-Wise Error Rate) | Pocas métricas de decisión (2–5), se necesita certeza fuerte |
| Holm-Bonferroni | FWER (más power que Bonferroni) | Misma situación, ligeramente menos conservador |
| Benjamini-Hochberg | FDR (False Discovery Rate) | Muchas métricas exploratorias, tolerable que alguna sea falso positivo |
| Sin corrección | α por métrica | OEC única pre-registrada; guardrails con dirección pre-especificada |

> [!tip] Recomendación práctica
> - **OEC pre-registrada**: sin corrección (ya es una sola métrica).
> - **Guardrails (3–5)**: Holm-Bonferroni o simplemente one-sided tests con α = 0.01 por cada una.
> - **20+ métricas exploratorias**: FDR (Benjamini-Hochberg) para screening, luego confirmar con test dedicado.

---

## 7. Trampas comunes

### 7.1 Novelty effect y primacy effect

💡 [Analogía] El novelty effect es como cuando pruebas un restaurante nuevo: las primeras veces prestas más atención, pero luego se normaliza. Si mides engagement solo en la primera semana, sobreestimas el efecto a largo plazo.

🔧 [Técnico] **Diagnóstico**: segmentar resultados por cohorte de exposición (día en que el usuario vio el tratamiento por primera vez). Si el efecto decae con el tiempo de exposición → novelty. Si crece → primacy/learning effect.

**Mitigación**: excluir los primeros N días de exposición del análisis, o correr el test lo suficiente para alcanzar estado estable (típicamente 2–4 semanas).

### 7.2 SRM (Sample Ratio Mismatch)

🧭 [Puente] Si diseñaste un split 50/50 pero observas 51.2%/48.8% con un chi-squared test significativo, algo está roto en el pipeline de asignación:

```
Causas comunes de SRM:
┌────────────────────────────────────────────────────────┐
│ • Bots/crawlers asignados pero filtrados asimétricamente│
│ • Redirect que falla más en una variante               │
│ • Logging diferencial (eventos no se registran en      │
│   control porque el feature no existe)                 │
│ • Trigger condition que depende del tratamiento        │
│ • Users que abandonan antes de ser loggeados           │
└────────────────────────────────────────────────────────┘
```

> [!danger] Regla absoluta
> Si hay SRM, **no interpretes los resultados**. Primero diagnostica y corrige la causa. Un test con SRM no tiene validez interna.

### 7.3 Interference / Spillover (network effects)

🔧 [Técnico] El supuesto SUTVA (Stable Unit Treatment Value Assumption) requiere que el tratamiento de un usuario no afecte los outcomes de otro. Se viola en:

- **Redes sociales**: si tratas a un usuario con una feature de sharing, sus amigos en control también se benefician → dilución del efecto.
- **Marketplaces**: reducir precio para tratamiento roba demand del control → sobreestima efecto.
- **Contenido viral**: tratamiento puede generar contenido que llega a control.

**Mitigaciones**:
- Cluster randomization (randomizar por comunidad, no por individuo).
- Ego-network randomization.
- Diseños de dos niveles: randomizar clusters Y dentro del cluster randomizar individuos.
- Modelar spillover con estimadores de efecto directo + indirecto.

### 7.4 Simpson's paradox en segmentos

🧭 [Puente] El tratamiento puede ser positivo en cada segmento pero **negativo** en el agregado (o viceversa) si el tratamiento cambia la composición de los segmentos:

```
Ejemplo:
                  Control        Tratamiento
  Segmento A:    10% conv        12% conv  (+2 pp) ✓
  Segmento B:     2% conv         3% conv  (+1 pp) ✓

  Pero si tratamiento atrae más usuarios del segmento B (baja conversión):
  Agregado:       8% conv         7% conv  (-1 pp) ✗ ???

  Composición Control:  80% A, 20% B → conv = 0.8×10 + 0.2×2 = 8.4%
  Composición Trata:    40% A, 60% B → conv = 0.4×12 + 0.6×3 = 6.6%
```

🔧 [Técnico] **Diagnóstico**: si la composición de segmentos cambia entre variantes, el efecto agregado mezcla el efecto causal con el efecto composicional. Solución: reportar efecto **dentro de cada segmento** (condicional) y decidir si el cambio composicional es deseable o no.

---

## 8. Caso de negocio

> [!example] Caso: Optimización de flujo de onboarding en app de finanzas personales

👔 [Ejecutivo] **Contexto**: Una app de finanzas personales quiere simplificar su onboarding de 5 pasos a 3 pasos. Hipótesis: menos fricción → más usuarios completan el registro → más activación a 7 días.

**Diseño**:

```
Métricas:
  OEC: Activación a 7 días (usuario hace ≥1 transacción en primera semana)
  Guardrails: Fraud rate, crash rate, customer support tickets
  Secondary: Completion rate del onboarding, time-to-first-action

Baseline:
  Activación 7d actual: 34%
  MDE deseado: +2 pp (de 34% a 36%) — justificado por LTV analysis

Dimensionamiento:
  σ² ≈ p(1-p) = 0.34 × 0.66 = 0.2244
  n ≈ (1.96 + 0.84)² × 2 × 0.2244 / (0.02)²
    ≈ 7.84 × 0.4488 / 0.0004
    ≈ 8,797 por grupo → 17,594 total

  Tráfico: ~2,000 nuevos usuarios/día
  Duración: 17,594 / 2,000 ≈ 9 días → redondear a 14 días (2 semanas completas)

Variance reduction:
  CUPAC con predicted_activation_probability (de modelo histórico)
  ρ esperada ≈ 0.5 → reducción ~25% → podría acortar a 11 días
  Pero mantenemos 14 para capturar ciclo semanal completo

Diseño: sequential testing con O'Brien-Fleming spending function
  - Interim analysis al día 7
  - Análisis final al día 14
  - Futility boundary para parar si efecto < 0.5 pp al día 7
```

**Resultado simulado**:
- Día 7: activación +1.8 pp (p = 0.08) → no cruza boundary → continuar.
- Día 14: activación +2.3 pp (p = 0.02) → significativo → lanzar.
- Guardrails: fraud rate sin cambio, crash rate sin cambio, tickets −5% (no significativo pero direccionalmente positivo).
- Decisión: **ship**.

🧭 [Puente] Lecciones del caso:
1. El MDE se derivó del impacto en negocio (LTV), no de conveniencia estadística.
2. Se usó sequential testing para poder parar temprano si el efecto era claro.
3. Las guardrails protegieron contra degradaciones no anticipadas.
4. Se redondeó a semanas completas para evitar sesgos temporales.

---

## 9. Guía de decisión: ¿A/B, quasi-experiment, o bandit?

```
                    ¿Puedes aleatorizar?
                          │
              ┌───────────┴───────────┐
              │                       │
             SÍ                      NO
              │                       │
              ▼                       ▼
    ¿Hay interferencia        Quasi-experiment
    entre unidades?           [[18-Causalidad-Inferencia-Causal]]
              │               (DiD, RDD, IV, synthetic control)
    ┌─────────┴─────────┐
    │                   │
   NO              SÍ (network/
    │               marketplace)
    ▼                   ▼
  ¿Necesitas           Switchback / Cluster
  explorar muchas      randomization
  variantes?           (Sección 5)
    │
    ┌─────┴─────┐
    │           │
   NO          SÍ
    │           │
    ▼           ▼
  A/B test    ¿El costo de
  clásico     suboptimalidad
  (Secciones  durante el test
  2-4)        es alto?
                │
        ┌───────┴───────┐
        │               │
       SÍ              NO
        │               │
        ▼               ▼
   Multi-armed        A/B/n test
   bandit             (múltiples
   [[21-Multi-Armed-  variantes,
   Bandits]]          fixed allocation)
   (Thompson,
   UCB — minimiza
   regret)
```

### Resumen comparativo

| Criterio | A/B test | Quasi-experiment | Bandit |
|----------|----------|-----------------|--------|
| Aleatorización | Sí | No (natural/instrumental) | Sí (adaptativa) |
| Validez interna | Alta | Media (depende de supuestos) | Alta (pero sesgado para inferencia) |
| Óptimo para inferencia causal | ✓✓✓ | ✓✓ | ✓ (con correcciones) |
| Óptimo para optimización | ✓✓ | ✗ | ✓✓✓ |
| Requiere tráfico | Moderado-alto | Bajo (usa datos existentes) | Moderado |
| Complejidad de implementación | Media | Alta (análisis) | Alta (ingeniería) |

> [!tip] Heurística ejecutiva
> - **"Quiero saber si X causa Y con alta confianza"** → A/B test.
> - **"No puedo aleatorizar pero necesito estimar el efecto"** → Quasi-experiment.
> - **"Tengo muchas opciones y quiero encontrar la mejor rápido minimizando pérdida"** → Bandit.
> - **"Quiero saber Y, y además tengo interferencia de red"** → Cluster/switchback + A/B.

---

## 10. Checklist operativo para un A/B test

> [!success] Pre-lanzamiento
> - [ ] OEC definida y aprobada por stakeholders
> - [ ] MDE justificado por impacto en negocio
> - [ ] Sample size calculado con varianza estimada de datos históricos
> - [ ] Duración redondeada a semanas completas
> - [ ] CUPED/CUPAC evaluado para reducción de varianza
> - [ ] Guardrail metrics definidas con umbrales
> - [ ] Unidad de randomización elegida (sin interferencia)
> - [ ] Documentado: hipótesis, métricas, criterios de decisión (pre-registration)

> [!success] Durante el test
> - [ ] Verificar SRM en las primeras 24–48 horas
> - [ ] Monitorear guardrails con alertas automáticas
> - [ ] NO tomar decisiones basadas en resultados parciales (a menos que uses sequential testing)
> - [ ] Verificar que el trigger funciona correctamente (solo usuarios elegibles entran)

> [!success] Post-test
> - [ ] Análisis con método pre-especificado
> - [ ] Verificar SRM final
> - [ ] Reportar CI del efecto, no solo p-value
> - [ ] Segmentar para entender heterogeneidad (pero no cherry-pick)
> - [ ] Documentar decisión y rationale
> - [ ] Si se lanza: monitorear métricas post-launch (holdback si es posible)

---

## Referencias

1. **Kohavi, R., Tang, D., & Xu, Y.** (2020). *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*. Cambridge University Press. — El libro de referencia completo sobre experimentación en productos digitales; cubre diseño, análisis, plataformas y cultura de experimentación.

2. **Johari, R., Pekelis, L., & Walsh, D.** (2017). "Peeking at A/B Tests: Why It Matters, and What to Do About It." *Proceedings of the 23rd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining (KDD '17)*, pp. 1517–1525. — Formaliza el problema del peeking y propone always-valid inference para A/B tests.

3. **Deng, A., Xu, Y., Kohavi, R., & Walker, T.** (2013). "Improving the Sensitivity of Online Controlled Experiments by Utilizing Pre-Experiment Data." *Proceedings of the 6th ACM International Conference on Web Search and Data Mining (WSDM '13)*, pp. 123–132. — Introduce CUPED para variance reduction en experimentos online.

4. **Bojinov, I., Simchi-Levi, D., & Zhao, J.** (2023). "Design and Analysis of Switchback Experiments." *Management Science*, 69(7), pp. 3759–3777. — Formalización del diseño switchback con análisis de carryover effects.

5. **Larsen, N., Stallrich, J., Sengupta, S., Deng, A., Kohavi, R., & Stevens, N.** (2024). "Statistical Challenges in Online Controlled Experiments: A Review of A/B Testing Methodology." *The American Statistician*, 78(2), pp. 135–149. — Survey reciente de desafíos metodológicos en experimentación a escala.

6. **Howard, S. R., Ramdas, A., McAuliffe, J., & Sekhon, J.** (2021). "Time-uniform, Nonparametric, Nonasymptotic Confidence Sequences." *The Annals of Statistics*, 49(2), pp. 1055–1080. — Fundamentos teóricos de confidence sequences para always-valid inference.

---

## Conexiones con otros tomos

| Tomo | Relación |
|------|----------|
| [[18-Causalidad-Inferencia-Causal]] | Cuando no puedes aleatorizar: DiD, RDD, IV, synthetic control |
| [[21-Multi-Armed-Bandits]] | Alternativa al A/B cuando quieres minimizar regret en lugar de maximizar inferencia |
| [[23-Tabular-DL-vs-Boosting]] | Los modelos de CUPAC pueden ser gradient boosting sobre features históricas |
| [[07-Estadistica-Inferencial]] | Fundamentos de hypothesis testing, p-values, confidence intervals |
| [[14-Metricas-Evaluacion]] | Definición rigurosa de métricas que se usan como OEC y guardrails |

---

**Navegación:** ← [[23-Tabular-DL-vs-Boosting]] · Siguiente tomo pendiente
