# Oracle SQL*Plus: Connection and SQL Workbook

This lesson turns the original SQL\*Plus terminal session into a repeatable beginner workflow. It covers account setup, tables, row operations, transactions, structural changes, joins, sorting, operators, aggregate functions, and subqueries.

## 1. Connect to Oracle

From a command prompt on the database host, an administrator may connect with operating-system authentication:

```console
sqlplus / as sysdba
```

The original class used Oracle Database 11g Express Edition. Modern installations may use a pluggable-database service, so the exact connection string can differ.

> [!CAUTION]
> `SYSDBA` is an administrative connection. Use it only to perform administrator tasks, then reconnect as a restricted practice user.

### Create a practice user

The historical session used:

```sql
CREATE USER appdb2 IDENTIFIED BY test;
GRANT DBA TO appdb2;
CONNECT appdb2/test
```

That works in some older learning environments but grants far more authority than a student needs. A safer administrator-run example is:

```sql
CREATE USER appdb2 IDENTIFIED BY "choose-a-strong-password";
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW TO appdb2;
ALTER USER appdb2 QUOTA 50M ON users;
```

Then connect using the service configured for your database:

```console
sqlplus appdb2@service_name
```

SQL\*Plus prompts for the password without exposing it in shell history.

## 2. Create and inspect a table

A table stores rows; its columns define the attributes and allowed data types.

```sql
CREATE TABLE employee (
    emp_id     NUMBER(5)
        CONSTRAINT employee_pk PRIMARY KEY,
    emp_name   VARCHAR2(30) NOT NULL,
    department VARCHAR2(20),
    salary     NUMBER(8,2)
        CONSTRAINT employee_salary_ck CHECK (salary >= 0)
);
```

Inspect its definition in SQL\*Plus:

```sql
DESCRIBE employee
```

`DESCRIBE` (or `DESC`) is a SQL\*Plus command, not a SQL statement.

## 3. Insert data

Name the destination columns so the statement remains understandable if the table later changes:

```sql
INSERT INTO employee (emp_id, emp_name, department, salary)
VALUES (1001, 'Priya', 'HR', 35000);

INSERT INTO employee (emp_id, emp_name, department, salary)
VALUES (1002, 'Rajat', 'Sales', 25000);

INSERT INTO employee (emp_id, emp_name, department, salary)
VALUES (1003, 'Disha', 'Management', 50000);

INSERT INTO employee (emp_id, emp_name, department, salary)
VALUES (1004, 'Brajesh', 'Development', 55000);
```

The raw transcript's first ID appears once as `101` and later as `1001`; this edition consistently uses `1001`.

Save the inserts:

```sql
COMMIT;
```

## 4. Retrieve data with `SELECT`

Select every column:

```sql
SELECT * FROM employee;
```

Prefer an explicit column list in reusable code:

```sql
SELECT emp_id, emp_name, salary
FROM employee;
```

Expected data:

| EMP_ID | EMP_NAME | DEPARTMENT | SALARY |
|---:|---|---|---:|
| 1001 | Priya | HR | 35000 |
| 1002 | Rajat | Sales | 25000 |
| 1003 | Disha | Management | 50000 |
| 1004 | Brajesh | Development | 55000 |

### Filter with `WHERE`

`WHERE` contains a condition evaluated for each row:

```sql
SELECT emp_id, emp_name, salary
FROM employee
WHERE emp_id = 1001;
```

The correct condition is `column_name = value`, not “value = record of value.” SQL uses one equals sign (`=`), not `==`.

## 5. Change and remove rows

### Update

```sql
UPDATE employee
SET salary = 40000
WHERE emp_id = 1001;
```

### Delete and roll back

```sql
DELETE FROM employee
WHERE emp_id = 1001;

ROLLBACK;
```

Because the delete was not committed, the rollback restores Priya's row. A rollback cannot undo work after it has been committed.

> [!TIP]
> Before an `UPDATE` or `DELETE`, run a `SELECT` with the same `WHERE` clause. This confirms exactly which rows will be affected.

## 6. Useful SQL*Plus commands

Clear the SQL\*Plus screen:

```sql
CLEAR SCREEN
```

List objects owned by the current user using the SQL\*Plus compatibility view from the original notes:

```sql
SELECT * FROM tab;
```

A more explicit data-dictionary query is:

```sql
SELECT object_name, object_type
FROM user_objects
ORDER BY object_type, object_name;
```

## 7. `DELETE`, `TRUNCATE`, and `DROP`

```sql
DELETE FROM employee;      -- Removes rows; can be rolled back before commit.
TRUNCATE TABLE employee;  -- Removes all rows; table remains; implicit commit.
DROP TABLE employee;      -- Removes the table object.
```

| Operation | Rows | Structure | `WHERE` | Normal rollback in Oracle |
|---|---|---|---|---|
| `DELETE` | Selected/all removed | Kept | Allowed | Yes, before commit |
| `TRUNCATE` | All removed | Kept | Not allowed | No |
| `DROP` | Removed with object | Removed | Not allowed | No transaction rollback |

After `DROP TABLE employee`, `DESC employee` raises `ORA-04043: object employee does not exist`. Oracle may place a dropped table in the recycle bin, but that is a recovery feature rather than transaction rollback.

## 8. Alter and rename a table

Create the table from the original lesson:

```sql
CREATE TABLE login (
    user_id  VARCHAR2(20),
    password VARCHAR2(20)
);
```

> [!WARNING]
> A production application must never store plain-text passwords. Store a password hash produced by a suitable password-hashing algorithm, or use a trusted identity provider.

### Add a primary key

```sql
ALTER TABLE login
ADD CONSTRAINT login_pk PRIMARY KEY (user_id);
```

Adding a named constraint is clearer than attaching `PRIMARY KEY` to `MODIFY`. Existing values must already be non-null and unique.

### Add, modify, rename, and drop columns

```sql
ALTER TABLE login ADD user_type VARCHAR2(20);
ALTER TABLE login MODIFY user_type VARCHAR2(30);
ALTER TABLE login RENAME COLUMN password TO password_hash;
ALTER TABLE login DROP COLUMN user_type;
```

### Rename the table

Oracle syntax is:

```sql
RENAME login TO login_info;
```

The form `RENAME TABLE ...` shown in one raw quick-reference example is MySQL-style syntax, not Oracle syntax.

## 9. Primary and foreign keys

Create a parent table and a related child table:

```sql
CREATE TABLE employee (
    emp_id   NUMBER(5) CONSTRAINT employee_pk PRIMARY KEY,
    emp_name VARCHAR2(20) NOT NULL
);

CREATE TABLE product (
    product_id   NUMBER(5) CONSTRAINT product_pk PRIMARY KEY,
    product_name VARCHAR2(20) NOT NULL,
    emp_id       NUMBER(5),
    CONSTRAINT product_employee_fk
        FOREIGN KEY (emp_id)
        REFERENCES employee (emp_id)
);
```

Example data:

```sql
INSERT INTO employee VALUES (1001, 'John');
INSERT INTO employee VALUES (1002, 'Brown');
INSERT INTO employee VALUES (1003, 'Green');
INSERT INTO employee VALUES (1004, 'Lily');

INSERT INTO product VALUES (101, 'Printer',   1001);
INSERT INTO product VALUES (102, 'Scanner',   1002);
INSERT INTO product VALUES (103, 'Plotter',   1002);
INSERT INTO product VALUES (104, 'Laptop',    1003);
INSERT INTO product VALUES (105, 'Projector', NULL);
INSERT INTO product VALUES (106, 'Tablet',    NULL);
```

`NULL` is allowed in the foreign key because the column was not declared `NOT NULL`. A non-null `emp_id` must match an employee.

## 10. Join related tables

### Inner join

```sql
SELECT e.emp_name, p.product_name
FROM employee e
JOIN product p ON p.emp_id = e.emp_id;
```

| EMP_NAME | PRODUCT_NAME |
|---|---|
| John | Printer |
| Brown | Scanner |
| Brown | Plotter |
| Green | Laptop |

The original quick example filtered only `product.emp_id = 1001` without relating the two tables, which produces a Cartesian product: every employee paired with the printer. The `ON p.emp_id = e.emp_id` condition is essential.

### Left outer join

Keep every employee, even one without a product:

```sql
SELECT e.emp_name, p.product_name
FROM employee e
LEFT JOIN product p ON p.emp_id = e.emp_id
ORDER BY e.emp_id, p.product_id;
```

Lily appears with a null product name.

### Right outer join

Keep every product, even one without an employee:

```sql
SELECT e.emp_name, p.product_name
FROM employee e
RIGHT JOIN product p ON p.emp_id = e.emp_id
ORDER BY p.product_id;
```

Projector and Tablet appear with a null employee name. Oracle also supports `FULL OUTER JOIN` when unmatched rows from both sides are required.

## 11. Sorting rows

Without `ORDER BY`, row order is not guaranteed.

```sql
SELECT * FROM employee ORDER BY emp_name;       -- Ascending by default
SELECT * FROM employee ORDER BY salary ASC;
SELECT * FROM employee ORDER BY emp_name DESC;
```

Multiple sort keys are allowed:

```sql
SELECT *
FROM employee
ORDER BY department, salary DESC;
```

## 12. Comparison and filtering operators

| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal | `emp_id = 1001` |
| `<>` or `!=` | Not equal | `department <> 'HR'` |
| `<`, `>` | Less/greater than | `salary > 50000` |
| `<=`, `>=` | Inclusive comparison | `salary >= 65000` |
| `LIKE` | Pattern matching | `emp_name LIKE '%Singh'` |
| `IN` | Member of a list/query | `department IN ('HR', 'Sales')` |
| `BETWEEN` | Inclusive range | `salary BETWEEN 50000 AND 65000` |
| `IS NULL` | Null test | `department IS NULL` |

### `LIKE` wildcards

- `%` matches zero or more characters.
- `_` matches exactly one character.

```sql
WHERE emp_name LIKE '%Singh'  -- Ends with Singh
WHERE emp_name LIKE 'A%'      -- Starts with A
WHERE emp_name LIKE '%a%'     -- Contains lowercase a under a case-sensitive comparison
WHERE emp_name LIKE '___s%'   -- Fourth character is s
```

The raw note describes `'___s%'` as placing `s` in the third position; three underscores mean `s` is actually the fourth character.

## 13. Aggregate functions

Given salaries `50000`, `65000`, `80000`, and `55000`:

```sql
SELECT
    SUM(salary)   AS total_salary,
    MAX(salary)   AS maximum_salary,
    MIN(salary)   AS minimum_salary,
    AVG(salary)   AS average_salary,
    COUNT(salary) AS non_null_salaries,
    COUNT(*)      AS row_count
FROM emp_info;
```

Expected values are `250000`, `80000`, `50000`, `62500`, `4`, and `4`. `COUNT(column)` ignores nulls; `COUNT(*)` counts rows.

Use `DISTINCT` to remove duplicates from a result:

```sql
SELECT DISTINCT department
FROM emp_info;
```

`DISTINCT` is a query keyword, not a function, so it does not require parentheses.

## 14. Subqueries

Find every employee tied for the maximum salary:

```sql
SELECT *
FROM emp_info
WHERE salary = (SELECT MAX(salary) FROM emp_info);
```

The inner query returns one value; the outer query returns the matching rows. If the subquery can return several values, use an appropriate operator such as `IN`, `EXISTS`, `ANY`, or `ALL`.

## 15. Oracle date values

The raw syntax sheet mentions MySQL functions `CURDATE()` and `NOW()`. Oracle equivalents include:

```sql
SELECT SYSDATE, SYSTIMESTAMP FROM dual;

INSERT INTO audit_log (created_at)
VALUES (SYSTIMESTAMP);
```

Use explicit conversion for date text:

```sql
TO_DATE('2026-10-28', 'YYYY-MM-DD')
```

## Practice tasks

1. Add an email column to `employee` and require it to be unique.
2. Increase every Sales salary by 10%, previewing the affected rows first.
3. List all employees and the number of products assigned to each, including employees with zero products.
4. Find the second-highest distinct salary.
5. Delete one uncommitted row, inspect the result, and restore it with `ROLLBACK`.

### Suggested answers

```sql
ALTER TABLE employee ADD email VARCHAR2(100);
ALTER TABLE employee ADD CONSTRAINT employee_email_uk UNIQUE (email);

SELECT * FROM employee WHERE department = 'Sales';
UPDATE employee SET salary = salary * 1.10 WHERE department = 'Sales';

SELECT e.emp_id, e.emp_name, COUNT(p.product_id) AS product_count
FROM employee e
LEFT JOIN product p ON p.emp_id = e.emp_id
GROUP BY e.emp_id, e.emp_name
ORDER BY e.emp_id;

SELECT MAX(salary) AS second_highest_salary
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);

DELETE FROM product WHERE product_id = 106;
SELECT * FROM product;
ROLLBACK;
```

Continue with [views and constraints](VIEW.md) or keep [syntax.md](syntax.md) open as a quick reference.

> Original classroom source: [Connection.txt](Connection.txt)
