---
title: SQL Queries
parent: SQL, JPA and Hibernate
nav_order: 1
---

# SQL Queries
{: .no_toc }

Classic SQL interview queries with solutions, explanations, and examples.
{: .fs-6 .fw-300 }

Questions on this page
{: .text-delta }

1. TOC
{:toc}

---

## Sample Database Schema

The following examples use MySQL 8.0+ syntax unless otherwise mentioned.

**Employee**

| id | name | salary | dept_id | email |
|---:|---|---:|---:|---|
| 1 | Alice | 90000 | 1 | alice@example.com |
| 2 | Bob | 80000 | 2 | bob@example.com |
| 3 | Charlie | 80000 | 1 | charlie@example.com |
| 4 | David | 70000 | 2 | david@example.com |
| 5 | Eva | 60000 | 1 | eva@example.com |

**Department**

| id | name |
|---:|---|
| 1 | IT |
| 2 | HR |
| 3 | Finance |

---

## How do you find the maximum salary?

Easy
{: .label .label-green }

**Answer**

```sql
SELECT MAX(salary) AS highest_salary
FROM employee;
```

**Explanation**

- `MAX(salary)` returns the largest salary.
- The result contains one row.
- `MAX()` ignores `NULL` values.

---

## How do you find the second-highest salary?

Easy
{: .label .label-green }

**Answer — second-highest distinct salary**

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employee
WHERE salary < (
    SELECT MAX(salary)
    FROM employee
);
```

**Explanation**

1. The subquery finds the highest salary.
2. The outer query excludes salaries equal to the maximum.
3. `MAX()` returns the largest remaining salary.

If the highest salary is 90,000 and the next distinct salary is 80,000, the result is `80000`.

**Alternative using `DENSE_RANK()`**

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employee
) ranked
WHERE salary_rank = 2;
```

This returns every employee salary tied at the second-highest distinct salary.

---

## How do you find the Nth-highest salary?

Medium
{: .label .label-yellow }

**Answer — Nth-highest distinct salary**

```sql
-- Example: Find the 5th-highest distinct salary.

SELECT DISTINCT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employee
) ranked
WHERE salary_rank = 5;
```

**Explanation**

- `DENSE_RANK()` assigns the same rank to equal salaries.
- The next distinct salary receives the next consecutive rank.
- Filtering on rank 5 returns the fifth distinct salary.

**Difference between ranking functions**

| Function | Behavior |
|---|---|
| `ROW_NUMBER()` | Assigns a unique number to every row, even when salaries tie. |
| `RANK()` | Equal salaries share a rank; subsequent ranks may have gaps. |
| `DENSE_RANK()` | Equal salaries share a rank; subsequent ranks have no gaps. |

For example, salaries `100000, 90000, 90000, 80000` produce:

| Salary | `ROW_NUMBER()`* | `RANK()` | `DENSE_RANK()` |
|---:|---:|---:|---:|
| 100000 | 1 | 1 | 1 |
| 90000 | 2 | 2 | 2 |
| 90000 | 3 | 2 | 2 |
| 80000 | 4 | 4 | 3 |

*The exact row numbers for tied salaries depend on the ordering unless a tie-breaker is added.*

**If the interviewer asks for the fifth employee row by salary**, use `ROW_NUMBER()` with a deterministic tie-breaker:

```sql
SELECT id, name, salary
FROM (
    SELECT id,
           name,
           salary,
           ROW_NUMBER() OVER (
               ORDER BY salary DESC, id ASC
           ) AS row_num
    FROM employee
) ranked
WHERE row_num = 5;
```

---

## How do you find and remove duplicate records?

Medium
{: .label .label-yellow }

### Find duplicate email addresses

```sql
SELECT email, COUNT(*) AS occurrences
FROM employee
GROUP BY email
HAVING COUNT(*) > 1;
```

**Explanation**

- `GROUP BY email` groups rows with the same email.
- `COUNT(*)` counts records in each group.
- `HAVING COUNT(*) > 1` returns groups containing duplicates.

### Find duplicate records and keep the lowest ID

Assume employees are considered duplicates when they have the same email address.

```sql
SELECT id, email
FROM (
    SELECT id,
           email,
           ROW_NUMBER() OVER (
               PARTITION BY email
               ORDER BY id ASC
           ) AS row_num
    FROM employee
) ranked
WHERE row_num > 1;
```

This identifies duplicate rows while retaining the row with the smallest ID for each email.

### Delete duplicate records

```sql
DELETE FROM employee
WHERE id IN (
    SELECT id
    FROM (
        SELECT id,
               ROW_NUMBER() OVER (
                   PARTITION BY email
                   ORDER BY id ASC
               ) AS row_num
        FROM employee
    ) ranked
    WHERE row_num > 1
);
```

**Important:** Back up the table and inspect the rows selected for deletion before running a `DELETE` statement in production. Define duplicates using the appropriate business columns. If `email` can be `NULL`, decide whether multiple `NULL` emails should count as duplicates.

---

## How do you find the highest salary in each department?

Medium
{: .label .label-yellow }

**Answer — maximum salary per department**

```sql
SELECT d.name AS department,
       MAX(e.salary) AS highest_salary
FROM department d
JOIN employee e
    ON e.dept_id = d.id
GROUP BY d.id, d.name;
```

**Explanation**

- `JOIN` associates employees with their departments.
- `MAX(e.salary)` finds the largest salary in each department.
- `GROUP BY` produces one result per department represented by employees.

### Find all employees earning the highest salary in their department

```sql
SELECT e.id,
       e.name,
       e.salary,
       d.name AS department
FROM employee e
JOIN department d
    ON d.id = e.dept_id
JOIN (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM employee
    GROUP BY dept_id
) highest
    ON highest.dept_id = e.dept_id
   AND highest.max_salary = e.salary;
```

This version returns employee details and preserves ties when multiple employees share the highest salary.

---

## How do you count employees in each department?

Easy
{: .label .label-green }

**Answer — include departments with zero employees**

```sql
SELECT d.name AS department,
       COUNT(e.id) AS employee_count
FROM department d
LEFT JOIN employee e
    ON e.dept_id = d.id
GROUP BY d.id, d.name
ORDER BY d.name;
```

**Explanation**

- `LEFT JOIN` retains every department, even when it has no employees.
- `COUNT(e.id)` counts matching employees.
- Departments without employees receive a count of `0`.

**Interview tip:** `COUNT(*)` counts the department row even when no employee matches. For this query, use `COUNT(e.id)` instead.

If you only need departments that have employees, use an `INNER JOIN`.

---

## How do you find employees earning more than the average salary?

Easy
{: .label .label-green }

```sql
SELECT id, name, salary
FROM employee
WHERE salary > (
    SELECT AVG(salary)
    FROM employee
);
```

**Explanation**

The subquery calculates the average salary. The outer query returns employees whose salaries exceed that average.

### Find employees earning more than their department's average salary

```sql
SELECT e.id,
       e.name,
       e.salary,
       e.dept_id
FROM employee e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employee e2
    WHERE e2.dept_id = e.dept_id
);
```

This is a correlated subquery because the inner query references the current employee's department.

---

## How do you find employees whose salary is greater than their manager's salary?

Medium
{: .label .label-yellow }

Assume the `employee` table contains a `manager_id` column referencing `employee.id`.

```sql
SELECT e.name AS employee,
       e.salary AS employee_salary,
       m.name AS manager,
       m.salary AS manager_salary
FROM employee e
JOIN employee m
    ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

**Explanation**

This is a **self-join**: the employee table is joined to itself to compare each employee's salary with their manager's salary.

---

## How do you find employees who do not belong to any department?

Easy
{: .label .label-green }

```sql
SELECT e.id, e.name
FROM employee e
LEFT JOIN department d
    ON d.id = e.dept_id
WHERE d.id IS NULL;
```

This returns employees whose department reference does not match any department.

If `dept_id` is nullable, employees with a `NULL` department are also included.

---

## How do you find the top three salaries in each department?

Medium
{: .label .label-yellow }

**Answer — top three distinct salary levels**

```sql
SELECT id,
       name,
       dept_id,
       salary
FROM (
    SELECT id,
           name,
           dept_id,
           salary,
           DENSE_RANK() OVER (
               PARTITION BY dept_id
               ORDER BY salary DESC
           ) AS salary_rank
    FROM employee
) ranked
WHERE salary_rank <= 3
ORDER BY dept_id, salary DESC;
```

**Explanation**

- `PARTITION BY dept_id` ranks employees separately in each department.
- `ORDER BY salary DESC` ranks the highest salary first.
- `DENSE_RANK()` keeps employees with equal salaries at the same rank.
- `salary_rank <= 3` returns the top three distinct salary levels, potentially more than three employees.

Use `ROW_NUMBER()` instead if the requirement is to return at most three employee rows per department.

---

## How do you find students enrolled in courses priced above 50,000?

Easy
{: .label .label-green }

Assume these tables:

- `students(student_id, student_name)`
- `enrollments(student_id, course_id)`
- `courses(course_id, course_name, price)`

**Answer**

```sql
SELECT DISTINCT s.student_id,
       s.student_name
FROM students s
JOIN enrollments e
    ON e.student_id = s.student_id
JOIN courses c
    ON c.course_id = e.course_id
WHERE c.price > 50000;
```

**Explanation**

1. Join students to their enrollments.
2. Join enrollments to courses.
3. Filter courses whose price exceeds 50,000.
4. Use `DISTINCT` so each student appears only once, even if they are enrolled in multiple qualifying courses.

---

## How do you find departments with more than five employees?

Easy
{: .label .label-green }

```sql
SELECT d.id,
       d.name,
       COUNT(e.id) AS employee_count
FROM department d
JOIN employee e
    ON e.dept_id = d.id
GROUP BY d.id, d.name
HAVING COUNT(e.id) > 5;
```

**Explanation**

- `WHERE` filters individual rows before grouping.
- `HAVING` filters groups after aggregate calculations.
- `COUNT(e.id) > 5` retains departments with more than five employees.

---

## How do you find employees hired in the last 30 days?

Easy
{: .label .label-green }

Assume `hire_date` is a `DATE` or `DATETIME` column in MySQL.

```sql
SELECT id, name, hire_date
FROM employee
WHERE hire_date >= CURRENT_DATE - INTERVAL 30 DAY
  AND hire_date < CURRENT_DATE + INTERVAL 1 DAY;
```

This returns employees hired during the last 30 days, including today, with date-time values handled through the end of today.

**Interview tip:** Date arithmetic varies by database. SQL Server, PostgreSQL, and Oracle have their own date functions and interval syntax.

---

## How do you find records present in one table but missing from another?

Medium
{: .label .label-yellow }

For example, find students who have never enrolled in a course.

```sql
SELECT s.student_id,
       s.student_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

**Explanation**

`NOT EXISTS` returns students for whom no matching enrollment exists.

This is generally a safe alternative to `NOT IN` when the subquery may contain `NULL` values.

---

## How do you update salaries by a percentage?

Easy

{: .label .label-green }

Increase salaries in department 1 by 10%.

```sql
UPDATE employee
SET salary = salary * 1.10
WHERE dept_id = 1;
```

**Explanation**

- Multiplying by `1.10` increases salary by 10%.
- The `WHERE` clause restricts the update to department 1.

Always verify the target rows before executing an update on production data.

---

## How do you find the total, average, minimum, and maximum salary?

Easy
{: .label .label-green }

```sql
SELECT COUNT(*) AS employee_count,
       SUM(salary) AS total_salary,
       AVG(salary) AS average_salary,
       MIN(salary) AS minimum_salary,
       MAX(salary) AS maximum_salary
FROM employee;
```

Aggregate functions generally ignore `NULL` salary values, except `COUNT(*)`, which counts rows regardless of whether individual column values are `NULL`.

---

## How do you find employees whose names start with A?

Easy
{: .label .label-green }

```sql
SELECT id, name
FROM employee
WHERE name LIKE 'A%';
```

Common `LIKE` patterns:

| Pattern | Meaning |
|---|---|
| `'A%'` | Starts with A |
| `'%A'` | Ends with A |
| `'%A%'` | Contains A |
| `'_A%'` | Has A as the second character |

Case sensitivity depends on the database and column collation.

---

## How do you create an employee and address database design?

Medium
{: .label .label-yellow }

### Option A: One address per employee

If each employee has one address, create the address table first, then reference it from the employee table.

```sql
CREATE TABLE addresses (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    street VARCHAR(150) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    postal_code VARCHAR(15) NOT NULL,
    country VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    age INT,
    email VARCHAR(150) NOT NULL UNIQUE,
    address_id BIGINT,
    CONSTRAINT fk_employee_address
        FOREIGN KEY (address_id)
        REFERENCES addresses(id)
);
```

**Relationship:** Many employees can reference the same address. If the requirement is strictly one-to-one, add a `UNIQUE` constraint to `employees.address_id`.

### Option B: Multiple addresses per employee

If an employee can have a home address, office address, and mailing address, use a separate relationship table.

```sql
CREATE TABLE employees (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE
);

CREATE TABLE addresses (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    street VARCHAR(150) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    postal_code VARCHAR(15) NOT NULL,
    country VARCHAR(100) NOT NULL
);

CREATE TABLE employee_addresses (
    employee_id BIGINT NOT NULL,
    address_id BIGINT NOT NULL,
    address_type VARCHAR(20) NOT NULL,
    PRIMARY KEY (employee_id, address_id),
    FOREIGN KEY (employee_id) REFERENCES employees(id),
    FOREIGN KEY (address_id) REFERENCES addresses(id)
);
```

**Interview tip:** Choose the schema according to the business relationship. A foreign key enforces referential integrity; a unique constraint enforces uniqueness.

---

## How do you retrieve employees along with their department names?

Easy
{: .label .label-green }

```sql
SELECT e.id,
       e.name,
       e.salary,
       d.name AS department
FROM employee e
JOIN department d
    ON d.id = e.dept_id;
```

Use `LEFT JOIN` instead of `JOIN` if you want to include employees who have no matching department.

---

## How do you find the number of employees earning each salary?

Easy
{: .label .label-green }

```sql
SELECT salary,
       COUNT(*) AS employee_count
FROM employee
GROUP BY salary
ORDER BY salary DESC;
```

This groups employees by salary and counts how many employees earn each amount.

---

## What is the difference between WHERE and HAVING?

Easy
{: .label .label-green }

**Using `WHERE`**

```sql
SELECT *
FROM employee
WHERE salary > 50000;
```

Filters individual employees before grouping.

**Using `HAVING`**

```sql
SELECT dept_id,
       COUNT(*) AS employee_count
FROM employee
GROUP BY dept_id
HAVING COUNT(*) > 5;
```

Filters grouped results after aggregation.

---

## What is the difference between DELETE, TRUNCATE, and DROP?

Easy
{: .label .label-green }

| Command | Purpose |
|---|---|
| `DELETE` | Removes selected rows from a table; supports the `WHERE` clause. |
| `TRUNCATE` | Removes all rows from a table while retaining its structure, subject to database restrictions. |
| `DROP` | Removes the entire table, including its data and structure. |

Examples:

```sql
DELETE FROM employee
WHERE id = 10;

TRUNCATE TABLE employee;

DROP TABLE employee;
```

**Caution:** `TRUNCATE` and `DROP` have database-specific transaction, identity-reset, and foreign-key behavior. Never execute destructive commands without confirming the target.

---

## How do you find the second-highest salary in each department?

Medium
{: .label .label-yellow }

```sql
SELECT id,
       name,
       dept_id,
       salary
FROM (
    SELECT id,
           name,
           dept_id,
           salary,
           DENSE_RANK() OVER (
               PARTITION BY dept_id
               ORDER BY salary DESC
           ) AS salary_rank
    FROM employee
) ranked
WHERE salary_rank = 2;
```

This returns every employee whose salary is the second-highest distinct salary in their department. Departments with fewer than two distinct non-`NULL` salaries will not appear.

---

## How do you find employees who share the same salary?

Easy
{: .label .label-green }

```sql
SELECT salary,
       COUNT(*) AS employee_count
FROM employee
WHERE salary IS NOT NULL
GROUP BY salary
HAVING COUNT(*) > 1;
```

To retrieve the employee details for those salaries:

```sql
SELECT id, name, salary
FROM employee
WHERE salary IN (
    SELECT salary
    FROM employee
    WHERE salary IS NOT NULL
    GROUP BY salary
    HAVING COUNT(*) > 1
)
ORDER BY salary DESC, id;
```

---

## How do you find the department with the highest total salary expenditure?

Medium
{: .label .label-yellow }

```sql
SELECT d.id,
       d.name,
       SUM(e.salary) AS total_salary
FROM department d
JOIN employee e
    ON e.dept_id = d.id
GROUP BY d.id, d.name
ORDER BY total_salary DESC
LIMIT 1;
```

This returns one department with the highest total salary. If multiple departments tie and all must be returned, use a `DENSE_RANK()` window function over the aggregated totals instead.

---

## Quick Interview Revision

| Requirement | SQL concept |
|---|---|
| Highest salary | `MAX()` |
| Nth distinct salary | `DENSE_RANK()` |
| Nth row by salary | `ROW_NUMBER()` |
| Duplicate records | `GROUP BY`, `HAVING` |
| Delete duplicates | `ROW_NUMBER()` and `DELETE` |
| Highest salary per department | `GROUP BY`, `MAX()` |
| Employee count per department | `COUNT()`, `GROUP BY` |
| Departments with more than N employees | `HAVING` |
| Employee vs manager salary | Self-join |
| Employees without a department | `LEFT JOIN`, `IS NULL` |
| Students without enrollments | `NOT EXISTS` |
| Top N salaries per department | Window functions with `PARTITION BY` |
| Employee and department details | `JOIN` |
| Update salaries by percentage | `UPDATE` |
| Employee-address relationship | Primary key, foreign key, unique constraint |

## Best Practices

1. Understand whether “Nth highest salary” means the Nth row or the Nth distinct salary.
2. Use deterministic ordering, such as `ORDER BY salary DESC, id ASC`, when a unique row order is required.
3. Use `DENSE_RANK()` when ties should share the same salary rank.
4. Use `COUNT(column)` instead of `COUNT(*)` when counting matching rows after an outer join.
5. Use `NOT EXISTS` when checking for missing related records.
6. Test `UPDATE` and `DELETE` statements with a corresponding `SELECT` first.
7. Choose indexes based on query patterns and validate performance with the database's execution plan.
8. Remember that SQL syntax and behavior can differ across MySQL, PostgreSQL, SQL Server, and Oracle.

---

## References

- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
- [MySQL Window Functions](https://dev.mysql.com/doc/refman/8.0/en/window-functions.html)
- [MySQL JOIN Syntax](https://dev.mysql.com/doc/refman/8.0/en/join.html)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)