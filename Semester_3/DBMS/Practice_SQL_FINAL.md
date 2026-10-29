# 🎯 DBMS & SQL Final Master Directory

Aapka saara data 2 comprehensive master files me perfectly organize kar diya gaya hai:

---

## 📂 1. SQL Complete Master Notes
👉 **File:** [`SQL_Master_Notes.md`](file:///c:/Users/devis/OneDrive/Desktop/CGxSU_Semester_1/Semester_3/DBMS/SQL_Master_Notes.md)

### Isme kya-kya covered hai:
- **Introduction & Architecture:** SQL kya hai, RDBMS vs NoSQL, Features, Limitations.
- **SQL 5 Categories:** DDL (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`), DML (`INSERT`, `UPDATE`, `DELETE`), DQL (`SELECT`), DCL (`GRANT`, `REVOKE`), TCL (`COMMIT`, `ROLLBACK`, `SAVEPOINT`).
- **Data Types & Comparison:** Data types, `DROP` vs `TRUNCATE` vs `DELETE`.
- **Clauses & Operators:** `WHERE`, `ORDER BY`, `LIMIT & OFFSET`, `GROUP BY`, `HAVING`, `DISTINCT`, `AS`, `LIKE` wildcards (`%`, `_`).
- **Logical Query Execution Order:** Database engine kis order me query execute karta hai.
- **Functions & Logic:** String functions (`UPPER`, `LOWER`, `LENGTH`, `SUBSTRING`), `CASE` statements.
- **JOINs (Sabhi Types):** `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, `CROSS JOIN`, `SELF JOIN`, `ON` vs `WHERE`.
- **Set Operations:** `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT / MINUS`, 4 rules.
- **Subqueries:** Single-row, Multi-row, Correlated, Nested, `EXISTS` vs `IN`.
- **Views:** Simple Views vs Complex Views, Table vs View.
- **Transaction Management & ACID:** Live Banking transfer practical, Dirty Read, Non-Repeatable Read, Phantom Read, Isolation Levels.
- **Full Practice Dataset & 70 Questions:** College Management System table schema + 70 topic-wise solved SQL queries + interview questions.

---

## 🐘 2. PostgreSQL Complete Master Notes
👉 **File:** [`PostgreSQL_Master_Notes.md`](file:///c:/Users/devis/OneDrive/Desktop/CGxSU_Semester_1/Semester_3/DBMS/PostgreSQL_Master_Notes.md)

### Isme kya-kya covered hai:
- **Introduction & Architecture:** ORDBMS concept, Server $\rightarrow$ Database $\rightarrow$ Schema (`public`) $\rightarrow$ Tables.
- **SQL vs PostgreSQL & PostgreSQL vs MySQL:** Language vs Engine, detailed feature comparison table.
- **Installation & psql CLI:** Setup guide, Port 5432, meta-commands (`\l`, `\c`, `\dt`, `\d`, `\du`, `\timing`, `\q`).
- **Modern Constraints & Data Types:** `SERIAL` vs `GENERATED ALWAYS AS IDENTITY`, `UUID` (`gen_random_uuid()`), Native Arrays (`TEXT[]`, `INT[]`), `JSON` vs `JSONB`, Custom `ENUM`s, Range Types.
- **PostgreSQL-Specific Superpowers:** `RETURNING *`, `ILIKE`, `ON CONFLICT` (UPSERT), `DISTINCT ON`, `COALESCE`, `NULLIF`.
- **Date & Time:** `TIMESTAMP`, `NOW()`, `INTERVAL` calculations, `EXTRACT()`, `DATE_TRUNC()`, `TO_CHAR()`.
- **CTEs (WITH Clause):** Basic CTEs, Multiple/Chained CTEs, Writable CTEs, Recursive CTEs (Hierarchy trees).
- **Window Functions:** Window functions vs `GROUP BY`, `OVER()`, `PARTITION BY`, `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`, `LEAD()`.
- **Stored Procedures & Functions (PL/pgSQL):** Procedure vs Function, full CRUD implementation (`add_student`, `get_students`, `update_student`, `delete_student`).
- **Administration & Performance:** B-tree, Hash, GIN, GiST, BRIN indexes, Partial & Expression indexes, Full-Text Search (`tsvector`, `tsquery`), Extensions (`pgcrypto`, `PostGIS`), MVCC, `VACUUM` & `ANALYZE`.
- **Full-Stack Node.js Integration:** Connection pooling with `pg`, Express REST APIs (GET, POST, PUT, DELETE), SQL injection prevention with `$1, $2`, password hashing with bcrypt & JWT.
- **Top Interview Q&As & Revision Table.**
