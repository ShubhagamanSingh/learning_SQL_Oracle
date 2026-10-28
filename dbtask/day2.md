# Day 2: DDL, Table Changes, and Foreign Keys

## Objectives

- compare `TRUNCATE` and `DROP`;
- inspect a table with `DESC`;
- add, modify, and rename columns;
- add a primary key;
- relate parent and child tables with a foreign key.

## 1. Remove the Day 1 table

The original session ran both statements in sequence:

```sql
TRUNCATE TABLE customer;
DROP TABLE customer;
```

`TRUNCATE` removes every row but leaves the table. `DROP` then removes the empty table object. Both are DDL and cause implicit commits in Oracle.

## 2. Create and alter a student table

Initial definition:

```sql
CREATE TABLE student (
    roll_no       NUMBER(5),
    student_name  VARCHAR2(50),
    contact_no    VARCHAR2(12),
    email_address VARCHAR2(50)
);

DESC student
```

The historical table initially called the name column `NAME`, then renamed it. This edition starts with the final clear name; the equivalent alteration sequence is:

```sql
ALTER TABLE student
ADD CONSTRAINT student_pk PRIMARY KEY (roll_no);

ALTER TABLE student ADD password VARCHAR2(20);

-- Historical rename, if the column was originally NAME:
ALTER TABLE student RENAME COLUMN name TO student_name;

ALTER TABLE student MODIFY contact_no VARCHAR2(11);
```

> [!WARNING]
> A real application should not store a plain password in this table. Store a password hash created by a suitable password-hashing algorithm or use an external identity system.

After the changes, `ROLL_NO` is non-null because it is the primary key, and the table contains `STUDENT_NAME`, `CONTACT_NO`, `EMAIL_ADDRESS`, and `PASSWORD`.

## 3. Create a parent and child table

The original `PERSONAL` table is the parent:

```sql
CREATE TABLE personal (
    person_id NUMBER(5)
        CONSTRAINT personal_pk PRIMARY KEY,
    name VARCHAR2(30) NOT NULL
);
```

`ORDERS` is the child:

```sql
CREATE TABLE orders (
    order_id   NUMBER(5)
        CONSTRAINT orders_pk PRIMARY KEY,
    order_name VARCHAR2(20),
    person_id  NUMBER(5),
    CONSTRAINT orders_person_fk
        FOREIGN KEY (person_id)
        REFERENCES personal (person_id)
);
```

The foreign key means each non-null `ORDERS.PERSON_ID` must already exist in `PERSONAL.PERSON_ID`.

## 4. Test referential integrity

```sql
INSERT INTO personal (person_id, name)
VALUES (1, 'Ravi');

INSERT INTO orders (order_id, order_name, person_id)
VALUES (101, 'Laptop', 1);
```

This statement fails because person 99 does not exist:

```sql
INSERT INTO orders (order_id, order_name, person_id)
VALUES (102, 'Printer', 99);
```

To make the relationship mandatory, declare `person_id NOT NULL`. To define parent-delete behavior, consider `ON DELETE CASCADE` or `ON DELETE SET NULL` only when it matches the business rule.

## Practice

1. Add `created_at TIMESTAMP DEFAULT SYSTIMESTAMP` to `ORDERS`.
2. Rename `ORDER_NAME` to `DESCRIPTION`.
3. Try deleting person 1 while the order references that row.
4. Explain why child rows are normally inserted after parent rows.

Continue to [Day 3](day3.md).

> Original SQL\*Plus transcript: [day2.txt](day2.txt)
