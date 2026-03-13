Great question. Many **NL → SQL projects fail in enterprises**, not because of the AI model, but because of **design mistakes**. If you avoid these 5 mistakes, your POC will look **much more mature to management**.

---

# 1️⃣ Not Giving the LLM Proper Schema Context (Biggest Mistake)

Many teams simply send the user query to the LLM like:

```
User Query:
Show default rate trend for RMBS deals
```

The LLM **does not know your database schema**.

So it guesses tables like:

```sql
SELECT default_rate
FROM rmbs
```

But real schema might be:

```
deal_master
performance_metrics
```

Result → **Wrong SQL**

### Correct Approach

Provide schema context in prompt.

Example prompt:

```
Database Schema:

Table: deal_master
columns:
deal_id
deal_type
region
issue_year

Table: performance_metrics
columns:
deal_id
default_rate
loss_rate
report_year
```

Then the model can generate:

```sql
SELECT report_year, AVG(default_rate)
FROM performance_metrics
JOIN deal_master USING(deal_id)
WHERE deal_type='RMBS'
GROUP BY report_year
```

✅ **Lesson:** LLM must understand schema.

---

# 2️⃣ Ignoring Business Terminology Mapping

Analysts speak **business language**, not database language.

Example:

User says:

```
Show RMBS default rate
```

But database stores:

```
deal_type = 'Residential Mortgage Backed Security'
```

Without mapping:

❌ AI generates wrong SQL

### Correct Approach

Create a **semantic layer / glossary**.

Example mapping:

```
RMBS → deal_type = 'Residential Mortgage Backed Security'
ABS → deal_type = 'Asset Backed Security'
default rate → performance.default_rate
```

Then inject this into prompt.

This dramatically improves accuracy.

---

# 3️⃣ No Query Guardrails (Dangerous in Production)

If you let AI generate SQL freely, it may produce:

```sql
SELECT * FROM transactions
```

On a table with **5 billion rows**.

Result:

💥 database overload
💥 slow queries
💥 system outage

### Correct Approach

Create **SQL validation layer**.

Rules:

```
Allow only SELECT
Auto add LIMIT 100
Block full table scans
Block joins >5 tables
Query timeout
```

Architecture:

```
LLM → SQL
      ↓
SQL Validator
      ↓
Database
```

This is **mandatory in banks**.

---

# 4️⃣ Trying to Support the Entire Database

Enterprise databases are huge.

Example:

```
300 tables
5000 columns
```

LLM cannot reason about everything.

Result:

❌ wrong joins
❌ wrong tables
❌ hallucinations

### Correct Approach

Limit scope.

Example:

```
Securitisation Dataset Only
Tables: 10–20
```

This dramatically improves accuracy.

For your POC:

```
Only securitisation research tables
```

Later you can expand.

---

# 5️⃣ Not Showing the Generated SQL to Users

Many systems hide SQL and only show results.

This creates **trust issues**.

Analysts will ask:

> How do I know this result is correct?

### Correct Approach

Always show:

```
Generated SQL
Explanation
```

Example UI:

```
User Query:
Show top 10 ABS deals in Europe

Generated SQL:
SELECT deal_name, issuance_size
FROM deals
WHERE deal_type='ABS'
AND region='Europe'
ORDER BY issuance_size DESC
LIMIT 10;
```

This builds **confidence and transparency**.

---

# Bonus Mistake (Very Common)

## 6️⃣ Expecting Perfect Accuracy Immediately

NL → SQL systems **require tuning**.

Typical accuracy:

```
Initial system: 60%
After tuning: 85–90%
```

Improvements come from:

* better prompts
* schema metadata
* example queries
* feedback loop

---

# What Successful Systems Do

Successful systems include:

```
Schema Context
Business Glossary
Query Guardrails
Domain Restriction
SQL Transparency
```

---

# What I Recommend for Your POC

Scope it like this:

### Domain

```
Securitisation research data
```

### Tables

```
10–15 tables max
```

### Features

✔ Natural language query
✔ SQL generation
✔ SQL validation
✔ Result display
✔ Show generated SQL

That is a **perfect enterprise POC**.

---

💡 One more thing that will really help you with this project:

There is a **special prompt structure used by companies like Snowflake and Databricks** to generate very accurate SQL from LLMs.

If you want, I can show you that **production-grade prompt template**. It can improve SQL accuracy by **2–3x**, and it will make your POC look very impressive internally.
