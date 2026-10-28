# PL/SQL Lessons

PL/SQL is Oracle's procedural extension to SQL. These lessons begin with block structure and control flow, then apply those ideas in small programs.

| Lesson | Topics |
|---|---|
| [PLSQL.md](PLSQL.md) | Blocks, variables, substitution variables, conditions, and three loop forms |
| [PLSQL_CODE.md](PLSQL_CODE.md) | Hello world, calculator, even/odd, input safety, and test cases |
| [fabonicci.md](fabonicci.md) | Correct Fibonacci program for exactly `n` terms |

## Before running an example

In SQL\*Plus or SQLcl, enable printed output:

```sql
SET SERVEROUTPUT ON;
```

The `/` after a PL/SQL block tells the client to execute the buffered block. It is not part of PL/SQL grammar.

Values such as `&n` are SQL\*Plus substitution variables. The client replaces them before Oracle compiles the PL/SQL block.

## Suggested progression

1. Run the hello-world block.
2. Change a declared variable and print it.
3. Trace each branch of the even/odd program.
4. Run every loop by hand for its first two iterations.
5. Test edge cases such as zero, a negative value, and division by zero.
6. Add an `EXCEPTION` section to a program.
