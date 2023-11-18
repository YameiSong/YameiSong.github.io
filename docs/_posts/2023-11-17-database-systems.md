---
layout: post
title:  "Database systems"
date:   2023-11-17 16:53 +1100
categories: database
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

![](/assets/images/er_example.png)

![](/assets/images/relation_model_example.png)

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

## Database languages

### Key points

- SQL: views, stored procedures, triggers, aggregates
- PostgreSQL: PLpgSQL

## Relational database design

### Key points

- Functional dependency
- Normal forms
- Design algorithms for 3NF and BCNF

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
