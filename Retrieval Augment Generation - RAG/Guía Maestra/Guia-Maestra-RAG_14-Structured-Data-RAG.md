---
title: "Tomo 14 — Structured Data RAG: Text2SQL, Table QA y consultas sobre datos tabulares"
tags: [rag, complemento, vanguardia, text2sql, table-qa, structured-data, sql-generation, schema-linking, pandas-qa, hybrid-rag]
audiencias: [tecnico, puente, ejecutivo]
tomo: 14
version: 1.0
status: done
type: apunte
project: guia-maestra-rag
author: El Egypcio
---

# Tomo 14 — Structured Data RAG: Text2SQL, Table QA y consultas sobre datos tabulares

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13 · Frameworks de orquestación]] · Siguiente → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]

---

> [!info] 🎯 ¿De qué trata este tomo?
> Este tomo cubre el territory donde RAG **deja de ser recuperación de documentos y se convierte en consulta de datos estructurados**: bases de datos relacionales, data warehouses, hojas de cálculo con esquema definido. El pipeline es fundamentalmente distinto — no hay chunks, no hay embeddings semánticos del contenido, no hay reranking de pasajes. Hay **esquemas, tablas, columnas, relaciones y SQL** (o su equivalente programático en pandas).
>
> Si los Tomos 1–13 cubren el mundo de "pregúntale a tus documentos", este tomo cubre el mundo de **"pregúntale a tus datos"**.

> [!abstract] 👔 Impacto ejecutivo
> - **Decisiones que habilita:** desplegar analytics self-service donde usuarios de negocio consultan datos en lenguaje natural sin saber SQL; elegir entre Text2SQL, Table QA o RAG documental según el caso; presupuestar el esfuerzo de un chatbot sobre el data warehouse.
> - **Costo o riesgo de hacerlo mal:** queries SQL incorrectas que generan números falsos y decisiones equivocadas — peor que no responder, porque el usuario *confía* en que el dato es correcto. O peor: un LLM que ejecuta `DROP TABLE` porque nadie puso sandboxing.
> - **Pregunta que responde:** *"¿Cómo permito que la gente de negocio consulte nuestras bases de datos en español sin intermediarios, sin romper nada y sin inventar números?"*

---

> [!tip] 💡 Analogía general
> Imagina que tienes un **bibliotecario** (RAG documental) y un **analista de datos** (Structured Data RAG). Ambos responden preguntas, pero de formas completamente diferentes:
>
> - El bibliotecario busca el párrafo relevante en miles de documentos y te lo cita.
> - El analista abre una base de datos, escribe una consulta, ejecuta el cálculo y te da el número exacto.
>
> Pedirle al bibliotecario "¿cuáles fueron las ventas totales del Q3?" es tan absurdo como pedirle al analista "¿cuál es nuestra política de devoluciones?". **Son herramientas distintas para preguntas distintas.** Este tomo te enseña cuándo necesitas al analista.

> [!example] 💼 Caso de negocio: Analytics Self-Service
> **Escenario:** Una empresa de retail tiene un data warehouse con 150 tablas (ventas, inventario, clientes, proveedores). Los analistas de negocio hacen 200+ solicitudes al mes al equipo de BI pidiendo reportes ad-hoc. Cada solicitud tarda 2–5 días en resolverse.
>
> **Solución:** Un chatbot Text2SQL que permite a los analistas de negocio preguntar en lenguaje natural: *"¿Cuánto vendimos en la región norte en julio, desglosado por categoría?"* — y obtener la respuesta en segundos.
>
> **Resultado:** 60–70% de consultas ad-hoc resueltas sin intervención de BI. Tiempo promedio de respuesta: 15 segundos vs. 3 días. El equipo de BI se dedica a problemas complejos, no a `SELECT ... GROUP BY`.
>
> **Riesgo controlado:** Las queries sólo tienen permisos de `SELECT` sobre vistas aprobadas — imposible modificar datos.

---

## 1. 🎯 ¿Por qué un tomo separado? El RAG estructurado es otro animal

Audiencia: 🔧 🧭 👔

### 1.1 La diferencia fundamental

El RAG que documentan los Tomos 1–12 asume un pipeline:

```
Pregunta → Embed → Buscar chunks similares → Reranking → Generar respuesta con contexto
```

Ese pipeline **no aplica** cuando la respuesta vive en una base de datos relacional. No tiene sentido hacer embedding de filas de una tabla de ventas — la pregunta "¿cuáles fueron las ventas totales de julio?" no se responde con *similitud semántica*, se responde con `SUM(ventas) WHERE mes = 7`.

```
┌──────────────────────────────────────────────────────────────────────┐
│              RAG DOCUMENTAL vs RAG ESTRUCTURADO                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   RAG Documental (Tomos 1-12)          Structured Data RAG (Tomo 14)│
│   ─────────────────────────────        ─────────────────────────────│
│                                                                      │
│   Fuente: documentos, PDFs, web        Fuente: tablas, SQL DBs, CSV │
│   Indexado: embeddings + chunks        Indexado: schema metadata     │
│   Retrieval: vector similarity         Retrieval: schema linking     │
│   Contexto: pasajes de texto           Contexto: DDL + sample rows   │
│   Generación: respuesta en prosa       Generación: SQL/código        │
│   Verificación: fuente citada          Verificación: resultado numérico│
│   Falla típica: hallucination          Falla típica: SQL incorrecto  │
│                                                                      │
│   "¿Cuál es nuestra política          "¿Cuánto vendimos en julio?"  │
│    de devoluciones?"                                                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### 1.2 El dato duro

🔧 [Técnico] Los benchmarks de Text2SQL (Spider, BIRD) miden **execution accuracy**: si el SQL generado produce el mismo resultado que el SQL gold. Los mejores sistemas (2024) alcanzan ~87% en Spider y ~65% en BIRD (que es más realista). Eso significa que **1 de cada 3 queries en bases complejas puede ser incorrecta** — y el usuario no tiene forma de saberlo sin inspeccionar el SQL.

👔 [Ejecutivo] Traducción: la tecnología funciona bien para el 65–85% de preguntas comunes, pero requiere guardrails serios para el resto. No es "deploy and forget" — es "deploy with review mechanisms".

---

## 2. 🔧 El pipeline Text2SQL

Audiencia: 🔧 🧭

### 2.1 Arquitectura general

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        PIPELINE TEXT2SQL                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐    ┌───────────────┐    ┌──────────────┐    ┌──────────┐ │
│  │ Pregunta │───▶│ Schema Linking │───▶│ SQL Generation│───▶│ Execution│ │
│  │ del      │    │               │    │              │    │          │ │
│  │ usuario  │    │ "¿Qué tablas  │    │ LLM genera   │    │ Ejecutar │ │
│  │          │    │  y columnas   │    │ el SQL       │    │ contra   │ │
│  │          │    │  son          │    │              │    │ la DB    │ │
│  │          │    │  relevantes?" │    │              │    │          │ │
│  └──────────┘    └───────────────┘    └──────────────┘    └────┬─────┘ │
│                                                                 │       │
│                                                                 ▼       │
│                  ┌──────────────────┐    ┌───────────────────────┐      │
│                  │ Answer Synthesis │◀───│ Resultados (rows/agg) │      │
│                  │                  │    └───────────────────────┘      │
│                  │ "Las ventas de   │                                    │
│                  │  julio fueron    │                                    │
│                  │  $2.3M, un 15%   │                                    │
│                  │  más que junio"  │                                    │
│                  └──────────────────┘                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Paso 1: Schema Linking

🔧 [Técnico] El schema linking es el paso **más crítico y menos visible**: dado el lenguaje natural del usuario, identificar qué tablas y columnas de la base de datos son relevantes.

**¿Por qué es difícil?**
- "Ventas" puede ser la tabla `sales`, `transactions`, `orders`, o una vista `v_monthly_revenue`
- "Clientes activos" puede significar `WHERE last_purchase_date > NOW() - INTERVAL '90 days'` o `WHERE status = 'active'`
- En una base con 200 tablas, pasar todo el schema al LLM excede el context window (o lo distrae)

**Estrategias comunes:**

| Estrategia | Cómo funciona | Cuándo usar |
|---|---|---|
| **Full schema in context** | Pasar todo el DDL al prompt | Bases pequeñas (< 20 tablas) |
| **Retrieval-based** | Embeddings de descripciones de tablas/columnas, buscar las más relevantes | Bases medianas (20–100 tablas) |
| **Two-stage** | Primero un LLM filtra tablas relevantes, luego otro genera SQL solo con esas | Bases grandes (100+ tablas) |
| **Metadata catalog** | Mantener un catálogo con sinónimos, descripciones de negocio y mapeos | Enterprise — máxima precisión |

**Código ejemplo — schema linking con embeddings:**

```python
"""
Schema Linking con embeddings: encontrar tablas/columnas relevantes
para la pregunta del usuario.
"""
from openai import OpenAI
import numpy as np
from typing import list

client = OpenAI()

# Catálogo de schema con descripciones de negocio
SCHEMA_CATALOG = [
    {
        "table": "sales",
        "description": "Transacciones de venta. Cada fila es una venta individual.",
        "columns": [
            {"name": "sale_id", "type": "INT", "desc": "ID único de la venta"},
            {"name": "sale_date", "type": "DATE", "desc": "Fecha de la transacción"},
            {"name": "amount", "type": "DECIMAL", "desc": "Monto total en MXN"},
            {"name": "region_id", "type": "INT", "desc": "FK a regions"},
            {"name": "product_id", "type": "INT", "desc": "FK a products"},
        ]
    },
    {
        "table": "products",
        "description": "Catálogo de productos. Incluye categoría y precio base.",
        "columns": [
            {"name": "product_id", "type": "INT", "desc": "ID único del producto"},
            {"name": "name", "type": "VARCHAR", "desc": "Nombre del producto"},
            {"name": "category", "type": "VARCHAR", "desc": "Categoría (Electrónica, Ropa, etc.)"},
            {"name": "base_price", "type": "DECIMAL", "desc": "Precio de lista sin descuentos"},
        ]
    },
    {
        "table": "regions",
        "description": "Regiones geográficas de operación.",
        "columns": [
            {"name": "region_id", "type": "INT", "desc": "ID de la región"},
            {"name": "name", "type": "VARCHAR", "desc": "Nombre (Norte, Sur, Centro, etc.)"},
        ]
    },
]


def get_embedding(text: str) -> list[float]:
    """Obtener embedding de OpenAI."""
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding


def build_schema_embeddings(catalog: list[dict]) -> list[dict]:
    """Pre-computar embeddings para cada tabla del catálogo."""
    enriched = []
    for table_info in catalog:
        # Construir texto rico para embedding
        cols_text = ", ".join(
            f"{c['name']} ({c['desc']})" for c in table_info["columns"]
        )
        full_text = f"Tabla: {table_info['table']}. {table_info['description']} Columnas: {cols_text}"
        
        enriched.append({
            **table_info,
            "embedding": get_embedding(full_text),
            "full_text": full_text,
        })
    return enriched


def find_relevant_tables(question: str, schema_embeddings: list[dict], top_k: int = 3) -> list[dict]:
    """Encontrar las tablas más relevantes para la pregunta."""
    q_embedding = np.array(get_embedding(question))
    
    scored = []
    for table in schema_embeddings:
        t_embedding = np.array(table["embedding"])
        # Cosine similarity
        similarity = np.dot(q_embedding, t_embedding) / (
            np.linalg.norm(q_embedding) * np.linalg.norm(t_embedding)
        )
        scored.append((similarity, table))
    
    scored.sort(key=lambda x: x[0], reverse=True)
    return [t for _, t in scored[:top_k]]


# Uso
schema_embeddings = build_schema_embeddings(SCHEMA_CATALOG)
question = "¿Cuánto vendimos en la región norte en julio?"

relevant_tables = find_relevant_tables(question, schema_embeddings, top_k=2)
print("Tablas relevantes:")
for t in relevant_tables:
    print(f"  - {t['table']}: {t['description']}")
```

### 2.3 Paso 2: SQL Generation

🔧 [Técnico] Una vez identificadas las tablas relevantes, el LLM genera SQL a partir de la pregunta + el DDL filtrado.

**El prompt es clave.** Un prompt efectivo para SQL generation incluye:

1. **DDL de las tablas relevantes** (con tipos, constraints, FKs)
2. **Sample rows** (3–5 filas de ejemplo para que el LLM entienda el formato de los datos)
3. **Instrucciones del dialecto** (PostgreSQL vs MySQL vs BigQuery — la sintaxis difiere)
4. **Restricciones** (solo SELECT, no subconsultas correlacionadas, etc.)

```python
"""
SQL Generation: construir el prompt y generar SQL con un LLM.
"""

def build_ddl_context(tables: list[dict]) -> str:
    """Construir DDL simplificado para el prompt."""
    ddl_parts = []
    for table in tables:
        cols = ",\n    ".join(
            f"{c['name']} {c['type']}  -- {c['desc']}"
            for c in table["columns"]
        )
        ddl_parts.append(
            f"CREATE TABLE {table['table']} (\n    {cols}\n);"
        )
    return "\n\n".join(ddl_parts)


def generate_sql(question: str, relevant_tables: list[dict], dialect: str = "PostgreSQL") -> str:
    """Generar SQL a partir de la pregunta y el schema."""
    ddl = build_ddl_context(relevant_tables)
    
    system_prompt = f"""Eres un experto en SQL ({dialect}). 
Tu tarea es convertir preguntas en lenguaje natural a queries SQL correctas.

REGLAS ESTRICTAS:
- Solo genera SELECT statements. NUNCA INSERT, UPDATE, DELETE, DROP, ALTER.
- Usa SOLO las tablas y columnas proporcionadas en el schema.
- Si la pregunta es ambigua, haz suposiciones razonables y documéntalas en un comentario SQL.
- Responde SOLO con el SQL, sin explicaciones adicionales.
- Usa alias descriptivos para legibilidad.
"""

    user_prompt = f"""## Schema disponible:

{ddl}

## Datos de ejemplo:
-- sales: (1, '2024-07-15', 1500.00, 1, 101), (2, '2024-07-20', 2300.00, 2, 102)
-- regions: (1, 'Norte'), (2, 'Sur'), (3, 'Centro')
-- products: (101, 'Laptop HP', 'Electrónica', 15000.00), (102, 'Camisa Polo', 'Ropa', 450.00)

## Pregunta del usuario:
{question}

## SQL:"""

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ],
        temperature=0.0,  # Determinismo máximo para SQL
        max_tokens=500,
    )
    
    sql = response.choices[0].message.content.strip()
    # Limpiar markdown si el LLM lo envuelve en ```sql ... ```
    if sql.startswith("```"):
        sql = sql.split("\n", 1)[1].rsplit("```", 1)[0].strip()
    
    return sql


# Ejemplo
question = "¿Cuánto vendimos en la región norte en julio de 2024?"
sql = generate_sql(question, relevant_tables)
print(f"SQL generado:\n{sql}")
# Esperado algo como:
# SELECT SUM(s.amount) AS total_ventas
# FROM sales s
# JOIN regions r ON s.region_id = r.region_id
# WHERE r.name = 'Norte'
#   AND s.sale_date >= '2024-07-01'
#   AND s.sale_date < '2024-08-01';
```

### 2.4 Paso 3: Execution + Answer Synthesis

🔧 [Técnico] Ejecutar el SQL generado y convertir los resultados crudos en una respuesta legible.

```python
"""
Execution + Answer Synthesis con sandboxing.
"""
import sqlalchemy
from sqlalchemy import text


def execute_sql_safe(sql: str, connection_string: str, timeout_seconds: int = 30) -> dict:
    """
    Ejecutar SQL con sandboxing básico.
    
    Validaciones de seguridad:
    1. Solo permite SELECT
    2. Timeout para evitar queries costosas
    3. Límite de filas en el resultado
    """
    # Validación de seguridad: solo SELECT
    sql_upper = sql.strip().upper()
    FORBIDDEN = ["INSERT", "UPDATE", "DELETE", "DROP", "ALTER", "CREATE", "TRUNCATE", "EXEC", "GRANT"]
    for keyword in FORBIDDEN:
        if keyword in sql_upper.split():  # split para evitar falsos positivos en nombres de columna
            return {"error": f"Operación prohibida detectada: {keyword}", "sql": sql}
    
    if not sql_upper.startswith("SELECT") and not sql_upper.startswith("WITH"):
        return {"error": "Solo se permiten queries SELECT o WITH (CTEs)", "sql": sql}
    
    # Ejecutar con timeout
    engine = sqlalchemy.create_engine(
        connection_string,
        connect_args={"options": f"-c statement_timeout={timeout_seconds * 1000}"}  # PostgreSQL
    )
    
    try:
        with engine.connect() as conn:
            result = conn.execute(text(sql))
            columns = list(result.keys())
            rows = result.fetchmany(1000)  # Límite de filas
            
            return {
                "success": True,
                "columns": columns,
                "rows": [dict(zip(columns, row)) for row in rows],
                "row_count": len(rows),
                "sql": sql,
            }
    except Exception as e:
        return {"error": str(e), "sql": sql}


def synthesize_answer(question: str, query_result: dict) -> str:
    """Convertir resultados SQL en respuesta legible."""
    if "error" in query_result:
        return f"No pude ejecutar la consulta: {query_result['error']}"
    
    # Formatear resultados para el LLM
    rows_preview = query_result["rows"][:20]  # Max 20 filas para el contexto
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": (
                    "Eres un analista de datos. Responde la pregunta del usuario "
                    "basándote EXCLUSIVAMENTE en los resultados de la query SQL. "
                    "Sé conciso, incluye los números relevantes, y si hay tendencias "
                    "interesantes en los datos, menciónalas brevemente."
                )
            },
            {
                "role": "user",
                "content": (
                    f"Pregunta original: {question}\n\n"
                    f"SQL ejecutado: {query_result['sql']}\n\n"
                    f"Resultados ({query_result['row_count']} filas):\n"
                    f"{rows_preview}"
                )
            },
        ],
        temperature=0.3,
    )
    
    return response.choices[0].message.content


# Pipeline completo
question = "¿Cuánto vendimos en la región norte en julio de 2024?"
CONNECTION_STRING = "postgresql://readonly_user:pass@localhost:5432/analytics"

# 1. Schema linking (ya hecho arriba)
relevant = find_relevant_tables(question, schema_embeddings, top_k=2)

# 2. SQL generation
sql = generate_sql(question, relevant)
print(f"SQL: {sql}")

# 3. Execution
result = execute_sql_safe(sql, CONNECTION_STRING)

# 4. Answer synthesis
answer = synthesize_answer(question, result)
print(f"\nRespuesta: {answer}")
```

> [!warning] ⚠️ Seguridad: el sandboxing no es opcional
> El código anterior implementa validación mínima. En producción necesitas:
> - **Usuario de DB con permisos de solo lectura** (defense in depth)
> - **Query review** para queries complejas (JOINs con más de 3 tablas, subqueries)
> - **Resource limits** a nivel de DB: `statement_timeout`, `max_rows`, `work_mem`
> - **Audit log** de cada query ejecutada
> - Considerar **vistas predefinidas** en vez de acceso directo a tablas base

---

## 3. 📊 Table QA (sin SQL)

Audiencia: 🔧 🧭

No toda pregunta sobre datos tabulares requiere SQL. Hay dos enfoques alternativos que evitan la generación de SQL:

### 3.1 Pandas-based QA: el LLM genera código Python

En vez de SQL, el LLM genera código `pandas` que se ejecuta sobre un DataFrame.

```python
"""
Table QA con Pandas: el LLM genera código pandas.
Ideal para archivos CSV/Excel que no están en una DB.
"""
import pandas as pd


def pandas_qa(question: str, df: pd.DataFrame, df_name: str = "df") -> str:
    """
    El LLM genera código pandas para responder la pregunta.
    """
    # Proveer schema + sample al LLM
    schema_info = f"""DataFrame '{df_name}':
- Shape: {df.shape}
- Columnas y tipos:
{df.dtypes.to_string()}

- Primeras 5 filas:
{df.head().to_string()}

- Estadísticas numéricas:
{df.describe().to_string()}
"""
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": (
                    "Genera código Python (pandas) que responda la pregunta del usuario. "
                    "El DataFrame ya está cargado en la variable 'df'. "
                    "Tu código debe terminar asignando el resultado a una variable llamada 'result'. "
                    "Solo genera código, sin explicaciones. "
                    "NUNCA uses exec, eval, os, subprocess, ni imports peligrosos."
                )
            },
            {
                "role": "user",
                "content": f"## Schema del DataFrame:\n{schema_info}\n\n## Pregunta:\n{question}"
            },
        ],
        temperature=0.0,
    )
    
    code = response.choices[0].message.content.strip()
    if code.startswith("```"):
        code = code.split("\n", 1)[1].rsplit("```", 1)[0].strip()
    
    # Ejecutar en sandbox (namespace restringido)
    namespace = {"df": df, "pd": pd, "np": np}
    
    # Validación básica de seguridad
    FORBIDDEN_PATTERNS = ["import os", "import subprocess", "exec(", "eval(", "__", "open("]
    for pattern in FORBIDDEN_PATTERNS:
        if pattern in code:
            return f"Código rechazado por seguridad: contiene '{pattern}'"
    
    try:
        exec(code, namespace)
        result = namespace.get("result", "No se encontró variable 'result'")
        return str(result)
    except Exception as e:
        return f"Error ejecutando código: {e}\n\nCódigo generado:\n{code}"


# Ejemplo
df_ventas = pd.DataFrame({
    "fecha": pd.date_range("2024-01-01", periods=365, freq="D"),
    "region": np.random.choice(["Norte", "Sur", "Centro"], 365),
    "monto": np.random.uniform(1000, 50000, 365),
    "categoria": np.random.choice(["Electrónica", "Ropa", "Hogar"], 365),
})

answer = pandas_qa("¿Cuál fue el promedio de ventas diarias por región en julio?", df_ventas)
print(answer)
```

### 3.2 Direct Table Reasoning: la tabla en el context window

Para tablas pequeñas (< 100 filas), a veces lo más simple es pasar la tabla completa al LLM y dejar que razone directamente.

```python
"""
Direct Table Reasoning: pasar la tabla directamente al LLM.
Sin código generado — el LLM razona sobre los datos.
"""


def direct_table_qa(question: str, df: pd.DataFrame, max_rows: int = 50) -> str:
    """Pasar la tabla al LLM y dejar que razone directamente."""
    if len(df) > max_rows:
        table_str = df.head(max_rows).to_markdown(index=False)
        note = f"\n\n(Mostrando {max_rows} de {len(df)} filas)"
    else:
        table_str = df.to_markdown(index=False)
        note = ""
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": (
                    "Eres un analista de datos experto. Responde la pregunta "
                    "basándote SOLO en la tabla proporcionada. Si necesitas hacer "
                    "cálculos (sumas, promedios, etc.), hazlos paso a paso."
                )
            },
            {
                "role": "user",
                "content": f"## Tabla:\n{table_str}{note}\n\n## Pregunta:\n{question}"
            },
        ],
        temperature=0.0,
    )
    
    return response.choices[0].message.content
```

### 3.3 Tabla de decisión: ¿cuándo usar qué?

| Criterio | Text2SQL | Pandas QA | Direct Reasoning |
|---|---|---|---|
| **Fuente de datos** | Base de datos relacional | CSV/Excel/DataFrame | Tabla pequeña (< 50 filas) |
| **Tamaño de datos** | Ilimitado (la DB maneja) | Medio (cabe en RAM) | Pequeño (cabe en context window) |
| **Complejidad de queries** | Alta (JOINs, CTEs, window functions) | Media (groupby, merge) | Baja (conteo, comparación simple) |
| **Precisión numérica** | ✅ Exacta (la DB calcula) | ✅ Exacta (pandas calcula) | ⚠️ El LLM puede equivocarse en aritmética |
| **Latencia** | Media (genera SQL + ejecuta) | Media (genera código + ejecuta) | Baja (una sola llamada al LLM) |
| **Riesgo de seguridad** | Alto (SQL injection, destructive queries) | Medio (code execution) | Bajo (no ejecuta nada) |
| **Setup requerido** | Conexión a DB + permisos + schema | Archivo cargado en memoria | Solo el LLM |

> [!tip] 💡 Regla rápida
> - **¿Datos en una DB relacional?** → Text2SQL
> - **¿CSV/Excel con > 50 filas?** → Pandas QA
> - **¿Tabla chica que quieres explorar rápido?** → Direct Reasoning
> - **¿No estás seguro?** → Empieza con Direct Reasoning (el más simple) y escala si falla

---

## 4. 🔀 Hybrid: tabular + documental

Audiencia: 🔧 🧭 👔

### 4.1 El problema

Muchas preguntas reales cruzan los dos mundos:

- *"¿Cuánto vendimos del producto X y cuál es su política de garantía?"* → SQL para ventas, RAG documental para la política.
- *"¿Qué clientes del segmento premium han reclamado y qué dice nuestro proceso de escalamiento?"* → SQL para identificar clientes, documentos para el proceso.

### 4.2 El router: decidir a dónde va la query

```
┌───────────────────────────────────────────────────────────────────────────┐
│                     ROUTER HYBRID: SQL + DOCS                             │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│                        ┌────────────────┐                                 │
│                        │  Pregunta del  │                                 │
│                        │    usuario     │                                 │
│                        └───────┬────────┘                                 │
│                                │                                          │
│                                ▼                                          │
│                    ┌───────────────────────┐                              │
│                    │   CLASSIFIER / LLM    │                              │
│                    │                       │                              │
│                    │ "¿Esta pregunta       │                              │
│                    │  necesita datos       │                              │
│                    │  numéricos, contexto  │                              │
│                    │  documental, o ambos?"│                              │
│                    └───┬──────┬────────┬───┘                              │
│                        │      │        │                                  │
│              ┌─────────┘      │        └──────────┐                       │
│              │                │                   │                        │
│              ▼                ▼                   ▼                        │
│    ┌─────────────┐  ┌──────────────┐   ┌──────────────────┐             │
│    │  SQL Path   │  │  BOTH Paths  │   │  Document Path   │             │
│    │             │  │              │   │                  │             │
│    │ Schema Link │  │ SQL + Vector │   │ Vector Search    │             │
│    │ → Generate  │  │ en paralelo  │   │ → Rerank         │             │
│    │ → Execute   │  │              │   │ → Contexto       │             │
│    └──────┬──────┘  └──────┬───────┘   └────────┬─────────┘             │
│           │                │                    │                         │
│           └────────────────┼────────────────────┘                         │
│                            ▼                                              │
│                  ┌──────────────────┐                                     │
│                  │  SYNTHESIS LLM   │                                     │
│                  │                  │                                     │
│                  │ Combinar datos + │                                     │
│                  │ contexto en una  │                                     │
│                  │ respuesta final  │                                     │
│                  └──────────────────┘                                     │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Implementación del router

```python
"""
Router Hybrid: clasificar la pregunta y dirigir al pipeline correcto.
"""
from enum import Enum


class QueryType(Enum):
    SQL = "sql"           # Datos numéricos, agregaciones, métricas
    DOCUMENT = "document" # Políticas, procedimientos, contexto cualitativo
    HYBRID = "hybrid"     # Necesita ambos


def classify_query(question: str) -> QueryType:
    """Clasificar la pregunta para decidir el pipeline."""
    response = client.chat.completions.create(
        model="gpt-4o-mini",  # Modelo barato para clasificación
        messages=[
            {
                "role": "system",
                "content": """Clasifica la pregunta del usuario en una de tres categorías:

- SQL: requiere datos numéricos, métricas, conteos, sumas, tendencias cuantitativas.
  Ejemplos: "¿cuánto vendimos?", "¿cuántos clientes hay?", "top 10 productos"
  
- DOCUMENT: requiere información cualitativa, políticas, procedimientos, guías.
  Ejemplos: "¿cuál es la política de devoluciones?", "¿cómo se escala un ticket?"

- HYBRID: necesita AMBOS — datos numéricos Y contexto documental.
  Ejemplos: "¿Cuánto vendimos del producto X y cuál es su garantía?"

Responde SOLO con una palabra: SQL, DOCUMENT, o HYBRID."""
            },
            {"role": "user", "content": question},
        ],
        temperature=0.0,
        max_tokens=10,
    )
    
    classification = response.choices[0].message.content.strip().upper()
    
    mapping = {"SQL": QueryType.SQL, "DOCUMENT": QueryType.DOCUMENT, "HYBRID": QueryType.HYBRID}
    return mapping.get(classification, QueryType.DOCUMENT)  # Default seguro


def hybrid_pipeline(question: str) -> str:
    """Pipeline completo con routing."""
    query_type = classify_query(question)
    
    results = {}
    
    if query_type in (QueryType.SQL, QueryType.HYBRID):
        # Ejecutar pipeline SQL
        tables = find_relevant_tables(question, schema_embeddings)
        sql = generate_sql(question, tables)
        sql_result = execute_sql_safe(sql, CONNECTION_STRING)
        results["sql"] = sql_result
    
    if query_type in (QueryType.DOCUMENT, QueryType.HYBRID):
        # Ejecutar pipeline documental (RAG clásico)
        # ... vector search + reranking (ver Tomos 3-6)
        results["documents"] = retrieve_and_rerank(question)  # Implementación de Tomos previos
    
    # Synthesis final
    return synthesize_hybrid_answer(question, results)
```

🧭 [Puente] El router es el **punto de decisión arquitectónico clave** en un sistema hybrid. Si clasifica mal, toda la respuesta falla. Por eso conviene que el clasificador sea conservador: ante la duda, usar HYBRID (buscar en ambos) es más lento pero más seguro.

---

## 5. ⚠️ Desafíos específicos del Structured Data RAG

Audiencia: 🔧 🧭 👔

### 5.1 Ambigüedad del lenguaje natural

> [!danger] 🚨 El problema más subestimado
> "Ventas del mes pasado" parece simple. Pero:
> - **¿Qué mes?** ¿Relativo a hoy? ¿Al último mes con datos completos?
> - **¿Qué tabla?** ¿`sales`? ¿`invoices`? ¿`orders`? (No son lo mismo)
> - **¿Qué moneda?** Si operas en múltiples países, ¿MXN? ¿USD? ¿Convertida?
> - **¿Brutas o netas?** ¿Con IVA o sin IVA? ¿Incluye devoluciones?
> - **¿Qué granularidad?** ¿Total? ¿Por día? ¿Por tienda?
>
> Un analista humano haría 3–4 preguntas de clarificación antes de escribir el SQL. Un sistema automatizado debe **o bien asumir defaults documentados, o bien preguntar al usuario**.

**Estrategias para manejar ambigüedad:**

| Estrategia | Implementación | Trade-off |
|---|---|---|
| **Defaults documentados** | "Ventas" siempre significa `invoices.amount_net_mxn` | Rápido pero puede ser incorrecto |
| **Clarificación interactiva** | El sistema pregunta antes de generar SQL | Más preciso pero más lento |
| **Assumptions explícitas** | Genera SQL + nota: "Asumí ventas netas en MXN" | Balance — el usuario verifica |
| **Glosario de negocio** | Mapping explícito: "ventas" → definición + tabla + columna | Máxima precisión, alto setup |

### 5.2 Schema complexity: bases con 200+ tablas

🔧 [Técnico] Pasar el DDL completo de 200 tablas a un LLM es:
1. **Imposible** si excede el context window (200 tablas × 10 columnas × 50 chars = ~100K tokens)
2. **Contraproducente** incluso si cabe: el LLM se distrae con tablas irrelevantes

**Solución: schema filtering progresivo**

```
Nivel 1: Categorización estática
─────────────────────────────────
  200 tablas → 15 "dominios" (Ventas, Inventario, RRHH, Finanzas...)
  La pregunta se clasifica en 1-2 dominios → ~30 tablas candidatas

Nivel 2: Schema linking por embeddings
──────────────────────────────────────
  30 tablas candidatas → top 5 por similitud semántica

Nivel 3: Context final
──────────────────────
  5 tablas → DDL completo + sample rows → prompt del LLM
```

### 5.3 Safety: el LLM puede generar queries destructivas

> [!danger] 🚨 Esto no es teórico
> Un LLM al que le pides "borra los registros duplicados" **sí generará un DELETE**. Y si tiene permisos para ejecutarlo, lo ejecutará sin pestañear.
>
> **Controles obligatorios (defense in depth):**
> 1. **DB user read-only** — conexión con permisos `SELECT` únicamente
> 2. **Query validation** — parsear el SQL y rechazar todo lo que no sea SELECT/WITH
> 3. **Vistas, no tablas base** — el LLM solo ve vistas curadas, no la DB real
> 4. **Row-level security** — cada usuario ve solo los datos que le corresponden
> 5. **Timeout + resource limits** — prevenir `SELECT * FROM tabla_de_100M_filas` sin WHERE
> 6. **Audit log** — registrar cada query generada y ejecutada

```python
"""
Validación de seguridad de SQL generado.
"""
import sqlparse


def validate_sql_safety(sql: str) -> tuple[bool, str]:
    """
    Validar que el SQL es seguro para ejecutar.
    Returns: (is_safe, reason)
    """
    parsed = sqlparse.parse(sql)
    
    for statement in parsed:
        stmt_type = statement.get_type()
        
        # Solo permitir SELECT (y CTEs que son WITH...SELECT)
        if stmt_type not in ("SELECT", None):  # None para CTEs complejos
            return False, f"Tipo de statement no permitido: {stmt_type}"
    
    # Buscar keywords peligrosas
    sql_upper = sql.upper()
    dangerous_keywords = [
        "DROP", "DELETE", "INSERT", "UPDATE", "ALTER", "CREATE",
        "TRUNCATE", "GRANT", "REVOKE", "EXEC", "EXECUTE",
        "INTO OUTFILE", "INTO DUMPFILE", "LOAD_FILE",
    ]
    
    for keyword in dangerous_keywords:
        # Buscar como palabra completa (no como parte de un nombre de columna)
        if f" {keyword} " in f" {sql_upper} " or sql_upper.startswith(keyword):
            return False, f"Keyword peligrosa detectada: {keyword}"
    
    # Validar que no hay stacking de statements (;)
    if sql.count(";") > 1:
        return False, "Múltiples statements detectados (posible SQL injection)"
    
    return True, "OK"
```

### 5.4 Evaluation: ¿cómo saber si el SQL generado es correcto?

🔧 [Técnico] Evaluar un sistema Text2SQL es más complejo que evaluar RAG documental. No basta con "¿la respuesta suena bien?" — necesitas métricas formales:

| Métrica | Qué mide | Limitación |
|---|---|---|
| **Execution Accuracy (EX)** | ¿El SQL generado produce el mismo resultado que el SQL gold? | Dos SQLs correctos pueden dar distinto orden de filas |
| **Exact Match (EM)** | ¿El SQL generado es idéntico al gold? | Demasiado estricto: `WHERE a=1 AND b=2` ≠ `WHERE b=2 AND a=1` pero ambos son correctos |
| **Valid Efficiency Score (VES)** | EX + penalización por queries ineficientes | Requiere medir tiempo de ejecución |
| **Semantic Equivalence** | ¿Son semánticamente equivalentes? (por ejecución sobre múltiples DBs de test) | Costoso de implementar |

> [!important] 🎯 La métrica que importa en producción
> En un entorno real, lo que importa es: **¿el usuario obtuvo el número correcto?** Esto combina:
> 1. ¿El SQL es sintácticamente válido? (no falla al ejecutar)
> 2. ¿Consulta las tablas correctas?
> 3. ¿Los filtros son correctos? (fechas, regiones, categorías)
> 4. ¿La agregación es correcta? (SUM vs COUNT vs AVG)
>
> Un buen sistema de evaluación continua incluye un **golden set** de preguntas con SQL verificado humano, más **feedback del usuario** ("este número no parece correcto").

---

## 6. 🛠️ Frameworks y herramientas

Audiencia: 🔧 🧭

### 6.1 Tabla comparativa

| Framework / Tool | Enfoque | Fortaleza principal | Limitación principal | Estado (2024) |
|---|---|---|---|---|
| **LangChain SQL Agent** | Agent que itera: genera SQL → ejecuta → corrige si falla | Integración con ecosistema LangChain, auto-corrección | Abstracción opaca, difícil de debuggear (ver [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex\|Tomo 13]]) | Producción |
| **LlamaIndex NLSQLTableQueryEngine** | Pipeline estructurado: schema → SQL → resultado | Buen schema management, composable con otros query engines | Menos flexible para queries complejas | Producción |
| **Vanna.ai** | Fine-tuning + RAG sobre SQL: aprende de queries pasadas | Mejora con uso, aprende el "dialecto" de tu empresa | Requiere corpus de queries históricas | Producción |
| **DIN-SQL** | Decomposed In-Context: descompone la query en sub-tareas | State-of-art en benchmarks (2023), schema linking sofisticado | Paper académico, no es librería lista | Research |
| **DAIL-SQL** | Selección inteligente de few-shot examples para SQL | Mejor selection strategy para in-context learning | Requiere pool de examples pre-catalogados | Research |
| **SQLCoder (Defog)** | Modelo fine-tuned específicamente para Text2SQL | Open-source, especializado, no depende de APIs | Menor capacidad de razonamiento que GPT-4 | Open-source |

### 6.2 Ejemplo con LangChain SQL Agent

```python
"""
LangChain SQL Agent — ejemplo funcional.
Referencia: docs.langchain.com/docs/use_cases/sql
"""
from langchain_community.utilities import SQLDatabase
from langchain_community.agent_toolkits import create_sql_agent
from langchain_openai import ChatOpenAI


# Conexión a la base de datos (solo lectura)
db = SQLDatabase.from_uri(
    "postgresql://readonly_user:pass@localhost:5432/analytics",
    include_tables=["sales", "products", "regions", "customers"],  # Whitelist explícita
    sample_rows_in_table_info=3,  # Filas de ejemplo en el schema
)

# Crear el agent
llm = ChatOpenAI(model="gpt-4o", temperature=0)

agent = create_sql_agent(
    llm=llm,
    db=db,
    agent_type="openai-tools",
    verbose=True,  # Para debugging — ver el SQL generado
    max_iterations=5,  # Límite de intentos de corrección
)

# Ejecutar
response = agent.invoke({
    "input": "¿Cuáles son los 5 productos más vendidos del último trimestre por unidades?"
})
print(response["output"])
```

> [!warning] ⚠️ Sobre el agent de LangChain
> El SQL Agent de LangChain es **conveniente para prototipos**, pero hereda todos los problemas del [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13]]: abstracción opaca, difícil de debuggear en producción, y dependencia de versiones inestables.
>
> Para producción seria, considera:
> - Construir tu propio pipeline (§2 de este tomo)
> - Usar Vanna.ai (que aprende de tu historial)
> - Un pipeline custom con las ideas de DIN-SQL (decomposition)

### 6.3 Ejemplo con LlamaIndex NLSQLTableQueryEngine

```python
"""
LlamaIndex NLSQLTableQueryEngine — pipeline estructurado.
"""
from llama_index.core import SQLDatabase, VectorStoreIndex
from llama_index.core.query_engine import NLSQLTableQueryEngine
from llama_index.llms.openai import OpenAI
from sqlalchemy import create_engine


# Setup
engine = create_engine("postgresql://readonly_user:pass@localhost:5432/analytics")
sql_database = SQLDatabase(engine, include_tables=["sales", "products", "regions"])

llm = OpenAI(model="gpt-4o", temperature=0.0)

# Query engine
query_engine = NLSQLTableQueryEngine(
    sql_database=sql_database,
    llm=llm,
    tables=["sales", "products", "regions"],
)

# Consultar
response = query_engine.query(
    "¿Cuál fue el ingreso total por categoría de producto en el último mes?"
)
print(f"Respuesta: {response.response}")
print(f"SQL usado: {response.metadata['sql_query']}")
```

---

## 7. 💼 Caso de negocio detallado: Analytics Self-Service

Audiencia: 👔 🧭

### 7.1 El problema de negocio

```
┌─────────────────────────────────────────────────────────────────┐
│                 ESTADO ACTUAL (sin Text2SQL)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Analista de negocio                  Equipo de BI              │
│  ┌──────────────────┐                 ┌──────────────┐          │
│  │ "Necesito saber  │────ticket────▶  │ Cola: 47     │          │
│  │  las ventas por  │                 │ solicitudes  │          │
│  │  región del Q3"  │                 │ pendientes   │          │
│  └──────────────────┘                 └──────┬───────┘          │
│                                              │                   │
│         Espera: 2-5 días                     │ Priorizar        │
│                                              ▼                   │
│                                       ┌──────────────┐          │
│                                       │ Analista BI  │          │
│  ┌──────────────────┐                 │ escribe SQL, │          │
│  │ Recibe reporte   │◀───email────── │ genera Excel │          │
│  │ (ya desactualizado│                 └──────────────┘          │
│  │  para cuando llega)│                                          │
│  └──────────────────┘                                           │
│                                                                  │
│  Costo: $150K/año en tiempo de BI dedicado a queries ad-hoc    │
│  Latencia: 2-5 días por solicitud                               │
│  Frustración: alta (ambos lados)                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 La solución

```
┌─────────────────────────────────────────────────────────────────┐
│                 ESTADO FUTURO (con Text2SQL)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Analista de negocio                  Sistema Text2SQL          │
│  ┌──────────────────┐                 ┌──────────────┐          │
│  │ "¿Cuáles fueron  │───chat/web───▶  │ Clasifica    │          │
│  │  las ventas por  │                 │ → Schema Link│          │
│  │  región del Q3?" │                 │ → Gen SQL    │          │
│  └──────────────────┘                 │ → Ejecuta    │          │
│         │                             │ → Sintetiza  │          │
│         │  15 segundos                └──────┬───────┘          │
│         │                                    │                   │
│         ▼                                    ▼                   │
│  ┌─────────────────────────────────────────────────┐            │
│  │ "Las ventas del Q3 por región fueron:           │            │
│  │  Norte: $2.3M (+12% vs Q2)                     │            │
│  │  Centro: $1.8M (-3% vs Q2)                     │            │
│  │  Sur: $1.1M (+7% vs Q2)                        │            │
│  │                                                  │            │
│  │  SQL ejecutado: SELECT ... [ver detalle]"       │            │
│  └─────────────────────────────────────────────────┘            │
│                                                                  │
│  65-80% de queries resueltas sin intervención humana            │
│  Equipo de BI liberado para análisis complejos y estratégicos   │
│  ROI estimado: 6-9 meses                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.3 Factores de éxito

👔 [Ejecutivo]

| Factor | Por qué importa | Cómo abordarlo |
|---|---|---|
| **Gobernanza de datos** | Si el data warehouse es un desorden, Text2SQL amplifica el caos | Primero curar: naming conventions, documentación de tablas, glosario de negocio |
| **Confianza del usuario** | Si el sistema da un número incorrecto una vez, pierdes adopción | Mostrar siempre el SQL generado + advertencia "verifica cifras críticas con BI" |
| **Scope controlado** | No lanzar sobre 200 tablas el día 1 | Empezar con 10–15 tablas de alta demanda, expandir gradualmente |
| **Feedback loop** | El sistema debe mejorar con el uso | Capturar queries fallidas, SQL corregido por humanos → fine-tuning |
| **Seguridad** | Datos sensibles (salarios, información personal) | Row-level security, tablas excluidas, audit trail |

---

## 8. 🧭 Guía de decisión: ¿Text2SQL, Table QA, o RAG documental?

Audiencia: 🔧 🧭 👔

### 8.1 Árbol de decisión

```
┌─────────────────────────────────────────────────────────────────────┐
│            ¿QUÉ TIPO DE RAG NECESITAS?                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ¿La respuesta requiere un CÁLCULO o un DATO ESPECÍFICO             │
│   de una base de datos / tabla?                                      │
│                                                                      │
│        NO                                          SÍ                │
│         │                                          │                 │
│         ▼                                          ▼                 │
│  ┌──────────────┐                    ¿Los datos están en una         │
│  │ RAG          │                     base de datos relacional?      │
│  │ DOCUMENTAL   │                                                    │
│  │ (Tomos 1-12) │                    SÍ              NO              │
│  └──────────────┘                    │               │               │
│                                      ▼               ▼               │
│                          ¿Más de 50 filas    ┌────────────────┐      │
│                           relevantes?        │ ¿Cabe en el    │      │
│                                              │  context window?│      │
│                          SÍ        NO        └───┬────────┬───┘      │
│                          │         │             │        │          │
│                          ▼         ▼            SÍ       NO         │
│                    ┌──────────┐ ┌──────────┐    │        │          │
│                    │ TEXT2SQL │ │ TEXT2SQL │    ▼        ▼          │
│                    │          │ │ (sigue   │ ┌────────┐┌─────────┐  │
│                    │ Pipeline │ │  siendo  │ │Direct  ││Pandas   │  │
│                    │ completo │ │  más     │ │Table   ││QA       │  │
│                    │ (§2)     │ │  preciso)│ │Reason. ││(§3.1)   │  │
│                    └──────────┘ └──────────┘ └────────┘└─────────┘  │
│                                                                      │
│  ¿La pregunta cruza AMBOS mundos (datos + documentos)?              │
│        → HYBRID con Router (§4)                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.2 Resumen ejecutivo de la decisión

| Si tu caso es... | Usa... | Porque... |
|---|---|---|
| Preguntas sobre **métricas, KPIs, datos cuantitativos** en una DB | **Text2SQL** | La precisión numérica requiere cálculo real, no inferencia del LLM |
| Preguntas sobre **políticas, procesos, documentación** | **RAG Documental** | La respuesta está en texto, no en una tabla |
| **Ambos** en la misma pregunta | **Hybrid con router** | Necesitas combinar datos numéricos con contexto cualitativo |
| **CSV/Excel pequeño** sin base de datos | **Pandas QA** o **Direct Reasoning** | No vale la pena montar infraestructura SQL |
| **Exploración rápida** de una tabla chica | **Direct Reasoning** | Mínimo setup, respuesta inmediata |

---

## 9. 📚 Checklist de implementación

Audiencia: 🔧

Para quien va a implementar un sistema Text2SQL en producción:

- [ ] **Schema documentado** — Cada tabla y columna tiene descripción de negocio
- [ ] **Glosario de términos** — "Ventas" = `invoices.amount_net` (no `orders.total_gross`)
- [ ] **Usuario DB read-only** — Conexión con permisos exclusivos de SELECT
- [ ] **Whitelist de tablas** — Solo las tablas aprobadas para consulta
- [ ] **Query validation** — Parsear SQL antes de ejecutar, rechazar DML/DDL
- [ ] **Timeout y resource limits** — Prevenir queries costosas
- [ ] **Sample rows actualizados** — Few-shot examples con datos reales (anonimizados)
- [ ] **Golden set de evaluación** — 50+ preguntas con SQL verificado humano
- [ ] **Feedback loop** — Capturar queries fallidas para mejora continua
- [ ] **Audit log** — Registrar usuario, pregunta, SQL generado, resultado
- [ ] **Disclaimers** — "Resultado generado automáticamente — verifica cifras críticas"
- [ ] **Fallback humano** — Escalar a BI cuando confidence < threshold

---

## 10. 📖 Referencias

| # | Referencia | Relevancia para este tomo |
|---|---|---|
| 1 | Yu, T., Zhang, R., Yang, K., et al. (2018). *Spider: A Large-Scale Human-Labeled Dataset for Complex and Cross-Domain Semantic Parsing and Text-to-SQL Task*. EMNLP 2018. | Benchmark fundacional de Text2SQL — 10,181 queries sobre 200 databases. Define las métricas EX y EM usadas en §5.4 |
| 2 | Rajkumar, N., Li, R., & Baber, D. (2022). *Evaluating the Text-to-SQL Capabilities of Large Language Models*. arXiv:2204.00498. | Primera evaluación sistemática de LLMs (Codex, GPT-3) en Text2SQL. Establece que los LLMs generalistas alcanzan ~67% EX en Spider sin fine-tuning |
| 3 | Pourreza, M. & Rafiei, D. (2023). *DIN-SQL: Decomposed In-Context Learning of Text-to-SQL with Self-Correction*. NeurIPS 2023. | Introduce el enfoque de descomposición (schema linking → classification → SQL generation → self-correction) que inspira el pipeline del §2 |
| 4 | Li, D., Wang, B., et al. (2024). *DAIL-SQL: Efficient Few-Shot Text-to-SQL with Optimized Example Selection*. VLDB 2024. | Demuestra que la selección inteligente de few-shot examples mejora significativamente la accuracy — relevante para el schema linking del §2.2 |
| 5 | Li, J., Hui, B., et al. (2024). *Can LLM Already Serve as A Database Interface? A BIg Bench for Large-Scale Database Grounded Text-to-SQLs*. NeurIPS 2024 (BIRD benchmark). | Benchmark más realista que Spider: bases reales con dirty data, valores ambiguos y schemas complejos. Los mejores sistemas alcanzan ~65% EX |
| 6 | LangChain Documentation. *SQL Agent*. docs.langchain.com/docs/use_cases/sql | Documentación oficial del SQL Agent usado en §6.2 |
| 7 | LlamaIndex Documentation. *NLSQLTableQueryEngine*. docs.llamaindex.ai | Documentación oficial del query engine usado en §6.3 |

---

> [!info] Navegación
> [[Guia-Maestra-RAG_00-MOC-Guia-Maestra-RAG|🗺️ Volver al índice]] · Anterior → [[Guia-Maestra-RAG_13-Frameworks-LangChain-LlamaIndex|Tomo 13 · Frameworks de orquestación]] · Siguiente → [[Guia-Maestra-RAG_15-Glosario-Ejecutivo|Tomo 15 · Glosario ejecutivo]]
