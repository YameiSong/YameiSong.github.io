---
layout: post
title:  "Database systems (I)"
date:   2023-11-17 16:53 +1100
categories: database
layout: math
---

## Data models

### Key points

- ER model
- Relational data model
- Mapping of ER to relational model

### ER model

#### Keys

- **Super key:** a set of one or more attributes that can uniquely identify an entity instance of an entity type.
- **Candidate key:** minimal superkey (no subset is a key).
- **Primary key:** a candidate key chosen by DB designer.
- **Partial key:** a key that is partially unique. i.e., only a subset of the attributes can be identified using it.

Keys are indicated in ER diagrams by <u>underlining</u>.

#### Entity

- **Week entity:** entity types that do not have a key of their own.
    - Identified by a *partial key* and by being related to another entity type - *owner*.
    - The relationship type between a weak entity type to its owner is the *identifying relationship* of the weak entity type.
- **Strong entity:** entity types that have a *primary key* that uniquely identifies all instances.

#### Relationships

- **Cardinality ratio:** the number of relationship instances an entity can
participate in.
- **Degree:** number of participating entity types of a relationship.

**Type constraint on relationships**
- M:N (many to many)
- 1:N (one to many)
- 1:1 (one to one)

![](/assets/images/relationships.png)

**Participation constraint on relationships**
- Total Participation " >=1 ": each entity instance must patriciate in at least one relationship instance. 
    > Example: We want this relationship to express all publications must be written by a person. 

![](/assets/images/total_participation.png)

- Partial Participation " >= 0 ": not necessarily total.
    > Example: Not every person has publication.

**Relationship attributes**

![](/assets/images/relation_attr.png)

**Question:** Why not put the attribute on either researcher or project?

**Answer:** If we put the attribute on the researcher, then we cannot know
which project the researcher has spent time on. If we put the attribute on the
project, then we cannot know who is spending time on that project.

#### Notations

![](/assets/images/er_notations.png)

#### Checklist on ER modeling

1. Did you model every significant entity that has independent instances?
2. Did you model the entity in the correct type? Strong entity or weak entity?
3. Did you capture all the main relationships between entities?
4. Does every relationship have the correct cardinality？
5. Did you correctly capture participation?
6. Is each attribute modeled with the most appropriate attribute type?
7. Did you use the right notation?

### Relational model

#### Keys

- **Primary key:** a designed candidate key.
    > When a relation schema has several candidate keys, choosing a primary key with a single attribute or a small number of attributes is usually better.
- **Foreign key:** an attribute that keeps the value of a primary key of another relation.

#### Integrity Constraints

A valid relation does not violate any integrity constraints:

- **Key constraint:** candidate key values must be unique for every relation
instance.
- **Entity integrity:** an attribute that is part of a primary key cannot be NULL.
- **Referential integrity:** The value of a foreign key must occur in the other relation or be entirely NULL.

Insertion, deletion, and modification may violate integrity constraints.

> Note: all relational integrity constraints have to do with the key values.

**Insertions:** When inserting, we need to check *key constraint*   - check whether
- that the candidate keys are not already present,
- that the value of each foreign key either
  - is all NULL, or
  - is all non-NULL and occurs in the referenced relation.

**Deletions:** When deleting, we need to check *referential integrity*  - check whether the primary key occurs in another relation. If so,
- Delete it (this requires another integrity check, possibly causing
a cascade of deletions), or
- Set the foreign key value to NULL (note this can’t be done if it
is part of a primary key) or other values

**Modifications:**
- If the modified attribute is the primary key
  - the same issues as deleting PK1 and then immediately inserting PK2.
  - make sure deletion and insertion don’t violate any steps.
- If the modified attribute is a foreign key
  - check that the new value refers to an existing tuple.

#### ER to Relational Model Mapping

**STEP 1. Mapping Strong Entity Types**

For each strong entity (not weak entity) type E, create a new relation R with
- Attributes: all simple attributes (and simple components of composite attributes) of E.
- Key: key of E as the primary key for the relation. 

**STEP 2. Mapping Weak Entity Types**

For each weak entity type W with the owner entity type E, create a new relation R with
- Attributes :
  - all simple attributes (and simple components of composite attributes) of W,
  - and include the primary key attributes of the relation derived from E as the foreign key.
- Key of R: foreign key to E and partial key of W.

**STEP 3. Mapping 1:1 Relationship Types**

For each 1:1 relationship type B. Let E and F be the participating entity types. Let S and T be the corresponding relations.
- If none or one of S and T participates totally,
  - choose one of S and T (let S be the one that **participates totally** if there is one).
  - add attributes from the primary key of T to S as a foreign key.
  - add all simple attributes (and simple components of composite attributes) of B as attributes > of S.
- If both participate totally and do not participate in other relationships
  - merge the two entity types and the relationship into a single relation.

**STEP 4. Mapping 1:N Relationship Types**

![](/assets/images/R_1toN.png)

For each 1:N relationship type B. Let E and F be the participating entity types. Let S and T be the corresponding relations. Let E be the entity on the 1 side and F **on the N side**.

Add to the relation belonging to entity T,
- the attributes from the primary key of S as a foreign key.
- any simple attributes (or simple components of composite attributes) from relationship B.

**STEP 5. Mapping M:N Relationship Types**

For each N:M relationship type B. Let E and F be the participating entity types. Let S and T be the corresponding relations.
Create a new relation R (cross-reference) with
- Attributes :
  - Attributes from the key of S as a foreign key,
  - Attributes from the key of T as a foreign key,
  - Simple attributes and simple components of composite attributes of relation B.
- Key: All attributes from the key of S and T.

**STEP 6. Mapping Multivalued Attributes**

For each multivalued attribute A, where A is an attribute of E, create a new relation R.
- If A is a multivalued simple attribute,
  - Attributes of R = Simple attribute A, and key of E as a foreign key.
- If A is a multivalued composite attribute,
  - Attributes of R = All simple components of A, and key of E as a foreign key.

In both cases, the primary key of R is the set of all attributes in R.

**STEP 7. Mapping N-ary Relationship Types**

For each N-ary relationship type (n > 2), create a new relation with
- Attributes: same as Step 5.
- Key: same as Step 5.

| ER MODEL                           | RELATIONAL MODEL                               |
| :--------------------------------- | :--------------------------------------------- |
| ![](/assets/images/er_example.png) | ![](/assets/images/relation_model_example.png) |


| ER MODEL                     | RELATIONAL MODEL                          |
| :--------------------------- | :---------------------------------------- |
| Entity Type                  | Entity relation                           |
| 1:1 or 1:N relationship type | Foreign key (or relationship relation)    |
| M:N relationship type        | Relationship relation and two foreign key |
| n-ary relationship type      | Relationship relation and n foreign key   |
| Simple Attribute             | Attribute                                 |
| Composite Attribute          | Set of simple component attributes        |
| Multivalued Attribute        | Relation and foreign key                  |

## Relational algebra

### Key points

- Be able to use relational algebra to answer question.

### Select

$$\sigma_{<selection\;condition>}(R)$$

#### Example

$$
\sigma_{(age<=24)}(People)\\
\sigma_{(supervisor=1\;AND\;degree\ne"PHD")}(Enrollment)\\
\sigma_{(supervisor=1\;\land\;degree\ne"PHD")}(Enrollment)\\
\sigma_{(color="red"\;OR\;color="blue")}(Item)\\
\sigma_{(color="red"\;\lor\;color="blue")}(Item)
$$

### Project

$$\pi_{<attribute\;list>}(R)$$

#### Example

$$
\pi_{\{department, degree\}}(Enrollment)\\
$$

**Question:** What if we do Projection on only one attribute?
**Answer:** Keep **distinct** values of this attribute.

> **Duplicate elimination:** Relational Algebra is based on sets, so no duplicates are allowed.

### Union

$$R \cup S = \{t: t \in R \;or\; t \in S\}$$

**Condition:** R and S must be union compatible.

> **Union compatibility:** there is a 1-1 correspondence between their attributes:
the same name and same domain.

#### Example

$$
\pi_{\{course \_ id\}}(\sigma_{(semester="Fall"\;\land\;year=2009)}(Courses))\;\cup\;\pi_{\{course \_ id\}}(\sigma_{(semester="Spring"\;\land\;year=2010)}(Courses))
$$

### Intersection

$$R \cap S = \{t: t \in R \;and\; t \in S \}$$

**Condition:** R and S must be union compatible.

#### Example

$$
\sigma_{(supervisor=1)}(Enrollment)\;\cap\;sigma_{(degree\ne"PHD")}(Enrollment)
$$

### Difference

$$R-S = \{t: t \in R \; and \; t \notin S\}$$

**Condition:** R and S must be union compatible.

### Cartesian product

$$R \times S = \{t_{1} \parallel t_{2} : t_{1} \in R \;and\; t_{2} \in S\}$$

- Intuition: every combination of tuples in R with tuples in S.
- t1 \|\| t2 indicates the concatenation of tuples.
- R and S not required to be union compatible, but the number of tuples in the output relations is always \|𝑅\|∗\|S\|.

### Join

#### Theta-join

$$R \bowtie _{<join\;condition>} S = \{t_{1} \parallel t_{2}: t_{1} \in R \;and\; t_{2} \in S \;and\; <join\;condition>\}$$

#### Equi-join

A type of theta-join where the only comparison operator used is “=” is called an Equi-join.

$$Enrollment \bowtie _{supervisor=person\_id}Researcher$$

#### Natural join

A type of equi-join that requires each pair of join attributes to have the same name and domain in both relations.

The join attributes only appear once in the resulting relation.

> Note: In a natural join, there may be several valid pairs of join attributes. Some of them might not be logically correct.

$$Enrollment \bowtie _{(department,\;name),\;(department,\;name)} Courses$$

If there are pairs of joining attributes identically named, we can write

$$Enrollment \bowtie Courses$$

Intuitions:
- Enforce equality on all attributes with same name
- Eliminate one copy of duplicated attributes

#### Compare joins

![](/assets/images/joins.png)

### Divide

The DIVISION operation is applied to two
Relations R and S, where the attributes of S are a subset of the attributes of R.
- The relation returned by the division operator will have attributes = (All attributes of R – All Attributes of S)
- Return all tuples from relation R which are **associated to every S’s tuple.**

<!-- ![](/assets/images/divide.png) -->
<img src="/assets/images/divide.png", width="300">

#### Example

1. Which courses are offered by all departments?

$$Courses \div (\pi _{department}Courses)$$

2. Which courses are offered by all degrees?

$$Courses \div (\pi _{degree}Courses)$$

### Rename

$$\rho_{(new\_name1,\;new\_name2,\;...)}(R)$$

Why use Rename?
- To unify schemas for set operators
- For disambiguation in “self-join”

### Aggregate

We can use an aggregation operator γ and a function such as SUM, AVG, MIN, MAX, or COUNT.

#### Example

Sum of all values of the attribute A:
$$\gamma_{SUM(A)}(R)$$

Group by B and calculate the average of all values of A in each group:
$$\gamma_{B,\;AVG(A)}(R)$$

### Notations of relational algebra

| OPERATION         | NOTATION                                                                 |
| :---------------- | :----------------------------------------------------------------------- |
| SELECT            | $$\sigma_{<selection\;condition>}(R)$$                                   |
| PROJECT           | $$\pi_{<attribute\;list>}(R)$$                                           |
| UNION             | $$R \cup S$$                                                             |
| INTERSECTION      | $$R \cap S$$                                                             |
| DIFFERENCE        | $$R-S$$                                                                  |
| CARTESIAN PRODUCT | $$R \times S$$                                                           |
| THETA-JOIN        | $$R \bowtie _{<join\;condition>} S$$                                     |
| EQUI-JOIN         | $$R \bowtie _{<equal\;join\;condition>} S$$                              |
| NATURALJOIN       | $$R \bowtie _{<(attribute\;list\;of\;R),\;(attribute\;list\;of\;S)>} S$$ |
| DIVISION          | $$R \div S$$                                                             |
| RENAME            | $$\rho_{(new\_name1,\;new\_name2,\;...)}(R)$$                            |
| AGGREGATE         | $$\gamma_{agg\_function(A)}(R)$$ $$\gamma_{B,\;agg\_function(A)}(R)$$    |
