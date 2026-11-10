---
title: "Tomo 03 — Almacenamiento y Bases de Datos"
tags: [aws, solutions-architect, almacenamiento, bases-de-datos, s3, ebs, rds, dynamodb]
curso: 1 · AWS Cloud Technical Essentials
modulo: "Semana 3: Almacenamiento y bases de datos en AWS"
audiencias: [tecnico, puente, ejecutivo]
tomo: 03
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🗄️ Tomo 03 — Almacenamiento y Bases de Datos

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[02-Computo-y-Redes|02 · Cómputo y Redes]] · Siguiente: [[04-Monitoreo-y-Optimizacion|04 · Monitoreo y Optimización ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Al cierre del tomo 02 la aplicación Employee Directory ya corría sobre EC2 dentro de una VPC, pero no funcionaba: no tenía dónde guardar las fotos de los empleados ni sus datos. Este tomo resuelve ambas piezas. Primero ordena los tres tipos de almacenamiento (file, block, object) y los servicios que los implementan —instance store, Amazon EBS, Amazon S3, Amazon EFS y Amazon FSx—; luego recorre las bases de datos administradas, con Amazon RDS y Amazon DynamoDB como protagonistas, y cierra con la idea de *purpose-built databases*: elegir la base según el caso de uso y no forzar los datos a una base única.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** dónde vive cada tipo de dato (sistema operativo, archivos compartidos, contenido estático, registros transaccionales), cuánta durabilidad se compra, qué se respalda y por cuánto tiempo, y si la base de datos se paga por hora de instancia o por uso.
> - **Costo de hacerlo mal:** datos perdidos al detener una instancia (instance store usado como almacenamiento persistente), buckets expuestos a internet, facturas de S3 infladas por versiones antiguas o clases de almacenamiento mal elegidas, y bases relacionales encendidas 24×7 para una simple tabla de búsqueda que nadie usa el fin de semana.
> - **Pregunta que responde:** *¿qué servicio de almacenamiento y qué base de datos corresponden a cada dato de mi aplicación, según cómo se accede a él, cuánto debe durar y cuánto estoy dispuesto a operar?*

> [!tip] 💡 Analogía general: la oficina y sus muebles
> El **instance store** es la pizarra de la sala de reuniones: rapidísima, incluida en el arriendo, pero se borra al irse. **EBS** es el disco externo que se conecta a un computador: persiste aunque el equipo se apague y puede cambiarse de escritorio (dentro del mismo piso, es decir, la misma AZ). **EFS/FSx** es la carpeta compartida de red que todos montan a la vez. **S3** es la bodega externa con guardias: cabe todo, cada caja tiene etiqueta única y nadie entra sin autorización. **RDS** es el archivo contable con un administrador contratado que lo mantiene; **DynamoDB** es un fichero de tarjetas infinito que se paga por consulta.

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Almacenamiento en AWS | 6 videos (intro, tipos, instance store y EBS, S3, juego «elija el servicio», demo bucket S3) · 4 lecturas (3.1–3.4) · 1 actividad coach | Tipos file/block/object; instance store vs EBS; tipos de volumen y snapshots; S3: buckets, acceso, políticas, cifrado, versionado, clases y ciclo de vida; EFS y FSx; criterios de elección |
| Bases de datos en AWS | 3 videos (explorar BD, RDS, BD específicas) + 1 video DynamoDB · 4 lecturas (3.5, 3.6, 3.8, 3.9) · 1 actividad coach | Relacional vs NoSQL; administrada vs no administrada; RDS (motores, instancias, VPC, backups, Multi-AZ); DynamoDB (tablas, ítems, atributos, claves); portafolio de bases *purpose-built* |
| Semana 3: ejercicio y evaluación | 2 videos (intro ejercicio, demo DynamoDB) · Lab 3 (calificado) · Cuestionario semanal | Conectar la aplicación a S3 y a DynamoDB y comprobar lectura/escritura |

> [!note] ➕ Nota del editor
> La numeración de lecturas del curso salta de 3.6 a 3.8: no aparece una «Lectura 3.7» en el material extraído. Las actividades *coach* («Selección del servicio de almacenamiento de AWS adecuado» y «Decisiones sobre arquitectura de bases de datos»), el Lab 3 y el cuestionario no traen texto; se describen solo por su objetivo.

---

## 1. Qué datos necesita guardar la aplicación

Audiencia: 🔧 🧭

La Employee Directory necesita tres clases de datos, y cada una pide un almacenamiento distinto:

| Dato | Ejemplo | Patrón de acceso | Almacenamiento adecuado |
|---|---|---|---|
| Sistema operativo, software, archivos de la aplicación | Volumen raíz de la instancia | Se escribe y actualiza seguido, alta tasa de transacciones | **Block** (EBS) |
| Activos estáticos | Fotos de los empleados | Se leen mucho, se modifican rara vez | **Object** (S3) |
| Datos estructurados | Nombre, cargo, ubicación, insignias | Consultas y altas/bajas/ediciones | **Base de datos** (RDS → finalmente DynamoDB) |

**🔧 La diferencia clave entre block y object** se ve con un ejemplo: un archivo de texto de 1 GB del que se quiere cambiar un solo carácter.

```
 BLOCK                                   OBJECT
 [blk1][blk2][blk3*][blk4]...[blkN]      [ ========= objeto completo ========= ]
        se reescribe solo blk3                 hay que reescribir TODO el objeto
```

- En **block** el archivo se parte en trozos de tamaño fijo; se modifica solo el bloque que contiene el carácter.
- En **object** el archivo es una unidad indivisible; cualquier cambio obliga a subir el objeto entero.

**👔 En una frase para el negocio:** lo que cambia seguido va a bloques; lo que se escribe una vez y se lee muchas, a objetos.

---

## 2. Los tres tipos de almacenamiento: file, block y object

Audiencia: 🔧 🧭

| | **File storage** | **Block storage** | **Object storage** |
|---|---|---|---|
| Estructura | Jerarquía de árbol (carpetas y subcarpetas) | Bloques de tamaño fijo, cada uno con dirección propia | Estructura **plana**; cada objeto con identificador único |
| Unidad | El archivo, con metadatos (nombre, tamaño, fecha de creación) y una ruta | El bloque; sin metadatos salvo la dirección | Objeto = datos + identificador + metadatos, empaquetados |
| Recuperación | El sistema usa la ruta (`computador/Archivos_de_aplicacion/Fotos_de_gatos/gatos-03.png`) | Reensambla los bloques en orden usando sus direcciones | Por identificador único |
| Modificar un carácter | — | Solo el bloque afectado → rápido y con menos ancho de banda | Reescribir el objeto entero |
| Fortaleza | Acceso centralizado y compartido por varios hosts; requiere file locking y protocolos de sistema de archivos | Baja latencia | Escalabilidad: casi cualquier tipo de dato, sin límite en número de objetos |
| Casos de uso | Grandes repositorios de contenido, entornos de desarrollo, directorios personales de usuarios | Bases de datos, ERP y cargas empresariales de alto rendimiento | Grandes conjuntos de datos, archivos no estructurados (multimedia), activos estáticos (fotos) |
| Equivalente on-premises | **NAS** (network-attached storage) | **DAS** (direct-attached storage) o **SAN** (storage area network) | — |

> [!tip] 💡 Analogía
> File storage es el Explorador de Windows o el Finder de macOS: carpetas dentro de carpetas. Block storage es un libro con páginas numeradas: para corregir una palabra se cambia una página. Object storage es una bodega de cajas selladas con código de barras: para cambiar algo de una caja se reemplaza la caja completa.

**🧭 Ventaja de la nube:** en un centro de datos agregar almacenamiento implica comprar, instalar y configurar equipos; en AWS se crean, eliminan y modifican soluciones de almacenamiento en minutos.

🎯 **Para el examen:**
- Asociar *jerarquía + varios hosts montando* → file; *baja latencia + transaccional* → block; *plano + escalable + vía web* → object.
- DAS/SAN ↔ block; NAS ↔ file.

---

## 3. Almacenamiento en bloque para EC2: instance store y Amazon EBS

Audiencia: 🔧 🧭

Toda instancia EC2 necesita almacenamiento en bloque, sea como **volumen de arranque** (sistema operativo) o como **volumen de datos** aparte. Igual que un portátil tiene un disco interno y puede usar discos externos, EC2 tiene dos opciones:

```
        ┌──────────── Host físico ────────────┐
        │  ┌───────────┐   ┌───────────────┐  │        red        ┌─────────────┐
        │  │ Instancia │───│ Instance store│  │ ───────────────── │ Volumen EBS │
        │  │   EC2     │   │ (disco local) │  │  (almac. en red)  │ (separado)  │
        │  └───────────┘   └───────────────┘  │                   └─────────────┘
        └─────────────────────────────────────┘
          muere con la instancia                 persiste aunque la instancia muera
```

### 3.1 Instance store

- Almacenamiento **temporal a nivel de bloque** en discos físicamente conectados al host que aloja la instancia.
- **Ventaja:** al estar tan cerca del servidor, es muy rápido.
- **Gran inconveniente:** su ciclo de vida está atado al de la instancia. Si la instancia se **detiene o termina**, los datos desaparecen. Por eso se le llama **almacenamiento efímero**.
- **Costo:** va incluido en el precio de la instancia.
- **Casos ideales:**
  - Aplicaciones que **replican datos** entre instancias, como clústeres **Hadoop**: suman la velocidad del disco local con la resiliencia de la réplica.
  - Datos temporales que cambian seguido: **buffers, cachés, memoria virtual (scratch data)** y otros contenidos transitorios.

### 3.2 Amazon EBS (Elastic Block Store)

- Dispositivos de almacenamiento en bloque, llamados **volúmenes EBS**, de tamaño definido por el usuario, **separados** de la instancia y conectados por red.
- Al adjuntar un volumen, la instancia tiene una línea de comunicación directa con él; nadie más habla directamente con ese volumen. **Se necesita una instancia EC2 para acceder a los datos de un volumen EBS.**
- **Relación con las instancias:**
  - La mayoría de los volúmenes son **uno a uno**: un volumen, una instancia a la vez.
  - Se puede **detener la instancia, desacoplar el volumen y adjuntarlo a otra instancia en la misma AZ**.
  - **EBS Multi-Attach:** según el tipo de instancia y de volumen, un volumen puede adjuntarse a varias instancias a la vez; no está disponible para todos los tipos y todas las instancias deben estar en la **misma AZ**.
  - Una instancia puede tener **muchos** volúmenes (uno a muchos).
- **Almacenamiento persistente:** si la instancia falla, se detiene o se termina, el volumen y sus datos siguen existiendo.
- **Límite de tamaño:** como un disco externo de 2 TB que solo admite 2 TB, el volumen tiene un máximo.

**Escalar EBS — dos vías:**
1. **Aumentar el tamaño del volumen** hasta el máximo de **16 TB** (por ejemplo, un volumen de 5 TB puede crecer hasta 16 TB).
2. **Adjuntar más volúmenes** a la misma instancia, durante o después de su creación.

**Casos de uso de EBS:**
- **Sistemas operativos:** volúmenes raíz o de arranque. El dispositivo raíz de una instancia lanzada desde una AMI suele ser un volumen EBS: son las **AMI respaldadas por EBS** (*EBS-backed AMIs*).
- **Bases de datos** sobre EC2 que dependen de lecturas y escrituras transaccionales.
- **Aplicaciones empresariales** críticas que necesitan bloque fiable.
- **Aplicaciones intensivas en throughput** con lecturas y escrituras largas y continuas.

### 3.3 Tipos de volumen EBS

Dos familias: **SSD** (gran rendimiento en E/S **aleatoria**; el rendimiento se mide en **IOPS**) y **HDD** (gran rendimiento en E/S **secuencial**; el rendimiento se mide en **MB/s**). AWS ofrece dos tipos de cada una:

| | **Provisioned IOPS SSD** | **General Purpose SSD** | **Throughput Optimized HDD** | **Cold HDD** |
|---|---|---|---|---|
| Descripción | SSD de mayor rendimiento para cargas transaccionales sensibles a la latencia | SSD que equilibra precio y rendimiento para variadas cargas transaccionales | HDD de bajo costo para cargas de acceso frecuente y alto throughput | HDD de bajo costo para cargas de acceso menos frecuente |
| Casos de uso | Bases relacionales y NoSQL de E/S intensiva | Volúmenes de arranque, apps interactivas de baja latencia, desarrollo y pruebas | Big data, data warehouses, procesamiento de logs | Datos fríos con pocos escaneos al día |
| Tamaño | 4 GB – 16 TB | 1 GB – 16 TB | 500 GB – 16 TB | 500 GB – 16 TB |
| IOPS máx./volumen | 64.000 | 16.000 | 500 | 250 |
| Throughput máx./volumen | 1.000 MB/s | 250 MB/s | 500 MB/s | 250 MB/s |

> [!note] ➕ Nota del editor
> En la consola estos tipos aparecen con sus códigos: `io1`/`io2` (Provisioned IOPS), `gp2`/`gp3` (General Purpose), `st1` (Throughput Optimized) y `sc1` (Cold HDD). Las cifras de la tabla son las del material del curso; AWS las actualiza con el tiempo, así que conviene contrastarlas con la documentación vigente antes del examen.

### 3.4 Beneficios de EBS (hoja de trucos)

- **Alta disponibilidad:** el volumen se replica automáticamente **dentro de su AZ** para evitar puntos únicos de fallo.
- **Persistencia:** sobrevive a la instancia.
- **Cifrado:** todos los volúmenes lo admiten.
- **Flexibilidad:** se puede cambiar el tipo, el tamaño y las IOPS **sin detener la instancia**.
- **Backups:** cualquier volumen puede respaldarse.

### 3.5 EBS snapshots

- Son los backups de los volúmenes EBS: **incrementales** (solo guardan los bloques cambiados desde el último snapshot).
  - Ejemplo del curso: un volumen con 10 GB de datos del que cambiaron 2 GB desde el último snapshot → solo esos 2 GB se escriben.
- Se guardan de forma **redundante en varias AZ usando Amazon S3**, pero ese S3 lo administra AWS: no se interactúa con S3, sino desde la consola de EBS (que forma parte de la consola de EC2).
- A partir de un snapshot se pueden crear **varios volúmenes nuevos**, en la misma AZ o en otra; cada uno es copia exacta del original en el momento del snapshot.

> [!warning] ⚠️ Regla crítica
> Un volumen EBS está atado a **una AZ**: se replica dentro de ella, y para moverlo a otra se pasa por un snapshot. Instance store no tiene snapshots ni sobrevive a un *stop*: nunca guarde ahí algo que no pueda reconstruir.

🎯 **Para el examen:**
- *Datos temporales, velocidad y costo, no importa la durabilidad* → instance store.
- *Base de datos sobre EC2, rápida y duradera* → EBS (Provisioned IOPS si es de E/S intensiva).
- *Big data, logs, lectura secuencial* → Throughput Optimized HDD; *datos fríos baratos* → Cold HDD.
- Snapshot = incremental, en S3, multi-AZ, sirve para crear volúmenes en otra AZ.

---

## 4. Amazon S3: almacenamiento de objetos para internet

Audiencia: 🔧 🧭

### 4.1 Por qué no guardar las fotos en EBS

1. La mayoría de los volúmenes EBS se conectan a **una sola instancia** a la vez; Multi-Attach no está en todos los tipos. Cuando la app escale a varias instancias, todas necesitarán ver las fotos.
2. EBS tiene **límite de tamaño**: con el tiempo, las fotos 4K de los empleados no cabrán.

**Amazon S3 (Simple Storage Service)** es almacenamiento **independiente del cómputo**: no se monta en las instancias, se accede **por URL desde cualquier lugar de la web** —de ahí su apodo, *storage for the internet*—.

### 4.2 Características

- Tipo subyacente: **object storage** (estructura plana, identificadores únicos; un objeto es un archivo más metadatos).
- **Objetos ilimitados**, con un máximo de **5 TB por objeto**.
- **Almacenamiento distribuido:** los datos se guardan en varias instalaciones dentro de la Región; según la lectura 3.3, en varios dispositivos repartidos en varias AZ.
- Diseñado para **99,99 % de disponibilidad** y **99,999999999 % (11 nueves) de durabilidad** por año.
- **Se paga por lo que se usa**; no se aprovisiona por adelantado.

### 4.3 Buckets, claves y URL

- Los objetos viven en **buckets**; no se puede subir nada sin crear primero un bucket.
- Al crear un bucket se eligen como mínimo:
  - **Región:** los buckets son regionales; conviene la misma Región del resto de la infraestructura (en el curso, **Oregón, `us-west-2`**).
  - **Nombre:** **único a nivel global** entre todas las cuentas de AWS y **compatible con DNS** (sin espacios ni caracteres especiales), porque AWS arma una URL con él. El nombre queda tomado hasta que se borra el bucket.
- Estructura de la URL de un objeto (ejemplo del material):

```
 http://doc.s3.amazonaws.com/2006-03-01/AmazonS3.html
        └┬┘ └──────┬───────┘ └───┬────┘ └─────┬─────┘
       bucket   servicio y     "carpeta"    objeto
                proveedor      implícita
                └──────────── object key = 2006-03-01/AmazonS3.html ───┘
```

- El nombre del objeto se llama **key** (clave).
- Las **carpetas** ayudan a organizar, pero no existe jerarquía real en el back end: todo vive en el mismo nivel; las carpetas solo *sugieren* una jerarquía para el ojo humano.

### 4.4 Casos de uso de S3

| Caso | Por qué S3 |
|---|---|
| Backup y almacenamiento | Altamente redundante; AWS guarda ahí los snapshots de EBS |
| Alojamiento de medios | Objetos ilimitados de hasta 5 TB: video, fotos, música |
| Entrega de software | Hospedar aplicaciones para descarga |
| Data lakes | Escala casi ilimitada, de GB a PB, pagando lo que se usa |
| Sitios web estáticos | HTML, CSS y scripts del lado del cliente, incluso con dominio propio |
| Contenido estático | Escala ilimitada, archivos grandes, acceso web en cualquier momento |

### 4.5 Quién puede acceder: privado por defecto

**Todo en S3 es privado por defecto:** buckets, carpetas y objetos solo los ve el usuario o la cuenta que los creó. En la demo, al hacer clic en la URL del objeto recién subido aparece **Access Denied**, porque el navegador actúa como usuario anónimo de internet.

Hacer público un objeto es deliberadamente engorroso, para **evitar exposiciones accidentales**. Pasos que muestra el video:

1. En el detalle del objeto, *Object actions → Make public using ACL* aparece **deshabilitado**.
2. Bucket → pestaña **Permissions** → **Block public access**: viene bloqueando todo. *Edit*, desmarcar la casilla superior (bloquear todo acceso público), guardar, escribir `confirm` y confirmar.
3. Misma pestaña → **Object Ownership** → *Edit*: cambiar de *ACLs disabled* (por defecto) a **ACLs enabled**, confirmar y guardar.
4. Volver al objeto → *Object actions* → **Make public using ACL**.
5. Abrir la URL del objeto: ahora se ve la foto.

> [!danger] 🚨 Error costoso
> Abrir un bucket «para probar» y olvidarlo. Público significa que **cualquiera en internet** puede verlo. Lo habitual es no querer un todo-o-nada, sino acceso granular con políticas.

### 4.6 IAM policies vs bucket policies

Ambas se escriben en el **mismo lenguaje de políticas en JSON**; la diferencia es a qué se adjuntan.

| | **IAM policy** | **S3 bucket policy** |
|---|---|---|
| Se adjunta a | Usuarios, grupos y roles IAM | **Solo buckets** (no a carpetas ni objetos; pero aplica a todos los objetos del bucket) |
| Alcance | Casi cualquier acción de AWS, no ligada a un servicio | Acciones permitidas o denegadas sobre ese bucket |
| Úsela cuando… | Hay **muchos buckets** con permisos distintos (evita mantener muchas bucket policies); se quiere **todo centralizado** en un lugar | Se necesita **acceso entre cuentas** (cross-account) simple sin roles IAM; las IAM policies chocan con su **límite de tamaño** (las bucket policies admiten más) |
| Ejemplos | — | Permitir que otra cuenta de AWS ponga objetos; dar lectura a visitantes anónimos |

Ejemplo de bucket policy de lectura pública que trae la lectura (bucket `employeebucket`; aquí con las claves en inglés, como las exige AWS, en lugar de la traducción automática del material):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::employeebucket/*"]
    }
  ]
}
```

### 4.7 Cifrado

S3 aplica cifrado **en tránsito** y **en reposo**.

- **En reposo:**
  - **Server-side encryption:** S3 cifra el objeto antes de escribirlo en disco y lo descifra al descargarlo.
  - **Client-side encryption:** el cliente cifra antes de subir; él administra el proceso, las claves y las herramientas.
- **En tránsito:** client-side encryption o **SSL** (Secure Sockets Layer).

### 4.8 Versionado

Sin versionado, subir otra vez `empleado.jpg` a la carpeta `empleados` **sobrescribe** el original: se pierde el anterior, y mantener variantes obliga a inventar nombres distintos que desordenan el bucket.

Con **versioning**, S3 asigna un **version ID único** a cada objeto almacenado: pueden convivir `empleadofoto.gif` (versión 111111) y `empleadofoto.gif` (versión 121212).

- **Borrar** no elimina definitivamente: S3 pone un **delete marker**; quitándolo, el objeto se restaura.
- **Sobrescribir** crea una versión nueva; las anteriores siguen accesibles.

**Estados del bucket** (aplican a todos sus objetos):

| Estado | Efecto |
|---|---|
| **Unversioned** (por defecto) | Ningún objeto, nuevo o existente, tiene versión |
| **Versioning-enabled** | Versionado para todos los objetos |
| **Versioning-suspended** | Los objetos nuevos no tienen versión; los existentes **conservan** las suyas |

> [!warning] ⚠️ Regla crítica
> Se paga el almacenamiento de **todas las versiones**. Para bajar la factura, elimine las versiones antiguas que ya no se usan (o automatícelo con ciclo de vida).

### 4.9 Clases de almacenamiento

Si no se especifica clase al subir, el objeto va a **S3 Standard**. Las clases permiten mover datos a medida que cambia su patrón de acceso.

| Clase | Para qué | Detalle del material |
|---|---|---|
| **S3 Standard** | Uso general: apps en la nube, sitios dinámicos, distribución de contenido, móviles y juegos, analítica de big data | Clase por defecto |
| **S3 Intelligent-Tiering** | Patrones de acceso **desconocidos o cambiantes** | Dos niveles (frecuente e infrecuente); S3 monitorea y mueve los objetos al más rentable |
| **S3 Standard-IA** | Acceso poco frecuente pero **rápido cuando se necesita** | Misma durabilidad, rendimiento y latencia que Standard; menor precio por GB almacenado, con cargo por GB recuperado. Backups de largo plazo, archivos de DR |
| **S3 One Zone-IA** | Infrecuente, sin necesidad de la resiliencia multi-AZ | Una sola AZ (las demás usan **mínimo 3 AZ**); **20 % más barata** que Standard-IA. Copias secundarias de backups on-premises o datos fácilmente recreables |
| **S3 Glacier Instant Retrieval** | Archivo de larga vida, poco acceso, **recuperación en milisegundos** | El archivo más barato con acceso inmediato |
| **S3 Glacier Flexible Retrieval** | Archivo accedido **1–2 veces al año**, recuperación **asíncrona** | Hasta 10 % más barato que Glacier Instant Retrieval |
| **S3 Glacier Deep Archive** | Retención de largo plazo y preservación digital (acceso 1–2 veces al año) | La clase **más barata** de S3; pensada para sectores regulados (finanzas, salud, sector público) que guardan datos **7–10 años o más** |
| **S3 on Outposts** | Almacenamiento de objetos en un entorno **AWS Outposts** local | — |

> [!note] ➕ Nota del editor
> El texto del curso anuncia «seis clases» pero enumera ocho (contando Outposts). Se conservan todas tal como las lista.

### 4.10 Ciclo de vida de objetos (lifecycle)

En lugar de mover objetos de clase a mano, una **lifecycle policy** automatiza dos tipos de acciones:

- **Transition actions:** cuándo pasar objetos a otra clase. Ej.: a **Standard-IA a los 30 días** de creados; a **Glacier al año**.
- **Expiration actions:** cuándo vencen y se **eliminan permanentemente**.

Buenos candidatos:
- **Logs periódicos** que la app necesita una semana o un mes y luego pueden borrarse.
- **Documentos cuyo acceso decae:** muy consultados un tiempo, luego poco, luego solo archivados por normativa, y finalmente eliminables.

```
 creación ──30 días──▶ Standard-IA ──1 año──▶ Glacier ──n años──▶ expiración (borrado)
 (Standard)
```

🎯 **Para el examen:**
- Nombre de bucket **global y DNS-compatible**; bucket **regional**.
- 11 nueves de durabilidad, 99,99 % de disponibilidad, objeto máximo 5 TB.
- Privado por defecto; Block Public Access y ACLs deshabilitadas son la configuración inicial.
- Cross-account simple → bucket policy; muchos buckets / centralizar → IAM policies.
- Delete marker con versionado; *suspended* no borra versiones existentes.
- Elegir clase por frecuencia de acceso y latencia de recuperación; One Zone-IA = una AZ.

---

## 5. Almacenamiento de archivos: Amazon EFS y Amazon FSx

Audiencia: 🔧 🧭

S3 es plano y no es un sistema de archivos; la mayoría de los volúmenes EBS se conectan a una sola instancia. Para **almacenamiento de archivos montable en varias instancias EC2** AWS ofrece:

| Servicio | Característica |
|---|---|
| **Amazon EFS** (Elastic File System) | Sistema de archivos **NFS** totalmente administrado |
| **Amazon FSx for Windows File Server** | Servidor de archivos administrado sobre Windows Server, compatible con **SMB** |
| **Amazon FSx for Lustre** | Sistema de archivos **Lustre** administrado que se **integra con S3** |

Rasgos comunes: son **file storage**, se **paga por lo que se usa** (sin aprovisionar) y **pueden montarse en varias instancias EC2**.

---

## 6. Cómo elegir el servicio de almacenamiento

Audiencia: 🔧 🧭 👔

### 6.1 Comparación final

| | Instance store | Amazon EBS | Amazon S3 | Amazon EFS / FSx |
|---|---|---|---|---|
| Tipo | Block efímero | Block persistente | Object | File |
| Conexión al cómputo | Integrado al host, no se separa | Adjunto por red a EC2 (normalmente 1 instancia) | No se monta; acceso por URL | Montable en muchas instancias |
| Pago | Incluido en la instancia | Por lo **aprovisionado** (por adelantado) | Por lo **usado** | Por lo **usado** |
| Replicación | — | Varios servidores en **una AZ** | Varias **AZ** de la Región | — (no detallado en el material) |
| Para qué | Buffers, cachés, scratch, clústeres con réplica | Datos que cambian seguido y deben sobrevivir a stop/terminate/fallos; SO, BD | Contenido web estático, multimedia, backups, archivado, analítica, sitios estáticos | Archivos compartidos entre instancias |

### 6.2 El juego «¿qué servicio uso?» (video)

| Escenario | Respuesta | Razonamiento |
|---|---|---|
| App que transcodifica videos grandes con **AWS Lambda**; guardar originales y transcodificados **al menos un año** por normativa | **Amazon S3** | EBS solo se adjunta a EC2 (y aquí hay Lambda); videos grandes exigirían muchos volúmenes, caro a largo plazo. Instance store queda fuera: no hay EC2 y es efímero |
| E-commerce con **MySQL sobre EC2**; pedidos y clientes, acceso y actualización frecuentes, debe ser **rápido y duradero** | **Amazon EBS** | Instance store daría velocidad, pero no la durabilidad que exigen los datos que mueven el negocio |
| App web que escribe a disco **datos temporales** de un cálculo; importan **velocidad y costo** | **EC2 instance store** | La durabilidad no importa (si falla, se reinicia el cálculo); instance store ya va incluido en el precio de la instancia |
| (Bonus) **WordPress en varias instancias**; las subidas de usuarios van al sistema de archivos local y deben compartirse | **Amazon EFS** | S3 no es un sistema de archivos ni se monta en instancias. Se mueve el directorio de instalación de WordPress a EFS y se monta en cada instancia al arrancar |

> [!tip] 💡 Regla mnemotécnica
> **¿Lo monta más de una instancia?** → EFS/FSx. **¿Lo usa algo que no es EC2 o se accede por web?** → S3. **¿Debe sobrevivir a la instancia?** → EBS. **¿Es desechable y debe ser barato y rápido?** → instance store.

---

## 7. Bases de datos en AWS: relacional, administrada y no administrada

Audiencia: 🔧 🧭

### 7.1 Algo de historia y qué es una base relacional

- Antes elegir base de datos era fácil: pocos proveedores, y a menudo se elegía **uno para todas las aplicaciones**, incluso antes de entender el caso de uso. Desde los **años 70** la opción dominante fue la **base de datos relacional**.
- Organiza los datos en **tablas** de **filas** (registros: toda la información de una entrada) y **columnas** (atributos). Las tablas se vinculan entre sí por columnas comunes: de ahí lo *relacional*. Ejemplo del material: tablas de libros (ISBN, título, autor, formato), ventas y autores, unidas por el atributo autor.
- El conjunto de tablas, filas, columnas y relaciones es el **esquema lógico**. En relacional el esquema es **fijo**: cambiarlo con la base en operación es difícil, así que el modelado se hace **por adelantado**.
- Un **RDBMS** permite crear, actualizar y administrar la base. Ejemplos: **MySQL, PostgreSQL, Oracle, SQL Server, Amazon Aurora**. Se habla con ellos en **SQL** (ej.: `SELECT * FROM nombre_tabla`); su poder está en las consultas que cruzan tablas mediante **joins** (ventas × libros para ver las ventas de un autor).

**Ventajas:**
- **Joins** para entender relaciones entre datos.
- **Menos redundancia:** se guarda un dato una vez y se referencia.
- **Familiaridad:** décadas de uso, mucha gente sabe operarlas.
- **Exactitud:** alta integridad y cumplimiento **ACID** (atomicity, consistency, isolation, durability).

**Casos de uso:**
- Aplicaciones con **esquema sólido que cambia poco**, como las de **lift and shift** (llevar una app on-premises a la nube con pocas o ninguna modificación).
- Aplicaciones que requieren persistencia **ACID**: **ERP**, **CRM**, aplicaciones comerciales y financieras.

### 7.2 Administrada vs no administrada

El dilema se parece al modelo de responsabilidad compartida ([[01-Vision-General-y-Seguridad]]): es un intercambio entre **comodidad y control**.

| Responsabilidad | On-premises | No administrada (BD sobre **EC2**) | Administrada (**RDS** y similares) |
|---|---|---|---|
| Seguridad física y energía del centro de datos | Usted | AWS | AWS |
| Hardware e infraestructura | Usted | AWS | AWS |
| Instalación del sistema operativo | Usted | AWS | AWS |
| Administración del host (parches, etc.) | Usted | **Usted** | AWS |
| Instalación del motor, replicación multi-AZ, parches y actualizaciones del motor | Usted | **Usted** | AWS (alta disponibilidad, escalado, parches, backups) |
| Esquema, índices, stored procedures, tuning, optimización de consultas | Usted | Usted | **Usted** |
| Cifrado, control de acceso, seguridad de los datos de clientes | Usted | Usted | **Usted** |
| Nivel de control / comodidad | Máximo control | Intermedio | Máxima comodidad, menor control |

- Correr la base en EC2 es buena opción para **migrar bases existentes** y ya simplifica respecto de on-premises.
- Con la oferta administrada, AWS se encarga del *undifferentiated heavy lifting* y el equipo se concentra en lo que diferencia a la aplicación.

> [!example] 📊 Caso del material
> Una empresa que vende equipos sanitarios quiere ser el proveedor número uno del noroeste del Pacífico. Tener una base de datos es necesario para lograrlo, pero administrarla no la acerca a esa meta: RDS le permite delegar aprovisionamiento, parches, escalado y restauración.

---

## 8. Amazon RDS

Audiencia: 🔧 🧭

**Amazon RDS** (Relational Database Service) facilita configurar, operar y escalar una base relacional en la nube sin la carga de la administración tradicional.

### 8.1 Motores soportados

| Categoría | Motores |
|---|---|
| Comerciales | Oracle, SQL Server |
| Open source | MySQL, PostgreSQL, MariaDB |
| Cloud native | **Amazon Aurora** |

**Amazon Aurora:** base de AWS construida para la nube, **compatible con MySQL y PostgreSQL**; hasta **5 veces más rápida que MySQL estándar** y **3 veces más rápida que PostgreSQL estándar**, más duradera y disponible que las versiones RDS de esos motores. Indicada cuando se requiere almacenar grandes volúmenes con alta disponibilidad, durabilidad y baja latencia de recuperación.

> [!note] ➕ Nota del editor
> La transcripción automática dice que Aurora está «diseñada para ser *in*compatible con MySQL o PostgreSQL»; es un error de traducción: la lectura 3.6 confirma que es **compatible** con ambos.

### 8.2 Instancias de base de datos y almacenamiento

- RDS se compone de **cómputo + almacenamiento**. El cómputo es la **DB instance**, que corre el motor. Una DB instance puede contener **varias bases del mismo motor**, cada una con varias tablas.
- Bajo la DB instance hay una **instancia EC2**, pero se administra desde la consola de RDS, no la de EC2.
- Se elige tipo y tamaño (como en EC2); determina CPU y memoria disponibles, y no todos los tipos están disponibles para todos los motores. Tres familias:
  - **Standard** (propósito general).
  - **Memory optimized** (apps intensivas en memoria).
  - **Burstable performance** (rendimiento base con ráfagas hasta el 100 % de CPU).
- El almacenamiento son **volúmenes EBS**, en tres opciones: **General Purpose (SSD)**, **Provisioned IOPS (SSD)** y **Magnetic** (no recomendado).

### 8.3 RDS dentro de una VPC

- Al crear la instancia se elige la **VPC** y luego las subredes: un **DB subnet group**, que especifica las **AZ** y las **subredes** de esas AZ donde vivirá la base.
- Las subredes deben ser **privadas** (sin ruta al internet gateway), de modo que solo el **backend de la aplicación** alcance la base.
- El tráfico se restringe aún más con **network ACLs** y **security groups** (capas de seguridad).
- Para limitar **qué acciones y recursos** pueden usar los empleados sobre RDS se usan **IAM policies**.

### 8.4 Backups

| | **Automated backups** | **Manual snapshots** |
|---|---|---|
| Activación | **Activadas por defecto** | A demanda, en cualquier momento |
| Qué respaldan | **Toda la DB instance** (no bases sueltas) + **transaction logs** | La instancia, como un snapshot EBS pero gestionado en la consola RDS |
| Retención | **0 a 35 días** (0 = deshabilitar, y **borra los backups automáticos existentes**) | Hasta que se eliminan (sirve para > 35 días, p. ej. retención de 1 año por cumplimiento) |
| Ventana | Se define una **backup window**, idealmente en baja actividad (puede subir latencia y causar indisponibilidad) | — |
| Restauración | **Point-in-time recovery:** crea una **nueva** DB instance restaurando el backup completo y aplicando/retrocediendo transacciones hasta el instante elegido | Crea una **nueva** DB instance con los datos del snapshot |

> [!tip] 💡 ¿Cuál usar?
> Casi siempre **ambas**: las automáticas dan recuperación a un punto en el tiempo; las manuales dan retención mayor a 35 días.

### 8.5 Alta disponibilidad: RDS Multi-AZ

Una DB instance vive en una subred, que pertenece a **una** AZ: no cumple la buena práctica de producción de replicar en **al menos dos AZ**. Con **Multi-AZ**:

```
                Endpoint DNS (no cambia)
                        │
          ┌─────────────┴─────────────┐
   AZ A   ▼                           │   AZ B
  ┌───────────────┐  réplica  ┌───────────────┐
  │   PRIMARIA    │──────────▶│   STANDBY     │
  │ (atiende app) │ síncrona  │ (no consultada)│
  └───────────────┘           └───────────────┘
   si falla ──▶ failover automático: la standby pasa a primaria
```

- RDS crea una copia **standby** en otra subred de otra AZ y **replica de forma síncrona** desde la primaria.
- La standby **no es una base activa**: las aplicaciones no la consultan.
- Ante un problema de disponibilidad (por ejemplo, pérdida de conectividad con la primaria) RDS ejecuta un **failover automático**: promueve la standby y redirige las consultas usando el **nombre DNS** de la instancia. El **endpoint no cambia**, no hay que tocar código.
- La aplicación solo debe saber **reconectarse** tras una interrupción momentánea (refrescar búsquedas DNS cacheadas).
- Para no perder Multi-AZ tras el failover, RDS crea una nueva standby: **degradando la antigua primaria** si sigue viva, o **levantando una instancia nueva**.
- Por esto se pueden seleccionar varias subredes al crear la base: use subredes de **AZ distintas** para primaria y standby.

### 8.6 Demo: crear una base RDS (video)

1. Consola de RDS → **Create database**.
2. Elegir **Easy create** (acepta buenas prácticas estándar de backup y alta disponibilidad); **Standard create** daría control granular.
3. Motor: se muestran MySQL, PostgreSQL, MariaDB, Microsoft SQL Server y Amazon Aurora. Como la app solo necesita una base simple, se elige **MySQL**.
4. Tamaño y tipo de DB instance: una elegible para **free tier**.
5. Nombre de la base, usuario y contraseña de administrador.
6. Aceptar el resto de valores de Easy create y crear. La instancia tarda unos minutos en arrancar.
7. Con la instancia en marcha, se conectaría y se cargaría el esquema.

🎯 **Para el examen:**
- Multi-AZ = **alta disponibilidad** con standby **síncrona** que no atiende lecturas; failover por DNS.
- Retención de backups automáticos: máximo **35 días**; más allá, snapshots manuales.
- Subredes **privadas** para RDS; security groups para dejar entrar solo al backend.

---

## 9. Bases de datos *purpose-built*

Audiencia: 🧭 👔

**Idea central:** elegir la base que se ajusta a los requisitos del negocio, no forzar los datos a una base elegida de antemano. **No hay una base para todo.**

**El replanteo de la Employee Directory:** se había elegido RDS, pero:
- La app solo guarda **un registro por empleado** en una tabla, sin relaciones complejas: es esencialmente una **tabla de búsqueda (lookup table)**. Las capacidades relacionales agregan sobrecarga innecesaria.
- RDS cobra **por hora de instancia en ejecución**, se use o no; y la app se usa mucho entre semana y **nada los fines de semana**.
- **DynamoDB** encaja mejor: NoSQL para pares clave-valor o documentos, escala masiva, **latencia de milisegundos** y cobro **por uso de la tabla y datos leídos**, no por hora.

**Otros casos del video:**

| Necesidad | Servicio |
|---|---|
| Sistema de gestión de contenidos, catálogos, perfiles de usuario | **Amazon DocumentDB** |
| Red social (quién conecta con quién), motores de recomendación, detección de fraude | **Amazon Neptune** (graph) |
| Cadena de suministro sin pérdidas, registros bancarios o financieros con **100 % de inmutabilidad**; industrias auditadas | **Amazon QLDB** (Quantum Ledger Database): registro inmutable donde **ninguna entrada puede eliminarse** |

### 9.1 Portafolio de bases de datos de AWS (lectura 3.9)

| Tipo | Casos de uso | Servicio |
|---|---|---|
| Relacional | Apps tradicionales, ERP, CRM, e-commerce | Amazon RDS, Amazon Aurora, Amazon Redshift |
| Key-value | Apps web de alto tráfico, e-commerce, juegos | Amazon DynamoDB |
| In-memory | Caché, gestión de sesiones, leaderboards de juegos, apps geoespaciales | Amazon ElastiCache for Memcached, Amazon ElastiCache for Redis |
| Document | Gestión de contenidos, catálogos, perfiles de usuario | Amazon DocumentDB (compatible con MongoDB) |
| Wide column | Apps industriales a gran escala: mantenimiento de equipos, gestión de flotas, optimización de rutas | Amazon Keyspaces (for Apache Cassandra) |
| Graph | Detección de fraude, redes sociales, motores de recomendación | Amazon Neptune |
| Time series | IoT, DevOps, telemetría industrial | Amazon Timestream |
| Ledger | Sistemas de registro, cadena de suministro, registros, transacciones bancarias | Amazon QLDB |

**Descomposición de aplicaciones:** las aplicaciones grandes ya no se apoyan en una sola base; se dividen en **servicios más pequeños**, cada uno con su base específica. La «base única para todo» se reemplaza por una **estrategia de bases complementarias**, cada una con la funcionalidad, rendimiento y escala que su carga requiere.

**👔 En una frase para el negocio:** operar bases a escala exige mucha experiencia; usar el servicio administrado adecuado evita tener que construir esa pericia en casa para cada tipo de base.

---

## 10. Amazon DynamoDB

Audiencia: 🔧 🧭

### 10.1 Qué es

- Base **NoSQL totalmente administrada** y **serverless**: no se administran instancias ni infraestructura subyacente; olvida aprovisionamiento de hardware, instalación, replicación, parches o escalado de clústeres.
- No se crea una base con tablas relacionadas: se crean **tablas independientes**.
- Gestiona el almacenamiento sola, tenga 1 ítem o 2 millones; no hay que escalar el almacenamiento.
- Tablas que guardan **cualquier cantidad de datos** y sirven **cualquier nivel de tráfico**; la capacidad de throughput se sube o baja **sin downtime ni degradación**. Métricas y uso visibles en la consola.
- Distribuye datos y tráfico entre suficientes servidores; datos en **SSD** y **replicados automáticamente en varias AZ** de la Región (alta disponibilidad y durabilidad incorporadas).
- **Tiempo de respuesta de milisegundos**, clave para apps con millones de usuarios.
- **Cifrado en reposo** incluido.

### 10.2 Esquema flexible: SQL vs NoSQL

| | Relacional (p. ej. el MySQL creado antes) | DynamoDB (NoSQL) |
|---|---|---|
| Esquema | Rígido y bien definido, una o muchas tablas relacionadas | **Flexible**: se agregan o quitan atributos en cualquier momento; no todos los ítems tienen los mismos atributos |
| Variación de datos en una tabla | No admite | Admite ítems heterogéneos |
| Consultas | Complejas, entre tablas | Más simples, centradas en una colección de ítems de **una** tabla |
| Bajo estrés | Puede tener problemas de rendimiento y escalabilidad | Muy rápida y altamente escalable |
| Cuándo | Esquemas y relaciones complejas | Datos menos rígidos, acceso a ritmo alto |

> [!warning] ⚠️ Regla crítica
> DynamoDB es *purpose-built*: tiene casos de uso concretos y **no es la mejor opción para cualquier carga**.

### 10.3 Componentes

| Componente | Qué es | Equivalente relacional | Ejemplo |
|---|---|---|---|
| **Table** | Colección de ítems | Tabla | `People`, `Cars` |
| **Item** | Grupo de atributos identificable de forma única; **sin límite** de ítems por tabla | Fila, registro, tupla | Una persona, un vehículo |
| **Attribute** | Dato fundamental, que no necesita descomponerse más | Campo, columna | `PersonID`, `LastName`, `FirstName`; `DepartmentID`, `Name`, `Manager` |
| **Primary key** | Identifica de forma única cada ítem; hay que designar atributos como clave | Clave primaria | ID de empleado |
| **Secondary indexes** | Dan más flexibilidad de consulta | Índices | — |

### 10.4 Demo en video: cambiar RDS por DynamoDB

La aplicación estaba diseñada para usar **RDS o DynamoDB** como backend, así que el cambio es rápido:
1. Consola → DynamoDB → crear tabla nueva.
2. Designar el **ID de empleado** como clave que hace único a cada ítem.
3. Aceptar valores por defecto y crear. La app busca una tabla con nombre específico (empleados).
4. En el sitio alojado en EC2, agregar un empleado.
5. Volver a DynamoDB, refrescar y **escanear** los ítems: el registro aparece.

---

## 11. Demos y laboratorios

Audiencia: 🔧

### 11.1 Demo «Creación de un bucket de Amazon S3» (y conexión de la app)

**Objetivo:** crear el bucket de fotos, darle acceso solo al rol de la aplicación y lanzar una versión de la app que lo use.
**Servicios:** Amazon S3, IAM (rol ya existente), Amazon EC2.

1. Entrar a la consola como el **usuario administrador** creado en semanas anteriores.
2. Buscar **S3** → **Create bucket**.
3. Nombre: `employee-photo-bucket-<iniciales>-<3 dígitos>` (en el video, `employee-photo-bucket-sr-963`). Región **Oregón (`us-west-2`)**, la misma del resto de la infraestructura. Dejar el resto por defecto → **Create bucket**.
4. Probar una subida: abrir el bucket → **Upload** (botón superior derecho) → **Add files** → elegir una foto de empleado → **Upload** → **Close**.
5. No se hace público el bucket: debe ser accesible **solo por la aplicación**. Bucket → **Permissions** → **Bucket policy** → **Edit**.
6. Pegar la política que traen las instrucciones del ejercicio y reemplazar los marcadores: **número de cuenta** e **nombre del bucket** (este en **dos** lugares), quitando los signos `< >`. **Save changes**. Ahora el rol de la app tiene acceso al bucket y sus objetos.
7. Ir a **EC2 → Instances**; seleccionar la instancia detenida del ejercicio anterior → **Actions → Image and templates → Launch more like this** (clona la configuración).
8. Renombrar a `employee-directory-app-s3`; misma AMI y tipo de instancia; **mismo key pair**.
9. En red, **Auto-assign public IP → Enable**.
10. **Advanced details:** verificar que el **rol** ya está asociado; en **User data**, escribir el **nombre del bucket** para que la app sepa cuál usar → **Launch instance**.
11. Esperar a **2/2 status checks passed**, copiar la IP pública y abrirla en otra pestaña: la app responde, aunque aún no se puede interactuar porque falta la base de datos.
12. **Limpieza:** detener la instancia y borrar el objeto subido para no acumular cargos.

**Errores comunes:** olvidar reemplazar el nombre del bucket en ambos lugares de la política o dejar los `< >`; no habilitar la IP pública; bucket en otra Región.

### 11.2 Demo de RDS (video «Servicio de base de datos relacional de Amazon»)

Ver los pasos en la sección 8.6. Es una demostración de creación con Easy create; la app termina usando DynamoDB.

### 11.3 Demo «Creación de una tabla de Amazon DynamoDB»

**Objetivo:** dejar la aplicación completamente funcional con DynamoDB y comprobar que escribe en la tabla y en el bucket.
**Servicios:** Amazon EC2, Amazon DynamoDB, Amazon S3.

1. Primero lanzar una instancia lista para usar la base: **EC2 → Instances**, seleccionar `employee-directory-app-s3` (la versión más reciente) → **Actions → Image and templates → Launch more like this**.
2. Renombrar con sufijo `-dynamodb`; verificar **mismo key pair** y **mismo rol**.
3. **Network settings → Auto-assign public IP → Enable**. Comprobar que el **user data** conserva el nombre del bucket → **Launch instance**.
4. Esperar de *Initializing* a **2/2 checks passed**; abrir la IP pública para confirmar que la app base corre (si no, revisar el lanzamiento antes de seguir).
5. Buscar **DynamoDB** → **Create table**.
6. **Table name:** `Employees` (la app está programada para ese nombre). **Partition key:** `id`, tipo **String**. Resto por defecto → **Create table** (tarda segundos).
7. Sin cargar ítems a mano, abrir la app: directorio vacío → **Add** → nombre, ubicación, cargo, algunas insignias (en el video: residente de Seattle, fan de Seattle, «coffee snob») y una **foto** → **Save**. Aparece la entrada.
8. Verificar en **S3**: la foto está en el bucket (con un nombre asignado por la app).
9. Verificar en **DynamoDB → Explore table items**: el ítem trae insignias, nombre, cargo, ubicación, nombre del objeto en S3 y el `id` (partition key).
10. **Limpieza:** dejar la tabla, pero **detener la instancia** para no generar cargos antes de la siguiente etapa.

### 11.4 Laboratorio 3: «Configuración de una aplicación web» (calificado, sin texto)

Según la introducción al ejercicio: crear un **bucket S3** y una **bucket policy** que permita al **rol IAM** trabajar con los objetos; subir manualmente un objeto para familiarizarse con S3; crear una **tabla DynamoDB** para la información de empleados; y comprobar que la aplicación **lee y escribe** datos. Consejo del curso: si se atasca, retroceda algunos pasos en las instrucciones y revise que no se saltó nada.

### 11.5 Actividades sin texto

- *Coach* «Selección del servicio de almacenamiento de AWS adecuado» — práctica de la sección 6.
- *Coach* «Decisiones sobre arquitectura de bases de datos» — práctica de las secciones 7–10.
- **Cuestionario de la semana 3** (calificado).

---

## 12. Aplicación de ejemplo: estado al cierre del módulo

Audiencia: 🧭

| Pieza | Antes (tomo 02) | Ahora |
|---|---|---|
| Red | VPC con subredes | Sin cambios |
| Cómputo | EC2 con la app (no funcional) | EC2 clonada: `employee-directory-app-s3` → `employee-directory-app-dynamodb`, con IP pública, rol IAM y bucket en user data |
| Almacenamiento de fotos | — | **Bucket S3** `employee-photo-bucket-…` en `us-west-2`, privado, con **bucket policy** para el rol de la app |
| Datos de empleados | — | **Tabla DynamoDB** `Employees`, partition key `id` (String). RDS se evaluó y se descartó para este caso |
| Volumen raíz | EBS de la instancia | Sin cambios |

```
                         Región us-west-2 (Oregón)
 ┌────────────────────────────── VPC ─────────────────────────────┐
 │  Subred pública                                                 │
 │  ┌──────────────────────────────┐                               │
 │  │ EC2  employee-directory-app  │  rol IAM                      │
 │  │ (volumen raíz EBS)           │───────────┐                   │
 │  └──────────────┬───────────────┘           │                   │
 └─────────────────┼───────────────────────────┼───────────────────┘
                   │ fotos (s3:GetObject/Put)  │ ítems (lectura/escritura)
                   ▼                           ▼
        ┌──────────────────────┐    ┌───────────────────────┐
        │ S3 bucket            │    │ DynamoDB table        │
        │ employee-photo-...   │    │ Employees (PK: id)    │
        │ bucket policy → rol  │    │ serverless, multi-AZ  │
        └──────────────────────┘    └───────────────────────┘
   Usuario ──HTTP (IP pública)──▶ EC2
```

La aplicación queda **funcional**: alta de empleados con foto, que se guarda en S3, y datos en DynamoDB. Pendiente para los tomos siguientes: monitoreo, balanceo y escalado ([[04-Monitoreo-y-Optimizacion]]).

---

## ✅ Autoevaluación

> [!question]- 1. Una instancia con datos en instance store se detiene para cambiar su tipo. ¿Qué pasa con esos datos?
> Se pierden. El instance store está atado al ciclo de vida de la instancia: al detenerla o terminarla, los datos desaparecen. Para persistencia se usa EBS.

> [!question]- 2. Tiene un volumen EBS de 5 TB que se queda corto. ¿Qué opciones tiene?
> Aumentar su tamaño hasta el máximo de 16 TB (incluso sin detener la instancia) o adjuntar volúmenes adicionales a la misma instancia.

> [!question]- 3. ¿Por qué un snapshot de EBS de un volumen de 10 GB puede ocupar solo 2 GB?
> Porque los snapshots son incrementales: guardan solo los bloques cambiados desde el snapshot anterior. Se almacenan en S3, de forma redundante en varias AZ.

> [!question]- 4. Necesita mover un volumen EBS a otra AZ. ¿Cómo?
> Un volumen solo se adjunta a instancias de su misma AZ. Se toma un snapshot y desde él se crea un volumen nuevo en la AZ de destino.

> [!question]- 5. ¿Por qué el nombre de un bucket debe ser único a nivel global y compatible con DNS, si el bucket es regional?
> Porque AWS construye con ese nombre la URL de acceso por HTTP/HTTPS; dos cuentas no pueden compartir un mismo nombre y no se admiten espacios ni caracteres especiales.

> [!question]- 6. Otra cuenta de AWS necesita subir objetos a su bucket. ¿IAM policy o bucket policy?
> Bucket policy: es la forma simple de dar acceso cross-account sin crear roles IAM.

> [!question]- 7. Con versionado activado, alguien borra `foto.jpg`. ¿Se perdió?
> No. S3 coloca un delete marker; al eliminar ese marcador el objeto vuelve. Las versiones anteriores siguen cobrándose.

> [!question]- 8. Datos de backup secundario, fácilmente recreables, poco accedidos y lo más baratos posible con acceso inmediato. ¿Qué clase S3?
> S3 One Zone-IA: una sola AZ, 20 % más barata que Standard-IA; adecuada cuando no se necesita la resiliencia multi-AZ.

> [!question]- 9. ¿Puede la aplicación leer de la standby de RDS Multi-AZ para repartir carga?
> No. La standby no es una base activa y las aplicaciones no la consultan; existe para failover. La replicación es síncrona y el endpoint DNS no cambia al conmutar.

> [!question]- 10. Debe conservar backups de RDS un año por cumplimiento. ¿Basta con los automáticos?
> No: los automáticos retienen como máximo 35 días. Se requieren snapshots manuales, que persisten hasta que se eliminan.

> [!question]- 11. ¿Por qué la Employee Directory terminó en DynamoDB y no en RDS?
> Es una tabla de búsqueda sin relaciones complejas; RDS cobra por hora de instancia aunque no se use (y la app no se usa los fines de semana), mientras DynamoDB es serverless, escala sola y cobra por uso.

## 🧾 Resumen en 10 líneas

1. Tres tipos de almacenamiento: **file** (jerarquía, compartido, NAS), **block** (bloques direccionables, baja latencia, DAS/SAN) y **object** (plano, escalable, por identificador).
2. **Instance store**: bloque efímero en el host, rápido e incluido en el precio; se pierde al detener o terminar.
3. **EBS**: bloque persistente en red, hasta 16 TB, replicado en su AZ, casi siempre 1 volumen ↔ 1 instancia (Multi-Attach limitado).
4. EBS SSD (IOPS: Provisioned IOPS 64.000, General Purpose 16.000) para transaccional; HDD (MB/s: Throughput Optimized, Cold) para secuencial.
5. **Snapshots** EBS: incrementales, en S3, multi-AZ; permiten crear volúmenes en otra AZ.
6. **S3**: objetos ilimitados hasta 5 TB, 11 nueves de durabilidad, 99,99 % de disponibilidad; bucket regional con nombre global DNS-compatible; **privado por defecto**.
7. Acceso a S3 con **IAM policies** (centralizado) o **bucket policies** (cross-account, más tamaño); cifrado server/client-side; **versionado** con delete markers; **clases** y **lifecycle** para optimizar costo.
8. **EFS** (NFS) y **FSx** (Windows/SMB, Lustre) cubren el almacenamiento de archivos montable en varias instancias.
9. **RDS**: relacional administrado (Oracle, SQL Server, MySQL, PostgreSQL, MariaDB, Aurora), en subredes privadas, backups automáticos 0–35 días + snapshots manuales, **Multi-AZ** con standby síncrona y failover por DNS.
10. **Purpose-built**: no hay base única; **DynamoDB** (serverless, NoSQL, milisegundos, pago por uso) quedó como base de la Employee Directory, con fotos en S3.

## 📖 Recursos del módulo

- [AWS: Qué es el almacenamiento en la nube](https://aws.amazon.com/what-is-cloud-storage/)
- [AWS: Tipos de almacenamiento en la nube](https://aws.amazon.com/what-is-cloud-object-storage/#types)
- [Documentación de AWS: almacén de instancias](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html)
- [Documentación de EBS Multi-Attach](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes-multi.html)
- [AWS: Amazon Elastic Block Store (Amazon EBS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html)
- [AWS: Preguntas frecuentes sobre Amazon EBS](https://aws.amazon.com/ebs/faqs/)
- [AWS: Almacenamiento](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html)
- [AWS: Almacenamiento en la nube en AWS](https://aws.amazon.com/products/storage/)
- [Amazon EFS: cómo funciona](https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html)
- [Amazon FSx para Windows File Server](https://aws.amazon.com/fsx/windows/)
- [Amazon FSx para Lustre](https://aws.amazon.com/fsx/lustre/)
- [AWS: ¿Qué es una base de datos relacional?](https://aws.amazon.com/relational-database/)
- [AWS: Bases de datos en AWS](https://aws.amazon.com/products/databases/)
- [Preguntas frecuentes sobre Amazon Aurora](https://aws.amazon.com/rds/aurora/faqs/?nc=sn&loc=6)
- [AWS: Trabajar con copias de seguridad](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- [AWS: Copia de seguridad y restauración de Amazon RDS](https://aws.amazon.com/rds/details/backup/)
- [AWS: Creación y uso de una política de IAM para el acceso a la base de datos de IAM](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.IAMPolicy.html)
- [AWS: Amazon Virtual Private Cloud VPCs y Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html)
- [Cifrado en reposo de DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html)
- [Introducción a Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [AWS: Blog de bases de datos de AWS](https://aws.amazon.com/blogs/database/?nc=sn&loc=4)
- [AWS: Libertad de base de datos](https://aws.amazon.com/products/databases/freedom/?nc=sn&loc=5)

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[02-Computo-y-Redes|02 · Cómputo y Redes]] · Siguiente: [[04-Monitoreo-y-Optimizacion|04 · Monitoreo y Optimización ➡]]
