# Functional Dependencies and Normalization (1NF to BCNF)

## What is it?

Normalization is a systematic database design technique used to organize relational schemas, eliminating data redundancy and preventing data modification anomalies (**Insertion**, **Deletion**, and **Update** anomalies). It evaluates schemas using **Functional Dependencies (FDs)** and decomposes tables into higher **Normal Forms (1NF, 2NF, 3NF, BCNF)** while ensuring **Lossless Joins** and **Dependency Preservation**.

## Simple Explanation

Imagine storing a university's entire data in a single giant spreadsheet: `Student_ID, Student_Name, Course_ID, Course_Name, Instructor_Name, Instructor_Office`.
- **Update Anomaly**: If Dr. Smith moves offices, you must update 500 individual student rows. If you miss one row, the database becomes contradictory.
- **Insertion Anomaly**: You cannot add a brand-new course until at least one student enrolls in it (because `Student_ID` is part of the primary key and cannot be NULL).
- **Deletion Anomaly**: If the only student enrolled in 'Quantum Physics' drops out, deleting their student row accidentally destroys the course and instructor records entirely!
Normalization decomposes this monster sheet into separate, clean, linked tables: `Students`, `Courses`, and `Enrollments`.

## Hinglish Explanation

Normalization redundant data ko hatata hai aur anomalies (Insert, Update, Delete problems) ko prevent karta hai.
- **1NF**: Saare attributes atomic hone chahiye (no comma-separated lists/arrays).
- **2NF**: No Partial Dependency (koi bhi non-prime attribute composite candidate key ke aade hisse par depend na kare).
- **3NF**: No Transitive Dependency (Non-prime attribute kisi doosre non-prime attribute par depend na kare; $X \to Y$ mein $X$ super key ho ya $Y$ prime attribute ho).
- **BCNF**: Strict 3NF ($X \to Y$ mein $X$ **must** be a Super Key).

## Why Do We Need It?

1. **Eliminates Storage Bloat**: Prevents repeating customer names and addresses 10,000 times for 10,000 orders.
2. **Guarantees Consistency**: Updating a customer's address in one single row immediately updates all their associated historical orders.
3. **Formal Mathematical Rigor**: Armstrong's axioms provide mathematical proofs that decomposed tables reconstruct original data without phantom or lost rows.

## How It Works

### The Hierarchy of Normal Forms

```text
┌─────────────────────────────────────────────────────────────┐
│                    1NF (Atomic Values)                      │
│      ┌───────────────────────────────────────────────┐      │
│      │        2NF (1NF + No Partial Dependency)      │      │
│      │      ┌─────────────────────────────────┐      │      │
│      │      │ 3NF (2NF + No Transitive Dep.)  │      │      │
│      │      │      ┌───────────────────┐      │      │      │
│      │      │      │       BCNF        │      │      │      │
│      │      │      │ (Every determinant│      │      │      │
│      │      │      │   is a Super Key) │      │      │      │
│      │      │      └───────────────────┘      │      │      │
│      │      └─────────────────────────────────┘      │      │
│      └───────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

---

### Step-by-Step Definitions & Tests

#### Prime vs. Non-Prime Attributes
- **Prime Attribute**: An attribute that is a member of **at least one Candidate Key**.
- **Non-Prime Attribute**: An attribute that does not appear in any candidate key.

#### 1. First Normal Form (1NF)
- Every column must contain only **atomic (indivisible) values**.
- No repeating groups or multi-valued columns (e.g., storing `phone = "9876, 5432"` violates 1NF).

#### 2. Second Normal Form (2NF)
- Must be in 1NF.
- **No Partial Dependencies**: Every non-prime attribute must be fully functionally dependent on the *entire* candidate key (cannot depend on a proper subset of a composite candidate key).
- *If all candidate keys consist of a single column, the table is automatically in 2NF!*

#### 3. Third Normal Form (3NF)
- Must be in 2NF.
- **No Transitive Dependencies**: For every non-trivial functional dependency $X \to Y$, at least one of the following conditions must hold:
  1. $X$ is a **Super Key**, OR
  2. $Y$ is a **Prime Attribute**.

#### 4. Boyce-Codd Normal Form (BCNF / 3.5NF)
- A stricter version of 3NF.
- For every non-trivial functional dependency $X \to Y$, **$X$ MUST be a Super Key**.
- (Removes the exception where $Y$ could be a prime attribute).

---

### The Two Golden Decomposition Rules

When decomposing relation $R$ into $R_1$ and $R_2$:
1. **Lossless-Join Decomposition (Mandatory!)**: Rejoining $R_1 \bowtie R_2$ must equal $R$ exactly (no spurious phantom rows).
   - *Test Rule*: The common attribute $(R_1 \cap R_2)$ must be a **Super Key** of either $R_1$ or $R_2$:
     $$(R_1 \cap R_2 \to R_1) \quad\text{OR}\quad (R_1 \cap R_2 \to R_2)$$
2. **Dependency Preservation**: All functional dependencies of $R$ can be verified using only $R_1$ or $R_2$ without expensive cross-table joins.
   - *3NF guarantees BOTH Lossless Join and Dependency Preservation.*
   - *BCNF guarantees Lossless Join, but CANNOT always preserve all functional dependencies!*

## Important Terminology

| Term | Formal Definition |
|---|---|
| **Functional Dependency ($X \to Y$)**| If two tuples have identical values for attribute set $X$, they must have identical values for $Y$ |
| **Trivial Dependency** | $X \to Y$ where $Y \subseteq X$ (e.g., $\{A, B\} \to A$) |
| **Attribute Closure ($X^+$)** | The complete set of attributes functionally determined by $X$ under a set of FDs |
| **Armstrong's Axioms** | Sound and complete rules: Reflexivity, Augmentation, Transitivity |
| **Canonical / Minimal Cover** | A simplified, minimal set of FDs equivalent to the original set without redundant attributes or dependencies |

## Example: Normalization Walkthrough

Given Relation: $R(A, B, C, D)$ with FDs:
$$AB \to C, \quad C \to D, \quad D \to A$$

1. **Find Candidate Keys**:
   - Compute $(AB)^+ = \{A, B, C, D\} \implies AB$ is a Candidate Key.
   - Compute $(BC)^+ = \{B, C, D, A\} \implies BC$ is a Candidate Key.
   - Compute $(BD)^+ = \{B, D, A, C\} \implies BD$ is a Candidate Key.
   - **Prime Attributes**: $\{A, B, C, D\}$ (Every attribute belongs to a candidate key!).
   - **Non-Prime Attributes**: None!
2. **Check Normal Form**:
   - **2NF**: Since there are no non-prime attributes, there are no partial dependencies $\implies$ **In 2NF**.
   - **3NF**: In $C \to D$, $C$ is not a super key, but $D$ is a prime attribute (Condition 2 holds!). In $D \to A$, $A$ is prime $\implies$ **In 3NF**.
   - **BCNF**: In $C \to D$, $C$ is NOT a super key $\implies$ **Fails BCNF!**
   - *Conclusion*: $R$ is in 3NF, but NOT in BCNF.

## Real-World Example

- In high-throughput OLTP banking systems, schemas are normalized strictly to **3NF or BCNF**. This ensures that updating a customer's KYC status or bank balance requires locking only a single row in a single table, maximizing concurrent transaction throughput.

## Important Properties

| Normal Form | Eliminates Anomalies? | Guarantees Lossless Join? | Guarantees Dependency Preservation? |
|---|---|---|---|
| **1NF** | No | Yes (trivial) | Yes |
| **2NF** | Partially (removes partial dep.) | Yes | Yes |
| **3NF** | Mostly (removes transitive dep.) | **Yes** | **Yes** |
| **BCNF** | **Completely** (all functional anomalies)| **Yes** | **No** (dependencies may be lost) |

### Common Misconceptions

* 3NF and BCNF are not identical; BCNF is strictly stronger than 3NF because BCNF does not allow non-super keys to determine prime attributes.
* Normalization does NOT make queries faster; normalization makes writes safe and updates fast, but can slow down reads because multiple tables must be rejoined.
* A decomposition that loses dependencies is not "broken" or illegal—it just means checking business rules requires foreign key triggers or cross-table join assertions.

### What Interviewers Usually Test

1. Calculating the attribute closure $X^+$ and finding all Candidate Keys.
2. Identifying the highest Normal Form of a given relation and FD set.
3. Proving whether a decomposition is Lossless or Lossy.
4. Explaining why 3NF is often preferred in industry over BCNF (due to dependency preservation).

### Common Follow-up Questions

1. How do you test if a decomposition is Lossless?
2. Why can't BCNF always preserve functional dependencies?
3. What are Armstrong's Axioms?
4. What is the difference between a Partial Dependency and a Transitive Dependency?
5. When would you deliberately choose 3NF over BCNF?

### Follow-up Answers

1. Check if the intersection of the two decomposed tables is a super key of at least one of the tables ($R_1 \cap R_2 \to R_1$ or $R_1 \cap R_2 \to R_2$).
2. Because in BCNF, every determinant must be a super key. If a dependency exists where a subset of attributes determines another attribute across table boundaries without being a super key, enforcing BCNF forces decomposing the table, separating the dependent attributes across tables.
3. The three foundational inference rules: Reflexivity (if $Y \subseteq X$, then $X \to Y$), Augmentation (if $X \to Y$, then $XZ \to YZ$), and Transitivity (if $X \to Y$ and $Y \to Z$, then $X \to Z$).
4. A partial dependency occurs when a non-prime attribute depends on *part* of a composite candidate key. A transitive dependency occurs when a non-prime attribute depends on another non-prime attribute via an intermediate key.
5. Whenever business constraints require verifying a functional dependency on every insert/update, and decomposing to BCNF would require an expensive multi-table join to validate that dependency.
