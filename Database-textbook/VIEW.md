# Oracle Views, Constraints, Composite Keys, and CTAS

This lesson expands the original view exercise and the related examples for validation, multi-column keys, and copying query results into a new table.

## 1. What is a view?

A regular view is a named, stored `SELECT` statement that can be queried like a table. It does not normally store a separate copy of the result rows; Oracle evaluates its query against the underlying objects.

Views are useful for:

- exposing only selected rows or columns;
- hiding query complexity behind a stable interface;
- presenting domain-friendly names or derived values;
- granting access to a view instead of the complete base table.

A view is not automatically a security boundary: privileges, owner rights, predicates, and the application's behavior must all be designed correctly.

## 2. Create the base table

```sql
CREATE TABLE emp_info (
    emp_id     NUMBER(5)
        CONSTRAINT emp_info_pk PRIMARY KEY,
    emp_name   VARCHAR2(20) NOT NULL,
    department VARCHAR2(20),
    salary     NUMBER(8,2)
        CONSTRAINT emp_info_salary_ck CHECK (salary >= 0)
);
```

In SQL\*Plus:

```sql
DESCRIBE emp_info
```

## 3. Create and inspect a view

The original exercise exposes `EMP_ID`, `EMP_NAME`, and `DEPARTMENT` while omitting salary:

```sql
CREATE VIEW emp AS
SELECT emp_id, emp_name, department
FROM emp_info;

DESCRIBE emp
```

Query it normally:

```sql
SELECT * FROM emp;
```

## 4. Insert through a simple view

```sql
INSERT INTO emp (emp_id, emp_name, department)
VALUES (1001, 'Shubh Singh', 'Development');

INSERT INTO emp (emp_id, emp_name, department)
VALUES (1002, 'Shikha Mishra', 'HR');

INSERT INTO emp (emp_id, emp_name, department)
VALUES (1003, 'Rajat Singh', 'Management');

COMMIT;
```

This simple single-table view is inherently updatable. The rows are inserted into `EMP_INFO`; the omitted `SALARY` column becomes `NULL` because it permits nulls and has no default.

| EMP_ID | EMP_NAME | DEPARTMENT | SALARY in base table |
|---:|---|---|---:|
| 1001 | Shubh Singh | Development | `NULL` |
| 1002 | Shikha Mishra | HR | `NULL` |
| 1003 | Rajat Singh | Management | `NULL` |

Update salary through the base table because salary is not exposed by the view:

```sql
UPDATE emp_info SET salary = 50000 WHERE emp_id = 1001;
UPDATE emp_info SET salary = 45000 WHERE emp_id = 1002;
UPDATE emp_info SET salary = 70000 WHERE emp_id = 1003;
COMMIT;
```

Drop only the view:

```sql
DROP VIEW emp;
```

The `EMP_INFO` table and its data remain.

### When is a view not directly updatable?

Operations such as grouping, aggregate functions, `DISTINCT`, set operators, and many joins can make a view non-updatable or restrict which columns may be changed. `WITH READ ONLY` deliberately prevents DML:

```sql
CREATE VIEW emp_public AS
SELECT emp_id, emp_name, department
FROM emp_info
WITH READ ONLY;
```

`WITH CHECK OPTION` ensures that inserted or updated rows continue to satisfy the view's filter.

## 5. `CHECK` constraints

A check constraint enforces a condition whenever relevant rows are inserted or updated.

Requirements from the original exercise:

- employee IDs start with `SPI`;
- names are uppercase;
- country is India or Nepal;
- salary does not exceed 100000.

```sql
CREATE TABLE spi_emp (
    emp_id VARCHAR2(10)
        CONSTRAINT spi_emp_pk PRIMARY KEY
        CONSTRAINT spi_emp_id_ck CHECK (emp_id LIKE 'SPI%'),
    emp_name VARCHAR2(30)
        CONSTRAINT spi_emp_name_nn NOT NULL
        CONSTRAINT spi_emp_name_ck CHECK (emp_name = UPPER(emp_name)),
    country VARCHAR2(5)
        CONSTRAINT spi_emp_country_nn NOT NULL
        CONSTRAINT spi_emp_country_ck CHECK (country IN ('India', 'Nepal')),
    salary NUMBER(6)
        CONSTRAINT spi_emp_salary_ck CHECK (salary BETWEEN 0 AND 100000)
);
```

Naming constraints makes error messages understandable. The raw transcript used system names such as `SYS_C007007`, which do not reveal the failed rule.

### Validation examples

These statements fail for different reasons:

```sql
-- ID does not start with SPI.
INSERT INTO spi_emp VALUES ('1001', 'BRIJESH', 'India', 50000);

-- Name is not uppercase.
INSERT INTO spi_emp VALUES ('SPI1001', 'Brijesh', 'India', 50000);

-- Country is outside the allowed set.
INSERT INTO spi_emp VALUES ('SPI1001', 'BRIJESH', 'China', 50000);

-- Salary is above the limit (and exceeds NUMBER(6) at 999999 only if within precision;
-- the CHECK is still the business rule to communicate).
INSERT INTO spi_emp VALUES ('SPI1001', 'BRIJESH', 'India', 999999);
```

This row satisfies every constraint:

```sql
INSERT INTO spi_emp (emp_id, emp_name, country, salary)
VALUES ('SPI1001', 'BRIJESH', 'India', 50000);
```

> [!NOTE]
> In Oracle a check passes when its condition is `TRUE` or `UNKNOWN`; a null input usually makes the condition unknown. Add `NOT NULL` when null must be forbidden, as shown for name and country.

## 6. Composite primary key

A shipment is identified by both supplier and part. Either value can appear in many rows, but the pair cannot repeat.

```sql
CREATE TABLE shipment (
    supplier_no VARCHAR2(5),
    part_no     VARCHAR2(5),
    quantity    NUMBER(5)
        CONSTRAINT shipment_quantity_ck CHECK (quantity >= 0),
    CONSTRAINT shipment_pk PRIMARY KEY (supplier_no, part_no)
);
```

Example data:

| SUPPLIER_NO | PART_NO | Description | QUANTITY |
|---|---|---|---:|
| S1 | P1 | Nut | 500 |
| S1 | P2 | Bolt | 1000 |
| S1 | P3 | Other part | 5000 |
| S2 | P1 | Nut | 200 |
| S3 | P2 | Bolt | 500 |

The primary key is **composite** because it contains two columns. If `(supplier_no, part_no)` is a minimal unique identifier, it is also a **candidate key**. “Composite” describes the number of columns; “candidate” describes minimal uniqueness.

Avoid special characters such as `S#` and `P#` in new schemas. Oracle permits `$`, `#`, and `_` in unquoted identifiers, but descriptive names are clearer and more portable.

## 7. Create a table from an existing query

The original lesson copies selected columns from `EMP_INFO`:

```sql
CREATE TABLE emp_info_2 AS
SELECT emp_id, emp_name, salary
FROM emp_info;
```

This technique is called **Create Table As Select (CTAS)**. It copies the result rows and derives compatible column definitions.

CTAS does **not** automatically reproduce all source metadata. Review and add the needed primary keys, foreign keys, checks, defaults, indexes, comments, triggers, and grants:

```sql
ALTER TABLE emp_info_2
ADD CONSTRAINT emp_info_2_pk PRIMARY KEY (emp_id);
```

Create only the structure, with no rows:

```sql
CREATE TABLE emp_info_empty AS
SELECT emp_id, emp_name, salary
FROM emp_info
WHERE 1 = 0;
```

## Practice

1. Create a view that exposes only employees in HR and protects the filter with `WITH CHECK OPTION`.
2. Add a rule that permits only positive shipment quantities.
3. Try to insert the same supplier/part pair twice and explain the error.
4. Use CTAS to copy employees earning at least 50000, then inspect which constraints were copied.

### Suggested answers

```sql
CREATE VIEW hr_emp AS
SELECT emp_id, emp_name, department
FROM emp_info
WHERE department = 'HR'
WITH CHECK OPTION;

-- Already included in the lesson's SHIPMENT definition:
-- CHECK (quantity >= 0)

INSERT INTO shipment VALUES ('S1', 'P1', 500);
INSERT INTO shipment VALUES ('S1', 'P1', 900); -- ORA-00001: duplicate key

CREATE TABLE well_paid_emp AS
SELECT * FROM emp_info WHERE salary >= 50000;
```

Continue with [PL/SQL fundamentals](PLSQL/PLSQL.md).

> Original classroom source: [VIEW.txt](VIEW.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
