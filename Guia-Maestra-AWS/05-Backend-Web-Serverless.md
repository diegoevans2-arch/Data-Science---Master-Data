---
title: "Tomo 05 — Diseño de un backend web serverless"
tags: [aws, solutions-architect, serverless, event-driven, lambda, api-gateway, sqs, sns, dynamodb]
curso: 2 · Architecting Solutions on AWS
modulo: Diseño de un backend web sin servidor en AWS
audiencias: [tecnico, puente, ejecutivo]
tomo: 05
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# ⚡ Tomo 05 — Diseño de un backend web serverless

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[04-Monitoreo-y-Optimizacion|04 · Monitoreo y Optimización]] · Siguiente: [[06-Analitica-de-Datos-Serverless|06 · Analítica de Datos Serverless ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Es el primer tomo del curso 2, que cambia de registro: ya no se aprende *qué es* cada servicio, sino *cuál elegir y por qué* frente a los requisitos de un cliente. El caso es un servicio de pedidos monolítico que se cae en los peaks de venta y que deja datos inconsistentes cuando falla a mitad de camino. La respuesta es un backend **serverless y event-driven** (API Gateway → SQS → Lambda → DynamoDB → DynamoDB Streams → Lambda → SNS), y el tomo recorre cada decisión con sus alternativas descartadas: Fargate/ECS, EC2, Aurora Serverless, EventBridge, integración directa con DynamoDB.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** migrar un servicio crítico con refactorización *cloud native* en vez de *lift-and-shift*; pagar solo por uso en cargas con demanda en peaks; dejar de mantener servidores web y bases de datos para una sola tabla.
> - **Costo de hacerlo mal:** caídas justo en los eventos de venta (cuando más se factura), pedidos perdidos o a medio registrar entre inventario, logística y contabilidad, hardware sobreaprovisionado que pasa ocioso la mayor parte del tiempo.
> - **Pregunta que responde:** *¿cómo recibo pedidos de forma rápida, sin perder ninguno en un peak y sin que la falla de un sistema aguas abajo arrastre a los demás?*

> [!tip] 💡 Analogía general: el mesón de una pastelería en día de oferta
> En la versión monolítica, el mismo vendedor toma el pedido, cobra, hornea, anota en el libro y avisa a bodega, contabilidad y despacho, y recién entonces atiende al siguiente cliente: la fila se eterniza y si el vendedor se desmaya a mitad de camino, bodega quedó avisada pero contabilidad no. En la versión del curso, un recepcionista (API Gateway) verifica el pedido y lo cuelga en un gancho de comandas (SQS) y le dice al cliente «listo, recibido»; los pasteleros (Lambda) toman comandas del gancho a su ritmo, las anotan en el libro (DynamoDB), y un mensajero (Streams + Lambda) pone cada pedido nuevo en el diario mural (SNS) que leen a la vez bodega, contabilidad y despacho.

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Bienvenido al curso | 2 videos · 1 lectura · 1 encuesta | Formato del curso: 4 clientes, 4 semanas, juego de roles Morgan Willis / Raf Lopes, dónde preguntar |
| Cliente n.º 1 — AnyCompany Online | 3 videos · 1 lectura | Llamada con el cliente, requisitos, recursos (Architecture Center, Well-Architected, capa gratuita) |
| Arquitectura de la solución | 8 videos · 4 lecturas | Elegir cómputo (Lambda vs Fargate vs EC2), base de datos (DynamoDB vs Aurora Serverless), router de eventos (SNS vs EventBridge), desacople (SQS vs DynamoDB directo); 4 exploraciones de consola |
| Presentación al cliente y próximos pasos | 3 videos · 2 lecturas (incluye el reto) · 1 encuesta | Arquitectura final, optimizaciones (DAX, Power Tuning, Layers, Powertools, EventBridge), recorrido de la prueba de concepto |

**Instructores:** Morgan Willis y Raf Lopes, ambos *Principal Cloud Technologists* de AWS Training and Certification. En la semana 1 Morgan hace de Solutions Architect y Raf de cliente.

---

## 1. Cómo funciona este curso

Audiencia: 🧭 👔

El curso 2 del certificado (de 4) se organiza en **cuatro clientes ficticios, uno por semana**. Cada semana sigue el mismo patrón:

1. **Llamada inicial** del Solutions Architect con el cliente: estado actual, necesidad de negocio, requisitos.
2. **Desglose de requisitos** y diagrama de la situación actual.
3. **Diseño pieza a pieza**: para cada componente se comparan servicios candidatos y se hacen llamadas de seguimiento al cliente cuando falta un dato que decide.
4. La arquitectura se arma con **bloques físicos** sobre un escritorio, con cámara cenital (cada servicio = un bloque de construcción).
5. **Presentación** de la solución al cliente.
6. **Optimización**: ambos instructores, ya como arquitectos, discuten cómo llevar la solución al siguiente nivel (al estilo de una revisión Well-Architected).

| Semana | Cliente | Problema | Servicios destacados |
|---|---|---|---|
| 1 | AnyCompany Online (e-commerce de productos de limpieza) | Migrar y hacer resiliente el servicio de pedidos | Lambda, API Gateway, SQS, SNS, DynamoDB |
| 2 | Empresa de software de menús por código QR | Analítica de clickstream para BI | S3, Kinesis Data Firehose, Athena, QuickSight |
| 3 | Compañía de seguros | Cargas de contenedores en modelo híbrido | Direct Connect, ECS, ECS Anywhere, RDS, DMS, NAT gateways, Systems Manager |
| 4 | Agencia de marketing / startup | Estrategia multicuenta y gobierno | Organizations, IAM Identity Center (sucesor de AWS SSO), Service Catalog, CloudTrail, *account vending machine* |

Los ejes de optimización que el curso declara por semana: rendimiento y costo (arquitecturas event-driven), servicios *purpose-built*, confiabilidad y eficiencia operativa (híbrido), seguridad y costo (multicuenta). El ritmo sugerido del certificado completo es de unos **6 meses**.

> [!warning] ⚠️ Advertencias que hacen los instructores
> - El curso **diseña, no construye**: muestra el *porqué* más que el *cómo*. Habrá otras soluciones correctas; a veces se simplifica a propósito.
> - En la vida real, tras el diseño suele venir una **prueba de concepto (POC)**. El curso trae instrucciones de POC solo para las semanas 1 y 2.
> - En AWS rara vez hay una única respuesta correcta: depende del caso de uso, los requisitos y lo que el cliente quiera optimizar.

**Dónde preguntar (video de comentarios):**
- Dudas sobre servicios de AWS o sobre su propio trabajo → **AWS re:Post** (`repost.aws`), comunidad abierta.
- Contenido del curso desactualizado, incorrecto o roto (por ejemplo, un tutorial con pasos que ya no calzan con la consola) → ticket en `support.aws.amazon.com/#/contacts/aws-training`. **No** se usa para problemas de la propia cuenta (el equipo no tiene acceso a ella).
- Certificados o la plataforma de aprendizaje → soporte de la plataforma (Coursera).

**Recursos generales de la lectura introductoria:**
- **AWS Architecture Icons** y herramientas de diagramación para dibujar las arquitecturas propias.
- **Capa gratuita (Free Tier)**: tres tipos de oferta — *12 meses gratis* (desde la activación de la cuenta), *siempre gratis* (mientras se sea cliente) y *pruebas de corta duración* (por plazo o por un límite único). Sobre los límites se paga la tarifa estándar *pay-as-you-go*.
- **AWS Architecture Center**: arquitecturas de referencia, soluciones validadas, prácticas Well-Architected, patrones, íconos.
- **AWS Well-Architected Framework**: base de la optimización de fin de semana; un SA puede tener que conducir una *Well-Architected review*.

---

## 2. El caso: AnyCompany Online y su servicio de pedidos

Audiencia: 🔧 🧭 👔

### 2.1 Situación actual

La empresa vende **productos de limpieza en todo el mundo**. Tiene tres *frontends*: sitio web público, app móvil y un sitio para compradores mayoristas que revenden en sus tiendas. Llevan años moviendo sistemas a AWS, pero mayormente con **lift-and-shift** (sin refactorizar). Ahora quieren **reescribir** el servicio de pedidos, la última pieza grande que sigue on-premises, de forma *cloud native*.

```
 SITUACIÓN ACTUAL (on-premises)
 ┌──────────┐
 │ Web      │──┐     ┌──────────────────────────────────────────┐
 │ App      │──┼────►│ Servidor: web server + servicio de pedidos│
 │ Mayorist.│──┘     │ (monolito: valida, autentica, acepta,    │──► MySQL on-prem
 └──────────┘        │  procesa, guarda, llama aguas abajo)     │   (1 tabla)
                     └──────────────┬───────────────────────────┘
                     llamadas síncronas en serie
                     ▼              ▼                ▼
                Inventario     Fulfillment      Contabilidad   (ya en AWS)
```

- El **pago queda fuera de alcance**: lo resuelve una pasarela de pagos; cuando el pedido llega al backend ya está pagado.
- El enrutamiento lo hace un **servidor web alojado en el mismo servidor** de la aplicación.
- La base es **MySQL on-prem** con **una sola tabla** de pedidos; mantener una instancia completa para tan pocos datos les consume tiempo. Están abiertos a cambiar de motor, siempre que sea **altamente disponible y durable** y más *hands-off*.
- Los servicios aguas abajo (inventario/fulfillment, contabilidad) ya están en AWS y tienen sus propios almacenes de datos.

### 2.2 Dolores

| Dolor | Causa | Efecto |
|---|---|---|
| Lentitud y caídas en peaks | Toda la lógica en un servicio; no escala rápido on-prem | Pedidos que tardan o fallan; hay que sobreaprovisionar hardware |
| Inconsistencia de datos | Componentes **fuertemente acoplados**: si la app cae tras llamar a inventario, fulfillment y contabilidad nunca se enteran | Datos de pedidos inconsistentes, clientes molestos |
| Demanda en peaks (*spiky*) | Ventas online y cupones generan tráfico enorme; en otros momentos, demanda casi cero | Servidores ociosos la mayor parte del tiempo |

### 2.3 Requisitos consolidados

Morgan los resume así tras la llamada:

1. **Escalado gestionado** para cómputo y base de datos → pensar *serverless* primero (y, «alerta de spoiler», casi nada de EC2).
2. **Desacoplar componentes** para maximizar resiliencia → eliminar puntos únicos de falla; aceptación, procesamiento y llamadas aguas abajo deben ser independientes; arquitectura **event-driven**.
3. **Monitoreo y logging centralizados**, idealmente un solo sistema → los servicios serverless se integran de forma nativa con **Amazon CloudWatch** y **CloudWatch Logs**.
4. **Optimizar costo, eficiencia de rendimiento y sobrecarga operativa**, con el costo como principal impulsor.
5. Implícito: las llamadas aguas abajo deben ser independientes entre sí para que no afecten el procesamiento del pedido.

> [!tip] 💡 Qué es *serverless* en el sentido del curso
> Servicios que escalan hacia arriba y hacia abajo con la demanda sin intervención del cliente. En algunos se pueden fijar parámetros o límites de escalado, pero el mecanismo lo gestiona AWS. Serverless tiende a optimizar a la vez costo, rendimiento y carga operativa, lo que alinea casi todos los requisitos.

---

## 3. Cómputo: Lambda frente a Fargate y EC2

Audiencia: 🔧 🧭

Morgan parte de la página `aws.amazon.com/products/compute`, que agrupa los servicios por categoría. Hábito recomendado: revisar ahí porque AWS cambia constantemente.

| Categoría | Servicios | ¿Serverless? |
|---|---|---|
| Instancias | EC2, Lightsail (EC2 simplificado para casos concretos), AWS Batch (procesos por lotes) | No |
| Contenedores | Amazon ECS (orquestación), Amazon EKS (Kubernetes), AWS Fargate (cómputo serverless para contenedores) | Fargate sí |
| Serverless | AWS Lambda | Sí |

### 3.1 Por qué no EC2

EC2 suele ser la opción «por defecto»: VMs a pedido, sistema operativo a elección, control total. Todo el servicio de pedidos podría moverse a EC2 y funcionaría. Pero la lección central es: **que una solución funcione no significa que sea la mejor**; hay que evaluar todas las alternativas. EC2 se descarta por:
- la **sobrecarga operativa** (capacidad, salud de la flota, AZ para tolerancia a fallas, todo a cargo del cliente);
- la **demanda en peaks**, donde Lambda minimiza recursos ociosos en los valles y en EC2 eso cuesta más lograrlo;
- el cliente tiene un caso simple y está dispuesto a reescribir el código.

### 3.2 Fargate (con ECS/EKS): técnicamente válido

Escalado gestionado, logs y métricas a CloudWatch, alto rendimiento, baja carga operativa y costo contenible con **Fargate Spot**. Cumple todo… salvo una pregunta abierta: ¿el cliente quiere contenedores?

> [!example] 📊 Llamada de seguimiento n.º 1 — ¿contenedores o Lambda?
> Raf (cliente) conoce los contenedores a alto nivel, pero **la empresa no los usa** y no tiene las habilidades internas para gestionarlos. Sus desarrolladores y DevOps ya están aprendiendo Lambda y DynamoDB por migraciones previas, y Lambda ya se usa en otras áreas. **Decisión: Lambda.** Fargate funcionaría técnicamente, pero introducir contenedorización en la pila del cliente es un costo que la arquitectura no justifica.

### 3.3 AWS Lambda: lo que hay que saber

- **Cloud native**: creado para la nube y sus beneficios. Serverless, escala automáticamente; puede requerir ajustes de configuración con el tiempo para escalar o ejecutar más rápido.
- **Cada solicitud o evento genera su propia instancia** de la función, que corre en una **microVM Firecracker**; la microVM se apaga sola tras un rato. Sin demanda, no hay instancias ociosas.
- **Duración máxima: 15 minutos.** Procesos más largos necesitan otro servicio de cómputo.
- Cada invocación corre en un **entorno de ejecución** seguro y aislado, que permanece activo un tiempo y luego se apaga.
- El cliente solo responde por **su código**. Lambda gestiona la flota (memoria, CPU, red), la capacidad, el monitoreo y el logging. **No se puede** iniciar sesión en las instancias ni personalizar el SO de los runtimes provistos.
- **Modelo de uso:** se crea una **función**, se sube el código y se elige un **trigger** (evento de otro servicio de AWS o externo, por ejemplo una solicitud HTTP vía Application Load Balancer o API Gateway).
- **Casos de uso típicos:** backends web, IoT, móviles; procesamiento de archivos o datos; procesamiento de streams o mensajes; cualquier cómputo de menos de 15 minutos. Encaja cuando se busca reducir carga operativa, costo o mejorar eficiencia: solo se paga lo que se usa.

| | Lambda | EC2 | Elastic Beanstalk | Fargate |
|---|---|---|---|---|
| Qué gestiona el cliente | Solo el código | SO, red, seguridad, toda la pila, capacidad, salud | Despliegue y escalado asistidos, pero el cliente conserva control y propiedad de las EC2 | Contenedor, CPU/memoria, políticas de red e IAM |
| Escalado | Automático por evento | Del cliente (Auto Scaling) | Asistido | Automático |
| Idle | No existe | Se paga aunque esté ocioso | Se paga | No se gestionan servidores |
| Límite | 15 min por ejecución | — | — | — |

### 3.4 API Gateway como puerta de entrada

Si el cómputo es Lambda, **Amazon API Gateway** es la elección natural para exponerlo:
- El cliente **deja de mantener un servidor web**: se define una API, sus recursos y métodos HTTP, y se reenvían las solicitudes al backend.
- API Gateway puede hacer la **autenticación** de la solicitud y la **validación** básica de formato que hoy hace el servicio de pedidos → **menos código** que migrar.
- Expone el backend **sin exponerlo a la Internet abierta**; se le puede añadir autenticación.
- Servicio totalmente gestionado para crear, publicar, mantener, monitorear y asegurar APIs a cualquier escala. Permite **APIs RESTful** y **APIs WebSocket** (comunicación bidireccional en tiempo real); sirve a cargas serverless, en contenedores y aplicaciones web.
- Se encarga de aceptar y procesar **hasta cientos de miles de llamadas concurrentes**: gestión de tráfico, **CORS**, autorización y control de acceso, **throttling**, monitoreo, versionado de la API.
- **Precio:** sin cuotas mínimas ni costos de arranque; se paga por llamadas recibidas y datos transferidos, con **precios escalonados** que bajan con el volumen.

Logs y métricas de Lambda y API Gateway van a CloudWatch / CloudWatch Logs y se analizan en un solo lugar para todas las instancias de la función.

```
 Bloques al cierre de la sección 3:   [API Gateway] ──► [Lambda]
```

### 3.5 Servicios de contenedores (lectura de referencia)

La gestión de contenedores se divide en **registro** (dónde guardar imágenes), **orquestación** (cuándo y dónde corren) y **cómputo** (motor que los ejecuta).

- **Amazon ECS**: orquestación totalmente gestionada; *serverless por defecto con Fargate* (se elige Fargate como *launch type* en consola o CLI); seguridad y aislamiento con permisos granulares por contenedor; sin plano de control, nodos ni add-ons que administrar; corre en la nube o on-premises.
- **Amazon EKS**: Kubernetes gestionado sin instalar ni operar plano de control o nodos. Ejecuta el plano de control en **varias AZ**, lo escala según carga, reemplaza instancias no saludables, aplica parches y actualizaciones de versión. Integra **ECR** (imágenes), **Elastic Load Balancing**, **IAM** (autenticación) y **VPC** (aislamiento). Compatible con cualquier Kubernetes estándar → migración casi sin cambios de código.
- **AWS Fargate**: contenedores sin servidores ni clústeres de EC2; se especifican CPU, memoria, red e IAM. **Cada tarea tiene su propio límite de aislamiento** (no comparte kernel, CPU, memoria ni ENI). Con *capacity providers* se combinan **Fargate** y **Fargate Spot**: tareas tolerantes a interrupción con descuento sobre capacidad sobrante; si AWS la recupera, la tarea recibe **aviso de 2 minutos**.

> [!tip] 🎯 Para el examen
> - Lambda: máximo **15 minutos**; sin acceso al SO; pago por uso; una instancia por evento.
> - «Menor carga operativa + demanda en peaks + dispuestos a reescribir» → Lambda. «Ya usan contenedores / Kubernetes» → Fargate/ECS/EKS.
> - Fargate Spot: interrupción con aviso de **2 minutos**.

---

## 4. Base de datos: DynamoDB frente a Aurora Serverless

Audiencia: 🔧 🧭

Punto de partida: `aws.amazon.com/products/databases`. Las bases de AWS son **purpose-built**: elegir la adecuada puede ahorrar configuración, mantenimiento e incluso código (si la base trae de fábrica funciones que de otro modo habría que programar). Antes se usaban relacionales para todo porque eran lo que había on-prem; en AWS se puede elegir según el caso y no según las habilidades internas de DBA.

| Tipo | Servicio citado | Uso |
|---|---|---|
| Relacional | Amazon RDS, Amazon Aurora (con opción **Aurora Serverless**) | Aplicaciones tradicionales, esquemas relacionados, SQL |
| Data warehouse | Amazon Redshift | Analítica |
| Key-value | **Amazon DynamoDB** | Alta concurrencia, consultas por clave |
| En memoria | Amazon ElastiCache | Caché |
| Grafos | Amazon Neptune | Relaciones tipo grafo (en vez de forzar un relacional) |

### 4.1 Aurora Serverless

- Aurora es un motor relacional **compatible con MySQL y PostgreSQL** (bajo el paraguas de RDS), diseñado para escala masiva y cargas de nivel empresarial. Compatible con el MySQL actual del cliente.
- Con algunas cargas, **hasta 5× el rendimiento de MySQL y 3× el de PostgreSQL** sin cambios en la mayoría de las aplicaciones.
- Subsistema de almacenamiento distribuido de alto rendimiento que **crece automáticamente** hasta **128 TiB** por volumen de clúster; automatiza clustering y replicación.
- Aurora «normal» no es totalmente gestionado en el sentido de que el cliente planifica y redimensiona instancias o agrega réplicas. **Aurora Serverless v2** es la configuración de **autoescalado bajo demanda**: ajusta capacidad según la demanda, se cobra solo lo consumido; se determinan capacidad mínima y máxima observando la carga. Ideal para bases multi-tenant, distribuidas, dev/test y cargas muy variables e impredecibles.

### 4.2 DynamoDB

- NoSQL **key-value y de documentos**, totalmente gestionado, serverless *de origen* (no como una opción, a diferencia de Aurora). Escala almacenamiento de forma automática: no existen tareas de DBA como agregar tablespaces o clústeres de almacenamiento (Morgan lo cuenta desde su experiencia previa como DBA).
- El foco pasa a los **patrones de uso y el throughput**: capacidad **provisionada** (lecturas/escrituras por segundo fijadas de antemano) u **on-demand** (escala con la demanda). Para un patrón *spiky* como el del cliente → **on-demand**.
- **Hay que conocer los patrones de acceso de antemano.** No hay «base de datos con varias tablas» ni **joins** nativos: las tablas son independientes y se modelan con **índices** que permitan las consultas necesarias. Si se necesitan joins o SQL complejo, DynamoDB probablemente no es la opción.
- Cifrado en reposo incluido; rendimiento rápido y predecible; capacidad ajustable con mínima degradación.
- Componentes: **tabla** (colección de ítems) → **ítem** (colección de atributos) → **atributo**. Claves primarias identifican cada ítem; **índices secundarios** dan flexibilidad de consulta; **DynamoDB Streams** captura modificaciones.

> [!example] 📊 Llamada de seguimiento n.º 2 — ¿qué consultas hacen?
> Raf: crear, leer, actualizar y eliminar pedidos **por ID de pedido**, y recuperar todos los pedidos **de un ID de cliente**. Nada de SQL complejo entre tablas: la tabla es solo un lugar donde guardar pedidos. **Decisión: DynamoDB.**

**Justificación de la decisión:**
1. El modelado de datos es trivial (dos patrones de acceso claros).
2. El cliente no necesita las funciones empresariales de Aurora.
3. DynamoDB es más simple de empezar y de operar en el tiempo.
4. **El problema de las conexiones**: con Lambda a gran escala, cada entorno de ejecución abre su propia conexión (100 entornos = 100 conexiones). En un evento de ventas eso puede inundar una base relacional. Con Aurora habría que poner **Amazon RDS Proxy** entre Lambda y la base → otro servicio que gestionar. Con DynamoDB el problema no existe.

> [!note] ➕ Nota del editor
> La transcripción automática dice que «con Aurora Serverless este problema se ha resuelto», pero el razonamiento siguiente (hay que añadir RDS Proxy) indica que el sentido es el contrario: el problema de conexiones **no** desaparece con Aurora Serverless y requiere RDS Proxy.

### 4.3 Amazon RDS Proxy (lectura)

- Las aplicaciones **agrupan y comparten conexiones** (pool) → mejor escalado sin la sobrecarga de CPU y memoria de abrir una conexión nueva cada vez.
- Mayor resiliencia: ante falla de la base se conecta automáticamente a una instancia **standby** preservando las conexiones de la aplicación.
- Permite **autenticación IAM** hacia la base y guardar credenciales en **AWS Secrets Manager**.
- Absorbe aumentos impredecibles de tráfico; se puede **limitar el número de conexiones** creadas para evitar sobresuscripción. Las conexiones que no pueden servirse de inmediato se **encolan o se estrangulan** (*throttle*): sube la latencia pero la aplicación no falla bruscamente.

```
 Bloques al cierre de la sección 4:   [API Gateway] ──► [Lambda] ──► [DynamoDB]
```

> [!tip] 🎯 Para el examen
> - Lambda + base relacional a gran escala → **RDS Proxy** (pool de conexiones).
> - Patrones de acceso conocidos, sin joins, alta concurrencia → **DynamoDB**; demanda en peaks → **on-demand**.
> - Aurora: MySQL/PostgreSQL, hasta 5×/3×, almacenamiento hasta **128 TiB**, Serverless v2 para cargas variables.

---

## 5. Arquitectura event-driven: SNS frente a EventBridge

Audiencia: 🔧 🧭

### 5.1 El problema que queda

Codificar en la misma Lambda las llamadas a los servicios aguas abajo reproduciría el acoplamiento actual: cualquier cambio en cómo se distribuyen los mensajes obligaría a tocar el código de procesamiento de pedidos (y a arriesgar errores). La función debe limitarse a **recibir, procesar y guardar**; la distribución se hace capturando el evento de escritura y publicándolo con un modelo **publish/subscribe**.

### 5.2 Conceptos (página `aws.amazon.com/event-driven-architecture`)

- Una **arquitectura event-driven** usa eventos para invocar y comunicar servicios desacoplados; es común en microservicios.
- Un **evento** es un cambio de estado o una actualización (por ejemplo, meter un artículo al carrito). Puede **llevar el estado** (artículo, precio, dirección) o ser solo un **identificador** (aviso de que se despachó un pedido).
- Tres componentes: **productor** → **router** (filtra y envía) → **consumidor**. Productor y consumidor están desacoplados: se escalan, actualizan y despliegan de forma independiente.
- Lambda encaja de forma natural: no corre todo el tiempo, se ejecuta en respuesta a eventos.

### 5.3 Comparación

| Criterio | Amazon SNS | Amazon EventBridge |
|---|---|---|
| Modelo | Pub/sub con **temas** (*topics*); uno a muchos (*fan-out*) | **Bus de eventos** serverless con **reglas** |
| Cuándo se recomienda | Mensajes de **alto throughput o baja latencia** entre aplicaciones/microservicios; fan-out muy alto (**miles o millones** de endpoints) | Reaccionar a eventos de **aplicaciones SaaS** o de servicios de AWS |
| Throughput | Casi ilimitado | Limitado al lanzamiento (ver cuotas), ampliable a solicitud |
| Latencia típica | **< 30 ms** | **≈ 0,5 s** |
| Formato | No estructurado, cualquier formato | Estructura JSON definida |
| Filtrado | Por **atributos del mensaje** (actualización posterior: también por *payload*) | **Reglas** sobre todo el cuerpo del evento (*event patterns*), filtrado mucho más profundo |
| Destinos | 6 tipos: Lambda, SQS, HTTP/S, SMS, push móvil, email (en consola también Kinesis Data Firehose) | **Más de 15** servicios de AWS (Lambda, SQS, SNS, Kinesis Data Streams, Kinesis Data Firehose…), destinos API HTTP, buses de otras cuentas |
| Fuentes | Más de 30 servicios de AWS publican en SNS (EC2, S3, RDS…) | Ingesta automática de eventos de **más de 90** servicios de AWS; **único** integrado directamente con socios SaaS de terceros |
| Extra | — | **Schema registry** (título, formato, reglas de validación del evento) |

**EventBridge en detalle:** recibe un evento (indicador de cambio en el entorno) y aplica una **regla** para enrutarlo a un **target**; las reglas casan por **patrón de evento** o por **programación** (*schedule*). Ejemplo: cuando una EC2 pasa de *pending* a *running*, una regla envía el evento a una Lambda. Todo evento pertenece a un **event bus**; cada regla está ligada a un único bus. La cuenta trae un **bus por defecto** (eventos de servicios de AWS) y se pueden crear **buses personalizados** para enviar o recibir eventos entre cuentas o regiones.

**SNS en detalle:** los *publishers* (productores) se comunican de forma **asíncrona** con los *subscribers* (consumidores) a través de un **tema**, punto de acceso lógico y canal de comunicación. Admite filtrado opcional para que solo algunos suscriptores reciban ciertos mensajes.

> [!example] 📊 Decisión: SNS
> Ambos son serverless, desacoplan y cumplen técnicamente. Pero los tres consumidores (inventario/fulfillment, contabilidad y el tercero aguas abajo) reciben **el mismo mensaje**, hay **un solo tipo de mensaje** y el cliente **controla su forma**: el filtrado profundo y el schema registry no aportan. **SNS gana por costo y simplicidad.** EventBridge es más potente y complejo.

### 5.4 Cómo llega el pedido a SNS: DynamoDB Streams + segunda Lambda

- **DynamoDB Streams** captura una **secuencia ordenada en el tiempo** de las modificaciones a nivel de ítem y la guarda en un log **hasta 24 horas**; las aplicaciones ven el ítem antes y después del cambio, casi en tiempo real.
- Garantías: **cada registro aparece exactamente una vez**; para cada ítem, los registros aparecen **en el mismo orden** que las modificaciones reales.
- Cifrado en reposo; funciona de forma **asíncrona** (no afecta el rendimiento de la tabla); se habilita al crear la tabla (CLI/SDK) o después, y se puede deshabilitar o reconfigurar. Datos con más de 24 h pueden recortarse en cualquier momento.
- Una **segunda Lambda** con el stream como trigger recibe el ítem nuevo en el payload y llama a la API de SNS para publicar.

**Beneficio clave:** el mecanismo de envío queda **fuera del código de negocio**. Si mañana aparece un servicio nuevo (por ejemplo analítica), basta con **suscribir su endpoint HTTPS al tema**, sin cambiar código existente.

```
 Bloques al cierre de la sección 5:
 [API Gateway] ─► [Lambda 1] ─► [DynamoDB] ─► (Stream) ─► [Lambda 2] ─► [SNS] ─► 3 servicios
```

> [!tip] 🎯 Para el examen
> - Mismo mensaje a muchos consumidores → **SNS fan-out**. Filtrado complejo, SaaS, mensajes heterogéneos → **EventBridge**.
> - SNS < 30 ms vs EventBridge ≈ 0,5 s. EventBridge: +90 servicios fuente, +15 destinos.
> - DynamoDB Streams: retención **24 h**, *exactly once* por registro, orden por ítem.

---

## 6. Desacople asíncrono: SQS y el patrón storage-first

Audiencia: 🔧 🧭

### 6.1 El problema de latencia

Aunque la lógica ya está separada de las llamadas aguas abajo, la **aceptación** sigue siendo **síncrona**: el usuario espera a que se valide, procese y guarde todo. En escalados, el backend empieza a **rechazar solicitudes** por saturación. Solución: **arquitectura asíncrona** con el patrón **storage-first**.

```
 SÍNCRONO (antes)                          STORAGE-FIRST (después)
 Cliente ─► API ─► Cómputo ─► Almac.       Cliente ─► API ─► Almacenamiento ──► "recibido" (rápido)
        ◄─────── respuesta ◄─────┘                              │
   (el usuario espera todo)                                     └─► Cómputo lee y procesa (asíncrono)
```

Por qué API Gateway lo hace posible:
1. Puede **autenticar, validar y transformar payloads** antes del cómputo, más cerca del cliente.
2. Tiene **integraciones directas con servicios de AWS**: escribir un ítem en DynamoDB, encolar en SQS o publicar en SNS **sin una Lambda de por medio**.

### 6.2 Alternativas para el búfer

| Opción | A favor | Condición |
|---|---|---|
| **DynamoDB directo** desde API Gateway | Ya se guardan ahí los pedidos; se podría **eliminar la primera Lambda** | Solo sirve si la Lambda **no tiene lógica de negocio** |
| **Amazon SQS** entre API Gateway y Lambda | Cola gestionada; SQS es fuente de eventos nativa de Lambda | Mantiene la Lambda con la lógica |

Ambas desacoplan la API del backend, reducen la latencia, son serverless y aumentan la resiliencia: aunque Lambda llegue a los límites fijados por el cliente, los pedidos quedan almacenados esperando, **no se pierde ninguno**.

> [!example] 📊 Llamada de seguimiento n.º 3 — ¿hay lógica de negocio?
> Raf (en el laboratorio probando unos guantes nuevos) confirma que el servicio tiene **lógica de negocio compleja** más allá de validar y mover datos. **Decisión: SQS** entre API Gateway y Lambda; la Lambda no puede eliminarse.

**Cuándo usar este patrón (lectura, guía de *microservices integration patterns*):**
- se busca una arquitectura débilmente acoplada;
- no todas las operaciones deben completarse en una única transacción; algunas pueden ser asíncronas;
- el sistema aguas abajo no soporta el TPS entrante: los mensajes se encolan y se procesan según disponibilidad.

**Desventaja:** aunque el llamador recibe respuesta, parte de la transacción puede seguir procesándose aguas abajo.

> [!note] ➕ Nota del editor
> La lectura del curso dice que la desventaja es que «las acciones de la transacción comercial son síncronas»; por el contexto, lo que se quiere decir es lo contrario (son **asíncronas**, y por eso el llamador recibe respuesta antes de que todo termine).

### 6.3 Amazon SQS: parámetros que hay que dominar

SQS es una cola de mensajes totalmente gestionada para desacoplar y escalar microservicios, sistemas distribuidos y aplicaciones serverless; elimina la gestión de *middleware* orientado a mensajes; envía, guarda y recibe a prácticamente cualquier volumen sin perder mensajes ni exigir que el otro lado esté disponible.

| Parámetro | Qué es | Valores citados |
|---|---|---|
| Tipo de cola | **Standard** (orden *best-effort*) o **FIFO** (orden estricto) | — |
| **Visibility timeout** | Tiempo en que un mensaje ya recibido por un consumidor queda **invisible** para los demás (evita procesamiento duplicado entre varios consumidores) | — |
| **Message retention period** | Cuánto se retiene un mensaje no procesado | **1 min a 14 días; por defecto 4 días** |
| **Delivery delay** | Retraso antes de entregar un mensaje nuevo (*delay queues*) | — |
| **Maximum message size** | Tamaño máximo | **1 KB a 256 KB** |
| **Receive message wait time** | Espera máxima del *polling* hasta que haya mensajes | **0 a 20 s** |
| Content-based deduplication | IDs de deduplicación generados a partir del cuerpo | FIFO |
| High throughput FIFO | Alto rendimiento para FIFO; ajusta ámbito de deduplicación y límite de throughput | FIFO |
| Redrive allow policy | Qué colas origen pueden usar esta como **dead-letter queue** | — |
| Access policy | **Resource policy**: quién envía y quién recibe | JSON |

- **Mensajes grandes:** SQS no es para mensajes grandes; se envía una **referencia** (índice, clave) a donde está el contenido (otra base, S3…).
- **SQS vs SNS:** en SQS el mensaje **permanece en la cola hasta ser procesado y eliminado** (los consumidores hacen *polling*); en SNS el mensaje se **empuja** al suscriptor y si lo recibió bien o no depende del suscriptor.
- En la consola de la cola: suscripción a temas SNS (SNS puede escribir en la cola), **triggers de Lambda**, monitoreo en CloudWatch (por ejemplo, muchos mensajes entrando y pocos *deleted* sugiere un bug: el código no borra tras procesar).

**Short polling vs long polling:**

| | Short polling | Long polling |
|---|---|---|
| Activación | Wait time = 0 | Wait time > 0 (máx. **20 s**) |
| Servidores consultados | Un **subconjunto** aleatorio ponderado | **Todos** |
| Respuesta | Puede no devolver todos los mensajes (con < 1.000 mensajes, una solicitud posterior los devuelve) | Espera hasta que haya al menos un mensaje; devuelve en cuanto están disponibles |
| Costo | Más respuestas vacías y falsas vacías | **Menos costo**: reduce respuestas vacías y falsas vacías |
| Recomendación del curso | Colas de alto volumen | Colas de bajo volumen |

> [!warning] ⚠️ Para qué sirve el visibility timeout
> Con cinco EC2 sondeando la misma cola, sin visibility timeout dos de ellas podrían procesar el mismo pedido. Al reclamar un mensaje empieza el timeout y nadie más lo ve. El consumidor debe **borrar** el mensaje tras procesarlo; si no, reaparece.

> [!tip] 🎯 Para el examen
> SQS: 256 KB, retención 4 días por defecto / 14 máx., long polling máx. 20 s, visibility timeout contra duplicados, DLQ vía redrive. SNS FIFO solo admite suscriptores SQS.

---

## 7. La solución presentada al cliente

Audiencia: 🔧 🧭 👔

Morgan la plantea como **servicio greenfield** («olvide cómo se hace hoy»). Recordatorio previo: el diseño inicial cambiará cuando aparezcan requisitos nuevos; el siguiente paso real sería una POC.

```
 ┌─────────────┐
 │ Frontends   │  (web, app, mayoristas)
 └──────┬──────┘
        ▼
 ┌──────────────┐ autentica + valida   ┌──────────┐ polling    ┌───────────┐  PutItem  ┌───────────┐
 │ API Gateway  │─────────────────────►│ SQS queue│──────────► │ Lambda 1  │──────────►│ DynamoDB  │
 └──────────────┘ responde de inmediato└──────────┘ (integrado)│ (lógica)  │           │  orders   │
                                                               └───────────┘           └─────┬─────┘
                                                                                             │ Stream (NEW_IMAGE)
                                                                                             ▼
                        ┌─────────────┐  ┌──────────────┐  ┌────────────┐   Publish  ┌───────────┐
                        │ Inventario/ │◄─│              │◄─│            │◄───────────│ Lambda 2  │
                        │ Fulfillment │  │  SNS topic   │  │ fan-out    │            └───────────┘
                        │ Contabilidad│◄─│              │  └────────────┘
                        └─────────────┘  └──────────────┘
           Logs y métricas de todos los servicios ──► CloudWatch / CloudWatch Logs
```

| Requisito | Cómo se cumple |
|---|---|
| Escalado gestionado | Todo serverless; se pueden fijar límites/tasas (concurrencia de Lambda, throughput de DynamoDB), mucho más *hands-off* que EC2 |
| Desacople y resiliencia | SQS separa API de cómputo (si Lambda llega a su límite, los mensajes esperan; se pueden subir los límites para ponerse al día); Streams + SNS separan la distribución de la lógica |
| Monitoreo centralizado | Integración nativa con CloudWatch y CloudWatch Logs, sin configuración extra |
| Costo / rendimiento / operación | Servicios que escalan hacia adentro cuando no se usan; sin servidores que mantener |

El cliente queda tranquilo con no recibir más llamadas por caídas en días de venta.

---

## 8. Llevar la arquitectura al siguiente nivel (optimización)

Audiencia: 🔧 🧭

Raf, ahora como SA, y Morgan revisan la arquitectura:

| Mejora | Qué es | Cuándo / por qué |
|---|---|---|
| **DynamoDB Accelerator (DAX)** | Caché en memoria totalmente gestionada y altamente disponible para DynamoDB, entre Lambda y la tabla | Hasta **100×** de mejora (de milisegundos a **microsegundos**), incluso con millones de solicitudes/s. Tiene costo: introducirla **solo cuando se haya demostrado** que el rendimiento no basta. Alternativa: más RCU o **remodelar la tabla con índices**. Probar con y sin DAX (A/B, pruebas de estrés con infraestructura como código) |
| **AWS Lambda Power Tuning** | Herramienta open source (máquina de estados en **Step Functions**) que invoca la función con distintas memorias (**128 MB a 10 GB**) y sugiere la mejor | Contraintuitivo: más memoria (y CPU) cuesta más por ms pero puede **bajar el costo total** porque la función dura menos y Lambda cobra por tiempo. Estrategias: **costo, velocidad o equilibrada**; recibe el **ARN** de la función; agnóstica al lenguaje; soporta invocaciones entre regiones y ejecución en paralelo; grafica costo y velocidad promedio por configuración |
| **Lambda Layers** | Código compartido (bibliotecas comunes) referenciado por varias funciones | Cambiar la biblioteca una vez, no en cada función; **despliegues más rápidos** (se sube menos código) |
| **Reutilización del entorno de ejecución** | Inicializar fuera del *handler* (clientes del SDK, conexiones a BD) y cachear estáticos en **/tmp** | Invocaciones siguientes en la misma instancia reutilizan recursos → menos tiempo y costo. **No** guardar datos de usuario, eventos ni información sensible en el entorno (fugas entre invocaciones); si hay estado mutable, funciones o versiones separadas por usuario |
| **SNS → EventBridge** | Poner EventBridge entre la Lambda y los consumidores; SNS puede ser uno de sus destinos, junto a una cola SQS, etc. | Si crece la necesidad de **filtrado profundo**, mensajes no uniformes o integración con SaaS de AWS Partners |
| **AWS Lambda Powertools** | Utilidades para adoptar buenas prácticas: **tracing (X-Ray)**, **logging estructurado**, **métricas personalizadas**, idempotencia, batching | Optimizar salida y **retención de logs** en CloudWatch Logs |

Según la lectura, DAX **no exige cambiar la lógica de la aplicación** (es compatible con las llamadas a la API de DynamoDB), gestiona por sí mismo la invalidación de caché, el poblamiento y el clúster, y **corre dentro de una VPC**, con acceso controlado mediante *security groups*.

> [!note] ➕ Nota del editor
> Como DAX vive en una VPC, la Lambda que lo consulte tiene que configurarse para ejecutarse en esa VPC (la opción *Enable VPC* que se ve en la exploración de Lambda).

Recursos de la lectura: optimización de costos de aplicaciones serverless (blog de cómputo) y la **Serverless Applications Lens** del Well-Architected Framework.

---

## 9. Demos y laboratorios

Audiencia: 🔧

### 9.1 Exploración de AWS Lambda (consola)

**Objetivo:** recorrer la creación y configuración de una función. **Servicios:** Lambda, API Gateway, CloudWatch Logs, IAM.

1. Lambda → **Create function** → nombre y **runtime**. Runtimes listados: .NET 6, .NET Core, Go, Java, Node.js, Python, Ruby (varias versiones), más **custom runtime** si el lenguaje no está. Se elige **Python 3.9**. Al ser serverless, el runtime ya está presente en el entorno.
2. **Execution role**: los permisos de la función. Si el código llama al SDK (publicar en SNS, poner un objeto en S3), hace falta un **IAM role** con esos permisos, que Lambda **asume** en cada ejecución. Se crea uno nuevo con permisos básicos de Lambda.
3. **Advanced settings → Enable VPC**: correr la función dentro de una VPC para alcanzar recursos privados (por ejemplo, un RDS en subnet privada). No se usa en la demo.
4. *Function overview*: **triggers** a la izquierda, función y **layers** al centro, **destinations** a la derecha.
5. **Add trigger**: lista larga de fuentes (API Gateway, Application Load Balancer, CloudFront, CloudWatch Logs, CodeCommit, DynamoDB, EventBridge, Kinesis…), además de fuentes externas/SaaS vía EventBridge. Se elige **API Gateway → Create a new API → HTTP API** (integra directo con Lambda; no se necesitan transformaciones) con seguridad **Open** (cualquiera puede invocarla; solo para la exploración). Al añadir, la API se crea en API Gateway y aparece un **API endpoint**.
6. Pestaña **Code**: editor para funciones pequeñas. El **handler** recibe `event` (payload de la solicitud) y `context` (datos de esa ejecución). Se deja el ejemplo «Hello from Lambda!».
7. Pestaña **Configuration**: **variables de entorno** (pares clave-valor accesibles al código), **tags** (facturación, filtrado, seguimiento de costos), **Monitoring and operations** (logs y métricas habilitados hacia CloudWatch Logs: tanto los del runtime como los de la aplicación).
8. Pestaña **Test**: crear evento con la plantilla *hello-world* → **Test** → respuesta «Hello from Lambda!».

### 9.2 Exploración de DynamoDB

1. **Create table** (en DynamoDB se crean tablas independientes, no bases con tablas). Nombre de ejemplo.
2. **Partition key** = esencialmente la clave primaria (identificador único), p. ej. `userid`. **Sort key** opcional: cuando la partition key no es única, *partition key + sort key = único*.
3. Vocabulario: **ítems** ≈ filas, **atributos** ≈ columnas.
4. Modo por defecto **Provisioned** con **5 RCU** (*read capacity units*) y **5 WCU** (*write capacity units*). Con **Customize settings**:
   - **Table class**: **DynamoDB Standard** (uso general) o **DynamoDB Standard-IA** (acceso infrecuente, menor costo de almacenamiento).
   - Capacidad **On-demand**: el throughput escala solo según el consumo de RCU/WCU.
5. **Secondary indexes**: **local** o **global**, para consultar atributos que no son la clave primaria sin hacer un **scan** completo.
6. **Encryption at rest** con AWS KMS: *Owned by Amazon DynamoDB* (por defecto), **AWS managed key** (en la cuenta, gestionada por AWS KMS) o **customer managed key**.
7. Tags opcionales → **Create table** (segundos; estado *Active*).
8. Pestaña **Monitor**: métricas de CloudWatch (uso de lectura/escritura, latencias de *query* y *scan*). Un scan revisa cada ítem porque el atributo no está indexado ni es clave → la mayor latencia.
9. **Exports and streams → Enable DynamoDB Streams**, tipo de vista: *Key attributes only*, *New image*, *Old image* o *New and old images*. Se elige **New image**.
10. **Global tables**: réplicas de la tabla en otras regiones, sincronizadas entre sí.

### 9.3 Exploración de Amazon SNS

1. **Create topic** (un tema es un canal de mensajes que reparte a todos los endpoints suscritos).
2. Tipo:
   - **FIFO**: orden estrictamente preservado, entrega **exactamente una vez**, hasta **300 publicaciones por segundo**; protocolo de suscripción solo **SQS**.
   - **Standard**: orden *best-effort*, mucho más throughput; admite SQS, Lambda, HTTP/S, SMS, email, push móvil. Se elige Standard (el orden no importa: el pago ya ocurrió).
3. **Cifrado en tránsito** habilitado por defecto; **cifrado del lado del servidor** (en reposo) hay que activarlo: cifra al recibir y descifra antes de entregar.
4. **Access policy** (resource policy): quién puede **publicar** (solo el dueño, todos, cuentas específicas) y quién puede **suscribirse** (además, solicitantes con ciertos endpoints). Vista previa JSON o modo **Advanced** para editarlo.
5. Suscripciones posibles: Kinesis Data Firehose, SQS, Lambda, email, HTTPS, SMS.

### 9.4 Exploración de Amazon SQS

1. **Create queue** → **Standard** o **FIFO**; nombre.
2. Repaso de **maximum message size** (1–256 KB), **retention** (1 min – 14 días; se fija 14), **receive message wait time** (0–20 s: long polling para colas de bajo volumen, short para alto volumen), **visibility timeout**.
3. **Access policy**: quién envía y quién recibe.
4. Tras crearla: suscribir la cola a temas SNS, configurar **Lambda triggers**, ver monitoreo en CloudWatch.

### 9.5 Ejercicio 1 — prueba de concepto serverless (recorrido)

**Objetivo:** construir en la propia cuenta, de forma opcional, la arquitectura de la semana y verificarla con un correo. **Región:** us-east-1 (N. Virginia). **Servicios:** IAM, SQS, DynamoDB, Lambda, SNS, API Gateway.

**Paso 1 — Políticas y roles IAM** (políticas *customer managed*; las *AWS managed* las actualiza AWS cuando salen funciones nuevas, las propias hay que editarlas a mano):

| Política | Acciones |
|---|---|
| `Lambda-Read-SQS` | `DeleteMessage`, `ReceiveMessage`, `GetQueueAttributes`, `ChangeMessageVisibility` |
| `Lambda-Write-DynamoDB` | `PutItem`, `DescribeTable` |
| `Lambda-DynamoDBStreams-Read` | `GetShardIterator`, `DescribeStream`, `ListStreams`, `GetRecords` |
| `Lambda-SNS-Publish` | `Publish`, `GetTopicAttributes`, `ListTopics` |

| Rol | Políticas |
|---|---|
| `APIGateway-SQS` | Política AWS managed para enviar logs a CloudWatch Logs (el permiso de envío a la cola se da en la access policy de la cola) |
| `Lambda-SQS-DynamoDB` | `Lambda-Read-SQS` + `Lambda-Write-DynamoDB` + `AWSLambdaBasicExecutionRole` (logs) |
| `Lambda-DynamoDBStreams-SNS` | `Lambda-DynamoDBStreams-Read` + `Lambda-SNS-Publish` + `AWSLambdaBasicExecutionRole` |

Se copian los **ARN** de los roles a un editor de texto para los pasos siguientes.

**Paso 2 — Cola SQS** `POC-Queue` (Standard, valores por defecto). En la **access policy**: quién envía = ARN del rol `APIGateway-SQS`; quién recibe = ARN del rol `Lambda-SQS-DynamoDB`. Copiar el ARN de la cola. (Morgan recomienda, para el examen, entender cada parámetro por defecto de la cola.)

**Paso 3 — Tabla DynamoDB** `orders`, partition key `orderID`, valores por defecto.

**Paso 4 — Lambda 1** `POC-Lambda-1`, *Author from scratch*, Python 3.9, rol existente `Lambda-SQS-DynamoDB`. Trigger: **SQS → POC-Queue**. Código provisto: importa **Boto3** (SDK de Python), crea el cliente y el objeto tabla `orders`, recorre los `Records` del evento, toma el cuerpo del mensaje como payload y hace `table.put_item` con un **UUID** como ID del pedido (con un par de `print` para verlo en los logs). **Deploy**, evento de prueba que imita la estructura que SQS entrega a Lambda → **Test** → verificar en DynamoDB → *Explore table items*.

**Paso 5 — Activar Streams** en `orders` con **New image** (necesario: el código de la segunda Lambda lee ese atributo). *Key attributes only* sirve cuando el cuerpo es grande y el payload vive en otro lugar (S3, otra tabla).

**Paso 6 — Tema SNS** `POC-Topic`, **Standard** (para poder suscribir email). Access policy en modo **Advanced**: reemplazar el principal `*` (cualquiera puede publicar) por el **ARN del rol** de la segunda Lambda. Crear **suscripción email** con la dirección propia (SMS depende del país; el ejemplo funciona para destinos de EE. UU.) y confirmarla. Copiar el ARN del tema.

**Paso 7 — Lambda 2** `POC-Lambda-2`, Python 3.9, rol `Lambda-DynamoDBStreams-SNS`. Trigger: **DynamoDB → tabla orders**. Código: cliente SNS; por cada registro, si el evento es **INSERT** toma la **NewImage** y llama a `client.publish` con `TargetArn` (ARN del tema), `Message` y `MessageStructure` = JSON. Pegar el ARN del tema y **Deploy**.

**Paso 8 — API Gateway (REST API)** `POC-API` (*New API*):
1. Recurso `order`; método **POST** (un frontend enviaría un POST a `/order`).
2. Tipo de integración **AWS Service** (integración directa con servicios), región `us-east-1`, servicio **SQS**, subdominio en blanco, método HTTP **POST**, **Path override** = `<número-de-cuenta>/<nombre-de-cola>` (el final del ARN cambiando `:` por `/`), **Execution role** = ARN de `APIGateway-SQS` → Save.
3. **Integration Request → HTTP Headers**: `Content-Type` mapeado desde `'application/x-www-form-urlencoded'` (con comillas simples).
4. **Mapping templates**: *request body passthrough* = **Never**; plantilla para `application/json` escrita en **VTL** que transforma el payload al lenguaje de la API de SQS:

```
Action=SendMessage&MessageBody=$input.body
```

5. Guardar (insisten: es fácil olvidarlo) → **Test** con un cuerpo de pedido (artículo «guantes de látex», ID de cliente 12345) → los logs muestran el envío a la cola y **status 200**.

**Verificación y troubleshooting por tramos:**
1. ¿API Gateway → SQS devuelve 200? (logs de la prueba).
2. ¿El ítem está en DynamoDB? Si no: la Lambda 1 no lee de SQS o no escribe en la tabla (casi siempre **permisos**).
3. ¿Llegó el correo? Si no: ¿la Lambda 2 lee del stream?, ¿el stream está bien configurado?, ¿tiene permiso para publicar en SNS?, ¿el email está bien escrito y confirmado?

**Limpieza:** borrar tabla (y con ella el stream), ambas Lambdas, el tema SNS, la API y la cola. **Reto sugerido:** rehacer todo con **AWS CLI** o **CloudFormation** (infraestructura como código) en lugar de la consola.

> [!danger] 🚨 Errores comunes en esta POC
> - Access policy de SNS dejada con principal `*`: cualquiera podría publicar.
> - Habilitar Streams con un tipo de vista distinto de *New image*: el código de la Lambda 2 falla al buscar `NewImage`.
> - Olvidar **Save** en la integración o en la plantilla VTL de API Gateway.
> - Elegir tema FIFO y luego intentar suscribir un email (FIFO solo acepta SQS).
> - No limpiar recursos al terminar.

El **Ejercicio 1** («Construyendo una prueba de concepto para una solución sin servidor») tiene instrucciones paso a paso en un HTML externo (enlace en Recursos). Las dos encuestas del módulo (previa y de mitad de curso, en Qualtrics) no tienen contenido de estudio.

---

## 10. Aplicación de ejemplo: estado al cierre del módulo

Audiencia: 🔧 🧭

El servicio de pedidos de AnyCompany Online pasó de un monolito síncrono on-premises con MySQL a seis bloques serverless:

| # | Bloque | Rol | Reemplaza a |
|---|---|---|---|
| 1 | Amazon API Gateway | Puerta de entrada; autenticación y validación; integración directa con SQS | Servidor web + parte del código del monolito |
| 2 | Amazon SQS | Búfer storage-first; desacopla API y cómputo | — (antes era síncrono) |
| 3 | AWS Lambda 1 | Lógica de negocio del pedido; guarda en DynamoDB | Servicio de pedidos en VMs |
| 4 | Amazon DynamoDB (on-demand) | Tabla de pedidos | MySQL on-prem |
| 5 | DynamoDB Streams + Lambda 2 | Captura del INSERT y publicación | Llamadas en serie desde el monolito |
| 6 | Amazon SNS | Fan-out a inventario/fulfillment y contabilidad | Llamadas acopladas a cada servicio |

```
 Frontends ─► API Gateway ─► SQS ─► Lambda 1 ─► DynamoDB ─► Streams ─► Lambda 2 ─► SNS ─┬─► Inventario/Fulfillment
                                                                                        ├─► Contabilidad
                                                                                        └─► (futuro: analítica, solo suscribirse)
                         todo ──► CloudWatch / CloudWatch Logs
 Mejoras posibles: DAX · Power Tuning · Layers · init fuera del handler · EventBridge · Powertools
```

---

## ✅ Autoevaluación

> [!question]- 1. El cliente no usa contenedores y ya está aprendiendo Lambda. ¿Por qué Fargate queda fuera si cumplía técnicamente?
> Porque introducir contenedorización exigiría habilidades que el equipo no tiene y una tecnología nueva en su pila; el arquitecto optimiza para el cliente concreto, no solo para lo técnicamente viable. Lambda cumple igual los requisitos sin ese costo.

> [!question]- 2. ¿Qué dato del cliente fue decisivo para elegir DynamoDB y no Aurora Serverless?
> Que sus consultas son CRUD por ID de pedido y listado por ID de cliente, sin joins ni SQL complejo. A eso se suma que con Lambda a escala una base relacional sufriría por exceso de conexiones y requeriría RDS Proxy.

> [!question]- 3. ¿Por qué no basta con que la Lambda de pedidos llame a los tres servicios aguas abajo?
> Porque reproduce el acoplamiento del monolito: si falla a mitad de camino quedan servicios sin avisar, y cualquier cambio en la distribución obliga a tocar la lógica de negocio. Con Streams + SNS la distribución queda fuera del código y un consumidor nuevo solo se suscribe.

> [!question]- 4. ¿En qué casos se habría preferido EventBridge en lugar de SNS?
> Con mensajes heterogéneos que requieran filtrado profundo por reglas sobre el cuerpo, integración con SaaS de terceros, necesidad de schema registry o ruteo entre cuentas/regiones con buses propios. Para un único tipo de mensaje enviado a todos, SNS es más simple, barato y de menor latencia (< 30 ms vs ≈ 0,5 s).

> [!question]- 5. ¿Qué es el patrón storage-first y qué problema resuelve aquí?
> Escribir la solicitud en almacenamiento (la cola SQS) justo después de que la API la valida, responder al usuario de inmediato y procesar después de forma asíncrona. Resuelve la latencia percibida y el rechazo de solicitudes en peaks: si Lambda llega a su límite, los pedidos esperan en la cola sin perderse.

> [!question]- 6. ¿Cuándo se podría haber eliminado la primera Lambda?
> Si no tuviera lógica de negocio: API Gateway podría escribir directamente en DynamoDB mediante su integración de servicio. Como el cliente confirmó lógica compleja, se mantuvo con SQS delante.

> [!question]- 7. Varios consumidores leen la misma cola y algunos pedidos se procesan dos veces. ¿Qué revisa?
> El visibility timeout (debe cubrir el tiempo de procesamiento) y que el consumidor borre el mensaje tras procesarlo; si no lo borra, el mensaje vuelve a ser visible.

> [!question]- 8. ¿Por qué asignar más memoria a una Lambda puede abaratarla?
> Porque más memoria trae más CPU, la función termina antes y Lambda cobra por duración. AWS Lambda Power Tuning prueba configuraciones de 128 MB a 10 GB y sugiere la óptima por costo, velocidad o equilibrio.

> [!question]- 9. ¿Qué puede y qué no debe guardarse fuera del handler?
> Sí: clientes del SDK, conexiones a base de datos, activos estáticos en /tmp, que se reutilizan entre invocaciones de la misma instancia. No: datos de usuario, eventos ni información sensible, porque podrían filtrarse entre invocaciones.

> [!question]- 10. ¿Qué limitación de SQS obliga a otro diseño para mensajes grandes?
> El tamaño máximo de 256 KB. Para cargas mayores se envía en el mensaje una referencia (clave en S3 o en una base) y el consumidor busca el contenido allí.

---

## 🧾 Resumen en 10 líneas

1. El curso 2 enseña a elegir servicios según los requisitos del cliente: 4 clientes, 4 semanas, diseño en bloques y optimización final.
2. AnyCompany Online tiene un servicio de pedidos monolítico y síncrono on-prem que se cae en los peaks y deja datos inconsistentes.
3. Requisitos: escalado gestionado, desacople y resiliencia, logs y métricas centralizados, optimizar costo, rendimiento y operación.
4. Cómputo: Lambda (máx. 15 min, sin servidores, pago por uso); Fargate descartado porque el cliente no usa contenedores; EC2 por carga operativa.
5. API Gateway reemplaza al servidor web y absorbe autenticación, validación y throttling.
6. Base: DynamoDB on-demand (CRUD por clave, sin joins); Aurora Serverless descartada por innecesaria y por el problema de conexiones (RDS Proxy).
7. Distribución: DynamoDB Streams (24 h, *exactly once*, orden por ítem) + Lambda 2 + SNS fan-out; EventBridge queda como mejora futura.
8. Desacople asíncrono: SQS entre API Gateway y Lambda (storage-first), porque la Lambda tiene lógica de negocio.
9. SQS: 256 KB, retención 4 días por defecto / 14 máx., long polling hasta 20 s, visibility timeout, DLQ.
10. Optimizar con DAX (microsegundos, con costo), Power Tuning, Layers, inicialización fuera del handler y Powertools.

---

## 📖 Recursos del módulo

**Generales**
- [AWS re:Post](https://repost.aws)
- [Tickets de contenido de AWS Training](https://support.aws.amazon.com/#/contacts/aws-training)
- [Iconos de arquitectura de AWS](https://aws.amazon.com/architecture/icons/)
- [¿Cómo creo y activo una cuenta de AWS?](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)
- [Capa gratuita de AWS](https://aws.amazon.com/free/)
- [Centro de Arquitectura de AWS](https://aws.amazon.com/architecture/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)

**Cómputo**
- [Computación en AWS](https://aws.amazon.com/products/compute/)
- [Preguntas frecuentes de AWS Lambda](https://aws.amazon.com/lambda/faqs/)
- [Guía del desarrollador de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Guía del operador de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/operatorguide/intro.html)
- [Arquitectura y prácticas recomendadas (Lambda)](https://docs.aws.amazon.com/lambda/latest/operatorguide/architecture-best-practice.html)
- [AWS Lambda — Charlas técnicas](https://aws.amazon.com/lambda/resources/webinars-and-talks/)
- [AWS Lambda — Talleres y tutoriales](https://aws.amazon.com/lambda/resources/workshops-and-tutorials/)
- [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
- [Preguntas frecuentes de API Gateway](https://aws.amazon.com/api-gateway/faqs/)
- [Introducción a las instancias Linux de Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [¿Qué es Amazon EC2?](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [Preguntas frecuentes de Amazon EC2](https://aws.amazon.com/ec2/faqs/)
- [Guía del usuario de Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Contenedores en AWS](https://aws.amazon.com/containers/services/)
- [Amazon ECS en AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Taller de Amazon ECS](https://ecsworkshop.com/)

**Bases de datos**
- [Bases de datos en la nube de AWS](https://aws.amazon.com/products/databases/)
- [Uso de Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)
- [Introducción a Amazon Aurora](https://aws.amazon.com/rds/aurora/getting-started/)
- [Utilización de Amazon RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-proxy.html)
- [¿Qué es Amazon DynamoDB?](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Preguntas frecuentes de DynamoDB](https://aws.amazon.com/dynamodb/faqs/)
- [Ejemplo de modelado de datos relacionales en DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-modeling-nosql-B.html)
- [Laboratorios prácticos para DynamoDB](https://amazon-dynamodb-labs.com/hands-on-labs.html)

**Event-driven y mensajería**
- [¿Qué es una arquitectura basada en eventos?](https://aws.amazon.com/event-driven-architecture/)
- [Destinos de EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)
- [Eventos de EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-events.html)
- [Reglas de EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html)
- [Patrones de eventos de EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)
- [Buses de eventos de EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-bus.html)
- [Preguntas frecuentes de EventBridge](https://aws.amazon.com/eventbridge/faqs/)
- [¿Qué es Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Filtrado de mensajes por payload en SNS](https://aws.amazon.com/blogs/compute/introducing-payload-based-message-filtering-for-amazon-sns/)
- [Preguntas frecuentes de SNS](https://aws.amazon.com/sns/faqs/)
- [¿Qué es Amazon SNS?](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [Introducción a Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html#step-create-queue)
- [Captura de datos de cambios para DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- [Tutorial: procesar nuevos elementos con DynamoDB Streams y Lambda](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.Lambda.Tutorial.html)
- [Patrones para la integración de microservicios](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-integrating-microservices/integrating-patterns.html)
- [Building storage-first serverless applications with HTTP APIs service integrations](https://aws.amazon.com/blogs/compute/building-storage-first-applications-with-http-apis-service-integrations/)
- [Crear una cola SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-create-queue.html)
- [Editar una cola SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-edit-queue.html)
- [Colas de retardo de SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-delay-queues.html)
- [Parámetros de la cola (consola)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-queue-parameters.html)
- [Configuración del long polling](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/working-with-messages.html#setting-up-long-polling)
- [¿Qué es Amazon SQS?](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [Preguntas frecuentes de SQS](https://aws.amazon.com/sqs/faqs/)
- [Uso de AWS Lambda con Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)

**Optimización**
- [Almacenamiento en caché (caching)](https://aws.amazon.com/caching/)
- [Amazon DynamoDB Accelerator (DAX)](https://aws.amazon.com/dynamodb/dax/)
- [AWS Lambda Power Tuning](https://github.com/alexcasalboni/aws-lambda-power-tuning)
- [AWS Lambda Powertools (Python)](https://awslabs.github.io/aws-lambda-powertools-python/latest/)
- [Prácticas recomendadas para AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Optimizing application costs (Well-Architected serverless, blog)](https://aws.amazon.com/blogs/compute/building-well-architected-serverless-applications-optimizing-application-costs/)
- [Serverless Applications Lens — Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)

**Ejercicio**
- [Ejercicio 1: prueba de concepto para una solución sin servidor](https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/DEV-AWS-MO-Architecting/exercise-1-serverless.html)

---

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[04-Monitoreo-y-Optimizacion|04 · Monitoreo y Optimización]] · Siguiente: [[06-Analitica-de-Datos-Serverless|06 · Analítica de Datos Serverless ➡]]
