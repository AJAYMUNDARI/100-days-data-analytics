![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Largest Salary by Department

## Business Context

This analysis helps HR and finance teams identify the **highest-paid employee within each department**. The result can be used for:

* Compensation benchmarking across departments.
* Budget and payroll analysis.
* Identifying salary outliers.
* Leadership and workforce planning discussions.
* Reviewing salary structures during annual compensation planning.

This type of query is commonly used in **HR analytics, payroll reporting, and compensation management**.

---

## 1. Understand the Requirement

The business wants to know the **largest salary paid in each department**.

In simple terms:

* Look at all employees in a department.
* Find the highest salary among them.
* Return one record per department.

Expected output:

| Column         | Meaning                           |
| -------------- | --------------------------------- |
| department     | Department name                   |
| largest_salary | Highest salary in that department |

---

## 2. Understand the Tables

| Table     | Purpose                                        | Key Columns            |
| --------- | ---------------------------------------------- | ---------------------- |
| employees | Stores employee information and salary details | id, department, salary |

---

## 3. Clarify the Grain

* **employees table:** one row per employee.
* **Final output:** one row per department.

Example:

* If the Engineering department has 50 employees, the output will contain **1 row for Engineering**.

---

## 4. Identify Relationships

No relationships are required because only one table is used in this problem.

---

## 5. Determine the Driving Table

**Driving table:** `employees`

Reason:

* The department and salary information are directly available in the employees table.
* No additional table is needed to compute the result.

---

## 6. Think About Join Type

No joins are required for this problem because all required columns exist in a single table.

| Join       | Required? | Reason                 |
| ---------- | --------- | ---------------------- |
| INNER JOIN | No        | Only one table is used |
| LEFT JOIN  | No        | Only one table is used |

---

## 7. Find the “No Match” Condition (if applicable)

**Not required for this problem.**

There is no need to identify unmatched records because no joins are performed.

---

## 8. Interpret Special Conditions / Notes

* Salaries are assumed to be numeric values.
* Each employee belongs to one department.
* Departments with multiple employees having the same highest salary will still return a single row with that salary value.
* The query returns the salary amount only; it does not return the employee ID or employee name associated with that salary.

---

## 9. Final Calculation Logic

Step-by-step logic:

1. Read all rows from the `employees` table.
2. Group rows by `department`.
3. Within each department, identify the maximum salary using `MAX(salary)`.
4. Return the department and its maximum salary.

Example:

| Department  | Salaries            | Largest Salary |
| ----------- | ------------------- | -------------- |
| Engineering | 60000, 75000, 72000 | 75000          |
| HR          | 50000, 52000        | 52000          |

---

## 10. SQL Solution

```sql
SELECT
    department,
    MAX(salary) AS largest_salary
FROM employees
GROUP BY department;
```

---

## 11. Output Explanation

| Output Column  | Description                                       |
| -------------- | ------------------------------------------------- |
| department     | Name of the department                            |
| largest_salary | Maximum salary among employees in that department |

Sample output:

| department  | largest_salary |
| ----------- | -------------- |
| Engineering | 75000          |
| HR          | 52000          |
| Finance     | 90000          |

---

## 12. Key SQL Concepts Used

* `GROUP BY`
* Aggregate function `MAX()`
* Column alias using `AS`

---

## 13. Explanation

I first identified that the required output grain is **one row per department**. Since the business only needs the highest salary value, I grouped the employees table by department and applied `MAX(salary)` to compute the largest salary in each group. This is an efficient aggregation-based solution with no joins required.
