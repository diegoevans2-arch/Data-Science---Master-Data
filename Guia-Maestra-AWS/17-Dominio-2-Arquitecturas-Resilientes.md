---
title: "Tomo 17 — Dominio 2: Design Resilient Architectures (26 %)"
tags: [aws, solutions-architect, saa-c03, resiliencia, alta-disponibilidad, desacoplamiento, disaster-recovery]
curso: "4 · Exam Prep: AWS Certified Solutions Architect – Associate"
modulo: "Dominio 2 · Ámbito 2: Diseñar arquitecturas resilientes"
audiencias: [tecnico, puente, ejecutivo]
tomo: 17
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🛡️ Tomo 17 — Dominio 2: Design Resilient Architectures (26 %)

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[16-Dominio-1-Arquitecturas-Seguras|16 · Dominio 1: Arquitecturas Seguras]] · Siguiente: [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento|18 · Dominio 3: Alto Rendimiento ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Según el Well-Architected Framework, una carga resiliente se recupera cuando la estresan la carga, los ataques (accidentales o deliberados) o la falla de cualquiera de sus componentes. El dominio 2 pesa **26 %** y se divide en dos task statements: arquitecturas **escalables y débilmente acopladas**, y arquitecturas **altamente disponibles y/o tolerantes a fallos**. Aquí se juegan los conceptos que más se confunden en el examen: escalado horizontal vs vertical, alta disponibilidad vs tolerancia a fallos vs disaster recovery, Read Replicas vs Multi-AZ.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** nivel de redundancia (AZ, Region), estrategia de DR según RTO/RPO, qué partes desacoplar con colas o eventos, cuánto pagar por tolerancia a fallos frente a alta disponibilidad.
> - **Costo de hacerlo mal:** caídas visibles para clientes al crecer la demanda, registros procesados dos veces, una base de datos que se convierte en punto único de falla.
> - **Pregunta que responde:** *¿qué pasa cuando falla cada componente, cuánto tiempo y cuántos datos podemos perder, y cómo escala cada pieza por separado?*

> [!tip] 💡 Analogía general: el restaurante con cocina y garzones
> Si el garzón espera parado en la cocina hasta que el plato sale (integración síncrona), un cocinero lento frena todo el salón. Con un riel de comandas (una cola), los garzones siguen tomando pedidos y la cocina saca platos a su ritmo: cada parte escala por separado. Tener dos cocinas encendidas es tolerancia a fallos; tener una de reserva que se enciende rápido es alta disponibilidad; tener el plan escrito de qué hacer si se incendia el local es disaster recovery.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Dominio 2 · Introducción | 1 video | Definición de resiliencia y los dos task statements |
| 2.1 Arquitecturas escalables y débilmente acopladas | 1 video + pregunta 2.1 | Escalado, elasticidad, compute, bases de datos, caché, edge, Transfer Family, microservicios, serverless, SQS, desacoplamiento |
| 2.2 Arquitecturas HA y/o tolerantes a fallos | 1 video + pregunta 2.2 | HA vs FT vs DR, RTO/RPO, estrategias de DR, servicios de BD para desastres, automatización, Route 53, RDS Proxy, observabilidad |
| Recapitulación y recursos | 1 video · 1 lectura | Lista de repaso, sticky sessions, estado en contenedores, 15 enlaces |

> [!note] ➕ Nota del editor
> El material menciona un segundo laboratorio, «Configurar la alta disponibilidad en su Amazon VPC», cuyo contenido no viene en el texto extraído.

---

## 1. Task Statement 2.1 — Design scalable and loosely coupled architectures

Audiencia: 🔧 🧭

### 1.1 Escalado y elasticidad

| Concepto | Definición según el curso |
|---|---|
| Scaling | Capacidad del sistema de crecer o encogerse según la carga o la demanda |
| Vertical | Agrandar el recurso (instancia más grande) |
| Horizontal | Agregar o quitar recursos (más instancias) |
| Elasticity | Automatización **+ escalado horizontal** para ajustar la capacidad a una demanda que casi nunca es lineal |

AWS ofrece launch configurations y Auto Scaling para sumar recursos cuando sube la demanda y bajar a menos servidores, o incluso a cero, cuando baja. Eso sirve a tres pilares a la vez: performance efficiency, operational excellence y cost optimization.

🎯 **Para el examen:** diferenciar **AWS Auto Scaling** de **Amazon EC2 Auto Scaling**, y conocer los tipos de scaling policies de EC2 Auto Scaling. También conviene probar las aplicaciones **a escala**.

### 1.2 Elegir cómputo, almacenamiento y bases de datos

- Saber cuándo conviene **contenedores, serverless o instancias EC2**, con beneficios y limitaciones.
- HPC en EC2: tipos de instancia y configuraciones específicas; **placement groups** y opciones de redes avanzadas. Lectura sugerida: High Performance Computing Lens del Well-Architected Framework.
- Bases de datos con propósito específico: **DynamoDB** (NoSQL, baja latencia a escala extrema), **RDS** (relacional administrada), **Aurora** (relacional cloud-native), **Redshift** (data warehouse).

| | RDS Read Replica | RDS Multi-AZ |
|---|---|---|
| Beneficio | **Rendimiento** (escala lecturas) **y** disponibilidad | Solo **alta disponibilidad** |
| ¿Se consulta? | Sí | No: el standby no es accesible ni suma rendimiento |

> [!warning] ⚠️ Una Read Replica no reemplaza a una caché
> Consultar una réplica sigue cargando la conexión, autenticación, parseo de SQL y optimización. Para descargar tráfico frecuente: **CloudFront, ElastiCache, DynamoDB Accelerator (DAX)**.

- **RDS Proxy:** hace las aplicaciones más escalables, más resistentes a fallas de la base y más seguras.
- **Edge:** CloudFront, Route 53 y **Global Accelerator** para transmitir datos de forma segura con menos latencia, cifrando, eliminando saltos de red y controlando acceso.

> [!example] 📊 Escenario: transferencia de archivos sin administrar servidores
> El administrador no quiere operar infraestructura, parchar, vigilar disponibilidad ni construir mecanismos para aprovisionar y auditar usuarios. Respuesta: **AWS Transfer Family**, que crea, automatiza y monitorea transferencias y procesamiento de archivos sin código ni infraestructura propios; soporta **hasta tres AZ** con flota redundante y auto scaling.

### 1.3 Microservicios, serverless y desacoplamiento

- **SOA:** componentes reutilizables mediante interfaces de servicio. **Microservicios:** van más allá, con componentes más pequeños y simples que se comunican por **APIs bien definidas**. Patrones: **API-driven, event-driven y data streaming**.
- En sistemas distribuidos la red puede perder datos o meter latencia; cada componente debe fallar sin arrastrar a los demás.
- **Serverless en AWS** = sin aprovisionar ni gestionar infraestructura, escala automática por unidad de consumo, pago por valor, y disponibilidad y tolerancia a fallos incorporadas.

| Servicio | Lo que destaca el curso |
|---|---|
| API Gateway | Escala solo; expone funciones Lambda como métodos de API |
| Lambda | Cómputo serverless orientado a eventos; para escalar, entender **concurrency** |
| SQS | Throughput muy alto si se escalan **horizontalmente productores y consumidores**; HA incorporada |
| SQS / DynamoDB como almacén durable | Separan la recepción de la solicitud de su procesamiento (flujos asíncronos para procesos largos) |
| EventBridge | Integraciones más complejas orientadas a eventos |
| ALB, ECS, EKS, Fargate, SNS, Secrets Manager | Parte del conjunto que el task statement exige conocer |

**Decoupling:** componentes autónomos que ignoran a los demás mientras cumplen su parte del sistema. Hay que distinguir:

| Integración | Característica |
|---|---|
| Síncrona | Ambos componentes deben estar disponibles siempre para que funcione |
| Asíncrona | Un intermediario durable (cola, evento) absorbe diferencias de ritmo y caídas |

> [!example] 📊 Frontend rápido, backend lento
> Si el frontend procesa mucho más rápido que el backend, se intercala una cola (SQS) para que cada lado escale de forma independiente.

---

## 2. Task Statement 2.2 — Design highly available and/or fault-tolerant architectures

Audiencia: 🔧 🧭 👔

### 2.1 HA, FT y DR: tres ideas distintas

| | High availability | Fault tolerance | Disaster recovery |
|---|---|---|---|
| Idea | Mantener el sistema en servicio el mayor tiempo posible, reemplazando rápido lo que falla | Seguir operando **durante** la falla de uno o más componentes | Planificar y ejecutar la recuperación ante un desastre |
| ¿Hay downtime? | Puede haber, breve | No debería haber | Sí, se acota con el plan |
| Ejemplo del curso | Un servidor activo y otro en standby: si cae el primero, failover; los empleados quizá deban reconectarse | Dos servidores **activos** atendiendo: si cae uno, el otro ya está sirviendo | Backups fuera del sitio, plantillas CloudFormation listas para levantar el entorno en AWS |
| Costo relativo | Menor | **Mayor** | Según estrategia |

> [!danger] 🚨 Backups en el mismo edificio
> Si el edificio se daña, el backup también. Guárdelos fuera del sitio (por ejemplo, en la nube) y **ensaye** el DR: el peor momento para improvisar una recuperación es en medio del desastre.

### 2.2 Diseñar contra puntos únicos de falla

- Dimensionar recursos para redundancia de servidores y bases, **y** para replicación, gestión de tráfico y detección de fallas.
- Entender por qué la **infraestructura global** (AZ, Regions) aporta HA y FT.
- Para aplicaciones heredadas que no pueden migrar componentes, saber cómo cumplir igual los requisitos de confiabilidad.
- **Trabajar hacia atrás desde la falla:** tomar cada componente y preguntar qué pasa si cae (servidor web, servidor de aplicación, base de datos), y cómo se mitiga.

### 2.3 RTO, RPO y estrategias

| Métrica | Definición | Pregunta |
|---|---|---|
| **RPO** | Tiempo máximo aceptable desde el último punto de recuperación de datos | ¿Cada cuánto hay que respaldar? |
| **RTO** | Tiempo máximo aceptable entre la interrupción y el restablecimiento | ¿Cuánto puede estar caída la aplicación? |

```
 ACTIVE / PASSIVE                                      ACTIVE / ACTIVE
 Backup & Restore ─► Pilot Light ─► Warm Standby ─►   Multi-site
 RTO/RPO altos                                        RTO/RPO mínimos
 costo bajo                                           costo alto
```

Las estrategias no son excluyentes; hay que saber cómo cada una mueve el RTO y el RPO.

**Capacidades por servicio que hay que conocer:**

| Servicio | Capacidad frente a desastres |
|---|---|
| Amazon S3 | Almacenamiento de objetos de bajo costo y extremadamente durable para backups |
| EFS, FSx | Almacenamiento de archivos: disponibilidad, cifrado y limitaciones |
| RDS Multi-AZ | Failover automático si cae la primaria; saber cuánto dura la indisponibilidad |
| Aurora Global Database | Failover **entre Regions**; saber cuánto tarda |
| DynamoDB global tables | Replicación multi-Region |
| AWS Elastic Disaster Recovery | DR para aplicaciones **on-premises y en la nube** |
| AMIs y EC2 Image Builder | Parte de la estrategia de DR para cargas en EC2 |

### 2.4 Red, automatización e integridad

- Red: route tables, VPC peering, Transit Gateway, Site-to-Site VPN, ubicaciones y gateways de Direct Connect, **Route 53 Resolver**.
- Automatizar despliegues: **Elastic Beanstalk, CloudFormation, OpsWorks**; contenedores con ECS y EKS.
- Vulnerabilidades en infraestructura y código: **Amazon Inspector** y **Amazon CodeGuru**.

### 2.5 Patrones de HA que el curso remarca

| Falla a tolerar | Respuesta |
|---|---|
| Una instancia EC2 o una AZ completa | **Elastic Load Balancer multi-AZ + EC2 Auto Scaling group** (entorno *self-healing*) |
| Una Region completa | **Route 53** con failover routing (o latency-based) y/o **Global Accelerator** |
| Base de datos saturada por conexiones serverless | **RDS Proxy**: pool y reutilización de conexiones; failover de RDS/Aurora hasta **66 %** más rápido; credenciales con Secrets Manager e IAM |

**Observabilidad y mejora continua:** CloudWatch y **AWS X-Ray** para ver qué pasa; **CloudWatch alarms** para acciones automáticas sobre métricas clave; **EventBridge** para reaccionar casi en tiempo real.

> [!example] 📊 Comprehend y Polly en un escenario de HA
> La instructora comenta que **Amazon Polly** vuelve a aparecer en el examen. El caso: muchas solicitudes de TI mal clasificadas por los usuarios retrasan la atención (y en una caída, más). **Amazon Comprehend** clasifica automáticamente los tickets, **Amazon Polly** agrega voz y puede usarse para autoservicio en **Amazon Connect**.

---

## 3. Preguntas de práctica resueltas en el curso

Audiencia: 🔧 🧭

### Pregunta 2.1 — Registros de sensores procesados más de una vez

**Escenario:** una API recibe gran volumen de datos de sensores, los escribe en una cola y luego se procesan para tendencias y pronósticos. Algunos registros se procesan dos veces. ¿Cómo evitar duplicados? Palabras clave: *gran volumen*, *se procesa después de la cola*, *más de una vez*.

| Opción | Veredicto | Razón |
|---|---|---|
| Kinesis Data Streams | ❌ | Puede entregar duplicados por reintentos del productor y del consumidor |
| Kinesis Data Firehose | ❌ | También puede producir duplicados |
| Amazon SNS | ❌ | Es mensajería pub/sub, no una cola donde esperar procesamiento |
| **SQS FIFO queue** | ✅ | Orden estricto *first-in-first-out* y procesamiento **exactly-once**; el mensaje queda hasta que el consumidor lo procesa y lo borra, sin duplicados |

### Pregunta 2.2 — Empresa de juegos con crecimiento exponencial

**Escenario:** clientes sin acceso en varias ocasiones; se busca una solución en la nube que iguale la resiliencia de **varios data centers** y aguante el crecimiento. Palabras clave: *no pueden acceder*, *seguir la demanda*, *múltiples data centers*, *crecimiento*.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| ALB → Auto Scaling group en **una** AZ | ❌ | Sin varias AZ no hay alta disponibilidad |
| Varias EC2 en VPC pareadas en dos AZ | ❌ | El peering comunica VPC, no da HA a la aplicación |
| **NLB → Auto Scaling group en dos AZ** | ✅ | El NLB maneja millones de solicitudes por segundo con latencia ultrabaja; con Auto Scaling absorbe tráfico volátil y, al repartir en varias AZ, queda altamente disponible |
| ALB → Auto Scaling group en dos **Regions** | ❌ | Los targets de un ALB deben estar en una sola Region |

> [!tip] 💡 Lectura de la diferencia
> A y C cambian dos cosas (tipo de balanceador y número de AZ); D cambia AZ por Region. Aislar esas diferencias lleva rápido a la respuesta: la clave es «multi-AZ», y lo de Region es un distractor técnicamente imposible.

---

## 4. Trampas típicas del dominio

> [!danger] 🚨 Errores costosos
> - Creer que **Multi-AZ** escala lecturas (solo da HA; las lecturas se escalan con Read Replicas o caché).
> - Usar Read Replicas como si fueran caché.
> - Elegir Kinesis cuando el requisito es **sin duplicados** y orden (va SQS FIFO).
> - Confundir SNS (notificación) con SQS (cola).
> - Poner un ALB con targets en varias Regions (imposible; multi-Region va con Route 53 o Global Accelerator).
> - Confundir HA (puede haber breve downtime) con FT (sin downtime, más caro).
> - Confundir escalado vertical con horizontal, o EC2 Auto Scaling con AWS Auto Scaling.

---

## 5. Lista de repaso del cierre del dominio

- La nube permite dejar de adivinar capacidad: escalar arriba y abajo, y pagar solo lo usado.
- Diferencia HA, FT y DR; infraestructura global para resiliencia entre AZ y Regions.
- Dos servicios para un entorno *self-healing* ante la caída de una instancia o una AZ: **ELB + EC2 Auto Scaling**. Para entre Regions: Route 53 (failover o latency) o Global Accelerator.
- Estrategias de DR: backup & restore, pilot light, warm standby.
- Monitoreo: saber elegir entre **CloudWatch** y **X-Ray** según el escenario.
- Escalado horizontal vs vertical; EC2 Auto Scaling vs AWS Auto Scaling y los servicios que soporta este último.
- Servicios administrados con resiliencia incorporada: **SQS, Lambda, Fargate**.
- Stateful vs stateless: una aplicación heredada con estado que escribe a disco puede escalar horizontalmente con almacenamiento compartido (**EFS, FSx**). Los contenedores pueden tener estado mediante volúmenes de red desacoplados de su ciclo de vida; la integración **ECS + EFS** da un sistema de archivos elástico, visto como almacenamiento serverless.
- Estado en memoria: **sticky sessions** del ALB atan la sesión de un usuario a un target (por defecto, el ALB enruta cada solicitud de forma independiente).
- Desacoplar con SQS y ELB; descargar tráfico frecuente con ElastiCache, DAX y Read Replicas.

---

## 6. Dónde se ve en esta guía

| Tema del dominio | Tomo |
|---|---|
| Regions, AZ, infraestructura global | [[01-Vision-General-y-Seguridad]] |
| EC2, tipos de instancia, VPC, peering | [[02-Computo-y-Redes]] |
| S3, EFS, FSx, RDS Multi-AZ, Read Replicas, DynamoDB | [[03-Almacenamiento-y-Bases-de-Datos]] |
| ELB, EC2 Auto Scaling, CloudWatch, Route 53 | [[04-Monitoreo-y-Optimizacion]] |
| API Gateway, Lambda, SQS, SNS, serverless | [[05-Backend-Web-Serverless]] |
| Kinesis, streaming | [[06-Analitica-de-Datos-Serverless]] |
| ECS, EKS, Fargate, VPN, Direct Connect, Transfer Family | [[07-Solucion-Hibrida-Contenedores]] |
| CloudFormation, automatización multicuenta | [[08-Gobierno-y-Multicuenta]] |

---

## ✅ Autoevaluación

> [!question]- ¿Qué es elasticidad según el curso?
> Automatización combinada con escalado horizontal para ajustar la capacidad a una demanda variable.

> [!question]- ¿Por qué RDS Multi-AZ no mejora el rendimiento de lectura?
> El standby no es accesible ni atiende consultas; solo existe para failover.

> [!question]- Diferencia práctica entre HA y FT en el ejemplo del servidor.
> HA: activo + standby con failover (posible reconexión breve). FT: dos servidores activos, sin downtime, a mayor costo.

> [!question]- Defina RPO y RTO.
> RPO: tiempo máximo aceptable desde el último punto de recuperación de datos. RTO: tiempo máximo aceptable entre la interrupción y el restablecimiento.

> [!question]- Una app serverless abre y cierra miles de conexiones y agota la memoria de la base. ¿Solución?
> RDS Proxy, que mantiene un pool de conexiones compartidas.

> [!question]- ¿Cuántas AZ soporta AWS Transfer Family según el curso?
> Hasta tres.

> [!question]- Aplicación heredada con estado que escribe a disco y debe escalar horizontalmente. ¿Qué servicio?
> Almacenamiento compartido como Amazon EFS (o FSx).

> [!question]- ¿Qué hacen las sticky sessions del ALB?
> Vinculan la sesión de un usuario a un target específico, en lugar de enrutar cada solicitud de forma independiente.

> [!question]- ¿Qué dos herramientas sirven para detectar vulnerabilidades en infraestructura y código?
> Amazon Inspector y Amazon CodeGuru.

## 🧾 Resumen en 10 líneas

1. Dominio 2 pesa 26 %: escalable/desacoplado y HA/FT.
2. Resiliencia = recuperarse de carga, ataques y fallas de componentes.
3. Elasticidad = automatización + escalado horizontal.
4. Read Replicas escalan lecturas; Multi-AZ solo HA; ninguna reemplaza una caché.
5. Desacoplar con SQS (FIFO si no se admiten duplicados), EventBridge y ELB.
6. Serverless: sin servidores, escala por consumo, pago por valor, HA incorporada.
7. HA tolera breve downtime; FT no, y cuesta más; DR es el plan previo.
8. RPO = cuántos datos perder; RTO = cuánto tiempo caído.
9. ELB multi-AZ + Auto Scaling para fallas de instancia o AZ; Route 53 o Global Accelerator para Regions.
10. RDS Proxy acelera el failover hasta 66 % y protege la base de tormentas de conexiones.

## 📖 Recursos del módulo

- [High Performance Computing Lens](https://docs.aws.amazon.com/wellarchitected/latest/high-performance-computing-lens/scenarios.html)
- [AWS Auto Scaling](https://aws.amazon.com/autoscaling/)
- [¿Qué es Amazon EC2 Auto Scaling?](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)
- [Edge networking con AWS](https://aws.amazon.com/products/networking/edge-networking/)
- [¿Qué es AWS Transfer Family?](https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html)
- [Serverless en AWS](https://aws.amazon.com/serverless/)
- [Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [¿Qué es Amazon SNS?](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [¿Qué es Amazon SQS?](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [Alta disponibilidad y escalabilidad en AWS](https://docs.aws.amazon.com/whitepapers/latest/real-time-communication-on-aws/high-availability-and-scalability-on-aws.html)
- [Tolerancia a fallos y aislamiento de fallos](https://docs.aws.amazon.com/whitepapers/latest/availability-and-beyond-improving-resilience/fault-tolerance-and-fault-isolation.html)
- [¿Qué es AWS Elastic Disaster Recovery?](https://docs.aws.amazon.com/drs/latest/userguide/what-is-drs.html)
- [Conectar su VPC a otras redes](https://docs.aws.amazon.com/vpc/latest/userguide/extend-intro.html)
- [¿Qué es Amazon Route 53?](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [Uso de Amazon RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-proxy.html)

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[16-Dominio-1-Arquitecturas-Seguras|16 · Dominio 1: Arquitecturas Seguras]] · Siguiente: [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento|18 · Dominio 3: Alto Rendimiento ➡]]
