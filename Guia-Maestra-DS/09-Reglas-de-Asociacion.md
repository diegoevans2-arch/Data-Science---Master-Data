---
title: "Tomo 09 — Reglas de Asociación"
tags: [data-science, machine-learning, association-rules, market-basket]
audiencias: [tecnico, puente, ejecutivo]
tomo: 09
version: 6.2
updated: 2026-08-28
---

# 🛒 Tomo 09 — Reglas de Asociación

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[08-Metricas-de-Evaluacion|08 · Métricas de Evaluación]] · Siguiente: [[10-Validacion-y-Leakage|10 · Validación y Leakage ➡]]

---

> [!info] 📌 ¿Por qué importa esta sección?
> Las reglas de asociación descubren patrones "invisibles" para el ojo humano: el clásico supermercado que descubrió que los viernes se compran pañales y cerveza juntos. La relación no es obvia, pero es **accionable**: ubicar productos, armar combos, recomendar. Descubren patrones del tipo "si A, entonces probablemente B" en colecciones de transacciones — y aplican mucho más allá del retail: diagnósticos que co-ocurren, eventos de logs que anticipan fallas, rutas de navegación web.

> [!abstract] 👔 Impacto ejecutivo
> Es la técnica de "qué va con qué": convierte millones de boletas o registros en reglas concretas que marketing y operaciones pueden ejecutar el lunes.
>
> - **Decisiones que habilita:** layout de tiendas y catálogos, bundles y cross-selling, recomendaciones simples sin infraestructura pesada, alertas por co-ocurrencia de eventos.
> - **Costo de hacerlo mal:** actuar sobre reglas espurias (alta confidence de puro popular que es B), inundarse de miles de reglas triviales, o descartar la técnica por calibrar mal min_support.
> - **Pregunta ejecutiva que responde:** *¿qué combinaciones de productos/eventos ocurren juntas más de lo que el azar explicaría — y cuáles valen dinero?*

> [!tip] 💡 Analogía general
> Un detective de boletas: revisa millones de tickets buscando parejas y tríos de productos que aparecen juntos **sospechosamente seguido**. No le interesa lo obvio (pan con casi todo — el pan es popular y ya), sino las coincidencias que superan al azar: esas esconden un comportamiento real de compra.

> [!example] 📊 Caso de negocio — Retail: del ticket a la góndola
> **Problema:** un supermercado quiere subir el ticket promedio. Intuiciones sobran; evidencia falta.
>
> **Técnica aplicada:** FP-Growth sobre 12 meses de transacciones (millones de boletas — Apriori no escala a ese volumen), min_support calibrado para producir miles (no millones) de itemsets, y filtrado por **lift > 1.3** con consecuentes accionables (se descartan reglas cuyo consecuente es una categoría presente en el 80% de las boletas). Las reglas sobrevivientes se priorizan por `support × (lift − 1)` como proxy de impacto incremental.
>
> **Resultado:** un puñado de reglas no obvias (ej.: categoría de snacks premium → vinos de cierto rango) se traducen en góndolas cruzadas, combos y recomendaciones en la app. El experimento controlado ([[02-Fundamentos-Matematicos]]) confirma alza en ticket promedio en las salas tratadas. La mitad del valor estuvo en el **filtrado**: sin él, el equipo se habría ahogado en reglas del tipo "bolsa → pan".

---

## 1. Conceptos Base

Audiencia: 🔧 🧭

| Concepto | Definición | Ejemplo |
|---|---|---|
| Itemset | Conjunto de uno o más ítems | `{Leche, Pan}` es un 2-itemset |
| Transacción | Un registro que contiene un itemset | Los ítems de una boleta; los diagnósticos de un paciente |
| Regla de asociación | `A → B`: si la transacción contiene A (antecedente), probablemente contiene B (consecuente) | `{pañales} → {cerveza}` |
| Itemset frecuente | Itemset cuyo support supera el umbral `min_support` | El paso computacionalmente costoso es encontrarlos todos |

---

## 2. Métricas de las Reglas

Audiencia: 🔧 🧭

> [!tip] 💡 Analogías de las métricas
> **Support:** ¿qué tan popular es la combinación? — como medir cuán seguido suena una canción en la radio. **Confidence:** si ya compraste pan, ¿qué tan probable es que lleves mantequilla? **Lift:** ¿comprar A realmente *empuja* a comprar B, o B se vende solo? Lift = 1 es independencia; lift = 3, A triplica la probabilidad de B. **Conviction:** ¿cuánto se derrumbaría la regla si fuera puro azar?

| Métrica | Fórmula | Rango | Interpretación | Cuándo es engañosa |
|---|---|---|---|---|
| Support | `freq(A∪B) / N` | [0,1] | Popularidad del patrón: proporción de transacciones con A y B juntos | Support bajo no implica irrelevante: un patrón raro puede ser muy valioso (compras de lujo) |
| Confidence | `freq(A∪B) / freq(A)` | [0,1] | `P(B│A)`: si la transacción tiene A, ¿qué tan probable es B? | Alta solo porque B es popular por sí solo, sin relación real con A |
| **Lift** | `Confidence(A→B) / Support(B)` = `Support(A∪B) / (Support(A)·Support(B))` | [0,∞) | > 1: co-ocurren más de lo esperado al azar (relación real); = 1: independientes; < 1: se repelen | Puede dispararse con ítems muy raros aunque la asociación sea frágil (pocas observaciones) |
| Conviction | `(1 − Support(B)) / (1 − Confidence(A→B))` | [0,∞) | Cuánto "depende del azar" la regla: ∞ = regla perfecta; 1 = independencia | Asimétrica: Conviction(A→B) ≠ Conviction(B→A) |
| Leverage | `Support(A∪B) − Support(A)·Support(B)` | [−1,1] | Diferencia absoluta entre co-ocurrencia observada y esperada; 0 = independencia | Escala absoluta: favorece ítems frecuentes y opaca patrones raros valiosos |
| Zhang's metric | Normalización de leverage | [−1,1] | +1 dependencia perfecta; −1 exclusión mutua; simétrica (considera A→B y B→A) | Menos conocida: cuidado al comunicarla sin explicación |
| Jaccard | `freq(A∪B) / (freq(A)+freq(B)−freq(A∪B))` | [0,1] | Similitud entre los conjuntos de transacciones de A y de B | Ignora la direccionalidad de la regla |

> [!warning] ⚠️ El trío mínimo de lectura
> Ninguna métrica basta sola: una regla útil necesita **support razonable** (ocurre lo suficiente para accionar), **confidence decente** (la implicación es fiable) y **lift > 1 con holgura** (no es un espejismo de popularidad). Reglas con lift ≤ 1 se descartan sin duelo.

---

## 3. Algoritmos

Audiencia: 🔧 🧭

### Apriori

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Revisar ingredientes de cocina nivel por nivel: primero qué ingredientes solos son populares, luego qué **parejas**, luego qué tríos… con un atajo brillante: si "aceitunas" solas ya son impopulares, ninguna combinación con aceitunas puede ser popular — ni te molestas en revisarlas.

**🔧 Definición técnica:** (Agrawal & Srikant, 1994). **Principio anti-monótono:** si un itemset es infrecuente, todos sus supersets también lo son → poda el espacio exponencial. Procedimiento: (1) 1-itemsets frecuentes; (2) generar candidatos k-itemsets combinando los (k−1) frecuentes; (3) filtrar por min_support; (4) repetir hasta agotar; (5) generar reglas desde los itemsets frecuentes y filtrar por min_confidence. **Limitación:** una pasada completa al dataset por nivel k + generación masiva de candidatos → lento con muchos ítems o min_support bajo.

**🧭 Dataset ideal y caso de uso:** datasets pequeños-medianos, pocos ítems por transacción; enseñanza y prototipos (mlxtend: `apriori()`).

### FP-Growth

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> En vez de releer todas las boletas una y otra vez, arma un **mapa mental comprimido** de todas las compras (el FP-Tree) en solo dos lecturas, y después explora ese mapa en memoria. Como resumir un libro en un esquema y estudiar del esquema.

**🔧 Definición técnica:** (Han et al., 2000). Solo **2 pasadas** sobre los datos: (1) construir el FP-Tree — árbol comprimido en memoria que codifica las transacciones ordenadas por frecuencia; (2) minarlo recursivamente vía conditional pattern bases. No genera candidatos explícitos. **Ventaja:** típicamente 10–100× más rápido que Apriori; el FP-Tree suele caber en memoria. Implementación: `mlxtend.frequent_patterns.fpgrowth()`; PyFIM para volúmenes masivos.

**🧭 Dataset ideal y caso de uso:** datasets grandes (millones de boletas, clickstream de e-commerce); el default de producción.

### Eclat

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Gira la pregunta: en lugar de "¿qué compró cada persona?", pregunta "¿**quiénes** compraron cada producto?" y busca intersecciones de listas de compradores. Dos productos van juntos si sus listas de clientes se traslapan mucho.

**🔧 Definición técnica:** (Zaki, 2000). Representación **vertical**: por cada ítem, la TID-list (conjunto de IDs de transacciones donde aparece). `Support({A,B}) = │TID(A) ∩ TID(B)│` — la intersección de conjuntos es muy eficiente. Profundiza en DFS sobre clases de equivalencia.

**🧭 Dataset ideal y caso de uso:** datasets **densos** (muchos ítems por transacción): logs de sistemas, síntomas por paciente, features binarias.

| Algoritmo | Pasadas al dataset | Estrategia | Escala a | Cuándo conviene |
|---|---|---|---|---|
| Apriori | Una por nivel k | Generar y podar candidatos (anti-monótono) | Pequeño-mediano | Didáctica, prototipos, pocos ítems |
| FP-Growth | 2 | Árbol comprimido en memoria (FP-Tree) | Grande | Producción retail/clickstream |
| Eclat | 1 (formato vertical) | Intersección de TID-lists | Mediano-grande denso | Transacciones densas (logs, medicina) |

---

## 4. Consideraciones Prácticas

Audiencia: 🔧 🧭 👔

- **Calibración de min_support:** muy alto → solo patrones archisabidos; muy bajo → explosión combinatoria de reglas triviales. Heurística de partida: apuntar a que se generen ≈1.000–10.000 reglas y ajustar iterativamente.
- **Filtrado post-generación:** eliminar lift ≤ 1 (sin relación real); eliminar consecuentes triviales (ítems presentes en > 80–90% de las transacciones); priorizar reglas **accionables** (consecuentes que se pueden promocionar, recomendar o alertar).
- **Tamaño útil de itemsets:** las reglas de mayor utilidad práctica son 2-itemsets y 3-itemsets; reglas de 5+ ítems son raras, hiper-específicas y difíciles de accionar.
- **Dominios no-retail (misma matemática, otra lectura):** medicina — ítem = diagnóstico/síntoma, transacción = paciente (comorbilidades); logs de sistemas — ítem = evento, transacción = sesión (secuencias que anticipan fallas); clickstream — ítem = página, transacción = visita (rutas de navegación).

**👔 En una frase para el negocio:** una regla solo vale si sobrevive tres filtros — ocurre bastante, implica de verdad, y **alguien puede hacer algo con ella** el lunes.

---

## 5. Sequential Pattern Mining — el orden importa

Audiencia: 🔧 🧭

> [!tip] 💡 Analogía
> Las reglas de asociación dicen "pan y mantequilla se compran juntos"; los patrones secuenciales dicen "primero compró pañales, Y DESPUÉS compró cerveza — en ese orden". La diferencia es la **flecha del tiempo**: no es lo mismo co-ocurrencia que secuencia.

**🔧 Definición técnica:** descubrir sub-secuencias frecuentes en colecciones de secuencias ordenadas de eventos. La estructura base no es un set (cesta) sino una **lista ordenada** de itemsets.

| Algoritmo | Mecanismo | Escala | Caso de uso |
|---|---|---|---|
| **GSP** (Generalized Sequential Patterns, Srikant & Agrawal, 1996) | Extensión de Apriori al dominio secuencial; genera candidatos nivel a nivel con restricciones de orden | Mediano | Clickstream corto, secuencias de diagnósticos |
| **PrefixSpan** (Pei et al., 2001) | Crecimiento basado en proyección de prefijos; no genera candidatos explícitos | Grande | El estándar actual: logs de sistemas, customer journeys largos |
| **SPADE** (Zaki, 2001) | Representación vertical (como Eclat) + intersecc. temporal de ID-lists | Mediano-grande denso | Secuencias de eventos con timestamps |

**🔧 Aplicaciones donde el orden es clave:**

- **Customer journey / clickstream:** "buscar → comparar → agregar al carro → abandonar" es un patrón secuencial accionable (¿en qué paso se pierden?).
- **Logs de sistemas / IoT:** la secuencia "warning_disk → error_memory → crash" anticipa fallas con minutos de ventaja. El evento suelto no dice nada; la **secuencia** sí.
- **Diagnósticos médicos:** secuencias de síntomas que preceden a un diagnóstico — detección temprana basada en trayectoria del paciente.
- **Rutas de navegación educativa:** ¿en qué orden los estudiantes consumen contenido antes de aprobar/reprobar?

**🔧 Diferencia con reglas de asociación clásicas:**

| Aspecto | Reglas de asociación | Sequential patterns |
|---|---|---|
| Estructura de la transacción | Set (sin orden) | Lista ordenada de itemsets |
| Métrica base | Support de co-ocurrencia | Support de sub-secuencia |
| Pregunta que responde | "¿Qué va CON qué?" | "¿Qué va DESPUÉS de qué?" |
| Algoritmo referencia | FP-Growth | PrefixSpan |

**🧭 Cuándo usarlo:** cuando la **secuencia temporal de eventos** contiene información que la co-ocurrencia pierde. Si el orden no importa para tu problema (market basket puro), quédate con FP-Growth.

**👔 En una frase para el negocio:** no solo "qué se compra junto" sino "qué se compra después de qué" — la diferencia entre armar una góndola y diseñar un funnel.

---

## 📖 Referencias de este tomo

- (Agrawal & Srikant, 1994) — Apriori.
- (Han et al., 2000) — FP-Growth.
- (Zaki, 2000) — Eclat.
- (Srikant & Agrawal, 1996) — *Mining Sequential Patterns: Generalizations and Performance Improvements*. EDBT.
- (Pei et al., 2001) — *PrefixSpan: Mining Sequential Patterns Efficiently by Prefix-Projected Pattern Growth*. ICDE.

Fichas completas con datos de publicación en [[16-Bibliografia]].

---

**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[08-Metricas-de-Evaluacion|08 · Métricas de Evaluación]] · Siguiente: [[10-Validacion-y-Leakage|10 · Validación y Leakage ➡]]

> **Próximo tomo:** [[10-Validacion-y-Leakage]] — lo que separa un modelo real de una coincidencia fortuita: esquemas de partición, comparación estadística de modelos y el catálogo completo del data leakage.
