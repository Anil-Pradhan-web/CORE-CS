# SQL Basics: DDL, DML, DCL, and TCL

## What is it?

Structured Query Language (SQL) is the standard declarative language used to define, manipulate, control, and query data in relational databases. SQL statements are categorized into functional sub-languages based on their operational scope: **DDL**, **DML**, **DQL**, **DCL**, and **TCL**.

## Simple Explanation

If a database is a bank:
- **DDL**: Constructing the building and vault walls (`CREATE TABLE`, `ALTER TABLE`).
- **DML**: Depositing cash, updating account balances, and withdrawing money (`INSERT`, `UPDATE`, `DELETE`).
- **DQL**: Asking the teller for your account statement (`SELECT`).
- **DCL**: The manager issuing security badges allowing staff into the vault (`GRANT`, `REVOKE`).
- **TCL**: Stamping the official deposit seal to finalize a transfer or canceling a mistyped voucher (`COMMIT`, `ROLLBACK`).

## Hinglish Explanation

SQL relational databases ki universal language hai jo 5 parts mein banti hoti hai:
- **DDL**: Schema/structure define karta hai (Tables create, alter, drop karna).
- **DML**: Table ke andar ka data modify karta hai (`INSERT`, `UPDATE`, `DELETE`).
- **DQL**: Data query aur fetch karta hai (`SELECT`).
- **DCL**: Permissions aur access control handle karta hai (`GRANT`, `REVOKE`).
- **TCL**: Transactions manage karta hai (`COMMIT`, `ROLLBACK`, `SAVEPOINT`).
Sabse famous interview question hai: **DELETE vs TRUNCATE vs DROP**.

## Why Do We Need It?

1. **Standardized Declarative Syntax**: Developers describe *what* data they need rather than writing manual loops to iterate through disk sectors.
2. **Strict Transaction Boundaries**: TCL ensures multi-statement operations either completely succeed or roll back cleanly without partial corruption.
3. **Role-Based Security**: DCL restricts database access, preventing unauthorized users from altering critical tables.

## How It Works

### The 5 SQL Sub-Languages

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                             The SQL Family                                  │
├─────────────┬─────────────┬─────────────┬─────────────┬─────────────────────┤
│     DDL     │     DML     │     DQL     │     DCL     │         TCL         │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────────┤
│ CREATE      │ INSERT      │ SELECT      │ GRANT       │ COMMIT              │
│ ALTER       │ UPDATE      │             │ REVOKE      │ ROLLBACK            │
│ DROP        │ DELETE      │             │             │ SAVEPOINT           │
│ TRUNCATE    │             │             │             │ SET TRANSACTION     │
│ RENAME      │             │             │             │                     │
└─────────────┴─────────────┴─────────────┴─────────────┴─────────────────────┘
```

---

### The Crucial Comparison: `DELETE` vs. `TRUNCATE` vs. `DROP`

```sql
-- 1. DELETE: DML command, deletes specific rows, logged row-by-row, fires triggers
DELETE FROM orders WHERE status = 'CANCELLED';

-- 2. TRUNCATE: DDL command, deallocates all data pages instantly, resets identity
TRUNCATE TABLE logs;

-- 3. DROP: DDL command, removes entire table schema and data from database
DROP TABLE obsolete_archive;
```

---

### Logical Query Processing Order (How SQL Actually Executes)

The order in which the SQL engine evaluates a query differs drastically from the order you write it:

```text
Written Order:                        Logical Execution Order:
1. SELECT                             1. FROM & JOIN (Identify tables)
2. FROM & JOIN                        2. WHERE (Filter individual rows)
3. WHERE                              3. GROUP BY (Aggregate into groups)
4. GROUP BY                           4. HAVING (Filter groups)
5. HAVING                             5. SELECT (Evaluate expressions)
6. ORDER BY                           6. DISTINCT (Deduplicate rows)
7. LIMIT / OFFSET                     7. ORDER BY (Sort final rows)
                                      8. LIMIT / OFFSET (Paginate results)
```
*Because `WHERE` executes before `SELECT`, you CANNOT use column aliases declared in `SELECT` inside a `WHERE` clause!*

## Important Terminology

| Command Group | Auto-commit? | Operates On | Can Rollback? |
|---|---|---|---|
| **DDL** | **YES** (implicit commit in MySQL/Oracle) | Schema / Table Structure | Generally No (Yes in PostgreSQL) |
| **DML** | No (governed by active transaction) | Table Rows / Data Tuples | Yes (via `ROLLBACK`) |
| **DQL** | Read-only | Table Rows | N/A |
| **DCL** | Auto-committed | Security Privileges | Generally No |
| **TCL** | Explicit control | Transaction Boundaries | N/A |

## Example: TCL in a Multi-Step Banking Transfer

```sql
BEGIN TRANSACTION;

-- Step 1: Deduct from Sender
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 101;

-- Set an intermediate recovery point
SAVEPOINT after_debit;

-- Step 2: Attempt credit to Receiver
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 999;

-- If receiver account is invalid, rollback only to the savepoint!
-- ROLLBACK TO after_debit;

-- Otherwise, finalize everything permanently:
COMMIT;
```

## Real-World Example

- **High-Volume Log Clearing**: Using `DELETE FROM audit_logs;` on a 50 GB table generates gigabytes of undo logs, saturates disk I/O, and takes 20 minutes. Using `TRUNCATE TABLE audit_logs;` completes in 50 milliseconds by deallocating the page extents directly in the storage engine.

## Important Properties

| Feature | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| **Category** | DML | DDL | DDL |
| **Scope** | Selected rows (`WHERE`) | All rows in table | Entire table structure & data |
| **Speed** | Slow (row-by-row logging) | Very Fast (deallocates pages) | Very Fast (metadata drop) |
| **Triggers** | Fires `ON DELETE` triggers | **Does NOT** fire triggers | Does not fire delete triggers |
| **Identity / Auto-inc**| Does not reset counter | Resets counter back to start | Completely destroyed |
| **Rollback in TX** | Always rollable back | Rollable in Postgres; No in MySQL | Rollable in Postgres; No in MySQL |

### Common Misconceptions

* `TRUNCATE` is not simply `DELETE` without a `WHERE` clause; `TRUNCATE` is a DDL operation that drops and recreates the data allocation pages at the storage engine level.
* You cannot use a column alias in a `WHERE` clause (e.g., `SELECT salary * 12 AS annual FROM emp WHERE annual > 50000` fails) because `WHERE` runs in Step 2, long before `SELECT` aliases are processed in Step 5.
* In PostgreSQL, DDL commands (including `CREATE TABLE` and `DROP TABLE`) are transactional and **can** be rolled back inside a `BEGIN...ROLLBACK` block!

### What Interviewers Usually Test

1. Detailed differences between `DELETE`, `TRUNCATE`, and `DROP`.
2. Step-by-step Logical Query Execution Order (why aliases fail in `WHERE`).
3. Using `SAVEPOINT` and `ROLLBACK TO SAVEPOINT` in financial transactions.
4. Differences between `WHERE` and `HAVING`.

### Common Follow-up Questions

1. Why can you use a column alias in `ORDER BY`, but not in `WHERE`?
2. What is the difference between `CHAR` and `VARCHAR`?
3. How does `TRUNCATE` reset auto-increment counters?
4. What happens when you execute DDL statements in MySQL during an active transaction?
5. What is the difference between `GRANT` and `REVOKE`?

### Follow-up Answers

1. Because of execution order: `WHERE` runs in Step 2 before column aliases are computed in Step 5 (`SELECT`), whereas `ORDER BY` runs in Step 7, after aliases are already available.
2. `CHAR(N)` is fixed-length (pads with trailing spaces if shorter; faster lookups); `VARCHAR(N)` is variable-length (stores only characters plus a length byte prefix; saves storage).
3. `TRUNCATE` deallocates the table's underlying storage extents and resets the table header's sequence counter back to its initial seed value (e.g., 1).
4. In MySQL, executing any DDL command (`CREATE`, `ALTER`, `TRUNCATE`) triggers an **implicit commit** of the current transaction, preventing any previous DML statements from being rolled back!
5. `GRANT` gives users or roles specific privileges (e.g., `GRANT SELECT, INSERT ON orders TO app_user;`); `REVOKE` withdraws previously granted permissions.
