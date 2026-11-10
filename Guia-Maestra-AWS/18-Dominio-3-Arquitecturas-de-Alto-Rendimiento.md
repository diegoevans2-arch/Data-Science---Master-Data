---
title: "Tomo 18 — Dominio 3: Design High-Performing Architectures (24 %)"
tags: [aws, solutions-architect, saa-c03, rendimiento, almacenamiento, computo, bases-de-datos, redes, ingesta]
curso: 4 · Exam Prep: AWS Certified Solutions Architect – Associate
modulo: "Dominio 3 · Diseñar arquitecturas de alto rendimiento"
audiencias: [tecnico, puente, ejecutivo]
tomo: 18
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# ⚡ Tomo 18 — Dominio 3: Design High-Performing Architectures (24 %)

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[17-Dominio-2-Arquitecturas-Resilientes|17 · Dominio 2: Arquitecturas Resilientes]] · Siguiente: [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo|19 · Dominio 4: Costo ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> El dominio 3 pesa **24 %** y es el más amplio: **cinco task statements** que recorren las capas de almacenamiento, cómputo, base de datos y red, más la sección nueva del C03 sobre **ingesta y transformación de datos**. No hay una solución óptima universal: las cargas bien diseñadas combinan servicios, activan características específicas de rendimiento y deciden con datos (benchmarking y pruebas de carga), no con los valores por defecto de la organización. Para un ingeniero de datos, la quinta tarea conecta directamente con los tomos 09–14.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** tipo de almacenamiento y de base de datos por patrón de acceso, modelo de cómputo (instancias, contenedores, funciones), topología de red híbrida y multicuenta, servicio de ingesta según volumen y frecuencia.
> - **Costo de hacerlo mal:** aplicaciones lentas en horas punta, trabajos que se pierden, conexiones entre VPC que no escalan, data lakes con más carga operativa de la necesaria.
> - **Pregunta que responde:** *dado este patrón de acceso, volumen y latencia, ¿qué servicio y qué configuración rinden mejor con el menor esfuerzo?*

> [!tip] 💡 Analogía general: la cocina profesional
> Almacenamiento es la despensa (estantes de objetos, cajones de bloque, armario compartido de archivos); cómputo, los cocineros (de planta, por turno o llamados al momento); la base de datos, el libro de recetas indexado; la red, los pasillos; la ingesta, el andén de descarga de proveedores. Una cocina rápida no es la que tiene el mejor horno, sino la que dimensiona cada parte para el plato que más se pide.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Dominio 3 · Introducción | 1 video | Las capas y los cinco task statements |
| 3.1 Almacenamiento de alto rendimiento y escalable | 1 video + pregunta 3.1 | Objeto/bloque/archivo, escalado de EBS vs EFS, S3, FSx, lifecycle, multipart, Transfer Acceleration |
| 3.2 Cómputo elástico y de alto rendimiento | 1 video + pregunta 3.2 | Instancias, contenedores, funciones; métricas de escalado; familias de instancia |
| 3.3 Bases de datos de alto rendimiento | 1 video + pregunta 3.3 | Motores con propósito, Aurora vs RDS, DynamoDB, caché, RDS Proxy, Aurora Serverless |
| 3.4 Redes de alto rendimiento y escalables | 1 video + pregunta 3.4 | VPC, híbrido, Transit Gateway, PrivateLink, Route 53, Global Accelerator, CloudFront, transferencia de datos, dos ejercicios de arquitectura |
| 3.5 Ingesta y transformación de alto rendimiento | 1 video | Patrones homogéneos/heterogéneos, Kinesis, MSK, Glue, EMR, Lake Formation, seguridad del data lake |
| Recapitulación y recursos | 1 video · 1 lectura | Lista de repaso y 17 enlaces |

> [!note] ➕ Nota del editor
> El material menciona un laboratorio práctico, «configurar una distribución de Amazon CloudFront con un origen Amazon S3», sin texto en lo extraído.

---

## 1. Task Statement 3.1 — Determine high-performing and/or scalable storage solutions

Audiencia: 🔧 🧭

La solución óptima depende de: **método de acceso, patrón de acceso, throughput requerido, frecuencia de acceso, frecuencia de actualización, disponibilidad y durabilidad**. Hay tres formas de almacenamiento: **objeto, bloque y archivo**.

### 1.1 Cómo escala cada servicio

| | Amazon EBS | Amazon EFS |
|---|---|---|
| Escalado | **Manual**: se modifica tipo de volumen, tamaño e IOPS | **Automático** al agregar o borrar archivos |
| ¿En producción? | Sí, admite cambios de configuración en vivo | No requiere acción |
| Pista de examen | — | «Menor soporte operativo para escalar el almacenamiento» → EFS |

Hay que conocer también los **límites superiores de capacidad** de cada servicio para elegir según el crecimiento futuro (ejemplo del curso: 3 TB hoy, 100 TB en cinco años).

### 1.2 Servicio por servicio

| Servicio | Palabras clave y claves del curso |
|---|---|
| **Amazon S3** (objeto) | Storage classes, llamadas básicas de API, multipart upload; ideal para big data y backup. Es un servicio de resiliencia **global** que corre en cada Region: los datos viven en una Region, toleran la caída de una AZ y pueden replicarse entre Regions |
| **Amazon EBS** (bloque) | *DAS, SAN, almacenamiento persistente para EC2*. Latencia extremadamente baja y rendimiento configurable según tipo de volumen e IOPS. **Snapshots** guardadas en S3 → resiliencia regional |
| **Instance store** | Almacenamiento local **efímero** de la instancia |
| **Amazon EFS** (archivo) | *Sistema de archivos de red, NFS, acceso híbrido (VPN/Direct Connect), instancias Linux*. Repositorios grandes, entornos de desarrollo, multimedia, directorios. Modos General Purpose y Max I/O; lifecycle policies para costo |
| **Amazon FSx** | FSx for Windows File Server: compartido totalmente administrado, **SMB, NTFS, Active Directory**, HA y escalable. **FSx for Lustre**: para Linux. Ambos útiles en híbrido |
| **AWS Storage Gateway** | Muy bueno para almacenamiento híbrido |

**Rendimiento en S3:** para subir datos, conocer la API y los comandos de la AWS CLI y el **multipart upload**; para subidas más rápidas, **S3 Transfer Acceleration**; para lecturas más rápidas, **caché con CloudFront**.

---

## 2. Task Statement 3.2 — Design high-performing and elastic compute solutions

Audiencia: 🔧 🧭

Estar en AWS no hace a una solución escalable por sí sola: depende del diseño. El cómputo viene en tres formas:

| Forma | Servicio | Lo que destaca el curso |
|---|---|---|
| **Instancias** | Amazon EC2 | Servidores virtuales en familias y tamaños (SSD, GPU…); cada tipo trae CPU, memoria, almacenamiento local, ancho de banda de red y capacidades distintas. EC2 **no es escalable por naturaleza** |
| **Contenedores** | Amazon ECS, Amazon EKS | ECS recibe contenedores con instrucciones de dónde y cómo correrlos; EKS ejecuta Kubernetes sobre EC2. ECS tiene dos modos: **EC2** (control del entorno) y **Fargate** (serverless). Integración con ALB para port mapping |
| **Funciones** | AWS Lambda | Fragmento de código en un runtime (Python, Java, Node.js…) sin instancia; se factura por duración, se invoca por eventos, **límite de 15 minutos** (más largo → **Step Functions**). Con CloudFront se despliega en edge locations para más capacidad de respuesta |

### 2.1 Métricas y escalado

- Desacoplar con **SQS** y **ELB** para que cada componente escale por separado.
- Visibilidad: métricas, alarmas y dashboards de **CloudWatch**; las alarmas se disparan cuando una métrica cruza un umbral durante un período definido y pueden accionar EC2 Auto Scaling, alertas y remediación.
- Saber **qué métricas no vienen por defecto**: la **memoria usada** en EC2 no es métrica estándar.
- Otras métricas de escalado: **custom metrics** y métricas del balanceador como **HealthyHostCount** y **SurgeQueueLength**.
- Lambda es elástica por naturaleza (1 o 100 invocaciones por segundo sin diseñar el escalado); EC2 no.

> [!tip] 💡 Elegir familia de instancia
> No hace falta memorizar todos los tipos. El enunciado dará las necesidades de cómputo, almacenamiento y red; usted debe saber qué familia calza y cómo impacta en rendimiento y escalabilidad (ver la página de tipos de instancia de EC2).

> [!example] 📊 Servicio web con uso muy variable durante el día, que debe ser HA y elástico
> EC2 + **Elastic Load Balancing** + **EC2 Auto Scaling**.

---

## 3. Task Statement 3.3 — Determine high-performing database solutions

Audiencia: 🔧 🧭

AWS ofrece motores con propósito específico: **relacional, key-value, documentos, in-memory, grafos, series de tiempo y ledger**, administrados (sin aprovisionar, parchar, instalar, respaldar ni recuperar a mano), con almacenamiento self-healing y escalado automático. La elección depende de **disponibilidad, consistencia, tolerancia a particiones, latencia, durabilidad, escalabilidad y capacidad de consulta**, y muchas veces conviene una base distinta por subsistema.

### 3.1 RDS vs Aurora

| | Amazon RDS | Amazon Aurora |
|---|---|---|
| Arquitectura | Instancia con almacenamiento local | **Clúster**: una primaria + cero o más réplicas |
| Réplicas | Read Replicas (lecturas); Multi-AZ aparte (HA) | Las réplicas dan **lecturas y beneficio Multi-AZ** a la vez |
| Almacenamiento | Local de la instancia | **Volumen de clúster compartido**: aprovisionamiento más rápido, más disponibilidad y rendimiento |
| Alcance | **Regional** (tolera la pérdida de una AZ) | **Global Database** que abarca varias Regions |

### 3.2 DynamoDB y caché

- **DynamoDB:** milisegundos de un dígito consistentes sin importar la carga ni ajustes; replicado en varios nodos por defecto, sobre SSD; backups, point-in-time recovery, cifrado en reposo e integración por eventos ante cambios en la tabla. Para «milisegundos de un dígito y volúmenes extremadamente altos» gana a Aurora.
- **ElastiCache:** caché in-memory administrada con dos motores, **Redis** y **Memcached**.
- **DAX:** acelerador de DynamoDB con dos cachés, **item cache** y **query cache**.
- Estrategias de caché: **lazy loading** y **write-through**.
- **RDS Proxy:** pool de conexiones establecidas; alivia CPU y memoria cuando, por ejemplo, Lambda abre muchas conexiones.

### 3.3 Capacidad y autoescalado

- Elegir tipo y tamaño de instancia.
- **RDS storage autoscaling** para cargas impredecibles; no se activa si el umbral de almacenamiento máximo es igual o menor que el incremento.
- **Aurora Serverless:** autoescalado bajo demanda, ideal para cargas infrecuentes, intermitentes o impredecibles. Mismo almacenamiento de clúster compartido (**seis copias en tres AZ**); se configura con **ACU** (Aurora Capacity Units) mínima y máxima; puede bajar a **cero ACU y pausarse** tras inactividad, pagando solo almacenamiento.

---

## 4. Task Statement 3.4 — Determine high-performing and/or scalable network architectures

Audiencia: 🔧 🧭

No se exige ser experto en redes, pero sí saber lo suficiente para decidir y diagnosticar. La red está entre todos los componentes y puede ayudar o arruinar el rendimiento.

| Tipo de optimización | Ejemplos que da el curso |
|---|---|
| Características de producto | Enhanced networking, instancias EBS-optimized, S3 Transfer Acceleration, CloudFront |
| Características de red | Route 53 latency routing, VPC endpoints, Direct Connect, Global Accelerator |
| Ubicación | Recursos cerca de los usuarios; Regions, placement groups y edge |
| Operación | Métricas de red para ajustar; red como **infraestructura como código** |

### 4.1 Construir la VPC

La VPC es **regional** y una custom VPC es **privada** salvo configuración explícita. Orden de creación que sugiere la instructora:

```
 VPC → subnets → route tables → internet gateway → network ACL → security groups
     → recursos → ajustar SG → NAT gateway (si aplica) → peering → endpoints
```

### 4.2 Conectividad

| Necesidad | Opción |
|---|---|
| On-premises ↔ AWS privado | **Site-to-Site VPN** y/o **Direct Connect** (comparar rendimiento y throughput según volumen, cumplimiento y requisitos) |
| Muchas VPC ↔ red remota | **Transit Gateway** (con VPN o Direct Connect); simplifica el peering |
| Hub and spoke de redes | **AWS VPN CloudHub** (a alto nivel) |
| VPC ↔ VPC (otra cuenta) | VPC peering, Transit Gateway o Direct Connect gateway |
| Exponer un servicio a otras VPC sin Internet ni peering | **PrivateLink** |
| Subnet privada sin NAT → S3/DynamoDB | **Gateway endpoints** |
| Enviar al usuario a la Region más cercana geográficamente | **Route 53 geoproximity routing** |
| Mejorar rendimiento de red global | **Global Accelerator** |
| Acercar contenido al usuario | **CloudFront** en edge locations |

**Transferencia y migración de datos:** AWS DataSync, AWS Snow Family, AWS Transfer Family, AWS Database Migration Service (DMS) y otros; la elección depende de cantidad y tipo de datos, origen y destino.

**Balanceo y escalado:** EC2 Auto Scaling con launch configurations; capacidad **mínima, máxima y deseada**; scaling policies; integrar Auto Scaling con el balanceador para health checks más ricos. Consejo: saber en qué **capa del modelo OSI** opera cada balanceador.

### 4.3 Dos ejercicios de arquitectura del curso

> [!example] 📊 Ejercicio 1: mejorar la escalabilidad de una arquitectura con requisitos de tiempo real
> Contexto: comunicaciones en tiempo real con niveles de servicio de **99,9 % a 99,999 %**. Directrices de AWS: sin punto único de falla; monitoreo, detección de fallas y failover automáticos para componentes con y sin estado. Herramientas: clúster de balanceadores en par activo-standby; Route 53 como DNS público o privado con health checks; Auto Scaling con lifecycle hooks, eventos de CloudWatch y Lambda.
> Mejoras sugeridas: **agregar CloudFront**, **agregar ElastiCache** y afinar las scaling policies usando **instancias más pequeñas** que escalen en incrementos más granulares.

> [!example] 📊 Ejercicio 2: solución serverless global de muy alto volumen y bajo costo
> Propuesta base: **S3 static website hosting** para contenido estático; **Route 53** con latency y failover routing; **DynamoDB global tables** para baja latencia y replicación entre Regions.
> Mejoras: **CloudFront** (rendimiento, confiabilidad, seguridad y caché) y considerar **Global Accelerator**, que según el curso tiene una función para **S3 Multi-Region Access Points** (endpoints que abarcan buckets en varias Regions).

---

## 5. Task Statement 3.5 — Determine high-performing data ingestion and transformation solutions

Audiencia: 🔧 🧭 👔

**Data ingestion** = obtener, recolectar, curar y preparar datos desde el origen hacia AWS (almacenamiento, data lakes, ML…), cumpliendo rendimiento, latencia, escala, seguridad y gobernanza.

| Patrón | Qué implica | Foco |
|---|---|---|
| **Homogéneo** | Mismo formato o mismo motor que el origen | Velocidad de transferencia, integridad, automatizar la ingesta continua (el curso cita servicios como Athena o EMR) |
| **Heterogéneo** | Se transforma al ingerir | Cambiar tipo/formato para el destino o derivar atributos con ML |

### 5.1 Streaming

| Servicio | Rol |
|---|---|
| **Kinesis Data Streams** | Ingesta de streaming en tiempo real, alta escalabilidad y durabilidad, varios GB por segundo desde múltiples fuentes; los productores escriben **records** que quedan disponibles para procesar |
| **Kinesis Data Firehose** | La forma más simple de capturar, transformar y **cargar** streams en almacenes de AWS |
| **Kinesis Data Analytics** | La opción de Kinesis con transformaciones básicas |
| **Kinesis Video Streams** | Ingesta de video en streaming |
| **Amazon MSK** | Apache Kafka administrado para streaming en tiempo real |

### 5.2 Transformación, catálogo y análisis

- **AWS Glue:** integración de datos para descubrir, preparar, mover e integrar datos de varias fuentes.
- **Amazon EMR** y Glue sobre S3: procesamiento distribuido masivo; S3 escala horizontalmente. Ajustes: número de peticiones concurrentes a S3, estrategia de reintentos, número de objetos procesados. Transformar a **Parquet** con EMR o mover datos con Lambda hacia el data lake.
- Análisis y visualización: **Athena, Lake Formation, QuickSight**.
- **Data lake:** repositorio central de datos estructurados y no estructurados de múltiples fuentes: ingesta rápida, elimina duplicación y dispersión, gobierno centralizado.
- Ingesta a un data lake con Lake Formation sobre S3: **Kinesis Data Firehose, Snow Family, Glue, DataSync, Transfer Family, Storage Gateway, Direct Connect, DMS**, según frecuencia y cambios de los datos.

### 5.3 Seguridad de la ingesta en entornos híbridos

- **Bucket policies** para dar acceso al lago a varias cuentas y usuarios; **user policies** de IAM para atar permisos a roles y herramientas de análisis.
- S3 Cross-Region Replication, **Object Lock**, versioning.
- Cifrado con las opciones de S3 y **KMS**; para más protección, **API Gateway** y **Cognito**; para cumplimiento con PII, **CloudHSM**.

---

## 6. Preguntas de práctica resueltas en el curso

Audiencia: 🔧 🧭

### Pregunta 3.1 — Subir muchos videos de 100–500 MB, reanudando fallas, en el menor tiempo

Palabras clave: *S3*, el tamaño concreto (*100–500 MB*: si dan números, importan), *reanudar fácilmente*, *menor tiempo*.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| Partir cada archivo en trozos de 5 MB, subirlos y unirlos con multipart | ❌ | Innecesario y más lento; partir antes de subir no reensambla el objeto original |
| **`aws s3 cp` con la CLI** | ✅ | Desde ~100 MB conviene multipart (mejor throughput y reinicio rápido ante fallas de red), y los comandos `aws s3` lo hacen **automáticamente** según el tamaño |
| Arrastrar y soltar en la consola | ❌ | Sin protección ante problemas de red ni multipart automático |
| SFTP con Transfer Family | ❌ | No resuelve la reanudación y exige más trabajo |

### Pregunta 3.2 — Procesamiento de fotos y videos lento en horas punta

**Escenario:** web y procesamiento en EC2 On-Demand, cada capa con su Auto Scaling group detrás de un ALB; trabajos de **hasta una hora**; en horas punta la app es lenta y pierde solicitudes; de noche está ociosa. Se pide procesar **todos** los trabajos de la forma **más costo-efectiva**.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| Instancias más grandes en ambas capas | ❌ | Sube el costo (también en horas ociosas) y no garantiza procesar todo |
| Spot Instances en ambas capas | ❌ | Más barato pero no más escalable; arrancan igual de lento; puede no haber capacidad Spot y se siguen perdiendo solicitudes |
| **Cola SQS standard entre web y procesamiento + métrica custom de la cola para escalar el ASG de procesamiento** | ✅ | La profundidad de la cola indica atraso y dispara el escalado; si el ASG llega al máximo, los trabajos esperan en la cola en vez de perderse |
| Lambda con más cuota de concurrencia | ❌ | Lambda tiene máximo de **15 minutos**; los trabajos duran hasta una hora |

### Pregunta 3.3 — IoT de turbinas eólicas a key-value y data lake (elija dos)

**Escenario:** datos de sensores con tráfico en picos (*spiky*), que deben guardarse en una base **key-value** y luego enviarse a un **data lake** central para transformar, analizar y combinar, con **mínima carga operativa**.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| **Kinesis entrega el streaming a un data lake en S3** | ✅ | S3 es el lugar habitual del data lake y Kinesis lo resuelve con la menor carga operativa |
| DocumentDB para los datos | ❌ | No es key-value |
| Funciones Lambda que entregan a S3 | ❌ | Funcionaría, pero exige código propio: más carga operativa |
| **DynamoDB para los datos + Kinesis Data Streams activado** | ✅ | DynamoDB es key-value, escala solo ante picos; Kinesis Data Streams captura cambios a nivel de ítem para replicarlos al lago |
| Kinesis a Redshift + Redshift Spectrum | ❌ | S3 es mejor data lake para combinar muchas fuentes con menos operación |

### Pregunta 3.4 — Más de 50 cuentas con 4+ VPC cada una hacia una VPC de servicios compartidos

**Escenario:** AWS Organizations con cuenta de gestión y más de 50 cuentas por país, cada una con al menos cuatro VPC; alta confianza; todas deben alcanzar una VPC de servicios compartidos con la **menor carga operativa**.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| ALB + registro **CAA** en Route 53 referenciado en route tables | ❌ | Un registro CAA solo indica qué autoridades pueden emitir certificados para un dominio |
| Peering de cada VPC con la compartida | ❌ | El distractor más tentador: una VPC acepta hasta **125** conexiones de peering y aquí se necesitan más de 200 |
| **NLB multi-AZ en la VPC compartida + rol de consumidor en IAM + aceptación automática del endpoint + interface endpoints en cada VPC apuntando al NLB** | ✅ | Es **PrivateLink**: acceso direccional de muchas VPC consumidoras a un servicio de la VPC proveedora |
| VPN entre cada VPC y la compartida | ❌ | Funciona, pero hay que operar y mantener todas las VPN |

> [!tip] 💡 Pasos de PrivateLink que deja la pregunta
> 1) NLB en la VPC proveedora; 2) rol de service consumer en IAM; 3) endpoint service con aceptación automática; 4) endpoints de consumidor en cada VPC apuntando al NLB.

---

## 7. Trampas típicas del dominio

> [!danger] 🚨 Errores costosos
> - Proponer **Lambda** para trabajos de más de 15 minutos.
> - Creer que **Spot** resuelve un problema de escalado (resuelve costo, no capacidad).
> - Olvidar que la **memoria** de EC2 no es métrica por defecto en CloudWatch.
> - Elegir **EBS** cuando piden escalado sin operación (va EFS) o **instance store** cuando piden persistencia.
> - Peering para cientos de VPC (límite de 125); la respuesta suele ser PrivateLink o Transit Gateway.
> - Confundir **Data Streams** (ingesta en tiempo real) con **Firehose** (carga a destinos) y con **Data Analytics** (transformación).
> - Escribir código Lambda cuando un servicio administrado hace lo mismo con menos operación.
> - Confundir DocumentDB (documentos) con DynamoDB (key-value).

---

## 8. Lista de repaso del cierre del dominio

- Objeto vs bloque vs archivo; S3, EFS, FSx, EBS.
- Escalar con EC2, **AWS Batch**, EMR, Fargate y Lambda.
- SQS y SNS: con una cola SQS procesada por Lambda, ¿cómo se manejan los errores a escala? ¿Qué cuotas duras o blandas afectan a Lambda?
- Serverless: API Gateway (**throttling y cuotas** para no saturar las APIs), Step Functions, EventBridge y **AWS AppSync** (interfaz **GraphQL** que combina datos de DynamoDB, Lambda y APIs HTTP).
- Capacidades y límites de RDS, Aurora, DynamoDB, ElastiCache, DAX y Redshift; qué servicios soportan Read Replicas y cómo funcionan; soporte entre Regions; bases globales; autoescalado de bases.
- Lazy loading vs write-through.
- DynamoDB nuevo con patrones de acceso desconocidos: ¿**on-demand** o capacidad aprovisionada?
- RDS MySQL migrado con tráfico mayoritariamente de lectura: ¿Read Replicas o ElastiCache?
- CloudFront, Global Accelerator y VPC endpoints para optimizar tráfico; VPC desde cero; híbrido con VPN y Direct Connect.
- Ingesta: DataSync, Storage Gateway, Transfer Family, Snow Family; Kinesis; Athena, Lake Formation, QuickSight; Glue; EMR.

---

## 9. Dónde se ve en esta guía

| Tema del dominio | Tomo |
|---|---|
| EC2, tipos de instancia, contenedores y Lambda como opciones de cómputo, VPC | [[02-Computo-y-Redes]] |
| S3, EBS, EFS, instance store, RDS, Aurora, DynamoDB | [[03-Almacenamiento-y-Bases-de-Datos]] |
| CloudWatch, métricas, ELB, Auto Scaling, Route 53 | [[04-Monitoreo-y-Optimizacion]] |
| API Gateway, Lambda, SQS, Step Functions, CloudFront | [[05-Backend-Web-Serverless]] |
| Kinesis, Glue, Athena, QuickSight | [[06-Analitica-de-Datos-Serverless]] |
| ECS, EKS, Fargate, Direct Connect, VPN, Storage Gateway, DataSync | [[07-Solucion-Hibrida-Contenedores]] |
| Transit Gateway y redes multicuenta | [[08-Gobierno-y-Multicuenta]] |
| Data lake: concepto y arquitectura | [[09-Introduccion-a-Data-Lakes]] |
| Ingesta, catálogo y preparación (Glue, DMS, Firehose) | [[10-Ingesta-Catalogo-y-Preparacion]] |
| Lake Formation y seguridad del lago | [[11-Data-Lake-con-Lake-Formation]], [[13-Lake-Formation-Avanzado]] |
| EMR, Parquet, procesamiento y análisis | [[12-Procesamiento-y-Analisis]] |
| Arquitectura de datos moderna | [[14-Arquitectura-de-Datos-Moderna]] |

---

## ✅ Autoevaluación

> [!question]- ¿Cómo se escala un volumen EBS y cómo EFS?
> EBS: modificando tipo, tamaño e IOPS (manual, admite cambios en vivo). EFS: automáticamente al agregar o borrar archivos.

> [!question]- ¿Qué palabras clave apuntan a FSx for Windows File Server?
> SMB, NTFS, Active Directory, instancias Windows.

> [!question]- ¿Qué métrica de EC2 no está disponible por defecto en CloudWatch?
> La memoria utilizada.

> [!question]- Un trabajo tarda 40 minutos. ¿Sirve Lambda?
> No, el máximo es 15 minutos; considerar Step Functions, contenedores o EC2.

> [!question]- ¿Qué ventaja tienen las réplicas de Aurora sobre las de RDS?
> Sirven a la vez para lecturas y para alta disponibilidad (beneficio Multi-AZ), sobre un volumen de clúster compartido.

> [!question]- ¿Cómo ahorra Aurora Serverless en períodos sin uso?
> Puede bajar a cero ACU y pausarse; se paga solo el almacenamiento.

> [!question]- ¿Cuántas conexiones de peering acepta una VPC según el curso?
> Hasta 125.

> [!question]- ¿Qué política de Route 53 envía al usuario a la Region más cercana geográficamente?
> Geoproximity routing.

> [!question]- ¿Qué servicio de Kinesis es la forma más simple de cargar streams en almacenes de AWS?
> Kinesis Data Firehose.

> [!question]- ¿Qué dos cachés tiene DAX?
> Item cache y query cache.

## 🧾 Resumen en 10 líneas

1. Dominio 3 pesa 24 % y tiene cinco task statements: almacenamiento, cómputo, bases, red, ingesta.
2. Decidir con datos (benchmarks, pruebas de carga), no con valores por defecto.
3. EBS escala a mano; EFS solo; FSx para Windows (SMB/NTFS/AD) y Lustre.
4. S3: multipart desde ~100 MB, `aws s3 cp` lo hace solo, Transfer Acceleration y CloudFront.
5. Cómputo: instancias, contenedores (ECS/EKS, EC2 o Fargate) y funciones (Lambda, 15 min).
6. Escalar por métricas correctas: custom, de cola SQS, HealthyHostCount, SurgeQueueLength.
7. Aurora: clúster con volumen compartido, réplicas para lectura y HA, Global Database; Serverless con ACU.
8. DynamoDB para ms de un dígito a escala; ElastiCache (Redis/Memcached) y DAX para caché.
9. Red: PrivateLink sobre peering a escala, Transit Gateway, Route 53 geoproximity, Global Accelerator, CloudFront.
10. Ingesta: Kinesis (Streams, Firehose, Analytics, Video), MSK, Glue, EMR, Lake Formation, con políticas y KMS.

## 📖 Recursos del módulo

- Almacenamiento en AWS · https://docs.aws.amazon.com/whitepapers/latest/aws-overview/storage-services.html
- Almacenamiento en la nube en AWS · https://aws.amazon.com/products/storage/
- Administración del ciclo de vida del almacenamiento · https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html
- Servicios de cómputo · https://docs.aws.amazon.com/whitepapers/latest/aws-overview/compute-services.html
- Bases de datos en AWS · https://docs.aws.amazon.com/whitepapers/latest/aws-overview/database.html
- Redes en Amazon EC2 · https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-networking.html
- Comenzar con Amazon VPC · https://docs.aws.amazon.com/vpc/latest/userguide/vpc-getting-started.html
- Conectar VPC con VPC peering · https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html
- Conectar su VPC a servicios con AWS PrivateLink · https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-services-overview.html
- Filtrar tráfico con AWS Network Firewall · https://docs.aws.amazon.com/vpc/latest/userguide/network-firewall.html
- Filtrar tráfico DNS con Route 53 Resolver DNS Firewall · https://docs.aws.amazon.com/vpc/latest/userguide/resolver-dns-firewall.html
- Escenarios con VPC · https://docs.aws.amazon.com/vpc/latest/userguide/vpc-scenarios-intro.html
- ¿Qué es AWS Global Accelerator? · https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html
- ¿Qué es Amazon CloudFront? · https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html
- Métodos de ingesta de datos · https://docs.aws.amazon.com/whitepapers/latest/building-data-lakes/data-ingestion-methods.html
- Procesamiento de datos · https://docs.aws.amazon.com/whitepapers/latest/data-warehousing-on-aws/data-processing.html
- ¿Qué es AWS Lake Formation? · https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[17-Dominio-2-Arquitecturas-Resilientes|17 · Dominio 2: Arquitecturas Resilientes]] · Siguiente: [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo|19 · Dominio 4: Costo ➡]]
