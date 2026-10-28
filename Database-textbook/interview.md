# DBMS, SQL, and PL/SQL Interview Questions

These answers expand and correct the questions in the original notes. Use them as revision prompts, not as sentences to memorize.

## 1. What is a DBMS?

A database management system is software that defines, stores, retrieves, changes, secures, and administers data. A relational DBMS such as Oracle Database represents related data mainly with tables and enforces rules using keys and constraints.

## 2. Who are database users?

Common groups are end users, application developers, data analysts, database designers, and database administrators. Each interacts with the data at a different level of abstraction.

## 3. What is a DBA?

A database administrator manages installation, configuration, accounts, privileges, security, backups, recovery, availability, monitoring, and performance. The exact responsibilities depend on the organization.

## 4. What is SQL?

SQL stands for Structured Query Language. It is a declarative language used to define relational structures, query data, change rows, control privileges, and manage transactions.

## 5. What is the difference between `ALTER` and `UPDATE`?

`ALTER` is DDL and changes an object's structure—for example, adding a table column. `UPDATE` is DML and changes values stored in existing rows.

```sql
ALTER TABLE employee ADD email VARCHAR2(100);

UPDATE employee
SET email = 'ravi@example.com'
WHERE employee_id = 1001;
```

## 6. How do `DELETE`, `TRUNCATE`, and `DROP` differ?

| Statement | Removes | `WHERE` allowed? | Table remains? | Normal rollback in Oracle? |
|---|---|---:|---:|---:|
| `DELETE` | Selected or all rows | Yes | Yes | Yes, before commit |
| `TRUNCATE` | All rows | No | Yes | No; it implicitly commits |
| `DROP` | The database object | No | No | No normal transaction rollback |

Triggers, storage behavior, recycle-bin behavior, and privileges also differ.

## 7. What is a primary key?

A primary key is the chosen column or set of columns that uniquely identifies each row. Its values are unique and never `NULL`. A table has at most one primary-key constraint, though that key may contain multiple columns.

## 8. What is a foreign key?

A foreign key is a column or set of columns whose non-null values must match a referenced primary or unique key, usually in another table. It enforces referential integrity. A table may contain several foreign keys.

## 9. What can `ALTER TABLE` do?

It can add, modify, rename, or drop columns; add or remove constraints; rename constraints; enable or disable constraints; and perform other Oracle-specific structural changes.

```sql
ALTER TABLE student ADD email VARCHAR2(100);
ALTER TABLE student MODIFY email VARCHAR2(150);
ALTER TABLE student RENAME COLUMN email TO email_address;
ALTER TABLE student DROP COLUMN email_address;
```

## 10. What is a join?

A join combines related rows from two or more row sources. An inner join keeps matches; outer joins can also keep unmatched rows from the left, right, or both sides.

```sql
SELECT e.employee_name, d.department_name
FROM employee e
JOIN department d ON d.department_id = e.department_id;
```

## 11. What is a nested query?

A nested query, or subquery, is a query used inside another SQL statement. The inner query supplies a value or row set to the outer statement.

```sql
SELECT *
FROM employee
WHERE salary = (SELECT MAX(salary) FROM employee);
```

## 12. What is a view, and why is it useful?

A view is a stored query presented like a table. It can simplify complex queries, expose only selected rows or columns, provide a stable interface, and support privilege separation. A regular view stores the query definition, not a separate copy of its rows. Some simple views are updatable; aggregated, joined, or otherwise complex views may not be.

## 13. What is a `CHECK` constraint?

A `CHECK` constraint requires every affected row to satisfy a Boolean condition. In Oracle, a row passes if the condition is `TRUE` or `UNKNOWN`, so use `NOT NULL` separately when nulls must be rejected.

```sql
salary NUMBER(8,2) CONSTRAINT employee_salary_ck CHECK (salary >= 0)
```

## 14. What is a `NOT NULL` constraint?

It requires a column to contain a value in every row. It does not make the value unique.

## 15. What is data integrity?

Data integrity is the accuracy, validity, and consistency of data throughout its lifecycle. Constraints, transactions, suitable data types, access control, and application rules all contribute to it.

## 16. What are the main relational integrity rules?

- **Entity integrity:** every table should have a key that uniquely identifies a row; primary-key values cannot be null.
- **Referential integrity:** each non-null foreign-key value must reference an existing candidate key value, unless the relationship's configured delete behavior applies.
- **Domain integrity:** values must belong to the permitted domain, enforced with data types, nullability, and checks.

“Atomicity” belongs to the ACID transaction properties; it is not one of these two classic relational integrity rules.

## 17. What are composite, candidate, and super keys?

- A **super key** is any set of attributes that uniquely identifies a row.
- A **candidate key** is a minimal super key—removing any attribute would destroy uniqueness.
- A **composite key** contains more than one column.
- The **primary key** is the candidate key selected as the table's main identifier.

A composite key can be a candidate key, but the terms are not synonyms.

## 18. What is PL/SQL?

PL/SQL is Oracle's procedural extension to SQL. It adds variables, branching, loops, cursors, exceptions, procedures, functions, packages, and triggers while allowing SQL statements inside program blocks.

## 19. What are important PL/SQL features?

Block structure, strong typing, procedural control flow, exception handling, stored program units, modular packages, cursor processing, transaction control, and tight integration with Oracle SQL. Running logic near the data can also reduce application-to-database round trips.

## 20. What is the syntax of a PL/SQL block?

```sql
DECLARE
    -- Optional declarations
BEGIN
    -- Required executable section
EXCEPTION
    -- Optional exception handlers
END;
/
```

The final `/` is a client command used by tools such as SQL\*Plus to submit the buffered block; it is not part of PL/SQL itself.

## 21. Is PL/SQL operating-system independent?

PL/SQL source normally runs inside Oracle Database, so it is largely independent of the client operating system. It is Oracle-specific, however, and external libraries, file paths, host commands, and database-version features can reduce portability.

## 22. What is the difference between `COMMIT` and `ROLLBACK`?

`COMMIT` permanently ends the current transaction and makes its changes visible to other sessions. `ROLLBACK` ends the transaction and undoes its uncommitted changes; `ROLLBACK TO` can return to a savepoint without ending all work.

## 23. How do a primary key and a unique key differ?

Both enforce uniqueness. A table has only one primary-key constraint, and all of its columns are implicitly `NOT NULL`. A table can have multiple unique constraints. Oracle permits multiple nulls in a single-column unique constraint because nulls are not treated as equal.

## 24. What is a `NOT NULL` constraint?

It prevents null values in one column. This repeated question is worth contrasting with `CHECK`: a check such as `salary >= 0` does not reject `NULL` by itself.

## 25. How do you create a foreign key?

```sql
CREATE TABLE department (
    department_id NUMBER(5)
        CONSTRAINT department_pk PRIMARY KEY,
    name VARCHAR2(50) NOT NULL
);

CREATE TABLE employee (
    employee_id   NUMBER(5)
        CONSTRAINT employee_pk PRIMARY KEY,
    employee_name VARCHAR2(50) NOT NULL,
    department_id NUMBER(5),
    CONSTRAINT employee_department_fk
        FOREIGN KEY (department_id)
        REFERENCES department (department_id)
);
```

The referenced parent column set must be protected by a primary-key or unique constraint, and parent rows must normally be inserted before child rows.

## Quick practice prompts

1. Why can a `CHECK (salary >= 0)` column still accept `NULL`?
2. When would a left outer join be more useful than an inner join?
3. Why is a composite primary key not automatically the same thing as every candidate key?
4. Which parts of a PL/SQL block are optional?

See [Database.md](Database.md), [Connection.md](Connection.md), and [PLSQL/PLSQL.md](PLSQL/PLSQL.md) for the longer lessons.

> Original question list: [interview.txt](interview.txt)
