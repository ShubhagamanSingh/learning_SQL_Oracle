# PL/SQL Worked Examples

The original SQL\*Plus session demonstrated console output, a calculator, and even/odd checks. This edition separates runnable code, expected output, and edge cases.

Before running the programs:

```sql
SET SERVEROUTPUT ON;
```

## 1. Hello world

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello World');
END;
/
```

Expected output:

```text
Hello World

PL/SQL procedure successfully completed.
```

In the historical transcript, the first run displayed only the success message because `SERVEROUTPUT` had not yet been enabled. `DBMS_OUTPUT` output is buffered; the client decides whether to show it.

## 2. Simple calculator

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Sum = ' || (a + b));
    DBMS_OUTPUT.PUT_LINE('Difference = ' || (a - b));
    DBMS_OUTPUT.PUT_LINE('Product = ' || (a * b));

    IF b = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Quotient = undefined (division by zero)');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Quotient = ' || (a / b));
    END IF;
END;
/
```

With `a = 10` and `b = 5`:

```text
Sum = 15
Difference = 5
Product = 50
Quotient = 2
```

### Why parentheses are used

`||` concatenates values into a string. Parentheses make arithmetic happen before concatenation:

```sql
'Sum = ' || (a + b)
```

### Test cases

| `a` | `b` | Important result |
|---:|---:|---|
| 10 | 5 | Quotient is 2 |
| 5 | 10 | Quotient is 0.5 |
| -3 | 2 | Negative results are handled |
| 4 | 0 | Friendly zero-division message |

## 3. Even or odd

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

Expected examples:

```text
Input: 3  -> 3 is odd.
Input: 4  -> 4 is even.
Input: 0  -> 0 is even.
Input: -5 -> -5 is odd.
```

The SQL operator may also be written as `n MOD 2` in PL/SQL, as in the original notes; `MOD(n, 2)` is often easier to recognize as a function.

## 4. Safer interactive text input

SQL\*Plus substitutes text before compilation. Put quotes around a character substitution:

```sql
DECLARE
    learner_name VARCHAR2(50) := '&learner_name';
BEGIN
    DBMS_OUTPUT.PUT_LINE('Welcome, ' || learner_name || '!');
END;
/
```

For reusable application code, prefer bind variables or parameters. Blind textual substitution can break on quotes and must never be built from untrusted input in a real application.

## 5. Calculator with exception handling

The explicit zero check is best when zero is an expected input. The same Oracle exception can also be handled:

```sql
DECLARE
    a NUMBER := &a;
    b NUMBER := &b;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Quotient = ' || (a / b));
EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Cannot divide by zero.');
END;
/
```

This illustrates exception syntax, but validation before the operation usually communicates intent more clearly.

## Challenges

1. Add remainder output to the calculator with `MOD(a, b)` when `b` is nonzero.
2. Classify a number as positive, negative, or zero with `IF / ELSIF / ELSE`.
3. Change the even/odd program to print all even numbers from 1 through a supplied limit.
4. Replace SQL\*Plus substitution with a procedure parameter.

### One possible answer to challenge 2

```sql
DECLARE
    n NUMBER := &n;
BEGIN
    IF n > 0 THEN
        DBMS_OUTPUT.PUT_LINE('Positive');
    ELSIF n < 0 THEN
        DBMS_OUTPUT.PUT_LINE('Negative');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Zero');
    END IF;
END;
/
```

Return to [PL/SQL fundamentals](PLSQL.md) or continue with [Fibonacci](fabonicci.md).

> Original SQL\*Plus transcript: [PLSQL_CODE.txt](PLSQL_CODE.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../../LICENSE.md).
