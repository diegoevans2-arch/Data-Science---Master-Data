---
title: "Tomo 12 — Procesamiento y Análisis de Datos"
tags: [aws, solutions-architect, data-lake, glue, etl, databrew, glue-studio, athena]
curso: 3 · Introduction to Designing Data Lakes on AWS
modulo: "Módulo 4: Tratamiento y análisis de datos"
audiencias: [tecnico, puente, ejecutivo]
tomo: 12
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# ⚙️ Tomo 12 — Procesamiento y Análisis de Datos

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[11-Data-Lake-con-Lake-Formation|11 · Data Lake con Lake Formation]] · Siguiente: [[13-Lake-Formation-Avanzado|13 · Lake Formation Avanzado ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Un lago lleno de datos no uniformes (fechas en dos formatos, teléfonos con y sin guiones, PII donde no debería estar) no sirve. Este módulo muestra cómo **AWS Glue** limpia y transforma: las fases de un **ETL job**, las tres formas de escribir el código (script editor, interactive sessions, **Glue Studio**), cómo se disparan los jobs y se encadenan en **workflows**, el perfilado y las reglas de calidad con **Glue DataBrew**, el enmascarado de PII sin escribir código, y una charla técnica sobre **Athena federated queries** y la **gravedad de los datos**.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** cuánto código escribir (visual vs script), cómo cumplir requisitos de privacidad antes de exponer datos, cuándo consultar en el origen y cuándo terminar de migrar.
> - **Costo de hacerlo mal:** equipos de datos que gastan cerca del 80 % de su tiempo limpiando a mano, PII expuesta en datasets de análisis, migraciones que se eternizan porque "total, la federated query funciona".
> - **Pregunta que responde:** *¿cómo convierto datos crudos y desordenados en un dataset limpio, seguro y catalogado, sin montar servidores?*

> [!tip] 💡 Analogía general: las tijeras y el pegamento
> El instructor bromea con que un data lake está hecho "de pegamento" (Glue). En la demo de ETL, el extract es tomar la hoja nueva del bucket, el transform son las "tijeras Python o Spark" que recortan lo que sobra, y el load es pegar el resultado en su lugar final. Glue pone la mesa, las tijeras y el pegamento; usted solo decide qué recortar.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Objetivos del módulo | 1 video | Procesamiento con Glue y generación de código |
| Transformación de datos | 1 video | Datos no uniformes, privacidad, enmascarado, DataBrew |
| Procesamiento de datos con AWS Glue | 1 video | Fases del ETL job, Spark/Python, tres formas de autoría |
| Trabajos y flujos de trabajo de AWS Glue | 1 video | Triggers (schedule, on demand, condicional) y workflows |
| Demo: Glue DataBrew | 1 video | Profile job con PII y duplicados; DQ Rules |
| Demo: Glue Studio | 1 video | Visual ETL que detecta y redacta PII |
| Charla técnica: federated queries Glue/Athena | 1 video | Migración híbrida, Lambda, seguridad, data gravity |

---

## 1. Por qué transformar

Audiencia: 🧭 👔

### 1.1 El problema de los datos no uniformes

Escenario del curso: un sitio web recoge datos de clientes (nombres, direcciones, códigos postales, cumpleaños). Al mirar de cerca:

| Campo | Inconsistencia |
|---|---|
| Fecha de nacimiento | Clientes europeos en día-mes-año; estadounidenses en mes-día-año |
| Teléfono | Algunos con guiones y código de área, otros sin |
| Código postal | Algunos con letras, otros solo números |

Mezclar muchas fuentes produce este problema de forma **permanente**. El curso cita que quienes construyen y mantienen data lakes dedican **casi el 80 % de su tiempo** a recolectar, limpiar y organizar datos.

### 1.2 Privacidad

Según el país, hay requisitos sobre qué datos se pueden almacenar. El instructor recuerda una empresa donde había que ocultar los **últimos tres dígitos de cada dirección IP**, un trabajo penoso antes de Glue. Con Glue se crean ETL jobs que limpian y **enmascaran**: en el diagrama del curso se eliminan el **apellido**, parte del **email** y el **TaxID**, dejando un dataset limpio en S3.

### 1.3 AWS Glue y DataBrew

- **AWS Glue**: herramienta administrada de **ETL**; toma datos raw desorganizados y deja un bucket S3 limpio para visualizar después.
- **AWS Glue DataBrew**: la opción **point-and-click** del ETL; más de **250 transformaciones prediseñadas**; **serverless** (escala solo, sin infraestructura que administrar).

---

## 2. Procesamiento con AWS Glue: el ETL job

Audiencia: 🔧 🧭

Antes, un ETL tradicional exigía armar una flota de servidores, mantener el software y cablear los triggers. Glue hace ese trabajo pesado; uno se concentra en **qué** debe hacer el job. Incluso se puede usar una **función Lambda** para disparar el ETL cuando llegan datos nuevos al bucket.

### 2.1 Las tres fases

```
 EXTRACT                         TRANSFORM                         LOAD
 Consultar la fuente:            Código Spark o Python que          Dejar el resultado donde
 "¿hay datos nuevos desde        recorta, corrige y da forma        corresponde: base de datos,
  la última vez?" o filtrar      (las "tijeras")                    data warehouse o bucket S3
 por período, tipo o campo
```

Glue admite entornos de scripting **Apache Spark** y **Python**.

### 2.2 Tres formas de escribir el código

| Opción | Qué ofrece | Para quién |
|---|---|---|
| **AWS Glue script editor** | Crear o subir scripts Spark o Python existentes | Quien quiere hacerlo todo por su cuenta |
| **AWS Glue interactive sessions** | Trabajar en un **Jupyter Notebook**: escribir uno propio o editar y subir uno existente | Desarrollo iterativo, exploración |
| **AWS Glue Studio** | **Editor visual** que genera el script; incluye **plantillas** para empezar | Quien quiere ayuda o no escribe código |

---

## 3. Jobs y workflows

Audiencia: 🔧

Cuando Glue ejecuta una de estas cargas de trabajo se habla de un **AWS Glue job**. Formas de dispararlo:

| Trigger | Cuándo | Ejemplo |
|---|---|---|
| **Scheduled** | A una hora fija | Todos los martes a las 13:00, porque a esa hora llega el raw o porque no es urgente |
| **On demand** | Solo cuando alguien lo ejecuta | Dataset de uso poco frecuente, o un job grande único ante una avalancha de datos |
| **Conditional** (por evento) | Cuando ocurre algo | Llegó un dataset nuevo al bucket S3 |

### 3.1 AWS Glue workflows

Cuando se necesitan varios jobs para transformar y catalogar, un **workflow** los **encadena**. Ejemplo del curso:

```
 Trigger: "hay datos nuevos en el bucket S3"
        │
        ├──► Job 1: deduplicar  ───┐
        └──► Job 2: normalizar     ├──► (ambos OK) ──► Crawler: catalogar ──► datos actualizados
             teléfonos ────────────┘
```

Ventaja: el workflow se **visualiza en la consola**; se puede seguir paso a paso, ver dónde falla o confirmar que todo quedó en verde.

🎯 **Para el examen:**
- ETL serverless sobre datos del lago → **AWS Glue** (Spark o Python).
- Limpieza visual sin código, 250+ transformaciones → **Glue DataBrew**.
- Construir ETL arrastrando nodos y obtener el script → **Glue Studio**.
- Orquestar varios jobs y crawlers con dependencias → **Glue workflows**.
- Disparar ETL al llegar un objeto → trigger condicional o Lambda.

---

## 4. Athena federated queries y data gravity (charla técnica)

Audiencia: 🔧 🧭 👔

**Contexto:** un equipo migra su data lake on-premises a AWS; temporalmente tiene recursos repartidos entre hardware propio y la nube. Ya construye ETL en Glue Studio (le gusta el editor WYSIWYG: fuente, transformación, destino), pero necesita **consultar y visualizar** datos que están tanto en S3 como on-premises.

### 4.1 La solución: Athena federated query

- Las tablas de Athena viven en el Glue Data Catalog y apuntan a S3 por la propiedad `location`; una **federated query** puede ir además **a cualquier otra fuente**, incluida la on-premises.
- Permite **unir** lo ya migrado con lo que aún está en el centro de datos: `SELECT … FROM tablaA, tablaB`, con A en S3 y B en otra fuente ("se federa a otra galaxia").
- **Por dentro funciona con una función Lambda** que el usuario controla: puede elegir la **VPC** donde corre e incluso insertar código propio para **transformar al vuelo**.

### 4.2 Por qué igual conviene terminar la migración

| Federated query | Datos centralizados en S3 |
|---|---|
| Flexible, consulta en el origen | Gobierno centralizado con **Lake Formation** |
| Costo de **rendimiento**: atraviesa la red (internet, potencialmente por VPN), y con cientos de GB o TB hay que traer mucho dato | Servicios integrados (DynamoDB, Redshift…) en la misma Region, latencia de **milisegundos** |
| Hay caché que ayuda en consultas repetidas | Los datos están físicamente juntos |

> [!tip] 💡 Data gravity: "los datos tienen gravedad"
> Las aplicaciones (y las consultas, y un dashboard) deben estar **cerca de los datos**, porque mover aplicaciones es fácil, sobre todo con contenedores, y mover datos es difícil: los datos "pesan". Concentrar los datos en un lugar permite acercarles las aplicaciones. El curso anuncia también el enfoque *outside-in* (se ve en [[14-Arquitectura-de-Datos-Moderna]]).

> [!warning] ⚠️ La federación no reemplaza la migración
> El riesgo que plantea la charla: que el equipo vea las federated queries y decida no terminar de migrar. La recomendación es seguir migrando y usar la federación como puente mientras tanto.

### 4.3 Seguridad de la federación

| Aspecto | Cómo se protege |
|---|---|
| Dónde corre | La Lambda se ubica en una **VPC** elegida |
| Tránsito (red) | Conectividad **VPN**: túnel privado y cifrado en capas bajas del modelo OSI |
| Tránsito (aplicación) | Las consultas pueden cifrarse además con **certificados** |
| Reposo on-premises | Gestión de claves y cifrado propios del centro de datos |
| Reposo en AWS | **AWS KMS**; el cifrado en S3 con KMS requiere mínimo esfuerzo y está activado por defecto |

> [!note] ➕ Nota del editor
> Es el mismo dilema de convivir con dos data lakes (uno antiguo y uno nuevo) durante una homologación: consultar ambos es útil como puente, pero la deuda solo se paga cuando las consultas apuntan a un único lugar gobernado.

---

## 5. Demos y laboratorios

### 5.1 Demo — AWS Glue DataBrew: perfilado y reglas de calidad

**Objetivo:** perfilar un dataset, detectar PII y duplicados, y validar calidad con reglas. **Servicios:** Glue DataBrew, S3, IAM.

**Parte A — Profile job sobre registros de clientes**

1. Abrir un dataset ya importado de **clientes**: nombre, segundo nombre, apellido, género, fecha de nacimiento, dirección y otros campos.
2. **Run data profile**, sobre **todo el dataset**.
3. Elegir el bucket S3 de salida para los resultados del job.
4. En la configuración: activar la detección de **filas duplicadas** y habilitar **PII statistics**; en el desplegable hay categorías como tarjetas de crédito, cuentas bancarias, licencias de conducir. Se marcan **todas las categorías** (tarda un poco más).
5. Elegir el **IAM role** que ejecutará el job → **Create** → crear el job → **Run** (un par de minutos).
6. Resultado:
   - **22 % de celdas vacías**: puede ser un campo opcional o un síntoma de problemas en la ingesta.
   - **Sin filas duplicadas**.
   - PII: casi todo el dataset se identifica como dato personal (nombres, direcciones, fecha de nacimiento y género como posible categoría).
   - Desglose estadístico adicional por columna.

> [!warning] ⚠️ Que haya PII no es un error
> No está mal que la PII esté en el dataset, pero detectarla obliga a tomar **precauciones de seguridad adicionales** para que no termine donde no debe.

**Parte B — DQ Rules sobre un dataset de ventas**

1. En el panel izquierdo, **DQ Rules** → **Create ruleset**; nombre del tipo "la calidad está de moda"; dataset de **ventas**.
2. La vista previa muestra **grandes bloques de ceros** en cantidad y ventas.
3. **Recommendations**: DataBrew sugiere buscar filas duplicadas y valores faltantes.
4. Regla 1: **filas duplicadas**, verificación individual, condición **igual a 0**.
5. Regla 2: **Common checks for selected columns** sobre `quantity` y `Total_Sales`, condición de valor numérico **mayor que 0** (marca lo que no cumpla).
6. Revisar el resumen → **Create ruleset**.
7. Aplicarlo: seleccionar el ruleset → **Create profile with ruleset**, job "Sales Profile Job", sobre todo el dataset, ubicación de salida, asignar role → **Save** → **Run**.
8. Resultado en la pestaña **Data quality rules**: el ruleset **falla**, con **99 % de éxito** y algo menos de **1 % de fallas**. El dataset necesita una transformación adicional (fuera del alcance de la demo).

### 5.2 Demo — AWS Glue Studio: ETL visual que redacta PII

**Objetivo:** construir un ETL sin escribir código que detecte y oculte datos sensibles. **Servicios:** Glue Studio, S3, Glue Data Catalog, IAM.

1. Glue Studio → **ETL jobs** → **Visual ETL**. Nombre descriptivo del job (p. ej. "Fancy Glue ETL para proteger la PII").
2. La pestaña **Script** muestra el código que el editor visual va escribiendo a medida que se agregan nodos.
3. **Job details**: asignar el **IAM role**; confirmar el número de **workers** y el resto de parámetros.
4. Pestaña **Visual** → **Add (+) → Sources → Amazon S3**. Nombrar el nodo ("datos PII"), elegir bucket y carpeta, formato **CSV**. La **data preview** muestra datos ficticios con números de seguro social y fechas de nacimiento.
5. **Add nodes → Transforms → Detect sensitive data** (queda enlazado a la fuente). En lugar de escanear todo, elegir patrones concretos: **tarjetas de crédito, números de seguro social (SSN) y direcciones de email**.
6. Acción al encontrarlos: en vez de solo detectar, **Redact**; texto de reemplazo personalizado: **"supersecreto"**.
7. **Add nodes → Change schema**, enlazado como siguiente paso, para **eliminar las entradas detectadas** del esquema.
8. **Add nodes → Targets → Amazon S3**: formato **JSON**, **sin compresión**, en la raíz de otro bucket.
9. Opción del target: **actualizar el Data Catalog**: crear una tabla y, en ejecuciones siguientes, **actualizar el esquema**; darle nombre.
10. **Save** → **Run**. En **Run details** y en los logs se sigue la ejecución (aprox. un minuto).
11. Revisar la pestaña **Script**: Glue Studio generó todo el código. Sirve de transición: se empieza visual y luego se puede personalizar el script.
12. Ver el JSON de salida: los SSN y demás campos marcados aparecen reemplazados por **"supersecreto"**.
13. En el **Data Catalog** aparece la nueva tabla con los datos limpios, lista para análisis.

> [!example] 📊 Lo que demuestra la demo
> El cumplimiento de privacidad (enmascarar o redactar PII) puede vivir **dentro del pipeline**, antes de que el dato llegue a quien lo consulta, y el catálogo queda actualizado en el mismo job.

**Errores comunes:** olvidar el IAM role del job; dejar marcadas todas las categorías de PII cuando se sabe cuáles hay (más lento); no activar la actualización del Data Catalog y luego no encontrar la tabla.

El módulo no tiene laboratorio autoguiado; la lección "Evaluación de la semana 4" no trae contenido textual adicional.

---

## 6. Arquitectura de referencia: estado al cierre del módulo

```
  S3 raw ── evento / schedule / on demand ──► Glue workflow
                                               ├─ Job: dedup
                                               ├─ Job: normalizar formatos
                                               ├─ Job (Glue Studio): Detect sensitive data → Redact → Change schema
                                               └─ Crawler / actualización del Data Catalog
                                                        │
  DataBrew: profile (PII, duplicados, vacíos) + DQ rules│
                                                        ▼
                              S3 limpio (JSON/Parquet) + tabla en Glue Data Catalog
                                                        │
                               Athena ◄─── federated query (Lambda en VPC, VPN) ──► fuente on-premises
```

---

## ✅ Autoevaluación

> [!question]- ¿Qué tres inconsistencias usa el curso para ilustrar datos no uniformes?
> Fechas de nacimiento en formatos día-mes-año vs mes-día-año, teléfonos con y sin guiones o código de área, y códigos postales con y sin letras.

> [!question]- ¿Cuáles son las tres formas de escribir el código de un Glue job?
> Script editor (Spark o Python propio), interactive sessions con Jupyter Notebook, y Glue Studio con editor visual y plantillas.

> [!question]- Mencione los tres tipos de trigger de un Glue job con un caso de uso.
> Scheduled (todos los martes 13:00 cuando llega el raw), on demand (job grande único) y condicional por evento (llegó un archivo nuevo a S3).

> [!question]- ¿Qué aporta un Glue workflow frente a jobs sueltos?
> Encadena jobs y crawlers con dependencias y se visualiza en la consola, lo que permite seguir la ejecución y ubicar dónde falló.

> [!question]- ¿Qué diferencia a DataBrew de Glue Studio?
> DataBrew es preparación y perfilado point-and-click con 250+ transformaciones y reglas de calidad; Glue Studio construye ETL jobs visuales que generan un script Spark editable.

> [!question]- En la demo de DataBrew, ¿qué significa el 22 % de celdas vacías?
> Puede ser un campo opcional que la gente omite o una señal de que la ingesta no está entregando bien la información; hay que investigarlo.

> [!question]- ¿Cómo funcionan por dentro las Athena federated queries y cómo se aseguran?
> Con una función Lambda controlada por el usuario, ubicable en una VPC elegida; el tránsito se protege con VPN y certificados, y el reposo con KMS en AWS o con las claves propias on-premises.

> [!question]- ¿Qué es data gravity y qué recomienda?
> La idea de que los datos "pesan" y es más fácil mover aplicaciones que datos; recomienda ubicar aplicaciones y consultas cerca de donde están los datos y concentrarlos en un lugar.

## 🧾 Resumen en 10 líneas

1. Mezclar fuentes genera datos no uniformes de forma permanente; limpiar consume cerca del 80 % del tiempo.
2. Glue es el ETL administrado del lago; también enmascara datos por privacidad.
3. ETL: extract (consultar lo nuevo), transform (Spark o Python), load (BD, warehouse o S3).
4. Código: script editor, interactive sessions (Jupyter) o Glue Studio visual con plantillas.
5. Triggers: scheduled, on demand o condicional por evento; también Lambda.
6. Workflows encadenan jobs y crawlers y se visualizan en la consola.
7. DataBrew: 250+ transformaciones, serverless; perfila PII, duplicados y vacíos, y aplica DQ rules.
8. Glue Studio: Detect sensitive data → Redact → Change schema → S3 + Data Catalog, con script generado.
9. Athena federated query une S3 con fuentes externas vía Lambda; útil como puente, con costo de rendimiento.
10. Data gravity: acercar aplicaciones a los datos y terminar la migración para gobernar con Lake Formation.

## 📖 Recursos del módulo

El material del módulo no trae enlaces ni lecturas con URLs.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[11-Data-Lake-con-Lake-Formation|11 · Data Lake con Lake Formation]] · Siguiente: [[13-Lake-Formation-Avanzado|13 · Lake Formation Avanzado ➡]]
