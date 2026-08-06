# Percentage of Users Who Were Data Analysts Immediately Before Becoming Data Scientists
# Business Context

This analysis is commonly used in **workforce analytics, talent management, and career-path analysis**. Organizations want to understand typical promotion or transition paths between roles.

The metric helps HR and leadership teams:

* identify common career progressions,
* evaluate internal promotion pipelines,
* plan training programs,
* measure transition rates from analyst to scientist roles,
* benchmark career development outcomes.

The key business rule is **immediacy**: the employee must move directly from *Data Analyst* to *Data Scientist* without any other role in between.

---

# Problem Statement

Determine the percentage of users who held the title **Data Analyst immediately before Data Scientist**.

Return the percentage as a single value.

---

# Input Tables

## Table: user_experiences

| Column        | Type     |
| ------------- | -------- |
| id            | INTEGER  |
| position_name | VARCHAR  |
| start_date    | DATETIME |
| end_date      | DATETIME |
| user_id       | INTEGER  |

---

# Expected Output

| Column     | Type  |
| ---------- | ----- |
| percentage | FLOAT |

---

# 1. Understand the Requirement

We need to:

1. Order each user’s job history chronologically.
2. Identify rows where the current role is `Data Scientist`.
3. Check whether the immediately preceding role is `Data Analyst`.
4. Count distinct users satisfying this condition.
5. Divide by the total distinct users.

---

# 2. Understand the Tables

| Table            | Purpose                 | Important Columns                  |
| ---------------- | ----------------------- | ---------------------------------- |
| user_experiences | Stores user job history | user_id, position_name, start_date |

---

# 3. Clarify the Grain

* `user_experiences`: one row per user-role experience.
* Intermediate result: one row per experience with previous role attached.
* Final output: one aggregated percentage value.

---

# 4. Identify Relationships

No joins between different tables are required because all information exists in a single table.

---

# 5. Determine the Driving Table

**Driving table:** `user_experiences`

Reason: job history and role sequence are stored directly in this table.

---

# 6. Think About Join Type

No join is required for the core logic.

The final join in the provided solution is used only to access the total user count, but it is not necessary in an optimized version.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rule:

> Immediate means there is **no other role between Data Analyst and Data Scientist**.

This is best implemented using the `LAG()` window function ordered by `start_date`.

---

# 9. Data Quality Considerations

Potential considerations:

* Experiences should be ordered by `start_date`.
* Overlapping jobs may create ambiguous sequences.
* Duplicate experience records could affect results.
* Users with only one role cannot satisfy the condition.

---

# 10. Final Calculation Logic

1. Sort experiences by user and start date.
2. Use `LAG()` to get the previous role for each experience.
3. Keep rows where current role = `Data Scientist` and previous role = `Data Analyst`.
4. Count distinct qualifying users.
5. Divide by total distinct users.
6. Return the percentage.

---

# 11. Improved SQL Solution

```sql id=
WITH added_previous_role AS (
  SELECT user_id, 
         position_name,
         LAG (position_name) OVER (PARTITION BY user_id) AS previous_role
  FROM user_experiences
),
 
experienced_subset AS (
  SELECT *
  FROM added_previous_role
  WHERE position_name = 'Data Scientist' 
    AND previous_role = 'Data Analyst'
)
 
SELECT COUNT(DISTINCT experienced_subset.user_id)/
       COUNT(DISTINCT user_experiences.user_id) 
AS percentage
FROM user_experiences
LEFT JOIN experienced_subset 
    ON user_experiences.user_id = experienced_subset.user_id
```
