# How to SELECT with optional parameters

This TIL only exist because of my sub par SQL skills. But I'm hoping that some recent experience with SQL will change that and this TIL was a penny dropping moment that really felt good.

## The problem

On at least four occations I've had a scenario where there's been a query with one or more mandatory parameters and also some optional parameters.

Let's say there's a table of `Employees` that look like this:

Id | Name | Role | Department
--- | --- | --- | ---
101 | Alice | Dev | STHLM
102 | Bob | Test | STHLM
103 | Carol | Dev | NYC

And in this example `Department` is mandatory but the user might also want to filter on `Role`.

## How I used to do it

```sql
-- Create a temp table
DECLARE @Emps TABLE(
  Name NVARCHAR(MAX),
  Role NVARCHAR(MAX),
  Department NVARCHAR(MAX)
);

INSERT INTO @Emps
SELECT
  Name,
  Role,
  Department
FROM
  Employees;

IF NOT @Role IS NULL DELETE FROM @Emps WHERE Role <> @Role;

SELECT * FROM @Emps;
```

## How I'm doing after enlightment :bulb:

```sql
SELECT
  Name,
  Role,
  Department
FROM
  Employees
WHERE
  Department = @Department
  AND Role = COALESCE(@Role, Role);
```
