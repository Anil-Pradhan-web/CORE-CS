# SQL Aggregations and Window Functions

## What is it?

SQL Aggregations collapse multiple rows into a single summary value per group using `GROUP BY`. **Window Functions** perform calculations across a set of table rows related to the current row without collapsing the rows—preserving the individual identity of each row while attaching calculated analytical metrics.

## Simple Explanation

- **`GROUP BY`**: A teacher reports only the average test score for each classroom. The final report has 5 rows (one for each class), and individual student names disappear.
- **Window Function**: The teacher hands back each student's exam paper showing their individual score, but prints their classroom ranking and class average on the margin. The report still has all 150 student rows intact.

## Hinglish Explanation

`GROUP BY` rows ko condense/collapse karke single aggregate row banata hai (jaise har department ka average salary nikalna). `HAVING` aggregate result ko filter karta hai, jabki `WHERE` raw individual rows ko pehle filter karta hai. **Window Functions** (`OVER (PARTITION BY ... ORDER BY ...)`) rows ko collapse kiye bina ranking (`RANK`, `DENSE_RANK`, `ROW_NUMBER`) ya running totals calculate karte hain.

## Why Do We Need It?

1. **Analytical Reporting**: Business metrics like "Top 3 highest-earning employees in each department" or "Month-over-month revenue growth" are clumsy and slow with self-joins, but clean and fast with Window Functions.
2. **Running Totals & Rolling Averages**: Financial ledgers require calculating cumulative balance row-by-row as transactions arrive.
3. **Deduplication**: Window functions effortlessly identify and purge duplicate entries while keeping the latest record.

## How It Works

### Group By vs. Window Function

```text
Input Rows:
┌──────────┬──────────┬──────────┐
│ Emp_ID   │ Dept     │ Salary   │
├──────────┼──────────┼──────────┤
│ 1        │ IT       │ 5000     │
│ 2        │ IT       │ 7000     │
│ 3        │ HR       │ 4000     │
└──────────┴──────────┴──────────┘

GROUP BY Dept (Rows Collapse to 2):
IT -> AVG = 6000
HR -> AVG = 4000

Window Function: AVG(Salary) OVER (PARTITION BY Dept) (All 3 rows remain):
Emp 1 (IT) | Salary 5000 | Dept_Avg: 6000
Emp 2 (IT) | Salary 7000 | Dept_Avg: 6000
Emp 3 (HR) | Salary 4000 | Dept_Avg: 4000
```

---

### `ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()`

Given Salaries: `[100, 100, 80, 70]`

| Salary | `ROW_NUMBER()` | `RANK()` | `DENSE_RANK()` | Explanation |
|---|---|---|---|---|
| 100 | **1** | **1** | **1** | First duplicate |
| 100 | **2** | **1** | **1** | Ties share same rank |
| 80 | **3** | **3** (skips 2!) | **2** (no skip) | `RANK()` has gaps; `DENSE_RANK()` has NO gaps |
| 70 | **4** | **4** | **3** | Continuous ranking |

---

### Lead and Lag (Looking Ahead & Looking Behind)

```sql
SELECT
    sale_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) AS prev_day_sales,
    LEAD(amount, 1) OVER (ORDER BY sale_date) AS next_day_sales,
    amount - LAG(amount, 1) OVER (ORDER BY sale_date) AS daily_growth
FROM daily_sales;
```

---

### Running Total with Window Framing

```sql
SELECT
    emp_id,
    dept_id,
    salary,
    -- Calculates running total within department ordered by emp_id
    SUM(salary) OVER (
        PARTITION BY dept_id
        ORDER BY emp_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM employees;
```

## Important Terminology

| Clause / Function | Meaning |
|---|---|
| **`PARTITION BY`** | Divides rows into logical subsets (similar to `GROUP BY`, but no collapsing) |
| **`ORDER BY` (in Over)**| Defines sorting sequence of rows inside each partition |
| **`ROWS BETWEEN`** | Defines the physical sliding window frame (e.g., preceding 3 rows) |
| **`DENSE_RANK()`** | Assigns consecutive ranks without skipping numbers on ties |
| **`NTILE(N)`** | Divides partition rows evenly into $N$ ranked buckets (e.g., quartiles) |
| **`HAVING`** | Filters groups *after* aggregation (operates on `SUM`, `COUNT`, etc.) |

## Example: Top 3 Highest Earning Employees Per Department

```sql
WITH RankedEmployees AS (
    SELECT
        name,
        dept_id,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY dept_id
            ORDER BY salary DESC
        ) AS rank_in_dept
    FROM employees
)
SELECT name, dept_id, salary, rank_in_dept
FROM RankedEmployees
WHERE rank_in_dept <= 3;
```

## Real-World Example

In Fintech and Stock Trading applications:
- Moving Averages (e.g., 50-day Simple Moving Average):
  `AVG(closing_price) OVER (ORDER BY trade_date ROWS BETWEEN 49 PRECEDING AND CURRENT ROW)`.
- Fraud detection pipelines track transaction velocity by comparing the current transaction timestamp with `LAG(created_at, 1)` to detect rapid-fire card swipes across different countries.

## Important Properties

| Feature | `WHERE` | `HAVING` | Window Function (`OVER`) |
|---|---|---|---|
| **Runs When?** | Before aggregation | After aggregation | After `HAVING`, during projection |
| **Allows Aggregates?** | **No** (`WHERE SUM(x)>0` errors)| **Yes** (`HAVING COUNT(*)>1`) | Yes (`AVG(x) OVER()`) |
| **Collapses Rows?** | No | Yes (with `GROUP BY`) | **Strictly No** |
| **Usable in WHERE?** | N/A | N/A | **Cannot** be placed in `WHERE` directly (requires CTE) |

### Common Misconceptions

* You cannot use a window function directly in a `WHERE` clause (e.g., `WHERE ROW_NUMBER() OVER (...) = 1` triggers a syntax error); you must compute the window function inside a CTE or subquery first.
* `COUNT(col)` ignores NULLs; `COUNT(*)` counts all rows regardless of NULL content.
* When `ORDER BY` is specified inside `OVER ()` without an explicit frame, SQL defaults to `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which treats duplicates identically rather than row-by-row.

### What Interviewers Usually Test

1. Difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`.
2. Differences between `WHERE` and `HAVING` with practical examples.
3. Writing queries for Top-K items per group (e.g., highest salary per department).
4. Calculating Running Totals, Moving Averages, and Year-over-Year growth using `LAG`/`LEAD`.

### Common Follow-up Questions

1. Why can't window functions be used directly in the `WHERE` clause?
2. What is the difference between `ROWS` and `RANGE` in window framing?
3. How do you calculate Month-over-Month percentage revenue growth in SQL?
4. What happens if `PARTITION BY` is omitted in an `OVER ()` clause?
5. How does `GROUP BY` handle `NULL` values?

### Follow-up Answers

1. Because of SQL's logical execution order: `WHERE` (Step 2) filters rows long before Window Functions are computed during the `SELECT` phase (Step 5). Use a CTE (Common Table Expression) to wrap the window calculation.
2. `ROWS` treats duplicate values as distinct physical rows in the window frame; `RANGE` groups rows with duplicate `ORDER BY` values together logically as peers.
3. Use `(revenue - LAG(revenue, 1) OVER (ORDER BY month)) / LAG(revenue, 1) OVER (ORDER BY month) * 100.0`.
4. If `PARTITION BY` is omitted, the entire result set is treated as a single undivided partition (the window function calculates metrics over all rows).
5. All `NULL` values are grouped together into a single collective group.
