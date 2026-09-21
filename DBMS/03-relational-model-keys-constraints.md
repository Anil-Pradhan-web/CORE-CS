# Relational Model, Keys, and Integrity Constraints

## What is it?

The Relational Model represents data as a collection of two-dimensional tables (called **Relations**), where each row represents a unique record (**Tuple**) and each column represents a characteristic property (**Attribute**). Mathematical integrity constraints and keys guarantee data correctness and enforce business rules across relations.

## Simple Explanation

Think of an Excel spreadsheet for bank customers:
- The whole sheet is a **Relation** (Table).
- A single customer row is a **Tuple**.
- The column headers (Account Number, Name, Balance) are **Attributes**.
- The Account Number is the **Primary Key**: no two customers can have the same number, and nobody can have an empty (NULL) number.
- A transaction sheet with a customer's Account Number is a **Foreign Key**: it guarantees transactions cannot be recorded for non-existent customer accounts.

## Hinglish Explanation

Relational model Codd's rules par based hai jisme data tables (relations) mein store hota hai:
- Row ko **Tuple** bolte hain, aur column ko **Attribute**.
- Total number of rows = **Cardinality**; total number of columns = **Degree (Arity)**.
- **Keys**: Super Key (unique pehchan), Candidate Key (minimal super key), Primary Key (chosen candidate key jo NOT NULL ho), aur Foreign Key (jo doosre table se relation establish karta hai).
- **Referential Integrity**: Agar parent row delete ho, toh child row ka kya hoga (`CASCADE`, `SET NULL`, ya `RESTRICT`).

## Why Do We Need It?

Without keys and constraints:
1. Duplicate rows corrupt financial calculations and inventory counts.
2. Orphaned records accumulate (e.g., an order referencing a deleted user ID).
3. Invalid data enters systems (e.g., negative age or strings inserted into currency fields).

## How It Works

### The Hierarchy of Keys

```text
┌──────────────────────────────────────────────────────────┐
│                       Super Keys                         │
│  Any set of attributes that uniquely identifies a tuple   │
│  e.g., {ID}, {ID, Email}, {ID, Phone}, {Email, Name}     │
│                                                          │
│      ┌────────────────────────────────────────────┐      │
│      │               Candidate Keys               │      │
│      │        Minimal Super Keys (No extra info)  │      │
│      │        e.g., {ID}, {Email}                 │      │
│      │                                            │      │
│      │     ┌────────────────────────────────┐     │      │
│      │     │          Primary Key           │     │      │
│      │     │  Chosen candidate key          │     │      │
│      │     │  (Must be UNIQUE and NOT NULL) │     │      │
│      │     │  e.g., {ID}                    │     │      │
│      │     └────────────────────────────────┘     │      │
│      │                                            │      │
│      │      Alternate Key = {Email}               │      │
│      └────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────┘
```

---

### Integrity Constraints in Action

```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,                       -- Entity Integrity
    dept_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,             -- Alternate Key
    salary DECIMAL(10, 2) CHECK (salary > 0),       -- Domain Constraint
    dept_id INT,
    CONSTRAINT fk_dept
        FOREIGN KEY (dept_id)
        REFERENCES departments(dept_id)             -- Referential Integrity
        ON DELETE RESTRICT                          -- Prevents deleting dept with employees
        ON UPDATE CASCADE                           -- Updates emp records if dept_id changes
);
```

## Important Terminology

| Term | Formal Term | Meaning |
|---|---|---|
| **Table** | Relation | Two-dimensional structure of rows and columns |
| **Row** | Tuple | A single, complete data record |
| **Column** | Attribute | A named domain property |
| **Table Width** | Degree (Arity) | Total number of attributes/columns |
| **Table Length** | Cardinality | Total number of tuples/rows |
| **Allowed Values** | Domain | Permitted set of atomic values for an attribute |
| **Natural Key** | Real-world unique attribute | Social Security Number, PAN, Email |
| **Surrogate Key** | System-generated integer/UUID | `BIGSERIAL`, `AUTO_INCREMENT`, `UUIDv4` |

## Example: Foreign Key Actions on Parent Deletion

Given: `Department(dept_id)` and `Employee(dept_id REFERENCES Department)`:

1. `ON DELETE RESTRICT / NO ACTION`: Deleting department 10 fails with an error if any employee belongs to department 10.
2. `ON DELETE CASCADE`: Deleting department 10 automatically deletes all employee rows belonging to department 10.
3. `ON DELETE SET NULL`: Deleting department 10 sets `dept_id = NULL` for all employees belonging to department 10.

## Real-World Example

In financial transaction ledgers:
- Natural keys (like Email) can change if users update their profile.
- Databases use **Surrogate Keys** (`BIGSERIAL` or UUID) as the Primary Key for internal joins and index stability, while placing a `UNIQUE` constraint on natural keys like `email`.

## Important Properties

| Key Type | Allows NULL? | Uniqueness Guaranteed? | Count per Table |
|---|---|---|---|
| **Super Key** | Yes (if part of composite) | Yes | Many |
| **Candidate Key**| Yes (theoretically) | Yes | One or more |
| **Primary Key** | **Strictly NO** | Yes | **Exactly One** |
| **Alternate Key** | Yes (unless `NOT NULL`) | Yes | Zero or more |
| **Foreign Key** | Yes (unless declared `NOT NULL`)| No (by default in 1:N) | Zero or more |

### Common Misconceptions

* All Candidate Keys are Super Keys, but not all Super Keys are Candidate Keys (a candidate key must be irreducible/minimal).
* A Primary Key does NOT have to be a single column; a **Composite Primary Key** combines two or more columns (e.g., `PRIMARY KEY (student_id, course_id)`).
* Foreign Keys do not necessarily point only to a Primary Key; standard SQL allows Foreign Keys to reference any column with a `UNIQUE` constraint.

### What Interviewers Usually Test

1. Calculating the total number of Candidate Keys from a set of Functional Dependencies.
2. Distinguishing between Super Key, Candidate Key, and Primary Key.
3. Referential Integrity cascading rules (`CASCADE` vs. `RESTRICT` vs. `SET NULL`).
4. Trade-offs between Natural Keys and Surrogate Keys in distributed architectures.

### Common Follow-up Questions

1. Can a Primary Key contain NULL values?
2. What is the difference between a Candidate Key and an Alternate Key?
3. Can a table have multiple Foreign Keys pointing to the same parent table?
4. What is the difference between `RESTRICT` and `NO ACTION` in PostgreSQL / MySQL?
5. Why are UUIDs controversial as Primary Keys in B+ Tree clustered indexes?

### Follow-up Answers

1. No. By the Entity Integrity Constraint, no component of a Primary Key may be NULL, ensuring every row is deterministically addressable.
2. Candidate Keys are all minimal sets of attributes capable of uniquely identifying tuples. The one chosen by the DBA is the Primary Key; all unselected candidate keys are Alternate Keys.
3. Yes. For example, a `Flight` table can have `origin_airport_id` and `destination_airport_id`, both independently referencing `Airport(id)`.
4. `RESTRICT` immediately rejects the parent delete before other constraints run; `NO ACTION` delays the check until the end of the statement/transaction (allowing deferred constraint verification).
5. Random UUIDv4 values cause massive B+ Tree page splits and index fragmentation on insertion because keys do not arrive in sequential order, degrading write throughput. (Sequential UUIDv7 solves this).
