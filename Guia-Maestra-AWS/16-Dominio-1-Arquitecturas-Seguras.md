---
title: "Tomo 16 — Dominio 1: Design Secure Architectures (30 %)"
tags: [aws, solutions-architect, saa-c03, seguridad, iam, kms, vpc]
curso: 4 · Exam Prep: AWS Certified Solutions Architect – Associate
modulo: "Dominio 1 · Ámbito 1: Diseñar arquitecturas seguras"
audiencias: [tecnico, puente, ejecutivo]
tomo: 16
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🔐 Tomo 16 — Dominio 1: Design Secure Architectures (30 %)

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[15-Examen-SAA-Estrategia|15 · Examen SAA: Estrategia]] · Siguiente: [[17-Dominio-2-Arquitecturas-Resilientes|17 · Dominio 2: Arquitecturas Resilientes ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Es el dominio de **mayor peso del SAA-C03 (30 %)** y, además, un tema transversal: AWS pide pensar en seguridad en cada etapa, nivel y capa, no solo en las preguntas «de seguridad». El curso lo divide en tres task statements —acceso seguro a recursos, cargas de trabajo seguras y controles de seguridad de datos— y para cada uno señala qué hay que dominar y resuelve una pregunta de práctica. Una idea que la instructora repite: en el examen, **la mejor solución suele ser la más segura**.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** modelo de identidades y federación, segmentación de red, servicio de gestión de claves (KMS vs CloudHSM), estrategia de backup y DR que cumpla la regulación.
> - **Costo de hacerlo mal:** root user comprometido = toda la cuenta comprometida; credenciales en el código; datos personales expuestos en S3 sin descubrir.
> - **Pregunta que responde:** *¿quién o qué puede hacer qué sobre qué recurso, por qué camino de red y con qué protección de los datos?*

> [!tip] 💡 Analogía general: el edificio corporativo
> IAM es la credencial de cada persona (qué pisos abre); las resource policies son el letrero en cada puerta (quién puede entrar); los security groups son el guardia junto a cada oficina, las network ACLs el control en la entrada de cada piso, y KMS la caja fuerte con registro de quién la abrió. La defensa en profundidad es tener todos a la vez.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Dominio 1 · Introducción | 1 video | Los tres task statements y la seguridad como tema transversal |
| 1.1 Diseñar acceso seguro a los recursos de AWS | 1 video + pregunta 1.1 | Cuentas, root user, IAM (users, groups, roles, policies), STS, federación, trazabilidad, multicuenta |
| 1.2 Diseñar cargas de trabajo y aplicaciones seguras | 1 video + pregunta 1.2 | VPC segura, subnets, security groups, NACL, endpoints, PrivateLink, VPN, Direct Connect, servicios de seguridad de aplicaciones |
| 1.3 Determinar controles de seguridad de datos | 1 video + pregunta 1.3 | Cifrado en reposo y tránsito, KMS vs CloudHSM, ACM, cifrado S3, cumplimiento, clasificación, backup y DR |
| Recapitulación y recursos | 1 video · 1 lectura | Lista de repaso y 23 enlaces de documentación |

> [!note] ➕ Nota del editor
> El cierre menciona un primer laboratorio práctico «Diseñar arquitecturas seguras»; su contenido no viene en el material extraído.

---

## 1. Task Statement 1.1 — Design secure access to AWS resources

Audiencia: 🔧 🧭

Abarca cómo personas, herramientas y aplicaciones acceden a servicios y datos: quién o qué puede lanzar o terminar recursos, cómo y cuándo se otorga el acceso, los permisos operativos y, en general, cualquier llamada a los servicios. Incluye distintos tipos de entorno: **público, privado, híbrido y multicloud**.

### 1.1 Cuentas y root user

- Las **AWS accounts** suelen pasarse por alto al estudiar, pero son fundamento: hay que entender qué son y cómo funcionan.
- Una cuenta nace con un único **root user** con todos los permisos. Sus permisos **no se pueden restringir**; si se compromete, cae todo el entorno.
- Protecciones: **MFA**, contraseña robusta, y crear identidades con permisos más limitados para el trabajo diario.

### 1.2 IAM

| Concepto | Lo que el curso destaca |
|---|---|
| Ámbito | IAM es un servicio **global**: su base de datos de identidades se protege en todas las Regions |
| Identidades nuevas | Los IAM users nacen **sin permisos**; se conceden explícitamente |
| Users, groups, roles | Saber crearlos, sus fortalezas y límites, y cuándo pasar de permisos por grupo a permisos por rol |
| Least privilege | Limita el **blast radius** |
| Credenciales | Buenas prácticas de gestión; **nunca** hardcodear credenciales en la aplicación |
| Asumir roles | Varias formas, incluido asignar roles a servicios de AWS; **STS** dentro y entre cuentas; roles en federación |
| Federación | Usuarios con identidad fuera de AWS (directorio corporativo, IdP externo) obtienen permisos vía un IAM role; saber federar Active Directory hacia roles o usuarios |

**Policies.** No se exige escribir políticas avanzadas, pero sí **leerlas e interpretarlas**: partes de un statement, qué es obligatorio, cómo dan granularidad y la **lógica de evaluación** cuando una identidad tiene varias políticas.

| Tipo | Se adjunta a | Controla | Diferencia estructural |
|---|---|---|---|
| Identity-based | User, group o role | **A qué** recursos accede esa identidad | Sin `Principal` |
| Resource-based | Bucket S3, cola SQS, VPC endpoint, key KMS… | **Quién** accede al recurso y qué acciones hace | Lleva `Principal`: define a quién afecta |

### 1.3 Trazabilidad y multicuenta

- La trazabilidad permite monitorear, alertar y auditar acciones y cambios en tiempo real, e integrar logs y métricas para investigar y actuar automáticamente.
- Se evalúa diseñar estrategias de seguridad **multicuenta**: visibilidad, cumplimiento de estándares, alertas y automatización. Servicios: **AWS Control Tower, AWS Organizations y Service Control Policies**.

🎯 **Para el examen:**
- Root user: MFA y contraseña compleja; nunca access keys de root.
- Identity policy = qué puede hacer la identidad; resource policy = quién puede tocar el recurso (`Principal`).
- Credenciales para aplicaciones: roles, nunca hardcodeadas.

---

## 2. Task Statement 1.2 — Design secure workloads and applications

Audiencia: 🔧 🧭

### 2.1 VPC y segmentación

| Pieza | Ámbito de resiliencia | Qué saber |
|---|---|---|
| Amazon VPC | **Regional** (una Region y una cuenta) | Default vs custom, y cómo viene configurada la seguridad de cada una |
| Subnet | **Zonal** (vive en una AZ) | Dónde corren los servicios; pública vs privada y prácticas habituales |
| Security group | Instancia | Reglas, procesamiento, configuración base |
| Network ACL | Borde de subnet | Reglas, orden de procesamiento, configuración base |
| Route table | Subnet / gateway | Mecanismo de enrutamiento |
| NAT gateway | — | Salida a Internet desde subnets privadas |

Hay que entender cómo funcionan **solos y combinados**, cómo escribir sus reglas, los errores a evitar y la lógica de procesamiento. Fundamentos de red: protocolos, CIDR, subnetting, routing, filtros, gateways.

> [!example] 📊 Escenario que plantea el curso
> Servidores de aplicación en una subnet privada deben aceptar cierto tráfico que llega desde on-premises por una VPN, sin quedar expuestos a Internet. Resolverlo exige combinar route tables, filtros de seguridad y saber elegir componentes de conectividad: VPC endpoints, PrivateLink, peering, Transit Gateway, VPN y Direct Connect.

### 2.2 Endpoints y PrivateLink

- Un **VPC endpoint** es un objeto tipo gateway que se crea dentro de la VPC (como un IGW o un NAT gateway) para llegar a servicios públicos de AWS **sin** pasar por Internet gateway ni NAT gateway.
- **AWS PrivateLink** expone una aplicación o servicio a decenas o cientos de VPC (incluso de otras cuentas) **sin peering, IGW ni NAT**. El VPC peering también conecta, pero agrega carga de administración al escalar y expone las demás aplicaciones de las VPC pareadas.
- Conexiones externas privadas: **Site-to-Site VPN, Client VPN y Direct Connect**, con sus opciones de capacidad, seguridad y resiliencia.

### 2.3 Servicios de seguridad de aplicaciones

| Servicio | Para qué (según el curso) |
|---|---|
| Amazon Macie | Usa machine learning para descubrir, clasificar y proteger datos sensibles (PII) en **S3** |
| Amazon Cognito | User pools e identity pools; SSO y federación de identidades para aplicaciones |
| Amazon GuardDuty | Detección de amenazas (citado junto a Macie y Cognito) |
| AWS Shield | Protección DDoS; distinguir **Standard** vs **Advanced** |
| AWS WAF | Filtra ataques web como SQL injection; solo se despliega en ciertos servicios: **ALB, API Gateway y CloudFront** |
| AWS IAM Identity Center | Acceso centralizado (antes AWS SSO) |
| AWS Secrets Manager | Diseñado para secretos; **rotación automática** cada N días; alto volumen de acceso |
| SSM Parameter Store | Parámetros y configuración; alternativa más simple a Secrets Manager |

> [!tip] 💡 Secrets Manager vs Parameter Store
> Si el escenario dice «secreto», «alto volumen de acceso» y «rotación automática de credenciales», la respuesta es Secrets Manager.

PII, según el curso: nombre, dirección, email, número de seguridad social, licencia de conducir, pasaporte, fecha de nacimiento, cuenta bancaria, tarjeta de crédito.

---

## 3. Task Statement 1.3 — Determine appropriate data security controls

Audiencia: 🔧 🧭

### 3.1 Cifrado: vocabulario mínimo

| Término | Significado |
|---|---|
| Plaintext | Datos no cifrados (no necesariamente texto: imágenes, documentos, aplicaciones) |
| Algorithm | Código que toma plaintext + key y produce datos cifrados |
| Key | Esencialmente una contraseña que se usa con el algoritmo |
| Ciphertext | Los datos cifrados |
| Simétrico / asimétrico | Los dos tipos de cifrado y de claves que hay que distinguir |

| | At rest | In transit |
|---|---|---|
| Protege contra | Acceso no autorizado y robo | Lectura durante la transferencia |
| Participantes | Normalmente una parte | Dos o más partes |
| Herramienta típica | KMS, cifrado de EBS/S3/RDS | TLS/HTTPS, **AWS Certificate Manager** (y renovación de certificados) |

### 3.2 KMS vs CloudHSM

| | AWS KMS | AWS CloudHSM |
|---|---|---|
| Modelo | Usa HSM para proteger y validar las keys, pero **AWS gestiona el software** de cifrado | AWS aprovisiona el **hardware**; el cliente usa su propio cliente para cifrar |
| FIPS 140-2 | **Nivel 2** | **Nivel 3** |
| Preguntas a dominar | Gestión de claves entre Regions, tipos de key y frecuencia de rotación, key policies, root keys vs data keys | Cuándo se prefiere o cómo combinarlo con KMS |

### 3.3 Cifrado en Amazon S3

- **Client-side:** el objeto se cifra antes de salir del origen; usted controla todo.
- **Server-side:** viaja por HTTPS y S3 cifra al llegar. Tres variantes: **SSE-C** (claves provistas por el cliente), **SSE-S3** (claves administradas por S3) y **SSE-KMS** (claves en AWS KMS).
- S3 permite asegurar buckets completos y también controlar por **prefijo u objeto**.
- Ciclo de vida: saber cuándo usar **S3 Lifecycle** vs **S3 Intelligent-Tiering**.

### 3.4 Cumplimiento, clasificación y defensa en profundidad

- **AWS Artifact:** repositorio central self-service de informes de cumplimiento y seguridad de AWS y de acuerdos en línea.
- **AWS Cloud Adoption Framework**, perspectiva de seguridad, con cinco capacidades: IAM, controles de detección, seguridad de infraestructura, protección de datos y respuesta a incidentes.
- No todos los datos son iguales: **clasificarlos** antes de protegerlos.
- **Defense in depth:** superponer controles **preventivos** y **detectivos** para tener redundancia.
- Preguntas guía: protección de datos por VPN, Direct Connect, entre VPC, entre S3 y la VPC, y hacia usuarios por Internet; diferencias S3 vs EBS; si el cifrado afecta el rendimiento (por ejemplo, RDS con KMS o lecturas de S3).

> [!example] 📊 EBS cifrado o S3 cifrado
> Datos generados en una EC2 con volumen EBS que deben protegerse manteniendo la durabilidad: el **menor esfuerzo** es usar un volumen EBS cifrado, no moverlos a un bucket S3.

### 3.5 Almacenamiento en la nube, backup y DR

- Almacenamiento en la nube: el proveedor opera el almacenamiento como servicio. Beneficios: sin hardware ni capex, despliegue rápido, lifecycle que automatiza y ahorra, bloqueo de datos para cumplimiento.
- Diseñar según durabilidad, disponibilidad, seguridad, regulación y gobierno, y requisitos funcionales; saber si el dato es **objeto, archivo o bloque**.
- Cinco usos: backup y recuperación, test y desarrollo, migración, cumplimiento, big data y data lakes.

**Cuatro estrategias de DR** (de menor a mayor costo y complejidad):

```
 Backup & Restore ──► Pilot Light ──► Warm Standby ──► Multi-site Active-Active
 (barato, simple)                                       (varias Regions activas)
        └─────────── active/passive ───────────┘
```

- **Active/passive:** un sitio (Region) activo atiende el tráfico; el pasivo, en otra Region, no recibe tráfico hasta un failover.
- Si el desastre es la pérdida de un data center en una carga ya altamente disponible, puede bastar **backup & restore**. Si el desastre se define a nivel de **Region** o la regulación lo exige, considerar pilot light, warm standby o multi-site.
- La **frecuencia** del backup determina el punto de recuperación alcanzable (RPO).

| Servicio | Mecanismo de backup que menciona el curso |
|---|---|
| EBS | Snapshots |
| DynamoDB | Backups (y point-in-time recovery en recursos) |
| RDS / Aurora | Snapshots |
| EFS | Vía AWS Backup |
| Redshift, Neptune, DocumentDB | Snapshots/backups propios |
| S3 | **Cross-Region Replication** asíncrona y continua + **versioning** para elegir punto de restauración |
| AWS Backup | Consola central para configurar, programar y monitorear backups de EBS, EC2, RDS, Aurora, DynamoDB, EFS, volúmenes de Storage Gateway, FSx for Windows File Server y FSx for Lustre; copia entre Regions |
| AWS Storage Gateway | Servicio para entornos **híbridos** |

---

## 4. Preguntas de práctica resueltas en el curso

Audiencia: 🔧 🧭

El método que se aplica en cada una: leer el enunciado (stem), identificar palabras clave, revisar opciones, elegir y luego explicar por qué fallan los distractores. **No hay crédito parcial** en preguntas de respuesta múltiple.

### Pregunta 1.1 — Proteger el inicio de sesión del root user (elija dos)

**Escenario:** el CIO quiere que el root user de la cuenta siga las buenas prácticas de inicio de sesión seguro. Palabras clave: *root user*, *best practices*, *inicio de sesión seguro*.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| Exigir access key ID + secret para el root | ❌ | Las access keys son para llamadas programáticas; las del root dan acceso total (incluida facturación) y no se pueden acotar |
| **Exigir MFA en el root** | ✅ | Buena práctica estándar |
| Que el root asuma un role para sus propios recursos | ❌ | El root no puede asumir un role en su propia cuenta; los roles son para servicios y acceso de terceros/otras cuentas |
| **Exigir contraseña compleja para el root** | ✅ | Buena práctica estándar |
| Eliminar la cuenta root | ❌ | Borrarla elimina todos los recursos: ya no habría entorno en AWS |

### Pregunta 1.2 — Dos aplicaciones en la misma subnet con reglas distintas

**Escenario:** App A en una EC2 debe aceptar solo HTTP entrante; App B, en otra EC2 **de la misma subnet**, solo HTTPS (bloqueando incluso HTTP). Palabras clave: *misma subnet*, *permitir/bloquear entrante*.

| Opción | Veredicto | Razón |
|---|---|---|
| Network ACLs | ❌ | Operan en el borde de la subnet; ambas instancias compartirían la misma NACL y no se pueden diferenciar |
| **Security groups** | ✅ | Firewall virtual a nivel de instancia; solo reglas *allow* y bloqueo implícito del resto; cada instancia puede tener reglas distintas |
| VPC peering | ❌ | Conecta VPC distintas; no bloquea tráfico |
| Route tables | ❌ | Dirigen el tráfico; no lo bloquean entre instancias de la misma subnet |

### Pregunta 1.3 — Cifrado para requisitos regulatorios

**Escenario:** se necesita una solución de cifrado segura y durable para generar, almacenar y controlar claves criptográficas de datos, con requisitos regulatorios. Palabras clave: *regulatorio*, *seguridad y durabilidad*, *generación, almacenamiento y control*, *más segura*.

| Opción (paráfrasis) | Veredicto | Razón |
|---|---|---|
| **KMS genera KMS keys y data keys; key policies controlan el acceso** | ✅ | Cubre generación, almacenamiento durable y control de acceso en un servicio diseñado para eso |
| KMS genera y luego se importan a ACM, con IAM policies | ❌ | ACM gestiona certificados SSL/TLS, no data keys |
| Herramienta de Marketplace + instance store cifrado + IAM | ❌ | Instance store es **efímero**: incumple durabilidad |
| OpenSSL + bucket S3 cifrado + key policies de KMS | ❌ | S3 no gestiona claves y las key policies solo aplican a keys que viven en KMS |

> [!tip] 💡 Contexto útil que da la instructora
> KMS y CloudHSM son compatibles con FIPS (nivel 2 y nivel 3). La palabra «regulatorio» no obliga por sí sola a CloudHSM: aquí ninguna opción lo ofrecía y KMS cumplía todos los requisitos.

---

## 5. Trampas típicas del dominio

> [!danger] 🚨 Errores costosos
> - Elegir **NACL** cuando las instancias comparten subnet y necesitan reglas distintas (va security group).
> - Creer que una **route table** o el **VPC peering** filtran tráfico.
> - Usar **ACM** para claves de datos (es para certificados).
> - Guardar algo que debe ser durable en **instance store**.
> - Proponer access keys para el **root** o borrar la cuenta root.
> - Hardcodear credenciales en la aplicación (la respuesta es un role).
> - Olvidar que WAF solo va en ALB, API Gateway o CloudFront.
> - Confundir Macie (PII en S3) con GuardDuty (amenazas) o Cognito (identidad de usuarios de la app).

---

## 6. Lista de repaso del cierre del dominio

- Least privilege en multicuenta: ¿Control Tower, Service Catalog u Organizations?
- IAM role vs IAM user: cuándo cada uno.
- Diferencias entre identity policy, resource policy, permissions policy y SCP; evaluación con allow y deny solapados.
- Formas de federarse en AWS; AWS SSO (IAM Identity Center) y casos de uso de AWS Directory Service.
- Monitoreo: CloudTrail, CloudWatch, VPC Flow Logs y otros.
- VPC propias con los controles adecuados.
- Casos de uso de Shield, WAF, Secrets Manager y Parameter Store.
- Datos en tránsito y en reposo; KMS vs CloudHSM.
- Saber elegir entre dos servicios parecidos según el stem.

---

## 7. Dónde se ve en esta guía

| Tema del dominio | Tomo |
|---|---|
| Shared Responsibility, root user, IAM, MFA, policies | [[01-Vision-General-y-Seguridad]] |
| VPC, subnets, security groups, NACL, route tables, NAT | [[02-Computo-y-Redes]] |
| Cifrado de S3/EBS, clases y lifecycle, snapshots y backups | [[03-Almacenamiento-y-Bases-de-Datos]] |
| CloudTrail, CloudWatch, trazabilidad | [[04-Monitoreo-y-Optimizacion]] |
| Cognito, API Gateway, WAF frente a una API serverless | [[05-Backend-Web-Serverless]] |
| VPN, Direct Connect, Storage Gateway (híbrido) | [[07-Solucion-Hibrida-Contenedores]] |
| Organizations, Control Tower, SCP, IAM Identity Center | [[08-Gobierno-y-Multicuenta]] |
| Permisos finos sobre datos, cifrado en el data lake | [[11-Data-Lake-con-Lake-Formation]], [[13-Lake-Formation-Avanzado]] |

---

## ✅ Autoevaluación

> [!question]- ¿Qué ámbito de resiliencia tienen IAM, una VPC y una subnet?
> IAM es global, la VPC es regional y la subnet es zonal.

> [!question]- ¿Qué elemento distingue estructuralmente una resource-based policy?
> El `Principal`, que define a quién afecta la política.

> [!question]- Una aplicación expone un servicio a 150 VPC de otras cuentas sin exponer el resto de las aplicaciones. ¿Qué usa?
> AWS PrivateLink; el peering no escala bien y expone las demás aplicaciones de las VPC pareadas.

> [!question]- Necesita un secreto con rotación automática cada 30 días y mucho tráfico de lectura. ¿Servicio?
> AWS Secrets Manager.

> [!question]- ¿En qué servicios se puede desplegar AWS WAF según el curso?
> Application Load Balancer, Amazon API Gateway y Amazon CloudFront.

> [!question]- ¿Qué nivel FIPS 140-2 tienen KMS y CloudHSM?
> KMS nivel 2; CloudHSM nivel 3.

> [!question]- ¿Cuáles son las tres variantes de server-side encryption en S3?
> SSE-C (claves del cliente), SSE-S3 (claves administradas por S3) y SSE-KMS (claves en KMS).

> [!question]- La regulación exige recuperarse ante la caída de una Region completa. ¿Basta backup & restore?
> Probablemente no: el curso indica considerar pilot light, warm standby o multi-site active-active cuando el desastre se define a nivel de Region o lo exige la regulación.

> [!question]- ¿Qué servicio descubre PII en buckets S3 con machine learning?
> Amazon Macie.

## 🧾 Resumen en 10 líneas

1. Dominio 1 pesa 30 % y tiene tres task statements: acceso, cargas de trabajo y datos.
2. Root user: MFA y contraseña fuerte; sin access keys; no se borra.
3. IAM es global; las identidades nacen sin permisos; least privilege reduce el blast radius.
4. Identity policy = qué hace la identidad; resource policy = quién toca el recurso.
5. Federación y STS para identidades externas y acceso entre cuentas; credenciales nunca en el código.
6. Multicuenta: Organizations, Control Tower y SCP.
7. VPC regional, subnet zonal; security group por instancia, NACL por subnet.
8. PrivateLink expone servicios a muchas VPC sin peering; endpoints evitan IGW/NAT.
9. KMS (FIPS 2, AWS gestiona el software) vs CloudHSM (FIPS 3, hardware dedicado); ACM para certificados.
10. DR: backup & restore → pilot light → warm standby → multi-site; AWS Backup centraliza y copia entre Regions.

## 📖 Recursos del módulo

- Administración de su cuenta de AWS · https://docs.aws.amazon.com/accounts/latest/reference/managing-accounts.html
- ¿Qué es IAM? · https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html
- ¿Qué es Amazon VPC? · https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
- Seguridad en Amazon VPC · https://docs.aws.amazon.com/vpc/latest/userguide/security.html
- Seguridad, identidad y conformidad · https://docs.aws.amazon.com/whitepapers/latest/aws-overview/security-services.html
- Cifrado de datos · https://docs.aws.amazon.com/whitepapers/latest/introduction-aws-security/data-encryption.html
- Instantáneas de Amazon EBS · https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html
- Point-in-time recovery para DynamoDB · https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html
- Backup y restauración de RDS · https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_CommonTasks.BackupRestore.html
- Snapshot de clúster Aurora · https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_CreateSnapshotCluster.html
- AWS Backup con Amazon EFS · https://docs.aws.amazon.com/efs/latest/ug/awsbackup.html
- Snapshots y backups de Amazon Redshift · https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-snapshots.html
- Backup y restauración en Neptune · https://docs.aws.amazon.com/neptune/latest/userguide/backup-restore-overview.html
- Backup y restauración en DocumentDB · https://docs.aws.amazon.com/documentdb/latest/developerguide/backup_restore.html
- S3 Cross-Region Replication · http://aws.amazon.com/s3/features/replication/
- AWS Backup · http://aws.amazon.com/backup
- Amazon EBS · http://aws.amazon.com/ebs/
- Amazon EC2 · http://aws.amazon.com/ec2/
- Amazon RDS · http://aws.amazon.com/rds/
- Amazon Aurora · http://aws.amazon.com/rds/aurora/
- Amazon DynamoDB · http://aws.amazon.com/dynamodb/
- Amazon EFS · http://aws.amazon.com/efs/
- AWS Storage Gateway · http://aws.amazon.com/storagegateway/
- Amazon FSx for Windows File Server y FSx for Lustre · http://aws.amazon.com/fsx/windows/

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[15-Examen-SAA-Estrategia|15 · Examen SAA: Estrategia]] · Siguiente: [[17-Dominio-2-Arquitecturas-Resilientes|17 · Dominio 2: Arquitecturas Resilientes ➡]]
