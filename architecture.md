For your **Natural Language → SQL AI assistant**, the architecture should be **simple, secure, and controllable**, especially in a banking environment like Deutsche Bank. Below is a **recommended enterprise-style architecture** you can present for your POC.

---

# AI Natural Language → SQL Architecture

## High-Level Architecture

![Image](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/02/22/ML-16098-image01-Architecture.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2ApVbnhRc2K2a1L4wXAdul6Q.png)

![Image](https://storage.googleapis.com/gweb-cloudblog-publish/images/2_Text-to-SQL_at_Google_Cloud.max-2200x2200.jpg)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AiBu5TtAg6je9QQiu.png)

### Step-by-Step Flow

```
Analyst
   │
   │ Natural language query
   ▼
AI Chat Interface (Web / Internal Tool)
   │
   ▼
API Gateway / AI Service (Spring Boot - Kotlin)
   │
   ▼
LLM Service (Azure OpenAI / Internal Model)
   │
   ▼
Prompt Builder
(Schema + Metadata + User Query)
   │
   ▼
SQL Generator
   │
   ▼
SQL Validation Layer
(Security + Cost Guardrails)
   │
   ▼
Database / Data Warehouse
(Postgres / Snowflake / Internal DB)
   │
   ▼
Result Processor
(Table / Charts)
   │
   ▼
Response to Analyst
```

---

# Key Components Explained

## 1️⃣ Chat Interface

Where analysts type queries.

Example:

```
Show top 10 RMBS deals issued in Europe in 2023
```

Options:

* Web UI
* Internal research portal
* Teams / Slack bot

---

## 2️⃣ AI Service Layer (Spring Boot)

This will likely be **your main service**.

Responsibilities:

* receive user query
* fetch schema metadata
* call LLM
* validate SQL
* execute query

Example structure:

```
AIQueryController
    ↓
QueryService
    ↓
LLMService
    ↓
SQLValidator
    ↓
DatabaseService
```

---

## 3️⃣ Prompt Builder

This is **the most critical component**.

LLM must know:

```
Tables
Columns
Relationships
Example Queries
```

Example prompt:

```
You are a SQL expert.

Database schema:
Table: deals
columns:
deal_id
deal_type
region
issue_year

Table: performance
deal_id
default_rate

Generate SQL for the user query.

User query:
Show average default rate of RMBS deals in US since 2020
```

---

## 4️⃣ SQL Validation Layer

This prevents **dangerous queries**.

Checks:

✔ Only SELECT allowed
✔ LIMIT added
✔ Query cost estimation
✔ Schema validation

Example rule:

```
Reject queries scanning >10M rows
```

---

## 5️⃣ Database

Could be:

* Snowflake
* PostgreSQL
* BigQuery
* Internal DB warehouse

Important:

Use **read-only access**.

---

## 6️⃣ Result Processing

Convert results to:

* tables
* charts
* summaries

Example output:

```
Default Rate Trend (2019-2024)
```

---

# Security Layer (Critical in Banks)

You should highlight this to management.

```
User
 │
 ▼
Authentication (SSO)
 │
 ▼
Role Based Access
 │
 ▼
Query Filtering
 │
 ▼
Database
```

Example:

```
Analyst A → only securitisation datasets
Analyst B → only credit datasets
```

---

# Optional Enhancements (Phase 2)

Once POC works:

### Query Learning

Store previous queries.

```
User Query → SQL
```

Improve accuracy.

---

### Semantic Layer

Map business terms.

Example:

```
"default rate" → performance.default_rate
```

---

### Chart Generation

LLM suggests chart type.

```
trend → line chart
distribution → histogram
```

---

# Simplified Architecture (Best for POC)

You can show this version in presentation:

```
User
 │
 ▼
Chat UI
 │
 ▼
Spring Boot AI Service
 │
 ▼
LLM (SQL Generator)
 │
 ▼
SQL Validator
 │
 ▼
Database
 │
 ▼
Results / Charts
```

---

# Biggest Design Decision

The **most important architectural component** is:

**Schema Context Management**

Without good schema context:

```
LLM → bad SQL
```

With good schema context:

```
LLM → accurate SQL
```

---

# If I Were Building This in Your Project

Given your stack:

* Kotlin
* Spring Boot
* Research Authoring System

I would design:

```
RCV2-AI-Service
```

Modules:

```
nl-query-controller
schema-context-service
llm-sql-generator
sql-validator
query-executor
result-formatter
```

---

💡 **Important advice for your POC**

Management usually expects too much.

Your **POC should only prove this:**

```
Natural Language
        ↓
AI
        ↓
Correct SQL
        ↓
Query DB
        ↓
Return result
```

That's enough to demonstrate value.

---

✅ If you want, I can also show you **one very important thing that most AI-SQL systems fail at**:

**The 5 biggest mistakes companies make when building NL → SQL systems.**

Knowing these will make your **POC far stronger than most internal AI projects.**
