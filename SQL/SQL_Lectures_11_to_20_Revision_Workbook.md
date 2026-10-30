# SQL Revision Workbook --- Lectures 11--20

**Database:** MySQL \| **Level:** Beginner revision + moderate practice\
**Instructions:** Read the topics and examples first. Attempt Sections
12 and 13 without looking ahead. The solutions are provided afterwards.
Use the setup script once in a fresh database.

## Contents

1.  Practice database setup
2.  Lecture notes and simple examples (11--20)
3.  Moderate practice questions (questions only)
4.  Real-world retail sales case (questions only)
5.  Lecture-wise solutions
6.  Real-world case solutions
7.  Quick revision notes

------------------------------------------------------------------------

# 1. Practice database setup

``` sql
CREATE DATABASE IF NOT EXISTS sql_l11_l20_revision;
USE sql_l11_l20_revision;

CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(40) NOT NULL
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(60),
    department_id INT,
    salary DECIMAL(10,2),
    hire_date DATE,
    city VARCHAR(40),
    manager_id INT,
    email VARCHAR(100)
);

CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    emp_id INT,
    product_name VARCHAR(60),
    quantity INT,
    unit_price DECIMAL(10,2),
    sale_date DATE,
    customer_name VARCHAR(60),
    city VARCHAR(40)
);

INSERT INTO departments VALUES
(1,'IT'),(2,'HR'),(3,'Finance'),(4,'Sales'),(5,'Marketing');

INSERT INTO employees VALUES
(101,'Rahul Sharma',1,65000,'2022-01-15','Pune',NULL,'rahul@company.com'),
(102,'Priya Patil',2,52000,'2021-06-10','Mumbai',101,'priya@company.com'),
(103,'Amit Shah',1,72000,'2020-03-20','Pune',101,'amit@company.com'),
(104,'Sneha Joshi',3,58000,'2023-08-05','Nashik',103,'sneha@company.com'),
(105,'Vijay More',4,45000,'2022-11-12','Pune',103,'vijay@company.com'),
(106,'Neha Kulkarni',2,61000,'2019-04-18','Mumbai',102,'neha@company.com'),
(107,'Karan Deshmukh',4,48000,'2024-02-01','Nashik',105,'karan@company.com'),
(108,'Om Jadhav',NULL,39000,'2024-05-14','Omerga',NULL,'om@company.com');

INSERT INTO sales VALUES
(1,101,'Laptop',2,55000,'2025-01-05','Aarav Traders','Pune'),
(2,102,'Mouse',5,500,'2025-01-10','Bright Stores','Mumbai'),
(3,103,'Laptop',1,55000,'2025-02-12','Aarav Traders','Pune'),
(4,104,'Keyboard',4,1200,'2025-02-18','City Mart','Nashik'),
(5,105,'Monitor',2,12000,'2025-03-02','Bright Stores','Pune'),
(6,101,'Mouse',10,500,'2025-03-15','City Mart','Pune'),
(7,106,'Laptop',1,55000,'2025-04-01','Aarav Traders','Mumbai'),
(8,107,'Keyboard',3,1200,'2025-04-11','City Mart','Nashik'),
(9,103,'Monitor',1,12000,'2025-05-08','Bright Stores','Pune'),
(10,105,'Mouse',8,500,'2025-05-20','Aarav Traders','Pune'),
(11,102,'Laptop',1,55000,'2025-06-04','City Mart','Mumbai'),
(12,104,'Monitor',2,12000,'2025-06-17','Bright Stores','Nashik');
```

Check the data:

``` sql
SELECT * FROM departments;
SELECT * FROM employees;
SELECT * FROM sales;
```

**Relationships:** `employees.department_id` relates to
`departments.department_id`; `sales.emp_id` relates to
`employees.emp_id`. Department 5 is intentionally empty, and employee
108 has no department.

------------------------------------------------------------------------

# 2. Lecture notes and simple examples (11--20)

## Lecture 11 --- Aggregate Functions

Aggregate functions calculate a result from multiple rows.

  Function    Meaning
  ----------- --------------------------------
  `COUNT()`   Counts rows or non-NULL values
  `SUM()`     Adds values
  `AVG()`     Calculates average
  `MIN()`     Smallest value
  `MAX()`     Largest value

``` sql
SELECT
    COUNT(*) AS employee_count,
    SUM(salary) AS total_salary,
    ROUND(AVG(salary),2) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees;
```

**Remember:** `COUNT(*)` counts all rows; `COUNT(column)` ignores NULL
values. Most aggregate functions ignore NULL values.
`COUNT(DISTINCT city)` counts unique non-NULL cities.

## Lecture 12 --- GROUP BY

`GROUP BY` creates groups of rows with the same value. Aggregate
functions calculate a result for each group.

``` sql
SELECT
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department_id;
```

Multiple columns can be grouped:

``` sql
SELECT city, department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY city, department_id;
```

**Remember:** A selected column that is not aggregated should normally
be included in `GROUP BY`. NULL values form their own group.

## Lecture 13 --- HAVING

`HAVING` filters groups after grouping. `WHERE` filters individual rows
before grouping.

``` sql
SELECT department_id, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;
```

Use WHERE and HAVING together:

``` sql
SELECT department_id, AVG(salary) AS average_salary
FROM employees
WHERE salary > 40000
GROUP BY department_id
HAVING AVG(salary) > 50000;
```

**Remember:** WHERE = row filter; HAVING = group filter.

## Lecture 14 --- CASE

`CASE` adds IF/ELSE-style conditional logic to a query.

``` sql
SELECT emp_name, salary,
    CASE
        WHEN salary >= 70000 THEN 'High'
        WHEN salary >= 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

Conditions are evaluated from top to bottom. `ELSE` is optional, but
useful as a default.

## Lecture 15 --- String Functions

  Function                Purpose
  ----------------------- --------------------------------
  `UPPER()` / `LOWER()`   Change letter case
  `LENGTH()`              Length in bytes
  `CHAR_LENGTH()`         Number of characters
  `CONCAT()`              Join strings
  `CONCAT_WS()`           Join strings using a separator
  `TRIM()`                Remove leading/trailing spaces
  `LTRIM()` / `RTRIM()`   Remove left/right spaces
  `SUBSTRING()`           Extract part of a string
  `REPLACE()`             Replace text
  `LEFT()` / `RIGHT()`    Extract from either side
  `LOCATE()`              Find a substring position

``` sql
SELECT
    UPPER(emp_name) AS employee_name,
    LOWER(city) AS city_lower,
    CHAR_LENGTH(emp_name) AS name_length,
    CONCAT(emp_name, ' - ', city) AS employee_location,
    TRIM(email) AS cleaned_email
FROM employees;
```

To display the email username:

``` sql
SELECT email, SUBSTRING_INDEX(email, '@', 1) AS email_username
FROM employees;
```

A SELECT expression displays cleaned data but does not save changes. Use
UPDATE only when you intentionally want to modify stored values.

## Lecture 16 --- Date and Time Functions

  Function                         Purpose
  -------------------------------- -----------------------------
  `CURDATE()`                      Current date
  `NOW()`                          Current date and time
  `YEAR()` / `MONTH()` / `DAY()`   Extract date parts
  `MONTHNAME()` / `DAYNAME()`      Month or weekday name
  `DATEDIFF()`                     Difference in days
  `DATE_ADD()` / `DATE_SUB()`      Add or subtract an interval
  `DATE_FORMAT()`                  Format a date
  `EXTRACT()`                      Extract a date part

``` sql
SELECT
    emp_name,
    hire_date,
    YEAR(hire_date) AS joining_year,
    MONTHNAME(hire_date) AS joining_month,
    DAYNAME(hire_date) AS joining_day
FROM employees;
```

Find employees hired in 2022:

``` sql
SELECT *
FROM employees
WHERE hire_date >= '2022-01-01'
  AND hire_date < '2023-01-01';
```

Monthly revenue:

``` sql
SELECT MONTH(sale_date) AS sale_month,
       SUM(quantity * unit_price) AS monthly_revenue
FROM sales
GROUP BY MONTH(sale_date)
ORDER BY sale_month;
```

Date range filtering is often preferable to applying YEAR() or MONTH()
to an indexed date column in the WHERE clause.

## Lecture 17 --- INNER JOIN

INNER JOIN returns only rows that have a match in both tables.

``` sql
SELECT e.emp_name, d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

Join three tables:

``` sql
SELECT e.emp_name, d.department_name, s.product_name, s.quantity
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN sales s ON e.emp_id = s.emp_id;
```

`ON` defines the matching condition. Table aliases make queries shorter.
INNER JOIN excludes unmatched rows.

## Lecture 18 --- LEFT JOIN and RIGHT JOIN

LEFT JOIN returns all rows from the left table and matching rows from
the right table. Missing matches become NULL.

``` sql
SELECT e.emp_name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

RIGHT JOIN preserves all rows from the right table:

``` sql
SELECT e.emp_name, d.department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.department_id;
```

Find departments with no employees:

``` sql
SELECT d.department_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE e.emp_id IS NULL;
```

**Remember:** Use `IS NULL`, not `= NULL`. After a LEFT JOIN,
`COUNT(right_table.id)` counts matches; `COUNT(*)` counts result rows. A
right-table filter in WHERE can remove unmatched rows.

## Lecture 19 --- FULL, CROSS and SELF JOIN

**FULL OUTER JOIN** returns matched rows and unmatched rows from both
sides. MySQL does not directly support FULL OUTER JOIN. Emulate it:

``` sql
SELECT e.emp_id, e.emp_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id

UNION ALL

SELECT e.emp_id, e.emp_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.emp_id IS NULL;
```

**CROSS JOIN** returns every possible combination:

``` sql
SELECT e.emp_name, d.department_name
FROM employees e
CROSS JOIN departments d;
```

If one table has 8 rows and another has 5, CROSS JOIN returns 40
combinations.

**SELF JOIN** joins a table with itself. Example: employee and manager:

``` sql
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

## Lecture 20 --- UNION and UNION ALL

UNION combines rows from SELECT results and removes duplicate complete
rows.

``` sql
SELECT emp_name FROM employees WHERE city = 'Pune'
UNION
SELECT emp_name FROM employees WHERE city = 'Mumbai';
```

UNION ALL keeps duplicates:

``` sql
SELECT emp_name FROM employees WHERE city = 'Pune'
UNION ALL
SELECT emp_name FROM employees WHERE city = 'Mumbai';
```

**Rules:** Same number of columns; corresponding columns should have
compatible data types; column order must match logically; output names
come from the first SELECT; put final ORDER BY at the end.

**UNION vs JOIN:** UNION stacks rows vertically; JOIN combines columns
horizontally using a relationship. UNION ALL is generally faster because
it does not remove duplicates.

------------------------------------------------------------------------

# 3. Moderate practice questions --- questions only

Attempt these 50 questions before opening Section 5.

## Lecture 11 --- Aggregate Functions

1.  Find total salary paid by the company.
2.  Count employees whose salary is at least 50,000.
3.  Count distinct cities in employees.
4.  Find minimum, maximum, and average unit price in sales.
5.  Calculate total revenue using quantity multiplied by unit price.

## Lecture 12 --- GROUP BY

6.  Count employees in each department, including the NULL department
    group.
7.  Find total salary for each city.
8.  Find average sale quantity for each product.
9.  Find total revenue for each customer.
10. Count sales made by each employee.

## Lecture 13 --- HAVING

11. Display departments having at least two employees.
12. Display products whose total quantity sold is greater than 5.
13. Display customers whose total revenue is greater than 50,000.
14. Find cities whose average employee salary is above 50,000.
15. Find employees who made at least two sales.

## Lecture 14 --- CASE

16. Categorize salaries: High ≥ 70,000; Medium ≥ 50,000; otherwise Low.
17. Categorize sale revenue: Large ≥ 50,000; Medium ≥ 10,000; otherwise
    Small.
18. Label employees hired before 2022 as Senior, others as Recent.
19. Count employees in each salary category.
20. Label a sale as Bulk if quantity ≥ 5, otherwise Regular.

## Lecture 15 --- String Functions

21. Display employee names in uppercase and cities in lowercase.
22. Display the character length of each employee name.
23. Extract the first four characters of every product name.
24. Combine employee name and email separated by `|`.
25. Display the part of each email before `@`.

## Lecture 16 --- Date and Time

26. Display employee name and joining year.
27. Find employees hired during 2022.
28. Display monthly sales revenue for 2025.
29. Find days between each sale date and `2025-07-01`.
30. Display sale date in `DD-Month-YYYY` format.

## Lecture 17 --- INNER JOIN

31. Display employee names with department names.
32. Display sale ID, employee name, product name, and quantity.
33. Find total revenue by employee, only for employees with sales.
34. Find total revenue by department.
35. Display customer, product, and employee names for each sale.

## Lecture 18 --- LEFT JOIN and RIGHT JOIN

36. Display all employees and department names, including employees
    without a department.
37. Display all departments and employees, including empty departments.
38. Find employees who have never made a sale.
39. Find departments with no employees.
40. Count employees in every department, including zero.

## Lecture 19 --- FULL, CROSS and SELF JOIN

41. Simulate FULL OUTER JOIN between employees and departments in MySQL.
42. Generate every employee-department combination.
43. Display employee and manager names using SELF JOIN.
44. Find employees without a manager.
45. Count employee-manager relationships.

## Lecture 20 --- UNION and UNION ALL

46. Combine employee names from Pune and Mumbai using UNION.
47. Combine employee names from Pune and Mumbai using UNION ALL.
48. Combine product names and customer names into one column called
    `name`.
49. Combine employees hired before 2022 and employees hired in 2022 or
    later, with a group label.
50. Explain why UNION ALL is usually preferred when combining
    transaction tables.

------------------------------------------------------------------------

# 4. End-to-end real-world data manipulation case --- questions only

## Business scenario: Retail Sales Analytics

You are a Junior Data Analyst. The manager provides three tables:
`employees`, `departments`, and `sales`. Create a report about data
quality, employee performance, customers, products, and revenue. Use the
database from Section 1.

### A. Understand the data

1.  Display all rows from the three tables.
2.  Find total employees, departments, and sales.
3.  Count employees with no department.
4.  Find sales with a missing employee reference, if any.

### B. Clean and prepare data

5.  Display employee names in uppercase and emails in lowercase.
6.  Display employee names with leading/trailing spaces removed.
7.  Find sales where quantity or unit price is zero or negative.
8.  Find duplicate-looking sales using employee, product, date, and
    customer fields. Do not delete anything.

### C. Employee and department analysis

9.  Display employee name, department name, city, and salary.
10. Display every department with its employee count.
11. Find departments with average salary above 50,000.
12. Find the highest-paid employee in each department.
13. Display employees and their manager names.
14. Find employees who have no department.

### D. Sales analysis

15. Calculate revenue for every sale.
16. Calculate total company revenue.
17. Calculate revenue by product.
18. Calculate revenue by city.
19. Calculate revenue by customer.
20. Calculate monthly revenue in chronological order.
21. Find the top three employees by revenue.
22. Find products with total quantity sold greater than 5.
23. Find customers whose total spending exceeds 50,000.

### E. Business reporting

24. Classify each sale as High, Medium, or Low Value.
25. Display employee name, department, number of sales, and total
    revenue.
26. Find employees with no sales.
27. Find departments with no employees.
28. Create a combined list of employee names and customer names using
    UNION.
29. Create a combined transaction report using UNION ALL.
30. Create a product report with total quantity, total revenue, and
    average unit price.

------------------------------------------------------------------------

# 5. Solutions --- lecture-wise practice

## Lecture 11 --- Aggregate Functions

``` sql
-- 1
SELECT SUM(salary) AS total_salary FROM employees;

-- 2
SELECT COUNT(*) AS employee_count
FROM employees WHERE salary >= 50000;

-- 3
SELECT COUNT(DISTINCT city) AS distinct_cities FROM employees;

-- 4
SELECT MIN(unit_price), MAX(unit_price), AVG(unit_price)
FROM sales;

-- 5
SELECT SUM(quantity * unit_price) AS total_revenue FROM sales;
```

## Lecture 12 --- GROUP BY

``` sql
-- 6
SELECT department_id, COUNT(*) AS employee_count
FROM employees GROUP BY department_id;

-- 7
SELECT city, SUM(salary) AS total_salary
FROM employees GROUP BY city;

-- 8
SELECT product_name, AVG(quantity) AS average_quantity
FROM sales GROUP BY product_name;

-- 9
SELECT customer_name, SUM(quantity * unit_price) AS total_revenue
FROM sales GROUP BY customer_name;

-- 10
SELECT emp_id, COUNT(*) AS total_sales
FROM sales GROUP BY emp_id;
```

## Lecture 13 --- HAVING

``` sql
-- 11
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) >= 2;

-- 12
SELECT product_name, SUM(quantity) AS total_quantity
FROM sales
GROUP BY product_name
HAVING SUM(quantity) > 5;

-- 13
SELECT customer_name, SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY customer_name
HAVING SUM(quantity * unit_price) > 50000;

-- 14
SELECT city, AVG(salary) AS average_salary
FROM employees
GROUP BY city
HAVING AVG(salary) > 50000;

-- 15
SELECT emp_id, COUNT(*) AS sale_count
FROM sales
GROUP BY emp_id
HAVING COUNT(*) >= 2;
```

## Lecture 14 --- CASE

``` sql
-- 16
SELECT emp_name, salary,
CASE
    WHEN salary >= 70000 THEN 'High'
    WHEN salary >= 50000 THEN 'Medium'
    ELSE 'Low'
END AS salary_category
FROM employees;

-- 17
SELECT sale_id, quantity * unit_price AS revenue,
CASE
    WHEN quantity * unit_price >= 50000 THEN 'Large'
    WHEN quantity * unit_price >= 10000 THEN 'Medium'
    ELSE 'Small'
END AS sale_category
FROM sales;

-- 18
SELECT emp_name, hire_date,
CASE WHEN hire_date < '2022-01-01'
     THEN 'Senior' ELSE 'Recent'
END AS employee_status
FROM employees;

-- 19
SELECT
CASE
    WHEN salary >= 70000 THEN 'High'
    WHEN salary >= 50000 THEN 'Medium'
    ELSE 'Low'
END AS salary_category,
COUNT(*) AS employee_count
FROM employees
GROUP BY
CASE
    WHEN salary >= 70000 THEN 'High'
    WHEN salary >= 50000 THEN 'Medium'
    ELSE 'Low'
END;

-- 20
SELECT product_name, quantity,
CASE WHEN quantity >= 5 THEN 'Bulk'
     ELSE 'Regular'
END AS quantity_label
FROM sales;
```

## Lecture 15 --- String Functions

``` sql
-- 21
SELECT UPPER(emp_name), LOWER(city) FROM employees;

-- 22
SELECT emp_name, CHAR_LENGTH(emp_name) AS name_length
FROM employees;

-- 23
SELECT product_name, LEFT(product_name, 4) AS short_name
FROM sales;

-- 24
SELECT CONCAT(emp_name, ' | ', email) AS employee_contact
FROM employees;

-- 25
SELECT email, SUBSTRING_INDEX(email, '@', 1) AS email_username
FROM employees;
```

## Lecture 16 --- Date and Time

``` sql
-- 26
SELECT emp_name, YEAR(hire_date) AS joining_year FROM employees;

-- 27
SELECT *
FROM employees
WHERE hire_date >= '2022-01-01'
  AND hire_date < '2023-01-01';

-- 28
SELECT MONTH(sale_date) AS sale_month,
       SUM(quantity * unit_price) AS monthly_revenue
FROM sales
WHERE sale_date >= '2025-01-01'
  AND sale_date < '2026-01-01'
GROUP BY MONTH(sale_date)
ORDER BY sale_month;

-- 29
SELECT sale_id, sale_date,
       DATEDIFF('2025-07-01', sale_date) AS days_difference
FROM sales;

-- 30
SELECT sale_date,
       DATE_FORMAT(sale_date, '%d-%M-%Y') AS formatted_date
FROM sales;
```

## Lecture 17 --- INNER JOIN

``` sql
-- 31
SELECT e.emp_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;

-- 32
SELECT s.sale_id, e.emp_name, s.product_name, s.quantity
FROM sales s
INNER JOIN employees e ON s.emp_id = e.emp_id;

-- 33
SELECT e.emp_id, e.emp_name,
       SUM(s.quantity * s.unit_price) AS total_revenue
FROM employees e
INNER JOIN sales s ON e.emp_id = s.emp_id
GROUP BY e.emp_id, e.emp_name;

-- 34
SELECT d.department_name,
       SUM(s.quantity * s.unit_price) AS total_revenue
FROM departments d
INNER JOIN employees e ON d.department_id = e.department_id
INNER JOIN sales s ON e.emp_id = s.emp_id
GROUP BY d.department_id, d.department_name;

-- 35
SELECT s.customer_name, s.product_name, e.emp_name
FROM sales s
INNER JOIN employees e ON s.emp_id = e.emp_id;
```

## Lecture 18 --- LEFT JOIN and RIGHT JOIN

``` sql
-- 36
SELECT e.emp_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- 37
SELECT d.department_name, e.emp_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id;

-- 38
SELECT e.emp_id, e.emp_name
FROM employees e
LEFT JOIN sales s ON e.emp_id = s.emp_id
WHERE s.sale_id IS NULL;

-- 39
SELECT d.department_id, d.department_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE e.emp_id IS NULL;

-- 40
SELECT d.department_id, d.department_name,
       COUNT(e.emp_id) AS employee_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;
```

## Lecture 19 --- FULL, CROSS and SELF JOIN

``` sql
-- 41: FULL OUTER JOIN simulation in MySQL
SELECT e.emp_id, e.emp_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id

UNION ALL

SELECT e.emp_id, e.emp_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.emp_id IS NULL;

-- 42
SELECT e.emp_name, d.department_name
FROM employees e CROSS JOIN departments d;

-- 43
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;

-- 44
SELECT emp_id, emp_name
FROM employees
WHERE manager_id IS NULL;

-- 45
SELECT COUNT(*) AS relationship_count
FROM employees
WHERE manager_id IS NOT NULL;
```

## Lecture 20 --- UNION and UNION ALL

``` sql
-- 46
SELECT emp_name FROM employees WHERE city = 'Pune'
UNION
SELECT emp_name FROM employees WHERE city = 'Mumbai';

-- 47
SELECT emp_name FROM employees WHERE city = 'Pune'
UNION ALL
SELECT emp_name FROM employees WHERE city = 'Mumbai';

-- 48
SELECT product_name AS name FROM sales
UNION
SELECT customer_name FROM sales;

-- 49
SELECT emp_name, 'Hired before 2022' AS employee_group
FROM employees
WHERE hire_date < '2022-01-01'
UNION ALL
SELECT emp_name, 'Hired in 2022 or later'
FROM employees
WHERE hire_date >= '2022-01-01';

-- 50
-- UNION ALL preserves every transaction row and avoids the duplicate-removal
-- work performed by UNION. It is usually suitable for transaction data.
```

------------------------------------------------------------------------

# 6. End-to-end case --- complete solution

## A. Understand the data

``` sql
-- 1
SELECT * FROM employees;
SELECT * FROM departments;
SELECT * FROM sales;

-- 2
SELECT 'Employees' AS table_name, COUNT(*) AS total_rows FROM employees
UNION ALL
SELECT 'Departments', COUNT(*) FROM departments
UNION ALL
SELECT 'Sales', COUNT(*) FROM sales;

-- 3
SELECT COUNT(*) AS employees_without_department
FROM employees WHERE department_id IS NULL;

-- 4
SELECT COUNT(*) AS sales_without_employee
FROM sales s
LEFT JOIN employees e ON s.emp_id = e.emp_id
WHERE e.emp_id IS NULL;
```

## B. Clean and prepare data

``` sql
-- 5
SELECT UPPER(TRIM(emp_name)) AS cleaned_name,
       LOWER(TRIM(email)) AS cleaned_email
FROM employees;

-- 6
SELECT emp_name, TRIM(emp_name) AS cleaned_name
FROM employees;

-- 7
SELECT *
FROM sales
WHERE quantity <= 0 OR unit_price <= 0;

-- 8
SELECT emp_id, product_name, sale_date, customer_name,
       COUNT(*) AS repeated_count
FROM sales
GROUP BY emp_id, product_name, sale_date, customer_name
HAVING COUNT(*) > 1;
```

The supplied sample has no duplicate-looking sales and no invalid
quantity or price. The queries are included to show how to check real
data.

## C. Employee and department analysis

``` sql
-- 9
SELECT e.emp_name, d.department_name, e.city, e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- 10
SELECT d.department_name, COUNT(e.emp_id) AS employee_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;

-- 11
SELECT d.department_name, AVG(e.salary) AS average_salary
FROM departments d
INNER JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
HAVING AVG(e.salary) > 50000;

-- 12: Returns all employees tied for the highest salary in their department
SELECT e.emp_name, e.department_id, e.salary
FROM employees e
INNER JOIN (
    SELECT department_id, MAX(salary) AS maximum_salary
    FROM employees
    WHERE department_id IS NOT NULL
    GROUP BY department_id
) x
ON e.department_id = x.department_id
AND e.salary = x.maximum_salary;

-- 13
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;

-- 14
SELECT emp_id, emp_name, city, salary
FROM employees
WHERE department_id IS NULL;
```

## D. Sales analysis

``` sql
-- 15
SELECT sale_id, product_name, quantity, unit_price,
       quantity * unit_price AS revenue
FROM sales;

-- 16
SELECT SUM(quantity * unit_price) AS company_revenue
FROM sales;

-- 17
SELECT product_name, SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY product_name
ORDER BY total_revenue DESC;

-- 18
SELECT city, SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY city
ORDER BY total_revenue DESC;

-- 19
SELECT customer_name, SUM(quantity * unit_price) AS total_spending
FROM sales
GROUP BY customer_name
ORDER BY total_spending DESC;

-- 20
SELECT YEAR(sale_date) AS sales_year,
       MONTH(sale_date) AS sales_month,
       SUM(quantity * unit_price) AS monthly_revenue
FROM sales
GROUP BY YEAR(sale_date), MONTH(sale_date)
ORDER BY sales_year, sales_month;

-- 21
SELECT e.emp_id, e.emp_name,
       SUM(s.quantity * s.unit_price) AS total_revenue
FROM employees e
INNER JOIN sales s ON e.emp_id = s.emp_id
GROUP BY e.emp_id, e.emp_name
ORDER BY total_revenue DESC
LIMIT 3;

-- 22
SELECT product_name, SUM(quantity) AS total_quantity
FROM sales
GROUP BY product_name
HAVING SUM(quantity) > 5;

-- 23
SELECT customer_name,
       SUM(quantity * unit_price) AS total_spending
FROM sales
GROUP BY customer_name
HAVING SUM(quantity * unit_price) > 50000;
```

## E. Business reporting

``` sql
-- 24
SELECT sale_id, product_name, quantity * unit_price AS revenue,
CASE
    WHEN quantity * unit_price >= 50000 THEN 'High Value'
    WHEN quantity * unit_price >= 10000 THEN 'Medium Value'
    ELSE 'Low Value'
END AS sale_category
FROM sales;

-- 25
SELECT e.emp_id, e.emp_name, d.department_name,
       COUNT(s.sale_id) AS number_of_sales,
       COALESCE(SUM(s.quantity * s.unit_price), 0) AS total_revenue
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
LEFT JOIN sales s ON e.emp_id = s.emp_id
GROUP BY e.emp_id, e.emp_name, d.department_name
ORDER BY total_revenue DESC;

-- 26
SELECT e.emp_id, e.emp_name
FROM employees e
LEFT JOIN sales s ON e.emp_id = s.emp_id
WHERE s.sale_id IS NULL;

-- 27
SELECT d.department_id, d.department_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE e.emp_id IS NULL;

-- 28
SELECT emp_name AS person_name FROM employees
UNION
SELECT customer_name FROM sales;

-- 29
SELECT sale_id, product_name AS item, quantity,
       quantity * unit_price AS revenue, 'Before April' AS source_period
FROM sales
WHERE sale_date < '2025-04-01'
UNION ALL
SELECT sale_id, product_name, quantity,
       quantity * unit_price, 'April onward'
FROM sales
WHERE sale_date >= '2025-04-01';

-- 30
SELECT product_name,
       SUM(quantity) AS total_quantity,
       SUM(quantity * unit_price) AS total_revenue,
       AVG(unit_price) AS average_unit_price
FROM sales
GROUP BY product_name
ORDER BY total_revenue DESC;
```

**Important data-analysis lesson:** When joining multiple one-to-many
tables, rows can multiply and inflate totals. Aggregate detail tables
first when needed, then join the aggregated results.

------------------------------------------------------------------------

# 7. Quick revision notes

## Aggregate functions

``` sql
COUNT(*)       -- All rows
COUNT(column)  -- Non-NULL values
SUM(column)    -- Total
AVG(column)    -- Average
MIN(column)    -- Minimum
MAX(column)    -- Maximum
```

## GROUP BY and HAVING

``` sql
SELECT department_id, AVG(salary)
FROM employees
WHERE salary > 40000
GROUP BY department_id
HAVING AVG(salary) > 50000;
```

WHERE filters rows; GROUP BY creates groups; HAVING filters groups.

## CASE

``` sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
```

## String functions

``` sql
UPPER(text)          LOWER(text)
TRIM(text)           LENGTH(text)
CHAR_LENGTH(text)    CONCAT(text1, text2)
SUBSTRING(text, 1, 3)
REPLACE(text, old, new)
LEFT(text, 3)        RIGHT(text, 3)
```

## Date functions

``` sql
CURDATE()
NOW()
YEAR(date_column)
MONTH(date_column)
MONTHNAME(date_column)
DATEDIFF(date1, date2)
DATE_ADD(date, INTERVAL 7 DAY)
DATE_SUB(date, INTERVAL 1 MONTH)
DATE_FORMAT(date, '%d-%m-%Y')
```

## JOIN summary

  Join              Result
  ----------------- -------------------------------------
  INNER JOIN        Matching rows from both tables
  LEFT JOIN         All left rows + matching right rows
  RIGHT JOIN        All right rows + matching left rows
  FULL OUTER JOIN   All rows from both sides
  CROSS JOIN        Every possible combination
  SELF JOIN         A table joined with itself

**MySQL:** FULL OUTER JOIN is not directly supported. Use LEFT JOIN plus
unmatched right-side rows with UNION ALL.

## UNION summary

  UNION                             UNION ALL
  --------------------------------- -------------------------
  Removes duplicate complete rows   Keeps duplicates
  Usually more work                 Usually faster
  Unique combined results           All transactions/events

## Common interview reminders

1.  `COUNT(*)` and `COUNT(column)` differ when NULL values exist.
2.  Use `IS NULL`, not `= NULL`.
3.  WHERE filters rows; HAVING filters groups.
4.  LEFT JOIN preserves every row from its left table.
5.  `COUNT(right_table.id)` counts matches after a LEFT JOIN.
6.  JOIN combines columns; UNION combines rows.
7.  UNION requires the same number of compatible columns.
8.  CASE checks conditions from top to bottom.
9.  Date range filters are often more index-friendly than wrapping a
    date column in YEAR() in WHERE.
10. Joining multiple one-to-many tables can multiply rows and inflate
    aggregates.

## Suggested revision plan

  Session   Work
  --------- ------------------------------------------------------
  Day 1     Lectures 11--14: aggregates, GROUP BY, HAVING, CASE
  Day 2     Lectures 15--16: string and date functions
  Day 3     Lectures 17--19: JOIN types
  Day 4     Lecture 20: UNION and UNION ALL
  Day 5     Solve all 50 moderate questions
  Day 6     Complete the retail sales case without solutions
  Day 7     Revise interview notes and retry incorrect questions

## Completion checklist

-   [ ] I can use aggregate functions correctly.
-   [ ] I understand GROUP BY and HAVING.
-   [ ] I can write CASE expressions.
-   [ ] I can clean and format text with string functions.
-   [ ] I can filter and group dates.
-   [ ] I can use INNER, LEFT, and RIGHT JOIN.
-   [ ] I understand FULL, CROSS, and SELF JOIN.
-   [ ] I can use UNION and UNION ALL.
-   [ ] I can solve the 50 moderate questions.
-   [ ] I can complete the retail sales case independently.

------------------------------------------------------------------------

**End of Lectures 11--20 Revision Workbook**\
**Next lesson:** SQL Lecture 21 --- Subqueries.
