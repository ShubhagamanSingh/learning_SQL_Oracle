# Oracle SQL and PL/SQL Practical Lab

These lessons reconstruct the six classroom sessions from 13–18 December 2021. Each day builds on the previous one, but every page explains its dependencies and includes a clean version of the code.

| Day | Main topics |
|---|---|
| [Day 1](day1.md) | Practice user, customer table, CRUD, commit, rollback |
| [Day 2](day2.md) | Truncate/drop, `ALTER TABLE`, primary and foreign keys |
| [Day 3](day3.md) | Employee data, sorting, `LIKE`, min/max, second-highest salary |
| [Day 4](day4.md) | Business tables, checks, simple views, view inserts and filters |
| [Day 5](day5.md) | PL/SQL variables, inserts, formulas, conditionals, salary and bill slabs |
| [Day 6](day6.md) | Account debit, stored calculations, loops, conditional transactions |
| [Classwork: 18 Dec 2021](CW_18-12-2021.md) | Safer account-withdrawal exercise |
| [Miscellaneous note](new%205.md) | The two greeting lines from the source archive |

## Setup

Use a disposable Oracle schema. The original exercises connected as `mydb/test`; substitute your own learner account. In SQL\*Plus or SQLcl:

```sql
SET SERVEROUTPUT ON;
```

## How to learn from the lab

1. Read the objective and predict each result.
2. Type the clean script rather than copying it.
3. Query the table after every DML statement.
4. Test one valid and one invalid value for each constraint or branch.
5. Use `ROLLBACK` deliberately before trying `COMMIT`.

The `.txt` files beside these lessons preserve the original SQL\*Plus output. The Markdown files correct syntax and logic while retaining the original topics, data, and outcomes.
