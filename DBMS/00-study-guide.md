# Database Management Systems (DBMS) Interview Preparation

This is a revision-first, high-yield knowledge base for software engineering, backend, systems, and campus placement interviews. Read the chapters in sequential order once, then revise via the SQL query cheat-sheets, transaction scenarios, and the 100 interview question bank.

## Learning Path

| Stage | Chapters | Outcome |
|---|---|---|
| **Foundations** | 01–05 | Master 3-schema architecture, ER modeling, relational keys, relational algebra, and SQL DDL/DML |
| **Querying & Design** | 06–10 | Write complex joins, window functions, and normalize schemas up to BCNF while managing denormalization |
| **Storage & Engines** | 11–15 | Understand slotted pages, B+ Tree index scans, LSM trees, query optimization with `EXPLAIN`, and buffer pools |
| **Transactions & Concurrency** | 16–20 | Reason about ACID, serializability graphs, 2PL, MVCC snapshot isolation, and ARIES crash recovery |
| **Distributed & System Design**| 21–24 | Design distributed databases, replication lag mitigations, sharding, 2PC, and evaluate SQL vs NoSQL |
| **Practice** | 25 | Speak fluently through exactly 100 curated technical interview questions |

## Speakable Interview Pattern

**Definition → Why → How → Example → Trade-off.** For example: “A clustered index determines the physical storage order of table rows on disk. It makes primary key lookups and range scans O(log N) fast because data pages are stored sequentially in the leaf nodes of a B+ Tree. However, every table can have only one clustered index, and secondary indexes must incur an extra pointer hop.”

## Topic Map

1. [DBMS basics and architecture](01-dbms-basics-architecture.md)
2. [Data models and ER diagrams](02-data-models-er-diagrams.md)
3. [Relational model, keys, and integrity constraints](03-relational-model-keys-constraints.md)
4. [Relational algebra and calculus](04-relational-algebra-calculus.md)
5. [SQL basics: DDL, DML, DCL, and TCL](05-sql-basics-ddl-dml-dcl-tcl.md)
6. [SQL queries, joins, and subqueries](06-sql-queries-joins-subqueries.md)
7. [SQL aggregations and window functions](07-sql-aggregations-window-functions.md)
8. [Functional dependencies and normalization (1NF–BCNF)](08-functional-dependencies-normalization.md)
9. [Higher normal forms and pragmatic denormalization](09-higher-normal-forms-denormalization.md)
10. [Views, indexes, constraints, and triggers](10-views-indexes-constraints-triggers.md)
11. [File organization and storage engine internals](11-file-organization-storage-engine.md)
12. [Indexing, B-Trees, and B+ Trees](12-indexing-b-trees-b-plus-trees.md)
13. [LSM Trees and specialized indexes](13-lsm-trees-specialized-indexes.md)
14. [Query processing and optimization](14-query-processing-optimization.md)
15. [Hashing and buffer pool management](15-hashing-buffer-management.md)
16. [Transactions and ACID properties](16-transactions-acid-properties.md)
17. [Concurrency control, schedules, and serializability](17-concurrency-control-schedules.md)
18. [Lock-based and timestamp protocols](18-lock-based-timestamp-protocols.md)
19. [MVCC and transaction isolation levels](19-mvcc-isolation-levels.md)
20. [Crash recovery, WAL, and ARIES](20-crash-recovery-wal-aries.md)
21. [Distributed databases and replication](21-distributed-databases-replication.md)
22. [Sharding, partitioning, and distributed transactions](22-sharding-partitioning-distributed-transactions.md)
23. [NoSQL vs. SQL and CAP/PACELC theorem](23-nosql-vs-sql-cap-theorem.md)
24. [DBMS in system design and production](24-dbms-system-design-production.md)
25. [100 interview questions](25-interview-questions.md)
