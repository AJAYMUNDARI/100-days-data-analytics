![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Second Highest Salary in Engineering Department

## Business Context

This type of query is commonly used in **HR analytics and compensation analysis**. A company may want to identify the second-highest salary within a department to:

* benchmark compensation levels,
* analyze salary distribution,
* identify senior employee pay bands,
* support promotion and succession planning,
* detect salary concentration at the top level.

The important business rule here is that if multiple employees share the highest salary, we should still return the **next distinct salary**, not the same highest salary again.

---

## 1. Understand the Requirement

We need to find the **second highest distinct salary** among employees who belong to the **engineering department**.

Expected output: one row containing the salary value.

---

## 2. Understand the Tables

| Table         | Purpose                                        | Key Columns                     |
| ------------- | ---------------------------------------------- | ------------------------------- |
| `employees`   | Stores employee details and salary information | `id`, `salary`, `department_id` |
| `departments` | Stores department names                        | `id`, `name`                    |

---

## 3. Clarify the Grain

* `employees`: one row per employee.
* `departments`: one row per department.
* Final output: one row containing one salary value.

---

## 4. Identify Relationships

* `employees.department_id = departments.id`

Many employees can belong to one department.

---

## 5. Determine the Driving Table

**Driving table:** `employees`

Reason: salary is stored in the employees table, and the calculation is performed on employee salary records.

---

## 6. Think About Join Type

Use an **INNER JOIN** because we only need employees that belong to a valid department and specifically the engineering department.

---

## 7. Find the “No Match” Condition

Not required for this problem.

---

## 8. Interpret Special Conditions / Notes

Important business rule:

> If multiple employees share the highest salary, return the next distinct salary.

To satisfy this, we must consider **distinct salary values** rather than employee rows. The query achieves this using `GROUP BY salary`.

---

## 9. Final Calculation Logic

1. Join employees with departments.
2. Filter rows where department name is `engineering`.
3. Keep distinct salary values using `GROUP BY salary`.
4. Sort salaries in descending order.
5. Skip the highest salary (`OFFSET 1`).
6. Return the next salary (`LIMIT 1`).

---

## 10. SQL Solution

```sql
SELECT salary
FROM employees
INNER JOIN departments
    ON employees.department_id = departments.id
WHERE departments.name = 'engineering'
GROUP BY salary
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

---

## 11. Output Explanation

| Column   | Meaning                                                      |
| -------- | ------------------------------------------------------------ |
| `salary` | Second highest distinct salary in the engineering department |

---

## 12. Key SQL Concepts Used

* `INNER JOIN`
* Filtering with `WHERE`
* `GROUP BY`
* `ORDER BY DESC`
* `LIMIT`
* `OFFSET`

---

## 13. Explanation

I first filtered employees belonging to the engineering department. Since the requirement asks for the second **distinct** highest salary, I grouped by salary before sorting in descending order. Finally, I skipped the top salary using `OFFSET 1` and returned the next salary with `LIMIT 1`.
