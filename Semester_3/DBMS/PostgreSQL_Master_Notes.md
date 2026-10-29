# 🐘 PostgreSQL Complete Master Notes (All-in-One Guide)

> **Comprehensive, Zero-to-Hero Guide to PostgreSQL Architecture, Advanced SQL Features, PL/pgSQL, Administration & Full-Stack Integration**  
> *Specially crafted for College Exams (Semester 3 DBMS), Technical Interviews, and Production Engineering.*

---

## 📑 Table of Contents

1. [Introduction to PostgreSQL & Architecture](#1-introduction-to-postgresql--architecture)
2. [SQL vs PostgreSQL: The Definitive Distinction](#2-sql-vs-postgresql-the-definitive-distinction)
3. [PostgreSQL vs MySQL Feature Matrix](#3-postgresql-vs-mysql-feature-matrix)
4. [Installation & Environment Setup](#4-installation--environment-setup)
5. [Client Tools & CLI Reference (`psql` Meta-Commands)](#5-client-tools--cli-reference-psql-meta-commands)
6. [PostgreSQL Data Types & Modern Constraints](#6-postgresql-data-types--modern-constraints)
   - [Modern Constraints](#modern-constraints)
   - [Auto-Increment: `SERIAL` vs `IDENTITY`](#auto-increment-serial-vs-identity)
   - [UUID (Universally Unique Identifier)](#uuid-universally-unique-identifier)
   - [Native Arrays](#native-arrays)
   - [JSON vs JSONB (Binary JSON)](#json-vs-jsonb-binary-json)
   - [Custom Types & ENUMs](#custom-types--enums)
   - [Range Data Types](#range-data-types)
7. [PostgreSQL-Specific SQL Superpowers](#7-postgresql-specific-sql-superpowers)
   - [RETURNING Clause](#returning-clause)
   - [ILIKE (Case-Insensitive Matching)](#ilike-case-insensitive-matching)
   - [ON CONFLICT (UPSERT)](#on-conflict-upsert)
   - [DISTINCT ON](#distinct-on)
   - [COALESCE & NULLIF](#coalesce--nullif)
8. [Date & Time Management in PostgreSQL](#8-date--time-management-in-postgresql)
   - [TIMESTAMP & TIMESTAMPTZ](#timestamp--timestamptz)
   - [INTERVAL Arithmetic](#interval-arithmetic)
   - [EXTRACT(), DATE_TRUNC() & TO_CHAR()](#extract-date_trunc--to_char)
   - [Date & Time Practice Solutions](#date--time-practice-solutions)
9. [Common Table Expressions (CTEs - `WITH` Clause)](#9-common-table-expressions-ctes---with-clause)
   - [Basic CTE & Aggregations](#basic-cte--aggregations)
   - [Multiple & Chained CTEs](#multiple--chained-ctes)
   - [Writable (Data-Modifying) CTEs](#writable-data-modifying-ctes)
   - [Recursive CTEs (Hierarchies & Graphs)](#recursive-ctes-hierarchies--graphs)
10. [Window Functions in PostgreSQL](#10-window-functions-in-postgresql)
    - [Window Function vs GROUP BY](#window-function-vs-group-by)
    - [PARTITION BY & ORDER BY Frame](#partition-by--order-by-frame)
    - [Ranking: ROW_NUMBER(), RANK(), DENSE_RANK()](#ranking-row_number-rank-dense_rank)
    - [Value Functions: LAG(), LEAD(), FIRST_VALUE()](#value-functions-lag-lead-first_value)
11. [Stored Procedures & User-Defined Functions (PL/pgSQL)](#11-stored-procedures--user-defined-functions-plpgsql)
    - [Procedure vs Function Comparison](#procedure-vs-function-comparison)
    - [PL/pgSQL Syntax & Control Structures](#plpgsql-syntax--control-structures)
    - [Complete CRUD Implementation](#complete-crud-implementation)
12. [Advanced Database Administration & Performance Tuning](#12-advanced-database-administration--performance-tuning)
    - [Specialized Indexes (B-tree, Hash, GIN, GiST, BRIN)](#specialized-indexes-b-tree-hash-gin-gist-brin)
    - [Partial & Expression Indexes](#partial--expression-indexes)
    - [Built-in Full-Text Search (tsvector, tsquery)](#built-in-full-text-search-tsvector-tsquery)
    - [Extension Ecosystem (`pgcrypto`, `PostGIS`)](#extension-ecosystem-pgcrypto-postgis)
    - [MVCC (Multi-Version Concurrency Control)](#mvcc-multi-version-concurrency-control)
    - [VACUUM & ANALYZE (Storage Reclamation)](#vacuum--analyze-storage-reclamation)
13. [Production Full-Stack Integration: PostgreSQL + Node.js](#13-production-full-stack-integration-postgresql--nodejs)
    - [Connection Pooling with `pg`](#connection-pooling-with-pg)
    - [REST API Endpoints with Express.js](#rest-api-endpoints-with-expressjs)
    - [SQL Injection Prevention with Parameterized Queries](#sql-injection-prevention-with-parameterized-queries)
    - [User Authentication (Bcrypt + JWT)](#user-authentication-bcrypt--jwt)
14. [Top PostgreSQL Interview Questions & Quick Reference](#14-top-postgresql-interview-questions--quick-reference)

---

# 1. Introduction to PostgreSQL & Architecture

### What is PostgreSQL?
**PostgreSQL** (often called simply **Postgres**) is a powerful, free, and open-source **Object-Relational Database Management System (ORDBMS)** with over 35 years of active development. It is known for its reliability, feature robustness, and strict adherence to ANSI SQL standards.

```
                            POSTGRESQL HIERARCHY
                            ┌─────────────────────┐
                            │  PostgreSQL Server  │ (Port: 5432)
                            └──────────┬──────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    ▼                                     ▼
             Database 1 (e.g. college)             Database 2
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 Schema: public          Schema: analytics
        │
   ┌────┴───────────────────────────┐
   ▼                                ▼
Tables (Students, Courses)       Views, Functions, Triggers
```

### Core Architecture Concepts
1. **Server (Cluster):** A single running instance of PostgreSQL managing physical storage files and listening on a network port (default `5432`).
2. **Database:** A logical container for tables and objects. Databases are completely isolated from each other.
3. **Schema:** A logical namespace within a database (default is `public`). Think of schemas as folders inside a database.
4. **Roles / Users:** PostgreSQL unifies users and groups into "Roles" with granular permissions.

---

# 2. SQL vs PostgreSQL: The Definitive Distinction

A common confusion among learners:
- **SQL (Structured Query Language):** The **standardized language** specification defined by ISO/ANSI (like the English grammar rules).
- **PostgreSQL:** The **database software engine** that reads, parses, optimizes, and executes SQL statements against persistent storage (like a person who speaks English, plus knows specialized technical slang).

```
   SQL (Language / Standard)
   ├── Supported by MySQL
   ├── Supported by Oracle
   ├── Supported by SQL Server
   └── Implemented by PostgreSQL
            │
            └── PLUS PostgreSQL Extensions:
                (JSONB, Arrays, UUID, ILIKE, RETURNING, ON CONFLICT, PL/pgSQL, GIN/GiST)
```

---

# 3. PostgreSQL vs MySQL Feature Matrix

| Feature | MySQL | PostgreSQL |
|---|---|---|
| **DBMS Classification** | Pure Relational (RDBMS) | Object-Relational (ORDBMS) |
| **Default Network Port** | `3306` | `5432` |
| **SQL Standards Compliance** | Partial / Pragmatic | Extremely Strict ANSI SQL Compliance |
| **JSON Handling** | Binary JSON document | Native `JSON` + Optimized Binary `JSONB` |
| **Arrays** | No native array type | Native first-class arrays (`TEXT[]`, `INT[]`) |
| **Case-Insensitive Search**| `LIKE` is case-insensitive by default in some collations | Explicit `ILIKE` operator |
| **Returning Modified Rows**| Not natively supported | Native `RETURNING *` for INSERT/UPDATE/DELETE |
| **UPSERT Syntax** | `INSERT ... ON DUPLICATE KEY UPDATE` | `INSERT ... ON CONFLICT (...) DO UPDATE / NOTHING` |
| **Table DDL in Transactions**| DDL triggers implicit commit (Cannot rollback DDL) | DDL is transactional (Can rollback `CREATE TABLE`, `DROP`) |
| **Concurrency Control** | MVCC + InnoDB Row Locks | Advanced MVCC without read-locks |
| **CLI Tool** | `mysql` | `psql` |
| **GUI Tool** | MySQL Workbench | pgAdmin 4 / DBeaver |

---

# 4. Installation & Environment Setup

### Windows Installation Steps:
1. Download official installer from: [postgresql.org/download/windows](https://www.postgresql.org/download/windows/)
2. Run `.exe` installer.
3. Select Components:
   - ✅ **PostgreSQL Server**
   - ✅ **pgAdmin 4** (Web-based GUI client)
   - ✅ **Command Line Tools** (`psql`, `pg_dump`, `createdb`)
4. Data Directory: Keep default (`C:\Program Files\PostgreSQL\<version>\data`).
5. **Superuser Password:** Enter a secure password for the `postgres` superuser (remember this password!).
6. **Port:** Keep default `5432`.
7. Finish installation. PostgreSQL automatically starts as a persistent Windows Service (`postgresql-x64-<version>`).

### Verify Installation:
Open PowerShell or Command Prompt:
```bash
psql --version
# Output: psql (PostgreSQL) 16.x or 17.x
```

Test query via CLI:
```bash
psql -U postgres
# Enter your superuser password
postgres=# SELECT version();
postgres=# \q
```

---

# 5. Client Tools & CLI Reference (`psql` Meta-Commands)

PostgreSQL's command-line interface `psql` includes powerful **meta-commands** (starting with backslash `\`) that are faster than writing SQL metadata queries:

```
                            ESSENTIAL psql CHEAT SHEET
 ┌───────────────┬────────────────────────────────────────────────────────┐
 │ Meta-Command  │ Purpose & Description                                  │
 ├───────────────┼────────────────────────────────────────────────────────┤
 │ \l            │ List all databases on the server                       │
 │ \c <dbname>   │ Connect to / switch to a specific database             │
 │ \dt           │ List all tables in the current schema (public)         │
 │ \dt *.*       │ List all tables across ALL schemas                     │
 │ \d <table_name> Describe table columns, types, nullability, constraints │
 │ \d+ <table>   │ Detailed description including storage & index info    │
 │ \dn           │ List all schemas in the current database               │
 │ \du           │ List all users, roles, and their privileges            │
 │ \df           │ List all stored functions and procedures               │
 │ \dv           │ List all views                                         │
 │ \di           │ List all indexes                                       │
 │ \x            │ Toggle expanded display (vertical output, great for CLI)│
 │ \timing       │ Toggle query execution timing in milliseconds          │
 │ \i <file.sql> │ Execute SQL commands from a local script file          │
 │ \q            │ Quit / Exit psql session                               │
 └───────────────┴────────────────────────────────────────────────────────┘
```

---

# 6. PostgreSQL Data Types & Modern Constraints

## Modern Constraints
Constraints enforce business rules and data consistency directly at the storage engine level:
```sql
CREATE TABLE Users (
    user_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    age INT CHECK (age >= 18),
    city VARCHAR(50) DEFAULT 'Ahmedabad'
);
```

---

## Auto-Increment: `SERIAL` vs `IDENTITY`
For years, PostgreSQL developers used the pseudo-type `SERIAL`. Modern PostgreSQL (version 10+) introduced standard SQL `IDENTITY` columns.

```sql
-- 1. Legacy Approach: SERIAL
CREATE TABLE Orders_Legacy (
    order_id SERIAL PRIMARY KEY,
    order_amount NUMERIC(10, 2)
);

-- 2. Modern ANSI Standard: IDENTITY (Recommended for all new projects!)
CREATE TABLE Orders_Modern (
    order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_amount NUMERIC(10, 2)
);
```

### `SERIAL` vs `IDENTITY` In-Depth Comparison:
| Feature | `SERIAL` (Legacy) | `IDENTITY` (Modern ANSI) |
|---|---|---|
| **Mechanism** | Creates an invisible sequence behind the scenes | Implements ANSI SQL:2006 standard identity |
| **Manual Override** | Easily bypassed if user passes an ID manually | `GENERATED ALWAYS` prevents accidental manual insertion |
| **Dropping Column**| Sequence ownership can become orphaned if dropped | Sequence is strictly tied to column life cycle |
| **Portability** | PostgreSQL proprietary | Fully portable across Oracle, DB2, and SQL Server |

---

## UUID (Universally Unique Identifier)
A **UUID** is a 128-bit number formatted as 32 hexadecimal digits (`550e8400-e29b-41d4-a716-446655440000`).

### Why use UUIDs instead of auto-incrementing integers?
1. **Microservices & Distributed Systems:** Multiple distributed nodes can generate IDs independently without coordinating with a central database.
2. **Security:** Sequential IDs (`/users/101`, `/users/102`) allow attackers to guess user counts and enumerate resources. UUIDs (`/users/c73295c1-...`) prevent enumeration attacks.
3. **Seamless Merging:** Data merged from multiple environments (staging $\rightarrow$ production) never collides.

### Enabling UUID in PostgreSQL:
```sql
-- Step 1: Enable pgcrypto extension (if not already enabled)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Step 2: Use gen_random_uuid() for default primary key
CREATE TABLE CustomerProfiles (
    customer_id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL
);

-- Step 3: Insert without specifying ID
INSERT INTO CustomerProfiles (full_name, email)
VALUES ('Adil Ahmad', 'adil@example.com')
RETURNING *;
```

---

## Native Arrays
PostgreSQL has first-class support for storing multi-element arrays in single table columns:

```sql
CREATE TABLE Developers (
    dev_id SERIAL PRIMARY KEY,
    dev_name VARCHAR(100),
    skills TEXT[],            -- Array of strings
    test_scores INT[]         -- Array of integers
);

-- Insert into arrays:
INSERT INTO Developers (dev_name, skills, test_scores)
VALUES
('Rahul', ARRAY['PostgreSQL', 'Node.js', 'React'], '{95, 88, 92}'),
('Priya', ARRAY['Python', 'PostgreSQL', 'Docker'], '{90, 94, 91}');

-- Query: Check if an array contains a specific element
SELECT dev_name, skills 
FROM Developers 
WHERE 'PostgreSQL' = ANY(skills);

-- Query: Array containment operator (@>)
SELECT dev_name 
FROM Developers 
WHERE skills @> ARRAY['Docker'];
```

---

## JSON vs JSONB (Binary JSON)
PostgreSQL supports two JSON data types:

| Metric | `JSON` | `JSONB` (Binary JSON) |
|---|---|---|
| **Storage** | Exact raw text copy of input JSON | Decomposed binary format with duplicate keys stripped |
| **Insert Speed** | Faster (No parsing or indexing required) | Slightly slower (parses JSON tree on insertion) |
| **Query Speed** | Slower (Re-parses text on every query) | **Extremely fast** (pre-parsed binary lookup) |
| **Index Support** | Limited | Supports specialized **GIN** indexes for sub-millisecond search |
| **Recommendation**| Use only if exact whitespace preservation is vital | **Default choice for 99% of production applications** |

```sql
CREATE TABLE ProductsCatalog (
    product_id SERIAL PRIMARY KEY,
    title VARCHAR(150),
    attributes JSONB
);

INSERT INTO ProductsCatalog (title, attributes)
VALUES (
    'Gaming Laptop', 
    '{"brand": "ASUS", "ram_gb": 16, "storage": {"ssd_gb": 512}, "tags": ["gaming", "sale"]}'
);

-- JSONB Query Operators:
-- 1. -> extracts as JSON object
SELECT title, attributes->'storage' FROM ProductsCatalog;

-- 2. ->> extracts as text string
SELECT title, attributes->>'brand' AS Brand FROM ProductsCatalog;

-- 3. Path navigation
SELECT title, attributes->'storage'->>'ssd_gb' AS SSD_Size FROM ProductsCatalog;

-- 4. Check if key exists (?)
SELECT * FROM ProductsCatalog WHERE attributes ? 'brand';
```

---

## Custom Types & ENUMs
PostgreSQL allows developers to define custom application domain types:

```sql
-- 1. Create an ENUM type
CREATE TYPE user_status AS ENUM ('pending', 'active', 'suspended', 'deleted');

CREATE TABLE Accounts (
    account_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR(50),
    status user_status DEFAULT 'pending'
);

-- 2. Create a Composite Type (Record structure)
CREATE TYPE address_type AS (
    street VARCHAR(100),
    city VARCHAR(50),
    pincode VARCHAR(10)
);

CREATE TABLE Branches (
    branch_id SERIAL PRIMARY KEY,
    branch_name VARCHAR(100),
    location address_type
);
```

---

## Range Data Types
Instead of storing two separate columns (`start_date`, `end_date`), PostgreSQL provides native **Range Types**:
- `int4range`, `numrange`, `daterange`, `tsrange`

```sql
CREATE TABLE HotelBookings (
    booking_id SERIAL PRIMARY KEY,
    room_number INT,
    stay_duration DATERANGE
);

INSERT INTO HotelBookings (room_number, stay_duration)
VALUES (101, '[2026-11-01, 2026-11-07)');

-- Overlap Operator (&&): Find bookings overlapping with a requested date range!
SELECT * FROM HotelBookings 
WHERE stay_duration && '[2026-11-05, 2026-11-10)'::DATERANGE;
```

---

# 7. PostgreSQL-Specific SQL Superpowers

### 1. `RETURNING` Clause
In traditional SQL databases, after executing an `INSERT` or `UPDATE`, a second query is required to fetch the generated ID or timestamps. In PostgreSQL, `RETURNING` sends modified rows directly back to the client!

```sql
-- Returns the newly generated identity ID and admission date immediately:
INSERT INTO Students (student_name, age, city)
VALUES ('Adil', 20, 'Ahmedabad')
RETURNING student_id, student_name;

-- Return all modified fields upon updating:
UPDATE Students
SET marks = marks + 5
WHERE department_id = 1
RETURNING student_id, student_name, marks;

-- Return deleted records for audit logging:
DELETE FROM Students
WHERE age < 18
RETURNING *;
```

---

### 2. `ILIKE` (Case-Insensitive Matching)
Standard SQL `LIKE` is case-sensitive (`'adil'` does not match `'Adil'`). PostgreSQL provides `ILIKE` for hassle-free case-insensitive pattern matching:

```sql
SELECT * FROM Students 
WHERE student_name ILIKE 'a%'; -- Matches 'Adil', 'ankit', 'AMIT'
```

---

### 3. `ON CONFLICT` (UPSERT)
**UPSERT** (Update or Insert) handles duplicate key errors gracefully without failing the transaction:

```sql
-- Scenario: Insert a student; if email already exists, do NOT fail:

-- Option A: Do Nothing
INSERT INTO Students (email, student_name, city)
VALUES ('adil@example.com', 'Adil Ahmad', 'Ahmedabad')
ON CONFLICT (email) 
DO NOTHING;

-- Option B: Do Update (Update city and name if email exists)
INSERT INTO Students (email, student_name, city)
VALUES ('adil@example.com', 'Adil Ahmad', 'Mumbai')
ON CONFLICT (email) 
DO UPDATE SET 
    student_name = EXCLUDED.student_name,
    city = EXCLUDED.city;
```
*(Here `EXCLUDED` represents the proposed row that caused the conflict).*

---

### 4. `DISTINCT ON`
Standard SQL `DISTINCT` applies across all selected columns. PostgreSQL's `DISTINCT ON` eliminates duplicate rows based only on specified leading expressions, paired with `ORDER BY`:

```sql
-- Find the highest-scoring student from EACH department in a single query:
SELECT DISTINCT ON (department_id) 
       department_id, student_name, marks
FROM Students
ORDER BY department_id, marks DESC;
```

---

### 5. `COALESCE` & `NULLIF`
- `COALESCE(val1, val2, ...)`: Returns the first non-NULL value in the argument list.
  ```sql
  SELECT student_name, COALESCE(phone, 'No Phone Provided') AS Contact
  FROM Students;
  ```
- `NULLIF(val1, val2)`: Returns `NULL` if $val1 = val2$. Useful to prevent **Division by Zero**:
  ```sql
  SELECT total_marks / NULLIF(total_subjects, 0) AS AverageMarks;
  ```

---

# 8. Date & Time Management in PostgreSQL

PostgreSQL has one of the most powerful date/time engines in the database industry.

```
                           DATE & TIME SYSTEM
 ┌───────────────────────┬────────────────────────────────────────────────┐
 │ NOW()                 │ Returns current timestamp with timezone        │
 │ CURRENT_TIMESTAMP     │ Standard SQL equivalent of NOW()               │
 │ CURRENT_DATE          │ Returns today's date (YYYY-MM-DD)              │
 │ CURRENT_TIME          │ Returns current time of day                    │
 └───────────────────────┴────────────────────────────────────────────────┘
```

---

### INTERVAL Arithmetic
Add or subtract periods using natural language units:

```sql
-- Date Addition & Subtraction
SELECT NOW() + INTERVAL '7 days' AS OneWeekLater;
SELECT NOW() - INTERVAL '1 month' AS OneMonthAgo;
SELECT NOW() + INTERVAL '2 years 3 months';
SELECT NOW() + INTERVAL '4 hours 30 minutes 15 seconds';

-- Find orders placed in the last 24 hours:
SELECT * FROM Orders 
WHERE order_date >= NOW() - INTERVAL '24 hours';
```

---

### `EXTRACT()`, `DATE_TRUNC()` & `TO_CHAR()`

```sql
-- 1. EXTRACT: Pull individual integer parts
SELECT EXTRACT(YEAR FROM NOW()) AS CurrentYear;
SELECT EXTRACT(MONTH FROM order_date) AS OrderMonth FROM Orders;
SELECT EXTRACT(DOW FROM NOW()) AS DayOfWeek; -- 0 = Sunday, 1 = Monday ...

-- 2. DATE_TRUNC: Truncate timestamp to specific precision
SELECT DATE_TRUNC('month', NOW()); -- Resets to first day of current month: YYYY-MM-01 00:00:00
SELECT DATE_TRUNC('day', NOW());   -- Resets to midnight today

-- 3. TO_CHAR: Custom string formatting
SELECT TO_CHAR(NOW(), 'DD-Mon-YYYY HH12:MI:SS AM') AS FormattedDate;
-- Output: 29-Oct-2026 08:30:15 PM
SELECT TO_CHAR(NOW(), 'Day, Month YYYY') AS FullDate;
```

---

### Date & Time Practice Solutions

```sql
-- Q1: Total orders placed in the month of June 2026:
SELECT COUNT(*) 
FROM Orders
WHERE EXTRACT(YEAR FROM order_date) = 2026 
  AND EXTRACT(MONTH FROM order_date) = 6;

-- Q2: Find delivery date scheduled exactly 5 business days later:
SELECT order_id, order_date, order_date + INTERVAL '5 days' AS ExpectedDelivery
FROM Orders;

-- Q3: Calculate student age in exact years from DateOfBirth:
SELECT student_name, 
       EXTRACT(YEAR FROM AGE(NOW(), date_of_birth)) AS AgeInYears
FROM Students;
```

---

# 9. Common Table Expressions (CTEs - `WITH` Clause)

A **Common Table Expression (CTE)** defines a temporary, named result set that exists only for the duration of a single SQL query. CTEs make complex multi-step queries clean, readable, and modular.

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

---

### 1. Basic CTE with Aggregations & Filtering
```sql
-- Find departments where the average student marks exceed 80:
WITH DeptAverages AS (
    SELECT department_id, 
           COUNT(*) AS TotalStudents,
           AVG(marks) AS AvgMarks
    FROM Students
    GROUP BY department_id
)
SELECT d.department_name, da.TotalStudents, ROUND(da.AvgMarks, 2)
FROM DeptAverages da
INNER JOIN Departments d ON da.department_id = d.department_id
WHERE da.AvgMarks > 80;
```

---

### 2. Multiple / Chained CTEs
You can chain multiple CTEs by separating them with commas:
```sql
WITH HighScorers AS (
    SELECT student_id, student_name, department_id, marks
    FROM Students
    WHERE marks >= 85
),
DeptSummary AS (
    SELECT department_id, COUNT(*) AS StarStudents
    FROM HighScorers
    GROUP BY department_id
)
SELECT d.department_name, COALESCE(ds.StarStudents, 0) AS StarCount
FROM Departments d
LEFT JOIN DeptSummary ds ON d.department_id = ds.department_id;
```

---

### 3. Writable (Data-Modifying) CTEs
In PostgreSQL, CTEs can contain `INSERT`, `UPDATE`, or `DELETE` with `RETURNING`:

```sql
-- Archive deleted inactive students into an archive table in a single atomic transaction:
WITH DeletedStudents AS (
    DELETE FROM Students
    WHERE age < 18
    RETURNING student_id, student_name, marks
)
INSERT INTO StudentArchives (student_id, student_name, marks)
SELECT student_id, student_name, marks FROM DeletedStudents;
```

---

### 4. Recursive CTEs (Hierarchies & Sequences)
A recursive CTE references itself to solve hierarchical or graph traversal problems:

```sql
-- Generate numbers 1 to 10 without loops:
WITH RECURSIVE NumberSeries AS (
    SELECT 1 AS n -- Base query
    UNION ALL
    SELECT n + 1 FROM NumberSeries WHERE n < 10 -- Recursive step
)
SELECT * FROM NumberSeries;

-- Traverse an Employee-Manager hierarchy tree:
WITH RECURSIVE OrgChart AS (
    -- Anchor member: The CEO (manager_id IS NULL)
    SELECT emp_id, emp_name, manager_id, 1 AS OrgLevel
    FROM Employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive member: Find all direct reports
    SELECT e.emp_id, e.emp_name, e.manager_id, o.OrgLevel + 1
    FROM Employees e
    INNER JOIN OrgChart o ON e.manager_id = o.emp_id
)
SELECT * FROM OrgChart ORDER BY OrgLevel, emp_name;
```

---

# 10. Window Functions in PostgreSQL

A **Window Function** performs calculations across a set of table rows related to the current row, **without collapsing the rows into a single summary output**.

```
   GROUP BY:                           WINDOW FUNCTION (OVER):
   Input 10 rows ──► Returns 2 rows    Input 10 rows ──► Returns 10 rows
   (Rows collapsed per group)          (Every row retains identity + group calculation)
```

---

### Core Syntax
```sql
function_name() OVER (
    [PARTITION BY partition_column]
    [ORDER BY sort_column ASC/DESC]
    [ROWS / RANGE frame_specification]
)
```

---

### 1. Ranking Functions: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`
Imagine three students scoring: `95, 95, 80`:

| Function | Output for 95 | Output for 95 | Output for 80 | Behavior |
|---|:---:|:---:|:---:|---|
| `ROW_NUMBER()` | `1` | `2` | `3` | Strictly sequential unique integers; breaks ties arbitrarily |
| `RANK()` | `1` | `1` | `3` | Equal scores get identical rank; **leaves gaps** |
| `DENSE_RANK()` | `1` | `1` | `2` | Equal scores get identical rank; **NO gaps** |

```sql
SELECT student_name, marks, department_id,
       ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY marks DESC) AS RowNum,
       RANK()       OVER(PARTITION BY department_id ORDER BY marks DESC) AS RankNum,
       DENSE_RANK() OVER(PARTITION BY department_id ORDER BY marks DESC) AS DenseRankNum
FROM Students;
```

---

### 2. Value / Offset Functions: `LAG()` & `LEAD()`
- `LAG(col, offset)`: Accesses data from previous rows without performing a self-join.
- `LEAD(col, offset)`: Accesses data from upcoming future rows.

```sql
-- Compare each student's marks with the student ranked immediately above them:
SELECT student_name, marks,
       LAG(marks, 1) OVER(ORDER BY marks DESC) AS PreviousStudentMarks,
       marks - LAG(marks, 1) OVER(ORDER BY marks DESC) AS DifferenceFromAbove
FROM Students;
```

---

### 3. Running Totals & Moving Averages
```sql
-- Compute running cumulative revenue:
SELECT order_id, order_date, amount,
       SUM(amount) OVER(ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RunningTotal
FROM Orders;
```

---

# 11. Stored Procedures & User-Defined Functions (PL/pgSQL)

PostgreSQL provides **PL/pgSQL** (Procedural Language / PostgreSQL), allowing variables, loops, conditionals, and error handling inside the database.

---

### Stored Procedure vs Function

| Dimension | Stored Procedure (`PROCEDURE`) | User-Defined Function (`FUNCTION`) |
|---|---|---|
| **Invocation** | `CALL procedure_name(args);` | `SELECT function_name(args);` |
| **Return Value** | Does not return data directly | **Must** declare `RETURNS <type>` |
| **Transactions** | Can manage transactions (`COMMIT`, `ROLLBACK` inside!) | Cannot manage transactions internally |
| **Primary Use** | Performing batch actions & administrative tasks | Computations, queries, returning virtual tables |

---

### Complete CRUD Implementation in PL/pgSQL

#### 1. CREATE: Stored Procedure to Add a Student
```sql
CREATE OR REPLACE PROCEDURE add_student(
    p_name VARCHAR,
    p_gender VARCHAR,
    p_age INT,
    p_city VARCHAR,
    p_marks DECIMAL,
    p_dept_id INT
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO Students (student_name, gender, age, city, marks, department_id)
    VALUES (p_name, p_gender, p_age, p_city, p_marks, p_dept_id);
    
    RAISE NOTICE 'Student % successfully registered.', p_name;
END;
$$;

-- Invoke procedure:
CALL add_student('Arun Kumar', 'Male', 21, 'Chennai', 89.5, 1);
```

#### 2. READ: Function Returning a Table (`RETURNS TABLE`)
```sql
CREATE OR REPLACE FUNCTION get_students_by_dept(p_dept_id INT)
RETURNS TABLE (
    sid INT,
    sname VARCHAR,
    smarks DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT student_id, student_name, marks
    FROM Students
    WHERE department_id = p_dept_id;
END;
$$;

-- Call function:
SELECT * FROM get_students_by_dept(1);
```

#### 3. UPDATE: Procedure to Update Student Marks
```sql
CREATE OR REPLACE PROCEDURE update_student_marks(
    p_student_id INT,
    p_bonus_marks DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE Students
    SET marks = LEAST(100.00, marks + p_bonus_marks)
    WHERE student_id = p_student_id;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Student ID % does not exist.', p_student_id;
    END IF;
END;
$$;

CALL update_student_marks(101, 5.0);
```

#### 4. DELETE: Procedure to Remove a Student
```sql
CREATE OR REPLACE PROCEDURE delete_student(p_student_id INT)
LANGUAGE plpgsql
AS $$
BEGIN
    DELETE FROM Students WHERE student_id = p_student_id;
END;
$$;

CALL delete_student(120);
```

---

# 12. Advanced Database Administration & Performance Tuning

## Specialized PostgreSQL Index Types
Unlike databases limited to standard B-trees, PostgreSQL provides 6 specialized index engines:

```
                            POSTGRESQL INDEX TYPES
 ┌──────────┬────────────────────────────────────────────────────────┐
 │ B-Tree   │ Default. General equality (=) and range (<, >, BETWEEN)│
 │ Hash     │ Optimized exclusively for exact equality (=) lookups   │
 │ GIN      │ Generalized Inverted Index (JSONB, Arrays, Text Search)│
 │ GiST     │ Generalized Search Tree (Geospatial / Geometric points)│
 │ BRIN     │ Block Range Index (Massive sequential time-series logs)│
 └──────────┴────────────────────────────────────────────────────────┘
```

```sql
-- 1. Standard B-Tree
CREATE INDEX idx_students_marks ON Students(marks);

-- 2. GIN Index on JSONB (Enables sub-millisecond JSON querying!)
CREATE INDEX idx_products_attributes ON ProductsCatalog USING GIN(attributes);

-- 3. GIN Index on Arrays
CREATE INDEX idx_dev_skills ON Developers USING GIN(skills);
```

---

## Partial & Expression Indexes

### 1. Partial Index
Indexes only rows matching a specific `WHERE` filter, saving 90% disk space:
```sql
-- Index only active accounts (ignoring millions of archived accounts):
CREATE INDEX idx_active_users_email 
ON Users(email) 
WHERE status = 'active';
```

### 2. Expression Index
Indexes the output of a function:
```sql
-- Speeds up queries searching by lowercase email:
CREATE INDEX idx_users_lower_email 
ON Users(LOWER(email));

-- The planner will automatically use this index for:
SELECT * FROM Users WHERE LOWER(email) = 'adil@example.com';
```

---

## Built-in Full-Text Search (tsvector, tsquery)
PostgreSQL includes an enterprise-grade search engine without needing external services like Elasticsearch:

- `tsvector`: Pre-processed text document parsed into normalized words (lexemes).
- `tsquery`: Search terms with boolean operators (`&`, `|`, `!`).
- `@@`: Full-text match operator.

```sql
SELECT title, content
FROM Articles
WHERE to_tsvector('english', content) @@ plainto_tsquery('english', 'database indexing');
```

---

## MVCC (Multi-Version Concurrency Control)
PostgreSQL handles high concurrency using **MVCC**:
- When a row is updated, PostgreSQL does **NOT** overwrite the existing row. It marks the old row version as expired and writes a brand new version of the row.
- **Readers never block writers; writers never block readers!**
- Transactions observe a consistent "snapshot" of data from when their query started.

---

## VACUUM & ANALYZE (Storage Reclamation)
Because MVCC writes new row versions, old deleted/updated rows ("dead tuples") accumulate on disk.

```sql
-- 1. VACUUM: Marks space occupied by dead tuples as reusable for future inserts
VACUUM Students;

-- 2. VACUUM ANALYZE: Reclaims space AND updates query planner statistics
VACUUM ANALYZE Students;

-- 3. VACUUM FULL: Re-writes entire table to disk, releasing storage to OS (Locks table!)
VACUUM FULL Students;
```
> 🛡️ PostgreSQL runs a background daemon called **Autovacuum** that automatically vacuums tables based on update/delete thresholds.

---

# 13. Production Full-Stack Integration: PostgreSQL + Node.js

Connecting PostgreSQL with Node.js and Express using the official `pg` library.

### 1. Architecture Setup
```bash
npm init -y
npm install express pg dotenv
npm install --save-dev nodemon
```

---

### 2. Connection Pool (`config/db.js`)
A connection pool maintains a pool of reusable connections, preventing overhead from opening and closing connections on every HTTP request:

```javascript
const { Pool } = require('pg');
require('dotenv').config();

const pool = new Pool({
    user: process.env.DB_USER || 'postgres',
    host: process.env.DB_HOST || 'localhost',
    database: process.env.DB_NAME || 'CollegeDB',
    password: process.env.DB_PASSWORD,
    port: parseInt(process.env.DB_PORT || '5432', 10),
    max: 20, // Max 20 concurrent connections in pool
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
});

pool.on('connect', () => {
    console.log('✅ Connected to PostgreSQL database pool');
});

pool.on('error', (err) => {
    console.error('❌ Unexpected error on idle client', err);
    process.exit(-1);
});

module.exports = pool;
```

---

### 3. REST API Server with Parameterized Queries (`server.js`)
> ⚠️ **CRITICAL SECURITY RULE:** Always use parameterized placeholders (`$1, $2`) to completely eliminate **SQL Injection**!

```javascript
const express = require('express');
const pool = require('./config/db');
require('dotenv').config();

const app = express();
app.use(express.json());

// 1. GET ALL STUDENTS
app.get('/api/students', async (req, res) => {
    try {
        const result = await pool.query(
            'SELECT student_id, student_name, marks, city FROM Students ORDER BY student_id ASC'
        );
        res.status(200).json({ success: true, count: result.rows.length, data: result.rows });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
});

// 2. GET SINGLE STUDENT BY ID (Parameterized query $1)
app.get('/api/students/:id', async (req, res) => {
    try {
        const { id } = req.params;
        const result = await pool.query('SELECT * FROM Students WHERE student_id = $1', [id]);
        
        if (result.rows.length === 0) {
            return res.status(404).json({ success: false, message: 'Student not found' });
        }
        res.status(200).json({ success: true, data: result.rows[0] });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
});

// 3. POST: INSERT NEW STUDENT USING RETURNING *
app.post('/api/students', async (req, res) => {
    try {
        const { name, gender, age, city, marks, department_id } = req.body;
        const insertQuery = `
            INSERT INTO Students (student_name, gender, age, city, marks, department_id)
            VALUES ($1, $2, $3, $4, $5, $6)
            RETURNING *;
        `;
        const values = [name, gender, age, city, marks, department_id];
        const result = await pool.query(insertQuery, values);

        res.status(201).json({ success: true, message: 'Student created', data: result.rows[0] });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
});

// 4. PUT: UPDATE STUDENT MARKS
app.put('/api/students/:id', async (req, res) => {
    try {
        const { id } = req.params;
        const { marks } = req.body;
        const result = await pool.query(
            'UPDATE Students SET marks = $1 WHERE student_id = $2 RETURNING *',
            [marks, id]
        );
        
        if (result.rows.length === 0) {
            return res.status(404).json({ success: false, message: 'Student not found' });
        }
        res.status(200).json({ success: true, data: result.rows[0] });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
});

// 5. DELETE: REMOVE STUDENT
app.delete('/api/students/:id', async (req, res) => {
    try {
        const { id } = req.params;
        const result = await pool.query('DELETE FROM Students WHERE student_id = $1 RETURNING student_id', [id]);
        
        if (result.rows.length === 0) {
            return res.status(404).json({ success: false, message: 'Student not found' });
        }
        res.status(200).json({ success: true, message: 'Student deleted successfully' });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));
```

---

# 14. Top PostgreSQL Interview Questions & Quick Reference

### Q1. What is the fundamental difference between `JSON` and `JSONB` in PostgreSQL?
**Answer:** `JSON` stores data as exact raw text, preserving whitespace and key order, but queries are slower because the text must be re-parsed each time. `JSONB` parses the text into a decomposed binary representation upon insertion; it removes duplicate keys and whitespace, enables high-speed lookups, and supports **GIN** indexing.

---

### Q2. How does `ILIKE` differ from standard SQL `LIKE`?
**Answer:** `LIKE` is case-sensitive (`'adil'` does not match `'Adil'`). `ILIKE` is a PostgreSQL-specific extension providing case-insensitive pattern matching.

---

### Q3. Explain how UPSERT works in PostgreSQL.
**Answer:** PostgreSQL uses the `INSERT INTO ... ON CONFLICT (target_column) DO UPDATE / DO NOTHING` clause. When a unique or primary key violation occurs on `target_column`, instead of aborting the transaction with an error, it redirects execution to the alternative branch, referencing incoming candidate values via the `EXCLUDED` pseudo-table.

---

### Q4. What is `DISTINCT ON` in PostgreSQL?
**Answer:** While standard ANSI `DISTINCT` eliminates duplicates considering the entire `SELECT` projection, PostgreSQL's `DISTINCT ON (col1, col2)` keeps only the very first row for each distinct set of values in the parenthesized columns, according to the order defined in the `ORDER BY` clause.

---

### Q5. What is MVCC and why does PostgreSQL require `VACUUM`?
**Answer:** Multi-Version Concurrency Control (MVCC) ensures that readers do not block writers and writers do not block readers by maintaining multiple versions of rows concurrently. When a row is updated or deleted, the old version ("dead tuple") remains on disk until transactions that could see it finish. `VACUUM` scans tables, marks dead tuples as reusable space for future inserts, and prevents table bloat.

---

### 🐘 PostgreSQL Quick Revision Master Table

| # | Feature | Standard SQL | PostgreSQL Implementation |
|---|---|---|---|
| 1 | **Identity Key** | `GENERATED ALWAYS AS IDENTITY` | Fully Supported + `SERIAL` convenience type |
| 2 | **Returning Rows** | No universal syntax | `RETURNING *` with INSERT, UPDATE, DELETE |
| 3 | **Case-Insensitive** | Vendor dependent | Native `ILIKE` operator |
| 4 | **Upsert** | Varies | `ON CONFLICT (...) DO UPDATE / NOTHING` |
| 5 | **Binary JSON** | Non-standard | First-class `JSONB` data type with GIN indexes |
| 6 | **Arrays** | Varies by DBMS | Native support: `TEXT[]`, `INT[]`, `@>` operator |
| 7 | **UUID** | Non-standard | Native `UUID` type + `gen_random_uuid()` |
| 8 | **Procedural Logic** | Varies | Built-in `PL/pgSQL` engine |
| 9 | **Common Table Expr** | Supported | Full support including recursive & writable CTEs |
| 10 | **Specialized Indexes** | Typically B-tree | B-Tree, Hash, GIN, GiST, SP-GiST, BRIN |
| 11 | **Storage Cleanup** | Vendor specific | `VACUUM`, `VACUUM ANALYZE`, `autovacuum` |
| 12 | **Client Interface** | Command line | `psql` with `\l`, `\c`, `\dt`, `\d`, `\du` |
