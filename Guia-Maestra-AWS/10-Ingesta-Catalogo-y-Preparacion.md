---
title: "Tomo 10 — Ingesta, Catálogo y Preparación de Datos"
tags: [aws, solutions-architect, data-lake, s3, glue, athena, ingesta]
curso: 3 · Introduction to Designing Data Lakes on AWS
modulo: "Módulo 2: Introducción, catalogación y preparación de datos"
audiencias: [tecnico, puente, ejecutivo]
tomo: 10
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 📥 Tomo 10 — Ingesta, Catálogo y Preparación de Datos

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[09-Introduccion-a-Data-Lakes|09 · Introducción a los Data Lakes]] · Siguiente: [[11-Data-Lake-con-Lake-Formation|11 · Data Lake con Lake Formation ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Cubre las capas que **todo** data lake tiene sí o sí: ingesta, almacenamiento, catálogo y procesamiento. Explica por qué S3 es el almacenamiento por defecto, qué servicio de ingesta corresponde a cada tipo de dato (archivos, transaccional, streaming), cómo un Glue crawler infiere el esquema y llena el Data Catalog, y las cuatro prácticas que deciden costo y rendimiento: **formato columnar, partición, compresión y compactación**. Cierra con Athena y una comparación medida CSV vs Parquet.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** qué servicio de ingesta usar según el origen, cómo organizar y formatear los datos en S3, cuánto costará consultarlos.
> - **Costo de hacerlo mal:** consultas que escanean el archivo completo en cada pregunta (y Athena cobra por dato escaneado), millones de archivos pequeños que disparan llamadas a la API de S3, un lago sin catálogo que nadie sabe usar.
> - **Pregunta que responde:** *¿cómo meto los datos al lago y los dejo en una forma que sea barata y rápida de consultar?*

> [!tip] 💡 Analogía general: la molienda del café
> El instructor compara partición y compactación con moler café: cada bebida pide una molienda distinta, y cada patrón de consulta pide un tamaño de archivo distinto. Ni tan grueso que haya que leerlo todo, ni tan fino que se vuelva polvo de archivos diminutos. Para saber la molienda correcta hay que saber **cómo se consultarán** los datos, y por eso el equipo de datos debe estar en las reuniones de negocio.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Objetivos del módulo | 1 video | Capas de un data lake y temas del módulo |
| Almacenamiento en lagos de datos | 1 video | Por qué S3; separación storage/cómputo; tipos de dato raw → publicado |
| Ingestión de datos | 1 video | Archivos, transaccional y streaming, con sus servicios |
| Datos de rastreo y catálogo | 1 video | Glue Data Catalog, crawlers, conexiones, classifiers |
| Demo: creación y ejecución de un Glue crawler | 1 video | Crawler sobre CSV de NYC TLC |
| Formateo de los datos en el lago | 1 video | Formatos columnares |
| Particionar, comprimir y compactar | 1 video | Partición, compresión, compactación y cifrado |
| Consulta de datos con Amazon Athena | 1 video | Athena: serverless, precio, federated query, integraciones |
| Demo: consulta de datos CSV con Athena | 1 video | Consultas sobre la tabla CSV |
| Demo: formatos columnares, rendimiento y costo | 1 video | Misma consulta en CSV y Parquet |
| Encuesta de mitad de curso | 1 lectura + 1 encuesta | Sin contenido técnico |

---

## 1. Las capas de un data lake

Audiencia: 🧭

La mayoría de las arquitecturas de data lake se componen de estas capas:

```
 Fuentes → Ingesta → Almacenamiento → Catálogo y procesamiento → Búsqueda y análisis → Visualización
           └──────────────── siempre presentes ───────────────┘                        (opcional)
```

Algunos lagos no tienen capa de visualización, pero **todos** tienen ingesta, almacenamiento, catalogación y procesamiento: esas son las capas de este módulo.

---

## 2. Almacenamiento: por qué Amazon S3

Audiencia: 🔧 🧭

Cuando se piensa en almacenamiento de un data lake en AWS, se piensa en **Amazon S3**, por tres factores: **escalabilidad masiva, costo y durabilidad**. S3 no trabaja aislado: es el componente central sobre el que después se suman consulta, búsqueda, procesamiento y visualización.

### 2.1 Separar almacenamiento y cómputo

El gran cambio respecto de los flujos tradicionales es **desacoplar storage de processing**, para escalar y administrar cada capa de forma independiente. S3 hace solo almacenamiento; el procesamiento lo hacen otras herramientas o bases de datos.

> [!tip] 💡 Analogía de los dos garajes
> Un garaje para guardar las herramientas y otro para las mesas de trabajo: se puede ampliar uno sin tocar el otro. Así se escala el cómputo según la demanda sin preocuparse del almacenamiento, y viceversa.

### 2.2 Fortalezas de S3 que destaca el curso

| Fortaleza | Detalle |
|---|---|
| Escala | Almacenamiento de objetos a escala de **exabytes** |
| Durabilidad | Replicación dentro de una Region y, si se quiere, entre Regions |
| Consistencia | **Strong read-after-write consistency** |
| Costo | Optimización mediante storage classes (niveles) |
| Acceso | APIs estandarizadas, arquitectura serverless |
| Agnóstico al dato | Video, texto, datos de sensores, binarios, historias clínicas, redes sociales: si se tiene permiso para `PutObject`, S3 lo guarda |
| Rendimiento | Altas tasas de throughput y transacciones de forma automática |

### 2.3 Del dato raw al dato publicado

El valor del dato **aumenta** a medida que pasa de raw a procesado (el diamante en bruto frente al diamante tallado). El curso distingue cuatro tipos:

| Tipo | Qué es | Uso |
|---|---|---|
| **Raw** | Formato original, sin modificar (logs, datos de sensores) | Ingenieros de infraestructura y administradores de datos |
| **Formatted** | Forma optimizada del raw: datos columnares e indexados | Descubrimiento e investigación |
| **Transformed** | Reglas de negocio aplicadas, limpio | Análisis más profundo |
| **Published** | Datasets gestionados rigurosamente | Uso generalizado, generación de insights, conectar dashboards |

Es habitual tener varios datasets para distintos propósitos; S3 ayuda a gestionarlos manteniendo el **data lineage** y almacenándolos de forma segura.

> [!note] ➕ Nota del editor
> En un lago institucional esto suele aterrizar como zonas o prefijos: raw (réplica fiel del origen), stage/curated (tipado y limpio) y analytics (tablas listas para Power BI o QuickSight). Los cuatro niveles del curso son una versión más fina de la misma idea.

### 2.4 Optimizar costos de almacenamiento

- **S3 Storage Class Analysis**: analiza los patrones de acceso a los datos.
- **S3 Lifecycle rules**: automatizan la migración de objetos a niveles más baratos.
- **Amazon S3 Glacier**: el nivel más barato, de costo ultrabajo, pensado para archivo.

---

## 3. Ingesta según el tipo de dato

Audiencia: 🔧 🧭

El servicio de ingesta depende del tipo de dato. Casi todo cae en una de tres categorías:

| Categoría | Ejemplos | Servicios de AWS | Open source mencionado |
|---|---|---|---|
| **Archivos y objetos** (ya existen como archivo) | Imágenes, video, PDF, logs de aplicación, texto | Amazon AppFlow, AWS Transfer Family, AWS Snow Family | Logstash, Apache Flume, Apache Log4j |
| **Transaccional** (migración de bases de datos) | Bases de datos operacionales con lecturas y escrituras en curso | AWS DMS, AWS SCT | — |
| **Streaming** (se genera en tiempo real) | Datos de juegos, sensores IoT, clickstream web | Amazon Kinesis | Fluentd, Apache Sqoop, Kafka |

### 3.1 Archivos y objetos

- **AWS Transfer Family**: transferencias de archivos con **SFTP, FTPS o FTP**, creando endpoints seguros con pocos clics en la consola.
- **Amazon AppFlow**: intercambio seguro de datos entre aplicaciones **SaaS** (por ejemplo Salesforce) y servicios de AWS como S3.
- **AWS Snow Family**: para entornos con conectividad limitada o nula. AWS envía hardware físico, se carga localmente y sin conexión, y luego se sube a S3. El curso cita capacidad de **hasta 210 TB** por dispositivo; ideal para cargas masivas.

### 3.2 Transaccional

La clave es replicar no solo la base completa sino **su uso actual**: las lecturas y escrituras deben replicarse en el destino con latencia mínima.

| Herramienta | Qué es | Detalle |
|---|---|---|
| **AWS DMS** (Database Migration Service) | Servicio administrado de migración y **replicación** | Traslada bases de datos y cargas analíticas a AWS con **mínimo downtime y sin pérdida de datos**; soporta **más de 20 motores** |
| **AWS SCT** (Schema Conversion Tool) | **Software** que se instala (no es un servicio en la nube) | Automatiza la conversión de esquemas; orígenes/destinos como Oracle, SQL Server, PostgreSQL, MySQL, Amazon Redshift |

> [!example] 📊 DMS + SCT trabajan juntos
> Migrar Oracle a **Amazon Aurora**: primero **SCT** convierte el esquema; luego **DMS** migra y sincroniza los datos hacia AWS.

### 3.3 Streaming: Amazon Kinesis

Amazon Kinesis es un servicio totalmente administrado para ingerir y procesar datos de streaming a cualquier escala, incluso **video y audio** con procesamiento en tiempo real. Arquitectura de ejemplo del curso:

```
 EC2 (app + Kinesis Agent) ──► Kinesis Data Firehose ──► S3 (raw)
                                       │
                                       ├──► AWS Lambda ─────┐
                                       └──► Kinesis Data    ├──► S3 (procesado) ──► consumidores
                                            Analytics ──────┘
```

Resultado: en S3 quedan **tanto los datos raw** que envió el agente **como los procesados**, listos para los consumidores.

🎯 **Para el examen:**
- SFTP/FTPS/FTP hacia S3 → **Transfer Family**. SaaS (Salesforce) → **AppFlow**. Sin conectividad o volumen masivo → **Snow Family**.
- Migrar bases con replicación continua → **DMS**; motores heterogéneos → **SCT** + DMS.
- Tiempo real, clickstream, IoT → **Kinesis** (Firehose para entregar a S3).

---

## 4. Catalogación con AWS Glue

Audiencia: 🔧 🧭

### 4.1 Para qué catalogar

Catalogar organiza y administra el gran volumen de datos del lago y agrega **descubrimiento, gestión de metadatos, gobierno, colaboración, eficiencia y mejor análisis**. Convierte un repositorio caótico de datos raw en un recurso organizado. Ingerir todo en S3 sin orden se sale de control rápido, y como S3 cobra por uso, el desorden cuesta.

### 4.2 Glue Data Catalog y crawlers

- **AWS Glue Data Catalog**: repositorio central de **metadatos**.
- **Crawler**: recorre el dataset, **infiere el esquema** y crea o actualiza tablas en el Data Catalog.
- **Las tablas son punteros**: guardan metadatos que apuntan al dato real (normalmente en S3, también otras fuentes). Glue **no almacena los datos** en sus tablas; la propiedad **`location`** indica dónde están.
- Ventaja: una tabla es un objeto sobre el que se pueden definir **restricciones de acceso** con permisos de AWS, y se pueden **unir tablas** que apuntan a fuentes distintas.

### 4.3 Cómo se conectan los crawlers

| Tipo de conexión | Fuentes |
|---|---|
| Clientes nativos | Amazon S3, Amazon DynamoDB |
| Driver **JDBC** | Amazon RDS, Amazon Aurora, MariaDB, Microsoft SQL Server, MySQL, Oracle, PostgreSQL |
| Modo compatible con **MongoDB** | MongoDB, Amazon DocumentDB |

### 4.4 Classifiers: dónde ocurre la inferencia

Al conectarse a una fuente, el crawler ejecuta un **classifier**, que es donde se infiere el esquema:

- **Built-in classifiers**: reconocen formatos comunes como **JSON, CSV y Apache Avro**, entre otros.
- **Custom classifiers**: para formatos propios, por ejemplo el archivo de log de una aplicación.

Al terminar con éxito, el crawler deja el esquema y **crea o actualiza** la tabla en el Data Catalog.

```
 Fuente (S3, JDBC, Mongo) ──► Crawler ──► Classifier (built-in / custom) ──► esquema inferido
                                                                              │
                                                                              ▼
                                                     Tabla en Glue Data Catalog (metadatos + location)
```

---

## 5. Formato, partición, compresión y compactación

Audiencia: 🔧 🧭 👔

Práctica común: ingerir el raw en su forma original y **conservarlo como fuente original de verdad**, pero **no consultarlo**: es computacionalmente caro y lento. El curso sugiere **Amazon S3 Glacier** como buen lugar para el raw, por su naturaleza de archivo. Luego se aplican cuatro prácticas.

### 5.1 Formato columnar

El CSV guarda **por filas**: para `SELECT id FROM tabla` hay que recorrer cada línea completa, potencialmente millones, para extraer una sola columna. Un formato **columnar** "transpone la planilla": guarda cada columna junta, de modo que leer una columna completa requiere **mucha menos I/O de disco**.

```
 Por filas (CSV)                   Columnar (Parquet / ORC)
 id,edad,estado                    id:     1 | 2 | 3 | …
 1,34,CA                           edad:   34| 51| 27| …
 2,51,FL                           estado: CA| FL| WA| …
 3,27,WA                           → SELECT id lee solo el bloque "id"
```

Consulta más rápida = recursos asignados menos tiempo = **menor costo**. **AWS Glue jobs** convierten datos a **Apache Parquet** u **ORC** sin implementarlo desde cero.

### 5.2 Partición

Dividir el dataset en **trozos que probablemente se consultarán**. Ejemplo del curso: viajes en taxi de Nueva York con un CSV por mes y miles de millones de registros; para calcular la propina promedio del 5 de febrero hay que leer todo el mes. Si los archivos se dividen **por día**, el motor lee solo el del día pedido y escanea mucho menos.

> [!warning] ⚠️ Partir según la consulta, no por partir
> La regla: dividir en los trozos con que **probablemente** se consultará. Demasiadas particiones generan el problema de los archivos pequeños (ver compactación).

### 5.3 Compresión

Reduce el tamaño de los datos. El algoritmo más común es **Snappy**; también **LZO** y **GZIP**. Menos datos = menor factura de almacenamiento (pay-as-you-go) y, en Athena, menos datos escaneados.

### 5.4 Compactación

Combinar los registros de **muchos archivos pequeños en pocos archivos grandes**. Siguiendo el ejemplo del taxi: diario está bien, pero un CSV **por hora** o, peor, **uno por viaje**, provoca enormes gastos y consultas lentas por la cantidad de llamadas a la API de S3 necesarias para leer millones de archivos. Lo ideal es una **granularidad media**. Los Glue jobs también pueden agregar datos en archivos más grandes.

### 5.5 Seguridad: el "job zero"

- **Cifrado en tránsito**: AWS lo aplica **por defecto** al interactuar con S3.
- **Cifrado en reposo**: mediante **server-side encryption** o **client-side encryption**.

| Práctica | Qué ataca | Herramienta |
|---|---|---|
| Formato columnar | I/O por columna innecesaria | Glue job → Parquet/ORC |
| Partición | Escaneo de períodos no pedidos | Prefijos tipo `año=/mes=/día=` |
| Compresión | Tamaño en disco y escaneo | Snappy, LZO, GZIP |
| Compactación | Exceso de archivos pequeños / llamadas API | Glue job que agrega |
| Cifrado | Exposición del dato | Tránsito (por defecto) + SSE / CSE |

> [!note] ➕ Nota del editor
> En Athena estas cuatro prácticas son la diferencia entre una vista que escanea gigas y una que escanea megas. Si una tabla de stage es una réplica diaria sobrescrita, la partición por fecha de carga ayuda poco; conviene partir por la columna que más filtran las queries (período académico, por ejemplo).

---

## 6. Consulta con Amazon Athena

Audiencia: 🔧 🧭

- **Qué es:** servicio interactivo para consultar datos de S3 con **SQL estándar**; no requiere conocer ningún SDK.
- **Serverless:** no hay sistema operativo, paquetes ni red que configurar. Basta una **tabla del Glue Data Catalog** y un **usuario IAM autorizado**.
- **Escala** enormemente y **cobra por cantidad de datos escaneados** (precio por TB escaneado; ver `aws.amazon.com/athena/pricing`).
- **Formatos:** CSV, JSON y formatos como Avro y ORC (el curso los agrupa como "columnares"; ver nota de datos dudosos).
- **Ahorro:** la página de precios indica que se puede ahorrar **hasta 90 % por consulta** y ganar rendimiento comprimiendo, particionando y convirtiendo a columnar.

### 6.1 Integraciones que propone el curso

| Integración | Para qué |
|---|---|
| **AWS Lambda** + **API Gateway** | Exponer el lago a fuentes externas, eventos programados o APIs internas |
| **Amazon QuickSight** | Dashboards que ejecutan las consultas bajo demanda y muestran siempre lo más actual |
| **Federated query** | Unir datos de S3 con datos fuera de S3 mediante conectores, sin copiarlos (antes había que copiarlos a S3, con riesgo de dato estancado) |

### 6.2 Athena y Glue: "mejores amigos"

Si se crea una tabla en Athena, aparece en el Glue Data Catalog. La tabla es una **entidad de metadatos**; los datos no le pertenecen. Por eso al crearla se especifica una **location** (S3, DynamoDB, etc.), y si se borra la tabla, **los datos permanecen** en la fuente.

> [!danger] 🚨 Borrar la tabla no borra el dato (y viceversa)
> `DROP TABLE` de una tabla externa elimina solo metadatos. Esto tranquiliza, pero también significa que limpiar el catálogo **no** libera almacenamiento en S3 ni reduce su costo.

---

## 7. Demos y laboratorios

### 7.1 Demo — Crear y ejecutar un Glue crawler sobre CSV

**Objetivo:** inferir el esquema de un CSV y crear la tabla en el Data Catalog. **Servicios:** S3, AWS CLI, Glue, CloudWatch Logs, Athena. **Dataset:** NYC **TLC** (Taxi & Limousine Commission), público, copiado por el instructor a un bucket propio.

1. Explorar el bucket con `aws s3 ls`: contiene carpetas `csv`, `parquet` y scripts. En `csv/` hay `type=fhv` (for-hire vehicle), `type=green`, `type=yellow`; dentro, `year=…` y `month=…` (ya particionado).
2. Ubicar el archivo de **junio 2018** del taxi amarillo (~733–769 MB según la vista).
3. Copiarlo al equipo local con `aws s3 cp … .` e inspeccionarlo con `head`: la primera línea trae el encabezado (`VendorID`, `tpep_pickup_datetime`, `tpep_dropoff_datetime`, `passenger_count`, `trip_distance`, …).
4. Copiarlo de bucket a bucket (`aws s3 cp`, mucho más rápido por ir dentro de AWS en la misma Region) a `tlc-demo-raf/csv/`: un subconjunto solo para la demo.
5. En **Glue → Databases → Add database**: crear la base **`tlc`**.
6. En **Glue → Crawlers → Create crawler**: nombre `CSV-Crawler`.
7. Data source: S3, ruta `s3://tlc-demo-raf/csv`.
8. IAM role: `AWSGlueServiceRole-S3FullAccess` (el crawler necesita leer S3 para inferir el esquema).
9. Target database: `tlc`; dejar sin cambios el prefijo de nombre de tabla y el umbral máximo de tablas. **Create crawler**.
10. El crawler se crea **on demand**: pulsar **Run crawler** y seguir el estado con el botón de refresco.
11. Seguir el log en vivo: los crawlers escriben en **CloudWatch Logs**, grupo `/aws-glue/crawlers`. El instructor usa la herramienta open source `cw` (`cw tail -f /aws-glue/crawlers`); desde la consola equivale a **View CloudWatch logs**. Tarda ~1,5 min.
12. Resultado: estado **Completed**, *1 table change, 0 partition changes*; se creó la tabla **`csv`** en la base `tlc`.
13. En Athena: **Generate table DDL** muestra una `EXTERNAL TABLE` con tipos inferidos (`vendorid` **bigint**, `fare_amount` **double**, `improvement_surcharge` double…) y la propiedad **`LOCATION`** apuntando a S3, más propiedades que registran qué crawler la creó.

**Lección clave:** si se elimina la tabla, en S3 no se borra nada; solo desaparecen los metadatos.

### 7.2 Demo — Consultar el CSV con Athena

1. **Preview table** = `SELECT * FROM … LIMIT 10`, para comprobar que la tabla funciona. Athena lee el CSV raw desde S3.
2. `SELECT count(*)` → **8.713.832** viajes en junio 2018; **113 ms**; escaneó **733 MB** (el archivo entero).
3. `SELECT avg(tip_amount)` → propina promedio **USD 1,90**; vuelve a escanear 733 MB.
4. `max(tip_amount)` → **USD 411**.

**Observación:** cada consulta escaneó el archivo completo aunque solo pedía una columna, porque CSV no es columnar.

### 7.3 Demo — CSV vs Parquet: rendimiento y costo

1. En el bucket origen ya existe la versión **Parquet** (comprimida y particionada: `parquet/type=yellow/year=2018/month=06/`), que pesa unos **19 MB** frente a ~730 MB del CSV.
2. `aws s3 sync` (porque son varios archivos) de `…/parquet/type=yellow` a `s3://tlc-demo-raf/parquet`. Ahora el bucket tiene dos **prefixes**: `csv/` y `parquet/`.
3. Crear un segundo crawler `Parquet-Crawler` con data source `s3://tlc-demo-raf/parquet/` (con barra final), base `tlc`. Se podría reutilizar el primero, pero se crea uno nuevo. **Run crawler** (~1,5 min, seguido en CloudWatch).
4. Resultado: nueva tabla **`parquet`** en `tlc`.
5. Misma consulta en ambas: `SELECT avg(fare_amount), avg(tip_amount)` → tarifa media **USD 13**, propina media **USD 1,9**.

| Tabla | Datos escaneados | Tiempo |
|---|---|---|
| `csv` | Archivo completo (~733 MB) | **2,18 s** |
| `parquet` | Solo las dos columnas pedidas (mucho menos) | **811 ms** |

**Conclusión del instructor:** transformar a Parquet con Glue jobs y consultar con Athena es siempre una buena práctica: más rápido y, como se paga por dato escaneado, más barato. El **lab autoguiado** del curso repite esta comparación.

**Errores comunes:** olvidar pulsar *Run crawler* (el crawler on demand no corre solo); elegir un role sin acceso a S3; apuntar el crawler a la carpeta equivocada.

---

## 8. Arquitectura de referencia: estado al cierre del módulo

```
 Archivos ─(Transfer Family / AppFlow / Snow)─┐
 Bases de datos ─(SCT + DMS)──────────────────┤
 Streaming ─(Kinesis Agent → Firehose)────────┤
                                              ▼
            S3 raw (fuente de verdad; archivable en Glacier)
                     │  Glue job: Parquet + Snappy + particiones + compactación
                     ▼
            S3 formatted / transformed / published
                     │
            Glue crawler ──► Glue Data Catalog (tablas = metadatos + LOCATION)
                                      │
                                      ▼
            Athena (SQL, pago por TB escaneado, federated query)
               ├─► QuickSight
               └─► Lambda + API Gateway
```

---

## ✅ Autoevaluación

> [!question]- ¿Qué ventaja concreta da separar almacenamiento y cómputo?
> Escalar cada capa de forma independiente: más cómputo cuando hay demanda sin mover el almacenamiento, y más storage sin pagar cómputo ocioso.

> [!question]- Un proveedor entrega archivos por SFTP y otro equipo usa Salesforce. ¿Qué servicios usaría?
> AWS Transfer Family para el SFTP y Amazon AppFlow para Salesforce, ambos con destino S3.

> [!question]- ¿Por qué DMS y SCT suelen ir juntos?
> SCT convierte el esquema entre motores distintos (p. ej. Oracle → Aurora) y DMS migra y replica los datos con mínimo downtime.

> [!question]- ¿Qué guarda realmente una tabla del Glue Data Catalog?
> Metadatos: esquema, tipos, propiedades y la `location` del dato. El dato sigue en S3 (u otra fuente); borrar la tabla no lo borra.

> [!question]- ¿Qué hace un classifier y cuándo crearía uno custom?
> Es el componente del crawler que infiere el esquema. Uno custom sirve para formatos propios, como logs de una aplicación, que los built-in (JSON, CSV, Avro…) no reconocen.

> [!question]- ¿Por qué particionar por hora o por viaje es mala idea?
> Genera millones de archivos pequeños: muchas llamadas a la API de S3, más costo y consultas lentas. Se busca granularidad media y se compacta.

> [!question]- En la demo, ¿por qué la consulta en Parquet escaneó menos que en CSV?
> Porque Parquet es columnar (y comprimido): Athena leyó solo las columnas `fare_amount` y `tip_amount`, no el archivo completo.

> [!question]- ¿Cómo reduce costos en Athena cada una de las prácticas de datos?
> Athena cobra por dato escaneado: columnar evita leer columnas no pedidas, partición evita leer períodos no pedidos y compresión reduce los bytes; la página de precios habla de hasta 90 % de ahorro.

> [!question]- ¿Qué es una federated query y qué problema resuelve?
> Una consulta de Athena que une datos de S3 con datos fuera de S3 mediante conectores; evita copiarlos a S3, con su riesgo de datos desactualizados.

## 🧾 Resumen en 10 líneas

1. Todo data lake tiene ingesta, almacenamiento, catálogo y procesamiento; la visualización es opcional.
2. S3 es el almacenamiento del lago: escala de exabytes, durable, strong consistency, barato y agnóstico al dato.
3. Separar storage y cómputo permite escalar cada uno por su lado.
4. Niveles de dato: raw → formatted → transformed → published; el valor crece con el procesamiento.
5. Ingesta: archivos (Transfer Family, AppFlow, Snow), transaccional (DMS + SCT), streaming (Kinesis).
6. Glue Data Catalog guarda metadatos; las tablas apuntan al dato vía `location`.
7. Crawlers se conectan por cliente nativo, JDBC o modo MongoDB y usan classifiers para inferir el esquema.
8. Columnar, partición, compresión (Snappy) y compactación bajan costo y tiempo; cifrar en tránsito y en reposo.
9. Athena: SQL serverless sobre S3, cobra por dato escaneado, admite federated query y se integra con QuickSight y Lambda.
10. Demo NYC TLC: la misma consulta tardó 2,18 s en CSV y 811 ms en Parquet, escaneando muchísimo menos.

## 📖 Recursos del módulo

- Precios de Amazon Athena · https://aws.amazon.com/athena/pricing (mostrado en video)

> [!warning] ⚠️ Datos que conviene verificar
> - En la demo Parquet el instructor dice que 730 MB pasaron a ~19 MB y lo llama "cinco veces menos"; la proporción real sería cercana a 38×. Se conservan las cifras, no la razón.
> - El tamaño del CSV aparece como 769 MB (listado) y 733 MB (descarga y escaneo en Athena).
> - El curso agrupa Avro con los formatos columnares; Avro es en realidad un formato por filas.
> - "Hasta 210 TB" para Snow Family es la cifra citada en el video; la capacidad depende del dispositivo.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[09-Introduccion-a-Data-Lakes|09 · Introducción a los Data Lakes]] · Siguiente: [[11-Data-Lake-con-Lake-Formation|11 · Data Lake con Lake Formation ➡]]
