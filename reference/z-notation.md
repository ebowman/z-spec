# Z Notation Reference (fuzz.sty)

Quick reference for Z notation LaTeX commands supported by fuzz.

## Document Structure

```latex
\begin{zed} ... \end{zed}           % General Z paragraphs
\begin{schema}{Name} ... \end{schema}  % Schema definitions
\begin{axdef} ... \end{axdef}       % Axiomatic definitions
\begin{gendef}[X] ... \end{gendef}  % Generic definitions
```

## Basic Types and Sets

| Command | Symbol | Meaning |
|---------|--------|---------|
| `[NAME]` | | Given set (basic type) |
| `\nat` | ℕ | Natural numbers |
| `\num` | ℤ | Integers |
| `\power X` | ℙX | Power set |
| `\finset X` | 𝔽X | Finite subsets |
| `\emptyset` | ∅ | Empty set |

## Set Operations

| Command | Symbol | Meaning |
|---------|--------|---------|
| `\cup` | ∪ | Union |
| `\cap` | ∩ | Intersection |
| `\setminus` | \ | Set difference |
| `\subseteq` | ⊆ | Subset |
| `\in` | ∈ | Membership |
| `\cross` | × | Cartesian product |

## Relations and Functions

| Command | Symbol | Meaning |
|---------|--------|---------|
| `\rel` | ↔ | Relation |
| `\pfun` | ⇸ | Partial function |
| `\fun` | → | Total function |
| `\pinj` | ⤔ | Partial injection |
| `\inj` | ↣ | Total injection |
| `\psurj` | ⤀ | Partial surjection |
| `\surj` | ↠ | Total surjection |
| `\bij` | ⤖ | Bijection |
| `\ffun` | ⇻ | Finite partial function |
| `\finj` | ⤕ | Finite partial injection |

## Relation Operators

| Command | Symbol | Meaning |
|---------|--------|---------|
| `\dom R` | dom R | Domain |
| `\ran R` | ran R | Range |
| `\comp` | ⨾ | Forward composition |
| `\circ` | ∘ | Backward composition |
| `\dres` | ◁ | Domain restriction |
| `\rres` | ▷ | Range restriction |
| `\ndres` | ⩤ | Domain anti-restriction |
| `\nrres` | ⩥ | Range anti-restriction |
| `\inv` | ∼ | Relational inverse |
| `\limg R \rimg` | R⦇...⦈ | Relational image |
| `\oplus` | ⊕ | Override |

## Sequences

| Command | Symbol | Meaning |
|---------|--------|---------|
| `\seq X` | seq X | Sequences of X |
| `\seq_1 X` | seq₁ X | Non-empty sequences |
| `\iseq X` | iseq X | Injective sequences |
| `\langle ... \rangle` | ⟨...⟩ | Sequence literal |
| `\cat` | ⌢ | Concatenation |
| `\dcat` | ⌢/ | Distributed concatenation |
| `\head`, `\tail`, `\last`, `\front` | | Sequence operations |
| `\rev` | | Reverse |
| `\squash` | | Squash (compact) |

## Logic

| Command | Symbol | Meaning |
|---------|--------|---------|
| `\land` | ∧ | Conjunction |
| `\lor` | ∨ | Disjunction |
| `\lnot` | ¬ | Negation |
| `\implies` | ⇒ | Implication |
| `\iff` | ⇔ | Equivalence |
| `\forall` | ∀ | Universal quantifier |
| `\exists` | ∃ | Existential quantifier |
| `\exists_1` | ∃₁ | Unique existence |

## Schema Notation

| Command | Meaning |
|---------|---------|
| `\Delta S` | State change (includes S and S') |
| `\Xi S` | No state change (S = S') |
| `\where` | Separates declarations from predicates |
| `x?` | Input variable |
| `x!` | Output variable |
| `x'` | After-state variable |
| `\theta S` | Binding of schema S |

## Arithmetic

| Command | Symbol | Meaning |
|---------|--------|---------|
| `\div` | div | Integer division |
| `\mod` | mod | Modulo |
| `\upto` | .. | Range (1 \upto 10) |
| `\leq` | ≤ | Less than or equal |
| `\geq` | ≥ | Greater than or equal |
| `\neq` | ≠ | Not equal |

## Keywords

| Command | Meaning |
|---------|---------|
| `\LET x == e @` | Local definition |
| `\IF p \THEN e_1 \ELSE e_2` | Conditional |
| `\lambda x : T @ e` | Lambda abstraction |
| `\mu x : T \| P` | Definite description |

## Free Types

```latex
\begin{zed}
Direction ::= receive | send
Status ::= pending | active | completed
\end{zed}
```

## Common Patterns

### State Schema with Invariant
```latex
\begin{schema}{Account}
balance : \nat \\
limit : \nat
\where
balance \leq limit
\end{schema}
```

### Operation Schema
```latex
\begin{schema}{Deposit}
\Delta Account \\
amount? : \nat_1
\where
balance' = balance + amount? \\
limit' = limit
\end{schema}
```

### Initialization Schema
```latex
\begin{schema}{InitAccount}
Account'
\where
balance' = 0 \\
limit' = 1000
\end{schema}
```

## Fuzz Limitations

- No tuple projection (`.1`, `.2`) - use named schema fields instead
- No `\boolean` type - define as free type: `ZBOOL ::= ztrue | zfalse`
- No `\min`/`\max` - use conditional predicates
- Cardinality `#` on expressions can be tricky - may need reformulation

## ProB Limitations

For probcli animation and model checking:

- **Avoid B keywords**: Don't use `BOOL`, `TRUE`, `FALSE`, `true`, `false` - use `ZBOOL`, `ztrue`, `zfalse`
- **Concrete function values**: Abstract functions (e.g., `f : A \pinj B` with only domain constraints) cannot be animated - provide explicit mappings
- **Bounded integers**: Add upper bounds (`x \leq 1000`) to avoid unbounded enumeration
- **Unified Init**: Create a single `Init` schema for initialization
- **Given set cardinality**: Sets default to size 2-5 in model checking
