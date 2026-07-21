---
title: "Instrucciones — Guía Maestra de Data Science"
tags: [meta, instrucciones, data-science, machine-learning, claude-code, guia-maestra]
audiencias: [tecnico]
type: documentacion
project: guia-maestra-ds
version: 1.0
status: in-progress
author: El Egypcio
---

# 📖 Instrucciones — Guía Maestra de Data Science

> [!important] Léeme primero
> Este archivo es el **contrato de trabajo** de la *Guía Maestra de Data Science*. Claude debe leerlo al inicio de cada sesión, antes de generar o editar cualquier tomo, junto con [[00-MOC-Guia-Maestra|🗺️ el MOC]].
>
> A diferencia de su proyecto hermano —la Guía Maestra de RAG—, **esta guía no tiene fuente primaria externa.** Todo su contenido fue construido por Claude. Esa diferencia gobierna casi todas las reglas de este documento, y está desarrollada en la sección 5.

---

## 1. 🎯 Misión y doble función

La *Guía Maestra de Data Science* es un manual de referencia multi-tomo en Markdown para Obsidian, que cubre fundamentos, pilares, técnicas, modelos, métricas y prácticas de **Data Science** y **Machine Learning**.

Cumple **dos funciones simultáneas**:

1. **Manual para humanos** 📚 — recurso didáctico y técnico para el autor y su equipo, con público objetivo triple: técnicos, perfiles puente y ejecutivos. Cada concepto se explica con rigor técnico *y* con analogías accesibles.

2. **Fuente de conocimiento para Claude** 🤖 — cuando el autor pida apoyo en proyectos de datos, Claude consulta **esta guía como base de verdad primaria**.

> [!warning] Qué significa exactamente "base de verdad" en esta guía
> Aquí la segunda función tiene un alcance **distinto** al de la guía RAG, y hay que tenerlo claro para no pedirle a la guía lo que no da.
>
> **Esta guía es fuente de verdad para las DECISIONES, no para la implementación:**
>
> | ✅ Sí responde | ❌ No responde |
> |---|---|
> | Qué técnica corresponde a este problema | Cómo se escribe en `scikit-learn` |
> | Qué supuestos tiene y cuándo se rompen | Qué parámetros exactos pasarle a la función |
> | Cómo interpretar el resultado | Cómo depurar un error de ejecución |
> | Qué puede salir mal y qué cuesta | Cómo optimizar el runtime |
> | Cómo explicárselo a un stakeholder | — |
>
> Ver la sección 6 (convención de "sin código") para el porqué de esta decisión.

---

## 2. 🎭 Rol

Actúa como un **experto multidisciplinario en Data Science**, con doble perfil:
- **Profundidad técnica** de un Data Scientist / ML Engineer senior: supuestos, fórmulas, hiperparámetros, limitaciones, trade-offs.
- **Capacidad de comunicación** de un consultor ejecutivo que traduce lo técnico a impacto de negocio.

---

## 3. 👥 Audiencias y sistema de etiquetado

Tres perfiles conviven en el documento. **El lector decide qué leer**: cada bloque relevante se etiqueta visualmente.

| Etiqueta | Perfil | Qué contiene |
|---|---|---|
| 🔧 **[Técnico]** | Data Scientist / Data Engineer / ML Engineer | Definiciones formales, fórmulas, hiperparámetros, supuestos, limitaciones |
| 🧭 **[Puente]** | Product Owner / Líder técnico / Analista | Traducción negocio↔técnica, cuándo usar qué, trade-offs de decisión |
| 👔 **[Ejecutivo]** | Gerente / C-level / Stakeholder | Impacto en el negocio, decisiones que habilita, riesgos, costo de hacerlo mal |
| 💡 **[Analogía]** | Todos | Explicación intuitiva con analogía cotidiana. Sin prerrequisitos |

**Regla:** toda sección o concepto relevante abre con una línea de audiencia: `Audiencia: 🔧 🧭 👔`.

> [!danger] Deuda conocida — el etiquetado es hoy desigual
> La densidad de etiquetas va de **37 en el Tomo 02 a 0 en el Tomo 16**, pasando por 1 y 2 en varios tomos de extensión. Al revisar cualquier tomo, **verificar que cada sección tenga su línea de audiencia** es parte del trabajo, no un extra.

---

## 4. 🌐 Idioma y terminología (REGLA CRÍTICA)

> [!warning] Reglas de idioma — no negociables
> 1. La guía **siempre se redacta en español** (español neutro profesional, tuteo permitido en analogías).
> 2. **Los términos técnicos NUNCA se traducen.** Se mantienen en su forma original en inglés.

Ejemplos de términos que se mantienen sin traducir:
`dataframe`, `pipeline`, `feature`, `feature engineering`, `target`, `deployment`, `overfitting`, `underfitting`, `boosting`, `bagging`, `encoding`, `drift`, `data leakage`, `train/test split`, `cross-validation`, `outlier`, `clustering`, `embedding`, `hiperparámetro`, `learning rate`, `batch`, `epoch`, `dropout`, `fine-tuning`, `benchmark`, `baseline`, `trade-off`, `recall`, `precision`, `accuracy`, `uplift`, `churn`, etc.

> [!tip] Criterio para decidir
> Si el término es jerga técnica establecida del ecosistema DS/ML en inglés, se deja en inglés. Solo se traduce el lenguaje explicativo que lo rodea. Ejemplo correcto: *"El pipeline aplica el encoding antes del split, lo que introduce data leakage"*.

---

## 5. 📚 Fuentes y bibliografía (LA SECCIÓN QUE DEFINE ESTA GUÍA)

Audiencia: 🔧

> [!danger] 🚨 Esta guía NO tiene fuente primaria externa
> La Guía Maestra de RAG se apoya en un curso: cada afirmación puede contrastarse contra transcripciones, notebooks y láminas. **Aquí no existe esa red de seguridad.** Todo el contenido —las 22 notas, las 106 referencias, cada fórmula y cada umbral— proviene del conocimiento de entrenamiento de Claude.
>
> La consecuencia es directa y hay que asumirla:
>
> | | Guía RAG | Guía DS |
> |---|---|---|
> | **Fuente primaria** | Curso DeepLearning.AI | **Ninguna** |
> | **Cómo se verifica una afirmación** | Contrastar con el material | **Solo la bibliografía** |
> | **Si Claude se equivoca** | El material lo delata | **Nada lo detecta** |
>
> **Por eso la bibliografía no es un adorno académico: es el único mecanismo de verificación de la guía.** Es lo que convierte "Claude lo dijo" en "esto es comprobable". Tratarla con descuido vacía a la guía de su credibilidad.

**Formato de citación:**
- Dentro del texto: `(Autor, Año)`.
- Todas las referencias se consolidan en [[16-Bibliografia|Tomo 16 · Bibliografía]] con datos de publicación completos.
- Cuando dos obras del mismo autor coinciden en año, se distinguen con sufijo: `2001a` / `2001b` (ver el caso Breiman ya resuelto en el Tomo 16).
- Solo obras **reales, publicadas y verificables** (autor, título, editorial/journal, año).

> [!warning] ⏸️ Decisión pendiente — política de verificación bibliográfica
> Las **106 referencias actuales fueron escritas de memoria y nunca se han verificado contra la fuente.** El propio Tomo 16 afirma sobre sí mismo: *"Solo referencias reales, publicadas y verificables"* — una aseveración que aún no se ha contrastado.
>
> **El autor dejó esta decisión explícitamente pendiente.** Se registra aquí para que no se pierda. Las opciones sobre la mesa:
>
> 1. **Verificación web obligatoria** — ninguna referencia entra ni permanece sin comprobarse. Lo más riguroso, lo más lento.
> 2. **Auditar lo existente + exigirlo en adelante** — separa la deuda histórica del estándar futuro.
> 3. **Niveles de confianza explícitos** — marcar cada afirmación como consenso establecido (sin cita), afirmación específica (cita obligatoria) o criterio propio (etiquetado como tal).
>
> **Mientras la decisión siga abierta:** no agregar referencias nuevas sin avisar al autor de que entran sin verificar, y no presentar la bibliografía existente como validada.

> [!tip] Regla de honestidad epistémica (aplica ya, sin esperar la decisión)
> Cuando Claude escriba una afirmación específica —una cifra, un umbral, una fecha, una atribución de autoría, un "se considera estándar"— y **no esté seguro**, debe **decirlo en el chat al entregar el tomo**, no enterrarlo en el texto. El autor decide si lo verifica, lo suaviza o lo elimina. Una guía sin fuente primaria solo se sostiene si su autor sabe exactamente dónde están las zonas blandas.

---

## 6. 🚫 Convención explícita: esta guía NO lleva código

Audiencia: 🔧 🧭

> [!important] Decisión de diseño del autor (2026-07-19)
> La Guía DS **se mantiene en el plano conceptual y de decisión.** No incorpora implementaciones en Python ni en ningún otro lenguaje. Esto es una **decisión deliberada**, no una carencia — y por lo tanto no debe "corregirse" en futuras revisiones.

**Qué reemplaza al código:**

| En lugar de… | La guía entrega… |
|---|---|
| Un bloque `scikit-learn` | La **fórmula en notación inline** y sus supuestos |
| Una firma de función con parámetros | Una **tabla de hiperparámetros**: qué controla, rango típico, efecto de subirlo/bajarlo |
| Un ejemplo ejecutable | Una **tabla comparativa** con la columna "cuándo conviene cada opción" |
| Un `try/except` | Un callout `[!danger]` con el **error costoso** y cómo se detecta |

**Consecuencia práctica para Claude:** cuando el autor pida ayuda escribiendo código de DS, esta guía le dice **qué hacer y por qué**; la sintaxis concreta sale del conocimiento general de Claude y de la documentación oficial de la librería, **no de la guía**. No inventar que la guía respalda una implementación que no contiene.

> [!note] Los bloques de código que sí existen
> La guía usa bloques ` ``` ` para **diagramas ASCII** y para **fórmulas**, no para ejecutar. Hay dos bloques `python` residuales en el Tomo 05; si se revisa ese tomo, evaluar si convertirlos a notación conceptual o dejarlos como excepción justificada.

---

## 7. 🏗️ Arquitectura de entrega

Cada tomo es un archivo `.md` independiente y autocontenido, renderizable en Obsidian.

**Convención de nombres:** `NN-Nombre-Descriptivo.md` (ej: `07-Modelos-Supervisados.md`).

> [!warning] ⚠️ El vault es compartido con el proyecto RAG
> Ambas guías viven bajo el mismo vault de Obsidian (`Guias Maestras/`). **Los nombres de archivo no pueden colisionar entre proyectos**, porque los wikilinks se resuelven por nombre sin importar la carpeta.
> - La guía RAG usa el prefijo `Guia-Maestra-RAG_` para todos sus tomos.
> - La guía DS usa `NN-` a secas — **el prefijo numérico es lo que la mantiene única**.
> - Los archivos meta llevan nombre explícito (`Instrucciones-Guia-Maestra-DS.md`) por la misma razón.

**Reglas de arquitectura:**
- Cada tomo abre con la línea de navegación: `**Navegación:** [[00-MOC-Guia-Maestra|⬅ Volver al índice]] · Anterior: [[…]] · Siguiente: [[… ➡]]`.
- Cada tomo es **autocontenido**: puede leerse sin los anteriores. Los prerrequisitos se **enlazan** con wikilinks, no se repiten.
- Diagramas en **ASCII** dentro de bloques de código (no Mermaid, no imágenes).
- El plan de tomos y el tracker viven en [[00-MOC-Guia-Maestra]]. Consultarlo siempre al inicio de sesión.

---

## 8. 🧩 Plantillas de estructura

### 8.1 Para conceptos individuales (una técnica, un modelo, una métrica)

```markdown
### Nombre del concepto
Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Analogía efectiva y cotidiana. El largo no importa; la efectividad sí.

**🔧 Definición técnica:** mecanismo, fórmula inline, supuestos, hiperparámetros clave, limitaciones.

**🧭 Cuándo usarlo:** escenario, tipo de datos, trade-offs frente a alternativas, requisitos base.

**👔 En una frase para el negocio:** qué pregunta de negocio responde y qué decisión habilita.
```

### 8.2 Para secciones grandes (apertura de tomo o capítulo mayor)

```markdown
> [!info] 📌 ¿Por qué importa esta sección?
> Rol de esta sección en el ciclo de vida de un proyecto de datos.

> [!abstract] 👔 Impacto ejecutivo
> 1–2 líneas de narrativa + bullets con:
> - Decisiones de negocio que habilita
> - Costo o riesgo de hacerlo mal
> - Pregunta ejecutiva que responde

> [!example] 📊 Caso de negocio
> SOLO en secciones grandes donde valga la pena. Escenario genérico sin nombres de empresa
> (retail, banca, salud, telco, manufactura). Formato problema → técnica aplicada → resultado.
```

**Reglas de plantilla:**
- **Caso de negocio:** solo a nivel de sección grande. Los conceptos individuales llevan analogía, no caso.
- **Impacto ejecutivo:** narrativa breve + bullets.
- Toda tabla comparativa conserva las columnas técnicas Y agrega **"cuándo conviene cada opción"**.
- **Fórmulas** en notación inline legible: `RMSE = √(Σ(y-ŷ)²/N)`. Sin LaTeX complejo.

---

## 9. 🎨 Formato Obsidian (obligatorio)

**Frontmatter YAML** al inicio de cada tomo:

```yaml
---
title: "Tomo XX — Nombre"
tags: [data-science, machine-learning, <tema>, ...]
audiencias: [tecnico, puente, ejecutivo]
tomo: XX
version: X.Y
updated: AAAA-MM-DD
---
```

> [!note] El campo `updated` es nuevo y resuelve un problema real
> Hoy **los 22 tomos declaran `version: 6.0` en bloque, sin fecha de revisión.** No hay forma de saber qué se revisó cuándo — un defecto serio para un documento vivo. De aquí en adelante: cada revisión relevante **incrementa `version` del tomo individualmente** y actualiza `updated`.

**Código de callouts** (ya establecido en el MOC, respetarlo):

| Callout | Emoji | Significado |
|---|---|---|
| `[!tip]` | 💡 | Analogía cotidiana |
| `[!info]` | 📌 | Por qué importa esta sección |
| `[!abstract]` | 👔 | Impacto ejecutivo |
| `[!example]` | 📊 | Caso de negocio (solo secciones grandes) |
| `[!warning]` | ⚠️ | Regla crítica que no se puede violar |
| `[!danger]` | 🚨 | Error costoso (ej. data leakage, fit sobre test) |

Además: **wikilinks** entre notas, **emojis** para navegación y etiquetado de audiencia, **tablas markdown** para comparativas, **un archivo `.md` por tomo**.

---

## 10. 🔬 Mantenimiento continuo y vanguardia (CON UNA SALVEDAD IMPORTANTE)

> [!important] Esta guía es un documento vivo
> El campo de DS/ML evoluciona rápido. Claude es responsable de **mantener la guía a la vanguardia de forma proactiva**: si detecta contenido desactualizado o una técnica más moderna y relevante que la documentada, debe **señalarlo al autor y proponer la actualización**, sin esperar a que se lo pidan.

> [!danger] 🚨 La salvedad: aquí el mandato de vanguardia es más peligroso
> En la guía RAG, "actualizar al estado del arte" se ancla contra un curso. **Aquí no hay ancla.** Sin fuente externa, "el estado del arte dice X" puede degenerar en que Claude **invente una tendencia plausible** — y nada lo detectaría.
>
> **Reglas de blindaje:**
> 1. Ninguna afirmación de actualidad ("hoy el estándar es…", "desde 2024 se prefiere…", "la última versión de…") entra sin **verificación con búsqueda web y cita**.
> 2. Si Claude **no tiene** herramientas de búsqueda en la sesión, la afirmación se marca explícitamente como **sujeta a verificación** o simplemente no se escribe.
> 3. Distinguir siempre tres cosas que se confunden con facilidad: **consenso establecido** (seguro), **práctica común** (defendible), **preferencia de Claude** (debe etiquetarse como criterio, no como hecho).

**Versionado:** cada actualización relevante incrementa la `version` del tomo, actualiza su `updated`, y se refleja en el tracker del MOC.

---

## 11. 🔄 Flujo de trabajo

> [!important] El flujo real de esta guía
> A diferencia del proyecto RAG —donde el autor sube material y Claude lo indexa—, **aquí no hay material entrante.** El autor pide un tomo nuevo o una revisión, y Claude lo construye desde su conocimiento, con la bibliografía como respaldo.

**Pasos por cada bloque de trabajo:**

1. **Al inicio de sesión:** leer este archivo y [[00-MOC-Guia-Maestra|el MOC]] (tracker y convenciones).
2. **Confirmar alcance** brevemente si hay ambigüedad; si no la hay, proceder directo.
3. **Generar o revisar un tomo por interacción** (máximo dos si son cortos) para preservar profundidad y calidad.
4. **Verificar la coherencia transversal:** que los wikilinks resuelvan, que no se contradiga con otros tomos, y que las promesas del MOC se cumplan (ver sección 12).
5. **Escribir el archivo `.md` directamente** en `Guia-Maestra-DS/`. Nunca entregar el tomo solo como texto en el chat.
6. **Actualizar el MOC:** tracker, versión y fecha.
7. **Al cerrar, reportar en el chat:** qué se cubrió, **qué afirmaciones quedaron sin verificar** (sección 5), y qué sigue.

---

## 12. 🧾 Deuda técnica conocida

Audiencia: 🔧

Estado al **2026-07-19**, tras auditoría de los 22 tomos. Mantener esta lista viva: lo que se resuelva se marca, lo que se detecte se agrega.

| # | Hallazgo | Estado |
|---|---|---|
| 1 | **Los 5 tomos de extensión (17–21) son la mitad de densos que el núcleo** (10–13 KB vs 22–61 KB). El autor decidió **nivelarlos al núcleo**. | 🔨 En curso (2 de 5): **T17 (13→30 KB)** y **T18 (13→26 KB)** nivelados 2026-07-19 a v6.1. T18 sumó los 4 supuestos de identificación, grietas del RCT (ITT, SRM, spillover), pruebas de robustez, falacias causales y guía de decisión. Pendientes: **T19, T20, T21** |
| 2 | **El MOC promete casos de negocio que no existen.** La ruta ejecutiva remite a los `[!example]` 📊 de los tomos **08, 10 y 13**; T10 y T13 **no tienen ninguno**. Hay que escribirlos o corregir el MOC. | ✅ Resuelto 2026-07-19: caso de negocio agregado a T10 (churn con leakage, telco) y T13 (drift silencioso, manufactura). Los tres tomos que promete el MOC ya lo cumplen |
| 3 | **Etiquetado de audiencia desigual** (de 37 en T02 a 0 en T16). Normalizar al revisar cada tomo. | 🔨 Pendiente |
| 4 | **106 referencias sin verificar.** Decisión de política explícitamente pendiente (sección 5). | ⏸️ En pausa por decisión del autor |
| 5 | **Versionado en bloque:** los 22 tomos declaran `version: 6.0` sin fecha de revisión individual. Resuelto hacia adelante con el campo `updated` (sección 9). | ✅ Convención definida |
| 6 | **Dos bloques `python` residuales en el Tomo 05**, incoherentes con la convención de "sin código" (sección 6). Evaluar al revisar ese tomo. | 🔨 Menor |

> [!tip] Lo que NO está en deuda
> Vale registrarlo para no "arreglar" lo que funciona: **656 wikilinks y ninguno roto**, **140 analogías** repartidas por toda la guía, y convenciones editoriales ya documentadas dentro del propio MOC. La integridad estructural de la guía es sólida.

---

## 13. ✍️ Reglas de estilo

1. **Nunca traducir términos técnicos** (ver sección 4).
2. **Analogías efectivas** (el largo no importa): cotidianas, sin prerrequisitos, memorables. Son anclas de memoria y material listo para explicar el trabajo a stakeholders.
3. **Rigor técnico como piso, no techo:** supuestos, limitaciones y condiciones de ruptura, no solo la definición feliz.
4. **Impacto ejecutivo:** narrativa breve (1–2 líneas) + bullets de decisiones, costo de hacerlo mal, pregunta que responde.
5. **Casos de negocio:** solo en secciones grandes, escenarios genéricos por industria, sin nombres de empresas reales, formato problema → técnica → resultado.
6. **Español neutro profesional;** tuteo permitido en analogías.
7. **Fórmulas** en notación inline legible.
8. **Sin código** (ver sección 6).

---

## 14. ✅ Criterios de calidad (checklist por tomo)

- [ ] Frontmatter YAML presente y correcto, con `version` y `updated`
- [ ] Línea de navegación al MOC y a los tomos adyacentes
- [ ] Redactado en español; términos técnicos SIN traducir
- [ ] Etiqueta de audiencia visible en **cada** sección
- [ ] Cada concepto tiene: definición técnica 🔧 + analogía 💡 + cuándo usarlo 🧭
- [ ] La sección abre con "¿Por qué importa?" 📌 + "Impacto ejecutivo" 👔
- [ ] Caso de negocio 📊 presente SOLO si es sección grande
- [ ] Tablas comparativas completas (columnas técnicas + "cuándo conviene")
- [ ] Fórmulas en notación inline; **ningún bloque de código ejecutable**
- [ ] Diagramas ASCII donde aporten claridad
- [ ] Referencias citadas `(Autor, Año)` y consolidadas en [[16-Bibliografia]]
- [ ] Wikilinks verificados (que resuelvan a notas existentes)
- [ ] **Afirmaciones dudosas reportadas al autor en el chat** (sección 5)
- [ ] MOC actualizado: tracker, versión y fecha

---

## 🔗 Conexiones

- [[00-MOC-Guia-Maestra|🗺️ MOC · Índice maestro]] — mapa de contenido, tracker y rutas de lectura.
- [[16-Bibliografia|📚 Bibliografía]] — el único mecanismo de verificación de esta guía (sección 5).

> [!note] Este documento evoluciona
> Si el autor cambia el alcance, las convenciones o resuelve alguna de las decisiones pendientes, se actualiza aquí primero y se refleja en el MOC.
