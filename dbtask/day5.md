# Day 5: PL/SQL Variables, Formulas, and Conditions

## Objectives

- declare and print variables;
- insert rows from a PL/SQL block;
- calculate circle measurements;
- find the largest of three values;
- calculate gross salary using slabs;
- calculate an electricity bill using progressive slabs.

Enable output first:

```sql
SET SERVEROUTPUT ON;
```

## 1. Create the staff table

```sql
CREATE TABLE staff (
    staff_id   NUMBER(5) CONSTRAINT staff_pk PRIMARY KEY,
    staff_name VARCHAR2(30) NOT NULL,
    department VARCHAR2(20),
    salary     NUMBER(8,2) CHECK (salary >= 0)
);
```

## 2. Read and print values

The original session prompted for ID, name, department, and salary, then printed them:

```sql
DECLARE
    v_id         staff.staff_id%TYPE := &staff_id;
    v_name       staff.staff_name%TYPE := '&staff_name';
    v_department staff.department%TYPE := '&department';
    v_salary     staff.salary%TYPE := &salary;
BEGIN
    DBMS_OUTPUT.PUT_LINE('ID = ' || v_id);
    DBMS_OUTPUT.PUT_LINE('Name = ' || v_name);
    DBMS_OUTPUT.PUT_LINE('Department = ' || v_department);
    DBMS_OUTPUT.PUT_LINE('Salary = ' || v_salary);
END;
/
```

The historical transcript entered quotes at the prompt for text values. Placing quotes around `&staff_name` and `&department` in the script lets the learner enter plain text instead.

## 3. Insert a staff row from PL/SQL

```sql
DECLARE
    v_id         staff.staff_id%TYPE := &staff_id;
    v_name       staff.staff_name%TYPE := '&staff_name';
    v_department staff.department%TYPE := '&department';
    v_salary     staff.salary%TYPE := &salary;
BEGIN
    INSERT INTO staff (staff_id, staff_name, department, salary)
    VALUES (v_id, v_name, v_department, v_salary);
END;
/
```

The original example inserted `(1001, 'Shubh', 'Manager', 80000)`. The block does not commit, so the caller can inspect the row and choose `COMMIT` or `ROLLBACK`.

## 4. Circle perimeter and area

```sql
DECLARE
    radius NUMBER := &radius;
    pi CONSTANT NUMBER := ACOS(-1);
BEGIN
    IF radius < 0 THEN
        DBMS_OUTPUT.PUT_LINE('Radius cannot be negative.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Perimeter = ' || (2 * pi * radius));
        DBMS_OUTPUT.PUT_LINE('Area = ' || (pi * radius * radius));
    END IF;
END;
/
```

The original used `3.14`, producing:

| Radius | Perimeter | Area |
|---:|---:|---:|
| 2 | 12.56 | 12.56 |
| 5 | 31.4 | 78.5 |

`ACOS(-1)` gives a more precise value of pi while retaining the same formula.

## 5. Largest of three numbers

The raw notes contained two attempts. The first missed an `END IF` for the nested branch; the second covered only strict orderings such as `a > b > c` and printed nothing for ties or some valid orders. This complete solution handles every order and ties:

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
    c NUMBER := &c;
    largest NUMBER;
BEGIN
    largest := GREATEST(a, b, c);
    DBMS_OUTPUT.PUT_LINE('Largest number is: ' || largest);
END;
/
```

An `IF` version for control-flow practice:

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
    c NUMBER := &c;
BEGIN
    IF a >= b AND a >= c THEN
        DBMS_OUTPUT.PUT_LINE('Largest number is: ' || a);
    ELSIF b >= a AND b >= c THEN
        DBMS_OUTPUT.PUT_LINE('Largest number is: ' || b);
    ELSE
        DBMS_OUTPUT.PUT_LINE('Largest number is: ' || c);
    END IF;
END;
/
```

For inputs `3`, `2`, and `9`, the result is 9, matching the original session.

## 6. Gross salary using slabs

The source calculates house-rent allowance (HRA), dearness allowance (DA), and gross salary from basic salary.

| Basic salary | HRA rate | DA rate |
|---:|---:|---:|
| 1–4000 | 10% | 50% |
| 4000–8000 | 20% | 60% |
| 8000–12000 | 25% | 70% |
| Above 12000 | 30% | 80% |

The boundaries overlap in that prose, so the code uses clear non-overlapping intervals: `<= 4000`, `<= 8000`, `<= 12000`, and above 12000.

```sql
DECLARE
    basic_salary NUMBER := &basic_salary;
    hra          NUMBER;
    da           NUMBER;
    gross_salary NUMBER;
BEGIN
    IF basic_salary <= 0 THEN
        DBMS_OUTPUT.PUT_LINE('Basic salary must be positive.');
        RETURN;
    ELSIF basic_salary <= 4000 THEN
        hra := basic_salary * 0.10;
        da  := basic_salary * 0.50;
    ELSIF basic_salary <= 8000 THEN
        hra := basic_salary * 0.20;
        da  := basic_salary * 0.60;
    ELSIF basic_salary <= 12000 THEN
        hra := basic_salary * 0.25;
        da  := basic_salary * 0.70;
    ELSE
        hra := basic_salary * 0.30;
        da  := basic_salary * 0.80;
    END IF;

    gross_salary := basic_salary + hra + da;
    DBMS_OUTPUT.PUT_LINE('HRA = ' || hra);
    DBMS_OUTPUT.PUT_LINE('DA = ' || da);
    DBMS_OUTPUT.PUT_LINE('Gross salary = ' || gross_salary);
END;
/
```

For a basic salary of 15000: HRA is 4500, DA is 12000, and gross salary is 31500, matching the original output.

## 7. Progressive electricity bill

The intended rules are:

- first 150 units at 2.40 per unit;
- next 150 units (151–300) at 3.00 per unit;
- units above 300 at 3.20 per unit.

The original third `ELSIF` mistakenly repeated `unit <= 150 AND unit >= 1`, making the above-300 branch unreachable. The corrected program is:

```sql
DECLARE
    units_used NUMBER := &units_used;
    bill       NUMBER;
BEGIN
    IF units_used < 0 THEN
        DBMS_OUTPUT.PUT_LINE('Units cannot be negative.');
        RETURN;
    ELSIF units_used <= 150 THEN
        bill := units_used * 2.40;
    ELSIF units_used <= 300 THEN
        bill := (150 * 2.40)
              + ((units_used - 150) * 3.00);
    ELSE
        bill := (150 * 2.40)
              + (150 * 3.00)
              + ((units_used - 300) * 3.20);
    END IF;

    DBMS_OUTPUT.PUT_LINE('Total bill to pay = ' || TO_CHAR(bill, 'FM9999990.00'));
END;
/
```

For 200 units, the bill is `150 × 2.40 + 50 × 3.00 = 510`, matching the session.

### Boundary tests

| Units | Calculation | Bill |
|---:|---|---:|
| 0 | `0 × 2.40` | 0.00 |
| 150 | `150 × 2.40` | 360.00 |
| 151 | `360 + 1 × 3.00` | 363.00 |
| 300 | `360 + 150 × 3.00` | 810.00 |
| 301 | `810 + 1 × 3.20` | 813.20 |

## Practice

1. Turn the gross-salary program into a function that returns the total.
2. Store electricity slabs in a table rather than hard-coding them.
3. Print whether the largest-number input contains a tie.
4. Add exception handling for invalid numeric substitution values.

Continue to [Day 6](day6.md).

> Original SQL\*Plus transcript: [day5.txt](day5.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../LICENSE.md).
