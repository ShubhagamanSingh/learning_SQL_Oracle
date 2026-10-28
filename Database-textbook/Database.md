# Database Management Systems and SQL Fundamentals

## What is a DBMS?

A **database management system (DBMS)** is software that lets people and applications define, store, retrieve, update, protect, and administer data. Oracle Database is a relational DBMS: it organizes related data mainly as tables made of rows and columns.

A database is the organized data; the DBMS is the software used to manage it.

## Database users

### End users

End users work with data through applications, websites, reports, or forms. They do not need to know how the database is physically stored or how its full schema is designed.

### Application programmers

Programmers build software that reads and changes data. They write SQL directly or use libraries, frameworks, stored procedures, and APIs.

### Database administrators (DBAs)

A DBA is responsible for tasks such as installation, configuration, user and privilege management, backup and recovery, availability, monitoring, performance tuning, and database security.

## What is SQL?

**SQL** stands for **Structured Query Language**. It is the standard language for defining relational database structures and working with their data. SQL statements are commonly grouped by purpose.

| Family | Meaning | Purpose | Common Oracle commands |
|---|---|---|---|
| DDL | Data Definition Language | Define or change database objects | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` |
| DML | Data Manipulation Language | Add or change table rows | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| Query | Data Query Language (often grouped with DML) | Retrieve rows | `SELECT` |
| DCL | Data Control Language | Manage privileges | `GRANT`, `REVOKE` |
| TCL | Transaction Control Language | End or undo transactions | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

> [!NOTE]
> `BACKUP` and `RESTORE` are database-administration concepts, but they are not portable SQL DDL commands. Oracle backup and recovery are normally performed with tools such as Recovery Manager (RMAN), Data Pump, and database-specific recovery operations.

## DDL: defining objects

DDL operates on structures such as tables, views, sequences, and indexes.

```sql
CREATE TABLE student (
    student_id NUMBER(5) PRIMARY KEY,
    name       VARCHAR2(50) NOT NULL
);

ALTER TABLE student ADD email VARCHAR2(100);
RENAME student TO students;
TRUNCATE TABLE students;
DROP TABLE students;
```

- `CREATE` creates an object.
- `ALTER` changes an existing object's definition.
- `RENAME` changes an object's name.
- `TRUNCATE` quickly removes all rows while keeping the table structure. In Oracle it performs an implicit commit and cannot be rolled back normally.
- `DROP` removes the object itself. In Oracle, dropped tables normally go to the recycle bin unless `PURGE` is used or the feature is disabled.

## DML and queries: working with rows

```sql
INSERT INTO student (student_id, name)
VALUES (1001, 'Ravi');

UPDATE student
SET name = 'Ravi Singh'
WHERE student_id = 1001;

SELECT student_id, name
FROM student
WHERE student_id = 1001;

DELETE FROM student
WHERE student_id = 1001;
```

An omitted `WHERE` clause makes `UPDATE` or `DELETE` affect every row, so verify the condition with a `SELECT` first.

## DCL: privileges

`GRANT` gives a privilege or role; `REVOKE` removes it.

```sql
GRANT CREATE SESSION, CREATE TABLE TO learner;
REVOKE CREATE TABLE FROM learner;
```

`RENAME` belongs to DDL, not DCL. Privilege changes are normally performed by a DBA or an authorized account.

## TCL: transactions

A transaction is a logical unit of work containing one or more related changes.

```sql
UPDATE accounts
SET balance = balance - 500
WHERE account_id = 'AC001';

SAVEPOINT after_debit;

-- Keep all changes made in the transaction.
COMMIT;
```

- `COMMIT` makes the transaction's changes permanent and visible to other sessions.
- `ROLLBACK` undoes uncommitted changes.
- `ROLLBACK TO savepoint_name` undoes only the work performed after a savepoint.

After a `COMMIT`, a normal `ROLLBACK` cannot undo the committed work.

## Knowledge check

1. What is the difference between a database and a DBMS?
2. Which SQL family changes the definition of a table?
3. Why should an `UPDATE` normally include a `WHERE` clause?
4. Can Oracle normally roll back a `TRUNCATE TABLE` statement?
5. Which commands manage object and system privileges?

### Answers

1. A database is the organized data; a DBMS is the software that manages it.
2. DDL.
3. Without it, every row is updated.
4. No. `TRUNCATE` implicitly commits in Oracle.
5. `GRANT` and `REVOKE`.

## Next lesson

Continue with the guided [Oracle SQL\*Plus workbook](Connection.md).

> Original classroom source: [Database.txt](Database.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
