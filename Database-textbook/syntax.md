# Oracle SQL and PL/SQL Quick Reference

This reference reorganizes the original mixed SQL/PLSQL syntax sheet. Placeholders in angle brackets describe values you must replace; do not type the brackets.

## Connect and prepare a learner account

```console
sqlplus / as sysdba
```

Administrator-run setup:

```sql
CREATE USER <user_name> IDENTIFIED BY <password>;
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW TO <user_name>;
ALTER USER <user_name> QUOTA 50M ON users;
```

Avoid granting the `DBA` role to a normal learner account.

## Tables and rows

### Create and inspect

```sql
CREATE TABLE <table_name> (
    <column_1> <data_type> [constraint],
    <column_2> <data_type> [constraint]
);

DESCRIBE <table_name>
```

Common Oracle data types include `NUMBER(p,s)`, `VARCHAR2(n)`, `CHAR(n)`, `DATE`, `TIMESTAMP`, `CLOB`, and `BLOB`.

### Insert

```sql
INSERT INTO <table_name> (<column_1>, <column_2>)
VALUES (<value_1>, <value_2>);
```

### Query

```sql
SELECT * FROM <table_name>;

SELECT <column_1>, <column_2>
FROM <table_name>
WHERE <condition>
ORDER BY <column_1> [ASC | DESC];
```

### Update and delete

```sql
UPDATE <table_name>
SET <column_1> = <value_1>
WHERE <condition>;

DELETE FROM <table_name>
WHERE <condition>;
```

Without `WHERE`, every row is affected.

## Transactions

```sql
COMMIT;
ROLLBACK;
SAVEPOINT <savepoint_name>;
ROLLBACK TO <savepoint_name>;
```

`COMMIT` saves the current transaction. `ROLLBACK` discards uncommitted work. Oracle DDL statements implicitly commit, so `TRUNCATE`, `DROP`, and most `ALTER` operations are not undone with normal rollback.

## Remove data or objects

```sql
DELETE FROM <table_name>;     -- DML; rollback is possible before commit
TRUNCATE TABLE <table_name>; -- DDL; removes every row and keeps the table
DROP TABLE <table_name>;     -- DDL; removes the table object
```

## Alter and rename

```sql
ALTER TABLE <table_name> ADD <column_name> <data_type>;
ALTER TABLE <table_name> MODIFY <column_name> <data_type>;
ALTER TABLE <table_name> DROP COLUMN <column_name>;
ALTER TABLE <table_name>
    RENAME COLUMN <old_column> TO <new_column>;

RENAME <old_table_name> TO <new_table_name>;
```

Oracle does not support the MySQL column-position clauses `ADD ... AFTER ...` or `ADD ... FIRST`, and its standalone rename syntax does not include the word `TABLE`.

## Constraints

```sql
CREATE TABLE employee (
    employee_id NUMBER(5)
        CONSTRAINT employee_pk PRIMARY KEY,
    email VARCHAR2(100)
        CONSTRAINT employee_email_uk UNIQUE,
    employee_name VARCHAR2(50) NOT NULL,
    salary NUMBER(8,2)
        CONSTRAINT employee_salary_ck CHECK (salary >= 0),
    department_id NUMBER(5),
    CONSTRAINT employee_department_fk
        FOREIGN KEY (department_id)
        REFERENCES department (department_id)
);
```

| Constraint | Rule |
|---|---|
| `PRIMARY KEY` | Unique, non-null row identifier |
| `FOREIGN KEY` | Non-null value must match a referenced primary/unique key |
| `UNIQUE` | Non-null values/combinations do not repeat |
| `NOT NULL` | Column must contain a value |
| `CHECK` | Row must not make the condition false |

## Join syntax

```sql
SELECT e.employee_name, p.product_name
FROM employee e
JOIN product p ON p.employee_id = e.employee_id;

SELECT e.employee_name, p.product_name
FROM employee e
LEFT JOIN product p ON p.employee_id = e.employee_id;

SELECT e.employee_name, p.product_name
FROM employee e
RIGHT JOIN product p ON p.employee_id = e.employee_id;
```

Always include the relationship in `ON`. Filtering only one table without relating the tables creates unwanted row combinations.

## Operators and predicates

| Syntax | Purpose |
|---|---|
| `=`, `<>`, `<`, `>`, `<=`, `>=` | Compare values |
| `LIKE` | Match a pattern (`%` any length, `_` one character) |
| `IN (...)` | Match a member of a list or subquery |
| `BETWEEN a AND b` | Match an inclusive range |
| `IS NULL`, `IS NOT NULL` | Test nullness |
| `AND`, `OR`, `NOT` | Combine or negate conditions |

```sql
SELECT * FROM emp_info WHERE employee_name LIKE '%Singh';
SELECT * FROM emp_info WHERE employee_name LIKE '___s%'; -- s is fourth
SELECT * FROM emp_info WHERE salary BETWEEN 50000 AND 65000;
SELECT * FROM emp_info WHERE department IN ('HR', 'Development');
```

## Aggregates and distinct values

```sql
SELECT
    SUM(salary) AS total_salary,
    MAX(salary) AS maximum_salary,
    MIN(salary) AS minimum_salary,
    AVG(salary) AS average_salary,
    COUNT(salary) AS non_null_salary_count,
    COUNT(*) AS row_count
FROM emp_info;

SELECT DISTINCT department
FROM emp_info;
```

`COUNT(column)` ignores nulls; `COUNT(*)` counts rows. `DISTINCT` is a keyword, not a function.

## Subqueries

```sql
SELECT *
FROM emp_info
WHERE salary = (SELECT MAX(salary) FROM emp_info);
```

The inner query runs as part of the outer query. A scalar comparison such as `=` requires the subquery to return at most one row.

## Dates and time in Oracle

```sql
SELECT SYSDATE, SYSTIMESTAMP FROM dual;

INSERT INTO event_log (event_time)
VALUES (SYSTIMESTAMP);

SELECT TO_DATE('2026-10-28', 'YYYY-MM-DD') FROM dual;
```

`CURDATE()` and `NOW()` are MySQL functions; use Oracle's `SYSDATE`, `CURRENT_DATE`, `SYSTIMESTAMP`, or `CURRENT_TIMESTAMP` as appropriate.

## Views

```sql
CREATE VIEW employee_public AS
SELECT employee_id, employee_name, department
FROM employee;

SELECT * FROM employee_public;
DROP VIEW employee_public;
```

A simple single-table view may be updatable. Joins, grouping, aggregates, `DISTINCT`, and other features can make a view non-updatable.

## Composite primary keys

```sql
CREATE TABLE shipment (
    supplier_no VARCHAR2(5),
    part_no     VARCHAR2(5),
    quantity    NUMBER(5),
    CONSTRAINT shipment_pk PRIMARY KEY (supplier_no, part_no)
);
```

This primary key is **composite** because it has two columns. If the pair is a minimal unique identifier, it is also a candidate key. Those terms are related but not interchangeable.

## Create a table from a query (CTAS)

```sql
CREATE TABLE emp_info_2 AS
SELECT employee_id, employee_name, salary
FROM emp_info;
```

CTAS copies the query result and compatible column definitions, but it does not automatically reproduce all keys, checks, defaults, indexes, or privileges from the source.

## Key vocabulary

- **Super key:** any column set that uniquely identifies a row.
- **Candidate key:** a minimal super key.
- **Primary key:** the chosen candidate key.
- **Alternate key:** a candidate key not selected as primary.
- **Composite key:** a key containing multiple columns.
- **Foreign key:** a reference to a primary or unique key.

“Component key” is not a standard relational-key category; it may informally mean one column within a composite key.

## Normalization overview

| Form | Main idea |
|---|---|
| 1NF | Each attribute contains atomic values; no repeating groups |
| 2NF | In 1NF, and every non-key attribute depends on the whole candidate key |
| 3NF | In 2NF, and non-key attributes do not depend transitively on a key |
| BCNF | Every determinant is a candidate key |
| 4NF | In BCNF and no non-trivial multivalued dependencies |

BCNF is normally studied after 3NF; the order in the original list has been corrected here.

## PL/SQL block

Enable printed output in SQL\*Plus:

```sql
SET SERVEROUTPUT ON;
```

General form:

```sql
DECLARE
    -- Optional variables, constants, cursors, local routines
BEGIN
    -- Required executable statements
EXCEPTION
    -- Optional exception handlers
END;
/
```

### Hello world

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello World');
END;
/
```

### Calculator

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Sum = ' || (a + b));
    DBMS_OUTPUT.PUT_LINE('Difference = ' || (a - b));
    DBMS_OUTPUT.PUT_LINE('Product = ' || (a * b));

    IF b = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Division is undefined: b is zero');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Quotient = ' || (a / b));
    END IF;
END;
/
```

`&a` and `&b` are SQL\*Plus substitution variables, not PL/SQL input operators.

## Conditional forms

```sql
IF condition THEN
    NULL;
END IF;

IF condition THEN
    NULL;
ELSE
    NULL;
END IF;

IF condition_1 THEN
    NULL;
ELSIF condition_2 THEN
    NULL;
ELSE
    NULL;
END IF;
```

PL/SQL spells the ladder keyword `ELSIF`, not `ELSEIF`.

### Even or odd

```sql
DECLARE
    n PLS_INTEGER := &n;
BEGIN
    IF MOD(n, 2) = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Even');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Odd');
    END IF;
END;
/
```

## Loop forms

### Basic loop

```sql
DECLARE
    i PLS_INTEGER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE(i);
        i := i + 1;
        EXIT WHEN i > 5;
    END LOOP;
END;
/
```

### While loop: multiplication table

```sql
DECLARE
    n PLS_INTEGER := &n;
    i PLS_INTEGER := 1;
BEGIN
    WHILE i <= 10 LOOP
        DBMS_OUTPUT.PUT_LINE(n || ' * ' || i || ' = ' || (n * i));
        i := i + 1;
    END LOOP;
END;
/
```

The original condition `WHILE n <= 10` never changed `n`, so inputs up to 10 caused an infinite loop. The counter `i` is the value that must be tested.

### For loop

```sql
BEGIN
    FOR i IN 1..10 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
END;
/
```

The integer loop variable is declared implicitly and is local to the loop.

## Related lessons

- [Full SQL\*Plus workbook](Connection.md)
- [Views and constraints](VIEW.md)
- [PL/SQL fundamentals](PLSQL/PLSQL.md)
- [PL/SQL worked examples](PLSQL/PLSQL_CODE.md)

> Original classroom source: [syntax.txt](syntax.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
