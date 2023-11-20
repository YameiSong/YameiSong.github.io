---
layout: post
title:  "Database systems II: Relational algebra"
date:   2023-11-17 16:53 +1100
categories: database
layout: math
---
# Database systems II: Relational algebra

## Key points

- Be able to use relational algebra to answer question.

## Select

$$\sigma_{<selection\;condition>}(R)$$

### Example

$$
\sigma_{(age<=24)}(People)\\
\sigma_{(supervisor=1\;AND\;degree\ne"PHD")}(Enrollment)\\
\sigma_{(supervisor=1\;\land\;degree\ne"PHD")}(Enrollment)\\
\sigma_{(color="red"\;OR\;color="blue")}(Item)\\
\sigma_{(color="red"\;\lor\;color="blue")}(Item)
$$

## Project

$$\pi_{<attribute\;list>}(R)$$

### Example

$$
\pi_{\{department, degree\}}(Enrollment)\\
$$

**Question:** What if we do Projection on only one attribute?
**Answer:** Keep **distinct** values of this attribute.

> **Duplicate elimination:** Relational Algebra is based on sets, so no duplicates are allowed.

## Union

$$R \cup S = \{t: t \in R \;or\; t \in S\}$$

**Condition:** R and S must be union compatible.

> **Union compatibility:** there is a 1-1 correspondence between their attributes:
the same name and same domain.

### Example

$$
\pi_{\{course \_ id\}}(\sigma_{(semester="Fall"\;\land\;year=2009)}(Courses))\;\cup\;\pi_{\{course \_ id\}}(\sigma_{(semester="Spring"\;\land\;year=2010)}(Courses))
$$

## Intersection

$$R \cap S = \{t: t \in R \;and\; t \in S \}$$

**Condition:** R and S must be union compatible.

### Example

$$
\sigma_{(supervisor=1)}(Enrollment)\;\cap\;sigma_{(degree\ne"PHD")}(Enrollment)
$$

## Difference

$$R-S = \{t: t \in R \; and \; t \notin S\}$$

**Condition:** R and S must be union compatible.

## Cartesian product

$$R \times S = \{t_{1} \parallel t_{2} : t_{1} \in R \;and\; t_{2} \in S\}$$

- Intuition: every combination of tuples in R with tuples in S.
- t1 \|\| t2 indicates the concatenation of tuples.
- R and S not required to be union compatible, but the number of tuples in the output relations is always \|𝑅\|∗\|S\|.

## Join

### Theta-join

$$R \bowtie _{<join\;condition>} S = \{t_{1} \parallel t_{2}: t_{1} \in R \;and\; t_{2} \in S \;and\; <join\;condition>\}$$

### Equi-join

A type of theta-join where the only comparison operator used is “=” is called an Equi-join.

$$Enrollment \bowtie _{supervisor=person\_id}Researcher$$

### Natural join

A type of equi-join that requires each pair of join attributes to have the same name and domain in both relations.

The join attributes only appear once in the resulting relation.

> Note: In a natural join, there may be several valid pairs of join attributes. Some of them might not be logically correct.

$$Enrollment \bowtie _{(department,\;name),\;(department,\;name)} Courses$$

If there are pairs of joining attributes identically named, we can write

$$Enrollment \bowtie Courses$$

Intuitions:
- Enforce equality on all attributes with same name
- Eliminate one copy of duplicated attributes

### Compare joins

![](/assets/images/joins.png)

## Divide

The DIVISION operation is applied to two
Relations R and S, where the attributes of S are a subset of the attributes of R.
- The relation returned by the division operator will have attributes = (All attributes of R – All Attributes of S)
- Return all tuples from relation R which are **associated to every S’s tuple.**

<!-- ![](/assets/images/divide.png) -->
<img src="/assets/images/divide.png", width="300">

### Example

1. Which courses are offered by all departments?

$$Courses \div (\pi _{department}Courses)$$

2. Which courses are offered by all degrees?

$$Courses \div (\pi _{degree}Courses)$$

## Rename

$$\rho_{(new\_name1,\;new\_name2,\;...)}(R)$$

Why use Rename?
- To unify schemas for set operators
- For disambiguation in “self-join”

## Aggregate

We can use an aggregation operator γ and a function such as SUM, AVG, MIN, MAX, or COUNT.

### Example

Sum of all values of the attribute A:
$$\gamma_{SUM(A)}(R)$$

Group by B and calculate the average of all values of A in each group:
$$\gamma_{B,\;AVG(A)}(R)$$

## Notations of relational algebra

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
