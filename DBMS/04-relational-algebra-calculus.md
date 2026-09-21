# Relational Algebra and Relational Calculus

## What is it?

Relational Algebra is a procedural mathematical query language that defines a step-by-step set of operations to transform one or more relations into a new relation. Relational Calculus is a non-procedural (declarative) query language based on predicate calculus that describes *what* data is desired rather than *how* to compute it. SQL is fundamentally rooted in these formalisms.

## Simple Explanation

- **Relational Algebra (Procedural)**: Like a recipe: "First filter all rows where salary > 50,000 ($\sigma$), then take only the Name and Dept columns ($\pi$), then sort."
- **Relational Calculus (Declarative)**: Like a restaurant order: "Give me the names and departments of all people who earn more than 50,000." You state what you want, and the database query engine decides how to fetch it.

## Hinglish Explanation

Relational Algebra theoretical foundation hai jahan har operation ek ya do tables lekar ek naya table return karta hai (closure property). Isme basic operations hote hain: Selection ($\sigma$ - rows filter karna), Projection ($\pi$ - columns filter karna), Cross Product ($\times$), Union ($\cup$), aur Difference ($-$). Relational Calculus declarative hai (`{t | P(t)}`) jisse modern SQL syntax inspire hua hai.

## Why Do We Need It?

1. **Query Optimization Foundation**: When you run a SQL query, the database query parser first converts your SQL into an internal **Relational Algebra tree**.
2. **Algebraic Rewriting**: The query optimizer applies mathematical equivalence rules (e.g., pushing $\sigma$ selection below $\bowtie$ join) to reduce intermediate row counts from millions to thousands.
3. **Formal Expressiveness**: Defines "Relational Completeness"—a language is relationally complete if it is at least as expressive as first-order relational algebra.

## How It Works

### The 6 Fundamental Relational Algebra Operators

```text
┌───────────────────────┬────────────┬───────────────────────────────────────┐
│       Operator        │   Symbol   │                Purpose                │
├───────────────────────┼────────────┼───────────────────────────────────────┤
│ Selection             │  σ (sigma) │ Filters rows matching predicate       │
│ Projection            │   π (pi)   │ Selects specific columns/attributes   │
│ Cartesian Product     │  ✕ (cross) │ Combines all pairs of tuples          │
│ Set Union             │  ∪ (union) │ Combines distinct tuples from both    │
│ Set Difference        │  - (minus) │ Tuples in Relation 1 but NOT in 2     │
│ Rename                │  ρ (rho)   │ Renames a relation or its attributes  │
└───────────────────────┴────────────┴───────────────────────────────────────┘
```

---

### Derived Operators

1. **Set Intersection ($\cap$)**:
   $$R \cap S = R - (R - S)$$
2. **Natural Join ($\bowtie$)**: Cartesian product followed by equality selection on all matching attribute names, and projecting away duplicate columns:
   $$R \bowtie S = \pi_{\text{attributes}}(\sigma_{R.A = S.A}(R \times S))$$
3. **Theta Join ($\bowtie_\theta$)**: Cross product filtered by an arbitrary conditional expression $\theta$:
   $$R \bowtie_\theta S = \sigma_\theta(R \times S)$$
4. **Division Operator ($\div$)**: Answers queries involving **"FOR ALL"** or **"EVERY"** (e.g., "Find students enrolled in *all* courses taught by Dr. Smith").

---

### Query Optimization via Relational Algebra Trees

```text
Unoptimized Tree:                        Optimized Tree (Selection Pushed Down):
         π (name)                                     π (name)
            │                                            │
   σ (dept = 'CS' AND salary > 80k)              ⋈ (emp.id = dept.id)
            │                                  ┌──────────┴──────────┐
            ⋈                                  │                     │
      ┌─────┴─────┐                   σ (salary > 80k)          σ (name = 'CS')
   Employee   Department                       │                     │
                                           Employee              Department
```
*Pushing selections down drastically reduces the size of relations before performing the expensive Join operation!*

## Important Terminology

| Term | Symbol | SQL Equivalent |
|---|---|---|
| **Selection** | $\sigma_{\text{condition}}(R)$ | `SELECT * FROM R WHERE condition` |
| **Projection** | $\pi_{A, B}(R)$ | `SELECT DISTINCT A, B FROM R` |
| **Cartesian Product**| $R \times S$ | `SELECT * FROM R CROSS JOIN S` |
| **Natural Join** | $R \bowtie S$ | `SELECT * FROM R NATURAL JOIN S` |
| **Union** | $R \cup S$ | `SELECT * FROM R UNION SELECT * FROM S` |
| **Set Difference** | $R - S$ | `SELECT * FROM R EXCEPT / MINUS SELECT * FROM S`|
| **Rename** | $\rho_{S}(R)$ | `SELECT * FROM R AS S` |

## Example: Formulating Queries

### Query 1: Find names of employees in department 10 earning > 50,000
$$\pi_{\text{name}}(\sigma_{\text{dept\_id} = 10 \;\land\; \text{salary} > 50000}(\text{Employee}))$$

### Query 2: Division Operator ("For All")
*Find IDs of customers who have rented ALL movies in the 'Sci-Fi' category:*
$$R = \pi_{\text{cust\_id, movie\_id}}(\text{Rentals})$$
$$S = \pi_{\text{movie\_id}}(\sigma_{\text{genre} = \text{'Sci-Fi'}}(\text{Movies}))$$
$$\text{Result} = R \div S$$

## Real-World Example

Every relational database engine (PostgreSQL, MySQL, CockroachDB):
- When you execute `EXPLAIN (FORMAT JSON) SELECT ...`, the printed execution plan is literally the evaluated Relational Algebra physical operator tree (`Seq Scan` = $\sigma$, `Hash Join` = $\bowtie$, `Index Scan`).

## Important Properties

| Feature | Relational Algebra | Relational Calculus (TRC/DRC) |
|---|---|---|
| **Nature** | Procedural (step-by-step evaluation order) | Declarative (predicates defining desired result) |
| **Underlying Math**| Set theory and relational operators | First-Order Predicate Logic |
| **Query Engine Role**| Target execution model of Query Optimizer | Conceptual basis for high-level SQL parsing |
| **Expressive Power**| Relationally Complete | Relationally Complete (Codd's Equivalence Theorem)|

### Common Misconceptions

* Relational algebra Projection ($\pi$) automatically eliminates duplicates by default (set semantics), whereas raw SQL `SELECT` retains duplicates unless `DISTINCT` is specified (bag semantics).
* Division ($\div$) is not a fundamental operator; it is derived from Projection, Cartesian Product, and Set Difference.
* Union, Intersection, and Set Difference require **Union Compatibility** (both relations must have the exact same number of attributes with compatible data types).

### What Interviewers Usually Test

1. Converting natural language queries into relational algebra expressions.
2. Formulating the Division operator for "all / every" questions.
3. Explaining how the Query Optimizer pushes down projections and selections to minimize I/O.
4. Differences between Tuple Relational Calculus (TRC) and Domain Relational Calculus (DRC).

### Common Follow-up Questions

1. What is Union Compatibility?
2. How is Division ($\div$) expressed using fundamental operators ($\times, -, \pi$)?
3. What is Codd's Theorem?
4. What is the difference between TRC and DRC?
5. What is a "Safe" expression in Relational Calculus?

### Follow-up Answers

1. Two relations are union-compatible if they have the exact same degree (number of columns) and the domains of corresponding attributes are type-compatible.
2. $R(A, B) \div S(B) = \pi_A(R) - \pi_A((\pi_A(R) \times S) - R)$.
3. Codd's Theorem states that Relational Algebra and Tuple Relational Calculus restricted to safe expressions are mathematically equivalent in expressive power.
4. Tuple Relational Calculus (TRC) uses variables that range over tuples (`{t | t ∈ Employee ∧ t.salary > 50000}`); Domain Relational Calculus (DRC) uses variables that range over attribute domain values (`{<n, s> | <id, n, s> ∈ Employee ∧ s > 50000}`).
5. A calculus expression is safe if it is guaranteed to produce a finite set of tuples regardless of the database contents; unsafe expressions (e.g., `{t | ¬(t ∈ R)}`) could evaluate to an infinite result set.
