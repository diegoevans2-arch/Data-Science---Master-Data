---
title: "Tomo 13 — Lake Formation Avanzado y QuickSight"
tags: [aws, solutions-architect, data-lake, lake-formation, lf-tags, quicksight, gobierno]
curso: 3 · Introduction to Designing Data Lakes on AWS
modulo: "Módulo 5: Configuraciones y capacidades adicionales de AWS Lake Formation"
audiencias: [tecnico, puente, ejecutivo]
tomo: 13
version: 1.0
updated: 2026-11-10
fuente: "Coursera · AWS Cloud Solutions Architect Professional Certificate"
---

# 🔐 Tomo 13 — Lake Formation Avanzado y QuickSight

**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[12-Procesamiento-y-Analisis|12 · Procesamiento y Análisis]] · Siguiente: [[14-Arquitectura-de-Datos-Moderna|14 · Arquitectura de Datos Moderna ➡]]

---

> [!info] 📌 ¿Por qué importa este tomo?
> Retoma Lake Formation donde lo dejó [[11-Data-Lake-con-Lake-Formation]] y suma tres piezas: **blueprints y workflows** para levantar el lago con plantillas, **fine-grained access control** a nivel de fila, columna y celda con credenciales temporales, y **LF-Tags** para que los permisos escalen sin administrarlos tabla por tabla. Cierra con **Amazon QuickSight**, la capa de visualización que convierte el lago en dashboards compartibles.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** estandarizar la creación del lago con plantillas; dar a marketing las ventas sin las finanzas de los clientes, desde el mismo dataset; gobernar por clasificación (secreto / no secreto) en vez de por tabla.
> - **Costo de hacerlo mal:** permisos "todo o nada", reglas que no escalan cuando cambia la estructura de datos, informes repartidos como CSV por correo.
> - **Pregunta que responde:** *¿cómo mantengo a las personas correctas, con los permisos correctos, en el momento correcto, sin que la administración de permisos frene el crecimiento del lago?*

> [!tip] 💡 Analogía general: el socorrista y las pulseras de colores
> El data lake administrator es el **socorrista** del lago. Las **LF-Tags** son pulseras de colores: en vez de anotar en una lista a quién deja nadar en cada zona, el socorrista pinta las zonas ("secreto", "no tan secreto") y entrega pulseras por color. Si cambia la piscina, se repintan las zonas; las pulseras siguen sirviendo.

## Mapa del módulo

| Lección | Ítems | Qué se aprende |
|---|---|---|
| Objetivos del módulo | 1 video | Blueprints, ETL, LF-Tags, QuickSight |
| Uso de planos y flujos de trabajo | 1 video | Blueprints, workflows y sus tres pasos de configuración |
| Control de acceso detallado | 1 video | Filas/columnas/celdas, credenciales temporales, LF-Tags, cross-account, roles sugeridos |
| Demo: LF-Tags | 1 video | Crear, asignar, heredar, sobrescribir y conceder permisos por tag |
| Demo: visualización con QuickSight | 1 video | Datasets, análisis, funnel chart, publicar dashboard |
| Laboratorio autoguiado (anunciado en el curso) | lab | Sin texto en el material extraído |

---

## 1. Blueprints y workflows

Audiencia: 🔧 🧭

"Cavar un hoyo y llenarlo de agua" es fácil en teoría y difícil en la práctica; lo mismo pasa con un data lake. Lake Formation entrega un conjunto de **estándares** para no tener que inventarlo todo.

### 1.1 Qué deja configurado un blueprint

Como el plano de un edificio, un **blueprint** de Lake Formation trae todo preparado desde el inicio. Lake Formation se integra con los servicios necesarios:

| Área | Qué configura |
|---|---|
| Almacenamiento | Guarda los datos en **S3 o Redshift**, según el tipo de información |
| ETL | Configura **AWS Glue** para los ETL jobs |
| Consulta | Configura **Amazon Athena** |
| Catálogo | **Rastrea y cataloga** automáticamente, facilitando el descubrimiento |
| Seguridad | Configura los **controles de seguridad y acceso** (control fino) |

### 1.2 Workflows

Debajo del blueprint están los **Lake Formation workflows**: el conjunto concreto de acciones sobre los datasets, es decir, **cómo se extraen, transforman y cargan** en el lago. Por dentro aprovechan **AWS Glue**.

### 1.3 Pasos para configurarlo

```
 ① Elegir un blueprint      ──►  ② Configurar origen   ──►  ③ Configurar el   ──►  Lake Formation procesa,
    de los que provee AWS          y destino de datos          workflow               cataloga y organiza
```

---

## 2. Fine-grained access control

Audiencia: 🔧 🧭 👔

### 2.1 Ni "nada" ni "todo"

Lake Formation evita tener que elegir entre **sin acceso** o **acceso total**.

> [!example] 📊 Caso del curso: la tienda web
> Marketing necesita los registros de ventas de todos los productos para ver los más y menos vendidos y armar una campaña. El **mismo dataset** contiene los registros financieros de los clientes, que marketing no necesita. Con Lake Formation, marketing ve solo **las filas** que le corresponden y finanzas accede a la información privada del cliente.

### 2.2 Cómo se entrega el dato

```
 Usuario: "quiero esos datos"
        │
        ▼
 Lake Formation revisa los permisos del usuario
        │ (no devuelve el dato)
        ▼
 Entrega CREDENCIALES TEMPORALES ──► otro servicio (motor) las usa ──► lee de S3 ──► devuelve el resultado
```

Para el usuario final es transparente: si tiene acceso, obtiene los datos. (El detalle paso a paso está en [[11-Data-Lake-con-Lake-Formation#4. Cómo se aplica un permiso: credential vending]].)

### 2.3 Granularidad

| Nivel | Cómo |
|---|---|
| **Filas** | Restringir qué registros ve un usuario o grupo |
| **Columnas** | Restringir qué campos ve |
| **Celdas** | Combinar filtros de filas y columnas para limitar a celdas específicas |

### 2.4 LF-Tags: permisos que escalan

Problema: ¿qué pasa si hay que cambiar permisos rápido, o si cambia la estructura de datos? La respuesta son las **Lake Formation tags (LF-Tags)**:

- Mismo formato que los tags de otros recursos de AWS: **pares clave-valor**. Ejemplos: `environment` = `qa` / `prod` / `dev`; o una clave `secret` con valor `super-duper`. Cada organización define su **estrategia de etiquetado**.
- Se asignan a una **base de datos, tabla o columna**, o a una combinación.
- Los **permisos se definen sobre los tags**: por ejemplo, el grupo IAM de developers tiene permisos sobre todo lo que lleve el tag de desarrollo. Como otras políticas basadas en tags, se ajustan fácilmente.
- **Cross-account**: se puede controlar el acceso desde una cuenta principal y conceder permisos para ver los datos en una **cuenta dedicada de data lake**.

### 2.5 Roles sugeridos

No hay una forma perfecta, pero el curso sugiere:

| Rol | Responsabilidad |
|---|---|
| **IAM administrator** | Administra todo IAM |
| **Data lake administrator** | Control total sobre todos los datos ("el socorrista") |
| **Data engineer** | Usuario sin privilegios de administrador, con algo más de control |
| **Data analyst** | Solo lectura |
| **Workflow role** | Rol que **asumen otros servicios de AWS** para actuar en nombre de uno y acceder al lago |

🎯 **Para el examen:**
- Permisos que deben escalar con muchas tablas o cambios de estructura → **LF-Tags** (tag-based access control).
- Restringir filas, columnas o celdas de una tabla en S3 → **Lake Formation fine-grained access**.
- Lake Formation no entrega el dato: entrega **credenciales temporales** al motor.
- Compartir datos entre cuentas con gobierno central → **Lake Formation cross-account**.

---

## 3. Demos y laboratorios

### 3.1 Demo — LF-Tags

**Objetivo:** controlar el acceso a la tabla de datos limpios creada en la demo de Glue Studio ([[12-Procesamiento-y-Analisis]]) mediante LF-Tags. **Servicios:** Lake Formation, Glue Data Catalog, IAM.

**Crear los tags**

1. Lake Formation → **LF-Tags** (no hay ninguno) → **Add LF-Tag**.
2. Tag 1: clave **`classification`**, valores **`secret`** y **`not-so-secret`**.
3. Tag 2: clave **`job`**, valores **`analyst`**, **`developer`**, **`admin`**.

**Asignar a la tabla**

4. **Data Catalog → Tables**; seleccionar la tabla de datos limpios → **Actions → Edit LF-Tags**.
5. Asignar `classification = secret` y `job = admin`. Es obligatorio elegir **un valor** para cada tag. **Save**.
6. Abrir la tabla → **Edit schema**: al pasar el cursor por el tag de cada columna se ve que los valores se **heredan** de la tabla. Funcionan como **valores por defecto**.

> [!tip] 💡 Por qué el default es "secreto"
> El instructor marca todo como secreto a propósito: prefiere clasificar de más por accidente que dejar como no secreto algo que sí lo era. Es el principio de *secure by default*.

**Sobrescribir por columna**

7. Seleccionar columnas menos sensibles: **name, address, city, state, phone, email**.
8. **Edit LF-Tags**: aparecen como *inherited*. Cambiarlas a valores que permitan el acceso de developers (`classification = not-so-secret`, `job = developer`) → **Save**.
9. Verificar con el cursor que los valores cambiaron en esas columnas; las demás conservan los heredados. **Save** de la tabla.

**Conceder permisos sobre los tags**

10. Los tags **no hacen nada por sí solos**: hay que dar permisos a usuarios y grupos. Ir a **LF-Tags and permissions → LF-Tag permissions**. El usuario con que se inició sesión ya tiene permisos sobre todos los tags creados.
11. **Grant permissions**. Dos opciones: permisos sobre un tag **con todos sus valores**, o sobre una **clave y un valor específicos**.
12. Elegir la opción específica, el usuario **dev**, y el par **`job = developer`**: los únicos datasets a los que ese grupo debe acceder.
13. Definir si el usuario tiene los permisos **para sí**, si puede **concederlos (grantable)** a otros, o ambos. En la demo: permisos completos sobre todo lo asociado al tag → **Grant**.
14. En la lista de permisos aparece el usuario developer con permisos sobre ese tag. El resto de asignaciones sigue el mismo procedimiento.

**Errores comunes:** crear y asignar tags sin conceder permisos sobre ellos (nadie gana acceso); olvidar que las columnas heredan el tag de la tabla; dar permisos sobre el tag completo cuando se quería un solo valor.

### 3.2 Demo — Visualización con Amazon QuickSight

**QuickSight** es un servicio de **business intelligence serverless y escalable**, integrado en AWS, que crea tablas, gráficos y otras visualizaciones.

1. Consola de QuickSight → **Datasets**: los datasets creados a partir del lago. Se pueden **importar manualmente** (como un archivo JSON) o tomar de **S3** (indicados con el ícono de S3).
2. Abrir un análisis existente: "resumen de personas", un dataset de **recursos humanos** con nombres, fechas de nacimiento, historial laboral.
3. El análisis tiene varias tablas y gráficos; en el panel izquierdo están los controles para **editar visuales** existentes o crear uno nuevo.
4. **Add** → nuevo **funnel chart**: ordenar la **antigüedad** de los empleados por **nombre** (arrastrar los campos).
5. **Publish** → ponerle nombre → publicar el **dashboard**.
6. **Compartirlo** con otras personas de la empresa.

> [!tip] 💡 Por qué publicar un dashboard
> Es mejor que repartir un CSV o un JSON y pedir que "le echen un vistazo": la misma información queda en una forma accesible para quien no domina herramientas de BI.

### 3.3 Laboratorio autoguiado

El curso anuncia un lab autoguiado para este módulo, pero sus instrucciones **no forman parte del material extraído**; solo se registra su existencia. La lección del módulo figura como "Lección sin título".

---

## 4. Arquitectura de referencia: estado al cierre del módulo

```
           ┌──────────────────────── AWS Lake Formation ─────────────────────────┐
           │ Blueprint ─► Workflow (Glue: crawlers + jobs) ─► S3 / Redshift       │
           │ LF-Tags: classification={secret,not-so-secret} · job={analyst,dev,admin} │
           │   asignados a DB / tabla / columna (herencia + sobrescritura)       │
           │ Permisos por tag ─► usuarios / grupos IAM · cross-account           │
           └───────────────┬─────────────────────────────────────────────────────┘
                           │ credenciales temporales (filtrado fila/columna/celda)
                           ▼
                Athena / Glue ──► Amazon QuickSight (análisis ─► dashboard publicado y compartido)

 Roles: IAM admin · data lake admin · data engineer · data analyst (read-only) · workflow role
```

---

## ✅ Autoevaluación

> [!question]- ¿Qué configura un blueprint de Lake Formation?
> Almacenamiento en S3 o Redshift, Glue para ETL, Athena para consulta, rastreo y catalogación automáticos, y los controles de seguridad y acceso.

> [!question]- ¿Cuál es la relación entre blueprint y workflow?
> El blueprint es la plantilla; el workflow, que está debajo, es el conjunto concreto de acciones ETL sobre los datasets y se ejecuta con AWS Glue.

> [!question]- En el ejemplo de la tienda web, ¿qué resuelve el control fino?
> Que marketing y finanzas usen el mismo dataset: marketing ve solo las filas de ventas que necesita y finanzas accede a la información financiera del cliente, sin duplicar datos.

> [!question]- ¿Cómo se limita a un usuario a celdas específicas?
> Combinando restricciones de filas y de columnas.

> [!question]- ¿Qué problema resuelven las LF-Tags frente a permisos por recurso?
> La escalabilidad: cuando hay que cambiar permisos rápido o cambia la estructura de datos, basta reetiquetar o ajustar el permiso del tag, en vez de editar cada tabla.

> [!question]- En la demo, ¿por qué las columnas nombre y email quedaron con otros valores que el resto?
> Porque se sobrescribió el tag heredado de la tabla (secret/admin) en esas columnas para dar acceso a developers; las demás mantienen el default.

> [!question]- Tras asignar tags a una tabla, ¿por qué un usuario aún no puede ver los datos?
> Porque los tags no conceden nada por sí solos: hay que otorgar LF-Tag permissions al usuario o grupo sobre la clave/valor correspondiente.

> [!question]- ¿Qué es el workflow role?
> Un rol que asumen otros servicios de AWS para actuar en nombre de uno y acceder al data lake.

## 🧾 Resumen en 10 líneas

1. Los blueprints estandarizan la creación del lago: almacenamiento, Glue, Athena, catálogo y seguridad.
2. Los workflows bajo el blueprint definen el ETL y corren con Glue.
3. Pasos: elegir blueprint, configurar origen y destino, configurar workflow.
4. Lake Formation da control fino: ni nada ni todo.
5. No devuelve el dato: entrega credenciales temporales que usa el motor para leer S3.
6. Granularidad por filas, columnas o celdas (combinación).
7. LF-Tags: pares clave-valor sobre bases, tablas y columnas; los permisos se dan sobre tags y escalan.
8. Los tags de la tabla se heredan en las columnas y se pueden sobrescribir; conviene un default restrictivo.
9. Roles sugeridos: IAM admin, data lake admin, data engineer, data analyst read-only, workflow role; soporta cross-account.
10. QuickSight: BI serverless; datasets desde S3 o archivos, análisis, visuales y dashboards publicados y compartidos.

## 📖 Recursos del módulo

El material del módulo no trae enlaces ni lecturas con URLs.

---
**Navegación:** [[00-MOC-Guia-Maestra-AWS|⬅ Volver al índice]] · Anterior: [[12-Procesamiento-y-Analisis|12 · Procesamiento y Análisis]] · Siguiente: [[14-Arquitectura-de-Datos-Moderna|14 · Arquitectura de Datos Moderna ➡]]
