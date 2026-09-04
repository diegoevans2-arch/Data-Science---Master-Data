---
title: "Tomo 14 — Anexo: Cómo Interpretar Resultados (para no técnicos)"
tags: [data-science, machine-learning, ejecutivo, interpretacion, anexo]
audiencias: [ejecutivo, puente]
tomo: 14
version: 6.1
updated: 2026-08-28
---

# 👓 Tomo 14 — Anexo: Cómo Interpretar Resultados (para no técnicos)

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[13-MLOps-XAI-Etica|13 · MLOps, XAI y Ética]] · Siguiente: [[15-Glosario-Ejecutivo|15 · Glosario Ejecutivo ➡]]

---

> [!info] 📌 ¿Por qué importa este anexo?
> Este tomo es para quien **no programa y no quiere programar**: gerentes, sponsors, líderes de área. Los cuatro gráficos que el equipo de datos te mostrará una y otra vez — matriz de confusión, curva ROC, feature importance y dashboard de drift — se leen en minutos si alguien te enseña cómo. Aquí está ese alguien. Cero fórmulas obligatorias; solo lectura guiada y las preguntas que te conviene hacer.

> [!abstract] 👔 Impacto ejecutivo
> Leer estos gráficos sin intermediarios cambia tu posición en la mesa: de espectador a interlocutor.
>
> - **Decisiones que habilita:** aprobar o cuestionar modelos con criterio propio, detectar señales de alerta en 5 minutos, hacer las preguntas que destapan problemas antes del deployment.
> - **Costo de no saber leerlos:** aprobar por confianza ciega, rechazar por desconfianza ciega, y no distinguir un buen modelo de una buena presentación.
> - **Pregunta que responde:** *¿qué me está mostrando realmente este gráfico — y qué le pregunto a quien lo trajo?*

---

## 1. Cómo leer una matriz de confusión, paso a paso

Audiencia: 👔 🧭

Supón un modelo que detecta clientes que se van a fugar (churn). El equipo te muestra esta tabla con los resultados sobre 1.000 clientes de prueba:

```
                        LO QUE DIJO EL MODELO
                     "Se fuga"      "Se queda"
 LO QUE      Se fugó     80             40        ← 120 fugas reales
 PASÓ DE
 VERDAD      Se quedó    60            820        ← 880 no-fugas reales
```

**Los cuatro pasos de lectura:**

1. **Diagonal principal = aciertos.** 80 + 820 = 900 de 1.000: el modelo acierta el 90% global. *No te detengas aquí: el 90% puede esconder lo importante.*
2. **Esquina superior derecha (40) = los que se escaparon.** Fugas reales que el modelo NO detectó (falsos negativos). Son tus clientes perdidos sin aviso. Pregunta clave: *¿cuánto cuesta cada uno?*
3. **Esquina inferior izquierda (60) = las falsas alarmas.** Clientes fieles marcados como fuga (falsos positivos). Son llamadas de retención desperdiciadas (o descuentos regalados a quien no se iba). Pregunta clave: *¿cuánto cuesta cada una?*
4. **Compara los dos errores con sus costos.** Aquí: detectas 80 de 120 fugas (dos de cada tres) al precio de 60 falsas alarmas. ¿Buen negocio? Depende de cuánto vale retener un cliente vs el costo de una llamada. **Esa es una decisión tuya, no del modelo.**

> [!warning] ⚠️ La trampa del "90% de acierto"
> Un modelo que dijera "nadie se fuga" acertaría 880 de 1.000 = 88%, casi lo mismo… y no detectaría **ninguna** fuga. Por eso el accuracy global es el número menos interesante de la tabla: mira siempre las dos esquinas de error por separado.

**Preguntas de ejecutivo ante una matriz de confusión:** ¿cuánto cuesta en pesos cada tipo de error? · ¿este balance de errores se puede mover? (sí: se llama umbral, y es un dial — [[08-Metricas-de-Evaluacion]]) · ¿estos 1.000 casos de prueba se parecen a mi operación real?

---

## 2. Cómo leer una curva ROC y qué significa el AUC

Audiencia: 👔 🧭

> [!tip] 💡 La idea en una imagen
> El modelo no dice "se fuga / no se fuga": entrega un **puntaje de riesgo** por cliente, y tú eliges dónde cortar. La curva ROC muestra TODOS los cortes posibles a la vez: cada punto de la curva es un balance distinto entre "detectar más fugas" (subir por el eje vertical) y "tragarse más falsas alarmas" (avanzar por el horizontal).

```
 % de fugas    1 ┤        ╭──────────●   ← curva del modelo
 detectadas      │      ╭─╯
 (mientras       │    ╭─╯    ╱
 más arriba,     │  ╭─╯    ╱   ← la diagonal = tirar una moneda
 mejor)          │ ╭╯    ╱
               0 └─┴───╱──────────────
                  0        % de falsas alarmas →
```

**Lectura práctica:**

- **Mientras más se "infla" la curva hacia la esquina superior izquierda, mejor** el modelo: detecta mucho tragándose pocas falsas alarmas.
- **La diagonal es el azar puro.** Un modelo pegado a la diagonal no aporta nada: es una moneda cara.
- **El AUC es el área bajo la curva** — un resumen de 0.5 (moneda) a 1.0 (perfecto). Lectura útil: *AUC = la probabilidad de que, tomando un cliente que se fugó y uno que no, el modelo le haya puesto más riesgo al que se fugó*. Referencias gruesas: 0.6 débil · 0.7 útil · 0.8 bueno · 0.9 excelente · **> 0.97 sospechoso** (huele a trampa en los datos, [[10-Validacion-y-Leakage]]).

**Preguntas de ejecutivo ante una ROC:** ¿en qué punto de la curva vamos a operar y por qué? · ¿el AUC se midió con datos que el modelo nunca vio, y de fecha posterior al entrenamiento? · si el AUC es altísimo, ¿ya auditaron que no haya información filtrada del futuro?

---

## 3. Cómo leer un feature importance / SHAP summary

Audiencia: 👔 🧭

> [!tip] 💡 La idea
> Es el ranking de "qué mira el modelo para decidir": las variables ordenadas por su influencia en las predicciones. El SHAP summary agrega un lujo: muestra además la **dirección** — si valores altos de esa variable empujan hacia "riesgo" o hacia "seguro".

```
 Importancia de variables (ejemplo churn)
 meses_sin_comprar     ████████████████████  ← domina la decisión
 reclamos_ultimo_año   ███████████
 uso_de_la_app         ████████
 antigüedad            █████
 edad                  ██
 comuna                ▌
```

**Lectura práctica:**

- **¿El top 3 tiene sentido de negocio?** Si las variables dominantes son las que tu experiencia señalaría (inactividad, reclamos), el modelo aprendió algo razonable. Si domina algo absurdo (el número interno de la sucursal), sospecha.
- **Una variable que domina aplastantemente es bandera roja:** puede ser información filtrada del futuro (leakage, [[10-Validacion-y-Leakage]]) — el modelo "adivina" porque le soplaron la respuesta.
- **En un SHAP summary**, cada punto es un cliente: el color indica si su valor en esa variable es alto o bajo, y el lado (izquierda/derecha) hacia dónde empujó su predicción. Busca coherencia: "muchos meses sin comprar" (alto) debería empujar hacia "riesgo de fuga", no al revés.

**Preguntas de ejecutivo ante un feature importance:** ¿las variables top estaban disponibles ANTES del momento de la decisión? · ¿alguna es un proxy de género/edad/zona que nos exponga a discriminación? ([[13-MLOps-XAI-Etica]]) · ¿qué pasa con el modelo si mañana dejamos de tener una de estas variables?

---

## 4. Cómo leer un dashboard de drift

Audiencia: 👔 🧭

> [!tip] 💡 La idea
> El modelo aprendió del mundo de ayer. El dashboard de drift vigila si el mundo de hoy **sigue pareciéndose** al de ayer: compara los datos que están entrando ahora contra los datos con que se entrenó. Es el control técnico del vehículo — no esperas a chocar para revisar los frenos.

```
 Panel típico de drift
 ┌────────────────────────────────────────────────────────┐
 │ Feature            drift score      estado             │
 │ ingreso_mensual       0.02          ✅ estable          │
 │ canal_de_compra       0.31          🚨 DRIFT ALTO       │
 │ edad                  0.05          ✅ estable          │
 │ ticket_promedio       0.12          ⚠️ vigilar          │
 ├────────────────────────────────────────────────────────┤
 │ % predicciones "riesgo alto":  8% → 19% este mes  🚨    │
 └────────────────────────────────────────────────────────┘
```

**Lectura práctica:**

- **Cada fila compara una variable hoy vs entrenamiento.** Score bajo = estable; alto = esa variable ya no se distribuye como antes (ej.: cambió el mix de canales porque lanzaron la app).
- **El % de predicciones por clase también se vigila:** si el modelo pasó de marcar 8% de clientes en riesgo a 19% sin campaña ni crisis de por medio, algo cambió — en los datos o en el mundo.
- **Drift NO significa automáticamente modelo malo:** significa que el modelo está opinando sobre un mundo que ya no es el suyo. La respuesta va de "vigilar" a "reentrenar" a "apagar" ([[13-MLOps-XAI-Etica]]).

**Preguntas de ejecutivo ante un dashboard de drift:** ¿quién recibe la alerta cuando algo se pone rojo, y qué hace? · ¿cuándo fue el último reentrenamiento y cuál es el criterio para el próximo? · ¿tenemos forma de saber la métrica real en producción (aunque llegue con retraso)?

### 4.1 Data drift vs concept drift — la diferencia que importa

Audiencia: 👔 🧭

| Tipo de drift | Qué cambió | Ejemplo | ¿El modelo se equivoca? |
|---|---|---|---|
| **Data drift** (covariate shift) | La distribución de las features de entrada cambió, pero la relación feature→target sigue igual | Antes el 20% de tus clientes eran jóvenes; ahora el 50% son jóvenes (por una campaña). La relación edad→churn no cambió. | Puede equivocarse más en los segmentos nuevos que no vio suficiente en train |
| **Concept drift** | La relación feature→target cambió — el mismo input ahora produce un output distinto | Antes, un saldo bajo predecía fuga. Post-pandemia, los saldos bajos son por cambio de hábitos, no por fuga. La relación se rompió. | **Sí, sistemáticamente** — y el drift de features puede no disparar alarma |
| **Prediction drift** (output drift) | Las predicciones del modelo cambiaron de distribución | El modelo empezó a marcar el doble de clientes como "riesgo alto" | Es **síntoma**, no diagnóstico: puede ser data drift, concept drift, o un bug de datos |

**👔 La pregunta clave:** "¿cambió el mundo o cambiaron mis datos?" Si cambió el mundo (concept drift), reentrenar es obligatorio. Si solo cambiaron los datos de entrada (data drift), a veces basta con verificar que el modelo sigue performando en los nuevos segmentos.

### 4.2 Qué pedir cuando te muestran un dashboard de drift

Audiencia: 👔

1. **"¿Tenemos la métrica real?"** — el drift de features es un proxy; lo que importa es si la **precisión real** del modelo cayó. Si tienes labels retrasados (churn se confirma a 30 días, fraude a la investigación), combinar drift + métrica real retrasada.

2. **"¿Cuál es el SLA de reentrenamiento?"** — ¿cada cuánto se reentrena el modelo? ¿Hay un trigger automático si el drift cruza un umbral? ¿O depende de que alguien mire el dashboard?

3. **"¿Qué feature driftó y por qué?"** — si `canal_de_compra` driftó, puede ser que lanzaron la app nueva (esperado, no peligroso) o que el pipeline de datos dejó de poblar ese campo (bug, peligroso).

4. **"¿Hay un plan B?"** — si el modelo se degrada, ¿qué se usa mientras se reentrena? ¿Reglas de negocio? ¿Un modelo anterior? ¿Nada?

---

## 4B. Cómo leer una confusion matrix multiclase — sin fórmulas

Audiencia: 👔 🧭

Cuando el modelo clasifica en **más de dos categorías** (no solo "sí/no"), la matriz crece. Ejemplo: un modelo que clasifica tickets de soporte en 4 categorías:

```
                PREDICHO →
              Factura  Envío  Técnico  Otro
 REAL ↓
 Factura      [ 85 ]    5      3       7     ← 100 tickets reales de Factura
 Envío           8    [ 72 ]   10      10    ← 100 tickets reales de Envío
 Técnico         2      5    [ 88 ]    5     ← 100 tickets reales de Técnico
 Otro           12     15      8     [ 65 ]  ← 100 tickets reales de Otro
```

**Lectura rápida para el no técnico:**

- **Diagonal = aciertos por categoría.** Factura: 85%, Envío: 72%, Técnico: 88%, Otro: 65%. "Otro" es la categoría peor clasificada — probablemente porque es un "cajón de sastre".

- **Fuera de la diagonal = confusiones.** Las celdas grandes off-diagonal te dicen **con qué se confunde**. "Otro" se confunde mucho con "Envío" (15 tickets). Pregunta: ¿hay tickets de envío que deberían ser "Otro"? ¿O la definición de "Otro" es ambigua?

- **Lectura por fila** → "de los 100 tickets reales de Envío, ¿cuántos clasificó bien?" (72 de 100 = 72% de recall para Envío).

- **Lectura por columna** → "de todos los que el modelo DIJO que eran de Factura, ¿cuántos realmente lo eran?" (85 de 107 = precision de Factura).

**Preguntas de ejecutivo ante una matrix multiclase:**
- ¿Qué categoría tiene el recall más bajo? → ahí se están "perdiendo" casos.
- ¿Cuál confusión es la más cara? (confundir "Técnico" con "Otro" retrasa la resolución más que al revés).
- ¿La categoría "Otro" es legítima o es un escape del equipo de etiquetado?

---

## 5. La batería de preguntas — resumen de bolsillo

Audiencia: 👔

| Cuando te muestren… | Pregunta siempre |
|---|---|
| Cualquier métrica de acierto | ¿Contra qué baseline? ¿Cuánto mejor que lo que hacemos hoy? |
| Una matriz de confusión | ¿Cuánto cuesta en pesos cada esquina de error? ¿Quién eligió el umbral y con qué criterio? |
| Una curva ROC / un AUC | ¿Medido en datos futuros nunca vistos? Si es > 0.97, ¿auditaron leakage? |
| Un feature importance | ¿Las variables top existían al momento de decidir? ¿Alguna discrimina indirectamente? |
| Un dashboard de drift | ¿Quién actúa cuando se enciende, y cuál es el plan de reentrenamiento? |
| Un piloto "exitoso" | ¿Cuál era el tamaño de muestra planificado? ¿Se corrigió por múltiples pruebas? ([[02-Fundamentos-Matematicos]]) |

> [!warning] ⚠️ La regla de oro del ejecutivo
> No necesitas entender el algoritmo; necesitas entender **el error**: cuánto cuesta, quién lo paga y cómo se vigila. Todo lo demás es implementación.

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[13-MLOps-XAI-Etica|13 · MLOps, XAI y Ética]] · Siguiente: [[15-Glosario-Ejecutivo|15 · Glosario Ejecutivo ➡]]

> **Próximo tomo:** [[15-Glosario-Ejecutivo]] — 45+ términos técnicos traducidos a una frase de negocio cada uno.
