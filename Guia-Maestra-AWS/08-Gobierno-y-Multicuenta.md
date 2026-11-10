---
title: "Tomo 08 — Gobierno de cuentas y estrategia multicuenta"
tags: [aws, solutions-architect, gobierno, multicuenta, organizations, iam, control-tower, cloudtrail]
curso: 2 · Architecting Solutions on AWS
modulo: "Diseñar una solución siguiendo las mejores prácticas de gobierno y gestión de cuentas"
audiencias: [tecnico, puente, ejecutivo]
tomo: 08
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🏛️ Tomo 08 — Gobierno de cuentas y estrategia multicuenta

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[07-Solucion-Hibrida-Contenedores|07 · Solución híbrida con contenedores]] · Siguiente: [[09-Introduccion-a-Data-Lakes|09 · Introducción a data lakes ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Muchas empresas empiezan en AWS con **una sola cuenta** que crece sin plan: todo mezclado, el usuario root como acceso diario, usuarios IAM repartidos sin control, etiquetas que nadie hace cumplir y una factura imposible de leer. Este tomo muestra cómo pasar de ese desorden a una **estrategia multicuenta gobernada**: cuentas separadas por cliente y entorno, roles IAM como mecanismo de acceso entre cuentas, AWS Organizations con SCP como guardarraíles, IAM Identity Center para single sign-on, CloudTrail centralizado y Control Tower + Service Catalog para que cada cuenta nueva nazca configurada.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** cómo dividir el entorno cloud (por cliente, unidad de negocio, entorno); quién puede entrar a producción; cómo asignar costos por cuenta; qué controles son innegociables para toda la organización.
> - **Costo de hacerlo mal:** un cambio accidental en el entorno de otro cliente; credenciales root expuestas; un desarrollador que lanza instancias enormes en dev; registros de auditoría borrables; facturas sin dueño claro; cuotas de servicio compartidas que se agotan para todos.
> - **Pregunta que responde:** *¿cómo crezco en AWS sin perder el control de quién hace qué, dónde y cuánto cuesta?*

> [!tip] 💡 Analogía general: del departamento compartido al edificio con conserjería
> La empresa vive hoy en un solo departamento donde todos tienen la llave maestra (root). La solución es un **edificio**: cada cliente y entorno tiene su propio departamento (cuenta); el **reglamento del edificio** (SCP) rige aunque el dueño del departamento no esté de acuerdo; la **conserjería** (cuenta de servicios compartidos) entrega una única tarjeta de acceso por persona (IAM Identity Center); la **cámara del hall** graba todo y nadie puede borrarla (CloudTrail centralizado); y la **constructora** (Control Tower) entrega cada departamento nuevo ya amoblado con el catálogo de muebles aprobados (Service Catalog).

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Cliente nº 4 — Una empresa emergente de rápido crecimiento | 3 videos | Caso de la agencia de marketing con una sola cuenta; requisitos definidos por el arquitecto |
| Arquitectura de la solución | 6 videos (4 demos de consola) / 5 lecturas | Por qué multicuenta; roles IAM y relaciones de confianza; Organizations, OU y SCP; IAM Identity Center; servicios de logging; Control Tower, Service Catalog y CloudFormation |
| Presentación de la solución al cliente y próximos pasos | 3 videos / 2 lecturas / 1 encuesta | Revisión con el cliente y plan de implementación; mejoras (SCP sobre root, CloudTrail, IP, tag policies, billing alarms, MFA, movimiento de cuentas); cierre del curso |

> [!note] ➕ Nota del editor
> La lección final incluye una **encuesta posterior al curso** (widget sin texto evaluable, alojada en Qualtrics). No hay laboratorios ni quiz en la extracción de este módulo; la práctica son cuatro demos de consola (sección 8).

---

## 1. El caso: la agencia de marketing con una sola cuenta

Audiencia: 🔧 🧭 👔

Esta semana los papeles se invierten: **Raf** es el arquitecto y **Morgan** interpreta al cliente. Raf cuenta que la situación le ocurrió a él casi una década atrás: en una startup, por saber Linux, terminó a cargo de todos los entornos, hasta que el gobierno de cuentas se volvió un trabajo de tiempo completo (su carrera pasó de desarrollador a SysOps, DevOps, Arquitecto de Soluciones y hoy Principal Cloud Technologist en AWS).

### 1.1 Situación actual

- Una **agencia de marketing** que construye sitios web para sus clientes. Según el proyecto, corren en **EC2 + RDS**, en **serverless** o en **contenedores**.
- **Todo, para todos los clientes, en una sola cuenta** llamada "prod".
- Usan infraestructura como código para *algunas* cargas, pero igual se puede tocar el *stack* equivocado; **ya han ocurrido** cambios accidentales en el recurso equivocado.
- Morgan, uno de los primeros empleados, quedó a cargo de todo, incluida la **facturación**. Hay etiquetas y reportes de costos por etiqueta, pero **no todo está etiquetado** y no hay forma de **hacer cumplir** el etiquetado.
- Nuevos desarrolladores piden acceso y no sabe cómo **retirarlo** ni cómo controlar lo que crean.
- **Acceso:** Morgan entra con su **correo personal**, es decir, con el **usuario root**; el personal nuevo usa **usuarios IAM** y "promete ser cuidadoso".

> [!danger] 🚨 Lo que hizo "respirar hondo" al arquitecto
> Usar el **usuario root** para el trabajo diario es, por desgracia, más común de lo que se cree. Confiar en permisos básicos y en la buena voluntad del equipo es, en palabras de Raf, "cuestión de tiempo antes de un accidente".

### 1.2 El arquitecto define los requisitos

El cliente no trae requisitos bien definidos ("no sabe lo que no sabe"). Parte del rol del arquitecto o consultor es **ayudarle a definirlos** a partir de buenas prácticas. Raf propone:

| # | Requisito | Justificación |
|---|---|---|
| R1 | **Múltiples cuentas** con **aprovisionamiento automático** | Separar cargas y entornos; reglas de gobierno claras |
| R2 | Una **cuenta de servicios compartidos** donde se inicia sesión primero | Punto de entrada único para administradores |
| R3 | **Single sign-on** hacia las demás cuentas | No replicar credenciales de usuario en cada cuenta |
| R4 | **Guardarraíles de seguridad** en las cuentas y reglas de configuración para las cuentas nuevas | Evitar recursos no autorizados y deriva |
| R5 | **Cuenta dedicada para logging** (seguridad, actividad de API) | Buena práctica de gobierno; permite controles más fuertes, como impedir que se borren los registros |

Las personas con acceso a las cuentas podrán formar parte, a futuro, de un **CCoE (Cloud Center of Excellence)**: el grupo con acceso a las cuentas y, en especial, a producción.

---

## 2. ¿Por qué una estrategia multicuenta?

Audiencia: 🔧 🧭 👔

No hay fórmula única para agrupar recursos; sí hay recomendaciones. Una cuenta de AWS es un **límite de seguridad, acceso y facturación**: lo que vive en una cuenta está lógicamente aislado de las demás, incluso dentro de su propio entorno.

### 2.1 Las seis ventajas (resumen de Raf)

1. Agrupar cargas por **propósito de negocio y propiedad**.
2. **Centralizar el logging**.
3. **Restringir el acceso a datos sensibles**.
4. **Limitar el radio de impacto** (*blast radius*) de eventos adversos.
5. **Gestionar mejor los costos**.
6. **Distribuir las cuotas de servicio** y los límites de tasa de API entre cuentas.

La lectura añade que una estrategia multicuenta ayuda a optimizar la mayoría de los pilares del **AWS Well-Architected Framework**: **excelencia operativa, seguridad, fiabilidad y optimización de costos**, y que forma parte de establecer los **cimientos** (*cloud foundation*) del entorno.

### 2.2 Detalle de cada motivo (lectura)

| Motivo | Idea central | Ejemplo del material |
|---|---|---|
| Propósito de negocio y propiedad | Alinear propiedad y decisiones; evitar dependencias entre cargas; unidades de negocio con más control descentralizado pero con controles generales; facilita **desinversiones** | Si adquiere una empresa que ya opera en AWS, puede mover sus cuentas **intactas** a su organización como primer paso de integración |
| Controles de seguridad distintos por entorno | Producción y no producción requieren políticas distintas; en cuentas separadas quedan aisladas **por defecto** | — |
| Limitar acceso a datos sensibles | Menos personas y procesos con acceso; *least privilege* más simple a nivel grueso | Un conjunto de cuentas designado para buckets S3 **públicos** y políticas que **prohíben** a todas las demás cuentas hacer públicos sus buckets |
| Innovación y agilidad | Cuentas propias para que los *builders* experimenten | **Sandbox**: desconectadas de servicios y datos internos, máxima libertad. **Desarrollo**: acceso limitado a servicios y datos de desarrollo. En ambas: guardarraíles y **presupuestos de costos** |
| Limitar el impacto | Fallos de aplicación, malas configuraciones o acciones maliciosas quedan contenidas | — |
| Múltiples modelos operativos de TI | Grupos de cuentas con controles distintos según el modelo | Ver tabla 2.3 |
| Costos | La cuenta es la unidad **por defecto** de asignación de costos; además, consolidación de costos entre cuentas y **cost allocation tags** para detalle | — |
| Cuotas y límites de API | Las **service quotas** (p. ej. número de buckets S3 por cuenta) y el *throttling* de API se aplican **por cuenta** | El servicio **Service Quotas** protege de aprovisionamientos excesivos o maliciosos |

> [!note] ➕ Nota del editor
> En AWS se llama **builders** a los tecnólogos que crean valor con los servicios: desarrolladores, ingenieros de datos, científicos y analistas de datos, ingenieros de seguridad e infraestructura (según la lectura).

### 2.3 Modelos operativos de TI

| Modelo | Equipos de aplicación | Equipo de plataforma cloud | Operaciones |
|---|---|---|---|
| **Tradicional** | Ingeniería de sus apps (custom y COTS), no su operación en producción | Ingeniería de la plataforma | Un equipo **separado** de cloud operations opera apps y plataforma |
| **CloudOps** | Ingeniería **y** operación en producción de sus apps | Ingeniería y operación de la plataforma común | Repartidas |
| **DevOps** | Además, ingeniería y operación de las capacidades de plataforma **específicas** de su app | Ingeniería y operación de capacidades **compartidas** | Distribuidas |

La **gestión de servicios de TI (ITSM)** es común a los tres; cambian los responsables y las soluciones.

### 2.4 Estrategia elegida para el cliente

Formas de agrupar: por **carga de trabajo**, por **cliente**, por **entorno** (dev, QA, prod), entre otras. Raf elige **por unidad de negocio / cliente** (una de las más usadas y la que calza con una agencia que atiende a clientes), combinada con **segmentación por carga de trabajo y entorno**:

```
 Root
 ├── Customer A (OU)
 │   ├── Dev (OU)   ── cuentas por desarrollador (IDE en la nube), SCP: sin instancias grandes
 │   ├── Test (OU)
 │   └── Prod (OU)  ── acceso solo para el CCoE / personas con más experiencia
 ├── Customer B (OU)
 │   └── …
 └── Shared Services (cuenta de administración)
```

> [!warning] ⚠️ Regla crítica
> Desde el momento en que tiene varias cuentas necesita dos cosas: **automatización** y **credenciales centralizadas**. Sin ellas, la gestión se vuelve inmanejable rápidamente.

---

## 3. Roles IAM: el mecanismo central de autenticación entre cuentas

Audiencia: 🔧 🧭

### 3.1 Entidades de IAM

| Entidad | Qué es | Credenciales |
|---|---|---|
| **IAM user** | Identidad con permisos | **Permanentes** (contraseña, access keys) |
| **IAM group** | Colección de usuarios | — |
| **IAM role** | Identidad con permisos que **cualquiera autorizado puede asumir** | **Temporales** (o una URL de inicio de sesión) al hacer **AssumeRole** |

Con varias cuentas se quiere **evitar replicar usuarios IAM en cada una**: ahí entran los roles.

**Quién puede asumir un rol:** un usuario IAM de la **misma cuenta**; un usuario IAM de **otra cuenta**; un **servicio de AWS** (p. ej. EC2, Lambda); un **usuario externo** autenticado por un IdP compatible con **SAML 2.0**, **OpenID Connect (OIDC)** o un *identity broker* personalizado.

### 3.2 Vocabulario de roles (lectura)

| Término | Definición resumida |
|---|---|
| **Service role** | Rol que un servicio asume para actuar en su cuenta en su nombre; usted lo crea, modifica y elimina desde IAM |
| **Service-linked role** | Tipo especial de service role **vinculado** a un servicio; permisos **predefinidos** por el servicio, que también define cómo se crea y elimina |
| **Delegación** | Conceder a alguien acceso a recursos que usted controla, estableciendo confianza entre la cuenta **que confía** (dueña del recurso) y la cuenta **en la que se confía** (dueña de los usuarios). Pueden ser la misma cuenta, dos de su organización o de organizaciones distintas |
| **Trust policy** | Política JSON **basada en recursos**, obligatoria, adjunta al rol: define qué **principals** (usuarios, roles, cuentas, servicios) pueden asumirlo. **No admite comodín `*`** como principal |
| **Permissions policy** | Define qué puede hacer quien asume el rol |
| **Federación** | Confianza entre un IdP externo y AWS: IdP web (Login with Amazon, Facebook, Google, cualquier OIDC) o empresarial SAML 2.0 (p. ej. AD FS). Al usuario se le asigna un rol y recibe credenciales temporales |
| **Federated user** | Identidad existente (Directory Service, directorio corporativo, IdP web) usada en lugar de un IAM user |
| **Permissions boundary** | Política avanzada que limita los permisos **máximos** que una política de identidad puede dar a un rol; **no** aplica a service-linked roles |
| **Principal** | Entidad que puede actuar y acceder a recursos: usuario root, IAM user o rol |
| **External ID** | Parámetro adicional para usar con seguridad roles entre cuentas que **no** son de la misma organización |

**Las dos mitades del acceso entre cuentas:**

```
  Cuenta B (en la que se confía)            Cuenta A (que confía, dueña del recurso)
  ┌──────────────────────────┐              ┌──────────────────────────────────────┐
  │ IAM user "rafael"        │  AssumeRole  │ IAM role "AdminRole"                 │
  │ + permissions policy que │ ───────────► │ + trust policy: principal = rafael@B │
  │   le permite asumir rol  │              │ + permissions policy: Administrator… │
  └──────────────────────────┘              └──────────────────────────────────────┘
```

Quien asume un rol **renuncia temporalmente** a sus propios permisos y toma los del rol; al salir, recupera los suyos.

🎯 **Para el examen:** roles = credenciales temporales; trust policy (quién) vs. permissions policy (qué); service-linked role sin permissions boundary; external ID para terceros; federación SAML 2.0 / OIDC.

---

## 4. AWS Organizations, OU y SCP

Audiencia: 🔧 🧭

**AWS Organizations** permite gestionar y gobernar centralmente el entorno a medida que crece:

- **Crear** cuentas, **añadir** o **invitar** cuentas existentes, y también **eliminar** cuentas (útil para limpiar sin dejar recursos sobrantes).
- Agruparlas en **organizational units (OU)**.
- Aplicar **service control policies (SCP)** y habilitar o deshabilitar servicios en las cuentas hijas.
- **Facturación separada por cuenta**; antes de Organizations esta función se llamaba **Consolidated Billing**.
- Al crear la organización, la cuenta actual pasa a ser la **management account** (en el caso, la de servicios compartidos).
- Al crear una cuenta, Organizations sugiere crear el rol **`OrganizationAccountAccessRole`**, que permite a la cuenta de administración entrar a la nueva por *switch role*.

**SCP:** mecanismo de permisos que gobierna todo el **límite de una OU o cuenta**. Con una **denegación explícita**, **ni siquiera el usuario root** de la cuenta miembro puede ejecutar la acción.

> [!example] 📊 SCP de la demo: `t2.micro_only`
> Una declaración permite crear instancias EC2; otra **deniega explícitamente** `RunInstances` si el `InstanceType` **no es** `t2.micro`. Adjunta a la OU *Dev Environments*, rige para toda cuenta dentro de ella. Una prueba con `c5.12xlarge` falla; con `t2.micro` la instancia se lanza (queda en *pending*).

> [!warning] ⚠️ Buena práctica de seguridad
> Aplicar en la **raíz de la organización** una SCP que **deniegue explícitamente** las llamadas de API que borrarían o alterarían la configuración de **CloudTrail** y los objetos del bucket de logs.

---

## 5. IAM Identity Center: single sign-on entre cuentas

Audiencia: 🔧 🧭

Saltar de cuenta en cuenta con *switch role* obliga a iniciar sesión primero en una cuenta. **AWS IAM Identity Center** (sucesor de **AWS Single Sign-On**) resuelve el single sign-on a todas las cuentas de la organización y es el **enfoque recomendado** por AWS para autenticar y autorizar a la fuerza de trabajo, en organizaciones de cualquier tamaño.

**Funciones principales (lectura):**

| Función | Qué hace |
|---|---|
| **Workforce identities** | Crear usuarios y grupos en Identity Center, o conectar y sincronizar con una fuente existente: **Microsoft Active Directory Domain Services**, **Okta Universal Directory**, **Microsoft Azure AD** (el video menciona federar con AD mediante **AWS AD Connector**) |
| **Asignaciones a aplicaciones SAML** | SSO a aplicaciones SAML 2.0 como **Salesforce** y **Microsoft 365** sin federar cada una por separado |
| **Aplicaciones habilitadas para Identity Center** | Servicios como **Amazon Managed Grafana**, **Amazon Monitron** y **SageMaker Studio Notebooks** se conectan solos para login y directorio |
| **Permisos multicuenta** | **Permission sets** definidos una vez y asignados a usuarios en cuentas específicas, sin configurar cada cuenta |
| **AWS access portal** | Portal web con acceso de un clic a todas las cuentas y aplicaciones asignadas |

**Relación con Organizations:** "son los mejores amigos": toda cuenta creada en Organizations aparece lista para asignarse en Identity Center. Al asignar un usuario y un permission set a una cuenta, Identity Center **crea los roles** necesarios en esa cuenta.

Desde el portal, el usuario elige **acceso a la consola** (SSO) o **acceso por línea de comandos/programático**, que entrega access key ID y secret **temporales**.

---

## 6. Servicios de logging

Audiencia: 🔧 🧭

### 6.1 Dos familias de registros

| Tipo | Contenido | Para qué | Dónde / servicios |
|---|---|---|---|
| **Logs de aplicación** | Accesos al servidor web, errores, logs de BD, excepciones del código | Identificar y resolver problemas de aplicación | En los servidores, **CloudWatch Logs** o **S3**. Integran logging con S3/CloudWatch Logs: Lambda, API Gateway, Elastic Load Balancing, hosting estático en S3, CloudFront; el destino puede estar en **otra cuenta** |
| **Logs de infraestructura** | Eventos de cambio (llamadas de API, diferencias antes/después), red y seguridad | Rastro de auditoría y análisis forense | **AWS CloudTrail, AWS Config, VPC Flow Logs, Amazon GuardDuty** |

En CloudWatch Logs se puede consultar desde un lugar central filtrando campos (p. ej. errores **HTTP 404** de los servidores web) y derivar una **métrica**, una **alarma** o detección de **anomalías** con machine learning.

### 6.2 Comparativa de los servicios de infraestructura

| Servicio | Pregunta que responde | Detalles del material |
|---|---|---|
| **AWS CloudTrail** | *¿Quién hizo qué llamada de API, cuándo?* | Registra acciones de consola, CLI, SDK y API de AWS. **No** registra llamadas a un backend propio en EC2 ni llamadas internas de una app que no sean APIs de AWS. Entrega logs en **unos 15 minutos** en promedio (no garantizado). Un **organization trail** desde la management account cubre todas las cuentas; las cuentas miembro lo **ven pero no pueden modificarlo ni borrarlo**, y por defecto **no** acceden a los archivos del bucket |
| **AWS Config** | *¿Cómo está (y estuvo) configurado cada recurso y cómo se relaciona con otros?* | Inventario, historial de configuración y relaciones; **Config rules** que marcan recursos **no conformes** y notifican; evaluación continua al crear/modificar/borrar; usos: gestión de recursos, auditoría y cumplimiento, análisis de impacto de cambios y "última configuración buena conocida", análisis de seguridad (p. ej. saber si un usuario tenía permiso para modificar una VPC en una fecha dada, o qué puertos abría un security group) |
| **VPC Flow Logs** | *¿Qué tráfico IP entra y sale de las interfaces de red?* | Publica en CloudWatch Logs o S3; diagnostica reglas de security group demasiado restrictivas, monitorea tráfico y su dirección; se recolecta **fuera del camino** del tráfico: no afecta rendimiento ni latencia |
| **Amazon GuardDuty** | *¿Hay actividad maliciosa o no autorizada?* | Monitoreo casi continuo de eventos de datos de CloudTrail para S3, eventos de gestión de CloudTrail, logs DNS, datos de volúmenes EBS, logs de auditoría de EKS y VPC Flow Logs; usa inteligencia de amenazas (IP/dominios maliciosos) y ML. Detecta escalada de privilegios, credenciales expuestas, comunicación con IP maliciosas, malware, minería de bitcoin, instancias en Regiones nunca usadas o llamadas inusuales (p. ej. debilitar la política de contraseñas) |

> [!tip] 💡 Analogía
> CloudTrail es el **libro de visitas** (quién entró y qué pidió); Config es el **álbum de fotos** del edificio en cada momento (cómo estaba cada cosa); VPC Flow Logs es el **contador de autos** en la entrada; GuardDuty es el **guardia** que mira todo eso buscando comportamiento sospechoso.

🎯 **Para el examen:** diferenciar CloudTrail (actividad de API) de Config (estado y cumplimiento de configuración); organization trail inmutable para cuentas miembro; Flow Logs sin impacto de rendimiento.

---

## 7. Automatizar el aprovisionamiento: CloudFormation, Service Catalog y Control Tower

Audiencia: 🔧 🧭

### 7.1 El caso de María

Una desarrolladora nueva, **María**, pasa el primer día con soporte TI y el segundo conociendo el código. Necesita un entorno de desarrollo **igual a producción**, sin configurarlo en su portátil: una cuenta AWS con un IDE **AWS Cloud9** y acceso a algunos servicios. ¿Se puede automatizar? Sí.

### 7.2 AWS CloudFormation

Servicio de **infraestructura como código (IaC)**: se escribe una **plantilla** con los recursos deseados (EC2, RDS…) y CloudFormation los aprovisiona y configura, resolviendo las dependencias. Es buena práctica desplegar infraestructura de forma automatizada y no a mano en la consola. **Control Tower y Service Catalog usan plantillas de CloudFormation.**

### 7.3 AWS Service Catalog: la máquina expendedora de recursos

- Los **administradores de portafolio** crean plantillas de CloudFormation → cada una es un **producto**; los productos se agrupan en **portafolios**; usuarios como María navegan y **lanzan** productos en sus cuentas.
- Ejemplo: portafolio *Development Environments* con el producto *Single Instance SQS Queue and API Gateway for Project A* (una t2.micro, una cola SQS y un endpoint de API Gateway apuntando a funciones Lambda *mock*).
- Permite estandarizar tipos de instancia, configuraciones, **estrategia de tags**, guardarraíles: todo lo que CloudFormation soporte.

**Beneficios (lectura):**

| Beneficio | Descripción |
|---|---|
| Estandarización | Restringir dónde se lanza, qué tipo de instancia, etc. |
| Autoservicio | El usuario encuentra y lanza el producto aprobado (*provisioned product*) |
| Control de acceso detallado | Portafolios con restricciones y tags; acceso vía usuarios y grupos IAM |
| Extensibilidad y versiones | Un producto en varios portafolios sin copiarlo; una nueva versión se propaga a todos |

### 7.4 AWS Control Tower: la máquina expendedora de cuentas

Las cuentas nuevas de Organizations nacen **prácticamente vacías**. **Control Tower** automatiza su implementación:

- Orquesta **Organizations, Service Catalog e IAM Identity Center** (entre otros) para construir una **landing zone**, normalmente en **menos de una hora**.
- La landing zone ejecuta CloudFormation automáticamente al crear cuentas: puede desplegar el Service Catalog y todo lo demás sin pasos manuales.
- Evita la **deriva** (*drift*): divergencias y falta de estandarización entre cuentas.
- Aplica **guardrails preventivos y detectivos** (p. ej. asegurar que los logs de seguridad y los permisos entre cuentas se creen y **no se alteren**).
- Usa **CloudFormation StackSets**: una **stack instance por cuenta y Región**.
- Conviene cuando hay **más de un puñado de cuentas**; puede ser la forma principal de aprovisionar cuentas e infraestructura.

| | AWS Service Catalog | AWS Control Tower |
|---|---|---|
| Analogía | Máquina expendedora de **recursos** | Máquina expendedora de **cuentas** |
| Ámbito | Mayormente **dentro** de una cuenta | **Crea** cuentas y las configura al nacer |
| Motor | CloudFormation | CloudFormation (StackSets) + Organizations + Identity Center |

Con Service Catalog se pueden desplegar entornos que ya envían sus **logs de aplicación** a **CloudWatch Logs** en la cuenta de servicios compartidos.

---

## 8. Demos y laboratorios

### 8.1 Demo — Rol IAM entre cuentas

- **Objetivo:** que el usuario IAM `rafael` de la **Cuenta B** asuma un rol en la **Cuenta A** (`developer-account-a`, ID terminado en 8173).
- **Pasos mostrados:**
  1. En la Cuenta A: IAM → **Roles** → **Create role**. Tipo de entidad de confianza: se ven opciones para EC2 y Lambda; se elige **AWS account** (rol *cross-account*) y se pega el **ID de la otra cuenta**.
  2. Permisos: se busca y adjunta **AdministratorAccess** (para un auditor se usaría una política de solo listar, que ya existe).
  3. Nombre `AdminRole` y descripción; **Create role**.
  4. En la **trust relationship**, la consola pone por defecto el **root** de la otra cuenta; se edita a `user/rafael`. El ARN de un usuario IAM tiene la forma `arn:aws:iam::<account-id>:user/<nombre>`.
  5. Se inicia sesión en la Cuenta B como `rafael` → menú superior derecho → **Switch role** → ID de la Cuenta A + `AdminRole` → se ejecuta **AssumeRole** y se opera en la Cuenta A con permisos de administrador.
- **Resultado:** acceso entre cuentas sin crear usuarios duplicados.

### 8.2 Demo — Organizations, OU y SCP

- **Pasos mostrados:**
  1. Consola de Organizations → **Create an organization** (la cuenta actual queda como management account).
  2. Root → Actions → **Create new organizational unit**: `Customer A`; dentro, `Dev Environments`.
  3. **Policies → Service control policies** → habilitar → **Create policy** `t2.micro_only` ("Solo permitir el tipo de instancia t2.micro") con el JSON descrito en la sección 4.
  4. En la OU `Dev Environments` → Policies → **Attach** `t2.micro_only`.
  5. **Add an AWS account** → nombre `Dev Workload A`, correo, rol sugerido `OrganizationAccountAccessRole` → **Create AWS account**; se revisan las solicitudes pendientes y se refresca.
  6. Se **mueve** la cuenta a `Dev Environments`: desde ese momento rige la SCP, incluso sobre root.
  7. **Switch role** con el ID de la nueva cuenta y `OrganizationAccountAccessRole`.
  8. Prueba en EC2: `c5.12xlarge` → **falla** el lanzamiento; `t2.micro` → se lanza. **Switch back** vuelve a la cuenta de servicios compartidos.

### 8.3 Demo — IAM Identity Center

- **Pasos mostrados:**
  1. Habilitar IAM Identity Center; usar su **directorio propio** (alternativa: federar con Active Directory vía AD Connector).
  2. **Users → Add user** `raf`: contraseña de un solo uso (copiarla o que el servicio envíe un correo con instrucciones), correo, datos opcionales; sin grupos.
  3. **Permission sets → Create permission set** → predefinido **AdministratorAccess**.
  4. **AWS accounts**: aparece ya `Dev Workload A` (creada en Organizations) → **Assign users or groups** → `raf` + permission set → **Submit** (se crean los roles en la cuenta).
  5. Copiar la URL del **AWS access portal**, entrar como `raf`, **cambiar la contraseña** en el primer inicio.
  6. El portal lista las cuentas autorizadas con dos opciones: **Management console** (SSO) o **Command line or programmatic access** (credenciales temporales).
  7. En la cuenta se ve la instancia t2.micro creada antes; la SCP sigue vigente.

### 8.4 Demo — CloudTrail centralizado (organization trail)

- **Pasos mostrados:**
  1. Buscar "trail" → CloudTrail → **Create trail** con nombre `centralized-logging`.
  2. Marcar **Enable for all accounts in my organization**; crear un **bucket S3 nuevo**; se **desactiva el cifrado KMS** solo para simplificar la demo.
  3. Tipo de eventos: solo **management events**. **Create trail**.
  4. Se genera tráfico: SSO a `Dev Workload A`, navegar EC2 (instancias, tipos), S3, crear un bucket, crear una cola SQS.
  5. Minutos después, en la cuenta de servicios compartidos: bucket `aws-cloudtrail-logs-<hash>` con la ruta `AWSLogs/o-<ID de organización>/<ID de cuenta>/CloudTrail/us-east-1/2022/…` y archivos **comprimidos** por cuenta (servicios compartidos terminaba en 3061; la cuenta de desarrollador en 4718).
- **Resultado:** los datos de CloudTrail de **todas** las cuentas, existentes o nuevas, en un único bucket de la cuenta central, configurado con un clic.
- **Mejora asociada:** SCP en la raíz que impida alterar CloudTrail y su bucket.

---

## 9. Solución final, plan de implementación y optimizaciones

Audiencia: 🔧 🧭 👔

### 9.1 Arquitectura propuesta

```
 ┌──────────────── Shared Services (management account) ────────────────┐
 │ AWS Organizations (+ SCP)   IAM Identity Center   AWS Control Tower   │
 │ AWS CloudTrail (organization trail → S3)   CloudWatch Logs (central)  │
 └──────────┬──────────────────────┬─────────────────────┬────────────────┘
            │ crea cuentas / SCP   │ SSO                 │ landing zone (StackSets)
   ┌────────▼────────┐    ┌────────▼────────┐    ┌───────▼──────────┐
   │ Developer acct  │    │ Workload A dev  │    │ Workload A prod  │  …
   │ IAM role (auto) │    │ Service Catalog │    │ solo CCoE        │
   │ Service Catalog │──► logs de aplicación → CloudWatch Logs central
   └─────────────────┘    └─────────────────┘    └──────────────────┘
```

| Requisito | Servicio |
|---|---|
| R1 Multicuenta con aprovisionamiento automático | AWS Organizations + AWS Control Tower |
| R2 Cuenta de servicios compartidos | Management account con los servicios de gobierno |
| R3 Single sign-on | IAM Identity Center |
| R4 Guardarraíles y configuración estándar | SCP, guardrails de Control Tower, Service Catalog |
| R5 Logging centralizado | CloudTrail organization trail + CloudWatch Logs central |

### 9.2 Plan de implementación (tres grandes pasos)

"No es algo que se haga en una tarde": lo presentado es el **andamiaje**.

1. Diseñar el plan y crear los **stacks de infraestructura como código**.
2. **Migrar cargas** a las cuentas nuevas, ganando experiencia con IaC y despliegue de cuentas.
3. Con madurez, migrar **bases de datos** a las cuentas nuevas, p. ej. con **AWS DMS** para mantenerlas en línea (ver [[07-Solucion-Hibrida-Contenedores]]).

### 9.3 Llevar la arquitectura al siguiente nivel

| # | Mejora | Detalle |
|---|---|---|
| M1 | **SCP que restringe al usuario root** de las cuentas miembro | Las cuentas nuevas tienen un root con contraseña desconocida, pero recuperable con "¿Olvidó su contraseña?"; una SCP puede negar al principal root una lista de acciones (o todas) |
| M2 | **SCP que impide deshabilitar CloudTrail** | En la **OU raíz** para cubrir todas las cuentas, incluida la de servicios compartidos |
| M3 | **SCP que niega inicios de sesión fuera de IP conocidas** | Con **condiciones** en la política: solo IP de la oficina o de los endpoints de la VPN corporativa usada por el CCoE |
| M4 | **Tag policies** de Organizations | Etiquetado uniforme en todas las cuentas. Las tags **distinguen mayúsculas**: `CostCenter` y `Costcenter` no se agrupan juntas |
| M5 | **Billing alarms por cuenta** | En CloudWatch de la cuenta de servicios compartidos hay una métrica de facturación por cuenta (10, 15, 50…) y se crean alarmas por subcuenta: atrapa instancias olvidadas o tipos equivocados |
| M6 | **MFA** en el SSO de Identity Center | Segundo factor además de usuario y contraseña |
| M7 | **SCP por OU** según necesidades | Dev más permisivo para experimentar; prod más restringido; contabilidad distinta de desarrollo |
| M8 | **SCP que impide mover cuentas entre OU** salvo a usuarios autorizados | Evita el truco de mover una cuenta de Prod a Dev, lanzar el recurso prohibido y devolverla |

### 9.4 Tag policies vs. SCP para el etiquetado

| | Tag policy | SCP con condición de tag |
|---|---|---|
| Qué impide | **Cambiar** una tag estandarizada en recursos existentes | **Crear** recursos sin la tag requerida |
| Qué **no** impide | Lanzar recursos nuevos sin tag o con tag no conforme | — |
| Cuándo basta | Si todo se aprovisiona con **IaC** (las tags van en la plantilla) | Cuando hay creación manual |

**Ejemplo de tag policy** (lectura): tag con clave `Environment`, valor `Production`, aplicada a instancias EC2. La lectura la muestra con claves traducidas automáticamente; con la sintaxis real de AWS sería:

```json
{
  "tags": {
    "Environment": {
      "tag_key":   { "@@assign": "Environment" },
      "tag_value": { "@@assign": ["Production"] },
      "enforced_for": { "@@assign": ["ec2:instance"] }
    }
  }
}
```

**Ejemplo de SCP** (lectura): cuatro declaraciones `Deny` que exigen las tags `Project` y `CostCenter` al crear secretos (`secretsmanager:CreateSecret`) y al lanzar instancias (`ec2:RunInstances` sobre `instance/*` y `volume/*`), usando la condición `Null` sobre `aws:RequestTag/<tag>`. Fragmento de una de ellas:

```json
{
  "Sid": "DenyRunInstanceWithNoProjectTag",
  "Effect": "Deny",
  "Action": "ec2:RunInstances",
  "Resource": ["arn:aws:ec2:*:*:instance/*", "arn:aws:ec2:*:*:volume/*"],
  "Condition": { "Null": { "aws:RequestTag/Project": "true" } }
}
```

> [!note] ➕ Nota del editor
> En la lectura en español los JSON aparecen con claves traducidas («Efecto», «Denegar», «GestorDeSecretos:CrearSecreto»…), lo que los hace inválidos. Aquí se restituyen los nombres reales de la sintaxis de AWS (`Effect`, `Deny`, `secretsmanager:CreateSecret`, `@@assign`). El mensaje de la lectura: **SCP y tag policies pueden y deben usarse juntas**.

La lectura de buenas prácticas también sugiere estructuras de OU por caso de uso (OU corporativa, OU de producción pública, OU de producción interna, OU de seguridad).

---

## 10. Cierre del curso 2: recapitulación de las cuatro semanas

| Semana | Cliente | Solución | Servicios clave | Tomo |
|---|---|---|---|---|
| 1 | Backend web | Serverless, event-driven y desacoplado | API Gateway, Lambda, DynamoDB, SQS, SNS | [[05-Backend-Web-Serverless]] |
| 2 | Restaurantes | Analítica de datos serverless | Kinesis Data Firehose, S3, Athena, QuickSight | [[06-Analitica-de-Datos-Serverless]] |
| 3 | Aseguradora | Híbrida con contenedores | ECS Anywhere, Systems Manager, DMS, Storage Gateway | [[07-Solucion-Hibrida-Contenedores]] |
| 4 | Agencia de marketing | Gobierno multicuenta | Organizations, IAM Identity Center, CloudTrail, Service Catalog, Control Tower | este tomo |

---

## ✅ Autoevaluación

> [!question]- 1. Nombre cuatro de las seis ventajas de una estrategia multicuenta.
> Agrupar por propósito de negocio y propiedad; centralizar logging; restringir acceso a datos sensibles; limitar el radio de impacto; gestionar mejor costos; distribuir cuotas de servicio y límites de API.

> [!question]- 2. ¿Qué diferencia hay entre la trust policy y la permissions policy de un rol?
> La trust policy (basada en recursos, adjunta al rol) dice **quién** puede asumirlo; la permissions policy dice **qué** puede hacer quien lo asume. En acceso entre cuentas, además, el usuario necesita en su cuenta una política que le permita asumir el rol.

> [!question]- 3. Una SCP deniega `RunInstances` salvo t2.micro. ¿Puede el usuario root de la cuenta miembro lanzar una c5.12xlarge?
> No: una denegación explícita en una SCP aplica a toda la cuenta, incluido el root.

> [!question]- 4. ¿Qué rol crea Organizations por defecto en una cuenta nueva y para qué sirve?
> `OrganizationAccountAccessRole`, que permite entrar desde la management account a la nueva cuenta mediante switch role / AssumeRole.

> [!question]- 5. ¿Por qué IAM Identity Center es mejor que switch role para muchas cuentas?
> Porque da single sign-on desde un portal con usuarios centralizados (propios o federados con AD, Okta, Azure AD), asigna permission sets a varias cuentas a la vez y entrega credenciales temporales para consola o CLI, sin depender de usuarios IAM en una cuenta de entrada.

> [!question]- 6. ¿Qué no registra CloudTrail?
> Llamadas de API a un backend propio alojado en EC2 y llamadas internas de una aplicación que no son APIs de AWS.

> [!question]- 7. Distinga AWS Config de CloudTrail en una frase cada uno.
> CloudTrail registra **quién hizo qué llamada de API**; Config registra **cómo está y estuvo configurado** cada recurso, sus relaciones, y evalúa reglas de cumplimiento.

> [!question]- 8. ¿Cuál es la diferencia de ámbito entre Service Catalog y Control Tower?
> Service Catalog ofrece productos aprobados (plantillas CloudFormation) para lanzar dentro de una cuenta; Control Tower crea y configura cuentas completas (landing zone, guardrails, StackSets) y puede desplegar en ellas el Service Catalog.

> [!question]- 9. ¿Basta una tag policy para impedir recursos sin etiquetar?
> No: impide cambiar tags estandarizadas en recursos existentes, pero no lanzar recursos sin tags. Para eso se necesita una SCP con `Deny` y condición `Null` sobre `aws:RequestTag/...`.

> [!question]- 10. ¿Qué truco evita una SCP sobre el movimiento de cuentas entre OU?
> Que alguien mueva una cuenta de Prod a Dev, lance allí un recurso prohibido en Prod y la devuelva.

---

## 🧾 Resumen en 10 líneas

1. La agencia de marketing tiene todo en una sola cuenta, usa root y no controla accesos ni etiquetas: el arquitecto **define los requisitos** por ella.
2. Una cuenta es un límite de seguridad, acceso y facturación; multicuenta da aislamiento, costos claros, cuotas repartidas y menor radio de impacto (pilares Well-Architected).
3. Estrategia elegida: OU por **cliente** y, dentro, por **entorno** (dev/test/prod); producción solo para el CCoE.
4. Requisitos de base con varias cuentas: **automatización** y **credenciales centralizadas**.
5. **Roles IAM** con trust policy + permissions policy dan acceso temporal entre cuentas sin duplicar usuarios.
6. **AWS Organizations** crea y elimina cuentas, agrupa en OU y aplica **SCP** que rigen incluso sobre root.
7. **IAM Identity Center** da SSO multicuenta con permission sets y portal de acceso; se integra con Organizations.
8. **CloudTrail** como organization trail centraliza la actividad de API; Config, VPC Flow Logs y GuardDuty completan el logging de infraestructura.
9. **Control Tower** (landing zone, guardrails, StackSets) y **Service Catalog** (portafolios de productos CloudFormation) hacen que cada cuenta nazca estandarizada.
10. Mejoras: SCP sobre root, CloudTrail, IP de origen y movimiento de cuentas; tag policies; billing alarms por cuenta; MFA.

---

## 📖 Recursos del módulo

- [Establishing Your Cloud Foundation on AWS (whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/establishing-your-cloud-foundation-on-aws/welcome.html)
- [AWS Well-Architected Framework](http://aws.amazon.com/architecture/well-architected/)
- [Build on AWS (constructores)](http://aws.amazon.com/campaigns/build-on-aws/)
- [Organizing Your AWS Environment Using Multiple Accounts (whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)
- [Términos y conceptos de roles (IAM)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html)
- [¿Qué es IAM Identity Center?](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [Cómo funciona CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/how-cloudtrail-works.html)
- [¿Qué es AWS Config?](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
- [Registro del tráfico IP con VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [¿Qué es Amazon GuardDuty?](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
- [¿Qué es AWS CloudFormation?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [Implementación de infraestructura multinivel fiable con CloudFormation (Well-Architected Labs)](https://www.wellarchitectedlabs.com/reliability/100_labs/100_deploy_cloudformation/)
- [Servicios integrados con AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/integrated-services.html)
- [¿Qué es AWS Control Tower?](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html)
- [¿Qué es AWS Service Catalog?](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/introduction.html)
- [Best practices for AWS Organizations service control policies in a multi-account environment (blog)](https://aws.amazon.com/blogs/industries/best-practices-for-aws-organizations-service-control-policies-in-a-multi-account-environment/)
- [Establishing your best practice AWS environment](https://aws.amazon.com/organizations/getting-started/best-practices/)
- [Requerir una etiqueta en los recursos creados especificados (SCP de ejemplo)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples_tagging.html#example-require-tag-on-create)

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[07-Solucion-Hibrida-Contenedores|07 · Solución híbrida con contenedores]] · Siguiente: [[09-Introduccion-a-Data-Lakes|09 · Introducción a data lakes ➡]]
