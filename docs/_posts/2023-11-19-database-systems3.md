---
layout: post
title:  "Database systems III"
date:   2023-11-19 12:08 +1100
categories: database
layout: math
---
# Database systems III

## Relational database design

### Key points

- Functional dependency
- Normal forms
- Design algorithms for 3NF and BCNF

### Functional dependency

Let X and Y be sets of attributes in R.

X (functionally) determines Y, X → Y , if t1[X] = t2[X] implies t1[Y] = t2[Y]. i.e., f(t[X]) = t[Y].

We also say X → Y is a functional dependency, and that Y is functionally
dependent on X.

X is called the left side, Y the right side of the dependency.

#### Armstrong’s Axioms

- Reflexivity
  $$if \beta \subseteq \alpha,\;then\; \alpha \to \beta$$
- Augmentation
  $$if\; \alpha \to \beta,\;then\; \gamma\alpha \to \gamma\beta$$
- Transitivity
  $$if\; \alpha \to \beta\;and\; \beta \to \gamma,\;then\; \alpha \to \gamma$$
- Additivity
  $$if\; \alpha \to \beta\;and\; \alpha \to \gamma,\; then\; \alpha \to \beta\gamma$$
- Projectivity
  $$if\; \alpha \to \beta\gamma,\;then\; \alpha \to \beta \;and\; \alpha \to \gamma$$
- Pseudo-transitivity
  $$if\; \alpha \to \beta\;and\; \beta\gamma \to \sigma,\; then\; \alpha\gamma \to \sigma$$

#### Closure of F

The set of all dependencies that can be inferred from F is called the closure of F.

- F+ denotes the closure of F.
- F+ includes dependencies in F.

#### Closure of Attributes

Given a set of attributes α, define the closure of α under F (denoted by α+) as the set of attributes that are functionally determined by α under F.

**Algorithm to compute X+**

```
X := X;
change := true;
while change do
    begin
        change := false;
        for each FD W → Z in F do
            begin
                if (W ⊆ X+) and  (Z ⊈ X+) then do
                    begin 
                    X+ := X+ ∪ Z;
                    change := true;
                    end
            end
    end
```

#### Minimal cover

A set F of FD’s is minimal if

1. Every FD X → Y in F is simple: Y consists of a single attribute,
2. Every FD X → A in F is left-reduced: there is no proper subset Y ⊂ X such that X → A can be replaced with Y → A.
3. No FD in F can be removed; that is, there is no FD X → A in F such that (F − {X → A})+ = F+.

#### Find the minimal cover of F**

**Step 1. Reduce right**

- **Input:** F.
- **Output:** right side reduced F'.

For each FD X → Y ∈ F where Y = {A1, A2, ..., Ak}, we use all X → {Ai} (for 1≤ i ≤ k) to replace X → Y.

**Step 2. Reduce left**

- **Input:** right side reduced F.
- **Output:** right and left side reduced F'.

For each X → {A} ∈ F where X = {Ai: 1 ≤ i ≤ k}, do the following: 

For i = 1 to k, replace X with X − {Ai} if A ∈ (X − {Ai})+.

**Step 3. Reduce redundancy**

- **Input:** right and left side reduced F.
- **Output:** a minimum cover F' of F.

For each FD X → {A} ∈ F, remove it from F if: 

A ∈ X+ with respect to F - {X →{A}}.

#### Keys

K is a super key for relation schema R if and only if K → R.

K is a candidate key for R if and only if
- K → R, and
- for no α ⊂ K, α → R

#### Compute all the candidate keys

**Idea:**
- Focus on "necessary" attributes that will definitely appear in ANY candidate key of R.
- Ignore "useless" attribute that will NEVER be part of a candidate key.

**Necessary attributes:**

An attribute A is said to be a necessary attribute if
- A occurs only in the L.H.S. (left hand side) of the fd's in F; or
- A is an attribute in relation R, but A does not occur in either L.H.S. or R.H.S. of any fd in F.

In other words, necessary attributes NEVER occur in the R.H.S. of any fd in F.

**Useless attributes:**

An attribute A is a useless attribute if A occurs ONLY in the R.H.S. of fd’s in F.

**Middle-ground attributes:**

An attribute A in relation R is a middle-ground attribute if A is neither necessary nor useless.

**Algorithm:**

**Input:**
- A relation R = {A1, A2, ..., An}
- F, a set of functional dependencies.

**Output:**
- K = {K1, ..., Kt}, the set of all candidate keys of R.

**Step 1.**

Set F' to a minimal cover of F (This is needed because otherwise we may not detect all useless attributes).

**Step 2.**

Partition all attributes in R into necessary, useless and
middle-ground attribute sets according to F'. 

Let
- X = {C1, ..., Cl} be the necessary attribute set, 
- Y = {B1, ..., Bk} be the useless attribute set, 
- and M = {A1, ..., An} − (X ∪ Y) be the middle-ground attribute set.

If X is empty, then go to step4.

**Step 3.**

Compute X+. If X+ = R, then set K= {X}, terminate.

**Step 4.**

Let L = \<Z1, Z2, ..., Zm\> be the list of all non-empty subsets of M (the middle-ground attributes) such that L is arranged in ascending order of the size of Zi.

$$|L| = 2^{|M|} - 1$$

```
Add all attributes in X (necessary attributes) to each Zi in L.

Set K = {}, i = 0.

While L is not empty do
    Begin
        i = i+1.
        Remove the first element Z from L.
        Compute Z+.
        If Z+ = R, then
            Begin
                Set K = K ∪ {Z};
                For any Zj ∈ L, 
                    if Z ⊂ Zj, then L = L − {Zj}.
            End
    End
```

### First normal form (1NF)

Attribute values are atomic.

> Atomic: multivalued attributes, composite attributes, and their combinations are disallowed.

### Second normal form (2NF)

A relation schema R is in second normal form (2NF) if every nonprime attribute A in R is
not partially dependent on any key of R.

**Prime attribute:** It is a member of some candidate key of R. 

**Full functional dependency:** In an FD X → Y, Y is fully functionally dependent on X if there is no Z ⊂ X such that Z → Y.

**Partial functional dependency:** In an FD X → Y , Y is partially functionally dependent on X if there is any Z ⊂ X such that Z → Y.

### Third normal form (3NF)

A relation scheme is in third normal form (3NF) if for all non-trivial FD’s of the form X → A
- Either X is a superkey
- or A is a prime attribute.

> The 3NF disallows transitive dependencies.

> 1NF - 2NF - 3NF:
> *"The key, the whole key, and nothing but the key, so help me Codd”*

### Boyce-Codd Normal Form (BCNF)

A relation scheme is in Boyce-Codd Normal Form (BCNF) if whenever X → A holds and X → A is non-trivial, X is a superkey.

| Property                                               | 3NF  | BCNF  |
| :----------------------------------------------------- | :--- | :---- |
| Elimination of redundancy due to functional dependency | Most | Yes   |
| Lossless Join                                          | Yes  | Yes   |
| Dependency preservation due to functional dependency   | Yes  | Maybe |

## Data storage

### Key points

- Record format
- Buffer management

## Query optimisation

### Key points

- Index
- Query plan
- Join order selection

## Transaction management

### Key points

- Transcation
- Concurrency control
- Recovery

## NoSQL

### Key points

- NoSQL concept
- Different data model
- Key-Value, Document, Column-family, Graph
