# Views, Indexes, Constraints, and Triggers

## What is it?

These four core database schema objects control how data is presented, searched, safeguarded, and acted upon automatically inside the DBMS engine:
- **Views**: Virtual tables representing saved queries.
- **Indexes**: Auxiliary data structures that accelerate row retrieval.
- **Constraints**: Declarative business rules preventing invalid data entry.
- **Triggers**: Automated procedures executed by the database in response to specific DML/DDL events.

## Simple Explanation

- **View**: A customized window looking into a room. It lets you see the furniture you care about while hiding private safes behind a wall.
- **Materialized View**: Taking a high-resolution printed photo through that window and hanging it in the lobby so guests don't have to keep walking upstairs to look.
- **Index**: An alphabetical index at the back of a 1,000-page textbook allowing you to jump directly to page 412 in 2 seconds.
- **Constraint**: A turnstile that rejects anyone without a valid ticket.
- **Trigger**: A motion-sensor alarm that automatically snaps a photo whenever someone enters the vault door.

## Hinglish Explanation

- **View**: Virtual table hai jo actual physical data store nahi karti, balki query ko encapsulate karti hai security ya simplification ke liye.
- **Materialized View**: Query ke result ko physically disk par save karti hai taaki heavy analytical queries super fast ho sakein.
- **Indexes**: Search ko $O(N)$ full table scan se $O(\log N)$ fast banate hain.
- **Triggers**: Database events (`BEFORE`/`AFTER INSERT/UPDATE/DELETE`) par automatic actions trigger karte hain (jaise audit logging ya validation).

## Why Do We Need It?

1. **Security & Abstraction (Views)**: Expose public customer data to frontend APIs without exposing credit card tokens or internal billing columns.
2. **Sub-Millisecond Queries (Indexes)**: Avoid scanning 10 million rows on disk when looking up a user by email.
3. **Automated Auditing (Triggers)**: Record compliance audit trails of all balance updates even if external developers forget to log changes in application code.

## How It Works

### 1. Simple View vs. Materialized View

```sql
-- Standard View: Virtual query executed on EVERY access
CREATE VIEW active_users_view AS
SELECT id, name, email FROM users WHERE is_active = TRUE;

-- Materialized View: Physically cached on disk!
CREATE MATERIALIZED VIEW monthly_sales_summary AS
SELECT dept_id, SUM(amount) AS total_revenue, COUNT(*) AS order_count
FROM sales
GROUP BY dept_id;

-- Fast non-blocking refresh in PostgreSQL:
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales_summary;
```

---

### 2. Specialized Indexes (Partial & Expression Indexes)

```sql
-- 1. Partial / Filtered Index: Indexes ONLY active rows (saves 90% index size!)
CREATE INDEX idx_active_orders ON orders(order_date)
WHERE status = 'PENDING';

-- 2. Expression / Functional Index: Accelerates queries with functions
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
-- Enables O(log N) index scan on: WHERE LOWER(email) = 'anil@test.com'
```

---

### 3. Triggers: Lifecycle and Events

```text
       Incoming INSERT / UPDATE / DELETE Statement
                         │
                         ▼
        ┌─────────────────────────────────┐
        │       BEFORE Trigger Fires      │ ◄── Validate / Modify NEW values
        └────────────────┬────────────────┘
                         │
                         ▼
             Physical Disk Write / Update
                         │
                         ▼
        ┌─────────────────────────────────┐
        │       AFTER Trigger Fires       │ ◄── Audit logs / CDC Event publish
        └─────────────────────────────────┘
```

#### Example Audit Trigger (PostgreSQL)
```sql
CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.salary != NEW.salary THEN
        INSERT INTO salary_audit_log(emp_id, old_salary, new_salary, changed_at)
        VALUES (OLD.id, OLD.salary, NEW.salary, NOW());
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_salary_audit
AFTER UPDATE OF salary ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_change();
```

## Important Terminology

| Object | Stored on Disk? | Modifiable directly? | Primary Purpose |
|---|---|---|---|
| **View** | No (only query definition)| Yes (if Simple View) | Security, access control, query simplification |
| **Materialized View**| **Yes (physical snapshot)**| Read-only (refreshed)| Heavy aggregation caching |
| **Index** | **Yes (auxiliary B+ Tree)**| Maintained by DBMS | Fast lookup ($O(\log N)$) |
| **Trigger** | Code stored in catalog | Executed on events | Auditing, data validation |
| **Stored Procedure**| Code stored in catalog | Executed on explicit call| Encapsulating multi-step transactions |

## Example: Updatable View Rules

A View is directly updatable (`UPDATE my_view SET ...`) ONLY if:
1. It queries exactly **one** base table.
2. It does NOT contain `DISTINCT`, `GROUP BY`, `HAVING`, or Aggregate functions (`SUM`, `AVG`).
3. It does NOT contain `UNION` or `UNION ALL`.
4. All `NOT NULL` columns without default values in the base table are included in the view.

## Real-World Example

- **E-Commerce Black Friday Performance**: A reporting dashboard running complex 6-table joins every 5 seconds crushed the master database. Engineers replaced it with a **Materialized View** refreshed every 5 minutes. Query execution dropped from 14 seconds to 3 milliseconds.
- **Production Warning on Triggers**: Modern engineering teams (Stripe, Uber) generally avoid complex business logic inside Triggers because triggers run invisibly inside database transactions, cause unexpected locking contention, and are difficult to unit-test and version-control.

## Important Properties

| Feature | Stored Procedure | User-Defined Function (UDF) |
|---|---|---|
| **Return Value** | Optional (zero, one, or multiple) | Must return a single value or table |
| **Usable in SELECT** | No (invoked via `CALL / EXEC`) | Yes (`SELECT my_func(col) FROM t;`) |
| **Transaction Control**| Can execute `COMMIT` / `ROLLBACK` | **Cannot** execute transaction controls |
| **Side Effects** | Allowed to modify DB state | Should generally be pure / read-only |

### Common Misconceptions

* Creating an index does not make all operations faster; indexes speed up `SELECT` and `WHERE` filters, but **slow down** `INSERT`, `UPDATE`, and `DELETE` because every write must update the table AND all auxiliary indexes.
* A standard view does not cache query results in RAM or disk; every time you select from a view, the underlying SQL is executed anew.
* `BEFORE` triggers can modify incoming data (`NEW.col = value`), but `AFTER` triggers cannot alter row data.

### What Interviewers Usually Test

1. View vs. Materialized View differences and refresh strategies.
2. When and why to use Partial / Filtered Indexes.
3. Updatable view restrictions.
4. Why heavy business logic in Triggers is considered an anti-pattern in modern microservices.

### Common Follow-up Questions

1. How does `REFRESH MATERIALIZED VIEW CONCURRENTLY` work in PostgreSQL?
2. What is an Expression / Functional Index?
3. What is the difference between Statement-Level and Row-Level triggers?
4. Can a view improve query performance over raw SQL?
5. What is the difference between an `INSTEAD OF` trigger and a standard trigger?

### Follow-up Answers

1. It refreshes the materialized view data without acquiring an exclusive lock on the view, allowing other queries to continue reading the old snapshot while the new dataset is built in the background (requires a UNIQUE index on the view).
2. An index built on the result of a function or computation rather than a raw column (e.g., `LOWER(email)` or `DATE(created_at)`).
3. A Row-Level trigger (`FOR EACH ROW`) executes once for every individual row affected by the query. A Statement-Level trigger (`FOR EACH STATEMENT`) executes once per query regardless of whether 1 row or 10,000 rows were updated.
4. No. A standard view is expanded into the underlying query during parsing; it provides architectural abstraction and security, not performance gains.
5. An `INSTEAD OF` trigger intercepts the DML operation entirely and executes custom procedural logic instead of the default insert/update/delete (commonly used to make complex multi-table views updatable).
