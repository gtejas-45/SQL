# SQL Quick Revision README --- Lectures 1--10

**Database:** MySQL\
**Goal:** Revise SQL fundamentals, practise common operations, then
continue to Lecture 11 (Aggregate Functions).

## 1. SQL Fundamentals

-   **SQL** (Structured Query Language) is used to create, read, update,
    delete, and analyze relational data.
-   **Database:** organized collection of data.
-   **Table:** data organized in rows and columns.
-   **Row/record:** one entry. **Column/field:** one attribute.
-   **DBMS:** software to manage databases. **RDBMS:** stores data in
    related tables.
-   **SQL vs MySQL:** SQL is a language; MySQL is an RDBMS that uses
    SQL.

  Group   Purpose            Commands
  ------- ------------------ ---------------------------------------
  DDL     Define structure   `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
  DML     Change data        `INSERT`, `UPDATE`, `DELETE`
  DQL     Retrieve data      `SELECT`
  DCL     Permissions        `GRANT`, `REVOKE`
  TCL     Transactions       `COMMIT`, `ROLLBACK`

Some textbooks classify `SELECT` as DML instead of DQL.

## 2. Create and inspect databases/tables

``` sql
CREATE DATABASE IF NOT EXISTS college_db;
USE college_db;
SHOW DATABASES;
SHOW TABLES;
DESCRIBE students; -- or DESC students
SHOW CREATE TABLE students;
```

`IF NOT EXISTS` avoids a creation error; it does not reset an existing
database.

## 3. Common data types

-   `INT`, `BIGINT`: whole numbers
-   `DECIMAL(10,2)`: exact decimal values, commonly money
-   `FLOAT`, `DOUBLE`: approximate decimal values
-   `CHAR(n)`: fixed-length text; `VARCHAR(n)`: variable-length text;
    `TEXT`: longer text
-   `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`: date/time data
-   `BOOLEAN`: MySQL treats it as `TINYINT(1)` (typically 0/1)

## 4. Create and alter tables

``` sql
CREATE TABLE students (
    student_id INT,
    name VARCHAR(50),
    department VARCHAR(30),
    marks INT
);

ALTER TABLE students ADD email VARCHAR(100);
ALTER TABLE students MODIFY name VARCHAR(80);
ALTER TABLE students RENAME COLUMN email TO student_email;
ALTER TABLE students DROP COLUMN student_email;
RENAME TABLE students TO college_students;
```

-   `DELETE FROM table WHERE condition` removes selected rows; without
    `WHERE`, all rows are deleted.
-   `TRUNCATE TABLE table` removes all rows; no `WHERE` clause.
-   `DROP TABLE table` removes the table structure and its data.
-   In MySQL, `TRUNCATE` and many DDL statements cause implicit commits
    and generally cannot be rolled back like a normal transactional
    `DELETE`.

## 5. Insert data

``` sql
INSERT INTO students (student_id, name, department, marks)
VALUES (101, 'Rahul', 'Computer', 85);

INSERT INTO students (student_id, name, department, marks)
VALUES
(102, 'Priya', 'IT', 92),
(103, 'Amit', 'Computer', 76),
(104, 'Sneha', 'IT', 88),
(105, 'Vijay', 'Mechanical', 67);

INSERT INTO students (student_id, name, department, marks)
VALUES (106, 'Karan', 'Computer', NULL);
```

Best practice: specify column names. Text uses quotes, dates commonly
use `'YYYY-MM-DD'`. `NULL` means missing/unknown; it is not the same as
zero, an empty string, or the text `'NULL'`.

## 6. Retrieve data: SELECT, DISTINCT, aliases

``` sql
SELECT * FROM students;
SELECT name, marks FROM students;
SELECT DISTINCT department FROM students;
SELECT name AS student_name, marks AS exam_marks FROM students;
SELECT name, marks + 5 AS updated_marks FROM students;
```

-   `*` selects all columns.
-   `DISTINCT` removes duplicates from the query result, not from the
    stored table.
-   `AS` creates a temporary result-column alias.
-   `marks + 5` calculates a result; it does not update stored marks.

## 7. Filter rows: WHERE and operators

``` sql
SELECT * FROM students WHERE marks >= 80;
SELECT * FROM students WHERE department = 'IT';
SELECT * FROM students WHERE marks >= 80 AND department = 'IT';
SELECT * FROM students WHERE department = 'IT' OR department = 'Computer';
SELECT * FROM students WHERE NOT department = 'IT';
SELECT * FROM students
WHERE (department = 'IT' OR department = 'Computer') AND marks >= 80;
```

Comparison operators: `=`, `!=`, `<>`, `>`, `<`, `>=`, `<=`.\
`AND` requires all conditions to be true; `OR` requires at least one;
`NOT` reverses a condition. Use parentheses for mixed conditions. `AND`
is evaluated before `OR` when parentheses are absent.

## 8. Sorting and limiting

``` sql
SELECT * FROM students ORDER BY marks ASC;
SELECT * FROM students ORDER BY marks DESC;
SELECT * FROM students ORDER BY department ASC, marks DESC;
SELECT * FROM students ORDER BY marks DESC LIMIT 3;
SELECT * FROM students ORDER BY marks DESC LIMIT 2 OFFSET 1;
```

-   `ASC` is ascending (default); `DESC` is descending.
-   `LIMIT n` returns at most n rows; `OFFSET k` skips k rows first.
-   Use `ORDER BY` for predictable ordering; add a tie-breaker when
    values can tie.
-   Second row after sorting is not necessarily the second-highest
    **distinct** value.

## 9. LIKE, IN, BETWEEN

``` sql
SELECT * FROM students WHERE name LIKE 'S%';
SELECT * FROM students WHERE name LIKE '%a';
SELECT * FROM students WHERE name LIKE '_i%';
SELECT * FROM students WHERE department IN ('IT', 'Computer');
SELECT * FROM students WHERE department NOT IN ('IT', 'Computer');
SELECT * FROM students WHERE marks BETWEEN 70 AND 90;
```

-   `%` matches zero or more characters; `_` matches exactly one.
-   `LIKE 'S%'`: starts with S; `LIKE '%a'`: ends with a; `LIKE '_i%'`:
    second character is i.
-   `IN` matches any listed value.
-   `BETWEEN` includes both endpoints.
-   `LIKE` case sensitivity depends on collation. Be careful with
    `NOT IN` lists containing `NULL`.

## 10. NULL handling

``` sql
SELECT * FROM students WHERE marks IS NULL;
SELECT * FROM students WHERE marks IS NOT NULL;
SELECT name, COALESCE(marks, 0) AS marks_display FROM students;
SELECT name, IFNULL(marks, 0) AS marks_display FROM students;
```

-   Use `IS NULL`, not `= NULL`.
-   `COALESCE(a,b,c)` returns the first non-NULL expression.
-   MySQL `IFNULL(value, replacement)` substitutes a replacement when
    value is NULL.
-   These functions change the query output, not stored data.
-   Do not automatically treat missing values as zero; they can mean
    different things.
-   `COUNT(*)` counts rows; `COUNT(column)` ignores NULL values.

## 11. Update and delete safely

``` sql
UPDATE students SET marks = 90 WHERE student_id = 101;
UPDATE students
SET department = 'Computer', marks = 82
WHERE student_id = 103;
UPDATE students SET marks = marks + 5 WHERE department = 'IT';
DELETE FROM students WHERE student_id = 106;
```

Before an update/delete, run a `SELECT` with the same `WHERE` condition
to check affected rows. Without `WHERE`, every row can be changed or
deleted.

``` sql
START TRANSACTION;
UPDATE students SET marks = 90 WHERE student_id = 101;
-- If wrong:
ROLLBACK;
-- If correct, use COMMIT instead of ROLLBACK:
-- COMMIT;
```

Rollback is useful only before committing and does not undo every kind
of operation.

## 12. Constraints and keys

  -----------------------------------------------------------------------
  Constraint                          Meaning
  ----------------------------------- -----------------------------------
  `PRIMARY KEY`                       Uniquely identifies each row;
                                      cannot be NULL

  `FOREIGN KEY`                       Links a value to a key in another
                                      table

  `UNIQUE`                            Prevents duplicates; MySQL
                                      generally permits multiple NULLs

  `NOT NULL`                          Disallows NULL, but not an empty
                                      string

  `CHECK`                             Requires a condition to be true
                                      (enforced in MySQL 8.0.16+)

  `DEFAULT`                           Supplies a value when a column is
                                      omitted or DEFAULT is specified
  -----------------------------------------------------------------------

``` sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    salary DECIMAL(10,2) DEFAULT 25000.00 CHECK (salary >= 0),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

ALTER TABLE students
ADD CONSTRAINT chk_marks CHECK (marks BETWEEN 0 AND 100);
```

A table has one primary-key constraint, but it may contain multiple
columns (composite key). A table can have multiple `UNIQUE` constraints.
Existing data must satisfy a new constraint. A foreign-key column can be
NULL unless also `NOT NULL`. Explicit NULL normally does not trigger a
default.

------------------------------------------------------------------------

# Sample Database for Practice

This creates a separate database, `sql_revision_lab`. It does not drop
an existing database.

``` sql
CREATE DATABASE IF NOT EXISTS sql_revision_lab;
USE sql_revision_lab;

CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    department VARCHAR(30),
    marks INT NULL,
    city VARCHAR(40) DEFAULT 'Omerga',
    email VARCHAR(100) UNIQUE,
    CHECK (marks IS NULL OR marks BETWEEN 0 AND 100)
);

INSERT INTO students (student_id, name, department, marks, city, email)
VALUES
(101, 'Rahul', 'Computer', 85, 'Pune', 'rahul@example.com'),
(102, 'Priya', 'IT', 92, 'Nashik', 'priya@example.com'),
(103, 'Amit', 'Computer', 76, 'Pune', 'amit@example.com'),
(104, 'Sneha', 'IT', 88, 'Mumbai', 'sneha@example.com'),
(105, 'Vijay', 'Mechanical', 67, 'Omerga', 'vijay@example.com'),
(106, 'Karan', 'Computer', NULL, 'Pune', 'karan@example.com'),
(107, 'Neha', 'IT', 92, 'Mumbai', 'neha@example.com'),
(108, 'Om', 'Mechanical', 54, 'Nashik', NULL);

SELECT * FROM students;
```

Expected initial rows: **8**. Run the `INSERT` block once; running it
again will cause primary-key/unique errors rather than adding duplicate
IDs/emails.

## Practice questions --- try first

1.  Display every column and row.
2.  Display only name and marks.
3.  Show unique departments.
4.  Find students with marks greater than 80.
5.  Find students from Pune.
6.  Find IT or Computer students.
7.  Find marks between 70 and 90 inclusive.
8.  Find names starting with S.
9.  Find students whose marks are missing.
10. Display marks with NULL shown as 0, without changing stored data.
11. Sort students by marks from highest to lowest.
12. Display the top three students by marks.
13. Show the second row after sorting by marks descending.
14. Sort by department ascending, then marks descending.
15. Find students with marks \>= 80, highest first.
16. Add a `phone VARCHAR(15)` column.
17. Update Rahul's marks to 90.
18. Add 3 marks to all IT students.
19. Delete student 108.
20. Describe the table and inspect its constraints.

## Answer key

``` sql
-- 1
SELECT * FROM students;
-- 2
SELECT name, marks FROM students;
-- 3
SELECT DISTINCT department FROM students;
-- 4
SELECT * FROM students WHERE marks > 80;
-- 5
SELECT * FROM students WHERE city = 'Pune';
-- 6
SELECT * FROM students WHERE department IN ('IT', 'Computer');
-- 7
SELECT * FROM students WHERE marks BETWEEN 70 AND 90;
-- 8
SELECT * FROM students WHERE name LIKE 'S%';
-- 9
SELECT * FROM students WHERE marks IS NULL;
-- 10
SELECT name, COALESCE(marks, 0) AS marks_display FROM students;
-- 11
SELECT * FROM students ORDER BY marks DESC, student_id ASC;
-- 12
SELECT * FROM students ORDER BY marks DESC, student_id ASC LIMIT 3;
-- 13
SELECT * FROM students ORDER BY marks DESC, student_id ASC LIMIT 1 OFFSET 1;
-- 14
SELECT * FROM students ORDER BY department ASC, marks DESC;
-- 15
SELECT * FROM students WHERE marks >= 80 ORDER BY marks DESC;
-- 16
ALTER TABLE students ADD phone VARCHAR(15);
-- 17
UPDATE students SET marks = 90 WHERE student_id = 101;
-- 18
UPDATE students SET marks = marks + 3 WHERE department = 'IT';
-- 19
DELETE FROM students WHERE student_id = 108;
-- 20
DESCRIBE students;
SHOW CREATE TABLE students;
```

Questions 17--19 change the sample data. Rebuild only if you do not need
to keep any data in this practice database, or restore the rows
manually.

------------------------------------------------------------------------

# Interview questions and short answers

1.  **What is SQL?** A language for defining, retrieving, and
    manipulating relational data.
2.  **SQL vs MySQL?** SQL is a language; MySQL is a database system that
    uses it.
3.  **DBMS vs RDBMS?** RDBMS organizes data into related tables and
    supports relational constraints.
4.  **Row vs column?** A row is a record; a column is an attribute.
5.  **WHERE vs ORDER BY?** WHERE filters; ORDER BY sorts.
6.  **DISTINCT vs primary key?** DISTINCT removes duplicates from query
    output; a primary key enforces uniqueness in the table.
7.  **DELETE vs TRUNCATE vs DROP?** DELETE removes rows (can filter);
    TRUNCATE removes all rows; DROP removes the table.
8.  **What is NULL?** Missing/unknown/not-applicable data; not zero or
    empty text.
9.  **Why IS NULL instead of = NULL?** NULL is checked with special
    predicates; `= NULL` does not evaluate to true.
10. **CHAR vs VARCHAR?** Fixed-length vs variable-length text.
11. **WHERE vs HAVING?** WHERE filters rows before grouping; HAVING
    filters groups after aggregation (later lecture).
12. **Primary key?** Unique row identifier that cannot be NULL.
13. **Foreign key?** Constraint connecting a column to a referenced key.
14. **UNIQUE vs PRIMARY KEY?** One primary-key constraint per table, no
    NULL key values; multiple unique constraints are allowed and MySQL
    generally allows multiple NULLs in a unique column.
15. **BETWEEN?** Tests an inclusive range.
16. **LIKE wildcards?** `%` means zero or more characters; `_` means
    exactly one.
17. **COALESCE?** Returns the first non-NULL expression.
18. **Why WHERE in UPDATE/DELETE?** It limits which rows are affected.
19. **DEFAULT?** Value used when a column is omitted or DEFAULT is
    requested.
20. **CHECK?** Constraint requiring a condition to hold.

## Common mistakes

-   Text needs quotes: `name = 'Rahul'`.
-   Use `IS NULL`, not `= NULL`.
-   Forgetting `WHERE` can affect every row.
-   `DISTINCT` does not delete stored duplicates.
-   `BETWEEN` includes both endpoints.
-   `LIMIT` without `ORDER BY` does not guarantee a meaningful order.
-   Replacing NULL with 0 may misrepresent the data.
-   `IF NOT EXISTS` does not reset existing tables.
-   Check existing rows before adding constraints.

## Revision checklist

-   [ ] Create/select a database and inspect tables.
-   [ ] Understand common data types.
-   [ ] Create/alter tables and insert rows.
-   [ ] Use SELECT, DISTINCT, and aliases.
-   [ ] Filter with WHERE, comparisons, AND/OR/NOT.
-   [ ] Sort and paginate with ORDER BY, LIMIT, OFFSET.
-   [ ] Use LIKE, IN, BETWEEN.
-   [ ] Handle NULL with IS NULL, IS NOT NULL, COALESCE.
-   [ ] Safely UPDATE and DELETE.
-   [ ] Explain primary/foreign keys and UNIQUE, NOT NULL, CHECK,
    DEFAULT.
-   [ ] Explain DELETE vs TRUNCATE vs DROP.

## Next: Lecture 11 --- Aggregate Functions

Next: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()`---for example,
student count, total marks, average marks, highest marks, and lowest
marks. Lecture 12 will cover `GROUP BY`.
