Below is a **production-grade NL → SQL architecture** typically used by fintech and banking companies. It adds **governance, schema retrieval, guardrails, and observability**, which are essential in regulated environments.

---

# Production-Grade NL → SQL Architecture (Fintech / Banking)

![Image](https://miro.medium.com/1%2ALVw-HA4IjIs2DMZcCZ3wUA.png)

![Image](https://miro.medium.com/1%2A_dIX29EvcgVybM8VvQ0VYw.jpeg)

![Image](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MTM2Mzg3LTU3OTQ2NWkxMzkyOTdDRTE5MjcyRkI4?image-dimensions=999x510\&revision=24)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2A__c1rlcILvA5vXuvTG3g9w.png)

---

# 1️⃣ High-Level Flow

```text
Analyst
   │
   ▼
Chat UI / Research Portal
   │
   ▼
API Gateway
   │
   ▼
AI Orchestrator Service
   │
   ├── Intent Detection
   ├── Schema Retrieval
   ├── Prompt Builder
   │
   ▼
LLM SQL Generator
   │
   ▼
SQL Guardrails / Validator
   │
   ▼
Query Execution Engine
   │
   ▼
Database / Data Warehouse
   │
   ▼
Result Formatter + Visualization
   │
   ▼
Response to Analyst
```

---

# 2️⃣ Key Layers (Important for Enterprise Design)

## User Interaction Layer

Where analysts ask questions.

Example:

```
Show top 10 RMBS deals issued in Europe in 2023
```

This could be:

* Internal research portal
* Web chat interface
* Teams / Slack bot

---

# AI Orchestrator Layer

This is the **brain of the system**.

Responsibilities:

* process user query
* fetch schema context
* build prompt
* call LLM
* validate SQL

Typical microservice:

```
nl-sql-service
```

If you build this in your stack:

```
Spring Boot + Kotlin
```

---

# Intent Detection Layer

Before generating SQL, the system determines:

* **Is this a data query?**
* **Which domain?**

Example:

```
Query:
Show ABS issuance in 2023
```

Detected domain:

```
Securitisation dataset
```

This helps restrict schema scope.

---

# Schema Retrieval Layer (Critical)

Instead of sending **entire database schema**, the system retrieves **relevant tables only**.

Technique:

```
Vector search on schema metadata
```

Example:

User query:

```
RMBS default rate trend
```

Retrieved tables:

```
deals
performance_metrics
```

Benefits:

* reduces hallucination
* improves accuracy
* reduces prompt size

Common tools:

* FAISS
* OpenSearch
* Pinecone

---

# Prompt Builder

Constructs the final prompt:

```
Role
Schema
Glossary
Examples
User Query
```

Example prompt content:

```
Tables:
deals
performance_metrics

Relationships:
deals.deal_id = performance_metrics.deal_id
```

---

# LLM SQL Generator

Model generates SQL.

Typical enterprise models:

* Azure OpenAI
* Claude
* Internal LLMs

Output:

```sql
SELECT report_year, AVG(default_rate)
FROM performance_metrics
JOIN deals USING(deal_id)
WHERE deal_type='RMBS'
GROUP BY report_year;
```

---

# SQL Guardrails Layer (Very Important in Banks)

This prevents unsafe or expensive queries.

Checks include:

✔ Only SELECT allowed
✔ LIMIT added automatically
✔ Join count restrictions
✔ Query cost estimation
✔ Schema validation

Example rule:

```
Reject queries scanning > 10M rows
```

---

# Query Execution Engine

Executes validated SQL.

Possible engines:

* Snowflake
* Postgres
* BigQuery
* Trino

Access should be **read-only**.

---

# Result Formatter

Transforms results into:

* tables
* charts
* summaries

Example:

```
Default Rate Trend
```

Graph automatically generated.

---

# Observability Layer

Enterprise systems always track:

```
User query
Generated SQL
Execution time
Error rate
Model accuracy
```

This helps improve prompts.

---

# Security Layer (Mandatory in Fintech)

```text
User
 │
 ▼
SSO Authentication
 │
 ▼
Role-Based Access Control
 │
 ▼
Row-Level Security
 │
 ▼
Database
```

Example:

```
Analyst A → RMBS datasets
Analyst B → Credit datasets
```

---

# Logging & Feedback Loop

All queries are stored.

Example log:

```
User query
Generated SQL
Execution result
User feedback
```

This helps:

* improve prompts
* retrain models

---

# Example End-to-End Flow

User asks:

```
Show RMBS default rate trend for last 5 years
```

Step 1 — Intent detection

```
Domain = securitisation
```

Step 2 — Schema retrieval

```
Tables = deals, performance
```

Step 3 — Prompt construction

Step 4 — LLM generates SQL

Step 5 — Validator checks query

Step 6 — Query executes

Step 7 — Results returned with chart

---

# Production Architecture Layers Summary

| Layer            | Purpose                 |
| ---------------- | ----------------------- |
| User Interface   | Analyst interaction     |
| API Gateway      | Request routing         |
| AI Orchestrator  | AI workflow             |
| Schema Retrieval | Select relevant tables  |
| Prompt Builder   | Structured prompt       |
| LLM              | SQL generation          |
| SQL Guardrails   | Security + cost control |
| Execution Engine | Run SQL                 |
| Visualization    | Display results         |

---

# What Makes This “Production Grade”

Compared to simple prototypes, production systems add:

✔ schema retrieval
✔ guardrails
✔ security
✔ observability
✔ query cost protection

Without these, NL → SQL systems **break quickly in enterprise environments**.

---

# How This Fits Your Project

Given your environment:

* **Spring Boot + Kotlin**
* research authoring platform

Your service could look like:

```
RCV2-AI-SQL-Service
```

Modules:

```
QueryController
IntentClassifier
SchemaRetriever
PromptBuilder
LLMService
SQLValidator
QueryExecutor
ResultFormatter
```

---

✅ **If you want, I can also show you something extremely useful for your POC presentation:**

**A very clean “NL → SQL architecture diagram you can directly present to management (PowerPoint style)”** — much simpler than enterprise diagrams but still impressive.
