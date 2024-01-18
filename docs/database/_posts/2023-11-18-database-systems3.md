---
layout: post
title:  "Database systems III: Database languages"
date:   2023-11-18 17:14 +1100
categories: database
layout: math
---
# Database systems III: Database languages

## Key points

- SQL: views, stored procedures, triggers, aggregates
- PostgreSQL: PLpgSQL

## SQL

### Classification

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

### String

**IMPORTANT:** SQL String uses single quote!

Double quotes are used to indicate identifiers within the database, which are objects like tables, column names, and roles. In contrast, single quotes are used to indicate string literals.

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

### Date

- Format is typically DD-Mon-YYYY, e.g., '18-Aug-1998'. Accepts other formats.
- Comparison operators implement before (<) and after (>).
- (start1, end1) OVERLAPS (start2, end2)
  ```sql
  SELECT (DATE'2001-02-16', DATE'2001-12-21') OVERLAPS (DATE'2001-10-30', DATE'2002-10-30'); 
  -- Result: true
  ```

### NULL

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

### AS

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

### Exists

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

### Quantifiers

**ALL**

Find the names of all instructors whose salary is greater than the salary of all instructors in the Comp. Sci. department.

```sql
SELECT name
FROM instructor
WHERE salary > ALL
(SELECT salary
FROM instructor
WHERE dept_name='Comp. Sci.');
```

**ANY**

Find the students who fail at least one course in T1.

```sql
SELECT zid, name
FROM students
WHERE 50 > ANY
(SELECT score
FROM enrollment
WHERE students.zid= enrollment.zid AND term='T1');
```

### Set operators

- union
- intersect
- expect

Find courses that ran in Fall 2009 or in Spring 2010.

```sql
(select course_id from section where sem = 'Fall' and year = 2009)
union
(select course_id from section where sem = 'Spring' and year = 2010)
```

Find courses that ran in Fall 2009 and in Spring 2010.

```sql
(select course_id from section where sem = 'Fall' and year = 2009)
intersect
(select course_id from section where sem = 'Spring' and year = 2010)
```

Find courses that ran in Fall 2009 but not in Spring 2010.

```sql
(select course_id from section where sem = 'Fall' and year = 2009)
except
(select course_id from section where sem = 'Spring' and year = 2010)
```

> Note: Each of the above operations will eliminate duplicates.
> 
> To keep duplicates, use union all, intersect all, except all.

### Divide

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

### Create table

```sql
CREATE TABLE employee (
  id INT PRIMARY KEY,
  name VARCHAR(20) NOT NULL,
  works_at VARCHAR(20),
  FOREIGN KEY (works_at) REFERENCES company(id) ON DELETE CASCADE 
);
```

### Drop table

```sql
DROP TABLE person;
```

### Insert

Insert tuples using values:

```sql
INSERT INTO Likes VALUES ('Justin', 'Old');

INSERT INTO Sells (price,bar) VALUES (2.50, 'Coogee Bay Hotel');
```

Insert tuples using select:

```sql
INSERT INTO Relation (Subquery);
```

### Delete tuples

```sql
DELETE FROM Likes
WHERE drinker = 'Justin'
AND beer = 'Sparkling Ale';
```

Omitting the WHERE Clause deletes all tuples from relation R.

```sql
DELETE FROM R;
-- This doesn't drop the table, the table still remains
```

### Update tuples

```sql
UPDATE Drinkers
SET addr = 'Coogee' , phone = '9665-4321'
WHERE name = 'John';
```

### View

Tables (created by CREATE TABLE) are base relations, which are physically stored in the DBMS.

Views are virtual relations in SQL, which are derived from base relations and does not take up space in the DBMS.

View are defined via:
```sql
CREATE VIEW View_name AS Query
```

Views may be removed via:
```sql
DROP VIEW View_name
```

Views update themselves automatically, if changes occur in the underlying relation(s).

### Alter table

Sometimes, we want to make changes to the table schema.

The definition of a base table or of other named schema elements can be changed by using the ALTER TABLE command.

- Add column(s) of table
  ```sql
  -- Add column phone numbers to table hotels.
  ALTER TABLE Bars
  ADD phone char(10) DEFAULT 'Unlisted';
  ```
- Delete column(s) of table
- Modify column(s) of table
  ```sql
  -- Changing the primary key
  ALTER TABLE Persons DROP PRIMARY KEY;
  -- OR
  ALTER TABLE Persons ADD PRIMARY KEY (ID);
  ```

### Create index

```sql
-- duplicate index values are allowed
CREATE INDEX index_name
ON table_name (column1, column2, ...);

-- no duplicate index value is allowed
CREATE UNIQUE INDEX index_name
ON table_name (column1, column2, ...); 
```

## PLpgSQL

### What pure SQL can't do?

- Implementing user interactions
- Control sequences of database operations
- Process query results in additional ways

These can be handled by PostgreSQL.

### User-defined data types

1. **Create Domain:** define a new atomic type.

    ```sql
    CREATE DOMAIN DomainName [ AS ] DataType
    [ DEFAULT expression ]
    [ CONSTRAINT ConstrName constraint ];
    ```

    ```sql
    Create Domain UnswCourseCode as text
    check ( value ~ '[A-Z]{4}[0-9]{4}' );
    ```

    `UnswCourseCode` can then be used like other SQL atomic types.

    ```sql
    Create Table Course (
      id integer,
      code UnswCourseCode, ...
    );
    ```

2. **Create type:** define a new tuple type.

    ```sql
    CREATE TYPE TypeName AS 
    (
      AttrName1 DataType1, 
      AttrName2 DataType2, 
      ...
    );
    ```

    ```sql
    Create type CourseInfo as 
    (
      course UnswCourseCode,
      syllabus text,
      lecturer text
    );
    ```

CREATE TYPE is different from CREATE TABLE:
- does not create a new (empty) table
- does not provide for key constraints
- does not have explicit specification of domain constraints
- used for specifying **return types of functions** that return tuples
or sets.

### Function

```sql
CREATE OR REPLACE FUNCTION
  funcName(param1, param2, ....)
  RETURNS rettype
AS $$
  DECLARE
    variable declarations
  BEGIN
    code for function
  END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE OR REPLACE FUNCTION
  add(x text, y text) RETURNS text
AS $$
  DECLARE
    result text; -- local variable
  BEGIN
    result := x||''''||y;
    return result;
  END;
$$ LANGUAGE 'plpgsql';
```

You can declare a variable to have the same type as a row from a table using `<table_name>%ROWTYPE`.

You may also refer to an attribute type using and specifying `<table_name>.<column_name>%TYPE`.

Example:
```sql
quantity INTEGER;
start_quantity quantity%TYPE;
employee Employees%ROWTYPE;
name Employees.name%TYPE;
```

You can capture query results via:

```sql
SELECT Expr1, Expr2
INTO Var1, Var2
From R
WHERE Condition
```

```sql
-- cost is local var, price is attr
SELECT price INTO cost
FROM StockList
WHERE item = 'Cricket Bat';

cost := cost * (1 + tax_rate);
total := total + cost ;
```

### Control structure

```sql
if condition_1 then
  statement_1;
elsif condition_2 then
  statement_2;
else
  else-statement;
end if;
```

```sql
LOOP
  statement
END LOOP;
```

```sql
FOR int_var IN low .. high LOOP
  statement
END LOOP;
```

### Exceptions

```sql
BEGIN
  Statements ...
EXCEPTION
  WHEN Exceptions1 THEN
    StatementsForHandler1
  WHEN Exceptions2 THEN
    StatementsForHandler2
  ...
END;
```

Example:

```sql
-- Table T contains one tuple ('Tom', 'Jones')
DECLARE
  x INTEGER := 3;
BEGIN
  UPDATE T SET firstname = 'Joe' WHERE lastname = 'Jones';
  -- Table T now contains ('Joe', 'Jones')
  x := x + 1;
  y := x / 0;
EXCEPTION
  WHEN division_by_zero THEN
  -- update on T is rolled back to ('Tom', 'Jones')
  RAISE NOTICE 'Caught division_by_zero';
  RETURN x ;
  -- value returned is 4
END ;
```

The RAISE operator generates server log entries, e.g.
```sql
RAISE DEBUG ' Simple message ';
RAISE NOTICE ' User = % ', user_id ;
RAISE EXCEPTION ' Fatal : value was % ', value ;
```

There are several levels of severity:
- DEBUG, LOG, INFO, NOTICE, WARNING, and EXCEPTION.
- not all severities generate a message to the client.

### Cursor

Implicit cursor in FOR LOOP:

```sql
Create Function totalSalary() Returns real As $$
Declare
  employee RECORD;
  totalSalary REAL:=0;
Begin
  FOR employee IN SELECT * FROM Employees
  Loop
    totalSalary:=totalSalary+employee.salary;
  End Loop;
  Return total;
End; 
$$ Language plpgsql; 
```

Explicit cursor:
- Bound cursor (bound to a specific query)
  ```sql
  <cursor_name_a> CURSOR FOR <query_b>;

  OPEN <cursor_name_a>;
  ...
  CLOSE <cursor_name_a>;
  ```
- Unbound cursor (declared without reference to any query)
  ```sql
  <cursor_name_c> REFCURSOR;

  OPEN <cursor_name_c> FOR <query_d>;
  ...
  CLOSE <cursor_name_c>;

  OPEN <cursor_name_c> FOR <query_e>;
  ...
  CLOSE <cursor_name_c>;
  ```

The fetch operator retrieves the next row from the cursor into a target.

```sql
FETCH e INTO me;
FETCH e INTO my_id , my_name , my_salary;
```

> Note: The variables need to match the corresponding type form the return table. 

Example:
```sql
DECLARE
  employee Employee%ROWTYPE;
  e CURSOR FOR Select * From Employees ;
  totalSalary REAL:=0;
Begin
  OPEN e;
    LOOP
      FETCH e INTO employee;
      EXIT WHEN NOT FOUND;
      totalSalary := totalSalary+employee.salary;
    END LOOP;
  CLOSE e;
End;
```

### Trigger

Event-condition-action rules approach:
- an event activates the trigger
- on activation, the trigger checks a condition
- if the condition holds, a procedure is executed (the action)

```sql
CREATE TRIGGER TriggerName
AFTER/BEFORE Event1 [OR Event2 ...]
ON TableName
FOR EACH ROW/STATEMENT
EXECUTE PROCEDURE FunctionName(args...);
```

PostgreSQL triggers provide a mechanism for INSERT, DELETE or UPDATE events to automatically activate PLpgSQL functions.

A trigger is defined, there needs to be a trigger procedure.

```sql
-- Create a trigger
CREATE TRIGGER TriggerName
...
EXECUTE PROCEDURE function_name(args...);

-- Create the trigger procedure
CREATE OR REPLACE FUNCTION function_name() RETURNS
TRIGGER
...
```

The trigger function also receives two variables NEW and OLD that contains the new and old row version, respectively.

Depending on the trigger, NEW and OLD variables can be accessed.

| Trigger | NEW  | OLD  |
| :------ | :--- | :--- |
| Insert  | Yes  | No   |
| Update  | Yes  | Yes  |
| Delete  | No   | Yes  |

**Example Scenario:**
- Employee(id, name, address, deptartment, salary)
- Department(id, name, manager, totSal)

These natural events could affect the validity of the database:
- a new employee beginning work in some department
- an employee getting a rise in salary
- an employee changing from one department to another
- an employee leaving the company 

Case 1: A new employees arrives
```sql
Create trigger TotalSalary1
after insert on Employees
for each row execute procedure totalSalary1();

Create function totalSalary1() returns trigger
as $$
begin
  if (new.dept is not null) then
    update Department
    set totSal = totSal + new.salary
    where Department.id = new.dept;
  end if;
  return new;
end;
$$ language plpgsql;
```

Case 2: An employees change departments/salaries
```sql
Create trigger TotalSalary2
after update on Employee
for each row execute procedure totalSalary2();

Create function totalSalary2() returns trigger
as $$
begin
  update Department
  set totSal = totSal + new.salary
  where Department.id = new.dept;

  update Department
  set totSal = totSal - old.salary 
  where Department.id = old.dept;
  
  return new;
end;
$$ language plpgsql;
```

Case 3: An employee leaves
```sql
Create trigger TotalSalary3
after delete on Employee
for each row execute procedure totalSalary3();
Create function totalSalary3() returns trigger
as $$
begin
  if (old.department is not null) then
    update Department
    set totSal = totSal - old.salary 
    where Department.id = old.deptartment;
  end if;
  return old;
end;
$$ language plpgsql;
```

General database trigger usage scenarios:

| Scenario                                                        | WHEN   |
| :-------------------------------------------------------------- | :----- |
| To maintain a separate table for summary data                   | AFTER  |
| Checking schema-level constraints (assertions) on insert/update | BEFORE |
| To perform updates across tables (to maintain assertions)       | AFTER  |
