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
- **Referential integrity:** The value of FK must occur in the other relation or be entirely NULL.

Insertion, deletion, and modification may violate integrity constraints.

> Note: all relational integrity constraints have to do with the key values.

**Insertions:** When inserting, we need to check *key constraint* – check whether
- that the candidate keys are not already present,
- that the value of each foreign key either
  - is all NULL, or
  - is all non-NULL and occurs in the referenced relation.

**Deletions:** When deleting, we need to check *referential integrity* – check whether the primary key occurs in another relation. If so,
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
