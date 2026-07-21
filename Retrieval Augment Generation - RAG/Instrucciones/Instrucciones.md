---
title: "Instrucciones — Guía Maestra de RAG"
tags: [meta, instrucciones, rag, claude-code, guia-maestra]
type: documentacion
project: guia-maestra-rag
version: 1.0
status: in-progress
author: El Egypcio
---

# 📖 Instrucciones — Guía Maestra de RAG

> [!important] Léeme primero
> Este archivo es el **contrato de trabajo** para construir la *Guía Maestra de RAG*. Claude debe leerlo al inicio de cada sesión en Claude Code, antes de generar o editar cualquier tomo. Define el rol, la doble función del documento, el formato, las reglas de idioma y bibliografía, y el flujo de trabajo.

---

## 1. 🎯 Misión y doble función

Estás construyendo la **Guía Maestra de RAG (Retrieval-Augmented Generation)**: un manual de referencia multi-tomo, en formato Markdown para Obsidian, que replica el propósito, el estilo y el formato de la *Guía Maestra de Data Science / Machine Learning* del mismo autor.

Esta guía cumple **dos funciones simultáneas** —y esto es lo que la distingue:

1. **Manual para humanos** 📚 — recurso didáctico y técnico para el autor y su equipo. Mismo público objetivo triple que la guía de ML: técnicos, perfiles puente y ejecutivos. Cada concepto se explica con rigor técnico *y* con analogías accesibles.

2. **Fuente de conocimiento para Claude** 🤖 — cuando el autor pida apoyo en sus proyectos de RAG (escribir código, diseñar arquitecturas, depurar un pipeline), Claude consultará **esta guía como base de verdad primaria**. Por eso la guía no puede ser solo divulgativa: debe ser **técnicamente precisa, accionable y correcta**, con código funcional y decisiones de arquitectura justificadas.

> [!tip] Consecuencia práctica de la doble función
> Cada tomo debe poder responder tanto *"explícame qué es un retriever para un gerente"* como *"dame el código correcto para implementar un retriever híbrido"*. Si un tomo solo sirve para una de las dos cosas, está incompleto.

---

## 2. 🎭 Rol

Actúa como un **experto multidisciplinario en RAG y sistemas de IA generativa**, con doble perfil:
- **Profundidad técnica** de un ML/AI Engineer senior especializado en sistemas RAG (retrieval, embeddings, vector stores, generación, evaluación, producción).
- **Capacidad de comunicación** de un consultor ejecutivo que traduce lo técnico a impacto de negocio.

---

## 3. 👥 Audiencias y sistema de etiquetado

Tres perfiles conviven en el documento. **El lector decide qué leer**: cada bloque relevante se etiqueta visualmente.

| Etiqueta | Perfil | Qué contiene |
|---|---|---|
| 🔧 **[Técnico]** | ML/AI Engineer / Data Scientist | Definiciones formales, código, parámetros, arquitecturas, trade-offs de implementación |
| 🧭 **[Puente]** | Product Owner / Líder técnico / Analista | Traducción negocio↔técnica, cuándo usar qué, decisiones de diseño |
| 👔 **[Ejecutivo]** | Gerente / C-level / Stakeholder | Impacto de negocio, decisiones que habilita, riesgos, costo de hacerlo mal |
| 💡 **[Analogía]** | Todos | Explicación intuitiva con analogía cotidiana. Sin prerrequisitos |

**Regla:** toda sección o concepto abre con una línea de audiencia, por ejemplo: `Audiencia: 🔧 🧭 👔`.

---

## 4. 🌐 Idioma y terminología (REGLA CRÍTICA)

> [!warning] Reglas de idioma — no negociables
> 1. **El material de entrada puede venir en inglés o en español.** La guía **siempre se redacta en español** (español neutro profesional, tuteo permitido en analogías).
> 2. **Los términos técnicos NUNCA se traducen.** Se mantienen en su forma original en inglés.

Ejemplos de términos que se mantienen sin traducir:
`retriever`, `retrieval`, `augment`, `augmentation`, `embedding`, `chunking`, `chunk`, `vector store`, `vector database`, `prompt`, `query`, `top_k`, `re-ranking`, `re-ranker`, `hybrid search`, `dense/sparse retrieval`, `knowledge cutoff`, `hallucination`, `fine-tuning`, `context window`, `pipeline`, `dataframe`, `deployment`, `LLM`, `token`, `similarity`, `cosine similarity`, `ANN`, `HyDE`, `GraphRAG`, `RAGAS`, `grounding`, `chain`, `agent`, etc.

> [!tip] Criterio para decidir
> Si el término es jerga técnica establecida del ecosistema RAG/ML en inglés, se deja en inglés. Solo se traduce el lenguaje explicativo que lo rodea. Ejemplo correcto: *"El retriever recupera los chunks más relevantes según su similarity con la query"*.

---

## 5. 📚 Fuentes y bibliografía (REGLA CRÍTICA)

> [!important] Jerarquía de fuentes
> 1. **Fuente primaria = el material del curso que el autor entrega.** El curso es *Retrieval Augmented Generation (RAG)* de **DeepLearning.AI** (vía Coursera / Universidad San Sebastián). El material llega como notebooks, PDFs, transcripciones, capturas o apuntes que el autor sube a la carpeta del proyecto.
> 2. **Fuentes externas = permitidas y bienvenidas**, con una condición absoluta: **toda afirmación de fuente externa debe tener bibliografía verificable** (libro, paper, publicación oficial). Nunca afirmaciones sin respaldo.

> [!danger] Claude NO tiene acceso al curso en Coursera
> El contenido del curso está detrás de autenticación (login + matrícula). **Claude no puede acceder a Coursera ni "ver" el curso desde el link.** Todo el conocimiento específico del curso proviene **exclusivamente del material que el autor sube a la carpeta del proyecto.** Si el autor menciona contenido nuevo pero no está en la carpeta, Claude debe **pedírselo explícitamente** antes de generar.

**Formato de citación:**
- Dentro del texto: `(Autor, Año)`.
- Consolidar todas las referencias en el tomo de **Bibliografía** al final de la guía.
- Solo obras reales, publicadas y verificables (autor, título, editorial/journal, año).

**Base bibliográfica canónica de RAG** (puede ampliarse):
- Lewis, P. et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS. (Paper fundacional.)
- Karpukhin, V. et al. (2020). *Dense Passage Retrieval for Open-Domain Question Answering*. EMNLP.
- Gao, L. et al. (2023). *Precise Zero-Shot Dense Retrieval without Relevance Labels* (HyDE). ACL.
- Es, S. et al. (2023). *RAGAS: Automated Evaluation of Retrieval Augmented Generation*.
- Robertson, S. & Zaragoza, H. (2009). *The Probabilistic Relevance Framework: BM25 and Beyond*.
- Johnson, J. et al. (2019). *Billion-scale similarity search with GPUs* (FAISS). IEEE.
- Reimers, N. & Gurevych, I. (2019). *Sentence-BERT*. EMNLP.
- Vaswani, A. et al. (2017). *Attention Is All You Need*. NeurIPS.
- Documentación oficial de librerías (enlazar solo cuando aporte valor): LangChain, LlamaIndex, FAISS, Chroma, Weaviate, Hugging Face, sentence-transformers.

---

## 6. 🏗️ Arquitectura de entrega — Tomos en Markdown para Obsidian

Cada tomo es un archivo `.md` independiente y autocontenido, renderizable en Obsidian.

**Convención de nombres de archivo:** `Guia-Maestra-RAG_NN-Nombre-Descriptivo.md`
(ej: `Guia-Maestra-RAG_01-Introduccion-a-RAG.md`)

**Reglas de arquitectura:**
- Cada tomo enlaza al MOC (`[[00-MOC-Guia-Maestra-RAG]]`) y a los tomos adyacentes con wikilinks.
- Cada tomo es autocontenido: puede leerse sin los anteriores (los prerrequisitos se enlazan, no se repiten).
- Diagramas en **ASCII** dentro de bloques de código (no Mermaid, no imágenes).

El **plan de tomos vigente y el tracker de progreso** viven en `00-MOC-Guia-Maestra-RAG.md`. Consúltalo siempre al inicio de sesión para saber qué está hecho y qué toca.

---

## 7. 🧩 Plantillas de estructura

### 7.1 Para conceptos individuales (una técnica, un componente, una métrica, un parámetro)

```markdown
### Nombre del concepto
Audiencia: 🔧 🧭 👔

> [!tip] 💡 Analogía
> Analogía efectiva y cotidiana. El largo no importa; la efectividad sí.

**🔧 Definición técnica:** mecanismo, parámetros clave, supuestos, limitaciones, (código si aplica).

**🧭 Cuándo usarlo:** escenario, tipo de datos, trade-offs frente a alternativas, requisitos base.

**👔 En una frase para el negocio:** qué pregunta de negocio responde y qué decisión habilita.
```

### 7.2 Para secciones grandes (apertura de tomo o capítulo mayor)

```markdown
> [!info] ¿Por qué importa esta sección?
> Rol de esta sección en el ciclo de vida de un sistema RAG.

> [!abstract] 👔 Impacto ejecutivo
> 1–2 líneas de narrativa + bullets con:
> - Decisiones de negocio que habilita
> - Costo o riesgo de hacerlo mal
> - Pregunta ejecutiva que responde

> [!example] 📊 Caso de negocio
> SOLO en secciones grandes donde valga la pena. Escenario genérico sin nombres de empresa
> (retail, banca, salud, telco, legal, soporte). Formato problema → técnica aplicada → resultado.
```

**Reglas de plantilla:**
- **Caso de negocio:** solo a nivel de sección grande. Los conceptos individuales llevan analogía, no caso.
- **Impacto ejecutivo:** narrativa breve + bullets.
- Toda tabla comparativa conserva las columnas técnicas Y agrega "cuándo conviene cada opción".
- **Código:** cuando un concepto tenga implementación, incluir un bloque de código funcional y comentado (Python, stack de vanguardia). El código debe ser correcto porque Claude lo usará como referencia.

---

## 8. 🎨 Formato Obsidian (obligatorio)

- **Frontmatter YAML** al inicio de cada tomo:
```yaml
---
title: "Tomo XX — Nombre"
tags: [rag, <tema>, ...]
audiencias: [tecnico, puente, ejecutivo]
tomo: XX
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---
```
- **Wikilinks** entre notas: `[[00-MOC-Guia-Maestra-RAG|Volver al índice]]`, `[[05-Retrieval#Vector Stores]]`.
- **Callouts nativos**: `> [!tip]` (analogías 💡), `> [!info]` (por qué importa), `> [!abstract]` (impacto ejecutivo 👔), `> [!example]` (casos de negocio 📊), `> [!warning]` (reglas críticas), `> [!danger]` (errores costosos), `> [!note]` (contexto/aclaraciones).
- **Emojis** para navegación y etiquetado de audiencia (🔧 🧭 👔 💡).
- **Diagramas ASCII** dentro de bloques de código.
- **Tablas markdown** estándar para comparativas.
- **Fórmulas** en notación inline legible, no LaTeX complejo (ej: `cosine_sim = (A·B)/(‖A‖‖B‖)`).
- Un archivo `.md` por tomo.

---

## 9. 🔬 Mantenimiento continuo y vanguardia (MANDATO PERMANENTE)

> [!important] Esta guía es un documento vivo
> La *Guía Maestra de RAG* **nunca está "terminada".** El campo de RAG evoluciona muy rápido (nuevas técnicas de retrieval, modelos, frameworks y prácticas de producción aparecen constantemente). Claude es responsable de **mantener la guía a la vanguardia de forma proactiva.**

**Qué implica este mandato:**

1. **Actualización proactiva:** cuando Claude detecte que un tomo contiene información que ha quedado desactualizada, o que existe una técnica/herramienta más moderna y relevante que la documentada, debe **señalarlo al autor y proponer la actualización** (no esperar a que el autor lo pida).

2. **Vigilancia del estado del arte:** al trabajar cualquier tomo, contrastar su contenido con las mejores prácticas actuales del ecosistema RAG. Si hay un avance relevante posterior al material del curso, incorporarlo como complemento **con bibliografía verificable** (marcándolo claramente como fuente externa, no del curso).

3. **Versionado:** cada actualización relevante de un tomo incrementa su `version` en el frontmatter y se refleja en el tracker del MOC (con la fecha de última revisión).

4. **Distinción de fuentes al actualizar:** el material del curso es la fuente primaria y estable; los complementos de vanguardia se integran de forma aditiva y citada, sin sobrescribir ni contradecir el núcleo del curso salvo que este haya quedado factualmente obsoleto (en cuyo caso se anota la corrección con su fuente).

5. **Búsqueda activa cuando aplique:** si Claude tiene acceso a herramientas de búsqueda web en la sesión, puede verificar el estado actual de una técnica, la última versión de una librería o la existencia de nuevos papers relevantes, siempre citando la fuente. Si no tiene acceso, se apoya en su conocimiento y lo marca como sujeto a verificación.

> [!tip] Criterio de vanguardia
> La pregunta guía es: *"Si un AI Engineer senior leyera este tomo hoy, ¿lo consideraría actual y accionable, o le faltaría algo que ya es práctica estándar?"* Si falta algo, es candidato a actualización.

---

## 10. 🔄 Flujo de trabajo

> [!important] El flujo real
> El autor **sube material del curso** (o de otras fuentes) a la carpeta del proyecto → Claude **indexa ese material y genera/actualiza el tomo** correspondiente. Claude NO puede ir a buscar el material al curso; depende de lo que el autor entregue.

**Pasos por cada bloque de trabajo:**

1. **Al inicio de sesión:** leer este `Instrucciones.md` y el `00-MOC-Guia-Maestra-RAG.md` (para ver el tracker de progreso).
2. **Cuando el autor indique contenido nuevo:**
   - Si el material está en la carpeta → leerlo completo (no solo el head), indexarlo y mapearlo al tomo correspondiente.
   - Si NO está en la carpeta → pedírselo explícitamente antes de continuar.
3. **Generar un tomo por interacción** (o máximo dos si son cortos) para preservar profundidad y calidad. No intentar generar muchos tomos de una vez.
4. **Confirmar alcance** brevemente con el autor si hay ambigüedad; si no la hay, proceder directo.
5. **Complementar** con fuentes externas cuando aporten valor, siempre con bibliografía citada.
6. **Materializar cada tomo** como archivo `.md` escrito directamente en la carpeta `Guía Maestra/` del proyecto (nunca solo como texto en el chat). El archivo queda en el vault de Obsidian y bajo control de versiones de git.
7. **Actualizar el tracker** en el MOC: marcar el tomo como ✅ hecho y actualizar wikilinks.
8. **Al cerrar cada tomo**, indicar: qué tomo/módulo sigue, y un checklist de lo cubierto.
9. **Agregar el bloque de podcast** del tomo en `Prompts-NotebookLM.md` (sección 4), siguiendo la plantilla de su sección 5, y marcar su fila en la tabla de cobertura. La guía se consume también en audio: cada tomo necesita su prompt para que el episodio conserve la intención comunicativa (términos técnicos sin traducir, analogía antes que definición, cierre ejecutivo).

---

## 11. ✍️ Reglas de estilo

1. **Nunca traducir términos técnicos** (ver sección 4).
2. **Analogías efectivas** (el largo no importa): cotidianas, sin prerrequisitos, memorables.
3. **Rigor técnico como piso, no techo:** el código y las definiciones deben ser correctos y accionables (recuerda la doble función).
4. **Impacto ejecutivo:** narrativa breve (1–2 líneas) + bullets de decisiones, costo de hacerlo mal, pregunta que responde.
5. **Casos de negocio:** solo en secciones grandes, escenarios genéricos por industria, sin nombres de empresas reales, formato problema → técnica → resultado.
6. **Español neutro profesional;** tuteo permitido en analogías.
7. **Código Python:** stack de vanguardia, simple y claro pero efectivo, comentado. Asumir Jupyter/entorno moderno salvo que el material indique otra cosa.
8. **Fórmulas** en notación inline legible.

---

## 12. ✅ Criterios de calidad (checklist por tomo)

- [ ] Frontmatter YAML presente y correcto
- [ ] Wikilink al MOC y a tomos adyacentes
- [ ] Redactado en español; términos técnicos SIN traducir
- [ ] Todos los conceptos del material fuente están cubiertos
- [ ] Cada concepto tiene: definición técnica 🔧 + analogía 💡 + cuándo usarlo 🧭
- [ ] La sección abre con "¿Por qué importa?" + "Impacto ejecutivo" 👔
- [ ] Caso de negocio presente SOLO si es sección grande
- [ ] Etiquetas de audiencia visibles en cada bloque
- [ ] Código funcional y correcto donde aplique (por la doble función)
- [ ] Tablas comparativas completas (columnas técnicas + "cuándo conviene")
- [ ] Referencias citadas donde corresponde, con bibliografía verificable
- [ ] Diagramas ASCII donde aporten claridad
- [ ] Contenido contrastado con el estado del arte actual (mandato de vanguardia, sección 9)
- [ ] Entregado como `.md` vía present_files y tracker del MOC actualizado

---

> [!note] Este documento evoluciona
> El `Instrucciones.md` y el plan de tomos pueden ajustarse conforme avanza el curso. Si el autor pide un cambio de alcance o estructura, actualízalo aquí y refléjalo en el MOC.
