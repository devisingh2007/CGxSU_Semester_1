# 📘 SQL Complete Master Notes (All-in-One Guide)

> **Comprehensive, Zero-to-Hero Guide to Structured Query Language (SQL)**  
> *Specially crafted for College Exams (Semester 3 DBMS), Technical Interviews, and Real-World Backend Development.*

---

## 📑 Table of Contents

1. [Introduction to SQL & Relational Databases](#1-introduction-to-sql--relational-databases)
2. [SQL Command Categories (The 5 Sub-Languages)](#2-sql-command-categories-the-5-sub-languages)
   - [DDL (Data Definition Language)](#ddl-data-definition-language)
   - [DML (Data Manipulation Language)](#dml-data-manipulation-language)
   - [DQL (Data Query Language)](#dql-data-query-language)
   - [DCL (Data Control Language)](#dcl-data-control-language)
   - [TCL (Transaction Control Language)](#tcl-transaction-control-language)
3. [SQL Data Types](#3-sql-data-types)
4. [DROP vs TRUNCATE vs DELETE](#4-drop-vs-truncate-vs-delete)
5. [SQL Clauses & Query Construction](#5-sql-clauses--query-construction)
   - [SELECT & FROM](#select--from)
   - [WHERE Clause & Operators](#where-clause--operators)
   - [ORDER BY (Sorting)](#order-by-sorting)
   - [LIMIT & OFFSET (Pagination)](#limit--offset-pagination)
   - [GROUP BY & Aggregate Functions](#group-by--aggregate-functions)
   - [HAVING Clause (Filtering Groups)](#having-clause-filtering-groups)
   - [DISTINCT Clause](#distinct-clause)
   - [Aliases (AS)](#aliases-as)
   - [LIKE Operator & Pattern Matching (Wildcards)](#like-operator--pattern-matching-wildcards)
6. [SQL Logical Query Execution Order](#6-sql-logical-query-execution-order)
7. [SQL String Functions & Conditional Expressions (`CASE`)](#7-sql-string-functions--conditional-expressions-case)
8. [SQL JOINs (Complete Multi-Table Relationships)](#8-sql-joins-complete-multi-table-relationships)
   - [INNER JOIN](#inner-join)
   - [LEFT (OUTER) JOIN](#left-outer-join)
   - [RIGHT (OUTER) JOIN](#right-outer-join)
   - [FULL OUTER JOIN](#full-outer-join)
   - [CROSS JOIN](#cross-join)
   - [SELF JOIN](#self-join)
   - [ON vs WHERE in JOINs](#on-vs-where-in-joins)
   - [JOIN vs UNION](#join-vs-union)
9. [SQL Set Operations](#9-sql-set-operations)
   - [UNION & UNION ALL](#union--union-all)
   - [INTERSECT](#intersect)
   - [EXCEPT / MINUS](#except--minus)
   - [The 4 Mandatory Rules of Set Operations](#the-4-mandatory-rules-of-set-operations)
10. [SQL Subqueries (Nested Queries)](#10-sql-subqueries-nested-queries)
    - [Single-Row & Multi-Row Subqueries](#single-row--multi-row-subqueries)
    - [Correlated Subqueries](#correlated-subqueries)
    - [Subqueries with DML (INSERT, UPDATE, DELETE)](#subqueries-with-dml-insert-update-delete)
    - [EXISTS vs IN (Performance & Semantics)](#exists-vs-in-performance--semantics)
11. [SQL Views (Virtual Tables)](#11-sql-views-virtual-tables)
    - [Simple vs Complex Views](#simple-vs-complex-views)
    - [Table vs View Comparison](#table-vs-view-comparison)
12. [Transaction Management & ACID Practical (TCL Deep Dive)](#12-transaction-management--acid-practical-tcl-deep-dive)
    - [COMMIT, ROLLBACK, SAVEPOINT](#commit-rollback-savepoint)
    - [Transaction Isolation Levels & Concurrency Anomalies](#transaction-isolation-levels--concurrency-anomalies)
13. [Complete College Management Database Practice Dataset](#13-complete-college-management-database-practice-dataset)
14. [70 Practice Questions with Solutions](#14-70-practice-questions-with-solutions)
15. [Company-Level & Advanced SQL Interview Problems](#15-company-level--advanced-sql-interview-problems)
16. [SQL Rapid Revision Cheat Sheet & Memory Tricks](#16-sql-rapid-revision-cheat-sheet--memory-tricks)

---

# 1. Introduction to SQL & Relational Databases

### What is SQL?
**SQL** stands for **Structured Query Language**. It is the international standard declarative language designed to create, query, modify, and manage relational database management systems (**RDBMS**).

```
   Application (Backend / API)
              │
              ▼ (SQL Queries)
   Database Engine (RDBMS)
              │
              ▼
   Tables (Relations) ──► Rows (Tuples) & Columns (Attributes)
```

### Popular SQL Database Engines
- **MySQL** (Widely used in web development, LAMP stack)
- **PostgreSQL** (Enterprise-grade, highly extensible ORDBMS)
- **Oracle Database** (Enterprise corporate standard)
- **Microsoft SQL Server** (Enterprise Windows ecosystem)
- **SQLite** (Self-contained, serverless embedded database)

---

### Why Do We Need SQL? (Real-World Analogy)
Imagine a college university managing **20,000 students** using plain text files or spreadsheets:
- **Searching:** Finding a specific student takes linear time $O(N)$ through gigabytes of text.
- **Concurrent Updates:** Two clerks updating marks at the same second will overwrite each other's changes.
- **Redundancy & Inconsistency:** Student address is duplicated across 10 department files; an update in one leaves others inconsistent.
- **Data Integrity:** Nothing stops someone from entering an age of `-50` or marks of `999`.

**SQL with an RDBMS solves all these issues:**
- ✅ High-speed data retrieval using B-tree indexing.
- ✅ ACID transaction guarantees preventing partial writes and lost updates.
- ✅ Constraints (`PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `NOT NULL`) ensuring strict data validity.
- ✅ Normalized tables reducing data duplication.

---

### SQL vs NoSQL Comparison Table

| Feature | SQL (Relational) | NoSQL (Non-Relational) |
|---|---|---|
| **Data Storage** | Structured tables with fixed schema (Rows & Columns) | Documents (JSON/BSON), Key-Value, Graphs, Wide-column |
| **Schema** | Rigid / Predefined schema | Dynamic / Flexible schema |
| **Relationships & JOINs** | Native support with powerful `JOIN` operations | Denormalized; JOINs are avoided or handled in application code |
| **Transaction Model** | Strict **ACID** (Atomicity, Consistency, Isolation, Durability) | Typically **BASE** (Basically Available, Soft state, Eventual consistency) |
| **Scaling** | Vertical scaling (Scale-up: more CPU, RAM, SSD) | Horizontal scaling (Scale-out: sharding across clusters) |
| **Examples** | PostgreSQL, MySQL, SQL Server, Oracle | MongoDB, Redis, Cassandra, Neo4j, DynamoDB |

---

# 2. SQL Command Categories (The 5 Sub-Languages)

SQL commands are logically grouped into **5 core categories**:

```
                                 SQL COMMANDS
                                      │
     ┌──────────────┬─────────────────┼─────────────────┬──────────────┐
     │              │                 │                 │              │
    DDL            DML               DQL               DCL            TCL
Data Definition Data Manipulation Data Query     Data Control   Transaction Control
```

---

## DDL (Data Definition Language)
DDL commands define, alter, and manage the physical schema and structures of databases, tables, and indexes.  
> **Note:** DDL statements automatically trigger an implicit `COMMIT` in most relational databases (such as MySQL).

### 1. `CREATE DATABASE`
```sql
CREATE DATABASE CollegeDB;
```

### 2. `USE DATABASE`
```sql
USE CollegeDB;
```

### 3. `CREATE TABLE`
Defines a new relation with typed columns and constraints:
```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FullName VARCHAR(100) NOT NULL,
    Age INT CHECK (Age >= 17),
    Email VARCHAR(120) UNIQUE,
    City VARCHAR(50) DEFAULT 'Delhi',
    AdmissionDate DATE NOT NULL
);
```

### 4. `DESCRIBE` / `DESC`
Displays the structural schema, column data types, and keys:
```sql
DESC Students;
-- Or in PostgreSQL / ANSI:
-- \d Students
```

### 5. `ALTER TABLE`
Modifies an existing table schema without dropping it:

```sql
-- 1. Add a new column
ALTER TABLE Students
ADD PhoneNumber VARCHAR(15);

-- 2. Modify an existing column data type or constraint
ALTER TABLE Students
MODIFY FullName VARCHAR(150); -- MySQL syntax
-- PostgreSQL: ALTER TABLE Students ALTER COLUMN FullName TYPE VARCHAR(150);

-- 3. Rename a column
ALTER TABLE Students
RENAME COLUMN City TO Location;

-- 4. Drop a column
ALTER TABLE Students
DROP COLUMN PhoneNumber;
```

### 6. `RENAME TABLE`
```sql
RENAME TABLE Students TO EnrolledStudents;
```

### 7. `TRUNCATE TABLE`
Fast removal of all data rows while keeping table schema intact:
```sql
TRUNCATE TABLE EnrolledStudents;
```

### 8. `DROP TABLE` & `DROP DATABASE`
Permanently destroys the object and all contained data from disk:
```sql
DROP TABLE EnrolledStudents;
DROP DATABASE CollegeDB;
```

---

## DML (Data Manipulation Language)
DML commands manage the actual data records (rows) inside existing tables.

### 1. `INSERT`
Adds new rows into a table:
```sql
-- Insert specific columns
INSERT INTO Students (StudentID, FullName, Age, Email, AdmissionDate)
VALUES (1, 'Adil Ahmad', 20, 'adil@example.com', '2026-08-01');

-- Multi-row insert
INSERT INTO Students (StudentID, FullName, Age, Email, AdmissionDate)
VALUES
(2, 'Rahul Sharma', 21, 'rahul@example.com', '2026-08-01'),
(3, 'Priya Singh', 19, 'priya@example.com', '2026-08-02'),
(4, 'Aman Verma', 22, 'aman@example.com', '2026-08-03');
```

### 2. `UPDATE`
Modifies values in existing rows:
```sql
UPDATE Students
SET Age = 21, City = 'Mumbai'
WHERE StudentID = 1;

-- ⚠️ WARNING: Always provide a WHERE clause, otherwise ALL rows will be updated!
```

### 3. `DELETE`
Removes specified rows from a table:
```sql
DELETE FROM Students
WHERE StudentID = 4;

-- ⚠️ WARNING: If WHERE is omitted, all rows are deleted, but table structure remains!
```

---

## DQL (Data Query Language)
Used to fetch and project data from the database.
```sql
SELECT StudentID, FullName, City
FROM Students
WHERE Age >= 20;
```

---

## DCL (Data Control Language)
Used by DBAs to configure user privileges and security roles.

- `GRANT`: Gives permissions to users.
  ```sql
  GRANT SELECT, INSERT ON Students TO 'reporting_user'@'localhost';
  ```
- `REVOKE`: Removes permissions from users.
  ```sql
  REVOKE INSERT ON Students FROM 'reporting_user'@'localhost';
  ```

---

## TCL (Transaction Control Language)
Manages logical units of work ensuring ACID consistency.

- `COMMIT`: Persists all modifications permanently to disk.
- `ROLLBACK`: Reverts uncommitted changes back to the previous stable state.
- `SAVEPOINT`: Sets intermediate checkpoints inside an open transaction.
- `ROLLBACK TO SAVEPOINT`: Selectively rolls back changes to a specific checkpoint.

---

# 3. SQL Data Types

| Category | Data Type | Storage | Description & Practical Usage |
|---|---|---|---|
| **Numeric** | `SMALLINT` | 2 Bytes | Integer from $-32,768$ to $+32,767$ (ideal for age, small counters) |
| | `INT` / `INTEGER` | 4 Bytes | Standard integer from $-2.14 \times 10^9$ to $+2.14 \times 10^9$ |
| | `BIGINT` | 8 Bytes | Large integer (user IDs, view counts, financial transaction IDs) |
| | `DECIMAL(p, s)` / `NUMERIC(p, s)` | Variable | Exact numeric where $p$ is total digits, $s$ is decimal places. e.g. `DECIMAL(10, 2)` for currency |
| | `FLOAT` / `REAL` | 4 Bytes | Single-precision floating point |
| | `DOUBLE PRECISION` | 8 Bytes | Scientific calculations with high precision |
| **Character** | `CHAR(n)` | Fixed $n$ | Fixed-length string. Pads with spaces if shorter. (e.g. Country code `CHAR(2)`, Gender `CHAR(1)`) |
| | `VARCHAR(n)` | Variable | Variable-length text up to $n$ characters. Stores only actual length + 1/2 bytes length prefix |
| | `TEXT` | Variable | Variable-length long text (blogs, essays, reviews) |
| **Date/Time** | `DATE` | 4 Bytes | Calendar date (`YYYY-MM-DD`) |
| | `TIME` | 3–8 Bytes | Time of day (`HH:MM:SS`) |
| | `TIMESTAMP` | 8 Bytes | Combined date and exact time (`YYYY-MM-DD HH:MM:SS`) |
| | `INTERVAL` | 16 Bytes | Duration/span of time (`INTERVAL '7 days'`, `INTERVAL '2 hours'`) |
| **Boolean** | `BOOLEAN` | 1 Byte | Stores `TRUE`, `FALSE`, or `NULL` (unknown) |

---

# 4. DROP vs TRUNCATE vs DELETE

This is one of the most tested questions in campus placements and technical interviews:

```
                    DATA REMOVAL COMPARISON
 ┌─────────────────┬─────────────────┬─────────────────┐
 │     DELETE      │    TRUNCATE     │      DROP       │
 ├─────────────────┼─────────────────┼─────────────────┤
 │  DML Command    │  DDL Command    │  DDL Command    │
 │  Removes rows   │  Removes all    │  Destroys entire│
 │  matching WHERE │  rows fast      │  table & schema │
 └─────────────────┴─────────────────┴─────────────────┘
```

| Dimension | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| **Command Category** | **DML** (Data Manipulation Language) | **DDL** (Data Definition Language) | **DDL** (Data Definition Language) |
| **Filter (`WHERE`)** | ✅ Supported (`DELETE FROM T WHERE id=1;`) | ❌ Not supported (removes all rows) | ❌ Not applicable |
| **Table Schema** | Remains intact | Remains intact | Completely destroyed from data dictionary |
| **Speed / Performance** | Slow (deletes row-by-row & logs each row) | Very Fast (deallocates data pages directly) | Very Fast (frees entire disk allocation) |
| **Transaction Rollback** | Can be rolled back inside transaction | Cannot be rolled back in MySQL (can in PostgreSQL) | Cannot be rolled back in MySQL (can in PostgreSQL) |
| **Auto-Increment Reset** | Does **NOT** reset auto-increment counter | Resets auto-increment back to 1 | Table is destroyed |
| **Triggers** | Fires `ON DELETE` row triggers | Does **NOT** fire delete triggers | Does not fire delete triggers |

---

# 5. SQL Clauses & Query Construction

A **clause** is a built-in constituent of an SQL statement performing a dedicated task such as projecting columns, filtering records, grouping aggregates, or sorting output.

---

### SELECT & FROM
Specifies which columns to project and the source relation:
```sql
SELECT ProductID, ProductName, Price
FROM Products;

-- Project all columns:
SELECT * FROM Products;
```

---

### WHERE Clause & Operators
Filters individual records before any aggregation takes place:
```sql
-- Comparison Operators: =, != (<>), <, >, <=, >=
SELECT * FROM Products WHERE Price >= 5000;

-- Logical Operators: AND, OR, NOT
SELECT * FROM Products 
WHERE Category = 'Electronics' AND Price < 20000;

-- Range Operator: BETWEEN ... AND ... (Inclusive)
SELECT * FROM Products 
WHERE Price BETWEEN 1000 AND 5000;

-- List Membership: IN (...)
SELECT * FROM Products 
WHERE Category IN ('Books', 'Stationery', 'Toys');

-- Null Checking: IS NULL / IS NOT NULL
SELECT * FROM Students 
WHERE Email IS NULL;
```

---

### ORDER BY (Sorting)
Sorts results by one or multiple expressions:
```sql
-- Ascending order (Default)
SELECT * FROM Products 
ORDER BY Price ASC;

-- Descending order
SELECT * FROM Products 
ORDER BY Price DESC;

-- Multi-column sorting (Sort by Category ASC, then Price DESC)
SELECT ProductName, Category, Price 
FROM Products 
ORDER BY Category ASC, Price DESC;
```

---

### LIMIT & OFFSET (Pagination)
Controls the number of records returned:
```sql
-- Top 3 cheapest products:
SELECT * FROM Products 
ORDER BY Price ASC 
LIMIT 3;

-- Pagination: Page 2 (Skip 5, take next 5)
SELECT * FROM Products 
ORDER BY ProductID ASC 
LIMIT 5 OFFSET 5;
```

---

### GROUP BY & Aggregate Functions
Groups individual rows sharing identical values into summary rows.

**Standard Aggregate Functions:**
- `COUNT(*)`: Total count of rows including NULLs.
- `COUNT(column)`: Count of non-NULL values in that column.
- `SUM(column)`: Arithmetic sum of numeric values.
- `AVG(column)`: Arithmetic mean.
- `MIN(column)`: Smallest value.
- `MAX(column)`: Largest value.

```sql
SELECT Category, 
       COUNT(*) AS TotalProducts,
       AVG(Price) AS AveragePrice,
       MIN(Price) AS LowestPrice,
       MAX(Price) AS HighestPrice
FROM Products
GROUP BY Category;
```

> ⚠️ **Rule of GROUP BY:** Every column in the `SELECT` list that is not part of an aggregate function **MUST** be included in the `GROUP BY` clause!

---

### HAVING Clause (Filtering Groups)
Filters grouped records **after** `GROUP BY` has aggregated them:

```sql
SELECT Category, COUNT(*) AS ItemCount, AVG(Price) AS AvgPrice
FROM Products
GROUP BY Category
HAVING COUNT(*) > 5 AND AVG(Price) > 1000;
```

#### Key Difference: `WHERE` vs `HAVING`
| Feature | `WHERE` | `HAVING` |
|---|---|---|
| **Applies to** | Individual rows before grouping | Aggregated groups after `GROUP BY` |
| **Position** | Before `GROUP BY` | After `GROUP BY` |
| **Aggregate Functions** | ❌ Cannot contain aggregates (e.g. `WHERE AVG(p) > 10` is invalid) | ✅ Designed specifically to filter aggregate expressions (`HAVING AVG(p) > 10`) |

---

### DISTINCT Clause
Eliminates duplicate rows from the projected result set:
```sql
SELECT DISTINCT City FROM Students;
```

---

### Aliases (AS)
Gives temporary, readable display names to projected columns or shortened aliases to tables:
```sql
SELECT p.ProductName AS ItemName,
       p.Price * 1.18 AS PriceWithGST
FROM Products AS p;
```

---

### LIKE Operator & Pattern Matching (Wildcards)
The `LIKE` operator matches text against string patterns using two standard wildcards:
1. `%` → Matches **zero or more** characters.
2. `_` → Matches **exactly one** character.

| Pattern | Meaning | Example Matches |
|---|---|---|
| `'A%'` | Starts with 'A' | *Apple, Acer, Adil* |
| `'%e'` | Ends with 'e' | *Apple, Mouse, Chrome* |
| `'%phone%'` | Contains 'phone' anywhere | *iPhone, Headphone, Smartphone* |
| `'_at'` | Exactly 3 characters ending in 'at' | *Cat, Bat, Hat, Rat* |
| `'_a%'` | Second letter is 'a' | *Samsung, Laptop, Camera* |
| `'_____'` | Exactly 5 characters long | *Apple, Mouse, Table* |
| `'A%m'` | Starts with 'A' and ends with 'm' | *Aman, Alarm, Algorithm* |

```sql
SELECT FullName, Email 
FROM Students 
WHERE FullName LIKE 'A%' AND Email LIKE '%@gmail.com';
```

---

# 6. SQL Logical Query Execution Order

When you write a SQL query, the database engine does **NOT** execute clauses in the order they are written (`SELECT` is written first, but executed near the end!).

```
       WRITTEN ORDER                             LOGICAL EXECUTION ORDER
   1. SELECT                                   1. FROM (Identify tables)
   2. FROM                                     2. JOIN (Combine tables)
   3. JOIN ... ON                              3. ON (Join conditions)
   4. WHERE                                    4. WHERE (Filter individual rows)
   5. GROUP BY           ────────►             5. GROUP BY (Group rows into buckets)
   6. HAVING                                   6. HAVING (Filter grouped buckets)
   7. ORDER BY                                 7. SELECT (Compute expressions & aliases)
   8. LIMIT                                    8. DISTINCT (Eliminate duplicates)
                                               9. ORDER BY (Sort final rows)
                                              10. LIMIT / OFFSET (Slice final page)
```

> 💡 **Why this matters in interviews:**
> - You **cannot** use a `SELECT` column alias inside the `WHERE` clause because `WHERE` runs in Step 4, while aliases are defined in Step 7!
> - You **can** use column aliases in `ORDER BY` because `ORDER BY` runs in Step 9.

---

# 7. SQL String Functions & Conditional Expressions (`CASE`)

### Built-in String Functions
```sql
-- 1. UPPER() & LOWER()
SELECT UPPER(FullName), LOWER(City) FROM Students;

-- 2. LENGTH() (Number of characters)
SELECT FullName, LENGTH(FullName) AS NameLen FROM Students;

-- 3. SUBSTRING(str, start_pos, length) -- 1-indexed
SELECT FullName, SUBSTRING(FullName, 1, 4) AS ShortCode FROM Students;

-- 4. CONCAT() (Combines multiple strings)
SELECT CONCAT(FullName, ' lives in ', City) AS Description FROM Students;

-- 5. TRIM() (Removes leading & trailing whitespace)
SELECT TRIM('   clean text   ') AS Cleaned;
```

---

### Conditional Logic: The `CASE` Statement
The SQL `CASE` statement implements conditional branching (`IF ... THEN ... ELSE`):

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE default_result
END
```

#### Example 1: Inventory Stock Status
```sql
SELECT ProductName, StockQuantity,
       CASE
           WHEN StockQuantity = 0 THEN 'Out of Stock'
           WHEN StockQuantity <= 10 THEN 'Critical Low Stock'
           ELSE 'Healthy Stock'
       END AS InventoryStatus
FROM Products;
```

#### Example 2: Student Grade Classification
```sql
SELECT StudentName, Marks,
       CASE
           WHEN Marks >= 90 THEN 'A+ (Distinction)'
           WHEN Marks >= 75 THEN 'A (First Class)'
           WHEN Marks >= 60 THEN 'B (Second Class)'
           WHEN Marks >= 40 THEN 'C (Pass)'
           ELSE 'F (Fail)'
       END AS Grade
FROM Students;
```

#### Important Rules of `CASE`:
- Evaluated **top-to-bottom**; stops at the first condition that evaluates to `TRUE`.
- `ELSE` is optional; if omitted and no condition matches, SQL returns `NULL`.
- Can be placed inside `SELECT`, `WHERE`, `ORDER BY`, or `GROUP BY`.

---

# 8. SQL JOINs (Complete Multi-Table Relationships)

A **JOIN** combines rows from two or more tables based on a related common column (typically a Foreign Key referencing a Primary Key).

```
                            SQL JOINS TAXONOMY
                                    │
       ┌─────────────────┬──────────┴──────────┬─────────────────┐
       │                 │                     │                 │
  INNER JOIN        LEFT JOIN              RIGHT JOIN     FULL OUTER JOIN
 (Intersection)    (All Left + Match)   (All Right + Match)  (Union of both)
                         │                     │
                    CROSS JOIN             SELF JOIN
                 (Cartesian Product)    (Joined to itself)
```

---

### Reference Schema for JOIN Examples

#### `Students` Table:
| StudentID | Name | CityID |
|---|---|---|
| 1 | Ankit | 1 |
| 2 | Rahul | 2 |
| 3 | Aman | 3 |
| 4 | Priya | 2 |
| 5 | Riya | NULL |

#### `Cities` Table:
| CityID | CityName |
|---|---|
| 1 | Ahmedabad |
| 2 | Delhi |
| 3 | Mumbai |
| 4 | Pune |

---

### 1. INNER JOIN
Returns **only matching records** present in both tables.
```sql
SELECT s.StudentID, s.Name, c.CityName
FROM Students AS s
INNER JOIN Cities AS c
ON s.CityID = c.CityID;
```
*Output:*
| StudentID | Name | CityName |
|---|---|---|
| 1 | Ankit | Ahmedabad |
| 2 | Rahul | Delhi |
| 3 | Aman | Mumbai |
| 4 | Priya | Delhi |
*(Riya is excluded because CityID is NULL; Pune is excluded because no student references it).*

---

### 2. LEFT (OUTER) JOIN
Returns **all rows from the left table**, along with matching rows from the right table. If no match exists, `NULL` is filled for the right table columns.
```sql
SELECT s.StudentID, s.Name, c.CityName
FROM Students AS s
LEFT JOIN Cities AS c
ON s.CityID = c.CityID;
```
*Output includes Riya:*
| StudentID | Name | CityName |
|---|---|---|
| 1 | Ankit | Ahmedabad |
| 2 | Rahul | Delhi |
| 3 | Aman | Mumbai |
| 4 | Priya | Delhi |
| 5 | Riya | **NULL** |

---

### 3. RIGHT (OUTER) JOIN
Returns **all rows from the right table**, along with matching rows from the left table.
```sql
SELECT s.Name, c.CityName
FROM Students AS s
RIGHT JOIN Cities AS c
ON s.CityID = c.CityID;
```
*Output includes Pune with NULL student:*
| Name | CityName |
|---|---|
| Ankit | Ahmedabad |
| Rahul | Delhi |
| Priya | Delhi |
| Aman | Mumbai |
| **NULL** | Pune |

---

### 4. FULL OUTER JOIN
Returns **all records** when there is a match in either the left or right table. Unmatched attributes become `NULL`.

```sql
-- ANSI Standard (Supported natively in PostgreSQL, SQL Server, Oracle):
SELECT s.Name, c.CityName
FROM Students AS s
FULL OUTER JOIN Cities AS c
ON s.CityID = c.CityID;

-- MySQL Workaround (Simulated using LEFT JOIN UNION RIGHT JOIN):
SELECT s.Name, c.CityName
FROM Students AS s
LEFT JOIN Cities AS c ON s.CityID = c.CityID
UNION
SELECT s.Name, c.CityName
FROM Students AS s
RIGHT JOIN Cities AS c ON s.CityID = c.CityID;
```

---

### 5. CROSS JOIN
Returns the **Cartesian Product** of two tables. Every row of Table A is paired with every row of Table B.
$$\text{Total Rows} = \text{Rows in A} \times \text{Rows in B}$$

```sql
SELECT s.Name, c.CityName
FROM Students AS s
CROSS JOIN Cities AS c;
-- 5 students * 4 cities = 20 total rows
```

---

### 6. SELF JOIN
A table joined with itself to model **hierarchical or peer-to-peer relationships** (e.g. Employee-Manager hierarchy, duplicate row detection).

#### `Employees` Table:
| EmpID | EmpName | ManagerID |
|---|---|---|
| 1 | Alice (CEO) | NULL |
| 2 | Bob (Manager) | 1 |
| 3 | Charlie (Developer) | 2 |

```sql
SELECT e.EmpName AS Employee,
       m.EmpName AS ReportsToManager
FROM Employees AS e
LEFT JOIN Employees AS m
ON e.ManagerID = m.EmpID;
```
*Output:*
| Employee | ReportsToManager |
|---|---|
| Alice | NULL |
| Bob | Alice |
| Charlie | Bob |

---

### `ON` vs `WHERE` in JOIN Queries
- **`ON` Clause:** Defines **how tables are linked together**. For outer joins (`LEFT JOIN`), conditions in the `ON` clause determine which right-table rows match without eliminating left-table rows.
- **`WHERE` Clause:** Filters the **assembled result set after joining**. Any condition in `WHERE` checking right-table columns will discard unmatched rows (often accidentally turning a `LEFT JOIN` into an `INNER JOIN`!).

---

# 9. SQL Set Operations

Set operations combine the result sets of two or more independent `SELECT` queries vertically into a single tabular output.

```
                  SET OPERATIONS VISUALIZATION
   Set A: {1, 2, 3}                         Set B: {3, 4, 5}

   UNION (Deduplicated):                    {1, 2, 3, 4, 5}
   UNION ALL (Preserves duplicates):        {1, 2, 3, 3, 4, 5}
   INTERSECT (Common elements):             {3}
   EXCEPT / MINUS (In A but not in B):      {1, 2}
```

---

### Syntax & Demonstration

```sql
-- 1. UNION (Combines and eliminates duplicate rows)
SELECT Name FROM Employees_India
UNION
SELECT Name FROM Employees_USA;

-- 2. UNION ALL (Combines everything directly, keeps duplicates, MUCH FASTER)
SELECT Name FROM Employees_India
UNION ALL
SELECT Name FROM Employees_USA;

-- 3. INTERSECT (Returns only records appearing in BOTH queries)
SELECT Name FROM Employees_India
INTERSECT
SELECT Name FROM Employees_USA;

-- 4. EXCEPT (ANSI / PostgreSQL / SQL Server) / MINUS (Oracle)
SELECT Name FROM Employees_India
EXCEPT
SELECT Name FROM Employees_USA;
```

---

### Why is `UNION ALL` Faster than `UNION`?
- **`UNION`:** Executes query A $\rightarrow$ executes query B $\rightarrow$ merges records $\rightarrow$ **sorts all merged rows in memory/temp disk** $\rightarrow$ scans and eliminates duplicates.
- **`UNION ALL`:** Executes query A $\rightarrow$ executes query B $\rightarrow$ appends results directly without sorting or duplicate checking.  
> 🚀 **Rule of Thumb:** Always use `UNION ALL` unless duplicate removal is strictly required!

---

### The 4 Mandatory Rules of Set Operations
1. **Identical Column Count:** Both queries must project the exact same number of columns.
2. **Compatible Data Types:** Corresponding columns must have compatible data types (e.g. `INT` with `INT`, not `INT` with `DATE`).
3. **Column Names:** Column headers in the output are always inherited from the **first** `SELECT` statement.
4. **Single `ORDER BY`:** Sorting cannot be applied inside individual queries; only one `ORDER BY` is permitted at the very end of the statement.

---

# 10. SQL Subqueries (Nested Queries)

A **subquery** (or inner query) is a query nested inside another SQL statement (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, or inside `FROM` / `WHERE` clauses), enclosed in parentheses `()`.

---

### Types of Subqueries

```
                               SUBQUERIES
                                    │
    ┌──────────────────────┬────────┴──────────────┬──────────────────────┐
    │                      │                       │                      │
Single-Row Subquery   Multi-Row Subquery   Correlated Subquery     Nested Subquery
Returns single value  Returns list of rows Depends on outer query  Subquery inside subquery
(=, >, <, >=, <=)     (IN, ANY, ALL)       (Evaluated per row)
```

#### 1. Single-Row Subquery
Returns exactly one column and one row:
```sql
-- Find employees who earn more than the company average salary:
SELECT EmpName, Salary
FROM Employees
WHERE Salary > (
    SELECT AVG(Salary) FROM Employees
);
```

#### 2. Multi-Row Subquery
Returns multiple rows using `IN`, `ANY`, or `ALL`:
```sql
-- Find students enrolled in either 'Computer Science' or 'Data Science':
SELECT StudentName 
FROM Students 
WHERE DepartmentID IN (
    SELECT DepartmentID 
    FROM Departments 
    WHERE DepartmentName IN ('Computer Science', 'Data Science')
);
```

#### 3. Correlated Subquery
The inner query references a column from the outer query. It executes **once for every candidate row** processed by the outer query:
```sql
-- Find employees earning more than the average salary of their OWN department:
SELECT e.EmpName, e.DepartmentID, e.Salary
FROM Employees AS e
WHERE e.Salary > (
    SELECT AVG(sub.Salary)
    FROM Employees AS sub
    WHERE sub.DepartmentID = e.DepartmentID
);
```

---

### `EXISTS` vs `IN` (Interview Favorite)
- `IN`: Evaluates the subquery completely, loads the result list into memory, and performs equality comparisons. Ideal for **small static sets**.
- `EXISTS`: Returns a boolean `TRUE` immediately upon finding the first matching record (short-circuit evaluation). Does not return data rows. Highly optimized for **large datasets** when used with correlated subqueries.

```sql
-- Find departments that have at least one enrolled student:
SELECT d.DepartmentName
FROM Departments AS d
WHERE EXISTS (
    SELECT 1 
    FROM Students AS s
    WHERE s.DepartmentID = d.DepartmentID
);
```

---

# 11. SQL Views (Virtual Tables)

A **View** is a named, virtual table defined by an underlying SQL query. It does not store actual data rows on disk (except in Materialized Views); it dynamically executes the query when accessed.

```sql
-- 1. Create a View
CREATE VIEW TopPerformers AS
SELECT StudentID, StudentName, Marks, City
FROM Students
WHERE Marks >= 85;

-- 2. Querying a View (Just like a normal table)
SELECT * FROM TopPerformers WHERE City = 'Ahmedabad';

-- 3. Modify / Replace a View
CREATE OR REPLACE VIEW TopPerformers AS
SELECT StudentID, StudentName, Marks, City, DepartmentID
FROM Students
WHERE Marks >= 80;

-- 4. Drop a View
DROP VIEW TopPerformers;
```

---

### Simple Views vs Complex Views

| Attribute | Simple View | Complex View |
|---|---|---|
| **Underlying Tables** | Single base table | Multiple tables (uses `JOIN`) |
| **Aggregations** | Contains no `GROUP BY` or aggregates | Contains `GROUP BY`, `SUM`, `AVG`, etc. |
| **DML Operations** | Updatable (`INSERT`, `UPDATE`, `DELETE` reflect on base table) | Generally Read-Only (Cannot directly perform DML) |

---

### Why Use Views?
1. **Security & Data Masking:** Hide sensitive columns (e.g. exposing an `EmployeesPublic` view without `Salary` or `SSN`).
2. **Query Simplicity:** Encapsulate complex 5-table joins into a reusable single object.
3. **Data Abstraction:** Applications query the view; if the underlying physical schema changes, update the view query without breaking application code.

---

# 12. Transaction Management & ACID Practical (TCL Deep Dive)

A **Transaction** is a logical unit of work consisting of one or more database operations that must either execute entirely or have no effect whatsoever.

### The ACID Properties:
- **A — Atomicity:** "All or Nothing". If any statement fails, the entire transaction rolls back.
- **C — Consistency:** Database transitions from one valid state satisfying all constraints to another.
- **I — Isolation:** Concurrent transactions execute without observing each other's uncommitted intermediate states.
- **D — Durability:** Once committed, changes survive crashes, power cuts, and server restarts.

---

### Practical Demonstration: Bank Transfer
```sql
-- Scenario: Transfer ₹1000 from Amit (Account 1) to Rahul (Account 2)

START TRANSACTION; -- Or BEGIN

-- Step 1: Debit Amit
UPDATE Accounts 
SET Balance = Balance - 1000 
WHERE AccountID = 1;

-- Checkpoint
SAVEPOINT DebitCompleted;

-- Step 2: Credit Rahul
UPDATE Accounts 
SET Balance = Balance + 1000 
WHERE AccountID = 2;

-- If system error occurs here:
-- ROLLBACK TO DebitCompleted; -- Reverts only Step 2
-- Or complete failure:
-- ROLLBACK; -- Reverts everything

-- If everything succeeds:
COMMIT; -- Permanently records transaction to disk
```

---

### Transaction Isolation Levels & Concurrency Anomalies

1. **Dirty Read:** Transaction A reads data modified by Transaction B that has **not yet been committed**. If B rolls back, A acted on invalid "dirty" data.
2. **Non-Repeatable Read:** Transaction A reads a row. Transaction B updates that same row and commits. Transaction A re-reads the row and sees different values.
3. **Phantom Read:** Transaction A queries rows matching a condition (`WHERE Age > 20`). Transaction B inserts a new matching row and commits. Transaction A re-queries and sees "phantom" new rows.

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|---|:---:|:---:|:---:|:---:|
| **READ UNCOMMITTED** | ⚠️ Allowed | ⚠️ Allowed | ⚠️ Allowed | Fastest |
| **READ COMMITTED** | 🛡️ Prevented | ⚠️ Allowed | ⚠️ Allowed | High (PostgreSQL default) |
| **REPEATABLE READ** | 🛡️ Prevented | 🛡️ Prevented | ⚠️ Allowed (MySQL prevents) | Moderate (MySQL default) |
| **SERIALIZABLE** | 🛡️ Prevented | 🛡️ Prevented | 🛡️ Prevented | Slowest (Strictest) |

---

# 13. Complete College Management Database Practice Dataset

Use this canonical dataset to practice all standard SQL queries.

```sql
CREATE DATABASE CollegeDB;
USE CollegeDB;

-- 1. Departments Table
CREATE TABLE Departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50) NOT NULL,
    hod_name VARCHAR(50) NOT NULL
);

INSERT INTO Departments VALUES
(1, 'Computer Science', 'Dr. Sharma'),
(2, 'Information Technology', 'Dr. Patel'),
(3, 'Mechanical', 'Dr. Singh'),
(4, 'Civil', 'Dr. Mehta'),
(5, 'Electrical', 'Dr. Gupta'),
(6, 'Electronics', 'Dr. Verma'),
(7, 'Artificial Intelligence', 'Dr. Khan'),
(8, 'Data Science', 'Dr. Shah');

-- 2. Students Table
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(50) NOT NULL,
    gender VARCHAR(10) NOT NULL,
    age INT CHECK (age >= 17),
    city VARCHAR(50) NOT NULL,
    marks DECIMAL(5,2) NOT NULL,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES Departments(department_id)
);

INSERT INTO Students VALUES
(101, 'Ankit', 'Male', 21, 'Ahmedabad', 88.00, 1),
(102, 'Rahul', 'Male', 20, 'Delhi', 75.00, 2),
(103, 'Priya', 'Female', 21, 'Mumbai', 92.00, 1),
(104, 'Aman', 'Male', 22, 'Jaipur', 68.00, 3),
(105, 'Sneha', 'Female', 20, 'Pune', 84.00, 4),
(106, 'Riya', 'Female', 19, 'Surat', 91.00, 5),
(107, 'Karan', 'Male', 22, 'Indore', 66.00, 6),
(108, 'Neha', 'Female', 21, 'Bhopal', 78.00, 7),
(109, 'Vikas', 'Male', 23, 'Lucknow', 81.00, 8),
(110, 'Anjali', 'Female', 20, 'Ahmedabad', 87.00, 1),
(111, 'Rohit', 'Male', 21, 'Delhi', 59.00, 2),
(112, 'Pooja', 'Female', 22, 'Mumbai', 73.00, 3),
(113, 'Arjun', 'Male', 20, 'Pune', 95.00, 1),
(114, 'Kavya', 'Female', 19, 'Jaipur', 82.00, 2);

-- 3. Courses Table
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(50) NOT NULL,
    credits INT NOT NULL,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES Departments(department_id)
);

INSERT INTO Courses VALUES
(1, 'Database Management Systems', 4, 1),
(2, 'Operating Systems', 4, 1),
(3, 'Computer Networks', 3, 2),
(4, 'Machine Learning', 4, 7),
(5, 'Applied Data Analytics', 3, 8),
(6, 'Thermodynamics', 4, 3),
(7, 'Structural Analysis', 3, 4);
```

---

# 14. 70 Practice Questions with Solutions

### Part 1: Basic Retrieval & Filtering (Q1 – Q10)
```sql
-- 1. Display all student records:
SELECT * FROM Students;

-- 2. Display student name, age, and marks:
SELECT student_name, age, marks FROM Students;

-- 3. Show all unique cities where students live:
SELECT DISTINCT city FROM Students;

-- 4. Find students with marks strictly greater than 80:
SELECT * FROM Students WHERE marks > 80;

-- 5. Find female students from Ahmedabad:
SELECT * FROM Students WHERE gender = 'Female' AND city = 'Ahmedabad';

-- 6. Find students whose age is between 20 and 22 inclusive:
SELECT * FROM Students WHERE age BETWEEN 20 AND 22;

-- 7. Find students who live in either 'Delhi', 'Mumbai', or 'Pune':
SELECT * FROM Students WHERE city IN ('Delhi', 'Mumbai', 'Pune');

-- 8. Find students whose marks are NOT between 70 and 90:
SELECT * FROM Students WHERE marks NOT BETWEEN 70 AND 90;

-- 9. Find students who belong to department 1 or department 2:
SELECT * FROM Students WHERE department_id IN (1, 2);

-- 10. Display all courses having 4 credits:
SELECT * FROM Courses WHERE credits = 4;
```

---

### Part 2: Sorting with ORDER BY & Slicing with LIMIT (Q11 – Q20)
```sql
-- 11. Sort students by marks in ascending order:
SELECT * FROM Students ORDER BY marks ASC;

-- 12. Sort students by marks in descending order:
SELECT * FROM Students ORDER BY marks DESC;

-- 13. Top 3 highest-scoring students:
SELECT * FROM Students ORDER BY marks DESC LIMIT 3;

-- 14. 3 lowest-scoring students:
SELECT * FROM Students ORDER BY marks ASC LIMIT 3;

-- 15. Sort students alphabetically by city, then by marks descending:
SELECT * FROM Students ORDER BY city ASC, marks DESC;

-- 16. Display the 4th and 5th highest-scoring students (OFFSET pagination):
SELECT * FROM Students ORDER BY marks DESC LIMIT 2 OFFSET 3;

-- 17. Sort departments alphabetically by name:
SELECT * FROM Departments ORDER BY department_name ASC;

-- 18. Sort courses by credits descending:
SELECT * FROM Courses ORDER BY credits DESC;

-- 19. Youngest 5 students in the university:
SELECT * FROM Students ORDER BY age ASC, marks DESC LIMIT 5;

-- 20. Oldest male student:
SELECT * FROM Students WHERE gender = 'Male' ORDER BY age DESC LIMIT 1;
```

---

### Part 3: Aggregation & Summarization (Q21 – Q30)
```sql
-- 21. Total number of students:
SELECT COUNT(*) AS TotalStudents FROM Students;

-- 22. Average marks of all students:
SELECT AVG(marks) AS OverallAverageMarks FROM Students;

-- 23. Highest and lowest marks:
SELECT MAX(marks) AS HighestMarks, MIN(marks) AS LowestMarks FROM Students;

-- 24. Total marks scored across all students:
SELECT SUM(marks) AS CumulativeMarks FROM Students;

-- 25. Number of distinct cities represented:
SELECT COUNT(DISTINCT city) AS CityCount FROM Students;

-- 26. Count students by gender:
SELECT gender, COUNT(*) AS TotalCount FROM Students GROUP BY gender;

-- 27. Average marks scored by each gender:
SELECT gender, AVG(marks) AS AvgMarks FROM Students GROUP BY gender;

-- 28. Count students in each department:
SELECT department_id, COUNT(*) AS StudentCount 
FROM Students 
GROUP BY department_id;

-- 29. Highest marks scored in each department:
SELECT department_id, MAX(marks) AS MaxMarks 
FROM Students 
GROUP BY department_id;

-- 30. Total credits offered by each department:
SELECT department_id, SUM(credits) AS TotalCredits 
FROM Courses 
GROUP BY department_id;
```

---

### Part 4: Grouping & HAVING Clause (Q31 – Q40)
```sql
-- 31. Departments having more than 2 students:
SELECT department_id, COUNT(*) AS StudentCount
FROM Students
GROUP BY department_id
HAVING COUNT(*) > 2;

-- 32. Cities where average student marks exceed 80:
SELECT city, AVG(marks) AS AvgMarks
FROM Students
GROUP BY city
HAVING AVG(marks) > 80;

-- 33. Departments where minimum marks scored is above 70:
SELECT department_id, MIN(marks) AS MinMarks
FROM Students
GROUP BY department_id
HAVING MIN(marks) > 70;

-- 34. Cities having more than 1 student:
SELECT city, COUNT(*) AS CitizenStudents
FROM Students
GROUP BY city
HAVING COUNT(*) > 1;

-- 35. Departments where cumulative marks exceed 200:
SELECT department_id, SUM(marks) AS TotalDeptMarks
FROM Students
GROUP BY department_id
HAVING SUM(marks) > 200;

-- 36. Credit counts having more than 2 courses:
SELECT credits, COUNT(*) AS CourseCount
FROM Courses
GROUP BY credits
HAVING COUNT(*) > 2;

-- 37. Departments where average student age is 21 or higher:
SELECT department_id, AVG(age) AS AvgAge
FROM Students
GROUP BY department_id
HAVING AVG(age) >= 21;

-- 38. Genders having at least 6 students:
SELECT gender, COUNT(*) AS Cnt
FROM Students
GROUP BY gender
HAVING COUNT(*) >= 6;

-- 39. Cities where highest marks is at least 90:
SELECT city, MAX(marks) AS TopScore
FROM Students
GROUP BY city
HAVING MAX(marks) >= 90;

-- 40. Departments having exactly 1 student:
SELECT department_id, COUNT(*) AS Cnt
FROM Students
GROUP BY department_id
HAVING COUNT(*) = 1;
```

---

### Part 5: LIKE Operator & Wildcard Pattern Matching (Q41 – Q50)
```sql
-- 41. Student names starting with 'A':
SELECT student_name FROM Students WHERE student_name LIKE 'A%';

-- 42. Student names ending with 'a':
SELECT student_name FROM Students WHERE student_name LIKE '%a';

-- 43. Student names containing 'ya' anywhere:
SELECT student_name FROM Students WHERE student_name LIKE '%ya%';

-- 44. Student names where second letter is 'r':
SELECT student_name FROM Students WHERE student_name LIKE '_r%';

-- 45. Student names with exactly 4 letters:
SELECT student_name FROM Students WHERE student_name LIKE '____';

-- 46. Cities starting with 'A' or 'B':
SELECT DISTINCT city FROM Students WHERE city LIKE 'A%' OR city LIKE 'B%';

-- 47. Department names containing 'Science':
SELECT * FROM Departments WHERE department_name LIKE '%Science%';

-- 48. HOD names starting with 'Dr. S':
SELECT * FROM Departments WHERE hod_name LIKE 'Dr. S%';

-- 49. Course names ending with 'Systems':
SELECT * FROM Courses WHERE course_name LIKE '%Systems';

-- 50. Student names whose third letter is 'i':
SELECT student_name FROM Students WHERE student_name LIKE '__i%';
```

---

### Part 6: Multi-Table Queries (JOINs) (Q51 – Q60)
```sql
-- 51. Display student name and their department name (INNER JOIN):
SELECT s.student_name, d.department_name
FROM Students s
INNER JOIN Departments d ON s.department_id = d.department_id;

-- 52. Display student name, marks, and HOD name:
SELECT s.student_name, s.marks, d.hod_name
FROM Students s
INNER JOIN Departments d ON s.department_id = d.department_id;

-- 53. Display all departments and their students (LEFT JOIN to keep empty departments):
SELECT d.department_name, s.student_name
FROM Departments d
LEFT JOIN Students s ON d.department_id = s.department_id;

-- 54. Find departments that have ZERO enrolled students:
SELECT d.department_name
FROM Departments d
LEFT JOIN Students s ON d.department_id = s.department_id
WHERE s.student_id IS NULL;

-- 55. Display department name along with count of students:
SELECT d.department_name, COUNT(s.student_id) AS TotalStudents
FROM Departments d
LEFT JOIN Students s ON d.department_id = s.department_id
GROUP BY d.department_name;

-- 56. Display department name, course name, and credits:
SELECT d.department_name, c.course_name, c.credits
FROM Departments d
INNER JOIN Courses c ON d.department_id = c.department_id;

-- 57. Display department name and average student marks:
SELECT d.department_name, ROUND(AVG(s.marks), 2) AS AvgMarks
FROM Departments d
INNER JOIN Students s ON d.department_id = s.department_id
GROUP BY d.department_name;

-- 58. Find pairs of students from the SAME city (SELF JOIN):
SELECT s1.student_name AS Student1, 
       s2.student_name AS Student2, 
       s1.city
FROM Students s1
INNER JOIN Students s2 ON s1.city = s2.city AND s1.student_id < s2.student_id;

-- 59. Find pairs of students having the SAME marks:
SELECT s1.student_name AS Student1, 
       s2.student_name AS Student2, 
       s1.marks
FROM Students s1
INNER JOIN Students s2 ON s1.marks = s2.marks AND s1.student_id < s2.student_id;

-- 60. Cartesian product of Courses and Departments:
SELECT c.course_name, d.department_name
FROM Courses c
CROSS JOIN Departments d;
```

---

### Part 7: Subqueries & Set Operations (Q61 – Q70)
```sql
-- 61. Find students scoring above the university overall average:
SELECT student_name, marks
FROM Students
WHERE marks > (SELECT AVG(marks) FROM Students);

-- 62. Find student(s) with the absolute highest marks:
SELECT student_name, marks
FROM Students
WHERE marks = (SELECT MAX(marks) FROM Students);

-- 63. Find the second highest marks in the university without using LIMIT:
SELECT MAX(marks) AS SecondHighestMarks
FROM Students
WHERE marks < (SELECT MAX(marks) FROM Students);

-- 64. Find students belonging to departments located in Dr. Sharma or Dr. Patel's jurisdiction:
SELECT student_name
FROM Students
WHERE department_id IN (
    SELECT department_id FROM Departments WHERE hod_name IN ('Dr. Sharma', 'Dr. Patel')
);

-- 65. Display all distinct department IDs present in Students UNION with Courses:
SELECT department_id FROM Students
UNION
SELECT department_id FROM Courses;

-- 66. Display all student names combined with HOD names using UNION ALL:
SELECT student_name AS EntityName, 'Student' AS Role FROM Students
UNION ALL
SELECT hod_name, 'HOD' FROM Departments;

-- 67. Find students who scored higher than ALL students in Department 2:
SELECT student_name, marks
FROM Students
WHERE marks > ALL (
    SELECT marks FROM Students WHERE department_id = 2
);

-- 68. Find students scoring above their own department average (Correlated Subquery):
SELECT s.student_name, s.marks, s.department_id
FROM Students s
WHERE s.marks > (
    SELECT AVG(sub.marks) 
    FROM Students sub 
    WHERE sub.department_id = s.department_id
);

-- 69. Create a view for Computer Science students:
CREATE VIEW CS_Students AS
SELECT student_id, student_name, marks
FROM Students
WHERE department_id = 1;

-- 70. Use CASE statement to categorize student performance:
SELECT student_name, marks,
       CASE 
           WHEN marks >= 90 THEN 'Outstanding'
           WHEN marks >= 75 THEN 'Good'
           ELSE 'Needs Improvement'
       END AS PerformanceBand
FROM Students;
```

---

# 15. Company-Level & Advanced SQL Interview Problems

### Problem 1: Find the $N$-th Highest Salary / Marks
**Generic Formula for $N$-th Highest without `LIMIT` (Correlated Subquery):**
```sql
-- Find the 3rd highest marks:
SELECT DISTINCT marks
FROM Students s1
WHERE 2 = (
    SELECT COUNT(DISTINCT s2.marks)
    FROM Students s2
    WHERE s2.marks > s1.marks
);
```

Using `LIMIT` and `OFFSET`:
```sql
SELECT DISTINCT marks
FROM Students
ORDER BY marks DESC
LIMIT 1 OFFSET 2; -- (N - 1) offset
```

---

### Problem 2: Delete Duplicate Rows Keeping the Smallest ID
Given a table `Employees(id, email)`:
```sql
DELETE e1
FROM Employees e1
INNER JOIN Employees e2
ON e1.email = e2.email AND e1.id > e2.id;
```

---

### Problem 3: Department-Wise Top 1 Highest Scorer
```sql
SELECT s.department_id, s.student_name, s.marks
FROM Students s
INNER JOIN (
    SELECT department_id, MAX(marks) AS max_marks
    FROM Students
    GROUP BY department_id
) top_scores
ON s.department_id = top_scores.department_id 
   AND s.marks = top_scores.max_marks;
```

---

# 16. SQL Rapid Revision Cheat Sheet & Memory Tricks

```
                         SQL MASTER CHEAT SHEET
 ┌─────────────────┬────────────────────────────────────────────────────────┐
 │ DDL             │ CREATE, ALTER, DROP, TRUNCATE, RENAME                  │
 │ DML             │ INSERT, UPDATE, DELETE                                 │
 │ DQL             │ SELECT                                                 │
 │ DCL             │ GRANT, REVOKE                                          │
 │ TCL             │ COMMIT, ROLLBACK, SAVEPOINT                            │
 ├─────────────────┼────────────────────────────────────────────────────────┤
 │ CLAUSE ORDER    │ FROM ──► JOIN ──► WHERE ──► GROUP BY ──► HAVING        │
 │                 │ ──► SELECT ──► DISTINCT ──► ORDER BY ──► LIMIT         │
 ├─────────────────┼────────────────────────────────────────────────────────┤
 │ JOIN TYPES      │ INNER: Intersection only                               │
 │                 │ LEFT: All left + matched right                         │
 │                 │ RIGHT: All right + matched left                        │
 │                 │ FULL: Union of all records                             │
 │                 │ CROSS: N × M rows                                      │
 │                 │ SELF: Table joined to itself                           │
 ├─────────────────┼────────────────────────────────────────────────────────┤
 │ SET OPERATIONS  │ UNION: Combines & deduplicates (Slower)                │
 │                 │ UNION ALL: Combines & keeps duplicates (Fastest)       │
 │                 │ INTERSECT: Overlapping records in both                 │
 │                 │ EXCEPT / MINUS: In first query, absent in second       │
 ├─────────────────┼────────────────────────────────────────────────────────┤
 │ TRANSACTIONS    │ COMMIT (Save), ROLLBACK (Undo), SAVEPOINT (Checkpoint) │
 └─────────────────┴────────────────────────────────────────────────────────┘
```

### 🧠 Mnemonics & Memory Tricks:
1. **Clause Execution Order:**  
   **F**resh **J**uice **W**ith **G**reat **H**oney **S**mells **D**elicious **O**ver **L**unch  
   $$\text{\textbf{F}ROM} \rightarrow \text{\textbf{J}OIN} \rightarrow \text{\textbf{W}HERE} \rightarrow \text{\textbf{G}ROUP BY} \rightarrow \text{\textbf{H}AVING} \rightarrow \text{\textbf{S}ELECT} \rightarrow \text{\textbf{D}ISTINCT} \rightarrow \text{\textbf{O}RDER BY} \rightarrow \text{\textbf{L}IMIT}$$

2. **Wildcards:**  
   - `%` looks like multiple dots/circles $\rightarrow$ **Zero or many** characters.  
   - `_` is a single dash $\rightarrow$ **Exactly one** character.

3. **`WHERE` vs `HAVING`:**  
   - **W**HERE filters **W**hole individual rows.  
   - **H**AVING filters **H**eaps of aggregated groups.
