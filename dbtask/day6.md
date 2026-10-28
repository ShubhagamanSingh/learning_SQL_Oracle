# Day 6: Account Transactions, Loops, and Conditional Commit

## Objectives

- validate an account debit;
- insert calculated circle areas;
- reverse an integer with a loop;
- update salaries and choose `COMMIT` or `ROLLBACK` from a business rule.

Before running the blocks:

```sql
SET SERVEROUTPUT ON;
```

## 1. Account table and sample data

The session first created `ACCOUNTS` without a check, dropped it, and recreated it with an `AC%` rule. The final definition is:

```sql
CREATE TABLE accounts (
    account_id VARCHAR2(10)
        CONSTRAINT accounts_pk PRIMARY KEY
        CONSTRAINT accounts_id_ck CHECK (account_id LIKE 'AC%'),
    account_name VARCHAR2(20) NOT NULL,
    balance NUMBER(10,2)
        CONSTRAINT accounts_balance_ck CHECK (balance >= 0)
);

INSERT INTO accounts VALUES ('AC001', 'Ajay',    5000);
INSERT INTO accounts VALUES ('AC002', 'Anand',  10000);
INSERT INTO accounts VALUES ('AC003', 'Yashi',   5000);
INSERT INTO accounts VALUES ('AC004', 'Brijesh',15000);
INSERT INTO accounts VALUES ('AC005', 'Rohit',  10000);
COMMIT;
```

## 2. Debit 3000 while retaining 1000

```sql
DECLARE
    requested_account accounts.account_id%TYPE := '&account_id';
    debit_amount      CONSTANT NUMBER := 3000;
    minimum_balance   CONSTANT NUMBER := 1000;
    current_balance   accounts.balance%TYPE;
BEGIN
    SELECT balance
    INTO current_balance
    FROM accounts
    WHERE account_id = requested_account
    FOR UPDATE;

    IF current_balance - debit_amount >= minimum_balance THEN
        UPDATE accounts
        SET balance = balance - debit_amount
        WHERE account_id = requested_account;

        COMMIT;
        DBMS_OUTPUT.PUT_LINE('Debited successfully.');
    ELSE
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Insufficient balance.');
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Account not found.');
END;
/
```

### Understanding the historical AC003 output

AC003 started at 5000. The first run calculated 2000, which satisfies the 1000 minimum, so it updated and committed. No success text appeared because `SERVEROUTPUT` was enabled only later. The second run tested another 3000 debit against 2000, rejected it, and printed “Insufficient Balance.” The final balance therefore remained 2000.

## 3. Calculate and store circle areas

```sql
CREATE TABLE areas (
    radius NUMBER(10,2) CONSTRAINT areas_radius_ck CHECK (radius >= 0),
    area   NUMBER(14,2) NOT NULL
);
```

Insert one calculation:

```sql
DECLARE
    radius_value NUMBER := &radius;
    area_value   NUMBER;
BEGIN
    IF radius_value < 0 THEN
        DBMS_OUTPUT.PUT_LINE('Radius cannot be negative.');
        RETURN;
    END IF;

    area_value := ACOS(-1) * radius_value * radius_value;

    INSERT INTO areas (radius, area)
    VALUES (radius_value, ROUND(area_value, 2));

    DBMS_OUTPUT.PUT_LINE('Area is: ' || ROUND(area_value, 2));
END;
/
```

The original program used pi = 3.14 and inserted radii 3 through 7:

| Radius | Stored area |
|---:|---:|
| 3 | 28.26 |
| 4 | 50.24 |
| 5 | 78.50 |
| 6 | 113.04 |
| 7 | 153.86 |

The `/` SQL\*Plus command reran the buffered block and prompted for a new radius each time.

## 4. Reverse an integer

```sql
DECLARE
    input_number PLS_INTEGER := &n;
    remaining    PLS_INTEGER;
    reversed     PLS_INTEGER := 0;
    digit        PLS_INTEGER;
    sign_value   PLS_INTEGER;
BEGIN
    sign_value := CASE WHEN input_number < 0 THEN -1 ELSE 1 END;
    remaining := ABS(input_number);

    WHILE remaining > 0 LOOP
        digit := MOD(remaining, 10);
        reversed := (reversed * 10) + digit;
        remaining := TRUNC(remaining / 10);
    END LOOP;

    reversed := reversed * sign_value;
    DBMS_OUTPUT.PUT_LINE('Reverse is: ' || reversed);
END;
/
```

For `123`, the result is `321`. The explicit `TRUNC` makes removal of the final digit unambiguous. Leading zeroes disappear because the result is numeric: reversing `120` produces `21`.

## 5. Employee-expense table

```sql
CREATE TABLE expense_employee (
    emp_no VARCHAR2(10)
        CONSTRAINT expense_employee_pk PRIMARY KEY
        CONSTRAINT expense_employee_no_ck CHECK (emp_no LIKE 'E%'),
    emp_name VARCHAR2(20) NOT NULL,
    salary NUMBER(8,2) CHECK (salary >= 0)
);

INSERT INTO expense_employee VALUES ('E001', 'Harry', 5000);
INSERT INTO expense_employee VALUES ('E002', 'Black', 1000);
INSERT INTO expense_employee VALUES ('E003', 'Jack',  5000);
INSERT INTO expense_employee VALUES ('E004', 'Clark', 1000);
COMMIT;
```

The initial total is 12000.

## 6. Conditional salary update

The exercise adds 2000 to E002 and 1500 to E004, then keeps the changes only if total salary remains below 20000.

```sql
DECLARE
    total_salary expense_employee.salary%TYPE;
BEGIN
    SAVEPOINT before_salary_update;

    UPDATE expense_employee
    SET salary = salary +
        CASE emp_no
            WHEN 'E002' THEN 2000
            WHEN 'E004' THEN 1500
        END
    WHERE emp_no IN ('E002', 'E004');

    SELECT SUM(salary)
    INTO total_salary
    FROM expense_employee;

    IF total_salary < 20000 THEN
        COMMIT;
        DBMS_OUTPUT.PUT_LINE(
            'Updated successfully. Total salary = ' || total_salary
        );
    ELSE
        ROLLBACK TO before_salary_update;
        COMMIT;
        DBMS_OUTPUT.PUT_LINE(
            'Update rejected. Proposed total = ' || total_salary
        );
    END IF;
END;
/
```

### Why repeated runs change the result

This update is cumulative. From the original data:

| Run | Proposed total | Decision | Persisted E002 / E004 after run |
|---:|---:|---|---:|
| 1 | 15500 | Commit | 3000 / 2500 |
| 2 | 19000 | Commit | 5000 / 4000 |
| 3 | 22500 | Roll back | 5000 / 4000 |

That explains the historical final values of 5000 and 4000 together with a later rollback message: two increases had already committed, and the next attempted increase exceeded the threshold.

`SAVEPOINT` limits the rollback to this block's proposed updates. In real application design, transaction boundaries are often controlled by the caller rather than hidden inside a reusable procedure.

## Review questions

1. Why does the debit query use `FOR UPDATE`?
2. Why should a rejected transaction explicitly roll back?
3. What happens when the reverse-number input ends in zero?
4. Why does the salary block eventually roll back when rerun?
5. How would you make the salary operation idempotent rather than cumulative?

### Short answers

1. It locks the selected row and prevents concurrent debits from using the same starting balance.
2. It releases locks and clearly discards pending changes.
3. Numeric output loses leading zeroes, so `120` becomes `21`.
4. Each successful run increases the stored salaries; the proposed total eventually reaches or exceeds 20000.
5. Store the intended final values or record a unique adjustment transaction and refuse to apply it twice.

See the related [18 December classwork](CW_18-12-2021.md).

> Original SQL\*Plus transcript: [day6.txt](day6.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
