---
title: "Tomo 00 — MOC · Guía Maestra AWS · Arquitecto de Soluciones"
tags: [aws, solutions-architect, moc, indice]
audiencias: [tecnico, puente, ejecutivo]
tomo: 00
version: 1.0
updated: 2026-11-10
type: vault-index
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate (Amazon Web Services)"
---

# 🗺️ Guía Maestra AWS — Arquitecto de Soluciones · Índice Maestro (MOC)

Audiencia: 🔧 🧭 👔

> [!info] 📌 ¿Qué es esta guía?
> Un manual de estudio, en español, que reúne **todo lo que muestra** el Certificado Profesional *AWS Cloud Solutions Architect* de Coursera: los 4 cursos, sus videos, lecturas, demos y laboratorios, ordenados en tomos. Sirve a la vez como **referencia técnica** (servicios, límites, decisiones de arquitectura) y como **preparación** para el examen *AWS Certified Solutions Architect – Associate (SAA-C03)*.
>
> Cada tomo responde tres preguntas:
>
> 1. **¿Qué es?** — el cómo técnico 🔧
> 2. **¿Para qué sirve?** — cuándo usar qué 🧭
> 3. **¿Por qué debería importarme?** — el impacto 👔

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** elegir servicios de cómputo, almacenamiento, base de datos y analítica con criterio de costo, seguridad y resiliencia; diseñar un data lake gobernado.
> - **Costo de no tenerla:** arquitecturas sobredimensionadas, cuentas sin protección del root user, permisos amplios y data lakes sin catálogo ni control de acceso.
> - **Pregunta que responde:** *¿cómo diseño en AWS una solución segura, resiliente, eficiente y económica, y cómo lo demuestro en la certificación?*

---

## 🧭 Cómo usar esta guía

| Etiqueta | Perfil | Qué contiene |
|---|---|---|
| 🔧 **[Técnico]** | Data Engineer / Cloud Engineer / Arquitecto | Servicios, configuraciones, límites, pasos de demos y labs |
| 🧭 **[Puente]** | Líder técnico / Analista | Cuándo usar qué, trade-offs, criterios de decisión |
| 👔 **[Ejecutivo]** | Jefatura / Stakeholder | Impacto, riesgos y costo de hacerlo mal |
| 💡 **[Analogía]** | Todos | Intuición con una analogía cotidiana |

| Callout | Emoji | Significado |
|---|---|---|
| `[!tip]` | 💡 | Analogía |
| `[!info]` | 📌 | Por qué importa |
| `[!abstract]` | 👔 | Impacto ejecutivo |
| `[!example]` | 📊 | Caso o escenario del curso |
| `[!warning]` | ⚠️ | Regla crítica |
| `[!danger]` | 🚨 | Error costoso |
| `[!note]` | ➕ | Nota del editor (no viene del curso) |
| `[!question]-` | ✅ | Autoevaluación plegable |

Las líneas **🎯 Para el examen** marcan lo que el curso señala como relevante para la certificación.

---

## 📚 Mapa de tomos

### Curso 1 · AWS Cloud Technical Essentials (≈21 h)
| Tomo | Tema |
|---|---|
| [[01-Vision-General-y-Seguridad]] | Nube, infraestructura global, formas de interactuar, responsabilidad compartida, root user, IAM |
| [[02-Computo-y-Redes]] | EC2, contenedores, serverless, VPC y redes |
| [[03-Almacenamiento-y-Bases-de-Datos]] | Instance store, EBS, S3, RDS, DynamoDB |
| [[04-Monitoreo-y-Optimizacion]] | CloudWatch, ELB, EC2 Auto Scaling, rediseño serverless e IA generativa (Bedrock, Amazon Q) |

### Curso 2 · Architecting Solutions on AWS (≈22 h)
| Tomo | Tema |
|---|---|
| [[05-Backend-Web-Serverless]] | Backend web sin servidor |
| [[06-Analitica-de-Datos-Serverless]] | Analítica de datos sin servidor |
| [[07-Solucion-Hibrida-Contenedores]] | Solución híbrida con contenedores |
| [[08-Gobierno-y-Multicuenta]] | Gobierno y gestión de cuentas |

### Curso 3 · Introduction to Designing Data Lakes on AWS (≈11 h)
| Tomo | Tema |
|---|---|
| [[09-Introduccion-a-Data-Lakes]] | Qué es un data lake |
| [[10-Ingesta-Catalogo-y-Preparacion]] | Ingesta, catalogación y preparación |
| [[11-Data-Lake-con-Lake-Formation]] | Construir el lake con AWS Lake Formation |
| [[12-Procesamiento-y-Analisis]] | Procesamiento y análisis |
| [[13-Lake-Formation-Avanzado]] | Configuraciones y capacidades adicionales de Lake Formation |
| [[14-Arquitectura-de-Datos-Moderna]] | Arquitectura de datos moderna |

### Curso 4 · Exam Prep: AWS Certified Solutions Architect – Associate (≈8 h)
| Tomo | Tema |
|---|---|
| [[15-Examen-SAA-Estrategia]] | SAA-C03: 65 preguntas, 130 min; estrategia y cómo medir la preparación |
| [[16-Dominio-1-Arquitecturas-Seguras]] | D1 · Design Secure Architectures (30 %) |
| [[17-Dominio-2-Arquitecturas-Resilientes]] | D2 · Design Resilient Architectures (26 %) |
| [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento]] | D3 · Design High-Performing Architectures (24 %) |
| [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo]] | D4 · Design Cost-Optimized Architectures (20 %) |

### Anexos
| Tomo | Tema |
|---|---|
| [[20-Glosario]] | Términos y servicios |
| [[21-Recursos-y-Referencias]] | Enlaces oficiales citados por el curso |

---

## 🛣️ Rutas de lectura

- **Certificación SAA:** 01 → 04, luego 05 → 08, y cerrar con 15 → 19 (usar los cuadros «Dónde se ve en esta guía» de cada dominio).
- **Ingeniería de datos:** 01 (IAM) → 03 (S3) → 06 → 09 → 14.
- **Lectura ejecutiva:** los callouts 👔 y el «Resumen en 10 líneas» de cada tomo.

> [!note] ➕ Alcance y límites del material
> - Los **laboratorios, quizzes y actividades interactivas** no tienen texto legible: en esos casos los tomos registran el objetivo y los pasos que se ven en los videos de introducción y en las demos.
> - Las transcripciones en español vienen de traducción automática. Cuando el curso trae un error evidente (cifras, términos o JSON traducidos), el tomo lo corrige o lo marca en una **Nota del editor** o en el callout «Datos que conviene verificar».

> [!warning] ⚠️ Vigencia
> AWS cambia rápido: consolas, límites y precios pueden diferir de lo que muestra el curso. Ante la duda, manda la documentación oficial ([[21-Recursos-y-Referencias]]).

---

*Fuente: material del Certificado Profesional AWS Cloud Solutions Architect (Coursera / Amazon Web Services), sintetizado y parafraseado para estudio personal. Revisado al 2026-11-10.*
