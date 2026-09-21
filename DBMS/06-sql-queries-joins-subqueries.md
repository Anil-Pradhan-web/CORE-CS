# SQL Queries: Joins, Subqueries, and Set Operations

## What is it?

SQL Joins combine columns from one or more tables based on a related column between them. Subqueries are queries nested inside another SQL statement. Set operations combine rows from multiple queries following set-theory rules.

## Simple Explanation

- **INNER JOIN**: Only attendees on the guest list who actually showed up at the party.
- **LEFT JOIN**: Every single person invited on the guest list, plus what food they ate (if they didn't attend, food is `NULL`).
- **FULL OUTER JOIN**: Everyone invited plus anyone who walked into the party uninvited.
- **Correlated Subquery**: An inspector checking every student's score against the average score of that specific student's school.

## Hinglish Explanation

- **INNER JOIN**: Dono tables ka matching data lata hai.
- **LEFT JOIN**: Left table ka saara data + right table ka matching data (non-matching par `NULL`).
- **FULL OUTER JOIN**: Dono tables ka poora data lata hai (matching jahan ho match karega, baaki jagah `NULL`).
- **SELF JOIN**: Table ka khud se join (jaise `Employee` table mein `Manager_ID` se manager ka name dhoondhna).
- **Subqueries**: `EXISTS` vs `IN` aur `UNION` vs `UNION ALL` ke interview traps standard hain.

## Why Do We Need It?

Normalized relational databases intentionally split data across tables to eliminate redundancy. Joins and subqueries reconstruct the comprehensive business view at query time with high flexibility.

## How It Works

### Visual Representation of SQL Joins

```text
Table A (Left)               Table B (Right)
┌──────────┬──────────┐      ┌──────────┬──────────┐
│ ID       │ Name     │      │ Emp_ID   │ Dept     │
├──────────┼──────────┤      ├──────────┼──────────┤
│ 1        │ Alice    │      │ 1        │ Eng      │
│ 2        │ Bob      │      │ 2        │ Sales    │
│ 3        │ Charlie  │      │ 4        │ HR       │
└──────────┴──────────┘      └──────────┴──────────┘

INNER JOIN (1, 2 only):
Matching IDs present in BOTH tables: {1: Alice - Eng, 2: Bob - Sales}

LEFT JOIN (1, 2, 3):
ALL rows from Table A:
{1: Alice - Eng, 2: Bob - Sales, 3: Charlie - NULL}

RIGHT JOIN (1, 2, 4):
ALL rows from Table B:
{1: Alice - Eng, 2: Bob - Sales, 4: NULL - HR}

FULL OUTER JOIN (1, 2, 3, 4):
All records from both sides with NULL fills where unmatched.
```

---

### Self Join: Finding Employee Managers

```sql
SELECT
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

---

### Correlated Subquery vs. Nested Subquery

#### 1. Independent Nested Subquery (Runs ONCE)
```sql
-- Finds employees earning more than company-wide average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

#### 2. Correlated Subquery (Evaluated FOR EACH outer row!)
```sql
-- Finds employees earning more than their department's average
SELECT e.name, e.salary, e.dept_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(d.salary)
    FROM employees d
    WHERE d.dept_id = e.dept_id -- References outer query row 'e'!
);
```

---

### The Dangerous `NOT IN (NULL)` Trap

```sql
-- Suppose department IDs are (10, 20, NULL)
SELECT * FROM employees WHERE dept_id NOT IN (10, 20, NULL);
-- THIS RETURNS ZERO ROWS!
```
**Why?** In SQL three-valued logic:
`dept_id NOT IN (10, 20, NULL)` translates to:
`dept_id != 10 AND dept_id != 20 AND dept_id != NULL`.
Any comparison with `NULL` yields `UNKNOWN`. `TRUE AND UNKNOWN` evaluates to `UNKNOWN`, filtering out every row!
**Fix**: Use `NOT EXISTS`:
```sql
SELECT * FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM departments d WHERE d.dept_id = e.dept_id
);
```

## Important Terminology

| Term | Meaning |
|---|---|
| **Equi-Join** | Join based on strict equality (`=`) condition |
| **Non-Equi Join**| Join based on inequalities (`<`, `>`, `BETWEEN`) |
| **Cross Join** | Unrestricted Cartesian Product ($M \times N$ rows) |
| **Correlated Subquery**| Subquery referencing columns from outer query table |
| **`UNION`** | Combines rows and **sorts to eliminate duplicates** (Expensive!) |
| **`UNION ALL`** | Combines rows **preserving duplicates** (Fast, no sort) |

## Example: Nth Highest Salary

A classic interview test query:

```sql
-- 2nd Highest Salary using DENSE_RANK() or Subquery:
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Generic N-th Highest using Correlated Subquery:
SELECT e1.salary
FROM employees e1
WHERE (N - 1) = (
    SELECT COUNT(DISTINCT e2.salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
);
```

## Real-World Example

In high-throughput microservice queries:
- Swapping `UNION` with `UNION ALL` in a customer search API cut execution time from 450 ms to 12 ms by eliminating unnecessary in-memory temporary table sorting.
- Replacing a correlated subquery with an indexed `INNER JOIN` over a derived table prevented full table scans across 5 million order records.

## Important Properties

| Operation | Deduplicates? | Performance | Preserves Left? |
|---|---|---|---|
| **INNER JOIN** | No | Fast (with indexes) | Only matched |
| **LEFT JOIN** | No | Fast (with indexes) | **Always 100%** |
| **FULL JOIN** | No | Slower (scans both) | Always 100% |
| **`UNION`** | **Yes** (sort/hash) | Slower (O(N log N))| N/A |
| **`UNION ALL`** | No | Very Fast (O(N) append)| N/A |

### Common Misconceptions

* `WHERE` clauses placed after a `LEFT JOIN` on the right table convert the join into an `INNER JOIN` unless conditions are placed in the `ON` clause or handle `IS NULL`.
* `COUNT(*)` counts all rows including NULLs; `COUNT(column_name)` ignores NULL values.
* `IN` evaluates the subquery once, whereas correlated subqueries conceptually re-evaluate per outer row (though modern query optimizers unnest them into joins).

### What Interviewers Usually Test

1. Differences between `UNION` and `UNION ALL`.
2. Explaining the `NOT IN (NULL)` trap and fixing it with `NOT EXISTS`.
3. Writing queries for Second/Nth Highest Salary and finding duplicate records.
4. Self Joins for hierarchical employee-manager tree structures.

### Common Follow-up Questions

1. When should you prefer `EXISTS` over `IN`?
2. What happens if a `LEFT JOIN` matches multiple rows in the right table?
3. How do you find duplicate rows in a table using SQL?
4. What is the difference between `ON` and `WHERE` in a `LEFT JOIN`?
5. How can you delete duplicate rows while keeping one original?

### Follow-up Answers

1. Use `EXISTS` when the inner query table is large and indexed (it short-circuits on the first match), and whenever subquery results might contain `NULL` values.
2. The left table row duplicates for each matching right row (cartesian expansion for matching keys), increasing the final row count beyond the original left table length.
3. Group by the identifying columns and filter with `HAVING COUNT(*) > 1`.
4. The `ON` clause determines how rows match *during* join generation; the `WHERE` clause filters rows *after* the join has produced its result set.
5. Use a Common Table Expression (CTE) with `ROW_NUMBER() OVER (PARTITION BY cols ORDER BY id)` and delete where `row_num > 1`.
