# Project Budget Forecast Based on Employee Salary Cost

---

# Business Context

This analysis is commonly used in **project finance, PMO (Project Management Office), and resource planning**. Organizations want to estimate whether a project is likely to exceed its approved budget based on employee salary costs over the project duration.

The query helps stakeholders:

* forecast project overspending,
* identify projects at financial risk,
* compare planned budget versus estimated labor cost,
* support staffing and budgeting decisions,
* prioritize budget reviews before project completion.

The business assumption is that employee salary cost accrues proportionally over the project duration.

---

# Problem Statement

For each project, determine whether the estimated employee salary cost exceeds the project budget.

Return:

* project title,
* forecast label (`overbudget` or `within budget`).

---

# Input Tables

## Table: projects

| Column     | Type     |
| ---------- | -------- |
| id         | INTEGER  |
| title      | VARCHAR  |
| start_date | DATETIME |
| end_date   | DATETIME |
| budget     | INTEGER  |

## Table: employee_projects

| Column      | Type    |
| ----------- | ------- |
| project_id  | INTEGER |
| employee_id | INTEGER |

## Table: employees

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| salary | INTEGER |

---

# Expected Output

| Column           | Type    |
| ---------------- | ------- |
| title            | VARCHAR |
| project_forecast | VARCHAR |

---

# 1. Understand the Requirement

We need to:

1. Calculate project duration in days.
2. Sum salaries of employees assigned to each project.
3. Estimate salary cost for the project duration.
4. Compare estimated cost with project budget.
5. Label the project as:

   * `overbudget` if estimated cost > budget,
   * `within budget` otherwise.

---

# 2. Understand the Tables

| Table             | Purpose                            | Important Columns            |
| ----------------- | ---------------------------------- | ---------------------------- |
| projects          | Stores project timeline and budget | start_date, end_date, budget |
| employee_projects | Maps employees to projects         | project_id, employee_id      |
| employees         | Stores employee salary             | id, salary                   |

---

# 3. Clarify the Grain

* `projects`: one row per project.
* `employee_projects`: one row per employee-project assignment.
* `employees`: one row per employee.
* Intermediate aggregation: one row per project with total salary.
* Final output: one row per project.

---

# 4. Identify Relationships

* `projects.id = employee_projects.project_id`
* `employee_projects.employee_id = employees.id`

One project can have many employees, and one employee can work on many projects.

---

# 5. Determine the Driving Table

**Driving table:** `projects`

Reason: the final output is project-level and must include all projects.

---

# 6. Think About Join Type

Use **LEFT JOINs**:

* `projects → employee_projects`
* `employee_projects → employees`

Reason: projects with no assigned employees should still appear in the output.

---

# 7. Find the “No Match” Condition

Projects without employees will have `NULL` salaries. The query handles this using:

```sql
COALESCE(salary, 0)
```

to treat missing salaries as zero.

---

## 8. Interpret Special Conditions / Notes

Estimated salary cost is calculated as:

```text
(project_days / 365) × total_salary
```

Assumption: annual salary is distributed evenly across the year.

---

## 9. Final Calculation Logic

1. Calculate project duration using `DATEDIFF`.
2. Sum employee salaries for each project.
3. Convert project days into a yearly fraction.
4. Multiply by total salary.
5. Compare with project budget.
6. Assign forecast label.

---

## 10. SQL Solution

```sql
SELECT 
    title,
    CASE 
        WHEN CAST(project_days AS DECIMAL(10,2)) / 365 * total_salary > budget 
        THEN 'overbudget'
        ELSE 'within budget'
    END AS project_forecast
FROM (
    SELECT 
        p.title,
        DATEDIFF(p.end_date, p.start_date) AS project_days,
        p.budget,
        SUM(COALESCE(e.salary, 0)) AS total_salary
    FROM projects AS p
    LEFT JOIN employee_projects AS ep
        ON p.id = ep.project_id
    LEFT JOIN employees AS e
        ON e.id = ep.employee_id
    GROUP BY p.title, project_days, p.budget
) AS temp;
```

---

## 11. Output Explanation

| Column           | Meaning                  |
| ---------------- | ------------------------ |
| title            | Project name             |
| project_forecast | Forecasted budget status |

---

## 12. Key SQL Concepts Used

* `LEFT JOIN`
* `SUM`
* `COALESCE`
* `DATEDIFF`
* `CASE WHEN`
* Derived table (subquery)
* `GROUP BY`

---

## 13. Explanation

I started from the projects table so that all projects were retained. I joined employee assignments and salaries, aggregated total salary per project, estimated labor cost using project duration, and then used a `CASE` expression to classify each project as overbudget or within budget.
