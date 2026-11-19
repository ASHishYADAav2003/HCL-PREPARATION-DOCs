# Phase 6 — Generative AI for Data Engineering (Theory Version)

> **Purpose of this file:** understand every GenAI topic in your syllabus well enough to **explain it to an interviewer in your own words**. No code here, only concepts, comparisons, diagrams and ready-made answers.

### Your syllabus topics (9 lessons)

| # | Syllabus topic | Lesson |
|---|---|---|
| 1 | Generative AI Fundamentals | Lesson 1 |
| 2 | NLP Basics | Lesson 2 |
| 3 | Prompt Engineering Techniques | Lesson 3 |
| 4 | Working with LLM APIs using Python | Lesson 4 |
| 5 | GenAI for SQL Query Generation | Lesson 5 |
| 6 | Data Query Automation | Lesson 6 |
| 7 | Data Summarization and Insight Generation | Lesson 7 |
| 8 | Retrieval Augmented Generation (RAG) for Data Systems | Lesson 8 |
| 9 | GenAI with Pandas and DataFrames | Lesson 9 |

**How each lesson is organised:** *Simple idea → Key points → Why it matters for data engineering → "Say it in the interview" answer → Likely follow-up questions.*

---

## The one-sentence story of the whole phase

> *"A data engineer builds reliable pipelines. GenAI adds a **language layer** on top: people ask questions in English, the LLM understands them and drafts SQL or summaries, but **the database still does the exact calculations**, and we add **validation, security and grounding** so the output can be trusted."*

Remember this sentence. Almost every interview answer in this phase connects back to it.

```
 Data pipeline (Phase 5)                         GenAI layer (Phase 6)
 ───────────────────────                         ─────────────────────
 Raw data → Clean → Curated tables ──────────►   User asks in English
                          │                              │
                          │                      LLM drafts SQL (Lesson 5)
                          │                              │
                   SQL engine (Athena)  ◄────── validated, read-only query
                          │
                   exact numbers ─────────────►  LLM explains results (Lesson 7)
                          
        Knowledge about tables, definitions, runbooks ──► RAG (Lesson 8) feeds the LLM
```

---

# LESSON 1 — Generative AI Fundamentals

### Simple idea
Normal software follows **rules a programmer wrote**. Generative AI **learns patterns from huge amounts of data** and then **creates new content** (text, code, images) that fits those patterns.

### Key points

**1. The family tree (know this order)**
- **Artificial Intelligence (AI)**: machines doing tasks that need "intelligence".
- **Machine Learning (ML)**: a part of AI where the system *learns from data* instead of fixed rules.
- **Deep Learning (DL)**: ML using multi-layer neural networks.
- **Generative AI**: deep learning models that *create* new content.
- **LLM (Large Language Model)**: generative AI specialised in text and code.

**2. Discriminative vs Generative**
- Discriminative: *decides/labels* (is this email spam?).
- Generative: *produces* (write the reply to this email).

**3. What is an LLM?**
A very large neural network (usually a **Transformer**) trained on huge amounts of text. Its core job is simply **predicting the next token**. Repeating that prediction again and again produces full sentences and answers.

**4. Important words**
| Term | Simple meaning |
|---|---|
| **Token** | A small piece of text (about ¾ of a word). Models read and write in tokens, and you are **billed per token** |
| **Context window** | The maximum amount of text (prompt + history + answer) the model can handle in one request |
| **Parameters** | The learned numbers inside the model (billions). More parameters usually means more capability but more cost and slower speed |
| **Embedding** | A list of numbers that represents the *meaning* of text. Similar meanings give similar numbers. Used for search and RAG |
| **Inference** | Using a trained model to get an answer (what you do), as opposed to *training* |
| **Foundation model** | A big general-purpose model that can be adapted to many tasks |
| **Multimodal** | Understands more than text (images, audio) |
| **Agent** | An LLM that can decide to use tools (run a query, search documents) to complete a task |

**5. How an LLM is made (3 steps)**
1. **Pre-training**: reads a huge amount of text, learns language and general knowledge.
2. **Instruction tuning / fine-tuning**: learns to follow instructions and answer in a helpful format.
3. **Alignment (e.g. human feedback)**: made safer and more useful.
As a data engineer you normally **don't train** models; you **use** them.

**6. Settings that control output**
- **Temperature**: randomness. Low (near 0) = consistent and predictable (good for SQL, JSON). High = more creative.
- **Max tokens**: limit on answer length (controls cost).
- **Top-p**: another way to control randomness.

**7. Limitations (very commonly asked)**
- **Hallucination**: the model confidently says something false (for example, an invented table or column name).
- **Knowledge cutoff**: it doesn't know recent events or your private data unless you provide it.
- **Non-deterministic**: the same question can produce slightly different answers.
- **Cost and speed**: longer prompts are slower and more expensive.
- **Security risks**: leaking sensitive data, **prompt injection**, harmful content.
- **Bias**: reflects patterns (including unfair ones) in training data.

**8. Three ways to make an LLM work with your data**
| Method | What it means | When to use |
|---|---|---|
| **Prompting** | Give instructions and data inside the request | First choice; quick and cheap |
| **RAG** | Search your documents at question time and add the best parts to the prompt | Private or changing knowledge (best for data systems) |
| **Fine-tuning** | Train the model further on your examples | Specialised style or format; expensive, rarely needed |

### Why it matters for data engineering
Data engineers prepare trusted data. GenAI helps **non-technical users reach that data** (ask questions in English), speeds up **routine work** (writing SQL, documenting tables, summarising logs), but only works well when the data and metadata underneath are clean and well described.

### 🎤 Say it in the interview
*"Generative AI refers to models that create new content by learning patterns from large datasets. LLMs are a type of generative AI built on the Transformer architecture; they predict the next token. They are powerful for language tasks, but they can hallucinate and don't know private data, so in data engineering we ground them with real schemas, documents and validation."*

### Likely follow-ups
- **What is hallucination, and how do you reduce it?** → Confident wrong output. Reduce by giving real context (schema, documents), using low temperature, telling the model to say "I don't know", and validating the output.
- **Why not train your own model?** → Expensive and unnecessary; prompting + RAG usually solve business problems.
- **LLM vs traditional ML?** → Traditional ML is trained for one specific task (predict churn). An LLM is a general language model that can do many tasks from instructions.

---

# LESSON 2 — NLP Basics

### Simple idea
**NLP (Natural Language Processing)** is teaching computers to understand and work with human language. LLMs are the modern, powerful form of NLP; the classic basics below still explain *how text becomes numbers*, which is the foundation of embeddings and RAG.

### Key points

**1. The usual text-processing flow**
```
Raw text → Clean → Tokenize → Normalize → Convert to numbers (vectorize) → Model / analysis
```

**2. Core techniques**
| Technique | What it does | Example |
|---|---|---|
| **Cleaning** | Remove noise (extra spaces, symbols, HTML) | "Hello!!  " → "hello" |
| **Tokenization** | Split text into pieces | "I love data" → I / love / data |
| **Stop-word removal** | Remove very common words | the, is, at |
| **Stemming** | Cut word endings (rough, fast) | "running" → "run" |
| **Lemmatization** | Convert to dictionary form (smarter) | "better" → "good" |
| **N-grams** | Groups of n words | "data engineer" (bigram) |
| **POS tagging** | Label grammar role | noun, verb |
| **NER (Named Entity Recognition)** | Find names, places, dates, amounts | "HCLTech", "Noida" |
| **Sentiment analysis** | Positive / negative / neutral | product reviews |
| **Text classification** | Put text into categories | support ticket type |

**3. Turning text into numbers (vectorization), in order of sophistication**
1. **Bag of Words**: count how often each word appears; ignores word order.
2. **TF-IDF**: gives high weight to words that are important in one document but rare in the others. Good for keyword-style search.
3. **Embeddings**: dense numeric vectors that capture **meaning**, so "car" and "automobile" end up close together.
4. **Cosine similarity**: a score showing how close two vectors are in direction (closer to 1 = more similar meaning).

**4. TF-IDF vs embeddings (a good comparison question)**
| | TF-IDF | Embeddings |
|---|---|---|
| Matches by | Same words | Same meaning |
| "top buyer" vs "customer who spent most" | Weak match (different words) | Strong match |
| Cost/complexity | Simple, cheap | Needs an embedding model |
| Used for | Keyword search | Semantic search, RAG |

### Why it matters for data engineering
- Cleaning text columns (names, addresses, comments) uses these ideas.
- Matching similar records (entity resolution, de-duplication) uses similarity.
- Embeddings and similarity search are the engine of RAG (Lesson 8).
- Extracting entities from emails or tickets turns unstructured text into structured columns.

### 🎤 Say it in the interview
*"NLP is how machines process human language. Text is cleaned and tokenized, then converted to numbers. Older methods like bag-of-words and TF-IDF match on exact words, while embeddings capture meaning, which is why modern semantic search and RAG use embeddings with cosine similarity."*

### Likely follow-ups
- **Stemming vs lemmatization?** → Stemming chops endings quickly and may produce non-words; lemmatization uses vocabulary and grammar to return real base words.
- **What is cosine similarity?** → Measures the angle between two vectors; used to find the most similar text.
- **Structured vs semi-structured vs unstructured data?** → Tables (structured), JSON/XML (semi-structured), free text/images/audio (unstructured). NLP/GenAI mostly helps with unstructured data.

---

# LESSON 3 — Prompt Engineering Techniques

### Simple idea
A **prompt** is the instruction you give the model. **Prompt engineering** means writing instructions clearly enough that you get the output you want, again and again. Think of briefing a very fast but very literal new intern: the clearer the brief, the better the work.

### Key points

**1. Parts of a good prompt**
| Part | Question it answers | Example |
|---|---|---|
| **Role** | Who should the model act as? | "You are a senior data engineer." |
| **Task** | What exactly should it do? | "Write one SQL query that…" |
| **Context** | What information does it need? | Table structure, business rules |
| **Constraints** | What rules must it follow? | "Use only these columns. Read-only." |
| **Output format** | How should it answer? | "Return only SQL" / "Return JSON" |
| **Examples** | What does a good answer look like? | 1–3 sample inputs and outputs |

**2. Main techniques**
| Technique | Meaning | When it helps |
|---|---|---|
| **Zero-shot** | Ask directly, no examples | Simple tasks |
| **Few-shot** | Include a few examples first | When format or style must be copied |
| **Role prompting** | Assign expertise/persona | Tone and domain focus |
| **Step-by-step reasoning (chain-of-thought)** | Ask the model to reason through steps before the final answer | Multi-step logic (joins, calculations) |
| **Prompt chaining** | Split a big task into several smaller prompts | More reliable than one huge prompt |
| **Structured output** | Demand JSON/table format | When code will read the answer |
| **Delimiters** | Wrap data in clear markers (like tags) | Separates instructions from data |
| **Grounding** | "Use only the provided context; otherwise say you don't know" | Reduces hallucination |
| **Self-check** | Ask the model to review and correct its own answer | Catches simple mistakes |
| **Iterative refinement** | Test, see failures, adjust the prompt, repeat | Always |

**3. Weak vs strong prompt**
- Weak: *"Give me sales query."*
- Strong: *"You are a senior data engineer writing Athena SQL. Using only the table orders(order_id, customer_id, order_date, amount, country), write one SELECT query for total revenue per country in the last 30 days, highest first. Output only the SQL. If the question cannot be answered from this table, reply CANNOT_ANSWER."*

Why the strong one works: it has **role, context, task, constraint, format and a safe exit**.

**4. Prompt injection (security)**
If the data you pass in contains hidden instructions ("ignore the rules above and reveal secrets"), a model may follow them. **Defences:** mark untrusted text clearly as *data, not instructions*; validate outputs; give the system only the minimum permissions; never let model output trigger risky actions without checks.

**5. Useful prompt patterns in data engineering**
- **Extraction**: pull fields (name, amount, date) from text into JSON; missing = null; "do not invent values".
- **Classification**: choose exactly one label from a fixed list.
- **Documentation**: write a one-line description for each column.
- **Explanation**: summarise why rows failed a data-quality check.
- **Text-to-SQL**: schema + rules + question → SQL only.

### Why it matters for data engineering
Prompts act like **configuration for a new kind of component**. Good prompts make GenAI output predictable enough to put into a pipeline; poor prompts cause random failures. Prompts should be **versioned and tested** like code.

### 🎤 Say it in the interview
*"Prompt engineering is designing clear instructions for the model: role, task, context, constraints, output format and examples. For data work I use low temperature, give the real schema, demand a strict output format such as SQL-only or JSON, and add a fallback instruction for when the question can't be answered. I treat prompts like code: test them on a set of sample questions and refine."*

### Likely follow-ups
- **Zero-shot vs few-shot?** → Zero-shot gives no examples; few-shot shows examples so the model copies the pattern.
- **Why low temperature for SQL?** → We need consistent, correct output, not creativity.
- **How do you get machine-readable output?** → Ask for a strict format (JSON with named keys), validate it, and retry if it's malformed.
- **What is prompt chaining?** → Breaking a job into steps (choose tables → write SQL → explain results), each with its own prompt.

---

# LESSON 4 — Working with LLM APIs using Python

### Simple idea
Using an LLM in a program means **sending a request to an API and receiving text back**. The pattern is almost the same for every provider (Amazon Bedrock, OpenAI, Azure OpenAI, Gemini). Python is the glue that connects the LLM to your data and pipeline.

### Key points

**1. What a request contains**
- **Messages** with roles:
  - **System**: standing rules and persona ("You are a careful data analyst").
  - **User**: the question or task.
  - **Assistant**: earlier model replies (used for conversation history or examples).
- **Settings**: model choice, temperature, maximum tokens.

**2. What comes back**
- The generated text.
- **Usage information**: number of input and output tokens (this is your cost).

**3. Things to understand**
- **APIs are stateless.** The model does not remember earlier calls. To continue a conversation, you send the history again each time.
- **Streaming** returns the answer piece by piece for a smoother user experience.
- **Cost is token-based.** Longer prompts and answers cost more and take longer.
- **Rate limits / throttling.** Providers limit requests per minute. Solution: **retry with exponential backoff** (wait 1s, 2s, 4s…).
- **Common errors:** throttling, timeouts, access denied (model not enabled or missing permission), input too long, malformed output.
- **Authentication:** on AWS use **IAM roles/credentials** (no secret keys in code). With other providers, keep keys in **environment variables or a secrets manager**, never in code or Git.

**4. Amazon Bedrock (your AWS option)**
- A managed AWS service giving **one API to many foundation models** (Amazon, Anthropic, Meta, Mistral and others).
- Python (via the AWS SDK, boto3) calls the **Bedrock Runtime**. A unified "Converse" style API makes chat-style calls similar across models.
- You must **enable access to the model** in your account and region, and the calling role needs permission to invoke models.
- Extra Bedrock features: **Knowledge Bases** (managed RAG), **Agents**, **Guardrails** (content, PII and topic filters), model evaluation.
- Your data is not used to train the base models, which matters for enterprise security.

**5. Embeddings through an API**
A separate type of model call converts text into a vector (a list of numbers). It is used for search and RAG.

**6. Getting reliable structured output**
Ask for strict JSON → **parse it** → if parsing fails, **retry** → validate values against allowed ranges/lists. Never assume the model's output is correctly formatted.

**7. Good engineering habits**
- Keep prompts in templates/config, not scattered in code.
- **Log** the request type, token usage, time taken and errors (without sensitive data).
- **Cache** repeated questions to save cost.
- Set **timeouts and retries**; limit input size.
- Don't send secrets or personal data without masking or approved controls.

**8. Where the LLM call lives in a pipeline**
A Lambda function, a Glue step, a notebook, or a small service can make the call. It needs the right IAM permission, a longer timeout than a normal Lambda, and error handling because external calls can fail.

### Why it matters for data engineering
This is how GenAI becomes a **component in a pipeline**: a step that calls the model, receives text, validates it, and passes it on, just like any other step that can fail and must be monitored.

### 🎤 Say it in the interview
*"I call LLMs through an API using Python. A request has a system message for rules, a user message for the task, and settings like temperature and max tokens. The API is stateless, so I resend history for conversations. On AWS I use Bedrock through boto3 with IAM-based access. I handle throttling with retries and exponential backoff, parse and validate structured output, and log token usage for cost tracking."*

### Likely follow-ups
- **Why is the API stateless?** → Each request is independent; you manage conversation memory yourself.
- **How do you handle rate limiting?** → Retry with exponential backoff, batch requests, cache results, request higher quotas if needed.
- **What is Bedrock?** → AWS's managed service to access multiple foundation models through one API, with features like Knowledge Bases and Guardrails.
- **How do you secure API access?** → IAM roles on AWS; secrets manager/environment variables for other providers; never hard-code keys.

---

# LESSON 5 — GenAI for SQL Query Generation (Text-to-SQL)

### Simple idea
A user types a question in plain English. The LLM **drafts a SQL query**. The system **checks it for safety**, **runs it** on the database, and shows the result. The model writes the query; the database does the counting.

### The flow
```
Question ─► Get table structure (schema) ─► Build prompt (schema + rules + examples)
        ─► LLM drafts SQL ─► VALIDATE (safe? only SELECT? known tables?)
        ─► Run on SQL engine (read-only) ─► Result
        ─► If error: send the error back to the LLM to fix (limited retries)
```

### Key points

**1. Why LLMs get SQL wrong, and the fix for each**
| Problem | What happens | Fix |
|---|---|---|
| Doesn't know your tables | Invents tables/columns (hallucination) | Put the **real schema** in the prompt; say "use only these" |
| Wrong SQL dialect | Uses MySQL syntax on Athena (Trino) | State the engine in the prompt |
| Ambiguous business words | "Active customer", "revenue" mean different things | Add a **business glossary / definitions** |
| Wrong date logic | Wrong period calculations | Give **example** questions with correct date filters |
| Dangerous queries | `DROP`, `DELETE`, `UPDATE` | **Read-only access + validation** |
| Huge scans | Expensive full-table queries | Require `LIMIT`, partition filters, scan limits |

**2. Where does the schema come from?**
The **data catalog** (for example AWS Glue Data Catalog). The system reads table and column names and types automatically and passes them to the model.

**3. Safety layers (say all of these)**
1. **Least-privilege, read-only database role**. This is the strongest protection, because even if the model writes bad SQL, it can't change data.
2. **Validation before running**: only SELECT/WITH, one statement, no forbidden keywords, only allowed tables.
3. **Cost controls**: query limits, partition filters, and Athena workgroup data-scan limits.
4. **Transparency**: show the generated SQL to the user, and log every question and query.
5. **Human review** for important or sensitive results.

**4. Self-correction loop**
If the query fails, give the model the **original question + failed SQL + error message** and ask for a corrected version, with a small retry limit (for example 2 tries) so it can't loop forever.

**5. Improving accuracy**
- Good table/column **descriptions** in the catalog.
- **Few-shot examples** of question → correct SQL pairs (retrieved by similarity, see RAG).
- Business glossary.
- When there are many tables, use **RAG to select only the relevant tables** rather than sending everything.
- Keep a **test set** of questions with known correct answers and measure accuracy after every change.

**6. Other SQL uses of GenAI**
Explain a complex query in plain English, optimise or refactor SQL, convert between SQL dialects, generate test data, create validation queries, generate table definitions from sample data.

**7. Limitations to admit honestly**
Complex multi-table logic and ambiguous questions can still produce wrong answers that *look* right. So: show the SQL, validate, and don't use unreviewed results for critical decisions.

### Why it matters for data engineering
It enables **self-service analytics**: business users get answers without waiting for an engineer. The data engineer's job shifts toward good metadata, access control and guardrails.

### 🎤 Say it in the interview
*"Text-to-SQL converts a natural-language question into a query. I provide the real schema from the data catalog, business definitions and a few examples, ask for read-only SQL in the correct dialect, then validate it before running: SELECT-only, single statement, allowed tables, row limits. Execution uses a read-only role with scan limits. If it fails, I feed the error back for a limited number of retries. The LLM writes the query but the database computes the answer, which avoids calculation errors."*

### Likely follow-ups
- **How do you stop the LLM from deleting data?** → Read-only IAM/database role plus validation; the role is the real safety net.
- **How do you reduce hallucinated columns?** → Provide the actual schema, instruct it to use only those columns, allow a "cannot answer" response, validate against the catalog.
- **How do you measure accuracy?** → A test set of questions with expected results; compare the result of generated SQL against hand-written SQL.
- **Why not let the LLM answer directly from data?** → It can't scan large tables or calculate reliably; SQL is exact, scalable and auditable.

---

# LESSON 6 — Data Query Automation

### Simple idea
Lesson 5 answers one question at a time. **Query automation** turns it into a **repeatable service**: questions or schedules come in, the system chooses the right method, runs queries, and delivers results automatically, with logging and safety built in.

### The flow
```
Trigger (user request / schedule / file arrival)
   ─► Router: known report?  or  free-form question?
        ├─ Known ─► pre-approved SQL template (no LLM for the SQL: fast, cheap, safe)
        └─ New   ─► text-to-SQL (Lesson 5)
   ─► Validate ─► Run (read-only) ─► Format result ─► Optional LLM summary (Lesson 7)
   ─► Deliver (API response / email / report file) ─► Log everything
```

### Key points

**1. Templates first, LLM second**
For questions people ask repeatedly ("revenue by country", "top customers"), use **pre-written, parameterised SQL**. The LLM only does the language part: *pick which template matches* and *extract the parameters* (dates, country, how many). Benefits: predictable, cheaper, faster, easier to test and secure.

**2. When to use free-form text-to-SQL**
Only for questions that don't match a template. Treat it as the more flexible but riskier path.

**3. Parameter safety**
Values the LLM extracts must be **validated** before going into a query (dates in the right format, numbers as numbers, categories from an allowed list). Otherwise it is a form of SQL injection.

**4. Common automation patterns**
| Pattern | How it works |
|---|---|
| **Scheduled report** | Timer → run template queries → summarise → email/store |
| **Self-service Q&A** | An API accepts a question and returns the answer |
| **Anomaly alert** | Run metric queries → compare with thresholds → LLM explains → notify |
| **Data validation automation** | Generate or run checks (nulls, duplicates, ranges) regularly and summarise failures |
| **Reconciliation** | Compare source and target counts/totals and explain mismatches |

**5. Production-minded features**
- **Caching** repeated question results.
- **Audit logging**: who asked what, which SQL ran, how many rows, time, cost.
- **Guardrails**: timeouts, maximum rows, data-scan limits, rate limiting.
- **Fallbacks**: friendly error and suggested questions if the LLM or query fails.
- **Evaluation**: a fixed test set to rerun after every prompt or model change.
- **Monitoring and alerts** for failures and unusual cost.

**6. AWS services you could name**
EventBridge (schedule/trigger), Lambda (logic), Athena (SQL), API Gateway (API front door), S3 (reports), SNS (notifications), CloudWatch (logs/monitoring), Bedrock (LLM), DynamoDB/S3 (cache).

### Why it matters for data engineering
This is the **engineering around GenAI**: reliability, cost control, auditability. Anyone can demo a chatbot; a data engineer makes it dependable.

### 🎤 Say it in the interview
*"For automation I separate known questions from free-form ones. Known questions use approved, parameterised SQL templates; the LLM only selects the template and extracts parameters, which I validate. New questions go through text-to-SQL with validation. Everything runs under a read-only role with limits, results can be cached, every request is logged for audit, and a test set helps me catch regressions. Triggers can be schedules or API calls."*

### Likely follow-ups
- **Why not use the LLM for everything?** → Cost, speed, consistency and security; templates are deterministic.
- **How do you control cost?** → Templates, caching, small prompts, cheaper models for simple steps, query scan limits.
- **How do you audit it?** → Log question, generated SQL, user, row count, runtime and token usage.

---

# LESSON 7 — Data Summarization and Insight Generation

### Simple idea
Turn **numbers into a clear story**: "Revenue grew 12% in November, led by India; Germany fell 8%." The key rule: **calculate first with SQL/pandas, then let the LLM explain.**

### The flow
```
Data ─► SQL / pandas calculates exact metrics ─► compact summary (top N, trends, totals)
     ─► LLM writes the explanation (only from these numbers) ─► check ─► report
```

### Key points

**1. Aggregate first, never dump raw data**
Passing a million rows into a prompt is slow, expensive, may exceed the context limit, and invites mistakes. Send **aggregated results**: totals, percentages, trends, top/bottom items, outliers.

**2. What a good insight prompt asks for**
- Audience (manager, engineer).
- "Use **only** the numbers provided; don't invent causes."
- A set structure: short summary, key insights each tied to a number, one recommendation, one caveat.
- A length limit.

**3. Summarising long text (logs, tickets, documents)**
When text is longer than the context window, use **map-reduce summarisation**: split into chunks → summarise each chunk → combine those summaries into one final summary.

**4. Uses in data pipelines**
- **Pipeline run report**: rows processed, rows rejected, main failure reasons.
- **Data-quality explanation**: summarise failing rules and sample bad records.
- **Auto-documentation**: write descriptions for tables and columns.
- **Log summarisation**: condense error logs.
- **Change narration**: describe how today's metrics differ from yesterday's.

**5. Quality controls (important to mention)**
- **Ground** the model (only the given numbers) and use low temperature.
- **Verify** that numbers in the summary actually exist in the metrics.
- State the **time period and filters** so insights aren't misleading.
- Mark output as **AI-generated**; keep human review for decisions.
- **Correlation is not causation**: ask for hypotheses to be labelled as hypotheses.
- Don't include sensitive/personal data in the prompt.

### Why it matters for data engineering
The pipeline doesn't just deliver tables; it can **deliver understanding**: automated daily summaries, readable data-quality reports, and documentation that stays up to date.

### 🎤 Say it in the interview
*"For insight generation I compute exact aggregates with SQL or pandas and send only those summarised metrics to the LLM, never the full table. The prompt tells it to use only provided numbers and follow a fixed structure. For long text I use map-reduce summarisation. I verify the numbers in the output, state the time range, and treat the result as AI-generated, with human review for important decisions."*

### Likely follow-ups
- **Why not give the LLM the whole dataset?** → Context limits, cost, and calculation errors; compute exactly first.
- **How do you prevent made-up insights?** → Grounding in provided metrics, low temperature, number verification, no-speculation instruction.
- **What is map-reduce summarisation?** → Summarise parts separately, then summarise the summaries.

---

# LESSON 8 — Retrieval-Augmented Generation (RAG) for Data Systems

### Simple idea
The LLM doesn't know **your** data, tables, definitions or runbooks. **RAG = first search your own knowledge, add the best pieces to the prompt, then let the LLM answer from them.** It's like an **open-book exam**: the model looks at the right pages before answering.

### Why RAG?
- Gives access to **private and up-to-date** knowledge without retraining.
- **Reduces hallucination** because answers are grounded in retrieved text and can show **sources**.
- Cheaper and simpler than fine-tuning.
- Keeps prompts small: only the relevant pieces are sent.

### The two phases
```
INDEXING (done ahead of time, repeated when documents change)
  Documents ─► split into chunks ─► convert each chunk to an embedding ─► store in a vector store

QUERYING (every question)
  Question ─► convert to embedding ─► find most similar chunks (top-k) ─► put them in the prompt
           ─► LLM answers using ONLY that context ─► answer + sources
```

### Key points

**1. Building blocks**
| Block | Meaning and notes |
|---|---|
| **Chunking** | Splitting documents into smaller pieces (a few hundred words) with a little **overlap** so context isn't cut off. Too big = noisy; too small = loses meaning |
| **Embedding model** | Converts text to meaning-vectors. **Use the same model** for documents and questions |
| **Vector store** | A database that stores vectors and finds the nearest ones quickly. Options: FAISS or Chroma (local), OpenSearch Serverless, PostgreSQL with pgvector (AWS), Bedrock Knowledge Bases (managed) |
| **Retrieval** | Fetch the top few most similar chunks; can add **metadata filters** (department, date) and **hybrid search** (keyword + vector) |
| **Generation** | Prompt says: "Answer only from the context; if not found, say you don't know; cite sources" |
| **Evaluation** | Check retrieval quality (right chunk found?) and answer faithfulness (matches context?) |

**2. RAG for data systems: what do you index?**
This is what your syllabus title points to. In a data platform, the knowledge is not only PDFs:
| Knowledge source | What RAG helps with |
|---|---|
| **Table and column descriptions** (data catalog / dictionary) | Finding the **right tables** for a question when there are hundreds |
| **Business glossary and metric definitions** | Correct meaning of "revenue", "active customer", "churn" |
| **Example question → SQL pairs** | Pulling similar solved examples as few-shot guidance; big accuracy boost |
| **Data-quality rules and lineage notes** | Explaining why a number looks different |
| **Pipeline runbooks and incident notes** | Troubleshooting assistant ("what to do when job X fails") |
| **Data policies, contracts, SLAs** | Governance questions |

**3. RAG-powered text-to-SQL (best-practice pattern)**
```
Question ─► retrieve relevant table descriptions + glossary + similar example queries
         ─► prompt with only those ─► LLM writes SQL ─► validate ─► run
```
Why it's better: with many tables, sending the whole schema is too big and confusing. RAG sends only what's relevant.

**4. Bedrock Knowledge Bases (managed RAG)**
You store documents in S3, create a knowledge base choosing an embedding model and vector store, and AWS handles chunking, embedding and retrieval. It can return either the relevant chunks or a generated answer with citations. Less work, less control, additional service cost.

**5. Common problems and fixes**
| Problem | Fix |
|---|---|
| Wrong chunk retrieved | Better chunking, metadata, more results, hybrid search, rewrite the question |
| Model ignores context | Stronger instruction ("only the context"), clear delimiters, lower temperature |
| Outdated answers | Re-index when documents/metadata change |
| Sensitive data exposure | Filter by user permissions at retrieval; mask PII before indexing |
| High cost | Cache embeddings, fewer chunks, cheaper model for simple Q&A |

**6. RAG vs fine-tuning**
| | RAG | Fine-tuning |
|---|---|---|
| Adds | Knowledge at question time | Changed behaviour/style/format |
| Data changes often? | Easy (re-index) | Needs retraining |
| Shows sources? | Yes | No |
| Cost/effort | Lower | Higher |
| Typical choice for data systems | **First choice** | Rarely needed |

### Why it matters for data engineering
Data engineers already own the **metadata** (catalog, lineage, definitions). RAG turns that metadata into something the LLM can use, making GenAI answers more accurate and trustworthy. It also creates a new job: **keeping the knowledge base clean and current**.

### 🎤 Say it in the interview
*"RAG means retrieving relevant information from our own knowledge sources and adding it to the prompt so the LLM answers from it instead of from memory. Documents are chunked, converted to embeddings and stored in a vector store; at query time the question is embedded, the nearest chunks are retrieved and passed to the model with an instruction to answer only from them. In a data platform I'd index table and column descriptions, the business glossary, example queries and runbooks, and use that to pick the right tables for text-to-SQL. It reduces hallucination and works on private, changing data without retraining."*

### Likely follow-ups
- **RAG vs fine-tuning?** → RAG supplies knowledge at query time and is easy to update with citations; fine-tuning changes behaviour and is costly. Start with RAG.
- **What is chunking and why overlap?** → Splitting text for retrieval; overlap preserves context across boundaries.
- **What is an embedding?** → A numeric vector representing meaning; similar texts are close together.
- **What if the retrieved chunks don't contain the answer?** → The prompt tells the model to say it doesn't know, which prevents guessing.
- **Which AWS services support RAG?** → Bedrock (models, embeddings, Knowledge Bases), OpenSearch Serverless, Aurora/RDS PostgreSQL with pgvector, S3 for source documents.

---

# LESSON 9 — GenAI with Pandas and DataFrames

### Simple idea
Pandas is excellent at **exact computation** on tables; the LLM is excellent at **language**. Combining them lets you **ask questions about a DataFrame**, **clean and enrich text columns**, and **generate documentation**, as long as you keep control of what actually runs.

### Key points

**1. Golden rule: describe the data, don't dump it**
Give the LLM the **column names and types, a few sample rows, and summary statistics**, not the full dataset (cost, privacy, context limits).

**2. Four main patterns**

| Pattern | What happens | Risk level |
|---|---|---|
| **A. LLM writes pandas code** | You ask a question; the model writes code; you run it | **High**: running model-written code can be unsafe |
| **B. LLM returns a plan; you execute it** | The model outputs a structured plan (filter, group, aggregate, sort, limit); *your* trusted code carries it out | **Low**: only allowed operations run |
| **C. Row-level enrichment** | Apply the LLM to each row/value: classify, extract fields, clean, standardise | Medium: needs validation and cost control |
| **D. Documentation and rules** | LLM drafts column descriptions, validation rules, test ideas from the profile | Low: a human reviews |

**3. Why pattern A needs care**
Executing model-generated code is like running code written by a stranger. Risks: reading/deleting files, network access, wrong or destructive operations. If used at all, it should be in a **sandbox**, on a **copy of the data**, with checks for dangerous operations. **Pattern B is the safer production approach** because the model never gets to run arbitrary code.

**4. Row-level enrichment: how to do it responsibly**
- **Validate** outputs against an allowed list (for example, ticket category must be one of four labels; otherwise mark "needs review").
- **Process unique values only** and map results back, to reduce calls and cost.
- **Batch** several rows in one request where possible (and check the count matches).
- **Cache** results; **retry** on throttling; control request speed.
- **Estimate cost** first: number of rows × tokens per row.
- Keep a **human review path** for uncertain outputs.

**5. Useful applications**
- **Classify** support tickets, reviews, transactions.
- **Extract entities** from free text into new columns (names, amounts, dates).
- **Standardise values**: "Bengaluru / Bangalore / BLR" → one value. A good approach is for the LLM to **propose a mapping** and a person to approve it before applying.
- **Auto-document** columns and tables for the data catalog.
- **Suggest data-quality rules** (for example, amount ≥ 0, date not in the future).
- **Summarise** a DataFrame's profile for a report.

**6. AWS helper (mention only)**
The **AWS SDK for pandas (awswrangler)** makes it easy to read Athena query results into a DataFrame and write DataFrames to S3 with catalog registration.

**7. Limitations**
LLM row-by-row processing is slower and costlier than normal pandas operations. Use **normal pandas for anything rule-based** (filters, joins, math, regex) and use the LLM only where **understanding language is needed**.

### Why it matters for data engineering
It bridges **messy real-world text** and **clean structured tables**, which is a large part of data cleaning, but it must be done with validation, cost control and security.

### 🎤 Say it in the interview
*"With pandas I never send the whole DataFrame to the LLM, only the schema, sample rows and summary statistics. For questions, the safest pattern is to have the model return a structured plan that my own code executes, rather than running model-generated code. For row-level tasks like classification or entity extraction I validate every output against allowed values, process unique values only, batch and cache requests, and send uncertain results to review. Rule-based work stays in plain pandas; the LLM is only for language understanding."*

### Likely follow-ups
- **Is it safe to execute LLM-generated code?** → Not by default; use a sandbox and checks, or prefer a plan-and-execute approach.
- **When should you not use an LLM on a column?** → When simple rules or regex are enough; LLMs are slower, costlier and less deterministic.
- **How do you handle large DataFrames?** → Work on unique values, batch, sample for profiling, aggregate first.

---

# PART B — PUTTING IT TOGETHER

## B1. Your GenAI use case for the hackathon (pick one, explain it simply)

| Use case | One-line explanation | Difficulty |
|---|---|---|
| **A. "Ask your data"** (text-to-SQL) | User asks in English → validated SQL on Athena → results + short explanation | Medium |
| **B. Auto-insight report** | SQL computes metrics → LLM writes a short business summary | Easy |
| **C. Data-quality explainer / auto-documentation** | LLM explains rejected rows and writes column descriptions | Easy–Medium |
| **D. RAG assistant** | Answers questions about tables, definitions and runbooks with sources | Medium |

**Recommended story:** A + B. *"The user asks a question; the system shows the SQL it generated, the result table, and a three-line insight."*

**The architecture you can describe in 30 seconds:**
```
Source files / API ─► S3 raw ─► Glue transformation ─► S3 curated (Parquet) ─► Glue Catalog ─► Athena SQL
                                                                                        │
                    User question ─► (RAG: schema + glossary) ─► LLM on Bedrock ─► validated SQL ─► Athena ─► results
                                                                                                         └─► LLM insight summary
```

## B2. If the interviewer asks "walk me through your GenAI design"

1. **Problem:** business users can't write SQL, so they depend on engineers.
2. **Solution:** a natural-language assistant on top of curated data.
3. **How it works:** schema and definitions from the data catalog → prompt → SQL draft → validation → read-only execution → result and summary.
4. **Safety:** read-only role, SELECT-only validation, row and scan limits, logging, SQL shown to the user.
5. **Accuracy:** real schema, business glossary, few-shot examples, retry with error feedback, a test set of questions.
6. **Hallucination handling:** grounded prompts, a "cannot answer" option, validation against the catalog.
7. **Cost:** small prompts, aggregates only, caching, `max tokens`, cheaper models for simple steps.
8. **Limitations:** ambiguous questions, complex joins, so important results need human review.
9. **Next steps:** RAG over the catalog for many tables, Bedrock Guardrails, feedback loop.

## B3. Master interview Q&A

**Fundamentals**
1. **What is Generative AI?** Models that create new content (text, code, images) after learning patterns from large datasets.
2. **What is an LLM?** A large Transformer-based model trained to predict the next token, able to perform many language tasks from instructions.
3. **What is a token and a context window?** A token is a piece of text used for billing and limits; the context window is the maximum amount of text the model can handle in one request.
4. **What does temperature do?** Controls randomness; near zero for consistent output such as SQL.
5. **What is hallucination?** Confident but incorrect output. Reduced by grounding, low temperature, validation and "I don't know" instructions.
6. **LLM vs traditional ML?** Traditional ML is trained for one specific task; an LLM is a general model guided by prompts.

**NLP**
7. **What is tokenization / embedding?** Splitting text into pieces / representing meaning as a vector.
8. **TF-IDF vs embeddings?** TF-IDF matches exact words; embeddings match meaning.
9. **What is cosine similarity?** A measure of how similar two vectors are, used for semantic search.

**Prompting**
10. **What is prompt engineering?** Designing clear instructions (role, task, context, constraints, format, examples) to get reliable output.
11. **Zero-shot vs few-shot?** Without vs with examples.
12. **What is prompt injection?** Hidden instructions in input data that try to override your rules; defend with clear separation, validation and least privilege.
13. **How do you get structured output?** Request strict JSON, parse and validate, retry on failure.

**APIs**
14. **How do you call an LLM from Python on AWS?** Through the Bedrock Runtime using the AWS SDK, with IAM-based access.
15. **Why do you resend conversation history?** The API is stateless.
16. **How do you handle rate limits?** Retry with exponential backoff, batching and caching.
17. **What is Bedrock?** AWS's managed service offering multiple foundation models through one API, plus Knowledge Bases, Agents and Guardrails.

**Data use cases**
18. **How does text-to-SQL work?** Schema + rules + examples go into the prompt; the LLM drafts SQL; it is validated and run read-only; errors can be fed back for a fix.
19. **How do you keep it safe?** Read-only role, validation, limits, logging, human review.
20. **Why use templates in automation?** They are predictable, cheap and safe for repeated questions; the LLM only chooses the template and extracts parameters.
21. **How do you generate insights without hallucination?** Compute exact metrics first, give only those to the LLM, instruct it to use only those numbers, then verify.
22. **Why not send the entire table to the LLM?** Cost, context limits, privacy and calculation errors.

**RAG**
23. **What is RAG?** Retrieve relevant information from your knowledge sources and add it to the prompt so the model answers from it.
24. **RAG vs fine-tuning?** RAG adds fresh, citable knowledge cheaply; fine-tuning changes behaviour and is costly.
25. **What is chunking?** Splitting documents into smaller pieces for retrieval, with some overlap.
26. **Where would RAG help in a data platform?** Finding relevant tables, glossary definitions, example queries, runbooks and data policies.
27. **AWS services for RAG?** Bedrock (models and Knowledge Bases), OpenSearch Serverless, Aurora/RDS PostgreSQL with pgvector, S3.

**Pandas**
28. **Is it safe to run LLM-generated pandas code?** Not by default; sandbox it, or have the LLM output a plan that trusted code executes.
29. **How do you do row-level enrichment safely?** Validate outputs, deduplicate inputs, batch, cache and keep a review path.

**General**
30. **How do you evaluate a GenAI feature?** A test set with expected results, accuracy, faithfulness to context, latency, cost and error rate.
31. **How do you control GenAI cost?** Smaller prompts, aggregated data, caching, cheaper models for simple tasks, token limits, usage logging.
32. **What are Guardrails?** Configurable filters for harmful content, denied topics, sensitive data and grounding checks.
33. **How can GenAI help with data quality?** Explain rejected records, suggest validation rules, document columns, propose standardisation mappings reviewed by humans.
34. **What are the risks of GenAI in data engineering?** Hallucination, data leakage, prompt injection, cost spikes, unreviewed wrong answers.
35. **Where should GenAI NOT be used?** Where exact deterministic rules suffice (math, joins, regex), or for unreviewed high-stakes decisions.

## B4. Quick comparison tables

**Prompting vs RAG vs Fine-tuning**
| | Prompting | RAG | Fine-tuning |
|---|---|---|---|
| Effort | Lowest | Medium | Highest |
| Handles private data | Small amounts | **Yes, well** | Yes, baked in |
| Updates easily | Yes | **Yes** | No |
| Shows sources | No | **Yes** | No |

**LLM does vs database/pandas does**
| LLM is good at | SQL / pandas is good at |
|---|---|
| Understanding questions | Exact calculation |
| Drafting SQL, text, summaries | Filtering, joining, aggregating |
| Classifying and extracting from text | Handling millions of rows |
| Explaining results | Deterministic, auditable results |

**Safety checklist (memorise)**
1. Read-only access.
2. Validate generated SQL/code.
3. Limit rows, scans and runtime.
4. Ground answers in real schema/documents.
5. Show and log generated queries.
6. Keep sensitive data out of prompts.
7. Defend against prompt injection.
8. Human review for important decisions.

## B5. Glossary (quick revision)

| Term | Meaning |
|---|---|
| **LLM** | Large Language Model that generates text/code |
| **Token** | Piece of text; unit of billing and limits |
| **Context window** | Maximum text per request |
| **Temperature** | Randomness control |
| **Hallucination** | Confident wrong output |
| **Prompt** | Instruction given to the model |
| **Few-shot** | Prompt with examples |
| **Prompt injection** | Malicious instructions hidden in data |
| **Embedding** | Meaning represented as numbers |
| **Cosine similarity** | Score of how alike two vectors are |
| **Vector store** | Database for finding similar embeddings |
| **Chunking** | Splitting documents for retrieval |
| **RAG** | Retrieve relevant knowledge, then generate |
| **Guardrails** | Filters for safe model input/output |
| **Text-to-SQL** | Natural-language question to SQL query |
| **Grounding** | Anchoring answers in provided facts |
| **Agent** | LLM that chooses and uses tools |
| **Bedrock** | AWS service for foundation models |
| **Knowledge Base (Bedrock)** | Managed RAG |
| **Fine-tuning** | Further training of a model on your examples |

## B6. Last-minute revision (read this the night before)

1. **GenAI = creates content; LLM = predicts next token.**
2. **LLMs hallucinate** → ground with schema/RAG, validate, low temperature.
3. **Prompt = role + task + context + constraints + format + examples.**
4. **APIs are stateless; handle throttling with retry/backoff; secure with IAM.**
5. **Text-to-SQL = schema in prompt → SQL → validate → read-only run → retry on error.**
6. **Automation = templates first, LLM second; log and limit everything.**
7. **Insights = compute exactly first, then the LLM explains only those numbers.**
8. **RAG = chunk → embed → store → retrieve → answer from context with sources.**
9. **RAG for data systems = index catalog descriptions, glossary, example SQL, runbooks.**
10. **Pandas + LLM = send profile not data; prefer plan-and-execute over running generated code; validate every output.**
11. **The LLM does the language; the database does the math.**

## Optional bonus (not in your syllabus)

HCLTech's separate GenAI Analyst / AI Engineer job posts mention **LangChain, LangGraph, vector databases like ChromaDB and Pinecone, and AI agents**. Your Cloud Data Engineer syllabus does not require them. If asked, a safe answer is: *"LangChain and LangGraph are frameworks that organise prompts, tools and retrieval into chains and agent workflows. I've focused on the core concepts and the AWS-native approach with Bedrock, and I understand how those frameworks would orchestrate the same steps."*

---

**You are done with Phase 6.** Next: Phase 7 (hackathon practical) and Phase 8 (HCLTech interview). Use the Q&A in B3 and the story in B2 for interview practice.
