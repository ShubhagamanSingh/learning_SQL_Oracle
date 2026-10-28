# PL/SQL Fundamentals

## What is PL/SQL?

**PL/SQL** stands for **Procedural Language extensions to SQL**. Oracle designed it to combine SQL data operations with procedural features such as variables, conditions, loops, exceptions, procedures, functions, packages, and triggers.

PL/SQL normally runs inside Oracle Database. This makes its source largely independent of the client operating system, but it remains Oracle-specific rather than universally portable SQL.

## Block structure

```sql
DECLARE
    -- Optional: variables, constants, types, cursors, local subprograms
BEGIN
    -- Required: executable PL/SQL and SQL statements
EXCEPTION
    -- Optional: exception handlers
END;
/
```

Only the executable `BEGIN ... END` section is required. Every statement ends with a semicolon. In SQL\*Plus, SQLcl, and many worksheet modes, `/` on a new line submits the completed block.

Enable console output:

```sql
SET SERVEROUTPUT ON;
```

## Hello world

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello World');
END;
/
```

`DBMS_OUTPUT.PUT_LINE` writes text to the session's output buffer. If server output is disabled, the block still succeeds but the client does not display the line.

## Variables and SQL*Plus substitution

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
BEGIN
    DBMS_OUTPUT.PUT_LINE('a = ' || a);
    DBMS_OUTPUT.PUT_LINE('b = ' || b);
END;
/
```

`&a` and `&b` are replaced by SQL\*Plus before compilation. They are convenient for classroom scripts but are not PL/SQL's input mechanism. Application code should use bind variables or stored-program parameters.

For text substitution, quoting is required in the script:

```sql
DECLARE
    learner_name VARCHAR2(50) := '&learner_name';
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello, ' || learner_name);
END;
/
```

## Simple calculator

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Sum = ' || (a + b));
    DBMS_OUTPUT.PUT_LINE('Difference = ' || (a - b));
    DBMS_OUTPUT.PUT_LINE('Product = ' || (a * b));

    IF b = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Quotient is undefined because b is zero.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Quotient = ' || (a / b));
    END IF;
END;
/
```

For `a = 10` and `b = 5`, the outputs are `15`, `5`, `50`, and `2`. The zero check improves the original example by preventing `ZERO_DIVIDE`.

## Conditional statements

### `IF`

```sql
IF condition THEN
    statements;
END IF;
```

### `IF ... ELSE`

```sql
IF condition THEN
    statements;
ELSE
    statements;
END IF;
```

### `IF ... ELSIF ... ELSE`

```sql
IF condition_1 THEN
    statements;
ELSIF condition_2 THEN
    statements;
ELSE
    statements;
END IF;
```

The PL/SQL keyword is `ELSIF`.

### Even or odd

```sql
DECLARE
    n PLS_INTEGER := &n;
BEGIN
    IF MOD(n, 2) = 0 THEN
        DBMS_OUTPUT.PUT_LINE(n || ' is even.');
    ELSE
        DBMS_OUTPUT.PUT_LINE(n || ' is odd.');
    END IF;
END;
/
```

`MOD(n, 2)` returns the remainder after division by 2. A zero remainder means the integer is even.

## Loop control

PL/SQL provides a basic loop, a `WHILE` loop, and a numeric `FOR` loop.

### Basic loop

A basic loop repeats until an `EXIT` or exception ends it.

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

This prints 1 through 5. Ensure that every basic loop has a reachable exit condition.

### `WHILE` loop

A `WHILE` loop checks its condition before every iteration:

```sql
WHILE condition LOOP
    statements;
END LOOP;
```

Correct multiplication-table program:

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

The original note used `WHILE n <= 10` while changing only `i`. For `n <= 10` that condition never became false, so the program ran indefinitely. The loop counter `i` must be tested.

### Numeric `FOR` loop

Use a `FOR` loop when the number of iterations is known:

```sql
BEGIN
    FOR i IN 1..10 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
END;
/
```

PL/SQL declares `i` automatically for the loop. Reverse iteration is also available:

```sql
BEGIN
    FOR i IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
END;
/
```

## SQL inside PL/SQL

PL/SQL can read and change table data:

```sql
DECLARE
    employee_salary employee.salary%TYPE;
BEGIN
    SELECT salary
    INTO employee_salary
    FROM employee
    WHERE emp_id = 1001;

    DBMS_OUTPUT.PUT_LINE('Salary = ' || employee_salary);
END;
/
```

`SELECT ... INTO` must return exactly one row. No rows raises `NO_DATA_FOUND`; multiple rows raise `TOO_MANY_ROWS`. `%TYPE` keeps the variable compatible with the table column.

## Basic exception handling

```sql
DECLARE
    employee_salary employee.salary%TYPE;
BEGIN
    SELECT salary
    INTO employee_salary
    FROM employee
    WHERE emp_id = &emp_id;

    DBMS_OUTPUT.PUT_LINE('Salary = ' || employee_salary);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No employee has that ID.');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('The lookup returned more than one row.');
END;
/
```

Handle exceptions you understand and can respond to. Avoid silently swallowing all errors with `WHEN OTHERS THEN NULL`.

## Exercises

1. Print the numbers 10 through 1 using `REVERSE`.
2. Extend the calculator to print a friendly message for division by zero.
3. Print only the even numbers from 1 through 20.
4. Add exception handling to a single-row employee lookup.
5. Compare the three loop forms and state when each is clearest.

Continue with [worked PL/SQL examples](PLSQL_CODE.md) and the [Fibonacci program](fabonicci.md).

> Original classroom source: [PLSQL.txt](PLSQL.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../../LICENSE.md).
