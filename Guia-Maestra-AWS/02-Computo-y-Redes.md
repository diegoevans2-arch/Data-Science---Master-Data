---
title: "Tomo 02 — Cómputo y Redes"
tags: [aws, solutions-architect, ec2, contenedores, serverless, lambda, vpc, networking]
curso: 1 · AWS Cloud Technical Essentials
modulo: "Semana 2: Informática y redes"
audiencias: [tecnico, puente, ejecutivo]
tomo: 02
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🖥️ Tomo 02 — Cómputo y Redes

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[01-Vision-General-y-Seguridad|01 · Visión General y Seguridad]] · Siguiente: [[03-Almacenamiento-y-Bases-de-Datos|03 · Almacenamiento y Bases de Datos ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Al cierre del tomo 01 la aplicación Employee Directory ya corría en una instancia EC2, pero dentro de la VPC por defecto y sin que se explicara qué es una instancia ni qué la rodea. Este tomo cubre las dos preguntas que vienen después: **dónde se ejecuta el código** (máquinas virtuales con EC2, contenedores con ECS/EKS, serverless con Lambda y Fargate) y **en qué red vive** (VPC, subnets, route tables, gateways, network ACL y security groups). Casi todas las arquitecturas que vienen más adelante se apoyan en estas dos capas.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** qué modelo de cómputo usar en cada carga (control frente a conveniencia), qué opción de compra de EC2 conviene (On-Demand, Reserved, Spot), cómo segmentar la red entre lo público y lo privado y cómo conectar el centro de datos propio con AWS (VPN o Direct Connect).
> - **Costo de hacerlo mal:** instancias encendidas todo el día para procesos que corren una vez por trimestre, cargas críticas sobre Spot que pueden interrumpirse con 2 minutos de aviso, datos de clientes en la VPC por defecto expuestos a Internet, y una aplicación caída entera porque todo vivía en una sola AZ.
> - **Pregunta que responde:** *¿qué tipo de servidor necesito para esta carga, cuánto me cuesta tenerlo encendido y quién puede llegar a él desde Internet?*

> [!tip] 💡 Analogía general: el edificio de oficinas
> La **VPC** es el edificio con su muro perimetral: nada entra ni sale sin permiso. Las **subnets** son los pisos o salas; las **route tables**, la señalética que dice por qué pasillo se llega a cada sala; el **internet gateway**, la puerta principal a la calle, y el **virtual private gateway**, la entrada de servicio hacia la sede central de la empresa. La **network ACL** es el guardia de cada piso, que revisa a todos los que entran *y* salen. El **security group** es la cerradura de cada oficina: recuerda a quién dejó entrar y le deja salir. Dentro de las oficinas trabajan los empleados: unos fijos con escritorio propio (**EC2**), otros que traen todo en una maleta y se instalan en cualquier escritorio (**contenedores**) y otros que solo llegan cuando se les llama y se van al terminar (**Lambda**).

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| **Informática de AWS** | 11 videos · 5 lecturas · 1 ítem interactivo (coach) | Computación como servicio; EC2 (AMI, tipos y familias de instancia, ciclo de vida, precios); demo de lanzamiento de Employee Directory; contenedores (Docker, ECS, EKS); serverless (Fargate, Lambda con demo de redimensión de imágenes); cómo elegir el servicio de cómputo |
| **Redes AWS** | 5 videos · 4 lecturas | Fundamentos de redes (IP, IPv4, CIDR); VPC, subnets, IP reservadas, internet gateway y virtual private gateway; route tables; network ACL y security groups; conectividad híbrida (VPN, Direct Connect); guía de troubleshooting |
| **Semana 2 Ejercicio y evaluación** | 2 videos · 1 lab · 1 quiz | Lab 2 «Creación de una VPC»; demo de relanzamiento de Employee Directory en la VPC propia; cuestionario de la semana |

Audiencia de las secciones: 🔧 técnico · 🧭 puente (decisión técnica y de negocio) · 👔 ejecutivo.

---

## 1. Computación como servicio

Audiencia: 🔧 🧭 👔

Toda empresa necesita capacidad de cómputo para servidores web, procesos batch, bases de datos, software de RR.HH., machine learning, etc. El curso contrasta dos maneras de conseguirla:

| Etapa | On-premises (flota propia) | Compute as a Service en AWS |
|---|---|---|
| Elegir hardware | Investigar qué servidores y cuántos comprar | Elegir un tipo de instancia; se puede cambiar después |
| Pago | Compra por adelantado (gasto de capital) | Pago por uso, por segundo u hora |
| Plazo | Semanas o meses de entrega | Minutos |
| Instalación | Llevar al data center, montar en rack, cablear, asegurar, energizar | AWS ya construyó y aseguró los data centers y montó los servidores |
| Capacidad sobrante | Se queda con los servidores aunque no los use; el dinero no se recupera y el tiempo tampoco | Se detiene o termina la instancia y deja de cobrarse |
| Capacidad faltante | Difícil sumar capacidad física de golpe | Se lanza más capacidad con una llamada a la API |

**🔧 El modelo cliente-servidor.** El primer bloque para alojar una aplicación es un **servidor**: atiende solicitudes (típicamente HTTP, aunque cualquier comunicación por API sigue el mismo modelo) y devuelve respuestas a un **cliente** (persona o computador que envía la solicitud). El servidor aporta CPU, memoria y red para convertir solicitudes en respuestas. Servidores HTTP de referencia:

- Windows: **Internet Information Services (IIS)**.
- Linux: **Apache HTTP Server**, **Nginx**, **Apache Tomcat**.

**Las tres categorías de cómputo en AWS:**

```
  CONTROL ◄──────────────────────────────────────────────► CONVENIENCIA
  Máquinas virtuales          Contenedores                Serverless
  (Amazon EC2)                (ECS / EKS sobre EC2)       (AWS Fargate, AWS Lambda)
  usted administra el SO      AWS administra más de la    no ve ni accede a la
  invitado, parches, escala   orquestación; con EC2 usted infraestructura subyacente
                              sigue manteniendo instancias
```

Una **máquina virtual** emula un servidor físico. Para ejecutarla se instala un **hypervisor** en una máquina anfitriona (host), que reparte los recursos entre las VM. En EC2, AWS opera los hosts y la capa del hypervisor, e instala el sistema operativo de la VM (**guest OS**). Varios servicios de cómputo de AWS se basan en EC2 o en virtualización, por eso conviene entender EC2 antes de pasar a contenedores y serverless.

> [!warning] ⚠️ La VM no es la respuesta por defecto
> El curso lo dice de forma explícita: según lo que se construya y lo que se quiera optimizar, a veces conviene diseñar la aplicación para contenedores o serverless desde el primer día.

---

## 2. Amazon EC2: instancias y AMI

Audiencia: 🔧 🧭

**🔧 Definición.** Amazon Elastic Compute Cloud (EC2) es un servicio web que entrega capacidad de cómputo segura y redimensionable en la nube, en forma de servidores virtuales llamados **instancias EC2**. Que AWS lo llame «servicio web» no significa que solo sirva para alojar servidores web. Las instancias se crean y administran desde la AWS Management Console, la AWS CLI, los AWS SDK o herramientas de automatización y orquestación de infraestructura.

Para crear una instancia se definen dos grupos de cosas:

| Especificaciones de hardware | Configuraciones lógicas |
|---|---|
| CPU, memoria, red, almacenamiento | Ubicación en la red, reglas de firewall, autenticación, sistema operativo |

**Facturación y elasticidad.** Se puede aprovisionar una o muchas instancias, se paga solo por lo usado (por segundo o por hora según el tipo de instancia) y, al detener o terminar una instancia, dejan de correr los cargos de cómputo. El curso destaca la consecuencia de negocio: como levantar servidores por poco tiempo es barato, se puede experimentar para encontrar la configuración óptima y luego **dimensionar correctamente** (right-sizing) en vez de sobreaprovisionar, que es lo habitual on-premises.

### 2.1 Amazon Machine Image (AMI)

En infraestructura tradicional, el sistema operativo se instala desde discos, unidades o asistentes por red. En AWS esa instalación ya viene dentro de la **AMI** que se elija. Una AMI define:

- el sistema operativo (Linux, macOS, Ubuntu, Windows, entre otros);
- el mapeo de almacenamiento;
- la arquitectura (32 bits, 64 bits o ARM de 64 bits);
- el software adicional preinstalado.

> [!tip] 💡 Analogía del curso: receta y pastel, clase y objeto
> La AMI es la receta y la instancia EC2 es el pastel horneado a partir de ella. En términos de programación, la AMI es la **clase** (lo que se modela y define) y la instancia es el **objeto** (aquello con lo que se interactúa, donde se instala el servidor web y se sirve contenido).

**Qué pasa al lanzar una instancia:** AWS asigna una VM sobre un hypervisor, copia la AMI al **root device volume** (el volumen que contiene la imagen de arranque) y entrega un servidor al que uno puede conectarse para instalar paquetes y software, por ejemplo el servidor web y el código de Employee Directory.

**Reutilización.** Si se necesita una segunda instancia idéntica, repetir a mano toda la configuración es lento y propenso a errores. Lo mejor es **crear una AMI a partir de la instancia en ejecución** y lanzar la nueva desde esa AMI. Una sola AMI puede lanzar muchas instancias con la misma configuración.

**Dónde se encuentran las AMI:**

| Origen | Qué ofrece |
|---|---|
| **Quick Start AMIs** | Prearmadas por AWS para partir rápido (p. ej., Amazon Linux 2, la que usó Employee Directory en el tomo 01: imagen Linux optimizada para EC2 con soporte de largo plazo de AWS) |
| **AWS Marketplace AMIs** | Software de código abierto y comercial de terceros |
| **My AMIs** | Creadas a partir de sus propias instancias |
| **Community AMIs** | Aportadas por la comunidad de usuarios |
| **EC2 Image Builder** | Construcción de imágenes personalizadas |

Cada AMI tiene un **AMI ID** con prefijo `ami-` seguido de un hash de letras y números, y ese ID es **único por región**. En la demo se agrega una práctica organizacional: crear AMI aprobadas internamente (con paquetes de seguridad o cumplimiento preinstalados) y exigir que solo se lance desde ellas.

> [!note] ➕ Nota del editor
> Como el AMI ID cambia de una región a otra, un script o plantilla que fija un `ami-...` deja de funcionar al llevarlo a otra región. Es un problema habitual al replicar entornos.

### 2.2 Tipos y familias de instancia

Una instancia combina **vCPU**, memoria, red y, en algunos casos, **instance store** y **GPU**. El **instance type** fija cuánto de cada cosa se obtiene. Se lee así:

```
   c5 . large
   │└─ generación (5.ª)
   └── familia (c = compute optimized)
         └── tamaño: desde nano hasta varios xlarge
```

Otros ejemplos que cita el curso: `t3`, `a1`, `t2.micro` (el tipo elegible para la capa gratuita, que usa la demo), la familia **G** para gráficos (visualización 3D, codificación de video) y **M5**, de propósito general, adecuada para servidores web como Employee Directory.

| Familia | Para qué está pensada | Casos de uso que cita el curso |
|---|---|---|
| **General purpose** | Balance de cómputo, memoria y red | Servidores web, microservicios en contenedores, flotas de caché, almacenes de datos distribuidos, entornos de desarrollo |
| **Compute optimized** | Aplicaciones limitadas por CPU que se benefician de procesadores de alto rendimiento | Servidores web de alto rendimiento, modelado científico, batch, analítica distribuida, HPC, ML/deep learning, publicación de anuncios, juegos multijugador |
| **Memory optimized** | Procesar grandes datasets en memoria | Bases de datos de alto rendimiento, cachés en memoria distribuidas a escala web, bases en memoria de tamaño medio, big data en tiempo real |
| **Accelerated computing** | Aceleradores o coprocesadores de hardware (coma flotante, gráficos, coincidencia de patrones) | Visualización y renderizado 3D, estaciones de trabajo remotas gráficas, streaming de aplicaciones, codificación de video |
| **Storage optimized** | Alto acceso secuencial de lectura/escritura sobre grandes datasets en almacenamiento local; decenas de miles de IOPS aleatorias de baja latencia | Bases NoSQL (Cassandra, MongoDB, Redis), bases en memoria, bases transaccionales escalables, data warehousing, Elasticsearch, analítica |

**🧭 Ventaja frente a on-premises:** ya no hay que comprometerse con el hardware desde el principio. Se elige un tipo, se mide el rendimiento con la carga real y después se cambia a otro, con unos clics o por API. La lista oficial de tipos se actualiza a medida que AWS lanza tipos nuevos.

> [!example] 📊 Caso: dimensionar para disponibilidad
> Poder elegir el tamaño permite cambiar pocas instancias grandes por **muchas pequeñas**. Con un frontend de una sola instancia, si esta falla se cae la aplicación. Con la carga repartida en 10 instancias, la caída de una supone perder el 10 % de la flota, y la disponibilidad apenas lo nota. La regla del curso: **al menos dos instancias en dos AZ distintas**, porque «dos es mejor que una y tres es mejor que dos».

### 2.3 Dónde vive una instancia: la VPC por defecto

Por defecto, las instancias se colocan en la **default VPC**, creada para empezar con EC2 sin aprender a configurar redes. Todo lo que se ponga en ella es **público y accesible desde Internet**, así que no se deben dejar ahí datos de clientes ni información privada. Una vez que se domina el tema de redes (sección 7 en adelante), lo correcto es usar VPC personalizadas con más controles de enrutamiento y conectividad. Dentro de la red, la instancia reside en una **Availability Zone** elegida por usted, y los servicios con alcance de AZ deben diseñarse pensando en alta disponibilidad.

🎯 **Para el examen:**
- AMI = SO + almacenamiento + arquitectura + software; el AMI ID es regional.
- Instance type = familia + generación + tamaño (`c5.large`).
- Asocie la familia con el caso: G/acelerada para gráficos, storage optimized para NoSQL e IOPS locales, memory optimized para bases y cachés en memoria.
- Alta disponibilidad: mínimo 2 instancias en 2 AZ.

---

## 3. Ciclo de vida de una instancia EC2

Audiencia: 🔧 🧭

```
                 launch (desde AMI)
                        │
                        ▼
                  ┌──────────┐
          ┌──────►│ pending  │  (arranca; aún no se factura)
          │       └────┬─────┘
          │            ▼
          │       ┌──────────┐   reboot (mismo host, mismas IP)
   start  │       │ running  │◄──────────────┐
          │       └┬────┬───┬┘───────────────┘
          │  stop  │    │   │ terminate
          │        ▼    │   ▼
          │  ┌─────────┐│ ┌──────────────┐
          │  │stopping ││ │shutting-down │
          │  └────┬────┘│ └──────┬───────┘
          │       ▼     │        ▼
          │  ┌─────────┐│ ┌──────────────┐
          └──┤ stopped ││ │  terminated  │  (sin vuelta atrás)
             └─────────┘│ └──────────────┘
                 ▲      │ stop-hibernate
                 └──────┘ (pasa por stopping → stopped; RAM guardada en EBS raíz)
```

| Estado / acción | Qué ocurre | Analogía del curso |
|---|---|---|
| **Pending** | AWS prepara la instancia: copia la AMI al root device y asigna los componentes de red. **No se factura.** | La VM arrancando |
| **Running** | Lista para usarse. **Empieza la facturación.** Desde aquí se puede hacer reboot, stop, stop-hibernate o terminate | — |
| **Reboot** | Equivale a reiniciar el SO. Se queda en el **mismo host**, conserva la IP pública y la privada y los datos del instance store. Suele tardar unos minutos | Reiniciar el portátil |
| **Stop → Start** | Puede reubicarse en **otro servidor físico**: se pierden los datos del instance store y de la RAM; obtiene **IP pública nueva** y **conserva la IP privada**. Al volver a arrancar pasa otra vez por pending | Apagar el portátil |
| **Stop-hibernate** | El SO hace *suspend-to-disk*: guarda el contenido de la RAM en el **volumen raíz EBS**. Al reanudar, todo sigue donde estaba, sin arranque completo | Cerrar la tapa del portátil |
| **Terminate** | Pasa por shutting-down a terminated. Se borra el instance store y se pierden **ambas IP**. La máquina deja de existir | Tirar el portátil al mar |

**Costos por estado:**
- Se cobra la instancia en **running**, y también en **stopping** cuando se está preparando para hibernar.
- En **stopped** no se cobra uso ni transferencia de datos, pero **sí el almacenamiento de los volúmenes EBS**.
- Mientras está detenida se pueden cambiar algunos atributos, por ejemplo el **instance type** (así se redimensiona).

> [!danger] 🚨 Terminar es definitivo
> Respalde antes cualquier dato o estado que tenga la instancia. Si preocupa que alguien la termine por accidente, active la **termination protection**. El almacenamiento persistente que sobrevive a la instancia se ve en [[03-Almacenamiento-y-Bases-de-Datos]].

> [!tip] 💡 Terminar no es algo malo
> Ante un problema técnico, un parche o una actualización, en vez de entrar a reparar la instancia se puede **lanzar una nueva con los cambios y terminar la antigua**. Las actualizaciones in place siguen siendo posibles, pero lanzar y terminar según la demanda es la base de la elasticidad (Auto Scaling se ve en el tomo 04).

> [!example] 📊 Caso del curso: cuándo sirve stop-hibernate
> En una aplicación de tres capas (web, aplicación y base de datos) que se volvió muy popular, se agrega una capa backend propia sobre EC2 que guarda en RAM una caché de la base de datos. Con stop-hibernate esa caché sobrevive al apagado sin escribir scripts para volcar la RAM antes de detener el servidor.

**🧭 Ahorro simple:** detener las aplicaciones que solo se usan en horario laboral cuando no hay empleados conectados, y volver a encenderlas al comenzar la jornada.

---

## 4. Precios de EC2

Audiencia: 🧭 👔

El curso separa el **costo de la instancia** (sus especificaciones) del costo total, que también incluye almacenamiento y red.

- La facturación se calcula **por segundo**, aunque los precios se publican **por hora** para simplificar. Ejemplo del curso: una instancia que corrió 5 min 38 s en el mes paga **338 segundos**.
- Excepción: las AMI de terceros de AWS Marketplace pueden tener una facturación mínima de **1 hora**.

| Opción | Cómo funciona | Cuándo conviene | Cuidado |
|---|---|---|---|
| **On-Demand** | Precio fijo por segundo, sin compromiso. Factura en running y deja de facturar en stopped/terminated | Cargas variables, pruebas, lo que se puede apagar | Para servidores 24/7 (como el frontend de Employee Directory) es la opción más cara |
| **Reserved Instances (RI)** | Tarifa por hora con descuento y reserva de capacidad opcional. Plazo de **1 o 3 años**. Pago **All Upfront**, **Partial Upfront** o **No Upfront** | Servidores que no se pueden apagar | Se paga aunque la instancia esté detenida, porque hay compromiso. El descuento va atado a un **instance type** (y a una AZ, según cómo se reserve), no a un instance ID |
| **Spot Instances** | Usan capacidad EC2 ociosa con hasta **90 %** de descuento frente a On-Demand. Usted fija un tope de precio por hora; si supera el precio Spot vigente y hay capacidad, recibe la instancia | Cargas tolerantes a fallos e interrumpibles: big data, contenedores, CI/CD, servidores web, HPC, renderizado de imágenes y medios, pruebas y desarrollo | AWS puede interrumpirla con **2 minutos de aviso** si se acaba la capacidad o el precio Spot supera su tope |

**Orden de descuentos de las RI:** All Upfront > Partial Upfront > No Upfront > On-Demand.

> [!warning] ⚠️ No Upfront no es lo mismo que On-Demand
> Ninguna de las dos exige pagar por adelantado, pero la On-Demand deja de cobrar cuando se detiene y la RI No Upfront sigue cobrando durante todo el plazo de 1 o 3 años.

> [!note] ➕ Nota del editor
> El video de Fargate menciona también los **Compute Savings Plans** como opción de compra (sección 6.2). La lectura de precios de EC2 de este módulo no los desarrolla; el tomo 19 trata la optimización de costos.

🎯 **Para el examen:** carga 24/7 y estable → RI; interrumpible y tolerante a fallos → Spot (aviso de 2 min); impredecible o de corta duración → On-Demand.

---

## 5. Contenedores en AWS: ECS y EKS

Audiencia: 🔧 🧭

### 5.1 Qué es un contenedor

La idea viene de los años 70, cuando algunos kernels de Linux ya podían aislar procesos, aunque había que configurarlo a mano. Hoy un **contenedor** es una unidad estandarizada que empaqueta el código con **todas sus dependencias y su configuración** en un solo ejecutable con entorno propio. Por eso se comporta igual en desarrollo, QA y producción, y se mueve con facilidad de on-premises a la nube. Se usa para aplicaciones web, migraciones *lift and shift*, aplicaciones distribuidas y para unificar los entornos de dev, test y prod.

**Docker** es el container runtime más popular: simplifica la gestión del stack de SO necesario para aislar contenedores (incluidas red y almacenamiento) y facilita crearlos, empaquetarlos, desplegarlos y ejecutarlos.

| | Contenedor | Máquina virtual |
|---|---|---|
| Sistema operativo | Comparte el SO y el **kernel** del host | Cada VM lleva su propio SO (se desperdicia espacio) |
| Peso | Liviano | Pesado |
| Arranque | Casi instantáneo, clave para escalar en ráfagas de E/S | Más lento |
| Ventaja | Velocidad y portabilidad | Toda la potencia de un SO: instalar paquetes, kernel dedicado, más recursos |

### 5.2 Orquestación

En AWS los contenedores corren sobre instancias EC2. Unos pocos contenedores en una instancia grande son fáciles de manejar, pero eso no da alta disponibilidad ni escalabilidad. Lo normal es tener muchos contenedores en muchas instancias repartidas en varias AZ, lo que se llama un **cluster**. A esa escala hay que resolver:

- dónde ubicar cada contenedor;
- qué pasa si falla un contenedor;
- qué pasa si falla una instancia;
- cómo monitorear los despliegues.

Iniciar, detener, reiniciar y monitorear contenedores en un cluster es la **orquestación**. Con cientos o miles de contenedores se vuelve muy compleja para hacerla uno mismo. AWS ofrece dos orquestadores: **Amazon ECS** y **Amazon EKS**. Usted interactúa con la API del orquestador y este ejecuta las tareas de gestión. Se puede automatizar tanto el escalado del cluster como el de los propios contenedores.

### 5.3 Amazon ECS

Orquestador nativo de AWS para lanzar y administrar contenedores en un cluster de instancias EC2.

- En cada instancia se instala el **Amazon ECS container agent** (código abierto, disponible para AMI Linux y Windows), que informa al servicio ECS los detalles de gestión del cluster. Una instancia con el agente se llama **container instance**.
- Con el cluster en marcha se puede: lanzar y detener contenedores, consultar el estado del cluster, escalar hacia adentro y hacia afuera, programar la ubicación de contenedores, asignar permisos y cumplir requisitos de disponibilidad.
- Para preparar la aplicación se escribe una **task definition**: un archivo de texto **JSON** que describe uno o varios contenedores y los recursos que necesitan (CPU, memoria, puertos, imágenes, almacenamiento, red). Funciona como el plano de la tarea.

Ejemplo de la lectura (servidor web Nginx para el directorio corporativo):

```json
{
  "family": "webserver",
  "containerDefinitions": [ {
      "name": "web",
      "image": "nginx",
      "memory": "100",
      "cpu": "99"
  } ],
  "requiresCompatibilities": [ "FARGATE" ],
  "networkMode": "awsvpc",
  "memory": "512",
  "cpu": "256"
}
```

> [!note] ➕ Nota del editor
> Fíjese en que la task definition ya declara `"requiresCompatibilities": ["FARGATE"]` y `"networkMode": "awsvpc"`: la tarea está pensada para Fargate (sección 6) y obtiene su propia interfaz de red dentro de la VPC.

### 5.4 Amazon EKS

**Kubernetes** es una plataforma de código abierto, portable y extensible para administrar cargas y servicios en contenedores. Une desarrollo y operaciones y tiene un ecosistema grande y consolidado. Si ya se usa Kubernetes, **Amazon EKS** permite orquestar esas cargas en AWS.

| Concepto | Amazon ECS | Amazon EKS |
|---|---|---|
| Instancia EC2 que corre contenedores | **Container instance** (con el ECS agent) | **Worker node** |
| Unidad que se ejecuta | **Task** | **Pod** |
| Tecnología base | Nativa de AWS | Kubernetes |
| Cuándo elegirlo | Orquestar a escala sin administrar software de orquestación propio | Ya se usa Kubernetes; orquestación avanzada con simplicidad, alta disponibilidad y control fino de la infraestructura |

> [!tip] 💡 Lo que más valora el instructor
> La **portabilidad**: un contenedor probado en el portátil y en QA debería comportarse igual en producción.

---

## 6. Serverless: Fargate y Lambda

Audiencia: 🔧 🧭 👔

### 6.1 Qué significa serverless

Con EC2, o con contenedores sobre EC2, usted sigue a cargo de parchear las instancias, configurar el escalado y diseñar para alta disponibilidad (al menos 2 AZ). Es menos trabajo que on-premises, pero los procesos de gestión los arma usted. Ese control a veces es justo lo que se busca. Otras veces se prefiere centrarse en el negocio y delegar el **undifferentiated heavy lifting** (el trabajo pesado que no diferencia a la empresa).

**Serverless** significa que **no se ve ni se accede** a la infraestructura ni a las instancias subyacentes: aprovisionamiento, escalado, tolerancia a fallos y mantenimiento quedan a cargo de AWS. Las cuatro características que repite toda definición:

1. No hay servidores que aprovisionar ni administrar.
2. Escala con el uso.
3. Nunca se paga por recursos ociosos.
4. Disponibilidad y tolerancia a fallos integradas.

**Impacto en el modelo de responsabilidad compartida** (tomo 01):

| Modelo | AWS responde por | Usted responde por |
|---|---|---|
| EC2 | Hardware físico | SO invitado, parches, red, seguridad, escalado |
| ECS/EKS sobre EC2 | Además: desplegar contenedores en instancias y administrar el cluster | Mantener las instancias EC2 subyacentes |
| Serverless (Fargate, Lambda) | La línea **sube**: SO, parches, capacidad | Cifrado de datos, gestión de accesos, su código y configuración |

> [!quote] Werner Vogels, CTO de Amazon
> «Ningún servidor es más fácil de administrar que ningún servidor.»

### 6.2 AWS Fargate

ECS y EKS son los **orquestadores**, que gestionan el ciclo de vida del contenedor. Además hace falta una **plataforma de cómputo** donde los contenedores corran. ECS y EKS admiten dos modos:

- **Modo Amazon EC2:** un cluster de instancias que usted controla y mantiene (no es serverless).
- **Modo AWS Fargate:** motor de cómputo serverless creado para contenedores.

Con Fargate el escalado y la tolerancia a fallos vienen integrados, y no hay que preocuparse del SO ni del entorno. Usted define el contenido de la aplicación, la red, el almacenamiento y los requisitos de escalado. No tiene que aprovisionar, parchear ni gestionar la capacidad del cluster. Fargate asigna la cantidad justa de cómputo, aísla las cargas por diseño, mantiene las mismas primitivas de ECS, su API y sus integraciones, y se integra de forma nativa con **IAM** y **Amazon VPC** (los contenedores Fargate se lanzan dentro de su red y se controla su conectividad).

**Flujo de trabajo con Fargate:**
1. Construir las imágenes de contenedor y subirlas (*push*) a un repositorio, por ejemplo **Amazon Elastic Container Registry (ECR)**, desde donde se descargan (*pull*) para desplegarlas.
2. Definir memoria y cómputo para la **task** (ECS) o el **pod** (EKS).
3. Ejecutar los contenedores.
4. Pagar solo por la vCPU, la memoria y el almacenamiento que consumen.

Fargate admite opciones de compra **Spot** y **Compute Savings Plans**, igual que EC2. Casos de uso: microservicios, procesamiento batch, aplicaciones de ML y migración de aplicaciones on-premises a la nube.

### 6.3 AWS Lambda

**🔧 Definición.** Se empaqueta y sube el código al servicio, lo que crea una **Lambda function**. La función no corre todo el tiempo: se ejecuta en respuesta a **triggers**. Según el tipo de trigger, Lambda espera el evento o hace *polling* para detectarlo. Al dispararse, el código corre en un entorno administrado que escala solo, es altamente disponible y lo mantiene AWS. Con 1 o con 1.000 eventos entrantes, Lambda escala la función y ejecuta cada invocación en su propio entorno aislado.

**Los tres componentes de una función:**

| Componente | Qué es | Detalle |
|---|---|---|
| **Código** | Lo que la función ejecuta | Se escribe desde cero, se parte de un **blueprint** de AWS o se toma del **AWS Serverless Application Repository** (ejemplos: hello world, Alexa Skill, redimensión de imágenes, codificación de video). Lambda también admite **container images** |
| **Configuración** | Cómo se ejecuta | Runtime, ubicación en la red, variables de entorno, memoria (y la CPU que va con ella), tipo de invocación, permisos (vía **IAM role**), dependencias |
| **Trigger** | Cuándo se ejecuta | Integra la función con otros servicios: solicitud HTTP, carga a S3, eventos de servicios AWS, actividad en apps móviles; en la consola aparecen, entre otros, API Gateway, Application Load Balancer y DynamoDB |

**Runtimes incorporados:** Python, Node.js, Ruby, Go, Java y .NET Core, además de runtimes personalizados.

**Límites y encaje:**
- Pensada para código que dura **menos de 15 minutos**. No sirve para procesos largos como deep learning o batch pesados.
- No es para alojar algo como un sitio WordPress.
- Encaja en procesamiento rápido: backend web que atiende solicitudes, procesamiento de reportes, microservicios, procesamiento de datos y de streams en tiempo real, ML, WebSockets, backends de IoT y móviles.

**Facturación:** se cobra por **número de invocaciones** (requests) y por **duración**, redondeada al **milisegundo** más cercano y sin mínimo. No se cobra el código que no corre, lo que la hace muy rentable para funciones de menos de 100 ms o API de baja latencia.

**El handler.** Es el método del código que procesa los eventos. Lambda lo ejecuta en cada invocación y, cuando termina o devuelve una respuesta, queda disponible para el evento siguiente. En Python la sintaxis general es:

```python
def handler_name(event, context):
    ...
    return some_value
```

El nombre del handler combina el **archivo** y la **función**. El valor por defecto en la consola es `lambda_function.lambda_handler` (función `lambda_handler` en el archivo `lambda_function.py`). Si se cambia, hay que actualizarlo en el panel de configuración del runtime.

### 6.4 Control frente a conveniencia

El instructor ordena los servicios en un espectro: en un extremo el **control** (EC2) y en el otro la **conveniencia** (Lambda). Contenedores y Fargate quedan en medio.

🎯 **Para el examen:**
- Lambda: máximo 15 min, cobro por invocaciones + duración en ms, se activa por eventos.
- Fargate = serverless **para contenedores**, compatible con ECS y EKS.
- ECS task ↔ EKS pod; container instance ↔ worker node.
- En serverless, usted sigue a cargo del cifrado y de los accesos.

---

## 7. Cómo elegir el servicio de cómputo

Audiencia: 🧭 👔

El curso lo plantea como un concurso con tres casos:

| Caso | Respuesta | Razonamiento |
|---|---|---|
| **1.** Tienda online en EC2. Hoy alguien carga a mano cada artículo nuevo en la base de inventario. Se quiere que una persona suba una planilla de inventario a S3 y que un proceso la cargue sola. **El inventario se actualiza una vez por trimestre.** | **AWS Lambda** | Una EC2 haciendo polling sobre S3 funcionaría, pero se pagaría por segundo u hora un recurso que casi no se usa. Lambda cobra solo mientras corre. Se configura un evento **PUT** de S3 como trigger, y la función lee la planilla e inserta cada artículo en la base |
| **2.** Aplicación on-premises en servidores Linux que hay que migrar **con la mínima refactorización** y que debe ser elástica ante demanda variable | **Amazon EC2** | Se lanza desde una AMI Linux y la aplicación se aloja igual que on-premises; EC2 escala hacia adentro y hacia afuera. Lambda exigiría refactorizar bastante y pasar a contenedores también requiere retrabajo |
| **3.** Aplicación nueva con diseño de **microservicios** u orientado a servicios, que escale rápido y reduzca el riesgo de desplegar cambios en producción | **Amazon ECS o EKS** | Los contenedores facilitan los microservicios, arrancan más rápido que las instancias (escalan más rápido) y su portabilidad evita los errores por diferencias de entorno entre laptop, QA y producción |

> [!warning] ⚠️ Regla del cierre
> No use el mismo servicio de cómputo para todo. Elija el adecuado para cada trabajo y **reevalúe las cargas existentes** a medida que AWS lanza y mejora servicios.

```
 ¿La carga es esporádica, dirigida por eventos y dura < 15 min? ──sí──► Lambda
            │no
 ¿Migración con mínima refactorización o necesito el SO? ─────sí──► EC2
            │no
 ¿Microservicios, portabilidad, escalado rápido? ────────────sí──► ECS / EKS
            │                                                       │
            │                     ¿No quiero administrar instancias?─┴─sí─► + Fargate
```

---

## 8. Fundamentos de redes: IP y CIDR

Audiencia: 🔧

**Networking** es conectar computadores para que se comuniquen. La infraestructura global de AWS (data centers, AZ y regiones) ya es un ejemplo de red.

**Analogía de la carta:** para enviar una carta se necesitan el contenido (*payload*), la dirección del remitente y la del destinatario, cada una con nombre, calle, ciudad, región, código postal y país. Sin una dirección completa no llega. Entre computadores, esa entrega se llama **routing** y la dirección es la **IP address**.

- Una dirección IPv4 tiene **32 bits** (unos y ceros).
- Los 32 bits se agrupan en 4 **octetos** de 8 bits, cada uno se pasa a decimal y se separan con puntos: eso es la **notación IPv4** (p. ej., `192.168.1.30`).

**CIDR (Classless Inter-Domain Routing)** es una forma compacta de escribir un **rango** de direcciones: una IP inicial, una barra y un número que indica cuántos bits quedan **fijos**.

```
 192.168.1.0/24
             └─ 24 bits fijos → 32 − 24 = 8 bits libres → 2^8 = 256 direcciones
                (de 192.168.1.0 a 192.168.1.255)
```

**Cuanto mayor es el número después de la barra, más pequeña es la red:** `/24` es menor que `/16`.

| En AWS | CIDR | Direcciones |
|---|---|---|
| Rango **más pequeño** permitido | `/28` | 16 |
| Rango **más grande** permitido | `/16` | 65.536 |

---

## 9. Amazon VPC, subnets y gateways

Audiencia: 🔧 🧭

### 9.1 VPC

Una **VPC (Virtual Private Cloud)** es una red aislada que usted crea en AWS, equivalente a la red de un data center tradicional y a sus muros: **nada entra ni sale sin permiso explícito**. Al crearla se eligen:

1. **Nombre.**
2. **Región**: la VPC abarca varias AZ de esa región.
3. **Rango IP en notación CIDR**, que define el tamaño de la red. Cada VPC admite hasta **cuatro rangos /16**.

AWS crea una **VPC por defecto en cada región** para empezar rápido, pero lo que se ponga en ella queda expuesto a Internet. Aunque una función Lambda puede no necesitar red, el curso subraya que la red es la base de la mayoría de las arquitecturas.

### 9.2 Subnets

Las **subnets** son segmentos más pequeños dentro de la VPC, el equivalente a las VLAN on-premises. Allí no sirven tanto para aislar u optimizar tráfico como en una red local: en AWS se usan para **alta disponibilidad** y para dar **distintas opciones de conectividad** (recursos públicos como el frontend de Employee Directory, y privados como una base de datos). Al crear una subnet se elige:

1. La **VPC** a la que pertenece.
2. La **AZ** donde vive (una subnet está en una sola AZ, y las instancias que se lanzan en ella quedan en esa AZ).
3. Un **CIDR que sea subconjunto** del de la VPC y **no se solape** con otras subnets.

> [!warning] ⚠️ Alta disponibilidad desde la red
> Cree al menos **dos subnets en dos AZ distintas** y aloje los recursos de forma redundante. «Todo falla todo el tiempo»: si cae una AZ, la otra toma el tráfico.

### 9.3 Las 5 IP reservadas

AWS reserva **cinco direcciones en cada subnet** para enrutamiento, DNS y administración de red.

> [!example] 📊 Cálculo de la lectura
> VPC `10.0.0.0/22` = 1.024 direcciones, dividida en 4 subnets `/24` de 256 direcciones cada una. Direcciones utilizables por subnet: 256 − 5 = **251**.

**Punto de partida recomendado para quien empieza:** VPC `/16` y subnets `/24`, que dan direcciones de sobra en ambos niveles.

### 9.4 Gateways

| Gateway | Para qué | Detalles |
|---|---|---|
| **Internet gateway (IGW)** | Conecta la VPC con Internet, como un módem | A diferencia del módem de casa, es altamente disponible y escalable. Hay que **adjuntarlo** a la VPC; si no, no sirve de nada. Relación **1 a 1**: un IGW solo se adjunta a una VPC |
| **Virtual private gateway (VGW)** | Conecta la VPC con otra red privada (data center, red corporativa) sin acceso directo desde Internet | Es el ancla del lado AWS. Del otro lado se instala un **customer gateway** (dispositivo físico o software). Con ambos se establece una **VPN cifrada** |

---

## 10. Enrutamiento: route tables

Audiencia: 🔧 🧭

Que el tráfico entre por la puerta (IGW) no garantiza que llegue a la sala correcta (la subnet). De eso se encargan las **route tables**: un conjunto de reglas (**routes**) que deciden hacia dónde se envía el tráfico. Se aplican a nivel de VPC o de subnet.

Cada ruta tiene dos partes:

| Campo | Qué indica | Ejemplo |
|---|---|---|
| **Destination** | Rango IP al que va el tráfico | `10.1.0.0/16` (la VPC) o `0.0.0.0/0` (cualquier lugar) |
| **Target** | Conexión por la que se envía | `local` o el `igw-...` |

**Main route table.** AWS la crea con la VPC y la aplica a toda la red. Su configuración por defecto permite el tráfico entre todas las subnets de la VPC mediante la **ruta local** (destino = CIDR de la VPC). A eso se refiere el curso cuando dice que una VPC está aislada: por defecto sus recursos solo se ven entre sí.

**Custom route tables.** Para enrutar de forma distinta cada subnet (frontend y base de datos, por ejemplo) se crean tablas propias. Si se asocia una tabla personalizada a una subnet, esta **la usa en lugar de la principal**. Toda tabla nueva trae ya la ruta local.

> [!warning] ⚠️ Lo que hace pública a una subnet es su route table
> Una subnet no es pública ni privada por sí misma. Es **pública** si su route table tiene una ruta `0.0.0.0/0 → internet gateway`, y **privada** si no la tiene. Las subnets sin asociación explícita siguen la main route table (solo tráfico local).

```
                         Internet
                            │
                      ┌─────┴─────┐
                      │  app-igw  │
   ┌──────────────────┴───────────┴───────────────────┐ app-vpc 10.1.0.0/16
   │  public-route-table: 10.1.0.0/16→local, 0.0.0.0/0→igw                    │
   │    ┌──────────────┐            ┌──────────────┐  │
   │    │Public Sub 1  │            │Public Sub 2  │  │
   │    │10.1.1.0/24   │            │10.1.3.0/24   │  │
   │    ├──────────────┤            ├──────────────┤  │
   │    │Private Sub 1 │            │Private Sub 2 │  │
   │    │10.1.2.0/24   │            │10.1.4.0/24   │  │
   │    └──────────────┘            └──────────────┘  │
   │       us-west-2a                  us-west-2b     │
   │  main route table (privadas): 10.1.0.0/16→local  │
   └──────────────────────────────────────────────────┘
```

---

## 11. Seguridad de la VPC: network ACL y security groups

Audiencia: 🔧 🧭

Una VPC nueva está aislada de Internet. En cuanto se abren rutas, hacen falta controles. AWS ofrece dos capas.

### 11.1 Network ACL: firewall de subnet

Controla qué tráfico entra a la subnet y qué tráfico sale de ella.

- La **network ACL por defecto permite todo** el tráfico de entrada y de salida, y es un buen punto de partida.
- Admite reglas de **allow y de deny**, evaluadas por número de regla. La regla `*` final deniega lo que no haya cubierto una regla anterior y no se puede modificar.
- Es **stateless**: hay que abrir de forma explícita **tanto la entrada como la salida** del protocolo.

Network ACL por defecto:

| Dirección | Regla | Tipo | Protocolo | Puertos | Origen/Destino | Acción |
|---|---|---|---|---|---|---|
| Inbound | 100 | Todo IPv4 | Todos | Todos | 0.0.0.0/0 | ALLOW |
| Inbound | * | Todo IPv4 | Todos | Todos | 0.0.0.0/0 | DENY |
| Outbound | 100 | Todo IPv4 | Todos | Todos | 0.0.0.0/0 | ALLOW |
| Outbound | * | Todo IPv4 | Todos | Todos | 0.0.0.0/0 | DENY |

Ejemplo restrictivo para servidores web (HTTPS desde cualquier lugar y RDP solo desde la red doméstica):

| Dirección | Regla | IP | Protocolo | Puerto | Acción | Para qué |
|---|---|---|---|---|---|---|
| Inbound | 100 | Todo IPv4 | TCP | 443 | ALLOW | HTTPS entrante desde cualquier lugar |
| Inbound | 130 | 192.0.2.0/24 | TCP | 3389 | ALLOW | RDP desde el rango público de la red doméstica (vía IGW) |
| Inbound | * | Todo IPv4 | Todos | Todos | DENY | Lo demás |
| Outbound | 120 | 0.0.0.0/0 | TCP | **1025-65535** | ALLOW | Respuestas a los clientes en Internet |
| Outbound | * | 0.0.0.0/0 | Todos | Todos | DENY | Lo demás |

> [!danger] 🚨 El error típico con una NACL stateless
> HTTPS inicia la conexión en el puerto 443, pero el servidor responde a un **puerto efímero** del cliente. Si solo se abre la entrada 443 y no la salida 1025-65535, la solicitud entra y la respuesta nunca sale de la subnet.

### 11.2 Security group: firewall de instancia

- Rodea a cada instancia EC2 y es **obligatorio**: toda instancia debe estar en un security group.
- Por defecto **bloquea todo el tráfico entrante y permite todo el saliente**.
- Solo admite reglas de **allow**.
- Es **stateful**: recuerda quién inició la conexión y deja pasar la respuesta sin reglas adicionales.
- Está **ligado a la VPC** (en la demo de relanzamiento, el security group de la VPC por defecto no se puede usar en la VPC nueva).

Reglas de entrada para un servidor web:

| Tipo | Protocolo | Puerto | Origen |
|---|---|---|---|
| HTTP | TCP (6) | 80 | 0.0.0.0/0 |
| HTTP | TCP (6) | 80 | ::/0 |
| HTTPS | TCP (6) | 443 | 0.0.0.0/0 |
| HTTPS | TCP (6) | 443 | ::/0 |

**Patrón de tres capas con security groups:** se crea un grupo por capa y se encadenan:

```
 Internet ──HTTPS──► [SG web tier] ──HTTP──► [SG app tier] ──MySQL──► [SG database tier]
```

On-premises ese aislamiento se hacía con VLAN. En AWS los security groups lo consiguen **sin atarlo a la topología de red**.

### 11.3 Comparación

| | Network ACL | Security group |
|---|---|---|
| Nivel | Subnet | Instancia EC2 |
| Estado | **Stateless** (abrir entrada y salida) | **Stateful** (la respuesta pasa sola) |
| Reglas | Allow y deny | Solo allow |
| Valor por defecto | Permite todo | Bloquea entrada, permite salida |
| ¿Obligatorio? | Toda subnet tiene una | Toda instancia debe tener uno |

**🧭 Qué hace el curso:** deja las NACL por defecto y restringe con security groups, lo que es cómodo y seguro. Para una capa adicional se pueden afinar las NACL.

🎯 **Para el examen:** stateless = NACL, stateful = SG. Solo la NACL puede **denegar** de forma explícita (por ejemplo, bloquear una IP). Recuerde los puertos efímeros 1025-65535 de salida en la NACL.

---

## 12. Conectividad híbrida

Audiencia: 🧭 👔

Employee Directory vive entera en la nube, pero muchas soluciones son **híbridas**: una parte en AWS y otra en un data center on-premises.

| Opción | Qué es | Cuándo conviene |
|---|---|---|
| **AWS Site-to-Site VPN** | Conecta una red remota (data center del cliente) con una VPC | Conexión segura y habitual entre sitios |
| **AWS Client VPN** | Conecta a personas (administradores) con AWS o con el data center, como la VPN del portátil para llegar a recursos corporativos | Acceso remoto de usuarios |
| **AWS Direct Connect** | Conexión privada **dedicada** a AWS mediante un socio de entrega de Direct Connect o mediante AWS. El tráfico se queda en la red global de AWS y **nunca toca la Internet pública** | Mucho volumen de tráfico, rendimiento más alto y confiable, menos cuellos de botella y menos saltos de latencia que una VPN |

AWS VPN se conecta a la VPC a través del **virtual private gateway**, que es para el data center privado lo que el IGW es para Internet. El VGW también sirve de puerta para Direct Connect. Las opciones se pueden combinar: **VPN como failover de Direct Connect**.

---

## 13. Troubleshooting: la instancia pública no carga

Audiencia: 🔧

Lista de verificación de la lectura, en orden:

| # | Qué revisar | Qué buscar |
|---|---|---|
| 1 | **Internet gateway** | Que exista y esté adjunto a la VPC |
| 2 | **Route table** de la subnet | Una ruta `0.0.0.0/0 → IGW` |
| 3 | **Security group** | Entrada 80 y/o 443 desde `0.0.0.0/0`; salida permitida |
| 4 | **Network ACL** | Entrada y salida para HTTP/HTTPS (es stateless) |
| 5 | **IP pública** | Que la instancia tenga una (ver detalles en la consola EC2); si no, revisar la configuración de asignación de IP pública |
| 6 | **HTTP o HTTPS** | En el curso la app corre por **HTTP**: verificar que el navegador no fuerce `https://`. Si se usa HTTPS, revisar los certificados SSL/TLS |
| 7 | **User data** | Logs `/var/log/cloud-init.log` y `/var/log/cloud-init-output.log` |
| 8 | **Permisos IAM** | Que la instancia tenga el rol y las políticas para S3, DynamoDB o RDS |
| 9 | **Red personal o corporativa** | Firewalls o proxies que bloqueen la IP o el puerto |
| 10 | **Aplicación** | Que el código esté desplegado y corriendo, revisar sus logs y que el servidor web (Apache, Nginx) esté activo |

---

## 14. Demos y laboratorios

Audiencia: 🔧

### 14.1 Demo: lanzar Employee Directory en la VPC por defecto

**Objetivo:** levantar la aplicación en EC2 con acceso público directo. **Servicios:** EC2, IAM, S3 (origen del código), VPC por defecto.

1. En el dashboard de EC2: **Launch instance**. Nombre: `employee directory app`.
2. **AMI:** Amazon Linux 2023 (también se podría usar el Marketplace o AMI propias aprobadas).
3. **Instance type:** `t2.micro` (capa gratuita, seleccionado por defecto).
4. **Key pair:** *Proceed without a key pair*. Para entrar a la instancia (por ejemplo, a ver logs) se usará el botón **Connect** de la consola, que no lo requiere.
5. **Network settings:** VPC y subnet por defecto, que ya tienen un IGW adjunto. **Auto-assign public IP: Enable.**
6. **Firewall (security group):** desactivar SSH y permitir **HTTP** y **HTTPS** (la app usa HTTP).
7. **Storage:** solo el volumen raíz, sin volúmenes EBS adicionales.
8. **Advanced details → IAM instance profile:** el rol de la aplicación web, con permisos para llamar a **S3** y **DynamoDB**. Así el AWS SDK del código obtiene **credenciales temporales**.
9. **User data** (script bash que corre al lanzar):
   - declara el intérprete bash;
   - `wget` descarga un zip de la app desde un bucket S3, lo descomprime y entra al directorio;
   - `yum` instala Python 3 y pip;
   - `pip` instala las dependencias de `requirements.txt` (la app es **Python + Flask**);
   - instala el paquete **stress**, que más adelante simula picos de CPU para probar Auto Scaling;
   - define tres variables de entorno: el bucket de fotos (por ahora un placeholder), la región por defecto (**Oregón**, `us-west-2`) y el modo DynamoDB activado;
   - ejecuta la app en el **puerto 80**.
10. **Launch instance**. En el detalle se ven la IP pública, la IP privada y el DNS público. El estado pasa a running, pero hay que esperar a que pasen los **dos status checks**.
11. Abrir la IP pública en otra pestaña. Resultado: **directorio vacío**, lo esperable porque aún no existen el bucket S3 ni la tabla DynamoDB.

**Errores comunes:** probar antes de que pasen los status checks, o que el navegador fuerce HTTPS cuando la app corre por HTTP.

### 14.2 Demo: Lambda que redimensiona imágenes

**Objetivo:** generar miniaturas uniformes de las fotos que se suben al directorio sin tener una EC2 encendida 24/7. **Servicios:** Lambda, S3, IAM.

```
 foto ──PUT──► S3 bucket /input/ ──trigger──► Lambda ResizeImage ──► S3 mismo bucket /output/ (miniatura)
```

1. Consola Lambda → **Create function** → **Author from scratch** (las alternativas son blueprint o container image).
2. Nombre `ResizeImage`, runtime **Python 3.9**.
3. Permisos: **Use an existing role**, `LambdaS3FullAccess` (lee y escribe en S3).
4. **Add trigger** → S3. Bucket `photo-upload-bucket-mw`, tipo de evento **PUT** y prefijo **`input/`**.
5. Confirmar el aviso de **invocación recursiva**: si la entrada y la salida están en el mismo bucket, la escritura de la miniatura podría volver a disparar la función. Se evita escribiendo la salida en otro prefijo (`output/`).
6. Pestaña **Code**: reemplazar el stub «Hello from Lambda» con **Upload from → .zip file** (`ResizeImage.zip`) → Save.
7. Prueba: en S3, subir una foto al prefijo de entrada. Aparece el prefijo de salida con la miniatura.

**Notas del material:** el código es una muestra que ilustra un patrón para generar recursos de forma diferida (*lazy*) con Lambda y S3. Para producción se recomienda **Serverless Image Handler**, desplegable con una plantilla de **AWS CloudFormation**. El instructor comenta que todo el backend de Employee Directory podría pasar a Lambda con algo de refactorización (se ve en tomos posteriores).

### 14.3 Construcción conceptual de la VPC (videos de redes)

En los videos, Seph y Morgan arman la red paso a paso:
1. Comprobar la región (**Oregón**) en la esquina superior derecha de la consola.
2. VPC → **Create VPC**, CIDR `10.1.0.0/16`, nombre `app-vpc`, el resto por defecto.
3. Subnets → **Create subnet**: Public Subnet 1 (`us-west-2a`, `10.1.1.0/24`) y Private Subnet 1 (`us-west-2a`, `10.1.3.0/24`).
4. Internet gateways → **Create** → Actions → **Attach to VPC** (`app-vpc`).
5. Por alta disponibilidad se duplican las subnets en una segunda AZ (4 subnets en total) y se lanza una instancia en una subnet pública.
6. Route tables: revisar la main route table de `app-vpc` (ruta local) y crear `app-routetable-public` con `0.0.0.0/0 → app IGW`, asociada a las dos subnets públicas. Para las privadas se seguirían los mismos pasos, pero sin ruta al IGW.

### 14.4 Lab 2: Creación de una VPC (ítem calificado, sin texto en el material)

**Objetivo deducible del video de introducción:** crear la red en AWS, lanzar una instancia EC2 en una subnet pública con el script de user data del curso y un security group que deje entrar el tráfico de Internet. El instructor recomienda recordar el propósito de cada paso y, si algo falla, retroceder unos pasos y repetir.

### 14.5 Demo: relanzar Employee Directory en la VPC propia

1. VPC → **Create VPC** → opción **VPC only** (el asistente también puede crear subnets y otros recursos), nombre `app-vpc`, CIDR `10.1.0.0/16`.
2. Subnets → **Create subnet** en `app-vpc`, las cuatro en una sola pasada:

| Subnet | AZ | CIDR |
|---|---|---|
| Public Subnet 1 | us-west-2a | 10.1.1.0/24 |
| Private Subnet 1 | us-west-2a | 10.1.2.0/24 |
| Public Subnet 2 | us-west-2b | 10.1.3.0/24 |
| Private Subnet 2 | us-west-2b | 10.1.4.0/24 |

   Los rangos no se solapan y todos son subconjunto del CIDR de la VPC.
3. Internet gateways → **Create**, nombre `app-igw` → **Attach to VPC** `app-vpc` (la VPC por defecto ya tiene el suyo; la relación es 1 a 1).
4. Route tables: ya existen las dos main route tables (de la VPC por defecto y de `app-vpc`). **Create route table** `public-route-table` en `app-vpc` → **Edit routes** → `0.0.0.0/0 → app-igw` → Save.
5. **Subnet associations** → **Edit** → marcar solo Public Subnet 1 y 2 → Save. En la pestaña Routes quedan la ruta local y la ruta a Internet.
6. EC2 → seleccionar la instancia original (que sigue en la VPC por defecto) → **Actions → Image and templates → Launch more like this**, que trae la configuración ya cargada.
7. Nombre `Employee Directory App 2`. Se mantienen la AMI y `t2.micro`. Key pair: proceder sin él.
8. **Network settings:** `app-vpc`, **Public Subnet 1**, auto-assign public IP habilitado.
9. **Security group:** aparece un error porque el SG anterior pertenece a la VPC por defecto. Se crea uno nuevo con HTTP 80 y HTTPS 443 desde cualquier lugar.
10. Advanced details: el IAM role y el user data vienen ya cargados → **Launch instance**.
11. Pasa de pending a running. Al abrir la nueva IP pública, la aplicación carga, lo que confirma que la red quedó bien configurada.

### 14.6 Ítems sin texto

- **Coach** «Conexión de los servicios de AWS con soluciones reales»: diálogo interactivo, sin contenido extraíble.
- **Cuestionario de la semana 2**: evaluación calificada, sin texto.

---

## 15. Aplicación de ejemplo: estado al cierre del módulo

Audiencia: 🔧 🧭

| Pieza | Tomo 01 | Al cierre del tomo 02 |
|---|---|---|
| Red | VPC por defecto | **`app-vpc` 10.1.0.0/16** con 4 subnets en 2 AZ de Oregón |
| Acceso a Internet | IGW de la VPC por defecto | **`app-igw`** + `public-route-table` asociada a las subnets públicas |
| Cómputo | Instancia EC2 | **Employee Directory App 2** (`t2.micro`) en Public Subnet 1 con IP pública |
| Firewall | SG de la VPC por defecto | **SG nuevo** en `app-vpc` (HTTP 80, HTTPS 443) |
| Identidad | IAM role | El mismo **instance profile** (S3 + DynamoDB) |
| Datos | — | Aún **sin** bucket S3 ni tabla DynamoDB: el directorio sigue vacío |
| Extra | — | Función Lambda de redimensión de fotos mostrada como demo aparte |

```
                              Internet
                                 │
                            ┌────┴────┐
                            │ app-igw │
 ┌──────────────────────────┴─────────┴───────────────────────────┐
 │ app-vpc 10.1.0.0/16  (us-west-2, Oregón)                        │
 │                                                                 │
 │  ┌── us-west-2a ─────────────┐    ┌── us-west-2b ─────────────┐ │
 │  │ Public Subnet 1 10.1.1.0/24│    │ Public Subnet 2 10.1.3.0/24│ │
 │  │  ┌──────────────────────┐ │    │                           │ │
 │  │  │ SG (80/443)          │ │    │   (lista para redundancia)│ │
 │  │  │ EC2 Employee Dir. 2  │ │    │                           │ │
 │  │  └──────────────────────┘ │    │                           │ │
 │  │ Private Subnet 1 10.1.2.0/24│  │ Private Subnet 2 10.1.4.0/24│ │
 │  └───────────────────────────┘    └───────────────────────────┘ │
 │  public-route-table → subnets públicas: local + 0.0.0.0/0→IGW   │
 │  main route table   → subnets privadas: solo local              │
 └─────────────────────────────────────────────────────────────────┘
        IAM instance profile → (S3, DynamoDB: se crean en el tomo 03)
```

**Lo que falta y viene después:** S3 para las fotos y DynamoDB para los datos ([[03-Almacenamiento-y-Bases-de-Datos]]), y después alta disponibilidad real con load balancer y Auto Scaling repartidos en ambas AZ (el paquete `stress` ya quedó instalado para esa prueba).

---

## ✅ Autoevaluación

> [!question]- 1. Una instancia se detiene y luego se vuelve a iniciar. ¿Qué cambia y qué se conserva?
> Puede quedar en otro host físico, así que se pierden los datos del instance store y de la RAM, y recibe una **IP pública nueva**. Conserva la **IP privada** y los volúmenes EBS, que se siguen cobrando mientras está detenida. Un reboot, en cambio, mantiene el host y las dos IP.

> [!question]- 2. ¿Por qué una RI No Upfront no equivale a On-Demand si ninguna exige pago adelantado?
> Porque la RI es un compromiso de 1 o 3 años y se paga aunque la instancia esté detenida. La On-Demand deja de cobrar al detenerla o terminarla.

> [!question]- 3. ¿Qué cargas son buenas candidatas para Spot y por qué?
> Las tolerantes a fallos e interrumpibles (big data, contenedores, CI/CD, HPC, renderizado, test/dev), porque AWS puede reclamar la instancia con 2 minutos de aviso a cambio de hasta 90 % de descuento.

> [!question]- 4. ¿Cuál es la diferencia entre ECS/EKS y Fargate?
> ECS y EKS son **orquestadores** que gestionan el ciclo de vida de los contenedores. Fargate es una **plataforma de cómputo** serverless donde corren esos contenedores, como alternativa al modo EC2, en la que usted administra las instancias.

> [!question]- 5. ¿Por qué el proceso trimestral de carga de inventario va en Lambda y no en EC2?
> Porque correría una vez por trimestre. Una EC2 encendida se pagaría por segundo u hora casi sin uso, mientras que Lambda cobra solo por invocación y duración y se dispara con el evento PUT de S3.

> [!question]- 6. Una VPC `10.0.0.0/22` se divide en 4 subnets iguales. ¿Cuántas IP utilizables tiene cada una?
> Cada una es `/24` (256 direcciones). Menos las 5 que reserva AWS quedan **251**.

> [!question]- 7. ¿Qué hace que una subnet sea pública?
> Solo su **route table asociada**: debe tener una ruta `0.0.0.0/0` hacia un internet gateway adjunto a la VPC. Para que la instancia sea alcanzable, además necesita IP pública y reglas que lo permitan en el SG y la NACL.

> [!question]- 8. Se abrió el puerto 443 de entrada en la NACL y el sitio no responde. ¿Qué falta?
> Una regla **de salida** para los puertos efímeros (1025-65535), porque la NACL es stateless. Con un security group no haría falta, porque es stateful.

> [!question]- 9. Al relanzar la instancia en `app-vpc`, la consola no deja reutilizar el security group original. ¿Por qué?
> Porque los security groups están ligados a una VPC. El original pertenecía a la VPC por defecto, así que hay que crear uno nuevo en `app-vpc`.

> [!question]- 10. ¿Cuándo conviene Direct Connect en vez de Site-to-Site VPN?
> Cuando se envía mucho volumen y se necesita rendimiento predecible. Direct Connect es una conexión privada dedicada que no pasa por la Internet pública, con menos cuellos de botella y menos variación de latencia. Se puede dejar la VPN como failover.

---

## 🧾 Resumen en 10 líneas

1. En AWS el cómputo se consume como servicio: sin compra adelantada, con pago por segundo u hora y capacidad en minutos.
2. Hay tres categorías: VM (EC2), contenedores (ECS/EKS) y serverless (Fargate, Lambda), en un espectro que va del control a la conveniencia.
3. Una instancia EC2 nace de una AMI (SO, almacenamiento, arquitectura y software; ID regional) y su instance type (familia + generación + tamaño) fija el hardware.
4. Ciclo de vida: pending → running → stop / stop-hibernate / reboot / terminate. El stop cambia la IP pública y terminate borra todo.
5. Precios: On-Demand flexible, RI de 1 o 3 años con descuento y compromiso, Spot hasta 90 % más barato pero interrumpible con 2 min de aviso.
6. Los contenedores empaquetan código y dependencias, arrancan rápido y son portables. ECS (task, container instance) es nativo de AWS y EKS (pod, worker node) usa Kubernetes.
7. Lambda corre código ante triggers, dura como máximo 15 min, cobra por invocación y por ms y no cobra el tiempo ocioso. Fargate es serverless para contenedores.
8. Una VPC es una red aislada por región con CIDR (de /16 a /28). Las subnets viven en una AZ, AWS reserva 5 IP en cada una y conviene tener al menos 2 AZ.
9. El IGW conecta con Internet y el VGW con redes privadas vía VPN. La route table es la que hace pública a una subnet (`0.0.0.0/0 → IGW`).
10. La NACL (subnet, stateless, allow/deny) y el security group (instancia, stateful, solo allow) forman dos capas de firewall. Direct Connect ofrece un enlace dedicado fuera de Internet.

---

## 📖 Recursos del módulo

**Lectura 2.1, Computación como servicio**
- [AWS: Compute Services Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/compute-services.html)
- [AWS: Computación en AWS](https://aws.amazon.com/products/compute/)
- [AWS: Blog de informática de AWS](https://aws.amazon.com/blogs/compute/)

**Lectura 2.2, Introducción a Amazon EC2**
- [AWS: Amazon EC2](https://aws.amazon.com/ec2/)
- [AWS: Imágenes de máquina de Amazon (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [AWS: Creación de una AMI Linux respaldada por Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html)
- [AWS: ¿Qué es EC2 Image Builder?](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html)

**Lectura 2.25, Ciclo de vida de las instancias EC2**
- [AWS: Amazon EC2](https://aws.amazon.com/ec2/)
- [AWS: VPC por defecto y subredes por defecto](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html)
- [AWS: Pilar de fiabilidad de AWS](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
- [AWS: Ciclo de vida de las instancias](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
- [AWS: Precios de Amazon EC2](https://aws.amazon.com/ec2/pricing/)
- [AWS: Precios de Amazon EC2 On-Demand](https://aws.amazon.com/ec2/pricing/on-demand/)
- [AWS: Precios de Amazon EC2 Spot Instances](https://aws.amazon.com/ec2/spot/pricing/)
- [AWS: Precios de las instancias reservadas de Amazon EC2](https://aws.amazon.com/ec2/pricing/reserved-instances/pricing/)

**Lectura 2.3, Servicios de contenedores**
- [AWS: Contenedores en AWS](https://aws.amazon.com/containers/services/)
- [Docker: ¿Qué es un contenedor?](https://www.docker.com/resources/what-container)
- [AWS: Amazon Elastic Container Service](https://aws.amazon.com/ecs/)
- [GitHub: Agente Amazon ECS](https://github.com/aws/amazon-ecs-agent)
- [AWS: Instancias de contenedor de Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_instances.html)
- [Coursera: Creación de aplicaciones en contenedores en AWS](https://www.coursera.org/learn/containerized-apps-on-aws)

**Lectura 2.4, Serverless y AWS Lambda**
- [AWS News Blog](https://aws.amazon.com/blogs/)
- [Serverless Image Handler](https://aws.amazon.com/answers/web-applications/serverless-image-handler/)
- [AWS News Blog (tutorial y código de la demo Lambda)](https://aws.amazon.com/blogs/compute/resize-images-on-the-fly-with-amazon-s3-aws-lambda-and-amazon-api-gateway/)
- [AWS: Serverless](https://aws.amazon.com/serverless/#:~:text=Serverless%20is%20the%20native%20architecture,services%20without%20thinking%20about%20servers).
- [Coursera: Creación de aplicaciones modernas de Python en AWS](https://www.coursera.org/learn/building-modern-python-applications-on-aws)
- [AWS: Recursos de AWS Serverless](https://aws.amazon.com/serverless/resources/?serverless.sort-by=item.additionalFields.createdDate&serverless.sort-order=desc)
- [AWS: Creación de aplicaciones con arquitecturas sin servidor](https://aws.amazon.com/lambda/serverless-architectures-learn-more/)
- [AWS: Prácticas recomendadas para organizar aplicaciones sin servidor de mayor tamaño](https://aws.amazon.com/blogs/compute/best-practices-for-organizing-larger-serverless-applications/)
- [AWS: Administración de funciones de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-functions.html)
- [AWS: 10 Things Serverless Architects Should Know](https://aws.amazon.com/blogs/architecture/ten-things-serverless-architects-should-know/)
- [AWS: ¡El ataque alienígena de AWS! Una aventura sin servidor](https://alienattack.workshop.aws/)

**Lectura 2.5, Redes en AWS**
- [Stanford: Introducción a las redes informáticas](https://web.stanford.edu/class/cs101/network-1-introduction.html)
- [Ionos: CIDR, ¿qué es el enrutamiento entre dominios sin clases?](https://www.ionos.com/digitalguide/server/know-how/cidr-classless-inter-domain-routing/)

**Lectura 2.6, Introducción a Amazon VPC**
- [AWS: VPC con subredes públicas y privadas (NAT)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
- [AWS: Tablas de rutas personalizadas](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html#CustomRouteTables)
- [AWS: Puerta de enlace del cliente](https://docs.aws.amazon.com/vpn/latest/s2svpn/how_it_works.html#CustomerGateway)
- [AWS: ¿Qué es Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [AWS: VPC y subredes](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)

**Lectura 2.7, Enrutamiento y seguridad de Amazon VPC**
- [AWS: Tablas de rutas](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [AWS: Ejemplo de opciones de enrutamiento](https://docs.aws.amazon.com/vpc/latest/userguide/route-table-options.html)
- [AWS: Trabajar con tablas de enrutamiento](https://docs.aws.amazon.com/vpc/latest/userguide/WorkWithRouteTables.html)
- [AWS: ACLs de red](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [AWS: Grupos de seguridad para su VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [AWS: Alojo un sitio web en una instancia EC2. ¿Cómo permito que mis usuarios se conecten por HTTP (80) o HTTPS (443)?](https://aws.amazon.com/premiumsupport/knowledge-center/connect-http-https-ec2/)

> [!note] ➕ Nota del editor: discrepancias del material
> - En el video «Introducción a Amazon VPC», la Private Subnet 1 se anuncia como `10.1.3.0/4` (error de dictado por `/24`) y luego se crea como `10.1.3.0/24`. En la demo de relanzamiento, la Private Subnet 1 es `10.1.2.0/24` y `10.1.3.0/24` pasa a ser la Public Subnet 2. Este tomo sigue el esquema de la demo de relanzamiento.
> - La primera demo usa la AMI **Amazon Linux 2023**, pero en el relanzamiento la consola muestra «Linux 2 AMI» y el texto de la lección dice que la app se lanzó con Amazon Linux 2.
> - La lectura 2.7 atribuye el puerto 443 a «HTTP». Debe leerse **HTTPS**.
> - La lectura 2.25 describe stop-hibernate bajo el rótulo «detener». Se corrigió el sentido: en stop normal se pierde la RAM y en stop-hibernate se guarda en el volumen raíz EBS.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[01-Vision-General-y-Seguridad|01 · Visión General y Seguridad]] · Siguiente: [[03-Almacenamiento-y-Bases-de-Datos|03 · Almacenamiento y Bases de Datos ➡]]
