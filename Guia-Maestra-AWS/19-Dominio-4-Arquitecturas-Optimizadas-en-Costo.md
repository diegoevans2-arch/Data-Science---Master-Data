---
title: "Tomo 19 — Dominio 4: Design Cost-Optimized Architectures (20 %)"
tags: [aws, solutions-architect, saa-c03, costos, finops, right-sizing, pricing]
curso: 4 · Exam Prep: AWS Certified Solutions Architect – Associate
modulo: "Dominio 4 · Ámbito 4: Diseñar arquitecturas de costos optimizados"
audiencias: [tecnico, puente, ejecutivo]
tomo: 19
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 💰 Tomo 19 — Dominio 4: Design Cost-Optimized Architectures (20 %)

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento|18 · Dominio 3: Alto Rendimiento]] · Siguiente: [[20-Glosario|20 · Glosario ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Optimizar costos es **entregar valor de negocio al menor precio**. El dominio 4 pesa **20 %**, pero AWS agrega cada vez más preguntas de costo en todos sus exámenes y el calificativo «más costo-efectiva» aparece en preguntas de cualquier dominio. Tiene cuatro task statements —almacenamiento, cómputo, bases de datos y redes— y una regla de oro que la instructora deja al cierre: **primero una solución que cumpla los requisitos; después, entre las que cumplen, la más barata**.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** modelo de compra de cómputo (On-Demand, Savings Plans, Reserved, Spot), clases de S3 y lifecycle, réplica o caché en vez de instancia más grande, VPN o Direct Connect, NAT compartido en desarrollo, estrategia de etiquetado para asignar costos.
> - **Costo de hacerlo mal:** pagar recursos ociosos, IOPS que no se usan, snapshots que nadie borra, transferencia entre AZ y Regions que nadie diseñó.
> - **Pregunta que responde:** *¿dónde se va la plata, quién la gasta y qué cambio de arquitectura la reduce sin romper los requisitos?*

> [!tip] 💡 Analogía general: la cuenta de la luz de la casa
> El right-sizing es no tener una ampolleta de 200 W en el pasillo; la elasticidad es apagar la luz al salir; el modelo de compra es elegir tarifa fija o variable; las etiquetas son los medidores por pieza que dicen quién consume; la transferencia de datos es el cargo de distribución que nadie lee hasta que llega la cuenta.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Dominio 4 · Introducción | 1 video | Definición de cost optimization y los cuatro task statements |
| 4.1 Almacenamiento costo-optimizado | 1 video + pregunta 4.1 | Instance store vs EBS vs S3, right-sizing, herramientas de costo, lifecycle, Requester Pays, migración de datos |
| 4.2 Cómputo costo-optimizado | 1 video + pregunta 4.2 | Modelos de compra, cinco prácticas de cost optimization, híbrido y edge, balanceadores y métricas |
| 4.3 Bases de datos costo-optimizadas | 1 video | Elegir el almacén correcto, migraciones parciales, réplica o caché, retención de backups |
| 4.4 Redes costo-optimizadas | 1 video + pregunta 4.4 | VPN vs Direct Connect, peering vs Transit Gateway, transferencia de datos, NAT, CloudFront, throttling |
| Resumen y recursos | 1 video · 1 lectura | Lista de repaso y 7 enlaces |

> [!note] ➕ Nota del editor
> La numeración salta de la pregunta 4.2 a la 4.4: el task statement de bases de datos no trae pregunta de práctica en el material. Se menciona además el laboratorio final «Construir una arquitectura serverless», sin texto en lo extraído.

---

## 1. Task Statement 4.1 — Design cost-optimized storage solutions

Audiencia: 🔧 🧭

Primero elegir el tipo correcto (si piden un sistema de archivos, S3 no sirve: es objeto), y después **dimensionar** sin sobreaprovisionar.

### 1.1 Opciones y su lógica de costo

| Opción | Costo | Uso |
|---|---|---|
| **Instance store** (efímero local) | El más barato: **incluido en la tarifa horaria** de la EC2 | Procesamiento local, nunca persistencia de largo plazo |
| **Amazon EBS** | **Costo adicional** | Bloque persistente, migrable entre instancias, snapshots para backup y migración |
| **Amazon S3** | Bajo costo | Accesible desde cualquier lugar con Internet, **11 nueves** de durabilidad, sin límite de capacidad, lifecycle hacia Glacier, varias storage classes |

### 1.2 Right-sizing de EBS

- Evaluar al inicio **y** después de un tiempo de operación.
- Comparar **IOPS usadas vs IOPS pagadas**: si paga Provisioned IOPS y la carga rinde bien con ráfagas, pasar a **gp2** sin perder rendimiento.
- **Trusted Advisor** destaca volúmenes EBS no adjuntos o infrautilizados; con una estrategia de archivado, borrarlos reduce costo.
- **Amazon Data Lifecycle Manager** y **AWS Backup** eliminan automáticamente snapshots y backups antiguos.

### 1.3 Visibilidad y herramientas de gestión de costos

- Monitorear uso y costo para atribuir el gasto: CPU, RAM, almacenamiento → qué instancias achicar o agrandar.
- Herramientas: **CloudWatch** (métricas y alarmas), **Trusted Advisor**, **AWS Well-Architected Tool**, **Cost Explorer**.
- Definir métricas, objetivos y una **estrategia de etiquetado** con **cost allocation tags** (por ejemplo, para filtrar vistas en Cost Explorer); revisar cambios de infraestructura con regularidad.

| Necesidad | Herramienta |
|---|---|
| Informes financieros interactivos de alto nivel, con drill-down | **Cost Explorer** |
| Desglose fino por hora, día, mes, producto, etiquetas | **Cost and Usage Report** |
| Alarmas de facturación, de free tier y por presupuesto; acciones automáticas al cruzar umbrales | **AWS Budgets** (y billing alarms) |
| Facturación centralizada, control de acceso, cumplimiento, compartir recursos | **AWS Organizations, Control Tower** (consolidated billing) |
| Tener solo las instancias necesarias | **AWS Auto Scaling, EC2 Auto Scaling** |

### 1.4 S3: clases, automatización y quién paga

- No hace falta saber precios exactos, sí el **trade-off costo de almacenamiento vs costo y tiempo de recuperación** de cada clase según el patrón de acceso.

| Necesidad | Capacidad |
|---|---|
| Transicionar **todos** los objetos de un bucket según el tiempo | **S3 Lifecycle configuration** |
| Mover **cada objeto** a la capa más barata cuando cambia su patrón de acceso | **S3 Intelligent-Tiering** |
| Que el solicitante pague las solicitudes y la descarga | **Requester Pays** |

### 1.5 Migración de datos híbrida

Opciones: **DataSync, Snow Family, Transfer Family, Storage Gateway**. Para mover terabytes a petabytes de forma rápida y segura, **Snowball** suele ser la opción sólida.

> [!example] 📊 250 TB de archivo on-premises a S3, lo más rápido y barato
> El curso compara Snowmobile, levantar Direct Connect, varios Snowball o subir por la línea actual, e indica que **varios dispositivos Snowball** suelen ser la respuesta sólida para ese volumen.

Para minimizar transferencia: **CloudFront** para cachear cerca de los usuarios o **Direct Connect** como enlace dedicado.

---

## 2. Task Statement 4.2 — Design cost-optimized compute solutions

Audiencia: 🔧 🧭 👔

Usar los recursos más eficaces; por ejemplo, servicios administrados en lugar de mantener servidores. Una buena selección de servicios también baja costo: CloudFront reduce transferencia y **Aurora y RDS eliminan licencias de bases de datos** caras. Evaluar tamaño o número de recursos, modelos de precio y costos de transferencia.

### 2.1 Las cinco prácticas de optimización de costo que recorre el curso

| # | Práctica | Aplicada a cómputo |
|---|---|---|
| 1 | **Right-sizing** (primer paso) | El tipo de instancia más barato que cumple el rendimiento; elegir familia (compute-optimized para batch, memory-optimized para datasets grandes en memoria) |
| 2 | **Aumentar la elasticidad** | Usar recursos solo cuando hacen falta; **más instancias pequeñas** en vez de pocas grandes; Auto Scaling hacia abajo fuera de horario (AWS ofrece una plantilla CloudFormation gestionada por **etiquetas**) |
| 3 | **Modelo de precio correcto** | On-Demand, Savings Plans, Reserved, Spot, Dedicated Hosts, Dedicated Instances, Scheduled Instances, Capacity Reservations |
| 4 | **Alinear el almacenamiento al uso** | Right-sizing del almacenamiento del entorno de cómputo |
| 5 | **Medir y monitorear** | Mejora continua con CloudWatch, Cost Explorer, etiquetas y revisiones periódicas |

> [!note] ➕ Nota del editor
> La transcripción llama «pilares» a estas cinco prácticas; no son los seis pilares del Well-Architected Framework, sino prácticas dentro del pilar Cost Optimization.

### 2.2 Modelos de compra en el examen

| Escenario | Elección |
|---|---|
| Batch que tolera interrupciones, costo mínimo | **Spot Instances** |
| Flotas donde servidores entran y salen: web stateless, batch, HPC, big data | **Spot** |
| No tolera interrupciones, máximo ahorro, flexibilidad entre **EC2, Fargate y Lambda** | **Savings Plans** |

### 2.3 Híbrido y edge

- **AWS Outposts** y **Snowball Edge**: AWS publica cálculos y ejemplos. En Outposts, los servicios administrados se cobran por hora de instancia usada, **excluyendo** la EC2 subyacente y el almacenamiento EBS.
- **Edge computing:** acercar el cómputo a los usuarios (CloudFront, soluciones IoT en zonas remotas o de baja latencia). El ahorro grande: no operar un data center ni administrar servidores.

### 2.4 Balanceadores y métricas de escalado

- Right-sizing + **Elastic Load Balancer** para escalar horizontalmente.
- Distinguir **ALB, NLB y Gateway Load Balancer** por casos de uso y costo.
- Escalar el ASG con métricas del balanceador, como **RequestCountPerTarget** del ALB.
- Agregar **ELB health checks** al ASG para reemplazar instancias no sanas, o una **alarma CloudWatch** si los hosts sanos del target group bajan del mínimo permitido.

---

## 3. Task Statement 4.3 — Design cost-optimized database solutions

Audiencia: 🔧 🧭

- Un RDBMS es el núcleo de la mayoría de los sistemas transaccionales, pero **no todo debe vivir ahí**: sube el costo total de propiedad y puede dañar el rendimiento. Elegir el almacén por **patrón de acceso, escala y crecimiento esperado, frecuencia de acceso**.
- Optimizaciones al migrar:
  1. Mover un subconjunto de la base relacional a **EC2 o RDS**.
  2. Mover **objetos grandes** a **S3**.
  3. Mover un subconjunto a un **NoSQL** como **DynamoDB**, que elimina aprovisionar hardware, instalar, configurar, replicar, parchar y escalar clústeres.
- Preguntas para decidir: ¿se usan características específicas del motor? ¿el esquema seguirá fijo a medida que evolucione la aplicación? ¿se quieren restricciones entre tablas?
- **Aurora Serverless** puede salir más barato que Aurora aprovisionado.
- RDS **storage autoscaling** para dimensionar bien el almacenamiento.

> [!example] 📊 CPU alta por picos de lectura
> Agrandar la instancia sube el costo. Escalar horizontalmente suele ser más barato, y lo **más costo-efectivo** es agregar una **Read Replica** o una **caché** que absorba lecturas.

- **Backups:** frecuencia según el **RPO**; saber qué servicios administrados soportan **point-in-time recovery**; política de **retención** que no guarde snapshots más allá de su vida útil.
- Conocer los distintos motores de RDS y por qué elegir uno u otro.
- **Servicios administrados** siempre que se pueda: operan a escala de nube y ofrecen menor costo por transacción.

---

## 4. Task Statement 4.4 — Design cost-optimized network architectures

Audiencia: 🔧 🧭 👔

En AWS la red está virtualizada y es un servicio administrado: no hay switches ni routers que operar y se paga por uso, pero **se olvidan las llamadas a API y la transferencia de datos**.

### 4.1 Decisiones de conectividad

| Decisión | Regla del curso |
|---|---|
| Conexión privada on-premises: Site-to-Site VPN vs Direct Connect dedicado | Direct Connect suele costar más; salvo que se necesite su rendimiento o seguridad, **VPN** es más costo-efectiva |
| Failover de un Direct Connect con costo mínimo | **Site-to-Site VPN**: más barata y rápida de montar que un segundo Direct Connect |
| Administrar instancias | SSH/RDP clásico, **Systems Manager Session Manager**, **EC2 Instance Connect**: conocer el costo de cada uno |
| VPC a VPC entre Regions | La transferencia entre Regions se cobra; **VPC peering** puede salir más barato que **Transit Gateway** |
| EC2 privada → S3/DynamoDB | **Gateway endpoints**: sin cargo de transferencia dentro de la misma Region |
| Tráfico entre AZ | Normalmente se cobra: usar recursos de la **misma AZ** cuando se pueda |
| NAT gateway | Producción: uno por AZ en subnet pública. **Desarrollo: un NAT compartido** para ahorrar |

### 4.2 Throttling en API Gateway

Para limitar cuántas transacciones por segundo y por semana usa cada cliente: después de probar y desplegar las APIs, crear **usage plans** y **API keys**; los clientes acceden a las APIs elegidas y las solicitudes se limitan según los límites y cuotas definidos.

### 4.3 S3 y CloudFront

- S3 cobra por **datos almacenados**, **llamadas a la API** y **transferencia hacia fuera** del bucket.
- Poner **CloudFront** delante reduce llamadas y transferencia (según volumen y patrón de acceso).
- Desde orígenes de AWS (**S3, EC2, ELB**) **no se cobra** la transferencia del origen a las edge locations de CloudFront.
- CloudFront ofrece **regional edge caches sin costo adicional**, que bajan la carga y el costo operativo de orígenes intensivos.
- CloudFront sirve para más que solo poner un bucket detrás de una distribución.

### 4.4 Monitoreo de red

CloudWatch, AWS Health Dashboard, **VPC Reachability Analyzer**, **Transit Gateway Network Manager**, AWS Config y Route Analyzer ayudan a encontrar la causa raíz de problemas de rendimiento.

**Cómo revisar una carga existente:** observar patrones de costo históricos; mantener datos en la misma AZ y Region; elegir **Regions más baratas** cuando se pueda; aprovechar el **free tier**.

---

## 5. Preguntas de práctica resueltas en el curso

Audiencia: 🔧 🧭

### Pregunta 4.1 — Servidores Windows con archivos compartidos y Active Directory

**Escenario:** mover servidores Microsoft Windows a AWS; necesitan un sistema de archivos compartido integrado con el **Active Directory existente** para permisos de archivos y carpetas, compartido entre AWS y on-premises, altamente disponible, con cifrado en reposo y en tránsito. Palabras clave: *Windows*, *sistema de archivos compartido*, *AD existente*, *HA*, *híbrido*, *cifrado*.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| S3 File Gateway unido al AD existente | ❌ | Expone S3 por NFS/SMB y autentica con AD on-premises, pero no una vez que los objetos están en S3 |
| **FSx for Windows File Server unido al AD existente** | ✅ | Sistema de archivos Windows nativo totalmente administrado; sin costo inicial de hardware/software, pago por uso, sin mínimos ni cargos de configuración |
| EFS unido a un AWS Managed AD | ❌ | EFS es para cargas Linux; no soporta instancias Windows |
| Bucket S3 montado en EC2 Windows multi-AZ | ❌ | S3 es objeto; puede respaldar un sistema de archivos, pero el curso indica que montar buckets en EC2 no está soportado |

### Pregunta 4.2 — Microservicios de una startup, sin infraestructura, escalado rápido y protección DDoS

**Escenario:** correr y exponer microservicios de forma costo-efectiva, **sin carga operativa** de infraestructura, que escalen **rápido** ante cambios bruscos de volumen y se protejan de **DDoS** comunes.

| Opción | Veredicto | Razón |
|---|---|---|
| Contenedores en Elastic Beanstalk | ❌ | Corre sobre EC2 + ASG + ELB: escalar tarda minutos (arranque de instancias) y paga recursos ociosos |
| **Lambda detrás de API Gateway** | ✅ | Serverless: sin infraestructura; escala rápido sin gestionarlo; difícil de saturar; API Gateway permite throttling por método (las solicitudes se descartan en API Gateway, no en el backend) y protege contra ataques de capa 7 y SYN floods; no se pagan recursos ociosos |
| EC2 en un Auto Scaling group | ❌ | Mismas razones que Beanstalk |
| Contenedores en ECS sobre EC2 | ❌ | Un servicio escala rápido, pero escalar el clúster tarda minutos y hay recursos ociosos |

> [!note] ➕ Nota del editor
> La explicación del curso ubica las SYN floods en la «capa 3» del modelo OSI; habitualmente se clasifican como ataques de capa 4 (transporte). Verificar antes de memorizarlo.

### Pregunta 4.4 — Acceso seguro de desarrolladores a EC2 en subnet privada con NAT

**Escenario:** instancias en subnet privada que salen a Internet por un NAT gateway; los desarrolladores necesitan iniciar sesión de forma segura y **lo más costo-efectiva** posible.

| Opción | Veredicto | Razón |
|---|---|---|
| **Systems Manager Session Manager** | ✅ | Acceso seguro y auditable sin abrir puertos de entrada, sin bastion host ni claves SSH; **sin cargo adicional** |
| Bastion host en subnet pública | ❌ | Funciona, pero el bastión es una EC2 que se paga |
| Usar el NAT gateway existente | ❌ | Imposible: el NAT solo permite tráfico **saliente** |
| Site-to-Site VPN | ❌ | Funciona, pero agrega costo |

---

## 6. Trampas típicas del dominio

> [!danger] 🚨 Errores costosos
> - Elegir **Spot** cuando la carga no tolera interrupciones (va Savings Plans o Reserved).
> - Escalar **verticalmente** la base de datos cuando una Read Replica o una caché resuelven más barato.
> - Confundir **Lifecycle** (por tiempo, todo el bucket) con **Intelligent-Tiering** (por objeto, según acceso).
> - Usar **Cost Explorer** cuando piden detalle por hora y etiqueta (va Cost and Usage Report).
> - Proponer un segundo **Direct Connect** como respaldo barato (va VPN).
> - Un **NAT gateway por AZ** en desarrollo, o creer que el NAT permite entrar a una instancia.
> - Ignorar la transferencia **entre AZ y entre Regions**.
> - Elegir EFS para Windows (va FSx for Windows File Server).

---

## 7. Lista de repaso del cierre del dominio

- Modelo de costo variable: iterar la factura mes a mes (right-sizing de EC2, luego Intelligent-Tiering en S3, luego la siguiente mejora).
- Cómputo: tipo y tamaño de instancia, luego modelo de precio (On-Demand, Reserved, Savings Plans, Dedicated, Spot).
- **Lambda** cobra por recursos consumidos y tiempo de ejecución: funciones más eficientes cuestan menos.
- **Contenedores**: imágenes más pequeñas y livianas; elegir EC2 o Fargate para el clúster.
- Etiquetas para los informes; Cost Explorer, Cost and Usage Report y Budgets.
- Clases de S3 y métodos automáticos de transición. Caso: logs de producción que casi no se leen tras 30 días y se archivan a los 90 → ¿Intelligent-Tiering o Lifecycle?
- Savings Plans, Reserved Instances y Spot; Auto Scaling e **hibernación**.
- EBS: tipos y tamaños correctos, lifecycle de snapshots y backups.
- Bases de datos: el motor correcto, caché antes que escalar siempre, afinar consultas y diseño.
- Red: minimizar NAT gateways en VPC de desarrollo; rutas que reduzcan transferencia Region↔Region y AZ↔AZ; VPC endpoints; Global Accelerator.
- Siempre: primero cumplir requisitos, luego optimizar costo entre las alternativas válidas.

---

## 8. Dónde se ve en esta guía

| Tema del dominio | Tomo |
|---|---|
| Modelo de pago por uso, free tier, facturación | [[01-Vision-General-y-Seguridad]] |
| Familias de instancia, modelos de compra de EC2, NAT gateway, VPC | [[02-Computo-y-Redes]] |
| Instance store vs EBS vs S3, storage classes, lifecycle, snapshots | [[03-Almacenamiento-y-Bases-de-Datos]] |
| CloudWatch, Auto Scaling, ELB, right-sizing | [[04-Monitoreo-y-Optimizacion]] |
| Lambda, API Gateway (usage plans, throttling), CloudFront | [[05-Backend-Web-Serverless]] |
| Costo de analítica serverless | [[06-Analitica-de-Datos-Serverless]] |
| Direct Connect, VPN, Snow Family, DataSync, Storage Gateway, Fargate | [[07-Solucion-Hibrida-Contenedores]] |
| Organizations, consolidated billing, etiquetas y gobierno de costos | [[08-Gobierno-y-Multicuenta]] |
| Formatos columnares y almacenamiento económico del lago | [[12-Procesamiento-y-Analisis]], [[14-Arquitectura-de-Datos-Moderna]] |

---

## ✅ Autoevaluación

> [!question]- ¿Cuál es la opción de almacenamiento de menor costo asociada al cómputo y por qué no sirve para todo?
> El instance store, incluido en la tarifa de la EC2; es efímero y no sirve para persistencia de largo plazo.

> [!question]- Un volumen con Provisioned IOPS rinde igual en ráfagas. ¿Qué se hace?
> Cambiarlo a gp2 para ahorrar sin perder rendimiento.

> [!question]- ¿Qué herramienta da el desglose por hora, producto y etiqueta?
> El AWS Cost and Usage Report.

> [!question]- Carga que no tolera interrupciones y corre en EC2, Fargate y Lambda. ¿Modelo de compra?
> Savings Plans.

> [!question]- ¿Qué servicio permite cobrar al solicitante la descarga de un bucket?
> S3 Requester Pays.

> [!question]- ¿Cuál es el respaldo más barato para una conexión Direct Connect?
> Una Site-to-Site VPN.

> [!question]- ¿Se cobra la transferencia de S3 a CloudFront?
> No, desde orígenes de AWS (S3, EC2, ELB) hacia las edge locations no hay cargo.

> [!question]- ¿Cómo se ahorra en NAT gateways en un entorno de desarrollo?
> Usando un NAT gateway compartido en lugar de uno por AZ.

> [!question]- ¿Cómo se limita el consumo de una API por cliente en API Gateway?
> Con usage plans y API keys que aplican límites y cuotas.

> [!question]- ¿Por qué Session Manager gana al bastion host en costo?
> No tiene cargo adicional y no requiere una EC2 extra, puertos de entrada ni claves SSH.

## 🧾 Resumen en 10 líneas

1. Dominio 4 pesa 20 %: almacenamiento, cómputo, bases de datos y redes.
2. Cost optimization = valor de negocio al menor precio; primero cumplir requisitos.
3. Instance store incluido en la EC2, EBS con costo, S3 barato y con 11 nueves.
4. Right-sizing de EBS por IOPS reales; Trusted Advisor, Data Lifecycle Manager y AWS Backup limpian.
5. Cost Explorer (alto nivel), Cost and Usage Report (detalle), Budgets (alarmas y acciones), etiquetas.
6. Lifecycle por tiempo, Intelligent-Tiering por objeto, Requester Pays; Snowball para cientos de TB.
7. Cinco prácticas: right-sizing, elasticidad, modelo de precio, almacenamiento alineado, medir.
8. Spot si tolera interrupciones; Savings Plans si no y se usa EC2/Fargate/Lambda.
9. Bases: el almacén correcto, réplica o caché antes que escalar, retención acotada.
10. Red: VPN antes que Direct Connect, gateway endpoints, misma AZ, NAT compartido en dev, CloudFront ante S3.

## 📖 Recursos del módulo

- Mejores prácticas de arquitectura para la optimización de costos · https://aws.amazon.com/architecture/cost-optimization/?cards-all.sort-by=item.additionalFields.sortDate&cards-all.sort-order=desc&awsf.content-type=*all&awsf.methodology=*all
- Pilar de optimización de costos · https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.pillar.costOptimization.en.html
- Optimización de costos de AWS · https://docs.aws.amazon.com/whitepapers/latest/how-aws-pricing-works/aws-cost-optimization.html
- Herramientas de AWS para informes y optimización de costos · https://docs.aws.amazon.com/whitepapers/latest/cost-optimization-laying-the-foundation/reporting-cost-optimization-tools.html
- Amazon Data Lifecycle Manager · https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/snapshot-lifecycle.html
- Precios de Amazon EC2 · https://aws.amazon.com/ec2/pricing/
- Recursos costo-efectivos · https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/cost-effective-resources.html

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento|18 · Dominio 3: Alto Rendimiento]] · Siguiente: [[20-Glosario|20 · Glosario ➡]]
