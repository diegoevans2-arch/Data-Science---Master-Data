---
title: "Tomo 09 — Introducción a los Data Lakes"
tags: [aws, solutions-architect, data-lake, analytics]
curso: 3 · Introduction to Designing Data Lakes on AWS
modulo: "Módulo 1: Introducción a los lagos de datos"
audiencias: [tecnico, puente, ejecutivo]
tomo: 09
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🏞️ Tomo 09 — Introducción a los Data Lakes

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[08-Gobierno-y-Multicuenta|08 · Gobierno y Multicuenta]] · Siguiente: [[10-Ingesta-Catalogo-y-Preparacion|10 · Ingesta, Catálogo y Preparación ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Abre el tercer curso del certificado, dedicado por completo a los data lakes. Antes de hablar de servicios, fija el vocabulario: qué es un data lake, en qué se diferencia de un data warehouse (schema-on-read frente a schema-on-write), por qué existe el riesgo del *data swamp* y cuál es el flujo mínimo —fuentes → ingesta → almacenamiento → catálogo → consulta → visualización— sobre el que se montan los tomos 10 a 14.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** cuándo conviene un data lake, cuándo un data warehouse y cuándo ambos; qué arquitectura mínima basta para empezar.
> - **Costo de hacerlo mal:** un lago sin catálogo ni control de acceso se vuelve un pantano de datos: nadie encuentra nada y nadie confía en lo que encuentra.
> - **Pregunta que responde:** *¿dónde dejo todos mis datos, de cualquier formato, para que cualquier área pueda hacerles preguntas rápido y con seguridad?*

> [!tip] 💡 Analogía general: el lago, la cerca y el mapa
> El curso juega todo el módulo con la metáfora del lago. Los ríos que lo alimentan son las fuentes (bases de datos, IoT, móviles, redes sociales); el agua de manantial que llega sin tocar son los **datos raw**; el **catálogo** es el mapa que dice qué hay en cada rincón; la **cerca alta** alrededor son los controles de acceso, para que nadie nade donde no debe. Sin mapa ni cerca, el lago se convierte en pantano.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Introducción y visión general del curso | 1 video | Estructura de los 6 módulos, laboratorios y prerrequisitos |
| Objetivos del módulo | 1 video | Qué se verá en la sección |
| ¿Por qué los lagos de datos? | 1 video | Definición, ventajas, democratización, riesgo del data swamp |
| Lago de datos frente a almacén de datos | 1 video | Comparación en 5 ejes: estructura, esquema, costo, calidad, usuarios |
| Componentes y arquitecturas | 1 video | Flujo de punta a punta y arquitectura básica S3 + Glue + Athena |
| Encuesta previa al curso | 1 lectura + 1 encuesta | Sin contenido técnico |

---

## 1. Panorama del curso

Audiencia: 🧭

El instructor (Raf Lopez, con diez años en AWS Training and Certification, antes formador técnico y solutions architect) presenta el recorrido del curso:

| Módulo del curso | Contenido | Tomo de esta guía | Lab |
|---|---|---|---|
| 1. Introducción a los lagos de datos | Propuesta de valor, lake vs warehouse, componentes y arquitecturas | 09 | — |
| 2. Ingesta, catalogación y preparación | Servicios de almacenamiento y procesamiento: S3, Glue, Athena y otros | [[10-Ingesta-Catalogo-y-Preparacion\|10]] | — |
| 3. Data lake con AWS Lake Formation | Características y componentes básicos de Lake Formation | [[11-Data-Lake-con-Lake-Formation\|11]] | Sí, autoguiado |
| 4. Procesamiento y análisis | Transformación con AWS Glue, Glue DataBrew y Glue Studio | [[12-Procesamiento-y-Analisis\|12]] | — |
| 5. Lake Formation avanzado | Blueprints, workflows, control de acceso fino, tags; Amazon QuickSight | [[13-Lake-Formation-Avanzado\|13]] | Sí |
| 6. Arquitectura de datos moderna | Arquitecturas modernas, movimiento de datos y modelos para compartir | [[14-Arquitectura-de-Datos-Moderna\|14]] | Sí |
| Cierre | Resumen y próximos pasos | — | — |

Lake Formation aparece **dos veces**: lo básico en el módulo 3 y las funciones ampliadas en el 5.

> [!warning] ⚠️ Prerrequisitos declarados
> El curso recomienda haber completado antes **AWS Technical Essentials** y **Data Analytics Fundamentals**. En esta guía, lo primero corresponde a [[01-Vision-General-y-Seguridad]] … [[04-Monitoreo-y-Optimizacion]], y la analítica serverless a [[06-Analitica-de-Datos-Serverless]].

---

## 2. Qué es un data lake y por qué usarlo

Audiencia: 🔧 🧭 👔

**Definición.** Un data lake es un **repositorio central** que guarda datos **estructurados y no estructurados a cualquier escala**: texto, video, imágenes, audio, logs o cualquier otra cosa. Crece con la organización, igual que un lago real.

**¿Por qué no meterlo todo en una base de datos?** Por volumen y por velocidad de respuesta. Una base de datos tradicional o incluso un data warehouse sirven para ciertos conjuntos de datos, pero imponen una barrera (modelar y cargar antes de preguntar). Hoy las preguntas de negocio deben responderse en días, horas o en tiempo real; el lago permite obtener respuestas, incluso en tiempo real, sobre lo que se va recopilando.

**Democratización del acceso.** Antes, quien quería un dato abría un ticket al equipo de datos, esperaba y luego decidía. Con un data lake, las personas pueden llevar sus preguntas directamente al lago y obtener la respuesta sin intermediarios.

### Las cinco funciones de un data lake

El curso lo resume en un gráfico que se puede leer así:

```
            ┌──────────────────────── DATA LAKE ────────────────────────┐
 fuentes →  │  1. Ingerir     2. Almacenar   3. Catalogar  4. Buscar /  │ → respuestas
            │                 (hasta saber   y buscar      consultar    │   rápidas
            │                  qué hacer)                               │
            │  5. Proteger: controles de acceso definidos ("la cerca")  │
            └───────────────────────────────────────────────────────────┘
```

1. **Repositorio central** para todos los datos.
2. **Ingiere y almacena**, y conserva los datos hasta que se decida qué hacer con ellos.
3. **Cataloga** la información para que se pueda **buscar**.
4. Permite **hacer preguntas y obtener respuestas rápidas** para decidir a tiempo.
5. **Protege** los datos con controles de acceso definidos.

### El riesgo: el data swamp

> [!danger] 🚨 De lago a pantano
> Un data lake debe procesar muchos tipos de datos, a alta velocidad y en formatos distintos, y no todos valen lo mismo. Si no hay **mecanismos definidos para catalogar y proteger** lo que entra, el resultado es un *data swamp*: no se encuentran los datos y no se puede confiar en ellos. El resto del curso presenta las herramientas de AWS para evitarlo (Glue Data Catalog, Lake Formation).

---

## 3. Data lake frente a data warehouse

Audiencia: 🔧 🧭 👔

| Eje | Data warehouse | Data lake |
|---|---|---|
| **Naturaleza** | Base de datos **relacional** (grande) | Colección **no relacional** de datos de casi cualquier fuente (IoT, logs de servicios, redes sociales…) |
| **Esquema** | **Schema-on-write**: el formato se planifica antes de cargar; hay reglas sobre cómo deben verse los datos | **Schema-on-read**: la estructura se aplica al momento de analizar, mediante la consulta |
| **Costo** | Modelo de facturación de base de datos tradicional: se paga por mantenerlo en línea y por el almacenamiento; la factura puede crecer según rendimiento y storage | Se paga el almacenamiento (típicamente S3, muy económico) y el **cómputo solo cuando se extraen y analizan** los datos |
| **Calidad** | Las barreras de calidad se construyen en la fase de diseño; el esquema impuesto crea una **single source of truth** | No siempre limpio; puede "contaminarse" según lo que se vierta. La calidad es responsabilidad del usuario |
| **Usuarios típicos** | **Business analysts**: BI, visualizaciones e informes para decisiones diarias y de largo plazo | **Data scientists**: machine learning, analítica predictiva, descubrimiento de datos, modelos de lenguaje y chatbots. Los analistas de negocio a veces consultan una sección curada |

> [!tip] 💡 Ventaja del schema-on-write
> Su punto a favor es que el trabajo de estructura ya está hecho: no hay que resolverlo después. La ventaja del schema-on-read es la libertad de recolectar todo sin decidir de antemano.

> [!warning] ⚠️ No es uno u otro
> El curso insiste en que ninguno es mejor: **ambos pueden convivir** en el mismo entorno. Lo importante es conocer las diferencias para elegir la herramienta adecuada para cada trabajo. Esta idea reaparece en la arquitectura de datos moderna ([[14-Arquitectura-de-Datos-Moderna]]).

🎯 **Para el examen:**
- Schema-on-read ↔ data lake; schema-on-write ↔ data warehouse.
- Lake: datos estructurados **y** no estructurados; warehouse: solo estructurados que respetan el esquema.
- Lake en S3: almacenamiento barato + cómputo bajo demanda.

---

## 4. Componentes y arquitecturas

Audiencia: 🔧 🧭

### 4.1 El flujo genérico de punta a punta

Antes de nombrar servicios, el curso propone los pasos para "llenar el lago":

```
 Fuentes de datos ──► Ingesta ──► Almacenamiento ──► Catálogo y ──► Búsqueda / ──► Respuestas
 (BD existentes,                  (otra BD o storage  procesamiento  consulta       (visualización)
  IoT, móviles…)                   no estructurado)
```

El catálogo es crítico: hay que saber qué contienen los datos **antes** de poder preguntarles algo.

### 4.2 Servicios de AWS en cada paso

| Paso | Opciones de AWS que menciona el curso | Criterio |
|---|---|---|
| Ingesta | **AWS DMS** (Database Migration Service) para mover datasets existentes a la nube; **Amazon Kinesis** para streaming; o una combinación | Algo que alimente el lago de forma continua |
| Almacenamiento | Base de datos tradicional, data warehouse (**Amazon Redshift**) o **bucket de S3** | ¿Se quiere estructura establecida desde ya, o resolverla después? |
| Catálogo | **AWS Glue Data Catalog**, con **schema inference** (descubre la estructura a partir del contenido) | Saber qué "flota" en el lago |
| Consulta | **Amazon Athena** (opción serverless), que usa el Data Catalog de Glue para las consultas schema-on-read | Sin servidores que administrar |
| Visualización | **Amazon QuickSight** o herramientas de terceros | Gráficos, tablas y diagramas para decidir |

No existe una solución universal: hay muchas combinaciones posibles y el curso las irá profundizando.

### 4.3 Arquitectura básica de data lake

```
  Fuentes diversas
        │
        ▼
 ┌───────────────┐      ┌──────────────────────────┐      ┌───────────────┐
 │  S3 · RAW     │ ───► │  AWS Glue                │ ───► │ S3 · TRANSFOR-│
 │  (sin cambios)│      │  · ETL jobs              │      │ MADO (limpio) │
 └───────────────┘      │  · Crawler → Data Catalog│      └──────┬────────┘
                        └──────────────────────────┘             │
                                                                 ▼
                                                         Amazon Athena (consulta)
```

- **Raw data:** llega directo de la fuente, sin modificaciones. Es "el agua de manantial", pero tal como está no es utilizable porque no se sabe qué contiene ni qué forma tiene.
- **AWS Glue** ejecuta los trabajos **ETL** (extract, transform, load) y un **crawler** genera el Data Catalog.
- **Datos transformados:** limpios y con la forma correcta para consultar. Pueden ir en una **sección separada del mismo bucket** o, mejor aún, en un **bucket completamente separado**.
- **Amazon Athena** consulta los datos transformados.

> [!tip] 💡 Simple es bueno
> El curso subraya que esta arquitectura es "lo más simple posible" y que eso es una virtud: se puede construir, empezar a iterar y no tiene que ser un proyecto enorme desde el día uno.

> [!note] ➕ Nota del editor
> Es exactamente el patrón de un data lake institucional típico en S3 + Glue + Athena: una zona raw (por ejemplo, la réplica diaria de un ERP académico) y una zona de stage o curada, catalogadas por Glue y consultadas con Athena. La separación raw/transformado en buckets distintos simplifica además los permisos: los analistas solo necesitan acceso al bucket curado.

---

## 5. Demos y laboratorios

Este módulo **no tiene demos ni laboratorios**. El video de objetivos anuncia "una aplicación de muestra rápida", pero en el material del módulo no aparece ninguna demo práctica; el contenido se limita a los diagramas de la sección 4. Los labs autoguiados llegan en los módulos 3, 5 y 6 (tomos [[11-Data-Lake-con-Lake-Formation|11]], [[13-Lake-Formation-Avanzado|13]] y [[14-Arquitectura-de-Datos-Moderna|14]]).

La lección "Evaluación de la semana 1" contiene solo la encuesta previa al curso (alojada en Qualtrics, datos propiedad de AWS según su aviso de privacidad); no tiene contenido técnico.

---

## 6. Arquitectura de referencia: estado al cierre del módulo

```
 BD existentes ──(AWS DMS)──┐
 IoT / móviles ─(Kinesis)───┤
                            ▼
                     S3 raw ──► Glue ETL ──► S3 transformado
                       │           │
                       └── Glue crawler ──► Glue Data Catalog
                                                  │
                                                  ▼
                                            Amazon Athena ──► Amazon QuickSight
```

Es el esqueleto que los tomos siguientes irán llenando: ingesta y preparación (10), gobierno con Lake Formation (11 y 13), procesamiento (12) y arquitectura moderna (14).

---

## ✅ Autoevaluación

> [!question]- ¿Qué distingue a un data lake de una base de datos tradicional en cuanto a tipos de datos?
> El data lake guarda datos estructurados y no estructurados (texto, imágenes, audio, video, logs) a cualquier escala; la base de datos exige datos estructurados que respeten un esquema.

> [!question]- Explique schema-on-read y schema-on-write con un ejemplo de cuándo conviene cada uno.
> Schema-on-write (warehouse): se define el formato antes de cargar; conviene para reportes de BI estables donde se quiere una única fuente de verdad. Schema-on-read (lake): se guarda todo y la estructura se aplica al consultar; conviene para exploración, ML o datos de fuentes heterogéneas cuyo uso aún no se conoce.

> [!question]- ¿Qué es un data swamp y qué lo previene?
> Un lago donde no se encuentran los datos ni se confía en ellos. Se previene con mecanismos definidos de catalogación y protección (control de acceso) de lo que entra.

> [!question]- ¿Por qué el data lake suele ser más barato que el data warehouse?
> Porque el almacenamiento (S3) es muy económico y el cómputo se paga solo cuando se analizan los datos; el warehouse cobra por mantenerse en línea además del almacenamiento.

> [!question]- Nombre un servicio de AWS para cada paso: ingesta, almacenamiento, catálogo, consulta y visualización.
> DMS o Kinesis; S3 (o Redshift); Glue Data Catalog; Athena; QuickSight.

> [!question]- En la arquitectura básica, ¿dónde deben quedar los datos transformados?
> En una sección separada del bucket o, preferiblemente, en un bucket completamente separado del raw.

> [!question]- ¿Quiénes son los usuarios típicos de cada solución?
> Warehouse: business analysts (BI e informes). Lake: data scientists (ML, analítica predictiva, descubrimiento), con visitas ocasionales de analistas a secciones curadas.

> [!question]- ¿Qué hace el crawler de Glue en la arquitectura básica?
> Recorre los datos, infiere su esquema y genera/actualiza las tablas del Glue Data Catalog que luego usa Athena.

## 🧾 Resumen en 10 líneas

1. Un data lake es un repositorio central de datos estructurados y no estructurados, a cualquier escala.
2. Permite responder preguntas de negocio rápido, incluso en tiempo real.
3. Democratiza el acceso: cada área consulta sin pasar por un ticket al equipo de datos.
4. Sus funciones: ingerir, almacenar, catalogar/buscar, consultar y proteger.
5. Sin catálogo ni control de acceso se convierte en un data swamp.
6. Warehouse = relacional, schema-on-write, calidad por diseño, BI; lake = no relacional, schema-on-read, ML y exploración.
7. Lake en S3: almacenamiento barato y cómputo solo al analizar.
8. Ambos pueden convivir; se elige la herramienta según el trabajo.
9. Flujo: fuentes → ingesta (DMS, Kinesis) → S3/Redshift → Glue Data Catalog → Athena → QuickSight.
10. Arquitectura mínima: S3 raw → Glue ETL + crawler → S3 transformado (idealmente otro bucket) → Athena.

## 📖 Recursos del módulo


El módulo no trae enlaces técnicos.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[08-Gobierno-y-Multicuenta|08 · Gobierno y Multicuenta]] · Siguiente: [[10-Ingesta-Catalogo-y-Preparacion|10 · Ingesta, Catálogo y Preparación ➡]]
