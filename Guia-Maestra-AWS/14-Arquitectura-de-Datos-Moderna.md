---
title: "Tomo 14 — Arquitectura de Datos Moderna en AWS"
tags: [aws, solutions-architect, data-lake, modern-data-architecture, data-mesh, datazone, well-architected]
curso: 3 · Introduction to Designing Data Lakes on AWS
modulo: "Módulo 6: Arquitectura de datos moderna en AWS"
audiencias: [tecnico, puente, ejecutivo]
tomo: 14
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🧭 Tomo 14 — Arquitectura de Datos Moderna en AWS

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[13-Lake-Formation-Avanzado|13 · Lake Formation Avanzado]] · Siguiente: [[15-Examen-SAA-Estrategia|15 · Examen SAA: Estrategia ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Cierra el curso de data lakes subiendo de nivel: del lago aislado a la **arquitectura de datos moderna**, donde el lago convive con data warehouse y almacenes especializados (búsqueda, NoSQL, grafos) bajo un gobierno unificado. Presenta los **cinco pilares** con su "hoja de servicios", los tres patrones de movimiento de datos (**inside-out, outside-in, around the perimeter**), la idea de **datos como producto** y **data mesh** con **Amazon DataZone**, y termina con las recomendaciones del **Well-Architected Data Analytics Lens**. Se inspira en el whitepaper de AWS *Modern Data Architecture Rationales on AWS*.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** qué servicio especializado usar para cada caso, hacia dónde deben fluir los datos, cómo repartir la propiedad de los datos entre áreas.
> - **Costo de hacerlo mal:** silos de datos, copias desordenadas entre almacenes que crecen sin control, nadie responsable de la calidad de cada dataset, decisiones con información desactualizada.
> - **Pregunta que responde:** *¿cómo organizo lago, warehouse y bases especializadas para que toda la empresa use la misma información actualizada, con gobierno y a bajo costo?*

> [!tip] 💡 Analogía general: el lago como plaza central
> El lago es la plaza central de la ciudad. Algunos productos salen de la plaza hacia tiendas especializadas (inside-out), otros llegan desde fuera para venderse en ella (outside-in), y a veces dos tiendas negocian directamente por la calle lateral sin pasar por la plaza (around the perimeter). La plaza sigue en el centro; lo que cambia es el sentido del tránsito.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Objetivos del módulo | 1 video | Lago centralizado, eliminar silos, whitepaper de referencia |
| Arquitectura moderna de datos | 1 video + 1 lectura | Los 5 pilares y sus servicios |
| Escenarios de movimiento de datos | 1 video + 1 lectura | Inside-out, outside-in, around the perimeter |
| Modelos de intercambio de datos | 1 video | Datos como producto, data mesh, Amazon DataZone, multicuenta |
| Charla técnica: función favorita | 1 video | QuickSight y formatos columnares (min/max por bloque en Parquet) |
| Resumen del curso | 1 video | Well-Architected Framework y Data Analytics Lens |
| Encuesta posterior al curso | 1 lectura + 1 encuesta | Sin contenido técnico |

---

## 1. Lago centralizado y compartido

Audiencia: 🧭 👔

Un patrón de diseño común es un **data lake centralizado** accesible desde distintos departamentos. Centralizar y compartir **elimina silos** y garantiza que todos accedan a la **misma información actualizada**.

La lectura define la arquitectura de datos moderna como la que **integra un data lake, un data warehouse y otros almacenes de propósito específico**, con **gobierno unificado** y **movimiento de datos fluido**. El problema que resuelve: las organizaciones llevan datos de varios silos al lago para analítica e IA/ML, pero también tienen datos en almacenes especializados (NoSQL, búsqueda, warehouse); moverlos dentro, fuera y entre ellos se vuelve complejo y desordenado a medida que crecen.

---

## 2. Los cinco pilares y sus servicios

Audiencia: 🔧 🧭

| Pilar | Objetivo | Servicios que propone el curso |
|---|---|---|
| **1. Scalable data lakes** | Almacenamiento seguro y gestión de metadatos de datasets diversos | **Amazon S3** (storage), **AWS Lake Formation** (montar el lago), **Amazon Athena** (consultar directo en S3) |
| **2. Purpose-built analytics services** | Emparejar cada caso de uso con el almacén y servicio de analítica adecuado | **Aurora** y **DynamoDB** (bases de datos), **OpenSearch Service** (búsqueda), **EMR** (big data), **Redshift** (data warehouse), **SageMaker** (ML), **Amazon MSK** (streaming), **Kinesis Data Analytics** (analítica en tiempo real sobre streaming) |
| **3. Unified data access** | Acceder a los datos donde residen y moverlos **selectivamente** cuando haga falta | **AWS Glue** (ETL), **Amazon Data Firehose** (streaming), **AWS DMS** (migración de bases), **AWS Transfer Family** (archivos), **Amazon MSK** (Kafka administrado), **Amazon AppFlow** (SaaS) |
| **4. Unified governance** | Autorizar, administrar y auditar el acceso | **AWS Lake Formation** (la opción preferida del instructor) |
| **5. Performance and cost-effectiveness** | Alto rendimiento a bajo costo | Prácticas: **compresión**, **alertas de facturación**, **partición**, **formatos columnares** |

La lectura agrega el resultado esperado: construir lagos escalables rápido, usar una colección amplia de servicios especializados, asegurar acceso, seguridad y gobierno unificados, escalar a bajo costo sin sacrificar desempeño y **compartir datos a través de los límites de la organización**, para decidir con velocidad y agilidad.

🎯 **Para el examen:**
- Búsqueda de texto / log analytics → **OpenSearch Service**. Grafos de conocimiento → **Neptune**. Data warehouse → **Redshift**. Big data con Spark/Hadoop → **EMR**. Kafka administrado → **MSK**.
- Gobierno unificado sobre el lago → **Lake Formation**.
- Ahorro: compresión + partición + columnar, y **billing alerts**.

---

## 3. Patrones de movimiento de datos

Audiencia: 🔧 🧭

En los tres el **lago sigue en el centro**; cambia la **dirección** del flujo.

```
         INSIDE-OUT                      OUTSIDE-IN                  AROUND THE PERIMETER
   ┌────────────┐                  ┌────────────┐                  ┌──────────┐    ┌──────────┐
   │ DATA LAKE  │──► warehouse,    │ DATA LAKE  │◄── DynamoDB,     │ Almacén A│───►│ Almacén B│
   │ (fuente    │    OpenSearch,   │ (procesa)  │    BD propias,   └──────────┘    └──────────┘
   │  única)    │    Neptune,      └────────────┘    terceros            (el dato no pasa por el lago)
   └────────────┘    socios
```

| Patrón | Idea | Ejemplo del video | Ejemplo de la lectura |
|---|---|---|---|
| **Inside-out** | Del lago hacia fuera: tomar datos internos y ponerlos a disposición para consumo externo o más amplio (socios, clientes, reguladores). El lago es la **single source of truth** | Parte de lo ingerido al lago se mueve a un **data warehouse** para reportes diarios | Un subconjunto del lago va a **OpenSearch Service** (search analytics) o **Amazon Neptune** (knowledge graph a partir de fuentes estructuradas y no estructuradas) |
| **Outside-in** | Hacia el lago: los datos viven sobre todo en otros servicios y el lago se activa cuando hay algo que procesar. Útil para datos de terceros (mercado, redes sociales) | Copiar resultados de consultas de las propias bases al lago para entrenar **algoritmos de recomendación** con más datos, usando Glue, **SageMaker** o **EMR** | Una empresa de juegos guarda estado del juego, jugadores, sesiones y rankings en **DynamoDB** y luego los exporta al lago para analizarlos |
| **Around the perimeter** | Entre almacenes especializados, sin atravesar el lago; los servicios coexisten en la arquitectura | Copiar datos de una base a un **servicio de búsqueda** para descargar de la base las consultas de búsqueda | Copiar perfiles de clientes de una base **relacional** a una **NoSQL** para dashboards de reportes |

> [!note] ➕ Nota del editor
> Un lago institucional que publica tablas analytics para Power BI es inside-out; traer una exportación de un ERP financiero o de un SaaS al lago es outside-in; y alimentar un índice de búsqueda directamente desde la base transaccional es around the perimeter. Nombrar el patrón ayuda a decidir dónde vive el gobierno de cada flujo.

---

## 4. Datos como producto, data mesh y Amazon DataZone

Audiencia: 🧭 👔

### 4.1 Compartir más allá de un área

Dentro de una sola línea de negocio compartir datos es sencillo. Compartir **entre equipos o incluso entre empresas** revela información que no se ve mirando solo una porción del pastel. Esa mentalidad lleva a tratar **los datos como un producto**.

| Aspecto | Qué plantea el curso |
|---|---|
| **Por qué** | **Propiedad**: cada equipo es dueño de su producto de datos y responde por él durante todo su ciclo de vida, de la creación al retiro; así los datos se mantienen y mejoran |
| **Cómo** | Empezar por **una línea de negocio** dueña de sus productos de datos, usados por consumidores internos y externos; estructura y responsabilidad claras. Luego **expandir** a otras unidades |
| **Impacto** | Con muchos productos y equipos, el **gobierno y cumplimiento** se vuelven centrales: descubrimiento, reportes y auditoría, para asegurar calidad y cumplimiento |

### 4.2 Data mesh

Cuando todo eso funciona "como una máquina bien aceitada", se tiene una organización **data mesh**. Objetivos de diseño:

1. Tratar los datos **como producto**.
2. **Gobierno de datos federado**.
3. **Acceso común** a los datos.

### 4.3 Amazon DataZone

Servicio de **gestión de datos** que facilita **catalogar, descubrir, compartir y gobernar** datos almacenados en AWS, on-premises y en fuentes de terceros. Se integra con **Lake Formation, Athena, Glue, QuickSight, Redshift** y otros.

Combinado con una **estrategia multicuenta** (ver [[08-Gobierno-y-Multicuenta]]), da una arquitectura sólida y resiliente: lagos y federaciones de datos independientes en varias cuentas, que pueden abarcar **varias Regions** o incluso **varios proveedores de nube**.

🎯 **Para el examen:**
- Catálogo de negocio para descubrir y compartir datos entre dominios con gobierno → **Amazon DataZone**.
- Data mesh = datos como producto + gobierno federado + acceso común.

---

## 5. Charla técnica: las funciones favoritas

Audiencia: 🔧 🧭

**Alex — Amazon QuickSight.** Como administrador de sistemas en una empresa pequeña, le pedían constantemente convertir pilas de CSV en gráficos para reuniones (el viernes a las 7 p.m.), a fuerza de tablas dinámicas. QuickSight le dio productividad para crear gráficos y presentaciones y, sobre todo, le permitió **ayudar a otros a crear sus propios gráficos** en lugar de hacerlos él.

**Raf — formatos columnares.** Desde su experiencia en almacenamiento: en formatos raw basados en filas, leer toda una columna exige muchas operaciones de I/O en disco y luego reensamblar la tabla; en columnar se leen columnas enteras. Además, **Parquet guarda el primer y el último registro (mín./máx.) de cada bloque**: si los datos están **ordenados** y se filtra por esa columna (`WHERE name = 'Alex'`), el motor sabe qué bloque contiene de la A a la F y salta el resto, leyendo **solo parte de la columna**.

- Apache Parquet es open source; los formatos columnares existen desde hace décadas.
- El dataset NYC TLC de la demo del módulo 2 se publicaba en CSV cuando se grabó; hoy su sitio lo entrega **ya en Parquet**.
- En una **Athena federated query** sobre un dataset grande, particionar al menos reduce lo que hay que descargar por la red.

> [!tip] 💡 Lo que une ambas respuestas
> Raf observa que, al habilitar a los usuarios con QuickSight, Alex creó un **producto de datos**: dejó de ser el cuello de botella y la gente accede por sí misma. Es la idea de la sección 4 en pequeño.

> [!note] ➕ Nota del editor
> El truco de mín./máx. por bloque explica por qué ordenar los datos al escribir Parquet (por ejemplo, por período o por RUT) puede bajar el escaneo de Athena incluso sin particionar por esa columna.

---

## 6. Resumen del curso: Well-Architected y Data Analytics Lens

Audiencia: 🔧 🧭 👔

### 6.1 AWS Well-Architected Framework (WAF)

Conjunto de buenas prácticas para diseñar y operar cargas en la nube, en **seis pilares**: operational excellence, security, reliability, performance efficiency, cost optimization y **sustainability**. Más información en `aws.amazon.com/well-architected`.

### 6.2 Data Analytics Lens: recomendaciones

El **Well-Architected Data Analytics Lens** es un subconjunto del marco con buenas prácticas específicas para cargas de analítica:

| Tema | Recomendación |
|---|---|
| **Ubicación / data gravity** | Los datos atraen aplicaciones y servicios; es más fácil desplegar aplicaciones (sobre todo con contenedores como Docker) que mover datos. Mantener la aplicación **cerca** de los datos reduce latencia y mejora rendimiento |
| **Partición** | Particionar en S3 aumenta considerablemente el rendimiento |
| **Tamaño de archivos y compactación** | Administrar el tamaño de archivo; **programar la compactación** según la frecuencia con que se ingiere y muta el dato |
| **Formato y compresión** | Convertir a columnar y comprimir para reducir costo de almacenamiento |
| **Storage class** | Elegir el nivel de almacenamiento según la **temperatura** del dato |
| **Descubrimiento** | Usar **Glue crawlers** para descubrir datasets nuevos, seguir el **lineage** y evitar una avalancha de datos |
| **Seguridad** | Control de acceso en **Lake Formation**, **IAM** para permisos de servicio, **AWS KMS** para claves de cifrado |
| **Centralización** | Mantener los datos en el lago y dejar que los consumidores accedan a ellos, en lugar de **copiarlos** entre lago y warehouse |
| **Monitoreo** | Supervisar componentes y rendimiento con notificaciones: **CloudWatch Events** (eventos del sistema en tiempo real), **CloudWatch Logs** (logs), **AWS CloudTrail** (llamadas a la API) |

> [!warning] ⚠️ Centralizar vs mover
> El lens recomienda **no copiar** entre lago y warehouse para acceder a los datos, mientras que el patrón inside-out mueve un subconjunto a un warehouse. No se contradicen: copiar es la excepción justificada por un caso de uso, no la forma por defecto de dar acceso.

---

## 7. Demos y laboratorios

Este módulo **no tiene demos en video**. El curso anunció un **laboratorio autoguiado** para el módulo 6, pero sus instrucciones **no forman parte del material extraído**; solo se registra su existencia. La lección figura como "Lección sin título" y cierra con la encuesta posterior al curso (Qualtrics, sin contenido técnico).

---

## 8. Arquitectura de referencia: estado al cierre del curso

```
            Fuentes: BD (DMS) · archivos (Transfer Family) · SaaS (AppFlow) · streaming (MSK / Data Firehose)
                                               │  outside-in
                                               ▼
 ┌─────────────────────────── DATA LAKE (S3 + Glue Data Catalog) ───────────────────────────┐
 │  Parquet + compresión + particiones + compactación · storage class por temperatura       │
 │  Glue crawlers (descubrimiento, lineage) · Athena (consulta directa)                     │
 │  Gobierno unificado: Lake Formation (+ IAM servicios, KMS claves)                         │
 └───────────────┬───────────────────────────────────────────────┬──────────────────────────┘
                 │ inside-out                                    │ consumo
                 ▼                                               ▼
   Redshift · OpenSearch · Neptune · socios          QuickSight · SageMaker · EMR
                 ▲
   Aurora / DynamoDB ──around the perimeter──► OpenSearch / NoSQL

 Capa organizacional: data products por dominio → data mesh → Amazon DataZone + multicuenta
 Operación: CloudWatch Events · CloudWatch Logs · CloudTrail · billing alerts
```

---

## ✅ Autoevaluación

> [!question]- ¿Cuáles son los cinco pilares de la arquitectura de datos moderna en AWS?
> Scalable data lakes, purpose-built analytics services, unified data access, unified governance, y performance and cost-effectiveness.

> [!question]- ¿Qué servicio asocia el curso a búsqueda, big data, ML y streaming dentro de los servicios especializados?
> OpenSearch Service (búsqueda), EMR (big data), SageMaker (ML), MSK y Kinesis Data Analytics (streaming y analítica en tiempo real).

> [!question]- Una empresa de juegos guarda sesiones en DynamoDB y quiere analizarlas en el lago. ¿Qué patrón es?
> Outside-in: el dato vive en un almacén especializado y se exporta al lago para análisis.

> [!question]- Copiar perfiles de clientes de una base relacional a una NoSQL para dashboards, ¿qué patrón es y qué lo distingue?
> Around the perimeter: el movimiento ocurre entre almacenes especializados sin atravesar el lago.

> [!question]- ¿Por qué tratar los datos como producto mejora su calidad?
> Porque cada equipo es dueño de su producto y responde por él en todo el ciclo de vida, de la creación al retiro, lo que obliga a mantenerlo y mejorarlo.

> [!question]- ¿Cuáles son los objetivos de diseño de data mesh y qué servicio de AWS lo apoya?
> Datos como producto, gobierno federado y acceso común; Amazon DataZone, integrado con Lake Formation, Athena, Glue, QuickSight y Redshift.

> [!question]- ¿Cómo aprovecha Parquet el orden de los datos?
> Guarda el primer y último valor de cada bloque; si los datos están ordenados y se filtra por esa columna, el motor salta los bloques que no pueden contener el valor.

> [!question]- Mencione cuatro recomendaciones del Data Analytics Lens.
> Aplicaciones cerca de los datos (data gravity), particionar en S3, compactar según la frecuencia de ingesta, columnar + compresión, storage class por temperatura, crawlers para lineage, Lake Formation + IAM + KMS, no copiar datos, monitorear con CloudWatch y CloudTrail.

## 🧾 Resumen en 10 líneas

1. Un lago centralizado y compartido elimina silos y da a todos la misma información actualizada.
2. La arquitectura moderna integra lago, warehouse y almacenes especializados con gobierno unificado.
3. Cinco pilares: lagos escalables, servicios especializados, acceso unificado, gobierno unificado, rendimiento y costo.
4. Servicios especializados: Aurora/DynamoDB, OpenSearch, EMR, Redshift, SageMaker, MSK, Kinesis Data Analytics.
5. Inside-out: del lago (fuente única) hacia warehouse, OpenSearch, Neptune o terceros.
6. Outside-in: de almacenes como DynamoDB o fuentes externas hacia el lago para procesarlos.
7. Around the perimeter: entre almacenes, sin pasar por el lago.
8. Datos como producto y data mesh (producto, gobierno federado, acceso común), apoyados por Amazon DataZone y multicuenta.
9. Parquet guarda mín./máx. por bloque: datos ordenados permiten leer solo parte de la columna.
10. Data Analytics Lens: data gravity, partición, compactación, columnar, storage class, crawlers, Lake Formation + IAM + KMS, no copiar, monitorear.

## 📖 Recursos del módulo

- [AWS Well-Architected](https://aws.amazon.com/well-architected) (citado en el video de cierre)
- Whitepaper *Modern Data Architecture Rationales on AWS* · mencionado como base del módulo, sin URL en el material

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[13-Lake-Formation-Avanzado|13 · Lake Formation Avanzado]] · Siguiente: [[15-Examen-SAA-Estrategia|15 · Examen SAA: Estrategia ➡]]
