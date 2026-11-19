# Phase 4: PySpark (12 lessons)

> HCLTech Cloud Data Engineer prep. Built from the syllabus (Programming Skills → Python & PySpark).
> Read top to bottom once. Each lesson: **concept → code → line-by-line explanation → what to remember**.
> Every code block is followed by an **Explanation** so nothing is left unexplained.

| # | Lesson |
|---|--------|
| 1 | Introduction to PySpark |
| 2 | Spark Architecture |
| 3 | Spark Setup and Environment |
| 4 | Running PySpark Apps and Spark UI |
| 5 | RDD Basics |
| 6 | RDD Operations |
| 7 | DataFrames: Creation |
| 8 | DataFrames: Transformations |
| 9 | DataFrames: Joins and Aggregations |
| 10 | Spark SQL (Queries and Views) |
| 11 | Window Functions |
| 12 | File Handling and Data Lake Integration |

---

## Lesson 1: Introduction to PySpark

**Apache Spark** is a distributed, in-memory engine for processing big data across many machines. **PySpark** is the Python API for Spark.

**Why Spark when Pandas exists?**
| Pandas | Spark |
|---|---|
| single machine, data must fit in RAM | cluster of machines, data of any size |
| good up to a few GB | good from GB to PB |
| runs immediately | lazy, distributed |

**Why Spark over Hadoop MapReduce?** Spark keeps data in memory (up to ~100x faster), has easier APIs, and one engine for batch, SQL, streaming and ML.

**Spark components:** Spark Core (RDD), Spark SQL / DataFrames, Structured Streaming, MLlib, GraphX.

**Remember:** Spark is written in Scala. PySpark lets you write Spark in Python. Data is split into **partitions** and processed in parallel.

---

## Lesson 2: Spark Architecture

```
          ┌────────────────────┐
          │   Driver Program   │  runs your code, holds SparkSession,
          │   (SparkContext)   │  builds the plan, schedules tasks
          └─────────┬──────────┘
                    │  asks for resources
          ┌─────────▼──────────┐
          │  Cluster Manager   │  YARN / Kubernetes / Standalone / Mesos
          └─────────┬──────────┘
        ┌───────────┼───────────┐
   ┌────▼────┐ ┌────▼────┐ ┌────▼────┐
   │Executor │ │Executor │ │Executor │   run tasks, store data in memory
   │ tasks   │ │ tasks   │ │ tasks   │   on worker nodes
   └─────────┘ └─────────┘ └─────────┘
```

**Explanation of the diagram (read from top to bottom)**
- **Driver Program** (top box): your Python program starts here. It creates the SparkSession, converts your code into a plan, and splits that plan into small tasks.
- The Driver **asks the Cluster Manager for resources** (arrow down).
- **Cluster Manager** (middle box): decides which machines give how much CPU and memory. Examples: YARN, Kubernetes, Standalone.
- It starts **Executors** (bottom boxes) on the worker machines.
- **Executors** run the tasks the driver sends them, keep data in memory for reuse, and send results back to the driver.
- Simple analogy: Driver = project manager, Cluster Manager = HR giving staff, Executors = workers doing the job.

| Component | Role |
|---|---|
| **Driver** | the "brain": runs `main()`, creates SparkSession, converts code into a plan (DAG), schedules tasks, collects results |
| **Executor** | the "workers": JVM processes on worker nodes that run tasks and cache data |
| **Cluster Manager** | allocates resources (CPU, memory) to the application |
| **Worker node** | machine that hosts executors |

### How a job runs (key vocabulary)
- **Application** → contains one or more **Jobs**.
- **Job** = created by each **action**.
- **Stage** = a set of tasks that can run without moving data between machines. Stages are split at **shuffle** boundaries.
- **Task** = the smallest unit of work, one task per partition.
- **DAG** (Directed Acyclic Graph) = the plan of transformations.

### Lazy evaluation (most important Spark concept)
**Transformations** (`filter`, `select`, `join`) only build a plan and do **nothing** immediately. Only an **action** (`show`, `count`, `collect`, `write`) triggers execution. This lets Spark optimize the whole plan first.

### Narrow vs wide transformations
| Narrow | Wide |
|---|---|
| each output partition depends on **one** input partition | depends on **many** partitions |
| no shuffle, fast | **shuffle** (network data movement), expensive |
| `map`, `filter`, `select` | `groupBy`, `join`, `distinct`, `orderBy`, `reduceByKey` |

### Deploy modes
- **Client mode:** driver runs on your machine (good for development).
- **Cluster mode:** driver runs inside the cluster (good for production).

---

## Lesson 3: Spark Setup and Environment

**Local setup (quickest)**
```bash
pip install pyspark
# needs Java installed (JDK 8/11/17)
```

**Explanation**
- `pip install pyspark` downloads and installs PySpark (it includes Spark itself) from the Python package index.
- The comment says Spark runs on the **JVM** (Java Virtual Machine), so a Java JDK (version 8, 11 or 17) must be installed first. Check with `java -version`.

Alternatives: Databricks Community Edition, Google Colab, AWS EMR / Glue, Azure Databricks.

**Creating a SparkSession (the entry point for everything)**
```python
from pyspark.sql import SparkSession

spark = (SparkSession.builder
         .appName("MyApp")
         .master("local[*]")                       # local, use all cores
         .config("spark.sql.shuffle.partitions", "8")
         .getOrCreate())

sc = spark.sparkContext      # lower-level entry point (for RDDs)
```

**Explanation**
- `from pyspark.sql import SparkSession`: imports the class used to start Spark.
- `SparkSession.builder`: starts a builder. The following dotted calls set options one at a time (this style is called method chaining; the outer brackets let it span several lines).
- `.appName("MyApp")`: a name for your application; it shows in the Spark UI.
- `.master("local[*]")`: where to run. `local[*]` means run on **your own machine** using **all CPU cores**. On a real cluster this would be `yarn` etc.
- `.config("spark.sql.shuffle.partitions", "8")`: sets a Spark setting. This one says: after a shuffle (join/groupBy) use 8 partitions instead of the default 200. For small local data, fewer is faster.
- `.getOrCreate()`: creates the session, or returns the existing one if already created. Result stored in `spark`.
- `sc = spark.sparkContext`: gets the lower-level **SparkContext**, which you need for RDD operations.

| `master` value | Meaning |
|---|---|
| `local` | 1 thread |
| `local[4]` | 4 threads |
| `local[*]` | all cores |
| `yarn` | Hadoop YARN cluster |
| `spark://host:7077` | standalone cluster |

**SparkSession vs SparkContext:** `SparkSession` (Spark 2.0+) is the unified entry point for DataFrames and SQL. `SparkContext` is the older entry point for RDDs, available as `spark.sparkContext`.

Stop when done: `spark.stop()` releases the resources.

---

## Lesson 4: Running PySpark Applications and Spark UI

### Ways to run
1. **Interactive:** `pyspark` shell or notebook (Databricks/Jupyter). `spark` is already created for you.
2. **Script with `spark-submit`:**
```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --num-executors 4 \
  --executor-memory 4g \
  --executor-cores 2 \
  my_job.py arg1 arg2
```

**Explanation**
- `spark-submit` is the command that sends a Python script to run on Spark. The `\` at the end of each line just continues the command on the next line.
- `--master yarn`: run on a YARN cluster.
- `--deploy-mode cluster`: the driver also runs **inside** the cluster (production style). `client` would run the driver on your machine.
- `--num-executors 4`: start 4 executors (workers).
- `--executor-memory 4g`: each executor gets 4 GB of memory.
- `--executor-cores 2`: each executor uses 2 CPU cores. So total parallel tasks = 4 × 2 = 8.
- `my_job.py`: your script.
- `arg1 arg2`: command-line arguments passed to your script (read them with `sys.argv`).

### A minimal application
```python
# my_job.py
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("demo").getOrCreate()
df = spark.read.csv("input.csv", header=True, inferSchema=True)
df.filter(df.amount > 100).write.mode("overwrite").parquet("output/")
spark.stop()
```

**Explanation**
- Line 1 is a comment with the file name.
- Imports `SparkSession`.
- `SparkSession.builder.appName("demo").getOrCreate()` starts Spark with the app name `demo`.
- `spark.read.csv("input.csv", header=True, inferSchema=True)` reads a CSV file into a DataFrame. `header=True` uses the first row as column names; `inferSchema=True` makes Spark guess column types (numbers vs text) instead of treating all as strings.
- `df.filter(df.amount > 100)` keeps only rows where the `amount` column is greater than 100 (a transformation, lazy).
- `.write.mode("overwrite").parquet("output/")`: writes the result as Parquet files in folder `output/`, replacing anything already there. **This is the action** that actually triggers the whole job.
- `spark.stop()` shuts Spark down.

### Spark UI (default `http://localhost:4040`)
| Tab | What you see |
|---|---|
| **Jobs** | one entry per action, status, duration |
| **Stages** | tasks per stage, shuffle read/write, skew |
| **Storage** | cached RDDs/DataFrames |
| **Environment** | configs |
| **Executors** | memory, cores, tasks per executor |
| **SQL** | query plans for DataFrame/SQL |

**Use it to:** find slow stages, spot data skew (one task much slower than others), check shuffle size, confirm caching. The UI is available only while the application is running.

---

## Lesson 5: RDD Basics

**RDD = Resilient Distributed Dataset:** the original low-level Spark data structure.
- **Resilient:** rebuilds lost partitions using lineage (fault tolerant).
- **Distributed:** split into partitions across the cluster.
- **Immutable:** you never change it, you create a new one.

### Creating RDDs
```python
rdd1 = sc.parallelize([1, 2, 3, 4, 5], numSlices=2)   # from a Python list
rdd2 = sc.textFile("data.txt")                         # from a file
rdd3 = rdd1.map(lambda x: x * 2)                       # from another RDD
```

**Explanation**
- `sc.parallelize([...], numSlices=2)`: takes a normal Python list and **distributes** it as an RDD. `numSlices=2` splits it into 2 partitions.
- `sc.textFile("data.txt")`: reads a text file; **each line becomes one element** of the RDD.
- `rdd1.map(lambda x: x * 2)`: creates a **new** RDD from `rdd1` by doubling every element. `rdd1` itself is unchanged (immutable). Nothing runs yet because `map` is lazy.

### Lineage
Spark remembers how each RDD was built (the chain of transformations). If a partition is lost, it **recomputes** only that partition from lineage instead of keeping copies. `rdd.toDebugString()` shows it.

### RDD vs DataFrame
| RDD | DataFrame |
|---|---|
| low-level, no schema | has schema (named columns) |
| no Catalyst optimizer | **optimized** by Catalyst + Tungsten |
| slower in Python | much faster |
| use for unstructured data / custom logic | **use by default** |

---

## Lesson 6: RDD Operations

### Transformations (lazy, return a new RDD)
```python
rdd = sc.parallelize([1, 2, 3, 4, 5, 6])

rdd.map(lambda x: x * 2)             # one output per input
rdd.filter(lambda x: x % 2 == 0)     # keep matching
rdd.flatMap(lambda x: [x, x * 10])   # one input -> many outputs, flattened
rdd.distinct()
rdd.union(other); rdd.intersection(other)
```

**Explanation**
- `rdd = sc.parallelize([1..6])` creates the base RDD.
- `map(lambda x: x * 2)`: each element is transformed into **exactly one** new element → 2,4,6,8,10,12.
- `filter(lambda x: x % 2 == 0)`: keeps only elements where the lambda returns True (even numbers) → 2,4,6.
- `flatMap(lambda x: [x, x * 10])`: each element produces a **list** of results, and Spark **flattens** them into one RDD. 1 becomes 1,10; 2 becomes 2,20; ... giving 1,10,2,20,3,30... (with `map` you would get a list of lists instead).
- `distinct()`: removes duplicates (a shuffle operation).
- `union(other)`: combines two RDDs; `intersection(other)`: keeps only elements present in both.
- None of these run yet. They only build the plan.

**Pair RDDs (key, value)**
```python
pairs = sc.parallelize([("a", 1), ("b", 2), ("a", 3)])
pairs.reduceByKey(lambda x, y: x + y)    # [("a",4),("b",2)]  (combines before shuffle: efficient)
pairs.groupByKey()                        # avoid for big data: shuffles everything
pairs.sortByKey()
pairs.mapValues(lambda v: v * 10)
pairs.join(other_pairs)
```

**Explanation**
- A **pair RDD** holds `(key, value)` tuples. Special operations work on the key.
- `reduceByKey(lambda x, y: x + y)`: groups by key and combines the values using the function. For key `"a"` the values 1 and 3 are added → 4. Result: `("a",4)`, `("b",2)`. Spark first combines values **inside each partition**, then shuffles only the small partial results → efficient.
- `groupByKey()`: gathers **all** values of each key into one place (`"a"` → [1, 3]). It shuffles every value across the network, so avoid it on big data.
- `sortByKey()`: sorts by key.
- `mapValues(lambda v: v * 10)`: applies the function only to the values and keeps keys → `("a",10)`, `("b",20)`, `("a",30)`.
- `join(other_pairs)`: joins two pair RDDs on matching keys (like SQL inner join).
- **Prefer `reduceByKey` over `groupByKey`.**

### Actions (trigger execution, return a result)
```python
rdd.collect()        # ALL data to driver (dangerous for big data)
rdd.take(3)          # first 3
rdd.first()
rdd.count()
rdd.reduce(lambda a, b: a + b)
rdd.foreach(print)
rdd.saveAsTextFile("out/")
```

**Explanation**
- Actions **start the real computation** and return a result (or write output).
- `collect()`: brings **all** elements to the driver as a Python list. If the data is huge the driver runs out of memory, so use only on small results.
- `take(3)`: returns just the first 3 elements (safe).
- `first()`: returns the first element.
- `count()`: returns the number of elements.
- `reduce(lambda a, b: a + b)`: combines all elements into a single value using the function (here a sum).
- `foreach(print)`: runs a function on each element **on the executors** (so on a cluster the output appears in executor logs, not on your screen).
- `saveAsTextFile("out/")`: writes the RDD as text files into folder `out/`, one file per partition.

### Classic example: word count
```python
text = sc.textFile("data.txt")
counts = (text.flatMap(lambda line: line.split())
              .map(lambda w: (w, 1))
              .reduceByKey(lambda a, b: a + b))
counts.take(10)
```

**Explanation** (the "Hello World" of Spark; say the file has `hi spark hi`)
- `sc.textFile("data.txt")`: each line of the file is one element.
- `.flatMap(lambda line: line.split())`: splits each line into words and flattens everything into one RDD of words → `hi`, `spark`, `hi`.
- `.map(lambda w: (w, 1))`: turns each word into a pair `(word, 1)` → `(hi,1)`, `(spark,1)`, `(hi,1)`.
- `.reduceByKey(lambda a, b: a + b)`: adds the 1s for each identical word → `(hi,2)`, `(spark,1)`.
- `counts.take(10)`: **action**; fetches 10 results and triggers all the steps above.
- The surrounding brackets let the chain span multiple lines.

### Caching
```python
rdd.cache()                         # keep in memory
rdd.persist(StorageLevel.MEMORY_AND_DISK)
rdd.unpersist()
```

**Explanation**
- By default Spark **recomputes** an RDD every time an action uses it. Caching stores it so reuse is fast.
- `rdd.cache()`: keeps the data in memory after it is first computed.
- `rdd.persist(StorageLevel.MEMORY_AND_DISK)`: lets you choose where to store it; this level keeps it in memory and **spills to disk** if memory is full. (Needs `from pyspark import StorageLevel`.)
- `rdd.unpersist()`: removes it from the cache to free memory.
- Cache only when the same data is reused multiple times.

### Partitions
`rdd.getNumPartitions()` shows how many partitions. `repartition(n)` makes exactly `n` partitions with a full shuffle (can increase or decrease). `coalesce(n)` only **decreases** partitions and avoids a full shuffle (cheaper).

---

## Lesson 7: DataFrames: Creation

A **DataFrame** is a distributed table with named columns and a schema. It is the main API you will use.

### From Python data
```python
data = [("Ash", 22, "Noida"), ("Ravi", 25, "Delhi")]
df = spark.createDataFrame(data, ["name", "age", "city"])
df.show()
df.printSchema()
```

**Explanation**
- `data` is a list of tuples; each tuple is one row (name, age, city).
- `spark.createDataFrame(data, ["name", "age", "city"])` builds a DataFrame from it; the second argument gives the **column names**. Spark guesses the types (string, long, string).
- `df.show()` prints the table (first 20 rows) in the console. This is an action.
- `df.printSchema()` prints the column names, data types and whether they can be null, as a tree.

### With an explicit schema (recommended for production)
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("name", StringType(), True),    # True = nullable
    StructField("age", IntegerType(), True),
    StructField("city", StringType(), True),
])
df = spark.createDataFrame(data, schema)
```

**Explanation**
- The import brings in the classes used to describe a schema.
- `StructType([...])` is the whole schema: a list of columns.
- `StructField("name", StringType(), True)` describes **one column**: its name (`"name"`), type (`StringType()`), and `True` meaning **nullable** (it may contain nulls). Use `False` to forbid nulls.
- The same is done for `age` (`IntegerType`) and `city`.
- `spark.createDataFrame(data, schema)`: builds the DataFrame with exactly these types instead of guessing. Safer and faster, and the schema is under your control.

### From files
```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
df = spark.read.json("data.json")
df = spark.read.parquet("data.parquet")
```

**Explanation**
- `spark.read` is the reader object. Each format has its own method.
- CSV: `header=True` uses the first line as column names; `inferSchema=True` detects types.
- JSON: reads JSON lines and infers the schema automatically.
- Parquet: reads the columnar format; the schema is stored inside the file.
- `inferSchema` costs an extra pass over the data. Define the schema manually for big files.

### From RDD or Pandas
```python
df = rdd.toDF(["name", "age"])
df = spark.createDataFrame(pandas_df)
pandas_df = df.toPandas()     # only for small results!
```

**Explanation**
- `rdd.toDF(["name", "age"])`: converts an RDD of tuples to a DataFrame with those column names.
- `spark.createDataFrame(pandas_df)`: converts a Pandas DataFrame into a Spark DataFrame.
- `df.toPandas()`: brings **all** Spark data to the driver as a Pandas DataFrame. Use only when the result is small, otherwise the driver runs out of memory.

### Inspecting
`df.show(5)` (first 5 rows), `df.printSchema()`, `df.columns` (list of names), `df.dtypes` (names with types), `df.count()` (row count), `df.describe().show()` (summary statistics).

---

## Lesson 8: DataFrames: Transformations

```python
from pyspark.sql.functions import col, lit, when, upper, round as sround, concat_ws
```

**Explanation**
- `pyspark.sql.functions` holds the built-in column functions. We import the ones we need.
- `col("age")` refers to a column by name; `lit(5)` creates a constant column; `when` builds conditions; `upper` makes text uppercase; `concat_ws` joins columns with a separator.
- `round as sround` imports `round` under the name `sround` so it doesn't hide Python's own `round`.

| Task | Code |
|---|---|
| Select columns | `df.select("name", "age")` or `df.select(col("name"))` |
| Filter rows | `df.filter(col("age") > 21)` or `df.where("age > 21")` |
| Multiple conditions | `df.filter((col("age") > 21) & (col("city") == "Delhi"))` (use `&`, `\|`, `~` with parentheses) |
| Add / change column | `df.withColumn("age_plus", col("age") + 1)` |
| Constant column | `df.withColumn("country", lit("India"))` |
| Rename | `df.withColumnRenamed("name", "emp_name")` |
| Drop column | `df.drop("city")` |
| Distinct | `df.distinct()`, `df.dropDuplicates(["name"])` |
| Sort | `df.orderBy(col("age").desc())` |
| Limit | `df.limit(10)` |
| Cast type | `df.withColumn("age", col("age").cast("int"))` |

**How to read the table**
- `select`: pick only the columns you want (like SQL SELECT).
- `filter` / `where`: keep rows meeting a condition (like SQL WHERE); both are the same.
- Multiple conditions: use `&` (and), `|` (or), `~` (not), and wrap **each** condition in brackets because of Python operator precedence. Do **not** use `and`/`or`.
- `withColumn(name, expression)`: adds a new column or **replaces** one with the same name.
- `lit("India")`: needed to put a fixed value in a column.
- `withColumnRenamed(old, new)`: renames a column.
- `drop`: removes a column.
- `distinct()`: removes fully duplicate rows; `dropDuplicates(["name"])` removes duplicates based only on the listed columns.
- `orderBy(col("age").desc())`: sort descending.
- `limit(10)`: keep only 10 rows.
- `cast("int")`: converts the column type.

### Conditional logic: `when / otherwise` (like SQL CASE)
```python
df = df.withColumn(
    "category",
    when(col("age") < 18, "Minor")
    .when(col("age") < 60, "Adult")
    .otherwise("Senior")
)
```

**Explanation**
- `df.withColumn("category", ...)` adds a new column called `category`. The assignment `df = ...` is needed because DataFrames are immutable (the transformation returns a new one).
- `when(col("age") < 18, "Minor")`: if age is below 18, the value is `"Minor"`.
- `.when(col("age") < 60, "Adult")`: otherwise, if age is below 60, `"Adult"`. Conditions are checked in order, so this only applies to ages 18 and above.
- `.otherwise("Senior")`: everything else (60 and above). Without `otherwise`, unmatched rows get null.
- Equivalent SQL: `CASE WHEN age<18 THEN 'Minor' WHEN age<60 THEN 'Adult' ELSE 'Senior' END`.

### Handling nulls
```python
df.dropna()                       # drop rows with any null
df.dropna(subset=["age"])
df.fillna({"age": 0, "city": "NA"})
df.filter(col("age").isNull())
```

**Explanation**
- `dropna()`: removes every row that has a null in any column.
- `dropna(subset=["age"])`: removes rows only where `age` is null.
- `fillna({"age": 0, "city": "NA"})`: replaces nulls using a dictionary of column → value: null ages become 0, null cities become `"NA"`.
- `filter(col("age").isNull())`: finds rows where age **is** null (`isNotNull()` for the opposite). Never use `== None`.

### String and date functions
```python
from pyspark.sql.functions import trim, lower, split, to_date, current_date, datediff, year
df.withColumn("city", upper(trim(col("city"))))
df.withColumn("dt", to_date(col("date_str"), "yyyy-MM-dd"))
```

**Explanation**
- The import lists useful functions: `trim` (remove spaces), `lower`, `split` (string to array), `to_date` (text to date), `current_date`, `datediff` (days between dates), `year` (extract year).
- Line 2: `trim(col("city"))` removes leading/trailing spaces, then `upper(...)` makes it uppercase; `withColumn("city", ...)` overwrites the same column with the cleaned values. Functions can be nested.
- Line 3: `to_date(col("date_str"), "yyyy-MM-dd")` converts text like `"2026-01-15"` into a real date type; the second argument is the format of the text. Stored in new column `dt`.

### Custom logic: UDF (use sparingly)
```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

@udf(StringType())
def grade(m): return "P" if m >= 40 else "F"
```

**Explanation**
- A **UDF** (user-defined function) lets you run your own Python function on a column.
- `@udf(StringType())` is a decorator: it registers the function below as a UDF and declares it **returns a string**.
- `def grade(m): return "P" if m >= 40 else "F"`: takes marks `m` and returns `"P"` (pass) if 40 or more, else `"F"`. Used like `df.withColumn("result", grade(col("marks")))`.
- UDFs are slow (data moves between the JVM and Python and Spark can't optimize them). **Prefer built-in functions** such as `when/otherwise` whenever possible.

### Transformations vs actions (DataFrame)
- Transformations (lazy): `select, filter, withColumn, groupBy, join, orderBy`
- Actions: `show, count, collect, take, write`

---

## Lesson 9: DataFrames: Joins and Aggregations

### Aggregations
```python
from pyspark.sql.functions import count, sum as _sum, avg, min as _min, max as _max, countDistinct

df.groupBy("city").agg(
    count("*").alias("emp_count"),
    _sum("salary").alias("total_salary"),
    avg("salary").alias("avg_salary"),
    _max("salary").alias("max_salary")
)

df.groupBy("city", "dept").count()
df.agg(_sum("salary"))                       # whole-table aggregate
```

**Explanation**
- The import brings aggregate functions. `sum`, `min`, `max` are renamed with `_` (`_sum`, ...) so they don't override Python's built-in `sum/min/max`.
- `df.groupBy("city")`: splits rows into groups by city (like SQL GROUP BY).
- `.agg(...)`: calculates several aggregates for each group at once:
  - `count("*").alias("emp_count")`: number of rows per city, named `emp_count`.
  - `_sum("salary").alias("total_salary")`: total salary per city.
  - `avg("salary").alias("avg_salary")`: average salary.
  - `_max("salary").alias("max_salary")`: highest salary.
- `.alias(name)` gives the result column a readable name (like SQL `AS`).
- `df.groupBy("city", "dept").count()`: groups by **two** columns and counts rows per combination.
- `df.agg(_sum("salary"))`: no groupBy, so it aggregates the **whole** table into one row.

**HAVING equivalent:** filter after aggregation.
```python
df.groupBy("city").agg(_sum("salary").alias("total")).filter(col("total") > 100000)
```

**Explanation**
- First group by city and compute total salary as `total`.
- Then `.filter(col("total") > 100000)` keeps only cities whose total is above 100000. This is what SQL `HAVING` does (a filter applied after grouping).

### Joins
```python
orders.join(customers, on="cust_id", how="inner")
orders.join(customers, orders.cust_id == customers.id, "left")
```

**Explanation**
- `orders.join(customers, on="cust_id", how="inner")`: joins when both tables share a column named `cust_id`. `how="inner"` keeps only matching rows. Using `on="cust_id"` keeps **one** `cust_id` column in the result.
- `orders.join(customers, orders.cust_id == customers.id, "left")`: when the key columns have **different names**, give a condition. `"left"` keeps all rows of `orders` and fills customer columns with null where no match exists. Here both key columns stay in the result.

| `how` | Result |
|---|---|
| `inner` | only matching rows (default) |
| `left` / `left_outer` | all left rows + matches |
| `right` / `right_outer` | all right rows + matches |
| `outer` / `full` | all rows from both |
| `left_semi` | left rows that **have** a match (no right columns) |
| `left_anti` | left rows that have **no** match |
| `cross` | every combination (Cartesian product) |

**Join tips (important for performance)**
- Joins cause a **shuffle** (expensive).
- **Broadcast join:** if one table is small, send a copy to all executors, which avoids shuffling the big table.
```python
from pyspark.sql.functions import broadcast
big.join(broadcast(small), "id")
```

**Explanation**
- `broadcast(small)` is a **hint** telling Spark to copy the small DataFrame to every executor.
- Each executor can then join its part of `big` locally with its own copy of `small`, so the large table is not moved over the network. This is much faster when `small` fits in memory.
- `"id"` is the join column present in both tables.

- Avoid duplicate column names after a join (join on a column name or alias the tables).
- Watch for **data skew** (one key has far more rows than others).

### Union
`df1.union(df2)` stacks rows of two DataFrames matching by column **position**. `df1.unionByName(df2)` matches by column **name** (safer). `union` does not remove duplicates (add `.distinct()` if needed).

---

## Lesson 10: Spark SQL (Queries and Views)

Spark SQL lets you query DataFrames using plain SQL. It uses the same engine, so SQL and DataFrame code have the same performance.

### Temporary views
```python
df.createOrReplaceTempView("employees")       # visible in this session only
df.createOrReplaceGlobalTempView("emp_g")     # across sessions, query as global_temp.emp_g

result = spark.sql("""
    SELECT city, COUNT(*) AS cnt, AVG(salary) AS avg_sal
    FROM employees
    WHERE age > 21
    GROUP BY city
    HAVING COUNT(*) > 1
    ORDER BY avg_sal DESC
""")
result.show()
```

**Explanation**
- `df.createOrReplaceTempView("employees")` registers the DataFrame under the name `employees` so SQL can use it like a table. It is not a copy of the data; it is just a name. It exists **only in this session**; "OrReplace" means it overwrites an older view with the same name.
- `createOrReplaceGlobalTempView("emp_g")`: a view shared across sessions in the same application. You must query it with the prefix `global_temp.emp_g`.
- `spark.sql("""...""")` runs an SQL query. Triple quotes allow the query to span several lines.
- The SQL reads: from `employees`, keep rows where `age > 21`, group by `city`, compute row count (`cnt`) and average salary (`avg_sal`), keep only groups with more than 1 row (`HAVING`), sort by average salary highest first.
- `result` is a **DataFrame**; `result.show()` prints it (action).

### Everything from your SQL phase works
Joins, subqueries, CTEs, CASE WHEN, aggregate functions:
```python
spark.sql("""
  WITH dept_avg AS (
      SELECT dept, AVG(salary) AS avg_sal FROM employees GROUP BY dept
  )
  SELECT e.*, d.avg_sal
  FROM employees e JOIN dept_avg d ON e.dept = d.dept
  WHERE e.salary > d.avg_sal
""")
```

**Explanation**
- `WITH dept_avg AS (...)` defines a **CTE** (a temporary named result): the average salary of each department.
- The main query joins `employees` (alias `e`) with `dept_avg` (alias `d`) on department.
- `SELECT e.*, d.avg_sal` returns all employee columns plus their department's average salary.
- `WHERE e.salary > d.avg_sal` keeps only employees who earn **more than their department's average**.

### Tables vs views
| | Temp view | Table (`saveAsTable`) |
|---|---|---|
| stored data? | no, just a name for a query | yes, in warehouse/metastore |
| lifetime | session | permanent |

### When to use which
- SQL: familiar, great for analysts, complex logic readable as one query.
- DataFrame API: easier to build step by step, programmatic, testable.
- Mix both freely.

---

## Lesson 11: Window Functions

Window functions calculate a value for each row **using a group of related rows**, **without collapsing rows** (unlike `groupBy`, which gives one row per group).

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, rank, dense_rank, lag, lead, sum as _sum

w = Window.partitionBy("dept").orderBy(col("salary").desc())
```

**Explanation**
- `Window` is used to define the "window" (the group of rows to look at).
- The imported functions are the ranking functions (`row_number`, `rank`, `dense_rank`) and value functions (`lag`, `lead`).
- `Window.partitionBy("dept")`: splits rows into groups by department (like GROUP BY, but rows are kept).
- `.orderBy(col("salary").desc())`: orders rows **inside each department** by salary, highest first.
- `w` is just the window **definition**; it is applied with `.over(w)`.

### Ranking functions
```python
df.withColumn("rn",  row_number().over(w))   # 1,2,3,4  (always unique)
df.withColumn("rnk", rank().over(w))         # 1,2,2,4  (ties share rank, gap after)
df.withColumn("drnk", dense_rank().over(w))  # 1,2,2,3  (ties share rank, no gap)
```

**Explanation**
- Each line adds a column by calling a ranking function **over** the window `w` (rank within each department by salary).
- `row_number()`: gives 1,2,3,4 in order; ties still get different numbers (arbitrary among ties).
- `rank()`: tied rows get the **same** rank, and the next rank **skips** numbers (1,2,2,**4**).
- `dense_rank()`: tied rows get the same rank and the next rank **does not skip** (1,2,2,**3**).

| Salary | row_number | rank | dense_rank |
|---|---|---|---|
| 90 | 1 | 1 | 1 |
| 80 | 2 | 2 | 2 |
| 80 | 3 | 2 | 2 |
| 70 | 4 | 4 | 3 |

### Value functions: previous and next row
```python
w2 = Window.partitionBy("emp").orderBy("month")
df.withColumn("prev_sales", lag("sales", 1).over(w2))
df.withColumn("next_sales", lead("sales", 1).over(w2))
```

**Explanation**
- `w2` groups rows by employee and orders them by month (time order inside each employee).
- `lag("sales", 1)`: gets the value of `sales` from **1 row before** the current row. The first row of each employee gets null.
- `lead("sales", 1)`: gets the value from **1 row after**. The last row gets null.
- Typical use: compare this month with last month (`sales - prev_sales`).

### Running totals and moving averages
```python
w3 = Window.partitionBy("dept").orderBy("date").rowsBetween(Window.unboundedPreceding, Window.currentRow)
df.withColumn("running_total", _sum("sales").over(w3))
```

**Explanation**
- `w3` groups by department and orders by date.
- `rowsBetween(Window.unboundedPreceding, Window.currentRow)` sets the **frame**: from the very first row of the group up to the current row.
- `_sum("sales").over(w3)` adds up `sales` across that frame, so each row shows the **cumulative (running) total** up to that date. For a moving average you would use a smaller frame like `rowsBetween(-2, 0)` (current row and two before).

### Classic interview pattern: Top N per group
```python
w = Window.partitionBy("dept").orderBy(col("salary").desc())
top2 = (df.withColumn("rn", row_number().over(w))
          .filter(col("rn") <= 2)
          .drop("rn"))
```

**Explanation**
- Define the window: per department, highest salary first.
- `withColumn("rn", row_number().over(w))`: number the rows 1,2,3... within each department.
- `.filter(col("rn") <= 2)`: keep only the top 2 rows of every department.
- `.drop("rn")`: remove the helper column.
- Same in SQL: `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)`.

---

## Lesson 12: File Handling and Data Lake Integration

### Common file formats
| Format | Type | Schema | Compression | Best for |
|---|---|---|---|---|
| **CSV** | row, text | none (guess) | poor | exchange, small data |
| **JSON** | row, text | self-describing, nested | poor | APIs, semi-structured |
| **Parquet** | **columnar**, binary | embedded | excellent | **analytics, data lakes (default choice)** |

**Why Parquet?** Columnar means Spark reads only the columns it needs, compresses well, stores the schema, and supports predicate pushdown (filtering at file-read time). Typically much smaller and faster than CSV.

### Reading
```python
df = (spark.read
      .option("header", True)
      .option("inferSchema", True)
      .option("delimiter", ",")
      .csv("s3://bucket/raw/sales/"))

spark.read.json("path", multiLine=True)          # multiLine for pretty-printed JSON
spark.read.parquet("path")
spark.read.format("csv").load("path")            # generic form
```

**Explanation**
- `spark.read.option(key, value)`: sets reading options one by one.
  - `header=True`: first line holds column names.
  - `inferSchema=True`: detect column types.
  - `delimiter=","`: the separator between values (use `"|"` or `"\t"` for other files).
- `.csv("s3://bucket/raw/sales/")` reads from a **folder** in S3; Spark reads all files in it.
- `spark.read.json("path", multiLine=True)`: by default Spark expects one JSON object **per line**. `multiLine=True` allows a normal pretty-printed JSON file spread over many lines.
- `spark.read.parquet("path")`: reads Parquet.
- `spark.read.format("csv").load("path")`: the **generic** way; works for any format by changing the format name.

### Writing
```python
(df.write
   .mode("overwrite")                 # overwrite | append | ignore | error(default)
   .partitionBy("year", "month")      # folder per value, enables partition pruning
   .parquet("s3://bucket/curated/sales/"))

df.write.csv("out", header=True)
df.write.json("out")
df.write.saveAsTable("db.sales")
```

**Explanation**
- `df.write` is the writer object (an action when the final call runs).
- `.mode("overwrite")`: what to do if the output already exists (see table below).
- `.partitionBy("year", "month")`: creates **sub-folders** by column value, e.g. `year=2026/month=11/`. Later queries filtering on year/month only read the matching folders (**partition pruning**), which is much faster.
- `.parquet("s3://...")`: writes Parquet files to that location.
- `df.write.csv("out", header=True)`: writes CSV files into folder `out` with a header line.
- `df.write.json("out")`: writes JSON lines.
- `df.write.saveAsTable("db.sales")`: saves as a **managed table** named `sales` in database `db`, registered in the metastore so you can query it by name.

| Write mode | Behavior |
|---|---|
| `overwrite` | replace existing data |
| `append` | add to existing data |
| `ignore` | do nothing if data exists |
| `error` / `errorifexists` | fail if data exists (default) |

**Output is a folder of part files** (`part-0000...`), one per partition, not a single file. Use `coalesce(1)` for a single file only on small data.

### Corrupt / bad data handling (read modes)
`mode="PERMISSIVE"` (default, puts nulls in bad fields), `DROPMALFORMED` (skips bad rows), `FAILFAST` (stops with an error on the first bad row). Example: `spark.read.option("mode", "DROPMALFORMED").csv("path")`.

### Data lake integration
Spark reads and writes directly to cloud storage using a path prefix:

| Cloud | Path example |
|---|---|
| AWS S3 | `s3://bucket/path/` (EMR/Glue) or `s3a://` |
| Azure ADLS Gen2 | `abfss://container@account.dfs.core.windows.net/path` |
| Google Cloud Storage | `gs://bucket/path` |
| HDFS | `hdfs://namenode/path` |

**Typical layered data lake flow (links to your cloud phase)**
```
Raw (CSV/JSON as received)  ->  Curated (cleaned, Parquet)  ->  Trusted / Gold (aggregated, business-ready)
```

**Explanation**
- **Raw:** data exactly as it arrived, never modified (so you can always reprocess).
- **Curated:** cleaned, deduplicated, typed, saved as Parquet.
- **Trusted / Gold:** aggregated, business-ready tables used by reports and analysts.
- The arrows show data moving forward through Spark jobs.

### Mini end-to-end pipeline (capstone style)
```python
raw = spark.read.option("header", True).csv("raw/sales.csv")

clean = (raw
    .dropDuplicates(["order_id"])
    .withColumn("amount", col("amount").cast("double"))
    .withColumn("order_date", to_date("order_date", "yyyy-MM-dd"))
    .filter(col("amount").isNotNull())
    .fillna({"city": "Unknown"}))

daily = clean.groupBy("order_date").agg(_sum("amount").alias("revenue"))

(daily.write.mode("overwrite")
      .partitionBy("order_date")
      .parquet("curated/daily_revenue/"))
```

**Explanation (step by step)**
- **Extract:** `raw = spark.read.option("header", True).csv("raw/sales.csv")` reads the CSV; without `inferSchema` every column is a string.
- **Transform** (building `clean`):
  - `.dropDuplicates(["order_id"])`: keep one row per order id (removes duplicate orders).
  - `.withColumn("amount", col("amount").cast("double"))`: convert the text amount into a decimal number.
  - `.withColumn("order_date", to_date("order_date", "yyyy-MM-dd"))`: convert the text date into a real date.
  - `.filter(col("amount").isNotNull())`: drop rows whose amount is missing or could not be converted.
  - `.fillna({"city": "Unknown"})`: replace missing cities with `"Unknown"`.
- **Aggregate:** `daily = clean.groupBy("order_date").agg(_sum("amount").alias("revenue"))` gives total revenue for each day.
- **Load:** the last block writes `daily` to Parquet, replacing old output (`overwrite`), with a folder for each `order_date` (`partitionBy`). This write is the **action** that makes Spark run all previous steps.
- This **extract → transform → load (ETL)** shape is exactly what your capstone project needs.
- (Needs `col`, `to_date`, `_sum` imported from `pyspark.sql.functions` as shown in earlier lessons.)

### Performance cheat sheet
| Topic | Rule |
|---|---|
| Format | Use Parquet |
| Shuffle | Minimize wide operations; filter early |
| Joins | Broadcast small tables |
| Partitions | Tune `spark.sql.shuffle.partitions` (default 200); avoid too many tiny files |
| `repartition` vs `coalesce` | `repartition` = full shuffle; `coalesce` = reduce without shuffle |
| Cache | Only for reused data; `unpersist()` after |
| `collect()` | Avoid on big data; use `take()` / `limit()` |
| UDFs | Avoid; use built-ins |
| Skew | Watch Spark UI for slow tasks |

### Phase 4 revision checklist
- [ ] Driver / Executor / Cluster Manager roles
- [ ] Transformations (lazy) vs actions; narrow vs wide; stage = split at shuffle
- [ ] RDD vs DataFrame; `reduceByKey` vs `groupByKey`
- [ ] `repartition` vs `coalesce`; `cache` vs `persist`
- [ ] DataFrame: select, filter, withColumn, when/otherwise, groupBy+agg
- [ ] All join types, broadcast join
- [ ] `createOrReplaceTempView` + `spark.sql`
- [ ] `row_number` vs `rank` vs `dense_rank`, `lag`/`lead`, top-N per group
- [ ] CSV vs JSON vs Parquet, write modes, `partitionBy`
- [ ] Reading from S3 / ADLS / GCS

---

# Quick Interview Questions (PySpark)

1. What is lazy evaluation? → transformations build a plan; actions run it.
2. Narrow vs wide transformation? → shuffle or not.
3. RDD vs DataFrame? → DataFrame has schema + Catalyst optimizer.
4. `repartition` vs `coalesce`? → full shuffle vs no shuffle (only decrease).
5. `reduceByKey` vs `groupByKey`? → map-side combine, less shuffle.
6. What is a broadcast join? → ship small table to all executors.
7. `rank` vs `dense_rank` vs `row_number`? → gaps vs no gaps vs unique.
8. Why Parquet? → columnar, compressed, schema, fast analytics.
9. What are driver and executors? → coordinator vs workers.
10. How do you find a slow Spark job? → Spark UI: stages, skew, shuffle size.

---

*Next: Phase 5 (AWS). Glue, EMR and Athena use the PySpark and SQL you learned here.*
