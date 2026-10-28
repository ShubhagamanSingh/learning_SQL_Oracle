# Fibonacci Series in PL/SQL

> [!NOTE]
> The original filename uses the spelling `fabonicci`; the sequence's standard spelling is **Fibonacci**. The Markdown filename is kept parallel to the source so the one-to-one conversion is easy to find.

## The sequence

The Fibonacci sequence begins with 0 and 1. Every later term is the sum of the previous two:

```text
0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...
```

## Print exactly `n` terms

```sql
SET SERVEROUTPUT ON;

DECLARE
    n PLS_INTEGER := &n;
    a NUMBER := 0;
    b NUMBER := 1;
    next_value NUMBER;
BEGIN
    IF n <= 0 THEN
        DBMS_OUTPUT.PUT_LINE('Enter a positive number of terms.');
    ELSE
        FOR position IN 1..n LOOP
            DBMS_OUTPUT.PUT_LINE(a);
            next_value := a + b;
            a := b;
            b := next_value;
        END LOOP;
    END IF;
END;
/
```

For `n = 9`, the program prints:

```text
0
1
1
2
3
5
8
13
21
```

## Why the original program printed extra terms

The historical version printed `0` and `1` before entering a loop that ran from `1..n`. Therefore it produced `n + 2` terms. With `n = 9`, it printed 11 values through 55. The corrected program performs all printing inside a loop that runs exactly `n` times.

## Trace the first iterations

| Position | Printed `a` | New `next_value` | New `a` | New `b` |
|---:|---:|---:|---:|---:|
| 1 | 0 | 1 | 1 | 1 |
| 2 | 1 | 2 | 1 | 2 |
| 3 | 1 | 3 | 2 | 3 |
| 4 | 2 | 5 | 3 | 5 |

The variables always retain two consecutive terms.

## Edge cases

| Input | Behavior |
|---:|---|
| `n < 0` | Prints a validation message |
| `n = 0` | Prints a validation message |
| `n = 1` | Prints only `0` |
| `n = 2` | Prints `0`, then `1` |

For large `n`, values eventually exceed the precision of `NUMBER`. A production version should define a practical limit or handle the result as text.

## Challenge: comma-separated output

Build one line using a string accumulator, taking care not to add a comma before the first term. This is useful practice for conditionals and string concatenation.

Return to the [PL/SQL lesson index](README.md).

> Original SQL\*Plus transcript: [fabonicci.txt](fabonicci.txt)

---

Copyright © 2021–2026 [Shubhagaman Singh](https://shubhagaman.in). Licensed under [CC BY-NC-SA 4.0](../../LICENSE.md).
