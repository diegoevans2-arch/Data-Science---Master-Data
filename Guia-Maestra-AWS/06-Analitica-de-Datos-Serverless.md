---
title: "Tomo 06 — Diseño de una solución de analítica de datos serverless"
tags: [aws, solutions-architect, analytics, clickstream, s3, kinesis, firehose, athena, quicksight, data-lake]
curso: 2 · Architecting Solutions on AWS
modulo: Diseño de una solución de análisis de datos sin servidor en AWS
audiencias: [tecnico, puente, ejecutivo]
tomo: 06
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 📊 Tomo 06 — Diseño de una solución de analítica de datos serverless

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[05-Backend-Web-Serverless|05 · Backend Web Serverless]] · Siguiente: [[07-Solucion-Hibrida-Contenedores|07 · Solución Híbrida con Contenedores ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Es el patrón analítico serverless más reproducido de AWS: **ingesta por HTTPS → buffer y entrega con Kinesis Data Firehose → data lake en S3 → SQL con Athena → BI con QuickSight**. El módulo lo construye para un cliente de clickstream, pero cada decisión (separar almacenamiento de cómputo, elegir Firehose frente a Data Streams por la latencia tolerada, Athena frente a S3 Select, Glue y EMR, QuickSight frente a Grafana) se transfiere a cualquier lago de datos. Cierra con las tres prácticas que más bajan el costo de Athena: **comprimir, particionar y convertir a formato columnar**.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** añadir analítica a un producto sin contratar equipo de big data; pagar por datos almacenados, registros ingeridos y bytes escaneados, no por horas de clúster; reutilizar la herramienta de BI que el equipo ya conoce.
> - **Costo de hacerlo mal:** clústeres (EMR, OpenSearch) encendidos a las 3 de la mañana sin nadie consultando; consultas que escanean un archivo de 5 TB entero para leer un mes; datos de un frontend mal configurado contaminando el lago sin posibilidad de reproceso.
> - **Pregunta que responde:** *¿cómo convierto los clics de miles de usuarios en información para el negocio, con poco personal, costo por uso y respaldo cifrado en otra región?*

> [!tip] 💡 Analogía general: el correo de una ciudad
> Los clics son cartas que se depositan en un buzón público con dirección conocida (API Gateway). El camión de correo (Firehose) no va al centro de clasificación por cada carta: espera a llenar una saca o a que pase cierto tiempo, y la descarga ordenada por fecha. El centro de archivo (S3) guarda todo tal como llega, sin importar quién lo vaya a leer. El analista (Athena) no traslada las cajas a su escritorio: va al archivo con una ficha que describe cómo están ordenadas y lee solo los estantes que le interesan. El diario mural (QuickSight) muestra los resultados al resto de la empresa. Si el archivo está ordenado por mes y las cartas vienen comprimidas, el analista recorre menos pasillos y cobra menos.

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Cliente n.º 2 — Software House | 3 videos | Caso de los menús por código QR, requisitos, alcance dentro/fuera |
| Arquitectura de la solución | 8 videos · 4 lecturas | Panorama de servicios de datos; almacenamiento (S3 vs EBS vs EFS); ingesta (Kinesis vs EMR, DMS, Data Exchange); consulta (Athena vs S3 Select, Glue, EMR); visualización (QuickSight vs CloudWatch, OpenSearch, Managed Grafana); exploraciones de S3, Firehose y Athena |
| Presentación al cliente y próximos pasos | 4 videos · 2 lecturas (incluye el reto) | Solución final, saneamiento con Lambda y doble bucket, optimizaciones (serverless, CloudFront, Cognito, CloudFormation, backoff exponencial, optimización para Athena), recorrido de la POC en 2 partes |

**Roles:** en esta semana **Raf Lopes** es el Solutions Architect y **Morgan Willis** hace de cliente.

---

## 1. El caso: Software House y sus menús por código QR

Audiencia: 🔧 🧭 👔

### 1.1 Situación actual

```
 SITUACIÓN ACTUAL (fuera del alcance del proyecto, ya construido y mantenido por el cliente)
 Comensal ─ escanea QR ─► Bucket S3 público (static website hosting) ─► index.html del menú
                                   ▲
 Administrador del restaurante ─► Sistema de actualización de menús (en EC2) ─ sube el objeto
                                   │
                         Generador de códigos QR (autoservicio para el restaurante)
```

- Una empresa de software ofrece **menús de restaurante en páginas HTML estáticas** alojadas en un bucket de **S3 con static website hosting**.
- Los administradores de cada restaurante entran a un sistema propio del cliente y **actualizan el objeto** en el bucket (precios, descripciones, fotos).
- Un sistema **genera el código QR** que apunta al bucket; el restaurante lo imprime y lo pone en las mesas.
- Problema de negocio: el comensal elige pero igual espera al mesero. El cliente quiere agregar una opción **«pedir este artículo»** que llame a una API de pagos de terceros que ya tiene (un QR por mesa servirá como identificador del pedido).
- **Lo nuevo que piden:** analítica para los restaurantes — qué platos se ven, si la gente recorre todo el menú o se queda en los aperitivos, cómo optimizar el diseño del menú.

Raf añade posibles insights: artículos cuya descripción se abre pero no se agregan a la cuenta, tiempo entre ampliar un artículo y pagarlo, recomendaciones cruzadas entre restaurantes basadas en **datos de sesión**. Todo eso es **análisis de clickstream**.

### 1.2 Alcance

| Dentro del alcance | Fuera del alcance |
|---|---|
| La **parte analítica**: recibir, almacenar, procesar y visualizar el clickstream | La **biblioteca JavaScript** del lado del cliente que genera los eventos (ya escrita) |
| | La **integración con la pasarela de pagos** |
| | El sitio estático, el sistema de menús y el generador de QR |

> [!tip] 💡 Buena práctica del SA
> Raf refuerza durante la llamada qué está dentro y fuera del alcance, y recomienda **documentarlo**: alinea expectativas entre lo que el cliente necesita y lo que el consultor entregará.

### 1.3 Requisitos

| # | Requisito | Lectura de Raf |
|---|---|---|
| 1 | **Endpoint HTTPS RESTful** donde la biblioteca JS hace POST | Probablemente buscan que alguien maneje la autenticación y poder cambiar el destino sin tocar el código del cliente → API Gateway |
| 2 | **Servicios gestionados**, sin EC2 ni sistemas operativos que mantener (personal reducido) | Serverless y gestionados |
| 3 | **Costo**: facturación por **uso granular** («uso refinado»), no por tiempo | Evitar todo lo basado en EC2; por ejemplo, S3 cobra por datos almacenados, no por horas de existencia del bucket |
| 4 | **Durabilidad**: datos ingeridos disponibles como **respaldo en otra región** | Replicación entre regiones del servicio de almacenamiento |
| 5 | **Cifrado en tránsito y en reposo** | Fácil de lograr en la nube (Raf cita la frase de Werner Vogels, CTO de Amazon, sobre cifrar como si todos estuvieran mirando) |

---

## 2. Panorama de servicios de datos en AWS

Audiencia: 🧭 👔

**Ideas fuerza de Raf:**
- El objetivo del análisis de datos es producir **insights** útiles para el negocio; el camino puede ser simple o requerir machine learning.
- **Si se controla cómo se producen los datos, todo es más fácil**: no hace falta transformarlos al llegar. Aquí el cliente controla la biblioteca JS → basta ingerir y almacenar tal cual.
- **Consejo n.º 1: pensar siempre en cómo llegarán los datos** a la arquitectura. Es aún más crítico en soluciones genéricas que reciben imágenes, audio, logs, fotos de distintas fuentes y dispositivos.
- **«La herramienta adecuada para el trabajo»**: hay tantos servicios que elegir el apropiado suele funcionar mejor y ahorrar dinero. Para elegir hay que entender qué hace cada uno y mapear cada requisito a un servicio.

**Preguntas para decidir entre servicios cuando más de uno sirve:**
1. ¿Qué **latencia** es aceptable entre la ingesta y la disponibilidad de los datos?
2. ¿De **dónde** vienen los datos? ¿Migración de base de datos, clickstream, carga de archivos por lotes?
3. ¿El cliente quiere usar **soluciones propias** o software con licencias y soporte vigentes?
4. ¿Necesita **federación de credenciales** propia e integración con proveedores de identidad existentes? Si es así, habrá que configurar servicios que hagan de puente.

La página `aws.amazon.com/big-data/datalakes-and-analytics` agrupa los servicios en **cuatro categorías**:

| Categoría | Servicio | Qué hace (según el curso) |
|---|---|---|
| **Data lakes y almacenamiento** | Amazon S3 | Almacenamiento de objetos escalable, disponible, seguro; base de data lakes, apps cloud native y móviles; clases de almacenamiento y controles de acceso finos |
| | S3 Glacier (clases) | Archivado de bajo costo, escalabilidad casi ilimitada, durabilidad de **11 nueves**, opciones de recuperación rápida |
| | AWS Lake Formation | Montar un data lake seguro **en días**. Data lake = repositorio centralizado, curado y seguro con datos en forma original y preparados para análisis; rompe silos |
| **Analítica** | Amazon Athena | Consulta interactiva con **SQL estándar sobre S3**, serverless, pago por consulta; se apunta a los datos, se define el esquema y se consulta; la mayoría de resultados en segundos; sin ETL complejo; para cualquiera que sepa SQL |
| | Amazon EMR | Big data a escala de **petabytes**, análisis interactivo y ML con Apache **Spark**, **Hive**, **Presto**; clústeres Hadoop gestionados |
| | Amazon OpenSearch Service | Análisis interactivo de logs, monitoreo en tiempo real, búsqueda en sitios. OpenSearch deriva de Elasticsearch; el servicio sucede a Amazon Elasticsearch Service, soporta **19 versiones** de Elasticsearch y visualización con OpenSearch Dashboards y Kibana |
| **Movimiento de datos** | Amazon Kinesis | Recolectar, procesar y analizar **streaming en tiempo real** (video, audio, logs, clickstream, telemetría IoT) a medida que llega, sin esperar a tener todo |
| | AWS Glue | **Integración de datos serverless**: descubrir, preparar y combinar datos para análisis, ML y desarrollo de aplicaciones «en minutos en vez de meses»; extraer, enriquecer, limpiar, normalizar, combinar, cargar en bases, warehouses y lagos |
| | AWS DMS | Migrar bases a AWS con la **base origen operativa** durante la migración (mínimo downtime); motores comerciales y open source |
| **Analítica predictiva y ML** | Amazon SageMaker | Construir, entrenar y desplegar modelos de ML con infraestructura y flujos gestionados; IDE para científicos de datos e interfaces sin código para analistas; tiene curva de aprendizaje |
| | Amazon Rekognition | Visión por computador por API (objetos en video en vivo, rostros y atributos: ojos abiertos, lentes, vello facial); analiza millones de imágenes y videos; no es analítica por sí solo pero puede ser parte de una solución |
| | Amazon Comprehend / Comprehend Medical | NLP con ML: insights en texto (centros de llamadas, documentos financieros); la variante Medical extrae información de texto médico no estructurado |

> [!note] ➕ Nota del editor
> La lectura afirma que Rekognition analiza millones de imágenes y videos «en 3 segundos»; la frase parece una mala traducción y no debe tomarse como una cifra de rendimiento. Por otra parte, desde 2023–2024 AWS renombró **Kinesis Data Firehose → Amazon Data Firehose** y **Kinesis Data Analytics → Amazon Managed Service for Apache Flink**; el curso usa los nombres antiguos y aquí se conservan para calzar con el material.

**Sobre EMR en este caso:** no se usa porque el cliente tiene poco personal y EMR exige curva de aprendizaje en frameworks open source («daría para un curso entero de 4 semanas»). Casos de uso que lista la lectura: procesamiento a gran escala y análisis *what-if* con modelos estadísticos y predictivos; extraer datos de diversas fuentes, procesarlos a escala y ponerlos a disposición; **pipelines de streaming** de larga duración, altamente disponibles y tolerantes a fallas (el que mejor calza con este escenario si el cliente tuviera conocimientos de big data); ML con Spark MLlib, TensorFlow, Apache MXNet; conexión con SageMaker Studio.

**Clickstream (definición del curso):** pequeños eventos con trozos de datos que se generan **continuamente, con gran velocidad y volumen**, normalmente desde frontends, sobre la interacción del usuario con el sistema. Sirve para analizar comportamiento (tiempo en la página, dónde empieza y termina la navegación) y, en tiempo real, para actualizar recomendaciones, hacer **pruebas A/B avanzadas** o enviar notificaciones push según la duración de la sesión.

---

## 3. Almacenamiento: por qué Amazon S3

Audiencia: 🔧 🧭

### 3.1 Comparación

| Criterio | Amazon EBS | Amazon EFS | **Amazon S3** |
|---|---|---|---|
| Tipo | Bloque | Sistema de archivos | Objetos |
| ¿Necesita estar adjunto a algo? | Sí, a una **instancia EC2** | Sí, a EC2, contenedores o servidores | **No: vive por sí mismo**, se usa por API desde cualquier lugar |
| Facturación | Por **capacidad aprovisionada** (un volumen de 50 GB se paga completo aunque se usen 2) | — | **Por uso granular**: 5 MB si se guardan 5 MB, 1 PB si se guarda 1 PB |
| Durabilidad | Menor que S3 | Replica dentro de la región (mayor que EBS) | **11 nueves** (99,999999999 %) anual; cada objeto se replica en distintas instalaciones físicas de la región |
| Veredicto | Descartado | Descartado (cumple durabilidad, pero no es una capa independiente) | **Elegido** |

### 3.2 Argumentos a favor de S3

- **«El almacenamiento de Internet»**: se interactúa con llamadas API (`PutObject`, `GetObject`, `DeleteObject`…) desde un portátil, una página web, un servidor o una app móvil.
- **Separa almacenamiento de procesamiento**, principio central de analítica y big data: permite cambiar o combinar mecanismos de ingesta y motores de procesamiento, cada uno el adecuado para su tarea. Por eso S3 se usa como **data lake** (el término evoca biomas en la naturaleza), y casi todos los servicios de AWS y muchas soluciones comerciales ya saben leer de S3.
- **Cumple todos los requisitos:** cobro por uso granular; **replicación entre regiones** sencilla; **cifrado en tránsito y en reposo sin costo adicional**; servicio gestionado sin instancias.
- **S3 Intelligent-Tiering** mueve objetos entre niveles automáticamente según patrones de acceso → ahorro automatizado.

### 3.3 Replicación entre regiones (S3 CRR)

- Replica objetos **con sus metadatos y etiquetas** hacia uno o varios buckets de destino en **otra región**.
- Se configura a nivel de **bucket**, de **prefijo compartido** o de **objeto** (con etiquetas de objeto).
- Puede **cambiar la propiedad de cuenta** de las réplicas para protegerlas de borrados accidentales.
- Casos de uso:
  - **Cumplimiento:** S3 ya guarda los datos en varias AZ geográficamente distantes, pero una norma puede exigir copias a **cientos de kilómetros**.
  - **Latencia:** copias cerca de usuarios distribuidos geográficamente.
  - **Eficiencia regional:** clústeres de cómputo en dos o más regiones analizando los mismos objetos.

### 3.4 Ciclo de vida, Intelligent-Tiering y Glacier

- **S3 Lifecycle:** reglas para **transicionar** objetos a otra clase. Ejemplos: a **S3 Standard-IA** si se accede poco; a **S3 Glacier Flexible Retrieval** si no se necesita acceso en tiempo real.
- **S3 Intelligent-Tiering:** para patrones de acceso **desconocidos, cambiantes o impredecibles**; sugerida como clase por defecto para casi cualquier carga, especialmente **data lakes**, analítica, apps nuevas y contenido generado por usuarios.
  - La única clase diseñada para ahorrar automáticamente al cambiar el patrón de acceso, **sin impacto en rendimiento ni carga operativa**.
  - Cobra una pequeña **tarifa mensual de monitoreo y automatización por objeto**; **sin cargos de recuperación**.
  - Sin tamaño mínimo de objeto, pero los objetos **menores de 128 KB no se escalonan**: siempre se cobran a tarifa de acceso frecuente y no pagan el cargo de monitoreo.
- **Clases S3 Glacier:** archivado de bajo costo, escalabilidad casi ilimitada, **11 nueves** de durabilidad y opciones de acceso rápido al archivo; se llega a ellas con políticas de lifecycle.

> [!note] ➕ Nota del editor
> Para un clickstream con Firehose, Intelligent-Tiering tiene un matiz práctico: si el buffer genera muchos objetos pequeños (bajo 128 KB), no se benefician del escalonamiento. Ajustar el tamaño del buffer de Firehose o compactar periódicamente ayuda a la vez a Intelligent-Tiering y a Athena (menos archivos que abrir por consulta).

> [!tip] 🎯 Para el examen
> - Capa de almacenamiento independiente del cómputo, pago por uso, 11 nueves → **S3**. Bloque adjunto a EC2 → EBS. Archivos compartidos montados → EFS.
> - Copia en otra región por cumplimiento, latencia o DR → **S3 CRR** (con metadatos y tags; por bucket, prefijo o tag).
> - Patrón de acceso desconocido → **Intelligent-Tiering** (objetos < 128 KB no se escalonan; sin cargo de recuperación).

---

## 4. Ingesta: Kinesis Data Firehose detrás de API Gateway

Audiencia: 🔧 🧭

### 4.1 Candidatos y descarte

| Candidato | Por qué no (o sí) |
|---|---|
| **Amazon EMR** + Spark Streaming | Funcionaría, pero cobra **por tiempo** de clúster y exige conocimientos de big data que un equipo pequeño quizá no tiene |
| **AWS DMS** | Pensado para **migrar bases de datos** manteniendo la origen operativa; aquí no hay base que migrar |
| **AWS Data Exchange** | Catálogos de **datos de terceros** para integrarlos a un lago; no es el propósito |
| **Kinesis Data Analytics** | Procesamiento **en tiempo real** (transformar, agregar, filtrar, limpiar) durante la ingesta; no hace falta porque el cliente controla el formato |
| **Kinesis Data Streams** | Menor latencia, pero exige **escribir código** de productor y consumidor |
| **Kinesis Data Firehose** | **Elegido**: más cómodo, sin código extra, entrega directa a S3 |

La familia Kinesis está diseñada para ingerir **grandes cantidades de datos pequeños**, justo el perfil del clickstream.

> [!example] 📊 Llamada de seguimiento n.º 1 — ¿qué latencia toleran?
> Raf llama a Morgan (que está jugando golf): ¿pueden esperar un par de minutos antes de que los datos estén en el bucket? Respuesta: sí, sin problema; es análisis para procesamiento asíncrono, **no hay dashboards en tiempo real**. **Decisión: Kinesis Data Firehose.** Raf subraya que Data Streams vs Firehose es **conveniencia frente a control**; no hay uno mejor, solo el recomendable para el escenario.

### 4.2 La familia Kinesis (lectura)

| Servicio | Para qué | Rasgos y cifras |
|---|---|---|
| **Kinesis Data Streams** | Streaming en tiempo real masivamente escalable y durable | Captura **gigabytes por segundo** desde **cientos de miles** de fuentes (clickstream, eventos de bases de datos, transacciones financieras, redes sociales, logs de TI, ubicación); datos disponibles en **milisegundos**; latencias **bajo 1 segundo**; casos: dashboards en tiempo real, detección de anomalías, precios dinámicos |
| **Kinesis Data Firehose** | Cargar streaming de forma confiable en lagos, warehouses y servicios de análisis | Destinos: **S3, Redshift, Elasticsearch/OpenSearch, endpoints HTTP genéricos y proveedores como Datadog, New Relic, MongoDB, Splunk**; totalmente gestionado, escala solo, casi sin administración; puede **agrupar en lotes, comprimir, transformar y cifrar** antes de cargar; mayor latencia (ejemplo: intervalo de lote de **60 segundos**) |
| **Kinesis Data Analytics** | Transformar y analizar streaming en tiempo real con **Apache Flink** | Reduce la complejidad de construir y operar apps Flink; escala solo; sin servidores, sin cuota mínima ni costo de configuración; pago por recursos consumidos |
| **Kinesis Video Streams** | Transmitir video desde dispositivos conectados para análisis, ML, reproducción | Escala a **millones de dispositivos**; almacena, cifra e indexa de forma durable; reproducción en vivo y a demanda; integra Rekognition Video y librerías ML (MXNet, TensorFlow, OpenCV); soporta **WebRTC** (videochat, streaming entre pares) |

### 4.3 El eslabón que falta: ¿cómo llega el POST a Kinesis?

Tanto Firehose como Data Streams se alimentan con llamadas a su API (por ejemplo `PutRecord`), y la biblioteca JS del cliente solo sabe hacer un **HTTPS POST**. Opciones:

| Opción | Cómo | Problema / ventaja |
|---|---|---|
| **Amazon Cognito** | *Identity pool* que entrega credenciales de AWS para llamar a la API de Kinesis desde el navegador | Obliga a **cambiar la biblioteca JS** del cliente |
| **Amazon API Gateway** | **Integración de servicio** que hace de *proxy* de las solicitudes HTTPS hacia Firehose | **Elegido**: no se toca la biblioteca y no se expone el delivery stream a Internet |

API Gateway **no es un servicio de ingesta**, pero sirve como proxy hacia otros servicios de AWS. En la presentación Raf añade que permite **dominio personalizado** y **certificados SSL propios**.

```
 Bloques al cierre de la sección 4:  [API Gateway] ──PutRecord──► [Kinesis Data Firehose] ──► [S3]
```

> [!tip] 🎯 Para el examen
> - Latencia de segundos-minutos tolerable, entrega a S3/Redshift/OpenSearch/Splunk sin código → **Firehose**. Sub-segundo, consumidores propios, reprocesamiento → **Data Streams**. SQL/Flink sobre el stream → **Data Analytics**. Video → **Video Streams**.
> - Exponer Kinesis a un cliente web sin cambiar su código → **API Gateway** con integración de servicio; alternativa con cambio de código → **Cognito** + SDK.

---

## 5. Acceso a los datos: Amazon Athena

Audiencia: 🔧 🧭

### 5.1 Candidatos

| Candidato | Ventaja | Por qué no |
|---|---|---|
| **S3 Select** | Serverless, gestionado, filtra el contenido de un objeto con SQL | Consulta **un solo objeto por consulta**; Firehose generará **muchos archivos pequeños** y no hay nada que los agregue |
| **AWS Glue** | Gran procesamiento; conversión de formatos, agregación, IA/ML | Curva de aprendizaje; brilla con datos **no estructurados** y frameworks de big data; contradice la consigna de arquitectura mínima |
| **Amazon EMR** | Ídem | Ídem, más cobro por tiempo |
| **Amazon Athena** | SQL estándar sobre S3, serverless, pago por consulta, «cualquiera con SQL» | **Elegido** |

> [!note] ➕ Nota del editor
> AWS cerró **S3 Select** a clientes nuevos en julio de 2024 y recomienda alternativas (Athena, filtrado del lado del cliente). La comparación del curso sigue sirviendo como razonamiento: un filtro por objeto no reemplaza a un motor de consulta sobre un prefijo completo.

### 5.2 Conceptos de Athena que exige el nivel Associate

1. **Athena no copia los datos.** Al crear una tabla se especifica un **prefijo de S3** donde buscar. Por eso el DDL es `CREATE EXTERNAL TABLE`: los datos siguen en S3, que sigue siendo el repositorio central y abierto a otras herramientas.
2. **La tabla debe calzar con el formato de los archivos.** Si los CSV traen `CustomerID`, `SessionTime`, fin de sesión y `NumberOfClicks`, la tabla debe tener esa estructura.
3. **Se pueden crear varias tablas y hacer JOIN entre ellas**, incluso si apuntan a buckets distintos; solo se exige que los datos de cada ubicación coincidan con el esquema de su tabla.
4. **SerDe (Serializer/Deserializer):** la tabla necesita un SerDe para entender los archivos. Ejemplos citados: **CSV**, **JSON** y uno de **expresiones regulares** (más flexible). En la demo aparece también **Parquet** (almacenamiento columnar).

> [!example] 📊 Llamada de seguimiento n.º 2 — ¿en qué formato llegan los datos?
> Raf quiere evitar un servicio extra para preparar y normalizar datos. Morgan consulta a sus desarrolladores: la biblioteca envía **JSON**. **Decisión: Athena** consultando el JSON tal cual llega, mientras se envíe siempre con la misma forma.

```
 Bloques al cierre de la sección 5:  [API Gateway] ─► [Firehose] ─► [S3] ◄─ [Athena]
```

> [!tip] 🎯 Para el examen
> - SQL ad hoc sobre S3 sin infraestructura → **Athena** (pago por datos escaneados). Un solo objeto → S3 Select (histórico). ETL y transformación compleja → Glue/EMR.
> - `CREATE EXTERNAL TABLE` + `LOCATION 's3://…'` + SerDe que calce con el formato.

---

## 6. Visualización: Amazon QuickSight

Audiencia: 🔧 🧭 👔

| Candidato | Qué ofrece | Veredicto |
|---|---|---|
| **Amazon CloudWatch** | Dashboards de **métricas operativas** (CPU de EC2, cantidad de objetos en un bucket), alarmas, detección de anomalías | Pensado para administradores de sistemas, no para BI → descartado |
| **Amazon OpenSearch Service** | Dashboards; suite open source (licencia Apache 2.0) de búsqueda y analítica | El clúster trae **almacenamiento y cómputo embebidos**; se quiere desacoplar usando S3, y su costo sería mayor que Athena + un servicio solo de visualización → descartado |
| **Amazon Managed Grafana** | Plataforma open source para consultar, visualizar, alertar sobre métricas; brilla con **series de tiempo** | Viable: el clickstream tiene marca de tiempo; aunque se usa sobre todo para monitoreo, sirve para BI → queda en la mesa |
| **Amazon QuickSight** | BI serverless, pago por uso, integración nativa con servicios de AWS (entre ellos **Athena**), seguridad incorporada | Viable → finalista |

> [!example] 📊 Llamada de seguimiento n.º 3 — ¿ya usan Grafana o QuickSight?
> Morgan confirma que usan **QuickSight** para dashboards de ventas y otro proyecto; el equipo de BI ya lo conoce (no sabe si se puede compartir la misma licencia). **Decisión: QuickSight.** Lección de Raf: *a veces el mejor servicio es el que el equipo ya sabe usar* (ahorra licencias y curva de aprendizaje).

### 6.1 Capacidades de QuickSight (lectura)

| Grupo | Capacidades |
|---|---|
| Conectar y escalar | Datos en AWS, en otras nubes u on-premises; motor en memoria **SPICE** para escalar la exploración a miles de usuarios; combinar fuentes y crear modelos de datos complejos compartidos de forma gobernada |
| Dashboards | Diseño personalizado por caso de uso; **informes y alertas por email**; acceso desde **iOS, Android** y web móvil |
| ML integrado | Preguntas en **lenguaje natural**; **detección de anomalías** continua; **forecasting** de métricas y análisis *what-if*; **narrativas automáticas** incrustadas en dashboards; búsqueda automática de patrones y outliers |
| BI de autoservicio | Preguntas simples sin formación en BI; autoría web; **embedding** de QuickSight en aplicaciones |
| Integración AWS | Conectividad **VPC privada** a Redshift, Snowflake, Exasol, RDS…; permisos **IAM** nativos para S3 y Athena con control fino; integración con **SageMaker** |
| Serverless y costo | Escala a **cientos de miles de usuarios** con alta disponibilidad; SPICE con autoescalado (no hay que escalar las bases); modelo **pago por sesión** (no comprar miles de licencias de usuario final) |
| Seguridad y cumplimiento | Cifrado de extremo a extremo y en reposo en SPICE; **seguridad a nivel de fila y de columna** con API; cumplimiento HIPAA, HITRUST CSF, GDPR, SOC, PCI, ISO 27001, FedRAMP High, entre otros |

---

## 7. La solución presentada al cliente

Audiencia: 🔧 🧭 👔

Raf presenta la propuesta en su oficina (Morgan llega «muy rápido»). Son **cinco servicios**, todos con facturación granular: a las 2, 3 o 4 de la mañana, sin comensales, el costo cae porque nada cobra por hora.

```
 FUERA DEL ALCANCE                              DENTRO DEL ALCANCE (solución analítica)
 ┌────────────────────────────┐
 │ Comensal escanea QR        │
 │   └► Bucket S3 (website)   │   biblioteca JS ── HTTPS POST (JSON) ──┐
 │ Admin restaurante ─► EC2   │                                        ▼
 │   (sistema de menús)       │                               ┌─────────────────┐
 └────────────────────────────┘                               │ Amazon API      │ dominio propio, SSL,
                                                              │ Gateway (REST)  │ no expone Firehose
                                                              └────────┬────────┘
                                                                       │ integración de servicio: PutRecord
                                                                       ▼
                                                              ┌─────────────────┐
                                                              │ Kinesis Data    │ agrupa en archivos
                                                              │ Firehose        │ (buffer por tiempo/tamaño)
                                                              └────────┬────────┘
                                                                       ▼
                                                              ┌─────────────────┐  CRR cifrada
                                                              │ Amazon S3       │──────────────► bucket en otra región
                                                              │ (data lake)     │
                                                              └────────┬────────┘
                                                                       │ CREATE EXTERNAL TABLE (JSON SerDe)
                                                                       ▼
                                                              ┌─────────────────┐
                                                              │ Amazon Athena   │ SQL estándar
                                                              └────────┬────────┘
                                                                       ▼
                                                              ┌─────────────────┐
                                                              │ Amazon          │ equipo de BI ya lo usa
                                                              │ QuickSight      │
                                                              └─────────────────┘
```

| Requisito | Cómo se cumple |
|---|---|
| Endpoint HTTPS RESTful | API Gateway como proxy hacia Firehose; sin cambios en la biblioteca JS |
| Servicios gestionados | Los cinco son serverless o gestionados; sin EC2 ni sistemas operativos |
| Costo por uso granular | Pago por llamadas, registros, GB almacenados, datos escaneados y sesiones |
| Respaldo en otra región | S3 Cross-Region Replication |
| Cifrado en tránsito y en reposo | HTTPS extremo a extremo; cifrado de S3 sin costo adicional; Firehose puede cifrar |
| «Sin escribir código» | Morgan celebra que todo sea serverless y no requiera código propio |

### 7.1 La pregunta del cliente: ¿y si un sitio envía datos mal formados?

Con miles de restaurantes, uno mal configurado podría enviar un formato incorrecto. Respuestas de Raf:
- **Transformación en Firehose:** mecanismos de manipulación de datos (Lambda) durante la entrega.
- **Acciones sobre el bucket (event notifications):** ejecutar una **Lambda por cada `PutObject`**. Como Firehose agrupa los registros en archivos, la Lambda corre **solo cuando llega un archivo nuevo**, no por cada clic.
- **Patrón de dos buckets:** conservar el **dato crudo** en un bucket y escribir el **dato procesado** en un bucket secundario. La misma Lambda puede **enriquecer** (consultar DynamoDB o RDS) y dejar el resultado enriquecido en el segundo bucket.

```
 Firehose ─► S3 raw ──(evento PutObject)──► Lambda: sanea / enriquece (DynamoDB, RDS) ──► S3 processed ─► Athena
```

**Cierre del SA:** pueden surgir requisitos de última hora que obliguen a revisar la arquitectura (no es lo ideal, pero ocurre). El SA puede detenerse en el diseño o seguir con horas de consultoría o una POC; depende del caso. Detenerse en el diseño suele tener sentido cuando el escenario se complica con federación de identidad de terceros, decenas de servicios e ingestas de múltiples fuentes.

---

## 8. Llevar la arquitectura al siguiente nivel (optimización)

Audiencia: 🔧 🧭

Ambos instructores, como SA, revisan también lo que estaba **fuera del alcance**. Idea guía: **repensar la arquitectura es un esfuerzo sin fin**; conviene revisarla periódicamente porque salen servicios nuevos o cambian las condiciones.

| # | Mejora | Detalle | Efecto |
|---|---|---|---|
| 1 | **Sistema de menús serverless** (o en contenedor) | Hoy corre en EC2 todo el día, pero los menús se actualizan poco; el bloque único se descompondría en varios componentes serverless. La lectura recuerda que lo serverless arranca con **Lambda**, integrada nativamente con **más de 200** servicios de AWS y SaaS, con autoescalado, alta disponibilidad incorporada y pago por uso | Menos costo ocioso, menos parches y provisión |
| 2 | **Amazon CloudFront** delante del bucket de menús | CDN con S3 como **origen**; cachea en **edge locations** y enruta a la de menor latencia; transferencia de datos más barata que S3; si el HTML está en caché la solicitud **no llega a S3** (S3 cobra por solicitud y por almacenamiento); certificados SSL de **ACM**, dominios propios y protección DDoS con **AWS WAF** y **AWS Shield** | Más rápido y más barato |
| 3 | **Amazon Cognito en lugar de API Gateway** | Identity pool: la biblioteca JS (con el **AWS SDK for JavaScript**) asume un rol con identidad web (`AssumeRoleWithWebIdentity` de STS) y llama directo a Kinesis. El rol debe tener una política **estricta**, por ejemplo solo `PutRecord` sobre un stream específico. Cognito escala a millones de usuarios y admite login social (Apple, Facebook, Google, Amazon) y empresarial (**SAML 2.0**, **OpenID Connect**) | Se elimina el costo y mantenimiento de API Gateway, **a cambio de refactorizar** la biblioteca JS |
| 4 | **Plantilla de AWS CloudFormation** (IaC) | Toda la solución como plantilla reproducible **en otra cuenta por cliente** (adelanto de la semana 4): mejor gobierno, **facturación separada por cuenta** (cobrar a una cadena de restaurantes por su pila), menos errores manuales | Replicar entornos en cuentas o regiones |
| 5 | **Reintentos con backoff exponencial** en el cliente | Si API Gateway (o Firehose) falla de forma intermitente, no inundarlo con reintentos: reintentar cada vez menos seguido. Definir **retardo máximo** y **número máximo de reintentos** según la operación y factores locales (latencia de red). Los **AWS SDK** ya aplican reintentos exponenciales por defecto, otra ventaja de la opción Cognito + SDK | Resiliencia sin agravar incidentes |
| 6 | **Datos «amigables para Athena»** | Comprimir, particionar y convertir JSON/CSV a **Apache Parquet** (columnar) | Consultas más rápidas y baratas (sección 8.1) |

**Plantilla CloudFormation que muestra la lectura** (bucket con static website hosting, `index.html` y `error.html`, política de lectura pública de objetos, `DeletionPolicy: Retain` para conservar el bucket si se borra la pila —buena práctica con servicios de datos— y salidas con las URL):

```yaml
AWSTemplateFormatVersion: 2010-09-09
Resources:
  S3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      AccessControl: PublicRead
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
    DeletionPolicy: Retain
  BucketPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      PolicyDocument:
        Id: MyPolicy
        Version: 2012-10-17
        Statement:
          - Sid: PublicReadForGetBucketObjects
            Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Join ['', ['arn:aws:s3:::', !Ref S3Bucket, /*]]
      Bucket: !Ref S3Bucket
Outputs:
  WebsiteURL:
    Value: !GetAtt [S3Bucket, WebsiteURL]
    Description: URL for website hosted on S3
  S3BucketSecureURL:
    Value: !Join ['', ['https://', !GetAtt [S3Bucket, DomainName]]]
    Description: Name of S3 bucket to hold website content
```

> [!note] ➕ Nota del editor
> El `AccessControl: PublicRead` de la plantilla usa ACL, que en buckets nuevos está deshabilitado por defecto desde abril de 2023 (*Object Ownership = Bucket owner enforced*); hoy la plantilla fallaría sin ajustar *ObjectOwnership* y el *Block Public Access*. La lógica que enseña (política pública + `DeletionPolicy: Retain` + outputs) sigue vigente. Las funciones intrínsecas se muestran aquí en forma compacta; el material las escribe en forma expandida.

### 8.1 Optimización de datos para Athena (la parte más importante para un ingeniero de datos)

**Athena cobra por cantidad de datos escaneados** (lo que lee para resolver la consulta). Las **tres mejores prácticas**: **comprimir**, **particionar** y **convertir a formato columnar**. Aplicadas, ahorran **entre 30 % y 90 %** del costo por consulta y mejoran el rendimiento. El blog *Top 10 performance tuning tips for Amazon Athena* profundiza.

**1. Compresión.** Archivos más chicos → menos bytes escaneados → consultas más baratas y más rápidas.

**2. Particionamiento.** Restringe lo que escanea cada consulta. Se puede particionar por cualquier clave; lo común es **por tiempo**, a menudo en varios niveles (año / mes / día / hora si los datos llegan cada hora).

> [!example] 📊 El ejemplo de los 5 TB
> Un archivo de **5 TB** con todo un año. Una consulta con `WHERE Month = 'February'` lee **los 5 TB completos** y descarta lo que no es febrero: lenta y cara. Si los datos están **particionados por mes**, la misma consulta solo escanea la partición de febrero.

```
 SIN PARTICIÓN                         CON PARTICIÓN POR MES
 s3://lake/clicks/anio_completo.json   s3://lake/clicks/month=January/...
   └ WHERE Month='February'            s3://lake/clicks/month=February/...  ◄── solo esto se lee
     → escanea 5 TB                    s3://lake/clicks/month=March/...
```

**3. Formatos columnares: Apache Parquet y ORC.** Optimizados para recuperación rápida y usados en las aplicaciones analíticas de AWS:
- **Compresión por columna**, con algoritmo elegido según el tipo de dato: menos espacio en S3, menos disco e I/O al consultar.
- **Predicate pushdown:** Athena lee solo los bloques necesarios usando **estadísticas por bloque** (máximos, mínimos) para decidir si leer u omitir cada bloque.
- **Splitting:** la lectura se reparte entre varios lectores → más **paralelismo**.

> [!note] ➕ Nota del editor — cómo se hace en la práctica (stack S3 + Glue + Athena)
> - **Dónde vive la tabla:** las tablas de Athena se registran en el **AWS Glue Data Catalog**; el módulo no lo menciona, pero es el mismo catálogo que usan Glue, EMR y Lake Formation.
> - **Conversión en la ingesta:** Firehose puede convertir JSON a **Parquet u ORC** usando un esquema del Glue Data Catalog (*record format conversion*), lo que evita un ETL aparte; alternativas: `CREATE TABLE AS SELECT` (CTAS) o `INSERT INTO` en Athena con `format = 'PARQUET'` y `partitioned_by`, o un job de Glue.
> - **Particiones nuevas:** con prefijos estilo Hive (`dt=2026-11-10/`) se registran con `MSCK REPAIR TABLE` o `ALTER TABLE ADD PARTITION`; con prefijos no Hive (como el `YYYY/MM/DD/HH/` por defecto de Firehose) conviene **partition projection**, que calcula las particiones desde propiedades de la tabla sin consultar el catálogo.
> - **Archivos pequeños:** muchos objetos chicos encarecen la apertura de archivos; subir el buffer de Firehose o compactar periódicamente.
> - **Siempre filtrar por la columna de partición** en el `WHERE`; si no, la partición no ahorra nada.

> [!tip] 🎯 Para el examen
> - Bajar costo de Athena → **comprimir + particionar + columnar (Parquet/ORC)**; ahorro 30–90 %.
> - Particionar por la dimensión con la que más se filtra (normalmente tiempo).
> - Predicate pushdown = estadísticas min/max por bloque en Parquet/ORC.

---

## 9. Demos y laboratorios

Audiencia: 🔧

### 9.1 Exploración de Amazon S3 (static website hosting)

**Objetivo:** reproducir lo que ya tiene el cliente: un bucket público que sirve el menú en HTML.

1. **Create bucket** → nombre `raf-restaurant-1`, región **us-east-1**. Los buckets son **regionales**: no se elige AZ (no existe ese nivel para un bucket).
2. **Block Public Access (bucket):** desmarcar *Block all public access* **no hace público el bucket**; solo **permite** que una configuración posterior lo haga público. Hay que reconocer la advertencia.
3. **Upload** de `index.html` (32 bytes) → se convierte en un objeto replicado en el bucket.
4. **Block Public Access a nivel de cuenta:** activado por defecto; aunque el bucket lo permita, hay que **desactivarlo también en la cuenta** (Edit → desmarcar → escribir *confirm*). Sin esto, la política pública no surte efecto.
5. **Properties → Static website hosting → Enable**, documento índice `index.html`.
6. **Permissions → Bucket policy:** política con `Sid` **PublicRead** que permite a todos (`*`) las acciones **`s3:GetObject`** y **`s3:GetObjectVersion`** sobre todos los objetos del bucket.
7. **Properties → URL del website:** cada bucket con hosting estático tiene su URL; por eso los nombres de bucket son **globalmente únicos** (no puede haber dos URL iguales apuntando a lugares distintos). Al abrirla se sirve `index.html` sin escribir la ruta («¡Esto podría ser el menú de un restaurante!»). Ese HTML podría incluir CSS y JS que envíe clickstream al endpoint de API Gateway.

> [!warning] ⚠️ Dos interruptores, no uno
> Block Public Access existe a nivel de **cuenta** y de **bucket**; ambos deben permitirlo y, además, se necesita una **bucket policy** que conceda la lectura. Para el bucket del data lake se hace lo contrario: privado, con todo el acceso público bloqueado.

### 9.2 Exploración de Kinesis Data Firehose

1. Consola → buscar *Kinesis*: aparecen Kinesis, Kinesis Analytics, Kinesis Video Streams. El hub **Kinesis** permite crear Data Streams, Data Firehose y Data Analytics.
2. **Create delivery stream** → **source**: *Direct PUT* (aquí, desde API Gateway, cuya integración se configura en API Gateway) o un **Kinesis Data Stream** previo (se pueden **encadenar**: Firehose detrás de un Data Stream).
3. **Destination:** Amazon S3, Datadog, Amazon Redshift, Amazon OpenSearch Service y muchos más; el más popular es S3.
4. Bucket de destino: crear `raf-kinesis-data-bucket` desde el asistente, **privado** (se mantiene el bloqueo de acceso público).
5. Opciones: **prefijo** de S3 para los objetos; ajustes avanzados como **cifrado del lado del servidor**.
6. **Create delivery stream** → estado *Creating* (unos **2 minutos**) → *Active*.
7. Tras crearlo: monitoreo de CloudWatch en la misma consola, edición de configuración y **transformación de datos con Lambda**.

### 9.3 Exploración de Amazon Athena

1. Athena → **Explore the query editor**: tablas existentes que apuntan a CSV en buckets de la cuenta.
2. Sobre la tabla `year_2018` → menú → **Generate table DDL**: muestra el `CREATE EXTERNAL TABLE` con los campos (`vendorid`, `passenger_count`…), la ubicación (bucket `nyc-tlc-raf`, con prefijos que contienen CSV), el **SerDe de CSV** y `FIELDS TERMINATED BY ','`. Son datos ficticios de taxis de Nueva York para la demo.
3. `SELECT * FROM <tabla> LIMIT 10` → **Run** → Athena consulta los CSV del bucket.
4. Resumen del flujo: crear tablas → apuntarlas a buckets → los archivos deben estar en un formato que Athena entienda (JSON, CSV, **Parquet**) → consultar e incluso unir tablas.

### 9.4 Ejercicio 2 — prueba de concepto analítica (recorrido en 2 partes)

**Objetivo:** demostrar al cliente que los servicios se integran como se dijo (no imitar producción). **Región:** us-east-1. **Flujo:** API Gateway → Firehose → Lambda (formato) → S3 → Athena → QuickSight.

**Parte 1**

**Paso 1 — Rol IAM `API-Firehose`** (rol de ejecución de API Gateway):
- Política *customer managed* `API-Firehose`: permite **`firehose:PutRecord`** sobre `*` (todos los delivery streams; en producción se restringiría al **ARN** del stream).
- Política *AWS managed* para que API Gateway envíe logs a **CloudWatch Logs**.

**Paso 2 — Bucket S3** con nombre globalmente único (ej. `testbucket-morgan-2022-1234`), valores por defecto (**privado**). Copiar su **ARN**.

**Paso 3 — Lambda `transform-data`** desde un **blueprint**: filtrar por *Kinesis* y elegir **«Process records sent to a Kinesis Firehose stream»** en **Python 3.8** (hay uno homónimo en Node.js; elegir el correcto). Motivo: **Athena necesita un registro JSON por línea**, y Firehose (al momento de la grabación) concatenaba todo en una sola línea. El código provisto (Boto3) recorre los registros, **decodifica** cada uno (base64), **añade un salto de línea**, lo **recodifica** y devuelve la lista de registros. **Deploy** → *Configuration → General* → **timeout de 10 s** (el defecto de 3 s difícilmente se agota en una POC, pero da margen ante cargas grandes) → copiar el ARN de la función.

**Paso 4 — Delivery stream de Firehose:** source **Direct PUT**, destination **S3**; nombre generado; **Transform and convert records → Data transformation: Enabled**, ARN de la Lambda, versión/alias **`$LATEST`**; destino = el bucket del paso 2, sin prefijo → **Create** (tarda unos minutos; refrescar hasta *Active*). Copiar el ARN del stream. En *Configuration → Permissions* está el **rol IAM que Firehose creó**: copiar su ARN.

**Paso 5 — Bucket policy** del bucket: `Allow` al **principal AWS = ARN del rol de Firehose** (sin los `< >` de la plantilla y con comillas dobles), las acciones necesarias para subir objetos, y como **Resource** el ARN del bucket **y** `ARN/*` (el bucket y todo su contenido). Existe el **policy generator** y ejemplos como ayuda.

**Paso 6 — API REST `clickstream-ingest-poc`** (REST y no HTTP API, porque se necesita **transformar el payload con mapping templates**), endpoint **Regional**:
1. **Create resource** `poc` (la URL quedará como `…/poc`) y método **POST**.
2. Integración **AWS Service**: región `us-east-1`, servicio **Firehose** (no «Kinesis»), subdominio vacío, método HTTP **POST**, *Use action name* con **Action = `PutRecord`**, **Execution role** = ARN de `API-Firehose`, *content handling* **Passthrough** → Save.
3. **Integration Request → Mapping templates**: *When there are no templates defined*; content type **`application/json`**; pegar la plantilla **VTL** de las instrucciones, que arma los parámetros que espera `PutRecord`: **`DeliveryStreamName`** (pegar el nombre del stream) y **`Record`** con **`Data`** = el cuerpo de entrada formateado como JSON, con reemplazo de caracteres y codificado en **Base64** → **Save** (paso crítico).
4. **Test** con un JSON simulado de clickstream con campos `element_clicked` (plato), `time_spent` (segundos), `source_menu` (menú del restaurante; recordar que el cliente tiene varios restaurantes) y `created_at` (fecha) → respuesta **200** desde Firehose.
5. Repetir **7 u 8 pruebas** para simular navegación. Firehose **agrega** los registros y los escribe en S3 tras un **retardo de buffer** (del orden de 60 s).

**Diagnóstico de códigos de respuesta:**

| Código | Causa probable |
|---|---|
| **400** | El delivery stream aún no está *Active*: esperar y reintentar |
| **500** | Revisar la plantilla **VTL** y la configuración de la integration request |
| Error de permisos | El rol de API Gateway no tiene permiso para publicar en Firehose |

**Parte 2**

**Paso 7 — Verificar S3:** aparece una «carpeta»; el objeto que dejó Firehose está **particionado por fecha** en la ruta, lo que importa para la tabla de Athena.

**Paso 8 — Athena:**
1. **Settings → Manage → Query result location**: prefijo S3 (el mismo bucket + `result`) → Save. **Obligatorio antes de crear la tabla** o ejecutar consultas.
2. Pegar el DDL de las instrucciones: crea la tabla **`my_ingested_data`** con columnas `element_clicked`, `time_spent`, `source_menu`, `created_at` (la misma estructura que envía el frontend); **`PARTITIONED BY (datehour string)`**; **SerDe JSON** con propiedades que mapean cada campo; y **propiedades de tabla** que indican cómo están particionados los datos por fecha y en qué formato, para que Athena encuentre los archivos.
3. Reemplazar el nombre del bucket en **dos lugares** (`LOCATION 's3://<bucket>/'` y la plantilla de ubicación de la línea 20) → **Run**.
4. `SELECT * FROM my_ingested_data;` → 7 filas. Consejo: con cientos de miles de filas, usar **`LIMIT`** o filtros para no lanzar consultas gigantes que cuestan dinero y tiempo.

```sql
-- Esqueleto reconstruido a partir de lo que se describe en el video (no es el DDL literal)
CREATE EXTERNAL TABLE my_ingested_data (
  element_clicked STRING,
  time_spent      INT,
  source_menu     STRING,
  created_at      STRING
)
PARTITIONED BY (datehour STRING)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://<bucket>/'
TBLPROPERTIES (
  -- propiedades de proyección de particiones por fecha/hora
  -- y plantilla de ubicación 's3://<bucket>/${datehour}/'
);
```

> [!note] ➕ Nota del editor
> El bloque anterior es una **reconstrucción orientativa**: el video no muestra el texto completo del DDL; los nombres de columna, el tipo de `time_spent`, la clase del SerDe y las propiedades exactas están en las instrucciones del Ejercicio 2 (enlace en Recursos). Por la descripción («propiedades de tabla que dicen cómo está particionado por fechas en un formato específico» y un segundo reemplazo del bucket en una plantilla de ubicación), se trata de **partition projection** sobre los prefijos `YYYY/MM/DD/HH` que Firehose escribe por defecto.

**Paso 9 — QuickSight:**
1. Crear cuenta de QuickSight si es la primera vez.
2. **Manage QuickSight → Security & permissions → QuickSight access to AWS services → Manage**: marcar **Amazon S3** y **el bucket específico**, y la casilla **Write permission for Athena Workgroup** → Finish → Save.
3. **Datasets → New dataset → Athena** → nombre de fuente `POC Clickstream` → *Create data source* → elegir la tabla `my_ingested_data` → **Import to SPICE for quicker analytics** → **Visualize**.
4. Elegir un tipo de visual (según los atributos y el insight buscado): **gráfico circular** por `element_clicked` → porciones iguales (un registro por plato).

**Limpieza (en orden inverso):** QuickSight (borrar dataset y análisis) → Athena (`DROP TABLE my_ingested_data;`) → API Gateway → delivery stream de Firehose (escribir el nombre) → Lambda → **vaciar** el bucket (*permanently delete*) y luego borrarlo → opcionalmente los roles IAM. Si algún recurso no se borra, abrir un ticket de soporte para no seguir pagando.

**Troubleshooting por tramos (casi siempre son permisos):**

| Tramo | Qué revisar |
|---|---|
| API Gateway → Firehose | Permisos del rol de ejecución; que el stream exista y esté *Active* |
| Firehose → Lambda | Que la función exista y que la **transformación de datos esté habilitada** en el stream |
| Firehose → S3 | La **bucket policy** debe permitir al rol de Firehose |
| Athena | **Query result location** configurada antes de consultar |
| QuickSight | Permiso de QuickSight sobre **el bucket específico** (y Athena workgroup) |

> [!danger] 🚨 Errores comunes en esta POC
> - Elegir el blueprint de Node.js en vez del de Python 3.8, o seleccionar «Kinesis» en lugar de «Firehose» en la integración de API Gateway.
> - Olvidar **Save** en la mapping template.
> - Dejar los `< >` o quitar las comillas al pegar el ARN en la bucket policy.
> - No configurar la ubicación de resultados de Athena.
> - Dar a QuickSight acceso a S3 pero no al bucket concreto.
> - Lanzar `SELECT *` sin `LIMIT` sobre tablas grandes.

El **Ejercicio 2** («Construyendo una prueba de concepto para el análisis de datos») tiene instrucciones completas en un HTML externo (enlace en Recursos).

---

## 10. Aplicación de ejemplo: estado al cierre del módulo

Audiencia: 🔧 🧭

El producto de menús por QR suma una **capa analítica de cinco bloques serverless**, más las mejoras propuestas para la parte existente:

| Bloque | Función | Requisito que cubre |
|---|---|---|
| Amazon API Gateway (REST) | Endpoint HTTPS, proxy `PutRecord` a Firehose, mapping VTL | Endpoint RESTful sin cambiar la biblioteca JS |
| Kinesis Data Firehose (+ Lambda de transformación en la POC) | Buffer, agregación en archivos, entrega a S3 | Gestionado, sin código, latencia de minutos aceptada |
| Amazon S3 (+ CRR) | Data lake desacoplado del cómputo, cifrado, respaldo en otra región | Durabilidad, cifrado, costo por uso |
| Amazon Athena | SQL sobre JSON en S3 con `CREATE EXTERNAL TABLE` | Consulta sin infraestructura ni ETL |
| Amazon QuickSight | Dashboards para los restaurantes | Herramienta que el equipo de BI ya usa |

```
 Comensal ─QR─► [CloudFront]* ─► S3 (menús) ── JS clickstream ──► API Gateway ─► Firehose ─► S3 raw ─┬─► Athena ─► QuickSight
                                                         (alt.* Cognito + SDK directo a Firehose)   │
 Admin ─► sistema de menús [serverless]*                                     Lambda por PutObject*  └─► S3 processed*
 * = mejoras propuestas en la optimización; + CloudFormation por cliente, backoff exponencial, Parquet + particiones
```

---

## ✅ Autoevaluación

> [!question]- 1. ¿Por qué Raf descarta EBS y EFS si EFS cumple el requisito de durabilidad?
> Porque ambos deben estar adjuntos a un recurso de cómputo (EBS a una EC2; EFS a EC2, contenedores o servidores), y una arquitectura analítica necesita una capa de almacenamiento que exista por sí sola y sea accesible por API desde cualquier herramienta. Además, EBS factura por capacidad aprovisionada y su durabilidad es menor que la de S3.

> [!question]- 2. ¿Qué dato del cliente decidió entre Kinesis Data Firehose y Kinesis Data Streams?
> La latencia tolerada: el análisis es asíncrono, sin dashboards en tiempo real, y unos minutos de espera son aceptables. Con eso gana la conveniencia de Firehose (sin código de productor y consumidor) frente al control y la latencia bajo un segundo de Data Streams.

> [!question]- 3. ¿Para qué sirve API Gateway en esta arquitectura si no es un servicio de ingesta?
> Como proxy: recibe el HTTPS POST que la biblioteca JS ya sabe hacer y lo traduce a una llamada `PutRecord` a Firehose mediante una integración de servicio y una mapping template. Así no se expone el delivery stream a Internet, se puede usar dominio y certificado propios y no hay que tocar el código del cliente.

> [!question]- 4. ¿Por qué S3 Select no sirve para consultar este clickstream?
> Porque consulta un solo objeto por llamada, y Firehose produce muchos archivos pequeños a lo largo del tiempo. Athena, en cambio, consulta todos los objetos bajo un prefijo como si fueran una tabla.

> [!question]- 5. ¿Qué significa EXTERNAL en `CREATE EXTERNAL TABLE` de Athena y qué implica?
> Que la tabla es solo metadatos (esquema, SerDe y ubicación en S3): Athena no copia ni mueve los datos. Por eso S3 sigue siendo el repositorio central, otras herramientas pueden leer los mismos archivos y el esquema tiene que calzar con su formato.

> [!question]- 6. ¿Por qué la POC necesita una Lambda que añade saltos de línea?
> Porque el SerDe JSON de Athena interpreta un registro por línea y Firehose, al momento de la grabación, concatenaba los registros en una sola línea. Sin la transformación, Athena leería mal los eventos.

> [!question]- 7. Una consulta con `WHERE Month = 'February'` sobre un archivo anual de 5 TB es lenta y cara. ¿Qué tres medidas aplicaría y por qué funcionan?
> Particionar por mes, para que Athena solo lea la partición de febrero; convertir a Parquet u ORC, para leer solo las columnas necesarias y saltar bloques por estadísticas min/max (predicate pushdown) con lecturas en paralelo; y comprimir, para escanear menos bytes. Athena cobra por datos escaneados, así que las tres medidas bajan el costo (30–90 %) y aceleran la consulta.

> [!question]- 8. ¿Cómo se protege la solución de un restaurante que envía datos mal formados sin perder el dato original?
> Con dos buckets: Firehose deja el dato crudo en un bucket raw; un evento `PutObject` dispara una Lambda que sanea (y opcionalmente enriquece con DynamoDB o RDS) y escribe en un bucket processed, que es el que consulta Athena. Como Firehose agrupa en archivos, la Lambda solo corre al llegar cada archivo.

> [!question]- 9. ¿Qué se gana y qué se pierde al reemplazar API Gateway por Cognito?
> Se gana menos costo y un servicio menos que mantener, además de los reintentos exponenciales del AWS SDK. Se pierde la ventaja de no tocar el cliente: hay que refactorizar la biblioteca JS para que asuma un rol con identidad web, que debe tener una política mínima (solo `PutRecord` sobre ese stream).

> [!question]- 10. ¿Por qué QuickSight y no Managed Grafana si ambos servían?
> Porque el cliente ya usa QuickSight en otros proyectos y su equipo de BI lo conoce: se evita curva de aprendizaje y posiblemente costo de licencias, y el costo es un requisito explícito. A veces el mejor servicio es el que el equipo ya sabe usar.

---

## 🧾 Resumen en 10 líneas

1. Software House sirve menús HTML desde S3 vía código QR y quiere analítica de clickstream para los restaurantes; la biblioteca JS y los pagos quedan fuera del alcance.
2. Requisitos: endpoint HTTPS RESTful, servicios gestionados, costo por uso granular, respaldo en otra región, cifrado en tránsito y en reposo.
3. Principios: pensar primero en cómo llegan los datos, usar la herramienta adecuada y separar almacenamiento de cómputo.
4. Almacenamiento: S3 (11 nueves, pago por uso, CRR, cifrado sin costo) frente a EBS y EFS, que dependen de un recurso de cómputo.
5. Ingesta: Kinesis Data Firehose, porque se toleran minutos de latencia y no hay que escribir código; Data Streams queda para el subsegundo.
6. API Gateway hace de proxy HTTPS → `PutRecord` sin cambiar la biblioteca JS; Cognito + SDK es la alternativa con refactorización.
7. Consulta: Athena con `CREATE EXTERNAL TABLE` y SerDe JSON; S3 Select (un objeto), Glue y EMR (curva y costo) descartados.
8. Visualización: QuickSight porque el equipo de BI ya lo usa; CloudWatch es operativo y OpenSearch acopla almacenamiento y cómputo.
9. Optimizaciones: menús serverless, CloudFront, Cognito, CloudFormation por cliente, backoff exponencial, bucket raw y processed con Lambda.
10. Athena cobra por bytes escaneados: comprimir, particionar (por tiempo) y pasar a Parquet/ORC ahorra entre 30 % y 90 %.

---

## 📖 Recursos del módulo

**Panorama de servicios de datos**
- Data Lakes and Analytics on AWS · https://aws.amazon.com/big-data/datalakes-and-analytics/
- Amazon S3 · https://aws.amazon.com/s3/
- Clases de almacenamiento Amazon S3 Glacier · https://aws.amazon.com/s3/storage-classes/glacier/
- AWS Lake Formation · https://aws.amazon.com/lake-formation/
- Amazon Athena · https://aws.amazon.com/athena/
- Amazon EMR · https://aws.amazon.com/emr/
- Amazon OpenSearch Service · https://aws.amazon.com/opensearch-service/
- Amazon Kinesis · https://aws.amazon.com/kinesis/
- AWS Glue · https://aws.amazon.com/glue/
- AWS Database Migration Service · https://aws.amazon.com/dms/
- Amazon SageMaker · https://aws.amazon.com/sagemaker/
- Amazon Rekognition · https://aws.amazon.com/rekognition/
- Amazon Comprehend · https://aws.amazon.com/comprehend/
- Amazon Comprehend Medical · https://aws.amazon.com/comprehend/medical/

**S3: replicación y ciclo de vida**
- Replicación de objetos (Guía del usuario de S3) · https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html
- S3 Intelligent-Tiering · https://aws.amazon.com/s3/storage-classes/intelligent-tiering/

**Kinesis**
- Amazon Kinesis Data Streams · https://aws.amazon.com/kinesis/data-streams/
- Amazon Kinesis Data Firehose · https://aws.amazon.com/kinesis/data-firehose/
- Amazon Kinesis Data Analytics · https://aws.amazon.com/kinesis/data-analytics/
- Amazon Kinesis Video Streams · https://aws.amazon.com/kinesis/video-streams/

**Optimizaciones**
- Serverless en AWS · https://aws.amazon.com/serverless/
- Amazon CloudFront · https://aws.amazon.com/cloudfront/
- Amazon Cognito · https://aws.amazon.com/cognito/
- AssumeRoleWithWebIdentity (STS API Reference) · https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html
- Fragmentos de plantillas de CloudFormation · https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/CHAP_TemplateQuickRef.html
- AWS CloudFormation · https://aws.amazon.com/cloudformation/
- Reintentos de error y backoff exponencial en AWS · https://docs.aws.amazon.com/general/latest/gr/api-retries.html
- Top 10 performance tuning tips for Amazon Athena · https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/
- Particionar datos en Athena · https://docs.aws.amazon.com/athena/latest/ug/partitions.html
- Conversión a formatos columnares · https://docs.aws.amazon.com/athena/latest/ug/convert-to-columnar.html

**Ejercicio**
- Ejercicio 2: prueba de concepto para el análisis de datos · https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/DEV-AWS-MO-Architecting/exercise-2-analytics.html

---

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[05-Backend-Web-Serverless|05 · Backend Web Serverless]] · Siguiente: [[07-Solucion-Hibrida-Contenedores|07 · Solución Híbrida con Contenedores ➡]]
