---
layout: post
title:  "Database systems II"
date:   2023-11-18 17:14 +1100
categories: database
layout: math
---
# Database systems II

## Database languages

### Key points

- SQL: views, stored procedures, triggers, aggregates
- PostgreSQL: PLpgSQL

### SQL

#### Classification

Data definition language (DDL):
- CREATE TABLE
- DROP TABLE

Data manipulation language (DML):
- SELECT
  - GROUP BY
  - HAVING
  - ORDER BY
- INSERT
- DELETE
- UPDATE
- ALTER
- ...

#### String

Quotes are escaped by doubling them -> ''

```sql
SELECT Name FROM Beers WHERE Manf = 'Toohey''s';
```

String operators:

| Expression                 | Function            |
| :------------------------- | :------------------ |
| str1 \|\| str2             | concatenate strings |
| LENGTH(str)                | length              |
| SUBSTR(str, start, length) | substring           |

LIKE operator:
- The symbol _ (underscore) matches any single characters
- The symbol % (percent) matches zero or more characters

#### Date

- Format is typically DD-Mon-YYYY, e.g., '18-Aug-1998'. Accepts other formats.
- Comparison operators implement before (<) and after (>).
- (start1, end1) OVERLAPS (start2, end2)
  ```sql
  SELECT (DATE'2001-02-16', DATE'2001-12-21') OVERLAPS (DATE'2001-10-30', DATE'2002-10-30'); 
  -- Result: true
  ```

#### NULL

1. Comparisons with null returns unknown.
    > Example: 5 < null, null <> null, null = null
2. Three-valued logic using the truth value unknown
   - OR
     - (unknown or true) = true,
     - (unknown or false) = unknown,
     - (unknown or unknown) = unknown
   - AND
     - (true and unknown) = unknown,
     - (false and unknown) = false,
     - (unknown and unknown) = unknown
   - NOT
     - (not unknown) = unknown
3. "P is unknown" evaluates to true if predicate P evaluates as unknown.
4. Result of where clause predicate is treated as false if it evaluates as unknown.
5. Aggregation ignores NULL values if present in the column specified.
   > All aggregate operations ignore tuples with null values on the aggregated attributes, except COUNT(*), which counts the number of
rows.

**Question:** What would count(A1) if all values in column in A1 were NULL?

**Answer:** 0

**Question:** What would max(A1) if all values in column in A1 were NULL? 

**Answer:** NULL

#### AS

- Renaming
- Expression as values in columns
  ```sql
  SELECT bar, beer, price*120 AS PriceInYen
  FROM Sells;
  ```
- Inserting text in result table
  ```sql
  SELECT drinker, 'likes Cooper''s' AS WhoLikes 
  FROM Likes 
  WHERE beer = 'Sparkling Ale';
  ```

#### Exists

Exists keyword returns true if the relation is non-empty.

Example: find those beers that are the unique beer by their manufacturer.

```sql
SELECT name
FROM Beers b1
WHERE NOT EXISTS
(SELECT *
FROM Beers
WHERE manf = b1.manf AND name != b1.name);
```

#### Quantifiers

**ALL**

Find the names of all instructors whose salary is greater than the salary of all instructors in the Comp. Sci. department.

```sql
SELECT name
FROM instructor
WHERE salary > ALL
(SELECT salary
FROM instructor
WHERE dept_name=’Comp. Sci.’);
```

**ANY**

Find the students who fail at least one course in T1.

```sql
SELECT zid, name
FROM students
WHERE 50 > ANY
(SELECT score
FROM enrollment
WHERE students.zid= enrollment.zid AND term=‘T1’);
```

#### Set operators

- union
- intersect
- expect

Find courses that ran in Fall 2009 or in Spring 2010.

```sql
(select course_id from section where sem = ‘Fall’ and year = 2009)
union
(select course_id from section where sem = ‘Spring’ and year = 2010)
```

Find courses that ran in Fall 2009 and in Spring 2010.

```sql
(select course_id from section where sem = ‘Fall’ and year = 2009)
intersect
(select course_id from section where sem = ‘Spring’ and year = 2010)
```

Find courses that ran in Fall 2009 but not in Spring 2010.

```sql
(select course_id from section where sem = ‘Fall’ and year = 2009)
except
(select course_id from section where sem = ‘Spring’ and year = 2010)
```

> Note: Each of the above operations will eliminate duplicates.
> 
> To keep duplicates, use union all, intersect all, except all.

#### Divide

Example: Find bars each of which sells all the beers Justin likes.

Relational Algebra:
$$\pi_{bar,\;beer}Sells \div (\pi_{beer}(\sigma_{drinker='Justin'}Likes))$$

```sql
SELECT DISTINCT a.bar
FROM sells a
WHERE NOT EXISTS
(
  (SELECT b.beer FROM likes b WHERE b.drinker = 'Justin')
  EXCEPT
  (SELECT c.beer FROM sells c WHERE c.bar = a.bar)
);
```

#### Create table

```sql
CREATE TABLE employee (
  id INT PRIMARY KEY,
  name VARCHAR(20) NOT NULL,
  works_at VARCHAR(20),
  FOREIGN KEY (works_at) REFERENCES company(id) ON DELETE CASCADE 
);
```

#### Drop table

```sql
DROP TABLE person;
```

#### Insert

Insert tuples using values:

```sql
INSERT INTO Likes VALUES (’Justin’, ’Old’);

INSERT INTO Sells (price,bar) VALUES (2.50, ’Coogee Bay Hotel’);
```

Insert tuples using select:

```sql
INSERT INTO Relation (Subquery);
```

#### Delete tuples

```sql
DELETE FROM Likes
WHERE drinker = ’Justin’
AND beer = ’Sparkling Ale’;
```

Omitting the WHERE Clause deletes all tuples from relation R.

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
