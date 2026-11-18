# Phase 2 --- SQL Master Study Guide

## Textbook-style preparation for the HCLTech Cloud Data Engineer Hackathon

**SQL dialect:** MySQL 8.0\
**Goal:** Learn SQL concepts deeply, practise working queries, connect
them to data engineering, and prepare for interviews.

> **Syllabus note:** The available preparation-tracker screenshot showed
> Phase 2 (SQL) at 6/15, but did not expose every original lesson title.
> This is an expanded, consolidated 15-lesson study path based on the
> topics already discussed; it is not claimed to be a verbatim copy of
> an unseen original syllabus.

## How to study this guide

For every lesson: 1. Understand the concept and when to use it. 2. Type
and run each query; do not only read it. 3. Predict the output before
executing the query. 4. Modify the query and explain the change. 5.
Complete the assignment and answer exactly three assessment questions.

This guide uses MySQL 8.0. Other database systems may differ. In
particular, **MySQL does not support native `FULL OUTER JOIN` syntax**;
an emulation is included.

------------------------------------------------------------------------

# Practice database setup

Run this setup once in MySQL 8.0. It creates a small
insurance/e-commerce dataset used throughout the lessons.

``` sql
CREATE DATABASE IF NOT EXISTS sql_phase2;
USE sql_phase2;

CREATE TABLE departments (
  department_id INT PRIMARY KEY,
  department_name VARCHAR(50) NOT NULL
);

CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  employee_name VARCHAR(80) NOT NULL,
  manager_id INT NULL,
  department_id INT NULL,
  salary DECIMAL(10,2),
  hire_date DATE,
  city VARCHAR(50),
  FOREIGN KEY (manager_id) REFERENCES employees(employee_id),
  FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(100) NOT NULL,
  city VARCHAR(50),
  signup_date DATE
);

CREATE TABLE policies (
  policy_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  policy_type VARCHAR(40),
  premium DECIMAL(10,2),
  start_date DATE,
  end_date DATE,
  status VARCHAR(20),
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE claims (
  claim_id INT PRIMARY KEY,
  policy_id INT NOT NULL,
  claim_date DATE,
  claim_amount DECIMAL(12,2),
  claim_status VARCHAR(20),
  FOREIGN KEY (policy_id) REFERENCES policies(policy_id)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATE,
  order_status VARCHAR(20),
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
  order_item_id INT PRIMARY KEY,
  order_id INT NOT NULL,
  product_name VARCHAR(100),
  quantity INT,
  unit_price DECIMAL(10,2),
  FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

INSERT INTO departments VALUES
(10,'Engineering'),(20,'Analytics'),(30,'Operations');

INSERT INTO employees VALUES
(1,'Asha',NULL,10,90000,'2020-01-15','Delhi'),
(2,'Ravi',1,10,65000,'2021-03-10','Noida'),
(3,'Meera',1,20,72000,'2022-07-01','Delhi'),
(4,'Kabir',2,20,52000,'2023-02-12','Greater Noida'),
(5,'Nisha',2,30,48000,'2024-04-20','Noida');

INSERT INTO customers VALUES
(101,'Arjun','Delhi','2023-01-05'),
(102,'Diya','Noida','2023-02-10'),
(103,'Ishaan','Delhi','2023-05-15'),
(104,'Sara','Jaipur','2024-01-20'),
(105,'Vivaan','Noida','2024-03-11');

INSERT INTO policies VALUES
(1001,101,'Health',12000,'2024-01-01','2024-12-31','Active'),
(1002,102,'Motor',8500,'2024-02-01','2025-01-31','Active'),
(1003,101,'Life',20000,'2023-06-01','2033-05-31','Active'),
(1004,103,'Health',15000,'2023-07-01','2024-06-30','Expired'),
(1005,104,'Motor',9500,'2024-04-01','2025-03-31','Active');

INSERT INTO claims VALUES
(5001,1001,'2024-03-10',3000,'Approved'),
(5002,1001,'2024-08-12',2500,'Pending'),
(5003,1002,'2024-05-05',7000,'Rejected'),
(5004,1004,'2024-02-18',5000,'Approved');

INSERT INTO orders VALUES
(9001,101,'2024-01-10','Delivered'),
(9002,102,'2024-02-12','Delivered'),
(9003,101,'2024-03-15','Cancelled'),
(9004,104,'2024-04-02','Delivered');

INSERT INTO order_items VALUES
(1,9001,'Laptop',1,60000),
(2,9001,'Mouse',2,700),
(3,9002,'Keyboard',1,1500),
(4,9003,'Monitor',1,12000),
(5,9004,'Laptop',1,58000);
```

Check the setup:

``` sql
SHOW TABLES;
DESCRIBE employees;
SELECT * FROM customers;
SELECT * FROM policies;
```

**Understand table grain before querying:** one row in `customers`
represents a customer; one row in `policies` represents a policy; one
row in `claims` represents a claim; one row in `order_items` represents
an item within an order. A customer can have multiple policies, and a
policy can have multiple claims. Joins across one-to-many relationships
can multiply rows.

------------------------------------------------------------------------

# Lesson 1 --- SQL and Relational Database Fundamentals

## 1.1 What SQL is

SQL (Structured Query Language) is a declarative language for defining,
querying, manipulating, and controlling data in relational database
systems. A relational database stores data in tables made of rows and
columns. SQL is declarative because you describe the result you want;
the database engine chooses an execution strategy.

## 1.2 Core terminology

-   **Table/relation:** structured collection of records.
-   **Row/tuple:** one record.
-   **Column/attribute:** one property of a record.
-   **Primary key:** uniquely identifies each row; cannot be NULL.
-   **Foreign key:** references a key in another table (or the same
    table).
-   **Candidate key:** a minimal column set that uniquely identifies a
    row.
-   **Composite key:** key made from multiple columns.
-   **NULL:** unknown, missing, or inapplicable value. It is not zero or
    an empty string.
-   **Grain:** the business meaning of one row in a table.

Example: `customers.customer_id` is the primary key of customers.
`policies.customer_id` references it. One customer can have many
policies, so `customer_id` is not unique in policies.

## 1.3 SQL command categories

  Category                     Purpose          Commands
  ---------------------------- ---------------- -------------------------------
  DDL                          Define objects   CREATE, ALTER, DROP, TRUNCATE
  DML                          Change data      INSERT, UPDATE, DELETE
  DQL (common teaching term)   Retrieve data    SELECT
  DCL                          Permissions      GRANT, REVOKE
  TCL                          Transactions     COMMIT, ROLLBACK, SAVEPOINT

## 1.4 Relationships

-   One-to-one: one record in A relates to at most one in B.
-   One-to-many: one customer can own many policies.
-   Many-to-many: many students take many courses; represent this using
    a junction table.

## 1.5 Why it matters in data engineering

If you do not understand table grain and cardinality, a join may
silently inflate totals. For example, joining one policy to three claims
creates three output rows for that policy. Summing the policy premium
after that join counts the premium three times.

``` sql
SELECT policy_id, customer_id, policy_type, premium
FROM policies;
```

This is one row per policy, not one row per customer.

### Interview Q&A

**Q: Primary key versus foreign key?** A primary key identifies a row in
its own table. A foreign key references a key and enforces a
relationship.

**Q: What is grain?** The real-world entity or event represented by one
row.

**Q: Why is NULL not zero?** Zero is a known numeric value; NULL is
unknown or missing. Comparisons involving NULL can evaluate to UNKNOWN.

### Assignment

1.  Run DESCRIBE on every table.
2.  Write the grain and primary key of each table.
3.  Identify all foreign-key relationships.
4.  Explain why `customer_id` repeats in policies.

### Assessment --- answer all three

1.  What does one row in claims represent, and how does it relate to
    policies?
2.  Explain primary, foreign, and composite keys.
3.  Why can a join return more rows than either input table?

------------------------------------------------------------------------

# Lesson 2 --- SELECT, DISTINCT, Aliases, and Expressions

## 2.1 SELECT and FROM

``` sql
SELECT customer_name, city
FROM customers;
```

`SELECT` chooses columns or expressions; `FROM` specifies the source.
Prefer explicit column lists over `SELECT *` in production ETL because
they document the output schema and avoid reading unnecessary columns.

## 2.2 Aliases and expressions

``` sql
SELECT
  policy_id,
  premium,
  premium * 1.18 AS premium_including_tax
FROM policies;
```

An alias gives an output expression a readable name. `AS` is optional in
MySQL but improves clarity. A select-list alias is generally available
in ORDER BY, but not WHERE because WHERE is logically processed earlier.

## 2.3 DISTINCT

``` sql
SELECT DISTINCT city
FROM customers;
```

With multiple selected columns, DISTINCT removes duplicate combinations
of all selected values, not duplicates of just one column.

``` sql
SELECT DISTINCT city, signup_date
FROM customers;
```

## 2.4 Logical query processing

A useful conceptual order is:

`FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` →
`ORDER BY` → `LIMIT`

The optimizer may physically execute operations differently while
preserving the query's semantics.

### Worked examples

``` sql
SELECT DISTINCT policy_type FROM policies;

SELECT policy_id, premium, premium * 0.05 AS service_fee
FROM policies;

SELECT customer_id AS client_id, customer_name AS client_name
FROM customers;
```

### Data-engineering use

Explicit aliases help create stable curated schemas with
business-friendly names. Avoid depending on a source table's changing
column order.

### Interview Q&A

**Q: SELECT versus SELECT DISTINCT?** SELECT can return duplicates;
DISTINCT removes duplicate output rows based on all selected
expressions.

**Q: Can a SELECT alias be used in WHERE?** Generally no; use the
expression again or put the query in a derived table/CTE.

**Q: Why avoid SELECT \* in ETL?** It can pull unnecessary data and make
downstream schemas vulnerable to source changes.

### Assignment

1.  Display customer names in uppercase.
2.  Return unique policy types.
3.  Calculate premium plus an 8% surcharge.
4.  Return unique city/signup-date pairs.

### Assessment --- answer all three

1.  What does DISTINCT deduplicate when three columns are selected?
2.  Why is a SELECT alias generally unavailable in WHERE?
3.  Write a query returning policy ID and premium increased by 8%.

------------------------------------------------------------------------

# Lesson 3 --- Filtering, Sorting, and Limiting

## 3.1 WHERE and comparison operators

`WHERE` filters rows before grouping. Operators include `=`, `<>`, `!=`,
`>`, `<`, `>=`, and `<=`.

``` sql
SELECT *
FROM policies
WHERE status = 'Active' AND premium > 10000;
```

## 3.2 AND, OR, NOT

``` sql
SELECT *
FROM customers
WHERE city = 'Delhi'
   OR (city = 'Noida' AND signup_date >= '2024-01-01');
```

AND has higher precedence than OR. Parentheses make intended logic
explicit.

## 3.3 IN, BETWEEN, LIKE

``` sql
SELECT * FROM policies
WHERE policy_type IN ('Health', 'Life');

SELECT * FROM policies
WHERE premium BETWEEN 9000 AND 15000;

SELECT * FROM customers
WHERE customer_name LIKE 'A%';
```

BETWEEN includes both endpoints. In LIKE, `%` means zero or more
characters and `_` means exactly one character. Case sensitivity depends
on collation.

## 3.4 NULL filtering

Incorrect: `WHERE manager_id = NULL`

Correct:

``` sql
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

Ordinary comparisons with NULL evaluate to UNKNOWN, not TRUE.

## 3.5 ORDER BY and LIMIT

``` sql
SELECT policy_id, premium
FROM policies
ORDER BY premium DESC, policy_id ASC
LIMIT 3;
```

Without ORDER BY, result order is not guaranteed. Add a tie-breaker for
deterministic top-N output.

Pagination:

``` sql
SELECT * FROM customers
ORDER BY customer_id
LIMIT 10 OFFSET 20;
```

For large sequential extracts, keyset pagination (for example,
`WHERE customer_id > last_seen_id`) can be more efficient than large
offsets.

## 3.6 Data-engineering date filters

Use half-open ranges for timestamps:

``` sql
SELECT *
FROM claims
WHERE claim_date >= '2024-01-01'
  AND claim_date <  '2025-01-01';
```

This includes the start and excludes the next period boundary, avoiding
end-of-day precision problems.

### Interview Q&A

**Q: WHERE versus HAVING?** WHERE filters rows before aggregation;
HAVING filters groups after aggregation.

**Q: Why IS NULL?** NULL is not an ordinary value, so equality
comparisons do not test it correctly.

**Q: Is LIMIT deterministic without ORDER BY?** No.

### Assignment

1.  Find active policies with premium above 10,000.
2.  Find customers from Delhi or Jaipur.
3.  Find claims in the first half of 2024.
4.  Return the two highest-premium policies with a tie-breaker.

### Assessment --- answer all three

1.  Explain IN, BETWEEN, and LIKE.
2.  Find employees whose manager_id is NULL.
3.  Why should a top-five query include ORDER BY and a tie-breaker?

------------------------------------------------------------------------

# Lesson 4 --- String, Numeric, Date, and NULL Functions

Functions transform values. Scalar functions generally return one value
per input row.

## 4.1 String functions

``` sql
SELECT
  customer_name,
  UPPER(customer_name) AS upper_name,
  LOWER(customer_name) AS lower_name,
  LENGTH(customer_name) AS byte_length,
  CHAR_LENGTH(customer_name) AS character_length,
  TRIM(customer_name) AS trimmed_name,
  CONCAT(customer_name, ' - ', city) AS customer_label
FROM customers;
```

Other useful MySQL functions: SUBSTRING, LEFT, RIGHT, REPLACE, LOCATE.
LENGTH counts bytes; CHAR_LENGTH counts characters.

## 4.2 Numeric functions

``` sql
SELECT
  premium,
  ROUND(premium * 1.18, 2) AS rounded_total,
  CEIL(premium / 12) AS monthly_ceiling,
  FLOOR(premium / 12) AS monthly_floor,
  ABS(premium - 10000) AS difference
FROM policies;
```

Use DECIMAL for money rather than approximate floating-point values.
Agree rounding rules with the business and avoid unnecessary
intermediate rounding.

## 4.3 Date functions

``` sql
SELECT
  claim_date,
  YEAR(claim_date) AS claim_year,
  MONTH(claim_date) AS claim_month,
  DATE_ADD(claim_date, INTERVAL 30 DAY) AS follow_up_date,
  DATEDIFF(CURDATE(), claim_date) AS age_days
FROM claims;
```

Useful functions include CURDATE, NOW, DATE_FORMAT, LAST_DAY, and
TIMESTAMPDIFF. Be aware of the difference between DATE, DATETIME, and
TIMESTAMP and define timezone rules in pipelines.

## 4.4 NULL-handling and conversion

``` sql
SELECT COALESCE(city, 'Unknown') AS city_label
FROM employees;

SELECT IFNULL(city, 'Unknown') AS city_label
FROM employees;

SELECT CAST('2024-05-01' AS DATE);
SELECT CAST('1250.75' AS DECIMAL(10,2));
```

COALESCE accepts multiple expressions and returns the first non-NULL
value. IFNULL in MySQL accepts two arguments. NULLIF(a,b) returns NULL
if a equals b. Do not replace NULL with a fabricated business value
without an agreed rule.

### Data-engineering use

Standardise strings and derive clean fields while retaining raw source
values when auditability is important. A cleaned column should not erase
the original source meaning.

### Interview Q&A

**Q: LENGTH versus CHAR_LENGTH?** LENGTH counts bytes; CHAR_LENGTH
counts characters.

**Q: COALESCE versus IFNULL?** Both handle NULL; COALESCE accepts
multiple expressions and is standard SQL, while MySQL IFNULL accepts
two.

**Q: Why use half-open date ranges?** They avoid timestamp boundary and
fractional-second problems.

### Assignment

1.  Display trimmed uppercase customer names.
2.  Calculate monthly premium rounded to two decimals.
3.  Add 15 days to each claim date.
4.  Display NULL employee cities as Unknown.

### Assessment --- answer all three

1.  Write a COALESCE expression for a missing city.
2.  Explain DATEDIFF versus TIMESTAMPDIFF.
3.  Why preserve raw source values during cleansing?

------------------------------------------------------------------------

# Lesson 5 --- Aggregate Functions, GROUP BY, and HAVING

## 5.1 Aggregates

-   COUNT(\*) counts rows.
-   COUNT(column) counts non-NULL values.
-   SUM and AVG ignore NULL values.
-   MIN and MAX return minimum and maximum non-NULL values.

``` sql
SELECT
  COUNT(*) AS policy_count,
  SUM(premium) AS total_premium,
  AVG(premium) AS average_premium,
  MIN(premium) AS minimum_premium,
  MAX(premium) AS maximum_premium
FROM policies;
```

## 5.2 GROUP BY

``` sql
SELECT policy_type,
       COUNT(*) AS policy_count,
       SUM(premium) AS total_premium
FROM policies
GROUP BY policy_type;
```

GROUP BY creates one output group per distinct grouping combination. In
strict MySQL mode, selected expressions must be grouped or aggregated.

## 5.3 HAVING and conditional aggregation

``` sql
SELECT customer_id, COUNT(*) AS policy_count
FROM policies
GROUP BY customer_id
HAVING COUNT(*) >= 2;
```

``` sql
SELECT
  policy_type,
  COUNT(*) AS total_policies,
  SUM(CASE WHEN status = 'Active' THEN 1 ELSE 0 END) AS active_count,
  SUM(CASE WHEN status = 'Expired' THEN 1 ELSE 0 END) AS expired_count
FROM policies
GROUP BY policy_type;
```

Conditional aggregation calculates multiple conditional metrics in one
grouped query.

## 5.4 Common traps

Joining claims before summing policy premiums can multiply each policy
by its number of claims. Aggregate claims to one row per policy first,
then join. Also remember COUNT(\*) differs from COUNT(column).

### Interview Q&A

**Q: WHERE versus HAVING?** WHERE filters source rows; HAVING filters
grouped results.

\*\*Q: COUNT(\*) versus COUNT(column)?\*\* COUNT(\*) counts rows;
COUNT(column) ignores NULLs.

**Q: What is conditional aggregation?** Using CASE inside SUM or COUNT
to calculate condition-specific metrics.

### Assignment

1.  Calculate total premium by policy type.
2.  Count claims by status.
3.  Find customers with more than one policy.
4.  Count active and expired policies by type.

### Assessment --- answer all three

1.  Find policy types whose total premium exceeds 20,000.
2.  Explain why a join can inflate a SUM.
3.  Count approved and rejected claims in one query.

------------------------------------------------------------------------

# Lesson 6 --- DDL and DML: Creating and Modifying Data

## 6.1 DDL

``` sql
CREATE TABLE payment_records (
  payment_id INT PRIMARY KEY,
  policy_id INT NOT NULL,
  amount DECIMAL(12,2) NOT NULL,
  payment_date DATE
);

ALTER TABLE payment_records ADD payment_status VARCHAR(20);
ALTER TABLE payment_records MODIFY payment_status VARCHAR(30);
```

DROP removes the table definition and its data. Use it carefully.

## 6.2 INSERT

``` sql
INSERT INTO customers (customer_id, customer_name, city, signup_date)
VALUES (106, 'Anaya', 'Pune', '2024-05-01');

INSERT INTO departments (department_id, department_name)
VALUES (40, 'Finance'), (50, 'Compliance');
```

Specify columns explicitly so the statement is not dependent on table
column order.

## 6.3 UPDATE and DELETE

``` sql
UPDATE policies
SET status = 'Expired'
WHERE policy_id = 1005;

DELETE FROM claims
WHERE claim_id = 5004;
```

Preview target rows with SELECT before running UPDATE or DELETE. A
missing WHERE clause can change or delete every row.

## 6.4 INSERT ... SELECT

``` sql
CREATE TABLE active_policy_snapshot AS
SELECT policy_id, customer_id, premium
FROM policies
WHERE status = 'Active';
```

This creates a table from query output; do not assume it copies every
index, constraint, or default from the source.

## 6.5 DELETE versus TRUNCATE versus DROP

-   DELETE removes selected rows and supports WHERE.
-   TRUNCATE empties the table; behavior around transactions, triggers,
    and identity values is DBMS-specific.
-   DROP removes the table itself.

MySQL DDL commonly causes implicit commits, so do not assume every
schema change can be rolled back.

## 6.6 Transactions for safer changes

``` sql
START TRANSACTION;
UPDATE policies
SET premium = premium + 500
WHERE policy_id = 1001;

SELECT * FROM policies WHERE policy_id = 1001;
ROLLBACK; -- choose COMMIT instead only after validation
```

### Interview Q&A

**Q: DELETE versus TRUNCATE?** DELETE can filter rows; TRUNCATE empties
the table and has different DBMS-specific semantics.

**Q: Why list INSERT columns?** It makes the value-to-column mapping
explicit and resilient to schema changes.

**Q: What is INSERT ... SELECT?** It inserts rows returned by a query
into a target table.

### Assignment

1.  Create a payments table with key, amount, and payment date.
2.  Add payment_mode.
3.  Insert two records.
4.  Update a test row inside a transaction and roll it back.

### Assessment --- answer all three

1.  Explain DELETE, TRUNCATE, and DROP.
2.  Copy active policies into a snapshot table using INSERT ... SELECT
    or CREATE TABLE AS SELECT.
3.  Why preview rows before UPDATE or DELETE?

------------------------------------------------------------------------

# Lesson 7 --- Joins I: INNER JOIN and LEFT JOIN

A join combines related rows. Relational databases separate entities
into tables, so joins are fundamental to reporting and ETL.

## 7.1 INNER JOIN

Returns only matching pairs:

``` sql
SELECT c.customer_id, c.customer_name, p.policy_id, p.policy_type
FROM customers c
INNER JOIN policies p
  ON c.customer_id = p.customer_id;
```

Customers without policies are not returned.

## 7.2 LEFT JOIN

Returns all left-side rows and matching right-side rows. When there is
no match, right-side columns are NULL.

``` sql
SELECT c.customer_id, c.customer_name, p.policy_id
FROM customers c
LEFT JOIN policies p
  ON c.customer_id = p.customer_id;
```

Find customers without policies:

``` sql
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN policies p
  ON c.customer_id = p.customer_id
WHERE p.policy_id IS NULL;
```

## 7.3 ON versus WHERE in outer joins

``` sql
-- Preserve all customers; only active policies can match
SELECT c.customer_name, p.policy_id
FROM customers c
LEFT JOIN policies p
  ON c.customer_id = p.customer_id
 AND p.status = 'Active';
```

``` sql
-- Filters after joining; customers without an active match disappear
SELECT c.customer_name, p.policy_id
FROM customers c
LEFT JOIN policies p
  ON c.customer_id = p.customer_id
WHERE p.status = 'Active';
```

A WHERE condition on the nullable right-side table can effectively
remove unmatched rows.

## 7.4 Cardinality and fan-out

If a customer owns three policies, that customer appears three times in
a customer-to-policy join. This is expected. Do not add DISTINCT
blindly; decide the intended output grain first.

### Data-engineering checklist

Before a join, check uniqueness of the expected one-side key, null keys,
unmatched records, row counts before and after, and whether the join can
multiply measures.

### Interview Q&A

**Q: INNER versus LEFT JOIN?** INNER retains matching pairs; LEFT
preserves every left row plus any matches.

**Q: How can LEFT JOIN behave like INNER JOIN?** A WHERE filter on the
right table can remove NULL-extended rows.

**Q: How find rows without a match?** LEFT JOIN plus right-side key IS
NULL, or NOT EXISTS.

### Assignment

1.  Display every customer with policy IDs.
2.  Display customers who have policies.
3.  Find customers with no policies.
4.  Show claims with customer name and policy type.

### Assessment --- answer all three

1.  Explain INNER JOIN versus LEFT JOIN with an example.
2.  Why can a WHERE predicate change a LEFT JOIN result?
3.  Write a query preserving every customer, including those with no
    policy.

------------------------------------------------------------------------

# Lesson 8 --- Joins II: RIGHT, FULL OUTER Emulation, and SELF JOIN

## 8.1 RIGHT JOIN

RIGHT JOIN preserves all rows from the right table:

``` sql
SELECT c.customer_name, p.policy_id
FROM customers c
RIGHT JOIN policies p
  ON c.customer_id = p.customer_id;
```

It can usually be rewritten by reversing the table order and using LEFT
JOIN.

## 8.2 FULL OUTER JOIN in MySQL

A full outer join returns matches, left-only rows, and right-only rows.
**MySQL 8.0 has no native FULL OUTER JOIN syntax.** Emulate it with a
LEFT JOIN and a second branch for right-only rows:

``` sql
SELECT c.customer_id, c.customer_name, p.policy_id
FROM customers c
LEFT JOIN policies p ON c.customer_id = p.customer_id

UNION ALL

SELECT c.customer_id, c.customer_name, p.policy_id
FROM policies p
LEFT JOIN customers c ON c.customer_id = p.customer_id
WHERE c.customer_id IS NULL;
```

The first branch contains all left rows and matches. The second adds
only right-side rows without a match. Use a non-NULLable key to detect
the missing match.

## 8.3 SELF JOIN

A self join joins a table to itself. Aliases represent separate roles:

``` sql
SELECT e.employee_name AS employee,
       m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.employee_id;
```

LEFT JOIN preserves employees with no manager.

Find pairs of employees in the same city without mirrored pairs or
self-pairs:

``` sql
SELECT e1.employee_name, e2.employee_name, e1.city
FROM employees e1
JOIN employees e2
  ON e1.city = e2.city
 AND e1.employee_id < e2.employee_id;
```

### Data-engineering use

Reconciliation compares source and target records. Full-outer-join logic
helps identify source-only records, target-only records, and matches
that need attribute comparison.

### Interview Q&A

**Q: Does MySQL support FULL OUTER JOIN?** No native syntax; emulate
with LEFT JOIN plus right-only rows using UNION ALL.

**Q: What is a self join?** A table joined to itself using aliases,
commonly for employee-manager relationships.

**Q: Why UNION ALL in the emulation?** The second branch is restricted
to unmatched right rows, so it does not duplicate matches.

### Assignment

1.  Display every employee and manager.
2.  Find employees with no manager.
3.  Find employee pairs in the same department.
4.  Emulate a full outer join between customers and policies.

### Assessment --- answer all three

1.  Write a self join to print employee and manager names.
2.  Explain why UNION ALL works in the full-join emulation.
3.  How does full-join logic support source-target reconciliation?

------------------------------------------------------------------------

# Lesson 9 --- Subqueries and Correlated Subqueries

A subquery is a query nested inside another SQL statement. It may appear
in WHERE, FROM, or SELECT.

## 9.1 Scalar subquery

Returns one value:

``` sql
SELECT policy_id, premium
FROM policies
WHERE premium > (SELECT AVG(premium) FROM policies);
```

If a scalar subquery returns multiple rows, MySQL raises an error.

## 9.2 IN and NOT EXISTS

``` sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (
  SELECT customer_id FROM policies
);
```

To find customers without policies:

``` sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
  SELECT 1
  FROM policies p
  WHERE p.customer_id = c.customer_id
);
```

NOT IN can behave unexpectedly if the subquery returns NULL; NOT EXISTS
is often safer for anti-join logic.

## 9.3 Correlated subquery

The inner query references the outer query:

``` sql
SELECT p.policy_id, p.customer_id, p.premium
FROM policies p
WHERE p.premium > (
  SELECT AVG(p2.premium)
  FROM policies p2
  WHERE p2.customer_id = p.customer_id
);
```

This returns policies whose premium exceeds that customer's average
premium. The optimizer may transform the query; do not assume it
literally runs the inner query once for every outer row.

## 9.4 Derived table

``` sql
SELECT customer_id, total_premium
FROM (
  SELECT customer_id, SUM(premium) AS total_premium
  FROM policies
  GROUP BY customer_id
) AS customer_totals
WHERE total_premium > 15000;
```

A derived table in MySQL requires an alias.

## 9.5 EXISTS

EXISTS checks whether at least one row is returned:

``` sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1
  FROM policies p
  JOIN claims cl ON cl.policy_id = p.policy_id
  WHERE p.customer_id = c.customer_id
);
```

### Interview Q&A

**Q: Correlated subquery?** A nested query that references columns from
the outer query.

**Q: IN versus EXISTS?** Both express membership/existence; performance
depends on data, indexes, query shape, and optimizer. Choose clear
semantics and inspect the plan.

**Q: Why can NOT IN be dangerous?** A NULL in the subquery can make
comparisons UNKNOWN. NOT EXISTS avoids that particular trap.

### Assignment

1.  Find policies above the overall average premium.
2.  Find customers with at least one claim.
3.  Find customers with no policy using NOT EXISTS.
4.  Find policies above their customer's average premium.

### Assessment --- answer all three

1.  Explain scalar and correlated subqueries.
2.  Write a NOT EXISTS query for customers without policies.
3.  Why check NULL behavior with NOT IN?

------------------------------------------------------------------------

# Lesson 10 --- CTEs and Recursive CTEs

## 10.1 Common Table Expression

A CTE is a named result set defined with WITH and available to the
statement that follows. It is not automatically a persisted table.

``` sql
WITH active_policies AS (
  SELECT policy_id, customer_id, premium
  FROM policies
  WHERE status = 'Active'
)
SELECT *
FROM active_policies
WHERE premium > 10000;
```

## 10.2 Multiple CTEs

``` sql
WITH customer_totals AS (
  SELECT customer_id, COUNT(*) AS policy_count,
         SUM(premium) AS total_premium
  FROM policies
  GROUP BY customer_id
),
high_value_customers AS (
  SELECT *
  FROM customer_totals
  WHERE total_premium > 15000
)
SELECT *
FROM high_value_customers;
```

A later CTE can reference an earlier CTE in the same WITH clause.

## 10.3 CTE versus subquery

Both can express many of the same transformations. CTEs often make
multi-step logic easier to read. A CTE is not automatically faster;
MySQL may merge or materialize it depending on the query and optimizer.

## 10.4 Recursive CTE

A recursive CTE refers to itself and has an anchor member plus a
recursive member:

``` sql
WITH RECURSIVE employee_hierarchy AS (
  -- Anchor: top-level employees
  SELECT employee_id, employee_name, manager_id, 0 AS level_no
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- Recursive member: employees reporting to prior level
  SELECT e.employee_id, e.employee_name, e.manager_id,
         h.level_no + 1
  FROM employees e
  JOIN employee_hierarchy h
    ON e.manager_id = h.employee_id
)
SELECT *
FROM employee_hierarchy
ORDER BY level_no, employee_id;
```

Recursion stops when no new rows are produced or a limit is reached.
Protect against cycles and excessive depth; MySQL has a configurable
recursion-depth limit.

### Data-engineering use

Use named CTE stages to make a transformation auditable: filter valid
rows, standardise fields, aggregate at the correct grain, and select the
final output.

### Interview Q&A

**Q: What is a CTE?** A named result set scoped to one statement.

**Q: What are recursive CTE components?** Anchor member and recursive
member.

**Q: Is a CTE always faster?** No; it primarily helps structure and
readability. Inspect the execution plan.

### Assignment

1.  Create a CTE for active policies.
2.  Create two CTEs for customer totals and high-value customers.
3.  Use a recursive CTE for employee hierarchy.
4.  Add a hierarchy depth column.

### Assessment --- answer all three

1.  Explain anchor and recursive members.
2.  Write a CTE that calculates claim totals by policy.
3.  Why plan for recursion termination and cycles?

------------------------------------------------------------------------

# Lesson 11 --- Window Functions and Window Frames

## 11.1 Window function structure

A window function calculates across related rows while preserving
individual rows in the output. Unlike GROUP BY, it does not collapse
each group to one row.

``` sql
function_name(expression) OVER (
  PARTITION BY grouping_columns
  ORDER BY ordering_columns
  frame_clause
)
```

PARTITION BY creates independent windows. ORDER BY defines row order. A
frame specifies which rows around the current row are considered by
frame-sensitive functions.

## 11.2 ROW_NUMBER, RANK, DENSE_RANK

``` sql
SELECT employee_name, department_id, salary,
  ROW_NUMBER() OVER (
    PARTITION BY department_id
    ORDER BY salary DESC, employee_id
  ) AS row_num,
  RANK() OVER (
    PARTITION BY department_id ORDER BY salary DESC
  ) AS salary_rank,
  DENSE_RANK() OVER (
    PARTITION BY department_id ORDER BY salary DESC
  ) AS dense_salary_rank
FROM employees;
```

-   ROW_NUMBER gives a unique sequence, even for ties.
-   RANK gives ties the same rank and leaves a gap afterward.
-   DENSE_RANK gives ties the same rank without a gap.

Use a deterministic tie-breaker when exactly one row must win.

## 11.3 Running totals and moving averages

``` sql
SELECT claim_id, claim_date, claim_amount,
  SUM(claim_amount) OVER (
    ORDER BY claim_date, claim_id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_claim_amount
FROM claims;
```

Three-row moving average:

``` sql
SELECT claim_id, claim_date, claim_amount,
  AVG(claim_amount) OVER (
    ORDER BY claim_date, claim_id
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) AS moving_avg
FROM claims;
```

ROWS counts physical rows. RANGE uses peer/value ranges and behaves
differently. Specify frames deliberately.

## 11.4 LAG and LEAD

``` sql
SELECT claim_id, claim_date, claim_amount,
  LAG(claim_amount) OVER (
    ORDER BY claim_date, claim_id
  ) AS previous_amount,
  LEAD(claim_amount) OVER (
    ORDER BY claim_date, claim_id
  ) AS next_amount
FROM claims;
```

LAG and LEAD support period comparisons and change detection.

## 11.5 FIRST_VALUE and LAST_VALUE

``` sql
SELECT employee_name, department_id, salary,
  FIRST_VALUE(employee_name) OVER (
    PARTITION BY department_id
    ORDER BY salary DESC, employee_id
  ) AS highest_paid_employee
FROM employees;
```

LAST_VALUE can surprise because the default frame may end at the current
row/peer group. To get the last value of the whole partition, specify a
frame ending with `UNBOUNDED FOLLOWING`.

## 11.6 Deduplication

``` sql
WITH ranked AS (
  SELECT r.*,
    ROW_NUMBER() OVER (
      PARTITION BY policy_id
      ORDER BY updated_at DESC, ingestion_id DESC
    ) AS rn
  FROM raw_policy_feed r
)
SELECT *
FROM ranked
WHERE rn = 1;
```

The ordering defines which version wins. Add deterministic tie-breakers
and confirm the business definition of "latest".

### Interview Q&A

**Q: GROUP BY versus window function?** GROUP BY collapses rows; a
window function retains rows while calculating over a related set.

**Q: RANK versus DENSE_RANK?** RANK leaves gaps after ties; DENSE_RANK
does not.

**Q: What does PARTITION BY do?** Divides rows into independent windows,
for example one ranking per department.

### Assignment

1.  Rank employees by salary within department.
2.  Calculate a running total of claims.
3.  Show each claim with the previous claim amount.
4.  Deduplicate a raw table using ROW_NUMBER and latest timestamp.

### Assessment --- answer all three

1.  Explain ROW_NUMBER, RANK, and DENSE_RANK with a tie example.
2.  Rank policies by premium within policy type.
3.  Why might LAST_VALUE need an explicit frame?

------------------------------------------------------------------------

# Lesson 12 --- CASE WHEN and Conditional Aggregation

## 12.1 Searched CASE

``` sql
SELECT policy_id, premium,
  CASE
    WHEN premium >= 15000 THEN 'High'
    WHEN premium >= 10000 THEN 'Medium'
    ELSE 'Low'
  END AS premium_band
FROM policies;
```

Conditions are tested in order; the first matching branch wins. Put
higher thresholds first.

## 12.2 Simple CASE

``` sql
SELECT claim_id,
  CASE claim_status
    WHEN 'Approved' THEN 'Success'
    WHEN 'Pending' THEN 'In progress'
    WHEN 'Rejected' THEN 'Unsuccessful'
    ELSE 'Other'
  END AS status_label
FROM claims;
```

## 12.3 Conditional aggregation

``` sql
SELECT policy_id,
  SUM(CASE WHEN claim_status = 'Approved'
           THEN claim_amount ELSE 0 END) AS approved_amount,
  SUM(CASE WHEN claim_status = 'Pending'
           THEN claim_amount ELSE 0 END) AS pending_amount
FROM claims
GROUP BY policy_id;
```

CASE is useful for source-code mapping, quality flags, business bands,
and conditional measures. If mappings are large or frequently changed, a
reference table may be easier to govern than a long CASE expression.

### Interview Q&A

**Q: What does CASE return?** The result for the first matching WHEN
branch, or ELSE if none matches.

**Q: Can CASE be used inside SUM?** Yes; it is a common conditional
aggregation pattern.

**Q: When is a mapping table preferable?** When mappings are large,
frequently changing, shared, or need ownership and audit history.

### Assignment

1.  Classify policies into High/Medium/Low premium bands.
2.  Label claims by status.
3.  Calculate approved and rejected claim totals per policy.
4.  Flag employees earning at least 70,000.

### Assessment --- answer all three

1.  Write the premium-band CASE expression.
2.  Explain CASE inside SUM.
3.  When should mappings be stored in a reference table?

------------------------------------------------------------------------

# Lesson 13 --- Set Operators and Combining Query Results

## 13.1 UNION and UNION ALL

UNION combines result sets vertically and removes duplicate rows. UNION
ALL retains duplicates.

``` sql
SELECT city FROM customers
UNION
SELECT city FROM employees;
```

``` sql
SELECT city FROM customers
UNION ALL
SELECT city FROM employees;
```

Each branch must return the same number of columns in corresponding
positions, with compatible types. UNION may require
duplicate-elimination work. UNION ALL is appropriate when duplicates are
meaningful or branches are known not to overlap.

## 13.2 INTERSECT and EXCEPT-style logic in MySQL

Some database systems support INTERSECT and EXCEPT/MINUS. For MySQL 8.0,
use EXISTS and NOT EXISTS patterns for these use cases.

Cities present in both tables:

``` sql
SELECT DISTINCT c.city
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM employees e WHERE e.city = c.city
);
```

Customer cities absent from employees:

``` sql
SELECT DISTINCT c.city
FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM employees e WHERE e.city = c.city
);
```

### Interview Q&A

**Q: UNION versus UNION ALL?** UNION removes duplicates; UNION ALL
retains them and avoids that deduplication step.

**Q: What must UNION branches have?** Same number of columns with
compatible types in corresponding positions.

**Q: How find values in one table but not another?** Use NOT EXISTS or
an anti-join.

### Assignment

1.  Combine customer and employee cities with UNION.
2.  Repeat with UNION ALL and explain the difference.
3.  Find cities represented in both tables.
4.  Find customer cities absent from employees.

### Assessment --- answer all three

1.  Why can UNION ALL be preferable in ETL?
2.  What if UNION branches return different column counts?
3.  Write NOT EXISTS logic for customer cities without an employee-city
    match.

------------------------------------------------------------------------

# Lesson 14 --- Constraints, Views, Indexes, EXPLAIN, and Transactions

## 14.1 Constraints

-   PRIMARY KEY: unique, non-NULL row identifier.
-   FOREIGN KEY: enforces a referenced relationship.
-   UNIQUE: prevents duplicate values (NULL behavior is DBMS-specific).
-   NOT NULL: requires a value.
-   CHECK: enforces a Boolean condition; MySQL 8.0.16+ enforces CHECK
    constraints.
-   DEFAULT: supplies a value when omitted.

``` sql
CREATE TABLE quality_demo (
  record_id INT PRIMARY KEY,
  email VARCHAR(150) UNIQUE,
  amount DECIMAL(10,2) NOT NULL CHECK (amount >= 0),
  record_status VARCHAR(20) DEFAULT 'New'
);
```

Constraints protect the database, but pipelines should also report and
route rejected records.

## 14.2 Views

A view is a named query that behaves like a virtual table:

``` sql
CREATE VIEW active_policy_view AS
SELECT policy_id, customer_id, policy_type, premium
FROM policies
WHERE status = 'Active';

SELECT * FROM active_policy_view;
```

A normal view stores the query definition, not a separate persisted copy
of the result. MySQL does not provide materialized views in the same
native manner as some other systems.

## 14.3 Indexes

An index can help locate rows without scanning the whole table:

``` sql
CREATE INDEX idx_policies_customer
ON policies(customer_id);

CREATE INDEX idx_claims_policy_date
ON claims(policy_id, claim_date);
```

Composite-index column order matters. An index on
`(policy_id, claim_date)` is generally useful for lookups beginning with
policy_id; it is not equivalent to `(claim_date, policy_id)`.

Indexes cost storage and slow writes because they must be maintained. Do
not index every column. Choose indexes based on actual filters, joins,
ordering, selectivity, and workload.

## 14.4 EXPLAIN

``` sql
EXPLAIN
SELECT *
FROM claims
WHERE policy_id = 1001;
```

EXPLAIN helps inspect the planned access path, join order, and index
choices. `EXPLAIN ANALYZE` executes the query and reports actual
execution information; use care with expensive queries.

## 14.5 Transactions and ACID

ACID means: - Atomicity: all operations in the transaction succeed or
fail as a unit. - Consistency: valid constraints and rules remain
satisfied. - Isolation: concurrent work is controlled according to
isolation level. - Durability: committed changes persist.

``` sql
START TRANSACTION;
UPDATE policies
SET premium = premium + 500
WHERE policy_id = 1001;

SELECT * FROM policies WHERE policy_id = 1001;
ROLLBACK; -- use COMMIT if verified and intended
```

MySQL transaction behavior depends on storage engine; InnoDB is the
common transactional engine. DDL often causes implicit commits.

### Interview Q&A

**Q: What is an index?** A data structure that can accelerate suitable
reads, with storage and write overhead.

**Q: What is a composite index?** An index over multiple columns in a
defined order.

**Q: Explain ACID.** Atomicity, Consistency, Isolation, Durability.

### Assignment

1.  Create a table with primary key, unique, not-null, and check
    constraints.
2.  Create a view for active policies.
3.  Index claims by policy_id and claim_date.
4.  Run EXPLAIN on a query filtered by policy ID.
5.  Demonstrate COMMIT and ROLLBACK on test data.

### Assessment --- answer all three

1.  Why can too many indexes hurt performance?
2.  Explain the leftmost-prefix idea for composite indexes.
3.  Explain all four ACID properties.

------------------------------------------------------------------------

# Lesson 15 --- Advanced SQL for Data Engineering

## 15.1 Deduplication with ROW_NUMBER

Raw feeds may contain repeated events. Keep the latest record per
business key:

``` sql
WITH ranked AS (
  SELECT r.*,
    ROW_NUMBER() OVER (
      PARTITION BY policy_id
      ORDER BY updated_at DESC,
               ingestion_time DESC,
               source_record_id DESC
    ) AS rn
  FROM raw_policy_events r
)
SELECT *
FROM ranked
WHERE rn = 1;
```

The business must define "latest". A stable tie-breaker makes the result
deterministic. Conflicting records with identical business key and
timestamps may indicate a data-quality problem.

## 15.2 Top-N per group

Top two premiums per policy type:

``` sql
WITH ranked AS (
  SELECT policy_id, policy_type, premium,
    ROW_NUMBER() OVER (
      PARTITION BY policy_type
      ORDER BY premium DESC, policy_id
    ) AS rn
  FROM policies
)
SELECT *
FROM ranked
WHERE rn <= 2;
```

Use RANK or DENSE_RANK if ties should share positions.

## 15.3 Incremental extraction and watermarks

A pipeline may store the latest successfully processed update timestamp:

``` sql
SELECT *
FROM source_table
WHERE updated_at >  '2024-05-01 00:00:00'
  AND updated_at <= '2024-05-02 00:00:00';
```

Persist the watermark and define retry behavior. Late-arriving data,
timestamp precision, duplicate delivery, and failed batches must be
considered. Advance the watermark only after the batch has been safely
written and validated. An overlap window plus idempotent deduplication
is one possible strategy.

## 15.4 Source-target reconciliation

``` sql
SELECT
  s.policy_id,
  s.premium AS source_premium,
  t.premium AS target_premium,
  CASE
    WHEN t.policy_id IS NULL THEN 'MISSING_IN_TARGET'
    WHEN NOT (s.premium <=> t.premium) THEN 'VALUE_MISMATCH'
    ELSE 'MATCH'
  END AS reconciliation_status
FROM source_policies s
LEFT JOIN target_policies t
  ON s.policy_id = t.policy_id;
```

MySQL's `<=>` is NULL-safe equality. Reverse the anti-join to find
target-only records. Reconciliation should also compare counts and
relevant aggregates.

## 15.5 Data-quality checks

Duplicate keys:

``` sql
SELECT policy_id, COUNT(*) AS row_count
FROM raw_policy_events
GROUP BY policy_id
HAVING COUNT(*) > 1;
```

Invalid negative premium:

``` sql
SELECT * FROM policies WHERE premium < 0;
```

Orphan customer references:

``` sql
SELECT s.*
FROM staging_policies s
LEFT JOIN customers c ON s.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

Missing or blank required names:

``` sql
SELECT *
FROM staging_customers
WHERE customer_name IS NULL OR TRIM(customer_name) = '';
```

## 15.6 Query optimisation workflow

1.  Confirm correctness and output grain.
2.  Select only required columns.
3.  Filter early when logically valid.
4.  Check join keys and cardinality.
5.  Inspect EXPLAIN/EXPLAIN ANALYZE.
6.  Check useful indexes and column order.
7.  Avoid unnecessary DISTINCT and repeated scans.
8.  Avoid wrapping indexed filter columns in functions when a range
    predicate works.
9.  Test with representative data volume.
10. Measure before and after.

For example, instead of:

``` sql
WHERE YEAR(claim_date) = 2024
```

prefer:

``` sql
WHERE claim_date >= '2024-01-01'
  AND claim_date <  '2025-01-01'
```

The range predicate is a strong general pattern for index-friendly date
filtering.

### Interview Q&A

**Q: How do you deduplicate in SQL?** Use ROW_NUMBER partitioned by
business key, ordered by the business-defined latest timestamp and
deterministic tie-breakers; retain rn = 1.

**Q: What is a watermark?** A persisted processing boundary, often
timestamp or increasing ID, used to select the next incremental batch.

**Q: How do you reconcile source and target?** Compare by business key;
classify source-only, target-only, and changed rows; validate counts and
aggregates too.

### Assignment

1.  Detect duplicate keys in a raw policy feed.
2.  Keep the latest record per policy using ROW_NUMBER.
3.  Return the top two premiums per policy type.
4.  Find source policies missing from target.
5.  Find negative premiums and blank customer names.
6.  Explain how you would handle late-arriving updates.

### Assessment --- answer all three

1.  Write a deterministic ROW_NUMBER deduplication query and explain its
    ordering.
2.  Explain a watermark strategy and one failure mode it must handle.
3.  Design reconciliation that detects source-only, target-only, and
    changed records.

------------------------------------------------------------------------

# Phase 2 Capstone --- Insurance SQL Analytics and Data Quality

## Business scenario

An insurer needs a reliable analytical view of customers, policies, and
claims. Build queries without inflating amounts or losing customers who
have no policies.

## Part A --- Basic

1.  List active policies with customer name, policy type, and premium.
2.  Calculate total premium by policy type.
3.  Find customers with more than one policy.
4.  Find policies with no claims.
5.  Calculate approved claim amount by policy.

## Part B --- Intermediate

6.  Find each customer's highest-premium policy.
7.  Rank claims by amount within each policy.
8.  Calculate claim total and claim count per policy.
9.  Find policies above the average premium for their policy type.
10. Show every customer, including customers with no policy.

## Part C --- Data engineering

11. Create a CTE for claim totals by policy.
12. Use ROW_NUMBER to select the latest raw event per business key.
13. Find orphan policy records in staging.
14. Reconcile source and target policies by ID and premium.
15. Explain a repeatable incremental load using an updated_at watermark.

## Part D --- Design review

Before writing final SQL, document: - What does one row in the output
represent? - Which table drives the result? - Which joins are one-to-one
or one-to-many? - Can any join multiply premium or claim amounts? - How
are NULLs represented? - What makes the output deterministic? - Which
quality checks should fail the pipeline? - How will late-arriving
records be handled?

## Suggested solution patterns

Pre-aggregate claims before joining them to policies:

``` sql
WITH claim_totals AS (
  SELECT policy_id,
         COUNT(*) AS claim_count,
         SUM(claim_amount) AS total_claim_amount
  FROM claims
  GROUP BY policy_id
)
SELECT p.policy_id, p.policy_type,
       COALESCE(ct.claim_count, 0) AS claim_count,
       COALESCE(ct.total_claim_amount, 0) AS total_claim_amount
FROM policies p
LEFT JOIN claim_totals ct
  ON p.policy_id = ct.policy_id;
```

This produces at most one claim-summary row per policy before the join,
preventing claim-level rows from multiplying policy records.

Policies above their policy-type average:

``` sql
SELECT policy_id, policy_type, premium
FROM (
  SELECT policy_id, policy_type, premium,
         AVG(premium) OVER (PARTITION BY policy_type) AS type_avg
  FROM policies
) x
WHERE premium > type_avg;
```

------------------------------------------------------------------------

# SQL Interview Revision Sheet

## Joins

-   INNER: matching rows only.
-   LEFT: all left rows plus matches.
-   RIGHT: all right rows plus matches.
-   FULL OUTER: all rows from both sides; emulate in MySQL using LEFT
    JOIN plus right-only rows with UNION ALL.
-   SELF: table joined to itself using aliases.

## Aggregation

-   WHERE filters rows; HAVING filters groups.
-   GROUP BY creates groups.
-   COUNT(\*) counts rows; COUNT(column) ignores NULL.
-   Check join fan-out before summing.

## Subqueries and CTEs

-   Scalar subquery returns one value.
-   Correlated subquery refers to outer-query columns.
-   EXISTS tests whether at least one row exists.
-   CTE names a result set for one statement.
-   Recursive CTE uses anchor and recursive members.

## Window functions

-   ROW_NUMBER gives a unique sequence.
-   RANK leaves gaps after ties.
-   DENSE_RANK does not leave gaps.
-   PARTITION BY creates independent windows.
-   LAG/LEAD access neighboring rows.
-   Specify window frames deliberately.

## Data engineering

-   Define grain before joining.
-   Deduplicate with ROW_NUMBER and deterministic ordering.
-   Use half-open timestamp ranges.
-   Persist and safely advance watermarks.
-   Reconcile source and target by business key.
-   Validate NULLs, duplicates, orphan keys, and invalid values.
-   Use EXPLAIN and measure before optimising.

------------------------------------------------------------------------

# Phase 2 Study Checklist

This checklist tracks this new study guide. It is deliberately separate
from the previously reported tracker baseline because the original
lesson-to-topic mapping was not fully visible.

-   [ ] Lesson 1 --- SQL and relational fundamentals
-   [ ] Lesson 2 --- SELECT, DISTINCT, aliases, expressions
-   [ ] Lesson 3 --- Filtering, sorting, limiting
-   [ ] Lesson 4 --- String, numeric, date, NULL functions
-   [ ] Lesson 5 --- Aggregates, GROUP BY, HAVING
-   [ ] Lesson 6 --- DDL and DML
-   [ ] Lesson 7 --- INNER JOIN and LEFT JOIN
-   [ ] Lesson 8 --- RIGHT JOIN, FULL OUTER emulation, SELF JOIN
-   [ ] Lesson 9 --- Subqueries and correlated subqueries
-   [ ] Lesson 10 --- CTEs and recursive CTEs
-   [ ] Lesson 11 --- Window functions and frames
-   [ ] Lesson 12 --- CASE and conditional aggregation
-   [ ] Lesson 13 --- Set operators and result combination
-   [ ] Lesson 14 --- Constraints, views, indexes, transactions
-   [ ] Lesson 15 --- Advanced SQL for data engineering
-   [ ] Complete the insurance capstone
-   [ ] Answer all 15 lessons' three-question assessments

## Rule for moving to Phase 3

Do not move to Phase 3 (Python) simply because this guide is available
or because some SQL topics were previously discussed. Study the complete
Phase 2 guide first.

When you are ready, send this exact confirmation:

**"I have studied the Phase 2 file. Move to Phase 3."**

Until then, continue Phase 2. The previously reported preparation
tracker showed SQL at **6/15**. Keep that baseline separate from this
new guide's checklist rather than assuming the two numbering systems
match.
