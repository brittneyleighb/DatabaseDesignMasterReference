# Database Design — Master Reference

This is synthesized from notes and concepts I worked through in graduate database design coursework; universal database knowledge, but the explanations, metaphors, and real-world scenarios. This single document combines four resources:

1. **Quick-reference cheatsheet:** compact tables for fast lookup
2. **In-depth concept summaries:** explanations with metaphors, everyday-language overviews, and real-world scenarios
3. **Glossary:** alphabetized definitions of every key term
4. **Documentation & further learning:** curated external links

> **A note on sources.** Concepts covered are standard database fundamentals you'll find in any major textbook (Silberschatz/Korth, Ramakrishnan/Gehrke, Kleppmann) and across the linked official documentation. If you're using this and want to dig deeper into a topic, see Part IV for primary sources.

---

## Table of Contents
 
### Part I — Quick Reference Cheatsheet
1. Database Roles & Components
2. SQL Sublanguages
3. Core SQL Syntax (MySQL)
4. NULL Logic — Truth Tables
5. Keys & Referential Integrity
6. Constraints
7. Joins at a Glance
8. Aggregates & Grouping
9. Subqueries
10. Views
11. Relational Algebra
12. ER Diagrams (Crow's Foot)
13. Normalization
14. Implementation Rules
15. Storage Media & Layout
16. Table Structures
17. Indexes
18. Tablespaces, Partitions, Shards
19. Transactions (ACID)
20. Concurrency Control
21. Recovery
22. Database Architectures
23. Complex Data Types
24. Database Programming
25. NoSQL Models
26. Common Pitfalls Checklist
### Part II — In-Depth Concept Summaries
Each concept opens with an everyday-language overview, then technical detail with metaphors, and closes with a real-world scenario.
 
1. The Relational Model — Why Tables Won
2. SQL — A Declarative Language
3. Keys, Constraints, and Referential Integrity
4. Joins and the Algebra Behind Them
5. Subqueries, Views, and Composability
6. Database Design — From Idea to Schema
7. Normalization — Eliminating Redundancy
8. Storage and Indexing Internals
9. Transactions and ACID
10. Concurrency Control — Locks and Snapshots
11. Recovery and Backup
12. Distributed Databases and CAP
13. Data Warehouses and Analytics
14. Complex Types and Object-Relational
15. NoSQL — When Tables Aren't Enough
16. Database Programming Patterns
### Part III — Glossary
 
### Part IV — Documentation & Further Learning
 
---
 
# Part I — Quick Reference Cheatsheet
 
## 1. Database Roles & Components
 
| Role | What they do |
|---|---|
| **DBA (administrator)** | Secures the system, manages users and access |
| **Designer** | Defines structure, types, keys, constraints |
| **Programmer** | Writes apps that use the database |
| **User** | Issues queries (directly or via apps) |
 
| DBMS Component | Job |
|---|---|
| **Query processor** | Parses queries, builds execution plan |
| **Storage manager** | Reads/writes blocks on disk; manages indexes |
| **Transaction manager** | ACID enforcement |
| **Catalog (data dictionary)** | Directory of objects (tables, columns, indexes) |
| **Log** | Append-only record of every insert/update/delete |
| **Buffer manager** | Caches blocks in RAM (LRU eviction) |
 
---
 
## 2. SQL Sublanguages
 
| Sublanguage | Verbs | Purpose |
|---|---|---|
| **DDL** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Define schema |
| **DML** | `INSERT`, `UPDATE`, `DELETE`, `MERGE` | Modify data |
| **DQL** | `SELECT` | Retrieve data |
| **DCL** | `GRANT`, `REVOKE` | Permissions |
| **TCL** | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Transactions |
 
**CRUD = Create, Read, Update, Delete** → `INSERT`, `SELECT`, `UPDATE`, `DELETE`.
 
---
 
## 3. Core SQL Syntax (MySQL)
 
```sql
-- Database management
CREATE DATABASE name;        DROP DATABASE name;
USE name;                    SHOW DATABASES; SHOW TABLES; SHOW COLUMNS FROM t;
 
-- Table management
CREATE TABLE t (
  id     INT AUTO_INCREMENT PRIMARY KEY,
  name   VARCHAR(50) NOT NULL DEFAULT 'unknown',
  email  VARCHAR(100) UNIQUE,
  salary DECIMAL(10,2) CHECK (salary >= 0),
  mgr_id INT,
  CONSTRAINT fk_mgr FOREIGN KEY (mgr_id) REFERENCES t(id)
    ON UPDATE CASCADE ON DELETE SET NULL
);
ALTER TABLE t ADD/MODIFY/DROP COLUMN ...;
DROP TABLE t;     TRUNCATE TABLE t;
 
-- DML
INSERT INTO t (name, email) VALUES ('Sam','sam@x.com');
UPDATE t SET salary = salary * 1.05 WHERE id = 7;
DELETE FROM t WHERE id = 7;
 
-- DQL
SELECT DISTINCT col1, col2 AS alias
FROM t
WHERE col IN (1,2,3) AND name LIKE 'A%' AND created BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY col1
HAVING COUNT(*) > 5
ORDER BY col2 DESC
LIMIT 10;
```
 
### Operator precedence (high → low)
`()` → unary `-`, `NOT` → `*`, `/`, `%` → `+`, `-` → comparison (`=`, `<`, `>`, `<=`, `>=`, `<>`) → `AND` → `OR`
 
---
 
## 4. NULL Logic — Truth Tables
 
| AND | T | F | NULL |  | OR | T | F | NULL |
|---|---|---|---|---|---|---|---|---|
| **T** | T | F | NULL |  | **T** | T | T | T |
| **F** | F | F | F |  | **F** | T | F | NULL |
| **NULL** | NULL | F | NULL |  | **NULL** | T | NULL | NULL |
 
`NOT NULL = NULL`. Comparisons with NULL → NULL (never TRUE). Use `IS NULL` / `IS NOT NULL`. A `WHERE` row is selected only when the condition is **TRUE** (FALSE *and* NULL are skipped).
 
---
 
## 5. Keys & Referential Integrity
 
| Key | Definition |
|---|---|
| **Primary key** | Unique + NOT NULL + minimal; identifies a row |
| **Simple PK** | One column |
| **Composite PK** | Multiple columns |
| **Candidate key** | Unique + minimal (PK is the chosen candidate) |
| **Foreign key** | Column(s) referencing a PK; can be NULL; non-NULL must match a PK value |
| **Artificial (surrogate) key** | DB-generated integer (e.g., `AUTO_INCREMENT`) |
 
**Referential-integrity actions** on `ON UPDATE` / `ON DELETE`:
 
| Action | Effect when violation would occur |
|---|---|
| `RESTRICT` | Reject the change |
| `CASCADE` | Propagate to FK |
| `SET NULL` | Set FK columns to NULL |
| `SET DEFAULT` | Set FK to its default |
 
---
 
## 6. Constraints
 
| Constraint | Where | What it does |
|---|---|---|
| `NOT NULL` | column | Forbids NULL |
| `UNIQUE` | column or table | Values unique |
| `PRIMARY KEY` | column or table | Unique + NOT NULL |
| `FOREIGN KEY ... REFERENCES` | table | Referential integrity |
| `CHECK (expr)` | column or table | Reject when expr is FALSE |
| `DEFAULT` | column | Value when omitted |
 
A `CHECK` is satisfied when expression is **TRUE or NULL**, violated only on FALSE.
 
---
 
## 7. Joins at a Glance
 
```
INNER  →  only matching rows (intersection)
LEFT   →  all left rows + matching right (NULLs for unmatched)
RIGHT  →  all right rows + matching left
FULL   →  all rows from both (NULLs where no match)
CROSS  →  Cartesian product, no ON clause
SELF   →  table joined to itself (use aliases)
```
 
**Equijoin** uses `=`. **Non-equijoin** uses `<`, `>`, `<>`, etc. **UNION** stacks two compatible result sets (drops duplicates; `UNION ALL` keeps them).
 
---
 
## 8. Aggregates & Grouping
 
| Function | Result |
|---|---|
| `COUNT(*)` | Rows |
| `COUNT(col)` | Non-NULL values |
| `SUM`, `AVG`, `MIN`, `MAX` | Numeric summary; ignore NULLs |
 
**Order of clauses:** `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`. `WHERE` filters rows; `HAVING` filters groups.
 
---
 
## 9. Subqueries
 
| Type | Description |
|---|---|
| **Scalar** | Returns one value, used like a constant |
| **Row** | Returns one row |
| **Table** | Returns a result set, used in `FROM` or with `IN` |
| **Correlated** | References outer query; runs once per outer row |
| **EXISTS / NOT EXISTS** | Check whether subquery returns any rows |
 
**Flattening** = rewriting a subquery as a join (often faster).
 
---
 
## 10. Views
 
```sql
CREATE VIEW v AS SELECT ... FROM base_table WHERE ...
WITH CHECK OPTION;     -- reject inserts/updates that violate the view's WHERE
```
 
A **base table** is the underlying table; a **materialized view** stores its result physically.
 
---
 
## 11. Relational Algebra Cheats
 
| Op | Symbol | SQL equivalent |
|---|---|---|
| Select (filter rows) | σ | `WHERE` |
| Project (pick cols) | π | `SELECT col1, col2` |
| Product | × | `CROSS JOIN` |
| Join (theta join) | ⋈ | `INNER JOIN ... ON` |
| Union | ∪ | `UNION` |
| Intersect | ∩ | `INTERSECT` |
| Difference | − | `MINUS` / `EXCEPT` |
| Rename | ρ | `AS` |
| Aggregate | ɣ | `GROUP BY` |
 
---
 
## 12. ER Diagrams (Crow's Foot Notation)
 
```
   |O      one and only one
   |O—     one optional
  ─||      one mandatory (exactly one)
  ─O<      zero or many
  ─|<      one or many
```
 
| Concept | Definition |
|---|---|
| **Entity** | A thing (person, place, product) |
| **Relationship** | Statement linking two entities |
| **Attribute** | Property of an entity |
| **Reflexive relationship** | Entity related to itself |
| **Strong entity** | Has its own identifying attribute |
| **Weak entity** | Identified via relationship to another (the *identifying entity*) |
| **Supertype/Subtype** | Inheritance via `IsA` relationship |
| **Cardinality** | Min and max of the relationship |
 
**Three design phases:** Conceptual → Logical → Physical.
 
---
 
## 13. Normalization
 
| Form | Rule (informal) |
|---|---|
| **1NF** | Every cell holds one value, table has a PK |
| **2NF** | 1NF **+** every non-key column depends on the *whole* PK (no partial dependency) |
| **3NF** | 2NF **+** non-key columns depend only on the key (no transitive dependency) |
| **BCNF** | For every dependency `A → B`, **B is unique** (a stricter 3NF) |
 
**Mnemonic (Bill Kent):** *"The key, the whole key, and nothing but the key — so help me Codd."*
 
**Denormalization** = intentional redundancy for performance.
 
---
 
## 14. Implementation Rules (Logical Design)
 
| ER element | Becomes |
|---|---|
| Strong entity | Strong table with own PK |
| Weak entity | Weak table; PK includes identifying entity's PK as FK |
| Supertype | Supertype table |
| Subtype | Subtype table; PK is also FK to supertype |
| 1:N relationship | FK on the "many" side |
| 1:1 relationship | FK on either side (typically the optional side) |
| M:N relationship | New **junction/bridge table** with composite PK |
| Plural attribute | New table |
 
---
 
## 15. Storage Media & Layout
 
| Medium | Volatile? | Speed |
|---|---|---|
| RAM (main memory) | Yes | Fastest |
| SSD / Flash | No | Fast |
| HDD / Magnetic disk | No | Slowest |
 
| Layout | Best for |
|---|---|
| **Row-oriented** | OLTP, full-row reads/writes |
| **Column-oriented** | Analytics, aggregating few columns |
 
**Block** = uniform unit moved between disk and RAM. **Sector** (HDD) ≈ 512 B–4 KB. **Page** (SSD) ≈ 2–16 KB.
 
---
 
## 16. Table Structures
 
| Structure | How rows are placed | Lookup |
|---|---|---|
| **Heap** | Unordered | Full scan |
| **Sorted** | Ordered by sort column | Binary search |
| **Hash** | Bucket = hash(key) | O(1) average |
| **Cluster (multi-table)** | Rows of related tables interleaved by cluster key | Fast joins on cluster key |
 
---
 
## 17. Indexes
 
| Index | What it is |
|---|---|
| **Single-level** | One file: value → row pointer |
| **Multi-level (B+tree, B-tree)** | Hierarchical; balanced |
| **Primary (clustering)** | On the sort column |
| **Secondary (nonclustering)** | Not on sort column |
| **Dense** | Entry per row |
| **Sparse** | Entry per block |
| **Hash index** | Buckets via hash function |
| **Bitmap** | Grid of bits; great for low-cardinality cols |
| **Function index** | Indexes f(col) instead of col |
| **Logical index** | Pointers are PK values, not block addresses |
| **R-tree** | B+tree variant for spatial data (MBRs) |
 
**B+tree vs B-tree:** B+ keeps all values + pointers at leaves (good for range scans). B-tree may store pointer at any level (no value duplication).
 
**Selectivity (hit ratio / filter factor):** % rows returned. Lower = index more useful.
 
```sql
CREATE INDEX idx_name ON t(col1, col2);
DROP INDEX idx_name ON t;
SHOW INDEX FROM t;
EXPLAIN SELECT ...;        -- see the execution plan
```
 
---
 
## 18. Tablespaces, Partitions, Shards
 
| Concept | Definition |
|---|---|
| **Tablespace** | Maps tables to a file |
| **Partition** | Subset of one table on the same machine (horizontal = rows, vertical = columns) |
| **Shard** | Like a partition but spread across machines |
 
**Partition strategies:** `RANGE` (by value range), `LIST` (explicit values), `HASH` (modulo), `KEY` (DB-managed hash).
 
---
 
## 19. Transactions (ACID)
 
| Letter | Means |
|---|---|
| **A**tomic | All-or-nothing |
| **C**onsistent | All rules satisfied at commit |
| **I**solated | Concurrent txns appear independent |
| **D**urable | Committed changes survive crashes |
 
| Read anomaly | Description |
|---|---|
| **Dirty read** | Reads uncommitted data |
| **Nonrepeatable read** | Re-reading returns different value |
| **Phantom read** | Re-running returns different row set |
 
| Isolation level | Dirty | Nonrep. | Phantom |
|---|---|---|---|
| `READ UNCOMMITTED` | ✓ | ✓ | ✓ |
| `READ COMMITTED` | ✗ | ✓ | ✓ |
| `REPEATABLE READ` | ✗ | ✗ | ✓ |
| `SERIALIZABLE` | ✗ | ✗ | ✗ |
 
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;       -- or BEGIN
  SAVEPOINT s1;
  ...
  ROLLBACK TO s1;
COMMIT;                  -- or ROLLBACK
```
 
---
 
## 20. Concurrency Control
 
| Lock | Allows |
|---|---|
| **Shared (S)** | Multiple readers |
| **Exclusive (X)** | One writer; blocks others |
 
**Two-phase locking (2PL):** grow phase (acquire) → shrink phase (release).
- **Strict 2PL** holds X locks until commit/rollback.
- **Rigorous 2PL** holds *both* S and X locks until commit/rollback.
**Deadlock prevention/resolution:** aggressive locking, data ordering, timeout, cycle detection.
 
**Snapshot isolation:** each txn sees a private snapshot of data; conflicts detected at commit.
 
---
 
## 21. Recovery
 
**Log records:** update, compensation/undo, transaction (start/commit/rollback), checkpoint.
 
**Recovery from system failure:**
1. **Redo** all transactions committed since last checkpoint.
2. **Undo** transactions that never committed.
**Backups:** *cold* (database paused, full copy) vs *hot* (synchronized secondary, no downtime).
 
---
 
## 22. Database Architectures
 
| Architecture | Definition |
|---|---|
| **Single-tier** | App + DB on same machine |
| **Multi-tier / Web** | UI tier → app tier → DB tier |
| **Cloud DB (PaaS)** | DB delivered over the internet |
| **Parallel DB** | Multi-CPU on one box (shared memory / storage / nothing) |
| **Distributed DB** | Multiple computers over WAN |
| **Replicated DB** | Multiple copies (primary/secondary or group replication) |
| **Federated DB** | Middleware over autonomous heterogeneous DBs |
| **In-memory DB** | RAM-resident |
| **Embedded DB** | Lives inside an app process (e.g., SQLite) |
| **Data warehouse** | Analytics-optimized DB; star schema (fact + dimension tables) |
| **Data lake** | Raw, unprocessed data store for analytics |
 
**CAP theorem:** A distributed DB can guarantee at most 2 of: **C**onsistency, **A**vailability, **P**artition-tolerance.
 
**Two-phase commit (distributed transaction):** prepare → commit phases coordinated by a transaction coordinator.
 
**Cloud service tiers:** IaaS (raw infra) → PaaS (managed services) → SaaS (full apps).
 
---
 
## 23. Complex Data Types
 
| Category | Examples |
|---|---|
| **Collection** | `SET`, `MULTISET`, `LIST`, `ARRAY` |
| **Document** | XML, JSON (`JSON_EXTRACT()`, `JSON_OBJECT()`) |
| **Spatial** | `POINT`, `LINESTRING`, `POLYGON`, `GEOMETRY` (WKT format) |
| **Object** | Composite types with methods, inheritance via `CREATE TYPE UNDER` |
 
Data structuring spectrum: **Structured → Semistructured (JSON/XML) → Unstructured**.
 
---
 
## 24. Database Programming
 
| Approach | Description |
|---|---|
| **Embedded SQL** | SQL inside C/Java/etc. via precompiler; uses *cursors* |
| **Procedural SQL (SQL/PSM)** | Stored procedures, functions, triggers (`CREATE PROCEDURE`, `CREATE FUNCTION`, `CREATE TRIGGER`) |
| **API (driver-based)** | ODBC, JDBC, DB-API (Python), ADO.NET, PDO (PHP) |
 
```python
# Python (Connector/Python)
import mysql.connector
conn = mysql.connector.connect(host='...', user='...', password='...', database='...')
cur  = conn.cursor()
cur.execute("SELECT * FROM Flights WHERE id=%s", (42,))   # parameterized → no SQL injection
rows = cur.fetchall()
cur.close(); conn.commit(); conn.close()
```
 
**Always parameterize** queries to prevent SQL injection.
 
---
 
## 25. NoSQL — Four Models
 
| Model | Data unit | Example DBs |
|---|---|---|
| **Key-value** | key → blob | Redis, DynamoDB, Riak |
| **Wide column** | row → column families | Cassandra, HBase, Bigtable |
| **Document** | key → JSON/XML doc | MongoDB, Couchbase |
| **Graph** | vertices + edges | Neo4j, JanusGraph |
 
**Scaling:** *Vertical* (bigger machine) vs *Horizontal* (more machines via sharding).
 
**MongoDB essentials:**
 
```js
db.students.insertOne({ name: 'Sue', gpa: 3.2 });
db.students.insertMany([...]);
db.students.find({ gpa: { $gte: 3.0 } });
db.students.updateOne({ name: 'Sue' }, { $set: { gpa: 3.3 } });
db.students.deleteOne({ name: 'Sue' });
```
 
| Operator | Meaning |
|---|---|
| `$eq`, `$ne` | equal / not equal |
| `$gt`, `$gte`, `$lt`, `$lte` | comparisons |
| `$in`, `$nin` | in / not in array |
| `$and`, `$or`, `$not` | logical |
| `$set`, `$inc`, `$push`, `$pull` | updates |
 
---
 
## 26. Common Pitfalls Checklist
 
- [ ] Forgetting `WHERE` on `UPDATE` / `DELETE` (whole table affected)
- [ ] Using `=` to compare with NULL (always returns NULL → use `IS NULL`)
- [ ] Mixing aggregates with non-grouped columns
- [ ] Implicit cross joins from comma-separated FROM with no WHERE
- [ ] Composite PK design when an artificial key is simpler
- [ ] Adding indexes everywhere (writes get slower)
- [ ] Long-running transactions holding locks
- [ ] Not parameterizing user input → SQL injection
---
 
# Part II — In-Depth Concept Summaries
 
## 1. The Relational Model — Why Tables Won
 
A relational database is a system of organized lists where each list is a table, each row is a record, and each table can be linked to others by shared values. Think of it like a really disciplined filing cabinet. Every drawer has a clear label, every folder follows the same template within a drawer, and you can cross-reference one folder to another by writing down the matching ID. This structure is what lets your bank track millions of customers and billions of transactions without losing a penny, and what lets a hospital pull up the right patient's record in milliseconds. Almost every business application you use (payroll, inventory, electronic medical records, airline reservations) is sitting on top of a relational database.
 
In 1970 E. F. Codd published a deceptively simple idea: store data in **relations** (tables). What made it revolutionary wasn't tables themselves. It was that the model also gave us a **mathematical algebra** for manipulating those tables, and a clean separation between *what* you want and *how* the database fetches it.
 
**Metaphor: a library catalog before and after.** Picture a 1960s library where you had to know which floor a book was on, which shelf, and the exact pull order to retrieve it. (This is the **navigational** style; older hierarchical and network databases worked like this.) Codd's model is like introducing a card catalog. You write down the *attributes* you want (author, year, topic) and let the librarian figure out where to find it. Codd called this property **data independence**: your queries don't change when storage changes.
 
Three pieces of every database model:
- **Data structures**, how data is shaped (tables of rows and columns)
- **Operations**, what you can do (relational algebra)
- **Rules**, what makes data valid (constraints)
Some terminology to keep straight. A **table** is a fixed sequence of named columns and a *set* of rows (so unordered, no duplicates by definition). A **column** has a name and a data type. A **row** is an unnamed tuple of values. A **cell** is one column of one row. **Relational rules** are part of the model itself (e.g. "primary keys must be unique"); **business rules** are organization-specific (e.g. "salary must be positive").
 
**Real-world scenario.** When you make a doctor's appointment online, the booking system stores your patient profile in a `Patients` table, available slots in an `Appointments` table, and the link between you and your slot in a join. Without the relational model's separation between the data and how it's stored, every change to the doctor's schedule would risk corrupting your record. The hospital's billing system, separate but referencing the same patient ID, can produce an invoice without anyone copy-pasting your name and address.
 
---
 
## 2. SQL — A Declarative Language
 
SQL is the language you use to talk to a relational database. The unusual thing about it is that you describe *what answer you want* rather than *how to find it*. You'd say "give me all customers in California who spent over $500 last month" and the database figures out the most efficient way to scan, filter, and combine the data. This is similar to telling a personal shopper your shopping list versus walking the aisles yourself: you save effort, but you have to trust the shopper. Almost every analytics tool, business dashboard, and CRUD app you've ever interacted with talks to its database in SQL behind the scenes.
 
SQL is the universal query language of relational databases. It's **declarative**: you describe the result you want, not the steps to compute it. Compare:
 
- *Imperative (Python):* "Open the file, loop through each row, check if `state == 'CA'`, append to list, return list."
- *Declarative (SQL):* `SELECT * FROM customers WHERE state = 'CA';`
The database's **query optimizer** is responsible for translating your declarative wish into an efficient **execution plan**. That's why you can have two SQL statements that look completely different but produce the same result, and the optimizer may even pick the same plan for both.
 
### SQL is actually five languages in one
 
| Sublanguage | Purpose | Verbs |
|---|---|---|
| DDL — Data **Definition** Language | Define schema | `CREATE`, `ALTER`, `DROP` |
| DML — Data **Manipulation** Language | Change data | `INSERT`, `UPDATE`, `DELETE` |
| DQL — Data **Query** Language | Read data | `SELECT` |
| DCL — Data **Control** Language | Permissions | `GRANT`, `REVOKE` |
| TCL — **Transaction** Control Language | Group changes | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
 
A **statement** ends with a semicolon. A statement is made of **clauses** like `SELECT`, `FROM`, `WHERE`, each beginning with a keyword. The **SQL standard** is jointly published by ANSI and ISO, but every vendor (MySQL, PostgreSQL, Oracle, SQL Server) adds their own dialect on top.
 
**Real-world scenario.** A marketing analyst at a streaming service like Spotify wants to find the top 10 most-played songs by users under 25 in Brazil last week. She doesn't write a 200-line script that opens files, filters records, and sorts them. She writes a single SQL query of maybe 8 lines. The database engine handles everything else: which indexes to use, whether to filter or join first, whether to parallelize across processors. That declarative magic is what makes data analysts productive at scale.
 
### NULL, the "I don't know" value
 
`NULL` represents *unknown* or *inapplicable* data. Not zero, not empty string. The trickiest thing about NULL is that it propagates through expressions: `5 + NULL = NULL`, `'a' = NULL` is `NULL` (not `FALSE`!), and `NOT NULL = NULL`.
 
**Three-valued logic.** SQL's truth values are TRUE, FALSE, and NULL. A `WHERE` clause selects a row only when the predicate is TRUE; both FALSE and NULL are skipped. This is why `WHERE col = NULL` finds nothing. You must use `IS NULL`.
 
**Metaphor: Schrödinger's cell.** A NULL cell is like a closed box. You can't say it equals another closed box (you don't know what's inside either one). All you can say is "this box exists but I haven't opened it" (`IS NULL`) or "this box has been opened" (`IS NOT NULL`).
 
**Real-world scenario.** When you sign up for a new account on an app, you might leave your phone number blank. That field is stored as NULL, meaning "we don't know," not "blank string." The app might want to distinguish "user has no phone" from "user explicitly entered nothing." When the marketing team queries `WHERE phone_number != '555-1234'`, NULL phones won't appear in the result, which can be a nasty surprise if they were trying to count "everyone except this number."
 
---
 
## 3. Keys, Constraints, and Referential Integrity
 
Keys are the way a database knows that "this row" is uniquely *this* row, and constraints are the rules the database enforces so that bad data simply can't get in. A primary key is the row's permanent ID badge. A foreign key is one row pointing at another row's badge to say "I belong with that one." Constraints are built-in rules like "this field can't be empty," "no two users can have the same email," or "salary must be positive." These guarantees mean you can trust the database to keep its house in order without your application code re-checking everything.
 
A **primary key (PK)** is the column (or set of columns) you nominate to identify each row uniquely. The PK has three jobs:
 
1. **Unique.** No two rows share the same PK value.
2. **Not NULL.** Every row has a PK.
3. **Minimal.** For composite PKs, every column is necessary.
A **foreign key (FK)** is a column in one table that points to a PK in another. It's how relationships are wired in. A FK can be NULL (meaning "no relationship right now"), but if it has a value, that value must exist as a PK somewhere.
 
**Metaphor: library books and library cards.** Each book has a unique call number (PK on `Books`). Each loan record has a column `borrower_card_id` that points to a row in `LibraryMembers` (FK). If a member isn't returned in `LibraryMembers`, you can't have a loan record pointing to them. That would be a phantom borrower.
 
**Real-world scenario.** Imagine an airline reservation system. Each passenger has a unique passenger ID (PK on `Passengers`), each flight has a flight number (PK on `Flights`), and each booking has both a passenger ID and a flight number as foreign keys (`Bookings`). Referential integrity means the airline literally cannot have a booking referencing a deleted flight or a non-existent passenger. Those error states are impossible by construction. When you cancel a flight, the database forces the airline to decide upfront: cascade-delete the bookings, set their flight reference to NULL, or block the cancellation.
 
### Referential integrity actions
 
What happens when you change or delete a row that's referenced? You declare that on the foreign key:
 
- `RESTRICT`. Refuse the change (most conservative).
- `CASCADE`. Propagate it. Deleting a customer deletes their orders.
- `SET NULL`. Orphan the children but keep them.
- `SET DEFAULT`. Orphan to a default value.
### Constraints, the database as enforcer
 
A constraint is a rule the database enforces automatically. It rejects any insert/update that violates it. You'll find:
 
- `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`.
- **Column constraints** appear inline; **table constraints** appear in a separate clause and can span multiple columns.
- A `CHECK` is *satisfied* when the expression is TRUE *or* NULL; only FALSE causes rejection. (Yes, that's a quirk that follows from SQL's three-valued logic.)
**Real-world scenario.** A ride-share app like Lyft uses a `CHECK` constraint to ensure trip distances are non-negative and ratings are between 1 and 5. A `UNIQUE` constraint on the email column prevents two users from accidentally claiming the same identity. A `NOT NULL` constraint on the driver's license number ensures no driver gets registered without verification. These rules live in the database itself, so even if a buggy mobile app build tries to send invalid data, it bounces off the database wall, saving the company from data corruption that could affect millions of riders.
 
---
 
## 4. Joins and the Algebra Behind Them
 
A join is how you combine information from two related tables into one result. If your customers are in one table and their orders are in another, a join lets you ask "who ordered what?" by lining up rows where the customer ID matches. Different join types control what happens when there's no match. Do you skip those rows, or include them with blanks for the missing side? Joins are the engine of almost every report you'll ever see, from a sales dashboard to a "books you might like" recommendation.
 
A **join** combines rows from two tables based on related column values. The mental model:
 
1. **Cross join** (Cartesian product). Line up every row of A with every row of B. With 100 rows in A and 50 in B, you get 5,000 combinations. Almost never useful by itself.
2. **Inner join.** Start with the cross join, then keep only rows where the join condition is TRUE. This is what you usually want.
3. **Outer joins** preserve rows from one or both sides even when they have no match, filling unmatched columns with NULL.
| Join | Keeps unmatched rows from… |
|---|---|
| `INNER JOIN` | neither side |
| `LEFT JOIN` | left only |
| `RIGHT JOIN` | right only |
| `FULL JOIN` | both |
 
**Equijoin** uses `=`. **Non-equijoin** uses other operators (e.g. range overlaps). **Self-join** joins a table to itself (think: employees and their managers, both rows in `Employee`). **UNION** stacks two compatible result sets vertically; both queries must return the same number of columns with compatible types.
 
**Real-world scenario.** Netflix's "what you watched" page uses a join between `Users`, `WatchHistory`, and `Titles`. An inner join shows only watches that match a known title (perfect for the homepage). A left join from `Users` to `WatchHistory` keeps users who haven't watched anything yet, useful for the "let's recommend something to start with" team. A self-join on the `Titles` table can reveal which movies share a director or franchise (joining the table to itself on `director_id`). Behind every "Customers also watched" row on Netflix is a chain of joins computed in milliseconds.
 
### Relational algebra
 
Codd's algebra defines a small, composable set of operations over tables. Every SQL query is ultimately a tree of these ops:
 
- **σ (Select).** Pick rows by predicate. (`WHERE`)
- **π (Project).** Pick columns. (`SELECT col1, col2`)
- **× (Product).** Cartesian product. (`CROSS JOIN`)
- **⋈ (Join).** Product followed by select. (`INNER JOIN ... ON`)
- **∪ ∩ −** Union, intersect, difference for compatible tables.
- **ρ (Rename).** Rename columns/tables. (`AS`)
- **ɣ (Aggregate).** Group and aggregate. (`GROUP BY`)
Two algebra expressions are **equivalent** if they always produce the same result. The query optimizer uses these equivalences to reorganize the tree into something cheaper.
 
**Metaphor: a factory assembly line.** Each operator is a station. The optimizer rearranges the line so the cheapest filtering happens early (less material flows downstream). For example, pushing a `σ` (selection) below a join means filtering rows *before* the expensive join, not after.
 
---
 
## 5. Subqueries, Views, and Composability
 
A subquery is a query nested inside another query. It's how you ask "find me X based on whatever Y satisfies these conditions" in a single shot. A view is a saved query you can reuse by name, like a smart shortcut. You define "premium customers in California" once and then everyone in the company queries `premium_ca_customers` instead of rewriting the criteria. Both let you compose complex questions out of simpler building blocks. They're how data teams keep huge codebases of SQL maintainable instead of having identical 50-line subqueries pasted everywhere.
 
A **subquery** is a query inside another query. It can appear in:
 
- The `WHERE` clause: `WHERE id IN (SELECT id FROM ...)`
- The `FROM` clause (as a derived table): `FROM (SELECT ...) AS t`
- The `SELECT` list (a scalar subquery returning one value)
A subquery is **correlated** when it references a column from the outer query, meaning it conceptually re-runs once per outer row. The optimizer often flattens correlated subqueries into joins (a process called *flattening*) for performance.
 
**EXISTS / NOT EXISTS** check whether a subquery returns any rows. They're often more efficient than `IN` for correlated patterns.
 
**Real-world scenario.** A SaaS company like Slack runs a nightly report on "active workspaces with no admin who logged in this week." That single business question requires a subquery: pick workspaces (outer query) where the admin user (correlated lookup) hasn't appeared in the login table (subquery with `NOT EXISTS`). Wrapping that as a view called `at_risk_workspaces` means the customer-success team can query it daily without rewriting the logic, and if the definition of "at risk" later changes, you fix it in one place.
 
### Views
 
A **view** is a saved `SELECT` statement that you can query as if it were a table. The actual rows aren't stored. The database expands the view query at runtime (this is similar to function inlining in programming).
 
Why use views?
- **Abstraction.** Hide complex joins behind a simple name.
- **Security.** Show users only certain rows/columns.
- **Backward compatibility.** Preserve an old API after schema changes.
A **materialized view** stores the result physically, refreshed periodically. Use it when the underlying query is expensive and the data isn't required to be real-time.
 
`WITH CHECK OPTION` makes a view *write-safe*: inserts/updates that would no longer satisfy the view's `WHERE` clause are rejected (so you can't insert rows that disappear from the view).
 
**Real-world scenario for materialized views.** A retail chain's executive dashboard shows last-quarter revenue by region. Computing that from millions of transaction rows takes 15 seconds, too slow for an interactive dashboard. The data team creates a materialized view that pre-aggregates revenue by region and refreshes overnight. Now the dashboard loads instantly. The trade-off: the numbers are 24 hours stale, which is fine for executive review but not for operational alerting.
 
---
 
## 6. Database Design — From Idea to Schema
 
Database design is the process of figuring out what tables you need and how they connect, *before* you write any SQL. You start by listing the things you care about (customers, products, orders), the relationships between them (a customer places many orders), and the properties of each (a customer has a name and email). Then you translate that picture into actual tables with primary keys and foreign keys. Skipping this step is the #1 cause of schemas that have to be painfully refactored two years in, when you realize "wait, a product can have multiple categories?" requires a table you never built.
 
Database design moves from abstract to concrete in three phases:
 
1. **Conceptual design.** Entity-relationship modeling. Ignore implementation.
2. **Logical design.** Convert ER model into tables, columns, keys.
3. **Physical design.** Add indexes, choose table structure, decide partitioning.
### ER modeling
 
An **entity-relationship (ER) model** describes the world in three nouns:
 
- **Entity.** A thing (Customer, Product, Order).
- **Attribute.** A property of an entity (Customer.email).
- **Relationship.** A connection between entities (Customer-places-Order).
The corresponding **types** vs **instances** distinction matters:
- *Entity type* = "all customers"; *entity instance* = "Sam Snead specifically".
A **reflexive relationship** relates an entity to itself (an Employee manages another Employee).
 
**Real-world scenario.** A university designing a new course-registration system starts by identifying entities (Students, Courses, Instructors, Sections), relationships (a Student *enrolls in* many Sections; an Instructor *teaches* many Sections), and attributes (Student.gpa, Course.credits). Skipping conceptual design (just diving into table creation) is how universities end up with messes like a single "courses" table where one row says "MWF 9am, room 101" mashed together as a string. With ER modeling, "section" becomes its own entity with its own properties, and querying "what's the room for CS 101 on Wednesday?" becomes trivial.
 
### Cardinality
 
**Cardinality** captures *how many*. It's the most-asked question in design.
 
- **Maximum.** One or many? "Each customer can place *many* orders, but each order has *one* customer."
- **Minimum.** Optional or required? "An order *must* have a customer; a customer *may* have zero orders."
Notation conventions vary. **Crow's foot** is the most common today; you'll also see UML, IDEF1X, and Chen notation in older texts.
 
**Real-world scenario.** A hospital's electronic medical record asks: how many doctors can a patient have? Most systems answer "one primary care doctor (1:1) plus many specialists (1:N)." Getting cardinality right at design time means the database supports the actual workflow. Get it wrong (say, modeling "one doctor per patient") and the system can't represent reality, forcing nurses into ugly workarounds like fake patient duplicates.
 
### Strong vs weak entities
 
A **strong entity** has its own identifying attribute (a SKU, an SSN, a generated ID). A **weak entity** can't be identified standalone. It depends on a parent via an *identifying relationship*. Classic example: an order *line* doesn't make sense without an order; its identity is "line 3 of Order #42".
 
### Supertypes and subtypes, inheritance in design
 
When two entities share many attributes/relationships, abstract the commonalities into a **supertype** entity, with the differences in **subtype** entities. Like classes and subclasses in OOP.
- A **partition** is a set of mutually exclusive subtypes (a Person is either an Employee or a Customer, not both).
- The **IsA relationship** ties subtype to supertype: "Employee IsA Person."
**Real-world scenario.** A bank with checking, savings, and credit-card accounts uses a supertype `Account` (with shared attributes like account_id, customer_id, balance) and subtypes `CheckingAccount` (overdraft_limit), `SavingsAccount` (interest_rate), `CreditCardAccount` (credit_limit). When the bank wants "show me all of customer #12345's accounts and their balances," it queries the supertype. When it wants "calculate this month's interest payouts," it queries just the savings subtype. Without supertypes, you'd have three nearly-identical tables and constant `UNION ALL` queries.
 
### Implementing the model
 
| ER element | Becomes |
|---|---|
| Strong entity | Strong table with own PK (often artificial integer) |
| Weak entity | Weak table whose PK includes the parent's PK as FK |
| Supertype | Supertype table |
| Subtype | Subtype table; PK is also FK to supertype |
| 1:N relationship | FK on the "many" side |
| 1:1 relationship | FK on either side (typically the optional side) |
| M:N relationship | New **junction table** with a composite PK of two FKs |
| Plural attribute (e.g. multiple phone numbers) | New table |
 
Choose **artificial keys** (auto-incrementing integers) over natural keys when the natural key isn't stable. (Phone numbers change. Database-generated IDs don't.)
 
---
 
## 7. Normalization — Eliminating Redundancy
 
Normalization is the discipline of *not* repeating yourself in your tables. If your customer's address shows up in 50 order rows, then changing their address means updating 50 rows, and if you miss any, your data is now lying. The fix is to put the address in *one* place (the customer table) and let the orders just reference the customer's ID. Normalization is to databases what DRY (Don't Repeat Yourself) is to source code: a discipline that prevents bugs.
 
Redundancy isn't just wasted disk space. It's an open invitation for inconsistency. If a customer's address appears in 100 order rows and the customer moves, you have to update 100 rows perfectly or your data lies.
 
**Functional dependence** is the formal idea: column A *depends on* column B (`B → A`) if every value of B uniquely determines a value of A. A primary key, by definition, determines every other column.
 
**Metaphor: a spreadsheet that's grown out of control.** Imagine a single sheet that lists, in each row, an order *plus* the customer's full mailing address *plus* every product in the order. Every duplicated address and product description is a chance for a typo to drift over time. Normalization is the process of slicing that sheet into smaller sheets that reference each other by ID.
 
**Real-world scenario.** An early-stage e-commerce startup tracks orders in a single mega-spreadsheet: each row has the order ID, customer name, customer email, product name, product price, quantity. When they get acquired and migrate to a real database, the new team discovers 47 different spellings of "Anna Bergstrom" because every order rewrote her name fresh. They normalize: customers live in `Customers` (ID, name, email), products in `Products` (ID, name, price), and orders reference both by ID. Now Anna's name lives in one row, exactly correct, and no future "rename" can drift.
 
### The normal forms
 
**1NF (First Normal Form).** Atomic cells, plus a primary key.
- *Bad:* a `phones` column with `"555-1234, 555-5678"`.
- *Fix:* split phones into a separate table or rows.
**2NF (Second Normal Form).** 1NF, plus every non-key column depends on the *whole* primary key (no partial dependencies). Only relevant when you have a composite PK.
- *Bad:* `OrderItems(order_id, product_id, product_name)`. `product_name` depends only on `product_id`, not the whole PK.
- *Fix:* move `product_name` to the `Products` table.
**3NF (Third Normal Form).** 2NF, plus non-key columns depend only on the key, not on other non-key columns (no transitive dependencies).
- *Bad:* `Employee(emp_id, dept_id, dept_name)`. `dept_name` depends on `dept_id`, not on `emp_id`.
- *Fix:* move `dept_name` to a `Departments` table.
**BCNF (Boyce-Codd Normal Form).** For every dependency `A → B`, B must be unique. Catches edge cases 3NF misses (mostly involving overlapping candidate keys).
 
**The mnemonic:** *"Every non-key column depends on the key, the whole key, and nothing but the key."*
 
### Denormalization
 
Sometimes you intentionally *un*-normalize for performance, typically in analytics tables (data warehouses). The trade-off: faster reads, harder writes/updates, more storage.
 
**Real-world scenario for denormalization.** A streaming service like Hulu has a perfectly normalized OLTP database for managing accounts and subscriptions. But its analytics warehouse, used to answer "what genre is most popular in Texas on Friday nights?", denormalizes by pre-joining titles with their categories, regions, and time slices into one wide table. Reads against this denormalized table are 100× faster than re-joining at query time. The cost (that the wide table has to be rebuilt nightly from the normalized source) is acceptable because reports don't need to be real-time.
 
---
 
## 8. Storage and Indexing Internals
 
Indexes are like the back-of-the-book index in a textbook. Instead of reading every page to find mentions of "photosynthesis," you flip to the index, see "photosynthesis: pp. 17, 42, 88," and jump straight there. Database indexes do the same thing for table rows. They let the database skip past the 99.9% of rows that don't match your query. The trade-off is that maintaining the index slows down inserts and updates (someone has to keep it sorted), so you don't index everything. Picking the right indexes is one of the highest-leverage things a database engineer does for query performance.
 
### Storage media, three tiers of memory
 
| Medium | Volatile? | Cost / GB | Access time |
|---|---|---|---|
| RAM | Yes | High | ~100 ns |
| SSD | No | Mid | ~100 µs |
| HDD | No | Low | ~10 ms |
 
The orders-of-magnitude gap is huge. Disk is roughly **100,000× slower** than RAM. That's why almost every storage decision in a database is about *minimizing trips to disk*.
 
Data moves between disk and RAM in fixed-size **blocks** (often 8 KB or 16 KB). Inside the block, you can pack rows in two orientations:
 
- **Row-oriented.** Each block stores complete rows. Great for OLTP where you fetch a few rows entirely.
- **Column-oriented.** Each block stores one column for many rows. Great for analytics where you scan a few columns of millions of rows.
**Real-world scenario.** A mobile banking app like Chime needs to fetch your full account record (balance, recent transactions, settings) when you open the app. Row-oriented storage is perfect because it pulls all your columns in one disk read. Meanwhile, the bank's fraud-detection team running "what's the average transaction size for charges over $500 across all 12 million accounts?" benefits from column-oriented storage, because it only reads the `amount` column instead of pulling every row's entire payload off disk.
 
### Table structures
 
- **Heap.** Unordered. Fast inserts, slow lookups (full scan).
- **Sorted.** Rows physically ordered by a sort column. Great for range queries; expensive to maintain.
- **Hash.** Rows assigned to *buckets* by a hash function. O(1) point lookups, useless for ranges.
- **Cluster (multi-table).** Interleave related rows of *multiple* tables in the same blocks (e.g., Customer rows next to their Order rows). Great for joins on the cluster key.
### Indexes, the secret weapon
 
An **index** is a sorted auxiliary data structure that maps column values to row pointers. Reading the index is much faster than scanning the whole table, at the cost of slowing down writes (every insert/delete must update the index too).
 
**Metaphor: the book index.** A book's back-of-the-book index lets you find every mention of "Codd" in seconds, instead of reading every page. The cost: someone has to maintain the index, and re-print it if pages get added.
 
**Single-level index.** One file with `(value → pointer)` entries. Lookup uses **binary search**, halving the search space at every step (O(log n)). Good for tens of thousands of rows.
 
**Multi-level index, the B+tree.** A balanced tree where each path from root to leaf is the same length. Branching factor is the **fan-out**. Three or four levels can index *billions* of rows.
- **B+tree.** All values appear in the leaves; pointers to data blocks live only there. Internal nodes are only routing keys. Great for range scans (leaves are linked).
- **B-tree.** Values can appear at any level; no duplication.
| Variant | Definition |
|---|---|
| **Primary (clustering) index** | Table sorted by this index; only one per table |
| **Secondary (non-clustering) index** | Independent of physical order; many per table |
| **Dense index** | One entry per row |
| **Sparse index** | One entry per block (only possible on a sorted table) |
| **Hash index** | Buckets via hash function; fast point lookups, not ranges |
| **Bitmap index** | Grid of bits, one per (row × distinct value). Excellent for low-cardinality (e.g., gender, status flags) |
| **Function index** | Indexes the result of `f(col)`; useful for case-insensitive search like `INDEX UPPER(email)` |
| **R-tree** | B+tree variant where keys are bounding rectangles; used for spatial data |
 
**Selectivity (filter factor)** is the percentage of rows a query returns. Indexes are most useful when selectivity is **low** (returning a tiny slice). For a query that returns 80% of the table, a full scan is faster than an index lookup followed by 80% as many random reads.
 
`EXPLAIN` in MySQL shows you exactly what indexes the optimizer chose. Your most important tuning tool.
 
**Real-world scenario.** Amazon has billions of products. A search for "wireless earbuds under $50" needs to filter by category and price range. A B+tree index on `(category_id, price)` lets the database jump straight to the wireless-earbuds section and walk through prices in order, returning the first 20 results in milliseconds. Without that index, every query scans 4+ billion rows. The choice of *which* columns to combine in the index, and in what order, is the difference between a 50ms search and a 50-second timeout.
 
### Tablespaces, partitioning, sharding
 
- **Tablespace.** A logical container mapping tables to physical files.
- **Partition.** Splitting one table into pieces on the same machine.
  - *Horizontal* (rows by range, list, hash, key)
  - *Vertical* (columns)
- **Shard.** Like a partition, but pieces live on *different machines*. The basis of horizontal scaling for big data systems.
**Real-world scenario.** Twitter shards its tweet storage across thousands of machines, with each user's tweets routed to a specific shard based on user ID. When you load your timeline, the system fetches your tweets from your shard rather than searching every machine on Earth. Sharding is what allows a single logical table (all tweets ever) to scale beyond the storage of any single computer.
 
---
 
## 9. Transactions and ACID
 
A transaction is a group of database changes that either *all* happen or *none* happen, like a bank transfer where you can't have the money leave your account without arriving in the recipient's. ACID is the four-letter promise that databases make about transactions: the changes are **A**ll-or-nothing, the database stays **C**onsistent (no rules broken), concurrent transactions don't trample each other (**I**solation), and once committed, the changes survive crashes (**D**urable). These properties are the reason banks, payment systems, ticketing platforms, and inventory systems can be trusted not to lose your money or double-sell the last concert ticket.
 
A **transaction** is a sequence of operations that must commit (succeed) or rollback (fail) as a *single unit*. The canonical example is a bank transfer: debit account A *and* credit account B, never one without the other.
 
ACID:
- **Atomic.** All or none.
- **Consistent.** No rule violations at commit.
- **Isolated.** Concurrent transactions don't see each other's intermediate state.
- **Durable.** Committed data survives crashes.
**Metaphor: a wedding ceremony.** The vows, the rings, the kiss all happen atomically. If anything goes wrong before "I do", the ceremony is rolled back. Once "I do" is said, even an earthquake can't undo it (durable).
 
**Real-world scenario.** When you book an Uber, the transaction has multiple steps: charge your card, decrement the driver's available status, create a trip record, send notifications. ACID ensures you're never charged without getting a driver, and a driver is never marked "busy" without a real trip attached. If the payment processor hiccups halfway through, the entire transaction rolls back and you see "Payment failed, please try again" rather than waiting confused for a phantom driver while your card is debited.
 
### Read anomalies, what isolation prevents
 
- **Dirty read.** You read another transaction's *uncommitted* change. If they roll back, your read was based on data that never existed.
- **Non-repeatable read.** You read the same row twice in your transaction and get different values (someone else committed between).
- **Phantom read.** You re-run a `SELECT ... WHERE ...` and get *different rows* (someone inserted/deleted matching rows).
**Real-world scenario.** Imagine an e-commerce inventory check. You start a transaction, see "5 left in stock," and your customer hits Buy. If the database allowed dirty reads, you might be seeing inventory another transaction temporarily lowered to 5 but is about to roll back to 100. You'd reject good orders. Phantom reads matter when you're computing summaries: a financial report queries "all transactions over $10K today" twice and gets different counts because new transactions snuck in. Higher isolation prevents these, at the cost of more contention.
 
### Isolation levels, the cost/safety dial
 
| Level | Dirty | Non-rep. | Phantom |
|---|---|---|---|
| `READ UNCOMMITTED` | possible | possible | possible |
| `READ COMMITTED` | safe | possible | possible |
| `REPEATABLE READ` (MySQL default) | safe | safe | possible |
| `SERIALIZABLE` | safe | safe | safe |
 
Higher isolation means stronger correctness guarantees but more lock contention and lower throughput. Most apps run at READ COMMITTED or REPEATABLE READ. SERIALIZABLE is the gold standard for correctness-critical code.
 
### Schedules
 
A **schedule** is the interleaving of operations from concurrent transactions. Two operations *conflict* if reordering them changes the outcome (e.g., a write after another transaction's read).
 
- **Serial schedule.** Transactions run one after another, no interleaving. The reference for "correct."
- **Serializable schedule.** Interleaving is allowed, but the *result* is identical to *some* serial schedule. The goal of concurrency control.
---
 
## 10. Concurrency Control — Locks and Snapshots
 
When 1,000 people try to buy the last 10 tickets at the same instant, how does the database avoid selling the same ticket to 7 of them? The answer is concurrency control. The database briefly "locks" rows that one transaction is touching, forcing others to wait their turn, like a single-occupancy bathroom stall. Modern databases also have an alternative called snapshot isolation. Each transaction works on its own private copy of the data and only checks for conflicts at the end, like everyone editing their own copy of a Google Doc that's then merged. Concurrency control is what makes ticketing platforms, flash sales, and bidding systems trustworthy.
 
### Two-phase locking (2PL)
 
The classical approach. Each transaction acquires **shared (S)** locks for reads and **exclusive (X)** locks for writes. Locking happens in two phases:
 
1. **Grow phase.** Acquire locks (no releases yet).
2. **Shrink phase.** Release locks (no acquisitions).
**Strict 2PL.** Hold X locks until commit/rollback (prevents cascading rollbacks).
**Rigorous 2PL.** Hold *all* locks until commit/rollback (simpler reasoning).
 
**Real-world scenario.** Ticketmaster's pre-sale for a Taylor Swift tour: thousands of fans hit "buy" simultaneously. The database places exclusive locks on individual seat rows during each purchase attempt. If two fans race for seat A23, only one wins. The other's transaction waits, sees the seat is taken, and offers a different one. Without locking, both could think they got A23 and the system would have to apologize to one of them later. The brief lock is the price of correctness.
 
### Deadlock, the dining-philosophers problem
 
When two transactions each hold a lock the other needs, neither can proceed. Strategies:
 
- **Aggressive locking.** Grab everything upfront; wait if you can't.
- **Data ordering.** Always acquire locks in a fixed global order (kills cycles by construction).
- **Timeout.** If you wait too long, roll back.
- **Cycle detection.** Periodically check the wait-for graph and abort the cheapest victim.
**Real-world scenario.** A bank has two ATM transactions running. Transaction A is transferring $100 from account X to Y. Transaction B is transferring $50 from Y to X. A locks X first, B locks Y first. Now A wants Y (held by B) and B wants X (held by A): deadlock. The database detects the cycle and kills the cheaper transaction (often the one that's done less work), rolls it back, and retries it. To the user, the transaction "took a moment longer." To the engineer, this is the routine reality of concurrent banking.
 
### Snapshot isolation, optimistic concurrency
 
Instead of locks, give each transaction a *private snapshot* of the database at its start time. Concurrent writers don't block each other. Conflicts are detected at commit time. Most modern databases (PostgreSQL, Oracle) use snapshot isolation by default.
 
**Trade-off:** snapshot isolation alone *isn't* serializable (it allows write skew). True serializable behavior requires *Serializable Snapshot Isolation* (SSI), which adds conflict detection on top.
 
**Real-world scenario.** Google Docs feels like it uses something snapshot-like. You and your collaborator type at the same time, each on your own version, and the server merges your edits. PostgreSQL works similarly: each long-running analytic query can run on its own consistent snapshot of the database without blocking customer writes. The reporting team gets stable numbers; the customers experience no slowdown. Compare to the lock-based world where a long report could block all writes for minutes.
 
---
 
## 11. Recovery and Backup
 
Recovery is the database's ability to come back to a correct state after a crash, power loss, or software failure. The trick is that the database keeps a journal (a log of every change in the order it happened) *before* applying changes to the actual data files. After a crash, the database replays the journal: redo the changes from completed transactions, undo the ones that were in-flight. Backups are the second line of defense: full or incremental copies of the data, ideally stored somewhere other than the running database. Together, recovery and backups are why a database can tell you with confidence "your committed data is safe."
 
The **recovery log** (also called write-ahead log, WAL) is the database's notebook. Every change is recorded *before* it's applied to disk. The log contains:
 
- **Update records.** What was changed.
- **Compensation/undo records.** What was reversed.
- **Transaction records.** Start, commit, rollback.
- **Checkpoint records.** Markers showing all earlier dirty data is now durable.
**Real-world scenario.** A hospital's electronic medical records server loses power mid-update, say while a nurse was charting a medication administration. When the server boots back up, it reads the recovery log to figure out: "Was that medication entry committed before the crash? If yes, redo it. If it was still in-flight, undo any partial writes so the database is consistent." The patient never sees a half-saved chart. Without WAL-style recovery, a single power blip could corrupt patient data and put lives at risk.
 
### Recovery procedure
 
After a crash:
1. **Redo phase.** Replay every committed transaction since the last checkpoint.
2. **Undo phase.** Reverse every uncommitted transaction.
This is called **ARIES recovery** (Algorithm for Recovery and Isolation Exploiting Semantics). Most relational DBs use a variant of it.
 
### Backups
 
- **Cold backup.** Pause the database, copy files. Simple, but downtime.
- **Hot backup.** Maintain a synchronized secondary that you can copy from while primary stays online. No downtime.
- **Availability** is the percentage of time the system is up. Three nines (99.9%) ≈ 8.7 hours of downtime per year; five nines (99.999%) ≈ 5 minutes.
**Real-world scenario.** A small accounting firm runs a cold backup every Sunday at 3 a.m. They shut the database down for 20 minutes and copy the files to an external drive. A global payment processor like Stripe absolutely cannot afford that downtime, so they run hot backups continuously: a secondary replica is always streaming the WAL from the primary, ready to take over within seconds if the primary fails. The cost difference between these strategies, measured in millions of dollars of infrastructure, is justified by what an outage would cost the business.
 
---
 
## 12. Distributed Databases and CAP
 
When a database grows beyond what a single computer can handle (in storage, requests per second, or geographic reach), you spread the data across many computers. That's powerful but introduces a hard truth called the CAP theorem. When computers can't talk to each other across the network (a "partition"), you have to choose between giving stale answers (sacrificing **C**onsistency) or refusing to answer at all (sacrificing **A**vailability). You can't have both. This trade-off shapes the architecture of every globally-scaled product you use, from Instagram to your bank's mobile app.
 
When data outgrows one machine, you have options:
 
- **Parallel database.** Multiple processors, one machine (shared memory / shared storage / shared nothing).
- **Cluster.** Multiple machines on a LAN, coordinated.
- **Distributed database.** Multiple machines on a WAN.
- **Replicated database.** Copies of the data on multiple machines.
**Real-world scenario.** Uber operates in over 70 countries. Their dispatch database can't live in one data center. It'd be too slow for drivers in Singapore to query a database in Virginia, and a single outage would shut down the world. Instead, Uber distributes data: each region has its own primary, replicating to others asynchronously. When you book a ride, the local cluster handles it. When Uber HQ wants global analytics, they aggregate across regions overnight.
 
### Two-phase commit (distributed transactions)
 
When a transaction touches multiple nodes, a **transaction coordinator** runs a two-phase protocol:
 
1. **Phase 1 (Prepare):** coordinator asks all nodes "can you commit?" Each node persists the change to its log and replies yes/no.
2. **Phase 2 (Commit/Abort):** if everyone said yes, coordinator says "commit"; otherwise "abort." Each node finalizes.
The cost: a coordinator failure mid-protocol can leave nodes blocked. This is why distributed transactions are expensive and many systems prefer **eventual consistency** instead.
 
### Replication strategies
 
- **Primary/secondary.** One writer, many readers. Updates flow primary to secondaries.
- **Group replication.** Any node can take writes; conflicts resolved at commit.
Updates are **synchronous** (all replicas in one transaction) or **asynchronous** (apply locally, propagate after).
 
### CAP theorem
 
In a network partition (some nodes can't reach others), a distributed system can preserve at most **two** of:
- **Consistency.** Every read sees the latest write.
- **Availability.** Every request gets a response.
- **Partition-tolerance.** The system keeps running despite dropped messages.
**The forced choice.** Because partitions *will* happen in any real distributed system, you really pick between Consistency and Availability when partitioned. CP systems (e.g. classical relational with 2PC) refuse to serve stale reads. AP systems (e.g. Cassandra, Dynamo) keep serving but may return stale data. **Eventual consistency** is the AP relaxation: replicas converge eventually if updates stop.
 
**Real-world scenario.** Picture a worldwide game like Fortnite. The leaderboard is an AP system: if a network partition splits North America from Europe for 30 seconds, both regions keep showing slightly stale scores rather than refusing to serve the leaderboard at all. Players don't notice the brief inconsistency, and the lists reconcile within seconds. Compare to a banking system, which is CP: during a partition, the bank refuses to authorize a withdrawal in Tokyo if it can't confirm with the home server in New York that you didn't already withdraw your last $1,000 in London. Better to delay than to risk a double-spend.
 
### Cloud database tiers
 
- **IaaS (Infrastructure-as-a-Service).** You rent VMs; install your own DB.
- **PaaS (Platform-as-a-Service).** Managed DB service (RDS, Cloud SQL).
- **SaaS (Software-as-a-Service).** Full apps; DB is hidden under the hood.
**Real-world scenario.** A startup launches an MVP with Heroku Postgres (PaaS). They don't want to spend engineering time on database operations, just write app code. Two years later, with billions of records, they migrate to AWS RDS (PaaS, but more configurable). Eventually they outgrow even RDS and move to self-hosted PostgreSQL on EC2 (IaaS) so they can fine-tune the storage layer. This progression from managed to less managed mirrors the typical scaling story of most tech companies.
 
---
 
## 13. Data Warehouses and Analytics
 
Your day-to-day operational database (orders coming in, accounts being created) is optimized for fast small writes, but it's terrible for big-picture questions like "what did we sell by region last quarter?" The fix is a separate database called a data warehouse, designed only for analytics. It gets a copy of operational data, organized differently (think one giant flat sales fact table surrounded by descriptive lookup tables, a "star schema"), and queries that would crush the live database fly through it. This separation is why business dashboards never slow down the e-commerce site you're shopping on.
 
**Operational data** runs the business: small transactions, current state. **Analytic data** explains the business: long-running queries, historical trends. The two have opposing requirements, so we usually separate them. Operational data lives in **OLTP** databases; analytic data lives in **data warehouses** (OLAP).
 
**Real-world scenario.** A retail chain like Target has an OLTP database that handles point-of-sale transactions (quick reads and writes for "did this customer's credit card succeed?"). Their analytics team needs to ask "across all 1,900 stores, how did winter coat sales correlate with weather patterns?" Running that query on the OLTP database during a weekend rush would slow checkout to a crawl and possibly take it down. Instead, an ETL pipeline copies sales nightly into a data warehouse where the analytics team can run multi-hour queries without affecting a single shopper.
 
### Star schema
 
A data warehouse usually uses a **dimensional design** (a.k.a. **star schema**):
- A central **fact table** holds quantitative measures (sales revenue, units sold) and FKs.
- Surrounding **dimension tables** hold descriptive attributes (date, product, region, customer).
A **dimension hierarchy** is a chain like Day → Month → Quarter → Year, encoded as columns in the dimension table.
 
**Date and time dimensions** are nearly always pre-built: 36,500 rows for 100 years of dates, 1,440 rows for minutes of a day.
 
### ETL — Extract, Transform, Load
 
The pipeline that moves operational data into the warehouse:
1. **Extract** from source systems.
2. **Transform** (clean, deduplicate, conform formats).
3. **Load** into the warehouse.
**Real-world scenario.** An insurance company's ETL pipeline pulls policy data from one OLTP database, claims data from another, and customer demographic data from a third (often in different formats and even different vendors). The Transform step harmonizes them (same customer ID conventions, same date formats, same currency conversions) and the Load step writes everything into the warehouse's star schema. By morning, the actuaries and risk team have fresh, unified data to query without ever touching the operational systems.
 
### Data lake
 
A **data lake** is the raw, schema-less precursor to a warehouse: store everything, structure it later. Modern setups often combine both (sometimes called "lakehouse").
 
---
 
## 14. Complex Types and Object-Relational
 
The original SQL world had simple types: numbers, strings, dates. But the real world has nested data. A customer's "addresses" might be a list of multiple addresses, a product might have a JSON blob of optional specs, a delivery zone might be a polygon on a map. Modern databases support these complex types directly so you don't have to flatten everything into flat columns. The choice between relational tables, JSON columns, and other complex types is one of the most consequential design decisions for any modern app. Get it wrong and you're rewriting in two years.
 
The relational model started with simple types (numbers, strings, dates). Modern SQL extends this:
 
### Collections
- **Set.** Unordered, no duplicates.
- **Multiset.** Unordered, allows duplicates.
- **List.** Ordered, allows duplicates.
- **Array.** An indexed list.
### Documents, XML and JSON
 
Data can be **structured** (fixed schema), **semistructured** (named elements but variable shape, like JSON, XML), or **unstructured** (free text, images).
 
```sql
SELECT JSON_EXTRACT(data, '$.user.email') FROM events;
```
 
**Real-world scenario.** An IoT company collects sensor data from millions of devices. Each device sends a slightly different JSON payload depending on model and firmware version. Forcing all those variations into rigid columns would mean schema migrations for every new device. Instead, they store the raw JSON in a single `payload` column and use `JSON_EXTRACT` to query specific fields. New devices integrate without database changes. The trade-off: the database can't enforce schema rules on the JSON fields. That responsibility shifts to the application.
 
### Spatial, geometry meets databases
 
Spatial types store geometric shapes specified in **WKT (Well-Known Text)** format like `POINT(1 2)`, `LINESTRING(0 0, 1 1)`, `POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))`.
 
Spatial indexes use **R-trees** indexed by **minimum bounding rectangles (MBRs)**, the smallest axis-aligned rectangle containing the shape.
 
Common functions: `ST_Distance()`, `ST_Area()`, `ST_Overlaps()`, `ST_Union()`.
 
**Real-world scenario.** A food delivery app like DoorDash uses spatial types for everything. Each restaurant's location is a `POINT`, each delivery zone is a `POLYGON`, each driver's current position updates every few seconds. When you open the app, a query like "find restaurants within 5 km of my POINT, that I'm inside their delivery POLYGON" runs in milliseconds thanks to R-tree spatial indexes. Without dedicated spatial types, the app would have to load every restaurant and compute distances in application code, which is far slower and more expensive.
 
### Object-relational
 
Modern databases support:
- **Composite types.** Bundle several properties.
- **Methods.** Functions attached to types.
- **Subtype/supertype.** Inheritance with `CREATE TYPE UNDER`.
- **Override.** Subtype redefines a supertype method.
An **object-relational mapping (ORM)** like SQLAlchemy or Hibernate translates between application objects and relational rows automatically.
 
**Real-world scenario.** A Django web app for a publishing company has a `Book` Python class. The Django ORM transparently turns `book.save()` into the right SQL `INSERT`, and `Book.objects.filter(author='Le Guin')` into `SELECT * FROM books WHERE author = 'Le Guin'`. The developer writes Python; the ORM handles SQL. The risk is that naive ORM use can produce N+1 query problems: looping over 100 books and triggering 100 separate `SELECT` statements for each book's author. Knowing when to bypass the ORM with a hand-tuned query is the mark of a good engineer.
 
---
 
## 15. NoSQL — When Tables Aren't Enough
 
NoSQL is a family of databases that ditch SQL's strict table structure in exchange for scale, flexibility, or specialized power. Want to handle 100 million simple key-value lookups per second? Use Redis. Want to store JSON documents that can have any shape? Use MongoDB. Want to store relationships in a way that walking from "friend of a friend" is instant? Use a graph database like Neo4j. These aren't replacements for relational databases. They're complements, used alongside the main database for the workloads relational systems aren't optimal for.
 
NoSQL ("not only SQL") covers four broad models, born from the need to scale *horizontally* across cheap commodity hardware:
 
### Key-value stores (Redis, DynamoDB)
The simplest model: the database is a giant hash map. Excellent for caching, session storage, leaderboards. No querying beyond "give me the value for this key."
 
**Real-world scenario.** When you log into Facebook, your session data (user ID, language preferences, recent activity) lives in Redis. Every page request needs that data instantly. Facebook isn't going to query a relational database every time. The round-trip would add 50ms to every page. Instead, Redis serves it in under 1ms. The data is also expendable: if Redis crashes, sessions get re-created from cookies. That tolerance for ephemerality is what unlocks the speed.
 
### Wide-column stores (Cassandra, HBase, Bigtable)
Like a key-value store where each value is a *column family*, a flexible bag of named columns. Excellent for time-series, IoT, large-scale logging. Inherits Bigtable's sparse-column-with-timestamps idea (each cell is versioned by timestamp).
 
**Real-world scenario.** Netflix uses Cassandra to store viewing history for hundreds of millions of users: billions of rows, growing constantly. Each user's history is a row keyed by user ID, with columns for each title they've watched, timestamped. Reads are blazing fast because each user's data lives on a known shard. Cassandra's "tunable consistency" lets Netflix accept eventually-consistent writes (you might miss your most recent view for a few seconds) in exchange for massive write throughput.
 
### Document stores (MongoDB, Couchbase)
Values are JSON/XML documents, schema-flexible. Each document is self-contained, like a row that *includes* its related rows. Sharding is by a chosen **shard key**; replication uses **primary replica + secondary replicas**.
 
**Real-world scenario.** A content management system for a magazine like The New Yorker stores each article as a document: title, author info, body, comments, tags, all together. Articles can have wildly different structures (a multimedia feature has video embeds; a quick blog post doesn't). MongoDB handles the variation naturally. Compared to a relational design with separate `articles`, `authors`, `comments`, `tags` tables joined every time, the document approach loads an article in one read and adapts to new article types without schema migrations.
 
### Graph databases (Neo4j, JanusGraph)
Vertices (nodes) and edges (links), each with properties. Excellent for highly-connected data: social networks, recommendation engines, fraud rings.
 
The killer feature is **index-free adjacency**: each vertex stores direct pointers to its neighbors. Walking the graph is O(degree) instead of repeated index lookups. (Compare with SQL, where each "step" through a many-many relationship requires a join.)
 
**Real-world scenario.** LinkedIn's "people you may know" feature traverses your second- and third-degree network, friends of friends of friends. In a relational model, that's three giant joins on a connections table, possibly across hundreds of millions of rows. In a graph database, it's "starting at this user, walk three hops out": O(degree³) in time, often returning in a few hundred milliseconds. Fraud detection at credit-card companies uses graph databases similarly to find suspicious patterns like "multiple cards used at the same merchant by people sharing addresses."
 
### Vertical vs horizontal scaling
 
- **Vertical (scale up).** Bigger machine. Capped by hardware limits.
- **Horizontal (scale out).** More machines. Requires sharding/replication. The NoSQL value proposition.
NoSQL gives up some relational guarantees (joins, ACID across documents) in exchange for horizontal scale and schema flexibility.
 
---
 
## 16. Database Programming Patterns
 
Database programming is how you actually wire your application code (Python, Java, JavaScript, etc.) to the database. The dominant pattern today is the driver/API approach: your app uses a library to send SQL strings and receive results back. Older code may have SQL embedded directly in the source, translated by a precompiler. And you can also push logic *into* the database via stored procedures and triggers that run server-side. Each approach has trade-offs around portability, performance, and where the business logic lives.
 
### Three approaches to talking to a database
 
1. **Embedded SQL.** Write SQL inline in C/Java; a precompiler translates it. Dated approach, used in legacy systems.
2. **Procedural SQL.** Stored procedures, functions, and triggers. Code lives *inside* the database.
3. **API (driver) approach.** Modern default. Your app uses a driver library (JDBC, Connector/Python, ADO.NET, PDO) to send query strings.
**Real-world scenario.** A bank's mainframe system from the 1980s might still use embedded SQL in COBOL. Every change requires a recompile, but it runs at metal speeds. A retail chain's back-office stored procedures handle nightly inventory reconciliation right inside the database, where they can iterate over millions of rows without network round trips. A modern Django web app uses the API approach with the `psycopg2` driver: flexible, easy to test, and language-agnostic on the database side. Most companies end up using all three patterns somewhere, suited to different use cases.
 
### Stored procedures and triggers
 
A **stored procedure** is a named program living in the database, called by `CALL`. It can have IN/OUT/INOUT parameters, control flow (`IF`, `WHILE`), and cursors.
 
A **stored function** returns a single value and can be used inside SQL expressions.
 
A **trigger** fires automatically on `INSERT`/`UPDATE`/`DELETE` events. Useful for auditing or enforcing rules that go beyond constraints. Be cautious: triggers add hidden behavior and can complicate debugging.
 
**Real-world scenario.** A hospital wants every change to a patient's medication list to be auditable: who changed it, when, and what changed. A trigger on the `medications` table fires on every UPDATE and writes a row to `medications_audit` with the user, timestamp, old value, and new value. The application code doesn't need to remember to log; the database guarantees it on every write. Regulators love this. Debugging gets trickier, though: when a developer wonders "why is this row in the audit table?", they have to know about the trigger to find the cause.
 
### Cursors
 
A **cursor** is a pointer into a result set you can step through one row at a time. In application code, you usually fetch rows with `cursor.fetchone()` or `cursor.fetchall()`.
 
### SQL injection, the eternal threat
 
The most common database security flaw: when user input is concatenated into a query string. Example of bad code:
 
```python
# DON'T DO THIS
cur.execute("SELECT * FROM users WHERE name = '" + user_name + "'")
```
 
If `user_name` is `' OR '1'='1`, the query becomes `SELECT * FROM users WHERE name = '' OR '1'='1'`, returning every user.
 
Always parameterize:
```python
cur.execute("SELECT * FROM users WHERE name = %s", (user_name,))
```
 
The driver sends value and SQL separately. The value can never be interpreted as code.
 
**Real-world scenario.** In 2008, Heartland Payment Systems was breached partly through SQL injection, exposing 134 million credit cards. In 2011, Sony's PlayStation Network was hit similarly. These weren't exotic attacks. They were the same trick covered in any intro security class, exploiting code that concatenated user input into SQL. The fix has always been: never trust user input, always parameterize. Every modern driver supports it. The only reason injection still happens is developer shortcuts.
 
### ORMs
 
An **object-relational mapping** library (SQLAlchemy, Django ORM, Hibernate, Entity Framework) lets you work with classes/objects instead of SQL strings. Trade-offs: less boilerplate, but you must understand the SQL it generates. Naive ORM use produces "N+1 query" problems where each iteration over a collection issues another query.
 
**Real-world scenario.** A team building an internal HR portal in Ruby on Rails uses ActiveRecord (an ORM) to write `Employee.where(department: 'Engineering').includes(:manager)`: clean Ruby code that handles the SQL behind the scenes. The `.includes(:manager)` is the magic word. It tells the ORM to fetch all managers in one extra query, not one query per employee. Without it, listing 500 employees would mean 501 database round trips and a frustrated user waiting for a slow page. Knowing how to coax efficient SQL out of your ORM is a hugely high-leverage skill.
 
---
 
# Part III — Glossary
 
**ACID:** Atomic, Consistent, Isolated, Durable. The four properties of a reliable transaction.
 
**Aggregate function:** A function that summarizes many rows into one value: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
 
**Alias:** A temporary name assigned to a column or table, declared with `AS`.
 
**ALTER TABLE:** DDL statement that changes a table's schema (add/drop/modify columns).
 
**API (Application Programming Interface):** A library exposing procedures or classes for accessing a service like a database.
 
**Array:** Ordered, indexed collection allowing duplicates.
 
**Artificial key:** A simple PK created by the designer (usually an auto-incrementing integer).
 
**Asynchronous update:** Update applied locally first and propagated later.
 
**Atomic transaction:** One that fully completes or fully reverses; never partial.
 
**AUTO_INCREMENT:** MySQL keyword for auto-generated integer PKs.
 
**Available:** In the CAP sense, every "live" node responds to queries.
 
**B-tree / B+tree:** Balanced multi-level index structures. B+ trees keep all values in the leaves.
 
**Base table:** A table referenced in a view query's `FROM` clause.
 
**BCNF (Boyce-Codd Normal Form):** Stricter than 3NF: for every dependency `A → B`, B must be unique.
 
**Big data:** Data of unprecedented volume and variability, often loosely structured.
 
**Binary search:** O(log n) search on a sorted index.
 
**Bitmap index:** Grid of bits, one per (row × distinct value). Excellent for low-cardinality columns.
 
**Block:** Uniform unit of data movement between disk and RAM.
 
**Buffer / buffer manager:** Main-memory cache of recently-used disk blocks.
 
**Bucket:** A block (or group) holding rows or index entries assigned by a hash function.
 
**Cache manager:** Caches reusable query data in RAM.
 
**Candidate key:** Any column or column-set that is unique and minimal; the PK is one chosen candidate.
 
**CAP theorem:** A distributed system can guarantee at most two of Consistency, Availability, Partition-tolerance.
 
**Cardinality:** Min/max counts in an ER relationship.
 
**CASCADE:** Referential-integrity action: propagate the change.
 
**Catalog (data dictionary):** Directory of database objects (tables, columns, indexes).
 
**Cell:** One column of one row.
 
**Checkpoint:** Marker saying all earlier dirty data is now durable on disk.
 
**CHECK constraint:** Rejects values where the expression is FALSE.
 
**Cluster:** A group of nodes connected by LAN, coordinated by cluster software.
 
**Column:** A named attribute of a table with a specific data type.
 
**Column-oriented (columnar) storage:** Each block stores values for one column. Optimized for analytics.
 
**COMMIT:** Save the current transaction's changes permanently.
 
**Compensation/undo record:** Log entry written when a transaction rolls back.
 
**Composite primary key:** A PK made of multiple columns.
 
**Concurrency system:** Component that manages concurrent transactions (locks, snapshot isolation).
 
**Conditional logical operator:** TRUE / FALSE / NULL three-valued logic.
 
**Connection:** A live link between an application and a DB server.
 
**Consistency (in ACID):** All rules valid at commit.
 
**Consistency (in CAP):** Every read sees the latest write.
 
**Constraint:** A rule the database enforces (NOT NULL, UNIQUE, PK, FK, CHECK).
 
**Correlated subquery:** Subquery whose `WHERE` references the outer query.
 
**CRUD:** Create, Read, Update, Delete.
 
**Cursor:** Pointer into a result set; lets you step through rows one at a time.
 
**Data dictionary:** See Catalog.
 
**Data independence:** Logical/physical schema changes don't require app changes.
 
**Data lake:** Raw, unprocessed analytic store.
 
**Data mart:** Warehouse for one business area.
 
**Data warehouse:** Database optimized for analytics, separated from OLTP.
 
**Database administrator (DBA):** Person managing access, security, and operations.
 
**Database system / DBMS:** Software that manages databases.
 
**Deadlock:** Cycle of transactions each blocked waiting for locks held by others.
 
**Denormalization:** Intentionally adding redundancy for performance.
 
**Dense index:** One entry per table row.
 
**DESC:** Descending order in `ORDER BY`.
 
**Difference (–):** Set operation: rows in A not in B.
 
**Dimensional design (star schema):** Fact table at center, dimension tables around it.
 
**Dirty read:** Reading uncommitted data from another transaction.
 
**Distributed database:** DB spread across machines on a WAN.
 
**Document database:** NoSQL category storing JSON/XML documents.
 
**Document type:** Column type holding XML or JSON.
 
**Driver:** Client library for talking to a specific database.
 
**DROP:** DDL verb that deletes objects.
 
**Durable:** Committed changes survive failures.
 
**Dynamic SQL:** SQL strings built at runtime.
 
**Edge:** A connection between graph vertices.
 
**Embedded database:** A DB packaged inside an application process (e.g., SQLite).
 
**Embedded SQL:** SQL coded inline in a host language; processed by a precompiler.
 
**Entity:** A thing being modeled.
 
**Entity-relationship (ER) model:** High-level data model.
 
**Equijoin:** Join using `=`.
 
**Equivalent algebra expressions:** Two expressions that always produce the same result.
 
**ETL:** Extract-Transform-Load pipeline for data warehouses.
 
**Eventual consistency:** Replicas eventually converge if updates stop.
 
**Exclusive (X) lock:** Write lock; blocks all other readers and writers.
 
**EXPLAIN:** MySQL statement showing the optimizer's chosen plan.
 
**Fact table:** Center of a star schema; numeric measures + FKs.
 
**Fan-out:** Number of index entries per node in a tree index.
 
**Federated database:** Middleware-coordinated collection of autonomous DBs.
 
**Filter factor / selectivity:** Fraction of table rows a query returns.
 
**First normal form (1NF):** Atomic cells + primary key.
 
**Flattening:** Rewriting a subquery as a join.
 
**Foreign key (FK):** Column that references a PK in another table.
 
**FROM:** Clause naming the table(s) a query reads.
 
**Full join:** Returns all rows from both tables, with NULLs where no match.
 
**Function index:** Index over `f(col)` rather than `col`.
 
**Functional dependence:** `B → A` means B determines A.
 
**Fuzzy checkpoint:** Checkpoint that doesn't pause processing.
 
**Glossary / data dictionary / repository:** Companion document to an ER diagram defining each entity, relationship, and attribute in detail.
 
**GRANT:** DCL verb granting permissions.
 
**Graph database:** NoSQL category modeling vertices + edges.
 
**GROUP BY:** Clause that groups rows for aggregate functions.
 
**HAVING:** Filter on aggregated groups (post-aggregation).
 
**Hash index:** Index using a hash function to assign entries to buckets.
 
**Hash partition:** Partitioning by `hash(col) % N`.
 
**Heap table:** Unordered table.
 
**Heterogeneous databases:** Multiple databases with different DBMSes or schemas.
 
**Hit ratio:** See Filter factor.
 
**Horizontal partition:** Subset of rows.
 
**Horizontal scaling:** Adding more machines.
 
**Host language:** Language hosting embedded SQL (C, Java).
 
**Identifying entity / relationship:** Strong entity that supplies identity to a weak entity via a relationship.
 
**Identifying attribute:** Unique, singular, required attribute of a strong entity.
 
**IDEF1X:** A formal ER notation popular in defense and government.
 
**Index-free adjacency:** Graph-database property: each vertex stores pointers to neighbors directly.
 
**Index scan:** Using an index instead of full table scan.
 
**INNER JOIN:** Returns matching rows only.
 
**Instance:** A specific value or thing (entity instance, attribute instance).
 
**Intersect (∩):** Set operation: rows present in both compatible tables.
 
**IsA relationship:** Inheritance relationship between subtype and supertype.
 
**Isolation:** Property that concurrent transactions don't interfere.
 
**JDBC:** Java database connectivity API.
 
**Join:** Operation combining rows from two tables based on a condition.
 
**JSON:** JavaScript Object Notation, used for semistructured data.
 
**Junction (bridge) table:** Implements a M:N relationship with composite PK of two FKs.
 
**Key partition:** Hash partition with a DB-managed expression.
 
**Least Recently Used (LRU):** Buffer eviction policy.
 
**LEFT JOIN:** Returns all left rows + matching right rows (NULLs when none).
 
**LIKE:** Wildcard pattern match: `%` (any chars), `_` (one char).
 
**LIMIT:** Restricts the number of rows returned.
 
**List partition:** Partitioning by an explicit list of values.
 
**Local area network (LAN):** Network within one facility (Ethernet).
 
**Local transaction:** Transaction touching only one node of a distributed DB.
 
**Lock:** Permission to read/write a piece of data.
 
**Lock manager:** DB component tracking locks.
 
**Log:** Append-only record of database changes.
 
**Logical design:** Translating ER model into tables, columns, keys.
 
**Logical index:** Index whose pointers are PK values, not block addresses.
 
**Many-to-many (M:N) relationship:** Implemented with a junction table.
 
**Materialized view:** View whose result is physically stored.
 
**Middleware:** Software layer between applications and database.
 
**Minimum bounding rectangle (MBR):** Smallest axis-aligned rectangle containing a spatial value.
 
**MongoDB:** Leading document NoSQL database.
 
**Multi-level index:** Hierarchical index (e.g., B-tree).
 
**Multi-tier architecture:** Layers of computers (UI → app → data).
 
**Multiset:** Unordered collection allowing duplicates.
 
**Network partition:** Network failure splitting cluster into isolated groups.
 
**Node:** A computer in a distributed system.
 
**Non-equijoin:** Join with operators other than `=`.
 
**Non-key column:** A column not part of any candidate key.
 
**Non-repeatable read:** Re-reading a row gives a different value.
 
**Non-volatile memory:** Retains data without power.
 
**NoSQL:** Non-relational databases optimized for big data.
 
**NOT NULL:** Constraint disallowing NULL.
 
**NULL:** Special value meaning unknown or inapplicable.
 
**ODBC:** Open database connectivity API.
 
**ON UPDATE / ON DELETE:** FK clauses specifying RI action.
 
**Operational data:** Daily transactional data.
 
**Optimistic concurrency:** Run without locks; resolve conflicts at commit.
 
**Optional / required:** Cardinality minimum 0 vs minimum 1.
 
**Order by:** Sort the result set.
 
**Outer join:** LEFT, RIGHT, or FULL JOIN.
 
**Override:** Subtype method that redefines supertype's behavior.
 
**Parallel database:** DB on a multi-CPU machine or cluster.
 
**Parameter:** A value passed into a stored procedure / prepared statement.
 
**Parser:** Component that checks SQL syntax.
 
**Partition:** Subset of a table on the same machine.
 
**Partition expression:** Function determining which partition each row goes to.
 
**Partition-tolerant:** Continues operating during network splits.
 
**Phantom read:** Re-running a query returns different rows.
 
**Physical design:** Indexes, table structures, partitions.
 
**Physical index:** Index whose pointers are direct block addresses.
 
**PL/SQL:** Oracle's procedural extension to SQL.
 
**Plural attribute:** An attribute with cardinality > 1; implemented in a separate table.
 
**Precompiler:** Translates embedded SQL to host-language calls.
 
**Primary key (PK):** Unique, NOT NULL, minimal identifier of a row.
 
**Primary index (clustering index):** Index on the sort column.
 
**Procedural SQL:** SQL extended with control flow (SQL/PSM, PL/SQL).
 
**Project (π):** Algebra operation choosing columns.
 
**Property (in graph DB):** Information attached to vertices/edges.
 
**Query language:** Computer language for writing DB queries.
 
**Query optimizer:** Picks the cheapest execution plan.
 
**Query parser:** Validates SQL syntax.
 
**Query plan / execution plan:** Step-by-step recipe to run a query.
 
**R-tree:** Spatial index using minimum bounding rectangles.
 
**Range partition:** Partitioning by ranges of values.
 
**READ COMMITTED / READ UNCOMMITTED / REPEATABLE READ / SERIALIZABLE:** Four standard isolation levels.
 
**Recovery log:** Sequential log of database operations.
 
**Recovery system:** Component enforcing atomicity and durability.
 
**Redo phase:** Re-applying committed transactions during crash recovery.
 
**Reflexive relationship:** Entity related to itself.
 
**Referential integrity:** FK values either fully NULL or matching some PK.
 
**Relation:** A table (in formal terms).
 
**Relational algebra:** Formal algebra of operations on relations.
 
**Relational model:** Codd's tabular data model.
 
**Relationship:** Statement linking two entities.
 
**Replica:** A copy of data.
 
**Replicated database:** DB maintaining 2+ replicas.
 
**REPLACE:** MySQL extension for upsert behavior.
 
**REVOKE:** DCL verb removing permissions.
 
**RIGHT JOIN:** All right rows + matching left rows.
 
**Row:** A single record in a table.
 
**Row-oriented storage:** Each block contains complete rows.
 
**ROLLBACK:** Reverse the current transaction.
 
**Savepoint:** Named point inside a transaction you can roll back to.
 
**Schema:** The set of definitions (tables, columns, etc.) for a database.
 
**Schedule:** Sequential ordering of operations across concurrent transactions.
 
**Second normal form (2NF):** 1NF + every non-key depends on the whole key.
 
**Secondary index (non-clustering index):** Index that isn't on the sort column.
 
**Select (σ):** Algebra operation choosing rows.
 
**Selectivity:** See Filter factor.
 
**Self-join:** Joining a table to itself with aliases.
 
**Semistructured data:** JSON, XML. Named elements but variable shape.
 
**Sequence:** Auto-incrementing counter (in some DBs, called a sequence object).
 
**Serial schedule:** Transactions executed one at a time.
 
**Serializable schedule:** Equivalent to some serial schedule.
 
**Session:** A connection from start to disconnect.
 
**Set (collection):** Unordered, no duplicates.
 
**Shard:** Subset of data on a separate machine.
 
**Shared (S) lock:** Read lock allowing other readers.
 
**Single-level index:** Flat index of (value → pointer).
 
**Snapshot isolation:** Each transaction sees a private copy.
 
**Sorted table:** Rows physically ordered by sort column.
 
**Spatial type:** Geometric data (points, lines, polygons).
 
**Sparse index:** One entry per block (sorted tables only).
 
**SQL injection:** Attack where user input is concatenated into SQL.
 
**SQL/CLI:** Standard call-level interface to SQL.
 
**SQL/MED:** Standard for federated databases.
 
**SQL/PSM:** Standard for procedural SQL.
 
**SRID (Spatial Reference System Identifier):** Identifies a spatial coordinate system.
 
**Star schema:** See Dimensional design.
 
**Statement:** A complete executable SQL command.
 
**Static SQL:** Embedded SQL fixed at compile time.
 
**Storage engine / manager:** Translates query operations into low-level disk commands.
 
**Stored procedure / function / trigger:** Server-side compiled code.
 
**Strong entity / table:** Has its own identifying attribute.
 
**Structured data:** Fixed schema, named columns.
 
**Subquery:** Query inside another query.
 
**Subtype / Supertype:** Inherited and parent entities.
 
**Synchronous update:** All replicas updated in one transaction.
 
**System failure:** Crash that loses RAM but not disk.
 
**Tablespace:** Logical container mapping tables to files.
 
**Table cluster (multi-table):** Interleaved storage of related tables.
 
**Table scan:** Reading the whole table without an index.
 
**Theta join:** Join using a general comparison (`<`, `>`, etc.).
 
**Third normal form (3NF):** 2NF + non-key columns depend only on the key.
 
**Three-valued logic:** TRUE/FALSE/NULL.
 
**Tier:** Layer in a multi-tier architecture.
 
**Timeout:** Lock-waiting cap; transaction rolled back on expiry.
 
**Transaction:** A unit of work that commits or rolls back as a whole.
 
**Transaction boundary:** First or last statement of a transaction.
 
**Transaction coordinator:** Orchestrator of distributed transactions.
 
**Transfer rate:** Speed of data after initial access.
 
**Trigger:** Procedure that fires automatically on data changes.
 
**Triple store:** RDF graph database.
 
**Truth table:** Table defining 3-valued logic.
 
**Tuple:** Ordered collection of values; mathematical name for a row.
 
**Two-phase commit (2PC):** Distributed-transaction protocol (prepare, commit).
 
**Two-phase locking (2PL):** Concurrency protocol (grow, shrink).
 
**UML:** Unified Modeling Language; software-design notation that includes ER constructs.
 
**Undo phase:** Rolling back uncommitted transactions during crash recovery.
 
**UNION:** Set operation combining two compatible result sets.
 
**UNIQUE:** Constraint forbidding duplicate values.
 
**Unit of work:** See Transaction.
 
**Unstructured data:** Free text or binary data without named elements.
 
**Update record:** Log entry for a data change.
 
**User-defined type:** Type created with `CREATE TYPE`.
 
**User mapping:** In SQL/MED, association between federated and participating DB users.
 
**Vertex / Node:** In a graph DB, a hub connected by edges.
 
**Vertical partition:** Subset of columns.
 
**Vertical scaling:** Adding bigger CPUs/disks.
 
**View / view query / view table:** Saved `SELECT`; queried as if it were a table.
 
**Volatile memory:** RAM; lost when power-off.
 
**Weak entity / table:** Identified via a parent's identifying relationship.
 
**WHERE:** Filter rows before grouping.
 
**Wide-area network (WAN):** Network spanning multiple facilities.
 
**Wide-column database:** NoSQL category storing column families per row (Cassandra).
 
**WITH CHECK OPTION:** Reject view inserts/updates that violate the view's WHERE.
 
**WKT (Well-Known Text):** Textual format for spatial values.
 
**XML:** Extensible Markup Language; tag-based document format.
 
---
 
---
 
# Part IV — Documentation & Further Learning
 
### Official documentation
 
**ANSI / ISO SQL standard**
- Part 11 of ISO/IEC 9075 (SQL Schemas): https://www.iso.org/standard/63556.html (paywalled)
- A free informative summary: https://en.wikipedia.org/wiki/SQL:2016 (and its newer successors)
**MySQL** (the DB used in your zyBooks course)
- Reference manual: https://dev.mysql.com/doc/refman/8.0/en/
- Tutorial: https://dev.mysql.com/doc/refman/8.0/en/tutorial.html
- Performance Schema: https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html
- InnoDB storage engine: https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html
**PostgreSQL** (free, open-source, strong on standards compliance)
- Documentation: https://www.postgresql.org/docs/
- Tutorial: https://www.postgresql.org/docs/current/tutorial.html
- Indexes deep dive: https://www.postgresql.org/docs/current/indexes.html
**SQLite** (file-based, embeddable)
- Documentation: https://sqlite.org/docs.html
**Oracle Database**
- Documentation: https://docs.oracle.com/en/database/
**Microsoft SQL Server**
- T-SQL reference: https://learn.microsoft.com/en-us/sql/t-sql/
**MongoDB**
- Manual: https://www.mongodb.com/docs/manual/
- University (free courses): https://learn.mongodb.com/
**Cassandra**
- Documentation: https://cassandra.apache.org/doc/latest/
**Redis**
- Documentation: https://redis.io/docs/
**Neo4j**
- Documentation: https://neo4j.com/docs/
### Driver / API documentation
 
- **JDBC**: https://docs.oracle.com/javase/tutorial/jdbc/
- **MySQL Connector/J (Java)**: https://dev.mysql.com/doc/connector-j/8.0/en/
- **MySQL Connector/Python**: https://dev.mysql.com/doc/connector-python/en/
- **Python DB-API 2.0 (PEP 249)**: https://peps.python.org/pep-0249/
- **PHP PDO**: https://www.php.net/manual/en/book.pdo.php
- **ADO.NET**: https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/
### Books — classics and modern
 
- **"Database System Concepts":** Silberschatz, Korth, Sudarshan. The standard graduate textbook (you may already own it).
- **"Database Management Systems":** Ramakrishnan & Gehrke. Another foundational text.
- **"Designing Data-Intensive Applications":** Martin Kleppmann. Outstanding modern survey of how distributed systems and databases interact. Highly recommended for any CS Master's student.
- **"SQL Performance Explained":** Markus Winand. Very readable index-and-query-optimization guide. See also https://use-the-index-luke.com/ (free).
- **"The Data Warehouse Toolkit":** Ralph Kimball. The canonical book on dimensional modeling.
- **"Seven Databases in Seven Weeks":** Eric Redmond, Jim Wilson. Hands-on tour of relational + NoSQL options.
- **"Readings in Database Systems"** ("Red Book"), edited by Hellerstein & Stonebraker. Free at https://redbook.io/. Curated foundational papers.
### Original papers (worth reading at least once)
 
- Codd, **"A Relational Model of Data for Large Shared Data Banks"** (1970). The founding paper. https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf
- Stonebraker & Hellerstein, **"What Goes Around Comes Around":** history and recurring patterns.
- Brewer, **CAP Theorem:** original formulation.
- DeWitt & Gray, **"Parallel Database Systems: The Future of High Performance Database Processing"** (1992)
- Mohan et al., **"ARIES: A Transaction Recovery Method"** (1992)
### Online courses & video
 
- **CMU 15-445 / 15-721 — Database Systems** (Andy Pavlo). Free, world-class. Lectures on YouTube. https://15445.courses.cs.cmu.edu/
- **Stanford CS145 / CS245** materials and online "Databases" course on edX.
- **MIT 6.830:** "Database Systems" course materials (free lecture notes).
- **Use The Index, Luke** (free book on indexing): https://use-the-index-luke.com/
### Practice & sandboxes
 
- **DB-Fiddle** (multi-DB online): https://www.db-fiddle.com/
- **SQLBolt** (interactive intro): https://sqlbolt.com/
- **Mode SQL Tutorial**: https://mode.com/sql-tutorial/
- **HackerRank SQL track**: https://www.hackerrank.com/domains/sql
- **LeetCode Database problems**: https://leetcode.com/problemset/database/
- **Mongo Playground**: https://mongoplayground.net/
### Reference comparisons & cheat sheets
 
- **DB-Engines ranking** (popularity tracker): https://db-engines.com/en/ranking
- **PostgreSQL vs MySQL comparisons** (search any vendor's docs)
- **NoSQL data modeling techniques** (great article): https://highlyscalable.wordpress.com/2012/03/01/nosql-data-modeling-techniques/
### Topics worth a deep dive (with starter searches)
 
- *"ACID isolation levels and anomalies."* Read PostgreSQL's explanation alongside MySQL's, then read Adya's PhD thesis on isolation if you really want depth.
- *"Query optimization and the Selinger paper."* System R cost-based optimizer, foundational.
- *"LSM trees."* Log-structured merge-trees, the storage engine behind RocksDB, Cassandra, LevelDB. Modern complement to B-trees.
- *"Distributed consensus, Paxos and Raft."* For the CAP corner you need C and P together. Raft has a famously readable paper.
- *"Vector databases and embeddings."* Newer trend (2022+) for AI/ML retrieval, e.g. Pinecone, pgvector, Milvus.
- *"Graph algorithms in databases."* PageRank, shortest-path, community detection.
---
 
*Final note: Bill Kent's mnemonic, "the key, the whole key, and nothing but the key, so help me Codd."*
