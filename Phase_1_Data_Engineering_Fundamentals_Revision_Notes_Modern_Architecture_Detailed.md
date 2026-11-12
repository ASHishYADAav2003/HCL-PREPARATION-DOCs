# Phase 1 Revision Notes --- Data Engineering Fundamentals

**Coverage:** Days 1--5 · Lessons 1--15\
**Status:** Completed\
**Purpose:** Revision before the AWS hackathon and HCLTech technical
interview.

> These notes consolidate the concepts, examples, diagrams and
> corrections covered in Phase 1. Olist project details are included
> only where confirmed. Suggested warehouse designs are examples, not
> claims about implemented features.

## Phase 1 at a glance

  ---------------------------------------------------------------------
  Day                                      Lessons Topics
  -------------------- --------------------------- --------------------
  1                                           1--3 Data Engineering,
                                                   OLTP/OLAP,
                                                   databases,
                                                   warehouses and lakes

  2                                           4--6 Lake vs warehouse vs
                                                   lakehouse, ETL/ELT,
                                                   batch/streaming

  3                                           7--9 Pipeline
                                                   architecture, data
                                                   layers, EDW and data
                                                   marts

  4                                         10--12 Data modelling,
                                                   keys, normalization
                                                   and denormalization

  5                                         13--15 Fact/dimension
                                                   tables,
                                                   Star/Snowflake
                                                   schemas, SCD
  ---------------------------------------------------------------------

------------------------------------------------------------------------

# Day 1 --- Data Engineering Fundamentals

## 1. What is Data Engineering?

**Data Engineering** is the discipline of designing, building and
maintaining systems that collect, move, clean, transform, store and
deliver data so it can be used reliably.

A **Data Engineer** builds and maintains those systems.

### Typical data pipeline

``` mermaid
flowchart LR
    A[Data Sources] --> B[Ingestion]
    B --> C[Processing and Transformation]
    C --> D[Storage]
    D --> E[Consumption]
    E --> F[Reports, Dashboards, Analytics]
```

-   **Source:** Where data originates, such as application databases,
    CSV files or APIs.
-   **Ingestion:** Bringing data into the pipeline.
-   **Processing:** Cleaning, validating and transforming data.
-   **Storage:** Saving data in an appropriate system and format.
-   **Consumption:** Making data available to analysts, applications,
    dashboards or machine-learning systems.

**Example:** An online store ingests order records, validates and
transforms them, stores analytical datasets, and makes them available
for sales reports.

### Olist project connection

  Technology              Role in the project
  ----------------------- -----------------------------------------------------
  Python and Pandas       Data handling, cleaning and transformation
  PySpark                 DataFrame processing and analysis
  PyArrow / Parquet       Parquet file support and outputs
  DuckDB and SQL          Local SQL analytics
  Amazon S3               Object storage for six uploaded Gold datasets
  AWS Glue Data Catalog   Metadata catalog for tables
  Amazon Athena           SQL queries over data in S3
  Streamlit               Business Dashboard and simple AI Business Assistant

**Accuracy note:** The project has Raw, Silver and Gold processing
stages, but the confirmed S3 upload was six Gold datasets. Do not claim
all Raw, Silver and Gold datasets were uploaded to S3.

## 2. OLTP vs OLAP

### OLTP --- Online Transaction Processing

Supports day-to-day business transactions.

Examples: placing an order, making a payment, updating a customer's
address, booking a ticket.

Typical characteristics: - Many small, frequent transactions. - Current
operational data. - Fast inserts, updates and deletes. - Often involves
individual records.

### OLAP --- Online Analytical Processing

Supports analysis and business decisions.

Examples: total sales by month, top-selling products, repeat customer
analysis, revenue trends by city.

Typical characteristics: - Read-heavy analytical queries. - Aggregations
over many records. - Historical and integrated data. - Supports
dashboards and business intelligence.

  -----------------------------------------------------------------------
  Feature                 OLTP                    OLAP
  ----------------------- ----------------------- -----------------------
  Main purpose            Run business operations Analyze business data

  Typical operation       Insert, update, delete, Filter, join, aggregate
                          lookup                  

  Data focus              Current transactions    Historical and
                                                  analytical

  Example                 Record an order         Calculate monthly sales
  -----------------------------------------------------------------------

**Olist connection:** The project analyzes historical CSV datasets.
DuckDB and Athena analysis is OLAP-style. Reading CSV files into a
pipeline is not itself an OLTP workload.

## 3. Database vs Data Warehouse vs Data Lake

-   **Database:** Stores and manages data for applications and
    operational workloads. Example: an online store's application
    database.
-   **Data Warehouse:** Integrates and organizes data for reporting,
    analytics and decision-making. Example: combining sales, customer
    and product data for business reports.
-   **Data Lake:** Stores large amounts of data in original or processed
    form, including structured, semi-structured and unstructured data.
    Examples include CSV, Parquet, JSON, logs and images.

  System           Main purpose                 Example
  ---------------- ---------------------------- ----------------------------
  Database         Application operations       Order transactions
  Data Warehouse   Structured analytics         Enterprise sales reporting
  Data Lake        Store varied data at scale   Files in Amazon S3

### AWS project connection

``` mermaid
flowchart TD
    A[Gold Parquet files] --> B[Amazon S3]
    B --> C[AWS Glue Data Catalog]
    C --> D[Amazon Athena]
    D --> E[SQL analysis and reporting]
```

-   S3 stores objects/files.
-   Parquet is a columnar file format, not a storage service.
-   Glue Data Catalog stores metadata about datasets and tables.
-   Athena uses table metadata to query data in S3.

------------------------------------------------------------------------

# Day 2 --- Data Storage and Processing Patterns

## 4. Data Lake vs Data Warehouse vs Lakehouse

-   **Data Lake:** Stores varied data types and formats; flexible for
    different workloads.
-   **Data Warehouse:** Provides curated, structured data optimized for
    business reporting and analytics.
-   **Lakehouse:** Combines aspects of data lakes and data warehouses,
    aiming to support flexible data storage with warehouse-style
    management and analytics capabilities.

  -----------------------------------------------------------------------
  Feature           Data Lake         Data Warehouse    Lakehouse
  ----------------- ----------------- ----------------- -----------------
  Data types        Structured and    Mostly structured Varied
                    unstructured                        

  Storage approach  Flexible          Managed           Lake storage with
                    file/object       analytical        additional
                    storage           storage           management
                                                        capabilities

  Common use        Raw and varied    BI and reporting  Analytics and
                    data                                broader data
                                                        workloads
  -----------------------------------------------------------------------

**Olist connection:** The project uses an S3-based, Parquet-backed
analytical workflow with Glue Catalog and Athena. Describe it as a
data-lake-style architecture with SQL analytics. Do not call it a full
lakehouse unless the relevant lakehouse capabilities are actually
implemented.

## 5. ETL vs ELT

### ETL --- Extract, Transform, Load

1.  Extract data from sources.
2.  Transform and clean it.
3.  Load the prepared output into the target.

``` mermaid
flowchart LR
    A[Source] --> B[Extract]
    B --> C[Transform]
    C --> D[Load]
    D --> E[Target]
```

### ELT --- Extract, Load, Transform

1.  Extract data.
2.  Load it into the target storage/platform.
3.  Transform it there.

``` mermaid
flowchart LR
    A[Source] --> B[Extract]
    B --> C[Load raw data]
    C --> D[Transform in target platform]
    D --> E[Curated data]
```

  ---------------------------------------------------------------------
  ETL                                ELT
  ---------------------------------- ----------------------------------
  Transform before loading to target Load before transforming

  Target receives transformed output Target may first receive raw data

  Transformations happen before      Target platform performs
  target storage                     transformations
  ---------------------------------------------------------------------

**Olist connection:** The local workflow that creates Silver and Gold
Parquet outputs before uploading selected Gold files to S3 is ETL-style.
A design that first lands raw data in S3 and transforms it later in AWS
would be ELT-style. Explain the actual sequence rather than forcing one
label onto every part of a pipeline.

## 6. Batch Processing vs Streaming

-   **Batch:** Processes a collection of records together at scheduled
    or chosen intervals. Examples: daily sales report, monthly payroll,
    historical CSV processing.
-   **Streaming:** Processes events continuously or with very low
    latency as they arrive. Examples: live payment events, delivery
    tracking, sensor readings.

  Feature      Batch                     Streaming
  ------------ ------------------------- -----------------------
  Processing   Groups of records         Continuous event flow
  Latency      Usually higher            Usually lower
  Example      Daily sales calculation   Live order tracking

**Olist connection:** Olist processes historical datasets in a
batch-oriented manner. Do not describe it as a real-time streaming
pipeline.

------------------------------------------------------------------------

# Day 3 --- Pipeline Architecture and Data Organization

## 7. Data Pipeline Architecture

A data pipeline moves data from sources through ingestion and processing
into storage and consumption systems.

``` mermaid
flowchart LR
    A[Sources: CSV / APIs / DB] --> B[Ingestion]
    B --> C[Validation and Cleaning]
    C --> D[Transformation]
    D --> E[Storage]
    E --> F[SQL / BI / Applications]
```

A reliable pipeline also needs: - Data quality checks. - Error
handling. - Logging and monitoring. - Repeatable execution. - Security
and access control. - Documentation.

**Orchestration** coordinates pipeline tasks and dependencies. For
example, ingestion can run first, followed by transformation, validation
and reporting.

**Accuracy note:** AWS Step Functions is a possible orchestration
service, but it was not confirmed as implemented in the Olist project.

## 8. Source, Staging, Processing and Consumption Layers

  -----------------------------------------------------------------------
  Layer                   Purpose                 Example
  ----------------------- ----------------------- -----------------------
  Source                  Original data producer  E-commerce CSV exports

  Staging                 Temporary or landing    Newly ingested files
                          area for incoming data  

  Processing              Clean and transform     Validated Silver
                          data                    datasets

  Consumption             Data ready for users    Gold datasets, SQL
                          and analytics           tables, dashboards
  -----------------------------------------------------------------------

### Medallion architecture

-   **Bronze:** Raw or minimally changed data.
-   **Silver:** Cleaned and standardized data.
-   **Gold:** Curated, business-ready data.

``` mermaid
flowchart LR
    A[Bronze: Raw] --> B[Silver: Cleaned]
    B --> C[Gold: Business-ready]
    C --> D[Analytics and Dashboards]
```

Source/staging/processing/consumption and Bronze/Silver/Gold are related
ways of describing data flow, but they are not interchangeable terms.

**Olist connection:** The project has Raw, Silver and Gold processing
stages. Six Gold datasets were confirmed as uploaded to S3. Do not claim
all layers are in S3.

## 9. Enterprise Data Warehouse (EDW) and Data Marts

-   **EDW:** Integrates data across an organization to support
    enterprise-wide analytics. Example: combining finance, sales,
    customers, operations and inventory data.
-   **Data Mart:** A smaller, subject- or department-focused analytical
    store. Examples: Sales Data Mart, Finance Data Mart, Customer Data
    Mart.

  EDW                              Data Mart
  -------------------------------- -----------------------------
  Enterprise-wide scope            Department or subject scope
  Integrates many business areas   Focuses on a specific area
  Broad analytics                  Targeted analytics

**Important:** Having several separate CSV files does not automatically
mean a project has data marts. Olist is a multi-dataset analytics
pipeline; formal Sales or Customer Data Marts were not confirmed.

------------------------------------------------------------------------

# Day 4 --- Data Modelling and Table Design

## 10. Data Modelling Fundamentals

**Data modelling** is the process of designing how data is organized,
represented, stored and related.

  -----------------------------------------------------------------------
  Model                   Meaning                 Example
  ----------------------- ----------------------- -----------------------
  Conceptual              High-level business     Customer places Order
                          entities and            
                          relationships           

  Logical                 Entities, attributes,   Customer(customer_id,
                          keys and relationships, name)
                          independent of a        
                          specific database       

  Physical                Actual implementation   SQL table definitions,
                          in a chosen             data types, constraints
                          database/platform       and indexes
  -----------------------------------------------------------------------

### E-commerce relationship example

``` mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : appears_in
    SELLER ||--o{ ORDER_ITEM : supplies
```

This is a conceptual illustration of common e-commerce relationships,
not a complete physical schema of the Olist project.

## 11. Primary Keys and Foreign Keys

### Primary Key (PK)

A primary key uniquely identifies each row in a table. - Must be
unique. - Must not be NULL.

Example:

  customer_id (PK)   customer_name
  ------------------ ---------------
  C101               Rahul
  C102               Priya

### Foreign Key (FK)

A foreign key is a column or set of columns that references a candidate
key in another table, commonly its primary key.

**Customers**

  customer_id (PK)   name
  ------------------ -------
  C101               Rahul
  C102               Priya

**Orders**

  order_id (PK)   customer_id (FK)
  --------------- ------------------
  O101            C101
  O102            C101
  O103            C102

One customer can have multiple orders: a one-to-many relationship.

### Common relationship types

-   **One-to-one (1:1):** One record relates to one record.
-   **One-to-many (1:M):** One customer can have many orders.
-   **Many-to-many (M:N):** Many students can take many courses;
    commonly represented using a junction table.

### Olist relationship examples

-   Customers to Orders through `customer_id`.
-   Orders to Order Items through `order_id`.
-   Products to Order Items through `product_id`.
-   Sellers to Order Items through `seller_id`.

In CSV datasets, relationships may be inferred from schema and data.
They are not necessarily enforced as database constraints.

**Important Olist detail:** `customer_unique_id` identifies a real
customer across multiple customer records and is important for
repeat-customer analysis.

## 12. Normalization vs Denormalization

### Normalization

Organizes relational data into related tables to reduce unnecessary
duplication and improve consistency and integrity.

**Customers**

  customer_id   name    city
  ------------- ------- --------
  C101          Rahul   Delhi
  C102          Priya   Mumbai

**Orders**

  order_id   customer_id     amount
  ---------- ------------- --------
  O101       C101              1500
  O102       C101              2500
  O103       C102              3000

Rahul's customer details are stored once and referenced by
`customer_id`.

### Normal Forms --- introductory definitions

-   **1NF:** Columns contain atomic values; avoid repeating groups or
    multiple values packed into one cell.
-   **2NF:** Satisfies 1NF and has no partial dependency of a non-key
    attribute on a proper subset of a candidate key. Especially relevant
    to composite keys.
-   **3NF:** Satisfies 2NF and has no transitive dependency of non-key
    attributes on a candidate key.

### Denormalization

Intentionally combines related data or introduces controlled redundancy
to make analytical queries simpler or faster.

  order_id   customer_name   product     amount
  ---------- --------------- --------- --------
  O101       Rahul           Laptop       50000
  O102       Rahul           Mouse         1000

  ---------------------------------------------------------------------
  Normalization                      Denormalization
  ---------------------------------- ----------------------------------
  Reduces redundancy through table   May introduce controlled
  design                             redundancy

  Related data stored in separate    Related data combined
  tables                             

  Often useful for transactional     Often useful for analytics and
  consistency                        reporting
  ---------------------------------------------------------------------

**Important distinction:** Data cleaning is not the same as
normalization. - Normalization is a database/table design technique. -
Data cleaning handles issues such as duplicate records, missing values,
invalid values and inconsistent formats.

**Olist connection:** Separate datasets for customers, orders, products
and sellers provide a relational-style organization. Cleaning duplicates
or missing values is a separate processing activity. Gold analytical
outputs may combine information, depending on their actual structures.

------------------------------------------------------------------------

# Day 5 --- Data Warehousing Models

## 13. Fact Tables and Dimension Tables

### Fact Table

Stores measurable business events or transactions, usually with keys
connecting to dimensions.

Typical measures: - Sales amount. - Quantity. - Discount. - Freight
amount. - Profit.

Example:

**FACT_SALES**

  order_id   product_id   customer_id     quantity   sales_amount
  ---------- ------------ ------------- ---------- --------------
  O101       P01          C01                    2           2000
  O102       P02          C02                    1           1500

### Dimension Table

Stores descriptive information that provides context for facts.

**DIM_PRODUCT**

  product_id   product_name   category
  ------------ -------------- -------------
  P01          Laptop         Electronics
  P02          Mouse          Accessories

**DIM_CUSTOMER**

  customer_id   name    city
  ------------- ------- --------
  C01           Rahul   Delhi
  C02           Priya   Mumbai

  Fact                Dimension
  ------------------- ---------------------
  Measurable event    Descriptive context
  Sales amount        Customer name
  Quantity            Product category
  Profit              City
  Transaction count   Date attributes

**Remember:** Order quantity is a fact/measure, not a dimension.

### Types of Fact Tables

1.  **Transaction fact:** One row per business event, such as an order
    item.
2.  **Periodic snapshot:** Measurements at regular intervals, such as
    daily inventory.
3.  **Accumulating snapshot:** Tracks milestones in a process, such as
    order placed, shipped and delivered.

### Olist warehouse-model example

A possible analytical model: - `FACT_ORDER_ITEMS` - `DIM_CUSTOMERS` -
`DIM_PRODUCTS` - `DIM_SELLERS` - `DIM_DATE`

Order Items is a fact-table candidate because each row represents an
order line and can contain measures such as item price and freight.

This is a suggested warehouse model, not a claim that the project
already implemented this exact star schema.

## 14. Star Schema vs Snowflake Schema

### Star Schema

A central fact table connects directly to multiple dimension tables.
Dimension tables are usually denormalized.

``` mermaid
flowchart TD
    P[DIM_PRODUCT] --- F[FACT_SALES]
    C[DIM_CUSTOMER] --- F
    D[DIM_DATE] --- F
    S[DIM_SELLER] --- F
```

Advantages: - Simple structure. - Generally fewer joins. - Convenient
for dashboards and reporting.

Trade-offs: - Dimension attributes may be repeated. - Can use more
storage due to redundancy.

### Snowflake Schema

Dimension tables are further normalized into smaller related tables.

``` mermaid
flowchart TD
    F[FACT_SALES] --- P[PRODUCT]
    P --- C[CATEGORY]
    F --- U[CUSTOMER]
    U --- L[CITY]
```

Advantages: - Reduces repeated dimension information. - Separates
descriptive data into related tables.

Trade-offs: - More joins. - More complex queries and model structure.

  Feature      Star                   Snowflake
  ------------ ---------------------- ----------------------------------
  Dimensions   Usually denormalized   Further normalized
  Structure    Simpler                More complex
  Joins        Generally fewer        Generally more
  Redundancy   More likely            Reduced
  Reporting    Often easier           Can require more complex queries

For straightforward sales dashboards, Star Schema is often a good
starting choice because its structure is simple and analytical queries
often require fewer joins. Snowflake can be useful when dimension data
has significant repetition and normalization is valuable.

**Olist connection:** A possible Star Schema could use Order Items as a
fact table and Customers, Products, Sellers and Date as dimensions. A
Snowflake design could split product category or geography into
additional tables. Neither exact model was confirmed as implemented in
the project.

## 15. Slowly Changing Dimensions (SCD)

SCD techniques manage changes in descriptive dimension attributes over
time.

Example: customer C101 moves from Delhi to Agra.

### Type 0 --- Retain original value

Do not change the original attribute.

  customer_id   city
  ------------- -------
  C101          Delhi

### Type 1 --- Overwrite

Replace the old value. Historical value is not retained in that
dimension record.

  customer_id   city
  ------------- ------
  C101          Agra

Use cases can include correcting misspellings or incorrect information
when history is not required.

### Type 2 --- Add a new row

Insert a new dimension row and preserve the previous row.

    customer_key customer_id   city    is_current
  -------------- ------------- ------- ------------
               1 C101          Delhi   No
               2 C101          Agra    Yes

A date-based version:

    customer_key customer_id   city    start_date   end_date
  -------------- ------------- ------- ------------ ------------
               1 C101          Delhi   2025-01-01   2025-06-30
               2 C101          Agra    2025-07-01   NULL

### Type 3 --- Keep limited history

Store current and previous values in separate columns.

  customer_id   current_city   previous_city
  ------------- -------------- ---------------
  C101          Agra           Delhi

### Comparison

  Type   Action                      History
  ------ --------------------------- ----------------------
  0      Keep original value         Original value only
  1      Overwrite old value         Not retained
  2      Insert a new row            Full tracked history
  3      Add previous-value column   Limited history

### Surrogate Key

A surrogate key is a system-generated identifier for a dimension record.
In SCD Type 2, each historical version commonly receives a different
surrogate key while the business/customer identifier remains the same.

**Olist connection:** SCD was taught as a possible warehouse
enhancement. Do not claim the existing Olist pipeline implemented SCD
unless it is actually built and verified.

------------------------------------------------------------------------

# Interview Revision Sheet

## Key definitions to practise

1.  **Data Engineering:** Building and maintaining systems that ingest,
    process, store and deliver reliable data.
2.  **OLTP:** Workloads that support operational transactions.
3.  **OLAP:** Workloads that support analytical queries and reporting.
4.  **Data Lake:** Storage for varied data types and formats, often at
    scale.
5.  **Data Warehouse:** Integrated, structured data for analytics and
    reporting.
6.  **ETL:** Extract, Transform, Load.
7.  **ELT:** Extract, Load, Transform.
8.  **Batch:** Process data in groups at intervals.
9.  **Streaming:** Process events continuously or with low latency.
10. **Primary Key:** Unique, non-NULL identifier for a row.
11. **Foreign Key:** Column(s) referencing a candidate key in another
    table.
12. **Normalization:** Organizing tables to reduce redundancy and
    improve consistency.
13. **Denormalization:** Combining related data or adding controlled
    redundancy for analytics.
14. **Fact Table:** Stores business events and measures.
15. **Dimension Table:** Stores descriptive context.
16. **Star Schema:** Central fact connected directly to dimensions.
17. **Snowflake Schema:** Dimensions split into related normalized
    tables.
18. **SCD Type 1:** Overwrite.
19. **SCD Type 2:** New row and historical tracking.
20. **Surrogate Key:** System-generated identifier for a dimension
    record.

## Common mistakes to avoid in interviews

-   Do not say missing-value removal is normalization. It is data
    cleaning.
-   Do not classify order quantity as a dimension; it is a measure/fact.
-   Do not say a sales department is a fact table; `FACT_SALES` is a
    table.
-   Do not call historical Olist batch processing real-time streaming.
-   Do not call the Olist project a full lakehouse without the relevant
    implementation.
-   Do not claim all Raw, Silver and Gold layers were uploaded to S3;
    six Gold datasets were confirmed uploaded.
-   Do not claim Step Functions, Glue ETL, Kinesis, Redshift, Bedrock or
    SCD were implemented unless they actually were.
-   Do not describe a suggested Star/Snowflake model as an existing
    project feature.
-   Distinguish `customer_id` from `customer_unique_id` when discussing
    repeat customers.

## Olist project explanation --- concise version

> I built an e-commerce data engineering pipeline using the Olist
> Brazilian e-commerce dataset. The project uses Python and Pandas for
> data handling and transformation, PySpark for DataFrame processing and
> analysis, Parquet for analytical file outputs, and DuckDB for local
> SQL analysis. I also uploaded six Gold datasets to Amazon S3,
> configured the AWS Glue Data Catalog, created Athena tables and
> queried the data using SQL. A Streamlit application provides a
> Business Dashboard and a simple AI Business Assistant.

Only use this description if it remains accurate to the actual
implementation. The AI Business Assistant is a simple answer-generating
chatbot; it should not be described as RAG-based or SQL-connected unless
those capabilities are later implemented.

------------------------------------------------------------------------

# Phase 1 Completion Checklist

-   [x] Lesson 1 --- What is Data Engineering?
-   [x] Lesson 2 --- OLTP vs OLAP
-   [x] Lesson 3 --- Database vs Data Warehouse vs Data Lake
-   [x] Lesson 4 --- Data Lake vs Data Warehouse vs Lakehouse
-   [x] Lesson 5 --- ETL vs ELT
-   [x] Lesson 6 --- Batch vs Streaming
-   [x] Lesson 7 --- Data Pipeline Architecture
-   [x] Lesson 8 --- Source, Staging, Processing and Consumption
-   [x] Lesson 9 --- EDW and Data Marts
-   [x] Lesson 10 --- Data Modelling Fundamentals
-   [x] Lesson 11 --- Primary and Foreign Keys
-   [x] Lesson 12 --- Normalization vs Denormalization
-   [x] Lesson 13 --- Fact and Dimension Tables
-   [x] Lesson 14 --- Star Schema vs Snowflake Schema
-   [x] Lesson 15 --- Slowly Changing Dimensions

**Phase 1 status: Completed.**

**Next phase:** Phase 2 --- SQL, beginning with Day 6, Lesson 16:
`SELECT`, `FROM` and `WHERE`.

*Revision tip: Before the hackathon, revise the diagrams, comparison
tables, Olist project connection notes and interview definitions.
Practise explaining each concept aloud without reading the answer.*

------------------------------------------------------------------------

------------------------------------------------------------------------

# Modern Data Architecture

These approaches address different problems. Lambda and Kappa focus
primarily on data-processing architecture, while a lakehouse focuses on
storage and analytical platform capabilities.

## 8.1 Lambda Architecture

Lambda Architecture combines batch processing and stream processing to
provide both comprehensive historical results and low-latency results
from recent events.

The architecture has three components:

-   **Batch layer --- cold path**
-   **Speed layer --- hot path**
-   **Serving layer**

### A. Batch layer --- cold path

The batch layer processes large volumes of historical data periodically.

Characteristics:

-   Handles large datasets.
-   Can perform comprehensive calculations.
-   Often uses scheduled batch jobs.
-   May take minutes or hours to refresh, depending on the workload.

**Example:** Calculate each customer's total purchases across the last
three years every night.

### B. Speed layer --- hot path

The speed layer processes new events quickly to provide results before
the next batch job finishes.

Characteristics:

-   Low-latency processing.
-   Handles recent or continuously arriving events.
-   Updates results as new data arrives.

**Example:** Update the current count of online orders as each new order
event arrives.

### C. Serving layer

The serving layer exposes the results from the batch and speed layers to
querying applications, dashboards, or APIs.

Depending on the implementation, the serving layer combines or
reconciles batch results with real-time results.

### Lambda architecture diagram

``` text
Incoming Events / Historical Data
                 |
          +------+------+
          |             |
     Batch Layer    Speed Layer
          |             |
Historical processing  Real-time processing
          |             |
          +------+------+
                 |
          Serving Layer
                 |
   Query combined analytical results
                 |
      Dashboard / API / Application
```

### Example: Video-streaming platform

Suppose a streaming service wants to calculate the number of viewers
watching a video.

-   The batch layer analyzes historical viewing events to calculate
    daily and monthly statistics.
-   The speed layer processes incoming viewing events to estimate
    current activity.
-   The serving layer provides historical and recent metrics to
    dashboards.

The system may not require a traditional EDW for every part of the
video-streaming service. It may use event logs, stream processors, data
lakes, analytical stores, or a warehouse depending on the requirements.

### Advantages of Lambda

-   Supports historical and near-real-time analytics.
-   Can provide comprehensive batch results and low-latency updates.
-   Lets each processing path use an appropriate technology.

### Disadvantages of Lambda

-   Often requires two separate processing pipelines.
-   The batch and speed paths can implement the same business logic
    twice.
-   Results from the two paths must be reconciled.
-   Operations, testing, and maintenance can become more complicated.

**Interview answer:** Lambda Architecture uses a batch layer for
comprehensive historical processing, a speed layer for low-latency
updates, and a serving layer to expose their results.

------------------------------------------------------------------------

## 8.2 Kappa Architecture

Kappa Architecture simplifies the design by using a streaming-processing
pipeline for event data instead of maintaining separate batch and speed
pipelines.

The central idea is that events are written to an ordered, replayable
log. New events are processed continuously, and historical events can be
replayed through the same processing logic when needed.

Apache Kafka is a common technology for implementing the event-log
portion.

### How does it work?

1.  Producers generate events.
2.  Events are appended to a durable event log.
3.  A stream-processing application reads the events.
4.  The application updates an analytical store or serving database.
5.  If the processing logic needs to change, retained events can be
    replayed or reprocessed.

### Kappa architecture diagram

``` text
Event Producers
(Orders · Payments · User activity)
                 |
                 v
             Event Log
     (Kafka or another durable log)
                 |
                 v
       Stream Processing Pipeline
       (Continuous processing/replay)
                 |
                 v
        Serving Store / Analytics
       (Dashboards · Apps · Reports)
```

### Example: Online order tracking

An e-commerce platform receives these events:

``` text
OrderCreated
PaymentCompleted
OrderShipped
OrderDelivered
```

A stream processor reads them in the required order and updates the
order's current status.

If the processing logic changes, retained events may be replayed to
rebuild the derived state, provided the log retains the necessary
history and the pipeline correctly handles duplicates and ordering.

### Advantages of Kappa

-   One main streaming-processing code path.
-   Less duplication of business logic.
-   Can support both live processing and replay-based reprocessing.
-   Useful for event-driven architectures.

### Disadvantages of Kappa

-   Requires reliable event retention and replay.
-   Reprocessing very large historical datasets can be expensive or
    slow.
-   Event ordering, duplicates, late events, and state management need
    careful handling.
-   Not every batch-oriented workload naturally fits a streaming model.

### Lambda vs. Kappa

  -----------------------------------------------------------------------
  Feature                 Lambda                  Kappa
  ----------------------- ----------------------- -----------------------
  Processing paths        Separate batch and      One main streaming path
                          speed paths             

  Historical analysis     Batch processing        Replay or reprocessing
                                                  of retained events

  Real-time analysis      Speed layer             Streaming pipeline

  Complexity              Higher because paths    Less duplicate
                          must be reconciled      processing logic

  Best fit                Workloads needing       Event-driven workloads
                          distinct batch and      that fit replayable
                          streaming processing    stream processing
  -----------------------------------------------------------------------

**Remember:** Lambda uses two processing paths; Kappa emphasizes one
replayable streaming path.

Neither architecture is universally better. The right choice depends on
latency, replay requirements, data volume, and the processing algorithms
involved.

------------------------------------------------------------------------

## 9. Data Lakehouse --- how it fits into modern architecture

A lakehouse can store raw data and curated analytical tables on a shared
storage foundation.

For example, a platform might use:

-   **Amazon S3** for object storage.
-   **Apache Iceberg or Delta Lake** for table management.
-   **Spark** for data processing.
-   **Athena or another compatible query engine** for analytical
    queries.

A lakehouse can support both batch and streaming ingestion. It does not
automatically replace every streaming processor or warehouse; its
capabilities depend on the implementation and query engine.

### Lambda vs. Kappa vs. Lakehouse

  -----------------------------------------------------------------------
  Concept                             Main question it addresses
  ----------------------------------- -----------------------------------
  Lambda Architecture                 How do we combine batch and
                                      low-latency stream processing?

  Kappa Architecture                  How do we process live and replayed
                                      events through one main streaming
                                      pipeline?

  Data Lakehouse                      How can we combine flexible lake
                                      storage with reliable analytical
                                      tables and querying?
  -----------------------------------------------------------------------

These are complementary concepts rather than three competing storage
systems.

------------------------------------------------------------------------

## Connecting the architectures to the Olist project

Your Olist project is a historical, batch-oriented data pipeline. Use it
to explain the concepts without claiming that you implemented
architectures that were not part of the project.

  -----------------------------------------------------------------------
  Architecture                        Relationship to the Olist project
  ----------------------------------- -----------------------------------
  Lambda                              Could be a future extension if the
                                      platform needed both comprehensive
                                      historical batch processing and a
                                      separate low-latency
                                      event-processing path. The current
                                      historical CSV workflow is not a
                                      Lambda implementation.

  Kappa                               Could be a future extension if
                                      order and other business changes
                                      arrived as retained events and were
                                      continuously processed with replay
                                      support. The current CSV batch
                                      workflow is not a Kappa
                                      implementation.

  Data Lakehouse                      The project has data-lake-style
                                      storage and analytical querying. Do
                                      not automatically call it a full
                                      lakehouse unless the
                                      table-management and transaction
                                      capabilities used are confirmed.
  -----------------------------------------------------------------------

**Accurate interview statement:**

"My Olist project is a batch-oriented data engineering pipeline. I
cleaned and transformed historical CSV datasets into Parquet Silver and
Gold outputs, uploaded six Gold datasets to S3, registered metadata in
Glue Data Catalog, and queried the data using Athena. Lambda and Kappa
are architectural patterns I understand as possible extensions for
event-driven, low-latency processing. I would describe my current
implementation as data-lake-style analytics rather than claim that I
built a complete Lambda, Kappa, or transactional lakehouse platform."

------------------------------------------------------------------------

## Modern architecture --- quick interview questions

**Q1. What are the three components of Lambda Architecture?**

The batch layer processes historical data, the speed layer processes
recent events with low latency, and the serving layer exposes or
combines their results.

**Q2. What is the cold path in Lambda Architecture?**

The batch path, which periodically processes large volumes of historical
data.

**Q3. What is the hot path in Lambda Architecture?**

The speed path, which processes incoming events with low latency.

**Q4. What is the central idea of Kappa Architecture?**

Use one main streaming-processing pipeline and replay retained events
when historical results need to be rebuilt or processing logic changes.

**Q5. What role can Apache Kafka play in Kappa Architecture?**

Kafka can provide the durable event log from which stream-processing
applications consume and, subject to retention and design, replay
events.

**Q6. What is the main difference between Lambda and Kappa?**

Lambda maintains separate batch and speed paths. Kappa emphasizes one
replayable streaming path.

**Q7. Does a lakehouse replace stream processing?**

No. A lakehouse is a storage and analytical platform approach. It can
receive batch or streaming data and can work alongside stream-processing
systems.

**Q8. Are Lambda, Kappa, and lakehouse competing architectures?**

No. Lambda and Kappa describe data-processing approaches; a lakehouse
describes a storage and analytical platform approach. They address
different architectural questions and can be complementary.

------------------------------------------------------------------------

## Architecture selection checklist

Before choosing an architecture, ask:

-   Do consumers need results within seconds, minutes, hours, or days?
-   Is the source naturally a stream of events, periodic files, or
    database extracts?
-   Must the platform recompute years of history?
-   How long must events be retained for replay?
-   Are separate batch and streaming paths justified?
-   What are the requirements for event ordering, duplicates, late
    events, and state?
-   What storage, table-management, query, and governance capabilities
    are required?

**Memory aid:**

-   **Lambda:** two processing paths --- batch + speed.
-   **Kappa:** one streaming-first path + replay.
-   **Lakehouse:** flexible lake storage + managed analytical tables and
    querying.
