# Day 1: User Setup, Customer Table, and Transactions

**Original session:** 13 December 2021, Oracle Database 11g Express Edition

## Objectives

- connect with SQL\*Plus;
- create and populate a table;
- select all or selected columns;
- filter rows;
- understand `COMMIT` and `ROLLBACK`;
- update and delete rows safely.

## 1. Learner account

The original administrative setup was:

```sql
CREATE USER mydb IDENTIFIED BY test;
GRANT DBA TO mydb;
CONNECT mydb/test
```

For a modern learning environment, grant only the necessary permissions instead of `DBA`:

```sql
CREATE USER mydb IDENTIFIED BY "choose-a-strong-password";
GRANT CREATE SESSION, CREATE TABLE TO mydb;
ALTER USER mydb QUOTA 50M ON users;
```

Reconnect as the learner before creating course objects.

## 2. Create the customer table

```sql
CREATE TABLE customer (
    customer_id   NUMBER(5)
        CONSTRAINT customer_pk PRIMARY KEY,
    customer_name VARCHAR2(30) NOT NULL,
    contact_no    VARCHAR2(15),
    email_address VARCHAR2(100)
        CONSTRAINT customer_email_uk UNIQUE
);
```

Telephone numbers use `VARCHAR2`, not `NUMBER`, because they are identifiers rather than values for arithmetic and can contain a leading zero or country prefix.

## 3. Insert the original sample data

```sql
INSERT INTO customer VALUES (1001, 'Ravi',  '9453318798', 'ravi@gmail.com');
INSERT INTO customer VALUES (1002, 'Priya', '9936652039', 'priya@gmail.com');
INSERT INTO customer VALUES (1003, 'Rohit', '7753001621', 'rohit@gmail.com');
INSERT INTO customer VALUES (1004, 'Disha', '7080102006', 'disha@gmail.com');
```

For reusable scripts, explicitly list columns:

```sql
INSERT INTO customer (customer_id, customer_name, contact_no, email_address)
VALUES (1001, 'Ravi', '9453318798', 'ravi@gmail.com');
```

## 4. Query the table

```sql
SELECT * FROM customer;

SELECT customer_id, customer_name, email_address
FROM customer;

SELECT customer_id, customer_name, email_address
FROM customer
WHERE customer_id = 1004;
```

Expected data:

| CUSTOMER_ID | CUSTOMER_NAME | CONTACT_NO | EMAIL_ADDRESS |
|---:|---|---|---|
| 1001 | Ravi | 9453318798 | ravi@gmail.com |
| 1002 | Priya | 9936652039 | priya@gmail.com |
| 1003 | Rohit | 7753001621 | rohit@gmail.com |
| 1004 | Disha | 7080102006 | disha@gmail.com |

## 5. Commit, delete, and roll back

The session first saved all inserts:

```sql
COMMIT;
```

It then deleted Disha's row and restored it:

```sql
DELETE FROM customer
WHERE customer_id = 1004;

ROLLBACK;
```

Because the delete occurred after the commit and had not itself been committed, rollback restored the row. A committed delete would not be undone by a normal rollback.

## 6. Update a contact number

The original session changed Rohit's number:

```sql
UPDATE customer
SET contact_no = '7080102008'
WHERE customer_id = 1003;
```

Preview the target before updating:

```sql
SELECT * FROM customer WHERE customer_id = 1003;
```

Choose whether to keep or discard the update:

```sql
COMMIT;
-- or ROLLBACK;
```

## Practice

1. Add a fifth customer using an explicit column list.
2. Change Priya's email, verify it, and roll it back.
3. Try to insert a duplicate customer ID and explain the error.
4. Query only customers whose names begin with `R`.

Continue to [Day 2](day2.md).

> Original SQL\*Plus transcript: [day1.txt](day1.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
