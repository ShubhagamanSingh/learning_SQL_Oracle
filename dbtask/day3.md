# Day 3: Sorting, Pattern Matching, and Salary Queries

## Objectives

- list schema objects;
- create and populate an employee table;
- sort ascending and descending;
- search text with `LIKE`;
- find minimum, maximum, and second-highest salaries.

## 1. Inspect existing objects

The original session showed `ORDERS`, `PERSONAL`, and `STUDENT`:

```sql
SELECT * FROM tab;
```

A more descriptive query is:

```sql
SELECT object_name, object_type
FROM user_objects
ORDER BY object_type, object_name;
```

## 2. Create and populate the employee table

```sql
CREATE TABLE spi_emp (
    emp_id     NUMBER(5)
        CONSTRAINT spi_emp_pk PRIMARY KEY,
    emp_name   VARCHAR2(30) NOT NULL,
    department VARCHAR2(20),
    salary     NUMBER(8,2)
        CONSTRAINT spi_emp_salary_ck CHECK (salary >= 0)
);

INSERT INTO spi_emp VALUES (1001, 'Rajat Singh',    'Management',  80000);
INSERT INTO spi_emp VALUES (1002, 'Brijesh Mishra', 'Development', 55000);
INSERT INTO spi_emp VALUES (1003, 'Rohit Singh',    'Development', 55000);
INSERT INTO spi_emp VALUES (1004, 'Anand Singh',    'Development', 50000);
INSERT INTO spi_emp VALUES (1005, 'Disha Verma',    'HR',          60000);
```

## 3. Sort employees

```sql
SELECT * FROM spi_emp ORDER BY emp_name;
SELECT * FROM spi_emp ORDER BY emp_name DESC;
```

Ascending is the default. Without `ORDER BY`, Oracle does not guarantee result order.

## 4. Match a name pattern

Find names ending in `Singh`:

```sql
SELECT *
FROM spi_emp
WHERE emp_name LIKE '%Singh';
```

`%` matches zero or more characters. This returns Rajat Singh, Rohit Singh, and Anand Singh.

## 5. Highest and lowest salaries

```sql
SELECT *
FROM spi_emp
WHERE salary = (SELECT MAX(salary) FROM spi_emp);

SELECT *
FROM spi_emp
WHERE salary = (SELECT MIN(salary) FROM spi_emp);
```

Using equality to the aggregate returns every employee tied at the extreme, not just one arbitrary row.

## 6. Second-highest distinct salary

The original nested query is correct for the second-highest distinct value:

```sql
SELECT *
FROM spi_emp
WHERE salary = (
    SELECT MAX(salary)
    FROM spi_emp
    WHERE salary < (SELECT MAX(salary) FROM spi_emp)
);
```

It returns Disha Verma at 60000. A modern analytic alternative is:

```sql
SELECT emp_id, emp_name, department, salary
FROM (
    SELECT e.*, DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM spi_emp e
)
WHERE salary_rank = 2;
```

`DENSE_RANK` handles ties explicitly.

## Practice

1. List Development employees from highest to lowest salary.
2. Find all employees whose name contains lowercase `i`.
3. Find the third-highest distinct salary using `DENSE_RANK`.
4. Calculate the average salary per department with `GROUP BY`.

Continue to [Day 4](day4.md).

> Original SQL\*Plus transcript: [day3.txt](day3.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
