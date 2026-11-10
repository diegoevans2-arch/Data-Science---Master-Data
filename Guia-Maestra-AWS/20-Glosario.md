---
title: "Tomo 20 — Glosario"
tags: [aws, solutions-architect, glosario]
audiencias: [tecnico, puente, ejecutivo]
tomo: 20
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 📘 Tomo 20 — Glosario

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo|19 · Dominio 4]] · Siguiente: [[21-Recursos-y-Referencias|21 · Recursos ➡]]

---

> [!info] 📌 ¿Cómo usar este glosario?
> Reúne los términos y servicios clave de los tomos 01–19, en orden alfabético. Los términos van en inglés, como en la consola y en el examen; la definición es breve y la columna **Tomo** lleva a donde se explica en detalle. Si un término aparece en varios tomos, el primero es el de la explicación principal.

---

## A

| Término | Definición | Tomo |
|---|---|---|
| ACU (Aurora Capacity Unit) | Unidad de capacidad (CPU + memoria) en la que escala Aurora Serverless. | [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] · [[05-Backend-Web-Serverless\|05]] |
| AMI (Amazon Machine Image) | Plantilla con sistema operativo y software desde la que se lanza una instancia EC2. | [[02-Computo-y-Redes\|02]] · [[01-Vision-General-y-Seguridad\|01]] |
| Amazon Q / Q Developer / Q Business | Asistentes de IA generativa de AWS: para desarrolladores (código, consola) y para datos de la empresa. | [[04-Monitoreo-y-Optimizacion\|04]] |
| API Gateway | Servicio administrado para publicar APIs REST, HTTP y WebSocket; puede integrarse directo con otros servicios. | [[05-Backend-Web-Serverless\|05]] · [[06-Analitica-de-Datos-Serverless\|06]] |
| AppFlow | Ingesta administrada desde aplicaciones SaaS (Salesforce, etc.) hacia S3 u otros destinos. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| AppSync | Servicio administrado de APIs GraphQL. | [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| Athena | Motor SQL serverless sobre datos en S3; cobra por bytes escaneados. | [[06-Analitica-de-Datos-Serverless\|06]] · [[10-Ingesta-Catalogo-y-Preparacion\|10]] · [[12-Procesamiento-y-Analisis\|12]] |
| Aurora | Base relacional de AWS compatible con MySQL y PostgreSQL, con volumen de clúster distribuido en 3 AZ. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| Auto Scaling group (ASG) | Grupo de instancias EC2 que escala entre un mínimo, un deseado y un máximo según políticas. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Availability Zone (AZ) | Uno o más centros de datos con energía y red redundantes dentro de una Region. | [[01-Vision-General-y-Seguridad\|01]] |
| AWS Artifact | Portal de informes de cumplimiento y acuerdos de AWS. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| AWS Budgets | Presupuestos con alertas sobre costo o uso. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| AWS Config | Registro y evaluación continua de la configuración de los recursos. | [[08-Gobierno-y-Multicuenta\|08]] |
| AssumeRole (STS) | Llamada que entrega credenciales temporales para actuar con un IAM role. | [[08-Gobierno-y-Multicuenta\|08]] · [[16-Dominio-1-Arquitecturas-Seguras\|16]] |

## B

| Término | Definición | Tomo |
|---|---|---|
| Backoff exponencial | Reintentos con esperas crecientes para no saturar un servicio que falla o limita. | [[06-Analitica-de-Datos-Serverless\|06]] |
| Bedrock | Servicio administrado para usar modelos fundacionales (FM) vía API, incluido RAG. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Blueprint (Lake Formation) | Plantilla que genera workflows de Glue para ingerir datos al lake. | [[11-Data-Lake-con-Lake-Formation\|11]] |
| Bucket policy | Política basada en recursos que controla el acceso a un bucket S3. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[16-Dominio-1-Arquitecturas-Seguras\|16]] |

## C

| Término | Definición | Tomo |
|---|---|---|
| CCoE (Cloud Center of Excellence) | Equipo transversal que define estándares y gobierno de la nube. | [[08-Gobierno-y-Multicuenta\|08]] |
| CIDR | Notación de rangos IP (p. ej. 10.1.0.0/16) usada para VPC y subnets. | [[02-Computo-y-Redes\|02]] |
| Classifier (Glue) | Regla que usa un crawler para reconocer el formato de los datos. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| CloudFormation | Infraestructura como código con plantillas JSON/YAML. | [[08-Gobierno-y-Multicuenta\|08]] · [[06-Analitica-de-Datos-Serverless\|06]] |
| CloudFront | CDN de AWS; sirve contenido en caché desde edge locations. | [[01-Vision-General-y-Seguridad\|01]] · [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| CloudHSM | Módulos HSM dedicados (FIPS 140-2 nivel 3) para claves propias. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| CloudTrail | Registro de llamadas a la API de AWS (auditoría). | [[08-Gobierno-y-Multicuenta\|08]] · [[14-Arquitectura-de-Datos-Moderna\|14]] |
| CloudWatch | Monitoreo: métricas, alarmas, logs y dashboards. | [[04-Monitoreo-y-Optimizacion\|04]] |
| CloudWatch alarm | Vigila una métrica; estados OK, ALARM e INSUFFICIENT_DATA; puede notificar por SNS o escalar. | [[04-Monitoreo-y-Optimizacion\|04]] |
| CloudWatch Logs | Log groups, log streams y log events; con metric filters se convierten en métricas. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Compactación | Unir muchos archivos pequeños del lake en pocos grandes para leer más rápido y barato. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] · [[14-Arquitectura-de-Datos-Moderna\|14]] |
| Connection draining (deregistration delay) | Tiempo en que el balanceador deja terminar las conexiones de un target que sale. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Control Tower / landing zone | Despliegue gobernado de un entorno multicuenta con guardrails. | [[08-Gobierno-y-Multicuenta\|08]] |
| Cost allocation tags | Etiquetas para repartir costos por proyecto, área o ambiente. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Cost Explorer / CUR | Análisis visual de costos / informe detallado de costo y uso (Cost and Usage Report). | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Crawler (Glue) | Recorre datos en S3 u otras fuentes, infiere el esquema y crea tablas en el Data Catalog. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| Credential vending | Lake Formation entrega credenciales temporales acotadas a los motores que leen el lake. | [[11-Data-Lake-con-Lake-Formation\|11]] |
| Cross-Region Replication (CRR) | Copia automática de objetos S3 a un bucket de otra Region. | [[06-Analitica-de-Datos-Serverless\|06]] |
| Customer gateway | Extremo del lado del cliente en una VPN Site-to-Site. | [[07-Solucion-Hibrida-Contenedores\|07]] |

## D

| Término | Definición | Tomo |
|---|---|---|
| Data Analytics Lens | Extensión del Well-Architected Framework para cargas analíticas. | [[14-Arquitectura-de-Datos-Moderna\|14]] |
| Data Catalog (Glue) | Metastore central de bases, tablas y particiones que usan Athena, Glue y otros. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| Data gravity | Los datos atraen a las aplicaciones y al cómputo: conviene procesar donde están. | [[12-Procesamiento-y-Analisis\|12]] · [[14-Arquitectura-de-Datos-Moderna\|14]] |
| Data lake | Repositorio central (en S3) para datos estructurados y no estructurados a cualquier escala, con schema-on-read. | [[09-Introduccion-a-Data-Lakes\|09]] · [[06-Analitica-de-Datos-Serverless\|06]] |
| Data mesh / data product | Datos gestionados como productos por dominios, con gobierno federado. | [[14-Arquitectura-de-Datos-Moderna\|14]] |
| Data swamp | Lake sin catálogo ni gobierno, donde los datos ya no se encuentran ni se confían. | [[09-Introduccion-a-Data-Lakes\|09]] |
| Data warehouse | Almacén analítico de datos estructurados con schema-on-write (p. ej. Redshift). | [[09-Introduccion-a-Data-Lakes\|09]] |
| DataBrew | Preparación visual de datos sin código: perfilado, recetas y reglas de calidad (DQ rules). | [[12-Procesamiento-y-Analisis\|12]] |
| DataZone | Catálogo y portal para publicar y consumir data products. | [[14-Arquitectura-de-Datos-Moderna\|14]] |
| DAX | Caché en memoria para DynamoDB (item cache y query cache). | [[05-Backend-Web-Serverless\|05]] · [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| DB subnet group | Conjunto de subnets (en varias AZ) donde RDS puede ubicar la base. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| Decoupling | Desacoplar componentes (colas, eventos) para que fallen y escalen por separado. | [[17-Dominio-2-Arquitecturas-Resilientes\|17]] · [[05-Backend-Web-Serverless\|05]] |
| Defense in depth | Controles de seguridad en todas las capas, no solo en el perímetro. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Delete marker | Marcador que S3 versionado crea al borrar; el objeto previo sigue recuperable. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| Direct Connect | Conexión de red dedicada y privada entre las instalaciones y AWS. | [[07-Solucion-Hibrida-Contenedores\|07]] · [[02-Computo-y-Redes\|02]] |
| Distractor / stem | En el examen: opción incorrecta plausible / enunciado de la pregunta. | [[15-Examen-SAA-Estrategia\|15]] |
| DLM (Data Lifecycle Manager) | Automatiza la creación y retención de snapshots EBS. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| DMS / SCT | Database Migration Service / Schema Conversion Tool para migrar bases. | [[07-Solucion-Hibrida-Contenedores\|07]] · [[09-Introduccion-a-Data-Lakes\|09]] |
| DynamoDB | Base NoSQL clave-valor serverless; partition key (y sort key), capacidad on-demand o provisionada (RCU/WCU). | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[05-Backend-Web-Serverless\|05]] |
| DynamoDB Streams | Flujo de cambios de una tabla DynamoDB, consumible por Lambda. | [[05-Backend-Web-Serverless\|05]] |

## E

| Término | Definición | Tomo |
|---|---|---|
| EBS (Elastic Block Store) | Volúmenes de bloque persistentes para EC2; snapshots en S3; Multi-Attach en io1/io2. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| EC2 | Máquinas virtuales en AWS; tipos y familias de instancia; On-Demand, Reserved, Spot y Savings Plans. | [[02-Computo-y-Redes\|02]] |
| ECS / EKS | Orquestadores de contenedores: propio de AWS (task definition) / Kubernetes administrado (pods, worker nodes). | [[02-Computo-y-Redes\|02]] · [[07-Solucion-Hibrida-Contenedores\|07]] |
| ECS Anywhere | Correr tareas ECS en servidores propios, gestionadas desde AWS. | [[07-Solucion-Hibrida-Contenedores\|07]] |
| Edge location | Punto de presencia de la red global para caché (CloudFront) y DNS. | [[01-Vision-General-y-Seguridad\|01]] |
| EFS / FSx | Sistemas de archivos administrados: NFS elástico / Windows, Lustre y otros. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[07-Solucion-Hibrida-Contenedores\|07]] |
| Elastic Disaster Recovery | Replicación continua de servidores a AWS para recuperación ante desastres. | [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| ELB: ALB / NLB / GWLB | Balanceadores de capa 7 (HTTP) / capa 4 (TCP/UDP) / para appliances de red. | [[04-Monitoreo-y-Optimizacion\|04]] |
| EventBridge | Bus de eventos serverless con reglas de enrutamiento. | [[05-Backend-Web-Serverless\|05]] |
| External ID | Valor extra en la trust policy para que terceros asuman un role de forma segura. | [[08-Gobierno-y-Multicuenta\|08]] |

## F – G

| Término | Definición | Tomo |
|---|---|---|
| Fargate | Cómputo serverless para contenedores (ECS/EKS) sin gestionar instancias; Fargate Spot más barato. | [[02-Computo-y-Redes\|02]] · [[07-Solucion-Hibrida-Contenedores\|07]] |
| Federated query | Consulta de Athena a fuentes fuera de S3 mediante conectores. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| Fine-grained access | Permisos a nivel de columna, fila o celda en Lake Formation. | [[11-Data-Lake-con-Lake-Formation\|11]] · [[13-Lake-Formation-Avanzado\|13]] |
| Firecracker | Microvirtualización sobre la que corre Lambda. | [[05-Backend-Web-Serverless\|05]] |
| Gateway endpoint | VPC endpoint gratuito para S3 y DynamoDB, vía route table. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Glacier (S3) | Clases S3 de archivo: Instant Retrieval, Flexible Retrieval y Deep Archive. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| Glue ETL job / Glue Studio | Trabajos Spark/Python de transformación / editor visual de esos trabajos. | [[12-Procesamiento-y-Analisis\|12]] |
| Glue workflow / triggers | Orquestación de crawlers y jobs con disparadores programados, a demanda o condicionales. | [[11-Data-Lake-con-Lake-Formation\|11]] · [[12-Procesamiento-y-Analisis\|12]] |
| Grantable | Permiso de Lake Formation que el receptor puede volver a otorgar. | [[13-Lake-Formation-Avanzado\|13]] |
| GuardDuty | Detección de amenazas a partir de logs (CloudTrail, VPC Flow Logs, DNS). | [[08-Gobierno-y-Multicuenta\|08]] |

## H – I

| Término | Definición | Tomo |
|---|---|---|
| HA / FT / DR | Alta disponibilidad (mínima caída) / tolerancia a fallos (sin caída) / recuperación ante desastres. | [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| Health check | Prueba periódica con la que ELB o Route 53 decide si un target está sano. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Hybrid access mode | Modo de Lake Formation que convive con permisos solo IAM durante la transición. | [[11-Data-Lake-con-Lake-Formation\|11]] |
| IAM group / user / policy / role | Identidades y permisos: grupo de usuarios, usuario con credenciales, documento JSON de permisos, identidad asumible con credenciales temporales. | [[01-Vision-General-y-Seguridad\|01]] · [[08-Gobierno-y-Multicuenta\|08]] |
| IAM Identity Center | Acceso centralizado (SSO) a varias cuentas mediante permission sets. | [[01-Vision-General-y-Seguridad\|01]] · [[08-Gobierno-y-Multicuenta\|08]] |
| Identity-based vs resource-based policy | Política adjunta a una identidad vs a un recurso (lleva `Principal`). | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Instance profile | Contenedor que permite asignar un IAM role a una instancia EC2. | [[01-Vision-General-y-Seguridad\|01]] |
| Instance store | Disco temporal ligado al host físico de EC2; se pierde al detener la instancia. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| Intelligent-Tiering | Clase S3 que mueve objetos entre niveles según el acceso, sin cargo de recuperación. | [[06-Analitica-de-Datos-Serverless\|06]] · [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Internet gateway / virtual private gateway | Puerta de la VPC hacia internet / hacia la red del cliente por VPN. | [[02-Computo-y-Redes\|02]] · [[07-Solucion-Hibrida-Contenedores\|07]] |
| IOPS / throughput | Operaciones de E/S por segundo / MB por segundo de un volumen. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |

## K – L

| Término | Definición | Tomo |
|---|---|---|
| Kinesis Data Streams | Streaming en tiempo real con shards y retención; consumidores propios. | [[06-Analitica-de-Datos-Serverless\|06]] · [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| Kinesis Data Firehose (Amazon Data Firehose) | Entrega administrada de streaming a S3, Redshift u OpenSearch con buffer, compresión y conversión. | [[06-Analitica-de-Datos-Serverless\|06]] · [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| KMS | Servicio de claves administradas para cifrado (SSE-KMS, etc.). | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Lake Formation | Construye, asegura y gobierna el data lake sobre S3 y el Data Catalog. | [[11-Data-Lake-con-Lake-Formation\|11]] · [[13-Lake-Formation-Avanzado\|13]] |
| Lake Formation roles | Data lake administrator, database creator, table creator. | [[11-Data-Lake-con-Lake-Formation\|11]] |
| Lambda | Funciones serverless dirigidas por eventos (trigger, handler); límite de 15 min por ejecución. | [[02-Computo-y-Redes\|02]] · [[05-Backend-Web-Serverless\|05]] |
| Lambda Layers / Power Tuning | Dependencias compartidas entre funciones / herramienta para elegir la memoria óptima. | [[05-Backend-Web-Serverless\|05]] |
| Launch template | Plantilla de configuración de instancias que usa un ASG. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Lazy loading / write-through | Estrategias de caché: cargar al fallar la lectura / escribir en caché al escribir en la base. | [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| Least privilege | Otorgar solo los permisos mínimos necesarios. | [[01-Vision-General-y-Seguridad\|01]] · [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| LF-Tags | Etiquetas de Lake Formation para otorgar permisos por atributo (con herencia). | [[13-Lake-Formation-Avanzado\|13]] |
| Lifecycle policy (S3) | Reglas que transicionan o expiran objetos según su antigüedad. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| Listener / target group / rule | Componentes de un ALB: puerto que escucha, grupo de destinos y reglas de enrutamiento. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Long polling / visibility timeout | SQS: esperar mensajes hasta 20 s / ocultar un mensaje mientras se procesa. | [[05-Backend-Web-Serverless\|05]] |

## M – N

| Término | Definición | Tomo |
|---|---|---|
| Macie | Descubre datos sensibles (PII) en S3. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| MFA | Segundo factor de autenticación; obligatorio para el root user. | [[01-Vision-General-y-Seguridad\|01]] |
| Metric / namespace / dimension | Serie de datos de CloudWatch, su espacio de nombres y sus atributos de filtro. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Modern data architecture | Lake + almacenes especializados + gobierno unificado; movimientos inside-out, outside-in y around the perimeter. | [[14-Arquitectura-de-Datos-Moderna\|14]] |
| MSK | Apache Kafka administrado. | [[14-Arquitectura-de-Datos-Moderna\|14]] · [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| Multi-account strategy | Separar cargas en varias cuentas para aislar seguridad, costos y límites. | [[08-Gobierno-y-Multicuenta\|08]] |
| Multipart upload / Transfer Acceleration | Subir objetos grandes en partes paralelas / acelerar subidas vía edge locations. | [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| NAT gateway / NAT instance | Salida a internet para subnets privadas: administrado / en una EC2 propia. | [[07-Solucion-Hibrida-Contenedores\|07]] · [[02-Computo-y-Redes\|02]] |
| Network ACL | Filtro de tráfico a nivel de subnet, stateless (reglas de entrada y salida). | [[02-Computo-y-Redes\|02]] |

## O – R

| Término | Definición | Tomo |
|---|---|---|
| OpenSearch Service | Búsqueda y analítica de logs administrada. | [[14-Arquitectura-de-Datos-Moderna\|14]] · [[06-Analitica-de-Datos-Serverless\|06]] |
| Organizations / OU / SCP | Agrupa cuentas en unidades organizativas y limita lo que pueden hacer con políticas de control de servicio. | [[08-Gobierno-y-Multicuenta\|08]] · [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Parquet / ORC | Formatos columnares comprimidos; reducen bytes escaneados (predicate pushdown, mín./máx. por bloque). | [[10-Ingesta-Catalogo-y-Preparacion\|10]] · [[06-Analitica-de-Datos-Serverless\|06]] |
| Partición | Organizar datos del lake por carpetas (p. ej. año/mes) para que las consultas lean solo lo necesario. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] · [[06-Analitica-de-Datos-Serverless\|06]] |
| Permissions boundary | Límite máximo de permisos que puede tener una identidad IAM. | [[08-Gobierno-y-Multicuenta\|08]] |
| Pilot light / warm standby | Estrategias de DR: núcleo mínimo encendido / copia reducida pero funcionando. | [[07-Solucion-Hibrida-Contenedores\|07]] · [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| Point-in-time recovery | Restaurar una base a cualquier segundo dentro de la retención de backups. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| PrivateLink / interface endpoint | Acceso privado a servicios desde la VPC sin pasar por internet. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Purpose-built databases | Elegir la base según el caso (relacional, clave-valor, grafo, series de tiempo…). | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| QuickSight / SPICE | BI serverless de AWS / su motor en memoria. | [[06-Analitica-de-Datos-Serverless\|06]] · [[13-Lake-Formation-Avanzado\|13]] |
| RAG | Generación aumentada con recuperación: el modelo responde con documentos propios como contexto. | [[04-Monitoreo-y-Optimizacion\|04]] |
| RDS / Multi-AZ / read replica | Bases relacionales administradas / réplica en espera en otra AZ / réplica de lectura. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] · [[07-Solucion-Hibrida-Contenedores\|07]] |
| RDS Proxy | Pool de conexiones administrado; acelera el failover y protege la base de Lambda. | [[05-Backend-Web-Serverless\|05]] · [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| Region | Zona geográfica con varias AZ; se elige por cumplimiento, latencia, precio y servicios. | [[01-Vision-General-y-Seguridad\|01]] |
| Requester Pays | Bucket donde quien descarga paga la transferencia. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Right-sizing | Ajustar el tamaño de los recursos a la demanda real. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Root user | Identidad inicial con acceso total a la cuenta; protegerla con MFA y no usarla a diario. | [[01-Vision-General-y-Seguridad\|01]] |
| Route table | Reglas que dirigen el tráfico de una subnet. | [[02-Computo-y-Redes\|02]] |
| RTO / RPO | Tiempo máximo de recuperación / pérdida máxima de datos aceptable. | [[17-Dominio-2-Arquitecturas-Resilientes\|17]] · [[07-Solucion-Hibrida-Contenedores\|07]] |

## S

| Término | Definición | Tomo |
|---|---|---|
| S3 | Almacenamiento de objetos: buckets, object keys, versioning, clases de almacenamiento. | [[03-Almacenamiento-y-Bases-de-Datos\|03]] |
| S3 File Gateway | Storage Gateway que expone S3 como recurso compartido NFS/SMB en las instalaciones. | [[07-Solucion-Hibrida-Contenedores\|07]] |
| SAA-C03 | Código actual del examen AWS Certified Solutions Architect – Associate (65 preguntas, 130 min). | [[15-Examen-SAA-Estrategia\|15]] |
| Savings Plans / Reserved / Spot | Compromiso de gasto por 1–3 años / reserva de capacidad / capacidad sobrante interrumpible. | [[02-Computo-y-Redes\|02]] · [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Scaling policies | Target tracking, step y simple scaling en EC2 Auto Scaling. | [[04-Monitoreo-y-Optimizacion\|04]] |
| Schema-on-read / on-write | El esquema se aplica al leer (lake) / al cargar (warehouse). | [[09-Introduccion-a-Data-Lakes\|09]] |
| Security group | Firewall stateful a nivel de instancia (solo reglas de permiso). | [[02-Computo-y-Redes\|02]] · [[01-Vision-General-y-Seguridad\|01]] |
| SerDe | Serializador/deserializador que indica a Athena/Glue cómo leer un formato. | [[06-Analitica-de-Datos-Serverless\|06]] |
| Service Catalog | Catálogo de productos aprobados (plantillas CloudFormation) para autoservicio. | [[08-Gobierno-y-Multicuenta\|08]] |
| Service-linked role | Role predefinido que un servicio de AWS usa en su nombre. | [[08-Gobierno-y-Multicuenta\|08]] |
| Session Manager | Acceso a instancias sin SSH ni bastión, vía Systems Manager. | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo\|19]] |
| Shared responsibility model | AWS asegura «la nube»; el cliente asegura lo que pone «en la nube». | [[01-Vision-General-y-Seguridad\|01]] |
| Shield / WAF | Protección DDoS (Standard gratis, Advanced pagado) / firewall de aplicaciones web. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Snow Family | Dispositivos físicos para mover grandes volúmenes de datos a AWS. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] |
| SNS (fan-out) | Pub/sub: un tema publica a muchos suscriptores (colas, Lambda, correo). | [[05-Backend-Web-Serverless\|05]] · [[04-Monitoreo-y-Optimizacion\|04]] |
| SQS / FIFO | Colas de mensajes; FIFO garantiza orden y procesamiento exactly-once. | [[05-Backend-Web-Serverless\|05]] · [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| SSE-S3 / SSE-KMS / SSE-C | Cifrado en servidor de S3 con claves de S3, de KMS o del cliente. | [[16-Dominio-1-Arquitecturas-Seguras\|16]] |
| Sticky sessions | El balanceador envía al mismo usuario siempre al mismo target. | [[04-Monitoreo-y-Optimizacion\|04]] · [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| Stop-hibernate | Detener una instancia EC2 guardando la RAM en el volumen raíz. | [[02-Computo-y-Redes\|02]] |
| Storage-first | Patrón serverless que guarda primero el evento (cola, stream) y lo procesa después. | [[05-Backend-Web-Serverless\|05]] |
| Subnet (pública / privada) | Rango de IP de una VPC en una AZ; es pública si su route table apunta a un internet gateway. | [[02-Computo-y-Redes\|02]] |
| Systems Manager (Run Command, Parameter Store) | Operación de flotas: ejecutar comandos y guardar parámetros y secretos. | [[07-Solucion-Hibrida-Contenedores\|07]] |

## T – Z

| Término | Definición | Tomo |
|---|---|---|
| Tag policy | Política de Organizations que estandariza las etiquetas. | [[08-Gobierno-y-Multicuenta\|08]] |
| Transfer Family | SFTP/FTPS administrado hacia S3 o EFS. | [[10-Ingesta-Catalogo-y-Preparacion\|10]] · [[17-Dominio-2-Arquitecturas-Resilientes\|17]] |
| Transit Gateway | Hub que interconecta muchas VPC y redes propias. | [[07-Solucion-Hibrida-Contenedores\|07]] |
| Trust policy | Parte de un IAM role que define quién puede asumirlo. | [[08-Gobierno-y-Multicuenta\|08]] |
| Undifferentiated heavy lifting | Trabajo de infraestructura que no distingue al negocio y conviene delegar en AWS. | [[01-Vision-General-y-Seguridad\|01]] |
| User data | Script que una instancia EC2 ejecuta al arrancar. | [[01-Vision-General-y-Seguridad\|01]] · [[02-Computo-y-Redes\|02]] |
| VPC | Red privada virtual aislada dentro de una Region. | [[02-Computo-y-Redes\|02]] |
| VPC peering | Conexión privada entre dos VPC (no transitiva). | [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento\|18]] |
| VPN Site-to-Site / Client VPN | Túnel cifrado por internet entre red propia y VPC / para usuarios individuales. | [[07-Solucion-Hibrida-Contenedores\|07]] · [[02-Computo-y-Redes\|02]] |
| VTL mapping template | Plantilla de API Gateway que transforma la petición antes de enviarla al servicio integrado. | [[06-Analitica-de-Datos-Serverless\|06]] |
| Well-Architected Framework | Seis pilares: excelencia operativa, seguridad, fiabilidad, eficiencia de rendimiento, optimización de costos y sostenibilidad. | [[08-Gobierno-y-Multicuenta\|08]] · [[15-Examen-SAA-Estrategia\|15]] |

---

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo|19 · Dominio 4]] · Siguiente: [[21-Recursos-y-Referencias|21 · Recursos ➡]]
