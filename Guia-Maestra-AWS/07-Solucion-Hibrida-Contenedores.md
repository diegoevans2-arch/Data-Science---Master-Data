---
title: "Tomo 07 — Solución híbrida para cargas de trabajo con contenedores"
tags: [aws, solutions-architect, hibrido, contenedores, direct-connect, ecs, rds, storage-gateway, systems-manager]
curso: 2 · Architecting Solutions on AWS
modulo: "Diseño de una solución híbrida para cargas de trabajo basadas en contenedores en AWS"
audiencias: [tecnico, puente, ejecutivo]
tomo: 07
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🏗️ Tomo 07 — Solución híbrida para cargas de trabajo con contenedores

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[06-Analitica-de-Datos-Serverless|06 · Analítica de datos serverless]] · Siguiente: [[08-Gobierno-y-Multicuenta|08 · Gobierno y multicuenta ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Casi ninguna empresa grande migra a la nube "de una vez": los contratos de centro de datos vencen en fechas distintas y durante años conviven dos mundos. Este tomo recorre, con un cliente concreto (una aseguradora), cómo diseñar esa convivencia: la red que une ambos lados, dónde corren los contenedores, cómo se migra la base de datos sin detener el negocio, cómo se comparten archivos sin reescribir aplicaciones y cómo se administra todo con las mismas herramientas. Es el tomo de las decisiones *híbridas*: Direct Connect vs. VPN, ECS vs. EKS, EC2 vs. Fargate, NAT gateway vs. NAT instance, EFS vs. Storage Gateway.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** qué enlace contratar entre el centro de datos y AWS; qué orquestador de contenedores adoptar sin sobrecargar al equipo; cómo migrar la base de datos con casi cero tiempo de inactividad; cómo bajar el costo de almacenamiento de archivos que envejecen.
> - **Costo de hacerlo mal:** latencia impredecible por usar Internet para tráfico masivo; un equipo atascado en la curva de aprendizaje de Kubernetes; una base de datos de instancia única que se cae sin failover; reescribir aplicaciones on-premises que se iban a jubilar de todos modos.
> - **Pregunta que responde:** *¿cómo muevo la mitad de mi carga a AWS hoy, sin reescribir código, sin perder disponibilidad y sin duplicar herramientas de operación, dejando el camino listo para mover la otra mitad después?*

> [!tip] 💡 Analogía general: la mudanza en dos etapas
> La aseguradora es una familia que se muda a una casa nueva, pero la mitad de los muebles debe quedarse en la casa vieja hasta que venza el arriendo. Necesita un **camino privado y rápido** entre ambas casas (Direct Connect), **el mismo mayordomo** que administre las dos (ECS Anywhere + Systems Manager), un **buzón en la casa vieja** que despache automáticamente las cajas al nuevo depósito (Storage Gateway → S3) y un **camión de mudanza que no interrumpe la vida diaria** mientras traslada el archivo familiar (DMS → RDS).

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Cliente nº 3 — Compañía de seguros Enterprise | 3 videos | Caso AnyCompany Insurance, requisitos y su desglose |
| Arquitectura de la solución | 7 videos (3 demos de consola) / 5 lecturas | Conectividad híbrida, contenedores y NAT, RDS + DMS, almacenamiento (S3, EFS, FSx, EBS, Storage Gateway), servicios híbridos (Outposts, ECS/EKS Anywhere, Backup, Systems Manager) |
| Presentación de la solución al cliente y próximos pasos | 2 videos / 1 lectura | Revisión de la arquitectura con el cliente y optimizaciones: VPN de respaldo, DR multirregión, escalado de contenedores, RDS y S3 |

> [!note] ➕ Nota del editor
> En la extracción de este módulo no aparecen ítems de laboratorio ni de quiz: el aprendizaje práctico son tres recorridos de consola (RDS, Storage Gateway, Systems Manager), descritos en la sección 8.

---

## 1. El caso: AnyCompany Insurance

Audiencia: 🔧 🧭 👔

Morgan (arquitecta de AWS) conversa con Raf, que esta semana interpreta al cliente: **AnyCompany Insurance**, una aseguradora empresarial que lleva tiempo migrando a AWS a medida que vencen sus contratos de centro de datos. Morgan advierte desde el inicio que el caso **simplifica deliberadamente** algunas partes para enfocarse en ciertos servicios; en la vida real los requisitos cambian durante el proyecto.

### 1.1 Situación actual

- Cargas de trabajo en **contenedores** on-premises para todas las aplicaciones, salvo las **bases de datos** (PostgreSQL, no contenedorizadas).
- La mitad de los servidores está en ubicaciones con contratos que vencen → se migra a AWS ahora. La otra mitad se queda **algunos años** on-premises → despliegue **híbrido**.
- Las aplicaciones son **internas**: no debe haber acceso desde Internet a los contenedores migrados.
- El orquestador actual es "muy viejo y con fallas"; quieren reemplazarlo.
- La parte on-premises genera **archivos** que consumen las aplicaciones en la nube (analítica y machine learning).

### 1.2 Requisitos extraídos

| # | Requisito | Origen en la conversación |
|---|---|---|
| R1 | Conectividad **dedicada**, baja latencia, rendimiento consistente y alto volumen entre centro de datos y AWS | "No queremos ralentizaciones por dividir la carga" |
| R2 | Seguir usando **contenedores** en AWS | Portabilidad de contenedores |
| R3 | Contenedores **privados** (sin acceso entrante desde Internet) | Aplicaciones internas |
| R4 | **Mismas herramientas** de gestión y orquestación en ambos entornos, cuando sea posible | No quieren dos orquestadores |
| R5 | Base de datos compatible con **PostgreSQL** en AWS | Migran algunas BD junto con los contenedores |
| R6 | **Lift-and-shift**: no reescribir código | Lo repiten varias veces |
| R7 | Almacenar en AWS los archivos generados on-premises, accesibles para los contenedores en la nube | Todo se moverá eventualmente |
| R8 | **Alta disponibilidad**, resiliencia, tolerancia a fallos; vigilar puntos únicos de fallo | "Lo más importante es el uptime" |

> [!warning] ⚠️ Regla del arquitecto en este caso
> Morgan se impone dos disciplinas que recorren todo el módulo: (1) **preguntarse en cada componente "¿qué pasa si esto se rompe?"** porque la resiliencia es prioridad del cliente, y (2) **dejar intacto el código de aplicación** tanto como se pueda (lift-and-shift puro), aceptando que algunas piezas quizás no mapeen sin retrabajo.

### 1.3 Llamadas de seguimiento (requisitos que aparecen después)

Durante el diseño, Morgan llama dos veces más al cliente. Lo que se aprende:

| Llamada | Pregunta | Respuesta del cliente | Consecuencia |
|---|---|---|---|
| Orquestación | ¿Qué usan hoy? ¿Consideraron Kubernetes? | Lo evaluaron, pero la curva de aprendizaje los abrumó; su operación es simple, solo tienen *muchos* contenedores | **Amazon ECS** |
| Cómputo | ¿Necesitan SSH al clúster? | Sí: quieren su **propia imagen** (AMI) y SSH cuando quieran; "un cambio a la vez" | **Tipo de lanzamiento EC2** (no Fargate) |
| Almacenamiento | ¿Cómo se usan los archivos? | Se escriben una vez, no se modifican, se leen para analítica/ML; se procesan a la semana, se consultan ocasionalmente y **después de un año ya no se miran** | Necesidad de **políticas de ciclo de vida** |
| Almacenamiento | ¿Qué protocolo usan? | Sistema de archivos Linux con **NFS**; quieren seguir con NFS (algún día reescribirán a almacenamiento de objetos) | Interfaz NFS obligatoria |
| Almacenamiento | ¿Latencia? | Las **escrituras** on-premises deben ser lo más rápidas posible, sin esperar el viaje a AWS | **Storage Gateway** sobre EFS |
| Revisión final | ¿Registro de contenedores? | Hoy usan **Docker Hub** | Puede seguir con Docker Hub o pasar a **Amazon ECR** |

---

## 2. Red base en AWS y conectividad híbrida

Audiencia: 🔧 🧭

### 2.1 Punto de partida en AWS

Para simplificar, el diseño parte con **una VPC, una cuenta y una Región**. Como las aplicaciones son internas, Morgan propone **más subnets privadas, y más grandes, que subnets públicas**.

### 2.2 Las tres opciones de conectividad

| Opción | Cómo funciona | Seguridad | Rendimiento | ¿Para este caso? |
|---|---|---|---|---|
| **Internet pública** (sin cifrar) | Tráfico abierto | Cualquiera podría "husmear" | Ancho de banda compartido con todos; varía según el uso en la zona | ❌ Descartada |
| **AWS VPN** (Site-to-Site) | Túnel IPsec privado **sobre Internet** | ✅ Cifrado | Sigue dependiendo de la disponibilidad de ancho de banda en Internet | ❌ No garantiza baja latencia (✅ sí como respaldo, ver §7) |
| **AWS Direct Connect** | Conexión **privada y dedicada**; el tráfico se queda en la red global de AWS y **nunca toca la Internet pública** | ✅ | Menos cuellos de botella y menos saltos inesperados de latencia | ✅ **Elegida** |

**Decisión y justificación:** el cliente enviará un **gran volumen** de tráfico y exige **rendimiento consistente** → Direct Connect.

> [!tip] 💡 Analogía
> La Internet pública es la carretera principal en hora punta; la VPN es la misma carretera pero en un vehículo blindado (seguro, pero igual atrapado en el taco); Direct Connect es una vía exclusiva que solo usa usted.

### 2.3 AWS VPN: dos servicios distintos

| Servicio | Para qué | Detalles del material |
|---|---|---|
| **AWS Site-to-Site VPN** | Conectar una **red remota** (centro de datos) a una VPC o a un **AWS Transit Gateway** | Soporta conexiones VPN **IPsec**. Por defecto, las instancias de una VPC **no** pueden hablar con su red remota: hay que crear la conexión y configurar el enrutamiento |
| **AWS Client VPN** | Conectar **personas** (administradores, trabajo remoto desde un portátil) a AWS o a la red local | Totalmente administrado y **elástico** (escala solo); compatible con **OpenVPN**; los usuarios acceden igual antes, durante y después de la migración |

**Componentes de Site-to-Site VPN:**

- **Virtual private gateway**: el concentrador VPN del **lado de Amazon**; se crea y se adjunta a la VPC. Incluye redundancia y failover automáticos del lado AWS, y admite **varias customer gateways** para que usted también tenga redundancia en su lado.
- **Customer gateway device**: dispositivo físico o de software que **usted** posee y configura en su red local.

**Escenarios de Client VPN** que muestra la lectura:

1. Acceso a **una sola VPC** de destino.
2. Acceso **solo a una red local**.
3. Acceso a una VPC **y** comunicación entre clientes: cada cliente recibe una IP única del **rango CIDR de clientes** del endpoint y pueden hablarse entre sí.

### 2.4 AWS Direct Connect en detalle

- Conexión **dedicada** (provista por AWS) u **hosted** (provista por un *Direct Connect delivery partner*).
- Disponible en **más de 100 ubicaciones** de Direct Connect en el mundo.
- **Direct Connect SiteLink**: envía datos *entre* ubicaciones de Direct Connect, creando redes privadas entre oficinas y centros de datos de su red global.
- Cada conexión dedicada es **un único enlace** entre los puertos de su router y un dispositivo de Direct Connect → **AWS recomienda una segunda conexión** por redundancia. Varios puertos en la misma ubicación se aprovisionan en equipos AWS redundantes.

**Comportamiento ante fallos** (según la lectura):

| Configuración de respaldo | Si falla Direct Connect… |
|---|---|
| VPN IPsec de respaldo configurada | Todo el tráfico de la VPC **conmuta automáticamente** a la VPN; el tráfico hacia recursos públicos (p. ej. S3) va por Internet |
| Sin Direct Connect de respaldo ni VPN | El tráfico de la VPC **se interrumpe**; el tráfico a recursos públicos va por Internet |

### 2.5 AWS Transit Gateway

Si el cliente tuviera **varias cuentas o Regiones** que conectar con Direct Connect, convendría añadir **Transit Gateway** (en este caso se mantiene simple y no se usa).

- Conecta VPC y redes locales a través de un **hub central**; elimina la malla de *peerings*. Funciona como un **router en la nube**: cada conexión nueva se hace **una sola vez**.
- Se pueden aplicar **tablas de rutas** en el Transit Gateway para conectar redes de forma centralizada.
- **Peering entre Regiones** de Transit Gateways sobre la red global de AWS: datos cifrados automáticamente, sin pasar por Internet.

```
  SIN Transit Gateway (malla)          CON Transit Gateway (hub)
  VPC1 ── VPC2                          VPC1   VPC2   VPC3
   │  ╲  ╱  │                              ╲    │    ╱
   │   ╳    │                               [ TGW ]──── DX ── On-prem
   │  ╱  ╲  │                              ╱    │
  VPC3 ── On-prem                       VPC4   VPN
```

> [!example] 📊 Patrón de la lectura
> Una red remota conectada por Direct Connect a un Transit Gateway en cada una de **tres Regiones**, cada Región con varias VPC.

🎯 **Para el examen:**
- Baja latencia + rendimiento consistente + alto volumen → **Direct Connect**. Cifrado rápido de configurar sobre Internet → **Site-to-Site VPN**. Usuarios individuales → **Client VPN**.
- Direct Connect de una sola conexión es un **punto único de fallo**: segunda conexión DX o VPN de respaldo.
- Muchas VPC/cuentas/Regiones → **Transit Gateway**.

---

## 3. Contenedores en AWS: orquestador y cómputo

Audiencia: 🔧 🧭

### 3.1 ¿Por qué un orquestador?

Lanzar, eliminar y escalar contenedores es manejable con pocos contenedores en pocas instancias; con **cientos o miles** (el caso del cliente) se necesita una herramienta de **orquestación** que haga el trabajo pesado. AWS ofrece dos, y ambas colocan contenedores sobre una plataforma de cómputo: clúster de **EC2** o **AWS Fargate** (serverless).

### 3.2 El espectro conveniencia ↔ control

```
  CONVENIENCIA ◄──────────────────────────────► CONTROL
  Orquestador:  Amazon ECS ............ Amazon EKS (Kubernetes)
  Cómputo:      AWS Fargate ........... Amazon EC2
```

| Decisión | Opción conveniente | Opción de control |
|---|---|---|
| **Orquestador** | **Amazon ECS**: diseñado para la simplicidad, "opinado", menos decisiones de red y seguridad; integración **nativa** con otros servicios (p. ej. varios tipos de load balancer) | **Amazon EKS**: comunidad Kubernetes vibrante, APIs open source consistentes, gran flexibilidad; curva de aprendizaje considerable. Lo eligen quienes ya usan Kubernetes on-premises o quieren máxima flexibilidad |
| **Cómputo (launch type)** | **AWS Fargate**: serverless, sin aprovisionar ni administrar infraestructura; **no** permite AMI propia ni SSH | **Amazon EC2**: AMI propia, usted gestiona parches y escalado, acceso **SSH** |

**Decisión:** **ECS** (operación simple con muchos contenedores; Kubernetes les resultó abrumador) + **tipo de lanzamiento EC2** (AMI personalizada y SSH; Fargate no admite ninguna de las dos).

- En el tipo EC2, el **ECS agent** se instala en cada instancia del clúster; es lo que permite a ECS gestionar los nodos.
- En el diagrama: **ECS** se ubica en la Región (fuera de la VPC); las instancias **EC2** en **subnets privadas** de la VPC, sin conectividad a Internet de entrada ni de salida.

### 3.3 El problema de la salida a Internet: NAT

Instancias privadas pueden necesitar **salir** a Internet; por ejemplo, un script de **user data** que en el arranque descarga e instala paquetes. Sin ruta a Internet, esas peticiones fallan. Solución: un recurso **NAT** (Network Address Translation) en una **subnet pública**:

1. La instancia privada envía la petición; la tabla de rutas la dirige al NAT.
2. El NAT reemplaza la IPv4 de origen por la suya y hace la llamada a Internet.
3. Recibe la respuesta, retraduce la dirección y la entrega a la instancia original.
4. Nadie desde fuera puede **iniciar** una conexión hacia la instancia privada.

| Criterio | **NAT instance** (software NAT en EC2, AMI propia) | **NAT gateway** (servicio administrado) |
|---|---|---|
| Rendimiento | Limitado por tipo y tamaño de la instancia | Escala masivamente hacia arriba y abajo |
| Costo | A menudo **más barata** | Mayor |
| Redundancia | **No** es redundante por sí misma: varias por AZ y failover gestionado por usted; o un Auto Scaling group *steady state* con mín = máx = deseado = 1 | Redundante **dentro de una AZ**; recomendación: **una por AZ** |
| Recomendación AWS | Cargas pequeñas y sensibles a costo | ✅ Mejor disponibilidad y ancho de banda, menos administración |

**Decisión:** **NAT gateway**, una por AZ: cliente empresarial, gran carga, sin puntos únicos de fallo.

**Tipos de conectividad del NAT gateway:**

| Tipo | Uso | Elastic IP | Ruta |
|---|---|---|---|
| **Público** (por defecto) | Salir a Internet desde subnets privadas | **Obligatoria** al crearla | NAT gateway → **internet gateway**; también puede ir a otras VPC/on-premises vía transit gateway o virtual private gateway |
| **Privado** | Conectar a otras VPC o a la red local | **No** se puede asociar | Vía transit gateway o virtual private gateway; si se enruta a un internet gateway, este **descarta** el tráfico |

> [!warning] ⚠️ Regla general de la VPC
> Para casi todo lo que vive dentro de una VPC: usar **al menos dos AZ** y repartir la carga entre ellas, para que la solución sea independiente de una AZ.

🎯 **Para el examen:** AMI propia o SSH → EC2, no Fargate. Equipo sin experiencia en Kubernetes y operación simple → ECS. Salida a Internet desde subnet privada → NAT gateway en subnet pública, una por AZ.

---

## 4. Base de datos: Amazon RDS y AWS DMS

Audiencia: 🔧 🧭

### 4.1 Amazon RDS para PostgreSQL

En lugar de comparar motores, Morgan parte del mejor ajuste: **Amazon RDS** con motor **PostgreSQL**. Se crea la instancia, se conecta, se carga el esquema y se migran los datos.

> [!danger] 🚨 Una instancia única no se arregla sola
> Con la configuración por defecto (una sola instancia de BD), si la instancia falla **no hay mecanismo mágico** que la reemplace. La alta disponibilidad hay que **configurarla**.

### 4.2 Multi-AZ

- **Dos instancias** en AZ distintas: una **primaria** y una **standby** (secundaria); RDS **replica de forma síncrona**.
- Las aplicaciones se conectan por un **nombre DNS** que apunta a la primaria. Si esta falla, el **failover ocurre vía DNS**: el endpoint no cambia y la standby pasa a ser la nueva primaria. Sincronización y failover los gestiona AWS, sin intervención manual.
- Después del failover, usted investiga por qué falló la primaria.

**Variante con dos standby legibles en tres AZ** (MySQL o PostgreSQL): failovers automáticos típicamente en **menos de 35 segundos**, latencia de *commit* de transacciones **hasta 2 veces más rápida** que Multi-AZ con un standby, capacidad de lectura adicional y opción de cómputo **Graviton2** o Intel.

### 4.3 Read replicas

Caso hipotético: informes de **BI** con SQL complejo consumen mucha CPU y afectan a las aplicaciones. Una **read replica** se crea desde la instancia fuente, se mantiene sincronizada y recibe las lecturas pesadas, liberando a la primaria **sin escalarla**.

- Disponibles en RDS para **MySQL, MariaDB, PostgreSQL, Oracle, SQL Server** y en **Aurora**.
- Para esos motores, RDS crea la réplica desde un **snapshot** de la fuente y luego usa la **replicación asíncrona nativa** del motor.
- Solo admite conexiones de **lectura**; replica **todas** las bases de la instancia; puede **promoverse** a instancia independiente.
- Algunos motores admiten réplicas **entre Regiones** (PostgreSQL incluido, ver §7).

| | Multi-AZ | Read replica |
|---|---|---|
| Propósito | Disponibilidad / failover | Rendimiento de lectura |
| Replicación | **Síncrona** | **Asíncrona** |
| ¿Sirve tráfico? | Standby clásico no (la variante de 2 standby sí es legible) | Sí, solo lectura |

### 4.4 Cómo escalar una instancia RDS

| Necesidad | Palanca |
|---|---|
| Más CPU **y** almacenamiento | **Escalado vertical**: tipo/tamaño de instancia mayor (no todos los tipos existen para todo motor, versión, edición o Región) |
| Más CPU, no más almacenamiento | **Read replicas** |
| Más almacenamiento, no más CPU | Añadir volúmenes manualmente o **RDS Storage Auto Scaling**: usted fija un **máximo**, RDS monitorea y crece cuando el uso se acerca a lo aprovisionado, con tiempo de inactividad prácticamente nulo; funciona en instancias nuevas y existentes |
| Más rendimiento de E/S | Cambiar tipo de almacenamiento (p. ej. Provisioned IOPS en vez de General Purpose) |

**Tipos de almacenamiento RDS** (respaldados por volúmenes EBS):

| Tipo | Rasgos del material |
|---|---|
| **General Purpose SSD** (gp2) | Costo intermedio, bueno para la mayoría; latencias del orden de un milisegundo (así lo dice la traducción; probablemente «single-digit millisecond» en el original) y capacidad de sostener **3.000 IOPS** por largos periodos; rendimiento base según el **tamaño** del volumen |
| **Provisioned IOPS** (io1) | Para cargas de E/S intensivas (bases de datos) que piden baja latencia y E/S consistente |
| **Magnetic** | El más lento y barato; solo por **compatibilidad hacia atrás**; máximo de almacenamiento menor. AWS recomienda no usarlo para nuevas necesidades |

> [!tip] 💡 Dato útil
> Más almacenamiento asignado puede significar **más IOPS**: el tamaño del volumen y el rendimiento están correlacionados.

### 4.5 AWS Database Migration Service (DMS)

**Qué es:** un servidor en la nube que ejecuta software de replicación. Usted define una **conexión de origen** (la BD on-premises) y una de **destino** (RDS) y programa una **tarea** de migración.

- La fuente sigue **totalmente operativa** durante la migración; los cambios se **replican continuamente** al destino → casi **cero tiempo de inactividad**.
- Al terminar, el destino puede seguir sincronizado el tiempo que usted quiera; el **cutover** se hace cuando convenga.
- DMS crea tablas y claves primarias si no existen (o usted las crea antes).

| Tipo de migración | Ejemplo | Herramientas |
|---|---|---|
| **Homogénea** (mismo motor o compatible) — **el caso del cliente** | PostgreSQL → RDS PostgreSQL; Oracle → RDS Oracle; MySQL → Aurora | Solo DMS; suele ser un **proceso de un paso** |
| **Heterogénea** (motores distintos) | Oracle → Aurora PostgreSQL (ejemplo genérico) | **AWS Schema Conversion Tool (SCT)** convierte esquema y código (tablas, índices, vistas, triggers…) y luego DMS migra los datos |

**Otros casos de uso de DMS** (página de producto que Morgan recorre en aws.amazon.com/dms): sincronizar **dev/test con producción**; **consolidar** varias bases en una (p. ej. en Aurora); **replicación continua** para **recuperación ante desastres**. Origen y destino pueden estar on-premises, en EC2 o en RDS.

```
  On-prem PostgreSQL ──(DX)──► [ DMS replication instance ] ──► RDS PostgreSQL (Multi-AZ)
        (sigue en uso)             full load + CDC continuo           (cutover cuando convenga)
```

> [!note] ➕ Nota del editor
> "CDC" (*change data capture*) es el nombre habitual de la replicación continua de cambios que describe el material; el curso no usa la sigla.

---

## 5. Almacenamiento de los archivos compartidos

Audiencia: 🔧 🧭

### 5.1 Repaso de servicios (aws.amazon.com/products/storage)

Morgan recomienda volver a la documentación de vez en cuando: los lanzamientos nuevos pueden cambiar el enfoque de arquitectura.

| Servicio | Tipo | Rasgos clave del material |
|---|---|---|
| **Amazon S3** | Objetos (API REST) | Serverless, escala solo; objetos de hasta **5 TB**, buckets sin límite de tamaño; ideal para datos que se **escriben poco y se leen mucho** (cambiar un objeto implica reescribirlo completo); versioning; clases de almacenamiento |
| **Amazon EFS** | Archivos (**NFS**, v4.0 y v4.1) | Serverless, crece y se reduce solo hasta **petabytes**; acceso concurrente desde EC2, ECS y Lambda; usable también desde on-premises; políticas de ciclo de vida |
| **Amazon FSx** | Archivos administrados | Para **NetApp ONTAP, OpenZFS, Windows File Server y Lustre**: útil si el cliente ya usa uno de ellos |
| **Amazon EBS** | Bloques | "Disco duro virtual" adjunto a EC2; ideal para BD en EC2; **no** escala solo; debe estar adjunto a una instancia; se escribe solo el bloque que cambió |

**Volúmenes EBS** (lectura): **SSD** (General Purpose y Provisioned IOPS) para cargas transaccionales donde dominan las **IOPS**; **HDD** (Throughput Optimized y Cold) para streaming grande donde domina el **throughput**; **previous generation** para datos pequeños de acceso infrecuente. En volúmenes de generación actual se puede aumentar tamaño, cambiar IOPS y tipo **en caliente**. Las IOPS crecen con el tamaño.

**Clases de S3** (lectura): S3 Standard (acceso frecuente), S3 Standard-IA y S3 One Zone-IA (infrecuente), S3 Glacier Instant Retrieval, Glacier Flexible Retrieval y Glacier Deep Archive (archivo), y **S3 Intelligent-Tiering** (patrones cambiantes o desconocidos; mueve datos entre **cuatro niveles**: dos de baja latencia —frecuente e infrecuente— y dos de archivo opcionales para acceso asíncrono).

### 5.2 Decisión: EFS vs. Storage Gateway

| Criterio | Amazon EFS | **AWS Storage Gateway (S3 File Gateway) + S3** |
|---|---|---|
| NFS | ✅ | ✅ (también SMB) |
| Acceso desde on-premises | ✅ vía Direct Connect | ✅ dispositivo local |
| Ciclo de vida a clases baratas | ✅ | ✅ (S3 Lifecycle) |
| **Latencia de escritura on-premises** | ⚠️ Cada escritura viaja a AWS y vuelve por Direct Connect | ✅ Se escribe en el dispositivo **local**; la subida a S3 es **asíncrona** |
| Futuro nativo en objetos | Requeriría otra migración | ✅ Los archivos **ya están en S3** |

**Decisión:** **S3 File Gateway** (en el diagrama aparece dos veces: en el centro de datos y del lado AWS) + **Amazon S3**. Justificación: NFS sin reescribir apps, escrituras locales de baja latencia, caché local para lecturas rápidas, ciclo de vida de S3 para el patrón "se procesa a la semana, se olvida al año", y cero migración secundaria cuando reescriban hacia almacenamiento de objetos.

### 5.3 AWS Storage Gateway

Conecta un **dispositivo de software on-premises** con almacenamiento en la nube. Tres familias:

| Gateway | Protocolo | Qué hace |
|---|---|---|
| **Amazon S3 File Gateway** | NFS / SMB | Archivos ↔ objetos en S3, con caché local |
| **Amazon FSx File Gateway** | SMB | Acceso a FSx for Windows File Server |
| **Tape Gateway** | iSCSI-VTL | Cintas virtuales en S3; archivo en **S3 Glacier Flexible Retrieval** o **Glacier Deep Archive** (útil para respaldos en cinta) |
| **Volume Gateway** | iSCSI | Volúmenes de bloques respaldados en S3 |

**S3 File Gateway en detalle:**

- Combina un servicio y un **appliance virtual** desplegado como VM en **VMware ESXi, Microsoft Hyper-V o Linux KVM** (la consola también ofrece EC2 y appliance de hardware).
- Protocolos: **NFS v3 y v4.1**, **SMB v2 y v3**.
- Los datos quedan accesibles directamente en S3 para cualquier servicio de AWS (aquí, los contenedores de analítica/ML).
- Gestión con **lifecycle policies**, **Cross-Region Replication (CRR)** y **versioning** de S3. Puede pensarse como un sistema de archivos montado sobre S3.
- Caché local transparente, transferencia optimizada y en paralelo, protección ante congestión de red y gestión del ancho de banda.

> [!tip] 💡 Analogía
> El File Gateway es el buzón del edificio viejo: usted deja la carta (escritura NFS) y se va de inmediato; el cartero (la subida asíncrona) la lleva al depósito central (S3) cuando corresponde, y el buzón guarda copia de lo reciente por si la vuelve a necesitar.

---

## 6. Servicios para despliegues híbridos

Audiencia: 🔧 🧭

Morgan recorre aws.amazon.com/hybrid y recomienda familiarizarse con migraciones y servicios híbridos *antes* de necesitarlos: "si no sabe lo que existe, no puede decidir si encaja".

| Categoría | Servicio | Qué aporta | ¿En la solución? |
|---|---|---|---|
| Cómputo | **AWS Outposts** | Hardware real (racks o servidores Outposts) entregado en su centro de datos para correr servicios AWS on-premises: latencia ultrabaja o regulación que exige ubicación específica de los datos | No |
| Contenedores | **Amazon ECS Anywhere** / **Amazon EKS Anywhere** | Mismo orquestador y mismas herramientas para contenedores on-premises y en la nube; reduce costos de soporte y esfuerzo redundante | ✅ **ECS Anywhere** |
| Almacenamiento | **AWS Storage Gateway** | Ver §5 | ✅ |
| Almacenamiento | **AWS Backup** | Protección de datos y cumplimiento centralizados en AWS e híbrido | ✅ (sugerido para que el cliente evalúe) |
| Redes | **AWS Direct Connect**, **Amazon Route 53 Resolver** | Conectividad y resolución DNS híbrida | ✅ Direct Connect |
| Gestión | **AWS Directory Service** (Microsoft Active Directory), **IAM**, **CloudWatch**, **X-Ray** | Identidad, monitoreo y observabilidad entre entornos | — |
| Gestión | **AWS Systems Manager** | Operar recursos a escala en AWS y on-premises | ✅ |

### 6.1 Amazon ECS Anywhere

Ejecuta y administra contenedores en **infraestructura del cliente** con la misma gestión de clústeres, scheduling y monitoreo que ECS. Requiere **redefinir y redesplegar** los despliegues on-premises (crear **task definitions**), pero los contenedores y su código probablemente no cambian.

**Pasos:**

1. Crear una **activation key** para registrar las VM on-premises.
2. Instalar con esa clave el **SSM Agent** y el **ECS agent** en esos servidores.
3. Definir y desplegar la aplicación igual que en ECS en AWS; ECS gestiona los contenedores on-premises.

### 6.2 AWS Systems Manager

Solución de gestión segura de extremo a extremo para nube híbrida: **visibilidad y control** con interfaz unificada, agrupación de recursos (EC2, buckets S3, instancias RDS) por aplicación y automatización de tareas operativas.

- Todo nodo administrado necesita el **SSM Agent** en ejecución. ECS Anywhere usa ese mismo agente para gestionar las VM locales.
- **Run Command**: ejecutar un script en una flota (agrupada por **tags** u otros metadatos) sin entrar una por una: evita trabajo lento y errores humanos.

**Cómo actúa Systems Manager** (diagrama de la lectura):

1. **Acceso**: consola o AWS CLI.
2. **Elegir una capacidad** (Run Command, Patch Manager, etc.).
3. **Verificación y procesamiento**: comprueba permisos IAM; en nodos administrados actúa el SSM Agent; en otros recursos, Systems Manager o el servicio correspondiente.
4. **Informes** de estado (puede enviarlos a otros servicios).
5. **Gestión de operaciones** (Explorer, OpsCenter, Incident Manager): agregan datos y crean **OpsItems** e incidentes con remediación automatizada.

Morgan sugiere conocer a alto nivel: **Run Command, Patch Manager, Maintenance Windows, Session Manager**, y leer las **FAQ** cuando la documentación abrume.

### 6.3 AWS Backup

Servicio administrado, **basado en políticas**, para centralizar y automatizar la protección de datos en servicios de AWS y cargas híbridas. Con **AWS Organizations** permite aplicar políticas de respaldo en todas las cuentas (enlace con [[08-Gobierno-y-Multicuenta]]).

**Recursos soportados** (lista del material): EC2; aplicaciones Windows compatibles con **VSS** (Windows Server, SQL Server, Exchange) en EC2; volúmenes EBS; buckets S3; RDS incluido Aurora; DynamoDB; Neptune; DocumentDB; EFS; FSx for NetApp ONTAP, Lustre, Windows File Server y OpenZFS; volúmenes de Storage Gateway; cargas **VMware** on-premises, en Outposts y en VMware Cloud on AWS.

---

## 7. Optimizaciones: llevar la arquitectura al siguiente nivel

Audiencia: 🔧 🧭 👔

En el cierre, Morgan y Raf (ahora ambos como arquitectos) revisan la solución buscando mejoras:

| # | Mejora | Problema que resuelve | Detalle |
|---|---|---|---|
| O1 | **Site-to-Site VPN como respaldo de Direct Connect** | El router del centro de datos o una mala configuración son puntos de fallo | Direct Connect ofrece dos endpoints en AZ distintas del lado AWS, pero el router on-premises es uno solo; la VPN sobre Internet sirve de failover (patrón documentado con Transit Gateway) |
| O2 | **RDS Storage Auto Scaling** | RDS es administrado, pero la instancia **no** escala sola como un servicio serverless | El almacenamiento crece al consumirse; útil con carga impredecible; se habilita por consola, API o CLI |
| O3 | **Región secundaria para DR** | Cliente empresarial | Estrategias de la tabla siguiente; **requiere Infrastructure as Code** para replicar exacto |
| O4 | **S3 Lifecycle + Cross-Region Replication** | Datos ya presentes en la Región de DR; costo de tener varias copias | CRR replica aunque el contenido esté cifrado; Lifecycle baja costo al duplicar |
| O5 | **DMS para replicación continua** a otra Región | Copia de datos si no se usa Aurora multi-Región | — |
| O6 | **Escalado propio de ECS** | EC2 Auto Scaling escala instancias, no contenedores | Ver abajo |
| O7 | **Read replicas entre Regiones** de RDS PostgreSQL | DR de base de datos | Cualquier réplica puede **promoverse** a primaria |
| O8 | **S3 Intelligent-Tiering** | Optimizar costo con patrones de acceso desconocidos | Pequeño cargo mensual de monitoreo por objeto; mueve objetos no accedidos a niveles más baratos; sirve como clase por defecto (data lakes, analítica, apps nuevas) |

### 7.1 Estrategias de recuperación ante desastres

```
  menor costo / complejidad ─────────────────────────────► menor RTO / RPO
  Backup & Restore  →  Pilot Light  →  Warm Standby  →  Multi-site active/active
  └──────────── activo/pasivo ─────────────────┘           (activo/activo)
```

| Estrategia | Qué está encendido en la Región de DR | Notas del material |
|---|---|---|
| **Backup and restore** | Solo copias de datos | Mitiga pérdida o corrupción de datos, desastre regional (replicando copias) o falta de redundancia en una sola AZ. Hay que redesplegar infraestructura, configuración y código → **IaC** (CloudFormation, AWS CDK) para no exceder el **RTO**; respaldar también código, configuraciones y **AMI**; **CodePipeline** para automatizar el redespliegue |
| **Pilot light** | Datos replicados y BD/almacenamiento **siempre encendidos**; servidores de aplicación cargados pero **apagados** | Mejor práctica: no desplegar el recurso apagado, sino tener lista la configuración para desplegarlo; se escala a producción encendiendo servidores |
| **Warm standby** | Copia **a escala reducida pero totalmente funcional** | Recuperación más rápida; facilita pruebas continuas de DR. Ej. de Raf: flota EC2 reducida en un Auto Scaling group |
| **Multi-site active/active** | Todo, sirviendo tráfico en **todas** las Regiones | El más complejo y costoso; RTO casi cero (la corrupción de datos aún puede exigir restaurar backups → RPO distinto de cero). Si se levanta un entorno completo, la mayoría prefiere usarlo activo/activo; si no, warm standby es más barato y simple |

> [!note] ➕ Nota del editor
> En el diálogo, Morgan y Raf mezclan por momentos "pilot light" y "warm standby" al describir la "flota reducida"; la lectura aclara la diferencia: en pilot light los servidores de aplicación están apagados; en warm standby corren a escala reducida.

### 7.2 Escalado de contenedores en ECS (dos capas)

| Capa | Mecanismo | Cómo funciona |
|---|---|---|
| **Clúster** (instancias EC2) | **Cluster auto scaling** vía un **Auto Scaling group capacity provider** con *managed scaling* | ECS crea **dos métricas personalizadas de CloudWatch** y una política de **target tracking** en el Auto Scaling group, y escala según la carga que imponen las tareas |
| **Servicio** (tareas/contenedores) | **Service auto scaling** con **Application Auto Scaling** | Ajusta el número deseado de tareas ante alarmas de CloudWatch. Políticas: **target tracking** (recomendada; como un termostato) y **step scaling** (ajustes por tramos según el tamaño de la brecha de la alarma) |

🎯 **Para el examen:** las cuatro estrategias de DR en orden de costo/RTO; DX + VPN de respaldo; escalado ECS en dos capas; Intelligent-Tiering para patrones desconocidos.

---

## 8. Demos y laboratorios

### 8.1 Demo — Exploración de Amazon RDS

- **Objetivo:** recorrer la creación de una instancia RDS PostgreSQL.
- **Servicios:** Amazon RDS (consola).
- **Pasos mostrados:**
  1. Consola → RDS → **Create database** → método **Standard create**.
  2. Motor: Aurora, MySQL, MariaDB, **PostgreSQL**, Oracle o SQL Server; elegir **la misma versión** que se usa on-premises para que las apps se comporten igual.
  3. Plantilla: Production, Dev/Test o **Free Tier** (se elige Free Tier; por eso "Availability and durability" queda deshabilitado en **Single DB instance**; en producción se elegiría Multi-AZ instance o cluster).
  4. **DB instance identifier** (se deja `database-1`), usuario y contraseña maestros (los usarán clientes como Workbench y las aplicaciones, junto con el endpoint).
  5. **DB instance class**: solo las elegibles Free Tier; se deja la por defecto.
  6. **Storage type**: General Purpose SSD (gp2), Provisioned IOPS (io1) o magnético; se elige gp2. Se fija el tamaño en GB (más almacenamiento puede dar más IOPS) y se habilita **storage autoscaling** con su umbral máximo.
  7. Conexión a un recurso de cómputo: **No**.
  8. **Performance Insights** activado (monitoreo avanzado para diagnosticar problemas de rendimiento).
  9. **Create database**; en unos minutos entrega un **endpoint** para conectar, cargar esquema y migrar datos.
- **Errores comunes:** dejar Single-AZ en producción; versión de motor distinta a la de origen.

### 8.2 Demo — Exploración de AWS Storage Gateway

- **Objetivo:** conocer las opciones de configuración (no se puede desplegar sin un entorno híbrido real).
- **Pasos mostrados:**
  1. Consola → Storage Gateway. Menú: File shares, FSx file systems, Volumes, Tape library.
  2. **Create gateway** → nombre (`test`) → **zona horaria** de donde estará el dispositivo on-premises (Eastern Time en la demo).
  3. Sección "How it works": crear el gateway, **conectarlo y activarlo** con AWS desde el dispositivo local, configurarlo y crear los recursos.
  4. Elegir tipo: S3 File Gateway, FSx File Gateway, Tape Gateway o Volume Gateway.
  5. Plataforma de host: **VMware, Hyper-V, Linux KVM, Amazon EC2 o appliance de hardware**; las instrucciones cambian según la elección (descargar la plantilla, instalarla, confirmar).
  6. Hasta ahí llega sin infraestructura híbrida: **Cancel** → **Exit**.

### 8.3 Demo — Exploración de AWS Systems Manager

- **Objetivo:** mostrar que Systems Manager es una colección de capacidades operativas muy distintas.
- **Pasos mostrados:**
  1. Consola → buscar **SSM** → Systems Manager. Recordatorio: los nodos necesitan el **SSM Agent**.
  2. **Operations Management → Explorer**: dashboards operativos personalizables con OpsData y OpsItems, agregados entre cuentas y Regiones.
  3. **Application Management → Parameter Store** → Create parameter: nombre `s3-bucket-for-app`, tier **Standard**, tipo **String**, data type texto, valor `morganexamplebucket`. Idea: **no hardcodear** constantes (como el nombre de un bucket) para no redesplegar código si cambian. El tipo **SecureString** cifra con **KMS**; **Secrets Manager** es otra opción para secretos.
  4. **Change Management → Maintenance Windows**: programar tareas de mantenimiento (p. ej. parches nocturnos): crear la ventana, asignar **targets** (nodos con SSM Agent), asignar **tasks** y revisar el estado tras la ejecución.
  5. **Run Command**: **documentos** (comandos predefinidos) que ejecutan scripts en instancias **sin SSH**. Ejemplo: `AWS-ConfigureWindowsUpdate` para habilitar/deshabilitar actualizaciones automáticas de Windows en un grupo de recursos; su contenido es un JSON con un script PowerShell. Se pueden crear documentos propios.

---

## 9. Aplicación de ejemplo: estado al cierre del módulo

La arquitectura final que Morgan presenta a Raf (versión digital, con piezas pequeñas añadidas al pasar del escritorio de bloques al diagrama):

```
 CENTRO DE DATOS (AnyCompany)                     AWS CLOUD · Región
 ┌───────────────────────────┐                   ┌──────────────────────────────────────────────┐
 │ VMs con contenedores      │                   │  Amazon ECS (control) · Amazon ECR (registro)│
 │  + ECS agent + SSM Agent  │◄── ECS Anywhere ──┤  Systems Manager · AWS Backup               │
 │ PostgreSQL (origen)       │── DMS ───────┐    │ ┌──────────── VPC ─────────────────────────┐ │
 │ S3 File Gateway (NFS)     │── asíncrono ─┼──► │ │  AZ-a                 AZ-b               │ │
 │ Router ─┬─ Direct Connect ════════════════════╡ │ [pub] NAT GW          [pub] NAT GW       │ │
 │         └─ (VPN respaldo, recomendada)  │    │ │ [priv] ALB → EC2 (ECS cluster, ASG)      │ │
 │ Systems Manager / Backup  │             │    │ │ [priv] RDS PostgreSQL primary ⇄ standby  │ │
 └───────────────────────────┘             └──► │ └──────────────────────────────────────────┘ │
                                                 │  Amazon S3 (Lifecycle, opc. Intelligent-Tiering, CRR) │
                                                 └──────────────────────────────────────────────┘
```

| Requisito | Cómo se cumple |
|---|---|
| R1 Conectividad dedicada | AWS Direct Connect (+ VPN de respaldo como mejora) |
| R2/R4 Contenedores y mismas herramientas | Amazon ECS en AWS y **ECS Anywhere** on-premises; Systems Manager y Backup en ambos lados |
| R3 Privacidad | ECS sobre EC2 y RDS en **subnets privadas**; NAT gateway + subnet pública por AZ solo para salida |
| R5 PostgreSQL | RDS PostgreSQL Multi-AZ, migrado con DMS (homogénea) |
| R6 Sin reescribir | Contenedores portables; task definitions nuevas pero código intacto; NFS conservado vía File Gateway |
| R7 Archivos en AWS | S3 File Gateway → Amazon S3 |
| R8 Resiliencia | Todo repartido en **dos AZ**; EC2 Auto Scaling; **Application Load Balancer**; Multi-AZ en RDS |

Registro de contenedores: el cliente puede seguir con **Docker Hub** (se integra con ECS) o pasar a **Amazon ECR**, que Morgan dejó en el diagrama como propuesta. Morgan insiste en que es una **primera pasada** que se iterará con el cliente.

---

## ✅ Autoevaluación

> [!question]- 1. ¿Por qué una Site-to-Site VPN no basta como enlace principal para este cliente, aunque esté cifrada?
> Porque sigue viajando sobre la Internet pública: su rendimiento depende del ancho de banda disponible en cada momento, y el cliente exige baja latencia y rendimiento consistente con alto volumen. Sí sirve como **respaldo** de Direct Connect.

> [!question]- 2. El cliente quiere AMI propia y SSH a los nodos. ¿Qué tipo de lanzamiento descarta y por qué?
> Descarta **Fargate**, que es serverless y no permite traer AMI ni entrar por SSH a la instancia subyacente. Se usa el tipo **EC2**, con el ECS agent en cada instancia.

> [!question]- 3. Compare NAT instance y NAT gateway en los tres criterios que menciona Morgan.
> Costo: la NAT instance suele ser más barata. Escalabilidad: la instance está limitada por su tipo/tamaño; el gateway escala solo. Fiabilidad: la instance no es redundante (hay que gestionar varias y su failover, o un ASG 1/1/1); el gateway es redundante dentro de su AZ y se despliega uno por AZ.

> [!question]- 4. ¿Qué diferencia operativa hay entre Multi-AZ y una read replica en RDS?
> Multi-AZ replica **sincrónicamente** a una standby para failover automático vía DNS (disponibilidad). La read replica replica **asincrónicamente** y sirve lecturas para descargar a la primaria (rendimiento); puede promoverse a independiente.

> [!question]- 5. ¿Qué herramienta adicional necesita una migración heterogénea con DMS?
> **AWS Schema Conversion Tool** para convertir esquema y código al motor destino; después DMS migra los datos y mantiene la sincronización.

> [!question]- 6. ¿Por qué Storage Gateway le gana a EFS en este caso si ambos hablan NFS?
> Porque con EFS cada escritura on-premises debe ir y volver de AWS por Direct Connect; con S3 File Gateway se escribe en un dispositivo local y la carga a S3 es asíncrona. Además, los datos quedan ya en S3 para la futura reescritura a almacenamiento de objetos.

> [!question]- 7. ¿Qué tres pasos requiere ECS Anywhere?
> Crear una activation key, instalar SSM Agent y ECS agent en las VM on-premises con esa clave, y definir/desplegar la aplicación como en ECS.

> [!question]- 8. En pilot light, ¿qué está encendido y qué apagado en la Región de recuperación?
> Encendidos: la replicación de datos y recursos como bases de datos y almacenamiento de objetos. Apagados (o listos para desplegar): los servidores de aplicación, que se encienden y escalan al invocar el failover.

> [!question]- 9. ¿Por qué no basta EC2 Auto Scaling para escalar una aplicación en ECS sobre EC2?
> Porque escala instancias, no tareas. Hacen falta dos capas: cluster auto scaling (capacity provider con managed scaling) para las instancias y service auto scaling (Application Auto Scaling, target tracking o step) para el número de tareas.

> [!question]- 10. ¿Para qué sirve Parameter Store en la demo?
> Para externalizar constantes (el nombre de un bucket) como pares clave-valor y referenciarlas desde código o comandos sin hardcodearlas; con SecureString se cifran con KMS.

---

## 🧾 Resumen en 10 líneas

1. AnyCompany Insurance migra la mitad de su carga de contenedores a AWS y deja la otra mitad on-premises por años: diseño **híbrido**, lift-and-shift y resiliencia como prioridad.
2. **Direct Connect** une ambos lados con rendimiento dedicado; Site-to-Site VPN queda como **respaldo**; Transit Gateway solo si hay muchas VPC/cuentas/Regiones.
3. **ECS** (simplicidad) sobre **EC2** (AMI propia y SSH) en subnets privadas, con ALB y EC2 Auto Scaling.
4. **NAT gateway** en subnet pública, **una por AZ**, para salida a Internet sin exposición.
5. **RDS PostgreSQL Multi-AZ** (failover síncrono por DNS); read replicas para lecturas pesadas y DR entre Regiones.
6. **DMS** migra homogéneamente con la fuente operativa y replicación continua; **SCT** para migraciones heterogéneas.
7. **S3 File Gateway** conserva NFS on-premises con escrituras locales y subida asíncrona a **S3**, más ciclo de vida.
8. **ECS Anywhere**, **Systems Manager** (Run Command, Parameter Store, Maintenance Windows) y **AWS Backup** unifican la operación en ambos entornos.
9. Mejoras: Storage Auto Scaling en RDS, escalado ECS en dos capas, CRR e Intelligent-Tiering en S3, Región secundaria con IaC.
10. DR en cuatro escalones: backup & restore, pilot light, warm standby y multi-site active/active, de menor a mayor costo y de mayor a menor RTO.

---

## 📖 Recursos del módulo

- [Opciones de conectividad de red a VPC de Amazon](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html)
- [¿Qué es AWS Direct Connect?](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html)
- [Preguntas frecuentes sobre AWS Direct Connect](https://aws.amazon.com/directconnect/faqs/?nc=sn&loc=6)
- [¿Qué es AWS Site-to-Site VPN?](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
- [¿Qué es AWS Client VPN?](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/what-is.html)
- [¿Qué es una transit gateway?](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
- [Contenedores en AWS](https://aws.amazon.com/containers/)
- [Preguntas frecuentes de contenedores](https://aws.amazon.com/containers/faqs/)
- [Tipos de lanzamiento de Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html)
- [Conectarse a Internet u otras redes con dispositivos NAT](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html)
- [Comparar NAT gateways y NAT instances](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)
- [Amazon RDS Multi-AZ](https://aws.amazon.com/rds/features/multi-az/)
- [Réplicas de lectura de Amazon RDS](https://aws.amazon.com/rds/features/read-replicas/)
- [Instancias de base de datos de Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.html)
- [¿Qué es AWS Database Migration Service?](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [¿Qué es Amazon S3 File Gateway?](https://docs.aws.amazon.com/filegateway/latest/files3/what-is-file-s3.html)
- [Características de AWS Storage Gateway](https://aws.amazon.com/storagegateway/features/)
- [Tipos de volumen de Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
- [¿Qué es Amazon EFS?](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
- [¿Qué es Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide//Welcome.html)
- [Introducción a Amazon S3](https://aws.amazon.com/s3/getting-started/?nc=sn&loc=6&dn=1)
- [Clases de almacenamiento de Amazon S3](https://aws.amazon.com/s3/storage-classes/)
- [Taller de Amazon ECS: ECS Anywhere](https://www.ecsworkshop.com/ecsanywhere/)
- [Ejecutar comandos de forma remota en una instancia EC2 con Systems Manager](https://aws.amazon.com/getting-started/hands-on/remotely-run-commands-ec2-instance-systems-manager/)
- [Taller de herramientas de administración y gobernanza: AWS Systems Manager](https://mng.workshop.aws/ssm.html)
- [¿Qué es AWS Systems Manager?](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
- [¿Qué es AWS Backup?](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [Amazon Machine Images (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [AWS CodePipeline](http://aws.amazon.com/codepipeline)
- [Disaster Recovery of Workloads on AWS: Recovery in the Cloud (whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [¿Cómo configuro la conmutación por error de Direct Connect y VPN con Transit Gateway?](https://aws.amazon.com/premiumsupport/knowledge-center/dx-configure-dx-and-vpn-failover-tgw/)
- [Escalado automático de clústeres de Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-auto-scaling.html)
- [Taller de Amazon ECS: autoescalado de clústeres](https://ecsworkshop.com/capacity_providers/ec2/)
- [Service auto scaling (Guía para desarrolladores de ECS)](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)
- [Storage autoscaling de Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIOPS.StorageTypes.html#USER_PIOPS.Autoscaling)
- [Amazon S3 Intelligent-Tiering](https://aws.amazon.com/s3/storage-classes/intelligent-tiering/)

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[06-Analitica-de-Datos-Serverless|06 · Analítica de datos serverless]] · Siguiente: [[08-Gobierno-y-Multicuenta|08 · Gobierno y multicuenta ➡]]
