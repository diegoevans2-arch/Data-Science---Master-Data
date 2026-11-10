---
title: "Tomo 11 — Data Lake con AWS Lake Formation"
tags: [aws, solutions-architect, data-lake, lake-formation, gobierno, seguridad]
curso: 3 · Introduction to Designing Data Lakes on AWS
modulo: "Módulo 3: Creación de un lago de datos con AWS Lake Formation"
audiencias: [tecnico, puente, ejecutivo]
tomo: 11
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🏗️ Tomo 11 — Data Lake con AWS Lake Formation

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[10-Ingesta-Catalogo-y-Preparacion|10 · Ingesta, Catálogo y Preparación]] · Siguiente: [[12-Procesamiento-y-Analisis|12 · Procesamiento y Análisis ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> El tomo 10 mostró cuántas piezas hay que coordinar: ingesta, formato, partición, catálogo, crawlers, permisos. **AWS Lake Formation** las orquesta sobre AWS Glue y, sobre todo, aporta un **modelo de permisos propio**, tipo base de datos (grant/revoke), con control fino a nivel de base, tabla, columna, fila y celda. Este módulo es introductorio: características, la guía de tres pasos y el modelo básico de permisos. Lo avanzado (blueprints en detalle, LF-Tags, data filters, sharing) está en [[13-Lake-Formation-Avanzado]].

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** centralizar el gobierno del lago en un solo lugar; dar acceso por columna o fila sin duplicar datasets por equipo.
> - **Costo de hacerlo mal:** copias del mismo dato por cada área (más storage y versiones contradictorias), accesos demasiado permisivos por quedarse con los valores por defecto, incumplimiento de normas de privacidad sin auditoría de quién vio qué.
> - **Pregunta que responde:** *¿cómo garantizo que cada persona vea exactamente los datos que necesita, ni más ni menos, desde un único lugar?*

> [!tip] 💡 Analogía general: el bibliotecario con llavero
> El Glue Data Catalog es el sistema que indica en qué estante está cada libro de una biblioteca enorme. Lake Formation agrega al **bibliotecario con el llavero**: revisa quién pide el libro, le presta una llave temporal solo para ese estante y, si corresponde, le tapa con cinta los capítulos que no puede leer (columnas, filas, celdas).

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Bienvenido al módulo 3 | 1 video | Temas del módulo |
| Vista general de AWS Lake Formation | 1 video | Qué automatiza, blueprints, seguridad y cifrado, guía de 3 pasos |
| Modelo básico de permisos | 1 video | Permisos unificados, fine-grained, IAM vs Lake Formation, personas |
| Lectura: conceptos básicos de Lake Formation | 1 lectura | Hybrid access mode, sharing, flujo de credential vending |
| Laboratorio autoguiado (anunciado) | lab | Interactuar con Lake Formation (sin texto en el material extraído) |

---

## 1. Qué es AWS Lake Formation

Audiencia: 🔧 🧭 👔

Lake Formation ayuda a **gobernar, proteger y compartir datos** de forma centralizada para analítica y machine learning. Administra el **control de acceso fino** sobre los datos del lago en **Amazon S3** y sobre sus **metadatos en el AWS Glue Data Catalog**.

### 1.1 Construido sobre AWS Glue

Lake Formation **se apoya en AWS Glue** y automatiza el armado del entorno. Por dentro usa **crawlers, Data Catalog y jobs**.

| Pieza de Glue | Rol dentro de Lake Formation |
|---|---|
| **Glue Data Catalog** | "El cerebro" del data lake: mantiene todo organizado |
| **Crawlers** | Descubren e infieren esquemas |
| **Jobs** | Transforman datos |
| **Glue workflows** | Orquestan jobs, crawlers y triggers |
| **Blueprints** (de Lake Formation) | Plantillas de workflow para ingerir y procesar fuentes comunes; pensadas para las actividades más habituales en un lago |

### 1.2 Seguridad y cifrado

- **Control de acceso centralizado**, con reglas de política respaldadas por **AWS IAM**.
- **Cifrado en reposo**: aprovecha el cifrado de los servicios de AWS; los datos se cifran del lado del servidor con claves administradas por **AWS KMS**.
- **Cifrado en tránsito**: con certificados **SSL**.

### 1.3 Romper silos (de la lectura)

El flujo que plantea la lectura para combinar datos estructurados y no estructurados en un repositorio central:

1. Identificar los almacenes existentes (S3, bases relacionales y NoSQL) y **mover los datos** al lago.
2. **Rastrear, catalogar y preparar** los datos para análisis.
3. Dar a los usuarios **acceso self-service seguro** mediante los servicios de analítica que elijan.

---

## 2. La guía de tres pasos del dashboard

Audiencia: 🔧

El dashboard de Lake Formation propone configurar el lago en tres pasos:

```
 ① Registrar ubicaciones S3  ──►  ② Crear bases de datos en el  ──►  ③ Otorgar permisos
    (data lake locations)            Data Catalog de Lake Formation      (base / tabla / columna)
                                     · durante la ingesta, LF genera
                                       tablas para workflows comunes
                                       (vía blueprints)
```

1. **Registrar** las ubicaciones de almacenamiento designadas en Amazon S3.
2. **Crear** una o más bases de datos en el Data Catalog. Durante la ingesta, Lake Formation genera tablas automáticamente para los workflows comunes, apoyándose en blueprints.
3. **Otorgar permisos** sobre los recursos del lago, a nivel de base de datos, tabla o columna.

---

## 3. Modelo básico de permisos

Audiencia: 🔧 🧭

### 3.1 Permisos unificados y finos

- **Un solo lugar**: Lake Formation unifica los permisos sobre los **motores de analítica, el Data Catalog y el almacenamiento**, de modo que el acceso se controla de forma uniforme.
- **Fine-grained, tipo base de datos**: restringe el acceso a **tablas, columnas e incluso filas** del Data Catalog (la lectura agrega **celdas**), con un mecanismo simple de **grant/revoke** muy parecido al de un RDBMS.
- **Unifica los permisos de S3**: el storage del lago se administra desde el mismo lugar.
- **Aumenta** (no reemplaza) el modelo de permisos de IAM.

### 3.2 Por qué importa: gobierno, cumplimiento y costo

| Beneficio | Explicación |
|---|---|
| Gobierno | Cada usuario accede solo a lo que necesita, una petición habitual de los clientes |
| **Costo** | Una **única fuente de verdad** con distintos permisos evita **replicar datasets** por equipo: menos storage, menos factura |
| Cumplimiento | Ayuda a cumplir políticas frente a leyes de privacidad cada vez más estrictas |
| Auditoría | Soporta **monitoreo y auditoría** del acceso: quién accedió, a qué datos, de dónde y cuándo |

### 3.3 IAM y Lake Formation: reparto de responsabilidades

> [!warning] ⚠️ Los valores por defecto pueden ser demasiado permisivos
> Recomendación del curso: **IAM** con permisos de alcance amplio para que los usuarios puedan **interactuar con los servicios de AWS** que la arquitectura necesita (menos políticas IAM que mantener), y **Lake Formation** para "la letra chica": permisos acotados de personas sobre el Data Catalog, las ubicaciones S3 y los datos subyacentes.

| Capa | Qué gobierna | Ejemplo |
|---|---|---|
| **IAM** | Permisos de usuarios y servicios sobre recursos de AWS | Poder usar Athena, Glue; acceso a buckets o tablas DynamoDB en general |
| **Lake Formation** | Acceso a los **datos** de los recursos que administra | `SELECT` sobre tres columnas de una tabla del catálogo |

El curso menciona que, por **compatibilidad hacia atrás**, Lake Formation trae **habilitada por defecto** la configuración de control de acceso **solo con IAM** ("Use only IAM access control"), lo que no sirve para el control fino buscado: hay que cambiarla para que rijan los permisos de Lake Formation.

### 3.4 Hybrid access mode (lectura)

El **modo de acceso híbrido** del Data Catalog permite proteger y acceder a los datos catalogados usando **a la vez** permisos de Lake Formation y políticas IAM para acciones de S3 y Glue. Así los administradores pueden **incorporar Lake Formation de forma selectiva e incremental**, un caso de uso a la vez.

> [!note] ➕ Nota del editor
> Es la vía realista para un lago que ya funciona con políticas IAM y buckets: no hay que migrar todo el catálogo de golpe; se activa Lake Formation primero en la base de datos que más lo necesita (por ejemplo, una con datos personales).

### 3.5 Las tres personas del modelo

| Persona | Permisos |
|---|---|
| **Data lake administrator** | Acceso total a recursos y permisos para **administrar data locations**; concede y revoca acceso a cualquier recurso (incluidos los propios); crea bases de datos y da permiso a otros para crearlas |
| **Database creator** | Todos los permisos sobre las **bases y tablas que crea**; puede designar a otros database creators por consola o API |
| **Table creator** | Permisos administrativos sobre las **tablas que crea**; puede ver las bases que las contienen |

### 3.6 Compartir datos

Lake Formation permite compartir datos **interna y externamente** entre cuentas de AWS, **AWS Organizations** o directamente con administradores IAM de otra cuenta, con acceso fino a los metadatos del catálogo y a los datos subyacentes. (Se profundiza en [[13-Lake-Formation-Avanzado]] y [[14-Arquitectura-de-Datos-Moderna]].)

### 3.7 Servicios integrados

Los permisos de Lake Formation se aplican, con control a nivel de columna, fila y celda, en **Amazon Athena, Amazon QuickSight, Amazon Redshift Spectrum, Amazon EMR y AWS Glue**.

---

## 4. Cómo se aplica un permiso: credential vending

Audiencia: 🔧

**Preparación (administrador):** el data lake administrator (o un usuario con permisos administrativos) configura las políticas de usuario sobre las tablas del catálogo con permisos de Lake Formation; luego él o un delegado **concede permisos** a usuarios sobre bases y tablas y **registra la ubicación S3** de la tabla en Lake Formation.

**Flujo de una consulta:**

```
 Principal ──consulta/ETL──► Motor integrado (Athena, Glue, EMR, Redshift Spectrum)
                                   │ ① Get metadata
                                   ▼
                             Glue Data Catalog ──② Check permissions──► Lake Formation
                                   │  devuelve solo los metadatos que el usuario puede ver
                                   │ ③ ¿Tabla administrada por LF?
                     ┌─────────────┴──────────────┐
                    Sí                            No
                     ▼                             ▼
     Lake Formation entrega credenciales     El motor va directo a S3;
     TEMPORALES al motor                     se evalúan bucket policy +
                     │ ④ Get data            política IAM del usuario
                     ▼
     Motor lee de S3, filtra columnas/filas/celdas y devuelve el resultado
```

1. **Get metadata:** el principal envía una consulta o script ETL al motor; el motor identifica la tabla y pide los metadatos al Data Catalog.
2. **Check permissions:** el catálogo verifica con Lake Formation; si el usuario está autorizado, devuelve los metadatos que puede ver.
3. **Get credentials:** el catálogo indica si la tabla está administrada por Lake Formation; si el dato está registrado, el motor pide a Lake Formation acceso temporal.
4. **Get data:** con esas credenciales temporales el motor lee de S3, aplica el filtrado (columnas, filas, celdas) y devuelve el resultado.

A esto se le llama **credential vending**. Si la tabla **no** está administrada por Lake Formation, la segunda llamada va **directo a S3** y se evalúan la **bucket policy** y la **política IAM** del usuario.

🎯 **Para el examen:**
- Control de acceso a nivel de **columna/fila/celda** sobre un data lake en S3 → **Lake Formation**.
- Evitar copias del dataset por equipo → permisos finos sobre una única fuente.
- Lake Formation **extiende** IAM; las tablas no registradas siguen el camino IAM + bucket policy.
- Seguir las buenas prácticas de seguridad de IAM al usar políticas IAM.

---

## 5. Demos y laboratorios

El módulo no tiene demos en video. Después de él, el curso ofrece un **laboratorio autoguiado** con entorno aprovisionado para **interactuar con AWS Lake Formation**; las instrucciones aparecen en la plataforma al llegar al lab y **no forman parte del material extraído**, por lo que aquí solo se registra su objetivo: poner en práctica la guía de tres pasos (registrar ubicación, crear base de datos, otorgar permisos). La lección "Evaluación de la semana 3" no trae otro contenido textual.

---

## 6. Arquitectura de referencia: estado al cierre del módulo

```
                 ┌────────────── AWS Lake Formation ──────────────┐
                 │  Permisos (grant/revoke: DB, tabla, col, fila) │
                 │  Data lake locations registradas · auditoría   │
                 │  Blueprints → Glue workflows (crawlers + jobs) │
                 └──────────────┬──────────────────────┬──────────┘
                                │                      │ credenciales temporales
                         Glue Data Catalog             ▼
   S3 (registrado, SSE-KMS) ◄────────────── Athena · Glue · EMR · Redshift Spectrum · QuickSight
                                IAM: acceso amplio a los servicios
```

---

## ✅ Autoevaluación

> [!question]- ¿Sobre qué servicio se construye Lake Formation y qué componentes de él usa?
> Sobre AWS Glue: Data Catalog, crawlers, jobs y workflows (más los blueprints propios de Lake Formation).

> [!question]- Enumere los tres pasos de la guía del dashboard.
> Registrar ubicaciones S3, crear bases de datos en el Data Catalog y otorgar permisos a nivel de base, tabla o columna.

> [!question]- ¿Cómo reduce costos el control fino de Lake Formation?
> Permite mantener una única fuente de verdad con permisos distintos por equipo, en vez de replicar datasets para cada uno; menos almacenamiento, menos pago.

> [!question]- ¿Cómo recomienda el curso repartir responsabilidades entre IAM y Lake Formation?
> IAM con alcance amplio para permitir el uso de los servicios del lago; Lake Formation para los permisos finos de personas sobre catálogo, ubicaciones S3 y datos.

> [!question]- ¿Qué es el hybrid access mode y para qué sirve?
> Usar a la vez permisos de Lake Formation y políticas IAM sobre el catálogo, para adoptar Lake Formation de forma incremental, caso de uso por caso de uso.

> [!question]- Describa el credential vending.
> El motor pide metadatos al catálogo, que verifica permisos con Lake Formation; si la tabla está administrada, Lake Formation entrega credenciales temporales al motor, que lee de S3 y filtra columnas, filas o celdas antes de devolver el resultado.

> [!question]- ¿Qué pasa si la tabla no está registrada en Lake Formation?
> El motor accede directamente a S3 y el acceso se decide con la bucket policy y la política IAM del usuario.

> [!question]- ¿Qué diferencia a un database creator de un table creator?
> El database creator tiene todos los permisos sobre las bases y tablas que crea y puede designar otros creadores; el table creator administra solo las tablas que crea y ve las bases que las contienen.

## 🧾 Resumen en 10 líneas

1. Lake Formation centraliza gobierno, protección y uso compartido de datos del lago.
2. Se construye sobre AWS Glue: Data Catalog (el cerebro), crawlers, jobs y workflows.
3. Los blueprints simplifican la ingesta y el procesamiento de fuentes comunes.
4. Cifra en reposo con KMS y en tránsito con SSL.
5. Guía de tres pasos: registrar S3, crear bases, otorgar permisos.
6. Permisos tipo RDBMS (grant/revoke) a nivel de base, tabla, columna, fila y celda.
7. Una sola fuente con permisos finos evita copias por equipo y abarata el storage.
8. IAM amplio para servicios; Lake Formation para el detalle; cambiar el default "solo IAM".
9. Hybrid access mode permite adoptar Lake Formation de forma incremental.
10. Credential vending: el motor recibe credenciales temporales y filtra el dato; sin registro, rige IAM + bucket policy.

## 📖 Recursos del módulo

- AWS Lake Formation Developer Guide · https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html
- Prácticas recomendadas de seguridad en IAM (Guía del usuario de IAM) · citada en la lectura, sin URL

> [!warning] ⚠️ Dato a verificar
> El video afirma que "para julio de 2024" Lake Formation tendrá activado por defecto el control de acceso solo con IAM; la redacción (traducción automática) es ambigua sobre la fecha. Lo firme es que el default "Use only IAM access control" existe por compatibilidad y debe desactivarse para usar permisos finos.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[10-Ingesta-Catalogo-y-Preparacion|10 · Ingesta, Catálogo y Preparación]] · Siguiente: [[12-Procesamiento-y-Analisis|12 · Procesamiento y Análisis ➡]]
