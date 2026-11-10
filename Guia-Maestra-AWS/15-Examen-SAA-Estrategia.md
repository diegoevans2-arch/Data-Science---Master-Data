---
title: "Tomo 15 — Examen SAA-C03: formato, dominios y estrategia"
tags: [aws, solutions-architect, certificacion, saa-c03, estrategia-examen]
curso: 4 · Exam Prep: AWS Certified Solutions Architect – Associate
modulo: "Bienvenido al curso · Determinar la preparación para el examen"
audiencias: [tecnico, puente, ejecutivo]
tomo: 15
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🎓 Tomo 15 — Examen SAA-C03: formato, dominios y estrategia

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[14-Arquitectura-de-Datos-Moderna|14 · Arquitectura de Datos Moderna]] · Siguiente: [[16-Dominio-1-Arquitecturas-Seguras|16 · Dominio 1: Arquitecturas Seguras ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Los tomos 01–14 enseñan los servicios; este tomo enseña **el examen**. El curso 4 del certificado no pretende cubrir todo el temario: su objetivo declarado es darle un método de **autoevaluación** para saber si está listo para presentarse al AWS Certified Solutions Architect – Associate (código **SAA-C03**). Aquí queda el formato de la prueba, los cuatro dominios con su peso real, la forma de leer y descartar respuestas y el plan para cerrar brechas. Los tomos 16–19 desarrollan cada dominio.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** cuándo agendar el examen, cuánto tiempo de estudio asignar por dominio, qué brechas cerrar con práctica en consola antes de pagar la inscripción.
> - **Costo de hacerlo mal:** USD 150 por intento, más el tiempo de preparación perdido si se presenta sin haber medido la preparación por dominio.
> - **Pregunta que responde:** *¿qué evalúa exactamente el SAA-C03, cómo se responde una pregunta de escenario y cómo sé que estoy listo?*

> [!tip] 💡 Analogía general: el examen de conducir con recorrido
> No basta con saberse el reglamento (los servicios): el examinador lo pone en una calle concreta —«lluvia, hora punta, presupuesto acotado»— y le pide la **mejor** maniobra para esas condiciones. Varias maniobras son legales; solo una es la que el examinador espera dadas las palabras clave del escenario.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Bienvenido · Conozca el examen | 2 videos · 2 lecturas | Perfil del candidato, los 4 dominios y sus pesos, servicios clave por dominio, tipos de pregunta, fundamentos, whitepapers y FAQ recomendados |
| Determinar la preparación · Resumen del curso | 1 lectura · 1 video | Inscripción al examen, consejos para el día de la prueba, próximos pasos |

> [!note] ➕ Nota del editor
> El video de bienvenida anuncia además labs prácticos y una prueba práctica oficial completa; en el material extraído de este módulo no aparece su contenido, solo la referencia. Las preguntas de práctica de cada dominio se tratan en los tomos 16–19.

---

## 1. Qué certifica el SAA-C03

Audiencia: 🔧 🧭

**Perfil del candidato.** Personas en un rol de arquitecto de soluciones con **uno o más años** de experiencia diseñando sistemas distribuidos que sean altamente disponibles, rentables, tolerantes a fallos y escalables. No hay requisitos técnicos formales para el curso ni para el examen, pero ayuda entender el rol y saber diseñar soluciones según el **AWS Well-Architected Framework**.

**Qué valida la certificación:**

1. Diseñar soluciones con servicios de AWS que respondan a los requisitos de negocio actuales **y** a las necesidades futuras proyectadas.
2. Diseñar arquitecturas **seguras, resilientes, de alto rendimiento y optimizadas en costo** (los cuatro dominios).
3. **Revisar** soluciones existentes y determinar mejoras.

Durante la prueba se evalúan tareas como el diseño de arquitecturas multinivel y de alta disponibilidad, el escalado de cómputo y almacenamiento, y la identificación de soluciones costo-eficientes.

**Novedades de la versión C03 frente a C02** (según el curso):
- Más de **30 servicios y funcionalidades nuevas** en el temario.
- Se incorpora el sexto pilar del Well-Architected Framework: **Sustainability**.
- El dominio 3 trae una sección totalmente nueva sobre **ingesta y almacenamiento de datos de alto rendimiento** (aparece Lake Formation).
- Kubernetes (Amazon EKS) gana presencia en los exámenes actualizados.
- AWS agrega más optimización de costos en todos sus exámenes.

---

## 2. Ficha técnica del examen

Audiencia: 🔧 🧭 👔

| Atributo | Valor |
|---|---|
| Código | SAA-C03 |
| Nivel | Associate |
| Formato | Solo preguntas de **opción múltiple** y de **respuesta múltiple** |
| Número de preguntas | **65** |
| Duración | **130 minutos** (≈ 2 minutos por pregunta) |
| Costo | **USD 150** |
| Modalidad | Centro de pruebas **Pearson VUE** o examen **supervisado en línea** |
| Idiomas | Inglés, francés (Francia), alemán, italiano, japonés, coreano, portugués (Brasil), chino simplificado, chino tradicional, español (Latinoamérica) y español (España) |
| Penalización | No se penaliza la respuesta incorrecta; en blanco vale cero |

**Los dos tipos de pregunta:**

| Tipo | Estructura | Implicancia |
|---|---|---|
| Multiple choice | 1 correcta + 3 incorrectas (**distractores**) | Descartar hasta quedar con una |
| Multiple response | **2 o más** correctas entre **5 o más** opciones | El enunciado dice cuántas elegir; cada opción se evalúa por separado |

> [!warning] ⚠️ Las preguntas son más largas de lo que se espera de un nivel Associate
> La mayoría trae un escenario de varias frases hasta un par de párrafos, y las opciones también suelen ocupar varias frases. Es común que las cuatro respuestas repitan casi el mismo texto y solo cambien **una o dos palabras**: esas palabras deciden cuál es la correcta y cuál el distractor.

---

## 3. Los cuatro dominios y sus pesos

Audiencia: 🔧 🧭 👔

| Dominio | Nombre oficial (guía SAA-C03) | Peso | Foco | Tomo |
|---|---|---|---|---|
| 1 | Design Secure Architectures | **30 %** | Acceso seguro a recursos, cargas de trabajo y aplicaciones seguras, controles de seguridad de datos | [[16-Dominio-1-Arquitecturas-Seguras]] |
| 2 | Design Resilient Architectures | **26 %** | Arquitecturas escalables y débilmente acopladas; alta disponibilidad y tolerancia a fallos | [[17-Dominio-2-Arquitecturas-Resilientes]] |
| 3 | Design High-Performing Architectures | **24 %** | Almacenamiento, cómputo, bases de datos, redes e ingesta/transformación de datos de alto rendimiento | [[18-Dominio-3-Arquitecturas-de-Alto-Rendimiento]] |
| 4 | Design Cost-Optimized Architectures | **20 %** | Almacenamiento, cómputo, bases de datos y redes costo-optimizados | [[19-Dominio-4-Arquitecturas-Optimizadas-en-Costo]] |

```
 Peso en el examen (65 preguntas ≈)
 D1 Seguras          ██████████████████████████████  30 %  ≈ 19-20 preguntas
 D2 Resilientes      ██████████████████████████      26 %  ≈ 17
 D3 Alto rendimiento ████████████████████████        24 %  ≈ 15-16
 D4 Costo            ████████████████████            20 %  ≈ 13
```

> [!note] ➕ Nota del editor
> La conversión a número de preguntas es una aproximación propia (peso × 65). AWS no publica el reparto exacto por formulario, y parte de las preguntas pueden no puntuar.

### 3.1 Servicios y conceptos que el curso pide dominar por dominio

**Dominio 1 — Seguras (30 %)**
- *Acceso seguro:* AWS IAM, AWS IAM Identity Center (antes AWS Single Sign-On), AWS Security Token Service (STS), AWS Control Tower, Service Control Policies (SCP).
- *Cargas de trabajo seguras:* Amazon Cognito, Amazon GuardDuty, Amazon Macie, AWS Shield, AWS WAF, AWS Secrets Manager, STS y los controles de seguridad de la VPC.
- *Datos:* AWS KMS, AWS Certificate Manager (ACM), requisitos de recuperación y retención, políticas de acceso a claves, rotación de claves y renovación de certificados, todos los servicios de almacenamiento.

**Dominio 2 — Resilientes (26 %)**
- *Escalables y débilmente acopladas:* Amazon API Gateway, AWS Transfer Family, Amazon SQS, AWS Secrets Manager, Application Load Balancer, Amazon SNS, AWS Fargate, AWS Lambda, Amazon ECS, Amazon EKS. Principios de diseño de microservicios multinivel y arquitecturas orientadas a eventos. **Escalado horizontal vs vertical** (la confusión más frecuente según la instructora).
- *Alta disponibilidad y tolerancia a fallos:* infraestructura global, Amazon Route 53, estrategias y diseños de backup, RDS Proxy, AWS X-Ray, infraestructura inmutable, buenas prácticas de disaster recovery, servicios administrados, balanceo de carga, patrones de diseño, opciones de almacenamiento según características, automatización y métricas para sostener la disponibilidad.

> [!note] ➕ Nota del editor
> La mención de Amazon Comprehend en el dominio 2 es intencional: el curso la explica con un caso de clasificación de tickets de TI (ver [[17-Dominio-2-Arquitecturas-Resilientes]]). La expresión «patrones de diseño desconfiados» de la transcripción automática probablemente traduce mal «distributed design patterns».

**Dominio 3 — Alto rendimiento (24 %)**
- Amazon CloudFront, AWS Direct Connect, AWS VPN, AWS PrivateLink, Amazon S3, Amazon EFS, Amazon EBS, AWS Batch, Amazon ElastiCache, Amazon Athena, AWS Lake Formation, Amazon QuickSight, AWS DataSync, AWS Storage Gateway, AWS Glue, Amazon EMR, Amazon Kinesis.
- Servicios de caché y patrones de acceso a datos; serverless con Lambda y Fargate; elegir tipo y tamaño de recurso.
- Dos preguntas guía: *¿cómo desacoplar para que cada componente escale por separado?* y *¿qué métricas y condiciones disparan las acciones de escalado?*
- Ingesta y transformación: asegurar los puntos de acceso de ingesta y dimensionar la velocidad de ingesta, transformación y streaming.

**Dominio 4 — Costo (20 %)**
- Herramientas de gestión de costos: AWS Cost Explorer, etiquetas (cost allocation tags), AWS Budgets, Savings Plans, Reserved Instances y reservas de capacidad, AWS Cost and Usage Report.
- **Right-sizing** de cómputo, instancias de base de datos **y almacenamiento**; costo de transferencia de datos; S3 Lifecycle.
- Conectividad: AWS Transit Gateway, VPC peering, DNS/Route 53, Elastic Load Balancing.

**Transversal:** los seis pilares del Well-Architected Framework, con preguntas nuevas sobre **Sustainability**.

---

## 4. Fundamentos previos a los dominios

Audiencia: 🔧 🧭

El curso insiste en que hay que entender **cómo puede fallar AWS** para diseñar alrededor de los fallos («todo falla todo el tiempo»). La lista de fundamentos:

| Fundamento | Qué dominar | Dónde se ve |
|---|---|---|
| Cloud computing | Qué es AWS y las cinco características de la computación en la nube | [[01-Vision-General-y-Seguridad]] |
| Infraestructura global | Regions, Availability Zones, edge locations | [[01-Vision-General-y-Seguridad]] |
| Ámbito de resiliencia | Servicios resilientes por **AZ**, por **Region** (aguantan la caída de una AZ) y **globales** (aguantan la caída de una Region) | [[02-Computo-y-Redes]], [[03-Almacenamiento-y-Bases-de-Datos]] |
| Shared Responsibility Model | Qué parte de la seguridad es de AWS y cuál del cliente | [[01-Vision-General-y-Seguridad]] |
| Well-Architected Framework | Los seis pilares, incluido Sustainability | [[04-Monitoreo-y-Optimizacion]], [[08-Gobierno-y-Multicuenta]] |
| Fundamentos generales | Redes, virtualización, cifrado, almacenamiento, seguridad, DNS, DNSSEC, bases de datos | Toda la guía |

**Tres formas de estudiar Well-Architected** que propone el curso: (1) la documentación en línea, leída varias veces; (2) la **AWS Well-Architected Tool**, para revisar el estado de cargas de trabajo reales; (3) los **AWS Well-Architected Labs**, para practicar los conceptos.

> [!example] 📊 Ámbitos de resiliencia, en un esquema
> ```
>  GLOBAL   ── aguanta la caída de una Region completa
>  REGIONAL ── aguanta la caída de una AZ
>  ZONAL    ── vive en una AZ; si la AZ cae, cae con ella
> ```
> Saber en qué nivel vive cada servicio es lo que permite responder «¿qué pasa si falla X?».

---

## 5. Estrategia para responder: leer, descartar, decidir

Audiencia: 🔧 🧭

El método que el curso enseña, condensado:

1. **Leer completa la pregunta y todas las opciones.** No responder con la primera opción que «suena bien».
2. **Subrayar mentalmente las palabras clave y los calificativos:** *menor costo*, *mínimo esfuerzo operativo*, *más resiliente*, *menor latencia*, *sin cambios en la aplicación*, *en tiempo real*, *cifrado en reposo/en tránsito*. Si la pregunta pide lo más barato y usted piensa en lo más resiliente, elegirá otra respuesta.
3. **Eliminar distractores** con lo que sabe: opciones técnicamente imposibles, servicios que no hacen lo que se describe, soluciones que violan un requisito explícito.
4. **Comparar y contrastar las que quedan** frente a los calificativos. Varias pueden funcionar; gana la que mejor cumple el criterio pedido.
5. **Gestionar el tiempo:** si se demora demasiado, marque su mejor respuesta, **flag for review** y siga. Nunca deje en blanco.

```
 Escenario ──► ¿Qué pide? (calificativo) ──► Descarte duro ──► Contraste fino ──► Respuesta
                 costo / HA / latencia /        (imposible,       (mejor ajuste       (o marcar
                 esfuerzo operativo / seguridad  no cumple)        al calificativo)    y volver)
```

> [!warning] ⚠️ Regla crítica: una o dos palabras cambian la respuesta
> Cuando las opciones se repiten casi idénticas, aíslelas por diferencia: escriba (mentalmente) solo lo que cambia entre A, B, C y D. Esa diferencia es lo que se está evaluando.

> [!tip] 💡 Analogía
> Descartar es como el «juego de las diferencias»: no compare cada opción con su conocimiento entero; compárelas entre sí y pregunte qué concepto toca cada diferencia.

🎯 **Para el examen:**
- «Menor esfuerzo operativo» suele empujar hacia servicios administrados o serverless.
- «Menor costo» obliga a revisar right-sizing, modelos de compra y clases de almacenamiento antes que la opción más robusta.
- En multiple response, cada opción es un verdadero/falso independiente; no elija una porque «acompaña bien» a otra.

---

## 6. Plan de estudio y recursos recomendados

Audiencia: 🔧 🧭 👔

**Orden recomendado por el curso:**

1. Partir por la **guía oficial del examen** (exam guide): describe el examen y es la base de este curso.
2. Estudiar cada dominio y sus **task statements** (tomos 16–19), evaluando en cada uno si está listo o necesita más profundidad.
3. Usar las **preguntas de práctica** para entrenar la lectura comprensiva y la detección de palabras clave.
4. Ganar **experiencia práctica** (labs, consola).
5. Rendir el **examen de práctica oficial** de AWS para identificar fortalezas y debilidades por dominio.
6. Estudiar las brechas detectadas y, cuando esté listo, **agendar**.

**Lecturas que el curso destaca:**

| Recurso | Por qué |
|---|---|
| AWS Well-Architected Framework (whitepaper) | El favorito de la instructora: los escenarios del examen se apoyan fuertemente en sus pilares |
| AWS Security Best Practices (whitepaper) | También clave para el examen |
| FAQ de Amazon EC2, S3, VPC, Route 53, RDS y SQS | Detalles finos de servicios muy preguntados |

AWS ofrece además una suscripción a consejos semanales de Training and Certification, guías y preguntas de muestra, y sesiones con instructores acreditados.

---

## 7. Cómo saber si está listo

Audiencia: 🧭 👔

El cierre del curso recalca que **no enseña todo el temario**: es un instrumento de autoevaluación. La preparación se mide así:

| Señal | Cómo medirla |
|---|---|
| Comprende cada task statement | Puede explicar qué servicio usar y por qué, para requisitos distintos de seguridad, resiliencia y costo |
| Supera las preguntas de práctica por dominio | Acierta y además sabe por qué fallan los distractores |
| Examen de práctica oficial | Revela el dominio más débil; ahí se concentra el estudio restante |
| Experiencia práctica | Ha hecho los labs y trabajado en consola, no solo lectura |

**Consejos para el día del examen** (cierre del curso): leer y comprender pregunta y opciones; identificar palabras, frases clave y calificativos; eliminar opciones con lo que sabe; comparar las restantes; si se demora, elegir la mejor y marcar para revisión; no dejar nada en blanco.

**Inscripción:** revisar las opciones de rendición (centro o en línea) y crear o acceder a la cuenta de AWS Certification para agendar.

> [!danger] 🚨 Error costoso
> Presentarse habiendo estudiado solo los servicios y nunca practicado la lectura de escenarios. El examen premia al que **lee el calificativo**, no al que conoce más servicios.

---

## ✅ Autoevaluación

> [!question]- ¿Cuántas preguntas tiene el SAA-C03, cuánto dura y cuánto cuesta?
> 65 preguntas, 130 minutos, USD 150.

> [!question]- ¿Qué diferencia hay entre una pregunta multiple choice y una multiple response?
> La primera tiene una correcta y tres distractores; la segunda tiene dos o más correctas entre cinco o más opciones.

> [!question]- ¿Cuál es el dominio de mayor peso y cuánto vale?
> Dominio 1, Design Secure Architectures, 30 %.

> [!question]- ¿Conviene dejar en blanco una pregunta dudosa?
> No. Las incorrectas no penalizan; en blanco vale cero. Elija la mejor y márquela para revisión.

> [!question]- ¿Qué pilar del Well-Architected Framework es nuevo en el C03?
> Sustainability, el sexto pilar.

> [!question]- Una pregunta pide «la solución de menor costo» y usted ve una opción multi-Region muy resiliente. ¿Qué hace?
> Verificar si esa resiliencia es un requisito explícito. Si no lo es, probablemente sea un distractor: el calificativo manda.

> [!question]- ¿Qué tres niveles de resiliencia de servicios conviene distinguir?
> Zonal (cae con su AZ), regional (soporta la caída de una AZ) y global (soporta la caída de una Region).

> [!question]- ¿Cuál es el primer recurso que recomienda el curso?
> La guía oficial del examen.

## 🧾 Resumen en 10 líneas

1. SAA-C03 certifica a arquitectos con ≥1 año diseñando sistemas distribuidos disponibles, rentables, tolerantes a fallos y escalables.
2. 65 preguntas, 130 minutos, USD 150, Pearson VUE o en línea supervisado.
3. Dos tipos: multiple choice (1 de 4) y multiple response (2+ de 5+).
4. Dominios: Seguras 30 %, Resilientes 26 %, Alto rendimiento 24 %, Costo 20 %.
5. C03 suma más de 30 servicios nuevos, el pilar Sustainability y una sección de ingesta de datos.
6. Fundamentos: cloud, infraestructura global, ámbitos de resiliencia, Shared Responsibility, Well-Architected.
7. Las preguntas son largas; una o dos palabras separan la correcta del distractor.
8. Método: leer todo, identificar calificativos, descartar, contrastar, marcar y seguir.
9. No hay penalización: nunca dejar en blanco.
10. El curso mide preparación; complementar con práctica, examen oficial de práctica y estudio de brechas.

## 📖 Recursos del módulo

- AWS Certified Solutions Architect – Associate (página del examen) · https://aws.amazon.com/certification/certified-solutions-architect-associate/
- AWS Security Best Practices (whitepaper) · https://docs.aws.amazon.com/whitepapers/latest/introduction-aws-security/security-of-the-aws-infrastructure.html
- Amazon EC2 FAQ · https://aws.amazon.com/ec2/faqs/?saa=sec&sec=prep
- Amazon S3 FAQ · https://aws.amazon.com/s3/faqs/?saa=sec&sec=prep
- Amazon VPC FAQ · https://aws.amazon.com/vpc/faqs/?saa=sec&sec=prep
- Amazon Route 53 FAQ · https://aws.amazon.com/route53/faqs/?saa=sec&sec=prep
- Amazon RDS FAQ · https://aws.amazon.com/rds/faqs/?saa=sec&sec=prep
- Amazon SQS FAQ · https://aws.amazon.com/sqs/faqs/?saa=sec&sec=prep
- Opciones para rendir el examen · https://aws.amazon.com/certification/certification-prep/testing/?ch=tile&tile=getstarted
- Cuenta de AWS Certification (agendar) · https://www.aws.training/certification/?ch=tile&tile=getstarted
- Guía del examen, preguntas de muestra y AWS Well-Architected Framework: mencionados en el material sin URL en el texto extraído.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[14-Arquitectura-de-Datos-Moderna|14 · Arquitectura de Datos Moderna]] · Siguiente: [[16-Dominio-1-Arquitecturas-Seguras|16 · Dominio 1: Arquitecturas Seguras ➡]]
