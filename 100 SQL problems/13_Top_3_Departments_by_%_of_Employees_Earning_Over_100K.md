# Top 3 Departments by Percentage of Employees Earning Over 100K

# Business Context

This analysis is commonly used in **HR analytics, compensation benchmarking, and workforce planning**. Organizations often compare departments based on the proportion of employees earning above a high-salary threshold.

The result helps leadership teams:

* identify highly compensated departments,
* benchmark compensation structures,
* understand talent concentration,
* support budgeting and workforce planning,
* compare departments with similar headcount sizes.

The requirement to include only departments with **at least 10 employees** ensures that the comparison is statistically more meaningful.

---

# Problem Statement

Select the **top 3 departments** that have **at least 10 employees** and rank them by the **percentage of employees earning more than 100,000**.

Return:

* percentage_over_100k
* department_name
* number_of_employees

---

# Input Tables

## Table: employees

| Column        | Type    |
| ------------- | ------- |
| id            | INTEGER |
| first_name    | VARCHAR |
| last_name     | VARCHAR |
| salary        | INTEGER |
| department_id | INTEGER |

## Table: departments

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

---

# Expected Output

| Column               | Type    |
| -------------------- | ------- |
| percentage_over_100k | FLOAT   |
| department_name      | VARCHAR |
| number_of_employees  | INTEGER |

---

# 1. Understand the Requirement

We need to:

1. Count employees in each department.
2. Keep only departments with **10 or more employees**.
3. Calculate the percentage of employees whose salary is **greater than 100,000**.
4. Sort departments by this percentage in descending order.
5. Return the top 3 departments.

---

# 2. Understand the Tables

| Table       | Purpose                                           | Important Columns     |
| ----------- | ------------------------------------------------- | --------------------- |
| employees   | Stores employee salary and department information | salary, department_id |
| departments | Stores department names                           | id, name              |

---

# 3. Clarify the Grain

* `employees`: one row per employee.
* `departments`: one row per department.
* Final output: one row per department.

---

# 4. Identify Relationships

* `employees.department_id = departments.id`

One department can have many employees.

---

# 5. Determine the Driving Table

**Driving table:** `departments`

Reason: the final output is department-level, and we need department names even before aggregation.

---

# 6. Think About Join Type

Use a **LEFT JOIN** from departments to employees.

Reason:

* It preserves department rows before aggregation.
* Departments with no employees would have a count of 0 and be removed by the `HAVING` clause.

An `INNER JOIN` would also work here because departments without employees cannot satisfy the `HAVING COUNT(*) >= 10` condition.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rules:

* Salary threshold is **strictly greater than 100,000**.
* Departments must have **at least 10 employees**.
* Return only the **top 3 departments** after ranking by percentage.

The expression:

```sql id=
SELECT AVG(CASE WHEN salary > 100000 
        THEN 1 ELSE 0 END) AS percentage_over_100k
      , d.name as department_name
      , COUNT(*) AS number_of_employees
FROM departments AS d
LEFT JOIN employees AS e
    ON d.id = e.department_id
GROUP BY d.name
HAVING COUNT(*) >= 10
ORDER BY 1 DESC
LIMIT 3
```
