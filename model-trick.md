How to make the LLM choose the correct tables automatically (schema retrieval trick).


This is one of the **most important problems in NL → SQL systems**.
If your database has **100+ tables**, sending the whole schema to the LLM will:

❌ confuse the model
❌ increase token cost
❌ generate wrong joins

So production systems use a technique called **Schema Retrieval** (or **Table Selection**).

Instead of giving the LLM **all tables**, we **retrieve only the relevant tables for the user query**.

---

# 1️⃣ Concept: Schema Retrieval

Instead of this:

```text
LLM Prompt
|
|--- All database tables (200 tables)
|
User query
```

We do this:

```text
User query
   ↓
Schema Retrieval
   ↓
Relevant Tables (3–5 tables)
   ↓
LLM Prompt
```

Example:

User asks:

```
Show RMBS default rate trend
```

Retrieved tables:

```
deals
performance
```

Only these are given to the LLM.

---

# 2️⃣ How Schema Retrieval Works

There are **two common approaches**.

| Method           | Complexity | Accuracy |
| ---------------- | ---------- | -------- |
| Keyword Matching | Easy       | Medium   |
| Vector Search    | Medium     | High     |

For your **POC**, I recommend starting with **Keyword Matching**.

---

# 3️⃣ Approach 1 — Keyword Based Table Selection (Best for POC)

Create metadata for tables.

Example:

```text
Table: deals
description: securitisation deals, issuance data, asset class

Table: performance
description: deal performance metrics like default rate, loss rate

Table: investors
description: investors participating in deals
```

Now when a query arrives:

```
Show RMBS default rate trend
```

Match keywords:

```
default rate → performance
RMBS → deals
```

Selected tables:

```
deals
performance
```

---

### Kotlin Example

```kotlin
data class TableMetadata(
    val name: String,
    val description: String
)
```

---

Table catalog:

```kotlin
val tables = listOf(
    TableMetadata("deals", "securitisation deals, asset class, region"),
    TableMetadata("performance", "default rate, loss rate, performance metrics"),
    TableMetadata("investors", "deal investors")
)
```

---

Simple retrieval:

```kotlin
fun findRelevantTables(query: String): List<TableMetadata> {

    return tables.filter {
        query.contains("deal", true) ||
        query.contains("RMBS", true) ||
        query.contains("default", true)
    }
}
```

---

Then send only these tables to prompt builder.

---

# 4️⃣ Approach 2 — Vector Search (Production Method)

Large companies use **vector embeddings**.

Steps:

### Step 1 — Create table embeddings

Example table description:

```
Table: performance
Columns: deal_id, default_rate, loss_rate
Description: securitisation deal performance metrics
```

Convert to embedding.

Store in vector DB.

---

### Step 2 — Query embedding

User query:

```
Show RMBS default rate trend
```

Convert to embedding.

---

### Step 3 — Vector similarity search

Find closest tables.

Result:

```
performance
deals
```

---

Tools used:

| Tool       | Purpose                  |
| ---------- | ------------------------ |
| FAISS      | vector search            |
| OpenSearch | enterprise vector search |
| Pinecone   | vector DB                |
| Weaviate   | vector DB                |

---

# 5️⃣ Final Prompt After Retrieval

Instead of sending 200 tables:

Send only this:

```text
DATABASE SCHEMA

Table: deals
columns:
deal_id
deal_type
region
issue_year

Table: performance
columns:
deal_id
default_rate
report_year
```

Then the LLM can easily generate correct SQL.

---

# 6️⃣ Real Production Flow

```text
User Query
   ↓
Intent Detection
   ↓
Schema Retrieval
   ↓
Prompt Builder
   ↓
LLM SQL Generation
   ↓
SQL Validator
   ↓
Database
```

---

# 7️⃣ Why This Is Important

Without schema retrieval:

```
200 tables
```

LLM hallucination rate:

```
~40%
```

With schema retrieval:

```
3–5 tables
```

Accuracy improves to:

```
~85–90%
```

This is exactly what systems from companies like **Databricks** and **Snowflake** implement internally.

---

# 8️⃣ What I Recommend For Your POC

Start simple:

✔ manually curate **10–20 tables**
✔ add **table descriptions**
✔ implement **keyword-based retrieval**

Architecture:

```text
User Query
   ↓
Table Retriever
   ↓
Prompt Builder
   ↓
LLM SQL Generator
```

Later upgrade to:

```
Vector schema retrieval
```

---

# 9️⃣ Simple POC Implementation

Add service:

```text
SchemaRetrievalService
```

Flow:

```text
QueryService
     ↓
SchemaRetrievalService
     ↓
PromptBuilderService
     ↓
LLMService
```

---

💡 **One more very important trick (used in advanced NL→SQL systems):**

Instead of asking the LLM **“write SQL”**, first ask it:

```
Which tables are needed for this query?
```

Then generate SQL in **two steps**.

This improves SQL accuracy dramatically.

---

If you want, I can also show you **the “2-step LLM reasoning pattern” used in enterprise NL→SQL systems**.
It’s a **very powerful trick** that can make your POC look **much smarter than typical AI demos.**
