![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Top 5 Most Expensive Projects by Budget per Employee

---

# Business Context

This problem is relevant to **project portfolio management, finance, and resource planning**. Organizations often evaluate how much budget is allocated relative to the number of employees assigned to a project.

This metric helps teams:

* identify projects with unusually high spending per employee,
* detect inefficient resource allocation,
* prioritize budget reviews,
* compare staffing intensity across projects,
* support executive portfolio decisions.

The question also introduces a **data quality issue (duplicate employee assignments)**, which is a realistic business scenario. Correctly handling duplicates is critical because failing to do so would understate the budget-per-employee ratio and lead to incorrect business conclusions.

---

# 1. Understand the Requirement

We need to identify the **top 5 projects with the highest budget-to-employee ratio**.

Additional business rule:

* The `employee_projects` table contains duplicate rows.
* Each employee should be counted only once per project.

Expected output:

* project title,
* budget per employee ratio.

---

# 2. Understand the Tables

| Table               | Purpose                               | Key Columns                 |
| ------------------- | ------------------------------------- | --------------------------- |
| `projects`          | Stores project information and budget | `id`, `title`, `budget`     |
| `employee_projects` | Maps employees to projects            | `project_id`, `employee_id` |

---

# 3. Clarify the Grain

* `projects`: one row per project.
* `employee_projects`: intended grain is one row per employee-project assignment, but duplicates exist.
* Deduplicated assignment grain: one row per `(project_id, employee_id)`.
* Final output: one row per project.

---

# 4. Identify Relationships

* `projects.id = employee_projects.project_id`

One project can have many employees.

---

# 5. Determine the Driving Table

**Driving table:** `projects`

Reason: the final result is project-level, and we need project attributes such as title and budget.

---

# 6. Think About Join Type

Use an **INNER JOIN** between projects and the employee count subquery.

Reason:

* We only want projects that have at least one employee assignment after deduplication.
* Projects without employees cannot produce a meaningful budget-per-employee ratio.

---

# 7. Find the “No Match” Condition

Not required for this problem.

---

# 8. Interpret Special Conditions / Notes

Important business rule:

> Duplicate employee-project rows must not increase the employee count.

To handle this, we first create a deduplicated set of `(project_id, employee_id)` pairs and then count employees from that cleaned dataset.

Another requirement is to return only the **top five projects** ordered by highest budget per employee.

---

# 9. Final Calculation Logic

1. Remove duplicate employee-project pairs.
2. Count unique employees for each project.
3. Join the employee counts with the projects table.
4. Calculate `budget / num_employees`.
5. Sort in descending order of the ratio.
6. Return the top 5 projects.

---

# 10. SQL Solution

```sql
SELECT 
    p.title, 
    budget/num_employees AS budget_per_employee
FROM projects AS p
INNER JOIN (
    SELECT project_id, COUNT(*) AS num_employees
    FROM (
        SELECT project_id, employee_id
        FROM employee_projects
        GROUP BY 1,2
    ) AS gb
    GROUP BY project_id
) AS ep
    ON p.id = ep.project_id
ORDER BY budget/num_employees DESC
LIMIT 5;
```
