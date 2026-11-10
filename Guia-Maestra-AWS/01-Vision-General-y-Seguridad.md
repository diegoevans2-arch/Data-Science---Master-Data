---
title: "Tomo 01 — Visión general y seguridad de AWS"
tags: [aws, solutions-architect, cloud-fundamentals, global-infrastructure, iam, shared-responsibility]
curso: 1 · AWS Cloud Technical Essentials
modulo: "Semana 1: Visión general y seguridad de AWS"
audiencias: [tecnico, puente, ejecutivo]
tomo: 01
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# ☁️ Tomo 01 — Visión general y seguridad de AWS

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[00-MOC-Guia-Maestra-AWS|00 · Índice]] · Siguiente: [[02-Computo-y-Redes|02 · Cómputo y Redes ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Es el piso sobre el que se construye todo el certificado. Aquí se fija el vocabulario (Region, Availability Zone, edge location, API call), la lógica económica de la nube (las seis ventajas) y, sobre todo, la seguridad desde el día uno: quién responde por qué (shared responsibility model), cómo blindar el root user y cómo funciona IAM (users, groups, policies, roles). Sin esto, cada servicio posterior —EC2, VPC, S3, RDS— se usa a ciegas. También arranca la aplicación de ejemplo, **Employee Directory**, que acompaña todo el curso.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** en qué Region alojar una carga (cumplimiento, latencia, precio, servicios disponibles), qué parte de la seguridad se delega en AWS y cuál queda en casa, cómo se reparte el acceso entre equipos sin compartir contraseñas.
> - **Costo de hacerlo mal:** una cuenta con el root user sin MFA es una invitación a que un tercero borre los datos y monte minería de criptomonedas con cargo a la factura propia; elegir mal la Region puede incumplir la soberanía de datos o encarecer la operación; credenciales permanentes incrustadas en código son una fuga esperando ocurrir.
> - **Pregunta que responde:** *¿qué es exactamente "la nube", dónde viven físicamente mis recursos y quién —y con qué permisos— puede tocarlos?*

> [!tip] 💡 Analogía general: el edificio de departamentos
> AWS es la constructora y administradora de un edificio enorme repartido por el mundo. Ella responde por los cimientos, los muros, los guardias de la entrada y los ductos (seguridad **de** la nube). Usted arrienda un departamento: decide quién recibe llaves, cierra su puerta y cuida lo que guarda dentro (seguridad **en** la nube). La llave maestra del contrato (root user) se guarda en caja fuerte; para el día a día se entregan llaves personales con acceso solo a las piezas necesarias (IAM users y groups), y a los técnicos que vienen a reparar se les da una tarjeta temporal que vence sola (IAM roles).

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Bienvenido al curso | 3 videos · 3 lecturas · 2 encuestas | Alcance del curso, temario de las 4 semanas, instructores (Morgan Willis y Seph), dónde pedir ayuda (re:Post) |
| Introducción a la nube de AWS | 3 videos · 3 lecturas · 1 coach ("Descubrir AWS") | Definición de cloud computing, seis ventajas, infraestructura global (Regions, AZ, edge), formas de interactuar (Console, CLI, SDK) |
| Seguridad en la nube de AWS | 2 videos · 2 lecturas · 1 coach (prácticas recomendadas) | Shared responsibility model, root user, autenticación vs autorización, MFA |
| Administración de identidades y acceso de AWS | 2 videos · 2 lecturas | IAM: users, groups, policies (JSON), roles, federación, IAM Identity Center, buenas prácticas |
| Semana 1: ejercicio y evaluación | 3 videos · 2 lecturas · 1 lab calificado · 1 quiz · 1 encuesta | Lab 1 de IAM, demo IAM (role + user + access keys), lanzamiento de Employee Directory en EC2, cambio de AMI por defecto |

---

## 1. El curso y la aplicación de ejemplo

Audiencia: 🔧 🧭

**AWS Cloud Technical Essentials** es un curso de nivel fundamental, pensado para profesionales de TI que conocen los temas generales pero aún no la nube de AWS. No requiere programar (el código fuente está disponible para quien quiera explorarlo). Lo dictan **Morgan Willis** (Principal Cloud Technologist) y **Seph** (Cloud Technologist), acompañados en pantalla por las mascotas Meowzy y Fluffy, que aparecen con un sonido de notificación para dar definiciones, tips y buenas prácticas.

**Temario por semana:**

| Semana | Tema | Servicios eje |
|---|---|---|
| 1 | Introducción a la nube, infraestructura global, interacción con AWS, IAM | IAM, EC2 (primer vistazo) |
| 2 | Cómputo y redes | EC2, contenedores (Amazon ECS), serverless (AWS Lambda), Amazon VPC |
| 3 | Almacenamiento y bases de datos | Amazon S3, Amazon EBS, Amazon RDS, Amazon DynamoDB |
| 4 | Monitoreo, optimización y serverless | Amazon CloudWatch, Elastic Load Balancing, Amazon EC2 Auto Scaling; escalado vertical vs horizontal |

**¿Dónde pedir ayuda?**

| Tipo de duda | Canal |
|---|---|
| Preguntas sobre servicios de AWS o sobre un proyecto propio | **AWS re:Post** (`repost.aws`), comunidad de preguntas y respuestas |
| Contenido del curso desactualizado, incorrecto o roto (p. ej., instrucciones de un ejercicio que ya no calzan con la consola) | Formulario de tickets de los creadores del curso |
| Problemas dentro de la propia cuenta de AWS durante un lab | No va por ticket: el equipo del curso no tiene acceso a su cuenta |
| Certificado o plataforma de aprendizaje | Soporte de la plataforma (Coursera) |

> [!warning] ⚠️ AWS cambia más rápido que el curso
> Los propios instructores advierten que la consola y las funciones evolucionan; parte del material puede verse distinto. El caso concreto de este módulo es la AMI por defecto de EC2 (sección 9.3).

### 1.1 Employee Directory: qué hace

Es una aplicación web **CRUD** (create, read, update, delete) que lleva el registro de los empleados de una empresa ficticia. En la demo Morgan:

1. Crea un empleado con nombre, ubicación (EE. UU.) y cargo (Cloud Technologist).
2. Le asigna *badges* (insignias), por ejemplo "Mac User" y "Photographer", y guarda.
3. Edita el registro para agregar la foto que olvidó.
4. Muestra que también se pueden eliminar empleados.

### 1.2 Arquitectura objetivo (la que se construirá durante el curso)

| Pieza | Servicio | Función |
|---|---|---|
| Red privada | Amazon VPC | Contiene servidores y base de datos |
| Backend | Amazon EC2 | Máquinas virtuales que ejecutan la app (Python + Flask) |
| Datos de empleados | Amazon RDS (más adelante también DynamoDB) | Base de datos dentro de la red |
| Fotos | Amazon S3 | Almacenamiento de objetos, ilimitado para cualquier tipo de archivo |
| Monitoreo | Amazon CloudWatch | Observar la solución |
| Escalabilidad y tolerancia a fallos | Elastic Load Balancing + Amazon EC2 Auto Scaling | Repartir tráfico y ajustar capacidad |
| Seguridad e identidad | AWS IAM | Controlar quién y qué accede |

```
                    Usuarios (navegador)
                           │
                 ┌─────────▼─────────┐
                 │ Elastic Load Bal. │
   ┌─────────────┴───────────────────┴──────────────┐
   │ Amazon VPC                                     │
   │   ┌────────────── EC2 Auto Scaling ─────────┐  │
   │   │  [EC2 Flask app]      [EC2 Flask app]   │  │       ┌───────────┐
   │   └───────────────┬─────────────────────────┘  │──────▶│ Amazon S3 │ (fotos)
   │                   ▼                            │       └───────────┘
   │             [Amazon RDS]                       │
   └────────────────────────────────────────────────┘
        IAM (identidad y permisos)  ·  CloudWatch (monitoreo)
```

El diagrama se irá modificando a medida que aparezcan nuevas ideas; se construye paso a paso con la AWS Management Console.

---

## 2. Qué es la nube

Audiencia: 🔧 🧭 👔

**Antes:** las organizaciones compraban y mantenían su propio hardware de cómputo, almacenamiento y red en sus centros de datos, con departamentos enteros dedicados a ello. Era caro, lento y volvía inviables ciertas cargas y la experimentación. Con la masificación de Internet la demanda creció y, para muchos, sostener esa presencia física dejó de ser viable.

**Definición del curso:** *cloud computing* es la **entrega bajo demanda de recursos de TI a través de Internet con precios de pago por uso**. El proveedor (AWS) es dueño de los centros de datos, los mantiene y ofrece tecnología de centro de datos virtualizada como servicio. En este contexto, "los recursos de TI" son precisamente los servicios de AWS.

### 2.1 On-premises vs nube: el caso del entorno de QA

Los desarrolladores necesitan probar una nueva funcionalidad en un entorno de QA idéntico a producción.

| Paso | On-premises | Nube |
|---|---|---|
| Conseguir infraestructura | Comprar e instalar hardware | Se aprovisiona lógicamente por Internet |
| Montaje | Cableado, energía, sistema operativo… | No hay montaje físico |
| Tiempo | Días | Minutos o incluso segundos |
| Réplicas adicionales | Repetir todo el proceso | Replicar el entorno cuantas veces se necesite |
| Efecto en el negocio | Aumenta el *time to market*, los devs esperan | El entorno no es el cuello de botella |

### 2.2 Undifferentiated heavy lifting

En toda aplicación hay partes que distinguen al negocio (el código, la lógica) y partes que son iguales en cualquier empresa (el cómputo sobre el que corre, instalar máquinas virtuales, guardar respaldos). Ese trabajo repetitivo que no diferencia se llama **undifferentiated heavy lifting**; la nube lo traslada a AWS para que el equipo se concentre en lo estratégicamente único.

> [!tip] 💡 Analogía
> Nadie gana clientes por tener la mejor sala eléctrica; los gana por su producto. Pagar a otro para que mantenga la sala eléctrica libera horas para el producto.

### 2.3 Las seis ventajas de la nube

| # | Ventaja | Idea central |
|---|---|---|
| 1 | **Pay as you go** (pague a medida que usa) | Se paga solo cuando se usan los recursos y solo por la cantidad usada; no se invierte antes de saber cómo se usarán |
| 2 | **Economías de escala masivas** | El uso agregado de cientos de miles de clientes permite a AWS bajar el precio por unidad más de lo que lograría una sola empresa |
| 3 | **Dejar de adivinar la capacidad** | Sin sobredimensionar (recursos ociosos caros) ni quedarse corto: se escala hacia arriba o abajo con minutos de aviso |
| 4 | **Velocidad y agilidad** | Recursos "a un clic": de semanas a minutos para entregarlos a los desarrolladores; experimentar cuesta menos tiempo y dinero |
| 5 | **Dejar de gastar en operar centros de datos** | Foco en clientes y proyectos diferenciadores, no en montar racks, apilar y alimentar infraestructura |
| 6 | **Global en minutos** | Desplegar en varias Regions con pocos clics: menor latencia y mejor experiencia a costo mínimo |

> [!example] 📊 Caso: Employee Directory
> El objetivo declarado es usar servicios de AWS para diseñar una infraestructura **escalable, altamente disponible y costo-eficiente**, y así lanzar la aplicación al mundo rápido y sin administrar hardware físico.

🎯 **Para el examen:**
- Reconocer cada una de las seis ventajas por su descripción (p. ej., "trade capital expense for variable expense" ≈ pay as you go; "stop guessing capacity").
- Saber qué es *undifferentiated heavy lifting*.

---

## 3. Infraestructura global de AWS

Audiencia: 🔧 🧭

La infraestructura física (centros de datos, conectividad) sigue existiendo: es la base de toda aplicación en la nube. En AWS se organiza de forma **redundante y anidada**.

### 3.1 De la foto en el portátil a la Region

El razonamiento del video, con las fotos de los empleados:

1. Las fotos solo están en un portátil → si se rompe, se pierden. Se suben a AWS: quedan copias y se accede desde cualquier lugar.
2. En AWS quedan en servidores dentro de **un centro de datos**. ¿Y si un desastre (o, en la broma del curso, un ataque alienígena) lo destruye? → AWS tiene un **segundo centro de datos** conectado por enlaces redundantes de alta velocidad y baja latencia.
3. Ese clúster de centros de datos es una **Availability Zone (AZ)**: uno o más centros de datos con energía, red y conectividad redundantes. Operan en instalaciones separadas con ubicaciones no divulgadas.
4. Un huracán podría afectar a una AZ completa → AWS agrupa varias AZ, conectadas también por enlaces redundantes de alta velocidad y baja latencia. Ese clúster de AZ es una **Region**.

```
 Region (p. ej. us-east-1, N. Virginia)
 ┌──────────────────────────────────────────────────────┐
 │  AZ us-east-1a        AZ us-east-1b       AZ us-east-1c│
 │ ┌──────────────┐    ┌──────────────┐    ┌──────────────┐│
 │ │ [DC] [DC]    │◀══▶│ [DC]         │◀══▶│ [DC] [DC]    ││
 │ └──────────────┘    └──────────────┘    └──────────────┘│
 │      enlaces redundantes, alta velocidad, baja latencia │
 └──────────────────────────────────────────────────────┘
        Data center ⊂ Availability Zone ⊂ Region
```

### 3.2 Regions y sus códigos

- Una **Region** es una ubicación geográfica donde AWS aloja centros de datos; se nombra por el lugar (Región de N. Virginia, Región de Oregón). Hay Regions en América del Norte, Asia-Pacífico, Canadá, Europa, Medio Oriente y Sudamérica, y la red sigue creciendo.
- Cada Region tiene un **nombre geográfico** y un **código de Region**:

| Código | Significado | Nombre geográfico |
|---|---|---|
| `us-east-1` | Primera Region creada en el este de EE. UU. | N. Virginia |
| `ap-northeast-1` | Primera Region creada en el noreste de Asia-Pacífico | Tokio |
| `sa-east-1` | (aparece en los ejemplos de AZ) | São Paulo |
| `eu-west-3` | (aparece en la demo de la consola) | París |

- **Las Regions son independientes entre sí:** los datos no se replican de una Region a otra sin consentimiento y autorización explícitos del cliente.

### 3.3 Códigos de AZ

Una AZ se nombra con el código de la Region más una letra:

- `us-east-1a` → AZ "a" de N. Virginia.
- `sa-east-1b` → AZ "b" de São Paulo.
- Un recurso en `us-east-1c` está en la AZ "c" de `us-east-1`.

### 3.4 Cómo elegir una Region: los cuatro factores

| Factor | Pregunta | Detalle del material |
|---|---|---|
| **Compliance** (cumplimiento) | ¿Hay normativa que obligue a que los datos residan en un territorio? | Se mira **primero**. Si los datos deben quedarse en el Reino Unido → Región de Londres, "punto final": los demás factores dejan de importar. En Canadá podría exigirse Canada (Central) |
| **Latencia** | ¿Qué tan cerca están los recursos de los usuarios? | Todo está limitado por la velocidad de la luz: usuarios en Oregón → Región de Oregón, no Brasil. Las apps síncronas (juegos, telefonía, WebSockets, IoT) sufren mucho con la latencia; incluso las asíncronas (e-commerce) se ven afectadas |
| **Precio** | ¿Cuánto cuesta allí? | Varía por Region según economía local, impuestos, conectividad, precio de equipos importados, aduanas, bienes raíces. São Paulo es un ejemplo de Region más cara por su estructura tributaria. AWS no cobra tarifa plana mundial |
| **Disponibilidad de servicios** | ¿Existe allí el servicio que necesito? | Los servicios y funciones nuevos no se lanzan en todas las Regions a la vez; la documentación tiene una tabla de servicios por Region |

> [!warning] ⚠️ Regla crítica
> El orden importa: **compliance manda**. Solo si no hay restricción regulatoria se compara latencia, precio y servicios. Las fotos de Employee Directory no tienen restricción normativa, por eso el curso pasa directamente a latencia.

> [!note] ➕ Nota del editor
> El video enumera los factores en el orden cumplimiento → latencia → precio → servicios; la lectura los lista como latencia, precio, disponibilidad y conformidad. Es la misma lista; la prioridad explícita (cumplimiento primero) la da el video.

### 3.5 Alcance (scope) de los servicios

Según el servicio, los recursos se despliegan a nivel **AZ**, **Region** o **Global**, y eso cambia la arquitectura:

| Scope | Cómo se reconoce | Quién se ocupa de la durabilidad/disponibilidad |
|---|---|---|
| Region | Solo se elige la Region; no se pide AZ | AWS toma acciones automáticas para aumentar durabilidad y disponibilidad |
| AZ | El servicio pide especificar una AZ | A menudo es **usted** quien debe replicar para lograr alta disponibilidad |
| Global | No depende de Region (ej. del módulo: **IAM**) | — |

### 3.6 Mantener la resiliencia

- Buena práctica conocida: **preferir servicios administrados de alcance regional**.
- Si no es posible, **replicar la carga en varias AZ; como mínimo, dos**. Si una AZ completa falla, la otra toma el tráfico.

### 3.7 Edge network: edge locations y regional edge caches

Además de Regions y AZ existe la **red global de borde**: *edge locations* y *regional edge caches*, que **cachean contenido cerca de los usuarios** para reducir latencia.

Ejemplo del curso: un sitio web alojado en la Región de Ohio con usuarios en todo el mundo. Sin caché, cada usuario lejano viaja hasta Ohio y vuelve; con caché, recibe una copia desde la edge location más cercana. El servicio que se menciona para esto es **Amazon CloudFront**.

```
 Usuario en Asia ──▶ [Edge location cercana] ──(si no está en caché)──▶ Region us-east-2 (Ohio)
                         ▲ respuesta rápida desde caché
```

🎯 **Para el examen:**
- Jerarquía: data center ⊂ AZ ⊂ Region; AZ = uno o más DC con energía/red redundantes.
- Los cuatro factores de elección de Region y que compliance prima.
- Alta disponibilidad: al menos dos AZ.
- Edge locations + CloudFront = caché cerca del usuario para reducir latencia.
- Los datos no salen de una Region sin acción explícita del cliente.

---

## 4. Formas de interactuar con AWS

Audiencia: 🔧 🧭

Cuando la infraestructura se vuelve virtual, ya no se administra físicamente sino **lógicamente, a través de la API de AWS**. Crear, borrar o modificar cualquier recurso es una **llamada a la API**, y **cada llamada es autenticada y autorizada**. Hay tres formas principales de hacerla:

| Herramienta | Qué es | Ventajas | Desventajas / cuándo no |
|---|---|---|---|
| **AWS Management Console** | Interfaz web, apuntar y hacer clic | Ideal para empezar; no requiere scripts ni sintaxis; muestra servicios recientes y categorías (compute, storage, database, security…) | Repetir configuraciones a mano invita al error humano (una casilla olvidada, un valor mal escrito) |
| **AWS CLI** | Herramienta de línea de comandos unificada, open source, para Windows, Linux y macOS; también accesible vía **AWS CloudShell** desde la consola | Un comando reemplaza varias pantallas; scripts repetibles y automatizables; menos errores accidentales | Costo inicial de aprender la sintaxis |
| **AWS SDKs** | Bibliotecas open source mantenidas por AWS (C++, Go, Java, JavaScript, .NET, Node.js, PHP, Python, Ruby…) | Integrar AWS en el código de la aplicación con condiciones, bucles, listas, etc. | Requiere programar |

### 4.1 La consola y las Regions

El **selector de Region** está en la esquina superior derecha. Al cambiarlo, las solicitudes van a otra Region y la URL cambia de subdominio: con París, el navegador apunta a `eu-west-3.console.aws.amazon.com`.

### 4.2 CLI: sintaxis y ejemplo

Estructura de un comando:

```
aws <servicio> <comando> [opciones]
aws ec2 describe-instances
```

Escenario de la lectura: decenas de servidores de frontend cuyos datos cambian; en vez de copiar y pegar desde la consola cada día, se programa un script de CLI que extrae el informe. La respuesta viene en JSON:

```json
{
  "Reservations": [
    {
      "Groups": [],
      "Instances": [
        { "AmiLaunchIndex": 0,
          ... }
```

> [!note] ➕ Nota del editor
> La lectura extraída no conserva la línea del comando, solo la respuesta; `aws ec2 describe-instances` es el comando que corresponde a esa salida (`Reservations` → `Instances`). Las claves del JSON se dejan en inglés, como las devuelve la API (la traducción automática las mostraba como "Reservas", "Grupos", "Instancias").

### 4.3 SDK: ejemplo con Python (boto3)

Employee Directory corre en **Python con Flask**; para subir las fotos a un servicio de almacenamiento se usaría el SDK de Python. Ejemplo de la lectura:

```python
import boto3
ec2 = boto3.client('ec2')
response = ec2.describe_instances()
print(response)
```

> [!tip] 💡 Regla práctica del curso
> Consola para aprender y explorar; CLI o SDK cuando ya conoce el servicio o trabaja en producción con gestión de riesgo (repetibilidad, menos error humano). El curso usa sobre todo la consola, pero invita a practicar con la CLI.

🎯 **Para el examen:**
- Toda acción en AWS = llamada a la API autenticada y autorizada.
- Console / CLI / SDK; CloudShell = CLI en el navegador sin instalar nada.

---

## 5. Shared responsibility model

Audiencia: 🔧 🧭 👔

¿Quién es responsable de la seguridad, el cliente o AWS? Respuesta del curso: **ambos**. AWS no ve una solución como un bloque único, sino como capas que se apoyan unas sobre otras; unas las asegura AWS y otras el cliente.

| | **AWS: seguridad DE la nube** | **Cliente: seguridad EN la nube** |
|---|---|---|
| Qué cubre | Regions, AZ y centros de datos, hasta la seguridad física de los edificios (hormigón, rejas, guardias); la red troncal global (fibra privada entre Regions); hardware, software y red que ejecutan los servicios: servidor físico, host OS, capa de virtualización | Configurar bien cada servicio y las aplicaciones; proteger sus datos |
| Ejemplo EC2 | Host físico e hipervisor; parchear el host OS o el hipervisor es tarea de AWS | Parchear el SO de las instancias, cifrar datos en tránsito y en reposo, configurar firewalls, controlar quién accede y con cuánto acceso |

> [!tip] 💡 Analogía del curso
> La constructora garantiza que el edificio sea estable y seguro; quien arrienda el departamento cierra su puerta. La seguridad del edificio y la de su departamento son cosas distintas.

**Lo principal:** el cliente es **dueño de sus datos** y responde por que estén cifrados, seguros y con controles de acceso adecuados. Muchas veces AWS ofrece las funciones nativas, pero activarlas depende del cliente.

### 5.1 La responsabilidad cambia según el tipo de servicio

| Categoría | Ejemplo | AWS administra | El cliente administra |
|---|---|---|---|
| **Infrastructure services** | Amazon EC2 | Infraestructura y servicios base | SO y plataforma de aplicaciones; cifrar, proteger y gestionar los datos |
| **Container services** | Amazon RDS | Lo anterior + SO + plataforma de aplicaciones | Datos del cliente: cifrado, protección con firewalls de red y respaldos |
| **Abstracted services** | Amazon S3 | Infraestructura, SO, plataformas, **y cifrado y protección de datos del lado del servidor** | Gestionar los datos y protegerlos con cifrado del lado del cliente |

> [!warning] ⚠️ "Container services" no es Docker
> En esta clasificación, *container services* significa que AWS abstrae los contenedores de aplicación tras bambalinas (como en RDS), no los servicios de contenedores tipo Docker.

**Ejemplos de responsabilidades del cliente en contexto:**
- Elegir la Region según las normas de soberanía de datos.
- Implementar protección de datos: cifrado y gestión de respaldos.
- Usar control de acceso para limitar quién llega a datos y recursos.

Además, conviene revisar cómo se alinea el modelo con los estándares de seguridad propios y con la ley aplicable.

🎯 **Para el examen:**
- "Security OF the cloud" (AWS) vs "security IN the cloud" (cliente).
- Parchear el SO invitado de EC2 = cliente; parchear el hipervisor = AWS.
- Cuanto más abstracto el servicio, menos responsabilidad operacional del cliente (pero los datos y el acceso siempre son suyos).

---

## 6. Proteger el root user

Audiencia: 🔧 🧭

### 6.1 Autenticación vs autorización

| Concepto | Pregunta que responde | Ejemplos |
|---|---|---|
| **Authentication** | ¿Es usted quien dice ser? | Usuario y contraseña (lo más común), tokens, biometría (huella) |
| **Authorization** | ¿Qué acciones puede realizar? | Leer, editar, eliminar o crear recursos |

### 6.2 Qué es el root user

Al crear la cuenta se registra un correo y una contraseña: esa identidad es el **AWS account root user**, con **acceso sin restricciones a todos los servicios y recursos** de la cuenta, incluida la información personal y de facturación. Iniciar sesión con correo + contraseña = iniciar sesión como root.

**Dos juegos de credenciales del root user:**

| Credencial | Uso | Forma |
|---|---|---|
| Correo + contraseña | Acceso a la AWS Management Console | — |
| **Access keys** | Solicitudes programáticas desde CLI o API | *Access key ID* (ej. `A2lAl5EXAMPLE`) + *secret access key* (ej. `wJalrFE/KbEKxE`); se necesitan ambas, como usuario + contraseña |

Las access keys deben protegerse con el mismo cuidado que el correo y la contraseña.

> [!danger] 🚨 El escenario que el curso quiere evitar
> Un atacante entra con el root user, borra datos y recursos y lanza una costosa operación de minería de criptomonedas. Usted queda con la factura y sin datos. Una contraseña fuerte sola es **autenticación de un factor**: basta con adivinarla.

### 6.3 Buenas prácticas para el root user

1. Contraseña robusta.
2. Nunca compartir la contraseña ni las access keys del root.
3. **Deshabilitar o eliminar las access keys del root**; si no existen, no crearlas salvo que sea absolutamente necesario.
4. **Habilitar MFA justo después de crear la cuenta.**
5. No usar el root ni para tareas administrativas ni cotidianas: solo unas pocas tareas lo requieren (el material enlaza la lista "Tasks that require root user credentials").

**Cómo eliminar una access key del root (pasos de la lectura):**
1. Ir a la página *My Security Credentials* en la consola e iniciar sesión con el correo y la contraseña del root.
2. Abrir la sección *Access keys*.
3. En *Actions*, elegir *Delete*.
4. Confirmar con *Yes*.

### 6.4 MFA

- La autenticación de un factor (contraseña, PIN, token) es la más simple y común, pero frágil: el ejemplo de **Bob**, cuya contraseña `IloveCats222` combina dos datos personales (le gustan los gatos, cumple el 22 de febrero), es fácil de adivinar por ingeniería social, bots o scripts.
- **MFA** exige **dos o más** métodos de tres categorías:

| Categoría | Ejemplo |
|---|---|
| Algo que sabe | Usuario y contraseña, PIN |
| Algo que tiene | Código de un dispositivo o app móvil |
| Algo que es | Huella, reconocimiento facial |

- En AWS, con MFA en el root se pide **algo que sabe** (correo + contraseña) y **algo que tiene** (código numérico temporal de un dispositivo MFA). Aunque la contraseña se filtre, sin el código no hay acceso. Morgan usa personalmente un MFA virtual en el teléfono.

**Dispositivos MFA admitidos:**

| Tipo | Descripción | Ejemplos |
|---|---|---|
| Virtual MFA | App en teléfono u otro dispositivo que genera un código de un solo uso; puede correr en dispositivos móviles no seguros, por lo que quizá no iguale la seguridad del hardware o U2F | Authy, Duo Mobile, LastPass Authenticator, Microsoft Authenticator, Google Authenticator |
| Hardware | Llavero (*key fob*) o tarjeta con pantalla que genera un código de **seis dígitos** de un solo uso | Key fob, display card |
| U2F security key | Dispositivo que se conecta a un puerto USB | YubiKey |

🎯 **Para el examen:**
- Primer paso tras crear la cuenta: MFA en el root, luego crear un IAM user administrador y dejar de usar el root.
- No crear access keys para el root.
- No se puede aplicar una IAM policy al root user (sección 7.6).

---

## 7. AWS Identity and Access Management (IAM)

Audiencia: 🔧 🧭

### 7.1 Los tres puntos de acceso de Employee Directory

| # | Necesidad | ¿La resuelve IAM? |
|---|---|---|
| 1 | Usuarios finales que inician sesión en la app (gestión de acceso a nivel de aplicación) | **No.** La app debe usar sus propios mecanismos de autenticación |
| 2 | El código en EC2 debe hacer llamadas firmadas a S3 para leer/escribir imágenes | **Sí** (mediante un IAM role) |
| 3 | Las personas que construyen la arquitectura necesitan iniciar sesión y permisos para crear red, instancias, etc. | **Sí** (IAM users, groups y policies) |

> [!warning] ⚠️ Regla crítica
> Que EC2 y S3 estén en la misma cuenta **no** autoriza automáticamente las llamadas entre ellos. **Toda llamada a la API debe estar firmada y autenticada**, vivan o no los recursos en la misma cuenta.

### 7.2 Qué es IAM y sus características

IAM es un servicio web para administrar el acceso a la cuenta y sus recursos, con una vista central de quién puede entrar (autenticación) y qué puede hacer (autorización). Permite compartir el acceso **sin compartir contraseñas ni access keys** y dar permisos granulares (p. ej., solo lectura sobre acciones y recursos concretos de un servicio).

- **Global**, no regional: se ve y usa igual desde cualquier Region de la consola.
- Integrado por defecto con muchos servicios de AWS.
- **Password policies**: complejidad y períodos de rotación obligatorios.
- Soporta **MFA**.
- Soporta **identity federation**: usuarios con contraseña en otro sistema (red corporativa, proveedor de identidad de Internet) obtienen acceso temporal a la cuenta.
- **Sin costo adicional** para cualquier cliente.

### 7.3 IAM users

- Representa a una persona o servicio que interactúa con AWS; se define dentro de la cuenta y **toda su actividad se factura a esa cuenta**.
- Cada persona debe tener **sus propias credenciales**: así se evita compartirlas.
- Consta de un nombre y credenciales. Al crearlo se elige:
  - **Acceso a la consola** → usuario y contraseña.
  - **Acceso programático** (CLI/API) → access keys.
- Las credenciales de un IAM user se consideran **permanentes**: duran hasta que un administrador fuerce su rotación.
- Se pueden dar permisos directo al user, pero no escala: con **3.000 usuarios** se vuelve imposible tener una visión clara de quién puede hacer qué.

### 7.4 IAM groups

- Colección de users; todos **heredan** los permisos del group. Es la forma recomendada (buena práctica).
- Organización típica por función: developers, security, admins.
- Ejemplos:
  - Llega un desarrollador → se crea su user y se agrega al group de developers, sin pensar permisos uno a uno.
  - Un desarrollador pasa a ingeniero de seguridad → se le saca del group antiguo y se mete en el nuevo.
- Reglas:
  - Un group puede tener muchos users.
  - Un user puede pertenecer a muchos groups.
  - **Los groups no pueden contener otros groups.**

> [!warning] ⚠️ Permisos por defecto
> El root user puede hacer todo por defecto. Las identidades IAM nuevas (users, groups, roles) **no pueden hacer nada** hasta que se les conceda permiso explícitamente mediante policies.

### 7.5 IAM policies

- Documentos **JSON** que se adjuntan a users, groups y roles. En cada solicitud, AWS evalúa todas las policies asociadas (las del group y las del user) para decidir permitir o denegar.
- En IAM, "acción" = una llamada a la API de AWS.

**Elementos de un statement:**

| Elemento | Qué define | ¿Obligatorio? | Ejemplo |
|---|---|---|---|
| `Version` | Versión del lenguaje de policies; usar `"2012-10-17"` antes de `Statement` para tener todas las funciones | (recomendado en todas) | `"Version": "2012-10-17"` |
| `Effect` | `Allow` o `Deny` explícito (no hay otra opción) | ✔ | `"Effect": "Deny"` |
| `Action` | Llamadas a la API permitidas/denegadas | ✔ | `"Action": "iam:CreateUser"` |
| `Resource` | Objeto(s) sobre los que aplica | ✔ | `"Resource": "arn:aws:iam::account-ID-without-hyphens:user/Bob"` |
| `Condition` | Restricciones adicionales (mencionado en el video) | Opcional | — |

**Ejemplo 1 — policy de administrador** (todas las acciones sobre todos los recursos; `*` es comodín):

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "*",
    "Resource": "*"
  }]
}
```

**Ejemplo 2 — granular:** el user solo puede cambiar su propia contraseña y leer su propia información, gracias a la variable `${aws:username}`:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "iam:ChangePassword",
      "iam:GetUser"
    ],
    "Resource": "arn:aws:iam::123456789012:user/${aws:username}"
  }]
}
```

**Ejemplo del video:** una policy con `"Action": "ec2:*"` permite cualquier acción de EC2; se puede restringir a una sola llamada (p. ej., solo lanzar instancias, `ec2:RunInstances`) para que el user no haga nada más en EC2.

> [!note] ➕ Nota del editor
> Los nombres de acción en el JSON se escriben en inglés y sin espacios (`iam:ChangePassword`, `iam:CreateUser`); la traducción automática del material los mostraba traducidos o con espacios. El nombre exacto `ec2:RunInstances` es la acción de API correspondiente a "run instances" que menciona el video.

### 7.6 Secuencia recomendada al crear una cuenta

1. Activar MFA en el root user.
2. Crear un IAM user con permisos de administrador.
3. Cerrar la sesión del root.
4. Iniciar sesión con ese IAM user y, desde él, crear el resto de users, groups y policies.

**¿Por qué?** Porque **no se puede aplicar una policy al root user**, pero sí a un IAM user.

---

## 8. Role-based access: IAM roles

Audiencia: 🔧 🧭

### 8.1 Por qué no un IAM user para la aplicación

Las solicitudes HTTP a AWS se **firman** programáticamente para que AWS verifique la identidad y que la solicitud sea legítima. Un IAM user firma con su access key ID y secret access key. Para el código que corre en EC2, el curso descarta explícitamente crear un IAM user con credenciales: la solución es un **IAM role**.

| | IAM user | IAM role |
|---|---|---|
| Credenciales de inicio de sesión | Usuario y contraseña | **No tiene** |
| Credenciales para firmar | Access keys **estáticas, de largo plazo** | **Temporales**, obtenidas programáticamente y **rotadas automáticamente** |
| Expiración | Solo cuando se rotan (o por password policy) | Entre **15 minutos y 36 horas** |
| Quién lo usa | Una persona o servicio concreto | Lo **asume** cualquiera autorizado: personas, servicios, usuarios federados |
| Mantenimiento | Más costoso | Más fácil (buena práctica: usar roles siempre que se pueda) |

### 8.2 Casos de uso

1. **Servicio a servicio:** EC2 asume un role para llamar a S3 y DynamoDB. Es muy común: si un servicio llama a otro, casi siempre lo hace asumiendo un role, obteniendo credenciales temporales y enviando la llamada, que el servicio destino verifica.
2. **Usuarios federados:** una empresa con **5.000 empleados técnicos** ya autenticados en su proveedor de identidad (IdP) no debería crear 5.000 IAM users; en su lugar, AWS asigna un role al usuario federado cuando pide acceso a través del IdP. Servicios como **AWS IAM Identity Center** facilitan esto.

### 8.3 Instance profile y SDK

En Employee Directory la instancia EC2 recibe un role a través del **instance profile**. El **AWS SDK** dentro de la app toma automáticamente las credenciales temporales, las renueva cuando hace falta y las usa para autenticar las llamadas a S3 y DynamoDB. Sin el role, la app no estaría autenticada y esas llamadas fallarían.

### 8.4 Resumen de buenas prácticas de IAM

| Práctica | En qué consiste |
|---|---|
| **Bloquear el root user** | No compartir sus credenciales, considerar eliminar sus access keys, activar MFA |
| **Least privilege** | Partir del mínimo de permisos e ir agregando solo lo necesario para la tarea |
| **Usar IAM para lo que es** | Asegura el acceso a la cuenta y recursos de AWS dentro de **una** cuenta. **No** sirve para login/registro de usuarios de un sitio web, ni para proteger sistemas operativos y redes |
| **Roles siempre que sea posible** | Credenciales temporales (15 min – 36 h) frente a credenciales de largo plazo de los users |
| **Considerar un IdP** | Fuente única de verdad para las identidades; los permisos se dan vía roles a identidades federadas. Ejemplo de **Martha**, con acceso a varias cuentas: se gestiona una vez en el IdP, no un user por cuenta; si cambia de área o se va, se actualiza solo en el IdP |
| **Considerar AWS IAM Identity Center** | Sucesor de AWS Single Sign-On. Portal con **un solo juego de credenciales** para todas las cuentas y aplicaciones asignadas. Ofrece directorio, groups y permisos como IAM, pero además sincroniza users y groups desde un IdP de terceros y separa funciones: la gestión de acceso a la nube no queda dentro ni dependiente del IdP |

> [!tip] 💡 Analogía
> El IAM user es una llave copiada que alguien lleva en el bolsillo indefinidamente; el IAM role es la tarjeta de visita de un hotel: se entrega a quien está autorizado, abre solo ciertas puertas y deja de funcionar sola al terminar la estadía.

🎯 **Para el examen:**
- App en EC2 que necesita S3/DynamoDB → IAM role vía instance profile, **nunca** access keys embebidas.
- Credenciales de role: temporales, 15 min – 36 h, rotación automática.
- Muchos empleados con IdP corporativo → federación + roles / IAM Identity Center.
- IAM no autentica usuarios finales de una aplicación web.
- Groups no anidan.

---

## 9. Demos y laboratorios

### 9.1 Lab 1 — Introducción a IAM (calificado)

*Ítem sin texto en la extracción (LTI); objetivo según el video de introducción.*

- **Objetivo:** aplicar las buenas prácticas de IAM vistas.
- **Servicios:** AWS IAM y otros servicios para probar permisos.
- **Qué se hace:**
  1. Entrar al IAM dashboard y explorar los groups, users, roles y policies que vienen precargados en el entorno.
  2. Gestionar users y groups: agregar users a groups para que hereden los permisos de ese group.
  3. Comprobar cómo funcionan los permisos explorando distintos servicios de AWS con distintos IAM users.
- Los labs siguientes del curso replican, a grandes rasgos, lo que se muestra en los videos.

### 9.2 Demo AWS IAM — role, user y access keys

**Objetivo:** crear el role de la aplicación y mostrar cómo se crean users y access keys.

**Parte A — Crear el role `EmployeeWebApp`**

1. IAM dashboard → **Roles** (barra izquierda) → **Create role**.
2. **Trusted entity type** (quién puede asumir el role; no cualquiera debe poder):

| Tipo | Para qué |
|---|---|
| AWS service | Un servicio como EC2 o Lambda que asume el role para llamar a la API |
| AWS account | Acceso entre cuentas a recursos de la propia cuenta |
| Web identity | Usuarios federados por identidad web |
| SAML 2.0 federation | Directorio corporativo on-premises que usa SAML |
| Custom trust policy | Política de confianza a medida |

3. Elegir **AWS service → EC2** (la app correrá en EC2) → *Next*.
4. **Add permissions:** aparecen las **AWS managed policies** (creadas y mantenidas por AWS). Al buscar S3 y expandir **AmazonS3FullAccess** se ve su JSON: `Effect: Allow`, `Action: s3:*` y `s3-object-lambda:*`, `Resource: *`. Es muy permisiva; en el mundo real se crearía una **custom policy** (botón *Create policy*) con solo las llamadas y recursos que la app necesita.
5. Marcar **AmazonS3FullAccess**; limpiar el filtro, buscar DynamoDB y marcar **AmazonDynamoDBFullAccess** (más adelante la app usará DynamoDB) → *Next*.
6. Nombre del role (`EmployeeWebApp`; en el video conceptual aparece como `EmployeeWebAppRole`).
7. Revisar la **trust policy**: permite la llamada `sts:AssumeRole` solo a `ec2.amazonaws.com`, es decir, solo una instancia EC2 puede asumirlo.
8. **Create role.** En el detalle del role se ve el **ARN** (Amazon Resource Name), los permisos adjuntos (se pueden agregar más), la simulación de permisos, las relaciones de confianza (*trust relationships*) y los **tags** (pares clave-valor).

**Parte B — Crear el user `EC2Admin` y su group**

1. **Users** → **Add users** → nombre `EC2Admin`.
2. Marcar **Enable console access** (viene desmarcado por defecto: crear un user no implica acceso a la consola).
3. Contraseña autogenerada y dejar marcado que el user cree una nueva en el próximo inicio de sesión → *Next*.
4. No hay groups aún → **Create group** `EC2Admins`, adjuntarle **AmazonEC2FullAccess** (buena práctica: policies al group, no al user) → *Create user group*.
5. Seleccionar el group → *Next*. El resumen muestra que el user heredará los permisos del group y recibirá directamente **IAMUserChangePassword** para cambiar su contraseña.
6. **Create user** → *Return to users list* (no se descarga la contraseña; el user es solo demostrativo).

**Parte C — Access keys**

1. Abrir el user → pestaña **Security credentials** → panel **Access keys**.
2. **Create access key** → caso de uso *Command Line Interface (CLI)*.
3. La consola advierte que existe **AWS CloudShell** como alternativa en el navegador; se marca la casilla "entiendo la recomendación" → *Next* → **Create access key**.
4. Se muestra el access key ID y el secret access key (oculto hasta pulsar *Show*), que se usarían para configurar la CLI localmente → *Done*.
5. **Limpieza:** *Actions → Deactivate* y luego *Actions → Delete*, pegando el access key ID para confirmar.

> [!danger] 🚨 Errores comunes
> - Dejar `*FullAccess` en producción: en la demo es aceptable por ser prueba de concepto, pero el curso insiste en restringirlo después.
> - Olvidar desactivar y borrar access keys de prueba.
> - Adjuntar policies directo a cada user en vez de al group.

### 9.3 Demo: alojar Employee Directory en EC2 (Seph y Morgan)

**Objetivo:** ver un servicio bajo demanda en acción, lanzando la app con los valores por defecto. Se recomienda **mirarla sin seguirla todavía**: en la semana siguiente hay ejercicios con instrucciones paso a paso en la propia cuenta.

**Contexto:** AWS entrega una **default VPC** (red privada para los recursos); toda instancia EC2 debe vivir en una red.

**Pasos:**

1. Consola → **Amazon EC2** → **Launch instance** (una *instance* = una máquina virtual).
2. Nombre: "employee web app".
3. **AMI** (Amazon Machine Image): Linux, opción free tier.
4. **Instance type:** `t2.micro` (free tier).
5. **Key pair:** normalmente se usa para entrar por SSH; aquí *Proceed without a key pair*.
6. **Network settings → Edit:** default VPC, subnet en *No preference*.
7. Crear un **security group** nuevo (firewall a nivel de instancia) con reglas de entrada para **HTTP y HTTPS**.
8. **Advanced details:** aceptar casi todo por defecto salvo:
   - **IAM instance profile:** el role de la app (se aprovechará cuando exista el bucket S3). En las demos posteriores el role se llama `S3DynamoDBFullAccessRole`, con las mismas dos policies que `EmployeeWebApp`.
   - **User data:** script que corre al iniciar la instancia; descarga el código, instala dependencias y arranca el servidor web. Alternativa manual: conectarse por SSH y configurar a mano; se automatiza con el script.
9. **Launch instance**, esperar unos minutos a que quede *running*.
10. Copiar la **IP pública** en el navegador → aparece la página de inicio **vacía**, lo esperado porque aún no hay base de datos que sirva datos.

**Scripts de user data** (con la AMI por defecto cambiada; ver aviso):

*Amazon Linux 2:*
```bash
#!/bin/bash -ex
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
unzip FlaskApp.zip
cd FlaskApp/
yum -y install python3 mysql
pip3 install -r requirements.txt
amazon-linux-extras install epel
yum -y install stress
export PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}
export AWS_DEFAULT_REGION=<INSERT REGION HERE>
export DYNAMO_MODE=on
FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
```

*Amazon Linux 2023:*
```bash
#!/bin/bash -ex
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
unzip FlaskApp.zip
cd FlaskApp/
yum -y install python3-pip
pip install -r requirements.txt
yum -y install stress
export PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}
export AWS_DEFAULT_REGION=<INSERT REGION HERE>
export DYNAMO_MODE=on
FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
```

Diferencias: en AL2023 desaparecen `mysql` y `amazon-linux-extras install epel`, y se instala `python3-pip` usando `pip` en vez de `pip3`. Hay que reemplazar `<INSERT REGION HERE>` por la Region en uso, **eliminando también los corchetes angulares**.

> [!warning] ⚠️ Cambio de AMI por defecto (aviso del curso)
> Desde el **15 de marzo de 2023** la AMI por defecto de EC2 es **Amazon Linux 2023**; los videos usan **Amazon Linux 2**. El script original falla en AL2023 y la app no arranca. Dos salidas: elegir Amazon Linux 2 con el script original, o AL2023 con el script actualizado.

> [!note] ➕ Nota del editor
> La versión en español del aviso venía traducida automáticamente dentro del propio código (`descomprima`, `requisitos.txt`, `BALSA_FOTOS=${BALA_FOTOS}`) y el script AL2023 aparecía sin `#!`. Arriba se restituyen los comandos y nombres originales (`unzip`, `requirements.txt`, `PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}`, shebang). El nombre de la variable de entorno debe confirmarse con el script de los ejercicios de la semana 2.

---

## 10. Aplicación de ejemplo: estado al cierre del módulo

Al terminar la semana 1, Employee Directory existe así:

| Pieza | Estado |
|---|---|
| IAM role `EmployeeWebApp` / `S3DynamoDBFullAccessRole` | Creado, con AmazonS3FullAccess + AmazonDynamoDBFullAccess; trust policy solo para EC2 |
| IAM user/group de ejemplo (`EC2Admin` / `EC2Admins`) | Demostrativos |
| Red | **Default VPC**, subnet sin preferencia |
| Cómputo | 1 instancia EC2 `t2.micro`, AMI Amazon Linux, sin key pair, con instance profile y user data |
| Firewall | Security group con entrada HTTP/HTTPS |
| App | Flask corriendo en el puerto 80; página de inicio vacía |
| Pendiente | Bucket S3 de fotos, base de datos (RDS/DynamoDB), VPC propia, ELB, Auto Scaling, CloudWatch |

```
  Navegador ──HTTP:80──▶ ┌──────────── default VPC ─────────────┐
                         │  [Security group: HTTP, HTTPS in]    │
                         │   ┌──────────────────────────────┐   │
                         │   │ EC2 t2.micro (Amazon Linux)  │   │
                         │   │  Flask app (user data)       │   │
                         │   │  instance profile ──▶ IAM role│──┼──▶ S3 / DynamoDB (aún no existen)
                         │   └──────────────────────────────┘   │
                         └──────────────────────────────────────┘
```

---

## ✅ Autoevaluación

> [!question]- Una empresa debe mantener datos de clientes dentro del Reino Unido, pero la Region más barata para sus usuarios está en otro país. ¿Qué Region elige?
> La de Londres. El cumplimiento normativo se evalúa primero y, si impone una ubicación, los demás factores (latencia, precio, servicios) dejan de pesar.

> [!question]- ¿Cuál es el mínimo de AZ recomendado para una carga altamente disponible y por qué?
> Dos. Si toda una AZ falla, la segunda conserva infraestructura funcionando para tomar el tráfico. Si existe un servicio administrado de alcance regional, es preferible usarlo.

> [!question]- En una instancia EC2, ¿a quién corresponde parchear el SO invitado y a quién el hipervisor?
> El SO de la instancia lo parchea el cliente (seguridad en la nube); el host OS y el hipervisor, AWS (seguridad de la nube).

> [!question]- ¿Qué diferencia de responsabilidad hay entre EC2, RDS y S3 según el modelo compartido?
> EC2 (infrastructure): el cliente gestiona SO, plataforma y datos. RDS (container): AWS asume además SO y plataforma; el cliente cuida datos, firewalls y respaldos. S3 (abstracted): AWS cubre incluso el cifrado del lado del servidor; el cliente gestiona datos, acceso y cifrado del lado del cliente.

> [!question]- ¿Por qué se recomienda crear un IAM user administrador y dejar de usar el root?
> Porque el root tiene poder ilimitado (incluida la facturación) y no se le pueden aplicar policies; un IAM user sí se puede limitar y auditar. El root se reserva para las pocas tareas que lo exigen.

> [!question]- Un nuevo desarrollador llega al equipo. ¿Cuál es la forma escalable de darle permisos?
> Crear su IAM user y agregarlo al group de developers, que ya tiene las policies; no adjuntar policies al user directamente.

> [!question]- ¿Qué hace la policy con `"Resource": "arn:aws:iam::123456789012:user/${aws:username}"` y acciones `iam:ChangePassword` e `iam:GetUser`?
> Permite a cada user cambiar solo su propia contraseña y consultar solo su propia información; la variable `${aws:username}` restringe el recurso al user que hace la solicitud.

> [!question]- La app en EC2 necesita escribir fotos en S3. ¿Por qué no se guardan access keys en el código?
> Serían credenciales permanentes expuestas. Lo correcto es un IAM role asignado vía instance profile: el SDK obtiene credenciales temporales (15 min – 36 h) y rotadas automáticamente.

> [!question]- ¿Puede IAM gestionar el login de los usuarios finales de Employee Directory?
> No. IAM protege el acceso a la cuenta y recursos de AWS; la autenticación de usuarios de la aplicación web requiere un mecanismo propio de la app.

> [!question]- ¿Qué pasa si lanza la instancia con Amazon Linux 2023 y el user data de los videos?
> El script falla y la app no arranca; se debe elegir Amazon Linux 2 o usar el script actualizado para AL2023.

---

## 🧾 Resumen en 10 líneas

1. Cloud computing = recursos de TI bajo demanda por Internet con pago por uso; AWS se encarga del *undifferentiated heavy lifting*.
2. Seis ventajas: pay as you go, economías de escala, no adivinar capacidad, velocidad y agilidad, no operar data centers, global en minutos.
3. Data center ⊂ AZ ⊂ Region, todo redundante; las Regions son independientes y los datos no salen sin permiso.
4. Elegir Region por compliance (primero), latencia, precio y servicios disponibles.
5. Alta disponibilidad: servicios regionales o al menos dos AZ; edge locations + CloudFront acercan el contenido.
6. Todo en AWS es una llamada a la API autenticada y autorizada, vía Console, CLI (o CloudShell) o SDK.
7. Shared responsibility: AWS asegura la nube; el cliente asegura lo que pone en ella, y su grado depende del tipo de servicio.
8. Root user: MFA de inmediato, sin access keys, sin uso diario; primero se crea un IAM user administrador.
9. IAM (global, gratis): users para personas, groups para permisos, policies JSON (Effect/Action/Resource), least privilege.
10. Roles para credenciales temporales (apps en EC2, federación, IAM Identity Center); Employee Directory ya corre en una EC2 de la default VPC.

---

## 📖 Recursos del módulo

**Lectura 1.2 — ¿Qué es AWS?**
- AWS: What is cloud computing? · https://aws.amazon.com/what-is-cloud-computing/
- AWS: Types of cloud computing · http://docs.aws.amazon.com/whitepapers/latest/aws-overview/types-of-cloud-computing.html
- AWS: Cloud computing with AWS · https://aws.amazon.com/what-is-aws/

**Lectura 1.3 — Infraestructura global**
- AWS Global Infrastructure · https://aws.amazon.com/about-aws/global-infrastructure/
- Documentación de la infraestructura global de AWS · https://docs.aws.amazon.com/whitepapers/latest/aws-overview/global-infrastructure.html
- Regions and Availability Zones · https://aws.amazon.com/about-aws/global-infrastructure/regions_az/
- AWS service endpoints · https://docs.aws.amazon.com/general/latest/gr/rande.html
- AWS Regional Services · https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/

**Lectura 1.4 — Interactuar con AWS**
- Working with the AWS Management Console · https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/getting-started.html
- AWS Command Line Interface · https://aws.amazon.com/cli/
- Tools to build on AWS · https://aws.amazon.com/tools/

**Lectura 1.5 — Shared responsibility model**
- Shared Responsibility Model · https://aws.amazon.com/compliance/shared-responsibility-model/

**Lectura 1.6 — Proteger el root user**
- My Security Credentials (consola) · https://console.aws.amazon.com/iam/home?#security_credential
- Enabling a hardware MFA device (console) · https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_physical.html
- Enabling a U2F security key (console) · https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_u2f.html
- Enabling a virtual MFA device (console) · https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html
- Supported MFA devices · https://aws.amazon.com/iam/features/mfa/
- Tasks that require root user credentials · https://docs.aws.amazon.com/general/latest/gr/root-vs-iam.html#aws_tasks-that-require-root

**Lectura 1.7 — Introducción a IAM**
- AWS services that work with IAM · https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html
- What is IAM? · https://docs.amazonaws.cn/en_us/IAM/latest/UserGuide/introduction.html
- AWS IAM identities · https://docs.amazonaws.cn/en_us/IAM/latest/UserGuide/id.html
- Access management with AWS IAM · https://docs.amazonaws.cn/en_us/IAM/latest/UserGuide/access.html

**Lectura 1.8 — Acceso basado en roles**
- Security best practices in IAM · https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- How to create and manage users within AWS IAM Identity Center · https://aws.amazon.com/blogs/security/how-to-create-and-manage-users-within-aws-sso/

**Roles IAM y Employee Directory**
- IAM roles for Amazon EC2 · https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html

**Otros**
- AWS re:Post (comunidad) · https://repost.aws
- Código de Employee Directory (user data) · https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip

> [!note] ➕ Nota del editor
> En la Lectura 1.6, el enlace del texto "CLI de AWS o la API de AWS" apunta a un módulo de Trailhead (`trailhead.salesforce.com/content/learn/modules/aws-cloud-technical-professionals`), probablemente un error de la versión del material; no se incluye como recurso.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[00-MOC-Guia-Maestra-AWS|00 · Índice]] · Siguiente: [[02-Computo-y-Redes|02 · Cómputo y Redes ➡]]
