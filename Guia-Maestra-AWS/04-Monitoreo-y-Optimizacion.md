---
title: "Tomo 04 — Monitoreo y Optimización"
tags: [aws, solutions-architect, cloudwatch, elb, auto-scaling, alta-disponibilidad, serverless, ia-generativa]
curso: 1 · AWS Cloud Technical Essentials
modulo: "Semana 4: Supervisión y optimización"
audiencias: [tecnico, puente, ejecutivo]
tomo: 04
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 📈 Tomo 04 — Monitoreo y Optimización

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[03-Almacenamiento-y-Bases-de-Datos|03 · Almacenamiento y Bases de Datos]] · Siguiente: [[05-Backend-Web-Serverless|05 · Backend Web Serverless ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Al cerrar el tomo 03, la aplicación Employee Directory ya era funcional (EC2 + DynamoDB + S3), pero tenía dos defectos: **no se veía qué le pasaba** y **no podía crecer ni sobrevivir a la caída de su única instancia**. Este tomo resuelve ambos: primero la observabilidad con **Amazon CloudWatch** (métricas, dashboards, logs, alarmas) y luego la optimización con **Elastic Load Balancing** y **EC2 Auto Scaling**, hasta dejar una arquitectura multi-AZ, elástica y autorreparable. Cierra el curso 1 con un rediseño serverless de la misma app y una introducción a los servicios de IA generativa (Amazon Bedrock y Amazon Q).

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** qué nivel de disponibilidad (cuántos «nueves») pagar, cuántas instancias mínimas y máximas tolerar, qué alertas despiertan a alguien y cuáles se remedian solas, cuándo pasar de servidores a serverless.
> - **Costo de hacerlo mal:** enterarse de las caídas por los usuarios; un punto único de falla que tumba todo el servicio; sobreaprovisionar «por si acaso» (pagar capacidad ociosa) o subaprovisionar (perder clientes en el pico); un ASG sin máximo que dispara la factura.
> - **Pregunta que responde:** *¿cómo sé, antes que mis usuarios, que algo anda mal, y cómo hago que el sistema absorba la demanda y las fallas sin intervención humana?*

> [!tip] 💡 Analogía general: el tablero del auto, el semáforo inteligente y la flota de taxis
> CloudWatch es el **tablero del auto**: velocímetro (métricas), bitácora del viaje (logs) y luces de advertencia que se encienden solo si el problema persiste (alarmas). El load balancer es el **despachador de una central de taxis**: el pasajero llama a un solo número y el despachador asigna el taxi libre, sin que el cliente conozca la patente de ninguno. Auto Scaling es el **jefe de flota**: si la fila de pasajeros crece saca más taxis, si baja los guarda, y si uno se avería lo reemplaza; siempre dentro de un mínimo y un máximo que fija el dueño.

---

## Mapa del módulo

| Lección | Ítems (videos / lecturas / labs / quiz) | Qué se aprende |
|---|---|---|
| Monitorización en AWS | 3 videos · 2 lecturas (4.1, 4.2) | Por qué monitorear, métricas vs estadísticas, línea de base; CloudWatch: monitoreo básico/detallado, namespaces, dimensions, custom metrics, dashboards, CloudWatch Logs, alarmas y sus 3 estados |
| Optimización | 3 videos · 3 lecturas (4.3, 4.4, 4.5) | Disponibilidad en «nueves», redundancia multi-AZ, activo-pasivo vs activo-activo; ELB (ALB, NLB, GWLB), listeners/target groups/rules, health checks; escalado vertical vs horizontal; EC2 Auto Scaling (launch template, ASG, scaling policies) |
| Semana 4: ejercicio y evaluación | 2 videos · 1 lab · 1 quiz | Lab 4: alta disponibilidad para la app; demo completa de ALB + ASG + prueba de estrés |
| Sin servidor | 1 video · 1 actividad autoevaluada (Capstone) | Rediseño serverless de Employee Directory: S3 static hosting, API Gateway, Lambda, DynamoDB, Route 53, CloudFront |
| Servicios de IA generativa en AWS | 5 videos | IA generativa, LLM y foundation models; Amazon Bedrock (playgrounds, fine-tuning, knowledge bases, RAG); Amazon Q, Q Business, Q Developer (chat, inline, security scan) |
| Evaluación de final de curso | 1 examen · encuesta | Evaluación final del curso 1 (sin texto) |

---

## 1. Por qué monitorear

Audiencia: 🔧 🧭

El curso arranca con una escena: lunes por la mañana, las páginas del directorio cargan lento. Si el equipo se entera porque un usuario llama o abre un ticket, ya llegó tarde, y además el usuario solo puede describir **su experiencia**, no la causa. ¿Es la instancia EC2? ¿La base de datos? ¿Un despliegue de código reciente? Sin monitoreo, hay que excavar a ciegas.

**Monitorear es un verbo:** recolectar métricas, recolectar logs, observar el tráfico de red. El objetivo es reaccionar a los problemas operativos **antes de que el usuario final los note**, y en la nube esto es aún más necesario porque el entorno crece, se reduce y cambia constantemente.

**Vocabulario base:**

| Término | Qué es | Ejemplo del curso |
|---|---|---|
| **Métrica** | Un punto de datos individual que emite un recurso | Uso de CPU actual de una instancia EC2 |
| **Estadística** | Métricas recolectadas y analizadas en el tiempo | CPU promedio en el tiempo, con un peak a la 1:30 |
| **Log** | Registro de actividad en forma de archivo | VPC Flow Logs (tráfico que entra y sale de la VPC), logs de aplicación |
| **Línea de base (baseline)** | Lo que se considera comportamiento normal, construido con datos históricos | Si la métrica se desvía demasiado de la baseline, se dispara una alerta |

**Qué genera cada capa:**

```
  Red (VPC)        → flow logs: tráfico entrante/saliente
  Servidores (EC2) → CPU, red, disco, memoria*, logs de aplicación
  Base de datos    → conexiones simultáneas, CPU, espacio en disco (RDS)
  Almacenamiento   → tamaño del bucket, n.º de objetos, requests (S3)
```

\* La lectura menciona la memoria entre las métricas de EC2 junto a red, disco y logs de aplicación.

Cada servicio emite **métricas distintas**: un bucket S3 no tiene «CPU», pero sí tamaño total, número de objetos y métricas de requests (lecturas/escrituras); RDS emite conexiones, CPU de la instancia y consumo de disco.

**Cómo leer una métrica de CPU alta:** puede ser una avalancha legítima de solicitudes o un proceso con error consumiendo CPU. La regla es fijarse en el **cruce de un umbral durante un tiempo inusual**, y usar ese evento anómalo como señal para resolver (manual o automáticamente, por ejemplo escalando).

**Preguntas que el monitoreo responde** (lectura 4.1): cuánta gente visita el sitio por día y cómo evoluciona; si hay problemas de rendimiento o disponibilidad; qué pasa si la instancia se queda sin capacidad; si habrá alerta cuando el sitio caiga.

### 1.1 Beneficios del monitoreo

| Beneficio | Cómo se materializa |
|---|---|
| **Responder proactivamente** | Vigilar tasa de errores o latencia de requests en el tiempo para anticipar una caída y actuar antes |
| **Mejorar rendimiento y fiabilidad** | Ver el sistema completo saca a la luz cuellos de botella y arquitecturas ineficientes |
| **Reconocer amenazas y eventos de seguridad** | Con una baseline se detectan anomalías: picos de tráfico raros, IPs inusuales accediendo |
| **Decisiones de negocio basadas en datos** | Métricas de aplicación, p. ej. cuántos usuarios usan una función nueva (el ejemplo de la «app de fotos de gatos») para decidir si invertir más en ella |
| **Soluciones más rentables** | Detectar recursos subutilizados y ajustarlos (*right-sizing*) para no gastar de más |

> [!tip] 💡 Analogía
> Un médico no espera a que el paciente se desmaye: mira presión, pulso y temperatura (métricas), conoce los valores normales del paciente (baseline) y actúa cuando un valor se sale del rango durante un tiempo, no por una medición aislada.

> [!warning] ⚠️ Regla crítica
> Esperar a que el usuario final avise de una caída es explícitamente una **mala práctica** según el curso. La solución de monitoreo debe **centralizar** los datos en un solo lugar y permitir disparar **tareas automáticas** a partir de ellos.

---

## 2. Amazon CloudWatch

Audiencia: 🔧 🧭

**Amazon CloudWatch** es el servicio de **monitoreo y observabilidad** de AWS: centraliza métricas, logs, eventos y tráfico de los distintos servicios (EC2, RDS, DynamoDB, etc.) en una vista unificada. Es **administrado**: basta una cuenta de AWS, no hay infraestructura que gestionar.

**Para qué sirve (según la lectura 4.1):**
- Detectar comportamiento anómalo en los entornos.
- Configurar alarmas que avisen cuando algo no va bien.
- Visualizar logs y métricas en la AWS Management Console.
- Ejecutar acciones automáticas, como el escalado.
- Solucionar problemas (troubleshooting).
- Descubrir *insights* para mantener sanas las aplicaciones.

### 2.1 Monitoreo básico vs detallado

| Modalidad | Frecuencia | Costo | Activación |
|---|---|---|---|
| **Basic monitoring** | 1 punto de datos por métrica cada **5 minutos** | Gratis | Automática: muchos servicios envían métricas sin hacer nada |
| **Detailed monitoring** (EC2) | 1 punto de datos cada **1 minuto** | Tarifa adicional (ver página de precios de CloudWatch) | Se habilita en la instancia |
| **High-resolution custom metrics** | Hasta **1 segundo** (1 punto por segundo por métrica) | — (no se detalla) | Métricas personalizadas publicadas por usted |

Para muchas aplicaciones, el monitoreo básico «hace el trabajo».

### 2.2 Anatomía de una métrica: namespaces y dimensions

- Toda métrica tiene **timestamp**.
- Las métricas se agrupan en contenedores llamados **namespaces**; métricas de namespaces distintos están **aisladas** entre sí (piénselas como categorías).
- Los servicios adjuntan **dimensions** a cada métrica: un par **nombre/valor** que forma parte de la identidad de la métrica y sirve para **filtrar**. Ejemplo: pedir estadísticas de una instancia concreta especificando la dimensión `InstanceId`.

```
  Namespace: AWS/EC2
     └─ Métrica: CPUUtilization
          └─ Dimension: InstanceId = i-0abc...   ← filtra a UNA instancia
```

> [!note] ➕ Nota del editor
> El nombre `AWS/EC2` y el nombre literal `CPUUtilization` son la notación habitual de CloudWatch; en el video aparecen como «EC2 → Per-Instance Metrics → CPU Utilization».

### 2.3 Custom metrics

Las métricas **a nivel de aplicación** (que la instancia no publica por defecto) se envían como **custom metrics**. Ejemplo central: **número de páginas vistas** del sitio. Otros ejemplos del curso:
- Tiempos de carga de páginas web.
- Tasas de error de requests.
- Número de procesos o threads en la instancia.
- Cantidad de trabajo realizado por la aplicación.

Se publican de forma programática con la API **`PutMetricData`**.

> [!warning] ⚠️ La CPU no es la salud de la aplicación
> Por defecto, EC2 + CloudWatch solo muestran la salud **de la instancia**. La aplicación puede estar rota mientras la CPU luce perfecta. Para una visión holística hay que añadir **custom metrics** a los dashboards.

### 2.4 Dashboards

- Páginas de inicio **personalizables** en la consola de CloudWatch que muestran una o varias métricas mediante **widgets** (gráficos, texto).
- Se pueden crear muchos, cada uno con una vista distinta del entorno, e incluso **combinar datos de varias Regiones** en un solo dashboard para una vista global.
- CloudWatch **agrega las estadísticas** según el periodo que se elija al crear el gráfico. Opción de mostrar **datos en vivo** (publicados en el último minuto, aún no totalmente agregados).
- Se pueden **compartir** con el equipo.
- No es obligatorio visualizar solo en CloudWatch: herramientas externas pueden leer las métricas con la API **`GetMetricData`**.
- Acceso a ver/administrar dashboards: **IAM policies** asociadas a IAM users, groups o roles.

### 2.5 CloudWatch Logs

CloudWatch Logs centraliza el almacenamiento y análisis de logs de aplicaciones en EC2, funciones Lambda y otras fuentes. Permite **consultar y filtrar** (p. ej., buscar el *stack trace* de un error lógico conocido) y crear **metric filters**, que convierten datos de log en **métricas numéricas** graficables y alarmables.

**Esfuerzo de integración según el servicio:**

| Fuente | Qué hay que hacer |
|---|---|
| **AWS Lambda** | Mínimo: dar a la función los permisos IAM correctos para escribir en CloudWatch Logs |
| **EC2** | Instalar y configurar el **CloudWatch Logs agent** en la instancia |

**Componentes del agente** (según la lectura): un plugin de la AWS CLI que envía los datos de log; un script que inicia el proceso de envío; y un **cron job** que garantiza que el daemon esté siempre corriendo.

> [!note] ➕ Nota del editor
> La lectura describe el agente clásico de CloudWatch Logs. Hoy AWS recomienda el **CloudWatch agent** unificado (métricas + logs); verifique la documentación vigente antes de instalar.

**Terminología jerárquica de CloudWatch Logs:**

```
  Log group   ── misma retención y mismos permisos
   ├─ Log stream (instancia A) ── secuencia de eventos de UN recurso
   │     ├─ Log event: timestamp + mensaje
   │     └─ Log event ...
   └─ Log stream (instancia B)
```

| Nivel | Definición |
|---|---|
| **Log event** | Registro de una actividad: **timestamp + mensaje** |
| **Log stream** | Secuencia de log events que pertenecen al **mismo recurso** monitoreado (p. ej., una instancia EC2) |
| **Log group** | Conjunto de log streams que comparten **configuración de retención y permisos** (p. ej., los streams de todas las instancias de la app) |

### 2.6 CloudWatch alarms

Una alarma inicia acciones automáticamente ante **cambios de estado sostenidos** de una métrica. Para definirla se eligen tres cosas: **métrica**, **umbral** y **periodo de tiempo** durante el cual la métrica debe cruzar el umbral.

**Los tres estados:**

| Estado | Significado |
|---|---|
| `OK` | La métrica está dentro del umbral; todo normal |
| `ALARM` | La métrica está fuera del umbral; posible problema operativo |
| `INSUFFICIENT_DATA` | La alarma recién empieza, la métrica no está disponible o no hay datos suficientes |

Una alarma puede **dispararse al transitar entre estados** (cualquiera de ellos). Al dispararse puede ejecutar:
- una **acción de Amazon EC2** (p. ej., reiniciar la instancia),
- una **acción de Auto Scaling** (escalar hacia fuera o hacia dentro),
- una **notificación a Amazon SNS** (Simple Notification Service: se crea un *topic* y se envían mensajes a sus *subscribers*, p. ej. correos).

**Encadenamiento para auto-remediación:** alarma → notificación SNS → función **AWS Lambda** → la Lambda llama a cualquier API de AWS para corregir el problema.

> [!tip] 💡 Analogía
> Una alarma de CloudWatch es el detector de humo con retardo: no suena por una tostada quemada (pico efímero), sino cuando el humo persiste. El periodo es el retardo: muy corto genera falsas alarmas; muy largo, reacción tardía.

> [!example] 📊 Caso del curso — errores 500 en Employee Directory
> 1. Se configura un **metric filter** en CloudWatch Logs que cuenta respuestas con código **500**.
> 2. Se crea una alarma sobre esa métrica: pasa a `ALARM` si hay **más de 5 errores 500 por hora** de forma sostenida.
> 3. Acción: notificación por correo o texto (SNS) a la persona de guardia para que investigue antes de que el problema crezca.
> Variante automática: la alarma reinicia la instancia, escala el servicio o dispara una Lambda remediadora.

**Umbrales que aparecen en el material** (todos son ejemplos, no reglas):

| Contexto | Umbral | Periodo |
|---|---|---|
| Lectura 4.2 | CPU > 80 % | 5 minutos o más |
| Demo de CloudWatch (alarma) | CPU > 70 % | 5 minutos |
| Lectura 4.5 (simple scaling) | CPU > 65 % → +1 instancia | — |
| Lectura 4.5 (step scaling) | 85 % → +2; 95 % → +4 | — |
| Demo/lab de Auto Scaling (target tracking) | CPU promedio 60 % | warm-up 300 s |

> [!warning] ⚠️ Inconsistencia del material
> En la demo de CloudWatch el instructor anuncia al inicio un umbral de **60 %** para la alerta, pero al configurarla usa **70 %**. El valor efectivamente configurado es 70 %; el 60 % reaparece después como *target value* del Auto Scaling.

🎯 **Para el examen:**
- Basic monitoring = 5 min gratis; detailed monitoring = 1 min con costo; high-resolution custom metrics = 1 s.
- `PutMetricData` publica custom metrics; `GetMetricData` las lee desde herramientas externas.
- Metric filter convierte logs en métricas.
- Estados de alarma: `OK`, `ALARM`, `INSUFFICIENT_DATA`. Acciones: EC2, Auto Scaling, SNS.
- Logs de EC2 requieren agente; Lambda solo permisos IAM.
- Dimension = par nombre/valor que identifica y filtra la métrica (p. ej. `InstanceId`).

---

## 3. Disponibilidad y redundancia

Audiencia: 🔧 🧭 👔

### 3.1 Estado de partida

Al iniciar la lección de optimización, la arquitectura es: **una sola instancia EC2** en **una sola AZ**, DynamoDB como base de datos y S3 para activos estáticos (fotos). DynamoDB y S3 son **altamente disponibles por diseño**, así que el eslabón débil es la instancia: si cae, nadie puede usar la app. Es un **single point of failure**.

### 3.2 Los «nueves»

La disponibilidad se expresa como porcentaje de tiempo activo en un año, o como número de nueves:

| Disponibilidad | Nombre | Inactividad por año |
|---|---|---|
| 90 % | un nueve | 36,53 días |
| 99 % | dos nueves | 3,65 días |
| 99,9 % | tres nueves | 8,77 horas |
| 99,95 % | tres nueves y medio | 4,38 horas |
| 99,99 % | cuatro nueves | 52,60 minutos |
| 99,995 % | cuatro nueves y medio | 26,30 minutos |
| 99,999 % | cinco nueves | 5,26 minutos |

Más disponibilidad exige **redundancia**: más data centers, servidores, bases de datos y replicación, es decir, **más costo**. Hay que trazar la línea donde agregar redundancia deja de ser rentable frente a los ingresos.

> [!abstract] 👔 En una frase para el negocio
> Pasar de tres a cuatro nueves reduce la caída anual de ~9 horas a ~53 minutos, pero cada nueve adicional cuesta más infraestructura: la pregunta es cuánto vale una hora caída para el negocio.

### 3.3 Segunda instancia, en otra AZ

La solución al punto único de falla es **añadir un servidor**, pero **dónde** importa: la falla puede estar en el software (SO, aplicación) o en el hardware (servidor físico, rack, data center o **la AZ completa**). Poner la instancia B en **otra Availability Zone** cubre ambos tipos de falla.

### 3.4 Los tres desafíos de tener más de un servidor

| Desafío | Detalle | Solución sugerida |
|---|---|---|
| **Replicación** | Copiar configuración, parches y la propia aplicación entre instancias | **Automatizar** todo lo posible |
| **Redirección de clientes** | Cómo saben los clientes que hay varios servidores | **DNS** con un registro que apunte a todas las IPs (problema: la **propagación** de cambios tarda) o un **load balancer** (hace health checks, reparte la carga y evita el problema de propagación) |
| **Tipo de alta disponibilidad** | Activo-pasivo o activo-activo | Depende de si la app tiene estado |

### 3.5 Activo-pasivo vs activo-activo

| | Activo-pasivo | Activo-activo |
|---|---|---|
| Instancias atendiendo | Solo una a la vez | Ambas |
| Apps **stateful** (sesión en el servidor) | Sin problema: el cliente siempre llega al mismo servidor | Problema si la sesión no está en ambos servidores |
| Escalabilidad | Débil: requiere escalado vertical | Fuerte: el segundo servidor absorbe carga |
| Ideal para | Apps con estado | Apps **stateless** |

🎯 **Para el examen:** tabla de nueves (al menos 99,9 % ≈ 8,77 h; 99,99 % ≈ 52,6 min; 99,999 % ≈ 5,26 min); multi-AZ para eliminar el SPOF; activo-activo requiere app stateless.

---

## 4. Escalado vertical vs horizontal

Audiencia: 🔧 🧭

Más servidores mejoran la disponibilidad, pero el sistema puede volver a caer por **falta de capacidad** si llegan demasiadas solicitudes.

### 4.1 Escalado vertical (scale up) — el caso activo-pasivo

Si en activo-pasivo el servidor activo se satura, cae y (con suerte) conmuta al pasivo, que también se saturará: no resuelve nada. Hay que **agrandar el servidor** (tipo de instancia mayor o distinto), lo que en EC2 **solo puede hacerse con la instancia detenida**. Procedimiento descrito:

1. Detener la instancia **pasiva** (no afecta, no recibe tráfico).
2. Cambiar su tamaño o tipo y volver a iniciarla.
3. Mover el tráfico a la pasiva, que pasa a ser activa.
4. Detener, redimensionar e iniciar la antigua activa, para que ambas coincidan.

Cuando la demanda baja, hay que repetir todo. **Desventajas:** mucho trabajo manual y un **tope**: un servidor solo crece hasta cierto límite. Superado ese límite, la única salida es otro par activo-pasivo y repartir funcionalidades, lo que puede exigir **reescribir masivamente la aplicación**.

### 4.2 Escalado horizontal (scale out) — el caso activo-activo

Si la aplicación es **stateless** (no guarda sesiones de cliente en el servidor), tener 2 o 4 servidores no exige cambios en la app: basta crear instancias cuando hace falta y apagarlas cuando el tráfico baja. Sin el límite superior del escalado vertical. **Hacer la app stateless habilita la escalabilidad.**

| | Vertical | Horizontal |
|---|---|---|
| Qué cambia | Tamaño de la instancia | Número de instancias (flota) |
| Límite | Tope del tipo de instancia más grande | Sin ese tope |
| Interrupción | Requiere detener la instancia | No |
| Requisito de la app | Ninguno | Stateless |
| Automatizable con | — | **EC2 Auto Scaling** |

### 4.3 Escalado tradicional vs auto scaling

- **Tradicional (on-premises):** comprar servidores para el **peak**; de noche sobra capacidad y se desperdicia dinero (apagarlos solo ahorra electricidad).
- **Nube (pago por uso):** hay que apagar lo que no se usa, sobre todo las instancias On-Demand. Añadir/quitar servidores a mano en horarios previstos falla ante **picos inusuales**: o se sobreaprovisiona (desperdicio) o se subaprovisiona (clientes perdidos).
- **Solución:** una herramienta que añada y quite instancias **automáticamente** según condiciones definidas: **EC2 Auto Scaling**.

---

## 5. Elastic Load Balancing (ELB)

Audiencia: 🔧 🧭

### 5.1 Qué es y cómo fluye el tráfico

Load balancing = distribuir tareas (requests) entre un conjunto de recursos (instancias EC2). El algoritmo más popular es **round-robin** (a cada servidor por turnos). Con varias instancias ya no se usan sus IPs públicas: el cliente apunta al load balancer.

```
  Navegador ──► Load balancer ──► EC2 (AZ-a)
      ▲              │       └──► EC2 (AZ-b)
      └──────────────┘  (la respuesta vuelve por el LB: está EN el camino del tráfico)
```

Aunque se puede instalar un balanceador de software propio en EC2, AWS ofrece **Elastic Load Balancing**, sin administración ni operación.

**Características de ELB:**
- Distribuye tráfico entre **instancias EC2, contenedores, direcciones IP y funciones Lambda**.
- Al aceptar **IPs como destino**, puede operar en modo **híbrido** (balancear también hacia servidores on-premises).
- **Altamente disponible:** es un servicio **regional**; solo hay que desplegarlo en **varias AZ**. Aunque en el diagrama parezca una sola caja (¿SPOF?), AWS mantiene su disponibilidad, como con S3.
- **Escala automáticamente** con el tráfico entrante, sin configuración.

### 5.2 Tipos de load balancer

| Tipo | Tráfico | Uso |
|---|---|---|
| **Application Load Balancer (ALB)** | HTTP, HTTPS | Tráfico web; capa 7 (aplicación) del modelo OSI |
| **Network Load Balancer (NLB)** | TCP, UDP, TLS | Capa de conexión (capa 4) |
| **Gateway Load Balancer (GWLB)** | — | Principalmente balancear requests hacia **aplicaciones de terceros** |

Para Employee Directory se usa **ALB**, porque se balancea tráfico web.

### 5.3 Componentes: listener, target group, rule

```
  Cliente ─► LISTENER (puerto + protocolo) ─► RULE (condición) ─► TARGET GROUP ─► targets
             "lado cliente"                                       "lado servidor" + health check
```

| Componente | Función | Detalle |
|---|---|---|
| **Listener** | Escucha solicitudes | Se define con **puerto + protocolo** (p. ej. 80/HTTP, 443/HTTPS). Puede haber muchos por load balancer |
| **Target group** | Agrupa los backends | Tipos de target: instancias EC2, funciones Lambda, direcciones IP. **Cada target group necesita un health check** |
| **Rule** | Asocia listener con target group | Cada listener tiene una **regla por defecto**; se pueden añadir más. Ejemplo: tráfico a `/info` → target group B |

### 5.4 Health checks

Definir bien el health check es **crítico**:
- Verificar solo que el **puerto esté abierto** no prueba que la app funcione.
- Llamar a la **página de inicio** tampoco basta.
- Employee Directory depende de la base de datos y de S3: el chequeo debería validarlos. Propuesta del curso: una página **`/monitor`** que se conecte a la base de datos, lea datos y llame a S3; el health check del load balancer apunta a `/monitor`.

**Dos tipos de health check de ELB:**
1. **TCP:** abrir conexión con la instancia; si conecta, está disponible.
2. **HTTP/HTTPS:** pedir una página específica y validar que devuelva un código de respuesta HTTP.

**Parámetros (vistos en la demo final):**

| Parámetro | Valor en la demo | Qué controla |
|---|---|---|
| Healthy threshold | 2 | Chequeos **exitosos consecutivos** para declarar sano un target |
| Unhealthy threshold | 5 | Chequeos **fallidos consecutivos** para declararlo no sano |
| Timeout | 30 s | Tiempo sin respuesta tras el cual el chequeo falla |
| Interval | 40 s | Frecuencia de envío de chequeos |

**Ciclo de vida con Auto Scaling:**
- Instancia nueva → solo recibe tráfico cuando **pasa el health check**.
- Instancia que deja de funcionar → ELB deja de enviarle tráfico y **avisa a EC2 Auto Scaling**, que la elimina del grupo y la reemplaza.
- Scale-in por política → Auto Scaling avisa a ELB que dará de baja instancias; ELB puede **retrasar la terminación** hasta que terminen las conexiones existentes, sin aceptar nuevas: **connection draining** (también «deregistration delay»).

### 5.5 Application Load Balancer en detalle

| Característica | Detalle |
|---|---|
| **Enrutamiento según la request** | Ruta URL (`/upload`), host, cabeceras HTTP, método HTTP, IP de origen del cliente (y query string, según la tabla comparativa) |
| **Respuestas directas** | Respuesta fija (p. ej., página HTML personalizada) o **redirect** (p. ej., HTTP → HTTPS) sin tocar el backend |
| **TLS offloading** | Entiende HTTPS; requiere un certificado SSL importado vía **IAM** o **AWS Certificate Manager (ACM)**, o creado **gratis en ACM**; cifra el tramo cliente ↔ ALB |
| **Autenticación de usuarios** | Antes de dejar pasar: **OpenID Connect**, e integración con otros servicios para SAML, LDAP, Microsoft AD, etc. |
| **Protección de tráfico** | Security group que define los rangos de IP admitidos |
| **Algoritmo round-robin** | Mismo número de requests por servidor; sirve para la mayoría de apps |
| **Algoritmo least outstanding requests** | Para requests de complejidad variable o targets de capacidad distinta. *Outstanding* = enviada y aún sin respuesta. Con instancias de tamaños distintos, round-robin sobrecarga a la pequeña; este algoritmo iguala la utilización |
| **Sticky sessions** | Para apps stateful: una **cookie HTTP** recuerda a qué servidor enviar al cliente |

### 5.6 Network Load Balancer en detalle

| Característica | Detalle |
|---|---|
| **Protocolos** | TCP, UDP, TLS. HTTPS viaja sobre TCP y TLS, pero el NLB **no entiende** HTTP: no hay reglas por ruta, ni autenticación, ni least outstanding requests |
| **Algoritmo flow hash** | Basado en protocolo, IP y puerto de origen, IP y puerto de destino, y número de secuencia TCP. Si todo coincide, los paquetes van al mismo target |
| **Sticky sessions** | Basadas en la **IP de origen** del cliente, no en cookie |
| **TLS offloading** | Sí, similar al ALB |
| **Rendimiento** | Maneja **millones de requests por segundo al instante**; el ALB puede llegar, pero necesita escalar y eso toma tiempo |
| **IP estática y Elastic IP** | Sí: útil cuando el cliente no puede usar DNS o exige reglas de firewall por IP |
| **Preserva la IP de origen** | El backend ve la IP real del cliente (con ALB vería la IP del load balancer) |

### 5.7 ALB vs NLB (tabla del curso)

| Característica | ALB | NLB |
|---|:-:|:-:|
| Protocolos | HTTP, HTTPS | TCP, UDP, TLS |
| Connection draining (deregistration delay) | ✔ | |
| Direcciones IP como targets | ✔ | ✔ |
| IP estática y Elastic IP | | ✔ |
| Preserva IP de origen | | ✔ |
| Routing por IP de origen, ruta, host, cabeceras, método y query string | ✔ | |
| Redirects | ✔ | |
| Respuesta fija | ✔ | |
| Autenticación de usuarios | ✔ | |

> [!note] ➕ Nota del editor
> La tabla reproduce la del curso. En la documentación actual de AWS el NLB también admite *deregistration delay* en sus target groups; contraste con la página de comparación enlazada en Recursos si la pregunta es fina.

> [!tip] 💡 Analogía
> El ALB es un recepcionista que **lee la carta** (ruta, cabeceras, cookie) antes de derivarla al departamento correcto; el NLB es un clasificador postal que **solo mira el sobre** (IP, puerto, protocolo), pero procesa millones de sobres por segundo y deja intacto el remitente.

🎯 **Para el examen:**
- HTTP/HTTPS, routing por ruta/host, redirects, autenticación, sticky por cookie → **ALB**.
- TCP/UDP/TLS, IP estática/Elastic IP, preservar IP del cliente, millones de rps instantáneos → **NLB**.
- Appliances de terceros → **GWLB**.
- Internet-facing vs **internal** load balancer (ver §7).
- Connection draining protege conexiones durante el scale-in.

---

## 6. Amazon EC2 Auto Scaling

Audiencia: 🔧 🧭

EC2 Auto Scaling **añade o elimina capacidad** para mantener un rendimiento estable y predecible **al menor costo posible**: se paga solo lo que la app necesita. Además hace **gestión de flota**: si una instancia falla, la reemplaza. Por eso sirve tanto para **escalar** como para **alta disponibilidad**, incluso con cargas constantes.

**Tres componentes, tres preguntas:**

| Componente | Pregunta | Contenido |
|---|---|---|
| **Launch template** (o launch configuration) | **¿Qué** se escala? | AMI, tipo de instancia, security group, volúmenes EBS, key pair, IAM role, user data… |
| **Auto Scaling group (ASG)** | **¿Dónde** (y cuántas)? | VPC, subnets (≥ 2 AZ), capacidad mínima/deseada/máxima, tipo de compra, load balancer |
| **Scaling policies** | **¿Cuándo** se añade o quita? | Simple, step o target tracking, apoyadas en métricas y alarmas de CloudWatch |

### 6.1 Launch templates

- Guardan todos los parámetros para crear instancias; sirven para lanzar a mano **o** con Auto Scaling.
- Admiten **versionado**: rollback rápido y una **versión por defecto** que otros siguen usando mientras usted itera sobre una nueva.
- **Tres formas de crearlas:**
  1. A partir de una **instancia EC2 existente** (la más rápida: todo ya está definido).
  2. A partir de otra plantilla o de una **versión anterior**.
  3. **Desde cero:** AMI ID, tipo de instancia, key pair, security group, almacenamiento y resource tags.

> [!warning] ⚠️ Launch configuration vs launch template
> La **launch configuration** es la alternativa antigua: **no permite versionar** usando otra configuración como plantilla ni crearse desde una instancia existente. El curso recomienda **usar launch templates** para obtener además las últimas características de EC2.

### 6.2 Auto Scaling group

- Define **dónde** se lanzan las instancias: **VPC y subnets**. Elegir **al menos dos subnets en AZ distintas**.
- Permite el **tipo de compra**: solo On-Demand, solo Spot o una **combinación**, aprovechando Spot con mínima carga administrativa.
- **Capacidades:**

| Ajuste | Significado |
|---|---|
| **Minimum** | Piso: nunca baja de aquí aunque se cumpla el umbral de reducción. Un mínimo de **2** es buena idea para alta disponibilidad |
| **Maximum** | Techo: nunca sube de aquí aunque se cumpla el umbral de aumento. Clave para **no exceder el presupuesto** |
| **Desired capacity** | Instancias que debería haber; se crea con este número y debe estar entre el mínimo y el máximo (o igual a ellos). Si baja, se elimina **la instancia más antigua** por defecto; si sube, se crean nuevas con la launch template |

```
  min=2 ───────── desired=2 ─────────────── max=4
   │ piso HA        │ arranque                │ techo de costo
   └── scale-in se detiene aquí               └── scale-out se detiene aquí
```

**Modo gestión de flota:** si min = max = desired (p. ej., **4**), el ASG no escala dinámicamente, pero **reemplaza** cualquier instancia que falle para mantener siempre 4.

### 6.3 Scaling policies

Por defecto el ASG se queda en su desired capacity inicial (se puede cambiar a mano). Las políticas usan **métricas y alarmas de CloudWatch** para saber cuándo actuar.

| Política | Cómo funciona | Limitación / ventaja |
|---|---|---|
| **Simple scaling** | Una alarma de CloudWatch y una acción: añadir/quitar N instancias, fijar la capacidad deseada, o un **porcentaje del grupo** (crece/encoge más rápido). Tras actuar, espera un **cooldown** (las instancias tardan en arrancar y la alarma puede seguir sonando). Ej.: +1 instancia si CPU > 65 % | Un solo escalón: si la CPU llega a 85 %, +1 puede ser insuficiente |
| **Step scaling** | Varios escalones; **responde a alarmas adicionales** incluso con una actividad de escalado o reemplazo en curso. Ej.: +2 a 85 %, +4 a 95 % | Más fino, pero hay que diseñar las alarmas |
| **Target tracking** | Usted da un **valor objetivo** (CPU promedio, red promedio de entrada/salida o **request count**) y AWS **crea las alarmas automáticamente** | La más simple; la recomendada para esas métricas |

> [!tip] 💡 Analogía
> Target tracking es el **termostato**: usted fija 60 % y el sistema prende o apaga calefactores (instancias) para mantenerse ahí. Simple y step scaling son interruptores manuales con reglas «si hace X grados, prende Y estufas».

> [!danger] 🚨 Error costoso
> Para limpiar un entorno con Auto Scaling, **elimine el Auto Scaling group**, no las instancias. Si borra las instancias, el ASG lanzará otras para volver a la capacidad deseada (y seguirá cobrando).

### 6.4 Integración ELB + Auto Scaling

Cuando el ASG añade o quita una instancia, **ELB recibe una notificación**; antes de enviarle tráfico valida con el **health check** que la aplicación responda. Al crear el ASG se marca **«Enable ELB health checks»**, para que el reemplazo se base en la salud vista por el load balancer (no solo en el estado de la instancia).

🎯 **Para el examen:**
- Launch template = qué; ASG = dónde/cuántas; scaling policy = cuándo.
- Launch template > launch configuration (versionado).
- Min/desired/max; scale-in elimina la instancia más antigua por defecto.
- Target tracking para CPU, red o request count; step scaling cuando se necesitan escalones; simple scaling espera cooldown.
- ASG en ≥ 2 AZ; On-Demand + Spot combinables.

---

## 7. Demos y laboratorios

Audiencia: 🔧

### 7.1 Demo: dashboard y alarma en CloudWatch

**Objetivo:** dashboard con la CPU de la instancia EC2 de Employee Directory y alerta por correo si la CPU supera el umbral durante un periodo. **Servicios:** CloudWatch, EC2, SNS.

**Dashboard:**
1. Consola → **CloudWatch** → **Dashboards** → **Create dashboard**, nombre `mydashboard`.
2. Elegir widget **Line** (gráfico de líneas) y fuente de datos **Metrics**.
3. Navegar por servicio: **EC2 → Per-Instance Metrics → CPUUtilization** de la instancia de la app.
4. **Save**: el dashboard muestra la CPU en el tiempo. Puede incluir recursos de **varias Regiones** y compartirse con el equipo.

**Alarma:**
1. **Alarms → Create alarm → Select metric → EC2 → Per-Instance Metrics → CPUUtilization**.
2. Periodo: **5 minutos** (equilibrio entre no reaccionar tarde y no reaccionar a cada pico efímero).
3. **Static threshold:** mayor que **70**.
4. Acción en la transición **OK → ALARM**: **Create new SNS topic** `CPU_Utilization_Topic` con una dirección de correo como suscriptor. (Se mencionan otras acciones posibles: EC2, Auto Scaling.)
5. **Next** → nombre y descripción → **Create alarm**.

**Resultado:** la alarma parte en `INSUFFICIENT_DATA` y tarda un rato en reunir datos y pasar a `OK`.

**Errores comunes:** confiar solo en la CPU (no mide la salud de la app); periodos demasiado cortos (falsas alarmas).

> [!note] ➕ Nota del editor
> SNS envía un correo de confirmación de suscripción; hasta que el destinatario no lo confirma, no recibe las alertas.

### 7.2 Demo: Application Load Balancer (lección de optimización)

**Objetivo:** repartir el tráfico entre dos instancias de la app ubicadas en **subnets privadas** de dos AZ. **Servicios:** EC2, ELB.

1. Consola **EC2** (ELB vive dentro de ella) → panel lateral **Load Balancers → Create load balancer** → **Application Load Balancer**.
2. Nombre: `app-elb`. Esquema: **Internet-facing**.
   - *Internet-facing:* enruta requests desde internet a los targets.
   - *Internal:* clientes con IP privada → targets con IP privada (p. ej., en una app de tres capas, del web tier al app tier).
3. Seleccionar la **VPC**, **ambas AZ** y las **dos subnets públicas**.
4. Security group que permite **puerto 80 desde cualquier lugar**.
5. Listener por defecto **HTTP:80** (se podría añadir HTTPS con **Add listener**).
6. **Create target group:** tipo **Instances**, nombre `app-target-group`, valores por defecto; registrar las dos instancias de subnets privadas → **Include as pending below** → **Create target group**.
7. Volver, refrescar, seleccionar el target group → **Create load balancer**.
8. Copiar el **DNS name** del ALB en una pestaña nueva: aparece la app. En **`/info`** se ve en qué AZ está la instancia que responde; al refrescar varias veces se alterna entre las dos AZ.

### 7.3 Demo: EC2 Auto Scaling con prueba de estrés (Morgan y Seth)

**Objetivo:** escalar horizontalmente de forma automática cuando la CPU sube. **Servicios:** EC2, Auto Scaling, CloudWatch, ALB.

**Funcionamiento planteado:** el tráfico llega a cualquiera de las instancias, todas idénticas y miembros de un ASG; un código de estrés hace creer a las instancias que están sobrecargadas; reportan su CPU a CloudWatch; CloudWatch entra en `ALARM` y pide más instancias al Auto Scaling; las nuevas pasan el health check del ALB y empiezan a recibir tráfico.

**Launch template:**
1. EC2 → **Launch Templates → Create launch template**; nombre `app-launch-template`, descripción «servidor web para la aplicación Employee Directory».
2. Marcar la casilla **Auto Scaling guidance** (se usará con EC2 Auto Scaling).
3. AMI **Amazon Linux**, tipo **t2.micro** (iguales a la instancia original).
4. Security group: el **web security group** creado antes en el curso.
5. **Advanced details:** IAM **instance profile** con el rol de la app y pegar el **user data** (descarga y descomprime el código fuente y arranca la app).
6. **Create**.

**Auto Scaling group:**
1. **Auto Scaling Groups → Create Auto Scaling group**; nombre `app-asg`; launch template `app-launch-template`.
2. Red: VPC `app-vpc`, subnets **Private A** y **Private B**.
3. **Attach to an existing load balancer** → elegir el target group; activar **ELB health checks**.
4. Capacidad: **min 2** (una por AZ), **max 4**, **desired 2**.
5. Scaling policy: **Target tracking**, métrica de utilización de CPU, **target value 60 %**, **300 s** de *warm-up* de instancias antes de contarlas en el escalado.
6. Notificaciones: opcionales (se omiten). Revisar → **Create Auto Scaling group**.

**Prueba:**
1. Abrir el endpoint del ALB → página **`/info`** → botón **Stress CPU** (en la vida real se usaría una herramienta de pruebas de carga) → **10 minutos**.
2. En CloudWatch se ve el resumen de alarmas: se superó el 60 % en todas las instancias y se lanzaron **2 instancias nuevas**; luego la CPU promedio baja porque hay más hosts recibiendo tráfico.
3. En **Target groups → app target group → Targets**: **4 instancias healthy**.
4. Si se espera, al bajar la carga el ASG termina instancias una a una hasta volver al **mínimo de 2**, sin intervención humana.

> [!danger] 🚨 Recordatorio del curso
> Al limpiar, borrar el **ASG** (no solo las instancias) para que no las recree.

### 7.4 Laboratorio 4: Configuring High Availability for Your Application

**Ítem sin texto en el material** (lab calificado). Según la introducción en video, el lab pide:
1. Revisar una instancia EC2 existente y validar su configuración.
2. Con esa información, crear una **launch template** para EC2 Auto Scaling.
3. Crear un **Application Load Balancer**.
4. Configurar un **Auto Scaling group** con la launch template, para escalar según la demanda.
5. Usar la **función de estrés** integrada en la app para simular carga y **validar que el escalado ocurrió**.

### 7.5 Demo final: walkthrough del ejercicio (balanceo + Auto Scaling)

**Objetivo:** reproducir el lab completo con el usuario **Admin** creado en semanas anteriores. Región **us-west-2** (Oregón).

**1. Nueva instancia de la app:**
1. EC2 → Instances → seleccionar la última instancia, `employee-directory-app-dynamodb` → **Actions → Image and templates → Launch more like this**.
2. Cambiar el sufijo del nombre de `dynamodb` a **`lb`**; mantener el mismo **key pair**; **habilitar IP pública** (para probarla).
3. En Advanced details, verificar el **rol de instancia** y el **user data** → **Launch instance**.
4. Esperar **2/2 status checks**; abrir la **IPv4 pública**: la app funciona y muestra los datos previos, porque accede al mismo bucket S3 y a la misma tabla DynamoDB.

**2. Application Load Balancer:**
1. EC2 → sección **Load Balancing → Load Balancers → Create load balancer → Application Load Balancer → Create**.
2. Nombre `app-elb`, **Internet-facing**.
3. **Network mapping:** `app-vpc`; AZ **us-west-2a** y **us-west-2b** con sus **subnets públicas**.
4. **Create new security group:** `load-balancer-sg`, descripción de acceso HTTP, VPC `app-vpc`, regla de entrada **HTTP desde cualquier lugar** → crear, refrescar y seleccionarlo (en lugar del default).
5. **Listeners and routing → Create target group:** tipo **Instances**, nombre `app-target-group`, VPC correcta; **Advanced health check settings:** healthy threshold **2**, unhealthy threshold **5**, timeout **30 s**, interval **40 s**.
6. Registrar la instancia nueva → **Include as pending below** → **Create target group**.
7. Refrescar el desplegable, elegir `app-target-group` → **Create load balancer** → esperar estado **Active** (un par de minutos).
8. Copiar el **DNS name** y abrirlo: el ALB enruta a la instancia con los datos cargados.

**3. Launch template:**
1. EC2 → Instances → **Launch Templates → Create launch template**: `app-launch-template`, descripción «servidor web para la aplicación Employee Directory», marcar **Auto Scaling guidance**.
2. AMI: opción **Currently in use** (toma la de la instancia en ejecución).
3. Tipo **t2.micro** (elegible para Free Tier, mantiene bajo el costo); mismo key pair; security group **web security group**.
4. Advanced details: **instance profile** con el rol que accede a los recursos; pegar el **user data** ajustando el **nombre del bucket** de la app y la **región us-west-2**.
5. **Create launch template**.

**4. Auto Scaling group:**
1. **Auto Scaling → Auto Scaling Groups → Create Auto Scaling group**: `app-asg`, launch template `app-launch-template`.
2. Red: `app-vpc`, **Public Subnet 1** y **Public Subnet 2**.
3. **Attach to an existing load balancer** → `app-target-group`; health checks a cargo del **ELB**.
4. Capacidad: **desired 2, min 2, max 4** (nunca menos de 2; si una no está sana, la reemplaza; con carga, hasta 4).
5. **Target tracking scaling policy:** CPU promedio **60**, warm-up **300 s** (5 minutos para calentar y pasar health checks antes de otra acción de escalado).
6. Notificaciones: se omiten (se podría usar un topic SNS con su correo para cada acción de escalado). Revisar → crear.

**5. Verificación y prueba:**
1. **Load Balancing → Target Groups → Targets:** el ASG lanzó **2 instancias adicionales, una por AZ**, ya healthy.
2. En la app, **`/info`**: al refrescar alterna entre instancias y AZ.
3. **Stress CPU** por **10 minutos** para superar el 60 %.
4. En **Instances** aparecen **2 instancias más** lanzadas por la acción de escalado, que se suman a la flota.

> [!warning] ⚠️ Diferencia entre demos
> En la demo de la lección de optimización el ASG usa las **subnets privadas** (Private A/B), mientras que en el walkthrough final usa las **subnets públicas** (Public Subnet 1/2). El diseño final que describe el curso (§9) ubica las instancias en **subnet privada** detrás del ALB, que es la práctica más segura.

### 7.6 Cuestionario de la semana 4, Capstone y evaluación final

- **Cuestionario de la semana 4:** ítem calificado sin texto.
- **Actividad autoevaluada: Proyecto Capstone:** ítem sin texto en el material.
- **Evaluación de final de curso:** examen sin texto.

---

## 8. Rediseño serverless de Employee Directory

Audiencia: 🔧 🧭 👔

### 8.1 Mantenimiento de la arquitectura basada en EC2

La arquitectura lograda funciona, pero exige: revisar que las **políticas de Auto Scaling** respondan a las expectativas (ajustes con el tiempo), **parchear** y actualizar EC2, y vigilar **nuevos tamaños o tipos de instancia** para optimizar. Hay muchas formas válidas de arquitecturar en AWS; depende de **para qué se optimiza**.

### 8.2 Separar presentación y aplicación

Employee Directory es una app **de tres capas**:

| Capa | Qué es | Dónde estaba | Dónde queda |
|---|---|---|---|
| Presentación | UI: HTML, CSS, JavaScript | EC2 (servidor web) | **Amazon S3** (static website hosting) |
| Aplicación | Lógica de negocio: ver, añadir, actualizar, eliminar empleados | EC2 | **AWS Lambda** expuesta con **Amazon API Gateway** |
| Datos | Base de datos y fotos | DynamoDB + S3 | **Sin cambios** |

- Separarlas evita que las instancias se sobrecarguen atendiendo dos tipos de requests a la vez.
- **¿S3 sirve para un sitio dinámico?** Sí: el sitio es estático, pero el **JavaScript** hace requests HTTP y modifica la página con los resultados.
- El front-end **no habla directo** con el back-end: se expone una **API** en API Gateway, con un **método por acción** sobre empleados. Puede haber una Lambda para todo o una por acción.
- Accesos entre servicios: **IAM roles**.
- Gracias al diseño **modular**, se cambiaron presentación y aplicación **sin tocar la capa de datos**.

**Servicios añadidos para explorar:** **Amazon Route 53** (gestión de nombres de dominio) y **Amazon CloudFront** (caché de los activos estáticos cerca de los usuarios en las **edge locations**).

### 8.3 Flujo de una consulta («ver todos los empleados»)

```
 Usuario ─(dominio)─► Route 53 ─► dirección del sitio estático
    │
    ▼
 Navegador ◄── HTML/CSS/JS ── S3 (vía CloudFront)
    │  JavaScript: llamada a la API
    ▼
 API Gateway ─ valida ─► Lambda ─ consulta ─► DynamoDB
    ▲                                            │
    └──────────── datos ◄────────────────────────┘
 JavaScript renderiza la respuesta en la página.  Fotos: S3.  Permisos: IAM roles.
```

### 8.4 Qué se gana

- Optimizada para **escalabilidad**, **menor carga operativa** y, según el uso, **costo**.
- **Sin parches ni gestión de AMI** con Lambda.
- **No hizo falta crear VPC, subnets, security groups ni network ACLs**: la red la gestiona AWS (aunque se puede integrar con una VPC por cumplimiento).
- Otra alternativa sería rediseñar con **servicios de contenedores**, y el diagrama cambiaría otra vez.
- Como todo en AWS es una **llamada a API**, el proceso se puede automatizar.

> [!tip] 💡 Analogía
> Pasar de EC2 a serverless es como pasar de tener un restaurante propio (local, cocina, mantención) a cocinar en una cocina compartida que se arrienda por plato: usted pone la receta (código) y solo paga cuando sale un pedido.

> [!danger] 🚨 Recordatorio de cierre del curso
> Borrar **todos los recursos** creados en la cuenta propia para no incurrir en costos.

---

## 9. Aplicación de ejemplo: estado al cierre del módulo

Audiencia: 🔧 🧭

**Lo que se agregó en la semana 4:** dashboard y alarma de CloudWatch con topic SNS; segunda (y luego más) instancias en otra AZ; **Application Load Balancer** internet-facing en subnets públicas con listener HTTP:80, target group y health checks; **launch template** `app-launch-template`; **Auto Scaling group** `app-asg` (min 2 / desired 2 / max 4) con **target tracking al 60 % de CPU**.

**Arquitectura final (descrita en el video de cierre):** múltiples instancias EC2 en **subnets privadas** de una VPC, miembros de un **EC2 Auto Scaling group**, con tráfico distribuido por un **ALB**; datos en **DynamoDB**, imágenes en **S3**.

```
                         Internet
                            │
                 ┌──────────▼───────────┐
                 │  ALB  app-elb (HTTP:80)│  load-balancer-sg
                 └───┬──────────────┬───┘
   VPC app-vpc       │ target group │ app-target-group (health checks)
  ┌──────────────────┼──────────────┼──────────────────┐
  │  AZ us-west-2a   │              │   AZ us-west-2b   │
  │  ┌─────────────┐ │              │ ┌─────────────┐   │
  │  │ EC2 (app)   │◄┘              └►│ EC2 (app)   │   │
  │  └─────────────┘   Auto Scaling   └─────────────┘   │
  │        ...         group app-asg        ...         │
  │                 min 2 · desired 2 · max 4           │
  └────────────────────────┬────────────────────────────┘
            IAM role       │         CloudWatch: CPU → target tracking 60 %
            ┌──────────────┴──────────────┐            alarma → SNS (correo)
            ▼                             ▼
        DynamoDB (empleados)          S3 (fotos)
```

**Evolución propuesta (serverless):** Route 53 → CloudFront + S3 (front-end) → API Gateway → Lambda → DynamoDB / S3.

---

## 10. IA generativa en AWS

Audiencia: 🧭 👔 🔧

### 10.1 Conceptos

- **IA generativa:** IA que **crea contenido nuevo** (conversaciones, historias, imágenes, video, música…) cuando se le pide, basándose en los datos con que se entrenó el modelo. Usos citados: asistentes virtuales, detección de fraude, chatbots de servicio al cliente, generación de código. En una app de chat, la entrada es un **prompt** en lenguaje natural y la salida es contenido generado.
- **LLM (large language models):** aplicación de la IA generativa especializada en **lenguaje natural**; entrenados con conjuntos de datos enormes; buenos para escribir, resumir, traducir.
- **Foundation models (FM):** modelos de ML masivos **preentrenados para fines generales** que conviene **ajustar** con datos propios para tareas especializadas.

### 10.2 Amazon Bedrock

Servicio **totalmente administrado** y **serverless** para acceder vía **API** a foundation models de empresas líderes de IA (al momento de la grabación: **Amazon, Anthropic, Cohere, Meta** y otras). Los modelos difieren en capacidades: hay que elegir el adecuado al caso. No hace falta construir un modelo propio; se experimenta con unos clics o llamadas a la API.

> [!note] ➕ Nota del editor
> La transcripción automática dice que Bedrock da acceso a «máquinas virtuales de alto rendimiento»; el sentido correcto es **foundation models (FM)** de alto rendimiento (confusión de la sigla «FMs»). Lo mismo ocurre con la descripción de los modelos Titan.

- **Familia Amazon Titan:** foundation models creados por AWS, de propósito general.
- **Personalización:**

| Opción | Datos | Qué logra |
|---|---|---|
| **Continued pre-training** | Datos nuevos **sin etiquetar** | Sigue entrenando el modelo con información nueva |
| **Fine-tuning** | Dataset **etiquetado** de la tarea | Mejora el rendimiento en tareas específicas |
| **Knowledge bases** | Fuentes de datos propias | Repositorio que **aumenta** prompts y respuestas con información propietaria vía **RAG** |

- **RAG (Retrieval Augmented Generation):** recupera datos de las fuentes de la empresa y enriquece el prompt para respuestas más relevantes y precisas; ayuda a reducir **alucinaciones** (información inexacta o inventada). Por detrás usa una **base de datos vectorial**: el texto se convierte en **vectores** (representaciones numéricas) y se recupera por **búsqueda de similitud**.
- **Playgrounds** en la consola: **chat**, **text** e **image**. Demo: se selecciona **Amazon Titan Text G1**, se inicia un chat y se pide un script Python que lea archivos locales de un directorio y los suba a S3; el modelo devuelve código de ejemplo. Se puede iterar el prompt, cambiar configuraciones del modelo y practicar *prompt engineering*.
- Para integrar en aplicaciones: llamar a las **APIs de Bedrock** con los **AWS SDK** y los parámetros correctos.

### 10.3 Amazon Q

Asistente de IA generativa **orientado al trabajo** y adaptable a la empresa, **construido sobre Amazon Bedrock**, con «más de 17 años» de conocimiento de AWS.

- **Qué hace:** generar código y pruebas, diagnosticar problemas de soluciones AWS, guiar en arquitecturas y buenas prácticas, preguntas y respuestas generales o de negocio, y **planificar y razonar en varios pasos** para transformar e implementar código.
- **Experto en:** AWS en general, **Amazon QuickSight**, **Amazon Connect** y (anunciado) la cadena de suministro de AWS; y en **su negocio**, conectándose a fuentes privadas.
- **Dónde se usa:** AWS Management Console, documentación de AWS, el **IDE** y la **CLI**.
- Mantiene el **contexto de la sesión** (preguntas de seguimiento).
- En la consola responde, p. ej., «¿formas de crear una app web en AWS?» con resumen y **enlaces** a documentación para verificar.
- **Troubleshoot with Amazon Q:** ante errores en la consola (permisos de EC2, configuración de S3) propone una solución; si no puede, ayuda a **abrir un caso con AWS Support**.
- **Integraciones:** QuickSight (generar visuales y dashboards), Connect (detectar la intención del cliente en llamadas y chats para ayudar al agente), **Amazon Redshift query editor** (lenguaje natural → recomendaciones de **SQL**).

**Amazon Q Business:** aplicaciones que se conectan a los datos de la empresa y responden a empleados **respetando los permisos** de las fuentes y **con citas** (mesa de ayuda TI, RR. HH., beneficios). Fuentes citadas: Amazon Kendra, S3, Microsoft SharePoint, Salesforce, bases en AWS vía RDS, Confluence, Microsoft Teams, Slack, Jira, entre otras. Ejemplo: preguntar qué características del producto generan más problemas a partir de los comentarios de clientes. **Plugins** prediseñados y personalizados para consultar datos en tiempo real (precios de acciones, saldo de vacaciones, ubicación de paquetes) y ejecutar acciones (solicitar días libres, enviar invitaciones).

### 10.4 Amazon Q Developer

Asistente de codificación con IA generativa para todo el ciclo de desarrollo: conversar sobre código, sugerencias **inline**, generar código nuevo, **escanear vulnerabilidades**, mejoras (actualizar versión de lenguaje, depurar, optimizar). Propósito general pero optimizado para AWS. Se usa con la **extensión Amazon Q del IDE**, iniciando sesión con un **AWS Builder ID**; por detrás usa **Amazon Bedrock**.

**Formas de interacción:**

| Modo | Qué hace |
|---|---|
| **Chat** (en el IDE) | Conversación con contexto de sesión; explicar código (todo el archivo o líneas seleccionadas, línea por línea), sugerir mejoras, refactorizar, buenas prácticas de AWS |
| **Comandos de chat** | `/dev` (agente que planifica e implementa funciones en varios archivos), `/transform` (actualizar versión, p. ej. Java 8 u 11 → **Java 17**), `/help`, `/clear` |
| **Inline suggestions** | Desde un comentario en lenguaje natural o mientras se escribe; de una línea a funciones completas; se recorren con flechas y se aceptan con **Tab** |
| **Security scan** | Análisis del proyecto o automático mientras se escribe |
| **CLI** | Autocompletado estilo IDE para cientos de CLIs (Git, npm, Docker, AWS) |

Las sugerencias se basan en LLM entrenados con miles de millones de líneas de código (de Amazon y open source) y en el contexto del IDE: fragmentos, comentarios, **posición del cursor** y archivos abiertos. Se pueden **personalizar** para ajustarse a bibliotecas internas, base de código, algoritmos propietarios y estilo del equipo.

**Demo 1 — chat en PyCharm (JetBrains):**
1. Extensión instalada y sesión iniciada con Builder ID → barra inferior **Amazon Q → Open chat panel**.
2. Archivo de ejemplo: una **Lambda** que revisa certificados de una lista de dominios (válidos o expirados). Pedir «explica qué hace este código».
3. Seleccionar líneas y pedir una explicación detallada.
4. Pedir mejoras según buenas prácticas básicas.
5. Pedir refactor para leer la lista de dominios desde una **variable de entorno** en vez de valores fijos; Q reescribe el script y añade **manejo de errores**. Se puede insertar en el cursor o copiar y reemplazar.

**Demo 2 — inline en un archivo Python vacío:**
1. Activar sugerencias (atajo **Option + C** en macOS).
2. `import boto3` (AWS SDK para Python); comentario «crear el cliente S3» → Tab para aceptar.
3. Comentario: función que recibe una cadena y crea un bucket S3 con manejo de errores; recorrer sugerencias con flechas y aceptar.
4. Comentario para llamar a la función: Q pone un nombre de marcador; se reemplaza por uno **único globalmente** (regla de nombres de S3). Ejecutar: bucket creado.
5. Comentario para **generar JSON de ejemplo** y otro para **subirlo a S3**; añadir a mano el nombre del bucket; ejecutar sin errores.
6. Sin comentario: escribir `def list_buckets` y Q sugiere el cuerpo.

**Buenas prácticas con Q Developer:**
- Usted es **dueño y responsable** del código, incluidas las sugerencias: **revíselas siempre** antes de aceptar.
- Comentarios **breves** y asociados a **tareas pequeñas y discretas**; funciones cortas.
- **Nombres intuitivos** para funciones y elementos de código; cuanto más contexto alrededor, mejor la sugerencia.
- Beneficios: menos código repetitivo (*boilerplate*), más tiempo para problemas complejos, respuestas sobre APIs de AWS, más seguridad.

**Demo 3 — security scan:**
- Los scans usan **detectores** basados en años de buenas prácticas de seguridad de AWS y Amazon.com, que se actualizan solos. Detectan: **fugas de recursos**, **inyección SQL**, **cross-site scripting**, **secretos** en código y archivos de texto (contraseñas *hardcoded*, connection strings, usuarios), y problemas de configuración, cumplimiento y seguridad en archivos de **infraestructura como código**. Alineados con OWASP y buenas prácticas de bibliotecas criptográficas.
- Código de ejemplo deliberadamente inseguro: recibe un argumento `Command`, arma un nombre de log, lo ejecuta en un **subprocess** (`kickoff_subprocess`) sin sanitizar, sube el log a S3 (`upload_output_to_s3`) e **imprime credenciales** y configuración.
- Pasos: barra inferior **Amazon Q → Run Project Scan** → expandir hallazgos del archivo → doble clic lleva a la línea → clic derecho **Explain** o **View details**.
- Hallazgos:

| Hallazgo | Riesgo | Corrección sugerida |
|---|---|---|
| Credenciales de AWS **hardcoded** | Exposición de credenciales | **IAM roles** / acceso basado en roles, o **AWS Secrets Manager** |
| **OS command injection** en el subprocess | Ejecución de código malicioso con entrada no sanitizada | Sanitizar la entrada |
| **Resource leaks** | Archivos abiertos sin cerrar; ralentizan o bloquean el sistema | Cerrar recursos en todos los caminos |
| **Logging de credenciales** | Exponerlas a un atacante | No registrarlas; cifrar datos sensibles antes de registrar |

> [!tip] 💡 Recomendación del curso
> Escanear el código **antes de hacer commit** para detectar los problemas de seguridad más difíciles de ver.

🎯 **Para el examen:** Bedrock = acceso administrado y serverless a FM vía API; RAG + knowledge bases para datos propios y menos alucinaciones; fine-tuning (etiquetado) vs continued pre-training (sin etiquetar); Amazon Q Business para datos internos con permisos; Q Developer para código y security scans; credenciales → IAM roles o Secrets Manager.

---

## ✅ Autoevaluación

> [!question]- 1. Su app corre en una sola instancia EC2 y usa DynamoDB y S3. ¿Dónde está el punto único de falla y cómo lo elimina?
> En la instancia EC2: DynamoDB y S3 son altamente disponibles por diseño. Se elimina con al menos dos instancias en **AZ distintas**, detrás de un **load balancer**, idealmente gestionadas por un **Auto Scaling group** con mínimo 2.

> [!question]- 2. ¿Qué diferencia hay entre basic monitoring, detailed monitoring y high-resolution custom metrics?
> Basic: 1 punto cada 5 min, gratis y automático. Detailed (EC2): 1 punto por minuto, con costo. High-resolution custom metrics: hasta 1 punto por segundo, para métricas propias publicadas con `PutMetricData`.

> [!question]- 3. Quiere una alarma cuando la app devuelva demasiados errores 500. ¿Qué piezas de CloudWatch combina?
> Los logs de la app en **CloudWatch Logs** (en EC2 con el agente), un **metric filter** que cuente los 500, una **alarma** sobre esa métrica (p. ej., > 5 por hora) y una acción, típicamente una notificación **SNS**, o una Lambda que remedie.

> [!question]- 4. ¿Por qué el health check no debería limitarse a comprobar que el puerto 80 esté abierto?
> Porque el puerto abierto no prueba que la app funcione ni que alcance sus dependencias. El curso propone una página `/monitor` que pruebe la conexión a la base de datos y a S3, y apuntar ahí el health check.

> [!question]- 5. Un cliente exige reglas de firewall por IP fija y el backend necesita la IP real del usuario. ¿ALB o NLB?
> **NLB**: admite IP estática / Elastic IP y preserva la IP de origen del cliente. El ALB mostraría la IP del propio load balancer.

> [!question]- 6. Su ASG tiene min 2, desired 2, max 4 y una política de target tracking al 60 % de CPU. Describa qué pasa durante una prueba de estrés de 10 minutos y después.
> La CPU promedio supera el 60 %, la alarma creada por la política se dispara y el ASG lanza instancias (hasta 4). Se esperan 300 s de warm-up antes de contarlas; deben pasar el health check del ALB para recibir tráfico, y la CPU promedio baja. Al terminar la carga, el ASG elimina instancias una a una hasta el mínimo de 2.

> [!question]- 7. ¿Cuándo usaría step scaling en vez de simple scaling?
> Cuando la respuesta debe ser proporcional a la gravedad (p. ej., +2 al 85 %, +4 al 95 %). Simple scaling tiene una sola acción y espera un cooldown; step scaling responde a alarmas adicionales aunque haya un escalado en curso.

> [!question]- 8. ¿Qué ocurre si, para limpiar la cuenta, termina las instancias pero deja el Auto Scaling group?
> El ASG detecta que está por debajo de la capacidad deseada y lanza instancias nuevas. Hay que eliminar el **ASG**.

> [!question]- 9. ¿Por qué una app activo-activo necesita ser stateless?
> Porque cualquier servidor puede atender cualquier request; si la sesión vive en un solo servidor, el siguiente request puede llegar a otro que no la tiene. Si la app es stateful, hay que usar sticky sessions (cookie en ALB) o activo-pasivo.

> [!question]- 10. En el rediseño serverless, ¿qué servicio reemplaza a cada función de EC2 y qué capa no cambia?
> El servidor web (presentación) pasa a **S3 static website hosting** (con CloudFront y Route 53); la lógica de negocio pasa a **Lambda** detrás de **API Gateway**. La capa de datos (DynamoDB + S3 para fotos) queda intacta.

> [!question]- 11. ¿Qué técnica de Bedrock usaría para que un modelo responda con documentos internos de la empresa, y cómo reduce alucinaciones?
> **Knowledge bases con RAG**: recupera por similitud (base vectorial) los fragmentos relevantes de las fuentes propias y los añade al prompt, de modo que la respuesta se apoya en datos reales y no solo en lo que el modelo recuerda.

---

## 🧾 Resumen en 10 líneas

1. Monitorear es recolectar métricas, logs y tráfico para actuar antes que el usuario; la baseline define lo normal.
2. CloudWatch centraliza todo: basic monitoring (5 min, gratis), detailed (1 min, pagado), custom metrics hasta 1 s vía `PutMetricData`.
3. Métricas en namespaces, identificadas y filtradas por dimensions (p. ej. `InstanceId`); dashboards multirregión y compartibles.
4. CloudWatch Logs: log event → log stream → log group; metric filters convierten logs en métricas; EC2 necesita agente, Lambda solo permisos.
5. Alarmas: métrica + umbral + periodo; estados `OK`/`ALARM`/`INSUFFICIENT_DATA`; acciones EC2, Auto Scaling o SNS (y Lambda vía SNS).
6. Disponibilidad en nueves (99,99 % ≈ 52,6 min/año); requiere redundancia multi-AZ; activo-activo exige apps stateless.
7. Escalado vertical tiene tope y downtime; el horizontal no, y se automatiza con EC2 Auto Scaling.
8. ELB es regional, HA y autoescalable: ALB (HTTP/HTTPS, capa 7, rutas, redirects, auth, cookies), NLB (TCP/UDP/TLS, IP estática, preserva IP), GWLB (appliances de terceros); listener + target group (con health check) + rule.
9. Auto Scaling = launch template (qué) + ASG (dónde, min/desired/max) + políticas simple/step/target tracking; la app final queda con ALB + ASG 2–4 al 60 % de CPU.
10. Cierre: rediseño serverless (S3 + CloudFront + Route 53 + API Gateway + Lambda + DynamoDB) e IA generativa con Bedrock (FM, RAG) y Amazon Q (Business, Developer, security scan).

---

## 📖 Recursos del módulo

**Lectura 4.1 — Monitorización en AWS**
- AWS: Amazon CloudWatch · https://aws.amazon.com/cloudwatch/

**Lectura 4.2 — Introducción a Amazon CloudWatch**
- AWS: Primeros pasos con Amazon CloudWatch · https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/GettingStarted.html
- AWS: ¿Qué es Amazon CloudWatch Logs? · https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html
- Servicios de AWS que publican métricas de CloudWatch · https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html
- AWS: Ver métricas disponibles · https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/viewing_metrics_with_cloudwatch.html
- AWS: Precios de Amazon CloudWatch · https://aws.amazon.com/cloudwatch/pricing/
- AWS: Amazon Simple Notification Service · https://aws.amazon.com/sns/
- AWS: Acciones de autoescalado de EC2 · https://aws.amazon.com/ec2/autoscaling/

**Lectura 4.3 — Optimización de soluciones en AWS**
- Alta disponibilidad y escalabilidad en AWS · https://docs.aws.amazon.com/whitepapers/latest/real-time-communication-on-aws/high-availability-and-scalability-on-aws.html
- AWS: Pilar de fiabilidad del AWS Well-Architected Framework · https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html

**Lectura 4.4 — Elastic Load Balancing**
- AWS: Comparación de productos de Elastic Load Balancing · https://aws.amazon.com/elasticloadbalancing/features/#Product_comparisons
- AWS: AWS Certificate Manager · https://aws.amazon.com/certificate-manager/
- AWS: Autenticación de usuarios mediante un Application Load Balancer · https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html
- AWS: Cómo funciona AWS WAF · https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works.html

**Lectura 4.5 — Amazon EC2 Auto Scaling**
- AWS: Amazon EC2 Auto Scaling · https://aws.amazon.com/ec2/autoscaling/
- AWS: Preguntas frecuentes sobre Amazon EC2 Auto Scaling · https://aws.amazon.com/ec2/autoscaling/faqs/
- AWS: Configuración de límites de capacidad para su grupo de Auto Scaling · https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-capacity-limits.html
- AWS: Políticas de escalado por pasos y simples para Amazon EC2 Auto Scaling · https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-simple-step.html
- AWS: Políticas de escalado de seguimiento de objetivos para Amazon EC2 Auto Scaling · https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html
- AWS: Creación de un grupo de Auto Scaling mediante una plantilla de lanzamiento · https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-asg-launch-template.html

---

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[03-Almacenamiento-y-Bases-de-Datos|03 · Almacenamiento y Bases de Datos]] · Siguiente: [[05-Backend-Web-Serverless|05 · Backend Web Serverless ➡]]
