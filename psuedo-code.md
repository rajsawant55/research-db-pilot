Great — since you already work with **Spring Boot + Kotlin**, I’ll show you a **clean production-style service design** for an **NL → SQL system**. This is **pseudo-code but very close to real implementation**, so you can convert it quickly.

We will design:

1️⃣ Controller
2️⃣ Service layer
3️⃣ Prompt builder
4️⃣ LLM integration
5️⃣ SQL validator
6️⃣ Query executor
7️⃣ Response model

---

# 1️⃣ Project Structure

```text
nl-sql-service
│
├── controller
│     AIQueryController.kt
│
├── service
│     QueryService.kt
│     PromptBuilderService.kt
│     LLMService.kt
│     SQLValidatorService.kt
│     QueryExecutionService.kt
│
├── model
│     QueryRequest.kt
│     QueryResponse.kt
│
├── config
│     OpenAIConfig.kt
│     DatabaseConfig.kt
│
└── repository
      DatabaseRepository.kt
```

---

# 2️⃣ Request & Response Models

### QueryRequest

```kotlin
data class QueryRequest(
    val query: String
)
```

---

### QueryResponse

```kotlin
data class QueryResponse(
    val userQuery: String,
    val generatedSQL: String,
    val rows: List<Map<String, Any>>
)
```

---

# 3️⃣ Controller

Receives user query.

```kotlin
@RestController
@RequestMapping("/ai")
class AIQueryController(
    private val queryService: QueryService
) {

    @PostMapping("/query")
    fun executeQuery(@RequestBody request: QueryRequest): QueryResponse {
        return queryService.processQuery(request.query)
    }
}
```

Endpoint:

```
POST /ai/query
```

Payload:

```json
{
 "query": "Show top 10 RMBS deals in Europe"
}
```

---

# 4️⃣ QueryService (Main Orchestrator)

This coordinates the entire flow.

```kotlin
@Service
class QueryService(
    private val promptBuilder: PromptBuilderService,
    private val llmService: LLMService,
    private val sqlValidator: SQLValidatorService,
    private val queryExecutor: QueryExecutionService
) {

    fun processQuery(userQuery: String): QueryResponse {

        // Step 1 build prompt
        val prompt = promptBuilder.buildPrompt(userQuery)

        // Step 2 generate SQL from LLM
        val generatedSQL = llmService.generateSQL(prompt)

        // Step 3 validate SQL
        sqlValidator.validate(generatedSQL)

        // Step 4 execute SQL
        val rows = queryExecutor.execute(generatedSQL)

        return QueryResponse(
            userQuery = userQuery,
            generatedSQL = generatedSQL,
            rows = rows
        )
    }
}
```

---

# 5️⃣ Prompt Builder

This is **very important** for accuracy.

```kotlin
@Service
class PromptBuilderService {

    fun buildPrompt(userQuery: String): String {

        val schema = """
        Table: deals
        columns:
        deal_id
        deal_name
        deal_type
        region
        issue_year
        issuance_amount
        
        Table: performance
        columns:
        deal_id
        report_year
        default_rate
        """

        val glossary = """
        RMBS = Residential Mortgage Backed Security
        ABS = Asset Backed Security
        """

        return """
        You are an expert SQL analyst.
        
        Generate SQL for the question.
        
        Rules:
        - Only SELECT queries
        - Use LIMIT 100
        
        DATABASE SCHEMA:
        $schema
        
        BUSINESS TERMS:
        $glossary
        
        USER QUESTION:
        $userQuery
        
        Return only SQL.
        """.trimIndent()
    }
}
```

---

# 6️⃣ LLM Service

Calls AI model.

Example using **OpenAI / Azure OpenAI**.

```kotlin
@Service
class LLMService(
    private val openAIClient: OpenAIClient
) {

    fun generateSQL(prompt: String): String {

        val request = ChatRequest(
            model = "gpt-4",
            messages = listOf(
                ChatMessage("system", prompt)
            )
        )

        val response = openAIClient.chat(request)

        return response.choices.first().message.content
    }
}
```

In real implementation you can use:

* OpenAI SDK
* LangChain4j

---

# 7️⃣ SQL Validator

Prevents dangerous queries.

```kotlin
@Service
class SQLValidatorService {

    fun validate(sql: String) {

        if (!sql.trim().lowercase().startsWith("select")) {
            throw RuntimeException("Only SELECT queries allowed")
        }

        if (!sql.lowercase().contains("limit")) {
            throw RuntimeException("Query must contain LIMIT")
        }

        if (sql.contains("delete", true) ||
            sql.contains("update", true)) {
            throw RuntimeException("Unsafe query detected")
        }
    }
}
```

In real systems this layer is **very important**.

---

# 8️⃣ Query Execution Service

Executes SQL using JDBC.

```kotlin
@Service
class QueryExecutionService(
    private val jdbcTemplate: JdbcTemplate
) {

    fun execute(sql: String): List<Map<String, Any>> {
        return jdbcTemplate.queryForList(sql)
    }
}
```

---

# 9️⃣ Example Flow

User sends:

```
Show top 5 RMBS deals in Europe in 2023
```

LLM generates:

```sql
SELECT deal_name, issuance_amount
FROM deals
WHERE deal_type = 'RMBS'
AND region = 'Europe'
AND issue_year = 2023
ORDER BY issuance_amount DESC
LIMIT 5;
```

System executes and returns:

```json
{
 "userQuery": "...",
 "generatedSQL": "...",
 "rows": [
   {"deal_name": "Deal A", "issuance_amount": 20000000},
   {"deal_name": "Deal B", "issuance_amount": 15000000}
 ]
}
```

---

# 10️⃣ Optional Improvements (Phase 2)

After POC works, add:

### Schema Retrieval

Instead of hardcoding schema.

Query:

```sql
information_schema.tables
```

---

### Query Explanation

Show explanation:

```
This query calculates average default rate
```

---

### Query Logging

Store:

```
User query
Generated SQL
Execution time
```

---

# 11️⃣ Simple POC Architecture

```
User
 ↓
React Chat UI
 ↓
Spring Boot AI Service
 ↓
Prompt Builder
 ↓
LLM SQL Generator
 ↓
SQL Validator
 ↓
Postgres
 ↓
Results
```

---

✅ If you want, I can also show you **one extremely useful thing for your project**:

**How to make the LLM choose the correct tables automatically (schema retrieval trick).**

This is the **#1 problem in NL → SQL systems in large databases**, and solving it will make your system far more accurate.
