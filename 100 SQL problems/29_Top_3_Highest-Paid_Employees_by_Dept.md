![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Top 3 Highest-Paid Employees by Department

---

## Business Context

This analysis is useful in **HR analytics, compensation analysis, and workforce planning**. Organizations often compare the highest-paid employees across departments to understand salary distribution and identify senior or highly compensated roles.

It can help businesses:

* analyze compensation structures,
* identify highly paid employees,
* compare salary levels across departments,
* support workforce and compensation planning,
* identify potential salary outliers.

The requirement to return up to the top 3 employees per department allows departments with fewer than 3 employees to return only the employees they have.

---

## Problem Statement

For every department, find the **top 3 highest salaries**.

Requirements:

* If a department has fewer than 3 employees, return all available employees.
* Display the employee's full name.
* Display the department name.
* Display the salary.
* Sort by department name ascending and salary descending.

---

## Input Tables

### employees

| Column        | Type    |
| ------------- | ------- |
| id            | INTEGER |
| first_name    | VARCHAR |
| last_name     | VARCHAR |
| salary        | INTEGER |
| department_id | INTEGER |

### departments

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

---

## Expected Output

| Column          | Type    |
| --------------- | ------- |
| employee_name   | VARCHAR |
| department_name | VARCHAR |
| salary          | INTEGER |

---

## 1. Understand the Requirement

We need to:

1. Join employees with their departments.
2. Create the employee's full name.
3. Rank employees within each department by salary.
4. Keep the top 3 ranks.
5. Sort the final result by department name and salary.

---

## 2. Understand the Tables

| Table       | Purpose                                | Key Columns           |
| ----------- | -------------------------------------- | --------------------- |
| employees   | Stores employee information and salary | department_id, salary |
| departments | Stores department names                | id, name              |

---

## 3. Clarify the Grain

* `employees`: one row per employee.
* `departments`: one row per department.
* Intermediate result: one row per employee with a department-level salary rank.
* Final output: up to 3 rows per department.

---

## 4. Identify Relationships

```text
employees.department_id = departments.id
```

One department can contain many employees.

---

## 5. Determine the Driving Table

**Driving table:** `employees`

Reason: the final result is based on employee salaries, while the departments table is used to retrieve the department name.

---

## 6. Think About Join Type

Use an **INNER JOIN**.

Reason: the problem states that every department has at least one employee, and we only need employees that belong to a department.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

Every department is assumed to have at least one employee.

---

## 8. Interpret Special Conditions / Notes

The ranking must restart for every department:

```sql
RANK() OVER (
    PARTITION BY department_id
    ORDER BY salary DESC
)
```

Only employees with rank ≤ 3 should be returned.

Using `RANK()` means employees with the same salary receive the same rank.

---

## 9. Final Calculation Logic

1. Join employees and departments.
2. Combine first and last name.
3. Partition employees by department.
4. Rank salaries from highest to lowest.
5. Filter ranks ≤ 3.
6. Sort by department name ascending and salary descending.

---

## 10. SQL Solution

```sql
SELECT
    employee_name,
    department_name,
    salary
FROM (
    SELECT
        CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
        d.name AS department_name,
        e.salary,
        RANK() OVER (
            PARTITION BY e.department_id
            ORDER BY e.salary DESC
        ) AS r
    FROM departments AS d
    INNER JOIN employees AS e
        ON e.department_id = d.id
) AS x
WHERE r <= 3
ORDER BY department_name ASC, salary DESC;
```

---

## 11. Why `RANK()`?

`RANK()` handles salary ties.

For example:

| Employee | Salary | Rank |
| -------- | -----: | ---: |
| A        |   150K |    1 |
| B        |   140K |    2 |
| C        |   140K |    2 |
| D        |   120K |    4 |

Filtering with:

```sql
WHERE r <= 3
```

would return A, B, and C.

So the query can return **more than 3 employees** in a department if there is a salary tie at the third rank.

---

## 12. `RANK()` vs `ROW_NUMBER()`

This is an important interview consideration.

### `RANK()`

Returns the same rank for tied salaries.

```text
150K → 1
140K → 2
140K → 2
120K → 4
```

### `ROW_NUMBER()`

Always gives every employee a unique position.

```text
150K → 1
140K → 2
140K → 3
120K → 4
```

If the requirement means **exactly 3 employees per department**, `ROW_NUMBER()` would be more appropriate.

If tied salaries should share the same ranking, `RANK()` is appropriate.

---

## 13. Output Explanation

| Column          | Meaning                        |
| --------------- | ------------------------------ |
| employee_name   | Employee's first and last name |
| department_name | Employee's department          |
| salary          | Employee's salary              |

---

## 14. Key SQL Concepts Used

* `INNER JOIN`
* `CONCAT()`
* `RANK()`
* Window functions
* `PARTITION BY`
* `ORDER BY`
* CTE/subquery
* Filtering window-function results

---

## 15. Edge Cases Considered

* Departments with 1 employee → 1 employee returned.
* Departments with 2 employees → up to 2 employees returned.
* Departments with 3+ employees → top 3 ranks returned.
* Tied salaries may cause more than 3 employees to be returned when using `RANK()`.
* Department names are sorted alphabetically.
* Salaries are sorted from highest to lowest within each department.

---

## 16. Explanation

I joined employees with departments and used `RANK()` partitioned by department to rank employees based on salary in descending order. I then filtered for the top three ranks and sorted the final output by department name and salary.

