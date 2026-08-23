![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Top 5 Actions During Thanksgiving Week

# Business Context

This analysis is commonly used in **product analytics and user engagement reporting**. Product teams want to understand which actions users perform most frequently during important seasonal periods such as Thanksgiving week.

The insights can help teams:

* identify the most engaging product features,
* measure holiday-week user behavior,
* prioritize feature improvements,
* plan seasonal marketing campaigns,
* monitor unusual spikes in platform activity.

Ranking actions also allows stakeholders to quickly identify the most popular interactions during the specified period.

---

# Problem Statement

The events table tracks every time a user performs an action on the platform.

Write a query to determine the **top 5 actions performed during Thanksgiving week (2020-11-22 to 2020-11-28)** and rank them by the number of times performed.

Requirements:

* Include `action` and `ranks`.
* Higher action count should receive a better rank.
* Actions with the same count should receive the same rank.

---

# Input Tables

## Table: events

| Column     | Type     |
| ---------- | -------- |
| user_id    | INTEGER  |
| created_at | DATETIME |
| action     | VARCHAR  |
| platform   | VARCHAR  |

---

# Expected Output

| Column | Type    |
| ------ | ------- |
| action | VARCHAR |
| ranks  | INTEGER |

---

# 1. Understand the Requirement

We need to:

1. Consider only events between **2020-11-22 and 2020-11-28**.
2. Count how many times each action was performed.
3. Rank actions by count in descending order.
4. Assign the same rank to ties.
5. Return the top 5 ranked actions.
6. Display results in ascending rank order.

---

# 2. Understand the Tables

| Table  | Purpose                     | Important Columns  |
| ------ | --------------------------- | ------------------ |
| events | Stores user activity events | created_at, action |

---

# 3. Clarify the Grain

* `events`: one row per user action event.
* Intermediate aggregation: one row per action.
* Final output: one row per ranked action.

---

# 4. Identify Relationships

No joins are required because all required information exists in the `events` table.

---

# 5. Determine the Driving Table

**Driving table:** `events`

Reason: action activity and timestamps are stored directly in this table.

---

# 6. Think About Join Type

No join is required.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rules:

* Date range is inclusive: **2020-11-22 through 2020-11-28**.
* Ranking should treat ties equally.
* `DENSE_RANK()` is appropriate because it assigns the same rank to equal counts without gaps.

---

# 9. Data Quality Considerations

Potential considerations:

* Multiple users performing the same action are all counted.
* Duplicate event rows, if present, would increase counts because each row represents an event occurrence.
* Null actions are not addressed in the problem statement.

---

# 10. Final Calculation Logic

1. Filter events to Thanksgiving week.
2. Count occurrences of each action.
3. Rank actions using `DENSE_RANK()` ordered by descending count.
4. Sort by rank.
5. Return the first five ranked rows.

---

# 11. SQL Solution

```sql id=
WITH action_counts AS (
    SELECT
        action,
        COUNT(*) AS action_count
    FROM events
    WHERE DATE(created_at) BETWEEN '2020-11-22' AND '2020-11-28'
    GROUP BY action
)
SELECT
    action,
    DENSE_RANK() OVER (ORDER BY action_count DESC) AS ranks
FROM action_counts
ORDER BY ranks
LIMIT 5;
```
