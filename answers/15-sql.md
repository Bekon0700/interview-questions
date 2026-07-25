---
title: SQL — Answers
topic: sql
tags: [interview, fullstack, sql, postgresql, mysql]
related: ["[[05-databases]]", "[[16-mongodb]]"]
---

# SQL — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example** (SQL query or schema), and a **Pros / Cons** box. Numbers match [[questions/15-sql|the questions file]]. General database concepts (ACID, CAP, caching) live in [[05-databases]]; MongoDB-specific depth in [[16-mongodb]].

---

## Beginner

### 1. SQL vs NoSQL

> [!question] Q1
> What is the difference between SQL and NoSQL? (also in databases file)

**SQL (relational)** databases store data in **tables** with a fixed **schema** — rows and columns with defined types. They excel at **relationships** (joins), **ACID transactions**, and **strong consistency**. Examples: PostgreSQL, MySQL.

**NoSQL** covers non-relational models — documents, key-value, column-family, graph — with flexible schemas and horizontal scaling patterns. Examples: MongoDB, Redis, Cassandra.

Choose **SQL** when data is structured, relationships matter, and integrity is critical (orders, payments, inventory). Choose **NoSQL** when schema evolves fast, you need massive scale on specific access patterns, or a specialized model fits better (sessions in Redis, flexible catalogs in MongoDB).

> [!example]
> ```sql
> -- SQL: orders linked to customers with enforced integrity
> SELECT o.id, c.name, o.total
> FROM orders o
> JOIN customers c ON c.id = o.customer_id
> WHERE o.status = 'paid';
> ```

> [!success] Pros / Cons
> **SQL pros:** Joins, ACID, mature tooling, strong data integrity.  
> **SQL cons:** Schema migrations required; horizontal write scaling is harder.  
> **NoSQL pros:** Flexible schema, horizontal scale, optimized access patterns.  
> **NoSQL cons:** Weaker default consistency; app often enforces rules SQL would enforce in the DB.

> [!tip] Interview tip
> Give a **concrete split**: "Orders and payments → PostgreSQL; product catalog with varying attributes → MongoDB; sessions → Redis." See [[05-databases#1 SQL vs NoSQL]] for the full comparison.

> [!info] Further study
> - [PostgreSQL Documentation](https://www.postgresql.org/docs/)
> - [MongoDB — SQL vs NoSQL](https://www.mongodb.com/resources/basics/databases/nosql-explained/nosql-vs-sql)

---

### 2. SQL clause order

> [!question] Q2
> What are the main SQL clauses and their order (SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY, LIMIT)?

**Written order** (how you type the query):
`SELECT` → `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `ORDER BY` → `LIMIT`

**Logical execution order** (how the engine processes it):
`FROM` / `JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`

This explains why you **cannot** use a SELECT alias in `WHERE` (WHERE runs before SELECT) but **can** in `ORDER BY` (ORDER BY runs after SELECT).

> [!example]
> ```sql
> SELECT dept, AVG(salary) AS avg_sal
> FROM employees
> WHERE status = 'active'           -- filter rows first
> GROUP BY dept                     -- one row per department
> HAVING AVG(salary) > 50000        -- filter groups (can use aggregate)
> ORDER BY avg_sal DESC             -- sort after SELECT (alias OK)
> LIMIT 10;
> ```

> [!success] Pros / Cons
> **Pros:** Predictable mental model for debugging query results; explains alias scoping rules.  
> **Cons:** Logical order differs from written order — a common source of confusion for beginners.

> [!info] Further study
> - [PostgreSQL — SELECT reference](https://www.postgresql.org/docs/current/sql-select.html)
> - [MySQL — SELECT statement](https://dev.mysql.com/doc/refman/8.0/en/select.html)

---

### 3. WHERE vs HAVING

> [!question] Q3
> What is the difference between `WHERE` and `HAVING`?

Both filter data, but at different stages.

**`WHERE`** filters **individual rows** before grouping — you cannot use aggregate functions here (no `COUNT(*)`, no `AVG()`).

**`HAVING`** filters **groups** after `GROUP BY` — you can use aggregate functions to keep or discard whole groups.

> [!example]
> ```sql
> -- Departments with more than 5 active employees earning avg > 60k
> SELECT dept, COUNT(*) AS headcount, AVG(salary) AS avg_sal
> FROM employees
> WHERE status = 'active'              -- row filter
> GROUP BY dept
> HAVING COUNT(*) > 5 AND AVG(salary) > 60000;  -- group filter
> ```

> [!success] Pros / Cons
> **WHERE pros:** Filters early — less data to aggregate, usually faster.  
> **WHERE cons:** Cannot reference aggregates.  
> **HAVING pros:** Expresses conditions on grouped results naturally.  
> **HAVING cons:** Applied after aggregation — filtering on non-aggregated columns in HAVING instead of WHERE is slower.

> [!tip] Interview tip
> Rule of thumb: put every row-level condition in **WHERE**; reserve **HAVING** for conditions on aggregates.

---

### 4. Types of JOINs

> [!question] Q4
> What are the types of JOINs (INNER, LEFT, RIGHT, FULL, CROSS)? (commonly asked)

JOINs combine rows from two tables based on a related column.

| Join | Result |
|------|--------|
| **INNER** | Only rows that match in **both** tables |
| **LEFT** | All rows from the **left** table + matched right (NULLs if no match) |
| **RIGHT** | All rows from the **right** table + matched left |
| **FULL OUTER** | All rows from **both**; NULL where no match |
| **CROSS** | Cartesian product — every row paired with every row (no ON clause) |

> [!example]
> ```sql
> -- All customers and their orders (including customers with zero orders)
> SELECT c.name, o.id AS order_id, o.total
> FROM customers c
> LEFT JOIN orders o ON o.customer_id = c.id;

> -- Only customers who have placed at least one order
> SELECT c.name, o.id, o.total
> FROM customers c
> INNER JOIN orders o ON o.customer_id = c.id;
> ```

> [!success] Pros / Cons
> **INNER pros:** Smallest result, clearest when you only want matches. **Cons:** Silently drops unmatched rows.  
> **LEFT pros:** Preserves the "primary" table (all customers, even without orders). **Cons:** NULL columns for unmatched side — handle in app or COALESCE.  
> **CROSS pros:** Useful for generating combinations (date × product grids). **Cons:** Explodes row count — use carefully.

> [!info] Further study
> - [PostgreSQL — Joins](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-FROM)
> - [W3Schools — SQL JOINs](https://www.w3schools.com/sql/sql_join.asp) (quick visual reference)

---

### 5. Primary key vs unique key vs foreign key

> [!question] Q5
> What is a primary key vs a unique key vs a foreign key?

**Primary key (PK):** Uniquely identifies each row. Must be **NOT NULL** and **unique**. One per table. Often an auto-increment `id` or a natural key like `email` (if guaranteed unique).

**Unique key:** Enforces uniqueness on a column (or column set) but allows **one NULL** in most databases (PostgreSQL treats NULLs as distinct in unique constraints). You can have multiple unique constraints per table.

**Foreign key (FK):** A column that **references** another table's primary key (or unique key), enforcing **referential integrity** — you can't insert an order with a `customer_id` that doesn't exist.

> [!example]
> ```sql
> CREATE TABLE customers (
>   id         SERIAL PRIMARY KEY,
>   email      VARCHAR(255) UNIQUE NOT NULL
> );

> CREATE TABLE orders (
>   id          SERIAL PRIMARY KEY,
>   customer_id INT NOT NULL REFERENCES customers(id),
>   total       NUMERIC(10,2) NOT NULL
> );
> ```

> [!success] Pros / Cons
> **PK pros:** Fast lookups, clear row identity, often the clustered index.  
> **Unique pros:** Prevents duplicates (emails, slugs) without being the primary identifier.  
> **FK pros:** DB-enforced relationships; prevents orphan rows.  
> **FK cons:** Slight write overhead; cascading deletes/updates must be designed carefully.

> [!info] Further study
> - [PostgreSQL — Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)

---

### 6. DELETE vs TRUNCATE vs DROP

> [!question] Q6
> What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?

All remove data, but at different levels:

| Command | Scope | WHERE? | Rollback? | Table structure |
|---------|-------|--------|-----------|-----------------|
| **DELETE** | Specific rows | Yes | Yes (in transaction) | Kept |
| **TRUNCATE** | All rows | No | Depends on DB/transaction | Kept |
| **DROP** | Entire table | N/A | No (destructive) | Removed |

`TRUNCATE` is faster than `DELETE` without a WHERE clause because it deallocates pages instead of row-by-row logging (implementation varies by DB).

> [!example]
> ```sql
> DELETE FROM logs WHERE created_at < '2024-01-01';  -- remove old rows
> TRUNCATE TABLE staging_import;                      -- empty staging table fast
> DROP TABLE temp_migration_backup;                     -- remove table entirely
> ```

> [!success] Pros / Cons
> **DELETE pros:** Granular, transactional, triggers fire. **Cons:** Slow on large tables.  
> **TRUNCATE pros:** Very fast full wipe. **Cons:** No WHERE; may not trigger row-level triggers; FK constraints can block it.  
> **DROP pros:** Clean removal. **Cons:** Irreversible without backup — data and schema gone.

> [!warning]
> Always confirm environment before `TRUNCATE` or `DROP`. Use transactions in production data changes.

---

### 7. COUNT variants

> [!question] Q7
> What is the difference between `COUNT(*)`, `COUNT(column)`, and `COUNT(DISTINCT column)`?

**`COUNT(*)`** — counts **all rows** in the group, including rows where every column is NULL.

**`COUNT(column)`** — counts rows where `column` is **NOT NULL**. Ignores NULLs.

**`COUNT(DISTINCT column)`** — counts **unique non-null** values in the column.

> [!example]
> ```sql
> SELECT
>   COUNT(*)                    AS total_rows,
>   COUNT(email)                AS rows_with_email,
>   COUNT(DISTINCT email)       AS unique_emails
> FROM users;
> -- total_rows: 1000, rows_with_email: 980, unique_emails: 950
> ```

> [!success] Pros / Cons
> **`COUNT(*)` pros:** Correct total row count; optimizers often treat it efficiently.  
> **`COUNT(column)` pros:** Useful when NULL means "unknown/missing." **Cons:** Not the same as total rows if NULLs exist.  
> **`COUNT(DISTINCT)` pros:** Deduplication in one query. **Cons:** More expensive — requires sort or hash of distinct values.

---

## Intermediate

### 8. Second highest salary

> [!question] Q8
> Write a query to find the second highest salary from an employees table. (classic)

Classic interview question. Multiple valid approaches — know at least two.

**Subquery approach:** Find the max salary, then find the max salary below that.  
**OFFSET approach:** Sort descending, skip the first, take one.  
**Window function:** `DENSE_RANK()` handles ties cleanly for "Nth highest."

> [!example]
> ```sql
> -- Subquery (handles no second salary → returns NULL)
> SELECT MAX(salary) AS second_highest
> FROM employees
> WHERE salary < (SELECT MAX(salary) FROM employees);

> -- OFFSET (simple; tie behavior differs)
> SELECT DISTINCT salary
> FROM employees
> ORDER BY salary DESC
> LIMIT 1 OFFSET 1;

> -- Window function (best for Nth rank with ties)
> SELECT salary
> FROM (
>   SELECT salary,
>          DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
>   FROM employees
> ) ranked
> WHERE rnk = 2;
> ```

> [!success] Pros / Cons
> **Subquery pros:** Works everywhere, clear intent. **Cons:** Two table scans.  
> **OFFSET pros:** Readable. **Cons:** `OFFSET 1` gives arbitrary row among tied top salaries unless you use DISTINCT carefully.  
> **Window pros:** Handles ties and "Nth" generically. **Cons:** Slightly more syntax.

> [!tip] Interview tip
> Ask clarifying questions: "Should ties count as the same rank?" and "Return one row or all employees at the second-highest salary?"

---

### 9. GROUP BY and aggregates

> [!question] Q9
> How do GROUP BY and aggregate functions (SUM, AVG, COUNT, MIN, MAX) work?

`GROUP BY` collapses rows sharing the same value(s) in the grouped column(s) into **one output row per group**. Aggregate functions compute a single value across each group:

- `SUM()` — total
- `AVG()` — mean
- `COUNT()` — row count in group
- `MIN()` / `MAX()` — smallest / largest

**Rule:** Every column in `SELECT` must either be in `GROUP BY` or wrapped in an aggregate — otherwise the DB doesn't know which row's value to show.

> [!example]
> ```sql
> SELECT
>   dept,
>   COUNT(*)           AS headcount,
>   AVG(salary)        AS avg_salary,
>   MIN(salary)        AS min_salary,
>   MAX(salary)        AS max_salary,
>   SUM(salary)        AS payroll_total
> FROM employees
> WHERE status = 'active'
> GROUP BY dept
> ORDER BY payroll_total DESC;
> ```

> [!success] Pros / Cons
> **Pros:** Powerful summarization in one query; foundation of reporting.  
> **Cons:** Easy to write invalid SQL (non-aggregated columns missing from GROUP BY); large groups without indexes can be slow.

---

### 10. Indexes and B-trees

> [!question] Q10
> What is an index and how does it work under the hood (B-tree)? What are the downsides?

An **index** is a separate lookup structure — usually a **B-tree** (balanced tree) — that maps indexed column values to row locations (or primary key references). Instead of scanning every row (**O(n)** full table scan), the DB traverses the tree in **O(log n)** to find matches. B-trees also support **range queries** (`WHERE age BETWEEN 18 AND 30`) and **ordered retrieval** (`ORDER BY created_at`).

**Downsides:**
- Extra **disk and memory** storage
- **Slower writes** — every INSERT/UPDATE/DELETE must update affected indexes
- **Wrong indexes** waste space without helping reads

Index columns you actually filter, join, or sort on in real queries.

> [!example]
> ```sql
> EXPLAIN ANALYZE
> SELECT * FROM users WHERE email = 'alice@example.com';
> -- Seq Scan on users  (cost=0..10000) — slow without index

> CREATE INDEX idx_users_email ON users(email);

> EXPLAIN ANALYZE
> SELECT * FROM users WHERE email = 'alice@example.com';
> -- Index Scan using idx_users_email  (cost=0..8) — fast
> ```

> [!success] Pros / Cons
> **Pros:** Dramatic read speedup on large tables; enables efficient sorts and joins.  
> **Cons:** Write amplification; index maintenance; choosing wrong columns or column order in composite indexes wastes effort.

> [!info] Further study
> - [PostgreSQL — Indexes](https://www.postgresql.org/docs/current/indexes.html)
> - [PostgreSQL — Index types (B-tree, GIN, GiST)](https://www.postgresql.org/docs/current/indexes-types.html)
> - [Use The Index, Luke](https://use-the-index-luke.com/) — practical index guide

---

### 11. Clustered vs non-clustered index

> [!question] Q11
> What is the difference between a clustered and non-clustered index?

**Clustered index:** Determines the **physical order** of rows in the table (or main storage). There is typically **one** clustered index per table — usually the primary key. Range scans on the clustered key are very fast because related rows are stored together.

**Non-clustered index:** A **separate structure** with indexed values plus pointers to the actual rows. You can have **many** non-clustered indexes per table.

In **PostgreSQL**, the table heap is unordered; the PK creates a unique B-tree index but data isn't physically sorted like SQL Server's clustered index. In **InnoDB (MySQL)**, the PK **is** the clustered index; secondary indexes store PK values as row pointers.

> [!example]
> ```sql
> -- PostgreSQL: PK creates a unique B-tree; secondary index is separate
> CREATE TABLE orders (
>   id          BIGSERIAL PRIMARY KEY,       -- indexed uniquely
>   customer_id INT NOT NULL,
>   created_at  TIMESTAMPTZ NOT NULL
> );
> CREATE INDEX idx_orders_customer ON orders(customer_id);  -- non-clustered style
> ```

> [!success] Pros / Cons
> **Clustered pros:** Fast range scans on PK; one less lookup for PK queries. **Cons:** Only one; inserts in middle of key range can cause page splits.  
> **Non-clustered pros:** Many indexes for different query patterns. **Cons:** Extra lookup (index → row); more storage.

> [!info] Further study
> - [MySQL InnoDB — Clustered index](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)
> - [PostgreSQL — Index-only scans](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)

---

### 12. Normalization and denormalization

> [!question] Q12
> What is normalization (1NF, 2NF, 3NF) and when would you denormalize?

**Normalization** organizes tables to **reduce redundancy** and update anomalies.

- **1NF:** Atomic values; no repeating groups (no `phone1, phone2, phone3` columns — use a separate table).
- **2NF:** No partial dependency — every non-key column depends on the **whole** composite primary key.
- **3NF:** No transitive dependency — non-key columns depend **only** on the primary key, not on other non-key columns.

**Denormalization** deliberately introduces redundancy (duplicate columns, summary tables) to **speed up reads** — common in reporting, dashboards, and read-heavy workloads where joins are too expensive.

> [!example]
> ```sql
> -- Normalized: order total computed from line items
> SELECT o.id, SUM(oi.qty * oi.unit_price) AS total
> FROM orders o
> JOIN order_items oi ON oi.order_id = o.id
> GROUP BY o.id;

> -- Denormalized: store total on orders for fast reads
> ALTER TABLE orders ADD COLUMN total NUMERIC(10,2);
> -- Updated by trigger or app when line items change
> ```

> [!success] Pros / Cons
> **Normalization pros:** Data integrity, no update anomalies, smaller storage. **Cons:** More joins at query time.  
> **Denormalization pros:** Faster reads, simpler hot-path queries. **Cons:** Must keep redundant data in sync; risk of inconsistency.

> [!tip] Interview tip
> Default answer: "Normalize for OLTP writes; denormalize selectively for read-heavy reports with clear sync strategy."

---

### 13. Subqueries vs CTEs

> [!question] Q13
> What are subqueries and CTEs (WITH clause)? When use each?

A **subquery** is a query nested inside another — in `WHERE`, `FROM`, or `SELECT`. A **CTE (Common Table Expression)** uses `WITH name AS (...)` to define a named temporary result you reference in the main query. CTEs can be **referenced multiple times** and support **`WITH RECURSIVE`** for hierarchical data (org charts, category trees).

> [!example]
> ```sql
> -- Subquery in WHERE
> SELECT name, salary
> FROM employees
> WHERE salary > (SELECT AVG(salary) FROM employees);

> -- CTE — clearer for multi-step logic
> WITH dept_avg AS (
>   SELECT dept, AVG(salary) AS avg_sal
>   FROM employees
>   GROUP BY dept
> )
> SELECT e.name, e.salary, d.avg_sal
> FROM employees e
> JOIN dept_avg d ON d.dept = e.dept
> WHERE e.salary > d.avg_sal;
> ```

> [!success] Pros / Cons
> **Subquery pros:** Compact for simple one-off filters. **Cons:** Deep nesting hurts readability.  
> **CTE pros:** Readable, reusable within the query, supports recursion. **Cons:** In some DBs/versions, CTEs can be optimization fences (PostgreSQL 12+ often inlines them).

> [!info] Further study
> - [PostgreSQL — WITH queries (CTEs)](https://www.postgresql.org/docs/current/queries-with.html)

---

### 14. N+1 query problem

> [!question] Q14
> What is the N+1 query problem and how do you fix it in SQL? (also in databases file)

The **N+1 problem** happens when your app runs **1 query** to fetch N parent rows, then **N additional queries** — one per row — to fetch related data. Example: fetch 100 orders, then loop and query each customer separately → 101 queries.

**Fix in SQL:** Use a **JOIN** or a single **`IN (...)`** batch query to fetch all related rows at once. In ORMs, use **eager loading** (`include`, `JOIN FETCH`, `.populate()`).

> [!example]
> ```sql
> -- BAD pattern (app loop): 1 + N queries
> -- SELECT * FROM orders LIMIT 100;
> -- foreach order: SELECT * FROM customers WHERE id = ?

> -- GOOD: one JOIN
> SELECT o.id, o.total, c.name, c.email
> FROM orders o
> JOIN customers c ON c.id = o.customer_id
> LIMIT 100;

> -- GOOD: batch IN
> SELECT * FROM customers WHERE id IN (1, 2, 3, ...);
> ```

> [!success] Pros / Cons
> **JOIN pros:** One round-trip, DB optimizes the plan. **Cons:** Can return duplicate parent columns; wide rows.  
> **IN batch pros:** Two queries, simple when ORM manages mapping. **Cons:** Large IN lists can degrade; some ORMs still lazy-load by default.

> [!tip] See also [[05-databases#14 N+1 query problem]] and ORM-specific fixes in your stack.

---

### 15. Finding duplicate rows

> [!question] Q15
> How do you find duplicate rows in a table?

Group by the column(s) that should be unique and keep groups where `COUNT(*) > 1`. To see the actual duplicate rows, join back to the table or use a window function.

> [!example]
> ```sql
> -- Which emails are duplicated?
> SELECT email, COUNT(*) AS cnt
> FROM users
> GROUP BY email
> HAVING COUNT(*) > 1;

> -- All rows that are duplicates (PostgreSQL)
> SELECT u.*
> FROM users u
> JOIN (
>   SELECT email
>   FROM users
>   GROUP BY email
>   HAVING COUNT(*) > 1
> ) d ON d.email = u.email
> ORDER BY u.email, u.id;

> -- Window alternative
> SELECT *
> FROM (
>   SELECT *, COUNT(*) OVER (PARTITION BY email) AS dup_count
>   FROM users
> ) t
> WHERE dup_count > 1;
> ```

> [!success] Pros / Cons
> **GROUP BY + HAVING pros:** Simple, fast for finding offending values. **Cons:** Doesn't return full rows without a join.  
> **Window pros:** Full rows in one pass. **Cons:** Slightly more syntax.

> [!tip] Prevention: add a `UNIQUE` constraint on the column so the DB rejects duplicates at insert time.

---

## Advanced

### 16. Window functions

> [!question] Q16
> What are window functions (ROW_NUMBER, RANK, DENSE_RANK, LAG/LEAD)? Give a use case.

**Window functions** compute values across a **window** of rows related to the current row **without collapsing** them into one row (unlike `GROUP BY`). You define the window with `OVER (PARTITION BY ... ORDER BY ...)`.

| Function | Behavior |
|----------|----------|
| `ROW_NUMBER()` | Unique sequential number (1, 2, 3) — ties get arbitrary order |
| `RANK()` | Same rank for ties; **gaps** after ties (1, 2, 2, 4) |
| `DENSE_RANK()` | Same rank for ties; **no gaps** (1, 2, 2, 3) |
| `LAG(col)` / `LEAD(col)` | Previous / next row's value in the partition |

**Use cases:** Rankings, running totals, period-over-period comparison, top-N per group.

> [!example]
> ```sql
> SELECT
>   name,
>   dept,
>   salary,
>   RANK()       OVER (PARTITION BY dept ORDER BY salary DESC) AS rank_in_dept,
>   LAG(salary)  OVER (PARTITION BY dept ORDER BY hire_date)    AS prev_hire_salary
> FROM employees;
> ```

> [!success] Pros / Cons
> **Pros:** Expressive analytics in SQL; avoids self-joins for rankings and comparisons.  
> **Cons:** Can be expensive on large partitions without supporting indexes on `PARTITION BY` / `ORDER BY` columns.

> [!info] Further study
> - [PostgreSQL — Window functions](https://www.postgresql.org/docs/current/functions-window.html)

---

### 17. Top N per group

> [!question] Q17
> Write a query to get the top N records per group (e.g., top 3 orders per customer).

Partition by the group column, order within each partition by your ranking criteria, assign `ROW_NUMBER()`, then filter `rn <= N`. This pattern appears constantly in interviews and production reporting.

> [!example]
> ```sql
> WITH ranked AS (
>   SELECT
>     o.*,
>     ROW_NUMBER() OVER (
>       PARTITION BY customer_id
>       ORDER BY total DESC, created_at DESC
>     ) AS rn
>   FROM orders o
> )
> SELECT id, customer_id, total, created_at
> FROM ranked
> WHERE rn <= 3;
> ```

> [!success] Pros / Cons
> **Pros:** Clear, works in PostgreSQL/MySQL 8+/SQL Server; handles ties with `RANK()` if you want all tied rows.  
> **Cons:** Full table scan + sort per partition on large data — indexes on `(customer_id, total DESC)` help.

> [!tip] For ties at the N boundary, switch `ROW_NUMBER()` to `RANK()` or use `DENSE_RANK()` depending on whether you want all tied rows included.

---

### 18. Transactions and isolation levels

> [!question] Q18
> What are transactions and isolation levels? (also in databases file)

A **transaction** groups multiple SQL statements into one atomic unit — either **all commit** or **all roll back** (`BEGIN` → `COMMIT` / `ROLLBACK`). This guarantees **ACID** properties for critical operations (transfer money, place order).

**Isolation levels** control how much one transaction sees of another's uncommitted or concurrent changes:

| Level | Dirty read | Non-repeatable read | Phantom read |
|-------|------------|---------------------|--------------|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | No | Possible | Possible |
| Repeatable Read | No | No | Possible* |
| Serializable | No | No | No |

*PostgreSQL's Repeatable Read also prevents phantom reads via MVCC.

Higher isolation = more consistency, less concurrency.

> [!example]
> ```sql
> BEGIN;
> UPDATE accounts SET balance = balance - 100 WHERE id = 1;
> UPDATE accounts SET balance = balance + 100 WHERE id = 2;
> COMMIT;
> -- Both updates succeed or neither does
> ```

> [!success] Pros / Cons
> **Transactions pros:** Data integrity for multi-step operations. **Cons:** Locks and contention under high concurrency.  
> **Higher isolation pros:** Fewer anomalies. **Cons:** More blocking, retries, deadlocks.

> [!tip] See [[05-databases]] for ACID, CAP, and replication context.

> [!info] Further study
> - [PostgreSQL — Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
> - [MySQL — InnoDB transaction isolation](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)

---

### 19. Optimizing a slow query

> [!question] Q19
> How do you optimize a slow SQL query? Walk through your approach (EXPLAIN/EXPLAIN ANALYZE).

Systematic approach:

1. **Reproduce** — identify the slow query and typical parameters.
2. **EXPLAIN / EXPLAIN ANALYZE** — read the execution plan: sequential scans, nested loops, sort/hash costs, rows examined vs returned.
3. **Fix the biggest cost first:**
   - Add or adjust **indexes** on WHERE/JOIN/ORDER BY columns (composite index column order matters).
   - **Rewrite the query** — avoid `SELECT *`, functions on indexed columns (`WHERE YEAR(created_at) = 2024`), unnecessary subqueries.
   - **Update statistics** — `ANALYZE` in PostgreSQL so the planner chooses good plans.
4. **Re-measure** after each change — one index can help one query and hurt another.

> [!example]
> ```sql
> EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
> SELECT o.id, c.name, o.total
> FROM orders o
> JOIN customers c ON c.id = o.customer_id
> WHERE o.status = 'pending'
>   AND o.created_at >= '2025-01-01'
> ORDER BY o.created_at DESC
> LIMIT 50;

> -- Fix: composite index matching filter + sort
> CREATE INDEX idx_orders_status_created ON orders(status, created_at DESC);
> ```

> [!success] Pros / Cons
> **Pros:** Data-driven optimization; often 10×–100× gains from one good index. **Cons:** Wrong indexes slow writes; over-indexing is real; ORM-generated SQL can hide the problem until production.

> [!tip] CV tie-in
> Same methodology as your MongoDB 37s → 1s win: measure, find the bottleneck, fix, re-measure. Mention `EXPLAIN ANALYZE` by name.

> [!info] Further study
> - [PostgreSQL — Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
> - [Use The Index, Luke](https://use-the-index-luke.com/)

---

### 20. Deadlocks

> [!question] Q20
> What is a deadlock and how do you prevent it?

A **deadlock** occurs when two (or more) transactions each hold a lock the other needs — both wait forever. The database **detects** this and **aborts one transaction** (the "deadlock victim"); the app should **retry** the aborted work.

**Prevention:**
- Acquire locks in a **consistent order** across all transactions (always update table A before table B).
- Keep transactions **short** — less time holding locks.
- Use appropriate **isolation levels** — don't over-use Serializable.
- Add **indexes** so queries lock fewer rows (row locks vs table locks).

> [!example]
> ```sql
> -- Transaction 1                    -- Transaction 2
> UPDATE accounts SET ... WHERE id=1;  UPDATE accounts SET ... WHERE id=2;
> UPDATE accounts SET ... WHERE id=2;  UPDATE accounts SET ... WHERE id=1;
> -- DEADLOCK — DB kills one transaction
> ```

> [!success] Pros / Cons
> **Retry-on-deadlock pros:** Simple, standard pattern in ORMs and connection pools. **Cons:** Retries under heavy contention can cascade.  
> **Consistent lock ordering pros:** Prevents most deadlocks. **Cons:** Requires discipline across the whole codebase.

> [!info] Further study
> - [PostgreSQL — Deadlocks](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-DEADLOCKS)
> - [MySQL — InnoDB deadlocks](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html)

---

### 21. OLTP vs OLAP

> [!question] Q21
> What is the difference between OLTP and OLAP?

**OLTP (Online Transaction Processing):** Many short, simple read/write operations — insert order, update inventory, fetch user profile. Optimized for **concurrency** and **consistency**. Normalized schemas, indexed for point lookups.

**OLAP (Online Analytical Processing):** Complex **read-heavy aggregations** over huge datasets — sales by region by quarter, funnel analysis. Optimized for **scan and aggregate**. Denormalized star/snowflake schemas, columnar warehouses (BigQuery, Redshift, ClickHouse).

Often OLTP data is **ETL'd** nightly (or streamed) into an OLAP store for BI dashboards.

> [!example]
> ```sql
> -- OLTP: fast point read/write
> INSERT INTO orders (customer_id, total) VALUES (42, 99.99);

> -- OLAP: heavy aggregation over millions of rows
> SELECT region, DATE_TRUNC('month', created_at), SUM(revenue)
> FROM fact_sales
> GROUP BY 1, 2;
> ```

> [!success] Pros / Cons
> **OLTP pros:** Low-latency transactions, integrity. **Cons:** Bad at large ad-hoc analytics.  
> **OLAP pros:** Fast reporting at scale. **Cons:** Stale data (batch lag), not for real-time writes.

---

### 22. Efficient pagination

> [!question] Q22
> How do you paginate efficiently in SQL (OFFSET vs keyset pagination)?

**OFFSET/LIMIT** is simple (`LIMIT 20 OFFSET 1000`) but **slow for deep pages** — the DB must scan and discard all skipped rows. Results can also **shift** if rows are inserted/deleted between page requests (duplicates or gaps).

**Keyset (cursor) pagination** uses the last seen value as a bookmark: `WHERE id < :lastId ORDER BY id DESC LIMIT 20`. With an index on `id`, each page is **O(log n)** regardless of depth — stable under concurrent inserts.

> [!example]
> ```sql
> -- OFFSET — fine for page 1–5, bad for page 5000
> SELECT id, title, created_at
> FROM posts
> ORDER BY created_at DESC, id DESC
> LIMIT 20 OFFSET 10000;

> -- Keyset — efficient infinite scroll
> SELECT id, title, created_at
> FROM posts
> WHERE (created_at, id) < ('2025-06-01', 12345)  -- cursor from previous page
> ORDER BY created_at DESC, id DESC
> LIMIT 20;
> ```

> [!success] Pros / Cons
> **OFFSET pros:** Jump to any page number easily. **Cons:** Slow and unstable at scale.  
> **Keyset pros:** Constant-time pages, stable cursors. **Cons:** Can't jump to arbitrary page N without scanning; API must pass cursor, not page number.

> [!info] Further study
> - [PostgreSQL — LIMIT/OFFSET](https://www.postgresql.org/docs/current/queries-limit.html)
> - [Use The Index, Luke — Pagination](https://use-the-index-luke.com/no-offset)

---

### 23. PostgreSQL vs MySQL

> [!question] Q23
> When would you choose PostgreSQL over MySQL and vice versa?

Both are excellent production relational databases. The choice depends on workload and team familiarity.

**Choose PostgreSQL when:**
- Complex queries, CTEs, window functions, advanced types (**JSONB**, arrays, GIS/PostGIS)
- Strict data integrity, rich constraints, full-text search built-in
- Need extensibility (custom types, extensions)

**Choose MySQL when:**
- Simple read-heavy web workloads with well-understood patterns
- Team/ecosystem already standardized on MySQL/MariaDB
- Specific managed offerings or replication setup you already operate

In 2025, feature gaps are smaller than a decade ago — both support JSON, replication, and solid performance. **Postgres** tends to win on complex/analytical queries and correctness; **MySQL** on simplicity and widespread LAMP-era deployment familiarity.

> [!example]
> ```sql
> -- PostgreSQL JSONB query (native, indexed)
> SELECT data->>'title' AS title
> FROM products
> WHERE data @> '{"category": "books"}';

> -- MySQL JSON (also supported)
> SELECT JSON_UNQUOTE(data->'$.title') AS title
> FROM products
> WHERE JSON_EXTRACT(data, '$.category') = 'books';
> ```

> [!success] Pros / Cons
> **PostgreSQL pros:** Feature-rich, standards-compliant, JSONB, extensions. **Cons:** Slightly steeper tuning curve for some admin tasks.  
> **MySQL pros:** Ubiquitous, fast simple reads, huge hosting support. **Cons:** Historically weaker on complex analytics (improving); dialect differences from standard SQL.

> [!info] Further study
> - [PostgreSQL Documentation](https://www.postgresql.org/docs/)
> - [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
> - [DB-Engines Ranking](https://db-engines.com/en/ranking) — popularity trends

---

## Related notes

- [[05-databases]] — General DB concepts: ACID, CAP, replication, caching, N+1
- [[16-mongodb]] — NoSQL alternative; when to leave SQL for documents
- [[06-system-design]] — Database scaling, sharding, and read replicas in system design
- [[questions/15-sql]] — Question list (companion to this answer note)

---

## References & Further Study

### PostgreSQL
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL — SELECT](https://www.postgresql.org/docs/current/sql-select.html)
- [PostgreSQL — Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [PostgreSQL — Window functions](https://www.postgresql.org/docs/current/functions-window.html)
- [PostgreSQL — Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL — Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)

### MySQL
- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
- [MySQL — JOINs](https://dev.mysql.com/doc/refman/8.0/en/join.html)
- [MySQL — InnoDB transaction isolation](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)

### Query design & performance
- [Use The Index, Luke](https://use-the-index-luke.com/) — indexes, joins, pagination
- [SQL Style Guide](https://www.sqlstyle.guide/) — readable, maintainable SQL
- [Mode — SQL Tutorial](https://mode.com/sql-tutorial/) — practical analytics SQL

### Concepts
- [Martin Fowler — OLAP vs OLTP (enterprise patterns)](https://martinfowler.com/eaaDev/) — enterprise application patterns
- [Wikipedia — Database normalization](https://en.wikipedia.org/wiki/Database_normalization)
